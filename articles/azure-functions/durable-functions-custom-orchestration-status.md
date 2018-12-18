---
title: Durable Functions 中的自訂協調流程狀態 - Azure
description: 了解如何設定及使用 Durable Functions 的自訂協調流程狀態。
services: functions
author: kadimitr
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 04/24/2018
ms.author: azfuncdf
ms.openlocfilehash: 840b96b9cfdb28ca1b17f54698677f4d491342c8
ms.sourcegitcommit: 6e43006c88d5e1b9461e65a73b8888340077e8a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/01/2018
ms.locfileid: "32310310"
---
# <a name="custom-orchestration-status-in-durable-functions-azure-functions"></a>Durable Functions 中的自訂協調流程狀態 (Azure Functions)

自訂協調流程狀態可讓您為協調器函式設定自訂狀態值。 該狀態可透過 HTTP GetStatus API 或 `DurableOrchestrationClient.GetStatusAsync` API 提供。

## <a name="sample-use-cases"></a>範例使用案例 

### <a name="visualize-progress"></a>進度視覺化

用戶端可以輪詢狀態結束點及顯示進度 UI，以視覺化方式呈現目前的執行階段。 以下範例示範進度共用：

```csharp
[FunctionName("E1_HelloSequence")]
public static async Task<List<string>> Run(
  [OrchestrationTrigger] DurableOrchestrationContextBase context)
{
  var outputs = new List<string>();

  outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Tokyo"));
  context.SetCustomStatus("Tokyo");
  outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Seattle"));
  context.SetCustomStatus("Seattle");
  outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "London"));
  context.SetCustomStatus("London");

  // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
  return outputs;
}

[FunctionName("E1_SayHello")]
public static string SayHello([ActivityTrigger] string name)
{
  return $"Hello {name}!";
}
```

接著，唯有當 `CustomStatus` 欄位設定為 "London" 時，用戶端才會接收到協調流程的輸出：

```csharp
[FunctionName("HttpStart")]
public static async Task<HttpResponseMessage> Run(
  [HttpTrigger(AuthorizationLevel.Function, methods: "post", Route = "orchestrators/{functionName}")] HttpRequestMessage req,
  [OrchestrationClient] DurableOrchestrationClientBase starter,
  string functionName,
  ILogger log)
{
    // Function input comes from the request content.
    dynamic eventData = await req.Content.ReadAsAsync<object>();
    string instanceId = await starter.StartNewAsync(functionName, eventData);

    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

    DurableOrchestrationStatus durableOrchestrationStatus = await starter.GetStatusAsync(instanceId);
    while (durableOrchestrationStatus.CustomStatus.ToString() != "London")
    {
      await Task.Delay(200);
      durableOrchestrationStatus = await starter.GetStatusAsync(instanceId);
    }

    HttpResponseMessage httpResponseMessage = new HttpResponseMessage(HttpStatusCode.OK)
    {
      Content = new StringContent(JsonConvert.SerializeObject(durableOrchestrationStatus))
    };

    return httpResponseMessage;
  }
}
```

### <a name="output-customization"></a>自訂輸出 

另一個有趣的情節是依照獨特的特性或互動傳回自訂輸出，藉此區分使用者。 在自訂協調流程狀態的協助之下，用戶端將能保有泛型的程式碼。 主要修改的內容全都發生在伺服器端，如以下範例所示：

```csharp
[FunctionName("CityRecommender")]
public static void Run(
  [OrchestrationTrigger] DurableOrchestrationContextBase context)
{
  int userChoice = context.GetInput<int>();

  switch (userChoice)
  {
    case 1:
    context.SetCustomStatus(new
    {
      recommendedCities = new[] {"Tokyo", "Seattle"},
      recommendedSeasons = new[] {"Spring", "Summer"}
     });
      break;
    case 2:
      context.SetCustomStatus(new
      {
        recommendedCity = new[] {"Seattle, London"},
        recommendedSeasons = new[] {"Summer"}
      });
        break;
      case 3:
      context.SetCustomStatus(new
      {
        recommendedCity = new[] {"Tokyo, London"},
        recommendedSeasons = new[] {"Spring", "Summer"}
      });
        break;
  }

  // Wait for user selection and refine the recommendation
} 
```

### <a name="instruction-specification"></a>指示規格 

協調器能透過自訂狀態將唯一指示提供給用戶端。 自訂狀態指示將與協調流程程式碼中的步驟相對應：

```csharp
[FunctionName("ReserveTicket")]
public static async Task<bool> Run(
  [OrchestrationTrigger] DurableOrchestrationContextBase context)
{
  string userId = context.GetInput<string>();

  int discount = await context.CallActivityAsync<int>("CalculateDiscount", userId);

  context.SetCustomStatus(new
  {
    discount = discount,
    discountTimeout = 60,
    bookingUrl = "https://www.myawesomebookingweb.com",
  });

  bool isBookingConfirmed = await context.WaitForExternalEvent<bool>("BookingConfirmed");

  context.SetCustomStatus(isBookingConfirmed
    ? new {message = "Thank you for confirming your booking."}
    : new {message = "The booking was not confirmed on time. Please try again."});

  return isBookingConfirmed;
}
```

## <a name="sample"></a>範例

以下範例先設定自訂狀態：

```csharp
public static async Task SetStatusTest([OrchestrationTrigger] DurableOrchestrationContext ctx)
{
    // ...do work...

    // update the status of the orchestration with some arbitrary data
    var customStatus = new { nextActions = new [] {"A", "B", "C"}, foo = 2, };
    ctx.SetCustomStatus(customStatus);

    // ...do more work...
}
```

當協調流程執行時，外部用戶端能擷取該自訂狀態：

```http
GET /admin/extensions/DurableTaskExtension/instances/instance123

```

用戶端將取得下列回應： 

```http
{
  "runtimeStatus": "Running",
  "input": null,
  "customStatus": { "nextActions": ["A", "B", "C"], "foo": 2 },
  "output": null,
  "createdTime": "2017-10-06T18:30:24Z",
  "lastUpdatedTime": "2017-10-06T19:40:30Z"
}
```

> [!WARNING]
>  自訂狀態承載僅限為 16 KB 的 UTF-16 JSON 文字，因為它必須符合 Azure 資料表儲存體資料行的大小。 開發人員如果需要較大的承載，可以使用外部儲存體。


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解 Durable Functions 中的 HTTP API](durable-functions-http-api.md)


