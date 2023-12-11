---
title: TiDB 1.0リリースノート
aliases: ['/docs/dev/releases/release-1.0-ga/','/docs/dev/releases/ga/']
---

# TiDB 1.0リリースノート

2017年10月16日、TiDB 1.0がリリースされました！このリリースでは、MySQL互換性、SQLの最適化、安定性、およびパフォーマンスに焦点を当てています。

## TiDB

- SQLクエリの最適化:
    - コストモデルの調整
    - 解析プッシュダウン
    - 関数シグネチャのプッシュダウン
- インターナルデータ形式を最適化して、中間データサイズを削減
- MySQL互換性を強化
- ストレージエンジンでの`NO_SQL_CACHE`構文のサポートとキャッシュ使用量の制限
- ハッシュアグリゲータオペレータをリファクタリングしてメモリ使用量を削減
- ストリームアグリゲータオペレータをサポート

## PD

- 読み取りフローに基づくバランシングをサポート
- ストアの重みと重みに基づくバランシングの設定をサポート

## TiKV

- Coprocessorはより多くのプッシュダウン関数をサポート
- サンプリング操作のプッシュダウンをサポート
- 空間の迅速な収集のために、手動でデータのコンパクションをトリガーするサポート
- パフォーマンスと安定性を向上
- デバッグのためのDebug APIを追加
- TiSparkベータリリース:
  - 構成フレームワークのサポート
  - ThriftSever/JDBCおよびSpark SQLのサポート

## 謝辞

### 以下の企業およびチームに特別な感謝を捧げます

- Archon
- Mobike
- Samsung Electronics
- SpeedyCloud
- Tencent Cloud
- UCloud

### 以下の組織および個人からのオープンソースソフトウェアとサービスに感謝します

- Asta Xie
- CNCF
- CoreOS
- Databricks
- Docker
- Github
- Grafana
- gRPC
- Jepsen
- Kubernetes
- Namazu
- Prometheus
- RedHat
- RocksDB Team
- Rust Team

### 個々の貢献者に感謝します

（略）