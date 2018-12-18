---
title: 長期函式中的人為互動和逾時 - Azure
description: 了解如何處理 Azure Functions 之長期函式延伸模組中的人為互動和逾時。
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
ms.openlocfilehash: a62baf64e35dfad55f76138e2f1aaef65dd434be
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39036300"
---
# <a name="human-interaction-in-durable-functions---phone-verification-sample"></a>長期函式中的人為互動 - 電話驗證範例

這個範例會示範如何建置[長期函式](durable-functions-overview.md)協調流程，其中牽涉到人為互動。 每當在自動化程序中牽涉到真人，處理程序必須能夠傳送通知給人員，並以非同步方式接收回應。 它也必須允許人員沒有空的可能性。 (此最後一個部分就是逾時變得重要的地方。)

這個範例會實作以 SMS 為基礎的電話驗證系統。 這些類型的流量通常會在驗證客戶的電話號碼時，或針對多重要素驗證 (MFA) 使用。 這是功能強大的範例，因為整個實作是使用幾個小型函式來完成。 不需要外部資料存放區，例如資料庫。

## <a name="prerequisites"></a>必要條件

* [安裝 Durable Functions](durable-functions-install.md)。
* 完成 [Hello 序列](durable-functions-sequence.md)逐步解說。

## <a name="scenario-overview"></a>案例概觀

電話驗證是用來驗證您應用程式的使用者不是濫發垃圾郵件者，而且確實是他們所聲稱的人。 多重要素驗證是保護使用者帳戶免於駭客入侵的常見使用案例。 實作您自己的電話驗證的挑戰，是它需要與真人的**可設定狀態互動**。 終端使用者通常會提供一些代碼 (例如，4 位數數字)，並且必須在**合理的時間量**之內回應。

一般而言，Azure Functions 是無狀態 (如同其他平台上的其他許多雲端端點)，所以這些互動類型會明確牽涉到在外部的資料庫或其他某些永續性存放區中管理狀態。 此外，互動必須細分成可以協調在一起的多個函式。 例如，您需要至少一個函式以決定程式碼、保存在某處，以及將它傳送到使用者的電話。 而且您還需要至少一個其他的函式，以接收來自使用者的回應，並且以某種方式將其對應回原始函式呼叫，以進行程式碼驗證。 逾時也是確保安全性的重要層面。 很快就會變得相當複雜。

當您使用長期函式時，此案例的複雜性就會大幅減少。 如同您在這個範例所見，協調器函式可以輕鬆地管理可設定狀態互動，不需要涉及任何外部資料存放區。 因為協調器函式是「長期」的，所以這些互動流量也非常可靠。

## <a name="configuring-twilio-integration"></a>設定 Twilio 整合

[!INCLUDE [functions-twilio-integration](../../includes/functions-twilio-integration.md)]

## <a name="the-functions"></a>函式

本文會逐步解說範例應用程式中的下列函式：

* **E4_SmsPhoneVerification**
* **E4_SendSmsChallenge**

下列各節說明用於 C# 指令碼和 JavaScript 的設定和程式碼。 適用於 Visual Studio 開發的程式碼顯示在本文結尾。
 
## <a name="the-sms-verification-orchestration-visual-studio-code-and-azure-portal-sample-code"></a>SMS 驗證協調流程 (Visual Studio Code 和 Azure 入口網站範例程式碼) 

**E4_SmsPhoneVerification** 函式會針對協調器函式使用標準 function.json。

[!code-json[Main](~/samples-durable-functions/samples/csx/E4_SmsPhoneVerification/function.json)]

以下是實作函式的程式碼：

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E4_SmsPhoneVerification/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E4_SmsPhoneVerification/index.js)]

一旦啟動，此協調器函式會執行下列作業：

1. 取得電話號碼，SMS 通知會「傳送」給該號碼。
2. 呼叫 **E4_SendSmsChallenge** 以將 SMS 訊息傳送給使用者，並且傳回預期的 4 位數挑戰碼。
3. 建立長期計時器，它會觸發從目前時間起算 90 秒的時間。
4. 與計時器同時，等候來自使用者的 **SmsChallengeResponse** 事件。

使用者會收到具有四位數代碼的 SMS 訊息。 他們有 90 秒的時間將相同的 4 位數代碼傳送回協調器函式執行個體，以完成驗證程序。 如果提交錯誤的代碼，有額外三次嘗試可以進行修正 (在相同的 90 秒時間內)。

> [!NOTE]
> 起初可能不明顯，不過這個協調器函式完全具有決定性。 這是因為 `CurrentUtcDateTime` 屬性是用來計算計時器到期時間，且這個屬性會在協調器程式碼中此點的每個重新執行都傳回相同的值。 這對於確保每次重複呼叫 `Task.WhenAny` 都有相同 `winner` 結果而言非常重要。

> [!WARNING]
> 如果您已經在上述範例中接受挑戰回應，且不再需要使用計時器，請務必[取消計時器](durable-functions-timers.md)。

## <a name="send-the-sms-message"></a>傳送 SMS 訊息

**E4_SendSmsChallenge** 函式會使用 Twilio 繫結，將具有 4 位數代碼的 SMS 訊息傳送給使用者。 function.json 定義如下：

[!code-json[Main](~/samples-durable-functions/samples/csx/E4_SendSmsChallenge/function.json)]

以下是代碼，它會產生 4 位數挑戰碼並且傳送 SMS 訊息：

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E4_SendSmsChallenge/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E4_SendSmsChallenge/index.js)]

這個 **E4_SendSmsChallenge** 函式只會呼叫一次，即使處理程序損毀或重新執行也是如此。 這樣是好的，因為您不會想要讓使用者收到多則 SMS 訊息。 `challengeCode` 傳回值會自動保存，讓協調器函式一定知道正確的代碼是什麼。

## <a name="run-the-sample"></a>執行範例

使用範例中所包含的 HTTP 觸發函式，您可以透過傳送下列 HTTP POST 要求來啟動協調流程：

```
POST http://{host}/orchestrators/E4_SmsPhoneVerification
Content-Length: 14
Content-Type: application/json

"+1425XXXXXXX"
```
```
HTTP/1.1 202 Accepted
Content-Length: 695
Content-Type: application/json; charset=utf-8
Location: http://{host}/admin/extensions/DurableTaskExtension/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

{"id":"741c65651d4c40cea29acdd5bb47baf1","statusQueryGetUri":"http://{host}/admin/extensions/DurableTaskExtension/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}","sendEventPostUri":"http://{host}/admin/extensions/DurableTaskExtension/instances/741c65651d4c40cea29acdd5bb47baf1/raiseEvent/{eventName}?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}","terminatePostUri":"http://{host}/admin/extensions/DurableTaskExtension/instances/741c65651d4c40cea29acdd5bb47baf1/terminate?reason={text}&taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}"}
```

   > [!NOTE]
   > 目前，JavaScript 協調流程入門函式無法傳回執行個體管理 URI。 這項功能會在未來版本中新增。

協調器函式會接收提供的電話號碼，並且立即傳送 SMS 訊息給該電話號碼，訊息包含隨機產生的 4 位數驗證碼&mdash;，例如 2168。 此函式接著會在 90 秒的時間內等候回應。

若要回覆代碼，您可以使用另一個函式內的 `RaiseEventAsync`，或叫用上述 202 回應中參考的 **sendEventUrl** HTTP POST Webhook，將 `{eventName}` 取代為事件的名稱，`SmsChallengeResponse`：

```
POST http://{host}/admin/extensions/DurableTaskExtension/instances/741c65651d4c40cea29acdd5bb47baf1/raiseEvent/SmsChallengeResponse?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
Content-Length: 4
Content-Type: application/json

2168
```

如果您計時器到期之前傳送，協調流程就會完成且 `output` 欄位會設定為 `true`，指出驗證成功。

```
GET http://{host}/admin/extensions/DurableTaskExtension/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```
```
HTTP/1.1 200 OK
Content-Length: 144
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":"+1425XXXXXXX","output":true,"createdTime":"2017-06-29T19:10:49Z","lastUpdatedTime":"2017-06-29T19:12:23Z"}
```

如果您讓計時器到期，或者如果您輸入錯誤的代碼四次，您可以查詢狀態，並且看到 `false` 協調流程函式輸出，指出電話驗證失敗。

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 145

{"runtimeStatus":"Completed","input":"+1425XXXXXXX","output":false,"createdTime":"2017-06-29T19:20:49Z","lastUpdatedTime":"2017-06-29T19:22:23Z"}
```

## <a name="visual-studio-sample-code"></a>Visual Studio 範例程式碼

以下是 Visual Studio 專案中的單一 C# 檔案所示範的協調流程：

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/PhoneVerification.cs)]

## <a name="next-steps"></a>後續步驟

此範例已示範一些長期函式的進階功能，尤其是 `WaitForExternalEvent` 和 `CreateTimer`。 您已經看到這些功能如何與 `Task.WaitAny` 結合，以實作可靠的逾時系統，對於與真人的互動通常相當有用。 藉由閱讀一系列深入探討特定主題的文章，您可以深入了解如何使用長期函式。

> [!div class="nextstepaction"]
> [系列中第一篇文章](durable-functions-bindings.md)
