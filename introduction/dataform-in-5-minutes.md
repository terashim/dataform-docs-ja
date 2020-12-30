---
title: ５分で分る SQLX と Dataform
subtitle: ここでは Detaform と SQLX を使ってデータウェアハウス上のデータを管理する方法について学びます
priority: 2
---

（原文: [SQLX and Dataform in 5 minutes | Dataform](https://docs.dataform.co/introduction/dataform-in-5-minutes),
ソース: <https://github.com/dataform-co/dataform/blob/master/content/docs/introduction/dataform-in-5-minutes.md>）

## イントロダクション

モダンな分析アプローチでは、企業内で発生する全てのローデータを一つのデータウェアハウスに集めることが前提になっています。集めたローデータを BI ツールや分析プロジェクトで使えるようにするには、変換・集約・標準化・結合・フィルタリングなどの処理を行う必要があります。Dataform はデータ管理チームがローデータをきちんと定義された、信頼性の高い、テストされた、ドキュメントのあるテーブルに変換することで企業の分析活動を支えるのに役立ちます。

### ETL から ELT へ

伝統的に用いられてきた ETL （抽出 Extraction, 変換 Transformation, ロード Loading の略）プロセスは、今日では ELT へと進化しています。つまり

1. データソースのシステムから抽出（Extract）されたローデータがデータウェアハウスにロード（Load）され
2. データウェアハウス上でローデータが変換（Transform）される

という順序になっています。Dataform はこの最後のパート、つまりデータウェアハウス上でのデータ変換を管理するのに使えます。

<div className="bp3-callout bp3-icon-info-sign bp3-intent-primary" markdown="1">
ELT とモダンなデータ基盤における Dataform の使いどころについてより詳しく知るには

<a href="/introduction/modern-data-stack">><button intent="primary">"モダンなデータ基盤における Dataform の使いどころ" をご覧ください</button></a></div>

### データの Single Source of Truth を構築する

データウェアハウスにローデータがロードされたら、それを変換して組織全体で利用できる Single Source of Truth（信頼できる唯一の情報源）にするのがデータ管理チームの仕事です。Dataform はデータ管理チームが業界のベストプラクティス

- データは（SQLの）コードで管理する
- バージョン管理などの標準開発プロセスを整備する
- データ品質をテストする
- テーブルのドキュメントを作成する

に沿った仕事ができるように設計されています。

<div className="bp3-callout bp3-icon-info-sign bp3-intent-none" markdown="1">
<h4 class="bp3-heading">GUI についての注意</h4>
グラフィカルユーザーインターフェース（GUI）はとっつきやすく、非技術職のユーザーがデータパイプラインを構築するのに便利です。しかし実務においては、テーブル数が 10 から 20 を超えたあたりでパイプラインの管理、検索、調査が急激に難しくなるというのが私たちの知るところです。SQL は複雑なデータ処理ロジックを表すのにベストな抽象表現の一つと言えるでしょう。
</div>

## SQLX の紹介

SQL はクラウドデータウェアハウス上のデータ処理に利用されるデファクトの言語です。SQL には次のように多くの利点があります。

- データウェアハウス上でのスケーラブルな処理が可能
- 通常は パイプラインを SQL で表現するのが一番シンプルで容易となる
- 多くのチームやシステムで共通に使われる言語である
- 何か問題が起きたときの調査が容易である
- 素早いフィードバックループのおかげで高速な開発が可能

### SQL にある多少の制約

既存の SQL ワークフローは必ずしも開発のベストプラクティスに沿うものにはなっていません。というのも、コードを書く上でいくつか重要な機能が既存の SQL の実装には欠けているのです。例えば

- 複数のスクリプト間で簡単には **コードを再利用することができません** 。
- データの整合性を保証するための **テストを書く方法がありません** 。
- 別のシステムが必要となるため **依存関係の管理が難しい** です。実務においては多くのチームがデータ処理が正しい順序で行われるようにするために 1000 行ほどの長いクエリを書いています。
- **データにドキュメントが付いていないことがよくあります**。これはドキュメントが別のシステムを使ってコードの外側で管理されているためです。そのためドキュメントを常に最新の状態に保つことが難しくなります。

### SQLX とは何か

SQLX はオープンソースの SQL 拡張です。拡張ということは、SQL ファイルはすべて SQLX ファイルとして有効になります。**SQLX は SQL に追加機能を持ち込み、高速で・信頼性の高い・スケーラブルな開発を可能にしたものです**。SQLX には依存関係の管理、データ品質の自動テスト、データのドキュメンテーションなどの多くの関数が含まれています。

### SQLX の中身

SQLX の主要部は各データウェアハウス方言の SQL （BigQuery なら Standard SQL、Snowflake なら SnowSQL など）です。

<img src="https://assets.dataform.co/docs/introduction/sqlx_simple_example.png" max-width="661"  alt="SQLX example" />
<figcaption>このイラストでは BigQuery の Standard SQL を使用しています。SQLX は他の SQL 方言でも同様に動作します。</figcaption>

<h4><span className="numberTitle">1</span> Config ブロック</h4>

SQLX では、SELECT 文だけが使えます。スクリプトの結果をどのように出力したいかは config ブロックに書きます。出力の形式には `view` や `table` などのタイプがあります。

`CREATE OR REPLACE` 文や `INSERT` 文などのテンプレートを追加することに関しては Dataform が自動で取り扱います。

<h4><span className="numberTitle">2</span> ref 関数と依存性管理</h4>

`ref` 関数は Dataform において極めて重要なコンセプトです。`ref` 関数を使うと、スキーマやテーブルの名前をハードコードせずに dataform プロジェクト内で定義されたテーブルやビューを参照することができます。

また Dataform では作成・更新の対象となる全テーブルの依存関係ツリーを作成する際に `ref` 関数を利用します。Dataform がデータウェアハウス上で動作するとき、テーブルが正しい順序で処理されることがこの依存関係ツリーによって保証されます。

下記の画像はシンプルな Dataform プロジェクトとその依存関係ツリーを表しています。実務では１つの Dataform プロジェクトで数百のテーブルに関する依存関係ツリーを扱うことも可能です。

<img src="https://assets.dataform.co/docs/introduction/ref_illustration_code.png" max-width="426" alt="Ref function illustration" />
<figcaption>１つの Dataform プロジェクトに含まれるスクリプト群</figcaption>

<img src="https://assets.dataform.co/docs/introduction/simple_dag.png" max-width="885" alt="dependency tree" />
<figcaption>Dataform プロジェクトの依存関係ツリー</figcaption>

依存関係を `ref` 関数で管理することには大変な利点があります:

- 依存関係ツリーの複雑性が無視できるようになります。開発者はただ `ref` 関数を使い依存関係を示すだけで大丈夫です。
- 1000 行におよぶような長いクエリを書かなくても、より小さく、再利用性が高く、モジュール化されたクエリが書けるようになります。それによってパイプラインのデバッグが簡単になります。
- 依存関係の不足や循環参照のような問題に対してリアルタイムでアラートが出るようにできます。

### SQLX = 変換ロジック + データ品質テスト + ドキュメンテーション

変換ロジックとデータ品質テストのルール、およびテーブルに対するドキュメンテーションを単一のファイルで定義できることは SQLX の最もパワフルな性質の一つです。

<div className="bp3-callout bp3-icon-info-sign bp3-intent-primary" markdown="1">
SQLX の各機能は段階的に導入していくことができます。最初はシンプルなスクリプトからスタートして、パイプラインが複雑になるにつれて徐々に利用する SQLX の機能を増やしていくチームが多いようです。
</div>

<img src="https://assets.dataform.co/docs/introduction/sqlx_second_example.png" max-width="481" alt="SQLX second example" />

<figcaption>ここではドキュメンテーションとデータ品質テストの機能を利用する SQLX ファイルの例を示します</figcaption>

<h4><span className="numberTitle">1</span> データのドキュメンテーション</h4>

SQLX ファイルの config ブロックには、テーブルやフィールドの説明文を直接書き込むことができます。テーブルに付けた説明文は Dataform のデータカタログで表示されるようになります。

説明文をデータと同じファイルで定義することで、ドキュメントが常に最新の状態に保ちやすくなります。

<div className="bp3-callout bp3-icon-info-sign bp3-intent-none" markdown="1">
Dataform で作成したドキュメンテーションは機械可読になります。したがって、ドキュメントをパースして他のツールに渡すことも可能です。
</div>

<h4><span className="numberTitle">2</span> データ品質テスト</h4>

データ品質テストのことをアサーションといいます。アサーションは SQLX ファイルの config ブロックに直接書き込んで定義することができます。アサーションを使うと一意性、NULL値、あるいは任意のカスタム条件でデータをチェックできます。

config ブロックでアサーションを定義すると、依存関係ツリーでそのテーブル作成アクションの次にそのアサーションが追加されます。

<img src="https://assets.dataform.co/docs/introduction/assertion_dag.png" max-width="605" alt="DAG with assertions" />
<figcaption>config ブロックで定義されたアサーションとそのプロジェクトの依存関係ツリー。アサーションはテーブル作成／更新の後に実行される。</figcaption>

より高度なユースケースでは、アサーションを独立の SQLX ファイルで定義することもできます。これに関してはこのドキュメントの [アサーションのページ](https://docs.dataform.co/guides/assertions) をご覧ください。

### その他の SQLX の機能

SQLX にはデータウェアハウス上のデータを管理し信頼性の高いデータパイプラインを素早く構築するのに使える機能が数多くあります。

- インクリメンタルなテーブルとスナップショット
- 再利用可能な関数と変数
- ソースデータの宣言
- その他多くの機能

詳細についてはドキュメントをご確認ください。

## SQLX 開発の流れ

<img src="https://assets.dataform.co/docs/introduction/how%20it%20works.png" max-width="1265"  alt="" />

**Step 1.** 開発者はローカル環境または Dataform Web のエディタを使って SQLX ファイルを編集し、パイプラインを開発します。

**Step 2.** プロジェクトの内容は自動的にデータウェアハウス上で実行可能なネイティブ SQL へとコンパイルされます。このプロセスはリアルタイムに実行され、依存関係の解決やエラーの検査が行われます。このとき問題があればアラートが発生します。

**Step 3.** Dataform はデータウェアハウスに接続し、コンパイルされた SQL コマンド群を実行します。このステップは手動実行、スケジュール実行、または API 呼び出しによって開始されます。

**Step 4.** 実行が終了したときには、テスト済みかつドキュメント付きの分析用テーブル群が出来上がっています。

<div className="bp3-callout bp3-icon-info-sign bp3-intent-success" markdown="1">
<h4 class="bp3-heading">SQLX はオープンソース</h4>
この SQLX コンパイラーとランナーはオープンソースなので、ローカル環境や自分のサーバ上で動かすことができます。
</div>
<br/>
<div className="bp3-callout bp3-icon-info-sign" markdown="1">
Dataform の動作についてより詳しく知るには

<a href="/introduction/modern-data-stack"><button>Dataform の仕組みを理解する</button></a></div>

## Dataform Web でベストプラクティスに沿った開発を行い生産性を上げる

Dataform Web はデータ管理チーム向けに作られたウェブアプリケーションです。このウェブアプリはリッチな統合開発環境（IDE）、パイプラインの実行スケジューラ、ログビューア、データカタログをまとめたパッケージになっています。

<img src="https://assets.dataform.co/landing/ide-mockup.png" max-width="1230" alt="" />

Dataform Web はデータ管理チームが１つの環境で共同作業を行えるようにできています。この環境には次のようなメリットがあります:

- データウェアハウス上でデータパイプラインを実行するための **マネージドなインフラ**
- **パイプラインの保守に掛かる時間を最小にする** ためのアラートシステムと詳細ログ
- バージョン管理や開発環境など **エンジニアリングのベストプラクティス導入のハードルを下げる** 直感的な UX。
- 開発中の即時フィードバックによる **生産性の向上**
- テーブルやパイプラインを素早く探すための **データカタログ**

Dataform Web についてより詳しく知りたい方は、[dataform.co/product](https://dataform.co/product) をチェックしてください。
