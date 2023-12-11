---
title: TiDBとJavaを使用してシンプルなCRUDアプリを構築する
summary: TiDBとJavaを使用してシンプルなCRUDアプリケーションを構築する方法を学びます。
---

<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD029 -->

# TiDBとJavaを使用したシンプルなCRUDアプリの構築

このドキュメントでは、TiDBとJavaを使用してシンプルなCRUDアプリケーションを構築する方法について説明します。

> **注：**
>
> Java 8またはそれ以降のバージョンを使用することをお勧めします。
>
> アプリケーション開発にSpring Bootを使用する場合は、[Spring Bootを使用したTiDBアプリケーションの構築](/develop/dev-guide-sample-application-java-spring-boot.md)を参照してください。

## ステップ1. TiDBクラスタを起動する

<CustomContent platform="tidb">

以下では、TiDBクラスタの起動方法について紹介します。

**TiDBサーバーレスクラスタを使用する**

詳細な手順については、[TiDBサーバーレスクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md#step-1-create-a-tidb-serverless-cluster)を参照してください。

**ローカルクラスタを使用する**

詳細な手順については、[ローカルテストクラスタをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[TiUPを使用してTiDBクラスタをデプロイする](/production-deployment-using-tiup.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[TiDBサーバーレスクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md#step-1-create-a-tidb-serverless-cluster)を参照してください。

</CustomContent>

## ステップ2. コードを取得する

```shell
git clone https://github.com/pingcap-inc/tidb-example-java.git
```

<SimpleTab groupId="language">

<div label="Mybatisを使用する（推奨）" value="mybatis">

[Mybatis](https://mybatis.org/mybatis-3/index.html)と比較して、JDBC実装は最良の慣習ではないかもしれません。なぜなら、エラーハンドリングロジックを手動で記述する必要があり、コードを簡単に再利用できないため、コードがやや冗長になるからです。

Mybatisは人気のあるオープンソースのJavaクラス永続性フレームワークです。次は、[MyBatis Generator](https://mybatis.org/generator/quickstart.html)をMavenプラグインとして使用し、永続性層コードを生成する方法です。

`plain-java-mybatis`ディレクトリに移動します：

```shell
cd plain-java-mybatis
```

このディレクトリの構造は以下のようになります：

```
.
├── Makefile
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── pingcap
        │           ├── MybatisExample.java
        │           ├── dao
        │           │   └── PlayerDAO.java
        │           └── model
        │               ├── Player.java
        │               ├── PlayerMapper.java
        │               └── PlayerMapperEx.java
        └── resources
            ├── dbinit.sql
            ├── log4j.properties
            ├── mapper
            │   ├── PlayerMapper.xml
            │   └── PlayerMapperEx.xml
            ├── mybatis-config.xml
            └── mybatis-generator.xml
```

自動生成されるファイルは以下の通りです：

- `src/main/java/com/pingcap/model/Player.java`: `Player`エンティティクラスです。
- `src/main/java/com/pingcap/model/PlayerMapper.java`: `PlayerMapper`のインタフェースです。
- `src/main/resources/mapper/PlayerMapper.xml`: `Player`のXMLマッピングです。 Mybatisはこの構成を使用して`PlayerMapper`インタフェースの実装クラスを自動生成します。

これらのファイルを生成するための戦略は、[Mybatis Generator](https://mybatis.org/generator/quickstart.html)の設定ファイルである`mybatis-generator.xml`に記述されています。使用方法については、以下の設定ファイル内のコメントを参照してください。

```xml
<!DOCTYPE generatorConfiguration PUBLIC
 "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
 "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--
        <context/> entire document: https://mybatis.org/generator/configreference/context.html

        context.id: A unique identifier you like
        context.targetRuntime: Used to specify the runtime target for generated code.
            It has MyBatis3DynamicSql / MyBatis3Kotlin / MyBatis3 / MyBatis3Simple 4 selection to choice.
    -->
    <context id="simple" targetRuntime="MyBatis3">
        <!--
            <commentGenerator/> entire document: https://mybatis.org/generator/configreference/commentGenerator.html

            commentGenerator:
                - property(suppressDate): remove timestamp in comments
                - property(suppressAllComments): remove all comments
        -->
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true" />
        </commentGenerator>

        <!--
            <jdbcConnection/> entire document: https://mybatis.org/generator/configreference/jdbcConnection.html

            jdbcConnection.driverClass: The fully qualified class name for the JDBC driver used to access the database.
                Used mysql-connector-java:5.1.49, should specify JDBC is com.mysql.jdbc.Driver
            jdbcConnection.connectionURL: The JDBC connection URL used to access the database.
        -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
            connectionURL="jdbc:mysql://localhost:4000/test?user=root" />

        <!-- 以下省略 -->
```

`mybatis-generator.xml`は、`pom.xml`にMavenプラグインとして含まれています。

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.4.1</version>
    <configuration>
        <configurationFile>src/main/resources/mybatis-generator.xml</configurationFile>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
    </configuration>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.49</version>
        </dependency>
    </dependencies>
</plugin>
```

Mavenプラグインに含まれてから、古い自動生成されたファイルを削除し、`mvn mybatis-generate`を使用して新しいファイルを生成するか、同時に`make gen`を使用することができます。

> **注：**
>
> `mybatis-generator.xml`の`configuration.overwrite`プロパティは、生成されたJavaコードファイルを上書きすることを保証しますが、XMLマッピングファイルは追加されたままで書かれます。そのため、Mybatis Generatorが新しいファイルを生成する前に古いファイルを削除することをお勧めします。

```java
package com.pingcap.model;

public class Player {
    private String id;
    private Integer coins;
    private Integer goods;
    
    public Player(String id, Integer coins, Integer goods) {
        this.id = id;
        this.coins = coins;
        this.goods = goods;
    }
    
    public Player() {
        super();
    }
    
    public String getId() {
        return id;
    }
    
    public void setId(String id) {
        this.id = id;
    }
    
    public Integer getCoins() {
        return coins;
    }
    
    public void setCoins(Integer coins) {
        this.coins = coins;
    }
    
    public Integer getGoods() {
        return goods;
    }
    
    public void setGoods(Integer goods) {
        this.goods = goods;
    }
}
```

```java
package com.pingcap.model;

import com.pingcap.model.Player;

public interface PlayerMapper {
    int deleteByPrimaryKey(String id);

    int insert(Player row);

    int insertSelective(Player row);

    Player selectByPrimaryKey(String id);

    int updateByPrimaryKeySelective(Player row);

    int updateByPrimaryKey(Player row);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
  <resultMap id="BaseResultMap" type="com.pingcap.model.Player">
    <constructor>
      <idArg column="id" javaType="java.lang.String" jdbcType="VARCHAR" />
      <arg column="coins" javaType="java.lang.Integer" jdbcType="INTEGER" />
      <arg column="goods" javaType="java.lang.Integer" jdbcType="INTEGER" />
    </constructor>
  </resultMap>
  <sql id="Base_Column_List">
    id, coins, goods
  </sql>
  <select id="selectByPrimaryKey" parameterType="java.lang.String" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from player
    where id = #{id, jdbcType=VARCHAR}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.String">
    delete from player
    where id = #{id, jdbcType=VARCHAR}
  </delete>
  <insert id="insert" parameterType="com.pingcap.model.Player">
    insert into player (id, coins, goods
      )
    values (#{id, jdbcType=VARCHAR}, #{coins, jdbcType=INTEGER}, #{goods, jdbcType=INTEGER}
      )
  </insert>
  <insert id="insertSelective" parameterType="com.pingcap.model.Player">
    insert into player
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="coins != null">
        coins,
      </if>
      <if test="goods != null">
        goods,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id, jdbcType=VARCHAR},
      </if>
      <if test="coins != null">
        #{coins, jdbcType=INTEGER},
      </if>
      <if test="goods != null">
        #{goods, jdbcType=INTEGER},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="com.pingcap.model.Player">
    update player
    <set>
      <if test="coins != null">
        coins = #{coins, jdbcType=INTEGER},
      </if>
      <if test="goods != null">
        goods = #{goods, jdbcType=INTEGER},
      </if>
    </set>
    where id = #{id, jdbcType=VARCHAR}
  </update>
  <update id="updateByPrimaryKey" parameterType="com.pingcap.model.Player">
    update player
    set coins = #{coins, jdbcType=INTEGER},
      goods = #{goods, jdbcType=INTEGER}
    where id = #{id, jdbcType=VARCHAR}
  </update>
</mapper>
```

```sql
USE test;
DROP TABLE IF EXISTS player;

CREATE TABLE player (
    `id` VARCHAR(36),
    `coins` INTEGER,
    `goods` INTEGER,
    PRIMARY KEY (`id`)
);
```

```java
package com.pingcap.model;

import java.util.List;

public interface PlayerMapperEx extends PlayerMapper {
    Player selectByPrimaryKeyWithLock(String id);
    List<Player> selectByLimit(Integer limit);
    Integer count();
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapperEx">
  <resultMap id="BaseResultMap" type="com.pingcap.model.Player">
    <constructor>
      <idArg column="id" javaType="java.lang.String" jdbcType="VARCHAR" />
      <arg column="coins" javaType="java.lang.Integer" jdbcType="INTEGER" />
      <arg column="goods" javaType="java.lang.Integer" jdbcType="INTEGER" />
    </constructor>
  </resultMap>
  <sql id="Base_Column_List">
    id, coins, goods
  </sql>
  <select id="selectByPrimaryKeyWithLock" parameterType="java.lang.String" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from player
    where `id` = #{id, jdbcType=VARCHAR}
    for update
  </select>
  <select id="selectByLimit" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from player
    limit #{id, jdbcType=INTEGER}
  </select>
  <select id="count" resultType="java.lang.Integer">
    select count(*) from player
  </select>
</mapper>
```

```java
package com.pingcap.dao;

import com.pingcap.model.Player;
import com.pingcap.model.PlayerMapperEx;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

import java.util.List;
import java.util.function.Function;

public class PlayerDAO {
    public static class NotEnoughException extends RuntimeException {
        public NotEnoughException(String message) {
            super(message);
        }
    }

    public Object runTransaction(SqlSessionFactory sessionFactory, Function<PlayerMapperEx, Object> fn) {
        Object resultObject = null;
        SqlSession session = null;

        try {
            session = sessionFactory.openSession(false);
            PlayerMapperEx playerMapperEx = session.getMapper(PlayerMapperEx.class);

            resultObject = fn.apply(playerMapperEx);
            session.commit();
            System.out.println("APP: COMMIT;");
        } catch (Exception e) {
            if (e instanceof NotEnoughException) {
                System.out.printf("APP: ROLLBACK BY LOGIC; \n%s\n", e.getMessage());
            } else {
                System.out.printf("APP: ROLLBACK BY ERROR; \n%s\n", e.getMessage());
            }

            if (session != null) {
                session.rollback();
            }
        } finally {
            if (session != null) {
                session.close();
            }
        }

        return resultObject;
    }

    public Function<PlayerMapperEx, Object> createPlayers(List<Player> players) {
        return playerMapperEx -> {
            Integer addedPlayerAmount = 0;
            for (Player player: players) {
                playerMapperEx.insert(player);
                addedPlayerAmount ++;
            }
```
```java
package com.pingcap;

import com.pingcap.dao.PlayerDAO;
import com.pingcap.model.Player;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.Arrays;
import java.util.Collections;

public class MybatisExample {
    public static void main( String[] args ) throws IOException {
        // 1. Create a SqlSessionFactory based on our mybatis-config.xml configuration
        // file, which defines how to connect to the database.
        InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2. And then, create DAO to manager your data
        PlayerDAO playerDAO = new PlayerDAO();

        // 3. Run some simple examples.

        // Create a player who has 1 coin and 1 goods.
        playerDAO.runTransaction(sessionFactory, playerDAO.createPlayers(
                Collections.singletonList(new Player("test", 1, 1))));

        // Get a player.
        Player testPlayer = (Player)playerDAO.runTransaction(sessionFactory, playerDAO.getPlayerByID("test"));
        System.out.printf("PlayerDAO.getPlayer:\n    => id: %s\n    => coins: %s\n    => goods: %s\n",
                testPlayer.getId(), testPlayer.getCoins(), testPlayer.getGoods());

        // Count players amount.
        Integer count = (Integer)playerDAO.runTransaction(sessionFactory, playerDAO.countPlayers());
        System.out.printf("PlayerDAO.countPlayers:\n    => %d total players\n", count);

        // Print 3 players.
        playerDAO.runTransaction(sessionFactory, playerDAO.printPlayers(3));

        // 4. Getting further.

        // Player 1: id is "1", has only 100 coins.
        // Player 2: id is "2", has 114514 coins, and 20 goods.
        Player player1 = new Player("1", 100, 0);
        Player player2 = new Player("2", 114514, 20);

        // Create two players "by hand", using the INSERT statement on the backend.
        int addedCount = (Integer)playerDAO.runTransaction(sessionFactory,
                playerDAO.createPlayers(Arrays.asList(player1, player2)));
        System.out.printf("PlayerDAO.createPlayers:\n    => %d total inserted players\n", addedCount);

        // Player 1 wants to buy 10 goods from player 2.
        // It will cost 500 coins, but player 1 cannot afford it.
        System.out.println("\nPlayerDAO.buyGoods:\n    => this trade will fail");
        Integer updatedCount = (Integer)playerDAO.runTransaction(sessionFactory,
                playerDAO.buyGoods(player2.getId(), player1.getId(), 10, 500));
        System.out.printf("PlayerDAO.buyGoods:\n    => %d total update players\n", updatedCount);

        // So player 1 has to reduce the incoming quantity to two.
        System.out.println("\nPlayerDAO.buyGoods:\n    => this trade will success");
        updatedCount = (Integer)playerDAO.runTransaction(sessionFactory,
                playerDAO.buyGoods(player2.getId(), player1.getId(), 2, 100));
        System.out.printf("PlayerDAO.buyGoods:\n    => %d total update players\n", updatedCount);
    }
}
```
```java
    @Column(name = "goods")
    private Integer goods;

    public PlayerBean() {
    }

    public PlayerBean(String id, Integer coins, Integer goods) {
        this.id = id;
        this.coins = coins;
        this.goods = goods;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public Integer getCoins() {
        return coins;
    }

    public void setCoins(Integer coins) {
        this.coins = coins;
    }

    public Integer getGoods() {
        return goods;
    }

    public void setGoods(Integer goods) {
        this.goods = goods;
    }

    @Override
    public String toString() {
        return String.format("    %-8s => %10s\n    %-8s => %10s\n    %-8s => %10s\n",
                "id", this.id, "coins", this.coins, "goods", this.goods);
    }
}

/**
 * Main class for the basic Hibernate example.
 **/
public class HibernateExample
{
    public static class PlayerDAO {
        public static class NotEnoughException extends RuntimeException {
            public NotEnoughException(String message) {
                super(message);
            }
        }

        // Run SQL code in a way that automatically handles the
        // transaction retry logic so we don't have to duplicate it in
        // various places.
        public Object runTransaction(Session session, Function<Session, Object> fn) {
            Object resultObject = null;

            Transaction txn = session.beginTransaction();
            try {
                resultObject = fn.apply(session);
                txn.commit();
                System.out.println("APP: COMMIT;");
            } catch (JDBCException e) {
                System.out.println("APP: ROLLBACK BY JDBC ERROR;");
                txn.rollback();
            } catch (NotEnoughException e) {
                System.out.printf("APP: ROLLBACK BY LOGIC; %s", e.getMessage());
                txn.rollback();
            }
            return resultObject;
        }

        public Function<Session, Object> createPlayers(List<PlayerBean> players) throws JDBCException {
            return session -> {
                Integer addedPlayerAmount = 0;
                for (PlayerBean player: players) {
                    session.persist(player);
                    addedPlayerAmount ++;
                }
                System.out.printf("APP: createPlayers() --> %d\n", addedPlayerAmount);
                return addedPlayerAmount;
            };
        }

        public Function<Session, Object> buyGoods(String sellId, String buyId, Integer amount, Integer price) throws JDBCException {
            return session -> {
                PlayerBean sellPlayer = session.get(PlayerBean.class, sellId);
                PlayerBean buyPlayer = session.get(PlayerBean.class, buyId);

                if (buyPlayer == null || sellPlayer == null) {
                    throw new NotEnoughException("sell or buy player not exist");
                }

                if (buyPlayer.getCoins() < price || sellPlayer.getGoods() < amount) {
                    throw new NotEnoughException("coins or goods not enough, rollback");
                }

                buyPlayer.setGoods(buyPlayer.getGoods() + amount);
                buyPlayer.setCoins(buyPlayer.getCoins() - price);
                session.persist(buyPlayer);

                sellPlayer.setGoods(sellPlayer.getGoods() - amount);
                sellPlayer.setCoins(sellPlayer.getCoins() + price);
                session.persist(sellPlayer);

                System.out.printf("APP: buyGoods --> sell: %s, buy: %s, amount: %d, price: %d\n", sellId, buyId, amount, price);
                return 0;
            };
        }

        public Function<Session, Object> getPlayerByID(String id) throws JDBCException {
            return session -> session.get(PlayerBean.class, id);
        }

        public Function<Session, Object> printPlayers(Integer limit) throws JDBCException {
            return session -> {
                NativeQuery<PlayerBean> limitQuery = session.createNativeQuery("SELECT * FROM player_hibernate LIMIT :limit", PlayerBean.class);
                limitQuery.setParameter("limit", limit);
                List<PlayerBean> players = limitQuery.getResultList();

                for (PlayerBean player: players) {
                    System.out.println("\n[printPlayers]:\n" + player);
                }
                return 0;
            };
        }

        public Function<Session, Object> countPlayers() throws JDBCException {
            return session -> {
                Query<Long> countQuery = session.createQuery("SELECT count(player_hibernate) FROM PlayerBean player_hibernate", Long.class);
                return countQuery.getSingleResult();
            };
        }
    }

    public static void main(String[] args) {
        // 1. Create a SessionFactory based on our hibernate.cfg.xml configuration
        // file, which defines how to connect to the database.
        SessionFactory sessionFactory
                = new Configuration()
                .configure("hibernate.cfg.xml")
                .addAnnotatedClass(PlayerBean.class)
                .buildSessionFactory();

        try (Session session = sessionFactory.openSession()) {
            // 2. And then, create DAO to manager your data.
            PlayerDAO playerDAO = new PlayerDAO();

            // 3. Run some simple example.

            // Create a player who has 1 coin and 1 goods.
            playerDAO.runTransaction(session, playerDAO.createPlayers(Collections.singletonList(
                    new PlayerBean("test", 1, 1))));

            // Get a player.
            PlayerBean testPlayer = (PlayerBean)playerDAO.runTransaction(session, playerDAO.getPlayerByID("test"));
            System.out.printf("PlayerDAO.getPlayer:\n    => id: %s\n    => coins: %s\n    => goods: %s\n",
                    testPlayer.getId(), testPlayer.getCoins(), testPlayer.getGoods());

            // Count players amount.
            Long count = (Long)playerDAO.runTransaction(session, playerDAO.countPlayers());
            System.out.printf("PlayerDAO.countPlayers:\n    => %d total players\n", count);

            // Print 3 players.
            playerDAO.runTransaction(session, playerDAO.printPlayers(3));

            // 4. Getting further.

            // Player 1: id is "1", has only 100 coins.
            // Player 2: id is "2", has 114514 coins, and 20 goods.
            PlayerBean player1 = new PlayerBean("1", 100, 0);
            PlayerBean player2 = new PlayerBean("2", 114514, 20);

            // Create two players "by hand", using the INSERT statement on the backend.
            int addedCount = (Integer)playerDAO.runTransaction(session,
                    playerDAO.createPlayers(Arrays.asList(player1, player2)));
            System.out.printf("PlayerDAO.createPlayers:\n    => %d total inserted players\n", addedCount);

            // Player 1 wants to buy 10 goods from player 2.
            // It will cost 500 coins, but player 1 can't afford it.
            System.out.println("\nPlayerDAO.buyGoods:\n    => this trade will fail");
            Integer updatedCount = (Integer)playerDAO.runTransaction(session,
                    playerDAO.buyGoods(player2.getId(), player1.getId(), 10, 500));
            System.out.printf("PlayerDAO.buyGoods:\n    => %d total update players\n", updatedCount);

            // So player 1 have to reduce his incoming quantity to two.
            System.out.println("\nPlayerDAO.buyGoods:\n    => this trade will success");
            updatedCount = (Integer)playerDAO.runTransaction(session,
                    playerDAO.buyGoods(player2.getId(), player1.getId(), 2, 100));
            System.out.printf("PlayerDAO.buyGoods:\n    => %d total update players\n", updatedCount);
        } finally {
            sessionFactory.close();
        }
    }
}
```

</div>

<div label="Using JDBC" value="jdbc">

Change to the `plain-java-jdbc` directory:

```shell
cd plain-java-jdbc
```

The structure of this directory is as follows:

```
.
├── Makefile
├── plain-java-jdbc.iml
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── pingcap
        │            └── JDBCExample.java
        └── resources
             └── dbinit.sql
```

You can find initialization statements for the table creation in `dbinit.sql`:

```sql
USE test;
DROP TABLE IF EXISTS player;

CREATE TABLE player (
    `id` VARCHAR(36),
    `coins` INTEGER,
    `goods` INTEGER,
   PRIMARY KEY (`id`)
);
```

`JDBCExample.java` is the main body of the `plain-java-jdbc`. TiDB is highly compatible with the MySQL protocol, so you need to initialize a MySQL source instance `MysqlDataSource` to connect to TiDB. Then, you can initialize `PlayerDAO` for object management and use it to read, edit, add, and delete data.

`PlayerDAO` is a class used to manage data, in which `DAO` means [Data Access Object](https://en.wikipedia.org/wiki/Data_access_object). The class defines a set of data manipulation methods to provide the ability to write data.

`PlayerBean` is a data entity class that is a mapping for tables. Each property of a `PlayerBean` corresponds to a field in the `player` table.
```java
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.*;

/**
 * 基本的なJDBCの例のためのメインクラス。
 **/
public class JDBCExample
{
    public static class PlayerBean {
        private String id;
        private Integer coins;
        private Integer goods;

        public PlayerBean() {
        }

        public PlayerBean(String id, Integer coins, Integer goods) {
            this.id = id;
            this.coins = coins;
            this.goods = goods;
        }

        public String getId() {
            return id;
        }

        public void setId(String id) {
            this.id = id;
        }

        public Integer getCoins() {
            return coins;
        }

        public void setCoins(Integer coins) {
            this.coins = coins;
        }

        public Integer getGoods() {
            return goods;
        }

        public void setGoods(Integer goods) {
            this.goods = goods;
        }

        @Override
        public String toString() {
            return String.format("    %-8s => %10s\n    %-8s => %10s\n    %-8s => %10s\n",
                    "id", this.id, "coins", this.coins, "goods", this.goods);
        }
    }

    /**
     * 'ExampleDataSource'で使用されるデータアクセスオブジェクト。
     * CURDと一括挿入の例。
     */
    public static class PlayerDAO {
        private final MysqlDataSource ds;
        private final Random rand = new Random();

        PlayerDAO(MysqlDataSource ds) {
            this.ds = ds;
        }

        /**
         * PlayerBeanのリストを渡してプレイヤーを作成する。
         *
         * @param players 作成するプレイヤーリスト
         * @return 作成されたアカウントの数
         */
        public int createPlayers(List<PlayerBean> players){
            int rows = 0;

            Connection connection = null;
            PreparedStatement preparedStatement = null;
            try {
                connection = ds.getConnection();
                preparedStatement = connection.prepareStatement("INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)");
            } catch (SQLException e) {
                System.out.printf("[createPlayers] ERROR: { state => %s, cause => %s, message => %s }\n",
                        e.getSQLState(), e.getCause(), e.getMessage());
                e.printStackTrace();

                return -1;
            }

            try {
                for (PlayerBean player : players) {
                    preparedStatement.setString(1, player.getId());
                    preparedStatement.setInt(2, player.getCoins());
                    preparedStatement.setInt(3, player.getGoods());

                    preparedStatement.execute();
                    rows += preparedStatement.getUpdateCount();
                }
            } catch (SQLException e) {
                System.out.printf("[createPlayers] ERROR: { state => %s, cause => %s, message => %s }\n",
                        e.getSQLState(), e.getCause(), e.getMessage());
                e.printStackTrace();
            } finally {
                try {
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

            System.out.printf("\n[createPlayers]:\n    '%s'\n", preparedStatement);
            return rows;
        }

        /**
         * 商品を購入し、1つのトランザクションで1つのプレイヤーから別のプレイヤーに資金を移し、商品を購入します。
         * @param sellId 売るプレイヤーのID
         * @param buyId 買うプレイヤーのID
         * @param amount 商品の量。売るプレイヤーが十分な商品を持っていない場合、取引は中断されます。
         * @param price 支払う必要がある価格。買うプレイヤーが十分なコインを持っていない場合、取引は中断されます。
         *
         * @return 影響を受けたプレイヤーの数
         */
        public int buyGoods(String sellId, String buyId, Integer amount, Integer price) {
            int effectPlayers = 0;

            Connection connection = null;
            try {
                connection = ds.getConnection();
            } catch (SQLException e) {
                System.out.printf("[buyGoods] ERROR: { state => %s, cause => %s, message => %s }\n",
                        e.getSQLState(), e.getCause(), e.getMessage());
                e.printStackTrace();
                return effectPlayers;
            }

            try {
                connection.setAutoCommit(false);

                PreparedStatement playerQuery = connection.prepareStatement("SELECT * FROM player WHERE id=? OR id=? FOR UPDATE");
                playerQuery.setString(1, sellId);
                playerQuery.setString(2, buyId);
                playerQuery.execute();

                PlayerBean sellPlayer = null;
                PlayerBean buyPlayer = null;

                ResultSet playerQueryResultSet = playerQuery.getResultSet();
                while (playerQueryResultSet.next()) {
                    PlayerBean player =  new PlayerBean(
                            playerQueryResultSet.getString("id"),
                            playerQueryResultSet.getInt("coins"),
                            playerQueryResultSet.getInt("goods")
                    );

                    System.out.println("\n[buyGoods]:\n    '商品とコインが十分か確認'");
                    System.out.println(player);

                    if (sellId.equals(player.getId())) {
                        sellPlayer = player;
                    } else {
                        buyPlayer = player;
                    }
                }

                if (sellPlayer == null || buyPlayer == null) {
                    throw new SQLException("プレイヤーが存在しません。");
                }

                if (sellPlayer.getGoods().compareTo(amount) < 0) {
                    throw new SQLException(String.format("売るプレイヤー%sの商品が不足しています。", sellId));
                }

                if (buyPlayer.getCoins().compareTo(price) < 0) {
                    throw new SQLException(String.format("買うプレイヤー%sのコインが不足しています。", buyId));
                }

                PreparedStatement transfer = connection.prepareStatement("UPDATE player set goods = goods + ?, coins = coins + ? WHERE id=?");
                transfer.setInt(1, -amount);
                transfer.setInt(2, price);
                transfer.setString(3, sellId);
                transfer.execute();
                effectPlayers += transfer.getUpdateCount();

                transfer.setInt(1, amount);
                transfer.setInt(2, -price);
                transfer.setString(3, buyId);
                transfer.execute();
                effectPlayers += transfer.getUpdateCount();

                connection.commit();

                System.out.println("\n[buyGoods]:\n    '取引成功'");
            } catch (SQLException e) {
                System.out.printf("[buyGoods] ERROR: { state => %s, cause => %s, message => %s }\n",
                        e.getSQLState(), e.getCause(), e.getMessage());

                try {
                    System.out.println("[buyGoods] ロールバック");

                    connection.rollback();
                } catch (SQLException ex) {
                    // 特に何もしない
                }
            } finally {
                try {
                    connection.close();
                } catch (SQLException e) {
                    // 特に何もしない
                }
            }

            return effectPlayers;
        }

        /**
         * IDでプレイヤー情報を取得する。
         *
         * @param id プレイヤーID
         * @return このIDのプレイヤー
         */
        public PlayerBean getPlayer(String id) {
            PlayerBean player = null;

            try (Connection connection = ds.getConnection()) {
                PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM player WHERE id = ?");
                preparedStatement.setString(1, id);
                preparedStatement.execute();

                ResultSet res = preparedStatement.executeQuery();
                if(!res.next()) {
                    System.out.printf("ID %sのテーブルにプレイヤーがいません", id);
                } else {
                    player = new PlayerBean(res.getString("id"), res.getInt("coins"), res.getInt("goods"));
                }
            } catch (SQLException e) {
                System.out.printf("PlayerDAO.getPlayer ERROR: { state => %s, cause => %s, message => %s }\n",
                        e.getSQLState(), e.getCause(), e.getMessage());
            }

            return player;
        }

        /**
         * JDBCの高速なパスを使用して、ランダムなアカウントデータ（id、coins、goods）を挿入します。
         * TiDBにデータを取得する最速の方法は、
         * TiDB Lightning (https://docs.pingcap.com/tidb/stable/tidb-lightning-overview) を使用することです。
         * ただし、INSERT SQLを使用してアプリケーションから一括挿入を行う必要がある場合は、ここで示した方法が最適です。
         * 以下が必要になります：
         *
         *    JDBC接続設定に `rewriteBatchedStatements=true` を追加します。
         *    rewriteBatchedStatements をtrueに設定すると、バッチ引数を持つCallableStatementsが
         *    "CALL (...); CALL (...); ..." の形式で書き換えられ、できるだけ少ないクライアント/サーバーラウンドトリップでバッチを送信します。
         *    https://dev.mysql.com/doc/relnotes/connector-j/5.1/en/news-5-1-3.html
         *
         *    `rewriteBatchedStatements` パラメータの効果ロジックは、
         *    `com.mysql.cj.jdbc.StatementImpl.executeBatchUsingMultiQueries` 関数で確認できます。
         *
         * @param total 追加プレイヤーの数
         * @param batchSize 1バッチあたりの一括挿入サイズ
         *
         * @return 挿入された新しいアカウントの数
         */
        public int bulkInsertRandomPlayers(Integer total, Integer batchSize) {
            int totalNewPlayers = 0;

            try (Connection connection = ds.getConnection()) {
                // コミットライフサイクルを自分で管理しているので、
                // バッチ挿入のサイズを制御できます。
                connection.setAutoCommit(false);

                // この例では、データベースに500行を追加しています。
                // 任意の数でも構いません。重要なのは、バッチサイズが128であることです。
                try (PreparedStatement pstmt = connection.prepareStatement("INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)")) {
                    for (int i=0; i<=(total/batchSize);i++) {
```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="false"/>
        <setting name="aggressiveLazyLoading" value="true"/>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
    <typeAliases>
        <package name="com.pingcap.dao"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <!-- JDBC transaction manager -->
            <transactionManager type="JDBC"/>
            <!-- Database pool -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:4000/test"/>
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

        ...
            <!-- Database pool -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://xxx.tidbcloud.com:4000/test?sslMode=VERIFY_IDENTITY&amp;enabledTLSProtocols=TLSv1.2,TLSv1.3"/>
                <property name="username" value="2aEp24QWEDLqRFs.root"/>
                <property name="password" value="123456"/>
            </dataSource>
        ...

</configuration>
```

If you are using a TiDB Serverless cluster, modify the `hibernate.connection.url`, `hibernate.connection.username`, `hibernate.connection.password` in `hibernate.cfg.xml`.

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>

        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.dialect">org.hibernate.dialect.TiDBDialect</property>
        <property name="hibernate.connection.url">jdbc:mysql://xxx.tidbcloud.com:4000/test?sslMode=VERIFY_IDENTITY&amp;enabledTLSProtocols=TLSv1.2,TLSv1.3</property>
        <property name="hibernate.connection.username">2aEp24QWEDLqRFs.root</property>
        <property name="hibernate.connection.password">123456</property>
        <property name="hibernate.connection.autocommit">false</property>

        <!-- Required so a table can be created from the 'PlayerDAO' class -->
        <property name="hibernate.hbm2ddl.auto">create-drop</property>

        <!-- Optional: Show SQL output for debugging -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

If you are using a TiDB Serverless cluster, modify the parameters of the host, port, user, and password in `JDBCExample.java`:

```java
mysqlDataSource.setServerName("xxx.tidbcloud.com");
mysqlDataSource.setPortNumber(4000);
mysqlDataSource.setDatabaseName("test");
mysqlDataSource.setUser("2aEp24QWEDLqRFs.root");
mysqlDataSource.setPassword("123456");
mysqlDataSource.setSslMode(PropertyDefinitions.SslMode.VERIFY_IDENTITY.name());
mysqlDataSource.setEnabledTLSProtocols("TLSv1.2,TLSv1.3");
```

To run the code, you can run `make prepare`, `make gen`, `make build` and `make run` respectively:

```shell
make prepare
# this command executes :
# - `mysql --host 127.0.0.1 --port 4000 -u root < src/main/resources/dbinit.sql`
# - `mysql --host 127.0.0.1 --port 4000 -u root -e "TRUNCATE test.player"`

make gen
# this command executes :
# - `rm -f src/main/java/com/pingcap/model/Player.java`
# - `rm -f src/main/java/com/pingcap/model/PlayerMapper.java`
# - `rm -f src/main/resources/mapper/PlayerMapper.xml`
# - `mvn mybatis-generator:generate`

make build # this command executes `mvn clean package`
make run # this command executes `java -jar target/plain-java-mybatis-0.0.1-jar-with-dependencies.jar`
```
```plaintext
java -jar target/plain-java-jdbc-0.0.1-jar-with-dependencies.jar
```

または`make`コマンドを直接実行するか、「make build」と「make run」の組み合わせである`make`コマンドを実行します。

</div>

</SimpleTab>

## ステップ4. 期待される出力

<SimpleTab groupId="language">

<div label="Mybatisを使用する（推奨）" value="mybatis">

[Mybatis 期待される出力](https://github.com/pingcap-inc/tidb-example-java/blob/main/Expected-Output.md#plain-java-mybatis)

</div>

<div label="Hibernateを使用する（推奨）" value="hibernate">

[Hibernate 期待される出力](https://github.com/pingcap-inc/tidb-example-java/blob/main/Expected-Output.md#plain-java-hibernate)

</div>

<div label="JDBCを使用する" value="jdbc">

[JDBC 期待される出力](https://github.com/pingcap-inc/tidb-example-java/blob/main/Expected-Output.md#plain-java-jdbc)

</div>

</SimpleTab>
```