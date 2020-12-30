---
title: ELT とモダンなデータ基盤
subtitle: ELT とその中における Dataform の使いどころについてご紹介します
priority: 1
---

（原文: [ELT and the modern data stack | Dataform](https://docs.dataform.co/introduction/modern-data-stack),
ソース: <https://github.com/dataform-co/dataform/blob/master/content/docs/introduction/index.md>）

## ETL から ELT へ

伝統的に用いられてきた ETL （抽出 Extraction, 変換 Transformation, ロード Loading の略）プロセスは、今日では ELT へと進化しています。つまり、データが元のシステムから抽出（Extract）されてデータウェアハウスにロード（Load）された後で、データウェアハウス上で変換（Transform）されるのです。Dataform はこの最後のパート、つまりデータウェアハウス上での変換を管理するためのツールです。

## シンプルな ELT の例

EC ショップのデータ基盤を構築する場面を考えましょう。この事業ではオンライン店舗として Shopify を、支払い処理に Stripe を、そして CRM システムとして Salesforce を利用しており、この３つがデータソースになります。これらすべてのデータを使って KPI 追跡用のレポートを作ったり、ダッシュボードを作ったり、ビジネス理解のためにアドホック分析を行ったりしたいとします。例えば BI ツール上で全顧客情報を反映したダッシュボードを作って社内の誰でも見られるようにするのが最初のタスクになるかもしれません。このとき、このダッシュボードには顧客に関する全データが含まれることになります。

<img src="https://assets.dataform.co/docs/introduction/example_simple_schema.png" max-width="753"  alt="" />

今回の例では、手持ちのすべての（Shopify, Stripe, Salesforce の）データを使った統合ダッシュボードを作ります。

### データウェアハウス

データウェアハウスはモダンなデータ基盤の心臓部です。企業全体から来るあらゆる生データがこのデータウェアハウスに集積されます。データはデータウェアハウス上で変換されます。BI ツールや分析ツールはこのデータウェアハウスからデータを読み取ります。

<img src="https://assets.dataform.co/docs/introduction/datastack_simple_schema.png" max-width="819"  alt="" />

今日では多くの事業において Google BigQuery などクラウドのデータウェアハウスが利用されています。

### 抽出とロード

データ基盤を構築するための最初のステップは、全てのデータソースから生データを抽出してデータウェアハウスにロードすることです。これにはサードパーティ製ツールを使うか、独自のスクリプトを書くことで対応します。

<img src="https://assets.dataform.co/docs/introduction/elt_illustration_step1.png" max-width="1100"  alt="" />

こうしてデータウェアハウスにロードされたデータはまだ未処理のローデータです。それぞれのデータソースについて多数のテーブルがデータウェアハウス上に作成されることになります。この段階では、データはまだ分析用に使える状態になっていません。**"どの顧客が一番多くの商品を注文しているか？"** というようなシンプルな疑問に答えるのにも数時間掛けて複雑なクエリを書く必要があるかもしれません。

### データを変換する

次のステップはデータを変換することです。データウェアハウス上に作られた何百というローデータのテーブルを元として、このビジネスの Single Source of Truth （信頼できる唯一の情報源）を作りたいところです。

今回作成する顧客ダッシュボードの例では、異なるデータソースから抽出したデータを一つに統合し、フィールドの正規化や不正なデータのフィルタリングを行って単一の顧客テーブルを作成します。その顧客テーブルには顧客に関する全ての情報が含まれていて、"どの顧客が一番多くの商品を注文しているか？" といった疑問に瞬時に答えられるようにします。

<img src="https://assets.dataform.co/docs/introduction/elt_illustration_step2.png" max-width="1100"  alt="" />

ダッシュボードに使うテーブルもこれと同じ顧客テーブルです。つまり社内の全員がこのデータを見て、このデータを元に意思決定を行うことになります。そのためには、このデータが信頼できるようテストされていなければいけません。また、ダッシュボードに最新の情報が反映されるためにはこのデータを頻繁に更新する必要があります。さらに、誰でも各フィールドの意味が分かるようドキュメントを整備しておく必要もあります。

<div className="bp3-callout bp3-icon-info-sign bp3-intent-primary" markdown="1">
<h4 class="bp3-heading">Dataform が役に立つのはこのステップです</h4>
Dataform はデータウェアハウス上にあるローデータをビジネス理解のためのテーブルへと変換するときに役立ちます。

Dataform は **きちんと定義され**、**テストされていて**、**ドキュメントの整備された**テーブルを作成することによって分析業務全体を支えるのに役立つのです。
</div>

### 変換したデータを分析に利用する

データを変換したら、BI ツールや他の分析ツールを使ってダッシュボードを構築したり分析を実施したりします。

<img src="https://assets.dataform.co/docs/introduction/elt_illustration_step3.png" max-width="1100" alt="" />
