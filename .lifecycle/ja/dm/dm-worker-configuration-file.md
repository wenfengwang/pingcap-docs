---
title: DM-workerの構成ファイル
summary: DM-workerの構成ファイルについて学ぶ
aliases: ['/docs/tidb-data-migration/dev/dm-worker-configuration-file/', '/docs/tidb-data-migration/dev/dm-worker-configuration-file-full/']
---

# DM-workerの構成ファイル

このドキュメントでは、DM workerの構成について、構成ファイルのテンプレートとこのファイル内の各構成パラメータの説明を紹介します。

## 構成ファイルのテンプレート

以下はDM-workerの構成ファイルのテンプレートです：

```toml
# Workerの構成。
name = "worker1"

# ログの構成。
log-level = "info"
log-file = "dm-worker.log"

# DM-workerのリッスンアドレス。
worker-addr = ":8262"
advertise-addr = "127.0.0.1:8262"
join = "http://127.0.0.1:8261,http://127.0.0.1:8361,http://127.0.0.1:8461"

keepalive-ttl = 60
relay-keepalive-ttl = 1800 # DM v2.0.2で追加された項目。
# relay-dir = "relay_log" # 5.4.0で追加された項目。相対パスを使用する場合は、DM-workerのデプロイと開始方法を確認してフルパスを決定してください。

ssl-ca = "/path/to/ca.pem"
ssl-cert = "/path/to/cert.pem"
ssl-key = "/path/to/key.pem"
cert-allowed-cn = ["dm"]
```

## 構成パラメータ

### グローバル

| パラメーター | 説明 |
| :------------ | :--------------------------------------- |
| `name` | DM-workerの名前。 |
| `log-level` | `debug`、`info`、`warn`、`error`、`fatal`からログレベルを指定します。デフォルトのログレベルは`info`です。 |
| `log-file` | ログファイルのディレクトリを指定します。このパラメーターが指定されていない場合、ログは標準出力に出力されます。 |
| `worker-addr` | サービスを提供するDM-workerのアドレスを指定します。IPアドレスを省略し、ポート番号のみを指定することもできます（例：":8262"）。 |
| `advertise-addr` | DM-workerが外部に公表するアドレスを指定します。 |
| `join` | DM-master構成ファイルの1つまたは複数の[`master-addr`](/dm/dm-master-configuration-file.md#global-configuration)に対応します。 |
| `keepalive-ttl` | DM-workerノードが上流データソースで中継ログを有効にしていない場合の、DM-workerノードからDM-masterノードへのキープアライブ時間（秒単位）。デフォルト値は60秒です。|
| `relay-keepalive-ttl` | DM-workerノードが上流データソースで中継ログを有効にした場合の、DM-workerノードからDM-masterノードへのキープアライブ時間（秒単位）。デフォルト値は1800秒です。このパラメーターはDM v2.0.2以降で追加されました。|
| `relay-dir` | DM-workerがバインドされた上流データソースで中継ログが有効になっている場合、DM-workerはこのディレクトリに中継ログを保存します。このパラメーターはv5.4.0で新しく追加され、上流データソースの構成より優先されます。 |
| `ssl-ca` | DM-workerが他のコンポーネントとの接続に使用する信頼されたSSL CAのリストを含むファイルのパス。 |
| `ssl-cert` | DM-workerが他のコンポーネントとの接続に使用するPEM形式のX509証明書を含むファイルのパス。 |
| `ssl-key` | DM-workerが他のコンポーネントとの接続に使用するPEM形式のX509キーを含むファイルのパス。 |
| `cert-allowed-cn` | コモンネームのリスト。 |