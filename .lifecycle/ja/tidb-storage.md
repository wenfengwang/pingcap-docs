---
title: TiDB ストレージ
summary: TiDBデータベースのストレージレイヤーを理解する。

# TiDB ストレージ

このドキュメントは、[TiKV](https://github.com/tikv/tikv) のいくつかの設計思想とキーコンセプトを紹介します。

![storage-architecture](/media/tidb-storage-architecture-1.png)

## キー・バリューペア

データストレージシステムを決定する最初のことは、データの保存形式、つまりデータがどのような形式で保存されるかです。TiKVの選択肢はキー・バリューモデルであり、順次走査方法が提供されます。TiKVのデータストレージモデルの2つの重要な点があります：

+ これは巨大なMap（C++の`std::Map`に似ています）であり、キー・バリューペアが格納されています。
+ マップ内のキー・バリューペアは、キーのバイナリ順に並べ替えられており、これにより特定のキーの位置を検索し、次にこのキーよりも大きいキー・バリューペアを順次取得できます。

この文書で説明されているTiKVのKVストレージモデルは、SQLのテーブルとは何の関係もありません。この文書はSQLに関連する概念については議論せず、TiKVなどの高性能で信頼性の高い分散型キー・バリューストレージの実装方法に焦点を当てています。

## ローカルストレージ（RocksDB）

永続的なストレージエンジンでは、データは最終的にディスクに保存されますが、TiKVも例外ではありません。TiKVはデータを直接ディスクに書き込むのではなく、データをRocksDBに保存し、それがデータストレージを担当します。その理由は、スタンドアロンのストレージエンジンを開発することが非常にコストがかかりますし、特に注意深い最適化が必要な高性能なスタンドアロンエンジンの場合です。

RocksDBは、Facebookがオープンソース化した優れたスタンドアロンストレージエンジンです。このエンジンは単一のエンジンでTiKVのさまざまな要件を満たすことができます。ここでは、RocksDBを単一の永続的なキー・バリューマップとして単純に考えることができます。

## Raftプロトコル

さらに、TiKVの実装はより困難な課題に直面します：単一のマシンの障害時にデータの安全性を確保することです。

単純な方法は、データを複数のマシンにレプリケートしておくことであり、そのため、1台のマシンが障害を起こしても、他のマシンのレプリカは利用可能です。言い換えれば、信頼性があり、効率的で、障害レプリカを処理できるデータレプリケーションスキームが必要です。これらすべてを実現するためにRaftアルゴリズムが使用されています。

Raftはコンセンサスアルゴリズムです。このドキュメントではRaftを簡単に紹介します。詳細については、[In Search of an Understandable Consensus Algorithm](https://raft.github.io/raft.pdf) を参照してください。Raftにはいくつかの重要な機能があります：

- リーダー選出
- メンバーシップの変更（レプリカの追加、削除、リーダーの移行など）
- ログのレプリケーション

TiKVはRaftを使用してデータをレプリケートします。各データ変更はRaftログとして記録されます。Raftログのレプリケーションにより、データはRaftグループの複数のノードに安全かつ信頼性の高くレプリケートされます。ただし、Raftプロトコルによると、成功した書き込みは多数のノードにのみデータがレプリケートされる必要があります。

![TiDBのRaft](/media/tidb-storage-1.png)

要約すると、TiKVは単独のマシンRocksDBを介してデータをすばやくディスクに保存し、マシンの障害が発生した場合にはRaftを使用してデータを複数のマシンにレプリケートします。データはRocksDBではなくRaftのインターフェースを介して書き込まれます。Raftの実装により、TiKVは分散型キー・バリューストレージとなります。いくつかのマシンの障害が発生しても、TiKVはアプリケーションに影響を与えずに、ネイティブなRaftプロトコルを利用して自動的にレプリカを完了することができます。

## リージョン

理解しやすくするために、すべてのデータが1つのレプリカしか持たないと仮定しましょう。先述のように、TiKVは大きな順次KVマップと見なすことができますので、データは水平スケーラビリティを実現するために複数のマシンに分散されます。KVシステムでは、複数のマシンにデータを分散するために2つの典型的な解決策があります：

* ハッシュ：キーによってハッシュを作成し、ハッシュ値に従って対応するストレージノードを選択します。
* レンジ：キーによって範囲を区切り、シリアルキーのセグメントがノードに保存されます。

TiKVは、後者の解決策を選択し、全体のキー・バリュースペースをいくつかの連続したキーセグメントに分割します。各セグメントはリージョンと呼ばれます。各リージョンは`[StartKey, EndKey)`、つまり左閉区間と右開区間で表されます。各リージョンのデフォルトのサイズ制限は96 MiBであり、このサイズは設定可能です。

![TiDBのリージョン](/media/tidb-storage-2.png)

ここでのリージョンは、SQLのテーブルとは何の関係もありません。この文書ではSQLについては忘れて、現時点ではKVに焦点を当てます。データをリージョンに分割した後、TiKVは2つの重要なタスクを実行します：

* クラスタ内のすべてのノードにデータを分散し、リージョンを基本単位として使用します。各ノード上のリージョンの数がおおよそ同じになるようにします。
* リージョンでのRaftレプリケーションとメンバーシップ管理を実行します。

これら2つのタスクは非常に重要であり、一つずつ紹介されます。

* まず、データはキーに従って多くのリージョンに分割され、各リージョンのデータは1つのノード上にのみ保存されます（複数のレプリカは無視します）。TiDBシステムにはPDコンポーネントがあり、このコンポーネントはクラスタ内のすべてのノードにリージョンをできるだけ均等に広げる責任があります。この方法で、一方ではストレージ容量を水平にスケールさせることができます（他のノードのリージョンは自動的に新たに追加されたノードにスケジュールされます）し、他方では負荷分散が達成されます（1つのノードに多くのデータが格納され、他のノードにはほとんどデータがない状況は発生しません）。

    同時に、上位クライアントが必要なデータにアクセスできるように、システムにはノード上のリージョンの分布を記録するコンポーネント（PD）があります。つまり、任意のキーを通じて、リージョンの正確なリージョンとそのリージョンが配置されたノードを記録します。

* 2つめのタスクについて、TiKVはリージョンでデータをレプリケートし、つまり1つのリージョンのデータは複数のレプリカを持ち、その名前は"Replica"です。リージョンの複数のレプリカは異なるノードに保存され、Raftアルゴリズムを介して一貫性を保ちます。

    レプリカの1つがグループのリーダーを務め、他はフォロワーを務めます。デフォルトでは、すべての読み取りと書き込みはリーダーを介して処理され、読み取りが行われ、書き込みはフォロワーにレプリケートされます。次のダイアグラムはリージョンとRaftグループに関する全体像を示しています。

![TiDBストレージ](/media/tidb-storage-3.png)

リージョンでデータを分散・レプリケートすることにより、ある程度の災害回復能力を持つ分散型キー・バリューシステムが構築されます。容量やディスク障害、データ損失について心配する必要はもはやありません。

## MVCC

多くのデータベースはマルチバージョン同時実行制御（MVCC）を実装しており、TiKVも例外ではありません。2つのクライアントが同時に1つのキーの値を変更する状況を想像してみてください。MVCCがないと、データはロックされる必要があります。分散シナリオでは、パフォーマンスの問題やデッドロックの問題を引き起こす可能性があります。TiKVのMVCC実装は、キーにバージョン番号を追加することで実珸されます。要するに、MVCCがない場合、TiKVのデータレイアウトは次のようになります：

```
キー1 -> 値
キー2 -> 値
……
キーN -> 値
```

MVCCがある場合、TiKVのキー配列は次のようになります：

```
キー1_バージョン3 -> 値
キー1_バージョン2 -> 値
キー1_バージョン1 -> 値
……
キー2_バージョン4 -> 値
キー2_バージョン3 -> 値
キー2_バージョン2 -> 値
キー2_バージョン1 -> 値
……
キーN_バージョン2 -> 値
キーN_バージョン1 -> 値
……
```

同じキーの複数のバージョンの場合、より大きな番号のバージョンが先に配置されます（順にキーが配置される[キー・バリュー](#key-value-pairs) のセクションを参照してください）。そのため、キー+バージョンを使って値を取得する場合、MVCCのキーはキーとバージョン、つまり`キー_バージョン`と構築され、RocksDBの`SeekPrefix(キー_バージョン)` APIを使用して、これに大なり記号か等しい最初の位置を直接見つけることができます。

## 分散ACIDトランザクション

TiKVのトランザクションは、GoogleのBigTableで使用されているモデル「[Percolator](https://research.google.com/pubs/pub36726.html)」を採用しています。TiKVの実装はこの論文から着想を得ており、多くの最適化が行われています。詳細については、[transaction overview](/transaction-overview.md) をご覧ください。