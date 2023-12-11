---
title: AIパワードのChat2Query（ベータ版）でデータを探索する
summary: Chat2Queryを使用して、TiDB Cloudコンソール内のAIパワードSQLエディタを使用して、データの価値を最大化する方法について学びます。

# AIパワードのChat2Query（ベータ版）でデータを探索する

TiDB CloudはAIで動作しています。[TiDB Cloudコンソール](https://tidbcloud.com/)のChat2Query（ベータ版）を使用すると、データの価値を最大化できます。

Chat2Queryでは、単に`--`を入力してから指示を入力することでAIにSQLクエリを自動生成させるか、SQLクエリを手動で記述し、ターミナルを使用せずにデータベースに対してSQLクエリを実行することができます。クエリの結果は直感的にテーブルで表示され、クエリログを簡単に確認できます。

> **注意:**
>
> Chat2Queryは、v6.5.0以降のTiDBクラスタでサポートされており、AWS上でホストされています。
>
> - [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタでは、Chat2Queryがデフォルトで利用可能です。
> - [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタでは、Chat2Queryはリクエストに応じてのみ利用可能です。TiDB DedicatedクラスタでChat2Queryを使用するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

## 使用ケース

Chat2Queryの推奨使用ケースは次のとおりです。

- Chat2QueryのAI機能を使用して複雑なSQLクエリを即座に生成する
- TiDBのMySQL互換性を素早くテストする
- TiDB SQL機能を簡単に探索する

## 制限

- AIによって生成されたSQLクエリは100%正確ではない場合があり、さらなる調整が必要になることがあります。
- [Chat2Query API](/tidb-cloud/use-chat2query-api.md)は[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタで利用可能です。[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタでChat2Query APIを使用するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

## Chat2Queryへのアクセス

1. プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動します。

    > **ヒント:**
    >
    > 複数のプロジェクトを持っている場合は、左下隅の<MDSvgIcon name="icon-left-projects" />をクリックして別のプロジェクトに切り替えることができます。

2. クラスタ名をクリックし、左側のナビゲーションペインで**Chat2Query**をクリックします。

    > **注意:**
    >
    > 次の場合、**Chat2Query**エントリがグレー表示されており、クリックできません。
    >
    > - TiDB Dedicatedクラスタがv6.5.0よりも前である場合。Chat2Queryを使用するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してクラスタをアップグレードする必要があります。
    > - TiDB Dedicatedクラスタが作成されたばかりで、Chat2Queryの実行環境がまだ準備中である場合。この場合は数分待ってからChat2Queryが利用可能になります。
    > - TiDB Dedicatedクラスタが[一時停止](/tidb-cloud/pause-or-resume-tidb-cluster.md)されている場合。

## AIにSQLクエリを生成させるか無効にする

PingCAPでは、ユーザーデータのプライバシーとセキュリティを最優先としています。 Chat2QueryのAI機能は、データ自体ではなくデータベースのスキーマにだけアクセスする必要があります。詳細については、[Chat2Query Privacy FAQ](https://www.pingcap.com/privacy-policy/privacy-chat2query)を参照してください。

Chat2Queryに初めてアクセスすると、PingCAPとOpenAIにコードスニペットを使用してサービスを改良するためのリサーチを許可するかどうかについてのダイアログが表示されます。

- AIによるSQLクエリの生成を許可するには、チェックボックスを選択して**保存して開始**をクリックします。
- AIによるSQLクエリの生成を無効にするには、このダイアログを直接閉じます。

初回アクセス後も、AIの設定を変更できます。

- AIを有効にするには、Chat2Queryの右上隅にある**AIパワーでデータを探索**をクリックします。
- AIを無効にするには、[TiDB Cloudコンソール](https://tidbcloud.com/)の左下隅の<MDSvgIcon name="icon-top-account-settings" />をクリックし、**アカウント設定**をクリックし、**プライバシー**タブをクリックしてから**AIパワードデータ探索**オプションを無効にします。

## SQLクエリの記述と実行

Chat2Queryでは、独自のデータセットを使用してSQLクエリを記述および実行できます。

1. SQLクエリを記述します。

    - AIが有効の場合は、単に`--`を入力してから指示を入力することでAIにSQLクエリを自動生成させるか、SQLクエリを手動で記述します。

        AIによって生成されたSQLクエリの場合、<kbd>Tab</kbd>を押してそれを受け入れ、必要に応じてさらに編集するか、<kbd>Esc</kbd>を押してそれを拒否します。

    - AIが無効の場合は、SQLクエリを手動で記述します。

2. SQLクエリを実行します。

    <SimpleTab>
    <div label="macOS">

    macOSの場合:

    - エディタにクエリが1つだけある場合、実行するには**⌘ + Enter**を押すか、<svg width="1rem" height="1rem" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M6.70001 20.7756C6.01949 20.3926 6.00029 19.5259 6.00034 19.0422L6.00034 12.1205L6 5.33028C6 4.75247 6.00052 3.92317 6.38613 3.44138C6.83044 2.88625 7.62614 2.98501 7.95335 3.05489C8.05144 3.07584 8.14194 3.12086 8.22438 3.17798L19.2865 10.8426C19.2955 10.8489 19.304 10.8549 19.3126 10.8617C19.4069 10.9362 20 11.4314 20 12.1205C20 12.7913 19.438 13.2784 19.3212 13.3725C19.307 13.3839 19.2983 13.3902 19.2831 13.4002C18.8096 13.7133 8.57995 20.4771 8.10002 20.7756C7.60871 21.0812 7.22013 21.0683 6.70001 20.7756Z" fill="currentColor"></path></svg>**実行**をクリックします。

    - エディタに複数のクエリがある場合、対象のクエリの行をカーソルで選択し、**⌘ + Enter**を押すか、**実行**をクリックしてそれらを順番に実行します。

    - エディタのすべてのクエリを順番に実行するには、**⇧ + ⌘ + Enter**を押すか、カーソルですべてのクエリの行を選択し、**実行**をクリックします。

    </div>

    <div label="Windows/Linux">

    WindowsまたはLinuxの場合:

    - エディタにクエリが1つだけある場合、実行するには**Ctrl + Enter**を押すか、<svg width="1rem" height="1rem" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M6.70001 20.7756C6.01949 20.3926 6.00029 19.5259 6.00034 19.0422L6.00034 12.1205L6 5.33028C6 4.75247 6.00052 3.92317 6.38613 3.44138C6.83044 2.88625 7.62614 2.98501 7.95335 3.05489C8.05144 3.07584 8.14194 3.12086 8.22438 3.17798L19.2865 10.8426C19.2955 10.8489 19.304 10.8549 19.3126 10.8617C19.4069 10.9362 20 11.4314 20 12.1205C20 12.7913 19.438 13.2784 19.3212 13.3725C19.307 13.3839 19.2983 13.3902 19.2831 13.4002C18.8096 13.7133 8.57995 20.4771 8.10002 20.7756C7.60871 21.0812 7.22013 21.0683 6.70001 20.7756Z" fill="currentColor"></path></svg>**実行**をクリックします。

    </div>
    </SimpleTab>
    </div>
```
- エディター内で複数のクエリを実行する場合は、対象のクエリ行をカーソルで選択し、次に**Ctrl + Enter**を押すか、**Run**をクリックします。

- エディター内のすべてのクエリを順次実行するには、**Shift + Ctrl + Enter**を押すか、すべてのクエリ行をカーソルで選択し、**Run**をクリックします。

クエリを実行した後、ページの一番下でクエリログと結果をすぐに確認できます。

## SQLファイルを管理する

Chat2Queryでは、異なるSQLファイルにSQLクエリを保存し、次のようにSQLファイルを管理できます:

- SQLファイルを追加するには、**SQL Files**タブの**+**をクリックします。
- SQLファイルの名前を変更するには、ファイル名にカーソルを合わせ、ファイル名の隣にある**...**をクリックし、**Rename**を選択します。
- SQLファイルを削除するには、ファイル名にカーソルを合わせ、ファイル名の隣にある**...**をクリックし、**Delete**を選択します。**SQL Files**タブにSQLファイルが1つしかない場合は削除できません。

## SQLファイルからエンドポイントを生成する

TiDBクラスターでは、TiDB Cloudは[データサービス（ベータ版）](/tidb-cloud/data-service-overview.md)機能を提供し、カスタムAPIエンドポイントを使用してHTTPSリクエストを介してTiDB Cloudデータにアクセスできます。Chat2Queryでは、次の手順に従って、SQLファイルからデータサービス（ベータ版）のエンドポイントを生成できます:

1. ファイル名にカーソルを合わせ、ファイル名の隣にある**...**をクリックし、**Generate endpoint**を選択します。
2. **Generate endpoint**ダイアログボックスで、エンドポイントを生成したいデータアプリを選択し、エンドポイント名を入力します。
3. **Generate**をクリックします。エンドポイントが生成され、その詳細ページが表示されます。

詳細については、[エンドポイントの管理](/tidb-cloud/data-service-manage-endpoint.md)を参照してください。

## Chat2Queryの設定を管理する

Chat2Queryでは、次の設定を変更できます:

- クエリ結果の最大行数
- **Schemas**タブでシステムデータベーススキーマを表示するかどうか
- [Chat2Query API](/tidb-cloud/use-chat2query-api.md)を有効にするかどうか

設定を変更するには、次の手順を実行します:

1. Chat2Queryの右上隅で**...**をクリックし、**Settings**を選択します。
2. 必要に応じて設定を変更します。
3. **Save**をクリックします。