---
title: TiDBデータベーススキーマ設計概要
summary: TiDBデータベーススキーマ設計の基礎を学ぶ。
---

# TiDBデータベーススキーマ設計概要

この文書では、TiDBデータベーススキーマ設計の基礎として、TiDB内のオブジェクト、アクセス制御、データベーススキーマ変更、およびオブジェクトの制限について説明します。

後続の文書では、[書店](/develop/dev-guide-bookshop-schema-design.md)を例にとり、データベースの設計およびデータの読み取りと書き込み操作方法を示します。

## TiDB内のオブジェクト

一般的な用語を区別するために、TiDBで使用される用語について簡単な合意があります。

- [データベース](https://en.wikipedia.org/wiki/Database)という一般的な用語と混同を避けるため、**この文書では**、**データベース**は論理オブジェクトを指し、**TiDB**はTiDB自体を指し、**クラスタ**は展開されたTiDBのインスタンスを指します。

- TiDBはMySQL互換の構文を使用し、ここでの**スキーマ**は、[MySQLのドキュメント](https://dev.mysql.com/doc/refman/8.0/en/create-database.html)の一般的な用語[スキーマ](https://en.wiktionary.org/wiki/schema)ではなく、データベース内の論理オブジェクトを意味します。詳細については、[MySQLのドキュメント](https://dev.mysql.com/doc/refman/8.0/en/create-database.html)をご覧ください。スキーマが論理オブジェクトとして存在するデータベースから移行する場合は、この違いに注意してください（たとえば、[PostgreSQL](https://www.postgresql.org/docs/current/ddl-schemas.html)、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/tdddg/creating-managing-schema-objects.html)、および[Microsoft SQL Server](https://docs.microsoft.com/en-us/sql/relational-databases/security/authentication-access/create-a-database-schema?view=sql-server-ver15)）。

### データベース

TiDB内のデータベースは、テーブルやインデックスなどのオブジェクトのコレクションです。

TiDBには、`test`というデフォルトのデータベースが付属していますが、`test`データベースを使用せず、独自のデータベースを作成することをお勧めします。

### テーブル

テーブルは、[データベース](#データベース)内の関連するデータのコレクションです。

各テーブルには**行**と**列**があります。行の各値は特定の**列**に属します。各列は単一のデータ型のみを許可します。列をさらに修飾するためには、いくつかの[制約](/constraints.md)を追加できます。演算を加速するために、[生成された列](/generated-columns.md)を追加することができます。

### インデックス

インデックスは、テーブル内の選択された列のコピーです。インデックスは、1つ以上の列を使用してテーブルのインデックスを作成できます。インデックスにより、TiDBはすべての行を毎回検索する必要なく、データを迅速に位置付けることができます。これによりクエリのパフォーマンスが大幅に向上します。

一般的なインデックスの種類には次のものがあります。

- **プライマリキー**：プライマリキーカラムに対するインデックス。
- **セカンダリインデックス**：プライマリキーカラム以外のカラムに対するインデックス。

> **注意:**
>
> TiDBでは、**プライマリキー**のデフォルトの定義が[InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)（MySQLの一般的なストレージエンジン）と異なります。
>
> - InnoDBでは、**プライマリキー**の定義はユニークであり、NULLでない **クラスタ化されたインデックス**である。
> - TiDBでは、**プライマリキー**の定義はユニークであり、NULLである。ただし、プライマリキーが**クラスタ化されたインデックス**であることは保証されません。プライマリキーがクラスタ化インデックスであるかどうかを指定するには、`CREATE TABLE`ステートメントの`PRIMARY KEY`の後に予約済みキーワード`CLUSTERED`または`NONCLUSTERED`を追加できます。ステートメントでこれらのキーワードを明示的に指定しない場合、デフォルトの動作はシステム変数`@@global.tidb_enable_clustered_index`によって制御されます。詳細については、[クラスタ化インデックス](/clustered-indexes.md)を参照してください。

#### 専門のインデックス

<CustomContent platform="tidb">

さまざまなユーザーシナリオのクエリパフォーマンスを向上させるために、TiDBはいくつかの特殊な種類のインデックスを提供しています。各タイプの詳細については、[インデックスと制約](/basic-features.md#indexing-and-constraints)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

さまざまなユーザーシナリオのクエリパフォーマンスを向上させるために、TiDBはいくつかの特殊な種類のインデックスを提供しています。各タイプの詳細については、[インデックスと制約](https://docs.pingcap.com/tidb/stable/basic-features#indexing-and-constraints)を参照してください。

</CustomContent>

### その他のサポートされる論理オブジェクト

TiDBでは、**テーブル**と同じレベルで以下の論理オブジェクトをサポートしています。

- [ビュー](/views.md)：ビューは、ビューを作成する`SELECT`ステートメントによって定義される仮想テーブルとして機能します。
- [シーケンス](/sql-statements/sql-statement-create-sequence.md)：シーケンスは連続したデータを生成および格納します。
- [一時テーブル](/temporary-tables.md)：データが永続的でないテーブル。

## アクセス制御

<CustomContent platform="tidb">

TiDBはユーザーベースとロールベースのアクセス制御をサポートしています。ユーザーがデータオブジェクトとデータスキーマを表示、変更、または削除する権限を許可するには、[特権](/privilege-management.md)を[ユーザー](/user-account-management.md)に直接付与するか、[ロール](/role-based-access-control.md)を介してユーザーに[特権](/privilege-management.md)を付与できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDBはユーザーベースとロールベースのアクセス制御をサポートしています。ユーザーがデータオブジェクトとデータスキーマを表示、変更、または削除する権限を許可するには、[特権](https://docs.pingcap.com/tidb/stable/privilege-management)を[ユーザー](https://docs.pingcap.com/tidb/stable/user-account-management)に直接付与するか、[ロール](https://docs.pingcap.com/tidb/stable/role-based-access-control)を介してユーザーに[特権](https://docs.pingcap.com/tidb/stable/privilege-management)を付与できます。

</CustomContent>

## データベーススキーマ変更

ベストプラクティスとして、データベーススキーマの変更は、ドライバーやORMではなく、[MySQLクライアント](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)またはGUIクライアントを使用して実行することをお勧めします。

## オブジェクトの制限事項

詳細については、[TiDB制限事項](/tidb-limitations.md)を参照してください。