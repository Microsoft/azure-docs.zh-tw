---
title: 設定 Azure 診斷將資料傳送至 Application Insights
description: 更新 Azure 診斷公用組態以傳送資料至 Application Insights。
services: azure-monitor
author: rboucher
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 03/19/2016
ms.author: robb
ms.component: diagnostic-extension
ms.openlocfilehash: 3e1f4076c7a90cbb348f31b7b92e745fff79a04f
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35262132"
---
# <a name="send-cloud-service-virtual-machine-or-service-fabric-diagnostic-data-to-application-insights"></a>將雲端服務、虛擬機器或 Service Fabric 診斷資料傳送至 Application Insights
雲端服務、虛擬機器、虛擬機器擴展集和 Service Fabric 全都使用 Azure 診斷擴充功能來收集資料。  Azure 診斷會將資料傳送至 Azure 儲存體資料表。  不過，您也可以使用 Azure 診斷擴充功能 1.5 或更新版本，將所有資料或一部分資料傳送至其他位置。

本文說明如何將資料從 Azure 診斷擴充功能傳送至 Application Insights。

## <a name="diagnostics-configuration-explained"></a>說明診斷組態
Azure 診斷擴充功能 1.5 引進接收器，這些是可供您傳送診斷資料的其他位置。

Application Insights 的接收器組態範例：

```XML
<SinksConfig>
    <Sink name="ApplicationInsights">
      <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>
      <Channels>
        <Channel logLevel="Error" name="MyTopDiagData"  />
        <Channel logLevel="Verbose" name="MyLogData"  />
      </Channels>
    </Sink>
</SinksConfig>
```
```JSON
"SinksConfig": {
    "Sink": [
        {
            "name": "ApplicationInsights",
            "ApplicationInsights": "{Insert InstrumentationKey}",
            "Channels": {
                "Channel": [
                    {
                        "logLevel": "Error",
                        "name": "MyTopDiagData"
                    },
                    {
                        "logLevel": "Error",
                        "name": "MyLogData"
                    }
                ]
            }
        }
    ]
}
```
- **Sink** name 屬性是可唯一識別接收器的字串值。

- **ApplicationInsights** 元素指定 Azure 診斷資料送出的 Application Insights 資源的檢測金鑰。
    - 如果您沒有現有的 Application Insights 資源，請參閱[建立新的 Application Insights 資源](../application-insights/app-insights-create-new-resource.md)，以取得建立資源及取得檢測金鑰的詳細資訊。
    - 如果您以 Azure SDK 2.8 和更新版本開發雲端服務，則會自動填入此檢測金鑰。 這個值是根據封裝雲端服務專案時的 **APPINSIGHTS_INSTRUMENTATIONKEY** 服務組態設定。 請參閱[使用含雲端服務的 Application Insights](../application-insights/app-insights-cloudservices.md)。

- **Channels** 元素包含一個或多個 **Channel** 元素。
    - name屬性可唯一參考該通道。
    - loglevel  屬性可讓您指定通道允許的記錄等級。 可用的記錄等級從最多到最少資訊依序為：
        - 詳細資訊
        - 資訊
        - 警告
        - Error
        - 重要

通道就像篩選條件，可讓您選取要傳送至目標接收器的特定記錄等級。 例如，您可以收集詳細記錄，將它們傳送至儲存體，但只將「錯誤」傳送至接收器。

下圖顯示此關聯性。

![診斷公用組態](./media/azure-diagnostics-configure-applicationinsights/AzDiag_Channels_App_Insights.png)

下圖摘要說明設定值及其運作方式。 您可以在組態中的不同階層等級中包含多個接收器。 在最上層指定的接收器會成為全域設定，而在個別元素指定的接收器就形同覆寫該全域設定。

![診斷接收器組態和 Application Insights](./media/azure-diagnostics-configure-applicationinsights/Azure_Diagnostics_Sinks.png)

## <a name="complete-sink-configuration-example"></a>完整接收器組態範例
以下是完整的公用組態檔範例，可以
1. 將所有錯誤傳送至 Application Insights (在 **DiagnosticMonitorConfiguration** 節點上指定)
2. 也會傳送應用程式記錄 (在 **Logs** 節點上指定) 的「詳細資訊」等級記錄。

```XML
<WadCfg>
  <DiagnosticMonitorConfiguration overallQuotaInMB="4096"
       sinks="ApplicationInsights.MyTopDiagData"> <!-- All info below sent to this channel -->
    <DiagnosticInfrastructureLogs />
    <PerformanceCounters>
      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
      <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
    </PerformanceCounters>
    <WindowsEventLog scheduledTransferPeriod="PT1M">
      <DataSource name="Application!*" />
    </WindowsEventLog>
    <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Verbose"
            sinks="ApplicationInsights.MyLogData"/> <!-- This specific info sent to this channel -->
  </DiagnosticMonitorConfiguration>

<SinksConfig>
    <Sink name="ApplicationInsights">
      <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>
      <Channels>
        <Channel logLevel="Error" name="MyTopDiagData"  />
        <Channel logLevel="Verbose" name="MyLogData"  />
      </Channels>
    </Sink>
  </SinksConfig>
</WadCfg>
```
```JSON
"WadCfg": {
    "DiagnosticMonitorConfiguration": {
        "overallQuotaInMB": 4096,
        "sinks": "ApplicationInsights.MyTopDiagData", "_comment": "All info below sent to this channel",
        "DiagnosticInfrastructureLogs": {
        },
        "PerformanceCounters": {
            "PerformanceCounterConfiguration": [
                {
                    "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                    "sampleRate": "PT3M"
                },
                {
                    "counterSpecifier": "\\Memory\\Available MBytes",
                    "sampleRate": "PT3M"
                }
            ]
        },
        "WindowsEventLog": {
            "scheduledTransferPeriod": "PT1M",
            "DataSource": [
                {
                    "name": "Application!*"
                }
            ]
        },
        "Logs": {
            "scheduledTransferPeriod": "PT1M",
            "scheduledTransferLogLevelFilter": "Verbose",
            "sinks": "ApplicationInsights.MyLogData", "_comment": "This specific info sent to this channel"
        }
    },
    "SinksConfig": {
        "Sink": [
            {
                "name": "ApplicationInsights",
                "ApplicationInsights": "{Insert InstrumentationKey}",
                "Channels": {
                    "Channel": [
                        {
                            "logLevel": "Error",
                            "name": "MyTopDiagData"
                        },
                        {
                            "logLevel": "Verbose",
                            "name": "MyLogData"
                        }
                    ]
                }
            }
        ]
    }
}
```
在先前的組態中，下列幾行的意義如下︰

### <a name="send-all-the-data-that-is-being-collected-by-azure-diagnostics"></a>傳送 Azure 診斷收集的所有資料

```XML
<DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="ApplicationInsights">
```
```JSON
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights",
}
```

### <a name="send-only-error-logs-to-the-application-insights-sink"></a>只將錯誤資料傳送至 Application Insights 接收器

```XML
<DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="ApplicationInsights.MyTopDiagdata">
```
```JSON
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights.MyTopDiagData",
}
```

### <a name="send-verbose-application-logs-to-application-insights"></a>將「詳細資訊」應用程式記錄傳送至 Application Insights

```XML
<Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Verbose" sinks="ApplicationInsights.MyLogData"/>
```
```JSON
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights.MyLogData",
}
```

## <a name="limitations"></a>限制

- **通道只記錄類型，不記錄效能計數器。** 如果您在通道中指定效能計數器元素，將會忽略。
- **通道的記錄等級不能超過 Azure 診斷收集的記錄等級。** 例如，您不能在 Logs 元素中收集「應用程式記錄」錯誤，也無法嘗試將「詳細資訊」記錄傳送至 Application Insight 接收器。 *ScheduledTransferLogLevelFilter* 屬性一律必須收集與您正嘗試傳送到接收器的記錄相等或更多個記錄。
- **您無法將 Azure 診斷擴充功能收集的 blob 資料傳送至 Application Insights。** 例如，Directories 節點下指定的任何項目。 針對損毀傾印，實際損毀傾印會傳送至 blob 儲存體，並只會將損毀傾印所產生的通知傳送至 Application Insights。

## <a name="next-steps"></a>後續步驟
* 了解如何在 Application Insights 中[檢視您的 Azure 診斷資訊](https://docs.microsoft.com/azure/application-insights/app-insights-cloudservices#view-azure-diagnostic-events)。
* 使用 [PowerShell](../cloud-services/cloud-services-diagnostics-powershell.md) 來為您的應用程式啟用 Azure 診斷擴充。
* 使用 [Visual Studio](../vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md) 來為您的應用程式啟用 Azure 診斷擴充
