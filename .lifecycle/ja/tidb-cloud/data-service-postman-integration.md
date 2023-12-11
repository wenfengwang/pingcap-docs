---
title: Postman で Data アプリを実行する
summary: Data アプリを Postman で実行する方法を学びます。

# Postman で Data アプリを実行する

[Postman](https://www.postman.com/) は、API ライフサイクルを簡素化し、API 開発をより迅速かつ高品質にするためのコラボレーションを促進する API プラットフォームです。

TiDB Cloud の [Data Service](https://tidbcloud.com/console/data-service) では、Data アプリを簡単に Postman にインポートし、Postman の豊富なツールを活用して API 開発体験を向上させることができます。

このドキュメントでは、Data アプリを Postman にインポートし、Postman で Data アプリを実行する方法について説明します。

## 開始する前に

Data アプリを Postman にインポートする前に、次のものを用意してください。

- [Postman](https://www.postman.com/) アカウント
- [Postman デスクトップアプリ](https://www.postman.com/downloads)（任意）。代わりに、アプリをダウンロードせずに Postman の Web バージョンを使用することもできます。
- 少なくとも1つの明確に定義された [エンドポイント](/tidb-cloud/data-service-manage-endpoint.md) を持つ [Data アプリ](/tidb-cloud/data-service-manage-data-app.md)。次の要件を満たすエンドポイントのみを Postman にインポートできます:

    - ターゲットクラスタが選択されている。
    - エンドポイントのパスとリクエストメソッドが構成されている。
    - SQL ステートメントが記述されている。

- Data アプリの [API キー](/tidb-cloud/data-service-api-key.md#create-an-api-key) 。

## ステップ 1. Data アプリを Postman にインポートする

Data アプリを Postman にインポートする手順は次のとおりです。

1. [TiDB Cloud コンソール](https://tidbcloud.com) から、プロジェクトの [**Data Service**](https://tidbcloud.com/console/data-service) ページに移動します。
2. 左ペインで、ターゲットの Data アプリの名前をクリックしてその詳細を表示します。
3. ページの右上隅にある **Run in Postman** をクリックします。インポート手順が記載されたダイアログが表示されます。

    > **注意:**
    >
    > - Data アプリに明確に定義されたエンドポイントがない場合（ターゲットクラスタ、パス、リクエストメソッド、SQL ステートメントが構成されている）、Data アプリの **Run in Postman** は無効のままです。
    > - Chat2Query Data アプリの場合、**Run in Postman** は利用できません。

4. Data アプリのインポートのためにダイアログで指示された手順に従います:

    1. 好みに応じて、**Run in Postman for Web** または **Run in Postman Desktop** のいずれかを選択して Postman ワークスペースを開き、ターゲットのワークスペースを選択します。

        - Postman にログインしていない場合は、まず Postman にログインするための画面の指示に従います。
        - **Run in Postman Desktop** をクリックした場合は、Postman デスクトップアプリを起動するための画面の指示に従います。

    2. Postman でのターゲットワークスペースのページで、左側のナビゲーションメニューで **Import** をクリックします。
    3. TiDB Cloud ダイアログから Data アプリの URL をコピーし、それを Postman にインポートします。

5. URL を貼り付けると、Postman は Data アプリを自動的に新しい [コレクション](https://learning.postman.com/docs/collections/collections-overview) としてインポートします。コレクションの名前は `TiDB Data Service - <Your App Name>` 形式です。

    コレクションでは、デプロイされたエンドポイントは **Deployed** フォルダにグループ化され、未デプロイのエンドポイントは **Draft** フォルダにグループ化されます。

## ステップ 2. Postman で Data アプリの API キーを構成する

Postman でインポートされた Data アプリを実行する前に、Data アプリの API キーを以下の手順で設定する必要があります:

1. Postman の左側のナビゲーションメニューで `TiDB Data Service - <Your App Name>` をクリックして右側にそのタブを開きます。
2. `TiDB Data Service - <Your App Name>` のタブの下にある **Variables** タブをクリックします。
3. 変数テーブルで、Data アプリの公開キーと秘密キーを **現在の値** 列に入力します。
4. `TiDB Data Service - <Your App Name>` タブの右上隅にある **Save** をクリックします。

## ステップ 3. Postman で Data アプリを実行する

Data アプリを Postman で実行する手順は次のとおりです:

1. Postman の左側のナビゲーションペインで、**Deployed** または **Draft** フォルダを展開し、エンドポイントの名前をクリックして右側にそのタブを開きます。
2. `<Your Endpoint Name>` のタブの下で、次のようにエンドポイントを呼び出すことができます:

    - パラメーターがないエンドポイントの場合、直接 **Send** をクリックして呼び出すことができます。
    - パラメーターを持つエンドポイントの場合、まずパラメーターの値を入力し、その後 **Send** をクリックする必要があります。

        - `GET` または `DELETE` リクエストの場合、パラメーターの値を **Query Params** テーブルに入力してください。
        - `POST` または `PUT` リクエストの場合、**Body** タブをクリックし、パラメーターの値を JSON オブジェクトとして入力してください。TiDB Cloud Data Service のエンドポイントで **Batch Operation** が有効になっている場合、パラメーターの値を JSON オブジェクトの配列として入力してください。

3. 下部のペインで応答を確認します。

4. 異なるパラメーター値でエンドポイントを再度呼び出したい場合は、パラメーター値を編集し、再度 **Send** をクリックしてください。

Postman の使用方法について詳しくは、[Postman ドキュメント](https://learning.postman.com/docs) をご覧ください。

## Data アプリの新しい変更に対処する

Data アプリが Postman にインポートされた後、TiDB Cloud Data Service は Data アプリの新しい変更を自動的に Postman に同期しません。

新しい変更を反映させる場合は、再度 [インポート手順](#step-1-import-your-data-app-to-postman) に従う必要があります。Postman ワークスペースでコレクション名が一意であるため、以前にインポートされたものを最新の Data アプリで置き換えるか、最新の Data アプリを新しいコレクションとしてインポートするかを選択できます。

また、Data アプリを再インポートした後は、Postman で再度 [新しくインポートされたアプリのために API キーを構成](#step-2-configure-your-data-app-api-key-in-postman) する必要があります。