---
title: GitHubとTiDBサーバーレス分岐（ベータ）の統合 
summary: TiDBサーバーレス分岐機能をGitHubと統合する方法について学びます。

# GitHubとTiDBサーバーレス分岐（ベータ）の統合 

> **メモ:**
>
> この統合は[TiDBサーバーレス分岐](/tidb-cloud/branch-overview.md)に基づいて構築されています。このドキュメントを読む前に、TiDBサーバーレス分岐の概要に精通していることを確認してください。

アプリケーション開発にGitHubを使用している場合、TiDBサーバーレス分岐をGitHubのCI/CDパイプラインに統合することで、本番データベースに影響を与えることなく、プルリクエストを自動的にブランチでテストできます。

この統合プロセスでは、[TiDB Cloud Branching](https://github.com/apps/tidb-cloud-branching) GitHub Appのインストールが求められます。このアプリは、GitHubリポジトリ内のプルリクエストに応じてTiDBサーバーレス分岐を自動的に管理できます。たとえば、プルリクエストを作成すると、アプリはTiDBサーバーレスクラスタの対応するブランチを作成します。ここでは、新機能やバグ修正を本番データベースに影響を与えることなく分離して作業できます。

このドキュメントでは、以下のトピックを取り上げます:

1. GitHubとTiDBサーバーレス分岐の統合方法
2. TiDB Cloud Branchingアプリの動作
3. 本番クラスタではなくブランチを使用して、すべてのプルリクエストをテストするブランチベースのCIワークフローの構築方法

## 開始する前に

統合を行う前に、以下の準備が整っていることを確認してください:

- GitHubアカウント
- アプリケーション用のGitHubリポジトリ
- [TiDBサーバーレスクラスタ](/tidb-cloud/create-tidb-cluster-serverless.md)

## GitHubリポジトリにTiDBサーバーレス分岐を統合

GitHubリポジトリにTiDBサーバーレス分岐を統合するために、以下の手順に従ってください:

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDBサーバーレスクラスタの概要ページに移動します。

2. 左側のナビゲーションパネルで **ブランチ** をクリックします。

3. **ブランチ** ページの右上隅にある **Connect to GitHub** をクリックします。

    - GitHubにログインしていない場合は、まずGitHubにログインするように求められます。
    - 統合を使用するのが初めての場合、**TiDB Cloud Branching**アプリの許可を求められます。

   <img src="https://download.pingcap.com/images/docs/tidb-cloud/branch/github-authorize.png" width="80%" />

4. **GitHubアカウント** のドロップダウンリストからGitHubアカウントを選択します。

    リストにアカウントが存在しない場合は、**別のアカウントをインストール**をクリックし、画面の指示に従ってアカウントをインストールしてください。

5. **GitHubリポジトリ** のドロップダウンリストで対象のリポジトリを選択します。リストが長い場合は、名前を入力してリポジトリを検索することができます。

6. TiDBサーバーレスクラスタとGitHubリポジトリを接続するために **Connect** をクリックします。

   <img src="https://download.pingcap.com/images/docs/tidb-cloud/branch/github-connect.png" width="40%" />

## TiDB Cloud Branchingアプリの動作

TiDBサーバーレスクラスタとGitHubリポジトリを接続した後、このリポジトリ内の各プルリクエストに対して、[TiDB Cloud Branching](https://github.com/apps/tidb-cloud-branching) GitHub Appは自動的に対応するTiDBサーバーレス分岐を管理できます。以下はプルリクエストの変更に対するデフォルトの動作の一覧です:

| プルリクエストの変更               | TiDB Cloud Branchingアプリの動作                                                                                                                                                                                                                                                                                                                                        |
|------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| プルリクエストの作成              | リポジトリでプルリクエストを作成すると、[TiDB Cloud Branching](https://github.com/apps/tidb-cloud-branching)アプリはTiDBサーバーレスクラスタのブランチを作成します。ブランチ名は`${github_branch_name}_${pr_id}_${commit_sha}`形式です。ブランチ数には[制限](/tidb-cloud/branch-overview.md#limitations-and-quotas)があります。 |
| プルリクエストに新しいコミットをプッシュ | リポジトリ内のプルリクエストに新しいコミットをプッシュするたびに、[TiDB Cloud Branching](https://github.com/apps/tidb-cloud-branching)アプリは以前のTiDBサーバーレスブランチを削除し、最新のコミット用の新しいブランチを作成します。                                                                                                                            |
| プルリクエストを閉じるまたはマージする      | プルリクエストを閉じるかマージすると、[TiDB Cloud Branching](https://github.com/apps/tidb-cloud-branching)アプリはこのプルリクエストのブランチを削除します。                                                                                                                                                                                                            |
| プルリクエストを再オープンする               | プルリクエストを再オープンすると、[TiDB Cloud Branching](https://github.com/apps/tidb-cloud-branching)アプリはプルリクエストの最新のコミット用にブランチを作成します。                                                                                                                                                                                                  |

## TiDB Cloud Branchingアプリの構成

[TiDB Cloud Branching](https://github.com/apps/tidb-cloud-branching)アプリの動作を構成するには、リポジトリのルートディレクトリに`tidbcloud.yml`ファイルを追加し、以下の指示に従ってこのファイルに必要な構成を追加します。

### branch.blockList

**Type:** 文字列の配列。 **デフォルト:** `[]`.

TiDB Cloud Branchingアプリを許可していないGitHubブランチを指定します。

```yaml
github:
    branch:
        blockList:
            - ".*_doc"
            - ".*_blackList"
```

### branch.allowList

**Type:** 文字列の配列。 **デフォルト:** `[.*]`.

TiDB Cloud Branchingアプリを許可するGitHubブランチを指定します。

```yaml
github:
    branch:
        allowList:
            - ".*_db"
```

### branch.autoReserved

**Type:** ブール値。 **デフォルト:** `false`.

`true`に設定すると、TiDB Cloud Branchingアプリは前のコミットで作成されたTiDBサーバーレスブランチを削除しません。

```yaml
github:
    branch:
        autoReserved: false
```

### branch.autoDestroy

**Type:** ブール値。 **デフォルト:** `true`.

`false`に設定すると、TiDB Cloud Branchingアプリはプルリクエストが閉じられるかマージされたときにTiDBサーバーレスブランチを削除しません。

```yaml
github:
    branch:
        autoDestroy: true
```

## ブランチベースのCIワークフローの作成

ブランチを使用したCIワークフローを作成するための最良の慣行の1つは、以下の主要な手順です:

1. [TiDBサーバーレス分岐をGitHubリポジトリに統合](#githubリポジトリにtidbサーバーレス分岐を統合)します。

2. ブランチ接続情報を取得します。

   [wait-for-tidbcloud-branch](https://github.com/tidbcloud/wait-for-tidbcloud-branch)アクションを使用してTiDBサーバーレスブランチの準備が整うのを待ち、ブランチの接続情報を取得できます。

    使用例:

   ```yaml
   steps:
     - name: TiDBサーバーレスブランチの準備が整うのを待つ
       uses: tidbcloud/wait-for-tidbcloud-branch@v0
       id: wait-for-branch
       with:
         token: ${{ secrets.GITHUB_TOKEN }}
         public-key: ${{ secrets.TIDB_CLOUD_API_PUBLIC_KEY }}
         private-key: ${{ secrets.TIDB_CLOUD_API_PRIVATE_KEY }}

     - name: TiDBサーバーレスブランチを使用してテスト
        run: |
           echo "ホスト: ${{ steps.wait-for-branch.outputs.host }}"
           echo "ユーザー: ${{ steps.wait-for-branch.outputs.user }}"
           echo "パスワード: ${{ steps.wait-for-branch.outputs.password }}"
   ```

3. テストコードを変更します。

   GitHub Actionsから接続情報を受け取るようにテストコードを変更します。たとえば、[ライブデモ](https://github.com/shiyuhang0/tidbcloud-branch-gorm-example)に示されているように、環境を介して接続情報を受け入れることができます。

## 次のステップ

ブランチGitHubの統合を使用せずに、ブランチベースのCI/CDワークフローを構築することもできます。たとえば、[`setup-tidbcloud-cli`](https://github.com/tidbcloud/setup-tidbcloud-cli)とGitHub Actionsを使用して、CI/CDワークフローをカスタマイズできます。