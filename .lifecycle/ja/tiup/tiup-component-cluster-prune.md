```yaml
---
title: tiupクラスターの削除
---

# tiupクラスターの削除

クラスターを[スケールイン](/tiup/tiup-component-cluster-scale-in.md)する際、一部のコンポーネントでは、TiUPは直ちにそのサービスを停止したりデータを削除したりしません。データのスケジューリングが完了するのを待ってから、手動で`tiup cluster prune`コマンドを実行してクリーンアップする必要があります。

## 構文

```shell
tiup cluster prune <クラスター名> [フラグ]
```

## オプション

### -h, --help

- ヘルプ情報を表示します。
- データタイプ：`BOOLEAN`
- デフォルト：false

## 出力

クリーンアッププロセスのログ。

[<< 前のページに戻る - TiUPクラスターコマンドリスト](/tiup/tiup-component-cluster.md#command-list)
```