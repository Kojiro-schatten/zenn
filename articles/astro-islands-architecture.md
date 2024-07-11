---
title: "Astro Islands Architectureを理解したい"
emoji: "🏝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["astro", "アイランドアーキテクチャ"]
published: false
---

こんにちは、アルダグラムのエンジニアの影山です。

以前、弊社では [Next.jsでSSGしてみよう](https://zenn.dev/aldagram_tech/articles/4c89e772ceddcb)という記事で、Gatsbyのアップデートの鈍化から、技術移行も踏まえた検証が必要というお話がありました。
この記事でもそれに関連して、 Astro と SSG について書こうと思ったのですが、その前に Astro が提供する Astro Islands Architecture について理解していなかったので、そちらを先に調査しブログにまとめることとしました。
調査内容であることに御留意いただけると幸いです。

## Astroとは

そもそも Astro とは？ですが、[公式ページ](astro.build) では、最初に
`The web framework for content-driven websites. Astro powers the world's fastest marketing sites, blogs, e-commerce websites, and more.` とあり、画像/文章/動画/音声などをメインに扱うサイト（ブログやeコマース）に対して、効率的に構築するためのWebフレームワークと言えます。

また、コアコンセプトのうち、[インテグレーションの拡張](https://astro.build/integrations/)や[別フレームワークをサポートする](https://docs.astro.build/ja/guides/framework-components/)など、カスタマイズ性も大きなメリットです。
そして、私が気に入ったのが、[簡単に使える](https://docs.astro.build/ja/concepts/why-astro/#%E7%B0%A1%E5%8D%98%E3%81%AB%E4%BD%BF%E3%81%88%E3%82%8B)という部分です。
新たなフレームワークを導入する際、導入コストが高いとそれだけで不採用になり得ますが、 Astro では `.astro` 拡張子であれば、それは HTML のスーパーセットであるため、HTML を書けることがそのまま Astro コンポーネントも書けるということになります。
加えて、JSX、Svelte、など、他フレームワーク製のコンポーネントも `.astro` 内であれば、import して利用できるため、Astro への移行がスムーズになりやすい、というのもポイントが高いです。
これら以外に Astro を普及させる要因として [Server First](https://docs.astro.build/ja/concepts/why-astro/#server-first) や、[Astro Islands Architecture](https://docs.astro.build/ja/concepts/islands/) 等が存在します。
早速、Astro Islands Architecture を見ていきたいですが、その前に Islands Architecture そのものについて触れていこうと思います。

## Islands Architectureとは

Islands Architecture は、Etsy社の Katie Sylor-Miller によって提唱されました。
https://jasonformat.com/islands-architecture/
このアーキテクチャのアイデアはシンプルで、サーバー側でHTMLページをレンダリングし、動的な領域にはプレースホルダー/スロットを挿入する、というものです。
ここで言う、プレースホルダー/スロットとは、サーバー側でレンダリングされたHTMLページ内で、動的な領域（例えば、ウィジェットやインタラクティブなコンポーネント）が配置される場所を示しています。
プレースホルダー/スロットには、対応するウィジェットのサーバー側レンダリング出力が含まれており、クライアント側でハイドレーションされることで、インタラクティブな機能を持つ独立したウィジェットになります。

下記の画像では、1枚のページの中に静的UIと、インタラクティブなコンポーネントが混じっていることを図示したものです。各コンポーネントが島のように独立しており、動的領域はプレースホルダー/スロットとして配置されていることが分かります。

![image2.png](/images/astro-islands-architecture/image2.png)
*https://jasonformat.com/islands-architecture/ から引用の上、注釈を追加*

## Islands Architectureを採用したフレームワークはいくつも存在する
私は最初、Astro と Islands Architecture がセットで言及されていることが多いために、 Astro と Islands Architecture 自体が根強く結びついていると思っていましたが、Astro がこのアーキテクチャを採用しただけで逆の関係は成り立たない、というのが理解できました。
他のフレームワークでは、例えば [Fresh](https://fresh.deno.dev/) などもこのアーキテクチャを採用しています。他にも採用例がいくつかあり、こちらで確認ができます。

https://github.com/lxsmnsyc/awesome-islands

それでは Astro Islands Architecture を見ていきます。

## Astroが描くIslands Architecture
Astro は、上記のアーキテクチャに加え、islands に対する柔軟なサポートをしております。
それは、React や Svelte といったフレームワークへのマルチな共存対応です。1ページ上に複数のフレームワークの共存が可能になり、任意のタイミングでハイドレーションを行えます。island は、他の island と干渉せず、常に独立して動作しお互いの状態を共有することができるため、複数のフレームワークの共存が可能だということです。下記のようなイメージです。

![image4.png](/images/astro-islands-architecture/image4.png)

また、ハイドレーションのタイミングもコントロールできます。これは、特定のトリガー（例えば、ユーザーのスクロールやビューポートへの表示）に基づいて、必要な時にのみクライアントサイドで JavaScript をロードしてハイドレーションすることができるからです。

https://docs.astro.build/ja/guides/framework-components/

## island化するには？
island 化するには、 `client:*` ディレクティブを追加すれば良いです。簡単ですね。

```
<SampleComponent client:load />
```

明示的に `client:*` と書かれたインタラクティブなコンポーネントを、Astro は island として認識します。

ただ、[アイランドとは？](https://docs.astro.build/ja/concepts/islands/#%E3%82%A2%E3%82%A4%E3%83%A9%E3%83%B3%E3%83%89%E3%81%A8%E3%81%AF)の通り、

> Astroにおける「island」とは、ページ上のインタラクティブなUIコンポーネントを指します。

とあり、**静的HTMLはサーバー側でレンダリングされるが、インタラクティブな機能を持たず、クライアント側でハイドレーションされる必要がないため、island とは呼べないことが分かります。** island の定義はあくまでインタラクティブなコンポーネントであることに注意です。


## サンプルコードで見るAstro Island
最後に、コードを少し眺めて終わります。
Astro を利用していれば `.astro` 拡張子にすることで、複数のフレームワークが混在できます。
勿論 props を渡すことも可能です。

```
---
import SampleReact from '../components/SampleReact.jsx';
import SampleSvelte from '../components/SampleSvelte.svelte';
---
<div>
  <SampleReact text="from React" />
  <SampleSvelte text="from Svelte" />
</div>
```

これは、例えば Gatsby -> Astro や、 Next.js -> Astro といった移行作業の時にも使えそうです。
注意点として、[Astroでは関数をシリアライズできないため、サーバーレンダリング間でのみ機能します。イベントハンドラーでの利用はエラーが発生します](https://docs.astro.build/ja/guides/framework-components/#%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%81%ABprops%E3%82%92%E6%B8%A1%E3%81%99)。

## tips: Gatsby から Astro への移行
実際に Astro への移行がどんな感じなのか。
移行記事の例として、下記がありました。
https://loige.co/migrating-from-gatsby-to-astro/

こちらでは、背景として Gatsby のバージョンアップデートのコスト・Gatsby 自体の複雑性について不満があり、移行先に対してはすでにある既存の JSX 構文のサポート・モダン技術等をメリットと捉えて模索していたところ、Astro 4 がマッチしていたと話しています（最初の Astro pre 1.0.0 では挫折したとのこと）。
記事中の、[Migration](https://loige.co/migrating-from-gatsby-to-astro/#:~:text=successfully%20this%20time!-,Migration,-Once%20I%20was) セクションでは、Gatsby にあった既存のページとコンポーネントをすべてリストアップし、1つずつ Astro に移植する中で、デザイン周りのリプレースに時間がかかったとありますが、これもテーマやカラーパレットの確立のために必要だった時間としており、Astro に不満がある内容ではありませんでした。
なるほど。今後は、Astroの移行作業についてどんなものか、より探っていきたいと思います。


## 参考

https://astro.build/

https://zenn.dev/morinokami/articles/islands-architecture-with-astro

https://jasonformat.com/islands-architecture/

https://loige.co/migrating-from-gatsby-to-astro/
