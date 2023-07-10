---
title: "Webフロントエンドに Integration Test を導入するに至った諸々の話"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["integration", "Storybook", "frontend"]
published: false
---

こんにちは、アルダグラムのエンジニアの影山です。

今回は KANNA（弊社アプリ）の Webフロントエンドに Integration Test を導入するにあたっての背景などを話そうと思います。

## Integration Test 導入によって達成したいこと

いきなり Integration Test を導入するとなっても、モチベーションとなる材料が薄ければ、おそらく途中で挫折してしまうでしょう。

ロジックのテストも増やしていきたいのに、何故さらにテスト構成を増やすのか。

弊社では、下記のような項目を達成したい動機がありました。

- フロントエンドのモジュラーモノリス化
- Presentational Component を実現するリファクタリングの実施前に、結合テストによる安全性を担保したい。
- Integration Test が通っていることで、リリースブロック判定を１段階緩めたい

1つずつ簡単に紹介します。

### フロントエンドのモジュラーモノリス化

背景は下記から。

[Turborepoライクで始めるモジュラーモノリス](https://zenn.dev/aldagram_tech/articles/cd3b961f84db42)

現状のモノリシックなフロントエンドの構成だと、依存関係やコードの見通しがネックになりそう、という話ですね。

マイクロフロントエンドだとリモデリング、CI/CD の再整備など移行コストが高そうなので、モジュラーモノリス化が実現できそうであれば(個人的には)賛成です。

Integration Test を導入することで依存関係によるエラーなどの検知は、少し改善されるかもしれません。

### Presentational Component を実現するリファクタリングの実施前に、結合テストによる安全性を担保したい。

現状の KANNA では UI とロジックが1つのファイルに記載されているため、処理が見えづらいということが挙げられます（DnD など仕様自体が複雑だったり）。弊社では、今後見通しを良くするために上記のリファクタリングを実施していきたいと考えています。

が、その前にテストによる安全性を担保すべきだよね、という話になり動機の1つに置きました。

### Integration Test が通っていることで、リリースブロック判定を１段階緩めたい

KANNA では E2E に [MagicPod](https://magicpod.com/)、 CSSスタイリングツールに `material-ui` を利用しているのですが、 これを v4 → v5 にするにあたって E2E が9割落ちるという自体に陥ってしまいました💦。

`material-ui` が [emotion を採用したり](https://www.notion.so/Web-Integration-Test-b91975e84f1940d18fb1689ab302e200?pvs=21)して内部状態の破壊的変更が多く仕方のないことではあるのですが、これによるE2Eの修正が大変でした。

 Integration Test が十分に備わっていたらリリースブロック判定を1段階緩められるカモと考えたため、動機の１つに置きました。

上記を踏まえ、チームの合意も取ることもできたので導入に至りました。

## 現状のフロントエンドテスト構成

フロントエンドではよく [The Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) がテスト構成の理想像として共有されますが、KANNA のフロントエンドでも、最終ゴールとして結合テストが一番多くなるような比重で組めたら良いなと考えています。

今現在だと、 KANNA では下記のようなテスト構成になっております。

| 静的解析 | Prettier |
| --- | --- |
| 静的解析 | ESLint |
| 静的解析 | TypeScript |
| ユニットテスト | Jest |
| Visual Regression Test | Storybook(Storycap) |
| E2E | MagicPod |

世の中では、割と Jest で Integration Test を書くサンプルが多いので、「あとは Integration Test を Jest を使って拡充するだけかな？」と、私は思っていました（実際に Integration Test を Jest に組み込む記事もいくつか拝見しました）。

が、最終的に Integration Test には Storybook の [integrations ページで紹介されている](https://storybook.js.org/integrations/) **Test runner** を採用することに決めました。

## なぜ Test runner なのか

そもそも、導入にあたって3つほど検討しておりました。

1. test-runner
2. stories ファイル を import した spec ファイル を作り、それをテストする
3. jest のみで integration test を書く

その中でも、test-runner は

1. 既に KANNA で作成された pages の stories ファイルに対してテストが追記できる。
2. stories ファイル上で完結するため、ファイル数自体は増えない。
3. CIにも組み込める
    1. Chromatic を使うこともできるが、5000スナップショット/月 以上だと有料化される。
4. ドキュメントは少ないが、内部が playwright x jest なので身構える必要はそこまで無い。
5. ブラウザで視覚的に確認できるため、エラー箇所に気づきやすい。

といった理由があり採用に至りました。

導入決定に至るまでにあたって、 Cybozu さんの記事を非常に参考にさせていただいておりました。

[Storybook をフル活用してテストを実装した話 - Cybozu Inside Out | サイボウズエンジニアのブログ](https://blog.cybozu.io/entry/2023/05/29/090000)

また、モックに関してですが、KANNA では既に `msw` を使って GraphQL のリクエストをインターセプトしている実装が出来上がっていたのでそこまで障壁とはなっておりません。

この動画は実際にテストを書いた時の Interactions(右側) の表示と、言語別で動いていることの確認です。

![](/images/kanna-integration-test/integrationMov.gif)

`npm run test-storybook` 実行時のテスト結果です。

![testRunner.png](/images/kanna-integration-test/testRunner.png)

そもそも Test runner が何をテストするのか？ですが、ボリューミーになるので、この記事を参照すると分かりやすいです。

[Storybookのテストランナー](https://zenn.dev/makotot/articles/b0729488282148#何をテストするものか)

## まとめ

以上になります。導入にあたって、チーム内外の方々と話しながら進めてきました。

今後は Test runner に決定してよかったなと思えるように日々テストと格闘していこうと思います。
