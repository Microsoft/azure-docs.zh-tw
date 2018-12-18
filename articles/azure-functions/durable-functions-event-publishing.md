---
title: 發佈至 Azure 事件方格 (預覽) 的 Durable Functions
description: 了解如何針對 Durable Functions 設定自動 Azure 事件方格發佈。
services: functions
author: ggailey777
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: glenga
ms.openlocfilehash: 31fe0f85b258524b4ac0af03ec19a91a93e1463b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968864"
---
# <a name="durable-functions-publishing-to-azure-event-grid-preview"></a>發佈至 Azure 事件方格 (預覽) 的 Durable Functions

本文說明如何設定 Azure Durable Functions，將協調流程生命週期事件 (例如已建立、已完成和失敗) 發佈至自訂 [Azure 事件方格主題](https://docs.microsoft.com/azure/event-grid/overview)。 

以下是適合使用此功能的一些案例：

* **類似 DevOps 案例的藍色/綠色部署**：您可能想知道在實作[並存部署策略](https://docs.microsoft.com/azure/azure-functions/durable-functions-versioning#side-by-side-deployments)前是否有任何工作正在執行中。

* **進階監視和診斷支援**：您可以在已針對查詢最佳化的外部存放區 (例如 SQL 資料庫或 CosmosDB) 中，追蹤協調流程狀態資訊。

* **長時間執行的背景活動**：如果您使用 Durable Functions 進行長時間執行的背景活動，這項功能可協助您知道目前的狀態。

## <a name="prerequisites"></a>必要條件

* 在您的 Durable Functions 專案中安裝 [Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask) 1.3.0-rc 或更新版本。
* 安裝 [Azure 儲存體模擬器](https://docs.microsoft.com/azure/storage/common/storage-use-emulator)。
* 安裝 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) 或使用 [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview)

## <a name="create-a-custom-event-grid-topic"></a>建立自訂事件方格主題

從 Durable Functions 建立可供傳送事件的事件方格主題。 下列指示說明如何使用 Azure CLI 建立主題。 如需如何使用 PowerShell 或 Azure 入口網站執行此作業的相關資訊，請參閱下列文章：

* [事件方格快速入門：建立自訂事件 - PowerShell](https://docs.microsoft.com/azure/event-grid/custom-event-quickstart-powershell)
* [事件方格快速入門：建立自訂事件 - Azure 入口網站](https://docs.microsoft.com/azure/event-grid/custom-event-quickstart-portal)

### <a name="create-a-resource-group"></a>建立資源群組

使用 `az group create` 命令建立資源群組。 事件方格目前不支援所有區域。 如需支援區域的相關資訊，請參閱[事件方格概觀](https://docs.microsoft.com/azure/event-grid/overview)。 

```bash
az group create --name eventResourceGroup --location westus2
```

### <a name="create-a-custom-topic"></a>建立自訂主題

事件方格主題會提供使用者定義的端點，作為您發佈事件的目的地。 以主題的唯一名稱取代 `<topic_name>`。 主題名稱必須是唯一的，因為它會變成 DNS 項目。

```bash
az eventgrid topic create --name <topic_name> -l westus2 -g eventResourceGroup 
```

## <a name="get-the-endpoint-and-key"></a>取得端點和金鑰

取得主題的端點。 使用您所選的名稱取代 `<topic_name>`。

```bash
az eventgrid topic show --name <topic_name> -g eventResourceGroup --query "endpoint" --output tsv
```

取得主題金鑰。 使用您所選的名稱取代 `<topic_name>`。

```bash
az eventgrid topic key list --name <topic_name> -g eventResourceGroup --query "key1" --output tsv
```

您現在可以將事件傳送至主題。

## <a name="configure-azure-event-grid-publishing"></a>設定 Azure 事件方格發佈

在 Durable Functions 專案中，尋找 `host.json` 檔案。

在 `durableTask` 屬性中新增 `EventGridTopicEndpoint` 和 `EventGridKeySettingName`。

```json
{
    "durableTask": {
        "EventGridTopicEndpoint": "https://<topic_name>.westus2-1.eventgrid.azure.net/api/events",
        "EventGridKeySettingName": "EventGridKey"
    }
}
```

可能的 Azure 事件方格設定屬性如下所示：

* **EventGridTopicEndpoint** - 事件方格主題的端點。 *%AppSettingName%* 語法可從應用程式設定或環境變數來解析此值。
* **EventGridKeySettingName** - 在您的 Azure Function 上設定的應用程式金鑰。 Durable Functions 會從此值取得事件方格主題金鑰。
* **EventGridPublishRetryCount** - [選擇性] 發佈到事件方格主題失敗時的重試次數。
* **EventGridPublishRetryInterval** - [選擇性] 事件方格發佈重試間隔，格式為 *hh:mm:ss*。 如果未指定，預設的重試間隔為 5 分鐘。

在您設定 `host.json` 檔案後，您的 Durable Functions 專案就會開始將生命週期事件傳送至事件方格主題。 這適用於當您在函式應用程式中執行以及當您在本機執行時。

在函式應用程式和 `local.setting.json` 中設定主題索引鍵的應用程式設定。 以下 JSON 是本機偵錯的 `local.settings.json` 範例。 以主題索引鍵取代 `<topic_key>`。  

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "AzureWebJobsDashboard": "UseDevelopmentStorage=true",
        "EventGridKey": "<topic_key>"
    }
}
```

確定[儲存體模擬器](https://docs.microsoft.com/azure/storage/common/storage-use-emulator)運作正常。 在執行之前，最好先執行 `AzureStorageEmulator.exe clear all` 命令。

## <a name="create-functions-that-listen-for-events"></a>建立可接聽事件的函式

建立函式應用程式。 最好在與事件方格主題的相同區域中找出它。

### <a name="create-an-event-grid-trigger-function"></a>建立事件方格觸發程序函式

建立可接收生命週期事件的函式。 選取 [自訂函式]。 

![選取 [建立自訂函式]。](media/durable-functions-event-publishing/functions-portal.png)

選擇事件方格觸發程序，然後選取 `C#`。

![選取事件方格觸發程序。](media/durable-functions-event-publishing/eventgrid-trigger.png)

輸入函式名稱，然後選取 `Create`。

![選取事件方格觸發程序。](media/durable-functions-event-publishing/eventgrid-trigger-creation.png)

隨即建立具有下列程式碼的函式： 

```csharp
#r "Newtonsoft.Json"
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
public static void Run(JObject eventGridEvent, TraceWriter log)
{
    log.Info(eventGridEvent.ToString(Formatting.Indented));
}
```

選取 `Add Event Grid Subscription`。 此作業會為您建立的事件方格主題新增事件方格訂用帳戶。 如需詳細資訊，請參閱 [Azure 事件方格概念](https://docs.microsoft.com/azure/event-grid/concepts)。

![選取事件方格觸發程序連結。](media/durable-functions-event-publishing/eventgrid-trigger-link.png)

針對 [主題類型]  選取 `Event Grid Topics`。 選取您為事件方格主題所建立的資源群組。 然後選取事件方格主題的執行個體。 按 `Create`。

![建立事件格線訂用帳戶。](media/durable-functions-event-publishing/eventsubscription.png)

您現在已經準備好接收生命週期事件。 

## <a name="create-durable-functions-to-send-the-events"></a>建立 函式 以傳送事件。

在您的 Durable Functions 專案中，開始在本機電腦上進行偵錯。  下列程式碼與 Durable Functions 的範本程式碼相同。 您已經在本機電腦上設定 `host.json` 和 `local.settings.json`。 

```csharp
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;

namespace LifeCycleEventSpike
{
    public static class Sample
    {
    {
        [FunctionName("Sample")]
        public static async Task<List<string>> RunOrchestrator(
            [OrchestrationTrigger] DurableOrchestrationContext context)
        {
            var outputs = new List<string>();

            // Replace "hello" with the name of your Durable Activity Function.
            outputs.Add(await context.CallActivityAsync<string>("Sample_Hello", "Tokyo"));
            outputs.Add(await context.CallActivityAsync<string>("Sample_Hello", "Seattle"));
            outputs.Add(await context.CallActivityAsync<string>("Sample_Hello", "London"));

            // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
            return outputs;
        }

        [FunctionName("Sample_Hello")]
        public static string SayHello([ActivityTrigger] string name, TraceWriter log)
        {
            log.Info($"Saying hello to {name}.");
            return $"Hello {name}!";
        }

        [FunctionName("Sample_HttpStart")]
        public static async Task<HttpResponseMessage> HttpStart(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")]HttpRequestMessage req,
            [OrchestrationClient]DurableOrchestrationClient starter,
            TraceWriter log)
        {
            // Function input comes from the request content.
            string instanceId = await starter.StartNewAsync("Sample", null);
            log.Info($"Started orchestration with ID = '{instanceId}'.");
            return starter.CreateCheckStatusResponse(req, instanceId);
        }
    }
}
```

如果您使用 Postman 或您的瀏覽器呼叫 `Sample_HttpStart`，Durable Function 就會開始傳送生命週期事件。 端點通常是用於本機偵錯的 `http://localhost:7071/api/Sample_HttpStart`。

請參閱您在 Azure 快速入門中建立的函式所提供的記錄。

```
2018-04-20T09:28:21.041 [Info] Function started (Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d)
2018-04-20T09:28:21.104 [Info] {
    "id": "054fe385-c017-4ce3-b38a-052ac970c39d",    
    "subject": "durable/orchestrator/Running",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Running"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2018-04-20T09:28:19.6492068Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}

2018-04-20T09:28:21.104 [Info] Function completed (Success, Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d, Duration=65ms)
2018-04-20T09:28:37.098 [Info] Function started (Id=36fadea5-198b-4345-bb8e-2837febb89a2)
2018-04-20T09:28:37.098 [Info] {
    "id": "8cf17246-fa9c-4dad-b32a-5a868104f17b",
    "subject": "durable/orchestrator/Completed",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Completed"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2018-04-20T09:28:36.5061317Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}
2018-04-20T09:28:37.098 [Info] Function completed (Success, Id=36fadea5-198b-4345-bb8e-2837febb89a2, Duration=0ms)
```

## <a name="event-schema"></a>事件結構描述

下列清單說明生命週期事件結構描述︰

* **id**：事件方格事件的唯一識別碼。
* **subject**：事件主體的路徑。 `durable/orchestrator/{orchestrationRuntimeStatus}` 。 `{orchestrationRuntimeStatus}` 會是 `Running`、`Completed`、`Failed` 和 `Terminated`。  
* **data**：Durable Functions 專屬參數。
    * **hubName**：[TaskHub](https://docs.microsoft.com/azure/azure-functions/durable-functions-task-hubs) 名稱。
    * **functionName**：協調器函式名稱。
    * **instanceId**：Durable Functions 執行個體識別碼。
    * **原因**：與追蹤事件相關聯的其他資料。 如需詳細資訊，請參閱 [Durable Functions 中的診斷 (Azure Functions)](https://docs.microsoft.com/azure/azure-functions/durable-functions-diagnostics)
    * **runtimeStatus**：協調流程執行階段狀態。 執行中、已完成、失敗、已取消。 
* **eventType**："orchestratorEvent"
* **eventTime**：事件時間 (UTC)。
* **dataVersion**：生命週期事件結構描述的版本。
* **metadataVersion**：中繼資料的版本。
* **topic**：事件方格主題資源。

## <a name="how-to-test-locally"></a>本機測試方式

若要在本機測試，請使用 [ngrok](functions-bindings-event-grid.md#local-testing-with-ngrok)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解 Durable Functions 中的執行個體管理](durable-functions-instance-management.md)

> [!div class="nextstepaction"]
> [了解 Durable Functions 中的版本控制](durable-functions-versioning.md)
