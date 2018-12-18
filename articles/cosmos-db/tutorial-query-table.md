---
title: 如何在 Azure Cosmos DB 中查詢資料表資料？ | Microsoft Docs
description: 了解如何在 Azure Cosmos DB 中查詢資料表資料
services: cosmos-db
author: kanshiG
manager: kfile
editor: ''
tags: ''
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: na
ms.topic: tutorial
ms.date: 11/15/2017
ms.author: govindk
ms.custom: mvc
ms.openlocfilehash: 8a629a7b1340547f43919a1e88abbda53e9e2ae0
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "34763386"
---
# <a name="tutorial-query-azure-cosmos-db-by-using-the-table-api"></a>教學課程：使用 Table API 查詢 Azure Cosmos DB

Azure Cosmos DB [資料表 API](table-introduction.md) 支援對索引鍵/值 (資料表) 資料進行 OData 和 [LINQ](https://docs.microsoft.com/rest/api/storageservices/fileservices/writing-linq-queries-against-the-table-service) 查詢。  

本文涵蓋下列工作： 

> [!div class="checklist"]
> * 使用資料表 API 來查詢資料

本文中的查詢使用下列範例 `People` 資料表：

| PartitionKey | RowKey | 電子郵件 | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0101 |
| Smith | Ben | Ben@contoso.com| 425-555-0102 |
| Smith | Jeff | Jeff@contoso.com| 425-555-0104 | 

如需怎樣使用資料表 API 進行查詢的詳細資料，請參閱[查詢資料表和實體](https://docs.microsoft.com/rest/api/storageservices/fileservices/querying-tables-and-entities)。 

如需 Azure Cosmos DB 所提供之進階功能的詳細資訊，請參閱 [Azure Cosmos DB：資料表 API](table-introduction.md) 和[在 .NET 中使用資料表 API 進行開發](tutorial-develop-table-dotnet.md)。 

## <a name="prerequisites"></a>先決條件

若要讓這些查詢能夠運作，您必須具備 Azure Cosmos DB 帳戶，並且在容器中有實體資料。 不符合上述其中任何一項條件嗎？ 請完成 [5 分鐘快速入門](create-table-dotnet.md)或[開發人員教學課程](tutorial-develop-table-dotnet.md)，以建立帳戶並在資料庫中填入資料。

## <a name="query-on-partitionkey-and-rowkey"></a>在 PartitionKey 和 RowKey 上執行查詢
由於 PartitionKey 和 RowKey 屬性會構成實體的主索引鍵，因此您可以使用下列特殊語法來識別實體： 

**查詢**

```
https://<mytableendpoint>/People(PartitionKey='Harp',RowKey='Walter')  
```
**結果**

| PartitionKey | RowKey | 電子郵件 | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0104 |

或者，您也可以在指定 `$filter` 選項時一併指定這些屬性，如下一節中所示。 請注意，索引鍵屬性名稱和常數值有區分大小寫。 PartitionKey 和 RowKey 屬性的類型都是 String。 

## <a name="query-by-using-an-odata-filter"></a>使用 OData 篩選進行查詢
建構篩選字串時，請牢記下列規則： 

* 使用「OData 通訊協定規格」所定義的邏輯運算子來比較屬性與值。 請注意，您無法比較屬性與動態值。 運算式的一端必須是常數。 
* 屬性名稱、運算子及常數值必須以 URL 編碼的空格分隔。 空格經 URL 編碼後會變成 `%20`。 
* 篩選字串的所有部分都區分大小寫。 
* 常數和屬性必須是相同的資料類型，篩選才能傳回有效的結果。 如需支援的屬性類型的詳細資訊，請參閱 [了解表格服務資料模型](https://docs.microsoft.com/rest/api/storageservices/understanding-the-table-service-data-model)。 

以下是一個範例查詢，說明如如何使用 OData `$filter` 依 PartitionKey 和 Email 屬性進行篩選。

**查詢**

```
https://<mytableapi-endpoint>/People()?$filter=PartitionKey%20eq%20'Smith'%20and%20Email%20eq%20'Ben@contoso.com'
```

如需如何針對各種資料類型建構篩選條件運算式的詳細資訊，請參閱[查詢資料表和實體](https://docs.microsoft.com/rest/api/storageservices/querying-tables-and-entities)。

**結果**

| PartitionKey | RowKey | 電子郵件 | PhoneNumber |
| --- | --- | --- | --- |
| Ben |Smith | Ben@contoso.com| 425-555-0102 |

## <a name="query-by-using-linq"></a>使用 LINQ 進行查詢 
您也可以使用 LINQ 進行查詢，這會轉譯成對應的 OData 查詢運算式。 以下是一個範例，說明如何使用 .NET SDK 來建置查詢：

```csharp
CloudTableClient tableClient = account.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("people");

TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>()
    .Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition(PartitionKey, QueryComparisons.Equal, "Smith"),
            TableOperators.And,
            TableQuery.GenerateFilterCondition(Email, QueryComparisons.Equal,"Ben@contoso.com")
    ));

await table.ExecuteQuerySegmentedAsync<CustomerEntity>(query, null);
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列操作：

> [!div class="checklist"]
> * 了解如何使用資料表 API 來進行查詢

您現在可以繼續進行到下一個教學課程，以了解如何全域散發您的資料。

> [!div class="nextstepaction"]
> [全域散發您的資料](tutorial-global-distribution-table.md)
