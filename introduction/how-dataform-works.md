---
title: Dataform の仕組み
subtitle: ここでは Dataform がプロジェクトをコンパイルしてデータウェアハウス上で実行する仕組みを学びます
priority: 3
---

（原文: [How Dataform works | Dataform](https://docs.dataform.co/introduction/how-dataform-works),
ソース: <https://github.com/dataform-co/dataform/blob/master/content/docs/introduction/how-dataform-works.md>）

## Dataform プロジェクトの概要

各 Dataform プロジェクトは、JSON 設定ファイル、SQLX ファイル、そして場合により JS ファイルを含んだリポジトリになっています。Dataform プロジェクトに含まれるファイルには次の３つの種類があります:

- 設定ファイル
- 定義ファイル
- インクルードファイル

**設定ファイル** では Dataform プロジェクトの設定を行うことができます。一般設定ファイルでは Dataform がテーブルやビューを作成するときに使用するデータウェアハウスの種類やスキーマなどを指定できます。また、スケジュールやパッケージ、環境などのより高度なユースケースに関する設定ファイルもこれに含まれます。

**定義ファイル** は新しいテーブルやビュー、アサーション（データ品質テスト）、その他の SQL 操作などを定義するための SQLX ファイルのことです。

**インクルードファイル** はプロジェクト全体で使える変数や関数の定義を行う JS ファイルのことです。インクルードについてより詳しくは [こちらのページ](https://docs.dataform.co/guides/javascript/includes) をご覧ください。

<img src="https://assets.dataform.co/docs/introduction/sample_folder.png" width="993"  alt="Dataform project files" />
<figcaption>GitHub 上にある Dataform プロジェクト</figcaption>

## Dataform の仕組み

<img src="https://assets.dataform.co/docs/introduction/how%20it%20works.png" max-width="1265"  alt="" />

### Step 1. SQLX による開発

Dataform では SQLX で開発を行います。SQLX は SQL の拡張なので、SQL ファイルはそのままで正しい SQLX ファイルにもなっています。典型的な SQLX ファイルは、新しいテーブルやビューを定義するための SELECT 文と、その手前にある config ブロックとで構成された内容になっています。

```sql
-- definitions/new_table.sqlx

config { type: "table" }

select
  order_date as date,
  order_id as order_id,
  order_status as order_status,
  sum(item_count) as item_count,
  sum(amount) as revenue

from ${ref("store_clean")}

group by 1, 2
```

<figcaption>サンプル SQLX ファイル</figcaption>

<div className="bp3-callout bp3-icon-info-sign bp3-intent-primary" markdown="1">
SQLX では <code>SELECT</code> 文以外は書く必要がありません。次のステップで <code>CREATE OR REPLACE</code> 文や <code>INSERT</code> 文などのテンプレートが必要になりますが、これについては Dataform が自動で取り扱います。

<a href="/introduction/dataform-in-5-minutes"><button intent="primary">5 分で分る Dataform と SQLX
 のまとめ</button></a></div>

</div>

### Step 2. プロジェクトはリアルタイムにコンパイルされる

定義されたテーブルの数に関わらず、Dataform ではプロジェクト全体がリアルタイムにコンパイルされます。このステップでは、すべての SQLX が対象のデータウェアハウス方言に合わせたピュア SQL に変換されます。このコンパイル中には次のような処理が行われます:

- config ブロックの内容に応じて `CREATE TABLE` 文や `INSERT` 文などのテンプレートが選択される。
- `インクルードファイル` の内容が SQL へとトランスパイルされる。
- `ref()` 関数から実際のテーブル名へと名前解決が行われる。
- 依存性解決が行われ、依存関係の不足や循環参照などのエラーがチェックされる。
- データウェアハウス上で実行予定のアクションに関する依存関係ツリーが作成される。

<img src="https://assets.dataform.co/docs/introduction/simple_dag.png" max-width="885" alt="dependency tree" />

<figcaption>依存関係ツリーの例</figcaption>

```sql
-- compiled.sql
create or replace table "dataform"."orders" as

select
  order_date as date,
  order_id as order_id,
  order_status as order_status,
  sum(item_count) as item_count,
  sum(amount) as revenue

from "dataform_stg"."store_clean"

group by 1, 2
```

<figcaption>コンパイルされた SQLX ファイルの例</figcaption>

<video  controls loop  muted  width="680" ><source src="https://assets.dataform.co/docs/compilation.mp4" type="video/mp4" ><span>Dataform Web におけるリアルタイムなコンパイル</span></video>

<figcaption>Dataform Web におけるオートセーブとリアルタイムなコンパイル</figcaption>

<video  controls loop  muted  width="680" ><source src="https://assets.dataform.co/docs/compilation_cli.mp4" type="video/mp4" ><span>Dataform CLI におけるリアルタイムなコンパイル</span></video>

<figcaption>Dataform CLI で 111 アクションが１秒以内にコンパイルされる様子</figcaption>

### Step 3. データウェアハウス上で依存関係ツリー（またはその一部）の内容が実行される

Dataform はデータウェアハウスに接続し、依存関係ツリーの順序に従って

- テーブルやビューを作成する
- テーブルやビューのデータが正しいかどうかをチェックするため、アサーション用クエリを実行する
- その他の SQL 操作を実行する

などのアクションを実行します。

#### ログを調べる

実行が終わった後でログを確認すれば、どのテーブルが作成されたか、アサーションは通ったのか、または落ちたのか、各アクションが完了するまでどのくらいの時間が掛かったのかなどの情報を調べることができます。また、実際にデータウェアハウス上で実行された SQL コードを確認することもできます。

<img src="https://assets.dataform.co/docs/introduction/run_logs.png" width="995"  alt="" />
<figcaption>Dataform web でログを調べる</figcaption>

### Step 4. データウェアハウス上にテーブル群が作成／更新される

こうして用意されたテーブル群を分析目的で利用することができます。
