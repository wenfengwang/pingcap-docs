---
title: TLS Connections to TiDB Dedicated
summary: TiDB DedicatedへのTLS接続の紹介
aliases: ['/tidbcloud/tidb-cloud-tls-connect-to-dedicated-tier']
---

# TiDB DedicatedへのTLS接続

TiDB Cloudでは、TLS接続の構築はTiDB Dedicatedクラスタに接続するための基本的なセキュリティプラクティスの1つです。クライアント、アプリケーション、および開発ツールからTiDB Dedicatedクラスタに複数のTLS接続を構成して、データ送信のセキュリティを保護できます。セキュリティのため、TiDB DedicatedはTLS 1.2およびTLS 1.3のみをサポートし、TLS 1.0およびTLS 1.1のバージョンはサポートしていません。

データのセキュリティを確保するために、TiDBクラスタCAはTiDB Dedicatedクラスタの[AWS Certificate Manager（ACM）](https://aws.amazon.com/certificate-manager/)にホストされ、TiDBクラスタのプライベートキーは[FIPS 140-2 Level 3](https://csrc.nist.gov/projects/cryptographic-module-validation-program/Certificate/3139)のセキュリティ基準を満たすAWS管理のハードウェアセキュリティモジュール（HSM）に保存されています。

## 前提条件

- [パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)または[SSO認証](/tidb-cloud/tidb-cloud-sso-authentication.md)を使用してTiDB Cloudにログインし、[TiDB Dedicatedクラスタを作成](/tidb-cloud/create-tidb-cluster.md)します。

- 安全な設定でクラスタへのアクセス用のパスワードを設定します。

    これを行うには、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、TiDB Dedicatedクラスタの行で**...**をクリックし、その後**セキュリティ設定**を選択します。セキュリティ設定では、**Generate**をクリックして、数字、大文字と小文字の文字、特殊文字を含む長さが16文字のルートパスワードを自動的に生成できます。

## TiDB Dedicatedクラスタへの安全な接続

[TiDB Cloudコンソール](https://tidbcloud.com/)では、異なる接続方法の例とTiDB Dedicatedクラスタへの接続方法について次のように示されます。

1. プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、TiDB Dedicatedクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の**Connect**をクリックします。ダイアログが表示されます。

3. このダイアログの**標準接続**タブで、3つのステップに従ってTLS接続を設定します。
   - ステップ1：トラフィックフィルターの作成
   - ステップ2：TiDBクラスタCAのダウンロード
   - ステップ3：SQLクライアントで接続

4. ダイアログの**ステップ1：トラフィックフィルターの作成**では、クラスタへのアクセスが許可されるIPアドレスを構成します。詳細は[標準接続でIPアクセスリストを構成](/tidb-cloud/configure-ip-access-list.md#configure-an-ip-access-list-in-standard-connection)を参照してください。

5. ダイアログの**ステップ2：TiDBクラスタCAのダウンロード**では、**TiDBクラスタCAのダウンロード**をクリックして、クライアントのTLS構成のためにローカルにダウンロードします。TiDBクラスタCAはTLS接続を安全かつ信頼性のあるものにします。

    > **注意:**
    >
    > TiDB DedicatedクラスタのCAをダウンロードした後、それをオペレーティングシステムのデフォルトの保存パスに保存するか、別の保存パスを指定することができます。次の手順のコード例で、自分自身のクラスタCAパスでCAパスを置き換える必要があります。

6. ダイアログの**ステップ3：SQLクライアントで接続**では、希望の接続方法のタブをクリックし、その後タブ上の接続文字列とサンプルコードを参照してクラスタに接続します。

以下の例では、MySQL、MyCLI、JDBC、Python、Go、Node.jsの接続文字列が示されています:

<SimpleTab>
<div label="MySQL CLI">

MySQL CLIクライアントはデフォルトでTLS接続を確立しようとします。TiDB Dedicatedクラスタに接続する際は、`ssl-mode`および`ssl-ca`を設定する必要があります。

```shell
mysql --connect-timeout 15 --ssl-mode=VERIFY_IDENTITY --ssl-ca=ca.pem --tls-version="TLSv1.2" -u root -h tidb.eqlfbdgthh8.clusters.staging.tidb-cloud.com -P 4000 -D test -p
```

パラメータの説明：

- `--ssl-mode=VERIFY_IDENTITY`を使用すると、MySQL CLIクライアントはTLSを強制的に有効にし、TiDB Dedicatedクラスタを検証します。
- `--ssl-ca=<CA_path>`を使用して、ダウンロードしたTiDBクラスタの`ca.pem`のローカルパスを指定します。
- `--tls-version=TLSv1.2`を使用してTLSプロトコルのバージョンを制限します。TLS 1.3を使用したい場合は、バージョンを`TLSv1.3`に設定できます。

</div>

<div label="MyCLI">

[MyCLI](https://www.mycli.net/)はTLS関連のパラメータを使用する場合に自動的にTLSを有効にします。TiDB Dedicatedクラスタに接続する際は、`ssl-ca`および`ssl-verify-server-cert`を設定する必要があります。

```shell
mycli --ssl-ca=ca.pem --ssl-verify-server-cert -u root -h tidb.eqlfbdgthh8.clusters.staging.tidb-cloud.com -P 4000 -D test
```

パラメータの説明：

- `--ssl-ca=<CA_path>`を使用して、ダウンロードしたTiDBクラスタの`ca.pem`のローカルパスを指定します。
- `--ssl-verify-server-cert`を使用して、TiDB Dedicatedクラスタを検証します。

</div>

<div label="JDBC">

[MySQL Connector/J](https://dev.mysql.com/doc/connector-j/en/)のTLS接続構成がここで例として使用されます。

TiDBクラスタCAをダウンロードした後、それをオペレーティングシステムにインポートしたい場合は、`keytool -importcert -alias TiDBCACert -file ca.pem -keystore <your_custom_truststore_path> -storepass <your_truststore_password>`コマンドを使用できます。

```shell
/* 以下の接続文字列のパラメータを置き換えることを忘れないでください。 */
/* バージョン >= 8.0.28 */
jdbc:mysql://tidb.srgnqxji5bc.clusters.staging.tidb-cloud.com:4000/test?user=root&password=<your_password>&sslMode=VERIFY_IDENTITY&tlsVersions=TLSv1.2&trustCertificateKeyStoreUrl=file:<your_custom_truststore_path>&trustCertificateKeyStorePassword=<your_truststore_password>
```

詳細なコード例を表示するには、**show example usage**をクリックすることができます。

```
import com.mysql.jdbc.Driver;
import java.sql.*;

class Main {
  public static void main(String args[]) throws SQLException, ClassNotFoundException {
    Class.forName("com.mysql.cj.jdbc.Driver");
    try {
      Connection conn = DriverManager.getConnection("jdbc:mysql://tidb.srgnqxji5bc.clusters.staging.tidb-cloud.com:4000/test?user=root&password=<your_password>&sslMode=VERIFY_IDENTITY&tlsVersions=TLSv1.2&trustCertificateKeyStoreUrl=file:<your_custom_truststore_path>&trustCertificateKeyStorePassword=<your_truststore_password>");
      Statement stmt = conn.createStatement();
      try {
        ResultSet rs = stmt.executeQuery("SELECT DATABASE();");
        if (rs.next()) {
          System.out.println("using db:" + rs.getString(1));
        }
      } catch (Exception e) {
        System.out.println("exec error:" + e);
      }
    } catch (Exception e) {
      System.out.println("connect error:" + e);
    }
  }
}
```

パラメータの説明：

- `sslMode=VERIFY_IDENTITY`を設定してTLSを有効にし、TiDB Dedicatedクラスタを検証します。
- `enabledTLSProtocols=TLSv1.2`を設定して、TLSプロトコルのバージョンを制限します。TLS 1.3を使用したい場合は、バージョンを`TLSv1.3`に設定できます。
- `trustCertificateKeyStoreUrl`をカスタムトラストストアのパスに設定します。
- `trustCertificateKeyStorePassword`をトラストストアのパスワードに設定します。

</div>

<div label="Python">

[mysqlclient](https://pypi.org/project/mysqlclient/)のTLS接続構成がここで例として使用されます。

```
host="tidb.srgnqxji5bc.clusters.staging.tidb-cloud.com", user="root", password="<your_password>", port=4000, database="test", ssl_mode="VERIFY_IDENTITY", ssl={"ca": "ca.pem"}
```

詳細なコード例を表示するには、**show example usage**をクリックすることができます。

```
import MySQLdb

connection = MySQLdb.connect(host="tidb.srgnqxji5bc.clusters.staging.tidb-cloud.com", port=4000, user="root", password="<your_password>", database="test", ssl_mode="VERIFY_IDENTITY", ssl={"ca": "ca.pem"})

with connection:
    with connection.cursor() as cursor:
        cursor.execute("SELECT DATABASE();")
        m = cursor.fetchone()
        print(m[0])
```

パラメータの説明：

- `ssl_mode="VERIFY_IDENTITY"`を設定してTLSを有効にし、TiDB Dedicatedクラスタを検証します。
- `ssl={"ca": "<CA_path>"}`を使用して、ダウンロードしたTiDBクラスタの`ca.pem`のローカルパスを指定します。

</div>

<div label="Go">

[Go-MySQL-Driver](https://github.com/go-sql-driver/mysql)のTLS接続構成がここで例として使用されます。

```
```
package main
import (
  "crypto/tls"
  "crypto/x509"
  "database/sql"
  "fmt"
  "io/ioutil"
  "log"

  "github.com/go-sql-driver/mysql"
)
func main() {
  rootCertPool := x509.NewCertPool()
  pem, err := ioutil.ReadFile("ca.pem")
  if err != nil {
    log.Fatal(err)
  }
  if ok := rootCertPool.AppendCertsFromPEM(pem); !ok {
    log.Fatal("Failed to append PEM.")
  }
  mysql.RegisterTLSConfig("tidb", &tls.Config{
    RootCAs:    rootCertPool,
    MinVersion: tls.VersionTLS12,
    ServerName: "tidb.srgnqxji5bc.clusters.staging.tidb-cloud.com",
  })
  db, err := sql.Open("mysql", "root:<your_password>@tcp(tidb.srgnqxji5bc.clusters.staging.tidb-cloud.com:4000)/test?tls=tidb")
  if err != nil {
    log.Fatal("failed to connect database", err)
  }
  defer db.Close()

  var dbName string
  err = db.QueryRow("SELECT DATABASE();").Scan(&dbName)
  if err != nil {
    log.Fatal("failed to execute query", err)
  }
  fmt.Println(dbName)
}
```

パラメータの説明：

- TLS接続構成の`tls.Config`を登録して、TLSを有効にし、TiDB専用クラスタを検証します。
- `MinVersion: tls.VersionTLS12`を設定して、TLSプロトコルのバージョンを制限します。
- `ServerName: "<host>"`を設定して、TiDB専用のホスト名を検証します。
- 新しいTLS構成を登録したくない場合は、接続文字列に`tls=true`を設定するだけで構いません。