---
title: 概要
subtitle: ここでは Dataform の基本、動作の仕組み、データ基盤内での使いどころについて学びます
priority: 0
icon: menu-open
---

（原文: [Introduction | Dataform](https://docs.dataform.co/introduction),
ソース: <https://github.com/dataform-co/dataform/blob/master/content/docs/introduction/index.md>）

## Dataform とは？

Dataform とは BigQuery, Snowflake, Redshift やその他のデータウェアハウス上にあるデータを管理するためのプラットフォームです。Dataform を使うと、ローデータを分析用のテーブルやビューに変換するためのデータパイプラインを構築することができます。

Dataform は **ELT の T** のプロセスを担います（ELT は抽出 Extraction、ロード Load、変換 Transform の略です）。データの抽出やデータウェアハウスへのロードは行いませんが、データウェアハウスにロード済みのデータを変換するときには強力なツールとなります。

<div className="bp3-callout bp3-icon-info-sign bp3-intent" markdown="1">
ELT とモダンなデータ基盤における Datafrom の使いどころについてもっと詳しく知りたい方は

<a href="/introduction/modern-data-stack.md"><button>"モダンなデータ基盤における Datafrom の使いどころ" をご覧ください</button></a></div>

Dataform とそのベストプラクティスを活用すれば、データ管理チームの生産性を上げることができます。また、データ管理チームが新しいテーブルを用意するときにはそれがきちんと定義され、テストが行われていて、かつ社内全体で使えるようドキュメントが整備された状態で提供できるようになります。

<img src="https://assets.dataform.co/blog/datastack_horizontal.png" width="2254"  alt="" />

## 何ができるのか

一言でいうと、Dataform を使うと SQL コマンドで新しいテーブルやビューを作成することができます。Dataform はデータ管理を改善しデータ管理チームの生産性を上げるための機能を多数用意しています。

- データウェアハウス上に作成される **テーブルやビューを定義する**
- テーブルやビューに **ドキュメントを加える**
- **データ品質をテストする** ためのアサーションを定義する
- 複数のスクリプトで **コードを再利用する**
- 任意の **SQL 操作** を実行する
- データの **スナップショット** を取る
- **既製の SQL パッケージ** をデータモデリングに利用する

## 動作の仕組み

1. まず、データウェアハウス上に作成しようとしているテーブルやビューを定義します
2. 開発の最中に、データウェアハウス上で実行予定のアクションに関する依存関係ツリーが自動で作成されます。この依存関係ツリーによって実行されるアクションの順序が固定されるので、テーブルが正しい順序で作成・更新されることが保証されます。
3. この依存関係ツリーの内容が実行されることで、データウェアハウス上に新しいテーブルやビューが作成されたり、その他の SQL コマンドが実行されたりします。

<div className="bp3-callout bp3-icon-info-sign" markdown="1">
Dataform の仕組みについてもっと詳しく知るには
<a href="./dataform-in-5-minutes.md"><button intent="primary">5 分で分かる Dataform まとめ</button></a>
<a href="./how-dataform-works.md"><button>Dataform の技術的な動作</button></a>

</div>

## 使い方

Dataform には主に２つの使い方があります。統合開発環境（IDE）の付いた <a target="_blank" rel="noopener" href="https://dataform.co">Dataform Web</a> アプリを利用するか、またはコマンドラインインターフェース（CLI）でローカル環境から Dataform を使うかの２つです。

Dataform のコア機能（compiler と runner）はオープンソースになっていて、これは CLI で操作できます。

## 誰が Dataform を使うと良いのか

Dataform はクラウドのデータウェアハウスを利用しているデータ専門家向けに作られたものです。ここで言うデータ専門家には データアナリスト、データエンジニア、データサイエンティストなど SQL クエリの書き方を知っている人全員が当てはまります。

Dataform を使うには SQL の理解が必要になります。SQL についてよく知らない方は Khan Academy の [Introduction to SQL course](https://www.khanacademy.org/computing/computer-programming/sql) や [Codeacademy](https://www.codecademy.com/learn/learn-sql) などをご覧ください。

JavaScript の知識もあると Dataform の高度な機能を活用するのに役立ちます。これは必須ではありませんが、利用すると高速かつ楽に開発を行うことができるようになります。JavaScript についてよく知らない方は MDN の [re-introduction to Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript) をご覧ください。

## Dataform を使うメリット

Dataform を利用すると、データ管理チームがデータウェアハウス上のテーブルを管理する際に様々なベストプラクティスやソフトウェア開発ワークフローに従うよう促されます。

Dataform とそのベストプラクティスを利用することで、データ管理チームは十分に素早く仕事をこなし、組織全体へ信頼性・理解性の高いデータを提供することができるようになります。

## 困ったときは

私たちの Slack グループに参加すれば、私たちのチームや Dataform を利用している何百人ものデータ専門家と相談することができます。

<a href="https://join.slack.com/t/dataform-users/shared_invite/zt-dark6b7k-r5~12LjYL1a17Vgma2ru2A" target="_blank" rel="noopener"><button intent="primary">Dataform slack に参加する</button></a></div>

もし Dataform web アプリ上で何か問題が発生したときは、ページ右下の Intercom メッセンジャーアイコンから私達のチームに連絡してください。
