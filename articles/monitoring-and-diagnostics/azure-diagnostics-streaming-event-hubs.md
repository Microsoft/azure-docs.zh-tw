---
title: 將 Azure 診斷資料串流至事件中樞
description: 使用事件中樞設定 Azure 診斷的完整步驟，包括常見案例的指引。
services: azure-monitor
author: rboucher
ms.service: azure-monitor
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 07/13/2017
ms.author: robb
ms.component: diagnostic-extension
ms.openlocfilehash: c87a4acb8ca333af73643a38ae1338c9c8769d13
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37341225"
---
# <a name="streaming-azure-diagnostics-data-in-the-hot-path-by-using-event-hubs"></a>使用事件中樞串流最忙碌路徑中的 Azure 診斷資料
Azure 診斷會提供彈性的方法，用來收集來自雲端服務虛擬機器 (VM) 的度量和記錄檔，再將結果傳輸至 Azure 儲存體。 從 2016 年 3 月 (SDK 2.9) 的時間範圍開始，您可以使用 [Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)，將 Azure 診斷傳送至自訂的資料來源，並立即傳輸最忙碌路徑資料。

支援的資料類型包括：

* Windows 事件追蹤 (ETW) 事件
* 效能計數器
* Windows 事件記錄檔
* 應用程式記錄檔
* Azure 診斷基礎結構記錄檔

本文說明如何使用事件中樞從端對端設定 Azure 診斷。 另提供以下常見案例的指引：

* 如何自訂傳送到事件中樞的記錄檔和計量
* 如何變更每個環境中的組態
* 如何檢視事件中樞串流資料
* 如何針對連線問題進行疑難排解  

## <a name="prerequisites"></a>先決條件
雲端服務、VM、虛擬機器擴展集，以及 Azure SDK 2.9 開始的 Service Fabric 和對應的 Azure Tools for Visual Studio，皆支援從 Azure 診斷接收資料的事件中樞。

* Azure 診斷擴充 1.6 ([Azure SDK for. NET 2.9 或更新版本](https://azure.microsoft.com/downloads/) 預設以此為目標)
* [Visual Studio 2013 或更新版本](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)
* 在應用程式中使用 *.wadcfgx* 檔案和以下任一方法的 Azure 診斷現有組態：
  * Visual Studio： [為 Azure 雲端服務和虛擬機器設定診斷功能](../vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md)
  * Windows PowerShell： [使用 PowerShell 在 Azure 雲端服務中啟用診斷](../cloud-services/cloud-services-diagnostics-powershell.md)
* 文章中佈建的事件中樞命名空間，[開始使用事件中樞](../event-hubs/event-hubs-csharp-ephcs-getstarted.md)

## <a name="connect-azure-diagnostics-to-event-hubs-sink"></a>將 Azure 診斷連接至事件中樞接收
根據預設，Azure 診斷一律會將記錄檔和計量傳送至 Azure 儲存體帳戶。 應用程式可能會將資料傳送至事件中樞，方法是將新的 [接收] 區段新增至 *.wadcfgx* 檔案的 **PublicConfig** / **WadCfg** 元素底下。 在 Visual Studio 中，*.wadcfgx* 檔案會儲存在以下路徑：[雲端服務專案] > [角色] > **(RoleName)** > **diagnostics.wadcfgx** 檔案。

```xml
<SinksConfig>
  <Sink name="HotPath">
    <EventHub Url="https://diags-mycompany-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" />
  </Sink>
</SinksConfig>
```
```JSON
"SinksConfig": {
    "Sink": [
        {
            "name": "HotPath",
            "EventHub": {
                "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
                "SharedAccessKeyName": "SendRule"
            }
        }
    ]
}
```

在此範例中，事件中樞 URL 設定為事件中樞的完整命名空間：事件中樞命名空間 + "/" + 事件中樞名稱。  

事件中樞 URL 會在 [Azure 入口網站](http://go.microsoft.com/fwlink/?LinkID=213885) 中的 [事件中樞] 儀表板上顯示。  

[接收]  名稱可以設定為任何有效的字串，只要在整個組態檔一致使用相同的值即可。

> [!NOTE]
> 此區段中可能有其他設定的接收，例如 *applicationInsights* 。 Azure 診斷可以定義一或多個接收，前提是每個接收也在 **PrivateConfig** 區段中宣告。  
>
>

事件中樞接收也必須宣告並定義於 **.wadcfgx** 組態檔的 *PrivateConfig* 區段。

```XML
<PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <StorageAccount name="{account name}" key="{account key}" endpoint="{optional storage endpoint}" />
  <EventHub Url="https://diags-mycompany-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="{base64 encoded key}" />
</PrivateConfig>
```
```JSON
{
    "storageAccountName": "{account name}",
    "storageAccountKey": "{account key}",
    "storageAccountEndPoint": "{optional storage endpoint}",
    "EventHub": {
        "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
        "SharedAccessKeyName": "SendRule",
        "SharedAccessKey": "{base64 encoded key}"
    }
}
```

`SharedAccessKeyName` 值必須符合共用存取簽章 (SAS) 金鑰，以及**事件中樞**命名空間中已定義的原則。 在 [Azure 入口網站](https://portal.azure.com)中瀏覽至 [事件中樞] 儀表板，按一下 [設定] 索引標籤，然後設定具有*傳送*權限的具名原則 (如 "SendRule")。 **StorageAccount** 也已經在 **PrivateConfig** 中宣告。 如果這裡的值可以運作，就不需要變更。 在此範例中，我們保留空白的值，這代表下游資產將會設定值。 例如，*ServiceConfiguration.Cloud.cscfg* 環境組態檔會設定適合環境的名稱和金鑰。  

> [!WARNING]
> 事件中樞 SAS 金鑰會以純文字儲存在 *.wadcfgx* 檔案中。 有時候，系統會將該金鑰簽入原始程式碼控制，或做為組建伺服器中的資產提供，因此您應該適當地保護它。 建議您在這裡使用具有「僅限傳送」  權限的 SAS 金鑰，讓惡意使用者只能寫入事件中樞，而無法接聽或加以管理。
>
>

## <a name="configure-azure-diagnostics-to-send-logs-and-metrics-to-event-hubs"></a>設定 Azure 診斷將記錄檔和計量傳送至事件中樞
如前文所述，所有預設和自訂診斷資料 (亦即，計量和記錄檔) 會以設定的間隔自動傳送到 Azure 儲存體。 您可以藉由事件中樞和任何其他接收，指定要傳送至事件中樞的階層中任何根節點或分葉節點。 這包括 ETW 事件、效能計數器、Windows 事件記錄檔和應用程式記錄檔。   

請務必考慮實際上應該將多少資料點傳輸至事件中樞。 一般而言，開發人員會傳輸必須快速取用及解譯的低延遲忙碌路徑資料。 監視警示或自動調整規則的系統即是一例。 開發人員也會設定替代分析存放區或搜尋存放區；例如，Azure 串流分析、Elasticsearch、自訂監視系統，或最愛的他牌監視系統。

以下是一些範例組態。

```xml
<PerformanceCounters scheduledTransferPeriod="PT1M" sinks="HotPath">
  <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
  <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
  <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
</PerformanceCounters>
```
```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "sinks": "HotPath",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        }
    ]
}
```

在上述範例中，接收會套用至階層中的父 **PerformanceCounters**，這表示所有子 **PerformanceCounters** 將傳送至事件中樞。  

```xml
<PerformanceCounters scheduledTransferPeriod="PT1M">
  <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
  <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
  <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" sinks="HotPath" />
  <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" sinks="HotPath"/>
  <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" sinks="HotPath"/>
</PerformanceCounters>
```
```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M",
            "sinks": "HotPath"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Rejected",
            "sampleRate": "PT3M",
            "sinks": "HotPath"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Queued",
            "sampleRate": "PT3M",
            "sinks": "HotPath"
        }
    ]
}
```

在上述範例中，接收只會套用至三個計數器︰**已排入佇列的要求**、**遭拒絕的要求**和 **% 處理器時間**。  

下列範例示範開發人員如何將傳送的資料量，限制為用於此服務之健全狀況的關鍵度量。  

```XML
<Logs scheduledTransferPeriod="PT1M" sinks="HotPath" scheduledTransferLogLevelFilter="Error" />
```
```JSON
"Logs": {
    "scheduledTransferPeriod": "PT1M",
    "scheduledTransferLogLevelFilter": "Error",
    "sinks": "HotPath"
}
```

在此範例中，接收會套用至記錄檔，並且只篩選為「錯誤」層級追蹤。

## <a name="deploy-and-update-a-cloud-services-application-and-diagnostics-config"></a>部署和更新雲端服務應用程式和診斷設定
Visual Studio 提供最簡單的路徑供您部署應用程式和事件中樞接收組態。 若要檢視及編輯檔案，請在 Visual Studio 中開啟 *.wadcfgx* 檔案，然後再加以編輯和儲存。 其路徑為 [雲端服務專案] > [角色] > (RoleName) > diagnostics.wadcfgx。  

此時，Visual Studio、Visual Studio Team System 中的所有部署和部署更新動作，以及所有根據 MSBuild 和使用 **/t:publish** 目標的命令或指令碼，都會在封裝程序中納入 *.wadcfgx* 。 此外，部署和更新會使用 VM 上適當的 Azure 診斷代理程式擴充功能將檔案部署到 Azure。

在部署應用程式和 Azure 診斷組態後，您會立即在事件中樞的儀表板中看到活動。 這表示您已準備就緒，可以繼續在接聽程式用戶端或所選的分析工具中檢視最忙碌路徑資料。  

在下圖中，事件中樞儀表板會顯示從晚上 11 點之後開始傳送到事件中樞，而且狀況良好的診斷資料傳送作業。 也就是使用更新的 *.wadcfgx* 檔案部署應用程式，而且已正確設定接收的時候。

![][0]  

> [!NOTE]
> 當您更新 Azure Diagnostics 組態檔 (.wadcfgx) 時，建議您透過使用 Visual Studio 發行或 Windows PowerShell 指令碼，將更新推送至整個應用程式以及組態。  
>
>

## <a name="view-hot-path-data"></a>檢視忙碌路徑資料
如前文所述，接聽和處理事件中樞資料有許多使用案例。

一個簡單的作法是建立小型測試主控台應用程式，接聽事件中樞並列印輸出串流。 您可以將下列程式碼 (會在[開始使用事件中樞](../event-hubs/event-hubs-csharp-ephcs-getstarted.md)中詳細說明) 放置在主控台應用程式中。  

請注意，主控台應用程式必須包含 [Event Processor Host NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)。  

請記得使用您資源的值取代 **Main** 函式中角括弧裡面的值。   

```csharp
//Console application code for EventHub test client
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.ServiceBus.Messaging;

namespace EventHubListener
{
    class SimpleEventProcessor : IEventProcessor
    {
        Stopwatch checkpointStopWatch;

        async Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
        {
            Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
            if (reason == CloseReason.Shutdown)
            {
                await context.CheckpointAsync();
            }
        }

        Task IEventProcessor.OpenAsync(PartitionContext context)
        {
            Console.WriteLine("SimpleEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
            this.checkpointStopWatch = new Stopwatch();
            this.checkpointStopWatch.Start();
            return Task.FromResult<object>(null);
        }

        async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
        {
            foreach (EventData eventData in messages)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                    Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
                        context.Lease.PartitionId, data));

                foreach (var x in eventData.Properties)
                {
                    Console.WriteLine(string.Format("    {0} = {1}", x.Key, x.Value));
                }
            }

            //Call checkpoint every 5 minutes, so that worker can resume processing from 5 minutes back if it restarts.
            if (this.checkpointStopWatch.Elapsed > TimeSpan.FromMinutes(5))
            {
                await context.CheckpointAsync();
                this.checkpointStopWatch.Restart();
            }
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            string eventHubConnectionString = "Endpoint= <your connection string>”
            string eventHubName = "<Event hub name>";
            string storageAccountName = "<Storage account name>";
            string storageAccountKey = "<Storage account key>”;
            string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", storageAccountName, storageAccountKey);

            string eventProcessorHostName = Guid.NewGuid().ToString();
            EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, eventHubName, EventHubConsumerGroup.DefaultGroupName, eventHubConnectionString, storageConnectionString);
            Console.WriteLine("Registering EventProcessor...");
            var options = new EventProcessorOptions();
            options.ExceptionReceived += (sender, e) => { Console.WriteLine(e.Exception); };
            eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>(options).Wait();

            Console.WriteLine("Receiving. Press enter key to stop worker.");
            Console.ReadLine();
            eventProcessorHost.UnregisterEventProcessorAsync().Wait();
        }
    }
}
```

## <a name="troubleshoot-event-hubs-sinks"></a>針對事件中樞接收進行疑難排解
* 事件中樞不會如預期般顯示傳入或傳出事件活動。

    檢查已成功佈建您的事件中樞。 **.wadcfgx** 中 *PrivateConfig* 區段的所有連接資訊必須符合在入口網站中所示的資源的值。 請確定您已在入口網站中定義 SAS 原則 (此範例中為 “SendRule”)，並且對其授與「傳送」  權限。  
* 執行更新之後，事件中樞不會再顯示傳入或傳出事件活動。

    首先，請確定事件中樞和組態資訊正確，如先前所述。 有時候，系統會在部署更新時重設 **PrivateConfig** 。 建議的修正方式是對專案中的 *.wadcfgx* 進行所有變更，然後再推送完整的應用程式更新。 如果不可行，請確定診斷更新推送完整的 **PrivateConfig** ，包括 SAS 金鑰。  
* 我試過上述建議，不過事件中樞仍然無法運作。

    請嘗試查看 Azure 儲存體資料表，其中包含記錄檔和 Azure 診斷本身的錯誤︰ **WADDiagnosticInfrastructureLogsTable**。 其中一個選項是使用類似 [Azure 儲存體總管](http://www.storageexplorer.com) 的工具連接到此儲存體帳戶、檢視此資料表，並且新增過去 24 小時內時間戳記的查詢。 您可以使用此工具來匯出 .csv 檔案，並在 Microsoft Excel 之類的應用程式中開啟。 Excel 能輕鬆地搜尋電話卡字串 (如 **EventHubs**)，以便查看系統回報了哪些錯誤。  

## <a name="next-steps"></a>後續步驟
•    [深入了解事件中樞](https://azure.microsoft.com/services/event-hubs/)

## <a name="appendix-complete-azure-diagnostics-configuration-file-wadcfgx-example"></a>附錄：完成 Azure 診斷組態檔 (.wadcfgx) 範例
```xml
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <WadCfg>
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="applicationInsights.errors">
        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" />
        <Directories scheduledTransferPeriod="PT1M">
          <IISLogs containerName="wad-iis-logfiles" />
          <FailedRequestLogs containerName="wad-failedrequestlogs" />
        </Directories>
        <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="HotPath">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
        </PerformanceCounters>
        <WindowsEventLog scheduledTransferPeriod="PT1M">
          <DataSource name="Application!*" />
        </WindowsEventLog>
        <CrashDumps>
          <CrashDumpConfiguration processName="WaIISHost.exe" />
          <CrashDumpConfiguration processName="WaWorkerHost.exe" />
          <CrashDumpConfiguration processName="w3wp.exe" />
        </CrashDumps>
        <Logs scheduledTransferPeriod="PT1M" sinks="HotPath" scheduledTransferLogLevelFilter="Error" />
      </DiagnosticMonitorConfiguration>
      <SinksConfig>
        <Sink name="HotPath">
          <EventHub Url="https://diageventhub-py-ns.servicebus.windows.net/diageventhub-py" SharedAccessKeyName="SendRule" />
        </Sink>
        <Sink name="applicationInsights">
          <ApplicationInsights />
          <Channels>
            <Channel logLevel="Error" name="errors" />
          </Channels>
        </Sink>
      </SinksConfig>
    </WadCfg>
    <StorageAccount>ACCOUNT_NAME</StorageAccount>
  </PublicConfig>
  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <StorageAccount name="{account name}" key="{account key}" endpoint="{storage endpoint}" />
    <EventHub Url="https://diageventhub-py-ns.servicebus.windows.net/diageventhub-py" SharedAccessKeyName="SendRule" SharedAccessKey="YOUR_KEY_HERE" />
  </PrivateConfig>
  <IsEnabled>true</IsEnabled>
</DiagnosticsConfiguration>
```

此範例的補充 *ServiceConfiguration.Cloud.cscfg* 如下所示。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceConfiguration serviceName="MyFixItCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="3" osVersion="*" schemaVersion="2015-04.2.6">
  <Role name="MyFixIt.WorkerRole">
    <Instances count="1" />
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="YOUR_CONNECTION_STRING" />
    </ConfigurationSettings>
  </Role>
</ServiceConfiguration>
```

虛擬機器的對等 JSON 設定如下所示：

公用設定：
```JSON
{
    "WadCfg": {
        "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": 4096,
            "sinks": "applicationInsights.errors",
            "DiagnosticInfrastructureLogs": {
                "scheduledTransferLogLevelFilter": "Error"
            },
            "Directories": {
                "scheduledTransferPeriod": "PT1M",
                "IISLogs": {
                    "containerName": "wad-iis-logfiles"
                },
                "FailedRequestLogs": {
                    "containerName": "wad-failedrequestlogs"
                }
            },
            "PerformanceCounters": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "HotPath",
                "PerformanceCounterConfiguration": [
                    {
                        "counterSpecifier": "\\Memory\\Available MBytes",
                        "sampleRate": "PT3M"
                    },
                    {
                        "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
                        "sampleRate": "PT3M"
                    },
                    {
                        "counterSpecifier": "\\Web Service(_Total)\\Bytes Total/Sec",
                        "sampleRate": "PT3M"
                    },
                    {
                        "counterSpecifier": "\\ASP.NET Applications(__Total__)\\Requests/Sec",
                        "sampleRate": "PT3M"
                    },
                    {
                        "counterSpecifier": "\\ASP.NET Applications(__Total__)\\Errors Total/Sec",
                        "sampleRate": "PT3M"
                    },
                    {
                        "counterSpecifier": "\\ASP.NET\\Requests Queued",
                        "sampleRate": "PT3M"
                    },
                    {
                        "counterSpecifier": "\\ASP.NET\\Requests Rejected",
                        "sampleRate": "PT3M"
                    },
                    {
                        "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
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
                "scheduledTransferLogLevelFilter": "Error",
                "sinks": "HotPath"
            }
        },
        "SinksConfig": {
            "Sink": [
                {
                    "name": "HotPath",
                    "EventHub": {
                        "Url": "https://diageventhub-py-ns.servicebus.windows.net/diageventhub-py",
                        "SharedAccessKeyName": "SendRule"
                    }
                },
                {
                    "name": "applicationInsights",
                    "ApplicationInsights": "",
                    "Channels": {
                        "Channel": [
                            {
                                "logLevel": "Error",
                                "name": "errors"
                            }
                        ]
                    }
                }
            ]
        }
    },
    "StorageAccount": "{account name}"
}

```

受保護的設定：
```JSON
{
    "storageAccountName": "{account name}",
    "storageAccountKey": "{account key}",
    "storageAccountEndPoint": "{storage endpoint}",
    "EventHub": {
        "Url": "https://diageventhub-py-ns.servicebus.windows.net/diageventhub-py",
        "SharedAccessKeyName": "SendRule",
        "SharedAccessKey": "YOUR_KEY_HERE"
    }
}
```

## <a name="next-steps"></a>後續步驟
您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](../event-hubs/event-hubs-what-is-event-hubs.md)
* [建立事件中樞](../event-hubs/event-hubs-create.md)
* [事件中樞常見問題集](../event-hubs/event-hubs-faq.md)

<!-- Images. -->
[0]: ../event-hubs/media/event-hubs-streaming-azure-diags-data/dashboard.png
