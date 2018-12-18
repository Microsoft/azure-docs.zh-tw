---
title: 在 Application Insights 中使用 PowerShell 設定警示 | Microsoft Docs
description: 自動化 Application Insights 的組態以取得有關度量變更的電子郵件。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 05d6a9e0-77a2-4a35-9052-a7768d23a196
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 10/31/2016
ms.author: mbullwin
ms.openlocfilehash: 8e317d44b7fdff1ef244e2e549fbbebf3f829740
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35632734"
---
# <a name="use-powershell-to-set-alerts-in-application-insights"></a>在 Application Insights 中使用 PowerShell 設定警示
您可以在 [Application Insights](app-insights-overview.md) 中自動設定[警示](app-insights-alerts.md)的組態。

此外，您可以[設定 webhook 以自動回應至警示](../monitoring-and-diagnostics/insights-webhooks-alerts.md)。

> [!NOTE]
> 如果您想要同時建立資源和警示，請考慮[使用 Azure Resource Manager 範本](app-insights-powershell.md)。
>
>

## <a name="one-time-setup"></a>單次設定
若您未曾將 PowerShell 與 Azure 訂用帳戶搭配使用：

在您要執行指令碼的電腦上，安裝 Azure Powershell 模組。

* 安裝 [Microsoft Web Platform Installer (v5 或更高版本)](http://www.microsoft.com/web/downloads/platform.aspx)。
* 使用該程式安裝 Microsoft Azure PowerShell

## <a name="connect-to-azure"></a>連接到 Azure
啟動 Azure PowerShell 並 [連接至您的訂用帳戶](/powershell/azure/overview)：

```PowerShell

    Add-AzureAccount
```


## <a name="get-alerts"></a>取得警示
    Get-AzureAlertRmRule -ResourceGroup "Fabrikam" [-Name "My rule"] [-DetailedOutput]

## <a name="add-alert"></a>Add alert
    Add-AlertRule  -Name "{ALERT NAME}" -Description "{TEXT}" `
     -ResourceGroup "{GROUP NAME}" `
     -ResourceId "/subscriptions/{SUBSCRIPTION ID}/resourcegroups/{GROUP NAME}/providers/microsoft.insights/components/{APP RESOURCE NAME}" `
     -MetricName "{METRIC NAME}" `
     -Operator GreaterThan  `
     -Threshold {NUMBER}   `
     -WindowSize {HH:MM:SS}  `
     [-SendEmailToServiceOwners] `
     [-CustomEmails "EMAIL1@X.COM","EMAIL2@Y.COM" ] `
     -Location "East US" // must be East US at present
     -RuleType Metric



## <a name="example-1"></a>範例 1
若伺服器針對 HTTP 要求的回應時間 (平均超過 5 分鐘) 慢於 1 秒，則傳送電子郵件給我。 我的 Application Insights 資源稱為 IceCreamWebApp，其位於 Fabrikam 資源群組。 我是 Azure 訂用帳戶的擁有者。

GUID 是該訂用帳戶的 ID (而非應用程式的檢測金鑰)。

    Add-AlertRule -Name "slow responses" `
     -Description "email me if the server responds slowly" `
     -ResourceGroup "Fabrikam" `
     -ResourceId "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Fabrikam/providers/microsoft.insights/components/IceCreamWebApp" `
     -MetricName "request.duration" `
     -Operator GreaterThan `
     -Threshold 1 `
     -WindowSize 00:05:00 `
     -SendEmailToServiceOwners `
     -Location "East US" -RuleType Metric

## <a name="example-2"></a>範例 2
我已有應用程式，並在其中使用 [TrackMetric()](app-insights-api-custom-events-metrics.md#trackmetric) 來報告名為 "salesPerHour" 的度量。 若 salesPerHour 超過 24 小時皆低於平均值 100，則傳送電子郵件給我的同事。

    Add-AlertRule -Name "poor sales" `
     -Description "slow sales alert" `
     -ResourceGroup "Fabrikam" `
     -ResourceId "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Fabrikam/providers/microsoft.insights/components/IceCreamWebApp" `
     -MetricName "salesPerHour" `
     -Operator LessThan `
     -Threshold 100 `
     -WindowSize 24:00:00 `
     -CustomEmails "satish@fabrikam.com","lei@fabrikam.com" `
     -Location "East US" -RuleType Metric

您亦可針對使用其他追蹤呼叫之 [測量參數](app-insights-api-custom-events-metrics.md#properties) (例如 TrackEvent 或 trackPageView) 報告的度量，使用相同的規則。

## <a name="metric-names"></a>度量名稱
| 度量名稱 | 畫面名稱 | 說明 |
| --- | --- | --- |
| `basicExceptionBrowser.count` |瀏覽器例外狀況 |在瀏覽器中擲回的未攔截例外狀況計數。 |
| `basicExceptionServer.count` |伺服器例外狀況 |應用程式擲回的未處理例外狀況計數 |
| `clientPerformance.clientProcess.value` |用戶端處理時間 |從接收上個文件位元組直至載入 DOM 的經過時間。 系統可能仍在處理非同步要求。 |
| `clientPerformance.networkConnection.value` |頁面載入網路連線時間 |瀏覽器連線至網路所需的時間。 可為 0 (若已快取)。 |
| `clientPerformance.receiveRequest.value` |接收回應時間 |瀏覽器傳送要求直至開始接收回應的經過時間。 |
| `clientPerformance.sendRequest.value` |傳送要求時間 |瀏覽器傳送要求所耗費的時間。 |
| `clientPerformance.total.value` |瀏覽器頁面載入時間 |從使用者要求直至載入 DOM、樣式表、指令碼和影像的經過時間。 |
| `performanceCounter.available_bytes.value` |可用的記憶體 |針對處理程序或系統用途的立即可用實體記憶體。 |
| `performanceCounter.io_data_bytes_per_sec.value` |處理程序 IO 速率 |每秒讀取與寫入檔案、 網路和裝置的總位元組數。 |
| `performanceCounter.number_of_exceps_thrown_per_sec.value` |例外狀況比率 |每秒擲回的例外狀況。 |
| `performanceCounter.percentage_processor_time.value` |處理程序 CPU |處理器針對應用程式處理程序執行指示，所用之全部處理程序執行緒的經過時間百分比。 |
| `performanceCounter.percentage_processor_total.value` |處理器時間 |處理器針對非閒置執行緒所耗費時間的百分比。 |
| `performanceCounter.process_private_bytes.value` |處理程序私人位元組 |以獨佔方式指派至監視應用程式處理程序的記憶體。 |
| `performanceCounter.request_execution_time.value` |ASP.NET 要求執行時間 |最近要求的執行時間。 |
| `performanceCounter.requests_in_application_queue.value` |執行佇列中的 ASP.NET 要求 |應用程式要求佇列的長度。 |
| `performanceCounter.requests_per_sec.value` |ASP.NET 要求率 |每秒從 ASP.NET 發出所有應用程式要求的速率。 |
| `remoteDependencyFailed.durationMetric.count` |相依性失敗 |伺服器應用程式針對外部資源的呼叫失敗計數。 |
| `request.duration` |伺服器回應時間 |從接收 HTTP 要求直至完成傳送回應的經過時間。 |
| `request.rate` |要求率 |每秒發出所有應用程式要求的速率。 |
| `requestFailed.count` |失敗的要求 |產生回應碼的 HTTP 要求計數 >= 400 |
| `view.count` |頁面檢視 |網頁的用戶端使用者要求計數。 系統會篩選掉綜合流量。 |
| {您的自訂度量名稱} |{您的度量名稱} |由 [TrackMetric](app-insights-api-custom-events-metrics.md#trackmetric) 或[追蹤呼叫之測量參數](app-insights-api-custom-events-metrics.md#properties)所報告的度量值。 |

此度量由不同遙測模組所傳送：

| 度量群組 | 收集器模組 |
| --- | --- |
| basicExceptionBrowser、<br/>clientPerformance、<br/>view |[瀏覽器 JavaScript](app-insights-javascript.md) |
| performanceCounter |[效能](app-insights-configuration-with-applicationinsights-config.md) |
| remoteDependencyFailed |[相依性](app-insights-configuration-with-applicationinsights-config.md) |
| request、<br/>requestFailed |[伺服器要求](app-insights-configuration-with-applicationinsights-config.md) |

## <a name="webhooks"></a>Webhook
您可以[自動回應至警示](../monitoring-and-diagnostics/insights-webhooks-alerts.md)。 Azure 會在出現警示時呼叫您選擇的網址。

## <a name="see-also"></a>另請參閱
* [用來設定 Application Insights 的指令碼](app-insights-powershell-script-create-resource.md)
* [從範本建立 Application Insights 和 Web 測試資源](app-insights-powershell.md)
* [自動化 Microsoft Azure 診斷與 Application Insights 的耦合](app-insights-powershell-azure-diagnostics.md)
* [自動回應至警示](../monitoring-and-diagnostics/insights-webhooks-alerts.md)
