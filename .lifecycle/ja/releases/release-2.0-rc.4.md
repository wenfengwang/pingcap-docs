---
title: TiDB 2.0 RC4リリースノート
aliases: ['/docs/dev/releases/release-2.0-rc.4/','/docs/dev/releases/2rc4/']
---

# TiDB 2.0 RC4リリースノート

2018年3月30日、TiDB 2.0 RC4がリリースされました。このリリースでは、MySQL互換性、SQLの最適化、および安定性に大幅な改善が加えられています。

## TiDB

- `SHOW GRANTS FOR CURRENT_USER();`のサポート
- `UnionScan`内の`Expression`がクローンされない問題を修正
- `SET TRANSACTION`構文のサポート
- `copIterator`の潜在的なgoroutineリークの問題を修正
- `admin check table`がnullを含む一意のインデックスを誤判断する問題を修正
- 浮動小数点数の表示に科学的表記法をサポート
- 2進リテラル計算中に型推論の問題を修正
- `CREATE VIEW`ステートメントの解析の問題を修正
- `ORDER BY`と`LIMIT 0`の両方を含むステートメントがパニックする問題を修正
- `DecodeBytes`の実行パフォーマンスを向上
- `LIMIT 0`を`TableDual`に最適化し、無意味な実行計画の構築を回避

## PD

- 単一のRegionでホットスポットを処理するためにRegionを手動で分割するサポート
- `pdctl`が`config show all`を実行するとラベルプロパティが表示されない問題を修正
- メトリクスとコード構造を最適化

## TiKV

- スナップショットの受信中のメモリ使用量を制限し、極端な状況でのOOMを回避
- Coprocessorが警告に遭遇したときの振る舞いを構成するサポート
- TiKV内のデータパターンのインポートをサポート
- 中間でRegionを分割するサポート
- CIテストの速度を向上
- `crossbeam channel`の使用
- TiKVが孤立したときにリーダーが欠落して多くのログが出力される問題を修正