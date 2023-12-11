---
title: tiup クラスター リプレイ
---

# tiup クラスター リプレイ

クラスターのアップグレードや再起動などのクラスター操作を行う際、クラスター環境の問題によって操作が失敗することがあります。もしもそのような操作を再実行する必要が生じた場合、最初から全ての手順を再実行する必要があります。クラスターが大規模な場合、これらの手順を再実行するには長い時間がかかります。このような場合、`tiup cluster replay` コマンドを使用して失敗したコマンドを再試行し、成功した手順をスキップすることができます。

## 構文

```shell
tiup cluster replay <audit-id> [flags]
```

- `<audit-id>`: 再試行するコマンドの `audit-id` です。過去のコマンドおよびそれらの `audit-id` は [`tiup cluster audit`](/tiup/tiup-component-cluster-audit.md) コマンドを使用して表示することができます。

## オプション

### -h, --help

ヘルプ情報を表示します。

## 出力

`<audit-id>` に対応するコマンドの出力。

[<< 前のページに戻る - TiUP クラスター コマンド一覧](/tiup/tiup-component-cluster.md#command-list)