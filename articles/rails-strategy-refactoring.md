---
title: "【Rails】Strategy パターンを使って、リファクタリングをしてみる"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rubyonrails", "designpattern", "strategy"]
published: true
---

こんにちは！影山です。
KANNAのグローバルチームでは、日々多言語化に向けて、

[少しでも世界で使われる可能性があるならDBのタイムゾーンはUTCにしよう！](https://zenn.dev/aldagram_tech/articles/b778a07304c52b)

[通知の言語を Web と App で分けた話](https://zenn.dev/aldagram_tech/articles/b649d98729e182)

の記事のように、機能開発や基盤作りをしております。
今回は、多言語化する上でAPI側で言語ごとにファイルを生成していた箇所のリファクタリング記事になります。

## 背景
リファクタリングに至る前に。
KANNAでは現在（2022年11月13日）日本語、英語、タイ語をサポート言語対象としています。
対応する言語が増えると

１：ファーストネームが先か、ラストネームが先か

２：日付の並び順（2022-01-13 か、13-01-2022か）

など、各言語で項目の形が変える必要が出てきます。
今回の実装部分では、上記を解決するために、言語ごとに生成すべき項目をファイル単位で実装していることが分かっていました。
そのため、生成すべき項目が変わる部分のみ Strategy パターンを使って改修にあたりました。

## 実装方針

丸々ファイルを増やさないためのアプローチは様々にありますが、今回は、あるメソッドの処理内容が、日本と他国でフォーマットの種類が違うのみで、他のメソッドは同じ処理でした。
そこで

１：言語が違っていても共通化できるメソッドは、共通化する。

２： 日本と他国とでエクセル項目の出力が違うため、その部分のみ「同じ箱だが中身の処理は入れ替えられる形」にする。

３：２を適用するにあたって、 `Strategy` パターンで改修していく。

のような方針で進めていこうと思いました。

## Strategy パターンを使う

では、その Strategy パターンについてですが、その説明に関してはとても参考になったQiitaの記事を添付しました。

[4章 アルゴリズムを交換する : Strategy](https://qiita.com/yuji_ariyasu/items/588fef6062b3c7149509#4%E7%AB%A0-%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0%E3%82%92%E4%BA%A4%E6%8F%9B%E3%81%99%E3%82%8B--strategy)

各役割としましては、
Context：Strategy を呼び出すことで、中にある ConcreteStrategy のメソッドが扱える

Interface：インターフェース

ConcreteStrategy：インターフェースの中身。具体的な処理が入る

となっています。

![](/images/rails-strategy/image_01.png)

そのため、呼び出し側（Context）では `Strategy.new(ConcreteStrategy1.new)` のような形で生成すると、必要に応じて `ConcreteStrategy` のメソッドを利用できます。

なぜ、利用できるかというと、それは `Interface` の役割を担っている `Strategy` が、実際の処理を `ConcreteStrategy` （今回なら ConcreteStategy1 の方）に委譲しているからです。

今回の実装も、上記に倣って、下記のような状態で利用しました。

![](/images/rails-strategy/image_02.png)

コードベースだとこんな形。日本か、それ以外の国かでフォーマットを分ける処理に適用しました（だいぶ簡略化しています）。

# Context
```
def generate_customized_cm_inputs(section, cm_template_defs)
  template = localize_data_import_template(cm_template_defs)

  # 実際に、ConcreteStategyのメソッドを、条件別で呼んでいる
  # 日本か、他国か、で提供されているメソッド名は同じなので、ここで言語を考える必要は無い
  case section.name
  when CustomizedCmSection::BUILT_IN_NAME[:project]
    template.customized_cm_section_for_project
  when CustomizedCmSection::BUILT_IN_NAME[:client]
    template.customized_cm_section_for_client
	# other method...,
  end
end

def localize_data_import_template(cm_template_defs)
  # インスタンス生成時に、日本か、他国か、で扱うフォーマットを変える。
  if @lang == Internationalization.support_language('ja')
    BuiltinSectionInputs.new(BuiltinSectionInputsJa.new, cm_template_defs)
  else
    BuiltinSectionInputs.new(BuiltinSectionInputsGlobal.new, cm_template_defs)
  end
end
```

```
# Strategy
class BuiltinSectionInputs
  attr_accessor :formatter, :cm_template_defs

  def initialize(formatter, cm_template_defs = nil)
    @formatter = formatter
    @cm_template_defs = cm_template_defs
  end

  def customized_cm_section_for_project
    @formatter.customized_cm_section_for_project(cm_template_defs)
  end

  def customized_cm_section_for_client
    @formatter.customized_cm_section_for_client
  end
end
```

```
# ConcreteStrategy1
class BuiltinSectionInputsJa
  def customized_cm_section_for_project(cm_template_defs)
    hoge
  end

  def customized_cm_section_for_client(cm_template_defs)
    huga
  end
end
```

```
# ConcreteStrategy2
class BuiltinSectionInputsGlobal
  def customized_cm_section_for_project(cm_template_defs)
    hogeGlobal
  end

  def customized_cm_section_for_client(cm_template_defs)
    hugaGlobal
  end
end
```

上記のような形にした結果、

- Context 側では、日本と海外で使われている共通な処理のメソッド、は `input_guides.rb` 内でまとまった（上記のサンプルには載せてないです）
- 言語ごとにファイルを生成していた部分を、最小限に留められた
- 委譲ベースなので、継承関係が無いことから、依存性が低くなった
- クラスの拡張をせず、振る舞いの変更が可能となった

のような結果を得られました。

ただ、インターフェースを組んだことで、「本当にメソッドを正しく呼べているかどうか」という部分をテストコードで確実に担保しないと信用性が落ちることも分かりました（先輩からのご指摘）。

なるほど、デザインパターンが銀の弾丸ではない、ことも学べられました。ただ、静的言語であればまた違った結果が得られるため、次は TypeScript でデザインパターンの演習を行おうと思っています。

## Rails の OSS コードにも組み込まれていた Strategy パターン

記事をまとめている中で、改めてパターンを調べていたら Rails のコードでも組まれていることを見つけました。

[Rails：Introduce "Execution Strategy" object for Migrations](https://github.com/rails/rails/pull/45324)

[週刊Railsウォッチ: マイグレーションをStrategyパターンで拡張可能にほか（20220704前編）](https://techracho.bpsinc.jp/hachi8833/2022_07_04/119289)

PR の中では、ユーザーがカスタムストラテジーを定義することで、マイグレーション時にユーザーのカスタムメソッドが扱える、というなんとも便利な形で紹介されていました。

カスタムストラテジーを使わない場合は、 Rails が持っているデフォルトのメソッドが呼ばれるのも変更が少なくて嬉しいですね。

デザインパターンは作られてからだいぶ日が経っていますが、まだまだ扱われていることも分かりました。

今後も継続的に学習を進めていきます。
