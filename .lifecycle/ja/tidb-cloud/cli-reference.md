---
title: TiDB Cloud CLI リファレンス
summary: TiDB Cloud CLI の概要を提供します。
---

# TiDB Cloud CLI リファレンス

TiDB Cloud CLI は、ターミナルからわずか数行のコマンドで TiDB Cloud を操作できるコマンドラインインターフェースです。TiDB Cloud CLI では、TiDB Cloud クラスターを簡単に管理したり、クラスターにデータをインポートしたり、さまざまな操作を行ったりできます。

## 開始する前に

最初に[TiDB Cloud CLI 環境をセットアップ](/tidb-cloud/get-started-with-cli.md)してください。`ticloud` CLI をインストールしたら、コマンドラインから TiDB Cloud クラスターを管理できます。

## 使用可能なコマンド

次の表に、TiDB Cloud CLI で使用可能なコマンドが一覧されています。

ターミナルで `ticloud [command] [subcommand]` を実行して `ticloud` CLI を使用するか、[TiUP](https://docs.pingcap.com/tidb/stable/tiup-overview)を使用している場合は代わりに`tiup cloud [command] [subcommand]`を実行します。

| コマンド    | サブコマンド                                           | 説明                                                                   |
|------------|-------------------------------------------------------|-------------------------------------------------------------------------|
| cluster    | create, delete, describe, list, connect-info           | クラスターを管理する                                                 |
| branch     | create, delete, describe, list, connect-info           | ブランチを管理する                                                   |
| completion | bash, fish, powershell, zsh                           | 指定されたシェル用の補完スクリプトを生成する                         |
| config     | create, delete, describe, edit, list, set, use        | ユーザープロファイルを構成する                                         |
| connect    | -                                                     | TiDB クラスターに接続する                                            |
| help       | cluster, completion, config, help, import, project, update | 任意のコマンドのヘルプを表示する                                       |
| import     | cancel, describe, list, start                         | [インポート](/tidb-cloud/tidb-cloud-migration-overview.md#ファイルから-tidb-cloud-へのデータのインポート)タスクを管理する  |
| project    | list                                                  | プロジェクトを管理する                                                 |
| update     | -                                                     | CLI を最新バージョンに更新する                                            |

## コマンドモード

TiDB Cloud CLI には、一部のコマンドに対して使用するための2つのモードがあります。

- インタラクティブモード

    フラグを指定せずにコマンドを実行でき、CLI が入力を要求します。

- インタラクティブモード以外

    コマンドを実行する際に必要なすべての引数とフラグを指定する必要があります。たとえば、`ticloud config create --profile-name <profile-name> --public-key <public-key> --private-key <private-key>`のようにです。

## ユーザープロファイル

TiDB Cloud CLI では、ユーザープロファイルとは、プロファイル名、公開鍵、およびプライベートキーなどのユーザーに関連するプロパティのコレクションです。TiDB Cloud CLI を使用するには、まずユーザープロファイルを作成する必要があります。

### ユーザープロファイルの作成

[`ticloud config create`](/tidb-cloud/ticloud-config-create.md) を使用してユーザープロファイルを作成します。

### すべてのユーザープロファイルを一覧表示する

[`ticloud config list`](/tidb-cloud/ticloud-config-list.md) を使用してすべてのユーザープロファイルを一覧表示します。

次は例です。

```
Profile Name
default (active)
dev
staging
```

この例の出力では、ユーザープロファイル `default` が現在アクティブです。

### ユーザープロファイルのプロパティを記述する

[`ticloud config describe`](/tidb-cloud/ticloud-config-describe.md) を使用してユーザープロファイルのプロパティを取得します。

次は例です。

```json
{
  "private-key": "xxxxxxx-xxx-xxxxx-xxx-xxxxx",
  "public-key": "Uxxxxxxx"
}
```

### ユーザープロファイルでプロパティを設定する

[`ticloud config set`](/tidb-cloud/ticloud-config-set.md) を使用してユーザープロファイルでプロパティを設定します。

### 他のユーザープロファイルに切り替える

[`ticloud config use`](/tidb-cloud/ticloud-config-use.md) を使用して他のユーザープロファイルに切り替えます。

次は例です。

```
Current profile has been changed to default
```

### 構成ファイルを編集する

[`ticloud config edit`](/tidb-cloud/ticloud-config-edit.md) を使用して構成ファイルを編集します。

### ユーザープロファイルを削除する

[`ticloud config delete`](/tidb-cloud/ticloud-config-delete.md) を使用してユーザープロファイルを削除します。

## グローバルフラグ

次の表に、TiDB Cloud CLI 用のグローバルフラグが一覧されています。

| フラグ               | 説明                                   | 必須     | 注釈                                                                                                                       |
|----------------------|----------------------------------------|----------|----------------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にする                 | いいえ   | 非インタラクティブモードのみ動作します。インタラクティブモードでは、一部の UI コンポーネントで無効にできない場合があります。|
| -P, --profile string | このコマンドで使用されるアクティブユーザープロファイルを指定します。 | いいえ   | インタラクティブモードと非インタラクティブモードの両方で動作します。                                                   |

## フィードバック

TiDB Cloud CLI に関するご質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、いかなる貢献も歓迎します。