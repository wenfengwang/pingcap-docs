---
title: 実行計画の制御
---

# 実行計画の制御

SQLチューニングの最初の2章では、TiDBの実行計画の理解方法とTiDBが実行計画を生成する方法について紹介します。この章では、実行計画に問題がある場合、実行計画の生成を制御するための方法を紹介します。この章では、主に次の3つの側面が含まれます。

- [オプティマイザヒント](/jp/optimizer-hints)では、ヒントを使用してTiDBに実行計画を生成するように指示する方法を学びます。
- ただし、ヒントはSQLステートメントを侵害的に変更します。一部のシナリオでは、ヒントを単純に挿入することができません。[SQLプラン管理](/jp/sql-plan-management)では、TiDBが別の構文を使用して実行計画の生成を非侵害的に制御する方法と、バックグラウンドでの自動実行計画進化の方法について説明します。この方法は、バージョンのアップグレードによって引き起こされる実行計画の不安定さやクラスタのパフォーマンス低下などの問題に対処するのに役立ちます。
- 最後に、[最適化ルールと式のプッシュダウンのブロックリスト](/jp/blocklist-control-plan)でブロックリストの使用方法を学びます。

<CustomContent platform="tidb">

上記の方法に加えて、実行計画はいくつかのシステム変数の影響を受けます。これらの変数をシステムレベルまたはセッションレベルで変更することで、実行計画の生成を制御できます。v7.1.0以降、TiDBは比較的特別な変数[`tidb_opt_fix_control`](https://docs.pingcap.com/tidb/v7.1/reference/system-variables/tidb_opt_fix_control)を導入しています。この変数は、オプティマイザの動作をより細かく制御し、クラスタのアップグレード後のオプティマイザの動作変更によるパフォーマンスの低下を防ぐために複数の制御項目を受け入れることができます。詳細については[Optimizer Fix Controls](https://docs.pingcap.com/tidb/v7.1/reference/optimizer-fix-controls)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

上記の方法に加えて、実行計画はいくつかのシステム変数の影響を受けます。これらの変数をシステムレベルまたはセッションレベルで変更することで、実行計画の生成を制御できます。v7.1.0以降、TiDBは比較的特別な変数[`tidb_opt_fix_control`](https://docs.pingcap.com/tidb/v7.1/reference/system-variables/tidb_opt_fix_control)を導入しています。この変数は、オプティマイザの動作をより細かく制御し、クラスタのアップグレード後のオプティマイザの動作変更によるパフォーマンスの低下を防ぐために複数の制御項目を受け入れることができます。詳細については[Optimizer Fix Controls](https://docs.pingcap.com/tidb/v7.2/reference/optimizer-fix-controls)を参照してください。

</CustomContent>