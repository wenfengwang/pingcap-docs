---
title: TiDB データ移行コマンドラインフラグ
summary: DM のコマンドラインフラグについて学ぶ
---

# TiDB データ移行コマンドラインフラグ

このドキュメントでは、DM のコマンドラインフラグについて紹介します。

## DM-master

### `--advertise-addr`

- DM-master がクライアントリクエストを受信するために使用する外部アドレス
- デフォルト値は `"{master-addr}"`
- オプションフラグ。`"domain-name:port"` の形式であることができます

### `--advertise-peer-urls`

- DM-master ノード間の通信に使用する外部アドレス
- デフォルト値は `"{peer-urls}"`
- オプションフラグ。`"http(s)://domain-name:port"` の形式であることができます

### `--config`

- DM-master の構成ファイルパス
- デフォルト値は `""`
- オプションフラグ

### `--data-dir`

- DM-master のデータを保存するディレクトリ
- デフォルト値は `"default.{name}"`
- オプションフラグ

### `--initial-cluster`

- DM-master クラスタをブートストラップするために使用する `"{node name}={external address}"` リスト
- デフォルト値は `"{name}={advertise-peer-urls}"`
- このフラグは、`join` フラグが指定されていない場合に指定する必要があります。3ノードクラスタの構成例は、`"dm-master-1=http://172.16.15.11:8291,dm-master-2=http://172.16.15.12:8291,dm-master-3=http://172.16.15.13:8291"` です

### `--join`

- DM-master ノードがこのクラスタに参加する際に指定する既存クラスタの `advertise-addr` リスト
- デフォルト値は `""`
- このフラグは、`initial-cluster` フラグが指定されていない場合に指定する必要があります。新しいノードが2つのノードから成るクラスタに参加する場合の構成例は、`"172.16.15.11:8261,172.16.15.12:8261"` です

### `--log-file`

- ログの出力ファイル名
- デフォルト値は `""`
- オプションフラグ

### `-L`

- ログレベル
- デフォルト値は `"info"`
- オプションフラグ

### `--master-addr`

- DM-master がクライアントのリクエストを受け付けるアドレス
- デフォルト値は `""`
- 必須フラグ

### `--name`

- DM-master ノードの名前
- デフォルト値は `"dm-master-{hostname}"`
- 必須フラグ

### `--peer-urls`

- DM-master ノード間の通信に使用するリスニングアドレス
- デフォルト値は `"http://127.0.0.1:8291"`
- 必須フラグ

## DM-worker

### `--advertise-addr`

- DM-worker がクライアントリクエストを受信するために使用する外部アドレス
- デフォルト値は `"{worker-addr}"`
- オプションフラグ。`"domain-name:port"` の形式であることができます

### `--config`

- DM-worker の構成ファイルパス
- デフォルト値は `""`
- オプションフラグ

### `--join`

- DM-worker がこのクラスタに登録する際に指定するクラスタ内の DM-master ノードの `{advertise-addr}` リスト
- デフォルト値は `""`
- 必須フラグ。3ノード（DM-master ノード）クラスタの構成例は、`"172.16.15.11:8261,172.16.15.12:8261,172.16.15.13:8261"` です

### `--log-file`

- ログの出力ファイル名
- デフォルト値は `""`
- オプションフラグ

### `-L`

- ログレベル
- デフォルト値は `"info"`
- オプションフラグ

### `--name`

- DM-worker ノードの名前
- デフォルト値は `"{advertise-addr}"`
- 必須フラグ

### `--worker-addr`

- DM-worker がクライアントのリクエストを受け付けるアドレス
- デフォルト値は `""`
- 必須フラグ

## dmctl

### `--config`

- dmctl の構成ファイルパス
- デフォルト値は `""`
- オプションフラグ

### `--master-addr`

- dmctl が接続するクラスタ内の任意の DM-master ノードの `{advertise-addr}`
- デフォルト値は `""`
- dmctl が DM-master とやり取りする際に必須フラグです

### `--encrypt`

- 平文のデータベースパスワードを暗号化して、暗号文にする
- デフォルト値は `""`
- このフラグが指定された場合、DM-master とはやり取りせず、平文を暗号化するだけです

### `--decrypt`

- dmctl で暗号化された暗号文を復号化して、平文にする
- デフォルト値は `""`
- このフラグが指定された場合、DM-master とはやり取りせず、暗号文を復号化するだけです