---
title: Azure Cosmos DB：大量執行程式 .NET API、SDK 和資源 | Microsoft Docs
description: 了解所有大量執行程式 .NET API 和 SDK 相關資訊，包括發行日期、停用日期及 Azure Cosmos DB 大量執行程式 .NET SDK 每個版本之間所作出的變更。
services: cosmos-db
author: tknandu
manager: kfile
editor: cgronlun
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 09/14/2018
ms.author: ramkris
ms.openlocfilehash: ffd8f438429cd8769ac0dbff7f489327166e0000
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46294453"
---
# <a name="net-bulk-executor-library-download-information"></a>.NET 大量執行程式程式庫：下載資訊 

> [!div class="op_single_selector"]
> * [.NET](sql-api-sdk-dotnet.md)
> * [.NET 變更摘要](sql-api-sdk-dotnet-changefeed.md)
> * [.NET Core](sql-api-sdk-dotnet-core.md)
> * [Node.js](sql-api-sdk-node.md)
> * [非同步 Java](sql-api-sdk-async-java.md)
> * [Java](sql-api-sdk-java.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](https://docs.microsoft.com/rest/api/cosmos-db/)
> * [REST 資源提供者](https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/)
> * [SQL](https://msdn.microsoft.com/library/azure/dn782250.aspx)
> * [大量執行程式 - .NET](sql-api-sdk-bulk-executor-dot-net.md)
> * [大量執行程式 - Java](sql-api-sdk-bulk-executor-java.md)

<table>

<tr><td>**說明**</td><td>大量執行程式程式庫可讓用戶端應用程式在 Azure Cosmos DB 帳戶中執行大量作業。 大量執行程式程式庫提供 BulkImport、BulkUpdate 和 BulkDelete 命名空間。 BulkImport 模組能夠以最佳化方式大量內嵌文件，如此就能充分取用為集合佈建的輸送量。 BulkUpdate 模組能夠以修補程式的形式，大量更新 Azure Cosmos DB 容器中的現有資料。 BulkDelete 模組能夠以最佳化方式大量刪除文件，如此就能充分取用為集合佈建的輸送量。</td></tr>

<tr><td>**SDK 下載**</td><td>[NuGet](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.BulkExecutor/)</td></tr>

<tr><td>**GitHub 中的 BulkExecutor 程式庫**</td><td>[GitHub](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started)</td></tr>

<tr><td>**API 文件**</td><td>[.Net API 參考文件](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor?view=azure-dotnet)</td></tr>

<tr><td>**開始使用**</td><td>[開始使用大量執行程式程式庫 .NET SDK](bulk-executor-dot-net.md)</td></tr>

<tr><td>**目前支援的架構**</td><td><ul><li>[Microsoft.Azure.DocumentDB](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/)(version >= 2.0.0)</li><li>
[Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/)(version >= 9.0.1)
</li></ul></td></tr>
</table></br>

## <a name="release-notes"></a>版本資訊

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0

* 新增對 Azure Cosmos DB SQL API 帳戶之 BulkDelete 作業的支援。
* 新增對 Azure Cosmos DB MongoDB API 帳戶之 BulkImport 作業的支援。
* 將 DocumentDB.NET SDK 相依性提高至版本 2.0.0。 

### <a name="a-name102102"></a><a name="1.0.2"/>1.0.2

* 新增對 Azure Cosmos DB Gremlin API 帳戶之 BulkImport 作業的支援。

### <a name="a-name101101"></a><a name="1.0.1"/>1.0.1

* 修正 Azure Cosmos DB SQL API 帳戶之 BulkImport 作業的次要錯誤。

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0

* 新增對 Azure Cosmos DB SQL API 帳戶之 BulkImport 和 BulkUpdate 作業的支援。
