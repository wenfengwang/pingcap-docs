---
title: tiup dm replay
---

# tiup dm 再生

クラスタのアップグレードや再起動などのクラスタ操作を実行すると、クラスタ環境の問題による操作の失敗が発生することがあります。もし操作を再実行する必要がある場合、最初からすべての手順を実行する必要があります。クラスタが大規模な場合、これらの手順を再実行するのには長い時間がかかります。このような場合、`tiup dm replay`コマンドを使用して失敗したコマンドを再試行し、成功した手順はスキップすることができます。

## 構文

```shell
tiup dm replay <監査ID> [フラグ]
```

- `<audit-id>`: 再試行するコマンドの`監査ID`。過去のコマンドとその`監査ID`は、[`tiup dm audit`](/tiup/tiup-component-dm-audit.md)コマンドを使用して表示できます。

## オプション

### -h, --help

ヘルプ情報を表示します。

## 出力

`<audit-id>`に対応するコマンドの出力。

[<<前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)