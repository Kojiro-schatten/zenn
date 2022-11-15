---
title: "[N番前じ]AWS SAA 2週間で合格した"
emoji: "🙌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "saa"]
published: true
---

## はじめに
AWSの実務経験が無い状態で（CLFを飛ばして）AWS SAAに合格しました。短期的な集中勉強のストレスか、吹き出物も出たので、もう少しじっくり勉強するのが吉だとは思います。
この記事では、どんな教材を使ったか、どう勉強したか、を振り返りつつ書いていきます。

## 私のスペック
・主にフロントエンド領域を触っている(2~3年)
・AWSで触ったことがあるのはCodeCommitとか、業務でコードプッシュするときに使うもの位

## 勉強期間
・14日間（60時間強）
・平日 2.5時間
・休日 8時間

## 試験当日
・自宅受験
・130分フルタイム使って、じっくり解きました
・PearsonVueのローディング時間が長かった

## 使用教材

https://www.amazon.co.jp/gp/product/4815607389/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1
とにかく全体概要を掴みたいと思ったため、購入。
最初の3日間ぐらいで1周して、2周目はルーズリーフに書いたり、じっくり読み書きして内容を頭に入れました。
古い情報があるので、そこだけ注意。

https://www.udemy.com/course/aws-associate/
この教材は、最初の一週間のみ使いました。
実際に、AWSを触るため、AWS実務経験が無い方にオススメです。
VPCの章までは実際に触ったりして進めていました。途中から時間が無かったため、進めませんでした。

https://www.udemy.com/course/aws-knan/
1週間目の休日くらいからガッツリ使いました。
2〜3周しました。一番時間を割いていたと思います。
ニッチな内容も入っていますが、試験本番ではそういった部分は一切出なかったです。取り上げられる内容の多いEC2, VPN, EFS, CloudFrontを使ったアクセス処理などの部分を重点的に仕上げていけば良いと思います。

## 本番
難易度は肌感けっこう難しいな、、、という印象でした。
[udemy](https://www.udemy.com/course/aws-knan/)の高難易度問題と大差いないと思います。
ただ、基本的なアーキテクチャ設計やVPNの設定等が問われることが多いため、やるべきことをキチンとやれば受かると思います。
また、印象強かったのは、暗号化系、EFSなどのファイルの取り扱い、CloudFrontの設定などです。Mac使いだからか、EFSのwindowsタイプが把握しづらかったので、Macerの人は要注意です。