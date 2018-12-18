---
title: 監視 Azure 搜尋服務中的使用量和統計資料 | Microsoft Docs
description: 追蹤 Azure 搜尋服務 (Microsoft Azure 上之託管的雲端搜尋服務) 的資源耗用量和索引大小。
author: HeidiSteen
manager: cgronlun
tags: azure-portal
services: search
ms.service: search
ms.devlang: na
ms.topic: conceptual
ms.date: 11/09/2017
ms.author: heidist
ms.openlocfilehash: 286569eef8e17909ecab017b67b0ffc044a4bfe4
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31795104"
---
# <a name="monitoring-an-azure-search-service"></a>監視 Azure 搜尋服務

Azure 搜尋服務提供各種資源，可追蹤搜尋服務的使用情形和效能。 它可讓您存取 Power BI 上的計量、記錄、索引統計資料和延伸監視功能。 本文說明如何啟用不同的監視策略，以及如何解譯產生的資料。

## <a name="azure-search-metrics"></a>Azure 搜尋服務計量
計量可讓您近乎即時地掌握您的搜尋服務，並可供每個服務使用，不需要額外設定。 它們可讓您追蹤最多 30 天的服務效能。

Azure 搜尋服務會收集三個不同計量的資料︰

* 搜尋延遲︰搜尋服務處理搜尋查詢所需的時間 (每分鐘彙總一次)。
* 每秒搜尋查詢 (QPS)︰每秒接收的搜尋查詢數 (每分鐘彙總一次)。
* 節流的搜尋查詢百分比︰已節流處理的搜尋查詢百分比 (每分鐘彙總一次)。

![QPS 活動的螢幕擷取畫面][1]

### <a name="set-up-alerts"></a>設定警示
您可以從計量詳細資料頁面中設定警示，以便在計量超出已定義的閾值時，觸發電子郵件通知或自動化的動作。

如需計量的詳細資訊，請查看 Azure 監視器上的完整文件。  

## <a name="how-to-track-resource-usage"></a>如何追蹤資源使用狀況
追蹤索引和文件大小的成長，可在達到您為服務所設定的上限之前，幫助您先主動調整容量。 您可以在入口網站或使用 REST API 以程式設計方式執行這項操作。

### <a name="using-the-portal"></a>使用入口網站

若要監視資源使用量，請在[入口網站](https://portal.azure.com)中檢視服務的計數和統計資料。

1. 登入[入口網站](https://portal.azure.com)。
2. 開啟 Azure 搜尋服務的服務儀表板。 可在首頁上找到並排的服務，或是您也可以使用 JumpBar 上的 [瀏覽] 來瀏覽服務。

[使用量] 部分包含計量，可告知您目前正在使用可用資源的哪些部分。 如需每個服務的索引、文件和儲存體限制資訊，請參閱[服務限制](search-limits-quotas-capacity.md)。

  ![使用量圖格][2]

> [!NOTE]
> 上面的螢幕擷取畫面適用於「免費」服務 (其最多各有一個複本和資料分割)，而且只裝載 3 個索引、10,000 份文件或 50 MB 的資料 (看何者先達到)。 在「基本」或「標準」層所建立的服務會有較大的服務限制。 如需選擇層的詳細資訊，請參閱[選擇層或 SKU](search-sku-tier.md)。
>
>

### <a name="using-the-rest-api"></a>使用 REST API
Azure 搜尋服務 REST API 和 .NET SDK 都提供以程式設計方式存取服務度量。  如果您使用[索引子](https://msdn.microsoft.com/library/azure/dn946891.aspx)從 Azure SQL Database 或 Azure Cosmos DB 載入索引，則有其他 API 可取得您所需的數字。

* [取得索引統計資料](/rest/api/searchservice/get-index-statistics)
* [文件計數](/rest/api/searchservice/count-documents)
* [取得索引子狀態](/rest/api/searchservice/get-indexer-status)

## <a name="how-to-export-logs-and-metrics"></a>如何匯出記錄和計量

您可以匯出您服務的作業記錄，以及上一節中所述之計量的未經處理資料。 作業記錄可讓您知道服務的使用方式，並且可以在將資料複製到儲存體帳戶時從 Power BI 取用。 Azure 搜尋服務會針對此用途提供監視 Power BI 內容套件。


### <a name="enabling-monitoring"></a>啟用監視
在 [Azure 入口網站](http://portal.azure.com)的 [啟用監視] 選項下將您的 Azure 搜尋服務開啟。

選擇您想要匯出的資料︰記錄、計量或兩者。 您可以將它複製到儲存體帳戶、將它傳送到事件中樞，或將它匯出成 Log Analytics。

![如何在入口網站中啟用監視][3]

若要使用 PowerShell 或 Azure CLI 來啟用，請參閱[這裡](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs#how-to-enable-collection-of-diagnostic-logs)的文件。

### <a name="logs-and-metrics-schemas"></a>記錄和計量的結構描述
當資料複製到儲存體帳戶時，資料會格式化為 JSON，且會放置在兩個容器中︰

* insights-logs-operationlogs：適用於搜尋流量記錄
* insights-metrics-pt1m：適用於計量

每個容器每小時會有一個 Blob。

範例路徑： `resourceId=/subscriptions/<subscriptionID>/resourcegroups/<resourceGroupName>/providers/microsoft.search/searchservices/<searchServiceName>/y=2015/m=12/d=25/h=01/m=00/name=PT1H.json`

#### <a name="log-schema"></a>記錄檔結構描述
記錄檔 Blob 包含您的搜尋服務流量記錄檔。
每個 Blob 會一個名為 **記錄** 的根物件，其中包含記錄檔物件的陣列。
在每個 blob 中，同一小時內發生的所有作業都有記錄。

| Name | 類型 | 範例 | 注意 |
| --- | --- | --- | --- |
| 分析 |Datetime |"2015-12-07T00:00:43.6872559Z" |作業的時間戳記 |
| ResourceId |字串 |"/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/> MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE" |您的 ResourceId |
| operationName |字串 |"Query.Search" |作業的名稱 |
| operationVersion |字串 |"2015-02-28" |使用的 api-version |
| category |字串 |"OperationLogs" |常數 |
| resultType |字串 |"Success" |可能的值：Success 或 Failure |
| resultSignature |int |200 |HTTP 結果碼 |
| durationMS |int |50 |作業的持續時間 (以毫秒為單位) |
| properties |物件 |請參閱下表 |包含作業特定資料的物件 |

**屬性結構描述**
| Name | 類型 | 範例 | 注意 |
| --- | --- | --- | --- |
| 說明 |字串 |"GET /indexes('content')/docs" |作業的端點 |
| 查詢 |字串 |"?search=AzureSearch&$count=true&api-version=2015-02-28" |查詢參數 |
| 文件 |int |42 |處理的文件數目 |
| IndexName |字串 |"testindex" |與作業相關聯的索引名稱 |

#### <a name="metrics-schema"></a>度量結構描述
| Name | 類型 | 範例 | 注意 |
| --- | --- | --- | --- |
| ResourceId |字串 |"/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/>MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE" |您的資源識別碼 |
| metricName |字串 |"Latency" |度量的名稱 |
| 分析 |Datetime |"2015-12-07T00:00:43.6872559Z" |作業的時間戳記 |
| average |int |64 |度量時間間隔中原始範例的平均值 |
| minimum |int |37 |度量時間間隔中原始範例的最小值 |
| maximum |int |78 |度量時間間隔中原始範例的最大值 |
| total |int |258 |度量時間間隔中原始範例的總和值 |
| count |int |4 |用來產生度量的原始樣本數 |
| timegrain |字串 |"PT1M" |採用 ISO 8601 的度量時間粒紋 |

每隔一分鐘就會回報所有計量。 每個度量會顯示每分鐘的最小值、最大值和平均值。

以 SearchQueriesPerSecond 度量來說，最小值是該分鐘內已註冊的每秒搜尋查詢次數最低值。 最大值依此類推。 平均值是一分鐘的彙總。
假設一分鐘內有這種情況：有 1 秒出現極高的負載，這是 SearchQueriesPerSecond 的最大值，接著有 58 秒的平均負載，最後的 1 秒只有一個查詢，而這會是最小值。

對於 ThrottledSearchQueriesPercentage，最小值、最大值、平均值和總計全是相同的值：在一分鐘內的搜尋查詢總數中，已節流處理的搜尋查詢百分比。

## <a name="analyzing-your-data-with-power-bi"></a>使用 Power BI 來分析資料

我們建議您使用 [Power BI](https://powerbi.microsoft.com) 來探索和視覺化資料。 您可以輕鬆地連線到您的 Azure 儲存體帳戶，並快速開始分析您的資料。

Azure 搜尋服務提供 [Power BI 內容套件](https://app.powerbi.com/getdata/services/azure-search)，可讓您利用預先定義的圖表和資料表來監視及了解您的搜尋流量。 它包含一組 Power BI 報告，且該報告會自動連線到您的資料，並以視覺化的方式提供您搜尋服務相關的深入分析。 如需詳細資訊，請參閱[內容套件說明網頁](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-search/)。

![Azure 搜尋服務的 Power BI 儀表板][4]

## <a name="next-steps"></a>後續步驟
如需如何平衡現有服務之資料分割與複本配置的指引，請檢閱[調整複本和資料分割](search-limits-quotas-capacity.md)。

如需服務管理的詳細資訊，請瀏覽[在 Microsoft Azure 上管理搜尋服務](search-manage.md)，或[效能和最佳化](search-performance-optimization.md)以取得微調指引。

深入了解如何建立令人讚嘆的報告。 如需詳細資訊，請參閱[開始使用 Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-getting-started/)

<!--Image references-->
[1]: ./media/search-monitor-usage/AzSearch-Monitor-BarChart.PNG
[2]: ./media/search-monitor-usage/AzureSearch-Monitor1.PNG
[3]: ./media/search-monitor-usage/AzureSearch-Enable-Monitoring.PNG
[4]: ./media/search-monitor-usage/AzureSearch-PowerBI-Dashboard.png
