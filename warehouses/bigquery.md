---
title: Google BigQuery
subtitle: Authentification, configuration options, and content for BigQuery.
priority: 1
---

【前略】

### BigQuery におけるパーティショニングされたインクリメンタルテーブルの最適化

BigQuery ではインラインの SELECT 文から計算される `WHERE` 文を最適化できないため、BigQuery でパーティショニング テーブル から インクリメンタル テーブル を作成する際には全テーブルのスキャンを避けるために多少の工夫が必要になります。

回避策としては、WHERE 句で使用される値を `pre_operations` ブロックに移動して [BigQuery スクリプト](https://cloud.google.com/bigquery/docs/reference/standard-sql/scripting) で変数に保存する方法があります。

このパターンを利用した簡単な例を以下に示します。ここではソーステーブル `raw_events` が既に `event_timestamp` 列でパーティショニングされているものとします。

```sql
config {
  type: "incremental",
}

pre_operations {
  declare event_timestamp_checkpoint default (
    ${when(incremental(),
    `select max(event_timestamp) from ${self()}`,
    `select timestamp("2000-01-01")`)}
  )
}

select
  *
from
  ${ref("raw_events")}
where event_timestamp > event_timestamp_checkpoint
```

これで、新しい行を挿入するときには最新のパーティションだけを見るようにすることで `raw_events` に対する全テーブルスキャンを回避しています。

【後略】
