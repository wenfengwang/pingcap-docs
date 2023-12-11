---
title: tiup mirror set
---

# tiup mirror set

`tiup mirror set`コマンドは、現在のミラーを切り替えるために使用され、ローカルファイルシステムとリモートネットワークアドレスの2つのフォームのミラーをサポートします。

公式ミラーのアドレスは`https://tiup-mirrors.pingcap.com`です。

## 構文

```shell
tiup mirror set <mirror-addr> [flags]
```

`<mirror-addr>`は、2つの形式があります。

- ネットワークアドレス: `http`または`https`で始まります。例: `http://172.16.5.5:8080`、 `https://tiup-mirrors.pingcap.com`.
- ローカルファイルパス: ミラーディレクトリの絶対パス。例: `/path/to/local-tiup-mirror`。

## オプション

### -r, --root

このオプションはルート証明書を指定します。

ミラーセキュリティの最も重要な部分であるルート証明書は、それぞれ異なります。ネットワークミラーを使用すると、中間者攻撃に遭う可能性があります。そのような攻撃を避けるためには、ルートネットワークミラーのルート証明書を手動でローカルにダウンロードすることをお勧めします:

```
wget <mirror-addr>/root.json -O /path/to/local/root.json
```

ルート証明書が正しいことを手動で確認し、その後、ルート証明書を手動で指定してミラーを切り替えます:

```
tiup mirror set <mirror-addr> -r /path/to/local/root.json
```

上記の手順で、`wget`コマンドの前にミラーが攻撃された場合、ルート証明書が間違っていることがわかります。`wget`コマンドの後にミラーが攻撃された場合、TiUPはミラーがルート証明書と一致しないことがわかります。

- データタイプ: `String`
- デフォルト: `{mirror-dir}/root.json`

## 出力

なし

[<<前のページに戻る - TiUPミラーコマンドリスト](/tiup/tiup-command-mirror.md#command-list)