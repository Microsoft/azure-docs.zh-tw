---
title: 長期函式中的檢查點和重新執行 - Azure
description: 了解如何在 Azure Functions 的「長期函式」延伸模組中進行檢查點檢查和重新執行工作。
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
ms.openlocfilehash: 39cdb9b2c6eae9a3176aedc64b8d187e298fdfdd
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33764565"
---
# <a name="checkpoints-and-replay-in-durable-functions-azure-functions"></a>長期函式中的檢查點和重新執行 (Azure Functions)

長期函式的其中一個索引鍵屬性是**可靠地執行**。 協調器函式和活動函式可能會在資料中心的不同 VM 上執行，這些 VM 或基礎網路基礎結構並非 100% 可靠。

儘管如此，長期函式可確保協調流程可靠地執行。 它是藉由使用儲存體佇列來驅動函式引動，以及將定期檢查點檢查執行歷程記錄記入儲存體資料表 (使用雲端設計模式，稱為[事件來源](https://docs.microsoft.com/azure/architecture/patterns/event-sourcing))，來完成這項作業。 然後該歷程記錄會重新執行以自動重建協調器函式的記憶體內部狀態。

## <a name="orchestration-history"></a>協調流程記錄

假設您有下列協調器函式：

#### <a name="c"></a>C#

```csharp
[FunctionName("E1_HelloSequence")]
public static async Task<List<string>> Run(
    [OrchestrationTrigger] DurableOrchestrationContext context)
{
    var outputs = new List<string>();

    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Tokyo"));
    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Seattle"));
    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "London"));

    // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
    return outputs;
}
```

#### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

```javascript
const df = require("durable-functions");

module.exports = df(function*(context) {
    const output = [];
    output.push(yield context.df.callActivityAsync("E1_SayHello", "Tokyo"));
    output.push(yield context.df.callActivityAsync("E1_SayHello", "Seattle"));
    output.push(yield context.df.callActivityAsync("E1_SayHello", "London"));

    return output;
});
```

在每個 `await` (C#) 或 `yield` 陳述式中，長期工作架構會將函式的執行狀態檢查點記入資料表儲存體。 此狀態就是「協調流程歷程記錄」。

## <a name="history-table"></a>歷程記錄資料表

一般而言，長期工作架構會在每個檢查點執行下列作業：

1. 將執行歷程記錄儲存到 Azure 儲存體資料表。
2. 將協調器想要叫用之函式的訊息加入佇列。
3. 將協調器本身的訊息加入佇列，例如，長期計時器訊息。&mdash;

一旦檢查點完成，協調器函式就可以安心地從記憶體移除，直到有更多工作要執行。

> [!NOTE]
> Azure 儲存體不提供將資料儲存至資料表儲存體和佇列之間的任何交易式保證。 為了處理失敗，長期函式儲存體提供者會使用「最終一致性」模式。 這些模式可確保如果在檢查點中間發生損毀或連線中斷，不會遺失任何資料。

完成時，稍早顯示的函式歷程記錄在 Azure 資料表儲存體中看起來如下所示 (針對示範目的縮寫)：

| PartitionKey (InstanceId)                     | EventType             | Timestamp               | 輸入 | Name             | 結果                                                    | 狀態 | 
|----------------------------------|-----------------------|----------|--------------------------|-------|------------------|-----------------------------------------------------------|---------------------| 
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:32.362Z |       |                  |                                                           |                     | 
| eaee885b | ExecutionStarted      | 2017-05-05T18:45:28.852Z | null  | E1_HelloSequence |                                                           |                     | 
| eaee885b | TaskScheduled         | 2017-05-05T18:45:32.670Z |       | E1_SayHello      |                                                           |                     | 
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:32.670Z |       |                  |                                                           |                     | 
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:34.232Z |       |                  |                                                           |                     | 
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.201Z |       |                  | """Hello Tokyo!"""                                        |                     | 
| eaee885b | TaskScheduled         | 2017-05-05T18:45:34.435Z |       | E1_SayHello      |                                                           |                     | 
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:34.435Z |       |                  |                                                           |                     | 
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:34.857Z |       |                  |                                                           |                     | 
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.763Z |       |                  | """Hello Seattle!"""                                      |                     | 
| eaee885b | TaskScheduled         | 2017-05-05T18:45:34.857Z |       | E1_SayHello      |                                                           |                     | 
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:34.857Z |       |                  |                                                           |                     | 
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:35.032Z |       |                  |                                                           |                     | 
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.919Z |       |                  | """Hello London!"""                                       |                     | 
| eaee885b | ExecutionCompleted    | 2017-05-05T18:45:35.044Z |       |                  | "[""Hello Tokyo!"",""Hello Seattle!"",""Hello London!""]" | Completed           | 
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:35.044Z |       |                  |                                                           |                     | 

資料行值的一些附註：
* **PartitionKey**：包含協調流程的執行個體識別碼。
* **EventType**：代表事件的類型。 可以是下列其中一個類型：
    * **OrchestrationStarted**：協調器函式會從等候繼續運作或是第一次執行。 `Timestamp` 資料行是用來填入 [CurrentUtcDateTime](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CurrentUtcDateTime) API 的決定性值。
    * **ExecutionStarted**：協調器函式第一次開始執行。 此事件也包含 `Input` 資料行中的函式輸入。
    * **TaskScheduled**：活動函式已排程。 活動函式的名稱是在 `Name` 資料行中擷取。
    * **TaskCompleted**：活動函式已完成。 函式的結果是在 `Result` 資料行中。
    * **TimerCreated**：建立長期計時器。 `FireAt` 資料行包含計時器到期的已排程 UTC 時間。
    * **TimerFired**：長期計時器已引發。
    * **EventRaised**：外部事件傳送至協調流程執行個體。 `Name` 資料行會擷取事件的名稱，而 `Input` 資料行會擷取事件的裝載。
    * **OrchestratorCompleted**：協調器函式已等候。
    * **ContinueAsNew**：協調器函式已完成，並且以新的狀態自行重新啟動。 `Result` 資料行包含值，可作為重新啟動執行個體的輸入。
    * **ExecutionCompleted**：協調器函式即將完成 (或失敗)。 函式或錯誤詳細資料的輸出會儲存在 `Result` 資料行。
* **Timestamp**：歷程記錄事件的 UTC 時間戳記。
* **Name**：被叫用之函式的名稱。
* **Input**：函式的 JSON 格式輸入。
* **Result**：函式的輸出，也就是它的傳回值。

> [!WARNING]
> 它作為偵錯工具相當有用，請勿在此資料表上採用任何相依性。 當 Durable Functions 擴充功能進化時，此相依性可能會改變。

每次函式從 `await` 繼續時，長期工作架構就會從頭重新執行協調器函式。 在每次重新執行時，它會查詢執行歷程記錄以判斷是否已發生目前的非同步作業。  如果作業已發生，架構會立即重新執行該作業的輸出，並且移至下一個 `await`。 此程序會繼續直到整個歷程記錄已重新執行，此時協調器函式中的所有本機變數會還原至其先前的值。

## <a name="orchestrator-code-constraints"></a>協調器程式碼條件約束

重新執行行為會建立可以在協調器函式撰寫之程式碼的類型條件約束：

* 協調器程式碼必須是**決定性**。 它會重新執行多次，每次都必須產生相同的結果。 例如，不直接呼叫以取得目前日期/時間、取得隨機數字、產生隨機 GUID，或呼叫遠端端點。

  如果協調器程式碼需要取得目前日期/時間，則應該使用 [CurrentUtcDateTime](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CurrentUtcDateTime) API，它對於重新執行是安全的。

  非決定性作業必須在活動函式中完成。 這包括與其他輸入或輸出繫結的任何互動。 這可確保任何非決定性值會在第一次執行時產生，並且儲存到執行歷程記錄。 然後，後續執行會自動使用儲存的值。

* 協調器程式碼應該是**非封鎖**。 例如，這表示沒有 `Thread.Sleep` 或對等 API 的 I/O 和呼叫。

  如果協調器需要延遲，可以使用 [CreateTimer](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CreateTimer_) API。

* 協調器程式碼必須**永不起始任何非同步作業**，除非使用 [DurableOrchestrationContext](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html) API。 例如，無 `Task.Run`、`Task.Delay` 或 `HttpClient.SendAsync`。 長期工作架構會在單一執行緒上執行協調器程式碼，並且無法與可以由其他非同步 API 排程的任何其他執行緒進行互動。

* 在協調器程式碼中**應該避免無限迴圈**。 由於長期工作架構會在協調流程函式進行時儲存執行歷程記錄，所以無限迴圈可能會造成協調器執行個體用盡記憶體。 在無限迴圈的案例中，使用例如 [ContinueAsNew](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_ContinueAsNew_) 的 API 以重新啟動函式執行，並捨棄先前的執行歷程記錄。

雖然這些條件約束乍看之下令人卻步，但是實際上遵循它們並不困難。 長期工作架構嘗試偵測上述規則的違規，並且擲回 `NonDeterministicOrchestrationException`。 不過，這個偵測行為是最佳方式，您不應該仰賴它。

> [!NOTE]
> 所有規則僅適用於 `orchestrationTrigger` 繫結所觸發的函式。 `activityTrigger` 繫結所觸發的活動函式，和使用 `orchestrationClient` 繫結的函式，沒有這類限制。

## <a name="durable-tasks"></a>長期工作

> [!NOTE]
> 本章節描述長期工作架構的內部實作詳細資料。 您不需要知道這項資訊就可以使用長期函式。 它只是用來協助您了解重新執行行為。

可以在協調器函式中安全地等候的工作偶爾會稱為「長期工作」。 這些工作是由長期工作架構建立和管理的工作。 範例是由 `CallActivityAsync`、`WaitForExternalEvent` 和 `CreateTimer` 傳回的工作。

這些「長期工作」是藉由使用 `TaskCompletionSource` 物件的清單在內部進行管理。 在重新執行期間，這些工作會建立為協調器程式碼執行的一部分，並且在發送器列舉對應歷程記錄事件時完成。 這項作業是以同步方式使用單一執行緒來完成，直到已重新執行所有歷程記錄。 歷程記錄重新執行結束時未完成的任何長期工作都有適當的執行動作。例如，訊息可以會被加入佇列以呼叫活動函式。

此處所述的執行行為應該可以協助您了解為什麼協調器函式程式碼不得為 `await` 非長期工作：發送器執行緒無法等候它完成，該工作的任何回呼都有可能會損毀協調器函式的追蹤狀態。 某些執行階段檢查正在進行中，以嘗試防止這個情況。

如何您想要關於長期工作架構如何執行協調器函式的詳細資訊，最好的方式是參閱 [GitHub 上的長期工作原始程式碼](https://github.com/Azure/durabletask)。 請特別參閱 [TaskOrchestrationExecutor.cs](https://github.com/Azure/durabletask/blob/master/src/DurableTask.Core/TaskOrchestrationExecutor.cs) 和 [TaskOrchestrationContext.cs](https://github.com/Azure/durabletask/blob/master/src/DurableTask.Core/TaskOrchestrationContext.cs)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解執行個體管理](durable-functions-instance-management.md)
