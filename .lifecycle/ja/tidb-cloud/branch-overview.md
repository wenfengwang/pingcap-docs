---
title: TiDB サーバーレス ブランチング (β) 概要
summary: TiDB サーバーレス ブランチの概念を学ぶ。
---

# TiDB サーバーレス ブランチング (β) 概要

TiDB Cloud では、TiDB サーバーレス クラスターのブランチを作成できます。クラスターのブランチとは、元のクラスターからのデータの分岐コピーを含む別個のインスタンスであり、独立した環境を提供します。これにより、元のクラスターに影響を与えることなく、自由に実験を行うことができます。

TiDB サーバーレス ブランチを利用することで、開発者は並行して作業し、新機能の繰り返しテストを迅速に行い、本番データベースに影響を与えることなく問題を解決し、必要に応じて変更を簡単に取り消すことができます。この機能により、開発および展開プロセスが合理化され、本番データベースの安定性と信頼性が確保されます。

## 実装

クラスターのブランチが作成されると、ブランチ内のデータが元のクラスターとは異なる方向に分かれることを意味します。これにより、元のクラスターまたはブランチで行われた後続の変更は互いに同期されなくなります。

TiDB サーバーレスは、元のクラスターとそのブランチ間でデータを共有するためのコピーオンライト技術を使用して、迅速かつシームレスなブランチの作成を実現しています。このプロセスは通常数分で完了し、ユーザーには気付かれずに行われます。これにより、元のクラスターのパフォーマンスに影響を与えることなく、ブランチの作成が実現されます。

## シナリオ

独立したデータ環境を簡単かつ迅速に作成するために、開発者またはチームが独立して作業し、変更をテストし、バグを修正し、新機能を実験し、本番データベースを中断させずに更新を展開する必要がある以下のシナリオでブランチが有益です。

- 機能開発: 開発者は、本番データベースに影響を与えることなく、新機能に取り組むことができます。各機能には独自のブランチを持たせており、他の進行中の作業に影響を与えることなく、繰り返しテストや実験を迅速に行うことができます。

- バグ修正: 開発者は特定のバグを修正するための専用のブランチを作成し、修正をテストし、検証後にマージすることができます。これにより、新しい問題を導入することなく、本番データベースに修正を適用することができます。

- 実験: 新機能の開発や変更を行っている際に、開発者はさまざまなアプローチや構成を実験するためにブランチを作成できます。これにより、異なるオプションを比較し、データを収集し、変更が本番データベースにマージされる前に情報を元に決定することができます。

- パフォーマンス最適化: データベースの変更は、パフォーマンスの向上のために行われることがあります。ブランチを利用することで、開発者は効率的なソリューションを特定するために、独立した環境でさまざまな構成、インデックス、またはアルゴリズムの実験や微調整を行うことができます。

- テストおよびステージング: チームはテストおよびステージングの目的でブランチを作成できます。変更が本番データベースにマージされる前に、品質保証、ユーザー受け入れテスト、またはステージングのカスタマイズのための制御された環境を提供します。

- 並行開発: 異なるチームや開発者が同時に別個のプロジェクトに取り組むことができます。各プロジェクトには独自のブランチを持たせることができ、独立した開発と実験を行うことができます。さらに、本番データベースに変更をマージすることができます。

## 制限とクォータ

現在、TiDB サーバーレス ブランチはβ版であり、無償です。

- 2023年7月5日以降に作成された TiDB サーバーレス クラスターにのみブランチを作成できます。

- TiDB Cloud の各組織では、デフォルトで全クラスターにつき最大5つの TiDB サーバーレス ブランチを作成できます。クラスターのブランチは、クラスターと同じリージョンで作成され、スロットル制限のあるクラスターや100 GiBを超えるクラスターのブランチを作成することはできません。

- 各ブランチには5 GiBのストレージが許可されています。ストレージを超過すると、このブランチの読み書き操作がスロットル制限されます。

クォータを追加する必要がある場合は、[TiDB Cloud サポートに連絡](/tidb-cloud/tidb-cloud-support.md)してください。

## 次のステップ

- [ブランチの管理方法を学ぶ](/tidb-cloud/branch-manage.md)