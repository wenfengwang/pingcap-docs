---
title: TiDB 2.0.1 リリースノート
aliases: ['/docs/dev/releases/release-2.0.1/','/docs/dev/releases/201/']
---

# TiDB 2.0.1 リリースノート

2018年5月16日にTiDB 2.0.1がリリースされました。このリリースはTiDB 2.0.0 (GA) と比較して、MySQL互換性とシステムの安定性に大幅な改善が行われています。

## TiDB

- `DDLジョブ情報に「索引の追加」の進捗をリアルタイムで更新するように変更
- `tidb_auto_analyze_ratio`セッション変数を追加し、自動統計情報の更新の閾値値を制御する
- トランザクションのコミットに失敗した際に残留する状態がすべてクリーンアップされない問題を修正
- 特定の条件下でのインデックスの追加に関するバグを修正
- DDLが同時実行される場合の操作の正確性に関する問題を修正
- 特定の条件下での`LIMIT`の結果が正しくない問題を修正
- `ADMIN CHECK INDEX`ステートメントのインデックス名を大文字小文字を区別しないように修正
- `UNION`ステートメントの互換性の問題を修正
- `TIME`型のデータ挿入時の互換性の問題を修正
- 特定の条件下での`copIteratorTaskSender`によるゴルーチンリークの問題を修正
- TiDBがBinlogの失敗の挙動を制御するオプションを追加
- `Coprocessor`スローログをリファクタし、処理時間が長いタスクと待機時間が長いタスクを区別するように変更
- MySQLプロトコルのハンドシェイクエラーが発生した際にログを出力しないようにし、負荷分散装置のKeep Aliveメカニズムによる大量のログ出力を避ける
- `Out of range value for column`エラーメッセージを改良
- `Update`ステートメントにサブクエリが含まれている場合のバグを修正
- `SIGTERM`の処理方法を変更し、すべてのクエリの終了を待機しないように変更

## PD

- 指定されたキー範囲を持つリージョンをバランスさせる「Scatter Range」スケジューラを追加
- マージリージョンのスケジューリングを最適化し、新しく分割されたリージョンがマージされないようにする
- レプリカ関連のメトリクスを追加
- リスタート後に誤ってスケジューラが削除される問題を修正
- 構成ファイルの解析時に発生するエラーを修正
- EtcdリーダーとPDリーダーの非同期レプリケーションされない問題を修正
- クローズ後もレプリカが表示される問題を修正
- パケットサイズが大きすぎるためにリージョンの読み込みに失敗する問題を修正

## TiKV

- `SELECT FOR UPDATE`が他の読み取りを妨げる問題を修正
- スロークエリログを最適化
- `thread_yield`呼び出し回数を減らす
- スナップショットの生成時にraftstoreがブロックされる問題を修正
- 特定の条件下でLearnerが正常に選出されない問題を修正
- 極端な条件下でのスプリットによるダーティリードの原因となる問題を修正
- 読み取りスレッドプール構成のデフォルト値を修正
- Delete Rangeの処理を高速化