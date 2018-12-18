---
title: Durable Functions 中的版本控制 - Azure
description: 了解如何在 Azure Functions 的 Durable Functions  擴充中實作版本控制。
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
ms.openlocfilehash: 0a86e4a87f5ec23c284aa4e5cfb2c67622b3ebe9
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22991283"
---
# <a name="versioning-in-durable-functions-azure-functions"></a>Durable Functions (Azure Functions) 中的版本控制

在應用程式的存留期之中，必然會新增、移除和變更函式。 [Durable Functions](durable-functions-overview.md) 可以採取以往不可能的方式將函式鏈結在一起，而此鏈結會影響您如何處理版本控制。

## <a name="how-to-handle-breaking-changes"></a>如何處理重大變更

有幾個重大變更的例子需要注意。 本文討論最常見的重大變更。 所有重大變更背後的主題都是：變更函式程式碼會影響新的和現有的函式協調流程。

### <a name="changing-activity-function-signatures"></a>變更活動函式簽章

簽章變更是指函式的名稱、輸入或輸出有所變更。 如果活動函式發生這種變更，則會中斷依賴此函式的協調器函式。 如果您更新協調器函式來配合此變更，可能會中斷現有的執行中執行個體。

舉例來說，假設我們有下列函式。

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] DurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

這個簡化的函式會取得 **Foo** 的結果，然後傳給 **Bar**。 假設我們需要將 **Foo** 的傳回值從 `bool` 變更為 `int`，以支援更多種不同的結果值。 結果如下所示：

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] DurableOrchestrationContext context)
{
    int result = await context.CallActivityAsync<int>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

在協調器函式的所有新執行個體中，此變更不會有問題，但會中斷任何執行中的執行個體。 例如，假設協調流程執行個體會呼叫 **Foo**、取回布林值，然後執行檢查點。 如果此時部署簽章變更，則通過檢查點的執行個體在繼續並重新呼叫 `context.CallActivityAsync<int>("Foo")` 時會立刻失敗。 這是因為記錄資料表中的結果是 `bool`，但新的程式碼卻嘗試將它還原序列化為 `int`。

這只是簽章變更可能中斷現有執行個體的各種情況之一。 一般而言，如果協調器需要變更呼叫函式的方式，則變更就很可能會有問題。

### <a name="changing-orchestrator-logic"></a>變更協調器邏輯

另一種版本設定問題起因於變更協調器函式程式碼，導致執行中之執行個體的重新執行邏輯發生混淆。

假設有下列協調器函式：

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] DurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

現在，假設您想要新增另一個函式呼叫，此變更看似無害。

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] DurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    if (result)
    {
        await context.CallActivityAsync("SendNotification");
    }

    await context.CallActivityAsync("Bar", result);
}
```

此變更會在 **Foo** 和 **Bar** 之間新增呼叫 **SendNotification** 函式。 簽章不變。 當現有的執行個體在呼叫 **Bar** 之後繼續時，就發生問題。 在重新執行期間，如果原始呼叫 **Foo** 傳回 `true`，則協調器重新執行會呼叫 **SendNotification**，但這不在其執行記錄中。 結果，「永久性工作架構」會失敗，並傳回 `NonDeterministicOrchestrationException`，因為原本認為應該呼叫 **Bar**，但卻發現呼叫 **SendNotification**。

## <a name="mitigation-strategies"></a>風險降低策略

以下是解決版本控制問題的一些策略：

* 不執行任何動作
* 停止所有執行中的執行個體
* 並存部署

### <a name="do-nothing"></a>不執行任何動作

處理重大變更最簡單的方式，就是讓執行中的協調流程執行個體失敗。 新的執行個體會成功執行已變更的程式碼。

這是否為問題取決於執行中之執行個體的重要性。 如果您正在進行開發，並不在意執行中的執行個體，這樣可能就很好。 不過，您必須處理診斷管道中的例外狀況及錯誤。 如果想要避免這些事情，請考慮其他版本設定選項。

### <a name="stop-all-in-flight-instances"></a>停止所有執行中的執行個體

另一個選項是停止所有執行中的執行個體。 作法是清除內部 **control-queue** 和 **workitem-queue**佇列中的內容。 執行個體會永遠卡住不動，但不會以錯誤訊息干擾遙測資料。 這在快速開發原型時很理想。

> [!WARNING]
> 這些佇列的詳細資料可能隨著時間而變更，請勿依賴此技術來處理生產工作負載。

### <a name="side-by-side-deployments"></a>並存部署

為了確保安全地部署重大變更，將重大變更與舊版並存部署就能萬無一失。 您可以採取下列任何技術：

* 將所有更新部署為全新的函式 (新名稱)。
* 以不同的儲存體帳戶，將所有更新部署為新的函式應用程式。
* 以更新的 `TaskHub` 名稱來部署函式應用程式的新複本。 這是建議使用的技巧。

### <a name="how-to-change-task-hub-name"></a>如何變更工作中樞名稱

您可以在 *host.json* 檔案中設定工作中樞，如下所示：

```json
{
    "durableTask": {
        "HubName": "MyTaskHubV2"
    }
}
```

預設值為 `DurableFunctionsHub`。

所有 Azure 儲存體實體都是依據 `HubName` 設定值來命名。 只要指定新名稱給工作中樞，就可以確保為應用程式的新版本建立個別的佇列和記錄資料表。

建議您將函式應用程式的新版本部署到新的[部署位置](https://blogs.msdn.microsoft.com/appserviceteam/2017/06/13/deployment-slots-preview-for-azure-functions/)。 部署位置可讓您並存執行函式應用程式的多個複本，而且其中只有一個是作用中的「生產」位置。 當您準備將新的協調流程邏輯公開到現有的基礎結構時，只要將新的版本調換到生產位置即可，就是這麼簡單。

> [!NOTE]
> 當您對協調器函式使用 HTTP 和 Webhook 觸發程序時，此策略最理想。 對於非 HTTP 觸發程序，例如佇列或事件中樞，應該從應用程式設定衍生觸發程序定義，此設定會隨著交換作業而更新。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何處理效能和級別問題](durable-functions-perf-and-scale.md)
