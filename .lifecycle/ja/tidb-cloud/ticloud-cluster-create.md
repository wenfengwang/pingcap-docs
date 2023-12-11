---
title: ticloud cluster create
summary: `ticloud cluster create`の参照。

# ticloud cluster create

クラスタを作成します：

```shell
ticloud cluster create [flags]
```

> **注：**
>
> 現在、前述のコマンドを使用して、[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタのみを作成できます。

## 例

インタラクティブモードでクラスタを作成：

```shell
ticloud cluster create
```

非インタラクティブモードでクラスタを作成：

```shell
ticloud cluster create --project-id <project-id> --cluster-name <cluster-name> --cloud-provider <cloud-provider> --region <region> --root-password <password> --cluster-type <cluster-type>
```

## フラグ

非インタラクティブモードでは、必要なフラグを手動で入力する必要があります。インタラクティブモードでは、CLIのプロンプトに従ってそれらを入力するだけで済みます。

| フラグ                   | 説明                                                | 必須     | 注                             |
|-------------------------|-------------------------------------------------------------|----------|-----------------------------------|
| --cloud-provider string | クラウドプロバイダ（現在、`AWS`のみサポートされています）                           | Yes      | 非インタラクティブモードでのみ機能します。 |
| --cluster-name string   | 作成するクラスタの名前                           | Yes      | 非インタラクティブモードでのみ機能します。 |
| --cluster-type string   | クラスタタイプ（現在、`SERVERLESS`のみサポートされています）    | Yes      | 非インタラクティブモードでのみ機能します。 |
| -h, --help              | このコマンドのヘルプ情報を取得   | No       | 非インタラクティブおよびインタラクティブの両モードで機能します。     |
| -p, --project-id string | クラスタが作成されるプロジェクトのID | Yes      | 非インタラクティブモードでのみ機能します。 |
| -r, --region string     | クラウドリージョン                                                | Yes      | 非インタラクティブモードでのみ機能します。 |
| --root-password string  | クラスタのルートパスワード                            | Yes      | 非インタラクティブモードでのみ機能します。 |

## 継承されたフラグ

| フラグ                 | 説明                                                                               | 必須 | 注                                                                                       |
|----------------------|-------------------------------------------------------------------------------------------|----------|--------------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にします。                                                                  | No       | 非インタラクティブモードでのみ機能します。インタラクティブモードでは、一部のUIコンポーネントでカラーの無効化が機能しない場合があります。 |
| -P, --profile string | このコマンドで使用されるアクティブな[user profile](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | No       | 非インタラクティブおよびインタラクティブの両モードで機能します。                                                                      |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、どんな貢献も歓迎します。