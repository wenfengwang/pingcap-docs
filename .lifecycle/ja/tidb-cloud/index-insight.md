---
title: インデックスインサイト (ベータ)
summary: TiDB Cloud の Index Insight 機能を使用して、スローダウンしているクエリに対するインデックスの最適化を学びます。
---

# インデックスインサイト (ベータ)

TiDB Cloud のインデックスインサイト (ベータ) 機能は、効果的にインデックスを使用していないスローダウンしているクエリに対するインデックスの最適化を行うための強力な機能を提供します。このドキュメントでは、インデックスインサイト機能の効果的な有効化と活用方法について説明します。

> **注記:**
>
> インデックスインサイトは現在ベータ版であり、[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタでのみ利用できます。

## イントロダクション

インデックスインサイト機能には、以下の利点があります:

- クエリパフォーマンスの向上: インデックスインサイトはスローダウンしているクエリを特定し、適切なインデックスを提案することで、クエリ実行を高速化し、応答時間を短縮し、ユーザーエクスペリエンスを向上させます。
- コスト効率: インデックスインサイトを使用してクエリパフォーマンスを最適化することで、余分なコンピューティングリソースの必要性が減少し、既存のインフラストラクチャをより効果的に使用することができます。これにより、運用コストの削減が可能です。
- 最適化プロセスの簡素化: インデックスインサイトはインデックスの改善の特定と実装を簡素化し、手動の分析や推測の必要性をなくします。その結果、正確なインデックスの推奨事項により、時間と労力を節約することができます。
- アプリケーション効率の向上: インデックスインサイトを使用してデータベースのパフォーマンスを最適化することで、TiDB Cloud 上で動作するアプリケーションはより大きなワークロードを処理し、同時により多くのユーザーにサービスを提供することができます。これにより、アプリケーションのスケーリング操作がより効率的に行えます。

## 使用方法

このセクションでは、インデックスインサイト機能を有効にし、スローダウンしているクエリに対する推奨インデックスを取得する方法について説明します。

### 開始する前に

インデックスインサイト機能を有効にする前に、TiDB Dedicated クラスタを作成していることを確認してください。まだ作成されていない場合は、[TiDB Dedicated クラスタの作成](/tidb-cloud/create-tidb-cluster.md)に従って作成してください。

### ステップ 1: インデックスインサイトを有効にする

1. [TiDB Cloud コンソール](https://tidbcloud.com) にアクセスし、TiDB Dedicated クラスタのクラスタ概要ページに移動して、左側のナビゲーションパネルで **Diagnosis** をクリックします。

2. **Index Insight BETA** タブをクリックします。**Index Insight 概要**ページが表示されます。

3. インデックスインサイト機能を使用するには、フィーチャをトリガーし、インデックスの推奨を受け取るために、専用のSQLユーザーを作成する必要があります。以下のSQLステートメントを使用して、`'index_insight_user'` と `'random_password'` を自分の値に置き換えて、必要な権限を持つ新しいSQLユーザーを作成します。

    ```sql
    CREATE user 'index_insight_user'@'%' IDENTIFIED by 'random_password';
    GRANT SELECT ON information_schema.* TO 'index_insight_user'@'%';
    GRANT SELECT ON mysql.* TO 'index_insight_user'@'%';
    GRANT PROCESS, REFERENCES ON *.* TO 'index_insight_user'@'%';
    FLUSH PRIVILEGES;
    ```

    > **注記:**
    >
    > TiDB Dedicated クラスタに接続する方法については、[TiDB Dedicated クラスタへの接続](/tidb-cloud/connect-to-tidb-cluster.md)を参照してください。

4. 前のステップで作成したSQLユーザーのユーザー名とパスワードを入力し、**Activate** をクリックしてアクティベーションプロセスを開始します。

### ステップ 2: インデックスインサイトを手動でトリガする

スローダウンしているクエリに対するインデックスの推奨を受け取るために、**Index Insight 概要**ページの右上隅にある **Check Up** をクリックして、インデックスインサイト機能を手動でトリガすることができます。

その後、機能は過去3時間の間に発生したスローダウンしているクエリをスキャンし始めます。スキャンが完了すると、解析に基づいたインデックスの推奨事項のリストが表示されます。

### ステップ 3: インデックスの推奨事項の表示

特定のインデックスの推奨事項の詳細を表示するには、リストから情報をクリックします。**Index Insight 詳細** ページが表示されます。

このページでは、インデックスの推奨事項、関連するスローダウンしているクエリ、実行プラン、および関連するメトリックスを見つけることができます。この情報により、パフォーマンスの問題をよりよく理解し、インデックスの推奨事項を実装することの潜在的な影響を評価することができます。

### ステップ 4: インデックスの推奨事項の実装

インデックスの推奨事項を実装する前に、**Index Insight 詳細** ページから推奨事項を確認して評価する必要があります。

インデックスの推奨事項を実装するには、以下の手順に従ってください:

1. 提案されたインデックスが既存のクエリとワークロードに与える影響を評価します。
2. インデックスの実装に伴うストレージ要件や潜在的なトレードオフを考慮します。
3. 適切なデータベース管理ツールを使用して、関連するテーブルにインデックスの推奨事項を作成します。
4. インデックスを実装した後、パフォーマンスを監視して改善を評価します。

## ベストプラクティス

このセクションでは、インデックスインサイト機能の使用に関するベストプラクティスをいくつか紹介します。

### 定期的にインデックスインサイトをトリガ

最適化されたインデックスを維持するためには、毎日など定期的にインデックスインサイト機能をトリガすることをおすすめします。また、クエリやデータベーススキーマに重大な変更があった場合にもトリガすることをおすすめします。

### インデックスの実装前に影響を分析

インデックスの推奨事項を実装する前に、クエリの実行計画、ディスクスペース、および関連するトレードオフを分析してください。最も重要なパフォーマンス向上をもたらすインデックスを優先して実装してください。

### パフォーマンスの監視

インデックスの推奨事項を実装した後、定期的にクエリパフォーマンスを監視してください。これにより、パフォーマンスの改善を確認し、必要に応じてさらなる調整を行うことができます。

## FAQ

このセクションでは、インデックスインサイト機能に関するよくある質問をいくつか紹介します。

### インデックスインサイトを無効化する方法は？

インデックスインサイト機能を無効にするには、以下の手順を実行してください:

1. **Index Insight 概要**ページの右上隅にある **Settings** をクリックします。**Index Insight 設定** ページが表示されます。
2. **Deactivate** をクリックします。確認ダイアログボックスが表示されます。
3. **OK** をクリックして無効化を確認します。

    インデックスインサイト機能を無効にした後、**Index Insight 概要** ページからすべてのインデックスの推奨事項が削除されます。ただし、機能のために作成されたSQLユーザーは削除されません。SQLユーザーは手動で削除することができます。

### インデックスインサイトを無効化した後、SQLユーザーを削除する方法は？

インデックスインサイト機能を無効にした後、`DROP USER` ステートメントを実行して機能のために作成されたSQLユーザーを削除することができます。以下は例です。`'username'` を自分の値に置き換えてください。

```sql
DROP USER 'username';
```

### アクティベーションまたはチェックアップ中に `invalid user or password` メッセージが表示されるのはなぜですか？

`invalid user or password` メッセージは、システムが提供した資格情報を認証できない場合に表示される場合があります。これは、ユーザー名やパスワードが間違っている、有効期限切れやロックされたユーザーアカウントなどのさまざまな理由で発生する可能性があります。

この問題を解決するには、次の手順を実行してください:

1. 資格情報を確認します: 提供したユーザー名とパスワードが正しいことを確認してください。大文字と小文字を区別することに注意してください。
2. アカウントのステータスを確認します: ユーザーアカウントが有効であること、有効期限切れやロックされていないことを確認してください。システム管理者や関連するサポートチャネルに連絡して確認できます。
3. 新しいSQLユーザーを作成する: 上記のステップで問題が解決しない場合は、以下のステートメントを使用して新しいSQLユーザーを作成することができます。`'index_insight_user'` と `'random_password'` を自分の値に置き換えてください。

    ```sql
    CREATE user 'index_insight_user'@'%' IDENTIFIED by 'random_password';
    GRANT SELECT ON information_schema.* TO 'index_insight_user'@'%';
    GRANT SELECT ON mysql.* TO 'index_insight_user'@'%';
    GRANT PROCESS, REFERENCES ON *.* TO 'index_insight_user'@'%';
    FLUSH PRIVILEGES;
    ```

これらの手順に従っても問題が解決しない場合は、[PingCAP サポートチーム](/tidb-cloud/tidb-cloud-support.md)に連絡することをおすすめします。

### アクティベーションまたはチェックアップ中に `no sufficient privileges` メッセージが表示されるのはなぜですか？

`no sufficient privileges` メッセージは、提供されたSQLユーザーがインデックスインサイトからインデックスの推奨事項を要求するための必要な権限を持っていない場合に表示される場合があります。

この問題を解決するには、次の手順を実行してください:

1. ユーザーの権限を確認します: ユーザーアカウントが、`information_schema` と `mysql` の読み取り権限、およびすべてのデータベースに対する `PROCESS` および `REFERENCES` 権限を持っているか確認してください。

2. 新しいSQLユーザーを作成する: 上記の手順で問題が解決しない場合は、以下のステートメントを使用して新しいSQLユーザーを作成することができます。`'index_insight_user'` と `'random_password'` を自分の値に置き換えてください。

    ```sql
    CREATE user 'index_insight_user'@'%' IDENTIFIED by 'random_password';
    GRANT SELECT ON information_schema.* TO 'index_insight_user'@'%';
    GRANT SELECT ON mysql.* TO 'index_insight_user'@'%';
    GRANT PROCESS, REFERENCES ON *.* TO 'index_insight_user'@'%';
    FLUSH PRIVILEGES;
    ```

これらの手順に従っても問題が解決しない場合は、[PingCAP サポートチーム](/tidb-cloud/tidb-cloud-support.md)に連絡することをおすすめします。

### インデックスインサイトの使用中に `operations may be too frequent` メッセージが表示されるのはなぜですか？

`operations may be too frequent` メッセージは、インデックスインサイトが設定された頻度や使用制限を超えた場合に表示される場合があります。

この問題を解決するには、次の手順を実行してください:

1. 操作を遅らせます: このメッセージを受け取った場合は、インデックスインサイト上での操作頻度を減らす必要があります。
2. サポートに連絡する: 問題が解消しない場合は、[PingCAP サポートチーム](/tidb-cloud/tidb-cloud-support.md)に連絡し、エラーメッセージ、アクション、およびその他の関連情報を提供してください。
### インデックスインサイトを使用中に `内部エラー` メッセージが表示されるのはなぜですか？

`内部エラー` メッセージは、システムが予期せぬエラーや問題に遭遇した際に通常表示されます。このエラーメッセージは一般的であり、根本的な原因についての詳細は提供されません。

この問題を解決するために、次の手順を実行してください：

1. 操作を再試行します。ページを更新するか、操作をもう一度試してください。エラーは一時的なものであり、簡単な再試行によって解決できる場合があります。
2. サポートに連絡します。問題が解消しない場合、[PingCAP サポートチーム](/tidb-cloud/tidb-cloud-support.md)に連絡し、エラーメッセージ、行った操作、およびその他関連する情報を提供してください。