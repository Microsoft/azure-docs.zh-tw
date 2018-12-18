---
title: 在 Durable Functions 中處理錯誤 - Azure
description: 了解如何在 Azure Functions 的 Durable Functions 擴充中處理錯誤。
services: functions
author: cgillum
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: azfuncdf
ms.openlocfilehash: 6bf9eb2cd2ebdf5f6d53e00923146bab49a142bf
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44377900"
---
# <a name="handling-errors-in-durable-functions-azure-functions"></a>在 Durable Functions (Azure Functions) 中處理錯誤

Durable Function 協調流程在程式碼中實作，而且可以使用程式設計語言的錯誤處理功能。 因此，將錯誤處理和補償納入協調流程時，確實已沒有任何新的概念需要了解。 不過，有一些行為值得注意。

## <a name="errors-in-activity-functions"></a>活動函式中的錯誤

活動函式中擲回的任何例外狀況會封送處理回到協調器函式，並以 `FunctionFailedException` 擲回。 您可以在協調器函式中撰寫符合需求的錯誤處理和補償程式碼。

例如，假設有下列協調器函式會從一個帳戶轉帳到另一個帳戶：

```csharp
#r "Microsoft.Azure.WebJobs.Extensions.DurableTask"

public static async Task Run(DurableOrchestrationContext context)
{
    var transferDetails = ctx.GetInput<TransferOperation>();

    await context.CallActivityAsync("DebitAccount",
        new
        { 
            Account = transferDetails.SourceAccount,
            Amount = transferDetails.Amount
        });

    try
    {
        await context.CallActivityAsync("CreditAccount",         
            new
            { 
                Account = transferDetails.DestinationAccount,
                Amount = transferDetails.Amount
            });
    }
    catch (Exception)
    {
        // Refund the source account.
        // Another try/catch could be used here based on the needs of the application.
        await context.CallActivityAsync("CreditAccount",         
            new
            { 
                Account = transferDetails.SourceAccount,
                Amount = transferDetails.Amount
            });
    }
}
```

如果對目的地帳戶呼叫 **CreditAccount** 函式失敗，協調器函式會將款項匯回到來源帳戶，以彌補此情況。

## <a name="automatic-retry-on-failure"></a>失敗時自動重試

當您呼叫活動函式或子協調流程函式時，您可以指定自動重試原則。 下列範例會嘗試呼叫函式最多 3 次，並於每次重試之間等待 5 秒鐘：

```csharp
public static async Task Run(DurableOrchestrationContext context)
{
    var retryOptions = new RetryOptions(
        firstRetryInterval: TimeSpan.FromSeconds(5),
        maxNumberOfAttempts: 3);

    await ctx.CallActivityWithRetryAsync("FlakyFunction", retryOptions, null);
    
    // ...
}
```

`CallActivityWithRetryAsync` API 接受 `RetryOptions` 參數。 使用 `CallSubOrchestratorWithRetryAsync` API 來呼叫子協調流程時可以使用這些相同的重試原則。

自訂自動重試原則有幾個選項。 其中包括：

* **嘗試次數上限**：重試嘗試次數上限。
* **第一次重試間隔**：第一次重試嘗試之前等候的時間量。
* **輪詢係數**：用來決定輪詢增加速率的係數。 預設值為 1。
* **最大重試間隔**：重試嘗試之間等候的最大時間量。
* **重試逾時**：花費在重試的最大時間量。 預設行為是無限期地重試。
* **控制代碼**：您可以指定使用者定義的回呼，以決定是否應該重試函式呼叫。

## <a name="function-timeouts"></a>函式逾時

如果協調器函式內的函式呼叫太久才會完成，您可以放棄此函式呼叫。 目前，適當的作法是使用 `context.CreateTimer` 搭配 `Task.WhenAny`來建立[永久性計時器](durable-functions-timers.md) ，如下列範例所示：

```csharp
public static async Task<bool> Run(DurableOrchestrationContext context)
{
    TimeSpan timeout = TimeSpan.FromSeconds(30);
    DateTime deadline = context.CurrentUtcDateTime.Add(timeout);

    using (var cts = new CancellationTokenSource())
    {
        Task activityTask = context.CallActivityAsync("FlakyFunction");
        Task timeoutTask = context.CreateTimer(deadline, cts.Token);

        Task winner = await Task.WhenAny(activityTask, timeoutTask);
        if (winner == activityTask)
        {
            // success case
            cts.Cancel();
            return true;
        }
        else
        {
            // timeout case
            return false;
        }
    }
}
```

> [!NOTE]
> 這項機制並不會實際終止進行中的活動函式執行。 只是讓協調器函式略過結果並繼續執行。 如需詳細資訊，請參閱[計時器](durable-functions-timers.md#usage-for-timeout)文件。

## <a name="unhandled-exceptions"></a>未處理的例外狀況

如果協調器函式失敗並傳回未處理的例外狀況，例外狀況的詳細資料會記錄下來，而在執行個體會以 `Failed` 狀態結束。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何診斷問題](durable-functions-diagnostics.md)
