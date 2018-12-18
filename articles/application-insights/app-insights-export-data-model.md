---
title: Azure Application Insights 資料模型 | Microsoft Docs
description: 描述從 JSON 中的連續匯出匯出的屬性，並做為篩選器。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: cabad41c-0518-4669-887f-3087aef865ea
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/21/2016
ms.author: mbullwin
ms.openlocfilehash: ee6597b78ac8de8fc3a7f3796010f22919243b23
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35294889"
---
# <a name="application-insights-export-data-model"></a>Application Insights 匯出資料模型
此表列出從 [Application Insights](app-insights-overview.md) SDK 傳送至入口網站的遙測屬性。
您會在 [連續匯出](app-insights-export-telemetry.md)的資料輸出中看到這些屬性。
它們也會出現在[計量瀏覽器](app-insights-metrics-explorer.md)和[診斷搜尋](app-insights-diagnostic-search.md)的屬性篩選中。

注意事項：

* `[0]` 代表您必須在路徑中插入索引的一點，但它未必是 0。
* 持續時間的長度單位是微秒，因此 10000000 == 1 秒。
* 日期和時間是 UTC，並以 ISO 格式 `yyyy-MM-DDThh:mm:ss.sssZ`


## <a name="example"></a>範例
    // A server report about an HTTP request
    {
    "request": [
      {
        "urlData": { // derived from 'url'
          "host": "contoso.org",
          "base": "/",
          "hashTag": ""
        },
        "responseCode": 200, // Sent to client
        "success": true, // Default == responseCode<400
        // Request id becomes the operation id of child events
        "id": "fCOhCdCnZ9I=",  
        "name": "GET Home/Index",
        "count": 1, // 100% / sampling rate
        "durationMetric": {
          "value": 1046804.0, // 10000000 == 1 second
          // Currently the following fields are redundant:
          "count": 1.0,
          "min": 1046804.0,
          "max": 1046804.0,
          "stdDev": 0.0,
          "sampledValue": 1046804.0
        },
        "url": "/"
      }
    ],
    "internal": {
      "data": {
        "id": "7f156650-ef4c-11e5-8453-3f984b167d05",
        "documentVersion": "1.61"
      }
    },
    "context": {
      "device": { // client browser
        "type": "PC",
        "screenResolution": { },
        "roleInstance": "WFWEB14B.fabrikam.net"
      },
      "application": { },
      "location": { // derived from client ip
        "continent": "North America",
        "country": "United States",
        // last octagon is anonymized to 0 at portal:
        "clientip": "168.62.177.0",
        "province": "",
        "city": ""
      },
      "data": {
        "isSynthetic": true, // we identified source as a bot
        // percentage of generated data sent to portal:
        "samplingRate": 100.0,
        "eventTime": "2016-03-21T10:05:45.7334717Z" // UTC
      },
      "user": {
        "isAuthenticated": false,
        "anonId": "us-tx-sn1-azr", // bot agent id
        "anonAcquisitionDate": "0001-01-01T00:00:00Z",
        "authAcquisitionDate": "0001-01-01T00:00:00Z",
        "accountAcquisitionDate": "0001-01-01T00:00:00Z"
      },
      "operation": {
        "id": "fCOhCdCnZ9I=",
        "parentId": "fCOhCdCnZ9I=",
        "name": "GET Home/Index"
      },
      "cloud": { },
      "serverDevice": { },
      "custom": { // set by custom fields of track calls
        "dimensions": [ ],
        "metrics": [ ]
      },
      "session": {
        "id": "65504c10-44a6-489e-b9dc-94184eb00d86",
        "isFirst": true
      }
    }
  }

## <a name="context"></a>Context
所有類型的遙測都會伴隨內容區段。 並非所有的欄位都會連同每個資料點傳輸。

| Path | 類型 | 注意 |
| --- | --- | --- |
| context.custom.dimensions [0] |物件 [ ] |由自訂屬性參數設定的索引鍵-值字串組。 索引鍵的最大長度 100，值的最大長度 1024。 超過 100 個唯一值時，屬性可供搜尋，但無法用來分割。 每個 ikey 的最大值 200 索引鍵。 |
| context.custom.metrics [0] |物件 [ ] |由自訂測量參數和 TrackMetrics 設定的索引鍵-值組。 索引鍵的最大長度 100，值可能為數值。 |
| context.data.eventTime |字串 |UTC |
| context.data.isSynthetic |布林值 |要求似乎來自 bot 或 web 測試。 |
| context.data.samplingRate |number |由傳送至入口網站之 SDK 所產生的遙測百分比。 範圍 0.0-100.0。 |
| context.device |物件 |用戶端裝置 |
| context.device.browser |字串 |IE, Chrome, ... |
| context.device.browserVersion |字串 |Chrome 48.0, ... |
| context.device.deviceModel |字串 | |
| context.device.deviceName |字串 | |
| context.device.id |字串 | |
| context.device.locale |字串 |en-GB, de-DE, ... |
| context.device.network |字串 | |
| context.device.oemName |字串 | |
| context.device.osVersion |字串 |主機作業系統 |
| context.device.roleInstance |字串 |伺服器主機的識別碼 |
| context.device.roleName |字串 | |
| context.device.type |字串 |PC, Browser, ... |
| context.location |物件 |衍生自 clientip。 |
| context.location.city |字串 |衍生自 clientip (如果已知) |
| context.location.clientip |字串 |最後一個八邊形匿名設定為 0。 |
| context.location.continent |字串 | |
| context.location.country |字串 | |
| context.location.province |字串 |州或省 |
| context.operation.id |字串 |具有相同作業識別碼的項目會在入口網站中顯示為相關項目。 通常為要求 id。 |
| context.operation.name |字串 |url 或要求名稱 |
| context.operation.parentId |字串 |允許巢狀的相關項目。 |
| context.session.id |字串 |來自相同來源的作業群組識別碼。 在 30 分鐘期間沒有發出工作階段結束訊號的作業。 |
| context.session.isFirst |布林值 | |
| context.user.accountAcquisitionDate |字串 | |
| context.user.anonAcquisitionDate |字串 | |
| context.user.anonId |字串 | |
| context.user.authAcquisitionDate |字串 |[已驗證的使用者](app-insights-api-custom-events-metrics.md#authenticated-users) |
| context.user.isAuthenticated |布林值 | |
| internal.data.documentVersion |字串 | |
| internal.data.id |字串 | 項目內嵌至 Application Insights 時所指派的的唯一識別碼 |

## <a name="events"></a>活動
[TrackEvent()](app-insights-api-custom-events-metrics.md#trackevent)產生的自訂事件。

| Path | 類型 | 注意 |
| --- | --- | --- |
| 事件 [0] 計數 |integer |100/([取樣](app-insights-sampling.md) 率)。 例如 4 =&gt; 25%。 |
| 事件 [0] 名稱 |字串 |事件名稱。  最大長度 250。 |
| 事件 [0] url |字串 | |
| 事件 [0] urlData.base |字串 | |
| 事件 [0] urlData.host |字串 | |

## <a name="exceptions"></a>例外狀況
回報在伺服器和瀏覽器中的 [例外狀況](app-insights-asp-net-exceptions.md) 。

| Path | 類型 | 注意 |
| --- | --- | --- |
| basicException [0] 組件 |字串 | |
| basicException [0] 計數 |integer |100/([取樣](app-insights-sampling.md) 率)。 例如 4 =&gt; 25%。 |
| basicException [0] exceptionGroup |字串 | |
| basicException [0] exceptionType |字串 | |
| basicException [0] failedUserCodeMethod |字串 | |
| basicException [0] failedUserCodeAssembly |字串 | |
| basicException [0] handledAt |字串 | |
| basicException [0] hasFullStack |布林值 | |
| basicException [0] id |字串 | |
| basicException [0] 方法 |字串 | |
| basicException [0] 訊息 |字串 |例外狀況訊息。 最大長度 10k。 |
| basicException [0] outerExceptionMessage |字串 | |
| basicException [0] outerExceptionThrownAtAssembly |字串 | |
| basicException [0] outerExceptionThrownAtMethod |字串 | |
| basicException [0] outerExceptionType |字串 | |
| basicException [0] outerId |字串 | |
| basicException [0] parsedStack [0] 組件 |字串 | |
| basicException [0] parsedStack [0] fileName |字串 | |
| basicException [0] parsedStack [0] 層級 |integer | |
| basicException [0] parsedStack [0] 列 |integer | |
| basicException [0] parsedStack [0] 方法 |字串 | |
| basicException [0] 堆疊 |字串 |最大長度 10k |
| basicException [0] typeName |字串 | |

## <a name="trace-messages"></a>追蹤訊息
由 [TrackTrace](app-insights-api-custom-events-metrics.md#tracktrace) 及[記錄配接器](app-insights-asp-net-trace-logs.md)傳送。

| Path | 類型 | 注意 |
| --- | --- | --- |
| 訊息 [0] loggerName |字串 | |
| 訊息 [0] 參數 |字串 | |
| 訊息 [0] 原始碼 |字串 |記錄檔訊息，最大長度 10k。 |
| 訊息 [0] severityLevel |字串 | |

## <a name="remote-dependency"></a>遠端相依性
由 TrackDependency 傳送。 用於回報伺服器中 [相依性呼叫](app-insights-asp-net-dependencies.md) 以及瀏覽器中 AJAX 呼叫的效能和使用情形。

| Path | 類型 | 注意 |
| --- | --- | --- |
| remoteDependency [0] async |布林值 | |
| remoteDependency [0] baseName |字串 | |
| remoteDependency [0] commandName |字串 |例如 "home/index" |
| remoteDependency [0] 計數 |integer |100/([取樣](app-insights-sampling.md) 率)。 例如 4 =&gt; 25%。 |
| remoteDependency [0] dependencyTypeName |字串 |HTTP、SQL、... |
| remoteDependency [0] durationMetric.value |number |從根據相依性呼叫回應完成開始計算的時間 |
| remoteDependency [0] id |字串 | |
| remoteDependency [0] 名稱 |字串 |Url。 最大長度 250。 |
| remoteDependency [0] resultCode |字串 |從 HTTP 相依性 |
| remoteDependency [0] 成功 |布林值 | |
| remoteDependency [0] 類型 |字串 |Http、Sql、... |
| remoteDependency [0] url |字串 |最大長度 2000 |
| remoteDependency [0] urlData.base |字串 |最大長度 2000 |
| remoteDependency [0] urlData.hashTag |字串 | |
| remoteDependency [0] urlData.host |字串 |最大長度 200 |

## <a name="requests"></a>Requests
由 [TrackRequest](app-insights-api-custom-events-metrics.md#trackrequest)傳送。 標準模組使用它回報在伺服器上測量的伺服器回應時間。

| Path | 類型 | 注意 |
| --- | --- | --- |
| 要求 [0] 計數 |integer |100/([取樣](app-insights-sampling.md) 率)。 例如：4 =&gt; 25%。 |
| 要求 [0] durationMetric.value |number |從要求抵達到回應的時間。 1e7 == 1s |
| 要求 [0] id |字串 |作業 id |
| 要求 [0] 名稱 |字串 |GET/POST + url 基底。  最大長度 250 |
| 要求 [0] responseCode |integer |傳送至用戶端的 HTTP 回應 |
| 要求 [0] 成功 |布林值 |預設值 == (responseCode &lt; 400) |
| 要求 [0] url |字串 |不包括主機 |
| 要求 [0] urlData.base |字串 | |
| 要求 [0] urlData.hashTag |字串 | |
| 要求 [0] urlData.host |字串 | |

## <a name="page-view-performance"></a>頁面檢視效能
由瀏覽器傳送。 測量處理頁面的時間，從使用者起始要求到顯示完成 (不包括非同步 AJAX 呼叫)。

內容值會顯示用戶端作業系統和瀏覽器版本。

| Path | 類型 | 注意 |
| --- | --- | --- |
| clientPerformance [0] clientProcess.value |integer |從接收 HTML 完成到顯示頁面的時間。 |
| clientPerformance [0] 名稱 |字串 | |
| clientPerformance [0] networkConnection.value |integer |建立網路連線所需的時間。 |
| clientPerformance [0] receiveRequest.value |integer |從傳送要求完成至接收回覆中 HTML 的時間。 |
| clientPerformance [0] sendRequest.value |integer |傳送 HTTP 要求所需的時間。 |
| clientPerformance [0] total.value |integer |從開始傳送要求到顯示頁面的時間。 |
| clientPerformance [0] url |字串 |此要求的 URL |
| clientPerformance [0] urlData.base |字串 | |
| clientPerformance [0] urlData.hashTag |字串 | |
| clientPerformance [0] urlData.host |字串 | |
| clientPerformance [0] urlData.protocol |字串 | |

## <a name="page-views"></a>頁面檢視
由 trackPageView() 或 [stopTrackPage](app-insights-api-custom-events-metrics.md#page-views) 傳送

| Path | 類型 | 注意 |
| --- | --- | --- |
| 檢視 [0] 計數 |integer |100/([取樣](app-insights-sampling.md) 率)。 例如 4 =&gt; 25%。 |
| 檢視 [0] durationMetric.value |integer |在 trackPageView() 中或由 startTrackPage() - stopTrackPage() 選擇性設定的值。 和 clientPerformance 的值不同。 |
| 檢視 [0] 名稱 |字串 |頁面標題。  最大長度 250 |
| 檢視 [0] url |字串 | |
| 檢視 [0] urlData.base |字串 | |
| 檢視 [0] urlData.hashTag |字串 | |
| 檢視 [0] urlData.host |字串 | |

## <a name="availability"></a>可用性
回報 [可用性 Web 測試](app-insights-monitor-web-app-availability.md)。

| Path | 類型 | 注意 |
| --- | --- | --- |
| 可用性 [0] availabilityMetric.name |字串 |Availability |
| 可用性 [0] availabilityMetric.value |number |1.0 或 0.0 |
| 可用性 [0] 計數 |integer |100/([取樣](app-insights-sampling.md) 率)。 例如 4 =&gt; 25%。 |
| 可用性 [0] dataSizeMetric.name |字串 | |
| 可用性 [0] dataSizeMetric.value |integer | |
| 可用性 [0] durationMetric.name |字串 | |
| 可用性 [0] durationMetric.value |number |測試持續時間。 1e7==1s |
| 可用性 [0] 訊息 |字串 |失敗診斷 |
| 可用性 [0] 結果 |字串 |通過/失敗 |
| 可用性 [0] runLocation |字串 |http req 的地理區域來源 |
| 可用性 [0] testName |字串 | |
| 可用性 [0] testRunId |字串 | |
| 可用性 [0] testTimestamp |字串 | |

## <a name="metrics"></a>度量
由 TrackMetric() 產生。

度量值位於 context.custom.metrics[0]

例如︰

    {
     "metric": [ ],
     "context": {
     ...
     "custom": {
        "dimensions": [
          { "ProcessId": "4068" }
        ],
        "metrics": [
          {
            "dispatchRate": {
              "value": 0.001295,
              "count": 1.0,
              "min": 0.001295,
              "max": 0.001295,
              "stdDev": 0.0,
              "sampledValue": 0.001295,
              "sum": 0.001295
            }
          }
         } ] }
    }

## <a name="about-metric-values"></a>關於度量值
在度量報告和其他位置中的度量值，會利用標準物件結構回報。 例如︰

      "durationMetric": {
        "name": "contoso.org",
        "type": "Aggregation",
        "value": 468.71603053650279,
        "count": 1.0,
        "min": 468.71603053650279,
        "max": 468.71603053650279,
        "stdDev": 0.0,
        "sampledValue": 468.71603053650279
      }

目前 - 不過未來可能會變更 - 在所有從標準 SDK 模組回報的值中，`count==1` 以及只有 `name` 和 `value` 欄位是有用的。 它們會有差異的唯一案例是，如果您撰寫自己的 TrackMetric 呼叫，而且您在其中設定其他參數。

其他欄位的目的是允許度量在 SDK 中彙總，以減少入口網站的流量。 例如，您可以在傳送每個度量報告之前平均數個連續的讀數。 然後您會計算 min、max、標準差和彙總值 (sum 或 average)，並將計數設為報告所代表的讀數數目。

在上述資料表中，我們省略了很少使用的欄位計數、min、max、stdDev 和 sampledValue。

除了使用預先彙總的度量，如果您需要減少遙測量，您可以改為使用 [取樣](app-insights-sampling.md) 。

### <a name="durations"></a>持續時間
除非另有說明，否則持續時間皆以十分之一微秒表示，所以 10000000.0 表示 1 秒。

## <a name="see-also"></a>另請參閱
* [Application Insights](app-insights-overview.md)
* [連續匯出](app-insights-export-telemetry.md)
* [程式碼範例](app-insights-export-telemetry.md#code-samples)
