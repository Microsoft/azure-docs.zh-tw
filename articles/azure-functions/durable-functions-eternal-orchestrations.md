---
title: Durable Functions 中的永久性協調流程 - Azure
description: 了解如何使用 Azure Functions 的 Durable Functions 擴充來實作永久性協調流程。
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
ms.openlocfilehash: 0af61ec3b22692402697df5331df80ca044759b5
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37340656"
---
# <a name="eternal-orchestrations-in-durable-functions-azure-functions"></a>Durable Functions (Azure Functions) 中的永久性協調流程

「永久協調流程」是指永不結束的協調流程函式。 想要在彙總工具和任何需要無限迴圈的情節下使用 [Durable Functions](durable-functions-overview.md) 時，永久性協調流程就很有用。

## <a name="orchestration-history"></a>協調流程記錄

如[檢查點檢查和重新執行](durable-functions-checkpointing-and-replay.md)中所述，「永久性工作架構」會持續追蹤每個函式協調流程的記錄。 只要協調器函式繼續排程新的工作，此記錄就會持續成長。 如果協調器函式進入無限迴圈，並持續排程工作，此記錄可能會變得非常大，而造成明顯的效能問題。 「永久協調流程」概念是設計讓需要無限迴圈的應用程式減少發生這類問題。

## <a name="resetting-and-restarting"></a>重設和重新啟動

協調器函式會呼叫 [ContinueAsNew](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_ContinueAsNew_) 方法來重設其狀態，而不使用無限迴圈。 此方法接受單一 JSON 序列化參數，此參數會變成新的輸入來產生下一個協調器函式。

呼叫 `ContinueAsNew` 時，執行個體在結束之前會將訊息加入其本身的佇列。 此訊息會以新的輸入值來重新啟動執行個體。 相同的執行個體識別碼會保留下來，但協調器函式的記錄實際上會截斷。

> [!NOTE]
> 「永久性工作架構」會維護相同的執行個體識別碼，但在內部會為 `ContinueAsNew` 所重設的協調器函式建立新的「執行識別碼」。 此執行識別碼通常不對外公開，但可能很助於了解偵錯協調流程執行。

> [!NOTE]
> `ContinueAsNew` 方法尚無法使用於 JavaScript。

## <a name="periodic-work-example"></a>定期工作範例

需要無限期地執行定期工作的程式碼，即為永久協調流程的一個使用案例。

```csharp
[FunctionName("Periodic_Cleanup_Loop")]
public static async Task Run(
    [OrchestrationTrigger] DurableOrchestrationContext context)
{
    await context.CallActivityAsync("DoCleanup");

    // sleep for one hour between cleanups
    DateTime nextCleanup = context.CurrentUtcDateTime.AddHours(1);
    await context.CreateTimer(nextCleanup, CancellationToken.None);

    context.ContinueAsNew(null);
}
```

此範例與計時器觸發函式之間的差異在於，此處的清理觸發程序時間不是根據排程。 例如，每小時執行函式的 CRON 排程會在 1:00、2:00、3:00 等時間執行它，很可能會陷入重疊問題。 不過，在此範例中，如果清除需要 30 分鐘，則會排程在 1:00、2:30、4:00 等時間，不會有機會重疊。

## <a name="exit-from-an-eternal-orchestration"></a>從永久協調流程離開

如果協調器函式最終一定要完成，則您完全「不」需要呼叫 `ContinueAsNew`，讓函式自然結束即可。

如果協調器函式在無限迴圈中，且必須停止，請使用 [TerminateAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_TerminateAsync_) 方法來停止它。 如需詳細資訊，請參閱[執行個體管理](durable-functions-instance-management.md)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何實作單次協調流程](durable-functions-singletons.md)
