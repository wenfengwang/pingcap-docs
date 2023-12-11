---
title: TiCDC ローデータのAvroに基づいたチェックサム検証
summary: TiCDCローデータのチェックサム検証の詳細な実装を紹介します。

# TiCDC ローデータのAvroに基づいたチェックサム検証

このドキュメントでは、Golangを使用してTiCDCによってKafkaに送信され、Avroプロトコルでエンコードされたデータを消費し、[シングル行データのチェックサム機能](/ticdc/ticdc-integrity-check.md)を使用してデータの検証を行う方法について説明します。

この例のソースコードは、[`avro-checksum-verification`](https://github.com/pingcap/tiflow/tree/master/examples/golang/avro-checksum-verification)ディレクトリで利用できます。 

このドキュメントの例では、[kafka-go](https://github.com/segmentio/kafka-go)を使用してシンプルなKafkaコンシューマプログラムを作成します。 このプログラムは指定されたトピックからデータを連続して読み取り、チェックサムを計算し、その値を検証します。

```go
＃省略
```

チェックサム値を計算するための主要なステップは、`getValueMapAndSchema()` および `CalculateAndVerifyChecksum()` です。次のセクションでは、これらの2つの関数の実装について説明します。

## データをデコードして対応するスキーマを取得

`getValueMapAndSchema()` メソッドはデータをデコードし、対応するスキーマを取得します。 このメソッドはデータとスキーマを `map[string]interface{}` のタイプとして返します。

```go
// dataは受信したkafkaメッセージのキーまたは値であり、urlはスキーマレジストリのURLです。
// この関数はデコードされた値および対応するスキーマをmapとして返します。
func getValueMapAndSchema(data []byte, url string) (map[string]interface{}, map[string]interface{}, error) {
    // 処理省略
}

// extractSchemaIDAndBinaryData 
func extractSchemaIDAndBinaryData(data []byte) (int, []byte, error) {
    // 処理省略
}

// GetSchemaはスキーマIDによってスキーマをスキーマレジストリから取得します。
// この関数はデータのエンコードとデコードに使用できるgoavro.Codec を返します。
func GetSchema(url string, schemaID int) (*goavro.Codec, error) {
    // 処理省略
}

type lookupResponse struct {
    Name     string `json:"name"`
    SchemaID int    `json:"id"`
    Schema   string `json:"schema"`
}
```

## チェックサム値の計算と検証

前のステップで取得した `valueMap` と `valueSchema` には、チェックサムの計算と検証に使用されるすべての要素が含まれています。

消費側でのチェックサム値の計算および検証プロセスは、次の手順を含みます：

1. 期待されるチェックサム値を取得します。
2. 各列を繰り返し処理し、列の値と対応するMySQLタイプに従ってバイト配列を生成し、チェックサム値を連続して更新します。
3. 前のステップで計算されたチェックサム値を受信したメッセージから取得したチェックサム値と比較します。 それらが同じでない場合、チェックサム検証に失敗し、データが破損している可能性があります。

サンプルコードは以下の通りです：

```go
func CalculateAndVerifyChecksum(valueMap, valueSchema map[string]interface{}) error {
    // fields変数にはデータ変更イベントごとの各列の型情報が格納されます。 列IDは、フィールドをソートするときに使用され、チェックサムが計算される順序と同じです。
    fields, ok := valueSchema["fields"].([]interface{})
    if !ok {
        return errors.New("schema fields should be a map")
    }
```
// 1. valueMapから期待されるチェックサム値を取得します。これは文字列としてエンコードされています。
// 期待されるチェックサム値が見つからない場合、TiCDCがデータを送信する際にチェックサム機能が有効にされていないことを意味します。この場合、この関数は直ちに返ります。
o, ok := valueMap["_tidb_row_level_checksum"]
if !ok {
    return nil
}
expected := o.(string)
if expected == "" {
    return nil
}

// expectedChecksumはTiCDCから渡された期待されるチェックサム値です。
expectedChecksum, err := strconv.ParseUint(expected, 10, 64)
if err != nil {
    return errors.Trace(err)
}

// 2. 各フィールドを反復処理し、チェックサム値を計算します。
var actualChecksum uint32
// bufは、チェックサム値を更新するたびに使用されるバイトスライスを格納します。
buf := make([]byte, 0)
for _, item := range fields {
    field, ok := item.(map[string]interface{})
    if !ok {
        return errors.New("schema field should be a map")
    }

    // チェックサムの計算にはtidbOpおよびその後の列は関与せず、これらはデータの消費を支援するためのものであり、実際のTiDB列データではありません。
    colName := field["name"].(string)
    if colName == "_tidb_op" {
        break
    }

    // holder変数は各列の型情報を格納します。
    var holder map[string]interface{}
    switch ty := field["type"].(type) {
    case []interface{}:
        for _, item := range ty {
            if m, ok := item.(map[string]interface{}); ok {
                holder = m["connect.parameters"].(map[string]interface{})
                break
            }
        }
    case map[string]interface{}:
        holder = ty["connect.parameters"].(map[string]interface{})
    default:
        log.Panic("type info is anything else", zap.Any("typeInfo", field["type"]))
    }
    tidbType := holder["tidb_type"].(string)

    mysqlType := mysqlTypeFromTiDBType(tidbType)

    // 各列の値をデコードされた値のマップから、各列の名前に応じて取得します。
    value, ok := valueMap[colName]
    if !ok {
        return errors.New("value not found")
    }
    value, err := getColumnValue(value, holder, mysqlType)
    if err != nil {
        return errors.Trace(err)
    }

    if len(buf) > 0 {
        buf = buf[:0]
    }

    // それぞれの列の値とmysqlTypeに応じてチェックサムを更新するためのバイトスライスを生成し、その後、チェックサム値を更新します。
    buf, err = buildChecksumBytes(buf, value, mysqlType)
    if err != nil {
        return errors.Trace(err)
    }
    actualChecksum = crc32.Update(actualChecksum, crc32.IEEETable, buf)
}

if uint64(actualChecksum) != expectedChecksum {
    log.Error("checksum mismatch",
        zap.Uint64("expected", expectedChecksum),
        zap.Uint64("actual", uint64(actualChecksum)))
    return errors.New("checksum mismatch")
}

log.Info("checksum verified", zap.Uint64("checksum", uint64(actualChecksum)))
return nil
}

func mysqlTypeFromTiDBType(tidbType string) byte {
    var result byte
    switch tidbType {
    case "INT", "INT UNSIGNED":
        result = mysql.TypeLong
    case "BIGINT", "BIGINT UNSIGNED":
        result = mysql.TypeLonglong
    case "FLOAT":
        result = mysql.TypeFloat
    case "DOUBLE":
        result = mysql.TypeDouble
    case "BIT":
        result = mysql.TypeBit
    case "DECIMAL":
        result = mysql.TypeNewDecimal
    case "TEXT":
        result = mysql.TypeVarchar
    case "BLOB":
        result = mysql.TypeLongBlob
    case "ENUM":
        result = mysql.TypeEnum
    case "SET":
        result = mysql.TypeSet
    case "JSON":
        result = mysql.TypeJSON
    case "DATE":
        result = mysql.TypeDate
    case "DATETIME":
        result = mysql.TypeDatetime
    case "TIMESTAMP":
        result = mysql.TypeTimestamp
    case "TIME":
        result = mysql.TypeDuration
    case "YEAR":
        result = mysql.TypeYear
    default:
        log.Panic("this should not happen, unknown TiDB type", zap.String("type", tidbType))
    }
    return result
}

// holderによって提供される型情報に応じて変換する必要があるため、値はインターフェース型です。
func getColumnValue(value interface{}, holder map[string]interface{}, mysqlType byte) (interface{}, error) {
    switch t := value.(type) {
    // nullableを持つ列はマップとしてエンコードされており、1つのキーバリューペアしかありません。キーは型であり、値は実際の値です。ここでは実際の値のみが関連します。
    case map[string]interface{}:
        for _, v := range t {
            value = v
        }
    }

    switch mysqlType {
    case mysql.TypeEnum:
        // Enumは文字列としてエンコードされ、ここではEnum定義に対応するint値に変換されます。
        allowed := strings.Split(holder["allowed"].(string), ",")
        switch t := value.(type) {
        case string:
            enum, err := types.ParseEnum(allowed, t, "")
            if err != nil {
                return nil, errors.Trace(err)
            }
            value = enum.Value
        case nil:
            value = nil
        }
    case mysql.TypeSet:
        // Setは文字列としてエンコードされ、ここではSet定義に対応するint値に変換されます。
        elems := strings.Split(holder["allowed"].(string), ",")
        switch t := value.(type) {
        case string:
            s, err := types.ParseSet(elems, t, "")
            if err != nil {
                return nil, errors.Trace(err)
            }
            value = s.Value
        case nil:
            value = nil
        }
    }
    return value, nil
}

// buildChecksumBytesはチェックサムを更新するために使用されるバイトスライスを生成します。参照：https://github.com/pingcap/tidb/blob/e3417913f58cdd5a136259b902bf177eaf3aa637/util/rowcodec/common.go#L308
func buildChecksumBytes(buf []byte, value interface{}, mysqlType byte) ([]byte, error) {
    if value == nil {
        return buf, nil
    }

    switch mysqlType {
    // TypeTiny、TypeShort、TypeInt32はint32としてエンコードされます。
    // TypeLongは符号付きの場合はint32としてエンコードされ、それ以外の場合はint64としてエンコードされます。
    // TypeLongLongは符号付きの場合はint64としてエンコードされ、それ以外の場合はuint64としてエンコードされます。
    // チェックサム機能が有効になっている場合、bigintUnsignedHandlingModeはstringに設定される必要があり、これはstringとしてエンコードされます。
    case mysql.TypeTiny, mysql.TypeShort, mysql.TypeLong, mysql.TypeLonglong, mysql.TypeInt24, mysql.TypeYear:
        switch a := value.(type) {
        case int32:
            buf = binary.LittleEndian.AppendUint64(buf, uint64(a))
        case uint32:
            buf = binary.LittleEndian.AppendUint64(buf, uint64(a))
        case int64:
            buf = binary.LittleEndian.AppendUint64(buf, uint64(a))
        case uint64:
            buf = binary.LittleEndian.AppendUint64(buf, a)
        case string:
            v, err := strconv.ParseUint(a, 10, 64)
            if err != nil {
                return nil, errors.Trace(err)
            }
            buf = binary.LittleEndian.AppendUint64(buf, v)
        default:
            log.Panic("unknown golang type for the integral value",
                zap.Any("value", value), zap.Any("mysqlType", mysqlType))
        }
    // float型はfloat64としてエンコードされ、double型はfloat64としてエンコードされます。
    case mysql.TypeFloat, mysql.TypeDouble:
        var v float64
        switch a := value.(type) {
        case float32:
            v = float64(a)
        case float64:
            v = a
        }
        if math.IsInf(v, 0) || math.IsNaN(v) {
            v = 0
        }
        buf = binary.LittleEndian.AppendUint64(buf, math.Float64bits(v))
    // 列挙型およびセットはuint64型としてエンコードされます。
    case mysql.TypeEnum, mysql.TypeSet:
        buf = binary.LittleEndian.AppendUint64(buf, value.(uint64))
    case mysql.TypeBit:
        // bit型は[]byteとしてエンコードされ、それをuint64に変換します。
        v, err := binaryLiteralToInt(value.([]byte))
        if err != nil {
            return nil, errors.Trace(err)
        }
        buf = binary.LittleEndian.AppendUint64(buf, v)
    // バイナリでないタイプは文字列としてエンコードされ、バイナリなタイプは[]byteとしてエンコードされます。
    case mysql.TypeVarchar, mysql.TypeVarString, mysql.TypeString, mysql.TypeTinyBlob, mysql.TypeMediumBlob, mysql.TypeLongBlob, mysql.TypeBlob:
        switch a := value.(type) {
        case string:
            buf = appendLengthValue(buf, []byte(a))
        case []byte:
            buf = appendLengthValue(buf, a)
        default:
            log.Panic("unknown golang type for the string value",
                zap.Any("value", value), zap.Any("mysqlType", mysqlType))
        }
```Go
    case mysql.TypeTimestamp, mysql.TypeDatetime, mysql.TypeDate, mysql.TypeDuration, mysql.TypeNewDate:
        v := value.(string)
        buf = appendLengthValue(buf, []byte(v))
    // チェックサム機能が有効になっている場合、decimalHandlingModeはstringに設定する必要があります。
    case mysql.TypeNewDecimal:
        buf = appendLengthValue(buf, []byte(value.(string)))
    case mysql.TypeJSON:
        buf = appendLengthValue(buf, []byte(value.(string)))
    // NullおよびGeometryは、チェックサムの計算に関与しません。
    case mysql.TypeNull, mysql.TypeGeometry:
    // 何もしない
    default:
        return buf, errors.New("チェックサムの計算に無効なタイプです")
    }
    return buf, nil
}

func appendLengthValue(buf []byte, val []byte) []byte {
    buf = binary.LittleEndian.AppendUint32(buf, uint32(len(val)))
    buf = append(buf, val...)
    return buf
}

// []byteをuint64に変換します。詳細はこちらを参照してください: https://github.com/pingcap/tidb/blob/e3417913f58cdd5a136259b902bf177eaf3aa637/types/binary_literal.go#L105
func binaryLiteralToInt(bytes []byte) (uint64, error) {
    bytes = trimLeadingZeroBytes(bytes)
    length := len(bytes)

    if length > 8 {
        log.Error("無効なビット値が見つかりました", zap.ByteString("value", bytes))
        return math.MaxUint64, errors.New("無効なビット値")
    }

    if length == 0 {
        return 0, nil
    }

    val := uint64(bytes[0])
    for i := 1; i < length; i++ {
        val = (val << 8) | uint64(bytes[i])
    }
    return val, nil
}

func trimLeadingZeroBytes(bytes []byte) []byte {
    if len(bytes) == 0 {
        return bytes
    }
    pos, posMax := 0, len(bytes)-1
    for ; pos < posMax; pos++ {
        if bytes[pos] != 0 {
            break
        }
    }
    return bytes[pos:]
}
```