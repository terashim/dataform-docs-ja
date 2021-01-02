---
title: インクリメンタルなデータセット
subtitle: ここではインクリメンタルに更新されるテーブルの作成方法について学びます
priority: 2
---

（原文: [Incremental datasets | Dataform](https://docs.dataform.co/guides/datasets/incremental#conditional-code-if-incremental),
ソース: <https://github.com/dataform-co/dataform/blob/master/content/docs/guides/datasets/incremental.md>）


## 概要

インクリメンタルなデータセットは、実行時に毎回ゼロから作り直されることはありません。その代わりに、データセットの設定で定められた条件に基づいて新しい行だけが挿入（またはマージ）されます。

Dataform では状態管理、データセットの作成、`INSERT`（または `MERGE`）文の生成が自動で取り扱われます。

### 例: 単にパフォーマンス目的の追記更新

ウェブのログやアナリティクスデータはインクリメンタルなデータセットにぴったりなユースケースです。
このような種類のデータソースに対して普通は古いデータを再処理せず新しいレコードだけを処理したいはずです。

### 例: レイテンシ改善目的のマイクロバッチ処理

インクリメンタルなデータセットの主な利点はパイプラインが素早く完了できることです。そのようなパイプラインを低コストで頻繁に実行することで、利用者にとってはデータ反映までの遅延が小さく抑えられることになります。

### 例: 日次スナップショットを作成する

インクリメンタルなデータセットのよくあるユースケースとしては、入力データセットの日次スナップショット取得があります。例えば、プロダクトの DB 内にあるユーザー情報について経時データ分析をしたい場合がこれにあたります。

## 重要なコンセプト

インクリメンタルなデータセットは本質的にステートフルであり、`INSERT` 文を利用して更新されるものです。
よって、新しい行を挿入する際にはデータが消えたり重複したりしないよう保証するための工夫が必要になります。

インクリメンタルなデータセットを定義するクエリには次の２つの部分が必要になります:

- すべての行を抽出するメインのクエリ
- 差分更新の実行時にどの行が処理対象になるかを表す `WHERE` 句

## WHERE 句

この `WHERE` 句はメインのクエリに適用され、対象のデータセットへ新しいデータだけが追加されることを保証するために利用されます。例えば次のような形になります：

```js
ts > (SELECT MAX(timestamp) FROM target_table)
```

## 簡単な例

ここではタイムスタンプ付きのユーザー行動イベントを格納した `weblogs.user_actions` というデータセットがあると仮定します。このデータセットにはデータウェアハウス外部の何らかのデータ統合ツールからデータがストリーム挿入されているとします。以下ではこれをソースデータセットとします。

<table className="bp3-html-table bp3-html-table-striped .modifier" style="width: 100%;">
  <thead>
    <tr>
      <th>timestamp</th>
      <th>user_id</th>
      <th>action</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1502920304</td>
      <td>03940</td>
      <td>create_project</td>
    </tr>
    <tr>
      <td>1502930293</td>
      <td>20492</td>
      <td>logout</td>
    </tr>
    <tr>
      <td>1502940292</td>
      <td>30920</td>
      <td>login</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
  </tbody>
</table>

このデータセットを元にシンプルな差分コピーを作成するため、`definitions/example_incremental.sqlx` というファイルを新規作成します:

```js
config { type: "incremental" }

SELECT timestamp, action
FROM weblogs.user_actions

${ when(incremental(), `WHERE timestamp > (SELECT MAX(timestamp) FROM ${self()})`) }
```

まず、このスクリプトではデータセットのタイプを `incremental` としています。

その下で `when()` 関数と `incremental()` 関数を使った `WHERE` 句が書かれています。

```js
${ when(incremental(), `WHERE timestamp > (SELECT MAX(timestamp) FROM ${self()})`) }
```

この `WHERE` 句によって、ソースデータセットのうち <b>timestamp が処理済みデータ中で最新の timestamp より大きい</b> 行だけが選択されるよう保証されています。

ここで対象データセットの名前を取得するのに `self()` が使われていることに注意してください。よって、この `WHERE` 句がコンパイルされると次のように展開されます:

```sql
timestamp > (SELECT MAX(timestamp) FROM default_schema.example_incremental)
```
<br />

<div className="bp3-callout bp3-icon-info-sign" markdown="1">
BigQuery でインクリメンタルなテーブルを作る場合、テーブル全体のスキャンを避けるために少々書き方を変える必要があります。詳しくは [BigQuery に関するガイド](https://docs.dataform.co/warehouses/bigquery#optimizing-partitioned-incremental-tables-for-bigquery) をご覧ください。

</div>

このデータセットはまだデータウェアハウス上に存在しない可能性もありますが、これは問題ありません。というのも、上の `WHERE` 句は新しいデータを既に存在するデータセットへ挿入する場合にのみクエリ末尾に追加されるからです。

<div className="bp3-callout bp3-icon-info-sign" markdown="1">
  インクリメンタルなデータセットにデータが挿入されるとき、書き込まれるのは既にデータセットに存在しているフィールドだけなので注意してください。
  クエリを変更した後で新しいフィールドが書き込まれるようにするには、コマンドラインなら <code>--full-refresh</code> オプション、Dataform Web なら <code>Run with full refresh</code> オプションを有効にして一からデータセットを作り直す必要があります。
</div>

### 生成される SQL

上の例で生成される SQL の内容はデータウェアハウスの種類によりますが、基本的に同じ形式になります。

もしデータセットがまだ存在しない場合は

```js
CREATE OR REPLACE TABLE default_schema.example_incremental AS
  SELECT timestamp, action
  FROM weblogs.user_actions;
```

のような形になり、インクリメンタルに新しい行を追加するときは

```js
INSERT INTO default_schema.example_incremental (timestamp, action)
  SELECT timestamp, user_action
  FROM weblogs.user_actions
  WHERE timestamp > (SELECT MAX(timestamp) FROM default_schema.example_incremental)
```

のようになります。

ユニークキーが指定されていない場合、マージ条件（この例では `T.user_id = S.user_id`）は `false` に設定されます。このとき全ての行がマージではなく挿入されることになります。

## マージの例

<div className="bp3-callout bp3-icon-info-sign" markdown="1">
  インクリメンタルマージ機能には <code>@dataform/core</code> バージョン <code>1.5.0</code> 以上が必要です。<br>
  インクリメンタルマージ機能は現在 Azure SQLDataWarehouse ではサポートされていません。
</div>

出力先テーブルについて、いくつかのキー列の組み合わせを取った値が一意になるように保証したい場合は `uniqeKey` を指定します。
`uniqueKey` が指定されると、新しい行のキーが既存の行のキーにマッチする場合、その既存の行は新しいデータで上書きされます。

```sql
config {
  type: "incremental",
  uniqueKey: ["transaction_id"]
}

SELECT timestamp, action FROM weblogs.user_actions
${ when(incremental(), `WHERE timestamp > (SELECT MAX(timestamp) FROM ${self()})`) }
```

**BigQuery** で `uniqueKey` を使用する場合は、一部のレコードだけをチェックするよう `updatePartitionFilter` を設定するのが推奨です。
`updatePartitionFilter` が設定されていないと、マッチする行を探す際にすべての行がスキャンされてしまうので、設定することでコスト削減に繋がります。

```sql
config {
  type: "incremental",
  uniqueKey: ["transaction_id"],
  bigquery: {
    partitionBy: "DATE(timestamp)",
    updatePartitionFilter:
        "timestamp >= timestamp_sub(current_timestamp(), interval 24 hour)"
  }
}

SELECT timestamp, action FROM weblogs.user_actions
${ when(incremental(), `WHERE timestamp > (SELECT MAX(timestamp) FROM ${self()})`) }
```

### 生成される SQL

前と同様に、この例で生成される SQL はデータウェアハウスの種類によりますが、基本的には同じ形式になります。

データセットがまだ存在しない場合は

```js
CREATE OR REPLACE TABLE default_schema.example_incremental PARTITION BY Date(timestamp) AS
  SELECT timestamp, action
  FROM weblogs.user_actions;
```

のように、新しい行をインクリメンタルにマージするときは

```js
MERGE default_schema.example_incremental T
USING (
  SELECT timestamp, action
  FROM weblogs.user_actions
  WHERE timestamp > (SELECT MAX(timestamp) FROM default_schema.example_incremental) S
ON T.user_id = S.user_id AND T.action = S.action AND timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 48 HOUR)
WHEN MATCHED THEN
  UPDATE SET timestamp = S.timestamp, user_id = S.user_id, action = S.action
WHEN NOT MATCHED THEN
  INSERT (timestamp, user_id, action) VALUES (timestamp, user_id, action)
```

のようになります。

## インクリメンタルなデータセットを利用した日次スナップショット

インクリメンタルなデータセットはミュータブルな外部データセットの日次スナップショットを取るのに利用できます。

`productiondb.customers` という外部データセットがあると仮定するとき、そのデータの日次スナップショットを取るためのインクリメンタルなデータセットは次のように書くことができます:

```js
config { type: "incremental" }

SELECT CURRENT_DATE() AS snapshot_date, customer_id, name, account_settings FROM productiondb.customers

${ when(incremental(), `WHERE snapshot_date > (SELECT MAX(snapshot_date) FROM ${self()})`) }
```

- `snapshot_date` として現在の日付を取ることによって、この定義では実際上 `productiondb.customers` データセットの日次スナップショットを出力先テーブルに追記することになります。
- `WHERE` 句では挿入される行を新しい `snapshot_date` を持つものに制限することで、同じ日のデータが２回以上挿入される（出力先データセットに重複が発生する）のを防いでいます。
- この Dataform プロジェクトは少なくとも１日１回以上の頻度で実行されるようスケジュールしておく必要があります。

## インクリメンタルなデータセットをデータ損失から保護する

コマンドラインインターフェースの <code>--full-refresh</code> オプションやDataform Web の <code>Run with full refresh</code> オプションを使えば、
インクリメンタルなデータセットを一から作り直すことが可能です。

もし一から作り直されることがないように保護したい場合、例えばソースデータが後に残らないような場合は、インクリメンタルなデータセットに `protected` のマークを付けることができます。こうするとたとえユーザが full refresh オプションの要求をしたとしてもこのデータセットは削除されないようになります。

`definitions/incremental_example_protected.sqlx`:

```js
config {
  type: "incremental",
  protected: true
}
SELECT ...
```

## テーブルのパーティショニング / 分散キー

通常インクリメンタルなデータセットはタイムスタンプベースなので、後のクエリの速度を上げるため、そのタイムスタンプのフィールドでパーティショニングしておくのがベストプラクティスです。

より詳しくは、[BigQuery](https://docs.dataform.co/warehouses/bigquery) や [Redshift](https://docs.dataform.co/warehouses/redshift) のガイドを確認してください。
