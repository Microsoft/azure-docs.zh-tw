## <a name="specifying-structure-definition-for-rectangular-datasets"></a>指定矩形資料集的結構定義
在資料集 JSON 中的結構區段 (“structure”) 是矩形資料表 (有資料列和資料行) 的**選擇性**區段，並包含該資料表的資料行集合。 結構區段將用於提供類型轉換的類型資訊或執行資料行對應。 下列各節更詳細說明這些功能。 

各資料行包含下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| name |資料行的名稱。 |是 |
| type |資料行的資料類型。 有關何時應指定類型資訊的詳細資訊，請參閱下文類型轉換的部份 |否 |
| culture |當有指定 type (類型) 時要使用的 .NET 型文化特性，且是 .NET 類型的 Datetime 或 Datetimeoffset。 預設值為 “en-us”。 |否 |
| format |當有指定 type (類型) 時要使用的格式字串，且是 .NET 類型的 Datetime 或 Datetimeoffset。 |否 |

下列範例顯示具有 userid、name、lastlogindate 三個資料行的資料表的結構區段 JSON。

```json
"structure": 
[
    { "name": "userid"},
    { "name": "name"},
    { "name": "lastlogindate"}
],
```

有關何時要包括 “structure” 資訊以及在 **結構** 區段中要包含哪些資訊，請遵循下列準則。

* **結構化的資料來源** (如 SQL Server、Oracle、Azure 資料表等來源)，只有當您想要進行資料行對應將特定來源資料行對應至接收中的特定資料行且其名稱不相同時，才應該指定 “structure” 區段 (詳細資訊請參閱下文資料行對應的部份)。 
  
    如上所述，“structure” 區段中的類型資訊是選擇性的。 針對結構化的來源，類型資訊已可使用做為資料存放區中的資料集定義的一部分，因此當您包含 “structure” 區段時不應包含類型資訊。
* **讀取的資料來源 (尤其是 Azure blob) 的結構描述**，您可以選擇儲存資料但不儲存任何結構描述或類型資訊。 針對這些類型的資料來源，在以下 2 種案例中應包含 “structure”：
  * 您想要進行對應。
  * 當資料集是「複製」活動中的來源時，您可以在 “structure” 中提供類型資訊，資料處理站轉換為原生類型以利接收時會使用此類型資訊。 如需詳細資訊請參閱 [移動資料進出 Azure Blob](../articles/data-factory/v1/data-factory-azure-blob-connector.md) 的文章。

### <a name="supported-net-based-types"></a>支援 .NET 型的類型
資料處理站支援下列符合 CLS 標準的 .NET 型類型，以利針對讀取的資料來源 (如 Azure blob) 的結構描述在 “structure” 中提供類型資訊。

* Int16
* Int32 
* Int64
* 單一
* 兩倍
* 十進位
* Byte[]
* Bool
* 字串 
* Guid
* Datetime
* Datetimeoffset
* Timespan 

針對 Datetime 和 Datetimeoffset，您可以選擇性地指定 “culture” 和 “format” 字串以幫助剖析您的自訂 Datetime 字串。 請參閱下方的類型轉換範例。

