---
title: サンプルデータベースのインポート
summary: サンプルの自転車シェアリングデータベースをインストールします。
aliases: ['/docs/dev/import-example-data/', '/docs/dev/how-to/get-started/import-example-database/']
---

# サンプルデータベースのインポート

TiDBマニュアルで使用されている例は、[Capital Bikeshare](https://www.capitalbikeshare.com/system-data)の[Capital Bikeshare Data License Agreement](https://www.capitalbikeshare.com/data-license-agreement)に基づいて公開されているシステムデータを使用しています。

## すべてのデータファイルをダウンロードする

システムデータは、年ごとに整理された.zipファイル形式で[ダウンロードできます](https://s3.amazonaws.com/capitalbikeshare-data/index.html)。すべてのファイルをダウンロードし解凍するには、約3GBのディスク容量が必要です。以下のbashスクリプトを使用して、2010年から2017年までのすべてのファイルをダウンロードします。

```bash
mkdir -p bikeshare-data && cd bikeshare-data

curl -L --remote-name-all https://s3.amazonaws.com/capitalbikeshare-data/{2010..2017}-capitalbikeshare-tripdata.zip
unzip \*-tripdata.zip
```

## データをTiDBにロードする

次のスキーマを使用して、システムデータをTiDBにインポートできます。

```sql
CREATE DATABASE bikeshare;
USE bikeshare;

CREATE TABLE trips (
 trip_id bigint NOT NULL PRIMARY KEY AUTO_INCREMENT,
 duration integer not null,
 start_date datetime,
 end_date datetime,
 start_station_number integer,
 start_station varchar(255),
 end_station_number integer,
 end_station varchar(255),
 bike_number varchar(255),
 member_type varchar(255)
);
```

こちらの例に、`LOAD DATA`コマンドを使用して個別のファイルをインポートすることもできます。また、以下のbashループを使用してすべてのファイルをインポートすることもできます。

```sql
LOAD DATA LOCAL INFILE '2017Q1-capitalbikeshare-tripdata.csv' INTO TABLE trips
  FIELDS TERMINATED BY ',' ENCLOSED BY '"'
  LINES TERMINATED BY '\r\n'
  IGNORE 1 LINES
(duration, start_date, end_date, start_station_number, start_station,
end_station_number, end_station, bike_number, member_type);
```

### すべてのファイルをインポートする

> **注意:**
>
> MySQLクライアントを起動する際には、`--local-infile=1`オプションを使用してください。

すべての`*.csv`ファイルをTiDBにインポートするためのbashループは以下の通りです。

```bash
for FILE in *.csv; do
 echo "== $FILE =="
 mysql bikeshare --local-infile=1 -e "LOAD DATA LOCAL INFILE '${FILE}' INTO TABLE trips FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);"
done;
```