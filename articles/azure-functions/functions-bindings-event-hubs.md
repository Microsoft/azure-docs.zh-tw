---
title: Azure Functions 的 Azure 事件中樞繫結
description: 了解如何在 Azure Functions 中使用 Azure 事件中樞繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.assetid: daf81798-7acc-419a-bc32-b5a41c6db56b
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 11/08/2017
ms.author: glenga
ms.openlocfilehash: c99ad77bba60a4573faae1c857b3e6dc0203c4ab
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434682"
---
# <a name="azure-event-hubs-bindings-for-azure-functions"></a>Azure Functions 的 Azure 事件中樞繫結

此文章說明如何針對 Azure Functions 使用 [Azure 事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)繫結。 Azure Functions 支援事件中樞的觸發程序和輸出繫結。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

對於 Azure Functions 1.x 版，[Microsoft.Azure.WebJobs.ServiceBus](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.ServiceBus) NuGet 套件 2.x 版中會提供事件中樞繫結。
套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.ServiceBus/EventHubs) GitHub 存放庫中。


[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

對於 Functions 2.x，請使用 [Microsoft.Azure.WebJobs.Extensions.EventHubs](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs) 套件 3.x 版。
套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/master/src/Microsoft.Azure.WebJobs.Extensions.EventHubs) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="trigger"></a>觸發程序

使用事件中樞觸發程序將回應傳送至事件中樞事件資料流。 您必須具有事件中樞的讀取存取權，才能設定觸發程序。

事件中樞觸發程序函式觸發時，觸發它的訊息會以字串形式傳遞至函式。

## <a name="trigger---scaling"></a>觸發程序 - 調整規模

由事件中樞所觸發的函式，其每個執行個體只會由 1 個 [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor) 執行個體來提供支援。 事件中樞會確保只有 1 個 [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor) 執行個體可以在指定的分割區上取得租用。

例如，請考慮以下事件中樞：

* 10 個分割區。
* 1000 個平均散佈於所有分割區上的事件，且每個分割區中有 100 個訊息。

當您的函式首次啟用時，只會有 1 個該函式的執行個體。 讓我們將這個函式執行個體稱為 `Function_0`。 `Function_0` 具有單一 [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor)執行個體，且這個執行個體在 10 個分割區全都擁有租用。 此執行個體會從分割區 0-9 讀取事件。 從這裡開始，會發生下列其中一件事：

* **不需要新的函式執行個體**：`Function_0` 能夠在 Functions 的規模調整邏輯開始生效之前完全處理這 1000 個事件。 在此情況下，`Function_0` 會完全處理這 1000 個訊息。

* **再新增 1 個函式執行個體**：Functions 的規模調整邏輯判斷 `Function_0` 所擁有的訊息數目比它可處理的還多。 在此情況下，系統會建立一個新的函式應用程式執行個體 (`Function_1`)，以及一個新的 [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor) 執行個體。 事件中樞偵測到新的主機執行個體正在嘗試讀取訊息。 事件中樞會在其主機執行個體之間進行分割區的負載平衡。 例如，分割區 0-4 可能會指派給 `Function_0`，分割區 5-9 則指派給 `Function_1`。 

* **再新增 N 個函式執行個體**：Functions 的規模調整邏輯判斷 `Function_0` 和 `Function_1` 所擁有的訊息數目都比它們可處理的還多。 系統會建立新的函式應用程式執行個體 `Function_2`...`Functions_N`，其中 `N` 大於事件中樞分割區的數目。 在本例中，事件中樞同樣會將分割區負載平衡，在此案例中，會跨執行個體 `Function_0`...`Functions_9` 來進行。 

請注意 Functions 調整為 `N` 個 (也就是大於事件中樞分割區數目的數字) 執行個體的時候。 這樣做是為了確保一律會有 [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor) 執行個體可用來在分割區變成可從其他執行個體使用時，取得這些分割區的鎖定。 您只需針對函式執行個體執行時所使用的資源付費，不需要針對這個過度佈建付費。

當所有函式執行完成時 (不論有無錯誤)，系統就會在相關聯的儲存體帳戶中新增檢查點。 當檢查點檢查成功時，就永遠不會再次擷取所有的 1000 個訊息。

## <a name="trigger---example"></a>觸發程序 - 範例

請參閱特定語言的範例：

* [C#](#trigger---c-example)
* [C# 指令碼 (.csx)](#trigger---c-script-example)
* [F#](#trigger---f-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---java-example)

### <a name="trigger---c-example"></a>觸發程序 - C# 範例

下列範例顯示的 [C# 函式](functions-dotnet-class-library.md)，可記錄事件中樞觸發程序的訊息本文。

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string myEventHubMessage, TraceWriter log)
{
    log.Info($"C# Event Hub trigger function processed a message: {myEventHubMessage}");
}
```

若要取得函式程式碼中[事件中繼資料](#trigger---event-metadata)的存取權，請繫結至 [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) 物件 (`Microsoft.ServiceBus.Messaging` 需要 using 陳述式)。 您也可以在方法簽章中使用繫結運算式，以存取相同的屬性。  下列範例示範取得相同資料的兩種方式：

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run(
    [EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] EventData myEventHubMessage, 
    DateTime enqueuedTimeUtc, 
    Int64 sequenceNumber,
    string offset,
    TraceWriter log)
{
    log.Info($"Event: {Encoding.UTF8.GetString(myEventHubMessage.GetBytes())}");
    // Metadata accessed by binding to EventData
    log.Info($"EnqueuedTimeUtc={myEventHubMessage.EnqueuedTimeUtc}");
    log.Info($"SequenceNumber={myEventHubMessage.SequenceNumber}");
    log.Info($"Offset={myEventHubMessage.Offset}");
    // Metadata accessed by using binding expressions
    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"SequenceNumber={sequenceNumber}");
    log.Info($"Offset={offset}");
}
```

若要以批次方式接收事件，請讓 `string` 或 `EventData` 成為陣列：

```cs
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string[] eventHubMessages, TraceWriter log)
{
    foreach (var message in eventHubMessages)
    {
        log.Info($"C# Event Hub trigger function processed a message: {message}");
    }
}
```

### <a name="trigger---c-script-example"></a>觸發程序 - C# 指令碼範例

下列範例示範 function.json 檔案中的事件中樞觸發程序繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式會記錄事件中樞觸發程序的訊息本文。

下列範例顯示 *function.json* 檔案中的事件中樞繫結資料。 第一個範例是針對 Functions 2.x，而第二個範例則是針對 Functions 1.x。 


```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```
```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

以下是 C# 指令碼程式碼：

```cs
using System;

public static void Run(string myEventHubMessage, TraceWriter log)
{
    log.Info($"C# Event Hub trigger function processed a message: {myEventHubMessage}");
}
```

若要取得函式程式碼中[事件中繼資料](#trigger---event-metadata)的存取權，請繫結至 [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) 物件 (`Microsoft.ServiceBus.Messaging` 需要 using 陳述式)。 您也可以在方法簽章中使用繫結運算式，以存取相同的屬性。  下列範例示範取得相同資料的兩種方式：

```cs
#r "Microsoft.ServiceBus"
using System.Text;
using System;
using Microsoft.ServiceBus.Messaging;

public static void Run(EventData myEventHubMessage,
    DateTime enqueuedTimeUtc, 
    Int64 sequenceNumber,
    string offset,
    TraceWriter log)
{
    log.Info($"Event: {Encoding.UTF8.GetString(myEventHubMessage.GetBytes())}");
    // Metadata accessed by binding to EventData
    log.Info($"EnqueuedTimeUtc={myEventHubMessage.EnqueuedTimeUtc}");
    log.Info($"SequenceNumber={myEventHubMessage.SequenceNumber}");
    log.Info($"Offset={myEventHubMessage.Offset}");
    // Metadata accessed by using binding expressions
    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"SequenceNumber={sequenceNumber}");
    log.Info($"Offset={offset}");
}
```

若要以批次方式接收事件，請讓 `string` 或 `EventData` 成為陣列：

```cs
public static void Run(string[] eventHubMessages, TraceWriter log)
{
    foreach (var message in eventHubMessages)
    {
        log.Info($"C# Event Hub trigger function processed a message: {message}");
    }
}
```

### <a name="trigger---f-example"></a>觸發程序 - F# 範例

下列範例示範 function.json 檔案中的事件中樞觸發程序繫結，以及使用此繫結的 [F# 函式](functions-reference-fsharp.md)。 此函式會記錄事件中樞觸發程序的訊息本文。

下列範例顯示 *function.json* 檔案中的事件中樞繫結資料。 第一個範例是針對 Functions 2.x，而第二個範例則是針對 Functions 1.x。 


```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```
```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

以下是 F# 程式碼：

```fsharp
let Run(myEventHubMessage: string, log: TraceWriter) =
    log.Info(sprintf "F# eventhub trigger function processed work item: %s" myEventHubMessage)
```

### <a name="trigger---javascript-example"></a>觸發程序 - JavaScript 範例

下列範例示範 function.json 檔案中的事件中樞觸發程序繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式會讀取[事件中繼資料](#trigger---event-metadata)和記錄訊息。

下列範例顯示 *function.json* 檔案中的事件中樞繫結資料。 第一個範例是針對 Functions 2.x，而第二個範例則是針對 Functions 1.x。 


```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```
```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, eventHubMessage) {
    context.log('Event Hubs trigger function processed message: ', myEventHubMessage);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('SequenceNumber =', context.bindingData.sequenceNumber);
    context.log('Offset =', context.bindingData.offset);
     
    context.done();
};
```

若要批次接收事件，請在 *function.json* 檔案中將 `cardinality` 設定為 `many`，如下列範例所示。 第一個範例是針對 Functions 2.x，而第二個範例則是針對 Functions 1.x。 

```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```
```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "path": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, eventHubMessages) {
    context.log(`JavaScript eventhub trigger function called for message array ${eventHubMessages}`);
    
    eventHubMessages.forEach((message, index) => {
        context.log(`Processed message ${message}`);
        context.log(`EnqueuedTimeUtc = ${context.bindingData.enqueuedTimeUtcArray[index]}`);
        context.log(`SequenceNumber = ${context.bindingData.sequenceNumberArray[index]}`);
        context.log(`Offset = ${context.bindingData.offsetArray[index]}`);
    });

    context.done();
};
```

### <a name="trigger---java-example"></a>觸發程序 - Java 範例

下列範例示範 function.json 檔案中的事件中樞觸發程序繫結，以及使用此繫結的 [Java 函式](functions-reference-java.md)。 此函式會記錄事件中樞觸發程序的訊息本文。

```json
{
  "type": "eventHubTrigger",
  "name": "msg",
  "direction": "in",
  "eventHubName": "myeventhubname",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

```java
@FunctionName("ehprocessor")
public void eventHubProcessor(
  @EventHubTrigger(name = "msg",
                  eventHubName = "myeventhubname",
                  connection = "myconnvarname") String message,
       final ExecutionContext context ) 
       {
          context.getLogger().info(message);
 }
 ```

 在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值來自事件中樞的參數使用 `EventHubTrigger` 註釋。 具有這些附註的參數會使得函式在事件抵達時執行。  此註釋可以搭配原生 Java 類型、POJO 或使用「選擇性」<T>的可為 Null 值一起使用。 

## <a name="trigger---attributes"></a>觸發程序 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [EventHubTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.EventHubs/EventHubTriggerAttribute.cs) 屬性。

此屬性的建構函式接受事件中樞的名稱、取用者群組的名稱，以及包含連接字串的應用程式設定名稱。 如需這些設定的詳細資訊，請參閱[觸發程序組態](#trigger---configuration)一節。 以下是 `EventHubTriggerAttribute` 屬性範例：

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string myEventHubMessage, TraceWriter log)
{
    ...
}
```

如需完整範例，請參閱[觸發程序 - C# 範例](#trigger---c-example)。

## <a name="trigger---configuration"></a>觸發程式 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `EventHubTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 `eventHubTrigger`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction** | n/a | 必須設為 `in`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 |
|**name** | n/a | 代表函式程式碼中事件項目的變數名稱。 | 
|**路徑** |**EventHubName** | 僅限 Functions 1.x。 事件中樞的名稱。 當事件中樞名稱也呈現於連接字串時，該值會在執行階段覆寫這個屬性。 | 
|**eventHubName** |**EventHubName** | 僅限 Functions 2.x。 事件中樞的名稱。 當事件中樞名稱也呈現於連接字串時，該值會在執行階段覆寫這個屬性。 |
|**consumerGroup** |**ConsumerGroup** | 選擇性屬性，可設定用來訂閱中樞內事件的[取用者群組](../event-hubs/event-hubs-features.md#event-consumers)。 如果省略，則會使用 `$Default` 取用者群組。 | 
|**基數** | n/a | 適用於 JavaScript。 設定為 `many` 才能啟用批次處理。  如果省略或設訂為 `one`，會傳遞單一訊息至函數。 | 
|**連接** |**連接** | 應用程式設定的名稱，其中包含事件中樞命名空間的連接字串。 按一下[命名空間](../event-hubs/event-hubs-create.md#create-an-event-hubs-namespace)的 [連接資訊] 按鈕 (而不是事件中樞本身)，來複製此連接字串。 此連接字串至少必須具備讀取權限，才能啟動觸發程序。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---event-metadata"></a>觸發程序 - 事件中繼資料

事件中樞觸發程序提供數個[中繼資料屬性](functions-triggers-bindings.md#binding-expressions---trigger-metadata)。 這些屬性可作為其他繫結中繫結運算式的一部分或程式碼中的參數使用。 這些是 [EventData](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata) 類別的屬性。

|屬性|類型|說明|
|--------|----|-----------|
|`PartitionContext`|[PartitionContext](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.partitioncontext)|`PartitionContext` 執行個體。|
|`EnqueuedTimeUtc`|`DateTime`|加入佇列的時間 (UTC)。|
|`Offset`|`string`|相對於事件中樞資料分割串流的資料位移。 此位移是事件中樞串流中事件的標記或識別碼。 此識別碼在事件中樞串流的資料分割內是唯一的。|
|`PartitionKey`|`string`|事件資料應該傳送至的資料分割。|
|`Properties`|`IDictionary<String,Object>`|事件資料的使用者屬性。|
|`SequenceNumber`|`Int64`|事件的邏輯序號。|
|`SystemProperties`|`IDictionary<String,Object>`|系統屬性，包括事件資料。|

請參閱稍早在此文章中使用這些屬性的[程式碼範例](#trigger---example)。

## <a name="trigger---hostjson-properties"></a>觸發程序 - host.json 屬性

[host.json](functions-host-json.md#eventhub) 檔案包含可控制事件中樞觸發程序行為的設定。

[!INCLUDE [functions-host-json-event-hubs](../../includes/functions-host-json-event-hubs.md)]

## <a name="output"></a>輸出

使用事件中樞輸出繫結將事件寫入事件串流。 您必須具備事件中樞的傳送權限，才能將事件寫入其中。

請確定所需的套件參考已就緒︰[Functions 1.x](#packages---functions-1.x) 或 [Functions 2.x](#packages---functions-2.x) 

## <a name="output---example"></a>輸出 - 範例

請參閱特定語言的範例：

* [C#](#output---c-example)
* [C# 指令碼 (.csx)](#output---c-script-example)
* [F#](#output---f-example)
* [JavaScript](#output---javascript-example)
* [Java](#output---java-example)

### <a name="output---c-example"></a>輸出 - C# 範例

下列範例顯示將訊息寫入事件中樞，並使用方法傳回值作為輸出的 [C# 函式](functions-dotnet-class-library.md)：

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, TraceWriter log)
{
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
    return $"{DateTime.Now}";
}
```

### <a name="output---c-script-example"></a>輸出 - C# 指令碼範例

下列範例示範 function.json 檔案中的事件中樞觸發程序繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式會將訊息寫入事件中樞。

下列範例顯示 *function.json* 檔案中的事件中樞繫結資料。 第一個範例是針對 Functions 2.x，而第二個範例則是針對 Functions 1.x。 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```
```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

以下是可建立一則訊息的 C# 指令碼程式碼：

```cs
using System;

public static void Run(TimerInfo myTimer, out string outputEventHubMessage, TraceWriter log)
{
    String msg = $"TimerTriggerCSharp1 executed at: {DateTime.Now}";
    log.Verbose(msg);   
    outputEventHubMessage = msg;
}
```

以下是可建立多則訊息的 C# 指令碼程式碼：

```cs
public static void Run(TimerInfo myTimer, ICollector<string> outputEventHubMessage, TraceWriter log)
{
    string message = $"Event Hub message created at: {DateTime.Now}";
    log.Info(message);
    outputEventHubMessage.Add("1 " + message);
    outputEventHubMessage.Add("2 " + message);
}
```

### <a name="output---f-example"></a>輸出 - F# 範例

下列範例示範 function.json 檔案中的事件中樞觸發程序繫結，以及使用此繫結的 [F# 函式](functions-reference-fsharp.md)。 此函式會將訊息寫入事件中樞。

下列範例顯示 *function.json* 檔案中的事件中樞繫結資料。 第一個範例是針對 Functions 2.x，而第二個範例則是針對 Functions 1.x。 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```
```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

以下是 F# 程式碼：

```fsharp
let Run(myTimer: TimerInfo, outputEventHubMessage: byref<string>, log: TraceWriter) =
    let msg = sprintf "TimerTriggerFSharp1 executed at: %s" DateTime.Now.ToString()
    log.Verbose(msg);
    outputEventHubMessage <- msg;
```

### <a name="output---javascript-example"></a>輸出 - JavaScript 範例

下列範例示範 function.json 檔案中的事件中樞觸發程序繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式會將訊息寫入事件中樞。

下列範例顯示 *function.json* 檔案中的事件中樞繫結資料。 第一個範例是針對 Functions 2.x，而第二個範例則是針對 Functions 1.x。 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```
```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

以下是可傳送單一訊息的 JavaScript 程式碼：

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();
    context.log('Event Hub message created at: ', timeStamp);   
    context.bindings.outputEventHubMessage = "Event Hub message created at: " + timeStamp;
    context.done();
};
```

以下是可傳送多則訊息的 JavaScript 程式碼：

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();
    var message = 'Event Hub message created at: ' + timeStamp;

    context.bindings.outputEventHubMessage = [];

    context.bindings.outputEventHubMessage.push("1 " + message);
    context.bindings.outputEventHubMessage.push("2 " + message);
    context.done();
};
```

### <a name="output---java-example"></a>輸出 - Java 範例

下列範例會示範 Java 函式如何將包含目前時間的訊息寫入事件中樞。

```java
@}FunctionName("sendTime")
@EventHubOutput(name = "event", eventHubName = "samples-workitems", connection = "AzureEventHubConnection")
public String sendTime(
   @TimerTrigger(name = "sendTimeTrigger", schedule = "0 *&#47;5 * * * *") String timerInfo)  {
     return LocalDateTime.now().toString();
 }
 ```

在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值要發佈至事件中樞的參數使用 `@EventHubOutput` 註釋。  參數的類型應為 `OutputBinding<T>`，其中 T 是 POJO 或任何原生 Java 類型。 

## <a name="output---attributes"></a>輸出 - 屬性

對於 [C# 類別庫](functions-dotnet-class-library.md)，請使用 [EventHubAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.ServiceBus/EventHubs/EventHubAttribute.cs) 屬性。

此屬性的建構函式接受事件中樞的名稱，以及包含連接字串的應用程式設定名稱。 如需這些設定的詳細資訊，請參閱[輸入 - 組態](#output---configuration)一節。 以下是 `EventHub` 屬性範例：

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, TraceWriter log)
{
    ...
}
```

如需完整範例，請參閱[輸出 - C# 範例](#output---c-example)。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `EventHub` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設定為 "eventHub"。 |
|**direction** | n/a | 必須設定為 "out"。 當您在 Azure 入口網站中建立繫結時，會自動設定此參數。 |
|**name** | n/a | 函式程式碼中所使用的變數名稱，代表事件。 | 
|**路徑** |**EventHubName** | 僅限 Functions 1.x。 事件中樞的名稱。 當事件中樞名稱也呈現於連接字串時，該值會在執行階段覆寫這個屬性。 | 
|**eventHubName** |**EventHubName** | 僅限 Functions 2.x。 事件中樞的名稱。 當事件中樞名稱也呈現於連接字串時，該值會在執行階段覆寫這個屬性。 |
|**連接** |**連接** | 應用程式設定的名稱，其中包含事件中樞命名空間的連接字串。 按一下*命名空間*的 [連接資訊] 按鈕 (而不是事件中樞本身)，來複製此連接字串。 此連接字串必須具有傳送權限，才能將訊息傳送至事件資料流。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>輸出 - 使用方式

在 C# 和 C# 指令碼中，使用方法參數 (例如 `out string paramName`) 來傳送訊息。 在 C# 指令碼中，`paramName` 是 *function.json* 之 `name` 屬性中指定的值。 若要寫入多則訊息，您可以使用 `ICollector<string>` 或 `IAsyncCollector<string>` 取代 `out string`。

在 JavaScript 中，使用 `context.bindings.<name>` 存取輸出事件。 `<name>` 是 function.json 之 `name` 屬性中指定的值。

## <a name="exceptions-and-return-codes"></a>例外狀況和傳回碼

| 繫結 | 參考 |
|---|---|
| 事件中樞 | [操作指南](https://docs.microsoft.com/rest/api/eventhub/publisher-policy-operations) \(英文\) |

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
