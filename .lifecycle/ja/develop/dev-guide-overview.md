---
title: 開発者ガイドの概要
summary: 開発者ガイドの概要を紹介します。
aliases: ['/tidb/dev/connectors-and-apis/','/appdev/dev/','/tidb/dev/dev-guide-outdated-for-laravel']
---

# 開発者ガイドの概要

このガイドはアプリケーション開発者向けに書かれていますが、TiDBの内部動作に興味がある場合やTiDBの開発に関わりたい場合は、TiDBのカーネル開発ガイドを詳細については[TiDBカーネル開発ガイド](https://pingcap.github.io/tidb-dev-guide/)をご覧ください。

<CustomContent platform="tidb">

このチュートリアルでは、TiDBを使用したアプリケーションの迅速な構築方法、TiDBの可能なユースケース、および一般的な問題の扱い方を紹介します。

このページを読む前に、[TiDBデータベースプラットフォームのクイックスタートガイド](/quick-start-with-tidb.md)をお読みいただくことをお勧めします。

</CustomContent>

<CustomContent platform="tidb-cloud">

このチュートリアルでは、TiDB Cloudを使用したアプリケーションの迅速な構築方法、TiDB Cloudの可能なユースケース、および一般的な問題の扱い方を紹介します。

</CustomContent>

## TiDBの基礎

TiDBを扱う前に、TiDBの動作メカニズムについていくつか理解する必要があります。

- TiDBでのトランザクションの動作を理解するために[TiDBトランザクション概要](/transaction-overview.md)を読んでください。またはアプリケーション開発に必要なトランザクションの知識については[アプリケーション開発者向けトランザクションノート](/develop/dev-guide-transaction-overview.md)をチェックしてください。
- [アプリケーションがTiDBとやり取りする方法](#the-way-applications-interact-with-tidb)を理解してください。
- 分散データベースTiDBおよびTiDB Cloudのコアコンポーネントとコンセプトを学ぶには、無料のオンラインコース[TiDB入門](https://eng.edu.pingcap.com/catalog/info/id:203/?utm_source=docs-dev-guide)を参照してください。

## TiDBのトランザクションメカニズム

TiDBは分散トランザクションをサポートし、**楽観的トランザクション**モードと**悲観的トランザクション**モードの両方を提供しています。現在のTiDBのバージョンでは、デフォルトで**悲観的トランザクション**モードが使用されており、これにより、従来のモノリシックデータベース（たとえばMySQL）と同様にTiDBとトランザクションを実行できます。

[`BEGIN`](/sql-statements/sql-statement-begin.md)を使用してトランザクションを開始し、`BEGIN PESSIMISTIC`を使用して**悲観的トランザクション**を明示的に指定するか、`BEGIN OPTIMISTIC`を使用して**楽観的トランザクション**を明示的に指定することができます。その後、コミット([`COMMIT`](/sql-statements/sql-statement-commit.md))またはロールバック([`ROLLBACK`](/sql-statements/sql-statement-rollback.md))でトランザクションを完了させることができます。

TiDBは`BEGIN`の開始から`COMMIT`や`ROLLBACK`の最後までの間に実行されたすべてのステートメントに対して、すべてのステートメントが全体として成功するか失敗するかを原子的に保証します。これは、アプリケーション開発に必要なデータの整合性を確保するために使用されます。

<CustomContent platform="tidb">

**楽観的トランザクション**の動作が分からない場合は、まだ使用しないでください。なぜなら、**楽観的トランザクション**では`COMMIT`ステートメントで返される[すべてのエラー](/error-codes.md)を正しく処理できるようにする必要があります。アプリケーションがそれらをどのように処理するかがわからない場合は、代わりに**悲観的トランザクション**を使用してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

**楽観的トランザクション**の動作が分からない場合は、まだ使用しないでください。なぜなら、**楽観的トランザクション**では`COMMIT`ステートメントで返される[すべてのエラー](https://docs.pingcap.com/tidb/stable/error-codes)を正しく処理できるようにする必要があります。アプリケーションがそれらをどのように処理するかがわからない場合は、代わりに**悲観的トランザクション**を使用してください。

</CustomContent>

## アプリケーションがTiDBとやり取りする方法

TiDBはMySQLプロトコルと高い互換性があり、[ほとんどのMySQLの構文と機能](/mysql-compatibility.md)をサポートしているため、ほとんどのMySQL接続ライブラリがTiDBと互換性があります。PingCAP公式のアダプテーションがない場合は、お使いのアプリケーションフレームワークや言語に公式の適応がない場合は、MySQLのクライアントライブラリを使用することをお勧めします。TiDBのさまざまな機能を積極的にサポートするサードパーティのライブラリが増えています。

TiDBはMySQLプロトコルおよびMySQL構文と互換性があるため、MySQLをサポートするORMのほとんどがTiDBと互換性があります。

## もっと読む

<CustomContent platform="tidb">

- [クイックスタート](/develop/dev-guide-build-cluster-in-cloud.md)
- [ドライバーまたはORMの選択](/develop/dev-guide-choose-driver-or-orm.md)
- [TiDBへの接続](/develop/dev-guide-connect-to-tidb.md)
- [データベーススキーマ設計](/develop/dev-guide-schema-design-overview.md)
- [データの書き込み](/develop/dev-guide-insert-data.md)
- [データの読み取り](/develop/dev-guide-get-data-from-single-table.md)
- [トランザクション](/develop/dev-guide-transaction-overview.md)
- [最適化](/develop/dev-guide-optimize-sql-overview.md)
- [サンプルアプリケーション](/develop/dev-guide-sample-application-java-spring-boot.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB Cloudで接続し、管理、開発するための追加リソースをこちらでご確認いただけます。

**データを探索する**

- [クイックスタート](/develop/dev-guide-build-cluster-in-cloud.md)
- [AIパワードSQLエディタ <sup>ベータ版</sup>を使用](/tidb-cloud/explore-data-with-chat2query.md)
- [VSCode](/develop/dev-guide-gui-vscode-sqltools.md)、[DBeaver](/develop/dev-guide-gui-dbeaver.md)、または[DataGrip](/develop/dev-guide-gui-datagrip.md)などのクライアントツールで接続

**アプリケーションを構築する**

- [ドライバーまたはORMの選択](/develop/dev-guide-choose-driver-or-orm.md)
- [TiDB CloudデータAPI <sup>ベータ版</sup>を使用](/tidb-cloud/data-service-overview.md)

**クラスタを管理する**

- [TiDB Cloudコマンドラインツール](/tidb-cloud/get-started-with-cli.md)
- [TiDB Cloud管理API](https://docs.pingcap.com/tidbcloud/api/v1beta1)

**TiDBについてもっと学ぶ**

- [データベーススキーマ設計](/develop/dev-guide-schema-design-overview.md)
- [データの書き込み](/develop/dev-guide-insert-data.md)
- [データの読み取り](/develop/dev-guide-get-data-from-single-table.md)
- [トランザクション](/develop/dev-guide-transaction-overview.md)
- [最適化](/develop/dev-guide-optimize-sql-overview.md)

</CustomContent>