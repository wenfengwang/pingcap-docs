---
title: キーワード
summary: キーワードと予約語
aliases: ['/docs/dev/keywords-and-reserved-words/','/docs/dev/reference/sql/language-structure/keywords-and-reserved-words/','/tidb/dev/keywords-and-reserved-words/']
---

# キーワード

この記事では、TiDBのキーワード、予約語と非予約語の違い、およびクエリのためのすべてのキーワードについてまとめています。

キーワードは、SQLステートメントで特別な意味を持つ単語のことであり、`SELECT`、`UPDATE`、`DELETE`などが該当します。そのうちのいくつかは直接識別子として使用できる**非予約キーワード**と呼ばれます。その他のいくつかは、識別子として使用する前に特別な扱いが必要であり、これらは**予約キーワード**と呼ばれます。ただし、特別な非予約キーワードの中には、引き続き特別な扱いが必要な場合があります。これらは予約キーワードとして扱うことをお勧めします。

予約キーワードを識別子として使用するには、バッククォート `` ` `` で囲む必要があります:

{{< copyable "sql" >}}

```sql
CREATE TABLE select (a INT);
```

```
ERROR 1105 (HY000): line 0 column 19 near " (a INT)" (total length 27)
```

{{< copyable "sql" >}}

```sql
CREATE TABLE `select` (a INT);
```

```
Query OK, 0 rows affected (0.09 sec)
```

非予約キーワードにはバッククォートは必要ありません。例えば `BEGIN` と `END` は、次の文で識別子として正常に使用できます:

{{< copyable "sql" >}}

```sql
CREATE TABLE `select` (BEGIN int, END int);
```

```
Query OK, 0 rows affected (0.09 sec)
```

特別な場合では、`.`デリミタと一緒に使用される予約キーワードにはバッククォートは不要です:

{{< copyable "sql" >}}

```sql
CREATE TABLE test.select (BEGIN int, END int);
```

```
Query OK, 0 rows affected (0.08 sec)
```

## キーワード一覧

以下の一覧は、TiDBのキーワードを示しています。予約キーワードは `(R)` でマークされます。[ウィンドウ関数](/functions-and-operators/window-functions.md)用の予約キーワードは `(R-Window)` でマークされます。バッククォート `` ` `` でエスケープする必要のある特別な非予約キーワードは `(S)` でマークされます。

<TabsPanel letters="ABCDEFGHIJKLMNOPQRSTUVWXYZ" />

<a id="A" class="letter" href="#A">A</a>

- ACCOUNT
- ACTION
- ADD (R)
- ADMIN (R)
- ADVISE
- AFTER
- AGAINST
- AGO
- ALGORITHM
- ALL (R)
- ALTER (R)
- ALWAYS
- ANALYZE (R)
- AND (R)
- ANY
- ARRAY (R)
- AS (R)
- ASC (R)
- ASCII
- ATTRIBUTE
- ATTRIBUTES
- AUTO_ID_CACHE
- AUTO_INCREMENT
- AUTO_RANDOM
- AUTO_RANDOM_BASE
- AVG
- AVG_ROW_LENGTH

<a id="B" class="letter" href="#B">B</a>

- BACKEND
- BACKUP
- BACKUPS
- BEGIN
- BERNOULLI
- BETWEEN (R)
- BIGINT (R)
- BINARY (R)
- BINDING
- BINDINGS
- BINDING_CACHE
- BINLOG
- BIT
- BLOB (R)
- BLOCK
- BOOL
- BOOLEAN
- BOTH (R)
- BTREE
- BUCKETS (R)
- BUILTINS (R)
- BY (R)
- BYTE

<a id="C" class="letter" href="#C">C</a>

- CACHE
- CALIBRATE
- CALL (R)
- CANCEL (R)
- CAPTURE
- CASCADE (R)
- CASCADED
- CASE (R)
- CAUSAL
- CHAIN
- CHANGE (R)
- CHAR (R)
- CHARACTER (R)
- CHARSET
- CHECK (R)
- CHECKPOINT
- CHECKSUM
- CIPHER
- CLEANUP
- CLIENT
- CLIENT_ERRORS_SUMMARY
- CLOSE
- CLUSTER
- CLUSTERED
- CMSKETCH (R)
- COALESCE
- COLLATE (R)
- COLLATION
- COLUMN (R)
- COLUMN_FORMAT
- COLUMNS
- COMMENT
- COMMIT
- COMMITTED
- COMPACT
- COMPRESSED
- COMPRESSION
- CONCURRENCY
- CONFIG
- CONNECTION
- CONSISTENCY
- CONSISTENT
- CONSTRAINT (R)
- CONTEXT
- CONTINUE (R)
- CONVERT (R)
- CPU
- CREATE (R)
- CROSS (R)
- CSV_BACKSLASH_ESCAPE
- CSV_DELIMITER
- CSV_HEADER
- CSV_NOT_NULL
- CSV_NULL
- CSV_SEPARATOR
- CSV_TRIM_LAST_SEPARATORS
- CUME_DIST (R-Window)
- CURRENT
- CURRENT_DATE (R)
- CURRENT_ROLE (R)
- CURRENT_TIME (R)
- CURRENT_TIMESTAMP (R)
- CURRENT_USER (R)
- CURSOR (R)
- CYCLE

<a id="D" class="letter" href="#D">D</a>

- DATA
- DATABASE (R)
- DATABASES (R)
- DATE
- DATETIME
- DAY
- DAY_HOUR (R)
- DAY_MICROSECOND (R)
- DAY_MINUTE (R)
- DAY_SECOND (R)
- DDL (R)
- DEALLOCATE
- DECIMAL (R)
- DECLARE
- DEFAULT (R)
- DEFINER
- DELAY_KEY_WRITE
- DELAYED (R)
- DELETE (R)
- DENSE_RANK (R-Window)
- DEPTH (R)
- DESC (R)
- DESCRIBE (R)
- DIGEST
- DIRECTORY
- DISABLE
- DISABLED
- DISCARD
- DISK
- DISTINCT (R)
- DISTINCTROW (R)
- DIV (R)
- DO
- DOUBLE (R)
- DRAINER (R)
- DROP (R)
- DUAL (R)
- DUPLICATE
- DYNAMIC

<a id="E" class="letter" href="#E">E</a>

- ELSE (R)
- ELSEIF (R)
- ENABLE
- ENABLED
- ENCLOSED (R)
- ENCRYPTION
- END
- ENFORCED
- ENGINE
- ENGINES
- ENUM
- ERROR
- ERRORS
- ESCAPE
- ESCAPED (R)
- EVENT
- EVENTS
- EVOLVE
- EXCEPT (R)
- EXCHANGE
- EXCLUSIVE
- EXECUTE
- EXISTS (R)
- EXIT (R)
- EXPANSION
- EXPIRE
- EXPLAIN (R)
- EXTENDED

<a id="F" class="letter" href="#F">F</a>

- FAILED_LOGIN_ATTEMPTS
- FALSE (R)
- FAULTS
- FETCH (R)
- FIELDS
- FILE
- FIRST
- FIRST_VALUE (R-Window)
- FIXED
- FLOAT (R)
- FLOAT4 (R)
- FLOAT8 (R)
- FLUSH
- FOLLOWING
- FOR (R)
- FORCE (R)
- FOREIGN (R)
- FORMAT
- FOUND
- FROM (R)
- FULL
- FULLTEXT (R)
- FUNCTION

<a id="G" class="letter" href="#G">G</a>

- GENERAL
- GENERATED (R)
- GLOBAL
- GRANT (R)
- GRANTS
- GROUP (R)
- GROUPS (R-Window)

<a id="H" class="letter" href="#H">H</a>

- HANDLER
- HASH
- HAVING (R)
- HELP
- HIGH_PRIORITY (R)
- HISTOGRAM
- HISTORY
- HOSTS
- HOUR
- HOUR_MICROSECOND (R)
- HOUR_MINUTE (R)
- HOUR_SECOND (R)
- HYPO

<a id="I" class="letter" href="#I">I</a>

- IDENTIFIED
- IF (R)
- IGNORE (R)
- ILIKE (R)
- IMPORT
- IMPORTS
- IN (R)
- INCREMENT
- INCREMENTAL
- INDEX (R)
- INDEXES
- INFILE (R)
- INNER (R)
- INOUT (R)
- INSERT (R)
- INSERT_METHOD
- INSTANCE
- INT (R)
- INT1 (R)
- INT2 (R)
- INT3 (R)
- INT4 (R)
- INT8 (R)
- INTEGER (R)
- INTERSECT (R)
- INTERVAL (R)
- INTO (R)
- INVISIBLE
- INVOKER
- IO
- IPC
- IS (R)
- ISOLATION
- ISSUER
- ITERATE (R)

<a id="J" class="letter" href="#J">J</a>

- JOB (R)
- JOBS (R)
- JOIN (R)
- JSON

<a id="K" class="letter" href="#K">K</a>

- KEY (R)
- KEYS (R)
- KEY_BLOCK_SIZE
- KILL (R)

<a id="L" class="letter" href="#L">L</a>

- LABELS
- LAG (R-Window)
- LANGUAGE
- LAST
- LAST_BACKUP
- LAST_VALUE (R-Window)
- LASTVAL
- LEAD (R-Window)
- LEADING (R)
- LEAVE (R)
- LEFT (R)
- LESS
- LEVEL
- LIKE (R)
- LIMIT (R)
- LINEAR (R)
- LINES (R)
- LIST
- LOAD (R)
- LOCAL
- LOCALTIME (R)
- LOCALTIMESTAMP (R)
- LOCATION
- LOCK (R)
- LOCKED
- LOGS
- LONG (R)
- LONGBLOB (R)
- LONGTEXT (R)
- LOW_PRIORITY (R)

<a id="M" class="letter" href="#M">M</a>

- MASTER
- MATCH (R)
- MAXVALUE (R)
- MAX_CONNECTIONS_PER_HOUR
- MAX_IDXNUM
- MAX_MINUTES
- MAX_QUERIES_PER_HOUR
- MAX_ROWS
- MAX_UPDATES_PER_HOUR
- MAX_USER_CONNECTIONS
- MB
- MEDIUMBLOB (R)
- MEDIUMINT (R)
- MEDIUMTEXT (R)
- MEMBER
- MEMORY
- MERGE
- MICROSECOND
- MIDDLEINT (R)
- MINUTE
- MINUTE_MICROSECOND (R)
- MINUTE_SECOND (R)
- MINVALUE
- MIN_ROWS
- MOD (R)
- MODE
- MODIFY
- MONTH

<a id="N" class="letter" href="#N">N</a>

- NAMES
- NATIONAL
- NATURAL (R)
- NCHAR
- NEVER
- NEXT
- NEXTVAL
- NO
- NOCACHE
- NOCYCLE
- NODEGROUP
- NODE_ID (R)
- NODE_STATE (R)
- NOMAXVALUE
- NOMINVALUE
- NONCLUSTERED
- NONE
- NOT (R)
- NOWAIT
- NO_WRITE_TO_BINLOG (R)
- NTH_VALUE (R-Window)
- NTILE (R-Window)
- NULL (R)
- NULLS
- NUMERIC (R)
- NVARCHAR

<a id="O" class="letter" href="#O">O</a>

- OF (R)
- OFF
- OFFSET
- OLTP_READ_ONLY
- OLTP_READ_WRITE
- OLTP_WRITE_ONLY
- ON (R)
- ON_DUPLICATE
- ONLINE
- ONLY
- OPEN
- OPTIMISTIC (R)
- OPTIMIZE (R)
- OPTION (R)
- OPTIONAL
- OPTIONALLY (R)
- OR (R)
- ORDER (R)
- OUT (R)
- OUTER (R)
- OUTFILE (R)
- OVER (R-Window)

<a id="P" class="letter" href="#P">P</a>

- PACK_KEYS
- PAGE
- PARSER
- PARTIAL
- PARTITION (R)
- PARTITIONING
- PARTITIONS
- PASSWORD
- PASSWORD_LOCK_TIME
- PAUSE
- PERCENT
- PERCENT_RANK (R-Window)
- PER_DB
- PER_TABLE
- PESSIMISTIC (R)
- PLACEMENT (S)
- PLUGINS
- POINT
- POLICY
- PRECEDING
- PRECISION (R)
- PREPARE
- PRESERVE
- PRE_SPLIT_REGIONS
- PRIMARY (R)
- PRIVILEGES
- PROCEDURE (R)
- PROCESS
- PROCESSLIST
- PROFILE
- PROFILES
- PROXY
- PUMP (R)
- PURGE

<a id="Q" class="letter" href="#Q">Q</a>

- QUARTER
- QUERIES
- QUERY
- QUICK

<a id="R" class="letter" href="#R">R</a>

- RANGE (R)
- RANK (R-Window)
- RATE_LIMIT
- READ (R)
- REAL (R)
- REBUILD
- RECOVER
- RECURSIVE (R)
- REDUNDANT
- REFERENCES (R)
- REGEXP (R)
- REGION (R)
- REGIONS (R)
- RELEASE (R)
- RELOAD
- REMOVE
- RENAME (R)
- REORGANIZE
- REPAIR
- REPEAT (R)
- REPEATABLE
- REPLACE (R)
- REPLICA
- REPLICAS
- REPLICATION
- REQUIRE (R)
- REQUIRED
- RESOURCE
- RESPECT
- RESTART
- RESTORE
- RESTORES
- RESTRICT (R)
- RESUME
- REUSE
- REVERSE
- REVOKE (R)
- RIGHT (R)
- RLIKE (R)
- ROLE
- ROLLBACK
- ROLLUP
- ROUTINE
- ROW (R)
- ROW_COUNT
- ROW_FORMAT
- ROW_NUMBER (R-Window)
- ROWS (R-Window)
- RTREE

<a id="S" class="letter" href="#S">S</a>

- SAMPLES (R)
- SAN
- SAVEPOINT
- SECOND
- SECOND_MICROSECOND (R)
- SECONDARY_ENGINE
- SECONDARY_LOAD
- SECONDARY_UNLOAD
- SECURITY
- SELECT (R)
- SEND_CREDENTIALS_TO_TIKV
- SEPARATOR
- SEQUENCE
- SERIAL
- SERIALIZABLE
- SESSION
- SET (R)
- SETVAL
- SHARD_ROW_ID_BITS
- SHARE
- SHARED
- SHOW (R)
- SHUTDOWN
- SIGNED
- SIMPLE
- SKIP
- SKIP_SCHEMA_FILES
- SLAVE
- SLOW
- SMALLINT (R)
- SNAPSHOT
- SOME
- SOURCE
- SPATIAL (R)
- SPLIT (R)
- SQL (R)
- SQL_BIG_RESULT (R)
- SQL_BUFFER_RESULT
- SQL_CACHE
- SQL_CALC_FOUND_ROWS (R)
- SQL_NO_CACHE
- SQL_SMALL_RESULT (R)
- SQL_TSI_DAY
- SQL_TSI_HOUR
- SQL_TSI_MINUTE
- SQL_TSI_MONTH
- SQL_TSI_QUARTER
- SQL_TSI_SECOND
- SQL_TSI_WEEK
- SQL_TSI_YEAR
- SQLEXCEPTION (R)
- SQLSTATE (R)
- SQLWARNING (R)
- SSL (R)
- START
- STARTING (R)
- STATS (R)
- STATS_AUTO_RECALC
- STATS_BUCKETS (R)
- STATS_COL_CHOICE
- STATS_COL_LIST
- STATS_EXTENDED (R)
- STATS_HEALTHY (R)
- STATS_HISTOGRAMS (R)
- STATS_META (R)
- STATS_OPTIONS
- STATS_PERSISTENT
- STATS_SAMPLE_PAGES
- STATS_SAMPLE_RATE
- STATUS
- STORAGE
- STORED (R)
- STRAIGHT_JOIN (R)
- STRICT_FORMAT
- SUBJECT
- SUBPARTITION
- SUBPARTITIONS
- SUPER
- SWAPS
- SWITCHES
- SYSTEM
- SYSTEM_TIME

<a id="T" class="letter" href="#T">T</a>

- TABLE (R)
- TABLES
- TABLESAMPLE (R)
- TABLESPACE
- TABLE_CHECKSUM
- TEMPORARY
- TEMPTABLE
- TERMINATED (R)
- TEXT
- THAN
- THEN (R)
- TIDB (R)
- TiDB_CURRENT_TSO (R)
- TIFLASH (R)
- TIKV_IMPORTER
- TIME
- TIMESTAMP
- TINYBLOB (R)
- TINYINT (R)
- TINYTEXT (R)
- TO (R)
- TOKEN_ISSUER
- TOPN (R)
- TPCC
- TPCH_10
- TRACE
- TRADITIONAL
- TRAILING (R)
- TRANSACTION
- TRIGGER (R)
- TRIGGERS
- TRUE (R)
- TRUNCATE
- TTL
- TTL_ENABLE
- TTL_JOB_INTERVAL
- TYPE

<a id="U" class="letter" href="#U">U</a>

- UNBOUNDED
- UNCOMMITTED
- UNDEFINED
- UNICODE
- UNION (R)
- UNIQUE (R)
- UNKNOWN
- UNLOCK (R)
- UNSIGNED (R)
- UNTIL (R)
- UPDATE (R)
- USAGE (R)
- USE (R)
- USER
- USING (R)
- UTC_DATE (R)
- UTC_TIME (R)
- UTC_TIMESTAMP (R)

<a id="V" class="letter" href="#V">V</a>

- VALIDATION
- VALUE
- VALUES (R)
- VARBINARY (R)
- VARCHAR (R)
- VARCHARACTER (R)
- VARIABLES
- VARYING (R)
- VIEW
- VIRTUAL (R)
- VISIBLE

<a id="W" class="letter" href="#W">W</a>

- WAIT
- WARNINGS
- WEEK
- WEIGHT_STRING
- WHEN (R)
- WHERE (R)
- WHILE (R)
- WIDTH (R)
- WINDOW (R-Window)
- WITH (R)
- WITHOUT
- WORKLOAD
- WRITE (R)

<a id="X" class="letter" href="#X">X</a>

- X509
- XOR (R)

<a id="Y" class="letter" href="#Y">Y</a>

- YEAR
- YEAR_MONTH (R)

<a id="Z" class="letter" href="#Z">Z</a>

- ZEROFILL (R)