---
title: TiDB CloudとNetlifyを統合する
summary: TiDB CloudクラスタをNetlifyプロジェクトに接続する方法を学びます。

# TiDB CloudとNetlifyの統合

[Netlify](https://netlify.com/)は、最新のWebプロジェクトの自動化のためのオールインワンプラットフォームです。ホスティングインフラストラクチャ、継続的インテグレーション、デプロイパイプラインを1つのワークフローに置き換え、サーバーレス機能、ユーザー認証、フォーム処理などの動的機能をプロジェクトが成長するにつれて統合します。

このドキュメントでは、TiDB Cloudをデータベースバックエンドとして使用してNetlifyにフルスタックアプリをデプロイする方法について説明します。

## 前提条件

デプロイする前に、以下の前提条件が満たされていることを確認してください。

### NetlifyアカウントとCLI

NetlifyアカウントとCLIが必要です。持っていない場合は、以下のリンクを参照して作成してください：

* [Netlifyアカウントにサインアップする](https://app.netlify.com/signup)。
* [Netlify CLIを入手する](https://docs.netlify.com/cli/get-started/)。

### TiDB CloudアカウントとTiDBクラスタ

TiDB Cloudのアカウントとクラスタが必要です。持っていない場合は、以下を参照して作成してください：

- [TiDBサーバーレスクラスタを作成する](/tidb-cloud/create-tidb-cluster-serverless.md)
- [TiDB専用クラスタを作成する](/tidb-cloud/create-tidb-cluster.md)

1つのTiDB Cloudクラスタは複数のNetlifyサイトに接続できます。

### TiDB CloudでのすべてのIPアドレスのトラフィックフィルターの許可

TiDB専用クラスタの場合、デプロイメントで動的IPアドレスが使用されるため、クラスタのトラフィックフィルターがすべてのIPアドレス（ `0.0.0.0/0`に設定）を許可することを確認してください。

TiDBサーバーレスクラスタはデフォルトですべてのIPアドレスの接続を許可しますので、トラフィックフィルターを構成する必要はありません。

## ステップ1. 例のプロジェクトと接続文字列を取得する

迅速に始められるように、TiDB Cloudでは、Next.jsを使用したTypeScriptのフルスタック例のアプリを提供しています。これは、自分のブログを投稿したり削除したりできるシンプルなブログサイトで、すべてのコンテンツはPrismaを介してTiDB Cloudに保存されます。

### 例のプロジェクトをフォークし、自分のスペースにクローンする

1. [Next.jsとPrismaを使用したフルスタック例](https://github.com/tidbcloud/nextjs-prisma-example)リポジトリをGitHubリポジトリにフォークします。

2. フォークしたリポジトリを自分のスペースにクローンします：

    ```shell
    git clone https://github.com/${your_username}/nextjs-prisma-example.git
    cd nextjs-prisma-example/
    ```

### TiDB Cloudの接続文字列を取得する

TiDBサーバーレスクラスタの場合、接続文字列は[TiDB Cloud CLI](/tidb-cloud/cli-reference.md)または[TiDB Cloudコンソール](https://tidbcloud.com/)から取得できます。

TiDB専用クラスタの場合、接続文字列はTiDB Cloudコンソールからのみ取得できます。

<SimpleTab>
<div label="TiDB Cloud CLI">

> **ヒント:**
>
> Cloud CLIをインストールしていない場合は、以下の手順に従ってクイックにインストールしてください。

1. インタラクティブモードでクラスタの接続文字列を取得します：

    ```shell
    ticloud cluster connect-info
    ```

2. クラスタ、クライアント、およびオペレーティングシステムを選択するように求められます。このドキュメントで使用されるクライアントは`Prisma`です。

    ```
    クラスタを選択
    > [x] Cluster0(13796194496)
    クライアントを選択
    > [x] Prisma
    オペレーティングシステムを選択
    > [x] macOS/Alpine (検出)
    ```

    次の出力は、`url`の値にPrisma用の接続文字列が含まれるため、次のようになります。

    ```
    datasource db {
    provider = "mysql"
    url      = "mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict"
    }
    ```

    > **注意:**
    >
    > 後で接続文字列を使用する際は、次の点に注意してください：
    >
    > - 接続文字列内のパラメータを実際の値で置き換えてください。
    > - このドキュメントで使用される例のアプリでは新しいデータベースが必要なので、 `<Database>` を一意の新しい名前に置き換える必要があります。

</div>
<div label="TiDB Cloudコンソール">

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの概要ページに移動して右上隅の**Connect**をクリックします。表示されるダイアログから、接続文字列から以下の接続パラメータを取得できます。

    - `${host}`
    - `${port}`
    - `${user}`
    - `${password}`

2. 以下の接続文字列に接続パラメータを入力します：

    ```
    mysql://<User>:<Password>@<Host>:<Port>/<Database>?sslaccept=strict
    ```

    > **注意:**
    >
    > 接続文字列を後で使用する際は、次の点に注意してください：
    >
    > - 接続文字列内のパラメータを実際の値で置き換えてください。
    > - このドキュメントで使用される例のアプリでは新しいデータベースが必要なので、 `<Database>` を一意の新しい名前に置き換える必要があります。

</div>
</SimpleTab>

## ステップ2. 例のアプリをNetlifyにデプロイする

1. Netlify CLIで、Netlifyアカウントに認証しアクセストークンを取得します。

    ```shell
    netlify login
    ```

2. 自動設定を開始します。このステップで連続的デプロイ用にリポジトリを接続するため、Netlify CLIはリポジトリにデプロイキーとウェブフックを作成するためのアクセス権が必要です。

    ```shell
    netlify init
    ```

    プロンプトが表示されたときには、**Create & configure a new site** を選択し、GitHubのアクセスを許可してください。他のオプションについてはすべてデフォルト値を使用してください。

    ```shell
    Adding local .netlify folder to .gitignore file...
    ? What would you like to do? +  Create & configure a new site
    ? Team: your_username’s team
    ? Site name (leave blank for a random name; you can change it later):

    Site Created

    Admin URL: https://app.netlify.com/sites/mellow-crepe-e2ca2b
    URL:       https://mellow-crepe-e2ca2b.netlify.app
    Site ID:   b23d1359-1059-49ed-9d08-ed5dba8e83a2

    Linked to mellow-crepe-e2ca2b


    ? Netlify CLI needs access to your GitHub account to configure Webhooks and Deploy Keys. What would you like to do? Authorize with GitHub through app.netlify.com
    Configuring Next.js runtime...

    ? Your build command (hugo build/yarn run build/etc): npm run netlify-build
    ? Directory to deploy (blank for current dir): .next

    Adding deploy key to repository...
    (node:36812) ExperimentalWarning: The Fetch API is an experimental feature. This feature could change at any time
    (Use `node --trace-warnings ...` to show where the warning was created)
    Deploy key added!

    Creating Netlify GitHub Notification Hooks...
    Netlify Notification Hooks configured!

    Success! Netlify CI/CD Configured!

    This site is now configured to automatically deploy from github branches & pull requests

    Next steps:

    git push       Push to your git repository to trigger new site builds
    netlify open   Open the Netlify admin URL of your site
    ```

3. 環境変数を設定します。自分のスペースとNetlifyスペースからTiDB Cloudクラスタに接続するために、`DATABASE_URL` を[ステップ1](#step-1-get-the-example-project-and-the-connection-string)で取得した接続文字列として設定する必要があります。

    ```shell
    # 自分のスペースの環境変数を設定する
    export DATABASE_URL='mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict'

    # Netlifyスペースの環境変数を設定する
    netlify env:set DATABASE_URL 'mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict'
    ```

    環境変数を確認します。

    ```shell
    # 自分のスペースの環境変数を確認する
    env | grep DATABASE_URL

    # Netlifyスペースの環境変数を確認する
    netlify env:list
    ```

4. アプリをローカルでビルドし、TiDB Cloudクラスタにスキーマをマイグレートします。

    > **ヒント:**
    >
    > ローカルデプロイをスキップし、アプリを直接Netlifyにデプロイしたい場合は、6番目に進んでください。

    ```shell
    npm install .
    npm run netlify-build
    ```

5. アプリケーションをローカルで実行します。ローカル開発サーバーを起動してサイトのプレビューを行うことができます。

    ```shell
    netlify dev
    ```

    次に、ブラウザで`http://localhost:3000/`に移動して、UIを見てみてください。
```
6. Netlifyにアプリケーションをデプロイします。ローカルプレビューに満足したら、次のコマンドを使用してサイトをNetlifyにデプロイできます。 `--trigger` はローカルファイルをアップロードせずにデプロイを意味します。 ローカルで変更を加えた場合は、GitHubリポジトリにコミットしていることを確認してください。

    ```shell
    netlify deploy --prod --trigger
    ```

    デプロイ状態を確認するために、Netlifyコンソールに移動してください。デプロイが完了したら、アプリのサイトはNetlifyによって提供されるパブリックIPアドレスを持ち、誰でもアクセスできるようになります。
```