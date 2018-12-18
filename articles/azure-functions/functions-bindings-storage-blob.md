---
title: Azure Functions 的 Azure Blob 儲存體繫結
description: 瞭解如何在 Azure Functions 中使用「Azure Blob 儲存體」觸發程序和繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 09/03/2018
ms.author: glenga
ms.openlocfilehash: 40ed6105dca5ea14c64fb2b103c5623cd56333af
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393361"
---
# <a name="azure-blob-storage-bindings-for-azure-functions"></a>Azure Functions 的 Azure Blob 儲存體繫結

此文章說明如何在 Azure Functions 中使用 Azure Blob 儲存體繫結。 Azure Functions 支援適用於 Blob 的觸發程序、輸入和輸出繫結。 此文章包含每個繫結的區段：

* [Blob 觸發程序](#trigger)
* [Blob 輸入繫結](#input)
* [Blob 輸出繫結](#output)

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!NOTE]
> 針對 Blob 儲存體帳戶，請使用「事件方格」觸發程序而不要使用 Blob 儲存體觸發程序，以提升延展性，或避免發生冷啟動延遲。 如需詳細資訊，請參閱[觸發程序](#trigger)一節。 

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs](http://www.nuget.org/packages/Microsoft.Azure.WebJobs) NuGet 套件 2.x 版中提供 Blob 儲存體繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Blob) GitHub 存放庫中。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) NuGet 套件 3.x 版中提供 Blob 儲存體繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="trigger"></a>觸發程序

偵測到新的或已更新的 Blob 時，Blob 儲存體觸發程序會啟動函式。 Blob 內容會當成函式輸入提供。

[Event Grid 觸發程序](functions-bindings-event-grid.md)內建對 [Blob 事件](../storage/blobs/storage-blob-event-overview.md)的支援，也能用於在偵測到全新或更新的 Blob 時啟動函式。 如需範例，請參閱[使用 Event Grid 進行映像大小調整](../event-grid/resize-images-on-storage-blob-upload-event.md)教學課程。

請使用 Event Grid 來因應以下情節的需求，避免使用 Blob 儲存體觸發程序：

* Blob 儲存體帳戶
* 高延展性
* 縮短冷啟動延遲

### <a name="blob-storage-accounts"></a>Blob 儲存體帳戶

支援使用 [Blob 儲存體帳戶](../storage/common/storage-account-overview.md#types-of-storage-accounts)進行 Blob 輸入和輸出繫結，但針對 Blob 觸發程序則不支援。 Blob 儲存體觸發程序需要一般用途的儲存體帳戶。

### <a name="high-scale"></a>高延展性

可將高延展性寬鬆地定義為可容納超過 100,000 個 Blob 的容器，或每秒 Blob 更新超過 100 次的儲存體帳戶。

### <a name="cold-start-delay"></a>冷啟動延遲

如果函式應用程式採用取用方案，當其進入閒置狀態，處理新 Blob 時最多會有 10 分鐘的延遲。 若要避免這樣的冷啟動延遲，您可以切換為 App Service 方案並啟用 Always On，或使用不同的觸發程序類型。

### <a name="queue-storage-trigger"></a>佇列儲存體觸發程序

除了 Event Grid 之外，處理 Blob 的另一個替代方式是佇列儲存體觸發程序，不過其並未內建 Blob 事件支援。 在建立或更新 Blob 時，必須建立佇列訊息。 有些範例預設您已符合前述需求，可參閱[此文章後續的 Blob 輸入繫結範例](#input---example)。

## <a name="trigger---example"></a>觸發程序 - 範例

請參閱特定語言的範例：

* [C#](#trigger---c-example)
* [C# 指令碼 (.csx)](#trigger---c-script-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---javascript-example)

### <a name="trigger---c-example"></a>觸發程序 - C# 範例

下列範例示範在 `samples-workitems` 容器中新增或更新 Blob 時，寫入記錄的 [C# 函式](functions-dotnet-class-library.md)。

```csharp
[FunctionName("BlobTriggerCSharp")]        
public static void Run([BlobTrigger("samples-workitems/{name}")] Stream myBlob, string name, TraceWriter log)
{
    log.Info($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

Blob 觸發程序路徑 `samples-workitems/{name}` 中的字串 `{name}` 會建立[繫結運算式](functions-triggers-bindings.md#binding-expressions-and-patterns)，您可以在函式程式碼中用來存取觸發 Blob 的檔案名稱。 如需詳細資訊，請參閱此文章稍後的 [Blob 名稱模式](#trigger---blob-name-patterns)。

如需有關 `BlobTrigger` 屬性的詳細資訊，請參閱[觸發程序 - 屬性](#trigger---attributes)。

### <a name="trigger---c-script-example"></a>觸發程序 - C# 指令碼範例

下列範例所示範的是使用繫結之 function.json 檔案和 [C# 指令碼 (.csx)](functions-reference-csharp.md) 中的 Blob 觸發程序繫結。 在 `samples-workitems` 容器中新增或更新 Blob 時，函數會寫入記錄。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

Blob 觸發程序路徑 `samples-workitems/{name}` 中的字串 `{name}` 會建立[繫結運算式](functions-triggers-bindings.md#binding-expressions-and-patterns)，您可以在函式程式碼中用來存取觸發 Blob 的檔案名稱。 如需詳細資訊，請參閱此文章稍後的 [Blob 名稱模式](#trigger---blob-name-patterns)。

如需 *function.json* 檔案屬性的詳細資訊，請參閱[設定](#trigger---configuration)一節中這些屬性的說明。

以下是繫結至 `Stream` 的 C# 指令碼：

```cs
public static void Run(Stream myBlob, TraceWriter log)
{
   log.Info($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

以下是繫結至 `CloudBlockBlob` 的 C# 指令碼：

```cs
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.WindowsAzure.Storage.Blob;

public static void Run(CloudBlockBlob myBlob, string name, TraceWriter log)
{
    log.Info($"C# Blob trigger function Processed blob\n Name:{name}\nURI:{myBlob.StorageUri}");
}
```

### <a name="trigger---javascript-example"></a>觸發程序 - JavaScript 範例

下列範例所示範的是使用繫結之 *function.json* 檔案和 [JavaScript 程式碼](functions-reference-node.md)中的 Blob 觸發程序繫結。 在 `samples-workitems` 容器中新增或更新 Blob 時，函數會寫入記錄。

以下是 *function.json* 檔案：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

Blob 觸發程序路徑 `samples-workitems/{name}` 中的字串 `{name}` 會建立[繫結運算式](functions-triggers-bindings.md#binding-expressions-and-patterns)，您可以在函式程式碼中用來存取觸發 Blob 的檔案名稱。 如需詳細資訊，請參閱此文章稍後的 [Blob 名稱模式](#trigger---blob-name-patterns)。

如需 *function.json* 檔案屬性的詳細資訊，請參閱[設定](#trigger---configuration)一節中這些屬性的說明。

以下是 JavaScript 程式碼：

```javascript
module.exports = function(context) {
    context.log('Node.js Blob trigger function processed', context.bindings.myBlob);
    context.done();
};
```

### <a name="trigger---java-example"></a>觸發程序 - Java 範例

下列範例示範的是使用繫結的 *function.json* 檔案和 [Java 程式碼](functions-reference-java.md)中的 Blob 觸發程序繫結。 在 `myblob` 容器中新增或更新 Blob 時，函數會寫入記錄。

以下是 *function.json* 檔案：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "file",
            "type": "blobTrigger",
            "direction": "in",
            "path": "myblob/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

以下是 Java 程式碼：

```java
 @FunctionName("blobprocessor")
 public void run(
    @BlobTrigger(name = "file",
                  dataType = "binary",
                  path = "myblob/filepath",
                  connection = "myconnvarname") byte[] content,
    @BindingName("name") String filename,
     final ExecutionContext context
 ) {
     context.getLogger().info("Name: " + name + " Size: " + content.length + " bytes");
 }

```


## <a name="trigger---attributes"></a>觸發程序 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用下列屬性以設定 Blob 觸發程序：

* [BlobTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/BlobTriggerAttribute.cs)

  該屬性的建構函式採用路徑字串，指示要監看的容器以及可選的 [Blob 名稱模式](#trigger---blob-name-patterns)。 以下是範例：

  ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}")] Stream image, 
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
  ```

  您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

   ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}", Connection = "StorageConnectionAppSetting")] Stream image, 
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
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
      [FunctionName("BlobTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ....
  }
  ```

要使用的儲存體帳戶按以下順序決定：

* `BlobTrigger` 屬性的 `Connection` 內容。
* `StorageAccount` 屬性套用至與 `BlobTrigger` 屬性相同的參數。
* `StorageAccount` 屬性套用至該函式。
* `StorageAccount` 屬性套用至該類別。
* 函數應用程式 (「AzureWebJobsStorage」應用程式設定) 的預設儲存體帳戶。

## <a name="trigger---configuration"></a>觸發程式 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `BlobTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 `blobTrigger`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction** | n/a | 必須設為 `in`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 例外狀況在[使用方式](#trigger---usage)一節中說明。 |
|**name** | n/a | 表示函式程式碼中 Blob 的變數名稱。 | 
|**路徑** | **BlobPath** |要監視的容器。  可能是 [Blob 名稱模式](#trigger-blob-name-patterns)。 | 
|**連接** | **連接** | 應用程式設定的名稱包含要用於此繫結的儲存體連接字串。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為「MyStorage」，則函式執行階段會尋找名稱為「AzureWebJobsMyStorage」的應用程式設定。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。<br><br>連接字串必須是用於一般用途的儲存體帳戶，而不是[Blob 儲存體帳戶](../storage/common/storage-account-overview.md#types-of-storage-accounts)。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>觸發程序 - 使用方式

在 C# 和 C# 指令碼中，您可以針對觸發 Blob 使用下列參數類型：

* `Stream`
* `TextReader`
* `string`
* `Byte[]`
* 可序列化為 JSON 的 POCO
* `ICloudBlob`<sup>1</sup>
* `CloudBlockBlob`<sup>1</sup>
* `CloudPageBlob`<sup>1</sup>
* `CloudAppendBlob`<sup>1</sup>

<sup>1</sup> 在 *function.json* 或 `FileAccess.ReadWrite` C# 類別庫中需要 "inout" 繫結 `direction`。

如果您嘗試繫結至其中一個儲存體 SDK 類型，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

由於會將整個 blob 內容載入記憶體中，因此只有在 Blob 大小很小時才建議繫結至 `string`、`Byte[]` 或 POCO。 一般而言，最好使用 `Stream` 或 `CloudBlockBlob` 類型。 如需詳細資訊，請參閱此文章稍後的[並行存取和記憶體使用量](#trigger---concurrency-and-memory-usage)。

在 JavaScript 中，使用 `context.bindings.<name from function.json>` 存取 Blob 的輸入資料。

## <a name="trigger---blob-name-patterns"></a>觸發程序 - Blob 名稱模式

您可以在 *function.json* 中的 `path` 屬性或 `BlobTrigger` 屬性建構函式中，指定 Blob 名稱模式。 名稱模式可以是[篩選條件或繫結運算式](functions-triggers-bindings.md#binding-expressions-and-patterns)。 下列各節提供相關範例。

### <a name="get-file-name-and-extension"></a>取得檔案名稱和副檔名

下列範例示範如何分別繫結至 Blob 檔案名稱和副檔名：

```json
"path": "input/{blobname}.{blobextension}",
```
如果 Blob 名稱為 *original-Blob1.txt*，則函式程式碼中的 `blobname` 和 `blobextension` 變數值為 *original-Blob1* 和 *txt*。

### <a name="filter-on-blob-name"></a>Blob 名稱上的篩選條件

下列範例只會觸發 `input` 容器中以字串 "original-" 開頭的 Blob：

```json
"path": "input/original-{name}",
```
 
如果 Blob 名稱為 *original-Blob1.txt*，則函式程式碼中 `name` 變數的值為 `Blob1`。

### <a name="filter-on-file-type"></a>檔案類型上的篩選條件

下列範例只會在 *.png* 檔案上觸發：

```json
"path": "samples/{name}.png",
```

### <a name="filter-on-curly-braces-in-file-names"></a>檔案名稱中大括號上的篩選條件

若要尋找檔案名稱中的大括號，請使用兩個括號逸出括號。 下列範例篩選名稱中有大括號的 Blob：

```json
"path": "images/{{20140101}}-{name}",
```

如果 Blob 名稱為 *{20140101}-soundfile.mp3*，則函式程式碼中的 `name` 變數值為 *soundfile.mp3*。 

## <a name="trigger---metadata"></a>觸發程序 - 中繼資料

Blob 觸發程序提供數個中繼資料屬性。 這些屬性可作為其他繫結中繫結運算式的一部分或程式碼中的參數使用。 這些值的語意與 [CloudBlob](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob?view=azure-dotnet) 類型相同。

|屬性  |類型  |說明  |
|---------|---------|---------|
|`BlobTrigger`|`string`|觸發 Blob 的路徑。|
|`Uri`|`System.Uri`|Blob 的主要位置 URI。|
|`Properties` |[BlobProperties](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.blob.blobproperties)|Blob 的系統屬性。 |
|`Metadata` |`IDictionary<string,string>`|Blob 的使用者定義中繼資料。|

例如，下列 C# 指令碼和 JavaScript 範例會記錄觸發 Blob 的路徑，包括容器在內：

```csharp
public static void Run(string myBlob, string blobTrigger, TraceWriter log)
{
    log.Info($"Full blob path: {blobTrigger}");
} 
```

```javascript
module.exports = function (context, myBlob) {
    context.log("Full blob path:", context.bindingData.blobTrigger);
    context.done();
};
```

## <a name="trigger---blob-receipts"></a>觸發程序 - Blob 回條

Azure Functions 執行階段可確保不會針對一樣新或更新的 blob 多次呼叫 blob 觸發程序函式。 為了判斷指定的 blob 版本是否已處理過，它會維護 *blob 回條*。

Azure Functions 會將 blob 回條儲存在您函數應用程式 (`AzureWebJobsStorage` 應用程式設定所定義) 的 Azure 儲存體帳戶中名為 *azure-webjobs-hosts*的容器中。 Blob 回條具有下列資訊：

* 已觸發的函數 ("&lt;函數應用程式名稱>.Functions.&lt;函數名稱>"，例如："MyFunctionApp.Functions.CopyBlob")
* 容器名稱
* Blob 類型 ("BlockBlob" 或 "PageBlob")
* Blob 名稱
* ETag (Blob 版本識別碼，例如："0x8D1DC6E70A277EF")

要強制重新處理某個 Blob，可以從 *azure-webjobs-hosts* 容器中手動刪除該 Blob 的 Blob 回條。

## <a name="trigger---poison-blobs"></a>觸發程序 - 有害的 Blob

當指定 blob 的 blob 觸發程序函式失敗時，Azure Functions 預設一共會重試該函式 5 次。 

如果 5 次嘗試全都失敗，Azure Functions 會將訊息新增至名為 *webjobs-blobtrigger-poison* 的儲存體佇列。 適用於有害 Blob 的佇列訊息是一個 JSON 物件，其中包含下列屬性：

* FunctionId (格式為 *&lt;function app name>*.Functions.*&lt;function name>*)
* BlobType ("BlockBlob" 或 "PageBlob")
* ContainerName
* BlobName
* ETag (Blob 版本識別碼，例如："0x8D1DC6E70A277EF")

## <a name="trigger---concurrency-and-memory-usage"></a>觸發程序 - 並行存取和記憶體使用量

Blob 觸發程序會在內部使用佇列，因此並行函式叫用數上限由 [host.json 中的佇列組態](functions-host-json.md#queues)所控制。 預設設定會將並行存取限制為 24 個叫用。 此限制會個別套用至使用 Blob 觸發程序的每個函式。

[取用方案](functions-scale.md#how-the-consumption-plan-works)會限制一個虛擬機器 (VM) 上的一個函式應用程式可使用 1.5 GB 的記憶體。 每個並行執行的函式執行個體和函式執行階段本身都會使用記憶體。 如果 Blob 觸發的函式將整個 Blob 載入記憶體中，則該函式用於 Blob 的記憶體上限為 24 * Blob 大小上限。 例如，若某個函式應用程式有三個 Blob 觸發的函式，則預設的每一 VM 並行存取上限將是 3 * 24 = 72 個函式叫用。

JavaScript 函式會將整個 Blob 載入記憶體中，而 C# 函式則會在您繫結至 `string`、`Byte[]` 或 POCO 時有此行為。

## <a name="trigger---polling"></a>觸發程序 - 輪詢

如果所監看的 blob 容器包含超過 10,000 個 blob，Functions 執行階段會掃描記錄檔以監看新增或變更的 blob。 此程序可能會導致延遲。 可能直到建立 Blob 之後數分鐘或更久，才會觸發函數。 此外，[會以「最大努力」建立儲存體記錄](/rest/api/storageservices/About-Storage-Analytics-Logging)。 並不保證會擷取所有事件。 在某些情況下可能會遺失記錄檔。 如果您需要更快或更可靠的 Blob 處理，請考慮在建立 Blob 時建立[佇列訊息](../storage/queues/storage-dotnet-how-to-use-queues.md)。 然後，使用[佇列觸發程序](functions-bindings-storage-queue.md) (而不是 Blob 觸發程序) 處理該 Blob。 另一個選項是使用 Event Grid；請參閱教學課程[使用 Event Grid 自動調整已上傳映像的大小](../event-grid/resize-images-on-storage-blob-upload-event.md)。

## <a name="input"></a>輸入

使用 Blob 儲存體輸入繫結來讀取 Blob。

## <a name="input---example"></a>輸入 - 範例

請參閱特定語言的範例：

* [C#](#input---c-example)
* [C# 指令碼 (.csx)](#input---c-script-example)
* [JavaScript](#input---javascript-example)
* [Java](#input---java-example)

### <a name="input---c-example"></a>輸入 - C# 範例

下列範例是使用一個佇列觸發程序和一個輸入 Blob 繫結的 [C# 函式](functions-dotnet-class-library.md)。 佇列訊息包含 Blob 的名稱，而函式會記錄 Blob 的大小。

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    TraceWriter log)
{
    log.Info($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```        

### <a name="input---c-script-example"></a>輸入 - C# 指令碼範例

<!--Same example for input and output. -->

下列範例所示範的是使用繫結之 function.json 檔案，以及 [C# 指令碼 (.csx)](functions-reference-csharp.md) 程式碼中的 Blob 輸入和輸出繫結。 此函式會建立文字 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 Blob 名稱為 *{originalblobname}-Copy*。

在 *function.json* 檔案中，`queueTrigger` 中繼資料屬性用於指定 `path` 屬性中的 Blob 名稱：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
``` 

[設定](#input---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

### <a name="input---javascript-example"></a>輸入 - JavaScript 範例

<!--Same example for input and output. -->

下列範例所示範的是 *function.json* 檔案中的 blob 輸入和輸出繫結，以及使用該繫結的 [JavaScript 程式碼](functions-reference-node.md)。 此函式會建立 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 Blob 名稱為 *{originalblobname}-Copy*。

在 *function.json* 檔案中，`queueTrigger` 中繼資料屬性用於指定 `path` 屬性中的 Blob 名稱：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
``` 

[設定](#input---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

### <a name="input---java-example"></a>輸入 - Java 範例

下列範例是使用一個佇列觸發程序和一個輸入 Blob 繫結的 Java 函式。 佇列訊息包含 Blob 的名稱，而函式會記錄 Blob 的大小。

```java
@FunctionName("getBlobSize")
@StorageAccount("AzureWebJobsStorage")
public void blobSize(@QueueTrigger(name = "filename",  queueName = "myqueue-items") String filename,
                    @BlobInput(name = "file", dataType = "binary", path = "samples-workitems/{queueTrigger") byte[] content,
       final ExecutionContext context) {
      context.getLogger().info("The size of \"" + filename + "\" is: " + content.length + " bytes");
 }
 ```

  在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值來自 Blob 的參數使用 `@BlobInput` 註釋。  此註釋可以搭配原生 Java 類型、POJO 或使用 `Optional<T>` 的可為 Null 值使用。 


## <a name="input---attributes"></a>輸入 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/BlobAttribute.cs)。

屬性的建構函式會採用 Blob 路徑和指示讀取或寫入的 `FileAccess` 參數，如下列範例所示：

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    TraceWriter log)
{
    log.Info($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}

```

您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read, Connection = "StorageConnectionAppSetting")] Stream myBlob,
    TraceWriter log)
{
    log.Info($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

您可以使用 `StorageAccount` 屬性來指定類別、方法或參數層級的儲存體帳戶。 如需詳細資訊，請參閱[觸發程序 - 屬性](#trigger---attributes)。

## <a name="input---configuration"></a>輸入 - 組態

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Blob` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 `blob`。 |
|**direction** | n/a | 必須設為 `in`。 例外狀況在[使用方式](#input---usage)一節中說明。 |
|**name** | n/a | 表示函式程式碼中 Blob 的變數名稱。|
|**路徑** |**BlobPath** | blob 的路徑。 | 
|**連接** |**連接**| 應用程式設定的名稱包含要用於此繫結的儲存體連接字串。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為「MyStorage」，則函式執行階段會尋找名稱為「AzureWebJobsMyStorage」的應用程式設定。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。<br><br>連接字串必須是用於一般用途的儲存體帳戶，而不是[Blob 儲存體帳戶](../storage/common/storage-account-overview.md#types-of-storage-accounts)。|
|n/a | **Access** | 指出您是否將讀取或寫入。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="input---usage"></a>輸入 - 使用方式

在 C# 和 C# 指令碼中，您可以針對 Blob 輸入繫結使用下列參數類型：

* `Stream`
* `TextReader`
* `string`
* `Byte[]`
* `CloudBlobContainer`
* `CloudBlobDirectory`
* `ICloudBlob`<sup>1</sup>
* `CloudBlockBlob`<sup>1</sup>
* `CloudPageBlob`<sup>1</sup>
* `CloudAppendBlob`<sup>1</sup>

<sup>1</sup> 在 *function.json* 或 `FileAccess.ReadWrite` C# 類別庫中需要 "inout" 繫結 `direction`。

如果您嘗試繫結至其中一個儲存體 SDK 類型，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

由於會將整個 blob 內容載入記憶體中，因此只有在 Blob 大小很小時才建議繫結至 `string` 或 `Byte[]`。 一般而言，最好使用 `Stream` 或 `CloudBlockBlob` 類型。 如需詳細資訊，請參閱此文章稍早的[並行存取和記憶體使用量](#trigger---concurrency-and-memory-usage)。

在 JavaScript 中，使用 `context.bindings.<name from function.json>` 存取 Blob 資料。

## <a name="output"></a>輸出

使用 Blob 儲存體輸出繫結來寫入 Blob。

## <a name="output---example"></a>輸出 - 範例

請參閱特定語言的範例：

* [C#](#output---c-example)
* [C# 指令碼 (.csx)](#output---c-script-example)
* [JavaScript](#output---javascript-example)
* [Java](#output---java-example)

### <a name="output---c-example"></a>輸出 - C# 範例

下列範例是使用一個 Blob 觸發程序和兩個輸出 Blob 繫結的 [C# 函式](functions-dotnet-class-library.md)。 此函式是透過在 *sample-images* 容器中建立映像 Blob 而觸發。 它會建立映像 Blob 的小型及中型複本。 

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image, 
    [Blob("sample-images-sm/{name}", FileAccess.Write)] Stream imageSmall, 
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageMedium)
{
    var imageBuilder = ImageResizer.ImageBuilder.Current;
    var size = imageDimensionsTable[ImageSize.Small];

    imageBuilder.Build(image, imageSmall,
        new ResizeSettings(size.Item1, size.Item2, FitMode.Max, null), false);

    image.Position = 0;
    size = imageDimensionsTable[ImageSize.Medium];

    imageBuilder.Build(image, imageMedium,
        new ResizeSettings(size.Item1, size.Item2, FitMode.Max, null), false);
}

public enum ImageSize { ExtraSmall, Small, Medium }

private static Dictionary<ImageSize, (int, int)> imageDimensionsTable = new Dictionary<ImageSize, (int, int)>() {
    { ImageSize.ExtraSmall, (320, 200) },
    { ImageSize.Small,      (640, 400) },
    { ImageSize.Medium,     (800, 600) }
};
```        

### <a name="output---c-script-example"></a>輸出 - C# 指令碼範例

<!--Same example for input and output. -->

下列範例所示範的是使用繫結之 function.json 檔案，以及 [C# 指令碼 (.csx)](functions-reference-csharp.md) 程式碼中的 Blob 輸入和輸出繫結。 此函式會建立文字 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 Blob 名稱為 *{originalblobname}-Copy*。

在 *function.json* 檔案中，`queueTrigger` 中繼資料屬性用於指定 `path` 屬性中的 Blob 名稱：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
``` 

[設定](#output---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

### <a name="output---javascript-example"></a>輸出 - JavaScript 範例

<!--Same example for input and output. -->

下列範例所示範的是 *function.json* 檔案中的 blob 輸入和輸出繫結，以及使用該繫結的 [JavaScript 程式碼](functions-reference-node.md)。 此函式會建立 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 Blob 名稱為 *{originalblobname}-Copy*。

在 *function.json* 檔案中，`queueTrigger` 中繼資料屬性用於指定 `path` 屬性中的 Blob 名稱：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
``` 

[設定](#output---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

### <a name="output---java-example"></a>輸出 - Java 範例

下列範例顯示 Java 函式中的 Blob 輸入和輸出繫結。 此函式會建立文字 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 Blob 名稱為 {originalblobname}-Copy

```java
@FunctionName("copyTextBlob")
@StorageAccount("AzureWebJobsStorage")
@BlobOutput(name = "target", path = "samples-workitems/{queueTrigger}-Copy")
public String blobCopy(
    @QueueTrigger(name = "filename", queueName = "myqueue-items") String filename,
    @BlobInput(name = "source", path = "samples-workitems/{queueTrigger}") String content ) {
      return content;
 }
 ```

 在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值要寫入至 Blob 儲存體中物件的函式參數使用 `@BlobOutput` 註釋。  參數類型應為 `OutputBinding<T>`，其中 T 是任何原生 Java 類型的 POJO。


## <a name="output---attributes"></a>輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/BlobAttribute.cs)。

屬性的建構函式會採用 Blob 路徑和指示讀取或寫入的 `FileAccess` 參數，如下列範例所示：

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image, 
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
{
    ...
}
```

您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image, 
    [Blob("sample-images-md/{name}", FileAccess.Write, Connection = "StorageConnectionAppSetting")] Stream imageSmall)
{
    ...
}
```

如需完整範例，請參閱[輸出 - C# 範例](#output---c-example)。

您可以使用 `StorageAccount` 屬性來指定類別、方法或參數層級的儲存體帳戶。 如需詳細資訊，請參閱[觸發程序 - 屬性](#trigger---attributes)。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Blob` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 `blob`。 |
|**direction** | n/a | 必須針對輸出繫結設定為 `out`。 例外狀況在[使用方式](#output---usage)一節中說明。 |
|**name** | n/a | 表示函式程式碼中 Blob 的變數名稱。  設為 `$return` 以參考函式傳回值。|
|**路徑** |**BlobPath** | blob 的路徑。 | 
|**連接** |**連接**| 應用程式設定的名稱包含要用於此繫結的儲存體連接字串。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為「MyStorage」，則函式執行階段會尋找名稱為「AzureWebJobsMyStorage」的應用程式設定。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。<br><br>連接字串必須是用於一般用途的儲存體帳戶，而不是[Blob 儲存體帳戶](../storage/common/storage-account-overview.md#types-of-storage-accounts)。|
|n/a | **Access** | 指出您是否將讀取或寫入。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>輸出 - 使用方式

在 C# 和 C# 指令碼中，您可以繫結至下列類型以寫入 Blob：

* `TextWriter`
* `out string`
* `out Byte[]`
* `CloudBlobStream`
* `Stream`
* `CloudBlobContainer`<sup>1</sup>
* `CloudBlobDirectory`
* `ICloudBlob`<sup>2</sup>
* `CloudBlockBlob`<sup>2</sup>
* `CloudPageBlob`<sup>2</sup>
* `CloudAppendBlob`<sup>2</sup>

<sup>1</sup> 在 *function.json* 或 `FileAccess.Read` C# 類別庫中需要 "in" 繫結 `direction`。 不過，您可以使用執行階段提供的容器物件來執行寫入作業，例如將 blob 上傳至容器。

<sup>2</sup> 在 *function.json* 或 `FileAccess.ReadWrite` C# 類別庫中需要 "inout" 繫結 `direction`。

如果您嘗試繫結至其中一個儲存體 SDK 類型，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

在非同步函式中，使用傳回值或 `IAsyncCollector`，而不是 `out` 參數。

由於會將整個 blob 內容載入記憶體中，因此只有在 Blob 大小很小時才建議繫結至 `string` 或 `Byte[]`。 一般而言，最好使用 `Stream` 或 `CloudBlockBlob` 類型。 如需詳細資訊，請參閱此文章稍早的[並行存取和記憶體使用量](#trigger---concurrency-and-memory-usage)。


在 JavaScript 中，使用 `context.bindings.<name from function.json>` 存取 Blob 資料。

## <a name="exceptions-and-return-codes"></a>例外狀況和傳回碼

| 繫結 |  參考 |
|---|---|
| Blob | [Blob 錯誤碼](https://docs.microsoft.com/rest/api/storageservices/fileservices/blob-service-error-codes) |
| Bob、資料表、佇列 |  [儲存體錯誤碼](https://docs.microsoft.com/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Bob、資料表、佇列 |  [疑難排解](https://docs.microsoft.com/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>後續步驟

* [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

<!---
> [!div class="nextstepaction"]
> [Go to a quickstart that uses a Blob storage trigger](functions-create-storage-blob-triggered-function.md)
--->
