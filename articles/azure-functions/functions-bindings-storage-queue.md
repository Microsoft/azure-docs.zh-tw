---
title: Azure Functions 的 Azure 佇列儲存體繫結
description: 了解如何在 Azure Functions 中使用 Azure 佇列儲存體觸發程序和輸出繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
editor: ''
tags: ''
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.service: functions
ms.devlang: multiple
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 09/03/2018
ms.author: glenga
ms.custom: cc996988-fb4f-47
ms.openlocfilehash: 996a53751d6b8c6dd06084da371badb0c31d367f
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43667514"
---
# <a name="azure-queue-storage-bindings-for-azure-functions"></a>Azure Functions 的 Azure 佇列儲存體繫結

本文說明如何在 Azure Functions 中使用 Azure 佇列儲存體繫結。 Azure Functions 支援適用於佇列的觸發程序和輸出繫結。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs](http://www.nuget.org/packages/Microsoft.Azure.WebJobs) NuGet 套件 2.x 版中提供佇列儲存體繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Queue) GitHub 存放庫中。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) NuGet 套件 3.x 版中提供佇列儲存體繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="trigger"></a>觸發程序

在佇列上收到新項目時，可使用佇列觸發程序以啟動函式。 佇列訊息會當成函式輸入提供。

## <a name="trigger---example"></a>觸發程序 - 範例

請參閱特定語言的範例：

* [C#](#trigger---c-example)
* [C# 指令碼 (.csx)](#trigger---c-script-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---Java-example)

### <a name="trigger---c-example"></a>觸發程序 - C# 範例

下列範例所示範的 [C# 函式](functions-dotnet-class-library.md)會輪詢 `myqueue-items` 佇列，並在每次處理佇列項目時寫入記錄。

```csharp
public static class QueueFunctions
{
    [FunctionName("QueueTrigger")]
    public static void QueueTrigger(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        TraceWriter log)
    {
        log.Info($"C# function processed: {myQueueItem}");
    }
}
```

### <a name="trigger---c-script-example"></a>觸發程序 - C# 指令碼範例

下列範例示範的是 *function.json* 檔案中的佇列觸發程序繫結，以及使用該繫結的 [C# 指令碼 (.csx)](functions-reference-csharp.md) 程式碼。 此函式會輪詢 `myqueue-items` 佇列，並在每次處理佇列項目時寫入記錄。

以下是 *function.json* 檔案：

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

[設定](#trigger---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, 
    DateTimeOffset expirationTime, 
    DateTimeOffset insertionTime, 
    DateTimeOffset nextVisibleTime,
    string queueTrigger,
    string id,
    string popReceipt,
    int dequeueCount,
    TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem.AsString}\n" +
        $"queueTrigger={queueTrigger}\n" +
        $"expirationTime={expirationTime}\n" +
        $"insertionTime={insertionTime}\n" +
        $"nextVisibleTime={nextVisibleTime}\n" +
        $"id={id}\n" +
        $"popReceipt={popReceipt}\n" + 
        $"dequeueCount={dequeueCount}");
}
```

[使用方式](#trigger---usage)章節會說明 `myQueueItem` (由 function.json 中的`name` 屬性命名)。  [訊息中繼資料區段](#trigger---message-metadata)會說明所有其他顯示的變數。

### <a name="trigger---javascript-example"></a>觸發程序 - JavaScript 範例

下列範例示範的是 *function.json* 檔案中的佇列觸發程序繫結，以及使用該繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式會輪詢 `myqueue-items` 佇列，並在每次處理佇列項目時寫入記錄。

以下是 *function.json* 檔案：

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

[設定](#trigger---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context) {
    context.log('Node.js queue trigger function processed work item', context.bindings.myQueueItem);
    context.log('queueTrigger =', context.bindingData.queueTrigger);
    context.log('expirationTime =', context.bindingData.expirationTime);
    context.log('insertionTime =', context.bindingData.insertionTime);
    context.log('nextVisibleTime =', context.bindingData.nextVisibleTime);
    context.log('id =', context.bindingData.id);
    context.log('popReceipt =', context.bindingData.popReceipt);
    context.log('dequeueCount =', context.bindingData.dequeueCount);
    context.done();
};
```

[使用方式](#trigger---usage)章節會說明 `myQueueItem` (由 function.json 中的`name` 屬性命名)。  [訊息中繼資料區段](#trigger---message-metadata)會說明所有其他顯示的變數。

### <a name="trigger---java-example"></a>觸發程序 - Java 範例

下列 Java 範例所示範的儲存體佇列觸發程序函式會記錄放入 `myqueuename` 佇列的已觸發訊息。
 
 ```java
 @FunctionName("queueprocessor")
 public void run(
    @QueueTrigger(name = "msg",
                   queueName = "myqueuename",
                   connection = "myconnvarname") String message,
     final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
 ```

## <a name="trigger---attributes"></a>觸發程序 - 屬性
 
在 [C# 類別庫](functions-dotnet-class-library.md)中，使用下列屬性以設定佇列觸發程序：

* [QueueTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/QueueTriggerAttribute.cs)

  屬性的建構函式會採用佇列名稱進行監視，如下列範例所示：

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items")] string myQueueItem, 
      TraceWriter log)
  {
      ...
  }
  ```

  您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items", Connection = "StorageConnectionAppSetting")] string myQueueItem, 
      TraceWriter log)
  {
      ....
  }
  ```
 
  如需完整範例，請參閱[觸發程序 - C# 範例](#trigger---c-example)。

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  提供另一種方式來指定要使用的儲存體帳戶。 建構函式採用的是內含儲存體連接字串的應用程式設定名稱。 屬性可以套用在參數、方法或類別層級。 下列範例所示範的是類別層級與方法層級：

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("QueueTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ...
  }
  ```

要使用的儲存體帳戶按以下順序決定：

* `QueueTrigger` 屬性的 `Connection` 內容。
* `StorageAccount` 屬性套用至與 `QueueTrigger` 屬性相同的參數。
* `StorageAccount` 屬性套用至該函式。
* `StorageAccount` 屬性套用至該類別。
* 「AzureWebJobsStorage」應用程式設定。

## <a name="trigger---configuration"></a>觸發程式 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `QueueTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a| 必須設為 `queueTrigger`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction**| n/a | 僅限在 *function.json* 檔案中。 必須設為 `in`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 |
|**name** | n/a |代表函式程式碼中佇列的變數名稱。  | 
|**queueName** | **QueueName**| 要輪詢的佇列名稱。 | 
|**連接** | **連接** |應用程式設定的名稱包含要用於此繫結的儲存體連接字串。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為「MyStorage」，則函式執行階段會尋找名稱為「AzureWebJobsMyStorage」的應用程式設定。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>觸發程序 - 使用方式
 
在 C# 和 C# 指令碼中，使用方法參數 (例如 `string paramName`) 來存取訊息資料。 在 C# 指令碼中，`paramName` 是 *function.json* 之 `name` 屬性中指定的值。 您可以繫結至下列任何類型：

* 物件：Functions 執行階段會將 JSON 裝載還原序列化為程式碼中所定義之任意類別的執行個體。 
* `string`
* `byte[]`
* [CloudQueueMessage]

如果您嘗試繫結至 `CloudQueueMessage`，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

在 JavaScript 中，使用 `context.bindings.<name>` 存取佇列項目承載。 如果承載為 JSON，則會將已序列化的承載還原為物件。

## <a name="trigger---message-metadata"></a>觸發程序 - 訊息中繼資料

佇列觸發程序提供數個[中繼資料屬性](functions-triggers-bindings.md#binding-expressions---trigger-metadata)。 這些屬性可作為其他繫結中繫結運算式的一部分或程式碼中的參數使用。 這些是 [CloudQueueMessage](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.queue.cloudqueuemessage) 類別的屬性。

|屬性|類型|說明|
|--------|----|-----------|
|`QueueTrigger`|`string`|佇列承載 (如果為有效字串)。 如果佇列承載為字串，`QueueTrigger` 具有相同於 *function.json* 中由 `name` 屬性命名之變數的值。|
|`DequeueCount`|`int`|此訊息已從佇列清除的次數。|
|`ExpirationTime`|`DateTimeOffset`|訊息到期時間。|
|`Id`|`string`|佇列訊息識別碼。|
|`InsertionTime`|`DateTimeOffset`|訊息新增至佇列的時間。|
|`NextVisibleTime`|`DateTimeOffset`|下次顯示訊息的時間。|
|`PopReceipt`|`string`|訊息的離開通知。|

## <a name="trigger---poison-messages"></a>觸發程序 - 有害訊息

當佇列觸發程序函數失敗時，Azure Functions 會針對指定的佇列訊息重試該函數最多五次，包括第一次嘗試。 如果五次嘗試全都失敗，Functions 執行階段會將訊息新增至名為 *&lt;originalqueuename>-poison* 的佇列。 您可以撰寫函數，透過記錄或傳送通知表示需要手動處理，來處理有害佇列中的訊息。

若要手動處理有害訊息，請檢查佇列訊息的 [dequeueCount](#trigger---message-metadata)。

## <a name="trigger---polling-algorithm"></a>觸發程序 - 輪詢演算法

佇列觸發程序會實作隨機指數型倒退演算法，以降低閒置佇列輪詢對儲存體交易成本的影響。  找到訊息時，執行階段會等待兩秒，然後檢查另一個訊息；當找不到任何訊息時，它會等候大約四秒，然後再試一次。 連續嘗試取得佇列訊息失敗後，等候時間會持續增加，直到它到達等待時間上限 (預設值為一分鐘)。 可透過 [host.json 檔案](functions-host-json.md#queues)中的 `maxPollingInterval` 屬性來設定最長等待時間。

## <a name="trigger---concurrency"></a>觸發程序 - 並行

有多個佇列訊息在等候時，佇列觸發程序會擷取批次訊息，並同時叫用函式執行個體來處理它們。 依預設，批次大小為 16。 當要處理的數目減少到 8 時，執行階段就會取得另一個批次，並開始處理那些訊息。 因此，每個函式在一個虛擬機器 (VM) 上並行處理的訊息上限為 24。 這項限制會個別套用至每個 VM 上每個佇列觸發的函式。 如果您的函式應用程式擴展至多個 VM，則每個 VM 會等候觸發程序，並嘗試執行函式。 例如，如果函式應用程式擴展到 3 個 VM，一個佇列觸發函式的並行執行個體數上限會預設為 72。

可在 [host.json 檔案](functions-host-json.md#queues)中設定批次大小和取得新批次的閾值。 如果您需要將一個函式應用程式中佇列觸發函式的平行執行最小化，可以將批次大小設定為 1。 只要您的函式應用程式在單一虛擬機器 (VM) 上執行，這項設定就只會將並行排除。 

佇列觸發程序會自動防止函式處理佇列訊息多次；不需將函式撰寫成等冪函式。

## <a name="trigger---hostjson-properties"></a>觸發程序 - host.json 屬性

[host.json](functions-host-json.md#queues) 檔案包含控制佇列觸發程序行為的設定。

[!INCLUDE [functions-host-json-queues](../../includes/functions-host-json-queues.md)]

## <a name="output"></a>輸出

使用 Azure 佇列儲存體輸出繫結，將訊息寫入佇列。

## <a name="output---example"></a>輸出 - 範例

請參閱特定語言的範例：

* [C#](#output---c-example)
* [C# 指令碼 (.csx)](#output---c-script-example)
* [JavaScript](#output---javascript-example)
* [Java](#output---java-example)

### <a name="output---c-example"></a>輸出 - C# 範例

下列範例所示範的 [C# 函式](functions-dotnet-class-library.md)會為每個收到的 HTTP 要求建立佇列訊息。

```csharp
[StorageAccount("AzureWebJobsStorage")]
public static class QueueFunctions
{
    [FunctionName("QueueOutput")]
    [return: Queue("myqueue-items")]
    public static string QueueOutput([HttpTrigger] dynamic input,  TraceWriter log)
    {
        log.Info($"C# function processed: {input.Text}");
        return input.Text;
    }
}
```

### <a name="output---c-script-example"></a>輸出 - C# 指令碼範例

下列範例示範的是 *function.json* 檔案中的 HTTP 觸發程序繫結，以及使用該繫結的 [C# 指令碼 (.csx)](functions-reference-csharp.md) 程式碼。 此函式會針對每個收到的 HTTP 要求，使用 **CustomQueueMessage** 物件承載來建立佇列項目。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "$return",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting",
    }
  ]
}
``` 

[設定](#output---configuration)章節會說明這些屬性。

以下是建立單一佇列訊息的 C# 指令碼程式碼：

```cs
public class CustomQueueMessage
{
    public string PersonName { get; set; }
    public string Title { get; set; }
}

public static CustomQueueMessage Run(CustomQueueMessage input, TraceWriter log)
{
    return input;
}
```

您可以使用 `ICollector` 或 `IAsyncCollector` 參數一次傳送多個訊息。 以下 C# 指令碼程式碼會傳送多個訊息，一個使用 HTTP 要求資料，一個使用硬式編碼值：

```cs
public static void Run(
    CustomQueueMessage input, 
    ICollector<CustomQueueMessage> myQueueItems, 
    TraceWriter log)
{
    myQueueItems.Add(input);
    myQueueItems.Add(new CustomQueueMessage { PersonName = "You", Title = "None" });
}
```

### <a name="output---javascript-example"></a>輸出 - JavaScript 範例

下列範例示範的是 *function.json* 檔案中的 HTTP 觸發程序繫結，以及使用該繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式會針對每個收到的 HTTP 要求建立佇列項目。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "$return",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting",
    }
  ]
}
``` 

[設定](#output---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, input) {
    context.done(null, input.body);
};
```

您可以定義 `myQueueItem` 輸出繫結的訊息陣列，藉此一次傳送多個訊息。 下列 JavaScript 程式碼會對每個接收到的 HTTP 要求，使用硬式編碼值傳送兩個佇列訊息。

```javascript
module.exports = function(context) {
    context.bindings.myQueueItem = ["message 1","message 2"];
    context.done();
};
```

### <a name="output---java-example"></a>輸出 - Java 範例

 下列範例所示範的 Java 函式會在經由 HTTP 要求觸發時，建立佇列訊息。

```java
@FunctionName("httpToQueue")
@QueueOutput(name = "item", queueName = "myqueue-items", connection = "AzureWebJobsStorage")
 public String pushToQueue(
     @HttpTrigger(name = "request", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS)
     final String message,
     @HttpOutput(name = "response") final OutputBinding&lt;String&gt; result) {
       result.setValue(message + " has been added.");
       return message;
 }
 ```

在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值要寫入至佇列儲存體的參數使用 `@QueueOutput` 註釋。  參數類型應為 `OutputBinding<T>`，其中 T 是任何原生 Java 類型的 POJO。


## <a name="output---attributes"></a>輸出 - 屬性
 
在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [QueueAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/QueueAttribute.cs)。

該屬性會套用至 `out` 參數或函式的傳回值。 該屬性的建構函式會採用佇列名稱，如下列範例所示：

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items")]
public static string Run([HttpTrigger] dynamic input,  TraceWriter log)
{
    ...
}
```

您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items", Connection = "StorageConnectionAppSetting")]
public static string Run([HttpTrigger] dynamic input,  TraceWriter log)
{
    ...
}
```

如需完整範例，請參閱[輸出 - C# 範例](#output---c-example)。

您可以使用 `StorageAccount` 屬性來指定類別、方法或參數層級的儲存體帳戶。 如需詳細資訊，請參閱[觸發程序 - 屬性](#trigger---attribute)。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Queue` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 `queue`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction** | n/a | 必須設為 `out`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 |
|**name** | n/a | 代表函式程式碼中佇列的變數名稱。 設為 `$return` 以參考函式傳回值。| 
|**queueName** |**QueueName** | 佇列的名稱。 | 
|**連接** | **連接** |應用程式設定的名稱包含要用於此繫結的儲存體連接字串。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為「MyStorage」，則函式執行階段會尋找名稱為「AzureWebJobsMyStorage」的應用程式設定。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>輸出 - 使用方式
 
在 C# 和 C# 指令碼中，藉由使用方法參數 (例如 `out T paramName`) 寫入單一佇列訊息。 在 C# 指令碼中，`paramName` 是 *function.json* 之 `name` 屬性中指定的值。 您可以使用方法傳回類型，而不是 `out` 參數，而且 `T` 可以是下列類型之一：

* 可序列化為 JSON 的物件
* `string`
* `byte[]`
* [CloudQueueMessage] 

如果您嘗試繫結至 `CloudQueueMessage`，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

在 C# 和 C# 指令碼中，藉由使用下列其中一個類型，寫入多個佇列訊息： 

* `ICollector<T>` 或 `IAsyncCollector<T>`
* [CloudQueue](/dotnet/api/microsoft.windowsazure.storage.queue.cloudqueue)

在 JavaScript 函式中，使用 `context.bindings.<name>` 存取輸出佇列訊息。 對於佇列項目承載，可使用字串或 JSON 可序列化物件。


## <a name="exceptions-and-return-codes"></a>例外狀況和傳回碼

| 繫結 |  參考 |
|---|---|
| 佇列 | [佇列錯誤碼](https://docs.microsoft.com/rest/api/storageservices/queue-service-error-codes) |
| Bob、資料表、佇列 | [儲存體錯誤碼](https://docs.microsoft.com/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Bob、資料表、佇列 |  [疑難排解](https://docs.microsoft.com/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [移至使用佇列儲存體觸發程序的快速入門](functions-create-storage-queue-triggered-function.md)

> [!div class="nextstepaction"]
> [移至使用佇列儲存體輸出繫結的教學課程](functions-integrate-storage-queue-output-binding.md)

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

<!-- LINKS -->

[CloudQueueMessage]: /dotnet/api/microsoft.windowsazure.storage.queue.cloudqueuemessage
