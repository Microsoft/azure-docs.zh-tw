---
title: Azure Functions F# 開發人員參考 | Microsoft Docs
description: 了解如何使用 F# 開發 Azure Functions。
services: functions
documentationcenter: fsharp
author: sylvanc
manager: jbronsk
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構, F#
ms.assetid: e60226e5-2630-41d7-9e5b-9f9e5acc8e50
ms.service: azure-functions
ms.devlang: fsharp
ms.topic: reference
ms.date: 09/09/2016
ms.author: syclebsc
ms.openlocfilehash: 5593f76511f43106d6743a158b051e118ef2a4a6
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46125250"
---
# <a name="azure-functions-f-developer-reference"></a>Azure Functions F# 開發人員參考

Azure Functions 的 F# 是可在雲端輕鬆執行程式碼片段或「函式」的解決方案。 資料會透過函式引數流入您的 F# 函式。 引數名稱會指定於 `function.json`中，而且有預先定義的名稱可用來存取函式記錄器和取消權杖等項目。

本文假設您已經讀過 [Azure Functions 開發人員參考](functions-reference.md)。

## <a name="how-fsx-works"></a>.fsx 的運作方式
`.fsx` 檔案是 F# 指令碼。 您可以將它視為包含在單一檔案內的 F# 專案。 檔案包含程式的程式碼 (在此案例中為您的 Azure 函式) 和用於管理相依性的指示詞。

當您使用 Azure 函式的 `.fsx` 時，其中會自動包含常用的必要組件，讓您得以專注於函式本身而非「重複使用」的程式碼。

## <a name="folder-structure"></a>資料夾結構

F# 指令碼專案的資料夾結構如下所示：

```
FunctionsProject
 | - MyFirstFunction
 | | - run.fsx
 | | - function.json
 | | - function.proj
 | - MySecondFunction
 | | - run.fsx
 | | - function.json
 | | - function.proj
 | - host.json
 | - extensions.csproj
 | - bin
```

共用的 [host.json] (functions-host-json.md) 檔案可用來設定函數應用程式。 每個函式都具有本身的程式碼檔案 (.fsx) 和繫結設定檔 (function.json)。

在函式執行階段的[版本 2.x](functions-versions.md) 中所需的繫結擴充功能，是以 `bin` 資料夾中的實際程式庫檔案在 `extensions.csproj` 檔案中所定義。 在本機開發時，您必須[註冊繫結擴充功能](functions-triggers-bindings.md#local-development-azure-functions-core-tools)。 開發 Azure 入口網站中的函式時，就會為您完成這項註冊。

## <a name="binding-to-arguments"></a>繫結至引數
如 [Azure Functions 觸發程序和繫結開發人員參考](functions-triggers-bindings.md)所述，每個繫結都支援某幾組引數。 例如，Blob 觸發程序支援的其中一個引數繫結是可使用 F# 記錄來表示的 POCO。 例如︰

```fsharp
type Item = { Id: string }

let Run(blob: string, output: byref<Item>) =
    let item = { Id = "Some ID" }
    output <- item
```

F# Azure 函式會採用一個或多個引數。 在談論 Azure Functions 引數時，我們指的是*輸入*引數和*輸出*引數。 輸入引數的用途正如其名︰輸入到 F# Azure 函式。 *輸出*引數是可變動的資料或 `byref<>` 引數，可用來將資料從函式*往外*送回。

在上述範例中，`blob` 是輸入引數，而 `output` 是輸出引數。 請注意，我們使用 `byref<>` 做為 `output` (不必加上 `[<Out>]` 註解)。 使用 `byref<>` 類型可讓您的函式變更引數所指稱的記錄或物件。

使用 F# 記錄做為輸入類型時，記錄定義必須標上 `[<CLIMutable>]` ，以便讓 Azure Functions 架構先適當地設定欄位，再將記錄傳遞給您的函式。 實際上， `[<CLIMutable>]` 會產生記錄屬性的 setter。 例如︰

```fsharp
[<CLIMutable>]
type TestObject =
    { SenderName : string
      Greeting : string }

let Run(req: TestObject, log: TraceWriter) =
    { req with Greeting = sprintf "Hello, %s" req.SenderName }
```

F# 類別也可用於輸入和輸出引數。 針對類別，屬性通常需要 getter 和 setter。 例如︰

```fsharp
type Item() =
    member val Id = "" with get,set
    member val Text = "" with get,set

let Run(input: string, item: byref<Item>) =
    let result = Item(Id = input, Text = "Hello from F#!")
    item <- result
```

## <a name="logging"></a>記錄
若要將輸出記錄至 F# 的[串流記錄](../app-service/web-sites-enable-diagnostic-log.md)，您的函式應該採用 `TraceWriter` 類型的引數。 為求一致，我們建議將此引數命名為 `log`。 例如︰

```fsharp
let Run(blob: string, output: byref<string>, log: TraceWriter) =
    log.Verbose(sprintf "F# Azure Function processed a blob: %s" blob)
    output <- input
```

## <a name="async"></a>非同步處理
可以使用 `async` 工作流程，但結果需要傳回 `Task`。 這可以透過 `Async.StartAsTask`來完成，例如︰

```fsharp
let Run(req: HttpRequestMessage) =
    async {
        return new HttpResponseMessage(HttpStatusCode.OK)
    } |> Async.StartAsTask
```

## <a name="cancellation-token"></a>取消權杖
如果您的函式需要正常地處理關閉，您可以為其指定 [`CancellationToken`](https://msdn.microsoft.com/library/system.threading.cancellationtoken.aspx) 引數。 這可與 `async`結合，例如︰

```fsharp
let Run(req: HttpRequestMessage, token: CancellationToken)
    let f = async {
        do! Async.Sleep(10)
        return new HttpResponseMessage(HttpStatusCode.OK)
    }
    Async.StartAsTask(f, token)
```

## <a name="importing-namespaces"></a>匯入命名空間
命名空間可透過一般方式開啟︰

```fsharp
open System.Net
open System.Threading.Tasks

let Run(req: HttpRequestMessage, log: TraceWriter) =
    ...
```

下列命名空間會自動開啟︰

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`。

## <a name="referencing-external-assemblies"></a>參考外部組件
同樣地，在新增架構組件參考時也可以加上 `#r "AssemblyName"` 指示詞。

```fsharp
#r "System.Web.Http"

open System.Net
open System.Net.Http
open System.Threading.Tasks

let Run(req: HttpRequestMessage, log: TraceWriter) =
    ...
```

Azure Functions 裝載環境會自動加入下列組件︰

* `mscorlib`,
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`。

此外，下列組件為特殊案例，可以使用簡單名稱來參考 (例如 `#r "AssemblyName"`)：

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNEt.WebHooks.Common`。

如果您需要參考私用組件，您可以將組件檔案上傳至您函式的相對 `bin` 資料夾，然後使用檔案名稱來參考它 (例如 `#r "MyAssembly.dll"`)。 如需如何將檔案上傳至函數資料夾的資訊，請參閱以下的＜封裝管理＞小節。

## <a name="editor-prelude"></a>編輯器序言
支援 F# 編譯器服務的編輯器不會知道 Azure Functions 自動包含的命名空間和組件。 因此，最好在其中包含序言以協助編輯器找到您使用的組件，並明確開啟命名空間。 例如︰

```fsharp
#if !COMPILED
#I "../../bin/Binaries/WebJobs.Script.Host"
#r "Microsoft.Azure.WebJobs.Host.dll"
#endif

open System
open Microsoft.Azure.WebJobs.Host

let Run(blob: string, output: byref<string>, log: TraceWriter) =
    ...
```

當 Azure Functions 執行程式碼時，它會在定義了 `COMPILED` 的情況下處理來源，所以編輯器序言會遭到忽略。

<a name="package"></a>

## <a name="package-management"></a>封裝管理
若要在 F# 函式中使用 NuGet 套件，請將 `project.json` 檔案新增至函式應用程式檔案系統中的函式資料夾。 以下的 `project.json` 範例檔案會對 `Microsoft.ProjectOxford.Face` 1.1.0 版新增 NuGet 封裝參考︰

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

只有 .NET Framework 4.6 會受到支援，因此請確認您的 `project.json` 檔案會如這裡所示指定 `net46`。

當您上傳 `project.json` 檔案時，執行階段會取得封裝並自動加入對封裝組件的參考。 您不需要加入 `#r "AssemblyName"` 指示詞。 只需將必要的 `open` 陳述式加入您的 `.fsx` 檔案。

您可能會想在編輯器序言中自動放入參考組件，來改善您的編輯器與 F# 編譯器服務的互動。

### <a name="how-to-add-a-projectjson-file-to-your-azure-function"></a>如何將 `project.json` 檔案新增至 Azure 函式
1. 首先，在 Azure 入口網站中開啟您的函式，以確定函式應用程式正在執行中。 這也可供存取將要顯示封裝安裝輸出的串流記錄檔。
2. 若要上傳 `project.json` 檔案，請使用 [如何更新函式應用程式檔案](functions-reference.md#fileupdate)中所述的其中一個方法。 如果您使用 [Azure Functions 的持續部署](functions-continuous-deployment.md)，您可以將 `project.json` 檔案新增預備分支，以便在試驗過後再將它新增至部署分支。
3. 加入 `project.json` 檔案之後，您會在函式的串流記錄中看到類似下列範例的輸出：

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
若要取得環境變數或應用程式設定值，請使用 `System.Environment.GetEnvironmentVariable`，例如：

```fsharp
open System.Environment

let Run(timer: TimerInfo, log: TraceWriter) =
    log.Info("Storage = " + GetEnvironmentVariable("AzureWebJobsStorage"))
    log.Info("Site = " + GetEnvironmentVariable("WEBSITE_SITE_NAME"))
```

## <a name="reusing-fsx-code"></a>重複使用 .fsx 程式碼
您可以使用 `#load` 指示詞以使用其他 `.fsx` 檔案中的程式碼。 例如︰

`run.fsx`

```fsharp
#load "logger.fsx"

let Run(timer: TimerInfo, log: TraceWriter) =
    mylog log (sprintf "Timer: %s" DateTime.Now.ToString())
```

`logger.fsx`

```fsharp
let mylog(log: TraceWriter, text: string) =
    log.Verbose(text);
```

提供給 `#load` 指示詞的路徑會相對於 `.fsx` 檔案的位置。

* `#load "logger.fsx"` 會載入位於函式資料夾中的檔案。
* `#load "package\logger.fsx"` 會載入位於函式資料夾的 `package`資料夾中的檔案。
* `#load "..\shared\mylogger.fsx"` 會載入位於與函式資料夾相同層級的 `shared`資料夾中的檔案 (也就是在 `wwwroot` 的正下方)。

`#load` 指示詞只能搭配 `.fsx` (F# 指令碼) 檔案運作，而不能與 `.fs` 檔案搭配。

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱下列資源：

* [F# 指南](/dotnet/articles/fsharp/index)
* [Azure Functions 的最佳作法](functions-best-practices.md)
* [Azure Functions 開發人員參考](functions-reference.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
* [Azure Functions 測試](functions-test-a-function.md)
* [Azure Functions 調整](functions-scale.md)

