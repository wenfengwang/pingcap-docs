---
title: gh-ostまたはpt-oscを使用するデータベースからの継続的なレプリケーション
summary: gh-ostまたはpt-oscを使用するデータベースからの増分データをDMを使用してレプリケーションする方法を学ぶ

---

# gh-ostまたはpt-oscを使用するデータベースからの継続的なレプリケーション

本番環境では、DDL実行中のテーブルロックによって、データベースへの読み取りや書き込みが一定程度ブロックされることがあります。そのため、オンラインDDLツールが使用され、DDLの実行による読み取りや書き込みへの影響を最小限に抑えます。一般的なDDLツールには、[gh-ost](https://github.com/github/gh-ost) および [pt-osc](https://www.percona.com/doc/percona-toolkit/3.0/pt-online-schema-change.html) があります。

MySQLからTiDBへのデータ移行にDMを使用する際は、DMとgh-ostまたはpt-oscの連携を可能にするために`online-ddl`を有効にできます。

詳しいレプリケーション手順については、以下のシナリオごとのドキュメントを参照してください。

- [MySQLからTiDBへの小規模データセットの移行](/migrate-small-mysql-to-tidb.md)
- [MySQLからTiDBへの大規模データセットの移行](/migrate-large-mysql-to-tidb.md)
- [小規模MySQLシャードのTiDBへの移行とマージ](/migrate-small-mysql-shards-to-tidb.md)
- [大規模MySQLシャードのTiDBへの移行とマージ](/migrate-large-mysql-shards-to-tidb.md)

## DMでのonline-ddlの有効化

DMのタスク構成ファイルで、グローバルパラメータ`online-ddl`を以下のように`true`に設定します。

```yaml
# ----------- Global configuration -----------
## ********* Basic configuration *********
name: test                      # タスクの名前。一意の名前にします。
task-mode: all                  # タスクモード。`full`、`incremental`、または`all`に設定できます。
shard-mode: "pessimistic"       # シャードのマージモード。オプションモードは`pessimistic`と`optimistic`です。`pessimistic`モードがデフォルトで使用されます。`optimistic`モードの原則と制限を理解した後、`optimistic`モードに設定できます。
meta-schema: "dm_meta"          # `meta`情報を格納するダウンストリームデータベース。
online-ddl: true                # upstreamデータベースの"gh-ost"および"pt-osc"の自動処理をサポートするため、DMでonline-ddlサポートを有効にします。
```

## online-ddlを有効にした後のワークフロー

DMでonline-ddlが有効になると、DMがgh-ostまたはpt-oscをレプリケートする際に生成されるDDLステートメントが変更されます。

gh-ostまたはpt-oscのワークフロー：

- DDLリアルテーブルのテーブルスキーマに基づいてゴーストテーブルを作成します。

- ゴーストテーブルにDDLを適用します。

- DDLリアルテーブルのデータをゴーストテーブルにレプリケートします。

- 2つのテーブル間でデータが一致した後、リネームステートメントを使用してリアルテーブルをゴーストテーブルで置き換えます。

DMのワークフロー：

- ダウンストリームでのゴーストテーブルの作成をスキップします。

- ダウンストリームで適用されたDDLを記録します。

- ゴーストテーブルからのデータのみをレプリケートします。

- ダウンストリームで記録されたDDLを適用します。

![dm-online-ddl](/media/dm/dm-online-ddl.png)

ワークフローの変更により、以下の利点がもたらされます。

- ダウンストリームのTiDBは、ゴーストテーブルを作成してレプリケートする必要がなくなり、ストレージスペースとネットワーク転送のオーバーヘッドを節約します。

- シャードされたテーブルからデータを移行およびマージする際に、レプリケーションの正確性を確保するため、各シャードのゴーストテーブルに対するRENAME操作が無視されます。

## 関連情報

[オンラインDDLツールとの連携に関するDMの作業詳細](/dm/feature-online-ddl.md#working-details-for-dm-with-online-ddl-tools)
