---
title: Azure Cosmos DB：SQL Java API、SDK 和資源 | Microsoft Docs
description: 了解所有 SQL Java API 和 SDK 相關資訊，包括發行日期、停用日期及 Azure Cosmos DB SQL Java SDK 每個版本之間所做的變更。
services: cosmos-db
author: rnagpal
manager: kfile
editor: cgronlun
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 06/29/2018
ms.author: rnagpal
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d7d00d6236b601d145be03e6086bec2d72faafcd
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37344932"
---
# <a name="azure-cosmos-db-java-sdk-for-sql-api-release-notes-and-resources"></a>Azure Cosmos DB Java SDK for SQL API：版本資訊與資源
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

SQL API Java SDK 支援同步作業。 如需非同步支援，請使用 [SQL API Async Java SDK](sql-api-sdk-async-java.md)。 

<table>

<tr><td>**SDK 下載**</td><td>[Maven](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.microsoft.azure%22%20AND%20a%3A%22azure-documentdb%22)</td></tr>

<tr><td>**API 文件**</td><td>[Java API 參考文件](/java/api/com.microsoft.azure.documentdb)</td></tr>

<tr><td>**參與 SDK**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-java/)</td></tr>

<tr><td>**開始使用**</td><td>[開始使用 Java SDK](sql-api-java-get-started.md)</td></tr>

<tr><td>**Web 應用程式教學課程**</td><td>[使用 Azure Cosmos DB 進行 Web 應用程式開發](sql-api-java-application.md)</td></tr>

<tr><td>**最低支援執行階段**</td><td>[JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)</td></tr>
</table></br>

## <a name="release-notes"></a>版本資訊

### <a name="a-name11621162"></a><a name="1.16.2"/>1.16.2
* 已新增串流容錯移轉支援。
* 已新增自訂中繼資料的支援。
* 已改善工作階段處理邏輯。
* 已修正分割區索引鍵範圍快取中的 Bug。
* 已修正直接模式中的 NPE Bug。

### <a name="a-name11611161"></a><a name="1.16.1"/>1.16.1
* 新增唯一索引的支援。
* 新增摘要選項中的接續權杖大小限制支援。
* 修正 Json 序列化的錯誤 (bug) (時間戳記)。
* 修正 Json 序列化的錯誤 (bug) (列舉)。
* 將 com.fasterxml.jackson.core:jackson-databind 的相依性升級至 2.9.5。

### <a name="a-name11601160"></a><a name="1.16.0"/>1.16.0
* 改善直接模式的連線共用。
* 增強跨資料分割查詢的非 orderby 預先擷取改良。
* 改善 UUID 產生作業。
* 改善工作階段一致性邏輯。
* 新增對 Multipolygon 的支援。
* 新增集合的分割區索引鍵範圍統計支援。
* 修正多重區域中的錯誤 (bug)。

### <a name="a-name11501150"></a><a name="1.15.0"/>1.15.0
* 以改善的 Json 序列化效能。
* 此 SDK 版本需要使用從 https://aka.ms/cosmosdb-emulator 下載之最新版本的 Azure Cosmos DB 模擬器。

### <a name="a-name11401140"></a><a name="1.14.0"/>1.14.0
* Microsoft 贊助者程式庫的內部變更。

### <a name="a-name11301130"></a><a name="1.13.0"/>1.13.0
* 修正讀取單一分割區金鑰範圍的問題。
* 修正會影響到名稱簡短的資料庫的 ResourceID 剖析問題。
* 修正分割區金鑰編碼產生的問題。

### <a name="a-name11201120"></a><a name="1.12.0"/>1.12.0
* 重大錯誤修正，要求在分割區分割期間處理。
* 已利用 Strong 和 BoundedStaleness 一致性層級修正問題。

### <a name="a-name11101110"></a><a name="1.11.0"/>1.11.0
* 已新增對名為 ConsistentPrefix 的新一致性層級的支援。
* 已修正工作階段模式中讀取集合的錯誤。

### <a name="a-name11001100"></a><a name="1.10.0"/>1.10.0
* 已啟用對低至 2,500 RU/秒且以 100 RU/秒遞增量進行調整之資料分割集合的支援。
* 已修正原生組件中的 Bug，這會在某些查詢中導致 NullRef 例外狀況。

### <a name="a-name196196"></a><a name="1.9.6"/>1.9.6
* 修正查詢引擎組態中可能對閘道模式中的查詢造成例外狀況的錯誤。
* 修正工作階段容器中可能在集合建立後立即對要求造成「找不到擁有者資源」例外狀況的幾個錯誤。

### <a name="a-name195195"></a><a name="1.9.5"/>1.9.5
* 新增彙總查詢的支援 (COUNT、MIN、MAX、SUM 和 AVG)。 請參閱[彙總支援](sql-api-sql-query.md#Aggregates)。
* 新增變更摘要的支援。
* 新增透過 RequestOptions.setPopulateQuotaInfo 收集配額資訊的支援。
* 新增透過 RequestOptions.setScriptLoggingEnabled 進行預存程序指令碼記錄的支援。
* 修正發生節流閥失敗時，DirectHttps 模式的查詢可能會停止回應的錯誤。
* 修正工作階段一致性模式中的錯誤。
* 修正當要求率過高時，可能在 HttpContext 中造成 NullReferenceException 的錯誤。
* 改善 DirectHttps 模式的效能。

### <a name="a-name194194"></a><a name="1.9.4"/>1.9.4
* 在 ConnectionPolicy.setProxy() API 加入簡單的用戶端執行個體 Proxy 支援。
* 加入的 DocumentClient.close() API 可正確關閉 DocumentClient 執行個體。
* 從原生組件 (而不是從閘道) 衍生的查詢計劃利用直接連線模式改善查詢效能。
* 設定 FAIL_ON_UNKNOWN_PROPERTIES = false，讓使用者不需要在其 POJO 定義 JsonIgnoreProperties。
* 重新建構記錄使用 SLF4J。
* 修正一致性讀取器中的其他幾個 Bug。

### <a name="a-name193193"></a><a name="1.9.3"/>1.9.3
* 修正連線管理中的 Bug，避免連線在直接連線模式中流失。
* 修正排名最前的查詢中可能會擲回 NullReferenece 例外狀況的 Bug。
* 透過減少內部快取的網路呼叫數來增進效能。
* 在 DocumentClientException 中的 ActivityID 和要求 URI 中加入狀態碼以協助疑難排解。

### <a name="a-name192192"></a><a name="1.9.2"/>1.9.2
* 修正連線管理中的問題以提供穩定性。

### <a name="a-name191191"></a><a name="1.9.1"/>1.9.1
* 新增對 BoundedStaleness 一致性層級的支援。
* 新增對已分割集合之 CRUD 作業的直接連線支援。
* 修正以 SQL 查詢資料庫時發生的錯誤。
* 修正工作階段權杖設定不正確之工作階段快取中的錯誤。

### <a name="a-name190190"></a><a name="1.9.0"/>1.9.0
* 新增對跨資料分割平行查詢的支援。
* 新增對已分割集合的 TOP/ORDER BY 查詢支援。
* 新增對強式一致性的支援。
* 新增對使用直接連線時之名稱型要求的支援。
* 修正以使得 ActivityId 在所有要求重試之間保持一致。
* 修正與以相同名稱重新建立集合時之工作階段快取相關的錯誤。
* 新增為異地隔離空間查詢指定集合索引編製原則時的 Polygon 和 LineString 資料類型。
* 修正適用於 Java 1.8 之 Java Doc 的錯誤。

### <a name="a-name181181"></a><a name="1.8.1"/>1.8.1
* 修正 PartitionKeyDefinitionMap 中將單一資料分割集合加入快取，而非提出額外擷取資料分割索引鍵要求的錯誤。
* 修正在提供不正確的資料分割索引鍵時不重試的錯誤。

### <a name="a-name180180"></a><a name="1.8.0"/>1.8.0
* 新增對多重區域資料庫帳戶的支援。
* 新增在已節流處理的要求上自動重試的支援，方法是提供選項來自訂重試次數上限和等待時間上限。  請參閱 RetryOptions 和 ConnectionPolicy.getRetryOptions()。
* 已淘汰以 IPartitionResolver 為基礎的自訂分割程式碼。 請針對更高的儲存體和輸送量使用分割集合。

### <a name="a-name171171"></a><a name="1.7.1"/>1.7.1
* 已新增速率限制的重試原則支援。  

### <a name="a-name170170"></a><a name="1.7.0"/>1.7.0
* 新加入文件的存留時間 (TTL) 支援。

### <a name="a-name160160"></a><a name="1.6.0"/>1.6.0
* 實作[已分割的集合](partition-data.md)和[使用者定義的效能等級](performance-levels.md)。

### <a name="a-name151151"></a><a name="1.5.1"/>1.5.1
* 修正 HashPartitionResolver 中的錯誤以產生與其他 SDK 一致的 little-endian 雜湊值。

### <a name="a-name150150"></a><a name="1.5.0"/>1.5.0
* 新增「雜湊和範圍」分割區解析程式來協助將應用程式跨多個分割區分區。

### <a name="a-name140140"></a><a name="1.4.0"/>1.4.0
* 實作 Upsert。 已新增新的 upsertXXX 方法以支援 Upsert 功能。
* 實作以識別碼為基礎的路由。 不需變更公用 API，所有變更皆為內部變更。

### <a name="a-name130130"></a><a name="1.3.0"/>1.3.0
* 已略過版本以配合其他 SDK 的版本號碼

### <a name="a-name120120"></a><a name="1.2.0"/>1.2.0
* 支援地理空間索引
* 驗證所有資源的識別碼屬性。 資源的識別碼不能包含 ?、/、#、\, 字元，或以空格作為結尾。
* 將新標頭「索引轉換進度」加至 ResourceResponse。

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0
* 實作 V2 索引原則

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* GA SDK

## <a name="release-and-retirement-dates"></a>發行和停用日期
Microsoft 至少會在停用 SDK 的 **12 個月** 之前提供通知，以供順利轉換至較新/支援的版本。

新的功能與最佳化項目只會新增至目前的 SDK，因此建議您一律盡早升級至最新的 SDK 版本。

服務將會拒絕使用已停用 SDK 的任何 Cosmos DB 要求。

> [!WARNING]
> 所有 **1.0.0** 版之前的 SQL SDK 版本都將於「2016 年 2 月 29 日」淘汰。
> 
> 

<br/>

| 版本 | 發行日期 | 停用日期 |
| --- | --- | --- |
| [1.16.2](#1.16.2) |2018 年 6 月 29 日 |--- |
| [1.16.1](#1.16.1) |2018 年 5 月 16 日 |--- |
| [1.16.0](#1.16.0) |2018 年 3 月 15 日 |--- |
| [1.15.0](#1.15.0) |2017 年 11 月 14 日 |--- |
| [1.14.0](#1.14.0) |2017 年 10 月 28 日 |--- |
| [1.13.0](#1.13.0) |2017 年 8 月 25 日 |--- |
| [1.12.0](#1.12.0) |2017 年 7 月 11 日 |--- |
| [1.11.0](#1.11.0) |2017 年 5 月 10 日 |--- |
| [1.10.0](#1.10.0) |2017 年 3 月 11 日 |--- |
| [1.9.6](#1.9.6) |2017 年 2 月 21 日 |--- |
| [1.9.5](#1.9.5) |2017 年 1 月 31 日 |--- |
| [1.9.4](#1.9.4) |2016 年 11 月24 日 |--- |
| [1.9.3](#1.9.3) |2016 年 10 月 30 日 |--- |
| [1.9.2](#1.9.2) |2016 年 10 月 28 日 |--- |
| [1.9.1](#1.9.1) |2016 年 10 月 26 日 |--- |
| [1.9.0](#1.9.0) |2016 年 10 月 3 日 |--- |
| [1.8.1](#1.8.1) |2016 年 6 月 30 日 |--- |
| [1.8.0](#1.8.0) |2016 年 6 月 14 日 |--- |
| [1.7.1](#1.7.1) |2016 年 4 月 30 日 |--- |
| [1.7.0](#1.7.0) |2016 年 4 月 27 日 |--- |
| [1.6.0](#1.6.0) |2016 年 3 月 29 日 |--- |
| [1.5.1](#1.5.1) |2015 年 12 月 31 日 |--- |
| [1.5.0](#1.5.0) |2015 年 12 月 4 日 |--- |
| [1.4.0](#1.4.0) |2015 年 10 月 5 日 |--- |
| [1.3.0](#1.3.0) |2015 年 10 月 5 日 |--- |
| [1.2.0](#1.2.0) |2015 年 8 月 5 日 |--- |
| [1.1.0](#1.1.0) |2015 年 7 月 9 日 |--- |
| [1.0.1](#1.0.1) |2015 年 5 月 12 日 |--- |
| [1.0.0](#1.0.0) |2015 年 4 月 7 日 |--- |
| 0.9.5-prelease |2015 年 3 月 9 日 |2016 年 2 月 29 日 |
| 0.9.4-prelease |2015 年 2 月 17 日 |2016 年 2 月 29 日 |
| 0.9.3-prelease |2015 年 1 月 13 日 |2016 年 2 月 29 日 |
| 0.9.2-prelease |2014 年 12 月 19 日 |2016 年 2 月 29 日 |
| 0.9.1-prelease |2014 年 12 月 19 日 |2016 年 2 月 29 日 |
| 0.9.0-prelease |2014 年 12 月 10日 |2016 年 2 月 29 日 |

## <a name="faq"></a>常見問題集
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另請參閱
若要深入了解 Cosmos DB，請參閱 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服務頁面。

