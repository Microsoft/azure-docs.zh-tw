---
title: 長期函式中的 HTTP API - Azure
description: 了解如何在 Azure Functions 的「長期函式」延伸模組中實作 HTTP API。
services: functions
author: cgillum
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 09/29/2017
ms.author: azfuncdf
ms.openlocfilehash: 3c000e268c4c926991c3f1928f226065a436c6d2
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "36264880"
---
# <a name="http-apis-in-durable-functions-azure-functions"></a>長期函式中的 HTTP API (Azure Functions)

「長期工作」延伸模組會公開一組 HTTP API，可以用來執行下列工作：

* 擷取協調流程執行個體的狀態。
* 傳送事件給等待中協調流程執行個體。
* 終止執行中協調流程執行個體。


每個 HTTP API 都是 Webhook 作業，由「長期工作」擴充功能直接處理。 它們並非函式應用程式中任何函式的特定項目。

> [!NOTE]
> 這些作業也可以透過使用 [DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) 類別上的執行個體管理 API 來直接叫用。 如需詳細資訊，請參閱[執行個體管理](durable-functions-instance-management.md)。

## <a name="http-api-url-discovery"></a>HTTP API URL 探索

[DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) 類別會公開 [CreateCheckStatusResponse](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_CreateCheckStatusResponse_) API，它可以用來產生 HTTP 回應裝載，其中包含所有支援作業的連結。 以下是範例 HTTP 觸發函式，它會示範如何使用這個 API：

[!code-csharp[Main](~/samples-durable-functions/samples/csx/HttpStart/run.csx)]

這個範例函式會產生下列 JSON 回應資料。 所有欄位的資料類型是 `string`。

| 欄位             |說明                           |
|-------------------|--------------------------------------|
| id                |協調流程執行個體的識別碼。 |
| statusQueryGetUri |協調流程執行個體的狀態 URL。 |
| sendEventPostUri  |協調流程執行個體的「引發事件」URL。 |
| terminatePostUri  |協調流程執行個體的「終止」URL。 |

以下是範例回應：

```http
HTTP/1.1 202 Accepted
Content-Length: 923
Content-Type: application/json; charset=utf-8
Location: https://{host}/runtime/webhooks/DurableTaskExtension/instances/34ce9a28a6834d8492ce6a295f1a80e2?taskHub=DurableFunctionsHub&connection=Storage&code=XXX

{
    "id":"34ce9a28a6834d8492ce6a295f1a80e2",
    "statusQueryGetUri":"https://{host}/runtime/webhooks/DurableTaskExtension/instances/34ce9a28a6834d8492ce6a295f1a80e2?taskHub=DurableFunctionsHub&connection=Storage&code=XXX",
    "sendEventPostUri":"https://{host}/runtime/webhooks/DurableTaskExtension/instances/34ce9a28a6834d8492ce6a295f1a80e2/raiseEvent/{eventName}?taskHub=DurableFunctionsHub&connection=Storage&code=XXX",
    "terminatePostUri":"https://{host}/runtime/webhooks/DurableTaskExtension/instances/34ce9a28a6834d8492ce6a295f1a80e2/terminate?reason={text}&taskHub=DurableFunctionsHub&connection=Storage&code=XXX"
}
```
> [!NOTE]
> Webhook URL 的格式依據您執行的 Azure Functions 主機版本，可能會有所不同。 上述範例適用於 Azure Functions 2.0 主機。

## <a name="async-operation-tracking"></a>非同步作業追蹤

先前所述之 HTTP 回應的設計目的，是協助搭配長期函式實作長時間執行 HTTP 非同步 API。 有時候稱為「輪詢取用者模式」。 用戶端/伺服器流量運作方式如下：

1. 用戶端會發出 HTTP 要求，以啟動長時間執行處理序，例如協調器函式。
2. 目標 HTTP 觸發程序會傳回 HTTP 202 回應，具有 `Location` 標頭和 `statusQueryGetUri` 值。
3. 用戶端會輪詢 `Location` 標頭中的 URL。 它會繼續查看具有 `Location` 標頭的 HTTP 202 回應。
4. 當執行個體完成 (或失敗) 時，`Location` 標頭中的端點會傳回 HTTP 200。

此通訊協定可以協調長時間執行處理序與外部用戶端或服務，它們支援輪詢 HTTP 端點，並且遵循 `Location` 標頭。 基本部分已內建至長期函式 HTTP API。

> [!NOTE]
> 根據預設，[Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/) 提供的所有 HTTP 型動作皆支援標準的非同步作業模式。 這項功能可以嵌入長期執行長期函式，作為 Logic Apps 工作流程的一部分。 Logic Apps 支援非同步 HTTP 模式的詳細資訊，可以在 [Azure Logic Apps 工作流程動作和觸發程序文件](../logic-apps/logic-apps-workflow-actions-triggers.md#asynchronous-patterns)中找到。

## <a name="http-api-reference"></a>HTTP API 參考

延伸模組實作的所有 HTTP API 會採用下列參數。 所有參數的資料類型是 `string`。

| 參數  | 參數類型  | 說明 |
|------------|-----------------|-------------|
| instanceId | URL             | 協調流程執行個體的識別碼。 |
| taskHub    | 查詢字串    | [工作中樞](durable-functions-task-hubs.md)的名稱。 如果未指定，則會假設為目前函式應用程式的工作中樞名稱。 |
| connection | 查詢字串    | 儲存體帳戶之連接字串的**名稱**。 如果未指定，則會假設為函式應用程式的預設連接字串。 |
| systemKey  | 查詢字串    | 叫用 API 所需的授權金鑰。 |
| showHistory| 查詢字串    | 選擇性參數。 如果設定為 `true`，則協調流程執行歷程記錄將會包含在回應承載中。| 
| showHistoryOutput| 查詢字串    | 選擇性參數。 如果設定為 `true`，則活動輸出將會包含在協調流程執行歷程記錄中。| 

`systemKey` 是由 Azure Functions 主機自動產生的授權金鑰。 它特別授與「長期工作」延伸模組 API 的存取權，並且可以用與[其他授權金鑰](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Key-management-API)相同的方式來管理。 探索 `systemKey` 值的最簡單方式是使用先前所述的 `CreateCheckStatusResponse` API。

接下來的幾個章節會涵蓋延伸模組支援的特定 HTTP API，並且提供如何使用的範例。

### <a name="get-instance-status"></a>取得執行個體狀態

取得指定協調流程執行個體的狀態。

#### <a name="request"></a>要求

對於 Functions 1.0，要求格式如下：

```http
GET /admin/extensions/DurableTaskExtension/instances/{instanceId}?taskHub={taskHub}&connection={connection}&code={systemKey}
```

Functions 2.0 格式具有完全相同的參數，但是 URL 首碼稍微不同：

```http
GET /runtime/webhooks/DurableTaskExtension/instances/{instanceId}?taskHub={taskHub}&connection={connection}&code={systemKey}&showHistory={showHistory}&showHistoryOutput={showHistoryOutput}
```

#### <a name="response"></a>Response

可以傳回幾個可能的狀態字碼值。

* **HTTP 200 (確定)**：指定的執行個體處於已完成狀態。
* **HTTP 202 (已接受)**：指定的執行個體正在進行中。
* **HTTP 400 (不正確的要求)**：指定的執行個體失敗或終止。
* **HTTP 404 (找不到)**：指定的執行個體不存在或尚未開始執行。

**HTTP 200** 和 **HTTP 202** 案例的回應承載是 JSON 物件，具有下列欄位：

| 欄位           | 資料類型 | 說明 |
|-----------------|-----------|-------------|
| runtimeStatus   | 字串    | 執行個體的執行階段狀態。 值包括 [執行中]、[擱置]、[失敗]、[已取消]、[終止]、[已完成]。 |
| input           | JSON      | JSON 資料，用來初始化執行個體。 |
| customStatus    | JSON      | 用於自訂協調流程狀態的 JSON 資料。 如果未設定，欄位會是 `null`。 |
| output          | JSON      | 執行個體的 JSON 輸出。 如果執行個體不是已完成狀態，則這個欄位是 `null`。 |
| createdTime     | 字串    | 執行個體建立的時間。 使用 ISO 8601 延伸標記法。 |
| lastUpdatedTime | 字串    | 執行個體保存的時間。 使用 ISO 8601 延伸標記法。 |
| historyEvents   | JSON      | 包含協調流程執行歷程記錄的 JSON 陣列。 這個欄位為 `null`，除非 `showHistory` 查詢字串參數設定為 `true`。  | 

以下是範例回應承載，其中包含協調流程執行歷程記錄和活動輸出 (針對可讀性格式化)：

```json
{
  "createdTime": "2018-02-28T05:18:49Z",
  "historyEvents": [
      {
          "EventType": "ExecutionStarted",
          "FunctionName": "E1_HelloSequence",
          "Timestamp": "2018-02-28T05:18:49.3452372Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello Tokyo!",
          "ScheduledTime": "2018-02-28T05:18:51.3939873Z",
          "Timestamp": "2018-02-28T05:18:52.2895622Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello Seattle!",
          "ScheduledTime": "2018-02-28T05:18:52.8755705Z",
          "Timestamp": "2018-02-28T05:18:53.1765771Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello London!",
          "ScheduledTime": "2018-02-28T05:18:53.5170791Z",
          "Timestamp": "2018-02-28T05:18:53.891081Z"
      },
      {
          "EventType": "ExecutionCompleted",
          "OrchestrationStatus": "Completed",
          "Result": [
              "Hello Tokyo!",
              "Hello Seattle!",
              "Hello London!"
          ],
          "Timestamp": "2018-02-28T05:18:54.3660895Z"
      }
  ],
  "input": null,
  "customStatus": { "nextActions": ["A", "B", "C"], "foo": 2 },
  "lastUpdatedTime": "2018-02-28T05:18:54Z",
  "output": [
      "Hello Tokyo!",
      "Hello Seattle!",
      "Hello London!"
  ],
  "runtimeStatus": "Completed"
}
```

**HTTP 202** 回應也包含**位置**回應標頭，它參考與先前所述 `statusQueryGetUri` 欄位相同的 URL。

### <a name="get-all-instances-status"></a>取得所有執行個體狀態

您也可以查詢所有執行個體狀態。 從「取得執行個體狀態」要求中移除 `instanceId`。 參數與「取得執行個體狀態」相同。 

#### <a name="request"></a>要求

對於 Functions 1.0，要求格式如下：

```http
GET /admin/extensions/DurableTaskExtension/instances/?taskHub={taskHub}&connection={connection}&code={systemKey}
```

Functions 2.0 格式具有完全相同的參數，但是 URL 首碼稍微不同： 

```http
GET /runtime/webhooks/DurableTaskExtension/instances/?taskHub={taskHub}&connection={connection}&code={systemKey}
```

#### <a name="response"></a>Response

以下是回應承載範例，包括協調流程狀態 (針對可讀性格式化)：

```json
[
    {
        "instanceId": "7af46ff000564c65aafbfe99d07c32a5",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2018-06-04T10:46:39Z",
        "lastUpdatedTime": "2018-06-04T10:46:47Z"
    },
    {
        "instanceId": "80eb7dd5c22f4eeba9f42b062794321e",
        "runtimeStatus": "Running",
        "input": null,
        "customStatus": null,
        "output": null,
        "createdTime": "2018-06-04T15:18:28Z",
        "lastUpdatedTime": "2018-06-04T15:18:38Z"
    },
    {
        "instanceId": "9124518926db408ab8dfe84822aba2b1",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2018-06-04T10:46:54Z",
        "lastUpdatedTime": "2018-06-04T10:47:03Z"
    },
    {
        "instanceId": "d100b90b903c4009ba1a90868331b11b",
        "runtimeStatus": "Pending",
        "input": null,
        "customStatus": null,
        "output": null,
        "createdTime": "2018-06-04T15:18:39Z",
        "lastUpdatedTime": "2018-06-04T15:18:39Z"
    }
]
```

> [!NOTE]
> 如果執行個體資料表中有大量資料列，則這項作業可能會耗用非常大量 Azure 儲存體 I/O。 在 [Durable Functions (Azure Functions) 中的效能和級別](https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-perf-and-scale#instances-table)文件中可以找到執行個體資料表的更多詳細資料。
> 

### <a name="raise-event"></a>引發事件

將事件通知訊息傳送至執行中協調流程執行個體。

#### <a name="request"></a>要求

對於 Functions 1.0，要求格式如下：

```http
POST /admin/extensions/DurableTaskExtension/instances/{instanceId}/raiseEvent/{eventName}?taskHub=DurableFunctionsHub&connection={connection}&code={systemKey}
```

Functions 2.0 格式具有完全相同的參數，但是 URL 首碼稍微不同：

```http
POST /runtime/webhooks/DurableTaskExtension/instances/{instanceId}/raiseEvent/{eventName}?taskHub=DurableFunctionsHub&connection={connection}&code={systemKey}
```

此 API 的要求參數包含先前所述的預設集合，以及下列的唯一參數：

| 欄位       | 參數類型  | 資料類型 | 說明 |
|-------------|-----------------|-----------|-------------|
| eventName   | URL             | 字串    | 目標協調流程執行個體等候之事件的名稱。 |
| {content}   | 要求內容 | JSON      | JSON 格式的事件裝載。 |

#### <a name="response"></a>Response

可以傳回幾個可能的狀態字碼值。

* **HTTP 202 (已接受)**：已接受引發的事件以進行處理。
* **HTTP400 (不正確的要求)**：要求內容不是類型 `application/json` 或不是有效的 JSON。
* **HTTP 404 (找不到)**：找不到指定的執行個體。
* **HTTP 410 (不存在)**：指定的執行個體已完成或失敗，且無法再處理任何引發的事件。

以下是範例要求，它會將 JSON 字串 `"incr"` 傳送給等候名為**作業** 之事件的執行個體：

```
POST /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/raiseEvent/operation?taskHub=DurableFunctionsHub&connection=Storage&code=XXX
Content-Type: application/json
Content-Length: 6

"incr"
```

此 API 的回應不包含任何內容。

### <a name="terminate-instance"></a>終止執行個體

終止執行中協調流程執行個體。

#### <a name="request"></a>要求

對於 Functions 1.0，要求格式如下：

```http
DELETE /admin/extensions/DurableTaskExtension/instances/{instanceId}/terminate?reason={reason}&taskHub={taskHub}&connection={connection}&code={systemKey}
```

Functions 2.0 格式具有完全相同的參數，但是 URL 首碼稍微不同：

```http
DELETE /runtime/webhooks/DurableTaskExtension/instances/{instanceId}/terminate?reason={reason}&taskHub={taskHub}&connection={connection}&code={systemKey}
```

此 API 的要求參數包含先前所述的預設集合，以及下列的唯一參數。

| 欄位       | 參數類型  | 資料類型 | 說明 |
|-------------|-----------------|-----------|-------------|
| 原因      | 查詢字串    | 字串    | 選用。 終止協調流程執行個體的原因。 |

#### <a name="response"></a>Response

可以傳回幾個可能的狀態字碼值。

* **HTTP 202 (已接受)**：已接受終止要求以進行處理。
* **HTTP 404 (找不到)**：找不到指定的執行個體。
* **HTTP 410 (不存在)**：指定的執行個體已完成或失敗。

以下是範例要求，它會終止執行中執行個體並且指定**錯誤**的原因：

```
DELETE /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/terminate?reason=buggy&taskHub=DurableFunctionsHub&connection=Storage&code=XXX
```

此 API 的回應不包含任何內容。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何處理錯誤](durable-functions-error-handling.md)
