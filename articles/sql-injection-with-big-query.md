---
title: "BigQuery におけるクエリパラメータを利用した SQL インジェクションの改善"
emoji: "👍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["MySQL", "SQL", "Security", "BigQuery", "Rails"]
published: false
---

こんにちは！[アルダグラム](https://aldagram.com/about/)でエンジニアをしている [@kageyama](https://zenn.dev/kouschatten) です

本記事は[株式会社アルダグラム Advent Calendar 2023](https://qiita.com/advent-calendar/2023/aldagram) 12日目の記事です。

SQLインジェクションを初めて聞いた時、injectは「注射する」とかだから、SQLを直すんかーと思っていたら、全然逆で虚をつかれた記憶があります。

この記事では、SQLインジェクションの対応をご紹介したいと思います。普段開発していると、なかなか脆弱性対応に巡り合う機会がありませんよね（というよりあってはならない）。

そのため、読者がそういった機会に遭遇した際、いわゆるSQLインジェクションの典型的なシナリオと、それをどう防ぐか、 BigQuery の `query parameter` とも含めて共有したいと思います。

## はじめに

アプリケーションは GraphQL を使用しており、リクエストは全て GraphQL(Apollo) を通じてバックエンド側に渡されることとします。

前提として、下記のような機能を持つWebアプリケーションを想定します。

- アプリケーションは、検索機能を持っている
- 検索機能は、2つのリクエストパラメータを受けつけている
    - 1つ目は userId で、これはアプリケーション側から付与される
    - 2つ目は date で、これはユーザーが検索モーダルから設定できる
- バックエンド側は、受け取ったパラメータを BigQuery に渡す必要があるため、userId は BigQuery 側で where 句を入れる箇所でも利用されている。

下記のようなイメージです。

![](/images/sql-injection-with-big-query/flow.png)

## 実際にSQLインジェクションを実行してみる

続いて、実際のSQLインジェクションになりうる例を提示します。

GraphQL Apollo は、[Chrome Extension から提供されているツール](https://chromewebstore.google.com/detail/apollo-client-devtools/jdkknkkbebbapilgoeccciglkfbmbnfm)を使うことで、簡易的に  query/mutation を叩くことができます。

以下は Apollo の Extension を使って、SQLインジェクションを行ってない場合と、行った場合のリクエスト/レスポンスの例です（大分簡略化しています）。

SQLインジェクションを行ってない場合のリクエスト/レスポンス

```json
{
  operationName: “GetSearchHoge”,
  variables: {
    userId: '1',
    date: ‘2023-10-30’
  }
}

// 200 OK
{
  "data": {
    "hoge": "hoge",
    "name": "name",
  },
}
```

SQLインジェクションを行った場合のリクエスト/レスポンス

```json

{
  operationName: “GetSearchHoge”,
  variables: {
    userId: '1' OR 1' = '1';--,
    date: ‘2023-10-30’
  }
}

// 200 OK
{
  "data": [
    {
      "hoge": "hoge",
      "name": "name",
    },
    {
      "hoge": "hoge",
      "name": "name",
    },
    ...,
  ]
}
```

上記より、データベースに保存されている情報漏洩や改竄などが発生してしまうため、防ぐ必要があります。

## 原因

BigQuery では `query parameter` を提供しているため、そちらを利用することで基本的にはSQLインジェクションは解決されます。

https://cloud.google.com/ruby/docs/reference/google-cloud-bigquery/latest#query-parameters

添付リンクの通りですが、 `Standard SQL` では、位置指定または名前指定のクエリパラメータを使用できます。クエリには複数の単語を含む配列がパラメータとして渡され、この配列はクエリ内で展開されます。なので、**`params`** オプションを使用することで **`standard_sql`** が設定され安全性が高まります。これを利用することで、ユーザー入力を安全に処理しSQLインジェクションのリスクを減らすことができます。

ただし、メソッド内で動的にSQLクエリを構築する際、ユーザー入力を直接組み込むとSQLインジェクションのリスクが生じる可能性があります。

```ruby
module Bigquery
  class LogDetail
    include Bigquery::Concerns::BigqueryCommonModule

    class << self
      def where(request_id:, date:,)
        bigquery = Google::Cloud::Bigquery.new
        bigquery.query(build_query(request_id, date))
      end

      private
        def build_query(request_id, date)
          <<~EOS
            SELECT
              log.hoge,
              log.name,
            FROM log_history
            ON data_history.request_id = operation_log.request_id
            #{build_condition(request_id, date)}
          EOS
        end

        # 渡ってきたrequest_idを使い、where句を動的に生成
        def build_condition(request_id, date)
          cond_list = []
          cond_list << "log_history.request_id = '#{request_id}'"

          log_search_range = #複雑なクエリ
          cond_list << log_history_search_range

          "WHERE #{cond_list.join(' AND ')}"
        end
```

## 解決策

ユーザー入力を直接SQLクエリに組み込む代わりに、**`query parameter`** を活用して `params` を指定してあげることで、安全にクエリを構築します。

```ruby
module Bigquery
  class LogDetail
    include ActiveModel::Model
    include Bigquery::Concerns::BigqueryCommonModule

    class << self
      def where(request_id:, date:,)
        bigquery = Google::Cloud::Bigquery.new
        query_params = {
          request_id: request_id,
          date_from: (date - 1.day).beginning_of_day,
        }
        bigquery.query(build_query, params: query_params)
      end

      private
        def build_query
          <<~EOS
            SELECT
              log.hoge,
              log.name,
            FROM log_history
            // そのままWHERE句を生成するようにする
            WHERE log_history.request_id = @request_id
              AND log_history.date >= @date_from
              AND 複雑なクエリ...
          EOS
        end
```

これで無事にSQLインジェクションを防げるようになりました、ヨカッタ！

## まとめ

SQLインジェクション対策は、今回のようなアプローチに限定されるものではありません。

実際には、バインド機構の利用、対応表や許可リストによるチェックなどのさまざまな方法があります。

状況に応じて最適なアプローチを選択し、アプリケーションのセキュリティを高めたいですね。

もっとアルダグラムエンジニア組織を知りたい人、ぜひ下記の情報をチェックしてみてください！
