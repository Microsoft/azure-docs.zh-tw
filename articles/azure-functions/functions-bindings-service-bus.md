---
title: Azure Functions 的 Azure 服務匯流排繫結
description: 瞭解如何在 Azure Functions 中使用「Azure 服務匯流排」觸發程序和繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 04/01/2017
ms.author: glenga
ms.openlocfilehash: baf08f18fae7f82fff57616e2d86d3426e32a237
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387730"
---
# <a name="azure-service-bus-bindings-for-azure-functions"></a>Azure Functions 的 Azure 服務匯流排繫結

本文說明如何在 Azure Functions 中使用 Azure 服務匯流排繫結。 Azure Functions 支援服務匯流排佇列和主題的觸發程序和輸出繫結。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.ServiceBus](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.ServiceBus) NuGet 套件 2.x 版中會提供服務匯流排繫結。 

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.ServiceBus](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.ServiceBus) NuGet 套件 3.x 版會提供服務匯流排繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="trigger"></a>觸發程序

使用服務匯流排觸發程序來回應來自服務匯流排佇列或主題的訊息。 

## <a name="trigger---example"></a>觸發程序 - 範例

請參閱特定語言的範例：

* [C#](#trigger---c-example)
* [C# 指令碼 (.csx)](#trigger---c-script-example)
* [F#](#trigger---f-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---java-example)

### <a name="trigger---c-example"></a>觸發程序 - C# 範例

下列範例示範的 [C# 函式](functions-dotnet-class-library.md)可讀取[訊息中繼資料](#trigger---message-metadata)和記錄服務匯流排佇列訊息：

```cs
[FunctionName("ServiceBusQueueTriggerCSharp")]                    
public static void Run(
    [ServiceBusTrigger("myqueue", AccessRights.Manage, Connection = "ServiceBusConnection")] 
    string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    TraceWriter log)
{
    log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"DeliveryCount={deliveryCount}");
    log.Info($"MessageId={messageId}");
}
```

這個範例適用於 Azure Functions 第 1.x 版；第 2.x 版中已[省略存取權限參數](#trigger---configuration)。
 
### <a name="trigger---c-script-example"></a>觸發程序 - C# 指令碼範例

下列範例示範 function.json 檔案中的服務匯流排觸發程序繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式可讀取[訊息中繼資料](#trigger---message-metadata)和記錄服務匯流排佇列訊息。

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

以下是 C# 指令碼程式碼：

```cs
using System;

public static void Run(string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    TraceWriter log)
{
    log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");

    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"DeliveryCount={deliveryCount}");
    log.Info($"MessageId={messageId}");
}
```

### <a name="trigger---f-example"></a>觸發程序 - F# 範例

下列範例示範 function.json 檔案中的服務匯流排觸發程序繫結，以及使用此繫結的 [F# 函式](functions-reference-fsharp.md)。 此函式可記錄服務匯流排佇列訊息。 

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

以下是 F# 指令碼程式碼：

```fsharp
let Run(myQueueItem: string, log: TraceWriter) =
    log.Info(sprintf "F# ServiceBus queue trigger function processed message: %s" myQueueItem)
```

### <a name="trigger---javascript-example"></a>觸發程序 - JavaScript 範例

下列範例示範 function.json 檔案中的服務匯流排觸發程序繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式可讀取[訊息中繼資料](#trigger---message-metadata)和記錄服務匯流排佇列訊息。 

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

以下是 JavaScript 指令碼程式碼：

```javascript
module.exports = function(context, myQueueItem) {
    context.log('Node.js ServiceBus queue trigger function processed message', myQueueItem);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('DeliveryCount =', context.bindingData.deliveryCount);
    context.log('MessageId =', context.bindingData.messageId);
    context.done();
};
```

### <a name="trigger---java-example"></a>觸發程序 - Java 範例

下列範例示範 function.json 檔案中的服務匯流排觸發程序繫結，以及使用此繫結的 [Java 函式](functions-reference-java.md)。 函式會由放在服務匯流排佇列上的訊息觸發，且函式會記錄佇列訊息。

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "queueName": "myqueuename",
    "connection": "MyServiceBusConnection",
    "name": "msg",
    "type": "ServiceBusQueueTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

以下是 Java 程式碼：

```java
@FunctionName("sbprocessor")
 public void serviceBusProcess(
    @ServiceBusQueueTrigger(name = "msg",
                             queueName = "myqueuename",
                             connection = "myconnvarname") String message,
   final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
 ```

## <a name="trigger---attributes"></a>觸發程序 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用下列屬性以設定服務匯流排觸發程序：

* [ServiceBusTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusTriggerAttribute.cs)

  該屬性的建構函式會採用佇列名稱或標題和訂用帳戶。 在 Azure Functions 第 1.x 版中，您也可以指定連線的存取權限。 如果您未指定存取權限，則預設值為 `Manage`。 如需詳細資訊，請參閱[觸發程序 - 組態](#trigger---configuration)一節。

  以下範例顯示搭配字串參數使用的屬性：

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue")] string myQueueItem, TraceWriter log)
  {
      ...
  }
  ```

  您可以設定 `Connection` 屬性來指定要使用的服務匯流排帳戶，如下列範例所示：

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
      string myQueueItem, TraceWriter log)
  {
      ...
  }
  ```

  如需完整範例，請參閱[觸發程序 - C# 範例](#trigger---c-example)。

* [ServiceBusAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusAccountAttribute.cs)

  提供另一種方式來指定要使用的服務匯流排帳戶。 建構函式會採用內含服務匯流排連接字串的應用程式設定名稱。 屬性可以套用在參數、方法或類別層級。 下列範例所示範的是類別層級與方法層級：

  ```csharp
  [ServiceBusAccount("ClassLevelServiceBusAppSetting")]
  public static class AzureFunctions
  {
      [ServiceBusAccount("MethodLevelServiceBusAppSetting")]
      [FunctionName("ServiceBusQueueTriggerCSharp")]
      public static void Run(
          [ServiceBusTrigger("myqueue", AccessRights.Manage)] 
          string myQueueItem, TraceWriter log)
  {
      ...
  }
  ```

要使用的服務匯流排帳戶按以下順序決定：

* `ServiceBusTrigger` 屬性的 `Connection` 內容。
* `ServiceBusAccount` 屬性套用至與 `ServiceBusTrigger` 屬性相同的參數。
* `ServiceBusAccount` 屬性套用至該函式。
* `ServiceBusAccount` 屬性套用至該類別。
* "AzureWebJobsServiceBus" 應用程式設定。

## <a name="trigger---configuration"></a>觸發程式 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `ServiceBusTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設定為 "serviceBusTrigger"。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction** | n/a | 必須設定為 "in"。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 |
|**name** | n/a | 代表函式程式碼中佇列或主題訊息的變數名稱。 設為 "$return" 以參考函式傳回值。 | 
|**queueName**|**QueueName**|要監視的佇列名稱。  只有在監視佇列時設定 (不適用於主題)。
|**topicName**|**TopicName**|要監視的主題名稱。 只有在監視主題時設定 (不適用於佇列)。|
|**subscriptionName**|**SubscriptionName**|要監視的訂用帳戶名稱。 只有在監視主題時設定 (不適用於佇列)。|
|**連接**|**連接**|應用程式設定的名稱包含要用於此繫結的服務匯流排連接字串。 如果應用程式設定名稱是以 "AzureWebJobs" 開頭，您只能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為 "MyServiceBus"，則 Functions 執行階段會尋找名稱為 "AzureWebJobsMyServiceBus" 的應用程式設定。 如果您將 `connection` 保留空白，則 Functions 執行階段會使用應用程式設定中名稱為 "AzureWebJobsServiceBus" 的預設服務匯流排連接字串。<br><br>若要取得連接字串，請遵循[取得管理認證](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md#obtain-the-management-credentials)所示的步驟。 連接字串必須是用於服務匯流排命名空間，而不限於特定佇列或主題。 |
|**accessRights**|**Access**|連接字串的存取權限。 可用值為 `manage` 和 `listen`。 預設值是 `manage`，這表示 `connection` 已具備**管理**權限。 如果您使用沒有**管理**權限的連接字串，請將 `accessRights` 設定為 "listen"。 否則，Functions 執行階段在嘗試執行需要管理權限的作業時可能會失敗。 在 Azure Functions 第 2.x 版中，這個屬性無法使用，因為最新版的儲存體 SDK 不支援管理作業。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>觸發程序 - 使用方式

在 C# 和 C# 指令碼中，您可以針對佇列或主題訊息使用下列參數類型：

* `string` - 如果訊息是文字。
* `byte[]` - 適用於二進位資料。
* 自訂類型 - 如果訊息包含 JSON，Azure Functions 會嘗試將 JSON 資料還原序列化。
* `BrokeredMessage` - 利用 [BrokeredMessage.GetBody<T>()](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody?view=azure-dotnet#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1) 方法提供您還原序列化的訊息。

這些參數適用於 Azure Functions 版本 1.x；針對 2.x，請使用 [`Message`](https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.message)，而不是使用 `BrokeredMessage`。

在 JavaScript 中，使用 `context.bindings.<name from function.json>` 來存取佇列或主題訊息。 服務匯流排訊息會以字串或 JSON 物件的形式傳遞至函式。

## <a name="trigger---poison-messages"></a>觸發程序 - 有害訊息

無法在 Azure Functions 中控制或設定有害訊息處理。 服務匯流排會自行處理有害訊息。

## <a name="trigger---peeklock-behavior"></a>觸發程序 - PeekLock 行為

Functions 執行階段會在 [PeekLock 模式](../service-bus-messaging/service-bus-performance-improvements.md#receive-mode)中收到訊息。 它會在函數成功完成時，於訊息中呼叫 `Complete`，或是在函數失敗時呼叫 `Abandon`。 如果函式執行時間較 `PeekLock` 逾時還長，只要函式正在執行中，就會自動更新鎖定。 

Functions 1.x 可讓您在 host.json 中設定 `autoRenewTimeout`，其對應至 [OnMessageOptions.AutoRenewTimeout](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.onmessageoptions.autorenewtimeout?view=azure-dotnet#Microsoft_ServiceBus_Messaging_OnMessageOptions_AutoRenewTimeout)。 根據服務匯流排文件，此設定允許的上限為 5 分鐘，而您可以將 Functions 時間限制從預設的 5 分鐘增加至 10 分鐘。 您不會想對服務匯流排函式這麼做，因為您會超過服務匯流排更新限制。

## <a name="trigger---message-metadata"></a>觸發程序 - 訊息中繼資料

服務匯流排觸發程序提供數個[中繼資料屬性](functions-triggers-bindings.md#binding-expressions---trigger-metadata)。 這些屬性可作為其他繫結中繫結運算式的一部分或程式碼中的參數使用。 這些是 [BrokeredMessage](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 類別的屬性。

|屬性|類型|說明|
|--------|----|-----------|
|`DeliveryCount`|`Int32`|傳遞數目。|
|`DeadLetterSource`|`string`|無效信件來源。|
|`ExpiresAtUtc`|`DateTime`|到期時間 (UTC)。|
|`EnqueuedTimeUtc`|`DateTime`|加入佇列的時間 (UTC)。|
|`MessageId`|`string`|服務匯流排可用來識別重複訊息的使用者定義值 (如果已啟用)。|
|`ContentType`|`string`|由傳送者和接收者使用於應用程式特定邏輯的內容類型識別碼。|
|`ReplyTo`|`string`|回覆佇列位址。|
|`SequenceNumber`|`Int64`|由服務匯流排指派給訊息的唯一編號。|
|`To`|`string`|傳送位址。|
|`Label`|`string`|應用程式專用標籤。|
|`CorrelationId`|`string`|相互關連識別碼。|
|`Properties`|`IDictionary<String,Object>`|應用程式專用訊息屬性。|

請參閱稍早在本文中使用這些屬性的[程式碼範例](#trigger---example)。

## <a name="trigger---hostjson-properties"></a>觸發程序 - host.json 屬性

[host.json](functions-host-json.md#servicebus) 檔案包含可控制服務匯流排觸發程序行為的設定。

[!INCLUDE [functions-host-json-event-hubs](../../includes/functions-host-json-service-bus.md)]

## <a name="output"></a>輸出

使用 Azure 服務匯流排輸出繫結來傳送佇列或主題訊息。

## <a name="output---example"></a>輸出 - 範例

請參閱特定語言的範例：

* [C#](#output---c-example)
* [C# 指令碼 (.csx)](#output---c-script-example)
* [F#](#output---f-example)
* [JavaScript](#output---javascript-example)
* [Java](#output--java-example)

### <a name="output---c-example"></a>輸出 - C# 範例

下列範例示範的 [C# 函式](functions-dotnet-class-library.md)可記錄服務匯流排佇列訊息：

```cs
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue", Connection = "ServiceBusConnection")]
public static string ServiceBusOutput([HttpTrigger] dynamic input, TraceWriter log)
{
    log.Info($"C# function processed: {input.Text}");
    return input.Text;
}
```

### <a name="output---c-script-example"></a>輸出 - C# 指令碼範例

下列範例示範 function.json 檔案中的服務匯流排輸出繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式會使用計時器觸發程序，每隔 15 秒傳送一則佇列訊息。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "bindings": [
        {
            "schedule": "0/15 * * * * *",
            "name": "myTimer",
            "runsOnStartup": true,
            "type": "timerTrigger",
            "direction": "in"
        },
        {
            "name": "outputSbQueue",
            "type": "serviceBus",
            "queueName": "testqueue",
            "connection": "MyServiceBusConnection",
            "direction": "out"
        }
    ],
    "disabled": false
}
```

以下是可建立單一訊息的 C# 指令碼程式碼：

```cs
public static void Run(TimerInfo myTimer, TraceWriter log, out string outputSbQueue)
{
    string message = $"Service Bus queue message created at: {DateTime.Now}";
    log.Info(message); 
    outputSbQueue = message;
}
```

以下是可建立多則訊息的 C# 指令碼程式碼：

```cs
public static void Run(TimerInfo myTimer, TraceWriter log, ICollector<string> outputSbQueue)
{
    string message = $"Service Bus queue messages created at: {DateTime.Now}";
    log.Info(message); 
    outputSbQueue.Add("1 " + message);
    outputSbQueue.Add("2 " + message);
}
```

### <a name="output---f-example"></a>輸出 - F# 範例

下列範例示範 function.json 檔案中的服務匯流排觸發程序繫結，以及使用此繫結的 [F# 指令碼函式](functions-reference-fsharp.md)。 此函式會使用計時器觸發程序，每隔 15 秒傳送一則佇列訊息。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "bindings": [
        {
            "schedule": "0/15 * * * * *",
            "name": "myTimer",
            "runsOnStartup": true,
            "type": "timerTrigger",
            "direction": "in"
        },
        {
            "name": "outputSbQueue",
            "type": "serviceBus",
            "queueName": "testqueue",
            "connection": "MyServiceBusConnection",
            "direction": "out"
        }
    ],
    "disabled": false
}
```

以下是可建立單一訊息的 F# 指令碼程式碼：

```fsharp
let Run(myTimer: TimerInfo, log: TraceWriter, outputSbQueue: byref<string>) =
    let message = sprintf "Service Bus queue message created at: %s" (DateTime.Now.ToString())
    log.Info(message)
    outputSbQueue = message
```

### <a name="output---javascript-example"></a>輸出 - JavaScript 範例

下列範例示範 function.json 檔案中的服務匯流排觸發程序繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式會使用計時器觸發程序，每隔 15 秒傳送一則佇列訊息。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "bindings": [
        {
            "schedule": "0/15 * * * * *",
            "name": "myTimer",
            "runsOnStartup": true,
            "type": "timerTrigger",
            "direction": "in"
        },
        {
            "name": "outputSbQueue",
            "type": "serviceBus",
            "queueName": "testqueue",
            "connection": "MyServiceBusConnection",
            "direction": "out"
        }
    ],
    "disabled": false
}
```

以下是可建立單一訊息的 JavaScript 指令碼程式碼：

```javascript
module.exports = function (context, myTimer) {
    var message = 'Service Bus queue message created at ' + timeStamp;
    context.log(message);   
    context.bindings.outputSbQueue = message;
    context.done();
};
```

以下是可建立多則訊息的 JavaScript 指令碼程式碼：

```javascript
module.exports = function (context, myTimer) {
    var message = 'Service Bus queue message created at ' + timeStamp;
    context.log(message);   
    context.bindings.outputSbQueue = [];
    context.bindings.outputSbQueue.push("1 " + message);
    context.bindings.outputSbQueue.push("2 " + message);
    context.done();
};
```


### <a name="output---java-example"></a>輸出 - Java 範例

下列範例所示範的 Java 函式，會在經由 HTTP 要求觸發時，傳送訊息給服務匯流排佇列 `myqueue`。

```java
@FunctionName("httpToServiceBusQueue")
@ServiceBusQueueOutput(name = "message", queueName = "myqueue", connection = "AzureServiceBusConnection")
public String pushToQueue(
  @HttpTrigger(name = "request", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS)
  final String message,
  @HttpOutput(name = "response") final OutputBinding<T> result ) {
      result.setValue(message + " has been sent.");
      return message;
 }
 ```

 在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值要寫入至服務匯流排佇列的函式參數使用 `@QueueOutput` 註釋。  參數類型應為 `OutputBinding<T>`，其中 T 是任何原生 Java 類型的 POJO。

## <a name="output---attributes"></a>輸出 - 屬性

在 [C# 類別程式庫](functions-dotnet-class-library.md)中，使用 [ServiceBusAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusAttribute.cs)。

該屬性的建構函式會採用佇列名稱或標題和訂用帳戶。 您也可以指定連線的存取權限。 [輸出 - 組態](#output---configuration)一節說明如何選擇存取權限設定。 以下範例顯示套用至函式傳回值的屬性：

```csharp
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue")]
public static string Run([HttpTrigger] dynamic input, TraceWriter log)
{
    ...
}
```

您可以設定 `Connection` 屬性來指定要使用的服務匯流排帳戶，如下列範例所示：

```csharp
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue", Connection = "ServiceBusConnection")]
public static string Run([HttpTrigger] dynamic input, TraceWriter log)
{
    ...
}
```

如需完整範例，請參閱[輸出 - C# 範例](#output---c-example)。

您可以使用 `ServiceBusAccount` 屬性來指定要在類別、方法或參數層級使用的服務匯流排帳戶。  如需詳細資訊，請參閱[觸發程序 - 屬性](#trigger---attributes)。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `ServiceBus` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 "serviceBus"。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction** | n/a | 必須設定為 "out"。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 |
|**name** | n/a | 代表函式程式碼中佇列或主題的變數名稱。 設為 "$return" 以參考函式傳回值。 | 
|**queueName**|**QueueName**|佇列的名稱。  只有在傳送佇列訊息時設定 (不適用於主題)。
|**topicName**|**TopicName**|要監視的主題名稱。 只有在傳送主題訊息時設定 (不適用於佇列)。|
|**連接**|**連接**|應用程式設定的名稱包含要用於此繫結的服務匯流排連接字串。 如果應用程式設定名稱是以 "AzureWebJobs" 開頭，您只能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為 "MyServiceBus"，則 Functions 執行階段會尋找名稱為 "AzureWebJobsMyServiceBus" 的應用程式設定。 如果您將 `connection` 保留空白，則 Functions 執行階段會使用應用程式設定中名稱為 "AzureWebJobsServiceBus" 的預設服務匯流排連接字串。<br><br>若要取得連接字串，請遵循[取得管理認證](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md#obtain-the-management-credentials)所示的步驟。 連接字串必須是用於服務匯流排命名空間，而不限於特定佇列或主題。|
|**accessRights**|**Access**|連接字串的存取權限。 可用值為 `manage` 和 `listen`。 預設值是 `manage`，這表示 `connection` 已具備**管理**權限。 如果您使用沒有**管理**權限的連接字串，請將 `accessRights` 設定為 "listen"。 否則，Functions 執行階段在嘗試執行需要管理權限的作業時可能會失敗。 在 Azure Functions 第 2.x 版中，這個屬性無法使用，因為最新版的儲存體 SDK 不支援管理作業。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>輸出 - 使用方式

在 Azure Functions 1.x 中，執行階段會建立佇列 (如果佇列不存在)，且您已將 `accessRights` 設為 `manage`。 在 Functions 第 2.x 版中，佇列或主題必須已經存在；如果您指定的佇列或主題不存在，此函式就會失敗。 

在 C# 和 C# 指令碼中，您可以針對輸出繫結使用下列參數類型：

* `out T paramName` - `T` 可以是任何可序列化 JSON 的類型。 當函式結束時，如果參數值為 Null，則 Functions 會使用 Null 物件建立訊息。
* `out string` - 當函式結束時，如果參數值為 Null，則 Functions 不會建立一則訊息。
* `out byte[]` - 當函式結束時，如果參數值為 Null，則 Functions 不會建立一則訊息。
* `out BrokeredMessage` - 當函式結束時，如果參數值為 Null，則 Functions 不會建立一則訊息。
* `ICollector<T>` 或 `IAsyncCollector<T>` - 適用於建立多個訊息。 當您呼叫 `Add` 方法時，就會建立一則訊息。

在非同步函式中，使用傳回值或 `IAsyncCollector`，而不是 `out` 參數。

這些參數適用於 Azure Functions 版本 1.x；針對 2.x，請使用 [`Message`](https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.message)，而不是使用 `BrokeredMessage`。

在 JavaScript 中，使用 `context.bindings.<name from function.json>` 來存取佇列或主題。 您可以指派字串、位元組陣列或 Javascript 物件 (還原序列化為 JSON) 給 `context.binding.<name>`。

## <a name="exceptions-and-return-codes"></a>例外狀況和傳回碼

| 繫結 | 參考 |
|---|---|
| 服務匯流排 | [服務匯流排錯誤碼](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-messaging-exceptions) |
| 服務匯流排 | [服務匯流排限制](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-quotas) |

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
