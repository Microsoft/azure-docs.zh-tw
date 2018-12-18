---
title: Durable Functions 的子協調流程 - Azure
description: 如何在 Azure Functions 的 Durable Functions 擴充中，從協調流程呼叫協調流程。
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
ms.openlocfilehash: 7545a371749ed9af88f08af23cce3a513f494374
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33761328"
---
# <a name="sub-orchestrations-in-durable-functions-azure-functions"></a>Durable Functions (Azure Functions) 中的子協調流程

除了呼叫活動函式，協調器函式還可以呼叫其他協調器函式。 例如，您可以從協調器函式庫中編譯更大的協調流程。 或者，也可以平行執行協調器函式的多個執行個體。

協調器函式可以呼叫 [CallSubOrchestratorAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CallSubOrchestratorAsync_) 或 [CallSubOrchestratorWithRetryAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CallSubOrchestratorWithRetryAsync_) 方法，以呼叫另一個協調器函式。 [錯誤處理和補償](durable-functions-error-handling.md#automatic-retry-on-failure)一文提供自動重試的詳細資訊。

從呼叫端的觀點來看，子協調器函式的行為就像活動函式一樣。 子協調器函式可以傳回值、擲回例外狀況，還可以由父代協調器函式來等候。

> [!NOTE]
> `CallSubOrchestratorAsync` 和 `CallSubOrchestratorWithRetryAsync` 方法尚無法使用於 JavaScript。

## <a name="example"></a>範例

下列範例說明 IoT (物聯網) 情節，其中有多個需要佈建的裝置。 有一個特別的協調流程是每個裝置都需要進行，看起來可能如下所示：

```csharp
public static async Task DeviceProvisioningOrchestration(
    [OrchestrationTrigger] DurableOrchestrationContext ctx)
{
    string deviceId = ctx.GetInput<string>();

    // Step 1: Create an installation package in blob storage and return a SAS URL.
    Uri sasUrl = await ctx.CallActivityAsync<Uri>("CreateInstallationPackage", deviceId);

    // Step 2: Notify the device that the installation package is ready.
    await ctx.CallActivityAsync("SendPackageUrlToDevice", Tuple.Create(deviceId, sasUrl));

    // Step 3: Wait for the device to acknowledge that it has downloaded the new package.
    await ctx.WaitForExternalEvent<bool>("DownloadCompletedAck");

    // Step 4: ...
}
```

這個協調器函式可直接用於一次性裝置佈建，也可以當作更大協調流程的一部分。 在後者的情況下，父代協調器函式可以使用 `CallSubOrchestratorAsync` API 來排程 `DeviceProvisioningOrchestration` 的執行個體。

以下示範如何平行執行多個協調器函式。

```csharp
[FunctionName("ProvisionNewDevices")]
public static async Task ProvisionNewDevices(
    [OrchestrationTrigger] DurableOrchestrationContext ctx)
{
    string[] deviceIds = await ctx.CallActivityAsync<string[]>("GetNewDeviceIds");

    // Run multiple device provisioning flows in parallel
    var provisioningTasks = new List<Task>();
    foreach (string deviceId in deviceIds)
    {
        Task provisionTask = ctx.CallSubOrchestratorAsync("DeviceProvisioningOrchestration", deviceId);
        provisioningTasks.Add(provisionTask);
    }

    await Task.WhenAll(provisioningTasks);

    // ...
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解工作中樞是什麼及如何設定](durable-functions-task-hubs.md)
