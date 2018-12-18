---
title: 長期函式中的繫結 - Azure
description: 如何使用 Azure Functions 之長期函式延伸模組的觸發程序和繫結。
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
ms.openlocfilehash: 370e6e2c569aaf6d9289bddccde2174b4dd2ee97
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33763351"
---
# <a name="bindings-for-durable-functions-azure-functions"></a>長期函式中的繫結 (Azure Functions)

[長期函式](durable-functions-overview.md)延伸模組引進了兩個新的觸發程序繫結，可控制協調器和活動函式的執行。 它也引進了輸出繫結，可作為長期函式執行階段的用戶端。

## <a name="orchestration-triggers"></a>協調流程觸發程序

協調流程觸發程序可讓您撰寫長期協調器函式。 這個觸發程序支援啟動新的協調器函式執行個體，以及繼續使用「等候」工作的現有協調器函式執行個體。

當您使用 Azure Functions 適用的 Visual Studio 工具時，協調流程觸發程序會使用 [OrchestrationTriggerAttribute](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.OrchestrationTriggerAttribute.html) .NET 屬性進行設定。

當您以指令碼語言撰寫協調器函式時 (例如，在 Azure 入口網站中)，會藉由遵循 function.json 檔案之 `bindings` 陣列中的 JSON 物件，定義協調流程觸發程序：

```json
{
    "name": "<Name of input parameter in function signature>",
    "orchestration": "<Optional - name of the orchestration>",
    "type": "orchestrationTrigger",
    "direction": "in"
}
```

* `orchestration` 是協調流程的名稱。 這是用戶端在想要啟動這個協調器函式的新執行個體時必須使用的值。 這是選用屬性。 如果未指定，會使用函式的名稱。

此觸發程序繫結會在內部輪詢函式應用程式之預設儲存體帳戶中的一系列佇列。 這些佇列是延伸模組的內部實作詳細資料，這就是為什麼它們未在繫結屬性中明確設定。

### <a name="trigger-behavior"></a>觸發程序行為

以下是協調流程觸發程序的一些附註：

* **單一執行緒** - 單一發送器執行緒用於單一主機執行個體上的所有協調器函式執行。 基於這個理由，請務必確保協調器函式程式碼有效率，且不會執行任何 I/O。 也務必要確保此執行緒不會執行除了等候長期函式特定工作類型以外的任何非同步工作。
* **有害訊息處理** - 協調流程觸發程序中並無有害訊息支援。
* **訊息可見度** - 協調流程觸發程序訊息會被清除佇列，並且在可設定持續期間保持不可見。 只要函式應用程式正在執行且狀況良好，就會自動更新這些訊息的可見度。
* **傳回值** - 傳回值會序列化為 JSON，保存到 Azure 資料表儲存體中的協調流程歷程記錄資料表。 協調流程用戶端繫結可以查詢這些傳回值，會在稍後說明。

> [!WARNING]
> 協調器函式應該永不使用協調流程繫結觸發程序繫結以外的任何輸入或輸出繫結。 這麼做有可能會造成長期工作延伸模組的問題，因為這些繫結可能不會遵照單一執行緒和 I/O 規則。

### <a name="trigger-usage"></a>觸發程序使用方式

協調流程觸發程序繫結支援輸入及輸出。 協助了解輸入和輸出處理的一些事項如下：

* **輸入** - 協調流程函式僅支援 [DurableOrchestrationContext](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html) 作為參數類型。 不支援直接在函式簽章中還原序列化輸入。 程式碼必須使用 [GetInput\<T>](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_GetInput__1) 方法來擷取協調器函式輸入。 這些輸入必須是 JSON 可序列化類型。
* **輸出** - 協調流程觸發程序支援輸出值以及輸入。 函式的傳回值會用來指派輸出值，而且必須是 JSON 可序列化。 如果函式傳回 `Task` 或 `void`，則 `null` 值將儲存為輸出。

### <a name="trigger-sample"></a>觸發程序範例

以下是最簡單的 "Hello World" 協調器函式外觀的範例：

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static string Run([OrchestrationTrigger] DurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    return $"Hello {name}!";
}
```

#### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

```javascript
const df = require("durable-functions");

module.exports = df(function*(context) {
    const name = context.df.getInput();
    return `Hello ${name}!`;
});
```

> [!NOTE]
> JavaScript 協調器應使用 `return`。 `durable-functions` 程式庫會負責呼叫 `context.done` 方法。

大部分協調器函式會呼叫活動函式；因此，這裡的 "Hello World" 範例示範如何呼叫活動函式：

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static async Task<string> Run(
    [OrchestrationTrigger] DurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    string result = await context.CallActivityAsync<string>("SayHello", name);
    return result;
}
```

#### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

```javascript
const df = require("durable-functions");

module.exports = df(function*(context) {
    const name = context.df.getInput();
    const result = yield context.df.callActivityAsync("SayHello", name);
    return result;
});
```

## <a name="activity-triggers"></a>活動觸發程序

活動觸發程序可讓您撰寫由協調器函式呼叫的函式。

如果您使用 Visual Studio，活動觸發程序會使用 [ActvityTriggerAttribute](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.ActivityTriggerAttribute.html) .NET 屬性進行設定。 

如果您使用 Azure 入口網站進行開發，活動觸發程序是由 function.json 之 `bindings` 陣列中的下列 JSON 物件定義：

```json
{
    "name": "<Name of input parameter in function signature>",
    "activity": "<Optional - name of the activity>",
    "type": "activityTrigger",
    "direction": "in"
}
```

* `activity` 是活動的名稱。 這是協調器函式用來叫用此活動函式的值。 這是選用屬性。 如果未指定，會使用函式的名稱。

此觸發程序繫結會在內部輪詢函式應用程式之預設儲存體帳戶中的佇列。 這個佇列是延伸模組的內部實作詳細資料，這就是為什麼它未在繫結屬性中明確設定。

### <a name="trigger-behavior"></a>觸發程序行為

以下是活動觸發程序的一些附註：

* **執行緒** - 與協調流程觸發程序不同，活動觸發程序在執行緒或 I/O 方面沒有任何限制。 它們可以被視為一般函式。
* **有害訊息處理** - 活動觸發程序中沒有有害訊息支援。
* **訊息可見度** - 活動觸發程序訊息會被清除佇列，並且在可設定持續期間保持不可見。 只要函式應用程式正在執行且狀況良好，就會自動更新這些訊息的可見度。
* **傳回值** - 傳回值會序列化為 JSON，保存到 Azure 資料表儲存體中的協調流程歷程記錄資料表。

> [!WARNING]
> 活動函式的儲存體後端是實作詳細資料，使用者程式碼不應直接與這些儲存體實體互動。

### <a name="trigger-usage"></a>觸發程序使用方式

活動觸發程序繫結支援輸入及輸出，就像協調流程觸發程序一樣。 協助了解輸入和輸出處理的一些事項如下：

* **輸入** - 活動函式原生使用 [DurableActivityContext](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableActivityContext.html) 作為參數類型。 或者，可以使用 JSON 可序列化的任何參數類型宣告活動函式。 當您使用 `DurableActivityContext` 時，可以呼叫 [GetInput\<T>](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableActivityContext.html#Microsoft_Azure_WebJobs_DurableActivityContext_GetInput__1) 以擷取和還原序列化活動函式輸入。
* **輸出** - 活動函式支援輸出值和輸入。 函式的傳回值會用來指派輸出值，而且必須是 JSON 可序列化。 如果函式傳回 `Task` 或 `void`，則 `null` 值將儲存為輸出。
* **中繼資料** - 活動函式可以繫結至 `string instanceId` 參數，以取得父代協調流程的執行個體識別碼。

### <a name="trigger-sample"></a>觸發程序範例

以下是簡單的 "Hello World" 活動函式外觀的範例：

#### <a name="c"></a>C#

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] DurableActivityContext helloContext)
{
    string name = helloContext.GetInput<string>();
    return $"Hello {name}!";
}
```

#### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

```javascript
module.exports = function(context) {
    context.done(null, `Hello ${context.bindings.name}!`);
};
```

`ActivityTriggerAttribute` 繫結的預設參數類型是 `DurableActivityContext`。 不過，活動觸發程序也支援直接繫結至JSON 可序列化類型 (包括基本類型)，所以相同函式可以簡化如下：

#### <a name="c"></a>C#

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] string name)
{
    return $"Hello {name}!";
}
```

#### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

```javascript
module.exports = function(context, name) {
    context.done(null, `Hello ${name}!`);
};
```

### <a name="passing-multiple-parameters"></a>傳遞多個參數。 

您不可能直接將多個參數傳遞至活動函式。 在此情況下，建議傳入物件陣列，或使用 [ValueTuples](https://docs.microsoft.com/dotnet/csharp/tuples) 物件。

下列範例會使用附加 [C# 7](https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-7#tuples) 的 [ValueTuples](https://docs.microsoft.com/dotnet/csharp/tuples) 新功能：

```csharp
[FunctionName("GetCourseRecommendations")]
public static async Task<dynamic> RunOrchestrator(
    [OrchestrationTrigger] DurableOrchestrationContext context)
{
    string major = "ComputerScience";
    int universityYear = context.GetInput<int>();

    dynamic courseRecommendations = await context.CallActivityAsync<dynamic>("CourseRecommendations", (major, universityYear));
    return courseRecommendations;
}

[FunctionName("CourseRecommendations")]
public static async Task<dynamic> Mapper([ActivityTrigger] DurableActivityContext inputs)
{
    // parse input for student's major and year in university 
    (string Major, int UniversityYear) studentInfo = inputs.GetInput<(string, int)>();

    // retrieve and return course recommendations by major and university year
    return new {
        major = studentInfo.Major,
        universityYear = studentInfo.UniversityYear,
        recommendedCourses = new []
        {
            "Introduction to .NET Programming",
            "Introduction to Linux",
            "Becoming an Entrepreneur"
        }
    };
}
```

## <a name="orchestration-client"></a>協調流程用戶端

協調流程用戶端繫結可讓您撰寫函式，與協調器函式互動。 例如，您可以下列方式處理協調流程執行個體：
* 啟動它們。
* 查詢其狀態。
* 終止它們。
* 在它們執行時傳送事件給它們。

如果您使用 Visual Studio，您可以使用 [OrchestrationClientAttribute](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.OrchestrationClientAttribute.html) .NET 屬性，繫結至協調流程用戶端。

如果您使用指令碼語言 (例如 .csx 檔案) 進行開發，協調流程觸發程序是由 function.json 之 `bindings` 陣列中的下列 JSON 物件定義：

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "orchestrationClient",
    "direction": "out"
}
```

* `taskHub` - 在多個函式應用程式共用相同儲存體帳戶，但是必須相互隔離的案例中使用。 若未指定，就會使用 `host.json` 的預設值。 此值必須符合目標協調器函式所使用的值。
* `connectionName` - 包含儲存體帳戶連接字串的應用程式設定名稱。 這個連接字串代表的儲存體帳戶，必須是目標協調器函式使用的相同帳戶。 如果未指定，則會使用函數應用程式的預設儲存體帳戶連接字串。

> [!NOTE]
> 在大部分情況下，建議您省略這些屬性，並依賴預設行為。

### <a name="client-usage"></a>用戶端使用方式

在 C# 函式中，您通常會繫結至 `DurableOrchestrationClient`，這可讓您具有長期函式支援之所有用戶端 API 的完整存取權。 用戶端物件上的 API 包括：

* [StartNewAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_StartNewAsync_)
* [GetStatusAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_GetStatusAsync_)
* [TerminateAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_TerminateAsync_)
* [RaiseEventAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_RaiseEventAsync_)

或者，您可以繫結至 `IAsyncCollector<T>`，其中 `T` 是 [StartOrchestrationArgs](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.StartOrchestrationArgs.html) 或 `JObject`。

請參閱 [DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) API 文件，以取得這些作業的其他詳細資料。

### <a name="client-sample-visual-studio-development"></a>用戶端範例 (Visual Studio 開發)

以下是範例佇列觸發函式，它會啟動 "HelloWorld" 協調流程。

```csharp
[FunctionName("QueueStart")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [OrchestrationClient] DurableOrchestrationClient starter)
{
    // Orchestration input comes from the queue message content.
    return starter.StartNewAsync("HelloWorld", input);
}
```

### <a name="client-sample-not-visual-studio"></a>用戶端範例 (不是 Visual Studio)

如果您不是使用 Visual Studio 進行開發，您可以建立下列 function.json 檔案。 此範例示範如何設定佇列觸發函式，它使用長期協調流程用戶端繫結：

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "queueTrigger",
      "queueName": "durable-function-trigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "orchestrationClient",
      "direction": "out"
    }
  ],
  "disabled": false
} 
```

以下是啟動新的協調器函式執行個體的特定語言範例。

#### <a name="c-sample"></a>C# 範例

下列範例會示範如何使用長期協調流程用戶端繫結以從 C# 指令碼函式啟動新的函式執行個體：

```csharp
#r "Microsoft.Azure.WebJobs.Extensions.DurableTask"

public static Task<string> Run(string input, DurableOrchestrationClient starter)
{
    return starter.StartNewAsync("HelloWorld", input);
}
```

#### <a name="javascript-sample"></a>JavaScript 範例

下列範例會示範如何使用長期協調流程用戶端繫結以從 JavaScript 函式啟動新的函式執行個體：

```js
module.exports = function (context, input) {
    var id = generateSomeUniqueId();
    context.bindings.starter = [{
        FunctionName: "HelloWorld",
        Input: input,
        InstanceId: id
    }];

    context.done(null, id);
};
```

啟動執行個體的更多詳細資料可以在[執行個體管理](durable-functions-instance-management.md)中找到。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解檢查點和重新執行行為](durable-functions-checkpointing-and-replay.md)

