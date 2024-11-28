---
title: "Datadog を使い、パフォーマンス計測とその改善をしていく"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["datadog", "Rails", "GraphQL"]
published: false
---

こんにちは！[アルダグラム](https://aldagram.com/about/)でエンジニアをしている[カゲヤマ](https://zenn.dev/kouschatten)です。

アルダグラム では、[**アルダグラムが使っている技術スタック（2021年 → 2023年）**](https://zenn.dev/aldagram_tech/articles/871b4dcc7b5d9c)の記事の通り、Datadog を以前から利用しています。

私は、最近まで新機能の開発をしていたのですが、パフォーマンス計測およびその改善の目処を立てるにあたって Datadog にとても助けられたので、どのように使用していたか書こうと思います。

# Datadogとは

Datadog は、SaaS 型の運用監視ツールであり、監視ツールの他、APMツール、ログ分析、ネットワーク管理、データベース管理などを、オールインワンに扱えます。

その中でも、パフォーマンスのボトルネックの特定に役立つ [APM](https://docs.datadoghq.com/ja/tracing/) を利用しました。

アルダグラムでもリクエスト単位でパフォーマンスを監視しています。GraphQL を採用しているため、GraphQLのリクエストが走ったタイミングで、そのレスポンスまでのログを良い感じに表示してくれます ↓

![1.png](/images/performance-improvement-with-datadog/1.png)

Datadog では、右から2番目にある `SQL Queries` タブでどんな SQL が発行されたか確認することができます。

![2.png](/images/performance-improvement-with-datadog/2.png)

上記では SQL Queries 11 となっており、11のクエリが動いていることが分かります。

`SQL Queries 99+` のような表示になっているとN+1 の疑いを持つため、このタブ内で同じクエリが発行されていないか確認することは、パフォーマンス改善にとても役立ちます！

# 計測方法

GraphQL のリクエスト単位で計測できることから、リリースする機能で扱う GraphQLの query/mutation を実際に計測していきます。

アルダグラムでは、[本番環境と近いデータ量を持つloadtest環境が用意されている](https://zenn.dev/aldagram_tech/articles/54195274379228)（めちゃめちゃ便利）ため、そちらにデプロイして Datadog での計測をしていきます。

この記事の中では、2つの新規の query/mutation 計測を取り上げます。

1. createHogeOshirase
2. getHogeOshirases

それぞれ、テスト環境で実際に叩いて Datadog にログを吐いていきます。

# 計測と改善

## 1. createHogeOshirase(Rails側の改修)

### **計測**

この mutation は、お知らせを作成できるもので、お知らせの対象者には他のユーザー/会社を含むことができます。そのため、ユーザーが多ければ多いほど、作成するレコードが増えていきます。

対象者が数百社ほどいる場合、下記のような結果になっていました。

![3.png](/images/performance-improvement-with-datadog/3.png)

SQL Queries が 99+ になっているので N+1 が怪しいです。

### **改善**

調べてみると、通知対象者を管理する別テーブルに対して、レコード分INSERT処理が走っていることが分かりました。

お知らせ自体の保存は `save!` でバリデーションをかけていますが、通知対象者のデータ保存に関しては `bulk_insert` を利用する決断に至りました。

その結果、下記のようなパフォーマンスになりました。

![4.png](/images/performance-improvement-with-datadog/4.png)

24200ms → 3050ms

改善前に比べて、大幅に改善されましたね⭐️

## 2. getHogeOshirases(GraphQL側の改修)

### **計測**

この query は、20件ごとでお知らせ一覧を受け取れるものです。

お知らせには、未読/既読のフラグをそれぞれ持っていますが、パフォーマンスの観点から別テーブルで未読/既読の管理を行なっています。

結果として、数は少ないものの未読/既読を取得するに当たって特定のSQLが毎回実行されており、N+1が起こっていました。

![5.png](/images/performance-improvement-with-datadog/5.png)

### **改善**

N+1が発生しているのは GraphQL の 特定の field 取得時であることが分かり、GraphQL::Batch を利用しました。

https://zenn.dev/aldagram_tech/articles/how-to-use-graphql-batch

公式ドキュメント：

https://github.com/Shopify/graphql-batch/blob/8f5b102abada9703617807cc32f68e96a7031fa2/examples/association_loader.rb

イメージとしては下記のような感じです。

```ruby
def is_read # ←特定のfield
  ::Loaders::AssociationLoader.for(
    ::HogeOshirase,
    :hoge_oshirase_histories,
  ).load(object).then do |histories|
    # 処理内容は割愛
  end
end
```

GraphQL::Batch の役割の1つとして、個々で走っているクエリを一つにまとめるバッチ処理を行うことができます。

これを採用することで、下記のような結果になりました。

![6.png](/images/performance-improvement-with-datadog/6.png)

6130ms -> 416ms

いい感じです！

今回取り上げた箇所以外にも、 Datadog を用いてパフォーマンス改善を行ないました。リクエスト単位で計測できるというのは、やはり便利ですね。

今後も駆使していきたいです。

もっとアルダグラムエンジニア組織を知りたい人、ぜひ下記の情報をチェックしてみてください！
