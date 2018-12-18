---
title: Durable Functions 函式類型與功能概觀 - Azure
description: 了解函式類型和角色，其可讓函式對函式通訊成為長期函式協調流程的一部分。
services: functions
author: jeffhollan
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 07/04/2018
ms.author: azfuncdf
ms.openlocfilehash: dfcf52c6d8d5c04e15d653376f52107fc4776d02
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38973150"
---
# <a name="overview-of-function-types-and-features-for-durable-functions-azure-functions"></a>Durable Functions (Azure Functions) 之函式類型與功能概觀

Azure Durable Functions 提供函式執行之具狀態的協調流程。  長期函式是由不同的 Azure Functions 所組成的解決方案。  每個函式均在協調流程中扮演不同的角色。  以下文件會概略說明長期函式協調流程中涉及的函式類型，  也會提及一些將函式連接在一起的常見模式。  

![長期函式類型][1]  

## <a name="types-of-functions"></a>函式類型

### <a name="activity-functions"></a>活動函式

活動函式是長期協調流程中的基工作本單位。  活動函式是在流程中受到協調處理的函式和任務。  例如，您可以建立一個長期函式來處理訂單，用來檢查庫存、向客戶收費、建立出貨等。  這類任務每項都是活動函式。  活動函式對於可在其中進行的工作類型沒有任何限制。  其可使用 Azure Functions 支援的任何語言撰寫。  長期任務架構可確保每個呼叫的活動函式在協調流程期間至少執行一次。

活動函式必須由[活動觸發程序](durable-functions-bindings.md#activity-triggers)觸發。  此函式會以參數形式接收到 [DurableActivityContext](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableActivityContext.html)。 您也可以將觸發程式繫結至任何其他物件，以將輸入的資訊傳遞至函式。  活動函式也可以將值傳回協調器。  若活動函式會傳出和傳回許多值，則可[利用元組或陣列](durable-functions-bindings.md#passing-multiple-parameters)。  活動函式只能由協調流程執行個體加以觸發。  雖然活動函式與另一個函式 (例如 HTTP 觸發函式) 之間，可能會共用一些程式碼，每個函式只能有一個觸發程序。

如需更多資訊和範例，請參閱[長期函式中的繫結 (Azure Functions)](durable-functions-bindings.md#activity-triggers)。

### <a name="orchestrator-functions"></a>協調器函式

協調器函式是長期函式的核心。  協調器函式描述了動作的執行方式與順序。  協調器函式描述了程式碼 (C# 或 JavaScript) 中的協調流程，如 [Durable Functions 概觀](durable-functions-overview.md) 所述。  協調流程可以有許多不同類型的動作，例如[活動函式](#activity-functions)、[子協調流程](#sub-orchestrations)、[等待外部事件](#external-events)和[計時器](#durable-timers)。  

協調器函式必須由[協調流程觸發程序](durable-functions-bindings.md#orchestration-triggers)觸發。

協調器由[協調器用戶端](#client-functions)啟動，該客戶端本身可以從任何來源 (HTTP、佇列、事件串流) 觸發。  協調流程的每個執行個體具有執行個體識別碼，可以自動產生 (建議選項)，也可以由使用者產生。  該識別碼可用於[管理協調流程的執行個體](durable-functions-instance-management.md)。

如需更多資訊和範例，請參閱[長期函式中的繫結 (Azure Functions)](durable-functions-bindings.md#orchestration-triggers)。

### <a name="client-functions"></a>用戶端函式

用戶端函式是觸發函式，會建立協調流程的新執行個體，  是建立長期協調流程執行個體的進入點。  用戶端函式可以由任何觸發程序 (HTTP、佇列、事件串流等) 觸發，並以應用程式支援的任何語言撰寫。  除了觸發程序以外，用戶端函式具有[協調流程用戶端](durable-functions-bindings.md#orchestration-client)繫結，使其可建立及管理長期協調流程。  用戶端函式的最基本範例是啟動協調器函式並傳回檢查狀態回應的 HTTP 觸發函式，如[以下範例所示](durable-functions-http-api.md#http-api-url-discovery)。

如需更多資訊和範例，請參閱[長期函式中的繫結 (Azure Functions)](durable-functions-bindings.md#orchestration-client)。

## <a name="features-and-patterns"></a>功能與模式

### <a name="sub-orchestrations"></a>子協調流程

除了呼叫活動函式，協調器函式還可以呼叫其他協調器函式。 例如，您可以從協調器函式庫中編譯更大的協調流程。 或者，也可以平行執行協調器函式的多個執行個體。

如需更多資訊和範例，請參閱[子協調流程相關文章](durable-functions-sub-orchestrations.md)。

### <a name="durable-timers"></a>永久性計時器

[Durable Functions](durable-functions-overview.md) 提供「永久性計時器」，用於協調器函式中實作延遲，或在非同步動作上設定逾時。 永久性計時器應該用於協調器函式中，以代替 `Thread.Sleep` 和 `Task.Delay` (C#)，或 `setTimeout()` 和 `setInterval()` (JavaScript)。

如需更多永久性計時器的資訊和範例，請參閱[計時器相關文章](durable-functions-timers.md)

### <a name="external-events"></a>外部事件

協調器函式可以等待外部事件以更新協調流程執行個體。 Durable Functions 的這項功能通常有助於處理人為互動或其他外部回呼。

如需更多資訊和範例，請參閱[外部事件相關文章](durable-functions-external-events.md)。

### <a name="error-handling"></a>錯誤處理

Durable Function 協調流程會在程式碼中實作，且可以使用程式設計語言的錯誤處理功能。  這表示 "try/catch" 之類的模式可在您的協調流程中運作。  長期函式也隨附一些內建的重試原則。  此動作可以在異常時自動延遲和重試活動。  重試可讓您處理暫時性例外狀況，而無須放棄協調流程。

如需更多資訊和範例，請參閱[錯誤處理相關文章](durable-functions-error-handling.md)。

### <a name="cross-function-app-communication"></a>跨函式應用程式通訊

雖然長期協調流程通常存在於單一函式應用程式的情況下，但仍有模式可以協調跨多個函式應用程式的協調流程。  儘管可以透過 HTTP 進行跨應用程式通訊，但若在每個活動上使用長期框架，就能維持跨兩個應用程式的長期流程。

以下提供以 C# 撰寫之跨函式應用程式協調流程的範例。  一個活將會啟動外部協調流程。 另一個活動則會取出狀態並傳回。  協調器會等待狀態完成後再繼續。

```csharp
[FunctionName("OrchestratorA")]
public static async Task RunRemoteOrchestrator(
    [OrchestrationTrigger] DurableOrchestrationContext context)
{
    // Do some work...

    // Call a remote orchestration
    string statusUrl = await context.CallActivityAsync<string>(
        "StartRemoteOrchestration", "OrchestratorB");

    // Wait for the remote orchestration to complete
    while (true)
    {
        bool isComplete = await context.CallActivityAsync<bool>("CheckIsComplete", statusUrl);
        if (isComplete)
        {
            break;
        }

        await context.CreateTimer(context.CurrentUtcDateTime.AddMinutes(1), CancellationToken.None);
    }

    // B is done. Now go do more work...
}

[FunctionName("StartRemoteOrchestration")]
public static async Task<string> StartRemoteOrchestration([ActivityTrigger] string orchestratorName)
{
    using (var response = await HttpClient.PostAsync(
        $"https://appB.azurewebsites.net/orchestrations/{orchestratorName}",
        new StringContent("")))
    {
        string statusUrl = await response.Content.ReadAsAsync<string>();
        return statusUrl;
    }
}

[FunctionName("CheckIsComplete")]
public static async Task<bool> CheckIsComplete([ActivityTrigger] string statusUrl)
{
    using (var response = await HttpClient.GetAsync(statusUrl))
    {
        // 200 = Complete, 202 = Running
        return response.StatusCode == HttpStatusCode.OK;
    }
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [繼續閱讀 Durable Functions 文件](durable-functions-bindings.md)

> [!div class="nextstepaction"]
> [安裝 Durable Functions 擴充功能和範例](durable-functions-install.md)

<!-- Media references -->
[1]: media/durable-functions-types-features-overview/durable-concepts.png