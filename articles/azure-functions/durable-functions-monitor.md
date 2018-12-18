---
title: Durable Functions 中的監視器 - Azure
description: 了解如何使用 Azure Functions 的 Durable Functions 擴充功能來實作狀態監視器。
services: functions
author: kashimiz
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 07/11/2018
ms.author: azfuncdf
ms.openlocfilehash: 02c068fc70748584583b2c71659b1a1abdc0a46d
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39035766"
---
# <a name="monitor-scenario-in-durable-functions---weather-watcher-sample"></a>Durable Functions 中的監視器案例 - 天氣監看員範例

監視器模式指的是工作流程中的彈性「週期性」程序，例如，輪詢直到符合特定條件。 本文會說明使用 [Durable Functions](durable-functions-overview.md) 來實作監視的範例。

## <a name="prerequisites"></a>必要條件

* [安裝 Durable Functions](durable-functions-install.md)。
* 完成 [Hello 序列](durable-functions-sequence.md)逐步解說。

## <a name="scenario-overview"></a>案例概觀

此範例會監視某地點的目前天氣狀況，並在天空放晴時用簡訊對使用者發出警示。 您可以使用一般的計時器觸發函式來檢查天氣並傳送警示。 不過，這個方法有一個問題，那就是**存留期管理**。 如果應傳送的警示只有一個，則監視器必須在偵測到天氣晴朗後自行停用。 監視模式除了可以結束自身的執行外，還有其他優點：

* 監視器會依間隔 (而非排程) 來執行：計時器觸發程序每小時「執行」一次；監視器會在動作之間「等候」一小時。 除非有指定，否則監視器的動作不會重疊，對於長時間執行的工作來說，這一點很重要。
* 監視器可具有動態間隔：等待時間可以根據某些條件來變更。
* 監視器可於某些條件成立時終止，或由其他程序加以終止。
* 監視器可以採用參數。 此範例會示範如何將相同的天氣監視程序套用至任何要求的地點和電話號碼。
* 監視器具有擴充性。 由於每個監視器都是協調流程執行個體，您可以建立多個監視器，而不必建立新函式或定義多個程式碼。
* 監視器可輕鬆整合至大型工作流程。 監視器可以是更複雜之協調流程函式的一個區段，也可以是[子協調流程](https://docs.microsoft.com/azure/azure-functions/durable-functions-sub-orchestrations)。

## <a name="configuring-twilio-integration"></a>設定 Twilio 整合

[!INCLUDE [functions-twilio-integration](../../includes/functions-twilio-integration.md)]

## <a name="configuring-weather-underground-integration"></a>設定 Weather Underground 整合

這個範例涉及使用 Weather Underground API 來檢查某地點的目前天氣狀況。

您首先需要的是 Weather Underground 帳戶。 您可以在 [https://www.wunderground.com/signup](https://www.wunderground.com/signup) 免費建立一個帳戶。 擁有帳戶後，您就需要取得 API 金鑰。 您可以造訪 [https://www.wunderground.com/weather/api](https://www.wunderground.com/weather/api) 並選取 [金鑰設定]，來取得此金鑰。 Stratus Developer 是免費的方案，且足已執行此範例。

擁有 API 金鑰後，將下列**應用程式設定**新增至您的函式應用程式。

| 應用程式設定名稱 | 值說明 |
| - | - |
| **WeatherUndergroundApiKey**  | 您的 Weather Underground API 金鑰。 |

## <a name="the-functions"></a>函式

本文說明範例應用程式中的函式如下：

* `E3_Monitor`：定期呼叫 `E3_GetIsClear` 的協調器函式。 如果 `E3_GetIsClear` 傳回 true，此函式會呼叫 `E3_SendGoodWeatherAlert`。
* `E3_GetIsClear`：會檢查某地點目前天氣狀況的活動函式。
* `E3_SendGoodWeatherAlert`：會透過 Twilio 傳送手機簡訊的活動函式。

下列各節說明用於 C# 指令碼和 JavaScript 的設定和程式碼。 適用於 Visual Studio 開發的程式碼顯示在本文結尾。
 
## <a name="the-weather-monitoring-orchestration-visual-studio-code-and-azure-portal-sample-code"></a>天氣監視協調流程 (Visual Studio Code 和 Azure 入口網站程式碼範例)

**E3_Monitor** 函式會使用協調器函式的標準 function.json。

[!code-json[Main](~/samples-durable-functions/samples/csx/E3_Monitor/function.json)]

以下是實作函式的程式碼：

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E3_Monitor/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_Monitor/index.js)]

此協調器函式會執行下列動作：

1. 取得 **MonitorRequest**，其中包含要監視的「地點」以及要用來作為簡訊通知傳送目的地的「電話號碼」。
2. 判斷監視器的到期時間。 為求簡單明瞭，這個範例會使用硬式編碼值。
3. 呼叫 **E3_GetIsClear** 來判斷所要求之地點的天空是否晴朗。
4. 如果天氣晴朗，則呼叫 **E3_SendGoodWeatherAlert** 以將簡訊通知傳送至要求的電話號碼。
5. 建立長期計時器以在下一個輪詢間隔繼續進行協調流程。 為求簡單明瞭，這個範例會使用硬式編碼值。
6. 持續執行直到 [CurrentUtcDateTime](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CurrentUtcDateTime) 超過監視器的到期時間或傳送了簡訊通知。

您可以藉由傳送多個 **MonitorRequests** 來同時執行多個協調器執行個體。 您可以指定要監視的地點以及要作為簡訊通知傳送目的地的電話號碼。

## <a name="strongly-typed-data-transfer-net-only"></a>強型別資料轉送 (僅限 .NET)

協調器需要許多資料，因此會使用[共用 POCO 物件](functions-reference-csharp.md#reusing-csx-code)在 C# 和 C# 指令碼中進行強型別資料轉送：[!code-csharp[Main](~/samples-durable-functions/samples/csx/shared/MonitorRequest.csx)]

[!code-csharp[Main](~/samples-durable-functions/samples/csx/shared/Location.csx)]

JavaScript 範例會使用一般 JSON 物件作為參數。

## <a name="helper-activity-functions"></a>協助程式活動函式

如同其他範例一樣，協助程式活動函式是使用 `activityTrigger` 觸發程序繫結的一般函式。 **E3_GetIsClear** 函式會使用 Weather Underground API 取得目前的天氣狀況，並判斷天空是否晴朗。 function.json 定義如下：

[!code-json[Main](~/samples-durable-functions/samples/csx/E3_GetIsClear/function.json)]

實作如下。 就像用於資料轉送的 POCO，用來處理 API 呼叫和剖析回應 JSON 的邏輯，在經過抽象化之後會成為 C# 中的共用類別。 您可以在 [Visual Studio 程式碼範例](#run-the-sample)中找到此邏輯。

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E3_GetIsClear/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_GetIsClear/index.js)]

**E3_SendGoodWeatherAlert** 函式會使用 Twilio 繫結來傳送手機簡訊，通知使用者這是散步的好時機。 其 function.json 很簡單：

[!code-json[Main](~/samples-durable-functions/samples/csx/E3_SendGoodWeatherAlert/function.json)]

以下是傳送手機簡訊的程式碼：

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E3_SendGoodWeatherAlert/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_SendGoodWeatherAlert/index.js)]

## <a name="run-the-sample"></a>執行範例

使用範例中所包含的 HTTP 觸發函式，您可以透過傳送下列 HTTP POST 要求來啟動協調流程：

```
POST https://{host}/orchestrators/E3_Monitor
Content-Length: 77
Content-Type: application/json

{ "Location": { "City": "Redmond", "State": "WA" }, "Phone": "+1425XXXXXXX" }
```
```
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Location: https://{host}/admin/extensions/DurableTaskExtension/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={SystemKey}
RetryAfter: 10

{"id": "f6893f25acf64df2ab53a35c09d52635", "statusQueryGetUri": "https://{host}/admin/extensions/DurableTaskExtension/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "sendEventPostUri": "https://{host}/admin/extensions/DurableTaskExtension/instances/f6893f25acf64df2ab53a35c09d52635/raiseEvent/{eventName}?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "terminatePostUri": "https://{host}/admin/extensions/DurableTaskExtension/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason={text}&taskHub=SampleHubVS&connection=Storage&code={systemKey}"}
```

   > [!NOTE]
   > 目前，JavaScript 協調流程入門函式無法傳回執行個體管理 URI。 這項功能會在未來版本中新增。

**E3_Monitor** 執行個體會啟動並查詢所要求地點的目前天氣狀況。 如果天氣晴朗，它會呼叫活動函式來傳送警示。否則，它會設定計時器。 當計時器時間結束時，協調流程會恢復執行。

在 Azure Functions 入口網站中查看函式記錄，就能看到協調流程的活動。

```
2018-03-01T01:14:41.649 Function started (Id=2d5fcadf-275b-4226-a174-f9f943c90cd1)
2018-03-01T01:14:42.741 Started orchestration with ID = '1608200bb2ce4b7face5fc3b8e674f2e'.
2018-03-01T01:14:42.780 Function completed (Success, Id=2d5fcadf-275b-4226-a174-f9f943c90cd1, Duration=1111ms)
2018-03-01T01:14:52.765 Function started (Id=b1b7eb4a-96d3-4f11-a0ff-893e08dd4cfb)
2018-03-01T01:14:52.890 Received monitor request. Location: Redmond, WA. Phone: +1425XXXXXXX.
2018-03-01T01:14:52.895 Instantiating monitor for Redmond, WA. Expires: 3/1/2018 7:14:52 AM.
2018-03-01T01:14:52.909 Checking current weather conditions for Redmond, WA at 3/1/2018 1:14:52 AM.
2018-03-01T01:14:52.954 Function completed (Success, Id=b1b7eb4a-96d3-4f11-a0ff-893e08dd4cfb, Duration=189ms)
2018-03-01T01:14:53.226 Function started (Id=80a4cb26-c4be-46ba-85c8-ea0c6d07d859)
2018-03-01T01:14:53.808 Function completed (Success, Id=80a4cb26-c4be-46ba-85c8-ea0c6d07d859, Duration=582ms)
2018-03-01T01:14:53.967 Function started (Id=561d0c78-ee6e-46cb-b6db-39ef639c9a2c)
2018-03-01T01:14:53.996 Next check for Redmond, WA at 3/1/2018 1:44:53 AM.
2018-03-01T01:14:54.030 Function completed (Success, Id=561d0c78-ee6e-46cb-b6db-39ef639c9a2c, Duration=62ms)
```

達到協調流程的逾時或偵測到天氣放晴後，協調流程就會[終止](durable-functions-instance-management.md#terminating-instances)。 您也可以使用其他函式內的 `TerminateAsync`，或叫用上述 202 回應中所參考的 **terminatePostUri** HTTP POST Webhook，將 `{text}` 取代為終止原因：

```
POST https://{host}/admin/extensions/DurableTaskExtension/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason=Because&taskHub=SampleHubVS&connection=Storage&code={systemKey}
```

## <a name="visual-studio-sample-code"></a>Visual Studio 範例程式碼

以下是 Visual Studio 專案中的單一 C# 檔案所示範的協調流程：

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/Monitor.cs)]

## <a name="next-steps"></a>後續步驟

這個範例已示範如何使用 Durable Functions 與[長期計時器](durable-functions-timers.md)和條件式邏輯來監視外部來源的狀態。 下一個範例會說明如何使用外部事件和[長期計時器](durable-functions-timers.md)來處理人為互動。

> [!div class="nextstepaction"]
> [執行人為互動範例](durable-functions-phone-verification.md)
