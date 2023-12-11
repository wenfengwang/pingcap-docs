---
title: DM DDLの特別な処理
summary: DMがDDL文を解析し、文のタイプに応じて処理する方法を学ぶ
---

# DM DDLの特別な処理

TiDBデータ移行（DM）がデータを移行する際、DDL文を解析し、文のタイプと現在の移行ステージに応じて処理します。

## DDL文のスキップ

以下の文はDMでサポートされていないため、解析後に直接スキップされます。

<table>
    <tr>
        <th>説明</th>
        <th>SQL</th>
    </tr>
    <tr>
        <td>トランザクション</td>
        <td><code>^SAVEPOINT</code></td>
    </tr>
    <tr>
        <td>全てのFLUSH SQLのスキップ</td>
        <td><code>^FLUSH</code></td>
    </tr>
    <tr>
        <td rowspan="3">テーブルのメンテナンス</td>
        <td><code>^OPTIMIZE\\s+TABLE</code></td>
    </tr>
    <tr>
        <td><code>^ANALYZE\\s+TABLE</code></td>
    </tr>
    <tr>
        <td><code>^REPAIR\\s+TABLE</code></td>
    </tr>
    <tr>
        <td>一時テーブル</td>
        <td><code>^DROP\\s+(\\/\\*\\!40005\\s+)?TEMPORARY\\s+(\\*\\/\\s+)?TABLE</code></td>
    </tr>
    <tr>
        <td rowspan="2">トリガー</td>
        <td><code>^CREATE\\s+(DEFINER\\s?=.+?)?TRIGGER</code></td>
    </tr>
    <tr>
        <td><code>^DROP\\s+TRIGGER</code></td>
    </tr>
    <tr>
        <td rowspan="3">プロシージャ</td>
        <td><code>^DROP\\s+PROCEDURE</code></td>
    </tr>
    <tr>
        <td><code>^CREATE\\s+(DEFINER\\s?=.+?)?PROCEDURE</code></td>
    </tr>
    <tr>
        <td><code>^ALTER\\s+PROCEDURE</code></td>
    </tr>
    <tr>
        <td rowspan="3">ビュー</td>
        <td><code>^CREATE\\s*(OR REPLACE)?\\s+(ALGORITHM\\s?=.+?)?(DEFINER\\s?=.+?)?\\s+(SQL SECURITY DEFINER)?VIEW</code></td>
    </tr>
    <tr>
        <td><code>^DROP\\s+VIEW</code></td>
    </tr>
    <tr>
        <td><code>^ALTER\\s+(ALGORITHM\\s?=.+?)?(DEFINER\\s?=.+?)?(SQL SECURITY DEFINER)?VIEW</code></td>
    </tr>
    <tr>
        <td rowspan="4">関数</td>
        <td><code>^CREATE\\s+(AGGREGATE)?\\s*?FUNCTION</code></td>
    </tr>
    <tr>
        <td><code>^CREATE\\s+(DEFINER\\s?=.+?)?FUNCTION</code></td>
    </tr>
    <tr>
        <td><code>^ALTER\\s+FUNCTION</code></td>
    </tr>
    <tr>
        <td><code>^DROP\\s+FUNCTION</code></td>
    </tr>
    <tr>
        <td rowspan="3">テーブルスペース</td>
        <td><code>^CREATE\\s+TABLESPACE</code></td>
    </tr>
    <tr>
        <td><code>^ALTER\\s+TABLESPACE</code></td>
    </tr>
    <tr>
        <td><code>^DROP\\s+TABLESPACE</code></td>
    </tr>
    <tr>
        <td rowspan="3">イベント</td>
        <td><code>^CREATE\\s+(DEFINER\\s?=.+?)?EVENT</code></td>
    </tr>
    <tr>
        <td><code>^ALTER\\s+(DEFINER\\s?=.+?)?EVENT</code></td>
    </tr>
    <tr>
        <td><code>^DROP\\s+EVENT</code></td>
    </tr>
    <tr>
        <td rowspan="7">アカウント管理</td>
        <td><code>^GRANT</code></td>
    </tr>
    <tr>
        <td><code>^REVOKE</code></td>
    </tr>
    <tr>
        <td><code>^CREATE\\s+USER</code></td>
    </tr>
    <tr>
        <td><code>^ALTER\\s+USER</code></td>
    </tr>
    <tr>
        <td><code>^RENAME\\s+USER</code></td>
    </tr>
    <tr>
        <td><code>^DROP\\s+USER</code></td>
    </tr>
    <tr>
        <td><code>^DROP\\s+USER</code></td>
    </tr>
</table>

## DDL文の書き換え

以下の文は、下流にレプリケーションされる前に書き換えられます。

|オリジナル文|書き換えられた文|
|-|-|
|`^CREATE DATABASE...`|`^CREATE DATABASE...IF NOT EXISTS`|
|`^CREATE TABLE...`|`^CREATE TABLE..IF NOT EXISTS`|
|`^DROP DATABASE...`|`^DROP DATABASE...IF EXISTS`|
|`^DROP TABLE...`|`^DROP TABLE...IF EXISTS`|
|`^DROP INDEX...`|`^DROP INDEX...IF EXISTS`|

## シャード統合移行タスク

DMが悲観的または楽観的モードでテーブルをマージおよび移行する場合、DDLのレプリケーションの動作は他のシナリオとは異なります。詳細については、[悲観的モード](/dm/feature-shard-merge-pessimistic.md) および [楽観的モード](/dm/feature-shard-merge-optimistic.md) を参照してください。

## オンラインDDL

オンラインDDL機能も、DDLイベントを特別な方法で処理します。詳細については、[GH-ost/PT-oscを使用するデータベースからの移行](/dm/feature-online-ddl.md) を参照してください。