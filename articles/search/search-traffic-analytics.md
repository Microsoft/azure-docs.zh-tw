---
title: Azure 搜尋服務的搜尋流量分析 | Microsoft Docs
description: 啟用 Azure 搜尋服務 (Microsoft Azure 上裝載的雲端搜尋服務) 的搜尋流量分析，來深入分析您的使用者與資料。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 04/05/2017
ms.author: heidist
ms.openlocfilehash: 4b40e8c9f681b7489c0ab2ffe7b369cc869c73e2
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33777254"
---
# <a name="what-is-search-traffic-analytics"></a>什麼是搜尋流量分析
搜尋流量分析是一種模式，可用來實作搜尋服務的意見反應管道。 這個模式描述必要的資料，以及如何使用監視多平台服務中的業界領導者 Application Insights 來加以收集。

搜尋流量分析可讓您掌握您的搜尋服務，並深入分析您的使用者及其行為。 擁有使用者選擇項目的相關資料，您所做的決定就能夠進一步改善搜尋體驗，並且在結果不如預期時予以中斷。

Azure 搜尋服務所提供的遙測解決方案可整合 Azure Application Insights 和 Power BI，以提供深入的監視和追蹤。 因為只能透過 API 與 Azure 搜尋服務進行互動，必須遵循這個頁面中的指示，由開發人員使用搜尋將遙測進行實作。

## <a name="identify-the-relevant-search-data"></a>識別相關的搜尋資料

如需取得實用的搜尋計量，就必須從搜尋應用程式的使用者記錄一些訊號。 這些訊號代表使用者感興趣且認為與其需求相關的內容。

搜尋流量分析需要兩個訊號︰

1. 使用者產生的搜尋事件︰僅使用者所起始的搜尋查詢才有趣。 用來填入 Facet、其他內容或任何內部資訊的搜尋要求並不重要，而且還會扭曲和偏差結果。

2. 使用者產生的點選事件︰透過此文件中的點選，我們意指選取搜尋查詢所傳回之特定搜尋結果的使用者。 點選通常表示文件為特定搜尋查詢的相關結果。

使用相互關聯的識別碼將搜尋和點選事件進行連結，就可以分析您應用程式上的使用者行為。 僅使用搜尋流量記錄並無法取得這些搜尋深入分析。

## <a name="how-to-implement-search-traffic-analytics"></a>如何實作搜尋流量分析

必須從搜尋應用程式收集前一節所述的訊號，因為使用者會與其進行互動。 Application Insights 是可延伸的監視解決方案，適用於多個平台，並具有彈性的檢測選項。 使用 Application Insights 可讓您充分利用 Azure 搜尋服務所建立的 Power BI 搜尋報告，使資料分析更輕鬆。

在 Azure 搜尋服務的[入口網站](https://portal.azure.com)頁面中，搜尋流量分析刀鋒視窗包含了遵循此遙測模式的小祕技。 您也可以選取或建立 Application Insights 資源，並查看必要的資料，這些全都放在同一個位置。

![搜尋流量分析指示][1]

### <a name="1-select-an-application-insights-resource"></a>1.選取 Application Insights 資源

如果您還沒有 Application Insights 資源，則必須選取 Application Insights 資源後，才可以加以使用或建立。 您可以使用已在使用中的資源，才可記錄必要的自訂事件。

在建立新的 Application Insights 資源時，所有應用程式類型都適用於這種情況。 選取最適合您所使用平台的 Application Insights 資源。

您需要檢測金鑰來建立您應用程式的遙測用戶端。 您可以從 Application Insights 入口網站儀表板取得，或可以在搜尋流量分析頁面上，選取您想要使用的執行個體來加以取得。

### <a name="2-instrument-your-application"></a>2.檢測應用程式

在此階段中，您可以使用上述步驟中建立的 Application Insights 資源，將自己的搜尋應用程式進行檢測。 這個程序有四個步驟︰

**I.建立用戶端遙測** 這是將事件傳送至 Application Insights 資源的物件。

*C#*

    private TelemetryClient telemetryClient = new TelemetryClient();
    telemetryClient.InstrumentationKey = "<YOUR INSTRUMENTATION KEY>";

*JavaScript*

    <script type="text/javascript">var appInsights=window.appInsights||function(config){function r(config){t[config]=function(){var i=arguments;t.queue.push(function(){t[config].apply(t,i)})}}var t={config:config},u=document,e=window,o="script",s=u.createElement(o),i,f;s.src=config.url||"//az416426.vo.msecnd.net/scripts/a/ai.0.js";u.getElementsByTagName(o)[0].parentNode.appendChild(s);try{t.cookie=u.cookie}catch(h){}for(t.queue=[],i=["Event","Exception","Metric","PageView","Trace","Dependency"];i.length;)r("track"+i.pop());return r("setAuthenticatedUserContext"),r("clearAuthenticatedUserContext"),config.disableExceptionTracking||(i="onerror",r("_"+i),f=e[i],e[i]=function(config,r,u,e,o){var s=f&&f(config,r,u,e,o);return s!==!0&&t["_"+i](config,r,u,e,o),s}),t}
    ({
    instrumentationKey: "<YOUR INSTRUMENTATION KEY>"
    });
    window.appInsights=appInsights;
    </script>

如需了解其他語言和平台，請參閱完整的[清單](https://docs.microsoft.com/azure/application-insights/app-insights-platforms)。

**II.要求相互關聯的搜尋識別碼** 若要透過點選將搜尋要求相互關聯，則必須擁有與這兩個不同事件相關的相互關聯識別碼。 當您使用標頭進行要求時，Azure 搜尋服務會為您提供搜尋識別碼︰

*C#*

    // This sample uses the Azure Search .NET SDK https://www.nuget.org/packages/Microsoft.Azure.Search

    var client = new SearchIndexClient(<ServiceName>, <IndexName>, new SearchCredentials(<QueryKey>)
    var headers = new Dictionary<string, List<string>>() { { "x-ms-azs-return-searchid", new List<string>() { "true" } } };
    var response = await client.Documents.SearchWithHttpMessagesAsync(searchText: searchText, searchParameters: parameters, customHeaders: headers);
    IEnumerable<string> headerValues;
    string searchId = string.Empty;
    if (response.Response.Headers.TryGetValues("x-ms-azs-searchid", out headerValues)){
     searchId = headerValues.FirstOrDefault();
    }

*JavaScript*

    request.setRequestHeader("x-ms-azs-return-searchid", "true");
    request.setRequestHeader("Access-Control-Expose-Headers", "x-ms-azs-searchid");
    var searchId = request.getResponseHeader('x-ms-azs-searchid');

**III.記錄搜尋事件**

每當使用者發出搜尋要求時，您應該將其記錄為搜尋事件，並包含下列關於 Application Insights 自訂事件的結構描述︰

**ServiceName**：(字串) 搜尋服務名稱 **SearchId**：(GUID) 搜尋查詢的唯一識別碼 (包含在搜尋回應中) **IndexName**︰(字串) 要查詢的搜尋服務索引 **QueryTerms**：(字串) 使用者所輸入的搜尋詞彙 **ResultCount**︰(int) 所傳回的文件數目 (包含在搜尋回應中) **ScoringProfile**︰(字串) 所使用的評分設定檔名稱 (如果有)

> [!NOTE]
> 將 $count=true 新增至搜尋查詢，要求使用者所產生查詢的計數。 請在[這裡](https://docs.microsoft.com/rest/api/searchservice/search-documents#request)參閱詳細資訊
>

> [!NOTE]
> 請記得，務必只記錄使用者所產生的搜尋查詢。
>

*C#*

    var properties = new Dictionary <string, string> {
    {"SearchServiceName", <service name>},
    {"SearchId", <search Id>},
    {"IndexName", <index name>},
    {"QueryTerms", <search terms>},
    {"ResultCount", <results count>},
    {"ScoringProfile", <scoring profile used>}
    };
    telemetryClient.TrackEvent("Search", properties);

*JavaScript*

    appInsights.trackEvent("Search", {
    SearchServiceName: <service name>,
    SearchId: <search id>,
    IndexName: <index name>,
    QueryTerms: <search terms>,
    ResultCount: <results count>,
    ScoringProfile: <scoring profile used>
    });

**IV.記錄點選事件**

每當使用者點選文件時，必須記錄這個訊號才可進行搜尋分析。 使用 Application Insights 自訂事件，記錄包含下列結構描述的這些事件：

**ServiceName**：(字串) 搜尋服務名稱 **SearchId**：(GUID) 相關搜尋查詢的唯一識別碼 **DocId**：(字串) 文件識別碼 **Position**(int) 文件在搜尋結果頁面中的排名

> [!NOTE]
> 位置是指您應用程式中的基本順序。 您可以自由地設定這個數字 (只要它一律相同) 以便進行比較。
>

*C#*

    var properties = new Dictionary <string, string> {
    {"SearchServiceName", <service name>},
    {"SearchId", <search id>},
    {"ClickedDocId", <clicked document id>},
    {"Rank", <clicked document position>}
    };
    telemetryClient.TrackEvent("Click", properties);

*JavaScript*

    appInsights.trackEvent("Click", {
        SearchServiceName: <service name>,
        SearchId: <search id>,
        ClickedDocId: <clicked document id>,
        Rank: <clicked document position>
    });

### <a name="3-analyze-with-power-bi-desktop"></a>3.使用 Power BI Desktop 進行分析

在您完成檢測您的應用程式，並確認您的應用程式正確地連線至 Application Insights 之後，可以使用 Azure 搜尋服務針對 Power BI Desktop 所建立的預先定義範本。
此範本包含圖表和資料表，可幫助您制定更明智的決策，以改善您的搜尋效能和相關性。

若要將 Power BI Desktop 範本具現化，您需要三項 Application Insights 的相關資訊。 當您選取要使用的資源時，可在搜尋流量分析頁面上找到這項資料

![搜尋流量分析刀鋒視窗中的 Application Insights 資料][2]

Power BI Desktop 範本中所包含的計量︰

*   點選率 (CTR)：點選特定文件的使用者數與總搜尋數的比率。
*   無點選搜尋︰註冊無點選的熱門查詢詞彙
*   最多點選的文件︰過去 24 小時、7 天和 30 天內，依識別碼排序最多點選的文件。
*   常用的詞彙文件組︰依點選排序，造成點選相同文件的詞彙。
*   點選的時間︰自搜尋查詢起按照時間分組的點選

![從 Application Insights 讀取的 Power BI 範本][3]


## <a name="next-steps"></a>後續步驟
檢測您的搜尋應用程式，取得搜尋服務的強大且詳細相關資料。

您可以在[這裡](https://go.microsoft.com/fwlink/?linkid=842905)找到 Application Insights 的詳細資訊。 若要深入了解其不同的服務層，請瀏覽 Application Insights [定價頁面](https://azure.microsoft.com/pricing/details/application-insights/)。

深入了解如何建立令人讚嘆的報告。 如需詳細資訊，請參閱[開始使用 Power BI Desktop](https://powerbi.microsoft.com/en-us/documentation/powerbi-desktop-getting-started/)

<!--Image references-->
[1]: ./media/search-traffic-analytics/AzureSearch-TrafficAnalytics.png
[2]: ./media/search-traffic-analytics/AzureSearch-AppInsightsData.png
[3]: ./media/search-traffic-analytics/AzureSearch-PBITemplate.png
