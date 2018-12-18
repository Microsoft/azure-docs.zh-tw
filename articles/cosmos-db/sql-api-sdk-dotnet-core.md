---
title: Azure Cosmos DB：SQL .NET Core API、SDK 和資源 | Microsoft Docs
description: 全面了解 SQL .NET Core API 和 SDK，包括發行日期、停用日期及 Azure Cosmos DB .NET Core SDK 每個版本之間的變更。
services: cosmos-db
author: rnagpal
manager: kfile
editor: cgronlun
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 03/22/2018
ms.author: rnagpal
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7000fc13a792f4f6def592e78dc759c8cf165881
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451913"
---
# <a name="azure-cosmos-db-net-core-sdk-for-sql-api-release-notes-and-resources"></a>Azure Cosmos DB .NET Core SDK for SQL API：版本資訊與資源
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

<table>

<tr><td>**SDK 下載**</td><td>[NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/)</td></tr>

<tr><td>**API 文件**</td><td>[.NET API 參考文件](/dotnet/api/overview/azure/cosmosdb?view=azure-dotnet)</td></tr>

<tr><td>**範例**</td><td>[.NET 程式碼範例](sql-api-dotnet-samples.md)</td></tr>

<tr><td>**開始使用**</td><td>[開始使用 Azure Cosmos DB .NET Core SDK 教學課程](sql-api-dotnetcore-get-started.md)</td></tr>

<tr><td>**Web 應用程式教學課程**</td><td>[使用 Azure Cosmos DB 進行 Web 應用程式開發](sql-api-dotnet-application.md)</td></tr>

<tr><td>**目前支援的架構**</td><td>[.NET Standard 1.6 和 .NET Standard 1.5](https://www.nuget.org/packages/NETStandard.Library)</td></tr>
</table></br>

## <a name="release-notes"></a>版本資訊

Azure Cosmos DB .NET Core SDK 有與最新版 [Azure Cosmos DB .NET SDK](sql-api-sdk-dotnet.md) 類似的功能。

### <a name="a-name211211"></a><a name="2.1.1"/>2.1.1

* 對多區域要求暫時性失敗新增更多復原能力。

### <a name="a-name210210"></a><a name="2.1.0"/>2.1.0

* 已新增多區域寫入支援。
* 透過 TOP 和 MaxBufferedItemCount 改善了跨分割區查詢效能。

### <a name="a-name200200"></a><a name="2.0.0"/>2.0.0

* 已新增要求取消支援。
* 已將 SetCurrentLocation 新增至 ConnectionPolicy，以根據區域自動填入所要的位置。
* 已修正具有 Min/Max 和篩選條件且不符合個別分割區上任何文件的跨分割區查詢所發生的錯誤。
* DocumentClient 方法現在會與 IDocumentClient 對應。
* 已更新直接 TCP 傳輸堆疊來減少所建立的連線數。
* 已新增對於非 Windows 用戶端的直接模式 TCP 支援。

### <a name="a-name200-preview2200-preview2"></a><a name="2.0.0-preview2"/>2.0.0-preview2

* 已新增要求取消支援。
* 已將 SetCurrentLocation 新增至 ConnectionPolicy，以根據區域自動填入所要的位置。
* 已修正具有 Min/Max 和篩選條件且不符合個別分割區上任何文件的跨分割區查詢所發生的錯誤。

### <a name="a-name200-preview200-preview"></a><a name="2.0.0-preview"/>2.0.0-preview

* DocumentClient 方法現在會與 IDocumentClient 對應。
* 已更新直接 TCP 傳輸堆疊來減少所建立的連線數。
* 已新增對於非 Windows 用戶端的直接模式 TCP 支援。

### <a name="a-name11001100"></a><a name="1.10.0"/>1.10.0

* 已將 ConsistencyLevel 屬性新增至 FeedOptions。
* 已將 JsonSerializerSettings 新增至 RequestOptions 和 FeedOptions。
* 已將 EnableReadRequestsFallback 新增至 ConnectionPolicy。

### <a name="a-name191191"></a><a name="1.9.1"/>1.9.1

* 修正跨分割區 Order By 查詢在邊角案例中的 KeyNotFoundException。
* 修正在 LINQ 查詢的 select 子句中，JsonPropery 屬性未被接受的錯誤 (bug)。

### <a name="a-name182182"></a><a name="1.8.2"/>1.8.2

* 修正會在競爭情形下發生的錯誤，其是在使用工作階段一致性層級時，導致間歇發生「Microsoft.Azure.Documents.NotFoundException: 讀取工作階段不適用於輸入工作階段權杖」錯誤。

### <a name="a-name181181"></a><a name="1.8.1"/>1.8.1

* 修正 FeedOptions.MaxItemCount = -1 擲回 System.ArithmeticException 的迴歸：頁面大小為負數。
* 在 QueryMetrics 加入了新的 ToString() 函式。
* 公開關於讀取收集的分割區統計資料。
* 在 ChangeFeedOptions 加入了 PartitionKey 屬性。
* 次要錯誤修正。

### <a name="a-name171171"></a><a name="1.7.1"/>1.7.1
 
 * 新增能力以使用 DocumentCollection 上的 UniqueKeyPolicy 屬性，指定文件的唯一索引。
 * 修正自訂 JsonSerializer 設定針對一些查詢和預存程序執行不被接受的錯誤。

### <a name="a-name170170"></a><a name="1.7.0"/>1.7.0
 
 * 在 API 參考文件、組件中的中繼資料資訊，以及 NuGet 封裝中，將商標從 Azure DocumentDB 變更為 Azure Cosmos DB。 
 * 將以直接連線模式傳送之請求的回應之診斷資訊和延遲加以公開。 屬性名稱是 ResourceResponse 類別上的 RequestDiagnosticsString 與 RequestLatency。
 * 此 SDK 版本需要使用從 https://aka.ms/cosmosdb-emulator 下載之最新版本的 Azure Cosmos DB 模擬器。
 
### <a name="a-name160160"></a><a name="1.6.0"/>1.6.0

* 新增的數個可靠性修正和改進。

### <a name="a-name151151"></a><a name="1.5.1"/>1.5.1 

* 與支援 [Microsoft.Azure.Graphs](https://docs.microsoft.com/azure/cosmos-db/graph-sdk-dotnet) 有關的內部變更

### <a name="a-name150150"></a><a name="1.5.0"/>1.5.0 

* 已新增將 PartitionKeyRangeId 做為 FeedOption 的支援，以供對特定分割區索引鍵範圍值限制查詢結果範圍。 
* 已新增將 StartTime 做為 ChangeFeedOption 的支援，以開始尋找該時間之後的變更。 

### <a name="a-name141141"></a><a name="1.4.1"/>1.4.1

*   修正 JsonSerializable 類別中可能會造成堆疊溢位例外狀況的問題。

### <a name="a-name140140"></a><a name="1.4.0"/>1.4.0

*   已新增對 [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient?view=azure-dotnet) 執行個體具現化時指定自訂 JsonSerializerSettings 的支援。

### <a name="a-name132132"></a><a name="1.3.2"/>1.3.2

*   支援 .NET Standard 1.5 作為其中一個目標架構。

### <a name="a-name131131"></a><a name="1.3.1"/>1.3.1

*   已修正當執行 Azure Cosmos DB 查詢時，受影響的 x64 機器不支援 SSE4 指令並且擲回 SEHException 的問題。

### <a name="a-name130130"></a><a name="1.3.0"/>1.3.0

*   已新增對名為 ConsistentPrefix 的新一致性層級的支援。
*   已新增對個別資料分割之查詢計量的支援。
*   已新增對限制查詢之接續權杖大小的支援。
*   已新增對失敗要求進行更詳細追蹤的支援。
*   SDK 中已有一些效能改進。

### <a name="a-name122122"></a><a name="1.2.2"/>1.2.2

* 修正將在彙總查詢之 FeedOptions 中提供的 PartitionKey 值忽略的問題。
* 修正在執行移動途中跨資料分割 Order By 查詢期間，進行資料分割管理的透明處理時發生的問題。

### <a name="a-name121121"></a><a name="1.2.1"/>1.2.1

* 已修正在 ASP.NET 內容內部使用時會在某些非同步 API 中造成死結的問題。

### <a name="a-name120120"></a><a name="1.2.0"/>1.2.0

* 修正來讓 SDK 在某些情況下能夠更有彈性地進行自動容錯移轉。

### <a name="a-name112112"></a><a name="1.1.2"/>1.1.2

* 修正偶爾會造成「WebException: 無法解析遠端名稱」的錯誤。
* 透過針對 ReadDocumentAsync API 新增多載，以新增直接讀取具類型文件的支援。

### <a name="a-name111111"></a><a name="1.1.1"/>1.1.1

* 新增彙總查詢的 LINQ 支援 (COUNT、MIN、MAX、SUM 和 AVG)。
* 針對使用事件處理常式所造成的 ConnectionPolicy 物件，修正記憶體流失問題。
* 修正使用 ETag 時 UpsertAttachmentAsync 無法運作的問題。
* 修正根據字串欄位排序時交叉資料分割排序依據查詢接續無法運作的問題。

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0

* 新增彙總查詢的支援 (COUNT、MIN、MAX、SUM 和 AVG)。 請參閱[彙總支援](sql-api-sql-query.md#Aggregates)。
* 已將分割區集合的最小輸送量從 10,100 RU/s 降低為 2500 RU/s。

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0

Azure Cosmos DB .NET Core SDK 可讓您建置快速、跨平台的 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 應用程式，以在 Windows、Mac 和 Linux 上執行。 最新版 Azure Cosmos DB .NET Core SDK 與 [Xamarin](https://www.xamarin.com) 完全相容，可用來建置以 iOS、Android 及 Mono (Linux) 為目標的應用程式。  

### <a name="a-name010-preview010-preview"></a><a name="0.1.0-preview"/>0.1.0-preview

Azure Cosmos DB .NET Core Preview SDK 可讓您建置快速、跨平台的 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 應用程式，以在 Windows、Mac 和 Linux 上執行。

Azure Cosmos DB .NET Core Preview SDK 有與最新版本 [Azure Cosmos DB .NET SDK](sql-api-sdk-dotnet.md) 的功能類似的功能，並且支援下列項目︰
* 所有[連接模式](performance-tips.md#networking)︰閘道器模式、直接 TCP 和 Direct HTTPs。 
* 所有[一致性層級](consistency-levels.md)︰強式、工作階段、限定過期和最終。
* [資料分割的集合](partition-data.md)。 
* [多重區域資料庫帳戶和異地複寫](distribute-data-globally.md)。

如果您有與此 SDK 相關的問題，請張貼至 [StackOverflow](http://stackoverflow.com/questions/tagged/azure-documentdb)，或是在 [Github 儲存機制](https://github.com/Azure/azure-documentdb-dotnet/issues)中提出問題。 

## <a name="release--retirement-dates"></a>發行和停用日期

| 版本 | 發行日期 | 停用日期 |
| --- | --- | --- |
| [2.1.1](#2.1.1) |2018 年 9 月 27 日 |--- |
| [2.1.0](#2.1.0) |2018 年 9 月 21 日 |--- |
| [2.0.0](#2.0.0) |2018 年 9 月 7 日 |--- |
| [1.9.1](#1.9.1) |2018 年 3 月 9 日 |--- |
| [1.8.2](#1.8.2) |2018 年 2 月 21 日 |--- |
| [1.8.1](#1.8.1) |2018 年 2 月 5 日 |--- |
| [1.7.1](#1.7.1) |2017 年 11 月 16 日 |--- |
| [1.7.0](#1.7.0) |2017 年 11 月 10 日 |--- |
| [1.6.0](#1.6.0) |2017 年 10 月 17 日 |--- |
| [1.5.1](#1.5.1) |2017 年 10 月 2 日 |--- |
| [1.5.0](#1.5.0) |2017 年 8 月 10 日 |--- | 
| [1.4.1](#1.4.1) |2017 年 8 月 7 日 |--- |
| [1.4.0](#1.4.0) |2017 年 8 月 2 日 |--- |
| [1.3.2](#1.3.2) |2017 年 6 月 12 日 |--- |
| [1.3.1](#1.3.1) |2017 年 5 月 23 日 |--- |
| [1.3.0](#1.3.0) |2017 年 5 月 10 日 |--- |
| [1.2.2](#1.2.2) |2017 年 4 月 19 日 |--- |
| [1.2.1](#1.2.1) |2017 年 3 月 29 日 |--- |
| [1.2.0](#1.2.0) |2017 年 3 月 25 日 |--- |
| [1.1.2](#1.1.2) |2017 年 3 月 20 日 |--- |
| [1.1.1](#1.1.1) |2017 年 3 月 14 日 |--- |
| [1.1.0](#1.1.0) |2017 年 2 月 16 日 |--- |
| [1.0.0](#1.0.0) |2016 年 12 月 21 日 |--- |
| [0.1.0-preview](#0.1.0-preview) |2016 年 11 月 15 日 |2016 年 12 月 31 日 |

## <a name="see-also"></a>另請參閱
若要深入了解 Cosmos DB，請參閱 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服務頁面。 

