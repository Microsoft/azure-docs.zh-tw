---
title: Azure Functions 的 host.json 參考
description: Azure Functions host.json 檔案的參考文件。
services: functions
author: ggailey777
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 02/12/2018
ms.author: glenga
ms.openlocfilehash: 9043add91022c2829c305425dba9c8f11b224fcf
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39345509"
---
# <a name="hostjson-reference-for-azure-functions"></a>Azure Functions 的 host.json 參考

*host.json* 中繼資料檔案所包含的全域設定選項會影響函式應用程式的所有函式。 本文列出可用的設定。 JSON 結構描述位於 http://json.schemastore.org/host。

在[應用程式設定](functions-app-settings.md)和 [local.settings.json](functions-run-local.md#local-settings-file) 檔案中，還有其他全域設定選項。

## <a name="sample-hostjson-file"></a>範例 host.json 檔案

下列範例 *host.json* 檔案已指定所有可能的選項。

```json
{
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    },
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    },
    "eventHub": {
      "maxBatchSize": 64,
      "prefetchCount": 256,
      "batchCheckpointFrequency": 1
    },
    "functions": [ "QueueProcessor", "GitHubWebHook" ],
    "functionTimeout": "00:05:00",
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    },
    "http": {
        "routePrefix": "api",
        "maxOutstandingRequests": 20,
        "maxConcurrentRequests": 10,
        "dynamicThrottlesEnabled": false
    },
    "id": "9f4ea53c5136457d883d685e57164f08",
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    },
    "queues": {
      "maxPollingInterval": 2000,
      "visibilityTimeout" : "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    },
    "serviceBus": {
      "maxConcurrentCalls": 16,
      "prefetchCount": 100,
      "autoRenewTimeout": "00:05:00"
    },
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    },
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    },
    "watchDirectories": [ "Shared" ],
}
```

本文的下列各節說明每個最上層屬性。 除非另有說明，否則全部都是選擇項目。

## <a name="aggregator"></a>aggregator

指定[計算 Application Insights 的計量](functions-monitoring.md#configure-the-aggregator)時彙總多少函式引動過程。 

```json
{
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    }
}
```

|屬性 |預設值  | 說明 |
|---------|---------|---------| 
|batchSize|1000|要彙總的要求數目上限。| 
|flushTimeout|00:00:30|要彙總的最長期間。| 

達到兩個限制的第一個限制時，會彙總函式引動過程。

## <a name="applicationinsights"></a>applicationInsights

控制 [Application Insights 中的取樣功能](functions-monitoring.md#configure-sampling)。

```json
{
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    }
}
```

|屬性  |預設值 | 說明 |
|---------|---------|---------| 
|isEnabled|true|啟用或停用取樣。| 
|maxTelemetryItemsPerSecond|5|取樣的開始臨界值。| 

## <a name="durabletask"></a>durableTask

[Durable Functions](durable-functions-overview.md) 的組態設定。

```json
{
  "durableTask": {
    "HubName": "MyTaskHub",
    "ControlQueueBatchSize": 32,
    "PartitionCount": 4,
    "ControlQueueVisibilityTimeout": "00:05:00",
    "WorkItemQueueVisibilityTimeout": "00:05:00",
    "MaxConcurrentActivityFunctions": 10,
    "MaxConcurrentOrchestratorFunctions": 10,
    "AzureStorageConnectionStringName": "AzureWebJobsStorage",
    "TraceInputsAndOutputs": false,
    "LogReplayEvents": false,
    "EventGridTopicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
    "EventGridKeySettingName":  "EventGridKey",
    "EventGridPublishRetryCount": 3,
    "EventGridPublishRetryInterval": "00:00:30"
  }
}
```

工作中樞名稱必須以字母開頭，且只包含字母和數字。 如果未指定，函式應用程式的預設工作中樞名稱是 **DurableFunctionsHub**。 如需詳細資訊，請參閱[工作中樞](durable-functions-task-hubs.md)。

|屬性  |預設值 | 說明 |
|---------|---------|---------|
|HubName|DurableFunctionsHub|替代[工作中樞](durable-functions-task-hubs.md)名稱可用來彼此隔離多個 Durable Functions 應用程式，即使它們使用相同的儲存體後端。|
|ControlQueueBatchSize|32|要從控制佇列中一次提取的訊息數。|
|PartitionCount |4|控制佇列的資料分割計數。 必須是介於 1 到 16 之間的正整數。|
|ControlQueueVisibilityTimeout |5 分鐘|已從控制佇列中清除之訊息的可見度逾時。|
|WorkItemQueueVisibilityTimeout |5 分鐘|已從工作項目佇列中清除之訊息的可見度逾時。|
|MaxConcurrentActivityFunctions |目前電腦上的 10 倍處理器數目|可以在單一主機執行個體上同時處理的活動函式數目上限。|
|MaxConcurrentOrchestratorFunctions |目前電腦上的 10 倍處理器數目|可以在單一主機執行個體上同時處理的活動函式數目上限。|
|AzureStorageConnectionStringName |AzureWebJobsStorage|具有 Azure 儲存體連接字串的應用程式設定名稱，而該連接字串用來管理基礎的 Azure 儲存體資源。|
|TraceInputsAndOutputs |false|此值指出是否要追蹤函式呼叫的輸入和輸出。 追蹤函式執行事件時的預設行為就是在函式呼叫的序列化輸入和輸出中包含位元組數目。 這可提供輸入和輸出的最基本資訊，而不會讓記錄變大，或者不小心在記錄中公開敏感性資訊。 將此屬性設定為 true，會導致預設函式記錄功能記錄函式輸入和輸出的整個內容。|
|LogReplayEvents|false|此值可指出是否要將協調流程重新執行事件寫入 Application Insights。|
|EventGridTopicEndpoint ||Azure 事件方格自訂主題端點的 URL。 若已設定這個屬性，協調流程生命週期通知事件就會發佈到此端點。 這個屬性支援應用程式設定解析。|
|EventGridKeySettingName ||應用程式設定的名稱，其中包含在 `EventGridTopicEndpoint` 用來向 Azure 事件方格自訂主題進行驗證的金鑰。|
|EventGridPublishRetryCount|0|如果發佈到 Event Grid 主題失敗，重試的次數。|
|EventGridPublishRetryInterval|5 分鐘|Event Grid 會以 *hh:mm:ss* 格式發佈重試間隔。|

上述許多屬性適用於將效能最佳化。 如需詳細資訊，請參閱[效能和級別](durable-functions-perf-and-scale.md)。

## <a name="eventhub"></a>eventHub

[事件中樞觸發程序和繫結](functions-bindings-event-hubs.md)的組態設定。

[!INCLUDE [functions-host-json-event-hubs](../../includes/functions-host-json-event-hubs.md)]

## <a name="functions"></a>functions

工作主機將執行的函式清單。 空陣列表示已執行所有函式。 預定只能在[本機執行](functions-run-local.md)時使用。 在函式應用程式中，使用 *function.json* `disabled` 屬性，而不是 *host.json* 中的這個屬性。

```json
{
    "functions": [ "QueueProcessor", "GitHubWebHook" ]
}
```

## <a name="functiontimeout"></a>functionTimeout

指出所有函式的逾時持續期間。 在取用量方案中，有效範圍是從 1 秒到 10 分鐘，而預設值是 5 分鐘。 在 App Service 方案中，沒有限制，而且預設值為 Null，這指出沒有逾時。

```json
{
    "functionTimeout": "00:05:00"
}
```

## <a name="healthmonitor"></a>healthMonitor

[主機健康情況監視器](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Host-Health-Monitor)的組態設定。

```
{
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    }
}
```

|屬性  |預設值 | 說明 |
|---------|---------|---------| 
|已啟用|true|是否啟用此功能。 | 
|healthCheckInterval|10 秒|定期背景健康情況檢查之間的時間間隔。 | 
|healthCheckWindow|2 分鐘|與 `healthCheckThreshold` 設定搭配使用的滑動時間範圍。| 
|healthCheckThreshold|6|在主機回收起始之前，健康情況檢查可以失敗的最大次數。| 
|counterThreshold|0.80|系統會將效能計數器視為狀況不良的閾值。| 

## <a name="http"></a>http

[HTTP 觸發程序和繫結](functions-bindings-http-webhook.md)的組態設定。

[!INCLUDE [functions-host-json-http](../../includes/functions-host-json-http.md)]

## <a name="id"></a>id

作業主機的唯一識別碼。 可以是已移除虛線的小寫 GUID。 在本機執行時為必要項目。 在 Azure Functions 中執行時，如果省略 `id`，則會自動產生識別碼。

如果您在多個函數應用程式中共用儲存體帳戶，請確定每個函數應用程式具有不同的 `id`。 您可以省略 `id` 屬性或將每個函數應用程式的 `id` 手動設定為不同的值。 計時器觸發程序會使用儲存體鎖定，以確保當函數應用程式相應放大至多個執行個體時，只會有一個計時器執行個體。 如果兩個函數應用程式共用相同的 `id`，且每一個都是使用計時器觸發程序，則只有一個計時器會執行。


```json
{
    "id": "9f4ea53c5136457d883d685e57164f08"
}
```

## <a name="logger"></a>logger

控制 [ILogger 物件](functions-monitoring.md#write-logs-in-c-functions)或 [context.log](functions-monitoring.md#write-logs-in-javascript-functions) 所寫入記錄的篩選。

```json
{
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    }
}
```

|屬性  |預設值 | 說明 |
|---------|---------|---------| 
|categoryFilter|n/a|指定依類別的篩選| 
|defaultLevel|資訊|針對 `categoryLevels` 陣列中未指定的任何類別，會將這個層級和以上層級的記錄傳送至 Application Insights。| 
|categoryLevels|n/a|一個類別陣列，指定針對每個類別傳送至 Application Insights 的最小記錄層級。 這裡指定的類別控制所有開頭為相同值的類別，但會優先使用較長的值。 在上述範例 *host.json* 檔案中，所有開頭為 "Host.Aggregator" 的類別都會記錄在 `Information` 層級。 所有開頭為 "Host" 的其他類別 (例如 "Host.Executor") 都會記錄於 `Error` 層級。| 

## <a name="queues"></a>queues

[儲存體佇列觸發程序和繫結](functions-bindings-storage-queue.md)的組態設定。

[!INCLUDE [functions-host-json-queues](../../includes/functions-host-json-queues.md)]

## <a name="servicebus"></a>serviceBus

[服務匯流排觸發程序和繫結](functions-bindings-service-bus.md)的組態設定。

[!INCLUDE [functions-host-json-service-bus](../../includes/functions-host-json-service-bus.md)]

## <a name="singleton"></a>singleton

Singleton 鎖定行為的組態設定。 如需詳細資訊，請參閱[單一支援的 GitHub 問題](https://github.com/Azure/azure-webjobs-sdk-script/issues/912)。

```json
{
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    }
}
```

|屬性  |預設值 | 說明 |
|---------|---------|---------| 
|lockPeriod|00:00:15|取得函式層級鎖定的期間。 鎖定會自動更新。| 
|listenerLockPeriod|00:01:00|接聽程式鎖定所需的期間。| 
|listenerLockRecoveryPollingInterval|00:01:00|啟動時無法取得接聽程式鎖定時，用於接聽程式鎖定復原的時間間隔。| 
|lockAcquisitionTimeout|00:01:00|執行階段將嘗試取得鎖定的時間量上限。| 
|lockAcquisitionPollingInterval|n/a|鎖定取得嘗試之間的間隔。| 

## <a name="tracing"></a>tracing

使用 `TraceWriter` 物件所建立記錄的組態設定。 請參閱 [C# 記錄](functions-reference-csharp.md#logging)和 [Node.js 記錄](functions-reference-node.md#writing-trace-output-to-the-console)。 

```json
{
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    }
}
```

|屬性  |預設值 | 說明 |
|---------|---------|---------| 
|consoleLevel|info|主控台記錄的追蹤層級。 選項為：`off`、`error`、`warning`、`info` 和 `verbose`。|
|fileLoggingMode|debugOnly|檔案記錄的追蹤層級。 選項為 `never`、`always`、`debugOnly`。| 

## <a name="watchdirectories"></a>watchDirectories

應該監視其變更的一組[共用程式碼目錄](functions-reference-csharp.md#watched-directories)。  請確定，這些目錄中的程式碼變更時，函式會反映變更。

```json
{
    "watchDirectories": [ "Shared" ]
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何更新 host.json 檔案](functions-reference.md#fileupdate)

> [!div class="nextstepaction"]
> [請參閱環境變數中的全域設定](functions-app-settings.md)
