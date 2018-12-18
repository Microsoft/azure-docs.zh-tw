---
title: Azure Functions 中的觸發程序和繫結
description: 了解如何在 Azure Functions 中使用觸發程序和繫結，將您的程式碼執行連接到線上事件和雲端服務。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 05/24/2018
ms.author: glenga
ms.openlocfilehash: cc965073863375d76efb969ad66cf5750c9755bb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969422"
---
# <a name="azure-functions-triggers-and-bindings-concepts"></a>Azure Functions 觸發程序和繫結概念

本文是 Azure Functions 中觸發程序和繫結的概念性概觀。 這裡描述所有繫結及所有支援語言的通用功能。

## <a name="overview"></a>概觀

「觸發程序」會定義叫用函數的方式。 一個函數只能恰有一個觸發程序。 觸發程序具有相關聯的資料，它通常是觸發函數的承載。

輸入和輸出「繫結」提供從您的程式碼內連線到資料的宣告式方法。 繫結是選擇性的，而且一個函數可以有多個輸入和輸出繫結。 

觸發程序和繫結可讓您避免將正在使用的服務詳細資料硬式編碼。 您的函式會接收函式參數中的資料 (例如佇列訊息的內容)。 您可以使用函式的傳回值來傳送資料 (例如用以建立佇列訊息)。 在 C# 和 C# 指令碼中，傳送資料的方式可以是 `out` 參數和[收集器物件](functions-reference-csharp.md#writing-multiple-output-values)。

當您使用 Azure 入口網站來開發函式時，觸發程序和繫結是在 *function.json* 檔案中進行設定。 入口網站提供此設定的 UI，但您可以變更為**進階編輯器**來直接編輯檔案。

當您使用 Visual Studio 建立類別庫來開發函式時，觸發程序和繫結是透過以屬性裝飾方法和參數來進行設定。

## <a name="example-trigger-and-binding"></a>觸發程序和繫結範例

假設您想要每當新訊息出現在 Azure 佇列儲存體時，就在 Azure 表格儲存體寫入新的資料列。 此案例可以使用 Azure 佇列儲存體觸發程序和 Azure 表格儲存體輸出繫結來實作。 

以下是此案例的 *function.json* 檔案。 

```json
{
  "bindings": [
    {
      "name": "order",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "myqueue-items",
      "connection": "MY_STORAGE_ACCT_APP_SETTING"
    },
    {
      "name": "$return",
      "type": "table",
      "direction": "out",
      "tableName": "outTable",
      "connection": "MY_TABLE_STORAGE_ACCT_APP_SETTING"
    }
  ]
}
```

`bindings` 陣列中的第一個元素是佇列儲存體觸發程序。 `type` 和 `direction` 屬性可識別觸發程序。 `name` 屬性可識別會接收佇列訊息內容的函式參數。 要監視的佇列名稱是在 `queueName` 中，而連接字串則是在 `connection` 所識別的應用程式設定中。

`bindings` 陣列中的第二個元素是 Azure 表格儲存體輸出繫結。 `type` 和 `direction` 屬性可識別繫結。 `name` 屬性指定函式如何提供新的資料表資料列，在本例中是透過函式傳回值。 資料表的名稱是在 `tableName` 中，而連接字串則是在 `connection` 所識別的應用程式設定中。

若要在 Azure 入口網站中檢視及編輯 *function.json* 的內容，請按一下函數的 [整合] 索引標籤上的 [進階編輯器] 選項。

> [!NOTE]
> `connection` 的值是包含連接字串的應用程式設定名稱，而不是連接字串本身。 繫結使用儲存在應用程式設定中的連接字串，以強制遵循 *function.json* 不包含服務祕密的最佳做法。

以下是可搭配此觸發程序和繫結使用的 C# 指令碼。 請注意，提供佇列訊息內容的參數名稱為 `order`；由於 *function.json* 中的 `name` 屬性值為 `order`，因此必須使用此名稱 

```cs
#r "Newtonsoft.Json"

using Newtonsoft.Json.Linq;

// From an incoming queue message that is a JSON object, add fields and write to Table storage
// The method return value creates a new row in Table Storage
public static Person Run(JObject order, TraceWriter log)
{
    return new Person() { 
            PartitionKey = "Orders", 
            RowKey = Guid.NewGuid().ToString(),  
            Name = order["Name"].ToString(),
            MobileNumber = order["MobileNumber"].ToString() };  
}
 
public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
    public string MobileNumber { get; set; }
}
```

相同的 function.json 檔案可搭配 JavaScript 函式使用：

```javascript
// From an incoming queue message that is a JSON object, add fields and write to Table Storage
// The second parameter to context.done is used as the value for the new row
module.exports = function (context, order) {
    order.PartitionKey = "Orders";
    order.RowKey = generateRandomId(); 

    context.done(null, order);
};

function generateRandomId() {
    return Math.random().toString(36).substring(2, 15) +
        Math.random().toString(36).substring(2, 15);
}
```

在類別庫中，會由屬性 (而非 function.json 檔案) 提供相同的觸發程序和繫結資訊 &mdash;佇列和資料表名稱、儲存體帳戶、輸入和輸出的函式參數&mdash;。 以下是範例：

```csharp
 public static class QueueTriggerTableOutput
 {
     [FunctionName("QueueTriggerTableOutput")]
     [return: Table("outTable", Connection = "MY_TABLE_STORAGE_ACCT_APP_SETTING")]
     public static Person Run(
         [QueueTrigger("myqueue-items", Connection = "MY_STORAGE_ACCT_APP_SETTING")]JObject order, 
         TraceWriter log)
     {
         return new Person() {
                 PartitionKey = "Orders",
                 RowKey = Guid.NewGuid().ToString(),
                 Name = order["Name"].ToString(),
                 MobileNumber = order["MobileNumber"].ToString() };
     }
 }

 public class Person
 {
     public string PartitionKey { get; set; }
     public string RowKey { get; set; }
     public string Name { get; set; }
     public string MobileNumber { get; set; }
 }
```

## <a name="supported-bindings"></a>支援的繫結

[!INCLUDE [Full bindings table](../../includes/functions-bindings.md)]

如需哪些繫結為預覽狀態或已核准可用於實際執行環境的資訊，請參閱[支援的語言](supported-languages.md)。

## <a name="register-binding-extensions"></a>註冊繫結延伸模組

在某些開發環境中，您必須明確「註冊」您想要使用的繫結。 NuGet 套件中會提供繫結擴充功能，若要註冊擴充功能，則需安裝套件。 下表指出何時要註冊繫結擴充功能，以及註冊方式。

|開發環境 |註冊<br/> 在 Functions 1.x 中  |註冊<br/> 在 Functions 2.x 中  |
|---------|---------|---------|
|Azure 入口網站|自動|[自動 (含提示)](#azure-portal-development)|
|在本機使用 Azure Functions 核心工具|自動|[使用核心工具 CLI 命令](#local-development-azure-functions-core-tools)|
|使用 Visual Studio 2017 的 C# 類別庫|[使用 NuGet 工具](#c-class-library-with-visual-studio-2017)|[使用 NuGet 工具](#c-class-library-with-visual-studio-2017)|
|使用 Visual Studio Code 的 C# 類別庫|N/A|[使用 .NET Core CLI](#c-class-library-with-visual-studio-code)|

下列繫結類型是例外，不需要明確註冊，因為這些類型會在所有版本和環境中自動註冊：HTTP 與計時器。

### <a name="azure-portal-development"></a>Azure 入口網站開發

本節僅適用於 Functions 2.x。 繫結延伸模組不需要先在 Functions 1.x 中明確註冊。

當您建立函式或新增繫結時，系統會在觸發程序或繫結的擴充功能需要註冊時提示您。 請按一下 [安裝] 來註冊擴充功能，以回應提示。 安裝在取用方案上可能需要多達 10 分鐘。

針對指定的函式應用程式，您只須安裝每個延伸模組一次。 

### <a name="local-development-azure-functions-core-tools"></a>本機開發 Azure Functions Core Tools

本節僅適用於 Functions 2.x。 繫結延伸模組不需要先在 Functions 1.x 中明確註冊。

[!INCLUDE [functions-core-tools-install-extension](../../includes/functions-core-tools-install-extension.md)]

<a name="local-csharp"></a>
### <a name="c-class-library-with-visual-studio-2017"></a>包含 Visual Studio 2017 的 C# 類別庫

在 **Visual Studio 2017** 中，您可以使用 [Install-Package](https://docs.microsoft.com/nuget/tools/ps-ref-install-package) 命令，從「套件管理員主控台」中安裝套件，如下列範例所示：

```powershell
Install-Package Microsoft.Azure.WebJobs.ServiceBus --Version <target_version>
```

在指定繫結的參考文章中，會提供要用於該繫結的套件名稱。 如需範例，請參閱[服務匯流排繫結參考文章的套件一節](functions-bindings-service-bus.md#packages---functions-1x)。

請以特定版本的套件 (例如 `3.0.0-beta5`) 取代範例中的 `<target_version>`。 有效的版本會列在 [NuGet.org](https://nuget.org) 的個別套件頁面上。對應至 Functions 執行階段 1.x 或 2.x 的主要版本，會在繫結的參考文章中指定。

### <a name="c-class-library-with-visual-studio-code"></a>包含 Visual Studio Code 的 C# 類別庫

在 **Visual Studio Code** 中，您可以在 .NET Core CLI 中使用 [dotnet add package](https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package) 命令，從命令提示字元安裝套件，如下列範例所示：

```terminal
dotnet add package Microsoft.Azure.WebJobs.ServiceBus --version <target_version>
```

.NET Core CLI 只能用於 Azure Functions 2.x 開發。

在指定繫結的參考文章中，會提供要用於該繫結的套件名稱。 如需範例，請參閱[服務匯流排繫結參考文章的套件一節](functions-bindings-service-bus.md#packages---functions-1x)。

請以特定版本的套件 (例如 `3.0.0-beta5`) 取代範例中的 `<target_version>`。 有效的版本會列在 [NuGet.org](https://nuget.org) 的個別套件頁面上。對應至 Functions 執行階段 1.x 或 2.x 的主要版本，會在繫結的參考文章中指定。

## <a name="binding-direction"></a>繫結方向

所有觸發程序和繫結在 function.json 檔案中都具有 `direction` 屬性：

- 對於觸發程序，方向一律為 `in`
- 輸入和輸出繫結使用 `in` 和 `out`
- 某些繫結支援特殊方向 `inout`。 如果您使用 `inout`，則 [整合] 索引標籤中只有 [進階編輯器] 可供使用。

當您使用[類別庫中的屬性](functions-dotnet-class-library.md)來設定觸發程序和繫結時，請在屬性建構函式中提供方向，或從參數類型推斷方向。

## <a name="using-the-function-return-value"></a>使用函式傳回值

在具有傳回值的語言中，您可以將「輸出繫結」繫結至傳回值：

* 在 C# 類別庫中，將輸出繫結屬性套用至方法傳回值。
* 在其他語言中，將 function.json 中的 `name` 屬性設定為 `$return`。

如果有多個輸出繫結，請只對其中一個使用傳回值。

在 C# 和 C# 指令碼中，傳送資料到輸出繫結的方式可以是 `out` 參數和[收集器物件](functions-reference-csharp.md#writing-multiple-output-values)。

請參閱示範傳回值用法的特定語言範例：

* [C#](#c-example)
* [C# 指令碼 (.csx)](#c-script-example)
* [F#](#f-example)
* [JavaScript](#javascript-example)

### <a name="c-example"></a>C# 範例

以下是使用輸出繫結傳回值的 C# 程式碼，接著則是非同步範例：

```cs
[FunctionName("QueueTrigger")]
[return: Blob("output-container/{id}")]
public static string Run([QueueTrigger("inputqueue")]WorkItem input, TraceWriter log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.Info($"C# script processed queue message. Item={json}");
    return json;
}
```

```cs
[FunctionName("QueueTrigger")]
[return: Blob("output-container/{id}")]
public static Task<string> Run([QueueTrigger("inputqueue")]WorkItem input, TraceWriter log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.Info($"C# script processed queue message. Item={json}");
    return Task.FromResult(json);
}
```

### <a name="c-script-example"></a>C# 指令碼範例

以下是 function.json 檔案中的輸出繫結：

```json
{
    "name": "$return",
    "type": "blob",
    "direction": "out",
    "path": "output-container/{id}"
}
```

以下是 C# 指令程式碼，接著則是非同步範例：

```cs
public static string Run(WorkItem input, TraceWriter log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.Info($"C# script processed queue message. Item={json}");
    return json;
}
```

```cs
public static Task<string> Run(WorkItem input, TraceWriter log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.Info($"C# script processed queue message. Item={json}");
    return Task.FromResult(json);
}
```

### <a name="f-example"></a>F# 範例

以下是 function.json 檔案中的輸出繫結：

```json
{
    "name": "$return",
    "type": "blob",
    "direction": "out",
    "path": "output-container/{id}"
}
```

以下是 F# 程式碼：

```fsharp
let Run(input: WorkItem, log: TraceWriter) =
    let json = String.Format("{{ \"id\": \"{0}\" }}", input.Id)   
    log.Info(sprintf "F# script processed queue message '%s'" json)
    json
```

### <a name="javascript-example"></a>JavaScript 範例

以下是 function.json 檔案中的輸出繫結：

```json
{
    "name": "$return",
    "type": "blob",
    "direction": "out",
    "path": "output-container/{id}"
}
```

在 JavaScript 中，傳回值是放在 `context.done` 的第二個參數中：

```javascript
module.exports = function (context, input) {
    var json = JSON.stringify(input);
    context.log('Node.js script processed queue message', json);
    context.done(null, json);
}
```

## <a name="binding-datatype-property"></a>繫結 dataType 屬性

在 .NET 中，使用參數類型可定義輸入資料的資料類型。 例如，使用 `string` 繫結至要以二進位格式和自訂類型讀取以還原序列化為 POCO 物件之佇列觸發程序和位元組陣列的文字。

對於 JavaScript 等具有動態類型的語言，則會使用 *function.json* 檔案中的 `dataType` 屬性。 例如，若要讀取二進位格式的 HTTP 要求內容，請將 `dataType` 設定為 `binary`：

```json
{
    "type": "httpTrigger",
    "name": "req",
    "direction": "in",
    "dataType": "binary"
}
```

`dataType` 也另具有 `stream` 和 `string` 兩種選項。

## <a name="binding-expressions-and-patterns"></a>繫結運算式和模式

觸發程序和繫結其中一個最強大的功能就是「繫結運算式」。 在 function.json 檔案以及函式參數與程式碼中，您可以使用多個運算式，這些運算式會將各種來源解析為相對應的多個值。

大多數運算式會藉由以大括號裹住來供您識別。 例如，在佇列觸發程序函式中，`{queueTrigger}` 會解析為佇列訊息文字。 如果 Blob 輸出繫結的 `path` 屬性是 `container/{queueTrigger}`，且函式是由佇列訊息 `HelloWorld` 所觸發，則系統會建立名為 `HelloWorld` 的 Blob。

繫結運算式的類型

* [應用程式設定](#binding-expressions---app-settings)
* [觸發程序檔案名稱](#binding-expressions---trigger-file-name)
* [觸發程序中繼資料](#binding-expressions---trigger-metadata)
* [JSON 承載](#binding-expressions---json-payloads)
* [新 GUID](#binding-expressions---create-guids)
* [目前的日期和時間](#binding-expressions---current-time)

### <a name="binding-expressions---app-settings"></a>繫結運算式 - 應用程式設定

為了遵循最佳做法，祕密和連接字串應使用應用程式設定來管理，而不是使用組態檔。 這會限制對這些祕密的存取，並保護儲存在公用原始檔控制存放庫的檔案，例如 function.json。

當您想要根據環境來變更設定時，應用程式設定也很有用。 例如，在測試環境中，您可能會想要監視不同佇列或 Blob 儲存體容器。

應用程式設定繫結運算式可藉由不同於其他繫結運算式的方式來識別：它們會以百分比符號裹住，而不是以大括號裹住。 例如，如果 Blob 輸出繫結路徑是 `%Environment%/newblob.txt` 而 `Environment` 應用程式設定值是 `Development`，則系統會在 `Development` 容器中建立 Blob。

在本機執行函式時，應用程式設定值來自 local.settings.json 檔案。

請注意，觸發程序和繫結的 `connection` 屬性是特殊案例，且會自動將值解析為應用程式設定，不含百分比符號。 

下列範例是使用應用程式設定 `%input-queue-name%` 來定義要觸發之佇列的 Azure 佇列儲存體觸發程序。

```json
{
  "bindings": [
    {
      "name": "order",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "%input-queue-name%",
      "connection": "MY_STORAGE_ACCT_APP_SETTING"
    }
  ]
}
```

您可以在類別庫中使用相同的方法：

```csharp
[FunctionName("QueueTrigger")]
public static void Run(
    [QueueTrigger("%input-queue-name%")]string myQueueItem, 
    TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
```

### <a name="binding-expressions---trigger-file-name"></a>繫結運算式 - 觸發程序檔案名稱

Blob 觸發程序的 `path` 可以是可讓您參考其他繫結和函式程式碼中之觸發 Blob 名稱的模式。 此模式也可以包含篩選條件，該條件會指定哪個 Blob 可以觸發函式引動過程。

例如，在下列 Blob 觸發程序繫結中，`path` 模式是 `sample-images/{filename}`，它會建立名為 `filename` 的繫結運算式：

```json
{
  "bindings": [
    {
      "name": "image",
      "type": "blobTrigger",
      "path": "sample-images/{filename}",
      "direction": "in",
      "connection": "MyStorageConnection"
    },
    ...
```

然後可以在輸出繫結中使用運算式 `filename` 以指定要建立之 Blob 的名稱：

```json
    ...
    {
      "name": "imageSmall",
      "type": "blob",
      "path": "sample-images-sm/{filename}",
      "direction": "out",
      "connection": "MyStorageConnection"
    }
  ],
}
```

函式程式碼可以使用 `filename` 作為參數名稱來存取此相同值：

```csharp
// C# example of binding to {filename}
public static void Run(Stream image, string filename, Stream imageSmall, TraceWriter log)  
{
    log.Info($"Blob trigger processing: {filename}");
    // ...
} 
```

<!--TODO: add JavaScript example -->
<!-- Blocked by bug https://github.com/Azure/Azure-Functions/issues/248 -->

類別庫中的所有屬性都可以使用繫結運算式和模式。 在下列範例中，屬性建構函式參數是與上述 function.json 範例相同的 `path` 值： 

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{filename}")] Stream image,
    [Blob("sample-images-sm/{filename}", FileAccess.Write)] Stream imageSmall,
    string filename,
    TraceWriter log)
{
    log.Info($"Blob trigger processing: {filename}");
    // ...
}

```

您也可以為檔案名稱的各個部分 (例如副檔名) 建立運算式。 如需如何在 Blob 路徑字串中使用運算式和模式的詳細資訊，請參閱[儲存體 Blob 繫結參考](functions-bindings-storage-blob.md)。
 
### <a name="binding-expressions---trigger-metadata"></a>繫結運算式 - 觸發程序中繼資料

除了觸發程序提供的資料承載 (例如觸發函式的佇列訊息內容) 之外，許多觸發程序都提供額外的中繼資料值。 這些值可以在 C# 和 F# 中作為輸入參數使用，或在 JavaScript 中做為 `context.bindings` 物件上的屬性使用。 

例如，Azure 佇列儲存體觸發程序支援下列屬性：

* QueueTrigger - 如果是有效字串，便觸發訊息內容
* DequeueCount
* ExpirationTime
* id
* InsertionTime
* NextVisibleTime
* PopReceipt

您可以在 *function.json* 檔案屬性中存取這些中繼資料值。 例如，假設您使用佇列觸發程序，且佇列訊息包含您想要讀取的 Blob 名稱。 在 *function.json* 檔案中，您可以使用 Blob `path` 屬性中的 `queueTrigger` 中繼資料屬性，如下列範例所示：

```json
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "direction": "in",
      "connection": "MyStorageConnection"
    }
  ]
```

在對應的參考文章中，會描述每個觸發程序之中繼資料屬性的詳細資料。 如需範例，請參閱[佇列觸發程序中繼資料](functions-bindings-storage-queue.md#trigger---message-metadata)。 您也可以在入口網站的 [整合] 索引標籤中，繫結設定區域之下的 [文件] 區段取得文件。  

### <a name="binding-expressions---json-payloads"></a>繫結運算式 - JSON 承載

當觸發程序承載為 JSON 時，您可以在相同函式與函式程式碼內之其他繫結的設定中參考其屬性。

下列範例針對接收 JSON 中 Blob 名稱的 Webhook 函式顯示 function.json 檔案：`{"BlobName":"HelloWorld.txt"}`。 Blob 輸入繫結會讀取 Blob，HTTP 輸出繫結則會在 HTTP 回應中傳回 Blob 內容。 請注意，Blob 輸入繫結會藉由直接參考 `BlobName` 屬性 (`"path": "strings/{BlobName}"`) 來取得 Blob 名稱

```json
{
  "bindings": [
    {
      "name": "info",
      "type": "httpTrigger",
      "direction": "in",
      "webHookType": "genericJson"
    },
    {
      "name": "blobContents",
      "type": "blob",
      "direction": "in",
      "path": "strings/{BlobName}",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ]
}
```

為了能夠在 C# 和 F# 中運作，您需要一個類別來定義要還原序列化的欄位，如下列範例所示：

```csharp
using System.Net;

public class BlobInfo
{
    public string BlobName { get; set; }
}
  
public static HttpResponseMessage Run(HttpRequestMessage req, BlobInfo info, string blobContents, TraceWriter log)
{
    if (blobContents == null) {
        return req.CreateResponse(HttpStatusCode.NotFound);
    } 

    log.Info($"Processing: {info.BlobName}");

    return req.CreateResponse(HttpStatusCode.OK, new {
        data = $"{blobContents}"
    });
}
```

JavaScript 會自動執行 JSON 還原序列化。

```javascript
module.exports = function (context, info) {
    if ('BlobName' in info) {
        context.res = {
            body: { 'data': context.bindings.blobContents }
        }
    }
    else {
        context.res = {
            status: 404
        };
    }
    context.done();
}
```

#### <a name="dot-notation"></a>點標記法

如果 JSON 承載中的某些屬性是具有屬性的物件，您可以使用點標記法來直接參考這些屬性。 例如，假設您的 JSON 看起來如下：

```json
{"BlobName": {
  "FileName":"HelloWorld",
  "Extension":"txt"
  }
}
```

您可直接以 `BlobName.FileName` 的形式參考 `FileName`。 在使用此 JSON 格式時，以下是上述範例中 `path` 屬性的樣貌：

```json
"path": "strings/{BlobName.FileName}.{BlobName.Extension}",
```

在 C# 中，您需要兩個類別：

```csharp
public class BlobInfo
{
    public BlobName BlobName { get; set; }
}
public class BlobName
{
    public string FileName { get; set; }
    public string Extension { get; set; }
}
```

### <a name="binding-expressions---create-guids"></a>繫結運算式 - 建立 GUID

`{rand-guid}` 繫結運算式可建立 GUID。 `function.json` 檔案中的下列 Blob 路徑會建立具有如 50710cb5-84b9-4d87-9d83-a03d6976a682.txt 之名稱的 Blob。

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{rand-guid}"
}
```

### <a name="binding-expressions---current-time"></a>繫結運算式 - 目前時間

繫結運算式 `DateTime` 會解析成 `DateTime.UtcNow`。 `function.json` 檔案中的下列 Blob 路徑會建立具有如 2018-02-16T17-59-55Z.txt 之名稱的 Blob。

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{DateTime}"
}
```

## <a name="binding-at-runtime"></a>執行階段的繫結

在 C# 和其他 .NET 語言中，您可以使用相對於 *function.json* 和屬性中宣告式繫結的命令式繫結模式。 當繫結參數需要在執行階段而不是設計階段中計算時，命令式繫結非常有用。 若要深入了解，請參閱 [C# 開發人員參考](functions-dotnet-class-library.md#binding-at-runtime)或 [C# 指令碼開發人員參考](functions-reference-csharp.md#binding-at-runtime)。

## <a name="functionjson-file-schema"></a>function.json 檔案結構描述

function.json 檔案結構描述可以在 [http://json.schemastore.org/function](http://json.schemastore.org/function) 取得。

## <a name="handling-binding-errors"></a>處理繫結錯誤

[!INCLUDE [bindings errors intro](../../includes/functions-bindings-errors-intro.md)]

如需 Functions 所支援各種服務的所有相關錯誤主題連結，請參閱[Azure Functions 錯誤處理](functions-bindings-error-pages.md) \(英文\) 概觀主題的[繫結錯誤碼](functions-bindings-error-pages.md#binding-error-codes)一節。  

## <a name="next-steps"></a>後續步驟

如需特定繫結的詳細資訊，請參閱下列文章：

- [HTTP 和 Webhook](functions-bindings-http-webhook.md)
- [計時器](functions-bindings-timer.md)
- [佇列儲存體](functions-bindings-storage-queue.md)
- [Blob 儲存體](functions-bindings-storage-blob.md)
- [資料表儲存體](functions-bindings-storage-table.md)
- [事件中樞](functions-bindings-event-hubs.md)
- [服務匯流排](functions-bindings-service-bus.md)
- [Azure Cosmos DB](functions-bindings-cosmosdb.md)
- [Microsoft Graph](functions-bindings-microsoft-graph.md)
- [SendGrid](functions-bindings-sendgrid.md)
- [Twilio](functions-bindings-twilio.md)
- [通知中樞](functions-bindings-notification-hubs.md)
- [行動應用程式](functions-bindings-mobile-apps.md)
- [外部檔案](functions-bindings-external-file.md)
