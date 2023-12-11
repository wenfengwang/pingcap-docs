---
title: PingCAP Clinicの概要
summary: PingCAP Clinic Diagnotic Service（PingCAP Clinic）について、ツールの構成要素、ユーザーシナリオ、実装原則を学びます。

# PingCAP Clinicの概要

PingCAP Clinic Diagnostic Service（PingCAP Clinic）は、TiUPまたはTiDB Operatorを使用して展開されたTiDBクラスタに対してPingCAPが提供する診断サービスです。このサービスを使用すると、リモートでクラスタの問題をトラブルシューティングし、ローカルでクラスタの状態を素早くチェックすることができます。PingCAP Clinicを使用すると、TiDBクラスタの安定した動作をフルライフサイクルで確保し、潜在的な問題を予測し、問題の発生確率を低減し、クラスタの問題を速やかにトラブルシューティングし、問題を修正できます。

PingCAP Clinicは、以下の2つの構成要素を提供して、クラスタの問題を診断します。

- [Diag client](https://github.com/pingcap/diag):

    Diag client（Diag）は、クラスタサイドに展開されたオープンソースの診断ツールです。Diagはクラスタの診断データを収集し、Clinic Serverに診断データをアップロードし、クラスタのローカルでクイックなヘルスチェックを実行するために使用されます。Diagによって収集できる診断データの完全なリストについては、[PingCAP Clinic Diagnostic Data](/clinic/clinic-data-instruction-for-tiup.md)を参照してください。

    > **ノート:**
    >
    > DiagはTiDB v4.0以降のバージョンをサポートしていますが、TiDB Ansibleを使用して展開されたクラスタからのデータ収集は**サポートされていません**。

- Clinic Server:

    Clinic Serverは、クラウドに展開されたクラウドサービスです。SaaSモデルで診断サービスを提供することにより、Clinic Serverはアップロードされた診断データを受け取るだけでなく、オンライン診断環境としてデータの保存、表示、およびクラスタの診断レポートを提供できます。Clinic Serverは、ストレージの場所に応じて2つの独立したサービスを提供します。

    - [国際ユーザ向けClinic Server](https://clinic.pingcap.com): データは米国のAWSに保存されます。
    - [中国本土のユーザ向けClinic Server](https://clinic.pingcap.com.cn): データは中国（北京）地域のAWSに保存されます。

## ユーザーシナリオ

- リモートでクラスタの問題をトラブルシューティングする

    クラスタにすぐに修正できない問題がある場合、PingCAPまたはコミュニティから[サポートを取得](/support.md)することができます。リモートアシストのために技術サポートに連絡する際には、クラスタからさまざまな診断データを保存し、データをサポートスタッフに転送する必要があります。この場合、Diagを使用してクリックで診断データを収集できます。Diagを使用すると、簡単な手作業のデータ収集操作を避け、完全な診断データを迅速に収集できます。データを収集した後、データをClinic Serverにアップロードして、PingCAPの技術サポートスタッフがクラスタの問題をトラブルシューティングできるようにします。Clinic Serverはアップロードされた診断データの安全な保存を提供し、オンライン診断をサポートし、トラブルシューティングの効率を大幅に向上させます。

- クラスタの状態を迅速にチェックする

    現在クラスタが安定して稼働している場合でも、定期的にクラスタをチェックして潜在的な安定性のリスクを検出する必要があります。PingCAP Clinicが提供するローカルおよびサーバーサイドのクイックチェック機能を使用して、クラスタの潜在的な健康リスクを特定できます。

## 実装原則

このセクションでは、Diagがクラスタから診断データを収集する方法の実装原則を紹介します。

まず、Diagは展開ツールTiUP（tiup-cluster）またはTiDB Operator（tidb-operator）からクラスタのトポロジ情報を取得します。その後、Diagは以下のさまざまなデータ収集方法を通じてさまざまなタイプの診断データを収集します。

- SCPを使ってサーバーファイルを転送

    TiUPで展開されたクラスタについて、DiagはSecure Copy Protocol（SCP）を通じて、ターゲットコンポーネントのノードからログファイルや構成ファイルを直接収集することができます。

- SSHを通じてリモートでコマンドを実行してデータを収集

    TiUPで展開されたクラスタについて、DiagはSSH（Secure Shell）を通じてターゲットコンポーネントシステムに接続し、システム情報（カーネルログ、カーネルパラメータ、システムおよびハードウェアの基本情報など）を取得するためのコマンド（例：Insight）を実行できます。

- HTTPコールを使ってデータを収集

    - TiDBコンポーネントのHTTPインターフェースを呼び出すことで、DiagはTiDB、TiKV、PDなどのコンポーネントのリアルタイムな構成サンプリング情報とパフォーマンスサンプリング情報を取得できます。
    - PrometheusのHTTPインターフェースを呼び出すことで、Diagは警告情報と監視メトリクスデータを取得できます。

- SQLステートメントを使ってデータベースパラメータをクエリ

    SQLステートメントを使用して、DiagはTiDBのシステム変数やその他の情報をクエリできます。この方法を使用するには、データを収集する際にTiDBへアクセスするためのユーザー名とパスワードを**追加で提供する**必要があります。

## Clinic Serverの制限事項

> **ノート:**
>
> - Clinic Serverは2022年7月15日から2024年7月14日まで無料です。それ以降、料金がかかる場合は2024年7月14日の前に電子メールで通知されます。
> - 利用制限を調整したい場合は、PingCAPから[サポートを取得](/support.md)してください。

| サービスタイプ| 制限 |
| :------ | :------ |
| クラスタ数 | 組織あたり10個 |
| ストレージ容量 | クラスタあたり50 GB |
| ストレージ期間 | 180日 |
| データサイズ | 1パッケージあたり3 GB |
| データリビルド環境の保存期間 | 3日 |

## 次のステップ

- オンプレミス環境でPingCAP Clinicを使用する
    - [PingCAP Clinicのクイックスタート](/clinic/quick-start-with-clinic.md)
    - [PingCAP Clinicを使用したクラスタのトラブルシューティング](/clinic/clinic-user-guide-for-tiup.md)
    - [PingCAP Clinic Diagnostic Data](/clinic/clinic-data-instruction-for-tiup.md)

- KubernetesでPingCAP Clinicを使用する
    - [PingCAP Clinicを使用したTiDBクラスタのトラブルシューティング](https://docs.pingcap.com/tidb-in-kubernetes/stable/clinic-user-guide)
    - [PingCAP Clinic Diagnostic Data](https://docs.pingcap.com/tidb-in-kubernetes/stable/clinic-data-collection)