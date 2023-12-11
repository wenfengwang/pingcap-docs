---
title: tiup mirror sign
---

# tiup mirror sign

`tiup mirror sign`コマンドは、TiUPの[mirror](/tiup/tiup-mirror-reference.md)で定義されたメタデータファイル（*.json）に署名するために使用されます。これらのメタデータファイルは、ローカルファイルシステムに格納されている場合と、HTTPプロトコルを使用してリモートに格納されている場合があります。

## 構文

```shell
tiup mirror sign <manifest-file> [flags]
```

`<manifest-file>`は署名するファイルのアドレスであり、次の2つの形式があります：

- HTTPまたはHTTPSで始まるネットワークアドレス。例：`http://172.16.5.5:8080/rotate/root.json`
- 相対パスまたは絶対パスのローカルファイルパス

ネットワークアドレスの場合、このアドレスは以下の機能を提供する必要があります：

- `http get`を介してアクセスすることで、署名済みファイルの完全なコンテンツ ( `signatures`フィールドを含む) を返すことができること。
- `http post`を介してアクセスすることで、クライアントが`http get`で返されるコンテンツの`signatures`フィールドに署名を追加し、このネットワークアドレスに投稿することができること。

## オプション

### -k, --key

- `{component}.json`ファイルに署名するために使用される秘密鍵の場所を指定します。
- データ型：`STRING`
- - もしコマンドでこのオプションが指定されていない場合、デフォルトで`"${TIUP_HOME}/keys/private.json"`が使用されます。

### --timeout

- ネットワークを介した署名のアクセスタイムアウト時間を指定します。単位は秒です。
- データ型: `INT`
- デフォルト: 10

> **注：**
>
> このオプションは、`<manifest-file>`がネットワークアドレスである場合にのみ有効です。

## 出力

- コマンドが正常に実行された場合、出力はありません。
- 指定されたキーによってファイルが署名されている場合、TiUPはエラー`Error: this manifest file has already been signed by specified key`を報告します。
- ファイルが有効なマニフェストでない場合、TiUPはエラー`Error: unmarshal manifest: %s`を報告します。

[<< 前のページに戻る - TiUP Mirrorコマンドリスト](/tiup/tiup-command-mirror.md#command-list)