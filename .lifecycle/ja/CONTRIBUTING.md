# TiDB Documentation Contributing Guide

[欢迎访问 TiDB](https://github.com/pingcap/tidb) 文档！我们对您加入 [TiDB 社区](https://github.com/pingcap/community/) 的潜力感到兴奋。

## 你可以做出贡献

您可以从以下任何一项开始，以帮助改进 [PingCAP 网站上的 TiDB 文档](https://docs.pingcap.com/tidb/stable)：

- 修复错别字或格式（标点、空格、缩进、代码块等）
- 修复或更新不合适或过时的描述
- 添加丢失的内容（句子、段落或新文档）
- 将文档变更从英文翻译为中文
- 提交、回复和解决 [文档问题](https://github.com/pingcap/docs/issues)
- （高级）审查他人创建的拉取请求

## 在您贡献之前

在您做出贡献之前，请快速了解一下 TiDB 文档维护的一些一般信息。这可以帮助您很快成为一名贡献者。

### 熟悉样式

- [提交消息风格](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#how-to-write-a-good-commit-message)
- [拉取请求标题风格](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#pull-request-title-style)
- [Markdown 规则](/resources/markdownlint-rules.md)
- [代码注释风格](https://github.com/pingcap/community/blob/master/contributors/code-comment-style.md)
- 图表样式：[Figma 快速入门指南](https://github.com/pingcap/community/blob/master/contributors/figma-quick-start-guide.md)

    为保持图表的一致风格，我们建议使用 [Figma](https://www.figma.com/) 来绘制或设计图表。如果您需要绘制图表，请参考指南并使用模板中提供的形状或颜色。

### 选择一个文档模板

我们提供了[几种文档模板](/resources/doc-templates)供您使用，以创建符合我们风格的文档。

在提交拉取请求之前，请查看这些模板：

- [概念](/resources/doc-templates/template-concept.md)
- [任务](/resources/doc-templates/template-task.md)
- [参考信息](/resources/doc-templates/template-reference.md)
- [新功能](/resources/doc-templates/template-new-feature.md)
- [故障排除](/resources/doc-templates/template-troubleshooting.md)

### 了解文档版本

目前，我们维护以下版本的 TiDB 文档，每个版本都有一个独立的分支：

| 文档分支名称 | 版本描述 |
| :--- | :--- |
| `master` 分支 | 最新的开发版本 |
| `release-6.1` 分支 | 6.1 LTS（长期支持）版本 |
| `release-6.0` 分支 | 6.0 开发里程碑发布 |
| `release-5.4` 分支 | 5.4 稳定版本 |
| `release-5.3` 分支 | 5.3 稳定版本 |
| `release-5.2` 分支 | 5.2 稳定版本 |
| `release-5.1` 分支 | 5.1 稳定版本 |
| `release-5.0` 分支 | 5.0 稳定版本 |
| `release-4.0` 分支 | 4.0 稳定版本 |
| `release-3.1` 分支 | 3.1 稳定版本 |
| `release-3.0` 分支 | 3.0 稳定版本 |
| `release-2.1` 分支 | 2.1 稳定版本 |

> **注意：**
>
> 以前，我们在 `master` 分支中维护所有版本，例如 `dev`（最新开发版本）、`v3.0` 等等。每个文档版本更新频率很高，对一个版本的更改通常也适用于另一个版本或其他版本。
>
> 自 2020 年 2 月 21 日起，为了减少版本间的手动编辑和更新工作，我们开始在单独的分支中维护每个版本，并引入了 sre-bot（现在是 ti-chi-bot）来自动为其他版本提交拉取请求，只要您给您的拉取请求添加了相应的 cherry-pick 标签。

### 使用 cherry-pick 标签

- 如果您的更改只适用于一个文档版本，请只向相应版本分支提交一个拉取请求。

- 如果您的更改适用于多个文档版本，则无需向每个分支提交拉取请求。取而代之，在您提交拉取请求后，通过根据需要添加以下一个或多个标签来触发 ti-chi-bot 向其他版本分支提交拉取请求。一旦当前拉取请求被合并，ti-chi-bot 将开始工作。
    - `needs-cherry-pick-6.1` 标签：ti-chi-bot 将向 `release-6.1` 分支提交拉取请求。
    - `needs-cherry-pick-6.0` 标签：ti-chi-bot 将向 `release-6.0` 分支提交拉取请求。
    - `needs-cherry-pick-5.4` 标签：ti-chi-bot 将向 `release-5.4` 分支提交拉取请求。
    - `needs-cherry-pick-5.3` 标签：ti-chi-bot 将向 `release-5.3` 分支提交拉取请求。
    - `needs-cherry-pick-5.2` 标签：ti-chi-bot 将向 `release-5.2` 分支提交拉取请求。
    - `needs-cherry-pick-5.1` 标签：ti-chi-bot 将向 `release-5.1` 分支提交拉取请求。
    - `needs-cherry-pick-5.0` 标签：ti-chi-bot 将向 `release-5.0` 分支提交拉取请求。
    - `needs-cherry-pick-4.0` 标签：ti-chi-bot 将向 `release-4.0` 分支提交拉取请求。
    - `needs-cherry-pick-3.1` 标签：ti-chi-bot 将向 `release-3.1` 分支提交拉取请求。
    - `needs-cherry-pick-3.0` 标签：ti-chi-bot 将向 `release-3.0` 分支提交拉取请求。
    - `needs-cherry-pick-2.1` 标签：ti-chi-bot 将向 `release-2.1` 分支提交拉取请求。
    - `needs-cherry-pick-master` 标签：ti-chi-bot 将向 `master` 分支提交拉取请求。

    有关如何选择文档版本，请参阅 [选择受影响版本的指南](#guideline-for-choosing-the-affected-versions)。

- 如果您的大部分更改适用于多个文档版本但各版本存在差异，您仍可以使用 cherry-pick 标签让 ti-chi-bot 为其他版本创建拉取请求。在 ti-chi-bot 成功提交到另一个版本的拉取请求后，您可以对该拉取请求进行更改。

## 如何贡献

请执行以下步骤来创建您的拉取请求到该存储库。如果不喜欢使用命令，您也可以使用 [GitHub Desktop](https://desktop.github.com/)，这样更容易入门。

> **注意：**
>
> 此部分以向 `master` 分支创建拉取请求为例。向其他分支创建拉取请求的步骤类似。

### 步骤 0：签署 CLA

只有在您签署 [贡献者许可协议](https://cla-assistant.io/pingcap/docs)（CLA）后，您的拉取请求才能被合并。在继续之前，请确保您已签署 CLA。

### 步骤 1：分叉存储库

1. 访问项目：<https://github.com/pingcap/docs>
2. 点击右上角的 **Fork** 按钮，等待分叉完成。

### 步骤 2：将分叉后的存储库克隆到本地

```
cd $working_dir # 进入要放置分叉的目录，例如 "cd ~/Documents/GitHub"
git clone git@github.com:$user/docs.git # 用您的 GitHub ID 替换 "$user"

cd $working_dir/docs
git remote add upstream git@github.com:pingcap/docs.git # 添加上游存储库
git remote -v # 确认您的远程是合理的
```

### 步骤 3：创建新的分支

1. 使您的本地主分支与上游主分支保持最新。

    ```
    cd $working_dir/docs
    git fetch upstream
    git checkout master
    git rebase upstream/master
    ```

2. 基于主分支创建新的分支。

    ```
    git checkout -b new-branch-name
    ```

### 步骤 4：做点什么

在 `new-branch-name` 分支上编辑一些文件并保存您的更改。您可以使用 Visual Studio Code 等编辑器打开和编辑 `.md` 文件。

### 步骤 5：提交您的更改

```
git status # 检查本地状态
git add <file> ... # 添加想要提交的文件。如果要提交所有更改，可以直接使用 `git add .`
git commit -m "commit-message：更新 xx"
```
[コミットメッセージのスタイル](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#how-to-write-a-good-commit-message) を参照してください。

### ステップ6: あなたのブランチをupstream/masterと同期させる

```
# あなたの新しいブランチ上で
git fetch upstream
git rebase upstream/master
```

### ステップ7: 変更内容をリモートにプッシュする

```
git push -u origin new-branch-name # "-u"はoriginからリモートブランチをトラッキングするために使用します
```

### ステップ8: プルリクエストを作成する

1. <https://github.com/$user/docs> であなたのフォークを訪れてください（`$user`をGitHubのIDに置き換えてください）
2. `new-branch-name`ブランチの隣にある`Compare & pull request`ボタンをクリックしてPRを作成してください。[プルリクエストのタイトルのスタイル](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#pull-request-title-style)を参照してください。

これで、あなたのPRは正常に提出されました！このPRがマージされると、自動的にTiDBのドキュメントの寄稿者になります。

## 対象バージョンの選択に関するガイドライン

プルリクエストを作成する際に、プルリクエストページの説明テンプレートでドキュメントの変更が適用されるリリースバージョンを選択する必要があります。

次のいずれかの状況に当てはまる場合は、**MASTER BRANCHのみを選択することをお勧めします**。PRがマージされると、変更内容はすぐに[PingCAPドキュメントのDevページ](https://docs.pingcap.com/tidb/dev/)に表示されます。次のTiDBのメジャーバージョンまたはマイナーバージョンがリリースされた後、変更内容は新バージョンのウェブサイトページにも表示されます。

- 書類の内容を補完するなど、ドキュメントの改善に関連する場合
- 値、説明、例、または誤字の不正確なドキュメント内容を修正する場合
- 特定のトピックモジュールでのドキュメントのリファクタリングが関わる場合

次のいずれかの状況に当てはまる場合は、**AFFECTED RELEASE BRANCH(ES) と MASTER を選択することをお勧めします**。

- 特定バージョンに関連する機能の動作変更が関わる場合
- 構成項目やシステム変数のデフォルト値を変更する互換性の変更が関わる場合
- 表示エラーを解決するためにフォーマットを修正する場合
- 壊れたリンクを修正する場合

## 連絡先

ディスカッションのために[TiDB Internalsフォーラム](https://internals.tidb.io/)に参加してください。