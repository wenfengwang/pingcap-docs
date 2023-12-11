---
title: SQLチューニング概要
summary: TiDBクラウドでSQLパフォーマンスをチューニングする方法について学ぶことができます。
---

# SQLチューニング概要

このドキュメントでは、TiDBクラウドでSQLパフォーマンスをチューニングする方法を紹介します。最良のSQLパフォーマンスを得るためには、以下のようにできます。

- SQLパフォーマンスをチューニングする。クエリステートメントの分析、実行計画の最適化、フルテーブルスキャンの最適化など、SQLパフォーマンスを最適化するための様々な方法があります。
- スキーマ設計を最適化する。ビジネスワークロードのタイプに応じて、トランザクションの競合やホットスポットを避けるためにスキーマを最適化する必要があります。

## SQLパフォーマンスのチューニング

SQLステートメントのパフォーマンスを向上させるために、以下の原則を考慮してください。

- スキャンされるデータの範囲を最小限に抑える。常にデータの最小限の範囲のみをスキャンし、すべてのデータをスキャンすることを避けるのが最良の方法です。
- 適切なインデックスを使用する。SQLステートメントの`WHERE`句の各列に対して、対応するインデックスがあることを確認してください。そうでない場合、`WHERE`句はフルテーブルをスキャンし、パフォーマンスが低下します。
- 適切な結合方法を使用する。クエリ内の各テーブルのサイズと相関関係に応じて、適切な結合方法を選択することが非常に重要です。一般に、TiDBのコストベースのオプティマイザーは自動的に最適な結合方法を選択します。ただし、一部の場合では結合方法を手動で指定する必要があります。詳細については、[Explain Statements That Use Joins](/explain-joins.md)を参照してください。
- 適切なストレージエンジンを使用する。Hybrid Transactional and Analytical Processing (HTAP) ワークロードでは、TiFlashストレージエンジンの使用が推奨されます。詳細については、[HTAP Queries](/develop/dev-guide-hybrid-oltp-and-olap-queries.md)を参照してください。

TiDBクラウドでは、クラスタ上の遅いクエリを分析するためのいくつかのツールが提供されています。次のセクションでは、遅いクエリを最適化するためのいくつかのアプローチについて説明します。

### 診断タブでのステートメントの使用

TiDB Cloudコンソールには、**[SQLステートメント](/tidb-cloud/tune-performance.md#statement-analysis)** タブが **診断** タブのサブタブとして提供されています。このタブでは、クラスタ上のすべてのデータベースのSQLステートメントの実行統計を収集します。ここでは、合計または単一の実行に長い時間を要するSQLステートメントを識別および分析することができます。

このサブタブでは、同じ構造のSQLクエリ（クエリパラメータが一致しなくても）は、同じSQLステートメントにグループ化されます。例えば、`SELECT * FROM employee WHERE id IN (1, 2, 3)` と `select * from EMPLOYEE where ID in (4, 5)` は、どちらも同じSQLステートメント `select * from employee where id in (...)` の一部です。

**ステートメント** では、以下の主要な情報を表示できます。

- SQLステートメントの概要：SQLダイジェスト、SQLテンプレートID、現在表示されている時間範囲、実行計画の数、および実行が行われるデータベースを含みます。
- 実行計画リスト：SQLステートメントに複数の実行計画がある場合、リストが表示されます。異なる実行計画を選択でき、選択した実行計画の詳細はリストの末尾に表示されます。1つの実行計画しかない場合、リストは表示されません。
- 実行計画の詳細：選択した実行計画の詳細が表示されます。異なる観点からそのようなSQLタイプの実行計画と対応する実行時間を収集して、より多くの情報を得るのに役立ちます。詳細については、[Execution plan in details](https://docs.pingcap.com/tidb/stable/dashboard-statement-details#statement-execution-details-of-tidb-dashboard)（画像下部のエリア3）を参照してください。

![Details](/media/dashboard/dashboard-statement-detail.png)

**ステートメント** ダッシュボードの情報に加えて、以下のセクションでTiDBクラウドのためのいくつかのSQLベストプラクティスも説明されています。

### 実行計画の確認

[`EXPLAIN`](/explain-overview.md) を使用して、TiDBがコンパイル時にステートメントのために計算した実行計画を確認できます。つまり、TiDBは数百または数千の可能な実行計画を推定し、最少のリソースを消費し、最も高速に実行される最適な実行計画を選択します。

TiDBによって選択された実行計画が最適でない場合、EXPLAINまたは[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md) を使用して診断できます。

### 実行計画の最適化

`parser` によって元のクエリテキストを解析し、基本的な妥当性の検証を行った後、TiDBはまずクエリにいくつかの論理的等価の変更を行います。詳細については、[SQL Logical Optimization](/sql-logical-optimization.md)を参照してください。

これらの等価変更により、クエリは論理実行計画で扱いやすくなることがあります。等価変更後、TiDBは元のクエリと等価のクエリプラン構造を取得し、その後データ分布とオペレータの具体的な実行オーバーヘッドに基づいて最終的な実行計画を取得します。詳細については、[SQL Physical Optimization](/sql-physical-optimization.md)を参照してください。

また、TiDBは、`PREPARE` ステートメントの実行中に実行計画の作成オーバーヘッドを削減するために実行計画キャッシュを有効にすることができます。詳細については、[Prepare Execution Plan Cache](/sql-prepared-plan-cache.md)を参照してください。

### フルテーブルスキャンの最適化

遅いSQLクエリの最も一般的な原因は、`SELECT` ステートメントがフルテーブルスキャンを実行するか、間違ったインデックスを使用していることです。クエリの実行計画を表示して、実行の遅延の原因を特定できます。[3つの方法](/develop/dev-guide-optimize-sql.md)で最適化することができます。

- セカンダリインデックスを使用する
- カバリングインデックスを使用する
- プライマリインデックスを使用する

### DMLベストプラクティス

[DML best practices](/develop/dev-guide-optimize-sql-best-practices.md#dml-best-practices) を参照してください。

### 主キーの選択時のDDLベストプラクティス

主キーを選択する際の[ガイドライン](/develop/dev-guide-create-table.md#guidelines-to-follow-when-selecting-primary-key)を参照してください。

### インデックスのベストプラクティス

インデックスの作成と使用の[ベストプラクティス](/develop/dev-guide-index-best-practice.md)については、インデックスの作成速度はデフォルトで控えめですが、特定のシナリオで変数を[修正](/develop/dev-guide-optimize-sql-best-practices.md#add-index-best-practices)することでインデックス作成プロセスを加速することができます。

<!--
### 遅いクエリログメモリマッピングテーブルの使用

[INFORMATION_SCHEMA.SLOW_QUERY](/identify-slow-queries.md#memory-mapping-in-slow-log) テーブルをクエリして、遅いクエリログの内容を調べ、[`SLOW_QUERY`](/information-schema/information-schema-slow-query.md) テーブル内の構造を見つけることができます。このテーブルを使用して、潜在的な問題を見つけるために異なるフィールドを使用してクエリを実行できます。

遅いクエリの推奨される分析プロセスは次のとおりです。

1. [クエリのパフォーマンスボトルネックを特定する](/analyze-slow-queries.md#identify-the-performance-bottleneck-of-the-query)。つまり、クエリプロセスの中で時間のかかる部分を特定します。
2. [システムの問題の分析](/analyze-slow-queries.md#analyze-system-issues)。ボトルネックポイントに応じて、当時のモニタリング、ログ記録などを組み合わせて、可能な原因を見つけます。
3. [オプティマイザの問題の分析](/analyze-slow-queries.md#analyze-optimizer-issues)。より良い実行計画があるかどうかを分析します。
-->

## スキーマ設計の最適化

SQLパフォーマンスチューニングを行ってもパフォーマンスが向上しない場合は、スキーマ設計およびデータ読み取りモデルを確認して、トランザクションの競合やホットスポットを避ける必要があります。

### トランザクションの競合

トランザクションの競合を特定および解決する方法についての詳細は、[トラブルシューティング ロックの競合](https://docs.pingcap.com/tidb/stable/troubleshoot-lock-conflicts#troubleshoot-lock-conflicts)を参照してください。

### ホットスポットの問題

[Key Visualizer](/tidb-cloud/tune-performance.md#key-visualizer)を使用してホットスポットの問題を分析できます。

Key Visualizerを使用して、TiDBクラスターのトラフィックの使用パターンを分析し、トラフィックのホットスポットに対処できます。このページでは、TiDBクラスターのトラフィックを時間の経過に沿って視覚的に表現します。

Key Visualizerでは、次の情報を観察できます。まず、まず、[基本コンセプト](https://docs.pingcap.com/tidb/stable/dashboard-key-visualizer#basic-concepts)を理解する必要があります。

- 時間の経過に伴う全体のトラフィックを示す大きなヒートマップ
- ヒートマップの座標の詳細情報
- 左側に表示されるテーブルやインデックスなどの識別情報

X軸方向とY軸方向の交互の明るさを持つヒートマップ結果の共通の4つのタイプがあります。

- 均等に分布したワークロード：望ましい結果
- X軸（時間）に沿って交互に明るさと暗さが交互に現れる：ピーク時のリソースをチェックする必要があります
- Y軸に沿って交互に明るさと暗さが交互に現れる：生成されたホットスポットの程度をチェックする必要があります
- 明るい対角線：ビジネスモデルを確認する必要があります

X軸とY軸の両方での交互の明るさと暗さの場合、読み書きのプレッシャーを解決する必要があります。

SQLパフォーマンスの最適化の詳細については、SQL FAQsの[SQL Optimization](https://docs.pingcap.com/tidb/stable/sql-faq#sql-optimization)を参照してください。