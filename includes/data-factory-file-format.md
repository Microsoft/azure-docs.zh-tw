## <a name="specifying-formats"></a>指定格式
Azure Data Factory 支援下列格式類型：

* [文字格式](#specifying-textformat)
* [JSON 格式](#specifying-jsonformat)
* [Avro 格式](#specifying-avroformat)
* [ORC 格式](#specifying-orcformat)
* [Parquet 格式](#specifying-parquetformat)

### <a name="specifying-textformat"></a>指定 TextFormat
如果您想要剖析文字檔，或以文字格式寫入資料，請將 `format``type` 屬性設定為 **TextFormat**。 您也可以在 `format` 區段中指定下列**選擇性**屬性。 關於如何設定，請參閱 [TextFormat 範例](#textformat-example)一節。

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| columnDelimiter |用來分隔檔案中的資料行的字元。 您可以考慮使用資料中不太可能存在的罕見不可列印字元：例如，指定 "\u0001" 代表標題開頭 (SOH)。 |只允許一個字元。 **預設值**是**逗號 (',')**。 <br/><br/>若要使用 Unicode 字元，請參閱 [Unicode 字元](https://en.wikipedia.org/wiki/List_of_Unicode_characters)以為它取得對應的程式碼。 |否 |
| rowDelimiter |用來分隔檔案中的資料列的字元。 |只允許一個字元。 **預設值**是下列任一個值：**["\r\n", "\r", "\n"]** (讀取時) 與 **"\r\n"** (寫入時)。 |否 |
| escapeChar |用來逸出輸入檔內容中的資料行分隔符號的特殊字元。 <br/><br/>您無法同時為資料表指定 escapeChar 和 quoteChar。 |只允許一個字元。 沒有預設值。 <br/><br/>例如，如果您以逗號 (',') 做為資料行分隔符號，但您想要在文字中使用逗號字元 (例如："Hello, world")，您可以定義 ‘$’ 做為逸出字元，並在來源中使用字串 "Hello$, world"。 |否 |
| quoteChar |用來引用字串值的字元。 引號字元內的資料行和資料列分隔符號會被視為字串值的一部分。 這個屬性同時適用於輸入和輸出資料集。<br/><br/>您無法同時為資料表指定 escapeChar 和 quoteChar。 |只允許一個字元。 沒有預設值。 <br/><br/>例如，如果您以逗號 (',') 做為資料行分隔符號，但您想要在文字中使用逗號字元 (例如：<Hello, world>)，您可以定義 " (雙引號) 做為引用字元，並在來源中使用字串 "Hello, world"。 |否 |
| nullValue |用來代表 null 值的一個或多個字元。 |一或多個字元。 **預設值**為 **"\N" 和 "NULL"** (讀取時) 及 **"\N"** (寫入時)。 |否 |
| encodingName |指定編碼名稱。 |有效的編碼名稱。 請參閱 [Encoding.EncodingName 屬性](https://msdn.microsoft.com/library/system.text.encoding.aspx)。 例如：windows-1250 或 shift_jis。 **預設值**為 **UTF-8**。 |否 |
| firstRowAsHeader |指定是否將第一個資料列視為標頭。 對於輸入資料集，Data Factory 會讀取第一個資料列做為標頭。 對於輸出資料集，Data Factory 會寫入第一個資料列做為標頭。 <br/><br/>相關範例案例請參閱[使用 `firstRowAsHeader` 和 `skipLineCount` 的案例](#scenarios-for-using-firstrowasheader-and-skiplinecount)。 |True<br/>**False (預設值)** |否 |
| skipLineCount |表示從輸入檔讀取資料時要略過的資料列數目。 如果指定 skipLineCount 和 firstRowAsHeader，則會先略過程式碼行，再從輸入檔讀取標頭資訊。 <br/><br/>相關範例案例請參閱[使用 `firstRowAsHeader` 和 `skipLineCount` 的案例](#scenarios-for-using-firstrowasheader-and-skiplinecount)。 |整數  |否 |
| treatEmptyAsNull |指定從輸入檔讀取資料時，是否將 null 或空字串視為 null 值。 |**True (預設值)**<br/>False |否 |

#### <a name="textformat-example"></a>TextFormat 範例
下列範例顯示 TextFormat 的一些格式屬性。

```json
"typeProperties":
{
    "folderPath": "mycontainer/myfolder",
    "fileName": "myblobname",
    "format":
    {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": ";",
        "quoteChar": "\"",
        "NullValue": "NaN",
        "firstRowAsHeader": true,
        "skipLineCount": 0,
        "treatEmptyAsNull": true
    }
},
```

若要使用 `escapeChar` 而不是`quoteChar`，請使用下列 escapeChar 取代含有 `quoteChar` 的那一行：

```json
"escapeChar": "$",
```

#### <a name="scenarios-for-using-firstrowasheader-and-skiplinecount"></a>使用 firstRowAsHeader 和 skipLineCount 的案例
* 您正從非檔案來源複製到文字檔，並想要加入標頭行，其中包含結構描述中繼資料 (例如︰SQL 結構描述)。 在此案例的輸出資料集，將 `firstRowAsHeader` 指定為 true。
* 您正從包含標頭行的文字檔複製到非檔案接收器，並想要刪除那一行。 在輸入資料集，將 `firstRowAsHeader` 指定為 true。
* 您正從文字檔複製，並想略過不包含資料或標頭資訊的開頭幾行。 指定 `skipLineCount` 以表示要略過的行數。 如果檔案其餘部分包含標頭行，您也可以指定 `firstRowAsHeader`。 如果 `skipLineCount` 和 `firstRowAsHeader` 都指定，則會先略過那幾行，再從輸入檔讀取標頭資訊

### <a name="specifying-jsonformat"></a>指定 JsonFormat
若要**將 JSON 檔案以現況匯入到/匯出自 Cosmos DB**，請參閱 Azure Cosmos DB 連接器中的[匯入/匯出 JSON 文件](../articles/data-factory/v1/data-factory-azure-documentdb-connector.md#importexport-json-documents)一節，以取得詳細資訊。

如果您想要剖析 JSON 檔案，或以 JSON 格式寫入資料，請將 `format``type` 屬性設定為 **JsonFormat**。 您也可以在 `format` 區段中指定下列**選擇性**屬性。 關於如何設定，請參閱 [JsonFormat 範例](#jsonformat-example)一節。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| filePattern |表示每個 JSON 檔案中儲存的資料模式。 允許的值為︰**setOfObjects** 和 **arrayOfObjects**。 **預設值**為 **setOfObjects**。 關於這些模式的詳細資訊，請參閱 [JSON 檔案模式](#json-file-patterns)一節。 |否 |
| jsonNodeReference | 如果您想要逐一查看陣列欄位內相同模式的物件並擷取資料，請指定該陣列的 JSON 路徑。 從 JSON 檔案複製資料時，才支援這個屬性。 | 否 |
| jsonPathDefinition | 指定 JSON 路徑運算式，以自訂資料行名稱來對應每個資料行 (開頭為小寫)。 從 JSON 檔案複製資料時，才支援這個屬性，您可以從物件或陣列中擷取資料。 <br/><br/> 如果是根物件下的欄位，請從根 $ 開始，如果是 `jsonNodeReference` 屬性所選陣列內的欄位，請從陣列元素開始。 關於如何設定，請參閱 [JsonFormat 範例](#jsonformat-example)一節。 | 否 |
| encodingName |指定編碼名稱。 如需有效編碼名稱的清單，請參閱： [Encoding.EncodingName](https://msdn.microsoft.com/library/system.text.encoding.aspx) 屬性。 例如：windows-1250 或 shift_jis。 **預設值**為 **UTF-8**。 |否 |
| nestingSeparator |用來分隔巢狀層級的字元。 預設值為 '.' (點)。 |否 |

#### <a name="json-file-patterns"></a>JSON 檔案模式

複製活動可以剖析的 JSON 檔案模式如下︰

- **類型 I：setOfObjects**

    每個檔案都會包含單一物件，或以行分隔/串連的多個物件。 在輸出資料集中選擇此選項時，複製活動會產生單一 JSON 檔案，每行一個物件 (以行分隔)。

    * **單一物件 JSON 範例**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        ```

    * **以行分隔的 JSON 範例**

        ```json
        {"time":"2015-04-29T07:12:20.9100000Z","callingimsi":"466920403025604","callingnum1":"678948008","callingnum2":"567834760","switch1":"China","switch2":"Germany"}
        {"time":"2015-04-29T07:13:21.0220000Z","callingimsi":"466922202613463","callingnum1":"123436380","callingnum2":"789037573","switch1":"US","switch2":"UK"}
        {"time":"2015-04-29T07:13:21.4370000Z","callingimsi":"466923101048691","callingnum1":"678901578","callingnum2":"345626404","switch1":"Germany","switch2":"UK"}
        ```

    * **串連的 JSON 範例**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        }
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
        ```

- **類型 II：arrayOfObjects**

    每個檔案都會包含物件的陣列。

    ```json
    [
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        },
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        },
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
    ]
    ```

#### <a name="jsonformat-example"></a>JsonFormat 範例

**案例 1︰從 JSON 檔案複製資料**

從 JSON 檔案複製資料時，請參閱以下兩個範例類型，並注意一般要點︰

**範例 1︰從物件和陣列擷取資料**

在此範例中，預計會有一個根 JSON 物件對應至表格式結果中的單一記錄。 如果您的 JSON 檔案含有下列內容：  

```json
{
    "id": "ed0e4960-d9c5-11e6-85dc-d7996816aad3",
    "context": {
        "device": {
            "type": "PC"
        },
        "custom": {
            "dimensions": [
                {
                    "TargetResourceType": "Microsoft.Compute/virtualMachines"
                },
                {
                    "ResourceManagmentProcessRunId": "827f8aaa-ab72-437c-ba48-d8917a7336a3"
                },
                {
                    "OccurrenceTime": "1/13/2017 11:24:37 AM"
                }
            ]
        }
    }
}
```
而且想要使用下列格式，將它複製到 Azure SQL 資料表，請同時從物件和陣列擷取資料︰

| id | deviceType | targetResourceType | resourceManagmentProcessRunId | occurrenceTime |
| --- | --- | --- | --- | --- |
| ed0e4960-d9c5-11e6-85dc-d7996816aad3 | PC | Microsoft.Compute/virtualMachines | 827f8aaa-ab72-437c-ba48-d8917a7336a3 | 1/13/2017 11:24:37 AM |

**JsonFormat** 類型的輸入資料集定義如下：(僅含相關元素的局部定義)。 具體而言：

- `structure` 區段定義自訂資料行名稱，以及轉換成表格式資料時對應的資料類型。 除非您需要對應資料行，否則這個區段是**選擇性**。 如需詳細資訊，請參閱[指定矩形資料集的結構定義](#specifying-structure-definition-for-rectangular-datasets)一節。
- `jsonPathDefinition` 指定每個資料行的 JSON 路徑，以指出從哪裡擷取資料。 若要從陣列複製資料，您可以使用 **array[x].property** 從第 x 個物件擷取指定屬性的值，也可以使用 **array[*].property** 從任何包含這類屬性的物件中尋找此值。

```json
"properties": {
    "structure": [
        {
            "name": "id",
            "type": "String"
        },
        {
            "name": "deviceType",
            "type": "String"
        },
        {
            "name": "targetResourceType",
            "type": "String"
        },
        {
            "name": "resourceManagmentProcessRunId",
            "type": "String"
        },
        {
            "name": "occurrenceTime",
            "type": "DateTime"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat",
            "filePattern": "setOfObjects",
            "jsonPathDefinition": {"id": "$.id", "deviceType": "$.context.device.type", "targetResourceType": "$.context.custom.dimensions[0].TargetResourceType", "resourceManagmentProcessRunId": "$.context.custom.dimensions[1].ResourceManagmentProcessRunId", "occurrenceTime": " $.context.custom.dimensions[2].OccurrenceTime"}      
        }
    }
}
```

**範例 2︰交叉套用陣列中具有相同模式的多個物件**

在此範例中，預計會將一個根 JSON 物件轉換成表格式結果中的多筆記錄。 如果您的 JSON 檔案含有下列內容：  

```json
{
    "ordernumber": "01",
    "orderdate": "20170122",
    "orderlines": [
        {
            "prod": "p1",
            "price": 23
        },
        {
            "prod": "p2",
            "price": 13
        },
        {
            "prod": "p3",
            "price": 231
        }
    ],
    "city": [ { "sanmateo": "No 1" } ]
}
```
您想要簡維陣列內的資料，將內容複製到下列格式的 Azure SQL 資料表，並與一般根資訊交叉聯結︰

| ordernumber | orderdate | order_pd | order_price | city |
| --- | --- | --- | --- | --- |
| 01 | 20170122 | P1 | 23 | [{"sanmateo":"No 1"}] |
| 01 | 20170122 | P2 | 13 | [{"sanmateo":"No 1"}] |
| 01 | 20170122 | P3 | 231 | [{"sanmateo":"No 1"}] |

**JsonFormat** 類型的輸入資料集定義如下：(僅含相關元素的局部定義)。 具體而言：

- `structure` 區段定義自訂資料行名稱，以及轉換成表格式資料時對應的資料類型。 除非您需要對應資料行，否則這個區段是**選擇性**。 如需詳細資訊，請參閱[指定矩形資料集的結構定義](#specifying-structure-definition-for-rectangular-datasets)一節。
- `jsonNodeReference` 表示逐一查看**陣列** orderlines 下相同模式的物件並擷取資料。
- `jsonPathDefinition` 指定每個資料行的 JSON 路徑，以指出從哪裡擷取資料。 在此範例中，"ordernumber"、"orderdate" 和 "city" 位於根物件下，JSON 路徑開頭為 "$."，而 "order_pd" 和 "order_price" 以衍生自陣列元素的路徑定義，不含 "$."。

```json
"properties": {
    "structure": [
        {
            "name": "ordernumber",
            "type": "String"
        },
        {
            "name": "orderdate",
            "type": "String"
        },
        {
            "name": "order_pd",
            "type": "String"
        },
        {
            "name": "order_price",
            "type": "Int64"
        },
        {
            "name": "city",
            "type": "String"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat",
            "filePattern": "setOfObjects",
            "jsonNodeReference": "$.orderlines",
            "jsonPathDefinition": {"ordernumber": "$.ordernumber", "orderdate": "$.orderdate", "order_pd": "prod", "order_price": "price", "city": " $.city"}         
        }
    }
}
```

**請注意下列幾點**：

* 如果 Data Factory 資料集中未定義 `structure`和 `jsonPathDefinition`，複製活動會偵測第一個物件的結構描述，並簡維整個物件。
* 如果 JSON 輸入具有陣列，依預設，複製活動會將整個陣列值轉換為字串。 您可以選擇使用 `jsonNodeReference` 及/或 `jsonPathDefinition` 從其中擷取資料，或不要在 `jsonPathDefinition` 中指定以略過它。
* 如果相同層級中有重複的名稱，複製活動會挑選最後一個。
* 屬性名稱會區分大小寫。 名稱相同但大小寫不同的兩個屬性會被視為兩個不同的屬性。

**案例 2︰將資料寫入 JSON 檔案**

如果您在 SQL Database 中有下列資料表︰

| id | order_date | order_price | order_by |
| --- | --- | --- | --- |
| 1 | 20170119 | 2000 | David |
| 2 | 20170120 | 3500 | Patrick |
| 3 | 20170121 | 4000 | Jason |

對於每一筆記錄，您預期寫入下列格式的 JSON 物件︰
```json
{
    "id": "1",
    "order": {
        "date": "20170119",
        "price": 2000,
        "customer": "David"
    }
}
```

**JsonFormat** 類型的輸出資料集定義如下：(僅含相關元素的局部定義)。 更具體來說，`structure` 區段在目的檔案中定義自訂屬性名稱，`nestingSeparator` (預設值是 ".") 用來識別名稱中的巢狀層。 除非您想要變更屬性名稱與來源資料行名稱之間的對照，或巢狀化某些屬性，否則這個區段是**選擇性**。

```json
"properties": {
    "structure": [
        {
            "name": "id",
            "type": "String"
        },
        {
            "name": "order.date",
            "type": "String"
        },
        {
            "name": "order.price",
            "type": "Int64"
        },
        {
            "name": "order.customer",
            "type": "String"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat"
        }
    }
}
```

### <a name="specifying-avroformat"></a>指定 AvroFormat
如果您想要剖析 Avro 檔案，或以 Avro 格式寫入資料，請將 `format``type` 屬性設定為 **AvroFormat**。 您不需要在 typeProperties 區段內的 Format 區段中指定任何屬性。 範例：

```json
"format":
{
    "type": "AvroFormat",
}
```

若要在 Hive 資料表中使用 Avro 格式，您可以參考 [Apache Hive 的教學課程](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe)。

請注意下列幾點：  

* 不支援[複雜資料類型](http://avro.apache.org/docs/current/spec.html#schema_complex) (記錄、列舉、陣列、對應、等位和固定)。

### <a name="specifying-orcformat"></a>指定 OrcFormat
如果您想要剖析 ORC 檔案，或以 ORC 格式寫入資料，請將 `format``type` 屬性設定為 **OrcFormat**。 您不需要在 typeProperties 區段內的 Format 區段中指定任何屬性。 範例：

```json
"format":
{
    "type": "OrcFormat"
}
```

> [!IMPORTANT]
> 如果您不會在內部部署與雲端資料存放區之間複製 ORC 檔案 **as-is** ，您需要在閘道機器上安裝 JRE 8 (Java 執行階段環境)。 64 位元閘道需要 64 位元 JRE，而 32 位元閘道需要 32 位元 JRE。 您可以從 [這裡](http://go.microsoft.com/fwlink/?LinkId=808605)找到這兩個版本。 請選擇適當的版本。
>
>

請注意下列幾點：

* 不支援複雜資料類型 (STRUCT、MAP、LIST、UNION)
* ORC 檔案有 3 種 [壓縮相關選項](http://hortonworks.com/blog/orcfile-in-hdp-2-better-compression-better-performance/)︰NONE、ZLIB、SNAPPY。 Data Factory 支援以這些壓縮格式的任一項從 ORC 檔案讀取資料。 它會使用中繼資料裡的壓縮轉碼器來讀取資料。 不過，寫入 ORC 檔案時，Data Factory 會選擇 ZLIB，這是 ORC 的預設值。 目前沒有任何選項可覆寫這個行為。

### <a name="specifying-parquetformat"></a>指定 ParquetFormat
如果您想要剖析 Parquet 檔案，或以 Parquet 格式寫入資料，請將 `format``type` 屬性設定為 **ParquetFormat**。 您不需要在 typeProperties 區段內的 Format 區段中指定任何屬性。 範例：

```json
"format":
{
    "type": "ParquetFormat"
}
```
> [!IMPORTANT]
> 如果您不是在內部部署與雲端資料存放區之間 **以原狀直接** 複製 Parquet 檔案，您需要在閘道機器上安裝 JRE 8 (Java 執行階段環境)。 64 位元閘道需要 64 位元 JRE，而 32 位元閘道需要 32 位元 JRE。 您可以從 [這裡](http://go.microsoft.com/fwlink/?LinkId=808605)找到這兩個版本。 請選擇適當的版本。
>
>

請注意下列幾點：

* 不支援複雜資料類型 (MAP、LIST)
* Parquet 檔案已有下列壓縮相關選項：NONE、SNAPPY、GZIP 和 LZO。 Data Factory 支援以這些壓縮格式的任一項從 ORC 檔案讀取資料。 它會使用中繼資料裡的壓縮轉碼器來讀取資料。 不過，寫入 Parquet 檔案時，Data Factory 會選擇 SNAPPY，這是 Parquet 格式的預設值。 目前沒有任何選項可覆寫這個行為。
