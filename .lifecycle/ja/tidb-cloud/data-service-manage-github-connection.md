---
title: GitHubを使用してデータアプリを自動デプロイする
summary: GitHubを使用してデータアプリを自動的にデプロイする方法について学びます。
---

# GitHubを使用してデータアプリを自動デプロイする

TiDB Cloudは、JSON構文を使用して、完全なデータアプリの構成をコードとして表現するConfiguration as Code (CaC)アプローチを提供します。

データアプリをGitHubに接続することで、TiDB CloudはCaCアプローチを使用し、好きなGitHubリポジトリとブランチにデータアプリの構成ファイルをプッシュすることができます。

GitHub接続で**自動同期＆デプロイ**が有効になっている場合、GitHub上でデータアプリの構成ファイルを更新することで、GitHubに構成ファイルの変更をプッシュすると、新しい構成は自動的にTiDB Cloudにデプロイされます。

このドキュメントでは、GitHubを使用してデータアプリを自動的にデプロイする方法と、GitHub接続の管理方法について紹介します。

## 開始する前に

データアプリをGitHubに接続する前に、以下のものを準備してください：

- GitHubアカウント
- ターゲットブランチを持つGitHubリポジトリ

> **注意:**
>
> GitHubリポジトリは、データアプリを接続した後にデータアプリの構成ファイルを保存するために使用されます。構成ファイル内の情報（クラスタIDやエンドポイントURLなど）が機密情報である場合は、公開リポジトリの代わりにプライベートリポジトリを使用してください。

## ステップ1. データアプリをGitHubに接続する

データアプリを作成する際にGitHubにデータアプリを接続することができます。詳細については[データアプリを作成する](/tidb-cloud/data-service-manage-data-app.md)を参照してください。

アプリ作成時にGitHub接続を有効にしなかった場合でも、以下の手順で有効にすることができます：

1. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページに移動します。
2. 左側のペインで、対象のデータアプリの名前をクリックして詳細を表示します。
3. **設定**タブで、**GitHubに接続**領域の**接続**をクリックします。接続設定用のダイアログボックスが表示されます。
4. ダイアログボックスで、以下の手順を実行します：

    1. **GitHubでインストール**をクリックし、対象リポジトリに**TiDB Cloud Data Service**をアプリとしてインストールするための画面に従います。
    2. **認可**をクリックしてGitHubでアプリにアクセス権を許可します。
    3. データアプリの構成ファイルを保存したい対象リポジトリ、ブランチ、ディレクトリを指定します。

        > **注:**
        >
        > - ディレクトリはスラッシュ（`/`）で始まる必要があります。例: `/mydata`。指定したディレクトリが対象リポジトリとブランチに存在しない場合、自動的に作成されます。
        > - リポジトリ、ブランチ、ディレクトリの組み合わせは、データアプリ間で一意である必要があります。指定したパスが他のデータアプリで使用されている場合は、代わりのパスを指定する必要があります。そうでない場合、TiDB Cloudコンソールで現在のデータアプリのエンドポイントを設定すると、指定したパスのファイルが上書きされます。
        > - 指定したパスに別のデータアプリからコピーした構成ファイルが含まれており、これらのファイルを現在のデータアプリにインポートしたい場合は、[既存のデータアプリの構成をインポートする](#既存のデータアプリの構成をインポートする)を参照してください。

    4. TiDB CloudコンソールまたはGitHubで行われたデータアプリの変更が互いに同期されるようにするには、**自動同期＆デプロイを構成**を有効にします。

        - 有効にすると、指定したGitHubディレクトリで行われた変更はTiDB Cloudで自動的にデプロイされ、TiDB Cloudコンソールで行われた変更はGitHubにプッシュされます。データアプリのデプロイ履歴に対応するデプロイメントとコミット情報が表示されます。
        - 無効にすると、指定したGitHubディレクトリで行われた変更はTiDB Cloudにはデプロイされず、TiDB Cloudコンソールで行われた変更もGitHubにプッシュされません。

5. **接続を確認**をクリックします。

## ステップ2. データアプリの構成をGitHubと同期する

データアプリのGitHub接続が有効になっている場合、[データアプリを作成](/tidb-cloud/data-service-manage-data-app.md)した後にTiDB Cloudは直ちにデータアプリの構成ファイルをGitHubにプッシュします。

アプリ作成後にGitHub接続が有効になった場合は、データアプリの構成をGitHubと同期するためにデプロイ操作を行う必要があります。例えば、**デプロイメント**タブをクリックして、このデータアプリのデプロイメントを再デプロイすることができます。

デプロイ操作後に、指定したGitHubディレクトリを確認すると、`tidb-cloud-data-service`によってデータアプリの構成ファイルがディレクトリにコミットされ、データアプリがGitHubに接続されたことを意味します。ディレクトリ構造は以下のようになります：

```
├── <あなたのデータアプリのGitHubディレクトリ>
│   ├── data_sources
│   │   └── cluster.json  # 関連付けられたクラスタを指定します。
│   ├── dataapp_config.json # Data APPのID、名前、タイプ、バージョン、説明を指定します。
│   ├── http_endpoints
│   │   ├── config.json # エンドポイントを指定します。
│   │   └── sql # エンドポイントのSQLファイルが含まれています。
│   │       ├── <method>-<endpoint-path1>.sql
│   │       ├── <method>-<endpoint-path2>.sql
│   │       └── <method>-<endpoint-path3>.sql
```

## ステップ3. データアプリを変更する

**自動同期＆デプロイ**が有効になっている場合、GitHubまたはTiDB Cloudコンソールを使用してデータアプリを変更することができます。

- [オプション1: GitHub上のファイルを更新してデータアプリを変更する](#オプション1-github上のファイルを更新してデータアプリを変更する)
- [オプション2: TiDB Cloudコンソールでデータアプリを変更する](#オプション2-tidb-cloudコンソールでデータアプリを変更する)

> **注意:**
>
> GitHubおよびTiDB Cloudコンソールでデータアプリを同時に変更した場合は、競合を解決するために、コンソールで行われた変更を破棄するか、コンソールの変更をGitHubの変更で上書きすることができます。

### オプション1: GitHub上のファイルを更新してデータアプリを変更する

構成ファイルを更新する際には、以下に注意してください：

| ファイルディレクトリ  | 注釈 |
| ---------|---------|
| `data_source/cluster.json`     | このファイルを更新する際には、関連するクラスタにアクセスできることを確認してください。クラスタIDはクラスタURLから取得できます。例: クラスタURLが`https://tidbcloud.com/console/clusters/1234567891234567890/overview`である場合、クラスタIDは`1234567891234567890`です。 |
| `http_endpoints/config.json`     | エンドポイントを変更する場合は、[HTTPエンドポイントの構成](/tidb-cloud/data-service-app-config-files.md#http-endpoint-configuration)に記載されている規則に従ってください。   |
| `http_endpoints/sql/method-<endpoint-path>.sql`| `http_endpoints/sql`ディレクトリにSQLファイルを追加または削除する場合は、対応するエンドポイントの構成も更新する必要があります。 |
| `datapp_config.json` | このファイル内の`app_id`フィールドを変更しないでください。このファイルが別のデータアプリからコピーされたもので、現在のデータアプリのIDに更新したい場合を除き、この変更によってトリガーされたデプロイは失敗します。

これらのファイルのフィールド構成についての詳細については、[データアプリの構成ファイル](/tidb-cloud/data-service-app-config-files.md)を参照してください。

ファイル変更がコミットおよびプッシュされた後、TiDB CloudはGitHubの最新の変更でデータアプリを自動的にデプロイします。デプロイのステータスやコミット情報はデプロイ履歴で確認できます。

### オプション2: TiDB Cloudコンソールでデータアプリを変更する

TiDB Cloudコンソールで[データアプリのエンドポイントを変更](/tidb-cloud/data-service-manage-endpoint.md)した後（エンドポイントを変更するなど）、変更をレビューしてGitHubにデプロイすることができます：

1. 右上隅の**デプロイ**をクリックします。変更内容を確認するためのダイアログボックスが表示されます。
2. レビューに基づいて、以下の操作を行います：

    - 現在のドラフトを元にさらに変更を行いたい場合、このダイアログを閉じて変更を行います。
    - 現在の変更を前回のデプロイに戻したい場合は、**ドラフトを破棄**をクリックします。
    - 現在の変更が正常に見える場合は、変更説明を記入（任意）し、**デプロイしてGitHubにプッシュ**をクリックします。デプロイのステータスはトップバナーに表示されます。

デプロイが成功すると、TiDB Cloudコンソールでの変更が自動的にGitHubにプッシュされます。

## 既存のデータアプリの構成をインポートする

既存のデータアプリの構成を新しいデータアプリにインポートするには、以下の手順を実行してください：

1. 既存のデータアプリの構成ファイルをGitHubの新しいブランチやディレクトリにコピーします。
2. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページで、GitHubに接続せずに[新しいデータアプリを作成](/tidb-cloud/data-service-manage-data-app.md#create-a-data-app)します。
3. **自動同期＆デプロイ**を有効にして、新しいデータアプリをGitHubに接続します。新しいデータアプリの対象リポジトリ、ブランチ、ディレクトリを指定する際は、コピーした構成ファイルを使用した新しいパスを使用します。
4. 新しいデータアプリのIDと名前を取得します。左ペインの新しいデータアプリの名前をクリックし、右ペインの**データアプリのプロパティ**エリアでApp IDと名前を取得することができます。

5. GitHub上の新しいパスで、`datapp_config.json`ファイルの中の`app_id`と`app_name`を取得したIDと名前に更新し、変更内容をプッシュしてください。

    GitHubにファイルの変更をプッシュした後、TiDB Cloudは自動的に最新の変更で新しいデータアプリを展開します。

6. GitHubからインポートされた構成を表示するには、TiDB Cloudコンソールのウェブページをリフレッシュしてください。

    デプロイメントの履歴で、展開状況とコミット情報も表示できます。

## GitHub接続の編集

データアプリのGitHub接続を編集したい場合（例：リポジトリ、ブランチ、ディレクトリの切り替え）、以下の手順を実行してください。

1. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページに移動してください。
2. 左ペインで対象のデータアプリの名前をクリックして、詳細を表示します。
3. **GitHubに接続**エリアで、<svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" color="gray.1"><path d="M11 3.99998H6.8C5.11984 3.99998 4.27976 3.99998 3.63803 4.32696C3.07354 4.61458 2.6146 5.07353 2.32698 5.63801C2 6.27975 2 7.11983 2 8.79998V17.2C2 18.8801 2 19.7202 2.32698 20.362C2.6146 20.9264 3.07354 21.3854 3.63803 21.673C4.27976 22 5.11984 22 6.8 22H15.2C16.8802 22 17.7202 22 18.362 21.673C18.9265 21.3854 19.3854 20.9264 19.673 20.362C20 19.7202 20 18.8801 20 17.2V13M7.99997 16H9.67452C10.1637 16 10.4083 16 10.6385 15.9447C10.8425 15.8957 11.0376 15.8149 11.2166 15.7053C11.4184 15.5816 11.5914 15.4086 11.9373 15.0627L21.5 5.49998C22.3284 4.67156 22.3284 3.32841 21.5 2.49998C20.6716 1.67156 19.3284 1.67155 18.5 2.49998L8.93723 12.0627C8.59133 12.4086 8.41838 12.5816 8.29469 12.7834C8.18504 12.9624 8.10423 13.1574 8.05523 13.3615C7.99997 13.5917 7.99997 13.8363 7.99997 14.3255V16Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg>をクリックします。接続設定のダイアログボックスが表示されます。
4. ダイアログボックスで、データアプリのリポジトリ、ブランチ、ディレクトリを変更してください。

    > **注意:**
    >
    > - ディレクトリはスラッシュ（`/`）で始める必要があります。例：`/mydata`。指定したディレクトリが対象リポジトリとブランチに存在しない場合は、自動的に作成されます。
    > - リポジトリ、ブランチ、ディレクトリの組み合わせは、データアプリ間で一意である必要があります。指定したパスが他のデータアプリで既に使用されている場合、代わりのパスを指定する必要があります。そうしないと、TiDB Cloudコンソールでのエンドポイントが指定したパスのファイルを上書きします。
    > - 指定したパスに別のデータアプリからコピーされた構成ファイルが含まれており、これらのファイルを現在のデータアプリにインポートしたい場合は、[既存のデータアプリの構成をインポートする](#import-configurations-of-an-existing-data-app)を参照してください。

5. TiDB Cloudコンソールでの変更とGitHubでのデータアプリの変更を互いに同期させるために、**自動同期と展開を設定**します。

    - 有効にすると、指定したGitHubディレクトリで行われた変更がTiDB Cloudに自動的に展開され、TiDB Cloudコンソールで行われた変更がGitHubにもプッシュされます。該当する展開とコミット情報はデータアプリの展開履歴で確認できます。
    - 無効にすると、指定したGitHubディレクトリで行われた変更はTiDB Cloudで展開されませんし、TiDB Cloudコンソールで行われた変更もGitHubにプッシュされません。

6. **接続を確認** をクリックしてください。

## GitHub接続の解除

データアプリをGitHubに接続解除したい場合は、以下の手順を実行してください：

1. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページに移動してください。
2. 左ペインで対象のデータアプリの名前をクリックして、詳細を表示します。
3. **設定**タブで、**GitHubに接続**エリアの**切断**をクリックします。
4. 切断を確認するために**切断**をクリックしてください。

接続が解除された後、データアプリの構成ファイルは引き続きGitHubディレクトリに残りますが、`tidb-cloud-data-service`によって同期されなくなります。