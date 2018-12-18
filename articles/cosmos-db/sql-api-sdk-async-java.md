---
title: Azure Cosmos DB：SQL Async Java API、SDK 和資源 | Microsoft Docs
description: 了解所有 SQL Async Java API 和 SDK 相關資訊，包括發行日期、停用日期及 Azure Cosmos DB SQL Async Java SDK 每個版本之間所做的變更。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 09/05/2018
ms.author: sngun
ms.openlocfilehash: e90c5640e571aaf28e184e9439f6228e3a5bbc6b
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44023557"
---
# <a name="azure-cosmos-db-async-java-sdk-for-sql-api-release-notes-and-resources"></a>Azure Cosmos DB Async Java SDK for SQL API：版本資訊與資源
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
> * [BulkExecutor - .NET](sql-api-sdk-bulk-executor-dot-net.md)
> * [BulkExecutor - Java](sql-api-sdk-bulk-executor-java.md)

SQL API Async Java SDK 與 SQL API Java SDK 的不同之處在於會提供非同步作業，並且支援 [Netty 程式庫](http://netty.io/)。 預先存在的 [SQL API Java SDK](sql-api-sdk-java.md) 不支援非同步作業。 

<table>

<tr><td>**SDK 下載**</td><td>[Maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb)</td></tr>

<tr><td>**API 文件**</td><td>[Java API 參考文件](https://docs.microsoft.com/java/api/com.microsoft.azure.cosmosdb.rx._async_document_client?view=azure-java-stable)</td></tr>

<tr><td>**參與 SDK**</td><td>[GitHub](https://github.com/Azure/azure-cosmosdb-java)</td></tr>

<tr><td>**開始使用**</td><td>[開始使用 Async Java SDK](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-async-java-getting-started)</td></tr>

<tr><td>**程式碼範例**</td><td>[Github](https://github.com/Azure/azure-cosmosdb-java#usage-code-sample)</td></tr>

<tr><td>**效能秘訣**</td><td>[Github 讀我檔案](https://github.com/Azure/azure-cosmosdb-java#guide-for-prod)</td></tr>

<tr><td>**最低支援執行階段**</td><td>[JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)</td></tr>
</table></br>

## <a name="release-notes"></a>版本資訊

### <a name="a-name210210"></a><a name="2.1.0"/>2.1.0
* 已新增 Proxy 的支援。
* 已新增資源權杖授權的支援。
* 修正處理大型分割區金鑰時發生的錯誤 ([github #63](https://github.com/Azure/azure-cosmosdb-java/issues/63))。
* 已改善文件。
* SDK 已重新建構為更精細的模組。

### <a name="a-name201201"></a><a name="2.0.1"/>2.0.1
* 已修正非英文地區設定的錯誤 (bug) ([github #51](https://github.com/Azure/azure-cosmosdb-java/issues/51))。
* 已在衝突資源中新增協助程式方法。

### <a name="a-name200200"></a><a name="2.0.0"/>2.0.0
* 因為效能理由及授權 ([github #29](https://github.com/Azure/azure-cosmosdb-java/issues/29))，以 jackson 取代 org.json 相依性。
* 已移除被取代的 OfferV2 類別。
* 已將存取子方法新增至輸送量內容的 Offer 類別。
* 文件/資源中任何傳回 org.json 類型的方法已變更為傳回 jackson 物件類型。
* 擴充 JsonSerializable 類別的 getObject(.) 方法已變更為傳回 jackson ObjectNode 類型。
* getCollection(.) 方法已變更為傳回 ObjectNode 的集合。
* 已使用 org.json.JSONObject 引數移除 JsonSerializable 子類別的建構函式。
* JsonSerializable.toJson (SerializationFormattingPolicy.Indented) 現在使用兩個空格進行縮排。
  
### <a name="a-name102102"></a><a name="1.0.2"/>1.0.2
* 新增唯一索引原則的支援。
* 新增摘要選項中的回應接續權杖大小限制支援。
* 新增跨資料分割查詢中的分割區分割支援。
* 修正 Json 時間戳記序列化的錯誤 (bug) ([github #32](https://github.com/Azure/azure-cosmosdb-java/issues/32))。
* 修正 Json 列舉序列化的錯誤 (bug)。
* 修正管理 2 MB 大小的文件時，所出現的錯誤 (bug) ([github #33](https://github.com/Azure/azure-cosmosdb-java/issues/33))。
* 因為錯誤 (bug) 而將 com.fasterxml.jackson.core:jackson-databind 相依性升級為 2.9.5 ([jackson-databind: github #1599](https://github.com/FasterXML/jackson-databind/issues/1599))
* 因為錯誤 (bug) 而將 rxjava-extras 上的相依性升級為 0.8.0.17 ([rxjava-extras: github #30](https://github.com/davidmoten/rxjava-extras/issues/30))。
* 將 POM 檔案中的中繼資料描述更新為與其餘文件一起內嵌。
* 語法改進 ([github #41](https://github.com/Azure/azure-cosmosdb-java/issues/41))、([github #40](https://github.com/Azure/azure-cosmosdb-java/issues/40))。

### <a name="a-name101101"></a><a name="1.0.1"/>1.0.1
* 已在查詢中新增背部壓力支援。
* 已在查詢中新增分割區索引鍵範圍識別碼的支援。
* 進行修正以在要求標頭中允許較大的接續權杖 (bugfix github #24)。
* Netty 相依性已升級至 4.1.22.Final，以確保 JVM 會在主執行緒完成後關閉。
* 進行修正以避免在讀取主要資源時傳遞工作階段權杖。
* 已新增更多範例。
* 已新增更多效能評定案例。
* 已修正 Java 標頭檔以產生適當的 java 文件。

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* 在閘道模式中使用 [Netty 程式庫](http://netty.io/)，端對端支援非封鎖 IO 的 GA SDK。 

## <a name="release-and-retirement-dates"></a>發行和停用日期
Microsoft 至少會在停用 SDK 的 **12 個月** 之前提供通知，以供順利轉換至較新/支援的版本。

新的特性和功能與最佳化僅新增至目前的 SDK。 因此，建議您務必盡早升級至最新的 SDK 版本。

服務將會拒絕使用已停用 SDK 的任何 Cosmos DB 要求。

<br/>

| 版本 | 發行日期 | 停用日期 |
| --- | --- | --- |
| [2.1.0](#2.1.0) |2018 年 9 月 5 日|--- |
| [2.0.1](#2.0.1) |2018 年 8 月 16 日|--- |
| [2.0.0](#2.0.0) |2018 年 6 月 20 日|--- |
| [1.0.2](#1.0.2) |2018 年 5 月 18 日|--- |
| [1.0.1](#1.0.1) |2018 年 4 月 20 日|--- |
| [1.0.0](#1.0.0) |2018 年 2 月 27 日|--- |

## <a name="faq"></a>常見問題集
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另請參閱
若要深入了解 Cosmos DB，請參閱 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服務頁面。

