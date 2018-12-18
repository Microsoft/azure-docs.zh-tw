---
title: 使用 Azure Cosmos DB 中的日期 | Microsoft Docs
description: 了解如何使用 Azure Cosmos DB 中的日期。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: sngun
ms.openlocfilehash: d7188270ff5b1edd3b5e396be0cd5fd22e6123c4
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37855501"
---
# <a name="working-with-dates-in-azure-cosmos-db"></a>使用 Azure Cosmos DB 中的日期
Azure Cosmos DB 透過原生 [JSON](http://www.json.org) 資料模型，提供結構描述的彈性和豐富的索引編製功能。 所有 Azure Cosmos DB 資源 (包括資料庫、容器、文件及預存程序) 都會建立模型，並以 JSON 文件的形式儲存。 為了滿足可攜性需求，JSON (和 Azure Cosmos DB) 僅支援一小組基本類型︰字串、數字、布林值、陣列、物件及 Null。 不過，JSON 具有彈性，讓開發人員和架構可以使用這些基本類型及加以組合為物件或陣列，來表示更複雜的類型。 

除了基本類型，許多應用程式需要 [DateTime](https://msdn.microsoft.com/library/system.datetime(v=vs.110).aspx) 類型來表示日期和時間戳記。 此文章說明開發人員如何使用 .NET SDK 來儲存、擷取、查詢 Azure Cosmos DB 中的日期。

## <a name="storing-datetimes"></a>儲存 DateTimes
根據預設，[Azure Cosmos DB SDK](sql-api-sdk-dotnet.md) 會將 DateTime 值序列化為 [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874) 字串。 大部分應用程式可以使用 DateTime 的預設字串表示法，原因如下︰

* 可以比較字串，且在將 DateTime 值轉換成字串時會保留這些字元的相對順序。 
* 這個方法不需要任何自訂程式碼或屬性來進行 JSON 轉換。
* 儲存在 JSON 中的日期是人類可閱讀的。
* 此方法可利用 Azure Cosmos DB 的索引來提升查詢效能。

例如，下列程式碼片段使用 .NET SDK 將 `Order` 物件 (包含 `ShipDate` 和 `OrderDate` 兩個 DateTime 屬性) 儲存為文件︰

    public class Order
    {
        [JsonProperty(PropertyName="id")]
        public string Id { get; set; }
        public DateTime OrderDate { get; set; }
        public DateTime ShipDate { get; set; }
        public double Total { get; set; }
    }

    await client.CreateDocumentAsync("/dbs/orderdb/colls/orders", 
        new Order 
        { 
            Id = "09152014101",
            OrderDate = DateTime.UtcNow.AddDays(-30),
            ShipDate = DateTime.UtcNow.AddDays(-14), 
            Total = 113.39
        });

這份文件會在 Azure Cosmos DB 中以下列方式儲存：

    {
        "id": "09152014101",
        "OrderDate": "2014-09-15T23:14:25.7251173Z",
        "ShipDate": "2014-09-30T23:14:25.7251173Z",
        "Total": 113.39
    }
    

或者，您可以將 DateTimes 儲存為 Unix 時間戳記，也就是用一個數字來表示自 1970 年 1 月 1 日之後經過的秒數。 Azure Cosmos DB 的內部時間戳記 (`_ts`) 屬性便是使用此方法。 您可以使用 [UnixDateTimeConverter](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.unixdatetimeconverter.aspx) 類別將 DateTime 序列化為數字。 

## <a name="indexing-datetimes-for-range-queries"></a>索引 DateTime 以進行範圍查詢
DateTime 值常用於範圍查詢。 例如，如果您需要尋找昨天之後的所有訂單，或在過去五分鐘內出貨的所有訂單，則需要執行範圍查詢。 若要有效率地執行這些查詢，您必須設定用於對字串進行範圍索引的集合。

    DocumentCollection collection = new DocumentCollection { Id = "orders" };
    collection.IndexingPolicy = new IndexingPolicy(new RangeIndex(DataType.String) { Precision = -1 });
    await client.CreateDocumentCollectionAsync("/dbs/orderdb", collection);

您可以參閱 [Azure Cosmos DB 索引編製原則](indexing-policies.md)，來深入了解如何設定索引編製原則。

## <a name="querying-datetimes-in-linq"></a>用 LINQ 查詢 DateTime
SQL .NET SDK 會自動支援透過 LINQ 查詢儲存在 Azure Cosmos DB 中的資料。 例如，下列程式碼片段中的 LINQ 查詢會篩選在過去三天出貨的訂單。

    IQueryable<Order> orders = client.CreateDocumentQuery<Order>("/dbs/orderdb/colls/orders")
        .Where(o => o.ShipDate >= DateTime.UtcNow.AddDays(-3));
          
    // Translated to the following SQL statement and executed on Azure Cosmos DB
    SELECT * FROM root WHERE (root["ShipDate"] >= "2016-12-18T21:55:03.45569Z")

您可以參閱[查詢 Cosmos DB](sql-api-sql-query.md)，來深入了解 Azure Cosmos DB 的 SQL 查詢語言和 LINQ 提供者。

在此文章中，我們已經了解如何進行 Azure Cosmos DB 中 DateTime 的儲存、索引編製及查詢。

## <a name="next-steps"></a>後續步驟
* 下載並執行 [GitHub 上的程式碼範例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples)
* 深入了解 [SQL 查詢](sql-api-sql-query.md)
* 深入了解 [Azure Cosmos DB 編製索引原則](indexing-policies.md)
