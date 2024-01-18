---
title: " [Firestore] CLIでcollection groupのルール・インデックス設定ができるまでに必要なこと"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Firestore", "collectiongroup", "Tech"]
published: false
---
こんにちは！[アルダグラム](https://aldagram.com/about/)でエンジニアをしている[kageyama](https://zenn.dev/kouschatten)です

Firestore の collection group を利用して、複数の collection に跨った batch 処理を対応する機会がありました。
今回、collection group を扱えるところから、firestore.indexes.jsonの設定方法などハマった箇所があったので、その知見をまとめていきます。
## collection group とは

> collection group は、同じ ID を持つすべてのコレクションで構成されています。
デフォルトで、クエリは、データベース内の単一コレクションから結果を取得します。
単一コレクションからではなく、コレクショングループからドキュメントを取得するには、
コレクショングループクエリを使用します。

https://firebase.google.com/docs/firestore/query-data/queries?hl=ja#collection-group-query

正直、最初読んだ時よく分からなかったです。
例えばですが
- users collection が複数の user document を持っている
- user document が単一の bookmarks collection を持っている
- bookmarks collection が複数の bookmark document を持っている

を前提として、collection group を使うことで条件に合致した bookmarks collection を全ユーザーから取得できます。 collection では、ユーザーを縦断してデータが取得できないのです（collection の場合、ループで回す必要が出てきます）。

![image1.png](/images/firestore-collection-group-cli/image1.png)

この機能は非常に強力であるため、クエリの条件を狭めて指定する必要がありますね。
以降、上記を前提として「全ユーザーのブックマークを取得したい」ことをベースに話していきます。

## collection group 利用で必要なこと

collection groupを利用するには、

1. インデックスを設定する
2. Web SDK, モバイル SDK の場合、ルールの追加

が、必要になります。
1.は Firebase の GUI 上で Firestore DB に飛んでいただきインデックスタブで設定するか、 firestore.indexes.json で CLI から設定するか、2パターンがあります。
2.はインデックス追加後にルールを制定する必要があります。こちらも GUI or CLI の2パターンがあります。

今回は、それぞれ CLI での説明をしていきますが、CLI の説明自体は省くため、公式ドキュメントを見ていただけると良いと思います。

https://firebase.google.com/docs/cli?hl=ja

## インデックスを設定する

### firestore.indexes.json の準備

Firestore のインデックス設定を CLI で行うためには、`firestore.indexes.json` ファイルを作成する必要があります。
ブックマークの例に基づくと、bookmarks の collection group に対するインデックスを設定する場合、以下のような内容になるでしょう。

```jsx
{
  "indexes": [
    {
      "collectionGroup": "bookmarks",
      "queryScope": "COLLECTION_GROUP",
      "fields": [
        { "fieldPath": "createdAt", "mode": "ASCENDING" },
        { "fieldPath": "title", "mode": "ASCENDING" }
      ]
    }
  ],
  "fieldOverrides": []
}
```

ここでは、 bookmarks collection group に対して **`createdAt`** と **`title`** フィールドに基づく昇順のインデックスを設定しています。

こちらをデプロイすることで、Firestore DB のインデックスタブにも反映されます。

イメージこんなです。

![image2.png](/images/firestore-collection-group-cli/image2.png)

CLI を使えば、GUI でのマニュアル設定が必要ないため、複数環境に適用させたい場合、手間が無くなるので良いですね！

### fieldOverrides の設定

ただ、上記の設定だけでは、特定の条件に合致した collection group を全ユーザーを縦断して取得することができないことがあります。これは、Firestore の自動インデックス設定によるもので、無効にしたいインデックスが有効になっていたり、その逆が発生していることがあるからです。

その場合、インデックス側で、 [fieldOverrides](https://firebase.google.com/docs/reference/firestore/indexes/?hl=ja#fieldoverrides) セクション（GUI上では除外という項目）を使用することで、特定のフィールドに対するインデックスをカスタマイズできるため、それを利用することで、適切なインデックス設定が可能になります。

例（bookmarks コレクションに views というフィールドがあると仮定し、そのインデックスをカスタマイズしたい場合）

```jsx
"fieldOverrides": [
    {
      "collectionGroup": "bookmarks",
      "fieldPath": "views",
      "indexes": [
        {
          "order": "ASCENDING",
          "queryScope": "COLLECTION_GROUP",
          "arrayConfig": "CONTAINS"
        },
        {
          "order": "DESCENDING",
          "queryScope": "COLLECTION_GROUP"
        }
      ]
    }
  ]
```

上記を、CLI からデプロイしていただければ、こちらも勝手に反映されます。
イメージはこんなです（ブログ用なので、ご了承ください）。

![image3.png](/images/firestore-collection-group-cli/image3.png)

除外(単一フィールドインデックス) に関しては、以下に詳しくまとめられていますhttps://firebase.google.com/docs/firestore/query-data/index-overview?hl=ja#single-field_index_exemptions

## ルールの制定

最後にルールです。今回は全ユーザーのブックマークを取ってくるので、 ルール側でもワイルドカード(`*` )を適用する必要があります。
collection group を正しく使うためには、今回であればこんな感じになると思います。

`firestore.rules`

```jsx
match /users/{uid=**}/bookmarks/{bookmarkId} {
  allow read, create, delete: if userIsDataOwner(uid);
}
```

全ユーザーが対象となるため、ワイルドカードの位置は `{uid=**}` になるのがポイントです。こちらも CLI 上からデプロイするだけで、ルールが反映されます。楽ちん。
これで、インデックス・ルールの準備ができたので、実際に batch 処理などが動くようになるかと思います！

イメージ：

```jsx
export class UserBookmarksRepository {
  private readonly fireStore: FirebaseFirestore.Firestore;

  constructor(
    firestore: FirebaseFirestore.Firestore,
  ) {
    this.fireStore = firestore;
  }

  fetchCollectionGroup = async (fieldPath: "views", document: string): Promise<BookmarkCollectionGroupType> => {
    const collectionGroup = await this.fireStore.collectionGroup(bookmark
      .where(fieldPath, "==", document)
      .get()
    return collectionGroup
  }
}

呼び出し側：
const userBookmarkRepo = new UserBookmarksRepository(firestore)
const fetchUserBookmark = userBookmarkRepo.fetchCollectionGroup("views", document) // fetchUserBookmarkが、対象のブックマークを全ユーザーから取得できる
userBookmarkRepo.updateHoge(fetchUserBookmarRoomName, Hoge)
```

## まとめ

`firestore.indexes.json`,  `firestore.rules`  ****を作ってあげることで、CLI を通じて簡単かつ効率的にインデックスを管理・デプロイできます。
dev, prod などの環境にいちいち設定する必要が無くなり、マニュアル設定によるミス防止も防げるので、個人的には CLI で管理してあげると凄く良いなーと触ってて思いました（所感）。
