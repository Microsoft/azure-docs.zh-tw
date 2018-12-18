---
title: 使用 Azure Cosmos DB 中的地理空間資料 | Microsoft Docs
description: 了解如何使用 Azure Cosmos DB 和 SQL API 建立與查詢空間物件，以及為其編製索引。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 10/20/2017
ms.author: sngun
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 74824af6f17a6c1d2638c8604edd38ffa419d607
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37858113"
---
# <a name="working-with-geospatial-and-geojson-location-data-in-azure-cosmos-db"></a>使用 Azure Cosmos DB 中的地理空間和 GeoJSON 位置資料
本文將介紹 [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 中的地理空間功能。 閱讀本文後，您將能夠回答下列問題：

* 如何在 Azure Cosmos DB 中儲存空間資料？
* 如何以 SQL 和 LINQ 查詢 Azure Cosmos DB 中的地理空間資料？
* 如何在 Azure Cosmos DB 中啟用或停用空間編製索引？

本文示範如何透過 SQL API 使用空間資料。 請參閱此 [GitHub 專案](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/Geospatial/Program.cs)中的程式碼範例。

## <a name="introduction-to-spatial-data"></a>空間資料簡介
空間資料可描述空間中物件的位置和形狀。 在大部分的應用程式中，這些會對應至地球上的物件和地理空間資料。 空間資料可以用來代表人、感興趣的地方、城市邊界或湖泊。 常見使用案例通常涉及鄰近性查詢，例如「尋找我目前位置附近的所有咖啡廳」。 

### <a name="geojson"></a>GeoJSON
Azure Cosmos DB 支援對使用 [GeoJSON 規格 (英文)](https://tools.ietf.org/html/rfc7946) 表示的地理空間點資料執行編製索引和查詢。 GeoJSON 資料結構一律為有效的 JSON 物件，因此可透過 Azure Cosmos DB 來儲存及查詢，無須使用任何特殊的工具或程式庫。 Azure Cosmos DB SDK 提供協助程式類別和方法，以便更容易使用空間資料。 

### <a name="points-linestrings-and-polygons"></a>點、LineString 和多邊形
**點** 代表空間中的單一位置。 在地理空間資料中，某個點所代表的確切位置可能是雜貨店的街道地址、電話亭、汽車或城市。  點會使用其座標組或經緯度，以 GeoJSON (和 Azure Cosmos DB) 來表示。 以下是點的 JSON 範例。

**Azure Cosmos DB 中的點**

```json
{
    "type":"Point",
    "coordinates":[ 31.9, -4.8 ]
}
```

> [!NOTE]
> GeoJSON 規格會先指定經度，然後再指定緯度。 如同其他的地圖應用程式，經度和緯度為角度，並以度為表示單位。 經度值是從本初子午線測量，並介於 -180 度和 180.0 度之間；緯度值是從赤道測量，並介於 -90.0 度和 90.0 度之間。 
> 
> Azure Cosmos DB 會依照 WGS-84 參考系統所表示的方式來解譯座標。 請參閱下方詳細的座標參考系統資料。
> 
> 

如這個包含位置資料的使用者設定檔範例所示，這可內嵌於 Azure Cosmos DB 文件中：

**儲存在 Azure Cosmos DB 中含有位置的使用設定檔**

```json
{
    "id":"cosmosdb-profile",
    "screen_name":"@CosmosDB",
    "city":"Redmond",
    "topics":[ "global", "distributed" ],
    "location":{
        "type":"Point",
        "coordinates":[ 31.9, -4.8 ]
    }
}
```

除了點之外，GeoJSON 也支援 Linestring 和多邊形。 **Linestring** 表示空間中一連串的點 (兩個以上)，以及連接這些點的線段。 在地理空間資料中，LineStrings 通常用來代表高速公路或河流。 **多邊形**是由連接的點組成邊界，並形成封閉的 LineString。 多邊形常用來代表自然構成物，例如湖泊，或代表政治管轄權，例如城市和州省。 以下是 Azure Cosmos DB 中多邊形的範例。 

**GeoJSON 中的多邊形**

```json
{
    "type":"Polygon",
    "coordinates":[ [
        [ 31.8, -5 ],
        [ 31.8, -4.7 ],
        [ 32, -4.7 ],
        [ 32, -5 ],
        [ 31.8, -5 ]
    ] ]
}
```

> [!NOTE]
> GeoJSON 規格需要此資料才能形成有效的多邊形；若要建立一個封閉的形狀，最後一個座標組應該與第一個座標組相同。
> 
> 多邊形內的點必須以逆時針順序指定。 以順時針順序指定的多邊形，代表區域內的反轉。
> 
> 

除了點、LineString 和多邊形之外，GeoJSON 也會指定如何將多個地理空間位置的表示加以分組，以及如何將任意屬性與地理位置產生關聯成為**特徵**的表示。 由於這些物件都是有效的 JSON，因此均可在 Azure Cosmos DB 中儲存及處理。 不過，Azure Cosmos DB 僅支援自動編製點的索引。

### <a name="coordinate-reference-systems"></a>座標參考系統
由於地球的形狀並不規則，地理空間資料的座標可以許多座標參考系統 (CRS) 來表示，而這些系統各有自己的參考框架和測量單位。 例如「英國國家格網參考系統」對英國而言是精確的參考系統，但對其他地區則不是。 

現今最常使用的 CRS 是「全球大地座標系統」[WGS-84](http://earth-info.nga.mil/GandG/wgs84/)。 GPS 裝置和許多地圖服務，包括 Google 地圖與 Bing Maps API 均是使用 WGS-84。 Azure Cosmos DB 僅支援對使用 WGS-84 CRS 的地理空間資料執行編製索引和查詢。 

## <a name="creating-documents-with-spatial-data"></a>建立具有空間資料的文件
當您建立包含 GeoJSON 值的文件時，值會根據容器的索引編製原則，自動以空間索引進行索引編製。 如果您是以動態類型的語言 (如 Python 或 Node.js) 使用 Azure Cosmos DB SDK，則必須建立有效的 GeoJSON。

**以 Node.js 建立具有地理空間資料的文件**

```json
var userProfileDocument = {
    "name":"cosmosdb",
    "location":{
        "type":"Point",
        "coordinates":[ -122.12, 47.66 ]
    }
};

client.createDocument(`dbs/${databaseName}/colls/${collectionName}`, userProfileDocument, (err, created) => {
    // additional code within the callback
});
```

如果您使用 SQL API，則可在 `Microsoft.Azure.Documents.Spatial` 命名空間中使用 `Point` 及 `Polygon` 類別，藉此將位置資訊內嵌在應用程式物件中。 這些類別可協助將空間資料序列化和還原序列化簡化成 GeoJSON。

**以 .NET 建立具有地理空間資料的文件**

```json
using Microsoft.Azure.Documents.Spatial;

public class UserProfile
{
    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("location")]
    public Point Location { get; set; }

    // More properties
}

await client.CreateDocumentAsync(
    UriFactory.CreateDocumentCollectionUri("db", "profiles"), 
    new UserProfile 
    { 
        Name = "cosmosdb", 
        Location = new Point (-122.12, 47.66) 
    });
```

如果您沒有經緯度資訊，但有實體地址或位置名稱，如城市或國家/地區，您可以使用像是 Bing Maps REST 服務之類的地理編碼服務，來查閱實際的座標。 在 [這裡](https://msdn.microsoft.com/library/ff701713.aspx)深入了解 Bing Maps 地理編碼。

## <a name="querying-spatial-types"></a>查詢空間類型
既然我們已經探討過如何插入地理空間資料，現在就來看看如何透過 SQL 和 LINQ 使用 Azure Cosmos DB 查詢此資料。

### <a name="spatial-sql-built-in-functions"></a>空間 SQL 內建函數
Azure Cosmos DB 支援下列開放地理空間協會 (OGC) 內建的地理空間查詢函式。 如需更多完整的 SQL 語言內建函式，請參閱[查詢 Azure Cosmos DB](sql-api-sql-query.md)。

<table>
<tr>
  <td><strong>使用量</strong></td>
  <td><strong>說明</strong></td>
</tr>
<tr>
  <td>ST_DISTANCE (spatial_expr, spatial_expr)</td>
  <td>傳回兩個 GeoJSON Point、Polygon 或 LineString 運算式之間的距離。</td>
</tr>
<tr>
  <td>ST_WITHIN (spatial_expr, spatial_expr)</td>
  <td>傳回布林運算式，指出第一個 GeoJSON 物件 (Point、Polygon 或 LineString) 是否位在第二個 GeoJSON 物件 (Point、Polygon 或 LineString) 內。</td>
</tr>
<tr>
  <td>ST_INTERSECTS (spatial_expr, spatial_expr)</td>
  <td>傳回布林運算式，指出兩個指定的 GeoJSON 物件 (Point、Polygon 或 LineString) 是否相交。</td>
</tr>
<tr>
  <td>ST_ISVALID</td>
  <td>傳回布林值，指出指定的 GeoJSON Point、Polygon 或 LineString 運算式是否有效。</td>
</tr>
<tr>
  <td>ST_ISVALIDDETAILED</td>
  <td>如果指定的 GeoJSON Point、Polygon 或 LineString 運算式有效，就傳回包含布林值的 JSON 值；但如果是無效的，就會額外加上做為字串值的原因。</td>
</tr>
</table>

空間函數可以用來對空間資料執行鄰近性查詢。 例如，以下查詢使用 ST_DISTANCE 內建函數傳回所有家族文件，且這些文件在 30 公里指定位置內。 

**查詢**

    SELECT f.id 
    FROM Families f 
    WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000

**結果**

    [{
      "id": "WakefieldFamily"
    }]

如果您將空間索引編製包含在索引編製原則中，則「距離查詢」將會透過索引獲得有效利用。 如需空間索引編製的詳細資料，請參閱下面的章節。 如果您沒有指定路徑的空間索引，仍然可以透過指定 `x-ms-documentdb-query-enable-scan` 要求標頭 (且值設定為 "true") 執行空間查詢。 在.NET 中，可以傳遞選用的 **FeedOptions** 引數至查詢 (且 [EnableScanInQuery](https://msdn.microsoft.com/library/microsoft.azure.documents.client.feedoptions.enablescaninquery.aspx#P:Microsoft.Azure.Documents.Client.FeedOptions.EnableScanInQuery) 設定為 true)，藉此執行此作業。 

ST_WITHIN 可用來檢查點是否在多邊形內。 多邊形常用來表示邊界，例如郵遞區號、州省邊界或自然構成物。 此外，如果您將空間索引編製包含在索引編製原則中，則「距離內」查詢將會透過索引獲得有效利用。 

ST_WITHIN 中的多邊形引數只可以包含單一通道，也就是 Polygon 本身不能有漏洞。 

**查詢**

    SELECT * 
    FROM Families f 
    WHERE ST_WITHIN(f.location, {
        'type':'Polygon', 
        'coordinates': [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]
    })

**結果**

    [{
      "id": "WakefieldFamily",
    }]

> [!NOTE]
> 與 Azure Cosmos DB 查詢中不相符類型的運作方式類似，如果任一引數中指定的位置值格式不正確或無效，則會評估為**未定義**，且會在查詢結果中略過已評估的文件。 如果您的查詢沒有傳回任何結果，請執行 ST_ISVALIDDETAILED 來偵錯，了解空間類型無效的原因。     
> 
> 

Azure Cosmos DB 也支援反向查詢，亦即您可以在 Azure Cosmos DB 中編製多邊形或線的索引，然後查詢包含指定點的區域。 這是物流業中常用的模式，例如，可用來識別卡車何時進入或離開指定的區域。 

**查詢**

    SELECT * 
    FROM Areas a 
    WHERE ST_WITHIN({'type': 'Point', 'coordinates':[31.9, -4.8]}, a.location)


**結果**

    [{
      "id": "MyDesignatedLocation",
      "location": {
        "type":"Polygon", 
        "coordinates": [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]
      }
    }]

ST_ISVALID 和 ST_ISVALIDDETAILED 可用來檢查空間物件是否有效。 例如，下列查詢以超出範圍的緯度值 (-132.8)，檢查點的有效性。 ST_ISVALID 只會傳回布林值，而 ST_ISVALIDDETAILED 會傳回布林和字串，字串中包含被視為無效的原因。

** 查詢 **

    SELECT ST_ISVALID({ "type": "Point", "coordinates": [31.9, -132.8] })

**結果**

    [{
      "$1": false
    }]

這些函數也可以用來驗證多邊形。 例如，這裡我們使用 ST_ISVALIDDETAILED 驗證未封閉的多邊形。 

**查詢**

    SELECT ST_ISVALIDDETAILED({ "type": "Polygon", "coordinates": [[ 
        [ 31.8, -5 ], [ 31.8, -4.7 ], [ 32, -4.7 ], [ 32, -5 ] 
        ]]})

**結果**

    [{
       "$1": { 
            "valid": false, 
            "reason": "The Polygon input is not valid because the start and end points of the ring number 1 are not the same. Each ring of a Polygon must have the same start and end points." 
          }
    }]

### <a name="linq-querying-in-the-net-sdk"></a>.NET SDK 中的 LINQ 查詢
SQL .NET SDK 也是虛設常式方法 `Distance()` 和 `Within()` 的提供者，供您在 LINQ 運算式中使用。 SQL LINQ 提供者會將這些方法呼叫，轉譯為同等的 SQL 內建函數呼叫 (分別為 ST_DISTANCE 和 ST_WITHIN)。 

以下是 LINQ 查詢的範例，該查詢會使用 LINQ 在 Azure Cosmos DB 集合中找出所有文件，而這些文件的「位置」值會在指定點的 30 公里半徑內。

**距離的 LINQ 查詢**

    foreach (UserProfile user in client.CreateDocumentQuery<UserProfile>(UriFactory.CreateDocumentCollectionUri("db", "profiles"))
        .Where(u => u.ProfileType == "Public" && a.Location.Distance(new Point(32.33, -4.66)) < 30000))
    {
        Console.WriteLine("\t" + user);
    }

同樣地，以下是尋找所有文件的查詢，這些文件的「位置」均在指定的方塊/多邊形內。 

**範圍內的 LINQ 查詢**

    Polygon rectangularArea = new Polygon(
        new[]
        {
            new LinearRing(new [] {
                new Position(31.8, -5),
                new Position(32, -5),
                new Position(32, -4.7),
                new Position(31.8, -4.7),
                new Position(31.8, -5)
            })
        });

    foreach (UserProfile user in client.CreateDocumentQuery<UserProfile>(UriFactory.CreateDocumentCollectionUri("db", "profiles"))
        .Where(a => a.Location.Within(rectangularArea)))
    {
        Console.WriteLine("\t" + user);
    }


既然我們已經探討過如何使用 LINQ 和 SQL 查詢文件，現在來看一下如何針對空間編製索引設定 Azure Cosmos DB。

## <a name="indexing"></a>編製索引
如我們在[使用 Azure Cosmos DB 進行無從驗證結構描述的編製索引 (英文)](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf) 文件中所述，我們設計的 Azure Cosmos DB 資料庫引擎真正是無從驗證結構描述的，並提供一流的 JSON 支援。 Azure Cosmos DB 的寫入最佳化資料庫引擎原生就能了解以 GeoJSON 標準表示的空間資料 (點、多邊形及線)。

簡單來說，大地座標的幾何會投影在 2D 平面上，然後使用 **quadtree**以漸進方式分成格子。 這些格子會根據 **希伯特空間填滿曲線**(Hilbert space filling curve) 內的格子位置對應至 1D，並保留點的位置。 此外，在為位置資料編製索引之後，會經過稱為**鑲嵌式**的處理程序，也就是會將位置上相交的所有格子識別為索引鍵並儲存在 Azure Cosmos DB 索引中。 在查詢時，點和多邊形之類的引數也會經過鑲嵌，以擷取相關的格子 ID 範圍，然後用來從索引擷取資料。

如果指定的索引編製原則包含 /* (所有路徑) 的空間索引，則表示集合中可找到的所有點均已編製索引，能進行有效率的空間查詢 (ST_WITHIN 和 ST_DISTANCE)。 空間索引沒有整數位數值，且一律使用預設的整數位數值。

> [!NOTE]
> Azure Cosmos DB 支援自動編製 Point、Polygon 及 LineString 的索引
> 
> 

以下 JSON 片段顯示已啟用空間索引的索引編製原則，也就是為文件中可找到的所有 GeoJSON 點編製索引，以用於空間查詢。 如果您要使用 Azure 入口網站修改索引編製原則，可以為索引編製原則指定以下 JSON，藉此啟用集合的空間索引編製。

**已針對點和多邊形啟用空間的集合索引編製原則 JSON**

    {
       "automatic":true,
       "indexingMode":"Consistent",
       "includedPaths":[
          {
             "path":"/*",
             "indexes":[
                {
                   "kind":"Range",
                   "dataType":"String",
                   "precision":-1
                },
                {
                   "kind":"Range",
                   "dataType":"Number",
                   "precision":-1
                },
                {
                   "kind":"Spatial",
                   "dataType":"Point"
                },
                {
                   "kind":"Spatial",
                   "dataType":"Polygon"
                }                
             ]
          }
       ],
       "excludedPaths":[
       ]
    }

以下是 .NET 的程式碼片段示範，供您了解如何建立集合，同時針對所有包含點的路徑啟用空間索引編製。 

**建立含空間索引編制的集合**

    DocumentCollection spatialData = new DocumentCollection()
    spatialData.IndexingPolicy = new IndexingPolicy(new SpatialIndex(DataType.Point)); //override to turn spatial on by default
    collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), spatialData);

而以下說明如何修改現有的集合，以利用儲存在文件內任何一個點上的空間索引編制。

**修改含空間索引編制的現有集合**

    Console.WriteLine("Updating collection with spatial indexing enabled in indexing policy...");
    collection.IndexingPolicy = new IndexingPolicy(new SpatialIndex(DataType.Point));
    await client.ReplaceDocumentCollectionAsync(collection);

    Console.WriteLine("Waiting for indexing to complete...");
    long indexTransformationProgress = 0;
    while (indexTransformationProgress < 100)
    {
        ResourceResponse<DocumentCollection> response = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));
        indexTransformationProgress = response.IndexTransformationProgress;

        await Task.Delay(TimeSpan.FromSeconds(1));
    }

> [!NOTE]
> 如果文件中的 GeoJSON 位置值格式不正確或無效，就不會為其編製索引以用於空間查詢。 您可以使用 ST_ISVALID 和 ST_ISVALIDDETAILED 驗證位置值。
> 
> 如果您的集合定義包含分割索引鍵，不會報告索引轉換進度。 
> 
> 

## <a name="next-steps"></a>後續步驟
既然您已了解如何開始使用 Azure Cosmos DB 中的地理空間支援，您可以接著：

* 使用 [GitHub 上的地理空間 .NET 程式碼範例](https://github.com/Azure/azure-documentdb-dotnet/blob/fcf23d134fc5019397dcf7ab97d8d6456cd94820/samples/code-samples/Geospatial/Program.cs)來開始轉寫程式碼
* 在 [Azure Cosmos DB 查詢園地 (英文)](http://www.documentdb.com/sql/demo#geospatial) 中瞭解地理空間查詢
* 深入了解 [Azure Cosmos DB 查詢](sql-api-sql-query.md)
* 深入了解 [Azure Cosmos DB 編製索引原則](indexing-policies.md)

