---
title: TiDB Binlog用語集
summary: TiDB Binlogで使用される用語を学ぶ
aliases: ['/docs/dev/tidb-binlog/tidb-binlog-glossary/','/docs/dev/reference/tidb-binlog/glossary/']
---

# TiDB Binlog用語集

このドキュメントでは、TiDB Binlogのログ、監視、設定、およびドキュメントで使用される用語をリストしています。

## Binlog

TiDB Binlogでは、binlogとはTiDBからのバイナリログデータを指します。また、DrainerがKafkaまたはファイルに書き込むバイナリログデータも指します。前者と後者は異なる形式です。さらに、TiDBのbinlogとMySQLのbinlogも異なる形式です。

## Binlogイベント

TiDBのDML binlogには、`INSERT`、`UPDATE`、`DELETE`の3種類のイベントがあります。Drainerの監視ダッシュボードで、レプリケーションデータに対応する異なるイベントの数を確認できます。

## チェックポイント

チェックポイントは、レプリケーションタスクが一時停止および再開される位置を示し、または停止および再起動される位置を示します。Drainerが下流にレプリケートするcommit-tsを記録します。再開時、Drainerはチェックポイントを読み取り、対応するcommit-tsからデータのレプリケーションを開始します。

## セーフモード

セーフモードとは、増分レプリケーションタスクでテーブルスキーマにプライマリキーまたはユニークインデックスが存在する場合に、DMLの冪等性インポートをサポートするモードを指します。

このモードでは、`INSERT`文が`REPLACE`に書き換えられ、`UPDATE`文は`DELETE`および`REPLACE`に書き換えられます。その後、書き換えられた文が下流で実行されます。セーフモードは、Drainerが起動してから5分以内に自動的に有効になります。`safe-mode`パラメータを構成ファイルで手動で変更することでモードを有効にできますが、この構成は下流がMySQLまたはTiDBの場合のみ有効です。