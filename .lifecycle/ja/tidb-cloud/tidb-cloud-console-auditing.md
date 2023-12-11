---
title: コンソール監査ログ
summary: TiDB Cloudコンソールの監査ログ機能について学ぶ。

# コンソール監査ログ

TiDB Cloudは、[TiDB Cloudコンソール](https://tidbcloud.com)でのユーザーのさまざまな振る舞いや操作を追跡するためのコンソール監査ログ機能を提供しています。たとえば、組織にユーザーを招待したり、クラスターを作成したりする操作を追跡できます。

## 前提条件

- TiDB Cloudの組織で、`Organization Owner`または`Organization Console Audit Admin`の役割である必要があります。それ以外の場合、TiDB Cloudコンソールでコンソール監査ログ関連のオプションを見ることはできません。`Organization Console Audit Admin`の役割は要求に応じてのみ表示されるため、直接`Organization Owner`の役割を使用することをお勧めします。`Organization Console Audit Admin`の役割を使用する必要がある場合は、[TiDB Cloudのユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照して、[TiDB Cloudコンソール](https://tidbcloud.com)の右下の**?**をクリックして**サポートをリクエスト**をクリックします。次に、「組織のコンソール監査管理者役割の申請」と入力し、**送信**をクリックします。
- コンソール監査ログは、組織の監査ログを有効または無効にできます。組織内のユーザーのアクションのみを追跡できます。
- コンソール監査ログを有効にした後は、TiDB Cloudコンソールのすべてのイベントの種類が監査され、特定のイベントのみを監査することはできません。

## コンソール監査ログの有効化

コンソール監査ログの機能はデフォルトで無効になっています。有効にするには、次の手順を実行してください。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)の左下隅で<MDSvgIcon name="icon-top-organization" />をクリックし、**Console Audit Logging**をクリックします。
2. 右上隅の**設定**をクリックし、コンソール監査ログを有効にします。

## コンソール監査ログの無効化

コンソール監査ログを無効にするには、次の手順を実行してください。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)の左下隅で<MDSvgIcon name="icon-top-organization" />をクリックし、**Console Audit Logging**をクリックします。
2. 右上隅の**設定**をクリックし、コンソール監査ログを無効にします。

## コンソール監査ログの表示

コンソール監査ログは、組織の監査ログのみを表示できます。

> **注:**
>
> - 組織がコンソール監査ログを有効にするのが初めての場合、監査ログは空です。監査されたイベントが実行された後、対応するログが表示されます。
> - コンソール監査ログが無効になってから90日以上経過した場合、ログは表示されません。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)の左下隅で<MDSvgIcon name="icon-top-organization" />をクリックし、**Console Audit Logging**をクリックします。
2. 監査ログの特定の部分を取得するには、イベントの種類、操作のステータス、および時間範囲をフィルタリングできます。
3. (オプション) より多くのフィールドをフィルタリングするには、**詳細フィルタリング**をクリックし、追加のフィルターを追加してから**適用**をクリックします。
4. ログの行をクリックして、右のペインで詳細情報を表示します。

## コンソール監査ログのエクスポート

組織のコンソール監査ログをエクスポートするには、次の手順を実行してください。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)の左下隅で<MDSvgIcon name="icon-top-organization" />をクリックし、**Console Audit Logging**をクリックします。
2. (オプション) 特定のコンソール監査ログの一部をエクスポートする必要がある場合は、さまざまな条件でフィルタリングできます。それ以外の場合は、この手順をスキップしてください。
3. **エクスポート**をクリックし、JSONまたはCSVのエクスポート形式を選択します。

## コンソール監査ログの保管ポリシー

コンソール監査ログの保存期間は90日であり、その後にログが自動的にクリアされます。

> **注:**
>
> - TiDB Cloudでコンソール監査ログの保存場所を指定することはできません。
> - 監査ログを手動で削除することはできません。

## コンソール監査イベントの種類

コンソール監査ログは、さまざまなイベントタイプを通じてTiDB Cloudコンソールでのユーザーアクティビティを記録します。

> **注:**
>
> 現在、TiDB Cloudコンソールでほとんどのイベントタイプを監査でき、それらを次の表に示します。まだカバーされていない残りのイベントタイプについては、TiDB Cloudは継続的に取り込む作業を行います。

| コンソール監査イベントの種類 | 説明 |
|--------------------------------|-----------------------------------------------------------|
| CreateOrganization             | 組織の作成                                               |
| LoginOrganization              | 組織へのログイン                                         |
| SwitchOrganization             | 現在の組織から別の組織に切り替える                       |
| LogoutOrganization             | 組織からログアウト                                       |
| InviteUserToOrganization       | ユーザーを組織に招待                                     |
| DeleteInvitationToOrganization | ユーザーの組織への招待を削除                             |
| ResendInvitationToOrganization | ユーザーの組織への招待を再送信                           |
| ConfirmJoinOrganization        | 招待されたユーザーが組織への参加を確認                   |
| DeleteUserFromOrganization     | 組織から参加したユーザーを削除                           |
| UpdateUserRoleInOrganization   | 組織内のユーザーの役割を更新                             |
| CreateAPIKey                   | APIキーの作成                                             |
| EditAPIKey                     | APIキーの編集                                             |
| DeleteAPIKey                   | APIキーの削除                                             |
| UpdateTimezone                 | 組織のタイムゾーンの更新                                   |
| ShowBill                       | 組織の請求書を表示                                       |
| DownloadBill                   | 組織の請求書をダウンロード                               |
| ShowCredits                    | 組織のクレジットを表示                                   |
| AddPaymentCard                 | 支払いカードの追加                                       |
| UpdatePaymentCard              | 支払いカードの更新                                       |
| DeletePaymentCard              | 支払いカードの削除                                       |
| SetDefaultPaymentCard          | デフォルトの支払いカードを設定                           |
| EditBillingProfile             | 請求プロファイル情報の編集                               |
| ContractAction                 | 契約に関連するアクティビティ                             |
| EnableConsoleAuditLog          | コンソール監査ログの有効化                               |
| ShowConsoleAuditLog            | コンソール監査ログの表示                                 |
... (remaining events)
| AddDBAuditFilter               | データベース監査ログフィルターの追加                       |
| DeleteDBAuditFilter            | データベース監査ログフィルターの削除                       |
| EditProject                    | プロジェクト情報の編集                                                |
| DeleteProject                  | プロジェクトの削除                                                                 |
| BindSupportPlan                | サポートプランのバインド                                                              |
| CancelSupportPlan              | サポートプランのキャンセル                                                            |
| UpdateOrganizationName         | 組織名の更新                                                     |
| SetSpendLimit                  | TiDB Serverlessクラスターの支出制限の編集                             |
| UpdateMaintenanceWindow        | メンテナンスウィンドウの開始時刻の変更                                             |
| DeferMaintenanceTask           | メンテナンスタスクの延期                                                         |
| CreateBranch                   | TiDB Serverlessブランチの作成                                                  |
| DeleteBranch                   | TiDB Serverlessブランチの削除                                                  |
| SetBranchRootPassword          | TiDB Serverlessブランチのルートパスワードの設定                                   |
| ConnectBranchGitHub            | クラスターをGitHubリポジトリに接続してブランチ統合を有効にする     |
| DisconnectBranchGitHub         | クラスターをGitHubリポジトリから切断してブランチ統合を無効にする |

## コンソール監査ログフィールド

TiDB Cloudは、各コンソール監査ログに対して以下のフィールドを提供して、ユーザーのアクティビティを追跡できるようにサポートしています。

| フィールド名 | データ型 | 説明 |
|---|---|---|
| type | string | イベントタイプ |
| ends_at | timestamp | イベント時刻 |
| operator_type | enum | オペレータータイプ: `user` または `api_key` |
| operator_id | uint64 | オペレーターID |
| operator_name | string | オペレーター名 |
| operator_ip | string | オペレーターのIPアドレス |
| operator_login_method | enum | オペレーターのログイン方法: `google`, `github`, `microsoft`, `email`, または `api_key` |
| org_id | uint64 | イベントが所属する組織のID |
| org_name | string | イベントが所属する組織の名前 |
| project_id | uint64 | イベントが所属するプロジェクトのID |
| project_name | string | イベントが所属するプロジェクトの名前 |
| cluster_id | uint64 | イベントが所属するクラスターのID |
| cluster_name | string | イベントが所属するクラスターの名前 |
| trace_id | string | オペレーターが開始したリクエストのトレースID。現在は空であり、将来のリリースで利用可能になります。 |
| result | enum | イベント結果: `success` または `failure` |
| details | json | イベントの詳細な説明 |