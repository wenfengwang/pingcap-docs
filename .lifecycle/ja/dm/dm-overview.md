---
title: TiDBデータ移行の概要
summary: データ移行ツール、アーキテクチャ、主要なコンポーネント、機能について学びます。
aliases: ['/docs/tidb-data-migration/dev/overview/','/docs/tidb-data-migration/dev/feature-overview/','/tidb/dev/dm-key-features']
---

<!-- markdownlint-disable MD007 -->

# TiDBデータ移行の概要

<!--
![star](https://img.shields.io/github/stars/pingcap/tiflow?style=for-the-badge&logo=github) ![license](https://img.shields.io/github/license/pingcap/tiflow?style=for-the-badge) ![forks](https://img.shields.io/github/forks/pingcap/tiflow?style=for-the-badge)
-->

[TiDBデータ移行](https://github.com/pingcap/tiflow/tree/master/dm)（DM）は、MySQL互換データベース（MySQL、MariaDB、Aurora MySQLなど）からTiDBへのフルデータ移行および増分データレプリケーションをサポートする統合データ移行タスク管理プラットフォームです。これにより、データ移行の運用コストを削減し、トラブルシューティングプロセスを簡素化できます。

## 基本機能

- **MySQLとの互換性。** DMはMySQL 5.7プロトコルとMySQL 5.7のほとんどの機能と構文に互換性があります。
- **DMLおよびDDLイベントのレプリケーション。** MySQLのバイナリログでのDMLおよびDDLイベントの解析とレプリケーションをサポートしています。
- **MySQLシャードの移行およびマージ。** DMは、上流の複数のMySQLデータベースインスタンスを下流の1つのTiDBデータベースに移行およびマージすることをサポートしています。異なる移行シナリオに対するレプリケーションルールをカスタマイズすることができます。上流のMySQLシャードのDDL変更を自動的に検出および処理できるため、運用コストが大幅に削減されます。
- **さまざまな種類のフィルター。** データ移行プロセス中にMySQLのバイナリログイベントを事前に定義されたイベントタイプ、正規表現、およびSQL式でフィルタリングすることができます。
- **集中管理。** DMはクラスタ内の数千のノードをサポートし、複数のデータ移行タスクを同時に実行および管理できます。
- **サードパーティ製オンラインスキーマ変更プロセスの最適化。** MySQLエコシステムでは、gh-ostやpt-oscなどのツールが広く使用されています。DMは、中間データの不要な移行を避けるために変更プロセスを最適化しています。詳細については、[online-ddl](/dm/dm-online-ddl-tool-support.md)を参照してください。
- **高可用性。** DMはデータ移行タスクを自由にスケジュールできるようにサポートしています。小数のノードがクラッシュしても実行中のタスクには影響しません。

## クイックインストール

次のコマンドを実行してDMをインストールします：

{{< copyable "shell-regular" >}}

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
tiup install dm dmctl
```

## 使用制限

DMツールを使用する前に、以下の制限事項に注意してください：

+ データベースバージョンの要件

    - MySQLバージョン 5.5 ~ 5.7
    - MySQLバージョン 8.0（実験的な機能）
    - MariaDBバージョン >= 10.1.2（実験的な機能）

    > **注意:**
    >
    > 上流のMySQL/MariaDBサーバー間にプライマリセカンダリ移行構造がある場合、以下のバージョンを選択してください。
    >
    > - MySQLバージョン > 5.7.1
    > - MariaDBバージョン >= 10.1.3

+ DDL構文の互換性

    - 現在、TiDBはMySQLがサポートするすべてのDDLステートメントと互換性がありません。DMはTiDBパーサを使用してDDLステートメントを処理するため、TiDBパーサでサポートされているDDL構文のみをサポートしています。詳細については、[MySQL互換性](/mysql-compatibility.md#ddl-operations)を参照してください。

    - DMは非互換なDDLステートメントに遭遇するとエラーを報告します。このエラーを解決するには、dmctlを使用して手動で処理する必要があります。この場合、このDDLステートメントをスキップするか、指定されたDDLステートメントで置換します。詳細については、[非互換なSQLステートメントのスキップまたは置換](/dm/dm-faq.md#how-to-handle-incompatible-ddl-statements)を参照してください。

    - DMはビュー関連のDDLステートメントおよびDMLステートメントを下流のTiDBクラスタにレプリケートしません。下流のTiDBクラスタでビューを手動で作成することをお勧めします。

+ GBK文字セットの互換性

    - DMは、v5.4.0より前のTiDBクラスタに`charset=GBK`テーブルを移行することをサポートしていません。

## 貢献方法

DMのオープンソースプロジェクトに参加することを歓迎します。あなたの貢献は大変評価されます。詳細については、[CONTRIBUTING.md](https://github.com/pingcap/tiflow/blob/master/dm/CONTRIBUTING.md)を参照してください。

## コミュニティサポート

オンラインドキュメントにてDMについて学ぶことができます。ご質問がありましたら、[GitHub](https://github.com/pingcap/tiflow/tree/master/dm)でお問い合わせください。

## ライセンス

DMはApache 2.0ライセンスに準拠しています。詳細については、[LICENSE](https://github.com/pingcap/tiflow/blob/master/LICENSE)を参照してください。

## DMバージョン

v5.4より前のDMドキュメントはTiDBドキュメントとは独立しています。これら以前のバージョンのDMドキュメントにアクセスするには、次のリンクのいずれかをクリックしてください：

- [DM v5.3 documentation](https://docs.pingcap.com/tidb-data-migration/v5.3)
- [DM v2.0 documentation](https://docs.pingcap.com/tidb-data-migration/v2.0/)
- [DM v1.0 documentation](https://docs.pingcap.com/tidb-data-migration/v1.0/)

> **注意:**
>
> - 2021年10月以降、DMのGitHubリポジトリは[pingcap/tiflow](https://github.com/pingcap/tiflow/tree/master/dm)に移動されました。DMに関する問題がある場合は、`pingcap/tiflow`リポジトリにご意見をお寄せください。
> - 以前のバージョン（v1.0およびv2.0）では、DMはTiDBとは独立したバージョン番号を使用していました。v5.3以降、DMはTiDBと同じバージョン番号を使用しています。DM v2.0からv5.3への変更には互換性の変更はありません。アップグレードプロセスは通常のアップグレードと同じです、バージョン番号の増加のみです。