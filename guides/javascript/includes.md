---
title: インクルードによるコードの再利用
subtitle: ここではインクルードを使ってプロジェクト内のコードを再利用する方法について学びます
priority: 1
---

（原文: [Re-use code with includes | Dataform](https://docs.dataform.co/guides/javascript/includes),
ソース: <https://github.com/dataform-co/dataform/blob/master/content/docs/guides/javascript/includes.md>）

## 概要


`includes/` フォルダに JavaScript ファイルを置けば、プロジェクト内で再利用可能な定数やマクロなどを定義することができます。
このフォルダに入れられたファイルはすべて他の SQL や JavaScript から利用可能になります。

`.sqlx` ファイルの中でも、`js {...}` ブロックで囲めば JavaScript を使うことができます。
注意: この方法で定義された関数、定数、マクロなどはその .sqlx ファイル内でのみ利用可能であり、プロジェクト全体で使えるわけではありません。

JavaScript について詳しくない方にとっては、下記の例でよくあるユースケースはカバーされているはずです。
詳しく学びたい方には <a target="_blank" rel="noopener" href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript">MDN</a> が良い学習資料になります。

## 例: 定数の定義と使用

`includes/` フォルダの中に次のような内容のファイルを作成します:

```js
// includes/constants.js
const PROJECT_ID = "my_project_name";
module.exports = { PROJECT_ID };
```
<br />
<callout intent="warning">
  関数や定数を他の場所から使えるようにするためには、<code>module.exports = {"{}"}</code> の文法を使ってエクスポートする必要があるので注意してください。
</callout>

## 例: インクルードを使用する

`include` の関数・定数・マクロを参照するには、ファイル名から `.js` の拡張子を除いた名前を取り、その後にエクスポートされた関数や定数の名前を付けます。

例えば、`includes/constants.js` ファイルの中の定数 `PROJECT_ID` を参照するには

```js
// definitions/query.sqlx
SELECT * FROM ${constants.PROJECT_ID}.my_schema_name.my_table_name
```

のようにします。このクエリは実行時に次のようなSQLへとコンパイルされます:

```js
SELECT * FROM my_project_name.my_schema_name.my_table_name
```

## 例: 関数を書く

関数を使うと、同じ SQL ロジックのかたまりを複数のスクリプトで何度も再利用できるようになります。関数は 0 個かそれ以上のパラメータを受け取り、文字列を返すようになっている必要があります。

以下の例では、関数 `countryGroup()` は国コードのフィールド名を受け取り、国コードを国グループへと変換するための CASE 文を返します。

```js
// includes/country_mapping.js
function countryGroup(countryCodeField) {
  return `CASE
          WHEN ${countryCodeField} IN ("US", "CA") THEN "NA"
          WHEN ${countryCodeField} IN ("GB", "FR", "DE", "IT", "PL") THEN "EU"
          WHEN ${countryCodeField} IN ("AU") THEN ${countryCodeField}
          ELSE "Other countries"
          END`;
}

module.exports = { countryGroup };
```

この関数は SQLX ファイル内で使うことができます：

```js
// definitions/revenue_by_country_group.sqlx
SELECT
  ${country_mapping.countryGroup("country_code")} AS country_group,
  SUM(revenue) AS revenue
FROM my_schema.revenue_by_country
GROUP BY 1

```

このクエリは実行時に次のような SQL へとコンパイルされます:

```js
SELECT
  CASE
    WHEN country_code IN ("US", "CA") THEN "NA"
    WHEN country_code IN ("GB", "FR", "DE", "IT", "PL") THEN "EU"
    WHEN country_code IN ("AU") THEN country_code
    ELSE "Other countries"
  END AS country_group,
  SUM(revenue) AS revenue
FROM my_schema.revenue_by_country
GROUP BY 1
```

## 例: パラメータを使うインクルード: groupBy An include with parameters: groupBy

次の例で定義される `groupBy()` 関数は、グループ化したいフィールドの個数を入力値として受け取り、それに応じた `GROUP BY` 文を生成します。

```js
// includes/utils.js
function groupBy(n) {
  var indices = [];
  for (var i = 1; i <= n; i++) {
    indices.push(i);
  }
  return `GROUP BY ${indices.join(", ")}`;
}

module.exports = { groupBy };
```

この関数は次のようにして SQL クエリ `definitions/example.sqlx` の中で使います:

```js
SELECT field1,
       field2,
       field3,
       field4,
       field5,
       SUM(revenue) AS revenue
FROM my_schema.my_table
${utils.groupBy(5)}
```

このクエリは実行時に次のような SQL へとコンパイルされます:

```js
SELECT field1,
       field2,
       field3,
       field4,
       field5,
       SUM(revenue) AS revenue
FROM my_schema.my_table
GROUP BY 1, 2, 3, 4, 5
```

## 例: クエリの生成

関数でクエリ全体を生成することもできます。これはよく似た構造のデータセットをたくさん作りたいときにとても便利な機能です。

以下の例はすべての指標をすべてのディメンションでグループ化して（`SUM` で）集計する関数です。

```js
// includes/script_builder.js
function renderScript(table, dimensions, metrics) {
  return `
      SELECT
      ${dimensions.map((field) => `${field} AS ${field}`).join(",\\n")},
      ${metrics.map((field) => `SUM(${field}) AS ${field}`).join(",\\n")}
      FROM ${table}
      GROUP BY ${dimensions.map((field, i) => `${i + 1}`).join(", ")}
    `;
}
module.exports = { renderScript };
```

この関数は次のようにして SQL クエリ `definitions/stats_per_country_and_device.sqlx` の中で使います:

```js
${script_builder.renderScript(
  ref("source_table"),
  ["country", "device_type"],
  ["revenue", "pageviews", "sessions"])}

```

依存関係が正しく解決されるようにするためには `ref()` などの関数呼び出しを SQL ファイル内で行い、それをインクルードされた関数に渡す必要があるので注意してください。

このクエリは実行時に次のような SQL へとコンパイルされます:

```js
SELECT country AS country,
       device_type AS device_type,
       SUM(revenue) AS revenue,
       SUM(pageviews) AS pageviews,
       SUM(sessions) AS sessions
FROM my_schema.source_table
GROUP BY 1, 2
```
