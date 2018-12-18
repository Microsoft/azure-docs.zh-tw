---
title: Azure Cosmos DB 編製索引原則 | Microsoft Docs
description: 了解 Azure Cosmos DB 中編製索引的運作方式。 了解如何設定編製索引原則，以自動編製索引並追求更高的效能。
keywords: 編製索引運作方式, 自動編製索引, 為資料庫編製索引
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: rafats
ms.openlocfilehash: fea3455b31ff2ea7119fa4146aa84f855a3b6e35
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44054667"
---
# <a name="how-does-azure-cosmos-db-index-data"></a>Azure Cosmos DB 如何為資料編製索引？

根據預設，會為所有 Azure Cosmos DB 資料編製索引。 儘管許多客戶都樂意讓 Azure Cosmos DB 自動處理所有編製索引層面，但在 Azure Cosmos DB 中，您也可以在建立時為集合指定自訂的「編製索引原則」。 Azure Cosmos DB 中的編制索引原則比其他資料庫平台中提供的次要索引更有彈性，功能更強大。 在 Azure Cosmos DB 中，您可以設計和自訂索引的圖形，卻不必犧牲結構描述彈性。 

若要了解編製索引如何在 Azure Cosmos DB 內運作，您必須了解透過管理編製索引原則，您可以在索引儲存空間負荷、寫入和查詢輸送量，以及查詢一致性之間進行細微的取捨。  

在下列影片中，Azure Cosmos DB 程式管理員 Andrew Liu 會示範 Azure Cosmos DB 自動編製索引的功能，以及如何在 Azure Cosmos DB 容器上微調和設定編製索引原則。 

>[!VIDEO https://www.youtube.com/embed/uFu2D-GscG0]

在本文中，我們將深入探討 Azure Cosmos DB 編製索引原則、自訂編製索引原則的方式及相關聯的取捨。 

閱讀本文後，您將能夠回答下列問題：

* 如何覆寫要在索引編製中包含或排除的屬性？
* 我要如何設定最終更新的索引？
* 如何設定編製索引來執行 Order By 或範圍查詢？
* 如何變更集合的索引編製原則？
* 如何比較不同索引編製原則的儲存空間和效能？

## <a id="Indexing"></a> Cosmos DB 索引編製

資料庫索引的目的是要在使用最少資源 (例如 CPU、輸入/輸出) 的情況下，以各種方式提供查詢，同時兼顧良好的輸送量和低延遲。 通常，選擇適用於資料庫查詢的正確索引需要經過相當的規劃和實驗。 此方式對資料不符合嚴謹結構描述的無結構描述資料庫而言是種挑戰，而且發展十分迅速。 

因此，我們在設計 Cosmos DB 索引子系統時，設定了下列目標：

* 在不需要結構描述的情況下，對文件編製索引：索引子系統不需要任何結構描述資訊，或提出任何文件結構描述的相關假設。  

* 支援有效率、豐富階層式及關聯式查詢：索引可有效率地支援 Cosmos DB 查詢語言，包括支援階層式和關聯式投射。  

* 支援在面對持續大量寫入時進行一致查詢：在面對持續大量寫入時，針對具有一致查詢的高寫入輸送量工作負載，會以累加、有效率及線上方式更新索引。 一致的索引更新對於提供一致性層級的查詢十分重要，在此層級中是由使用者設定文件服務。  

* 支援多重租用：由於是使用保留型模型進行跨租用戶的資源控管，因此會在為每一複本配置的系統資源 (CPU、記憶體，以及每秒的輸入/輸出作業) 預算內執行索引更新。  

* 儲存體效率：基於成本效益，索引的磁碟儲存體額外負荷有所限制且可預測。 這十分重要，因為 Cosmos DB 允許開發人員在索引額外負荷與查詢效能之間做出成本取捨。  

## 自訂集合的編製索引原則<a id="CustomizingIndexingPolicy"></a>   
您可以透過覆寫 Azure Cosmos DB 集合上的預設編製索引原則，來自訂在儲存空間、寫入/查詢效能及查詢一致性之間的取捨。 您可以設定下列層面：

* **包含或排除文件與索引的路徑**。 當您在集合中插入或取代文件時，您可以在索引中排除或包含特定文件。 您也可以在索引包含的文件中包含或排除要跨文件編制索引的特定 JSON 屬性，也稱為「路徑」。 路徑包含萬用字元模式。
* **設定各種索引類型**。 針對每個包含的路徑，您可以指定路徑需要的集合索引類型。 您可以依據路徑的資料、預期的查詢工作負載及數值/字串「精確度」來指定索引的類型。
* **設定索引更新模式**。 Azure Cosmos DB 支援三種編制索引模式：「一致」、「延遲」和「無」。 您可以透過 Azure Cosmos DB 集合上的編製索引原則來設定編制索引模式。 

下列 Microsoft .NET 程式碼片段示範如何在集合建立時設定自訂編制索引原則。 在此範例中，我們會以最大精確度來為字串和數字設定具有範圍索引的原則。 您可使用此原則對字串執行 ORDER BY 查詢。

    DocumentCollection collection = new DocumentCollection { Id = "myCollection" };

    collection.IndexingPolicy = new IndexingPolicy(new RangeIndex(DataType.String) { Precision = -1 });
    collection.IndexingPolicy.IndexingMode = IndexingMode.Consistent;

    await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), collection);   


> [!NOTE]
> 編製索引原則的 JSON 結構描述已隨著 2015-06-03 版的 REST API 發行而變更。 該版本中，編制索引原則的 JSON 結構描述支援針對字串的範圍索引。 .NET SDK 1.2.0 及 Java、Python 和 Node.js SDKs 1.1.0 支援新的原則結構描述。 舊版的 SDK 則使用 2015-04-08 版的 REST API。 它們支援先前的編制索引原則結構描述。
> 
> 根據預設，Azure Cosmos DB 會使用雜湊索引為文件內的所有字串屬性一致地編製索引。 它會使用範圍索引為文件內的所有數值屬性一致地編製索引。  
> 
> 

### <a name="customize-the-indexing-policy-in-the-portal"></a>自訂入口網站中的編製索引原則

您可以變更 Azure 入口網站中的集合編製索引原則： 

1. 在入口網站中，移至您的 Azure Cosmos DB 帳戶，然後選取您的集合。 
2. 在左側導覽功能表中，選取 [設定]，然後選取 [編製索引原則]。 
3. 在 [編製索引原則] 底下，變更您的編製索引原則，然後選取 [確定]。 

### 資料庫編製索引模式<a id="indexing-modes"></a>  
Azure Cosmos DB 支援三個編製索引模式，這些模式可以透過 Azure Cosmos DB 集合的編製索引原則來設定：「一致」、「延遲」和「無」。

**一致**：如果 Azure Cosmos DB 集合的原則為「一致」，特定 Azure Cosmos DB 集合上的查詢會依照與針對讀數所指定的相同一致性層級 (強式、限定過期、工作階段或最終) 進行。 索引會在文件更新 (在 Azure Cosmos DB 集合中插入、取代、更新和刪除文件) 時同步更新。

一致的編製索引支援一致的查詢，但代價可能是減少寫入輸送量。 這指的是減少需要編製索引的唯一路徑以及「一致性層級」的功能。 一致的索引編製模式是針對「快速寫入、立即查詢」工作負載而設計。

**延遲**：索引會在 Azure Cosmos DB 集合靜止 (亦即，未完整利用集合的輸送量容量來處理使用者要求的時候) 時，以非同步方式更新。  請注意，因為資料擷取與編製索引緩慢，因此可能會有不一致的結果。 這表示，指定時間的 COUNT 查詢或特定查詢結果可能不會一致或可重複。 

對於擷取的資料，索引通常處於追補模式。 使用「延遲」編製索引時，存留時間 (TTL) 變更會導致索引被卸除並重新建立。 這會使一段時間內的 COUNT 和查詢結果不一致。 大部分的 Azure Cosmos DB 帳戶都應使用「一致」編制索引模式。

**無**：含有「無」索引模式的集合沒有任何與其相關聯的索引。 如果將 Azure Cosmos DB 做為索引鍵值儲存體，且只能依據文件的 ID 屬性來存取它們，常會使用此選項。 

> [!NOTE]
> 將編製索引原則設定為「無」時，卸除任何現有的索引會有副作用。 如果您的存取模式只需要「識別碼」或自我連結，請使用此選項。
> 
> 

下表顯示根據針對集合設定的索引編製模式 (「一致」和「延遲」)，以及針對查詢要求指定的一致性層級，顯示查詢的一致性。 這適用於使用任何介面 (REST API、SDK)，或從預存程序和觸發程序內進行的查詢。 

|一致性|編制索引模式︰一致|編制索引模式：延遲|
|---|---|---|
|強式|強式|最終|
|界限-陳舊|界限-陳舊|最終|
|工作階段|工作階段|最終|
|最終|最終|最終|

Azure Cosmos DB 會針對在集合上所進行、且編製索引模式為「無」的查詢傳回錯誤。 只要在 REST API 中使用明確的 **x-ms-documentdb-enable-scan** 標頭，或使用 .NET SDK 利用 **EnableScanInQuery** 要求選項，仍然可以將查詢執行為掃描。 部分查詢功能 (例如 ORDER BY) 並不支援使用 **EnableScanInQuery**執行為掃描。

下表根據編製索引模式 (「一致」、「延遲」和「無」)，顯示指定 **EnableScanInQuery** 時查詢的一致性。

|一致性|索引模式︰一致|索引模式：緩慢|索引模式：無|
|---|---|---|---|
|強式|強式|最終|強式|
|界限-陳舊|界限-陳舊|最終|界限-陳舊|
|工作階段|工作階段|最終|工作階段|
|最終|最終|最終|最終|

下列程式碼範例示範如何在插入所有文件時，搭配使用 .NET SDK 與「一致」編製索引來建立 Azure Cosmos DB 集合。

     // Default collection creates a Hash index for all string fields and a Range index for all numeric    
     // fields. Hash indexes are compact and offer efficient performance for equality queries.

     var collection = new DocumentCollection { Id ="defaultCollection" };

     collection.IndexingPolicy.IndexingMode = IndexingMode.Consistent;

     collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("mydb"), collection);


### <a name="index-paths"></a>索引路徑
Azure Cosmos DB 將 JSON 文件和索引模型化為樹狀結構。 您可以為樹狀結構中的路徑微調原則。 在文件中，您可以選擇編制索引所包含或排除的路徑。 針對事先知道查詢模式的情況，這將可改善寫入效能並減少索引儲存空間。

索引路徑的開頭為根 (/)，且通常結尾為 ? 萬用字元運算子。 這代表有多個可能的首碼值。 例如，若要為 SELECT * FROM Families F WHERE F.familyName = "Andersen" 提供服務，您必須在集合的索引原則中包含 /familyName/? 的索引路徑。

索引路徑也可以使用 \* 萬用字元運算子來指定路徑首碼底下的遞迴行為。 例如，使用 /payload/* 可將 payload 屬性下的所有項目自編製索引作業中排除。

以下是指定索引路徑的常見模式：

| Path                | 描述/使用案例                                                                                                                                                                                                                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| /                   | 集合的預設路徑。 遞迴並套用至整個文件樹狀結構。                                                                                                                                                                                                                                   |
| /prop/?             | 為類似下列的查詢 (類型分別為雜湊或範圍) 提供服務而必要的索引路徑：<br><br>SELECT FROM collection c WHERE c.prop = "value"<br><br>SELECT FROM collection c WHERE c.prop > 5<br><br>SELECT FROM collection c ORDER BY c.prop                                                                       |
| /"prop"/*             | 指定之標籤底下所有路徑的索引路徑。 使用下列查詢<br><br>SELECT FROM collection c WHERE c.prop = "value"<br><br>SELECT FROM collection c WHERE c.prop.subprop > 5<br><br>SELECT FROM collection c WHERE c.prop.subprop.nextprop = "value"<br><br>SELECT FROM collection c ORDER BY c.prop         |
| /props/[]/?         | 為反覆項目及針對 ["a"、"b"、"c"] 等純量陣列進行之聯結查詢提供服務所需的索引路徑：<br><br>SELECT tag FROM tag IN collection.props WHERE tag = "value"<br><br>SELECT tag FROM collection c JOIN tag IN c.props WHERE tag > 5                                                                         |
| /props/[]/subprop/? | 為反覆項目及針對 [{subprop: "a"}, {subprop: "b"}] 等物件陣列進行之聯結查詢提供服務所需的索引路徑：<br><br>SELECT tag FROM tag IN collection.props WHERE tag.subprop = "value"<br><br>SELECT tag FROM collection c JOIN tag IN c.props WHERE tag.subprop = "value"                                  |
| /prop/subprop/?     | 為查詢 (類型分別為雜湊或範圍) 提供服務而必要的索引路徑：<br><br>SELECT FROM collection c WHERE c.prop.subprop = "value"<br><br>SELECT FROM collection c WHERE c.prop.subprop > 5                                                                                                                    |

> [!NOTE]
> 設定自訂的索引路徑時，您必須為由特殊路徑 "/*" 所表示的整份文件樹狀目錄指定預設的編制索引規則。 
> 
> 

下列範例會設定一個特定路徑，採用的是範圍索引，且自訂精確度值為 20 個位元組：

```
    var collection = new DocumentCollection { Id = "rangeSinglePathCollection" };    

    collection.IndexingPolicy.IncludedPaths.Add(
        new IncludedPath { 
            Path = "/Title/?", 
            Indexes = new Collection<Index> { 
                new RangeIndex(DataType.String) { Precision = 20 } } 
            });

    // Default for everything else
    collection.IndexingPolicy.IncludedPaths.Add(
        new IncludedPath { 
            Path = "/*" ,
            Indexes = new Collection<Index> {
                new HashIndex(DataType.String) { Precision = 3 }, 
                new RangeIndex(DataType.Number) { Precision = -1 } 
            }
        });

    collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), pathRange);
```

新增路徑來編製索引時，這些路徑中的數字和字串都會編製索引。 因此即使您只為字串定義編製索引，Azure Cosmos DB 也會新增數字的預設定義。 換句話說，Azure Cosmos DB 可以從索引編製原則中排除路徑，但無法從具體的路徑排除類型。 以下是一個範例，請注意，paths (Path =  "/*" 和 ath =  "/\"attr1\"/?") 二者只被指定一個索引，但是數字資料類型也會新增至結果。

```
var indices = new[]{
                new IncludedPath  {
                    Indexes = new Collection<Index>
                    {
                        new RangeIndex(DataType.String) { Precision = 3 }// <- note: only 1 index specified
                    },
                    Path =  "/*"
                },
                new IncludedPath  {
                    Indexes = new Collection<Index>
                    {
                        new RangeIndex(DataType.String) { Precision = 3 } // <- note: only 1 index specified
                    },
                    Path =  "/\"attr1\"/?"
                }
            };...

            foreach (var index in indices)
            {
                documentCollection.IndexingPolicy.IncludedPaths.Add(index);
            }
```

索引建立結果：

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*",
            "indexes": [
                {
                    "kind": "Range",
                    "dataType": "String",
                    "precision": 3
                },
                {
                    "kind": "Range",
                    "dataType": "Number",
                    "precision": -1
                }
            ]
        },
        {
            "path": "/\"attr\"/?",
            "indexes": [
                {
                    "kind": "Range",
                    "dataType": "String",
                    "precision": 3
                },
                {
                    "kind": "Range",
                    "dataType": "Number",
                    "precision": -1
                }
            ]
        }
    ],
}
```

### <a name="index-data-types-kinds-and-precisions"></a>索引資料類型、種類和精確度
當您設定路徑的編制索引原則時，會有多個選項。 您可以為每個路徑指定一或多個編製索引的定義：

* **資料類型**：String、Number、Point、Polygon 或 LineString (每個路徑每個資料類型只能包含一個項目)。
* **索引類型**：雜湊 (相等查詢)、範圍 (相等、範圍或 ORDER BY 查詢) 或空間 (空間查詢)。
* **精確度**：字串和數字的雜湊索引精確度，會在 1 到 8 之間變換。 預設值為 3。 針對範圍索引，這個值可以是 -1 (最大精確度)。 針對字串或數字值，它會在 1 到 100 (最大精確度) 之間變換。

#### <a name="index-kind"></a>索引類型
Azure Cosmos DB 支援每個路徑的雜湊和範圍索引種類 (可針對 String、Number 資料類型或兩者進行設定)。

* **雜湊** 支援有效率的相等查詢和 JOIN 查詢。 針對大多數使用案例，雜湊索引並不需要比預設值 (3 個位元組) 更高的精確度。 資料類型可以是 String 或 Number。
* **範圍**支援有效率的相等查詢、範圍查詢 (使用 >、<、>=、<=、!=) 和 ORDER BY 查詢。 根據預設，ORDER By 查詢也需要最大索引精確度 (-1)。 資料類型可以是 String 或 Number。

Azure Cosmos DB 針對每個路徑也支援空間索引類型 (可針對 Point、Polygon 或 LineString 資料類型加以指定)。 位於指定路徑的值必須是有效的 GeoJSON 片段，例如 `{"type": "Point", "coordinates": [0.0, 10.0]}`。

* **空間** 支援有效率的空間 (內部和距離) 查詢。 資料類型可以是 Point、Polygon 或 LineString。

> [!NOTE]
> Azure Cosmos DB 支援自動編製 Point、Polygon 及 LineString 資料類型的索引。
> 
> 

以下是支援的索引種類和可用來處理的查詢的範例：

| 索引類型 | 描述/使用案例                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 雜湊       | Hash over /prop/? (或 /) 可用來有效率地處理下列查詢︰<br><br>SELECT FROM collection c WHERE c.prop = "value"<br><br>Hash over /props/[]/? (或 / 或 /props/) 可用來有效率地處理下列查詢︰<br><br>SELECT tag FROM collection c JOIN tag IN c.props WHERE tag = 5                                                                                                                       |
| 範圍      | Range over /prop/? (或 /) 可用來有效率地處理下列查詢︰<br><br>SELECT FROM collection c WHERE c.prop = "value"<br><br>SELECT FROM collection c WHERE c.prop > 5<br><br>SELECT FROM collection c ORDER BY c.prop                                                                                                                                                                                                              |
| 空間     | Range over /prop/? (或 /) 可用來有效率地處理下列查詢︰<br><br>SELECT FROM collection c<br><br>WHERE ST_DISTANCE(c.prop, {"type": "Point", "coordinates": [0.0, 10.0]}) < 40<br><br>SELECT FROM collection c WHERE ST_WITHIN(c.prop, {"type": "Polygon", ... }) --with indexing on points enabled<br><br>SELECT FROM collection c WHERE ST_WITHIN({"type": "Point", ... }, c.prop) --with indexing on polygons enabled              |

根據預設，如果沒有 (任何精確度的) 範圍索引，以發出可能需要掃描才能進行查詢的訊號，則含有範圍運算子 (例如 >=) 的查詢都會傳回錯誤。 只要在 REST API 中使用 **x-ms-documentdb-enable-scan** 標頭，或使用 .NET SDK 利用 **EnableScanInQuery** 要求選項，仍然可以在沒有範圍索引的情況下執行範圍查詢。 如果在查詢中有 Azure Cosmos DB 可以使用索引據以篩選的其他任何篩選，就不會傳回任何錯誤。

相同的規則適用於空間查詢。 根據預設，如果沒有空間索引，而且沒有可從索引提供服務的其他篩選，則會針對空間查詢傳回錯誤。 您可以使用 **x-ms-documentdb-enable-scan** 或 **EnableScanInQuery** 將它們執行為掃描。

#### <a name="index-precision"></a>索引精確度
您可以使用索引精確度，在索引儲存空間額外負荷和查詢效能之間取捨。 對於數字，建議使用預設精確度組態 -1 (最大值)。 因為數字在 JSON 中為 8 個位元組，相當於 8 個位元組的組態。 挑選較低值的精確度，例如 1-7，表示在某個範圍內的值可對應至相同的索引項目。 因此，雖然減少索引儲存空間，但查詢執行可能需要處理更多文件。 因此，它在要求單位中會消耗更多的輸送量。

索引精準度組態對於字串範圍有更實際的應用。 因為字串可以是任意長度，索引精確度的選擇可能會影響字串範圍查詢的效能。 它也可能會影響需要的索引儲存空間數量。 字串範圍索引可以設定為 1-100 或 -1 (最大值)。 如果您想要針對字串屬性執行 ORDER BY 查詢，您必須為對應的路徑，將精確度指定為 -1。

空間索引針對所有類型 (Points、LineString 及 Polygon) 一律都使用預設的索引精確度。 空間索引的預設索引精確度無法覆寫。 

下列範例示範如何使用 .NET SDK 提高集合中範圍索引的精確度。 

**使用自訂索引精確度建立集合**

    var rangeDefault = new DocumentCollection { Id = "rangeCollection" };

    // Override the default policy for strings to Range indexing and "max" (-1) precision
    rangeDefault.IndexingPolicy = new IndexingPolicy(new RangeIndex(DataType.String) { Precision = -1 });

    await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), rangeDefault);   


> [!NOTE]
> 當查詢使用 ORDER BY，但沒有針對所查詢路徑之最大精確度的範圍索引時，Azure Cosmos DB 就會傳回錯誤。 
> 
> 

同樣地，您可以從編製索引完全排除路徑。 下一個範例示範如何使用 \* 萬用字元運算子將文件 (「樹狀子目錄」) 的整個區段自編製索引中排除。

    var excluded = new DocumentCollection { Id = "excludedPathCollection" };
    excluded.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
    excluded.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/nonIndexedContent/*" });

    await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), excluded);



## <a name="opt-in-and-opt-out-of-indexing"></a>選擇加入或退出編製索引
您可以選擇是否要讓集合自動編製所有文件的索引。 根據預設，所有文件會自動編制索引，但您可以關閉自動編制索引。 關閉編製索引時，只能透過文件自己的連結或使用文件識別碼透過查詢來存取文件。

在關閉自動索引編製功能的情況下，您仍然可以選擇性地只將特定的文件新增到索引中。 相反地，您也可以讓自動編製索引保持開啟，並選擇性地排除特定的文件。 當您只需要查詢文件的子集時，編製索引開/關組態相當有用。

下列範例示範如何使用 [SQL API .NET SDK](https://docs.microsoft.com/azure/cosmos-db/sql-api-sdk-dotnet) 和 [RequestOptions.IndexingDirective](http://msdn.microsoft.com/library/microsoft.azure.documents.client.requestoptions.indexingdirective.aspx) 屬性來明確地包含文件。

    // If you want to override the default collection behavior to either
    // exclude (or include) a document in indexing,
    // use the RequestOptions.IndexingDirective property.
    client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"),
        new { id = "AndersenFamily", isRegistered = true },
        new RequestOptions { IndexingDirective = IndexingDirective.Include });

## <a name="modify-the-indexing-policy-of-a-collection"></a>修改集合的編製索引原則
在 Azure Cosmos DB 中，您可以即時對集合的編製索引原則進行變更。 變更 Azure Cosmos DB 集合中的編制索引原則可能會導致索引的圖形變更。 變更會影響可以編製索引的路徑、其精確度和索引本身的一致性模型。 變更編製索引原則時，需要將舊的索引有效率地轉換成新的索引。 

**線上索引轉換**

![編製索引的運作方式 – Azure Cosmos DB 線上索引轉換](./media/indexing-policies/index-transformations.png)

索引轉換是在線上進行。 也就是說，每個舊原則編製的文件索引會根據每個新原則有效率地轉換，「而不會影響集合的寫入可用性或佈建的輸送量」。 使用 REST API、SDK 或從預存程序和觸發程序中建立的讀取和寫入作業一致性在索引轉換期間不會受到影響。 

變更索引原則是非同步程序，並完成此作業所需的時間取決於文件數目、佈建的 RU 和文件大小。 正在重新編製索引時，如果查詢使用正在修改的索引，查詢就可能不會傳回所有相符的結果，且查詢不會傳回任何錯誤/失敗。 正在重新編製索引時，不論索引編製模式設定為何 (「一致」或「延遲」)，查詢最終會是一致的。 在索引轉換完成後，您將繼續看見一致的結果。 這適用於來自所有介面 (REST API、SDK) 的查詢，以及來自預存程序和觸發程序內的查詢。 索引轉換就像延遲編製索引一樣，是以非同步方式，使用適用於特定複本的備用資源，在複本背景執行。 

索引轉換也可就地執行。 Azure Cosmos DB 不會維護兩份索引，而是會以新索引交換舊索引。 也就是說，執行索引轉換時，您的集合中不需要也不會耗用其他任何磁碟空間。

當您變更編製索引原則時，主要會依據編制索引模式組態來套用變更，以從舊索引變更為新索引。 編制索引模式組態的角色遠比其他值 (例如包含/排除的路徑、編制索引種類及精確度) 重要。 

如果您的舊原則和新原則使用一致的編製索引模式，Azure Cosmos DB 就會執行線上索引轉換。 您無法在轉換進行時，使用一致的編製索引模式套用另一個編製索引原則變更。 不過，您可以在轉換進行時，移到「延遲」或「無」編製索引模式： 

* 當您移到「延遲」時，索引原則變更會立即生效。 Azure Cosmos DB 會開始以非同步的方式重新建立索引。 
* 當您移到「無」時，就會立即卸除索引。 當您想要取消進行中的轉換，並重新開始其他編製索引原則時，移至「無」相當實用。 

以下是說明如何將集合的編製索引原則從「一致」編製索引模式修改為「延遲」編製索引模式的程式碼片段。 如果您使用 .NET SDK，您可以使用新的 **ReplaceDocumentCollectionAsync** 方法開始編制索引原則變更。

**將編製索引原則從一致修改為延遲**

    // Switch to Lazy indexing mode.
    Console.WriteLine("Changing from Default to Lazy IndexingMode.");

    collection.IndexingPolicy.IndexingMode = IndexingMode.Lazy;

    await client.ReplaceDocumentCollectionAsync(collection);

**追蹤索引轉換的進度**

您可以使用 **ReadDocumentCollectionAsync** 呼叫中的 **IndexTransformationProgress**回應屬性來追蹤轉換成「一致」索引的索引轉換百分比進度。 其他變更編製索引原則的 SDK 和 REST API 支援對等屬性和方法。 您可以呼叫 **ReadDocumentCollectionAsync** 以檢查轉換成「一致」索引的進度： 

    long smallWaitTimeMilliseconds = 1000;
    long progress = 0;

    while (progress < 100)
    {
        ResourceResponse<DocumentCollection> collectionReadResponse = await client.ReadDocumentCollectionAsync(
            UriFactory.CreateDocumentCollectionUri("db", "coll"));

        progress = collectionReadResponse.IndexTransformationProgress;

        await Task.Delay(TimeSpan.FromMilliseconds(smallWaitTimeMilliseconds));
    }

> [!NOTE]
> * **IndexTransformationProgress** 屬性只有轉換成「一致」索引時才適用。 使用 **ResourceResponse.LazyIndexingProgress** 屬性可追蹤轉換成「延遲」索引的進度。
> * 只有非分割集區 (建立時不含分割區索引鍵的集區) 會填入 **IndexTransformationProgress** 和 **LazyIndexingProgress** 屬性。
>

您可以移動到「無」索引編製模式，以卸除集合的索引。 如果您想要取消進行中的轉換，並立即開始新的轉換，這可能是實用的操作工具。

**卸除集合的索引**

    // Switch to Lazy indexing mode.
    Console.WriteLine("Dropping index by changing to the None IndexingMode.");

    collection.IndexingPolicy.IndexingMode = IndexingMode.None;

    await client.ReplaceDocumentCollectionAsync(collection);

您什麼時候會對 Azure Cosmos DB 集合進行編製索引原則變更？ 以下是最常見的使用案例：

* 在正常操作期間提供一致的結果，但在大量資料匯入期間，改回延遲編製索引模式。
* 在目前的 Azure Cosmos DB 集合上開始使用新的編制索引功能。 例如，您可以使用地理空間查詢或 ORDER BY/字串範圍查詢，這兩種查詢分別需要空間索引種類及字串範圍索引種類。
* 手動選取要編製索引的屬性，並在一段時間後變更。
* 調整編製索引精確度，以改善查詢效能或減少耗用的儲存空間。

> [!NOTE]
> 若要使用 **ReplaceDocumentCollectionAsync** 修改編製索引原則，您需要使用 .NET SDK 1.3.0 版或更新版本。
> 
> 為使索引轉換順利完成，您必須確定集合上有足夠的可用儲存空間。 如果集合已達到其儲存配額，將會暫停索引轉換。 一旦有可用的儲存空間 (例如您刪除某些文件)，索引轉換會自動繼續。
> 
> 

## <a name="performance-tuning"></a>效能微調
SQL API 會提供效能度量 (像是已使用的索引儲存體)，以及每個作業的輸送量成本 (要求單位) 等相關資訊。 這項資訊可以用來比較各種編製索引的原則，以及用來微調效能。

若要檢查集合的儲存體配額和使用量，請對集合資源執行 **HEAD** 或 **GET** 要求。 接著，檢查 **x-ms-request-quota** 和 **x-ms-request-usage**標頭。 在 .NET SDK 中，[ResourceResponse<T\>](http://msdn.microsoft.com/library/dn799209.aspx) 中的 [DocumentSizeQuota](http://msdn.microsoft.com/library/dn850325.aspx) 和 [DocumentSizeUsage](http://msdn.microsoft.com/library/azure/dn850324.aspx) 屬性包含這些對應的值。

     // Measure the document size usage (which includes the index size) against   
     // different policies.
     ResourceResponse<DocumentCollection> collectionInfo = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));  
     Console.WriteLine("Document size quota: {0}, usage: {1}", collectionInfo.DocumentQuota, collectionInfo.DocumentUsage);


若要測量每一個寫入作業 (建立、更新或刪除) 的編製索引負荷，請檢查 **x-ms-request-charge** 標頭 (或 .NET SDK 的 [ResourceResponse<T\>](http://msdn.microsoft.com/library/dn799209.aspx) 中同等的 [RequestCharge](http://msdn.microsoft.com/library/dn799099.aspx) 屬性) 來測量這些作業所耗用的要求單位數量。

     // Measure the performance (request units) of writes.     
     ResourceResponse<Document> response = await client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"), myDocument);              
     Console.WriteLine("Insert of document consumed {0} request units", response.RequestCharge);

     // Measure the performance (request units) of queries.    
     IDocumentQuery<dynamic> queryable =  client.CreateDocumentQuery(UriFactory.CreateDocumentCollectionUri("db", "coll"), queryString).AsDocumentQuery();

     double totalRequestCharge = 0;
     while (queryable.HasMoreResults)
     {
        FeedResponse<dynamic> queryResponse = await queryable.ExecuteNextAsync<dynamic>(); 
        Console.WriteLine("Query batch consumed {0} request units",queryResponse.RequestCharge);
        totalRequestCharge += queryResponse.RequestCharge;
     }

     Console.WriteLine("Query consumed {0} request units in total", totalRequestCharge);

## <a name="changes-to-the-indexing-policy-specification"></a>索引編製原則規格的變更
2015 年 7 月 7 日在 2015-06-03 版的 REST API 中引進編製索引原則結構描述的變更。 SDK 版本中的對應類別具有符合結構描述的新實作。 

在 JSON 規格中實作下列變更：

* 編製索引原則支援字串的範圍索引。
* 每個路徑可以有多個索引定義。 一個定義用於一種資料類型。
* 編製索引精確度針對數字支援 1-8、針對字串支援 1-100 和支援 -1 (最大精確度)。
* 路徑區段不需要雙引號來逸出每個路徑。 例如，您可以針對 **/title/?** 新增路徑，而不是 **/"title"/?**。
* 代表「所有路徑」的根路徑可以表示為 **/\*** (除了 **/** 以外)。

如果您的程式碼佈建的集合含有以 .NET SDK 1.1.0 版或較舊的版本撰寫的自訂編製索引原則，為移至 SDK 1.2.0 版，您必須變更您的應用程式程式碼以處理這些變更。 如果您沒有會設定編製索引原則的程式碼，或打算繼續使用舊的 SDK 版本，則不需要任何變更。

如需實用的比較，這裡提供使用 2015-06-03 版的 REST API 撰寫的自訂編制索引原則範例，後面則是使用先前的 2015-04-08 版的 REST API 撰寫的相同編製索引原則。

**目前的編製索引原則 JSON (2015-06-03 版的 REST API)**

    {
       "automatic":true,
       "indexingMode":"Consistent",
       "includedPaths":[
          {
             "path":"/*",
             "indexes":[
                {
                   "kind":"Hash",
                   "dataType":"String",
                   "precision":3
                },
                {
                   "kind":"Hash",
                   "dataType":"Number",
                   "precision":7
                }
             ]
          }
       ],
       "ExcludedPaths":[
          {
             "path":"/nonIndexedContent/*"
          }
       ]
    }


**先前的編製索引原則 JSON (2015-04-08 版的 REST API)**

    {
       "automatic":true,
       "indexingMode":"Consistent",
       "IncludedPaths":[
          {
             "IndexType":"Hash",
             "Path":"/",
             "NumericPrecision":7,
             "StringPrecision":3
          }
       ],
       "ExcludedPaths":[
          "/\"nonIndexedContent\"/*"
       ]
    }


## <a name="next-steps"></a>後續步驟
如需索引原則管理範例及深入了解 Azure Cosmos DB 查詢語言，請參考下列連結：

* [SQL API .NET 索引管理程式碼範例](https://github.com/Azure/azure-documentdb-net/blob/master/samples/code-samples/IndexManagement/Program.cs) \(英文\)
* [SQL API REST 集合作業](https://msdn.microsoft.com/library/azure/dn782195.aspx)
* [使用 SQL 查詢](sql-api-sql-query.md)

