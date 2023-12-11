---
title: TiDB ダッシュボードセッションを共有する
summary: 現在の TiDB ダッシュボードセッションを他のユーザーと共有する方法を学びます。

# TiDB ダッシュボードセッションを共有する

TiDB ダッシュボードの現在のセッションを他のユーザーと共有して、ユーザーパスワードを入力せずに TiDB ダッシュボードにアクセスし、操作できます。

## 招待者の手順

1. TiDB ダッシュボードにサインインします。

2. 左サイドバーのユーザー名をクリックして、構成ページにアクセスします。

3. **現在のセッションを共有** をクリックします。

   ![Sample Step](/media/dashboard/dashboard-session-share-settings-1-v650.png)

   > **注記:**
   >
   > セキュリティ上の理由から、共有されたセッションは再共有できません。

4. ポップアップダイアログで共有設定を調整します:

   - 有効期限: 共有されたセッションの有効期間です。現在のセッションからサインアウトしても、共有セッションの有効時間に影響しません。

   - 読み取り専用権限として共有: 共有セッションは読み取り操作のみを許可し、構成の変更などの書き込み操作は許可されません。

5. **認証コードを生成** をクリックします。

   ![Sample Step](/media/dashboard/dashboard-session-share-settings-2-v650.png)

6. 生成された **認証コード** を共有先のユーザーに提供します。

   ![Sample Step](/media/dashboard/dashboard-session-share-settings-3-v650.png)

   > **警告:**
   >
   > 認証コードを安全に保管し、信頼できない人に送信しないでください。そうしないと、許可なく TiDB ダッシュボードにアクセスし、操作できるようになります。

## 招待された側の手順

1. TiDB ダッシュボードのサインインページで、**別の認証を使用** をクリックします。

   ![Sample Step](/media/dashboard/dashboard-session-share-signin-1-v650.png)

2. **認証コード** をクリックして、それを使用してサインインします。

   ![Sample Step](/media/dashboard/dashboard-session-share-signin-2-v650.png)

3. 招待者から受け取った認証コードを入力します。

4. **サインイン** をクリックします。

   ![Sample Step](/media/dashboard/dashboard-session-share-signin-3-v650.png)