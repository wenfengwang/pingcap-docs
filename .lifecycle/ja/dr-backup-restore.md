---
title: BRに基づくDRソリューション
summary: TiDBのバックアップとリストア機能を活用した災害復旧の実装方法を学ぶ。

# BRに基づくDRソリューション

TiDBクラスターには複数のレプリカがあり、これにより単一のデータセンターやリージョンの障害に耐えることができ、サービスの提供を継続することができます。自然災害、ソフトウェアの脆弱性、ハードウェアの障害、ウイルス攻撃、または運用ミスなどの影響により、単一のデータセンターやリージョンよりも広い地域に影響を与える場合、TiDBのバックアップ＆リストア（BR）機能を使用することで、ユーザーデータを損傷から保護するためにデータを独立した災害復旧（DR）ストレージデバイスにバックアップすることができます。他のDRソリューションと比較して、BR機能は柔軟性があり、信頼性が高く、リカバリーが可能で、費用対効果が高いです。

- 柔軟性：いつでも任意の頻度でデータをバックアップできます。これにより、バックアップとリストアが柔軟であり、異なるビジネスシナリオに適応しやすくなります。
- 信頼性：バックアップデータは通常、独立したストレージデバイスに保存され、これによりデータセキュリティが向上します。
- リカバリー：予期しない状況によって元のデータに損失や損傷があった場合でも、バックアップデータをリストアすることで、データをリカバリーできます。これにより、BR機能は高いリカバリー性を持ち、データベースの正常な使用が保証されます。
- 費用対効果：高い防御能力を持つBRを使用することで、多額の費用をかけることなくデータベースを保護できます。

一般的に、BRはデータの安全性の最後の手段です。これにより、高い安全性と信頼性が確保され、費用負担が大幅に軽減されます。BRはさまざまな予期せぬ状況でデータを守り、データ損失や損傷のリスクを心配することなくデータベースを安全に使用できます。

## バックアップとリストアの実行

![BRのログバックアップとPITRアーキテクチャ](/media/dr/dr-backup-and-restore.png)

上記のアーキテクチャに示されているように、データを他のリージョンに位置するDRストレージデバイスにバックアップし、必要に応じてバックアップデータからデータをリカバリーすることができます。これにより、クラスターは単一のリージョンの障害に耐えることができ、回復目標復旧（RPO）は最大5分で、復旧時間目標（RTO）は数十分から数時間となります。ただし、データベースのサイズが大きい場合、RTOはより長くなる可能性があります。

> **注:**
>
> 本文書の「リージョン」とは、物理的な場所を指します。

一方、TiDBはブロックストレージのスナップショットを使用したバックアップとリストアを提供しています。この機能により、リカバリー時間を数時間、さらには1時間未満に短縮することができます。TiDBは引き続きバックアップとリストア機能を改良し、最適化しており、より良いサービスを提供しています。

また、TiDBは災害復旧シナリオでのバックアップとリストア機能の使用方法を理解するための詳細なドキュメントも提供しています。その中には、

- [TiDBバックアップとリストアの使用概要](/br/br-use-overview.md)：バックアップ戦略およびバックアップデータの構成を含むBR機能の概要です。
- [バックアップ＆リストアFAQ](/faq/backup-and-restore-faq.md)：TiDBバックアップ＆リストア（BR）のよくある質問（FAQ）とその解決策をリストアップしています。
- [TiDBバックアップ＆リストアアーキテクチャの概要](/br/backup-and-restore-design.md)：バックアップおよびリストアのプロセスとバックアップファイルの設計を含むBR機能の設計アーキテクチャについて説明しています。