---
title: DM-masterの構成ファイル
summary: DM-masterの構成ファイルについて学ぶ
aliases: ['/docs/tidb-data-migration/dev/dm-master-configuration-file/']
---

# DM-masterの構成ファイル

本書では、DM-masterの構成について、構成ファイルのテンプレートとそのファイル内の各構成パラメータの説明を紹介します。

## 構成ファイルのテンプレート

以下は、DM-masterの構成ファイルのテンプレートです。

```toml
name = "dm-master"

# ログの構成
log-level = "info"
log-file = "dm-master.log"

# DM-masterのリスニングアドレス
master-addr = ":8261"
advertise-addr = "127.0.0.1:8261"

# ピアトラフィック用のURL
peer-urls = "http://127.0.0.1:8291"
advertise-peer-urls = "http://127.0.0.1:8291"

# クラスタの構成
initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
join = ""

ssl-ca = "/path/to/ca.pem"
ssl-cert = "/path/to/cert.pem"
ssl-key = "/path/to/key.pem"
cert-allowed-cn = ["dm"] 
```

## 構成パラメータ

このセクションでは、DM-masterの構成パラメータについて紹介します。

### グローバル構成

| パラメータ     | 説明                                  |
| :------------ | :--------------------------------------- |
| `name` | DM-masterの名称です。 |
| `log-level` | `debug`、`info`、`warn`、`error`、`fatal`からログレベルを指定します。デフォルトのログレベルは`info`です。 |
| `log-file` | ログファイルのディレクトリを指定します。このパラメータが指定されていない場合、ログは標準出力に出力されます。 |
| `master-addr` | サービスを提供するDM-masterのアドレスを指定します。IPアドレスを省略し、ポート番号のみを指定することもできます（例: ":8261"）。 |
| `advertise-addr` | DM-masterが外部に公表するアドレスを指定します。 |
| `peer-urls` | DM-masterノードのピアURLを指定します。 |
| `advertise-peer-urls` | DM-masterが外部に公表するピアURLを指定します。`advertise-peer-urls`の値はデフォルトで`peer-urls`の値と同じです。 |
| `initial-cluster` | `initial-cluster`の値は、初期クラスタのすべてのDM-masterノードの`advertise-peer-urls`の値の組み合わせです。 |
| `join` | `join`の値は、クラスタ内の既存のDM-masterノードの`advertise-peer-urls`の値の組み合わせです。DM-masterノードが新しく追加された場合、`initial-cluster`を`join`で置き換えます。 |
| `ssl-ca` | DM-masterが他のコンポーネントと接続するための信頼されるSSL CAのリストを含むファイルのパスです。 |
| `ssl-cert` | DM-masterが他のコンポーネントと接続するためのPEM形式のX509証明書を含むファイルのパスです。 |
| `ssl-key` | DM-masterが他のコンポーネントと接続するためのPEM形式のX509キーを含むファイルのパスです。 |
| `cert-allowed-cn` | コモンネームのリストです。 |