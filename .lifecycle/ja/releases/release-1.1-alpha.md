---
title: TiDB 1.1 アルファ版リリースノート
aliases: ['/docs/dev/releases/release-1.1-alpha/','/docs/dev/releases/11alpha/']

# TiDB 1.1 アルファ版リリースノート

2018年1月19日、TiDB 1.1 アルファ版がリリースされました。このリリースには、MySQL 互換性、SQL の最適化、安定性、およびパフォーマンスの大幅な改善があります。

## TiDB

- SQL パーサー
    - より多くの構文をサポート
- SQL クエリオプティマイザ
    - 統計情報のメモリ使用量を削減するためにコンパクトな構造を使用
    - tidb-server の起動時に統計情報を高速に読み込む
    - より正確なクエリコスト評価を提供
    - ページ数・カウント・スケッチを使用して、一意のインデックスを使用するクエリのコストをより正確に見積もる
    - インデックスを最大限に活用するためのより複雑な条件をサポート
- SQL 実行エンジン
    - チャンクアーキテクチャを使用してすべての実行オペレーターを再構築し、解析ステートメントの実行パフォーマンスを向上させ、メモリ使用量を削減
    - `INSERT IGNORE` ステートメントのパフォーマンスを最適化
    - より多くの種類と機能を TiKV にプッシュダウン
    - より多くの `SQL_MODE` をサポート
    - `Load Data` パフォーマンスを最適化し、スピードを10倍に向上させる
    - `Use Database` パフォーマンスを最適化
    - 物理演算子のメモリ使用量の統計をサポート
- サーバー
    - PROXY プロトコルをサポート

## PD

- より多くの API を追加
- TLS をサポート
- 複数の Region サイズに適応するためのスケジューリングシミュレータの追加
- 異なるスケジューリングに関するいくつかのバグを修正

## TiKV

- Raft learner をサポート
- Raft Snapshot を最適化し、I/O 負荷を削減
- TLS をサポート
- パフォーマンスを向上させるために RocksDB 設定を最適化
- Coprocessor における `count (*)` および一意のインデックスのクエリパフォーマンスを最適化
- より多くのフェイルポイントと安定性テストケースを追加
- PD と TiKV 間の再接続問題を解決
- データリカバリーツール `tikv-ctl` の機能を強化
- Region 内のテーブルによる分割をサポート
- `Delete Range` 機能をサポート
- スナップショットによる I/O 制限の設定をサポート
- フロー制御メカニズムを改良