---
title: フォロワーリード
summary: クエリの性能を最適化するためのフォロワーリードの使用方法を学びます。

# フォロワーリード

このドキュメントでは、クエリの性能を最適化するためのフォロワーリードの使用方法について紹介します。

## 導入

TiDBは、データをクラスタ内のすべてのノードに配信する基本単位として[リージョン](/tidb-storage.md#region)を使用します。リージョンには複数のレプリカがあり、そのレプリカはリーダーと複数のフォロワーに分かれています。リーダーのデータが変更されると、TiDBはそのデータをフォロワーに同期的に更新します。

デフォルトでは、TiDBは同じリージョンのリーダーでのみデータの読み込みと書き込みを行います。リージョン内で読み取りホットスポットが発生すると、リージョンのリーダーはシステム全体の読み取りボトルネックとなります。このような状況では、フォロワーリード機能を有効にすることで、複数のフォロワー間で負荷を均等に分散することによって、リーダーの負荷を大幅に減らし、システム全体のスループットを向上させることができます。

## 使用するタイミング

### 読み取りホットスポットを減らす

<CustomContent platform="tidb">

[Key Visualizerページ](/dashboard/dashboard-key-visualizer.md)でアプリケーションにホットスポットリージョンがあるかどうかを視覚的に分析することができます。 "metrics selection box"を`Read (bytes)`または`Read (keys)`に選択することで、読み取りホットスポットが発生しているかどうかを確認できます。

ホットスポットの処理について詳しくは、[TiDB Hotspot Problem Handling](/troubleshoot-hot-spot-issues.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Key Visualizerページ](/tidb-cloud/tune-performance.md#key-visualizer)でアプリケーションにホットスポットリージョンがあるかどうかを視覚的に分析することができます。 "metrics selection box"を`Read (bytes)`または`Read (keys)`に選択することで、読み取りホットスポットが発生しているかどうかを確認できます。

ホットスポットの処理について詳しくは、[TiDB Hotspot Problem Handling](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues)を参照してください。

</CustomContent>

読み取りホットスポットが回避できない場合、または変更コストが非常に高い場合は、フォロワーリード機能を使用して、フォロワーリージョンへの読み取りリクエストの負荷をより均等に分散させることができます。

### 地理的に分散したデプロイメントの遅延を減らす

TiDBクラスタが地区やデータセンターにまたがって展開されている場合、リージョンの異なるレプリカが異なる地区やデータセンターに分散されています。この場合、TiDBを`closest-adaptive`または`closest-replicas`として設定することで、現在のデータセンターからの読み取りを優先させることができます。これにより、読み取り操作の遅延とトラフィックオーバーヘッドを大幅に減らすことができます。実装の詳細については、[Follower Read](/follower-read.md)を参照してください。

## フォロワーリードの有効化

<SimpleTab groupId="language">
<div label="SQL" value="sql">

フォロワーリードを有効にするには、変数`tidb_replica_read`（デフォルト値は`leader`）を`follower`、`leader-and-follower`、`prefer-leader`、`closest-replicas`、または`closest-adaptive`に設定します。

```sql
SET [GLOBAL] tidb_replica_read = 'follower';
```

この変数の詳細については、[Follower Read Usage](/follower-read.md#usage)を参照してください。

</div>
<div label="Java" value="java">

Javaでは、`FollowerReadHelper`クラスを定義してフォロワーリードを有効にします。

```java
public enum FollowReadMode {
    LEADER("leader"),
    FOLLOWER("follower"),
    LEADER_AND_FOLLOWER("leader-and-follower"),
    CLOSEST_REPLICA("closest-replica"),
    CLOSEST_ADAPTIVE("closest-adaptive"),
    PREFER_LEADER("prefer-leader");

    private final String mode;

    FollowReadMode(String mode) {
        this.mode = mode;
    }

    public String getMode() {
        return mode;
    }
}

public class FollowerReadHelper {

    public static void setSessionReplicaRead(Connection conn, FollowReadMode mode) throws SQLException {
        if (mode == null) mode = FollowReadMode.LEADER;
        PreparedStatement stmt = conn.prepareStatement(
            "SET @@tidb_replica_read = ?;"
        );
        stmt.setString(1, mode.getMode());
        stmt.execute();
    }

    public static void setGlobalReplicaRead(Connection conn, FollowReadMode mode) throws SQLException {
        if (mode == null) mode = FollowReadMode.LEADER;
        PreparedStatement stmt = conn.prepareStatement(
            "SET GLOBAL @@tidb_replica_read = ?;"
        );
        stmt.setString(1, mode.getMode());
        stmt.execute();
    }

}
```

フォロワーノードからデータを読む場合は、現在のセッションでリーダーノードとフォロワーノード間の負荷を均等に分散するフォロワーリード機能を有効にするために、`setSessionReplicaRead(conn, FollowReadMode.LEADER_AND_FOLLOWER)`メソッドを使用します。接続が切断されると、元のモードに戻ります。

```java
public static class AuthorDAO {

    // インスタンス変数の初期化を省略...

    public void getAuthorsByFollowerRead() throws SQLException {
        try (Connection conn = ds.getConnection()) {
            // フォロワーリード機能を有効にします。
            FollowerReadHelper.setSessionReplicaRead(conn, FollowReadMode.LEADER_AND_FOLLOWER);

            // 作家のリストを10万回読み取ります。
            Random random = new Random();
            for (int i = 0; i < 100000; i++) {
                Integer birthYear = 1920 + random.nextInt(100);
                List<Author> authors = this.getAuthorsByBirthYear(birthYear);
                System.out.println(authors.size());
            }
        }
    }

    public List<Author> getAuthorsByBirthYear(Integer birthYear) throws SQLException {
        List<Author> authors = new ArrayList<>();
        try (Connection conn = ds.getConnection()) {
            PreparedStatement stmt = conn.prepareStatement("SELECT id, name FROM authors WHERE birth_year = ?");
            stmt.setInt(1, birthYear);
            ResultSet rs = stmt.executeQuery();
            while (rs.next()) {
                Author author = new Author();
                author.setId( rs.getLong("id"));
                author.setName(rs.getString("name"));
                authors.add(author);
            }
        }
        return authors;
    }
}
```

</div>
</SimpleTab>

## もっと詳しく読む

- [フォロワーリード](/follower-read.md)

<CustomContent platform="tidb">

- [ホットスポットのトラブルシューティング](/troubleshoot-hot-spot-issues.md)
- [TiDBダッシュボード - Key Visualizerページ](/dashboard/dashboard-key-visualizer.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

- [ホットスポットのトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues)
- [TiDBクラウド Key Visualizerページ](/tidb-cloud/tune-performance.md#key-visualizer)

</CustomContent>