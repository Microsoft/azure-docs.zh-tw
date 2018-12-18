---
title: Azure Functions C# 指令碼開發人員參考
description: 了解如何使用 C# 指令碼開發 Azure Functions。
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
editor: ''
tags: ''
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構
ms.service: functions
ms.devlang: dotnet
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 12/12/2017
ms.author: glenga
ms.openlocfilehash: 8206b4a0673c9744abf74e75a06d20e064475349
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39344268"
---
# <a name="azure-functions-c-script-csx-developer-reference"></a>Azure Functions C# 指令碼 (.csx) 開發人員參考

<!-- When updating this article, make corresponding changes to any duplicate content in functions-dotnet-class-library.md -->

本文是使用 C# 指令碼 (.csx) 開發 Azure Functions 的簡介。

Azure Functions 支援 C# 和 C# 指令碼程式設計語言。 如果您要尋求[在 Visual Studio 類別庫專案中使用 C#](functions-develop-vs.md) 上的指引，請參閱 [C# 開發人員參考](functions-dotnet-class-library.md)。

本文假設您已閱讀過 [Azure Functions 開發人員指南](functions-reference.md)。

## <a name="how-csx-works"></a>.csx 的運作方式

Azure Functions 的 C# 指令碼體驗是以 [Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/wiki/Introduction) 為基礎。 資料會透過方法引數流入您的 C# 函式。 引數名稱指定於 `function.json` 檔案中，而且有預先定義的名稱可用來存取函式記錄器和取消權杖等項目。

.csx 格式可讓您撰寫較少「重複使用」文字，只專注於撰寫 C# 函式。 只需定義 `Run` 方法，而不用在命名空間和類別中包裝所有項目。 像往常一樣，在檔案開頭包含任何組件參考和命名空間。

初始化執行個體時，會編譯函數應用程式的 .csx 檔案。 此編譯步驟表示與 C# 類別庫相較之下，C# 指令碼函式的冷啟動這類項目可能需要較長的時間。 此編譯步驟也是可在 Azure 入口網站中編輯 C# 指令碼函式但無法編輯 C# 類別庫的原因。

## <a name="binding-to-arguments"></a>繫結至引數

會透過 function.json 設定檔中的 `name` 屬性，將輸入或輸出資料繫結至 C# 指令碼函式參數。 下列範例示範佇列觸發之函式的 function.json 檔案和 run.csx 檔案。 接收來自佇列訊息之資料的參數命名為 `myQueueItem`，因為這是 `name` 屬性的值。

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

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem.AsString}");
}
```

[稍後在本文中](#referencing-external-assemblies)會說明 `#r` 陳述式。

## <a name="supported-types-for-bindings"></a>支援的繫結類型

每個繫結都有自己支援的類型；例如，Blob 觸發程序可以與字串參數、POCO 參數、`CloudBlockBlob` 參數或任何數個其他支援的類型搭配使用。 [Blob 繫結的繫結參考文章](functions-bindings-storage-blob.md#trigger---usage)會列出 Blob 觸發程序支援的所有參數類型。 如需詳細資訊，請參閱[觸發程序和繫結](functions-triggers-bindings.md)以及[每個繫結類型的繫結參考文件](functions-triggers-bindings.md#next-steps)。

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="referencing-custom-classes"></a>參考自訂類別

如果您需要使用自訂簡單的 CLR 物件 (POCO) 類別，則可以包含相同檔案內的類別定義，或將它放在個別檔案中。

下列範例示範內含 POCO 類別定義的 run.csx 範例。

```csharp
public static void Run(string myBlob, out MyClass myQueueItem)
{
    log.Verbose($"C# Blob trigger function processed: {myBlob}");
    myQueueItem = new MyClass() { Id = "myid" };
}

public class MyClass
{
    public string Id { get; set; }
}
```

POCO 類別的每個屬性都必須定義 getter 和 setter。

## <a name="reusing-csx-code"></a>重複使用 .csx 程式碼

您可以在您的 *run.csx* 檔案中使用其他 *.csx* 檔案中定義的類別和方法。 若要這樣做，請在您的 *run.csx* 檔案中使用 `#load` 指示詞。 在下列範例中，名為 `MyLogger` 的記錄常式已在 *myLogger.csx* 中共用，並使用 `#load` 指示詞載入至 *run.csx*︰

範例 run.csx ：

```csharp
#load "mylogger.csx"

public static void Run(TimerInfo myTimer, TraceWriter log)
{
    log.Verbose($"Log by run.csx: {DateTime.Now}");
    MyLogger(log, $"Log by MyLogger: {DateTime.Now}");
}
```

範例 mylogger.csx ：

```csharp
public static void MyLogger(TraceWriter log, string logtext)
{
    log.Verbose(logtext);
}
```

當您想要將使用 POCO 物件之函式間傳遞的資料設為強式型別時，使用共用的 .csx 檔案是常見的模式。 在下列簡化的範例中，HTTP 觸發程序和佇列觸發程序共用一個名為 `Order` 的 POCO 物件，來使排序資料成為強式類型︰

HTTP 觸發程序的範例 *run.csx*︰

```cs
#load "..\shared\order.csx"

using System.Net;

public static async Task<HttpResponseMessage> Run(Order req, IAsyncCollector<Order> outputQueueItem, TraceWriter log)
{
    log.Info("C# HTTP trigger function received an order.");
    log.Info(req.ToString());
    log.Info("Submitting to processing queue.");

    if (req.orderId == null)
    {
        return new HttpResponseMessage(HttpStatusCode.BadRequest);
    }
    else
    {
        await outputQueueItem.AddAsync(req);
        return new HttpResponseMessage(HttpStatusCode.OK);
    }
}
```

佇列觸發程序的範例 *run.csx*︰

```cs
#load "..\shared\order.csx"

using System;

public static void Run(Order myQueueItem, out Order outputQueueItem,TraceWriter log)
{
    log.Info($"C# Queue trigger function processed order...");
    log.Info(myQueueItem.ToString());

    outputQueueItem = myQueueItem;
}
```

範例 *order.csx*：

```cs
public class Order
{
    public string orderId {get; set; }
    public string custName {get; set;}
    public string custAddress {get; set;}
    public string custEmail {get; set;}
    public string cartId {get; set; }

    public override String ToString()
    {
        return "\n{\n\torderId : " + orderId +
                  "\n\tcustName : " + custName +             
                  "\n\tcustAddress : " + custAddress +             
                  "\n\tcustEmail : " + custEmail +             
                  "\n\tcartId : " + cartId + "\n}";             
    }
}
```

您可以使用包含 `#load` 指示詞的相對路徑：

* `#load "mylogger.csx"` 會載入位於函式資料夾中的檔案。
* `#load "loadedfiles\mylogger.csx"` 會載入位於函式資料夾的資料夾中的檔案。
* `#load "..\shared\mylogger.csx"` 會載入位於與函式資料夾相同層級的資料夾中的檔案 (也就是在 [wwwroot] 的正下方)。

`#load` 指示詞只適用於 .csx 檔案，而不適用於 .cs 檔案。

## <a name="binding-to-method-return-value"></a>繫結至方法傳回值

您可以使用 *function.json* 中的名稱 `$return`，使用輸出繫結的方法傳回值。 如需範例，請參閱[觸發程序和繫結](functions-triggers-bindings.md#using-the-function-return-value)。

唯有成功的函式執行一律導致傳回值傳遞至輸出繫結時，才使用此傳回值。 否則，使用 `ICollector` 或 `IAsyncCollector`，如下一節所示。

## <a name="writing-multiple-output-values"></a>撰寫多個輸出值

若要將多個值寫入至輸出繫結，或者如果成功的函式引動過程可能未導致任何項目傳遞至輸出繫結，請使用 [`ICollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) 或 [`IAsyncCollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs) 類型。 這些類型是在方法完成時，寫入至輸出繫結的唯寫集合。

這個範例會使用 `ICollector` 將多個佇列訊息寫入相同佇列：

```csharp
public static void Run(ICollector<string> myQueue, TraceWriter log)
{
    myQueue.Add("Hello");
    myQueue.Add("World!");
}
```

## <a name="logging"></a>記錄

若要使用 C# 將輸出記錄至串流記錄，請包含 `TraceWriter` 類型的引數。 建議您將它命名為 `log`。 避免在 Azure Functions 中使用 `Console.Write`。 

`TraceWriter` 已定義於 [Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/TraceWriter.cs)。 可以在 [host.json](functions-host-json.md) 中設定 `TraceWriter` 的記錄層級。

```csharp
public static void Run(string myBlob, TraceWriter log)
{
    log.Info($"C# Blob trigger function processed: {myBlob}");
}
```

> [!NOTE]
> 如需您可使用之較新記錄架構 (而非 `TraceWriter`) 的資訊，請參閱**監視 Azure Functions** 一文中的[在 C# 函式中寫入記錄](functions-monitoring.md#write-logs-in-c-functions)。

## <a name="async"></a>非同步處理

若要讓函式變成[非同步](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/async/)，請使用 `async` 關鍵字並傳回 `Task` 物件。

```csharp
public async static Task ProcessQueueMessageAsync(
        string blobName,
        Stream blobInput,
        Stream blobOutput)
{
    await blobInput.CopyToAsync(blobOutput, 4096);
}
```

您無法在非同步函式中使用 `out` 參數。 針對輸出繫結，請改為使用[函式傳回值](#binding-to-method-return-value)或[收集器物件](#writing-multiple-output-values)。

## <a name="cancellation-tokens"></a>取消權杖

可以接受 [CancellationToken](https://msdn.microsoft.com/library/system.threading.cancellationtoken.aspx) 參數的函式，讓作業系統能夠在函式即將終止時通知您的程式碼。 您可以使用此通知來確保函數不會在讓資料維持不一致狀態的情況下意外終止。

下列範例示範如何檢查即將終止的函式。

```csharp
using System;
using System.IO;
using System.Threading;

public static void Run(
    string inputText,
    TextWriter logger,
    CancellationToken token)
{
    for (int i = 0; i < 100; i++)
    {
        if (token.IsCancellationRequested)
        {
            logger.WriteLine("Function was cancelled at iteration {0}", i);
            break;
        }
        Thread.Sleep(5000);
        logger.WriteLine("Normal processing for queue message={0}", inputText);
    }
}
```

## <a name="importing-namespaces"></a>匯入命名空間

如果您需要匯入命名空間，可以如往常一樣利用 `using` 子句。

```csharp
using System.Net;
using System.Threading.Tasks;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
```

下列命名空間會自動匯入，所以是選擇性的︰

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`

## <a name="referencing-external-assemblies"></a>參考外部組件

若為架構組件，請使用 `#r "AssemblyName"` 指示詞加入參考。

```csharp
#r "System.Web.Http"

using System.Net;
using System.Net.Http;
using System.Threading.Tasks;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
```

Azure Functions 裝載環境會自動加入下列組件︰

* `mscorlib`
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`

simple-name 可能會參考下列組件 (例如，`#r "AssemblyName"`)：

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNet.WebHooks.Common`
* `Microsoft.Azure.NotificationHubs`

## <a name="referencing-custom-assemblies"></a>參考自訂組件

若要參考自訂組件，您可以使用「共用」組件或「私人」組件：
- 共用組件會共用於函式應用程式內的所有函式。 若要參考自訂組件，請將組件上傳至[函式應用程式根資料夾](functions-reference.md#folder-structure) (wwwroot) 中名為 `bin` 的資料夾。 
- 私人組件屬於所指定函式的內容，而且支援不同版本的側載。 應該在函式目錄的 `bin` 資料夾中上傳私人組件。 使用檔案名稱 (例如 `#r "MyAssembly.dll"`) 參考組件。 

如需如何將檔案上傳至函式資料夾的資訊，請參閱[套件管理](#using-nuget-packages)一節。

### <a name="watched-directories"></a>監看的目錄

系統會自動監看包含函式指令碼檔案之目錄中的組件變更。 若要監看其他目錄中的組件變更，請將它們新增至 [host.json](functions-host-json.md) 中的 `watchDirectories` 清單。

## <a name="using-nuget-packages"></a>使用 NuGet 套件

若要在 C# 函式中使用 NuGet 套件，請將 project.json  檔案上傳至函式應用程式檔案系統中的函式資料夾。 以下是範例 project.json  檔案，該檔案會加入對 Microsoft.ProjectOxford.Face 1.1.0 版的參考：

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.ProjectOxford.Face": "1.1.0"
      }
    }
   }
}
```

在 Azure Functions 1.x 中，只支援 .NET Framework 4.6；因此，請確認您的 project.json 檔案指定 `net46`，如下所示。

當您上傳 project.json  檔案時，執行階段會取得封裝並自動加入對封裝組件的參考。 您不需要加入 `#r "AssemblyName"` 指示詞。 若要使用 NuGet 套件中所定義的類型，只需要將所需的 `using` 陳述式新增至 run.csx 檔案。 

在 Functions 執行階段中，NuGet 還原會透過比較 `project.json` 和 `project.lock.json` 來運作。 如果檔案的日期和時間戳記**不**相符，會執行 NuGet 還原，且 NuGet 會下載更新的套件。 不過，如果檔案的日期和時間戳記**相符**，NuGet 不會執行還原。 因此，不應該部署 `project.lock.json`，因為它會導致 NuGet 略過套件還原。 若要避免部署鎖定檔案，請將 `project.lock.json` 加入至 `.gitignore` 檔案。

若要使用自訂 NuGet 摘要，請在函式應用程式根目錄的 *Nuget.Config* 檔案中指定摘要。 如需詳細資訊，請參閱[設定 NuGet 行為](/nuget/consume-packages/configuring-nuget-behavior)。

### <a name="using-a-projectjson-file"></a>使用 project.json 檔案

1. 在 Azure 入口網站中開啟函式。 [記錄] 索引標籤會顯示套件安裝輸出。
2. 若要上傳 project.json 檔案，請使用 Azure Functions 開發人員參考主題的[如何更新函式應用程式檔案](functions-reference.md#fileupdate)所述的其中一個方法。
3. 上傳 project.json  檔案之後，您會在函式的串流記錄檔中看到如下列範例所示的輸出：

```
2016-04-04T19:02:48.745 Restoring packages.
2016-04-04T19:02:48.745 Starting NuGet restore
2016-04-04T19:02:50.183 MSBuild auto-detection: using msbuild version '14.0' from 'D:\Program Files (x86)\MSBuild\14.0\bin'.
2016-04-04T19:02:50.261 Feeds used:
2016-04-04T19:02:50.261 C:\DWASFiles\Sites\facavalfunctest\LocalAppData\NuGet\Cache
2016-04-04T19:02:50.261 https://api.nuget.org/v3/index.json
2016-04-04T19:02:50.261
2016-04-04T19:02:50.511 Restoring packages for D:\home\site\wwwroot\HttpTriggerCSharp1\Project.json...
2016-04-04T19:02:52.800 Installing Newtonsoft.Json 6.0.8.
2016-04-04T19:02:52.800 Installing Microsoft.ProjectOxford.Face 1.1.0.
2016-04-04T19:02:57.095 All packages are compatible with .NETFramework,Version=v4.6.
2016-04-04T19:02:57.189
2016-04-04T19:02:57.189
2016-04-04T19:02:57.455 Packages restored.
```

## <a name="environment-variables"></a>環境變數

若要取得環境變數或應用程式設定值，請使用 `System.Environment.GetEnvironmentVariable`，如下列程式碼範例所示：

```csharp
public static void Run(TimerInfo myTimer, TraceWriter log)
{
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
    log.Info(GetEnvironmentVariable("AzureWebJobsStorage"));
    log.Info(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
}

public static string GetEnvironmentVariable(string name)
{
    return name + ": " +
        System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
}
```

[System.Configuration.ConfigurationManager.AppSettings](https://docs.microsoft.com/dotnet/api/system.configuration.configurationmanager.appsettings) 屬性是用於取得應用程式設定值的替代 API，但建議您使用 `GetEnvironmentVariable`，如下所示。

<a name="imperative-bindings"></a> 

## <a name="binding-at-runtime"></a>執行階段的繫結

在 C# 和其他 .NET 語言中，您可以使用相對於 *function.json* 中[*宣告式*](https://en.wikipedia.org/wiki/Declarative_programming)繫結的[命令式](https://en.wikipedia.org/wiki/Imperative_programming)繫結模式。 當繫結參數需要在執行階段而不是設計階段中計算時，命令式繫結非常有用。 利用此模式，您可以快速在您的函式程式碼中繫結至支援的輸入和輸出繫結。

定義命令式繫結，如下所示︰

- **請勿**在 *function.json* 中為您所需的命令式繫結併入項目。
- 傳入輸入參數 [`Binder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) 或 [`IBinder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs)。
- 使用下列 C# 模式來執行資料繫結。

```cs
using (var output = await binder.BindAsync<T>(new BindingTypeAttribute(...)))
{
    ...
}
```

`BindingTypeAttribute` 是可定義繫結的.NET 屬性，而 `T` 是該繫結類型所支援的輸入或輸出類型。 `T` 不能是 `out` 參數類型 (例如 `out JObject`)。 例如，Mobile Apps 資料表輸出繫結支援[六個輸出類型](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs#L17-L22)，但您只可以對 `T` 使用 [ICollector<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) 或 [IAsyncCollector<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs)。

### <a name="single-attribute-example"></a>單一屬性範例

下列範例程式碼會使用在執行階段定義的 blob 路徑來建立[儲存體 blob 輸出繫結](functions-bindings-storage-blob.md#output)，然後將字串寫入 blob。

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    using (var writer = await binder.BindAsync<TextWriter>(new BlobAttribute("samples-output/path")))
    {
        writer.Write("Hello World!!");
    }
}
```

[BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/BlobAttribute.cs) 會定義[儲存體 blob](functions-bindings-storage-blob.md) 輸入或輸出繫結，而 [TextWriter](https://msdn.microsoft.com/library/system.io.textwriter.aspx) 是支援的輸出繫結類型。

### <a name="multiple-attribute-example"></a>多個屬性範例

先前的範例會取得函數應用程式主要儲存體帳戶連接字串的應用程式設定 (也就是 `AzureWebJobsStorage`)。 您可以指定要用於儲存體帳戶的自訂應用程式設定，方法是新增 [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) 並將屬性陣列傳遞至 `BindAsync<T>()`。 使用 `Binder` 參數，而不是 `IBinder`。  例如︰

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    var attributes = new Attribute[]
    {    
        new BlobAttribute("samples-output/path"),
        new StorageAccountAttribute("MyStorageAccount")
    };

    using (var writer = await binder.BindAsync<TextWriter>(attributes))
    {
        writer.Write("Hello World!");
    }
}
```

下表列出每個繫結類型的 .NET 屬性，以及定義它們的套件。

> [!div class="mx-codeBreakAll"]
| 繫結 | 屬性 | 新增參考 |
|------|------|------|
| Cosmos DB | [`Microsoft.Azure.WebJobs.DocumentDBAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.CosmosDB"` |
| 事件中樞 | [`Microsoft.Azure.WebJobs.ServiceBus.EventHubAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.ServiceBus/EventHubs/EventHubAttribute.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.ServiceBus/ServiceBusAccountAttribute.cs) | `#r "Microsoft.Azure.Jobs.ServiceBus"` |
| Mobile Apps | [`Microsoft.Azure.WebJobs.MobileTableAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.MobileApps"` |
| 通知中樞 | [`Microsoft.Azure.WebJobs.NotificationHubAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.NotificationHubs/NotificationHubAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.NotificationHubs"` |
| 服務匯流排 | [`Microsoft.Azure.WebJobs.ServiceBusAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.ServiceBus/ServiceBusAttribute.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.ServiceBus/ServiceBusAccountAttribute.cs) | `#r "Microsoft.Azure.WebJobs.ServiceBus"` |
| 儲存體佇列 | [`Microsoft.Azure.WebJobs.QueueAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/QueueAttribute.cs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
| 儲存體 Blob | [`Microsoft.Azure.WebJobs.BlobAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/BlobAttribute.cs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
| 儲存體資料表 | [`Microsoft.Azure.WebJobs.TableAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/TableAttribute.cs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
| Twilio | [`Microsoft.Azure.WebJobs.TwilioSmsAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/TwilioSMSAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.Twilio"` |

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解觸發程序和繫結](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 的最佳做法](functions-best-practices.md)
