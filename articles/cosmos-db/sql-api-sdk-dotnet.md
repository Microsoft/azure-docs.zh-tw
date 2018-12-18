---
title: Azure Cosmos DB：SQL .NET API、SDK 和資源 | Microsoft Docs
description: 全面了解 SQL .NET API 和 SDK，包括發行日期、停用日期及 Azure Cosmos DB .NET SDK 每個版本之間的變更。
services: cosmos-db
author: rnagpal
manager: kfile
editor: cgronlun
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 03/09/2018
ms.author: rnagpal
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b7de476c6b96856e9b783ed8182985846bb39cfa
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452389"
---
# <a name="azure-cosmos-db-net-sdk-for-sql-api-download-and-release-notes"></a>Azure Cosmos DB .NET SDK for SQL API：下載和版本資訊
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

<tr><td>**SDK 下載**</td><td>[NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/)</td></tr>

<tr><td>**API 文件**</td><td>[.NET API 參考文件](/dotnet/api/overview/azure/cosmosdb?view=azure-dotnet)</td></tr>

<tr><td>**範例**</td><td>[.NET 程式碼範例](sql-api-dotnet-samples.md)</td></tr>

<tr><td>**開始使用**</td><td>[開始使用 Azure Cosmos DB .NET SDK 教學課程](sql-api-get-started.md)</td></tr>

<tr><td>**Web 應用程式教學課程**</td><td>[使用 Azure Cosmos DB 進行 Web 應用程式開發](sql-api-dotnet-application.md)</td></tr>

<tr><td>**目前支援的架構**</td><td>[Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=30653)</td></tr>
</table></br>

## <a name="release-notes"></a>版本資訊
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

### <a name="a-name12201220"></a><a name="1.22.0"/>1.22.0

* 已將 ConsistencyLevel 屬性新增至 FeedOptions。
* 已將 JsonSerializerSettings 新增至 RequestOptions 和 FeedOptions。
* 已將 EnableReadRequestsFallback 新增至 ConnectionPolicy。

### <a name="a-name12111211"></a><a name="1.21.1"/>1.21.1

* 修正跨分割區 Order By 查詢在邊角案例中的 KeyNotFoundException。
* 修正在 LINQ 查詢的 select 子句中，JsonProperty 屬性未被接受的錯誤。

### <a name="a-name12021202"></a><a name="1.20.2"/>1.20.2

* 修正會在競爭情形下發生的錯誤，其是在使用工作階段一致性層級時，導致間歇發生「Microsoft.Azure.Documents.NotFoundException: 讀取工作階段不適用於輸入工作階段權杖」錯誤。

### <a name="a-name12011201"></a><a name="1.20.1"/>1.20.1

* 修正 FeedOptions.MaxItemCount = -1 擲回 System.ArithmeticException 的迴歸：頁面大小為負數。
* 在 QueryMetrics 加入了新的 ToString() 函式。
* 公開關於讀取收集的分割區統計資料。
* 在 ChangeFeedOptions 加入了 PartitionKey 屬性。
* 次要錯誤修正。

### <a name="a-name11911191"></a><a name="1.19.1"/>1.19.1

* 新增能力以使用 DocumentCollection 上的 UniqueKeyPolicy 屬性，指定文件的唯一索引。
* 修正自訂 JsonSerializer 設定針對一些查詢和預存程序執行不被接受的錯誤。

### <a name="a-name11901190"></a><a name="1.19.0"/>1.19.0

* 在 API 參考文件、組件中的中繼資料資訊，以及 NuGet 封裝中，將商標從 Azure DocumentDB 變更為 Azure Cosmos DB。 
* 將以直接連線模式傳送之請求的回應之診斷資訊和延遲加以公開。 屬性名稱是 ResourceResponse 類別上的 RequestDiagnosticsString 與 RequestLatency。
* 此 SDK 版本需要使用從 https://aka.ms/cosmosdb-emulator 下載之最新版本的 Azure Cosmos DB 模擬器。 

### <a name="a-name11811181"></a><a name="1.18.1"/>1.18.1 

* Microsoft 贊助者組件的內部變更。

### <a name="a-name11801180"></a><a name="1.18.0"/>1.18.0 

* 新增的數個可靠性修正和改進。

### <a name="a-name11701170"></a><a name="1.17.0"/>1.17.0 

* 已新增將 PartitionKeyRangeId 做為 FeedOption 的支援，以供對特定分割區索引鍵範圍值限制查詢結果範圍。 
* 已新增將 StartTime 做為 ChangeFeedOption 的支援，以開始尋找該時間之後的變更。

### <a name="a-name11611161"></a><a name="1.16.1"/>1.16.1
* 修正 JsonSerializable 類別中可能會造成堆疊溢位例外狀況的問題。

### <a name="a-name11601160"></a><a name="1.16.0"/>1.16.0
*   已修正需要重新編譯應用程式的問題，之所以有此問題，是因為在 DocumentClient 建構函式中導入 JsonSerializerSettings 來作為選擇性參數。
* 已將 DocumentClient 建構函式標記為過時，該建構函式需要 JsonSerializerSettings 來作為最後一個參數，以在傳遞 JsonSerializerSettings 參數時允許使用 ConnectionPolicy 和 ConsistencyLevel 參數的預設值。

### <a name="a-name11501150"></a><a name="1.15.0"/>1.15.0
*   已新增對 [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient?view=azure-dotnet) 具現化時指定自訂 JsonSerializerSettings 的支援。

### <a name="a-name11411141"></a><a name="1.14.1"/>1.14.1
*   已修正當執行 Azure Cosmos DB SQL 查詢時，受影響的 x64 機器不支援 SSE4 指令並且擲回 SEHException 的問題。

### <a name="a-name11401140"></a><a name="1.14.0"/>1.14.0
*   已新增對名為 ConsistentPrefix 的新一致性層級的支援。
*   已新增對個別資料分割之查詢計量的支援。
*   已新增對限制查詢之接續權杖大小的支援。
*   已新增對失敗要求進行更詳細追蹤的支援。
*   SDK 中已有一些效能改進。

### <a name="a-name11341134"></a><a name="1.13.4"/>1.13.4
* 在功能上與 1.13.3 相同。 已有一些內部變更。

### <a name="a-name11331133"></a><a name="1.13.3"/>1.13.3
* 在功能上與 1.13.2 相同。 已有一些內部變更。

### <a name="a-name11321132"></a><a name="1.13.2"/>1.13.2
* 修正將在彙總查詢之 FeedOptions 中提供的 PartitionKey 值忽略的問題。
* 修正在執行移動途中跨資料分割 Order By 查詢期間，進行資料分割管理的透明處理時發生的問題。

### <a name="a-name11311131"></a><a name="1.13.1"/>1.13.1
* 已修正在 ASP.NET 內容內部使用時會在某些非同步 API 中造成死結的問題。

### <a name="a-name11301130"></a><a name="1.13.0"/>1.13.0
* 修正來讓 SDK 在某些情況下能夠更有彈性地進行自動容錯移轉。

### <a name="a-name11221122"></a><a name="1.12.2"/>1.12.2
* 修正偶爾會造成「WebException: 無法解析遠端名稱」的錯誤。
* 透過針對 ReadDocumentAsync API 新增多載，以新增直接讀取具類型文件的支援。

### <a name="a-name11211121"></a><a name="1.12.1"/>1.12.1
* 新增彙總查詢的 LINQ 支援 (COUNT、MIN、MAX、SUM 和 AVG)。
* 針對使用事件處理常式所造成的 ConnectionPolicy 物件，修正記憶體流失問題。
* 修正使用 ETag 時 UpsertAttachmentAsync 無法運作的問題。
* 修正根據字串欄位排序時交叉資料分割排序依據查詢接續無法運作的問題。

### <a name="a-name11201120"></a><a name="1.12.0"/>1.12.0
* 新增彙總查詢的支援 (COUNT、MIN、MAX、SUM 和 AVG)。 請參閱[彙總支援](sql-api-sql-query.md#Aggregates)。
* 已將分割區集合的最小輸送量從 10,100 RU/s 降低為 2500 RU/s。

### <a name="a-name11141114"></a><a name="1.11.4"/>1.11.4
* 修正一個某些跨分割區查詢在 32 位元主機處理序中失敗的問題。
* 修正在閘道模式中未使用失敗要求的權杖來更新工作階段容器的問題。
* 修正縱向選取中具有 UDF 呼叫的查詢在某些案例中失敗的問題。
* 已修正用戶端效能，以提高要求的讀取和寫入輸送量。

### <a name="a-name11131113"></a><a name="1.11.3"/>1.11.3
* 修正未使用失敗要求的權杖來更新工作階段容器的問題。
* 新增 SDK 支援，以在 32 位元主機處理序中運作。 請注意，若使用跨分割區查詢，建議您使用 64 位元主機處理以獲得改進的效能。
* 改進關於 IN 運算式中涉及大量分割區索引鍵值之查詢案例的效能。
* 在設定 PopulateQuotaInfo 選項時，已在 ResourceResponse 中填入各種資源配額統計資料，以供文件集合讀取要求使用。

### <a name="a-name11111111"></a><a name="1.11.1"/>1.11.1
* 針對在 1.11.0 推出的 CreateDocumentCollectionIfNotExistsAsync API 小幅修正效能。
* 針對和高比例並行要求有關的案例，修正 SDK 中的效能。

### <a name="a-name11101110"></a><a name="1.11.0"/>1.11.0
* 支援新的類別和方法以在集合內處理文件的[變更摘要](change-feed.md)。
* 支援跨資料分割繼續查詢和改善跨資料分割查詢的一些效能。
* 新增 CreateDatabaseIfNotExistsAsync 和 CreateDocumentCollectionIfNotExistsAsync 方法。
* 針對下列系統函式支援 LINQ︰IsDefined、IsNull 和 IsPrimitive。
* 修正在搭配使用 Nuget 套件和具有 project.json 工具的專案時，自動將 Microsoft.Azure.Documents.ServiceInterop.dll 和 DocumentDB.Spatial.Sql.dll 組件的 bin 放入應用程式的 bin 資料夾的動作。
* 支援發出用戶端 ETW 追蹤，其可在偵錯案例中幫上忙。

### <a name="a-name11001100"></a><a name="1.10.0"/>1.10.0
* 新增分割集合的直接連線支援。
* 改進「界限-陳舊」一致性層級的效能。
* 新增為異地隔離空間查詢指定集合索引編製原則時的 Polygon 和 LineString 資料類型。
* 新增轉譯述詞時對 StringEnumConverter、IsoDateTimeConverter、UnixDateTimeConverter 的 LINQ 支援。
* 多項 SDK 錯誤修正。

### <a name="a-name195195"></a><a name="1.9.5"/>1.9.5
* 將造成下列 NotFoundException 的問題修正︰讀取工作階段不適用於輸入工作階段權杖。 查詢異地分散帳戶讀取區域時，在某些情況下發生此例外狀況。
* 公開 ResourceResponse 類別中的 ResponseStream 屬性，可讓您直接從回應存取基礎資料流。

### <a name="a-name194194"></a><a name="1.9.4"/>1.9.4
* 修改了 ResourceResponse、FeedResponse、StoredProcedureResponse 和 MediaResponse 類別以實作對應的公用介面，讓它們可以模擬用於測試導向部署 (TDD)。
* 修正了使用自訂 JsonSerializerSettings 物件來序列化資料時，會造成資料分割索引鍵標頭格式不正確的問題。

### <a name="a-name193193"></a><a name="1.9.3"/>1.9.3
* 已修正造成長時間執行的查詢失敗，並伴隨「授權權杖目前無效」錯誤的問題。
* 已修正會從跨資料分割的排名/排序依據查詢中移除原始 SqlParameterCollection 的問題。

### <a name="a-name192192"></a><a name="1.9.2"/>1.9.2
* 已新增分割集合的平行查詢支援。
* 已新增分割集合的跨資料分割 ORDER BY 和 TOP 查詢支援。
* 已修正使用 Azure Cosmos DB Nuget 套件參照來參考 Azure Cosmos DB 專案時，遺失所需的 DocumentDB.Spatial.Sql.dll 與 Microsoft.Azure.Documents.ServiceInterop.dll 參照。
* 已修正在 LINQ 中使用使用者定義的函式時，使用不同類型參數的能力。 
* 已修正廣域複寫帳戶中的的錯誤，此錯誤會使得 Upsert 呼叫導向讀取位置而非寫入位置。
* 已在遺失的 IDocumentClient 介面加入方法： 
  * UpsertAttachmentAsync 方法，會接受 mediaStream 及選項做為參數
  * CreateAttachmentAsync 方法，會接受選項做為參數
  * CreateOfferQuery 方法，會接受 querySpec 做為參數
* 已解除密封 IDocumentClient 介面中公開的公用類別。

### <a name="a-name180180"></a><a name="1.8.0"/>1.8.0
* 新增對多重區域資料庫帳戶的支援。
* 新增在已節流處理的要求上進行重試的支援。  使用者可以藉由設定 ConnectionPolicy.RetryOptions 屬性，來自訂重試次數和等待時間上限。
* 新增新的 IDocumentClient 介面，其中會定義所有 DocumenClient 屬性和方法的簽章。  做為此變更的一部分，也將建立 IQueryable 和 IOrderedQueryable 的擴充方法變更為 DocumentClient 類別本身上的方法。
* 新增組態選項，以針對指定的 Azure Cosmos DB 端點 URI 設定 ServicePoint.ConnectionLimit。  使用 ConnectionPolicy.MaxConnectionLimit 來變更預設值 (已設為 50)。
* 已淘汰 IPartitionResolver 及其實作。  現在不支援 IPartitionResolver。 建議您針對更高的儲存體和輸送量使用分割集合。

### <a name="a-name171171"></a><a name="1.7.1"/>1.7.1
* 根據會接受 RequestOptions 做為參數的 ExecuteStoredProcedureAsync 方法，新加入 URI 多載。

### <a name="a-name170170"></a><a name="1.7.0"/>1.7.0
* 新加入文件的存留時間 (TTL) 支援。

### <a name="a-name163163"></a><a name="1.6.3"/>1.6.3
* 修正 .NET SDK 的 Nuget 封裝錯誤，可供封裝為 Azure 雲端服務解決方案的一部分。

### <a name="a-name162162"></a><a name="1.6.2"/>1.6.2
* 實作[已分割的集合](partition-data.md)和[使用者定義的效能等級](performance-levels.md)。 

### <a name="a-name153153"></a><a name="1.5.3"/>1.5.3
* **[已修正]** 查詢 Azure Cosmos DB 端點時擲回：「System.Net.Http.HttpRequestException：將內容複製到資料流時發生錯誤」。

### <a name="a-name152152"></a><a name="1.5.2"/>1.5.2
* 擴充的 LINQ 支援包括新的分頁、條件式運算式以及範圍比較的運算子。
  * Take 運算子：可啟用 LINQ 中的 SELECT TOP 行為
  * CompareTo 運算子：可啟用字串範圍比較
  * 條件式 (?) 與聯合運算子 (??)
* **[已修正]** 將模型投射與 LINQ 查詢中的 Where-In 結合使用時發生 ArgumentOutOfRangeException。 [#81](https://github.com/Azure/azure-documentdb-dotnet/issues/81)

### <a name="a-name151151"></a><a name="1.5.1"/>1.5.1
* **[已修正]** 如果 Select 不是最後一個運算式，LINQ 提供者會假設沒有任何預測，並會產生不正確的 SELECT *。  [#58](https://github.com/Azure/azure-documentdb-dotnet/issues/58)

### <a name="a-name150150"></a><a name="1.5.0"/>1.5.0
* 實作 Upsert、新增 UpsertXXXAsync 方法
* 所有要求的效能改進
* LINQ 提供者支援字串的條件式、聯合和 CompareTo 方法
* **[已修正]** LINQ 提供者 --> 在 List 上實作 Contains 方法，以產生與 IEnumerable 和 Array 上相同的 SQL
* **[已修正]** 重試時，BackoffRetryUtility 會再次使用相同的 HttpRequestMessage，而非建立新的
* **[已過時]** UriFactory.CreateCollection --> 現應使用 UriFactory.CreateDocumentCollection

### <a name="a-name141141"></a><a name="1.4.1"/>1.4.1
* **[已修正]** 使用非 en 文化特性資訊時 (例如 nl-NL 等) 的當地語系化問題。 

### <a name="a-name140140"></a><a name="1.4.0"/>1.4.0
* 已新增以識別碼為基礎的路由
  * 新的 UriFactory 協助程式，可協助建構以識別碼為基礎的資源連結
  * DocumentClient 上新的多載可接受 URI
* LINQ 中新增用於地理空間的 IsValid() 和 isvaliddetailed ()
* 增強的 LINQ 提供者支援：
  * **算術** - Abs、Acos、Asin、Atan、Ceiling、Cos、Exp、Floor、Log、Log10、Pow、Round、Sign、Sin、Sqrt、Tan、Truncate
  * **字串** - Concat、Contains、EndsWith、IndexOf、Count、ToLower、TrimStart、Replace、Reverse、TrimEnd、StartsWith、SubString、ToUpper
  * **陣列** - Concat、Contains、Count
  * **IN** 運算子

### <a name="a-name130130"></a><a name="1.3.0"/>1.3.0
* 新增修改索引編製原則的支援。
  * DocumentClient 中新的 ReplaceDocumentCollectionAsync 方法
  * ResourceResponse 中新的 IndexTransformationProgress 屬性<T>可追蹤索引原則變更的百分比進度
  * DocumentCollection.IndexingPolicy 現在可變動
* 新增空間索引編製和查詢的支援。
  * 新的 Microsoft.Azure.Documents.Spatial 命名空間，可序列化/還原序列化空間類型，例如 Point 和 Polygon
  * 新的 SpatialIndex 類別，可對儲存在 Cosmos DB 中的 GeoJSON 資料編製索引
* **[已修正]**：從 LINQ 運算式產生的 SQL 查詢不正確 [#38](https://github.com/Azure/azure-documentdb-net/issues/38) \(英文\)。

### <a name="a-name120120"></a><a name="1.2.0"/>1.2.0
* 已新增對 Newtonsoft.Json v5.0.7 的相依性。
* 變更為支援 Order By：
  
  * LINQ 提供者支援 OrderBy() 或 OrderByDescending()
  * IndexingPolicy 支援 Order By 
    
    **可能使得舊程式碼無法運作的變更** 
    
    如果現有程式碼以自訂索引原則來佈建集合，則需要更新現有的程式碼，才能支援新的 IndexingPolicy 類別。 如果您沒有自訂的索引原則，這個變更不會影響到您。

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0
* 使用新的 HashPartitionResolver 和 RangePartitionResolver 類別及 IPartitionResolver，以新增對資料分割的支援。
* 已新增 DataContract 序列化。
* 已新增 LINQ 提供者中的 GUID 支援。
* 已新增 LINQ 中的 UDF 支援。

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* GA SDK

## <a name="release--retirement-dates"></a>發行和停用日期
Microsoft 至少會在停用 SDK 的 **12 個月** 之前提供通知，以供順利轉換至較新/支援的版本。

新的功能與最佳化項目只會新增至目前的 SDK，因此建議您一律盡早升級至最新的 SDK 版本。 

服務會拒絕使用已停用 SDK 的任何 Azure Cosmos DB 要求。

<br/>

| 版本 | 發行日期 | 停用日期 |
| --- | --- | --- |
| [2.1.1](#2.1.1) |2018 年 9 月 27 日 |--- |
| [2.1.0](#2.1.0) |2018 年 9 月 21 日 |--- |
| [2.0.0](#2.0.0) |2018 年 9 月 7 日 |--- |
| [1.22.0](#1.22.0) |2018 年 4 月 19 日 |--- |
| [1.21.1](#1.20.1) |2018 年 3 月 9 日 |--- |
| [1.20.2](#1.20.1) |2018 年 2 月 21 日 |--- |
| [1.20.1](#1.20.1) |2018 年 2 月 5 日 |--- |
| [1.19.1](#1.19.1) |2017 年 11 月 16 日 |--- |
| [1.19.0](#1.19.0) |2017 年 11 月 10 日 |--- |
| [1.18.1](#1.18.1) |2017 年 11 月 7 日 |--- |
| [1.18.0](#1.18.0) |2017 年 10 月 17 日 |--- |
| [1.17.0](#1.17.0) |2017 年 8 月 10 日 |--- |
| [1.16.1](#1.16.1) |2017 年 8 月 7 日 |--- |
| [1.16.0](#1.16.0) |2017 年 8 月 2 日 |--- |
| [1.15.0](#1.15.0) |2017 年 6 月 30 日 |--- |
| [1.14.1](#1.14.1) |2017 年 5 月 23 日 |--- |
| [1.14.0](#1.14.0) |2017 年 5 月 10 日 |--- |
| [1.13.4](#1.13.4) |2017 年 5 月 09 日 |--- |
| [1.13.3](#1.13.3) |2017 年 5 月 06 日 |--- |
| [1.13.2](#1.13.2) |2017 年 4 月 19 日 |--- |
| [1.13.1](#1.13.1) |2017 年 3 月 29 日 |--- |
| [1.13.0](#1.13.0) |2017 年 3 月 24 日 |--- |
| [1.12.2](#1.12.2) |2017 年 3 月 20 日 |--- |
| [1.12.1](#1.12.1) |2017 年 3 月 14 日 |--- |
| [1.12.0](#1.12.0) |2017 年 2 月 15 日 |--- |
| [1.11.4](#1.11.4) |2017 年 2 月 6 日 |--- |
| [1.11.3](#1.11.3) |2017 年 1 月 26 日 |--- |
| [1.11.1](#1.11.1) |2016 年 12 月 21 日 |--- |
| [1.11.0](#1.11.0) |2016 年 12 月 8 日 |--- |
| [1.10.0](#1.10.0) |2016 年 9 月 27 日 |--- |
| [1.9.5](#1.9.5) |2016 年 9 月 1 日 |--- |
| [1.9.4](#1.9.4) |2016 年 8 月 24 日 |--- |
| [1.9.3](#1.9.3) |2016 年 8 月 15 日 |--- |
| [1.9.2](#1.9.2) |2016 年 7 月 23 日 |--- |
| [1.8.0](#1.8.0) |2016 年 6 月 14 日 |--- |
| [1.7.1](#1.7.1) |2016 年 5 月 6 日 |--- |
| [1.7.0](#1.7.0) |2016 年 4 月 26 日 |--- |
| [1.6.3](#1.6.3) |2016 年 4 月 8 日 |--- |
| [1.6.2](#1.6.2) |2016 年 3 月 29 日 |--- |
| [1.5.3](#1.5.3) |2016 年 2 月 19 日 |--- |
| [1.5.2](#1.5.2) |2015 年 12 月 14 日 |--- |
| [1.5.1](#1.5.1) |2015 年 11 月 23 日 |--- |
| [1.5.0](#1.5.0) |2015 年 10 月 5 日 |--- |
| [1.4.1](#1.4.1) |2015 年 8 月 25 日 |--- |
| [1.4.0](#1.4.0) |2015 年 8 月 13 日 |--- |
| [1.3.0](#1.3.0) |2015 年 8 月 5 日 |--- |
| [1.2.0](#1.2.0) |2015 年 7 月 6 日 |--- |
| [1.1.0](#1.1.0) |2015 年 4 月 30 日 |--- |
| [1.0.0](#1.0.0) |2015 年 4 月 8 日 |--- |


## <a name="faq"></a>常見問題集
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另請參閱
若要深入了解 Cosmos DB，請參閱 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服務頁面。 

