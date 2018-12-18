---
title: Azure Functions 的 Event Grid 觸發程序
description: 了解如何在 Azure Functions 中處理 Event Grid 事件。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/23/2018
ms.author: glenga
ms.openlocfilehash: 850b30ff42b77fe0ab527a54b62ba0f77027f932
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746241"
---
# <a name="event-grid-trigger-for-azure-functions"></a>Azure Functions 的 Event Grid 觸發程序

本文說明如何在 Azure Functions 中處理 [Event Grid](../event-grid/overview.md) 事件。

Event Grid 是一項 Azure 服務，會傳送 HTTP 要求通知您「發行者」中發生的事件。 發行者是產生事件的服務或資源。 例如，Azure Blob 儲存體帳戶即為發行者，而 [Blob 的上傳或刪除是事件](../storage/blobs/storage-blob-event-overview.md)。 部分 [Azure 服務內建有將事件發佈至 Event Grid 的支援](../event-grid/overview.md#event-sources)。 

事件*處理常式*會接收及處理事件。 Azure Functions 是數個[有內建 Event Grid 事件處理支援的 Azure 服務](../event-grid/overview.md#event-handlers)之一。 在本文中，您將了解如何在接收到來自 Event Grid 的事件時使用 Event Grid 觸發程序來叫用函式。

如果您想要的話，您可以使用 HTTP 觸發程序來處理 Event Grid 事件；請參閱本文稍後的[以 HTTP 觸發程序作為 Event Grid 觸發程序](#use-an-http-trigger-as-an-event-grid-trigger)。 目前，當事件是在 [CloudEvents 結構描述](../event-grid/cloudevents-schema.md)中傳遞時，您無法針對 Azure Functions 應用程式使用事件格線觸發程序。 相反地，請使用 HTTP 觸發程序。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventGrid) NuGet 套件 1.x 版中提供 Event Grid 觸發程序。 套件的原始程式碼位於 [azure-functions-eventgrid-extension](https://github.com/Azure/azure-functions-eventgrid-extension/tree/master) GitHub 存放庫中。

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventGrid) NuGet 套件 2.x 版中提供 Event Grid 觸發程序。 套件的原始程式碼位於 [azure-functions-eventgrid-extension](https://github.com/Azure/azure-functions-eventgrid-extension/tree/v2.x) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="example"></a>範例

請參閱 Event Grid 觸發程序的語言特定範例：

* [C#](#c-example)
* [C# 指令碼 (.csx)](#c-script-example)
* [JavaScript](#javascript-example)
* [Java](#trigger---java-example)

如需 HTTP 觸發程序範例，請參閱本文稍後的[如何使用 HTTP 觸發程序](#use-an-http-trigger-as-an-event-grid-trigger)。

### <a name="c-example"></a>C# 範例

下列範例顯示繫結至 `JObject` 的 Functions 1.x [C# 函式](functions-dotnet-class-library.md)：

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTriggerCSharp")]
        public static void Run([EventGridTrigger]JObject eventGridEvent, TraceWriter log)
        {
            log.Info(eventGridEvent.ToString(Formatting.Indented));
        }
    }
}
```

下列範例顯示繫結至 `EventGridEvent` 的 Functions 2.x [C# 函式](functions-dotnet-class-library.md)：

```cs
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTest")]
        public static void EventGridTest([EventGridTrigger]EventGridEvent eventGridEvent, TraceWriter log)
        {
            log.Info(eventGridEvent.Data.ToString());
        }
    }
}
```

如需詳細資訊，請參閱[套件](#packages)、[屬性](#attributes)、[設定](#configuration)和[用法](#usage)。

### <a name="c-script-example"></a>C# 指令碼範例

下列範例示範 function.json 檔案中的觸發程序繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。

以下是 *function.json* 檔案中的繫結資料：

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

以下是繫結至 `JObject` 的 Functions 1.x C# 指令碼：

```cs
#r "Newtonsoft.Json"

using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public static void Run(JObject eventGridEvent, TraceWriter log)
{
    log.Info(eventGridEvent.ToString(Formatting.Indented));
}
```

以下是繫結至 `EventGridEvent` 的 Functions 2.x C# 指令碼：

```csharp
#r "Microsoft.Azure.EventGrid"
using Microsoft.Azure.EventGrid.Models;

public static void Run(EventGridEvent eventGridEvent, TraceWriter log)
{
    log.Info(eventGridEvent.Data.ToString());
}
```

如需詳細資訊，請參閱[套件](#packages)、[屬性](#attributes)、[設定](#configuration)和[用法](#usage)。

### <a name="javascript-example"></a>JavaScript 範例

下列範例示範的是使用繫結之 function.json 檔案，以及 [JavaScript 函式](functions-reference-node.md)中的觸發程序繫結。

以下是 *function.json* 檔案中的繫結資料：

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, eventGridEvent) {
    context.log("JavaScript Event Grid function processed a request.");
    context.log("Subject: " + eventGridEvent.subject);
    context.log("Time: " + eventGridEvent.eventTime);
    context.log("Data: " + JSON.stringify(eventGridEvent.data));
    context.done();
};
```

### <a name="trigger---java-example"></a>觸發程序 - Java 範例

下列範例示範 function.json 檔案中的觸發程序繫結，以及使用此繫結並輸出事件的 [Java 函式](functions-reference-java.md)。

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ]
}
```

以下是 Java 程式碼：

```java
@FunctionName("eventGridMonitor")
  public void logEvent(
     @EventGridTrigger(name = "event") String content,
      final ExecutionContext context
  ) { 
      context.getLogger().info(content);
    }
```

在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值來自事件方格的參數使用 `EventGridTrigger` 註釋。 具有這些附註的參數會使得函式在事件抵達時執行。  此註釋可以搭配原生 Java 類型、POJO 或使用 `Optional<T>` 的可為 Null 值使用。 
     
## <a name="attributes"></a>屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [EventGridTrigger](https://github.com/Azure/azure-functions-eventgrid-extension/blob/master/src/EventGridExtension/EventGridTriggerAttribute.cs) 屬性。

以下是方法簽章中的 `EventGridTrigger` 屬性：

```csharp
[FunctionName("EventGridTest")]
public static void EventGridTest([EventGridTrigger] JObject eventGridEvent, TraceWriter log)
{
    ...
}
 ```

如需完整範例，請參閱 [C# 範例](#c-example)。

## <a name="configuration"></a>組態

下表說明您在 function.json 檔案中設定的繫結設定屬性。 沒有要在 `EventGridTrigger` 屬性中設定的建構函式參數或屬性。

|function.json 屬性 |說明|
|---------|---------|----------------------|
| **type** | 必要項目 - 必須設定為 `eventGridTrigger`。 |
| **direction** | 必要項目 - 必須設定為 `in`。 |
| **name** | 必要項目 - 函式程式碼中用於接收事件資料之參數的變數名稱。 |

## <a name="usage"></a>使用量

使用 Azure Functions 1.x 中的 C# 和 F# 函式時，您可以為 Event Grid 觸發程序使用下列參數類型：

* `JObject`
* `string`

使用 Azure Functions 2.x 中的 C# 和 F# 函式時，您也可以為 Event Grid 觸發程序使用下列參數類型：

* `Microsoft.Azure.EventGrid.Models.EventGridEvent` - 定義所有事件類型通用之欄位的屬性。

> [!NOTE]
> 在 Functions v1 中，如果您嘗試繫結至 `Microsoft.Azure.WebJobs.Extensions.EventGrid.EventGridEvent`，編譯器將會顯示「已淘汰」訊息，並建議您改用 `Microsoft.Azure.EventGrid.Models.EventGridEvent`。 若要使用較新的類型，請參考 [Microsoft.Azure.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) NuGet 套件，並且為 `EventGridEvent` 類型名稱加上 `Microsoft.Azure.EventGrid.Models` 首碼，加以完整限定。 如需如何在 C# 指令碼函式中參考 NuGet 套件的相關資訊，請參閱[使用 NuGet 套件](functions-reference-csharp.md#using-nuget-packages)

對於 JavaScript 函式，由 *function.json* `name` 屬性命名的參數會參考事件物件。

## <a name="event-schema"></a>結構描述

接收到的 Event Grid 事件資料，會是 HTTP 要求本文中的 JSON 物件。 此 JSON 大致如下列範例所示：

```json
[{
  "topic": "/subscriptions/{subscriptionid}/resourceGroups/eg0122/providers/Microsoft.Storage/storageAccounts/egblobstore",
  "subject": "/blobServices/default/containers/{containername}/blobs/blobname.jpg",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2018-01-23T17:02:19.6069787Z",
  "id": "{guid}",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "{guid}",
    "requestId": "{guid}",
    "eTag": "0x8D562831044DDD0",
    "contentType": "application/octet-stream",
    "contentLength": 2248,
    "blobType": "BlockBlob",
    "url": "https://egblobstore.blob.core.windows.net/{containername}/blobname.jpg",
    "sequencer": "000000000000272D000000000003D60F",
    "storageDiagnostics": {
      "batchId": "{guid}"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

範例中顯示一個元素的陣列。 Event Grid 一律會傳送陣列，且在陣列中可能會傳送多個事件。 執行階段分別會為每個陣列元素叫用您的函式一次。

所有事件類型的事件 JSON 資料中會具有相同的最上層屬性，但 `data` 屬性的內容則會隨著事件類型而不同。 此處顯示的是 Blob 儲存體事件的範例。

如需通用屬性和事件特定屬性的說明，請參閱 Event Grid 文件中的[事件屬性](../event-grid/event-schema.md#event-properties)。

`EventGridEvent` 類型只會定義最上層屬性；`Data` 屬性為 `JObject`。 

## <a name="create-a-subscription"></a>建立訂用帳戶

若要開始接收 Event Grid HTTP 要求，請建立會指定端點 URL 以叫用函式的 Event Grid 訂用帳戶。

### <a name="azure-portal"></a>Azure 入口網站

對於您在 Azure 入口網站中使用 Event Grid 觸發程序開發的函式，選取**新增 Event Grid 訂用帳戶**。

![在入口網站中建立訂用帳戶](media/functions-bindings-event-grid/portal-sub-create.png)

當您選取此連結時，入口網站會開啟預先填入端點 URL 的**建立事件訂用帳戶**頁面。

![預先填入的端點 URL](media/functions-bindings-event-grid/endpoint-url.png)

如需如何使用 Azure 入口網站建立訂用帳戶的詳細資訊，請參閱 Event Grid 文件中的[建立自訂事件 - Azure 入口網站](../event-grid/custom-event-quickstart-portal.md)。

### <a name="azure-cli"></a>Azure CLI

若要使用 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) 建立訂用帳戶，請使用 [az eventgrid event-subscription create](https://docs.microsoft.com/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az-eventgrid-event-subscription-create) 命令。

此命令需要叫用函式的端點 URL。 下列範例顯示 URL 模式：

```
https://{functionappname}.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName={functionname}&code={systemkey}
```

系統金鑰是必須包含在端點 URL 中供 Event Grid 觸發程序使用的授權金鑰。 下一節將說明如何取得系統金鑰。

以下是訂閱 Blob 儲存體帳戶的範例 (以預留位置表示系統金鑰)：

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup \
--provider-namespace Microsoft.Storage --resource-type storageAccounts \
--resource-name glengablobstorage --name myFuncSub  \
--included-event-types Microsoft.Storage.BlobCreated \
--subject-begins-with /blobServices/default/containers/images/blobs/ \
--endpoint https://glengastorageevents.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName=imageresizefunc&code=LUwlnhIsNtSiUjv/sNtSiUjvsNtSiUjvsNtSiUjvYb7XDonDUr/RUg==
```

如需如何建立訂用帳戶的詳細資訊，請參閱 [Blob 儲存體快速入門](../storage/blobs/storage-blob-event-quickstart.md#subscribe-to-your-storage-account)或其他 Event Grid 快速入門。

### <a name="get-the-system-key"></a>取得系統金鑰

您可以使用下列 API (HTTP GET) 來取得系統金鑰：

```
http://{functionappname}.azurewebsites.net/admin/host/systemkeys/eventgridextensionconfig_extension?code={adminkey}
```

這是管理員 API，因此需要函式應用程式的[主要金鑰](functions-bindings-http-webhook.md#authorization-keys)。 請勿混淆系統金鑰 (用來叫用事件方格觸發程序函式) 與主要金鑰 (用來執行函式應用程式的管理工作)。 當您訂閱 Event Grid 主題時，請務必使用系統金鑰。 

以下範例說明提供系統金鑰的回應：

```
{
  "name": "eventgridextensionconfig_extension",
  "value": "{the system key for the function}",
  "links": [
    {
      "rel": "self",
      "href": "{the URL for the function, without the system key}"
    }
  ]
}
```

如需詳細資訊，請參閱 HTTP 觸發程序參考文章中的[授權金鑰](functions-bindings-http-webhook.md#authorization-keys)。 

或者，您可以傳送 HTTP PUT 以自行指定金鑰值。

## <a name="local-testing-with-viewer-web-app"></a>使用檢視器 Web 應用程式的本機測試

若要在本機測試 Event Grid 觸發程序，您必須取得從雲端中的原有位置傳遞到本機電腦的 Event Grid HTTP 要求。 其中一種方式是在線上擷取要求，然後手動將其重新傳送至您的本機電腦：

2. [建立檢視器 Web 應用程式](#create-a-viewer-web-app)，以擷取事件訊息。
3. [建立事件格線訂用帳戶](#create-an-event-grid-subscription)，以將事件傳送至檢視器應用程式。
4. [產生要求](#generate-a-request)，並從檢視器應用程式複製要求本文。
5. [手動將要求發佈至](#manually-post-the-request) Event Grid 觸發程序函式的 localhost URL。

完成測試後，您可以藉由更新端點，在生產環境使用相同的訂用帳戶。 使用 [az eventgrid event-subscription update](https://docs.microsoft.com/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az-eventgrid-event-subscription-update) Azure CLI 命令。

### <a name="create-a-viewer-web-app"></a>建立檢視器 Web 應用程式

若要簡化擷取事件訊息，您可以部署[預先建置的 Web 應用程式](https://github.com/Azure-Samples/azure-event-grid-viewer)，以顯示事件訊息。 已部署的解決方案包含 App Service 方案、App Service Web 應用程式，以及 GitHub 中的原始程式碼。

選取 [部署至 Azure]，將解決方案部署至您的訂用帳戶。 在 Azure 入口網站中，提供參數的值。

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>

部署需要幾分鐘的時間才能完成。 成功部署之後，檢視 Web 應用程式，確定它正在執行。 在網頁瀏覽器中，瀏覽至：`https://<your-site-name>.azurewebsites.net`

您看到網站，但其中尚未發佈任何事件。

![檢視新網站](media/functions-bindings-event-grid/view-site.png)

### <a name="create-an-event-grid-subscription"></a>建立事件格線訂用帳戶

建立您想要測試之類型的事件格線訂用帳戶，並從您的 Web 應用程式提供其 URL 作為端點的事件通知。 Web 應用程式的端點必須包含的尾碼 `/api/updates/`。 因此，完整 URL 是 `https://<your-site-name>.azurewebsites.net/api/updates`

如需如何使用 Azure 入口網站建立訂用帳戶的資訊，請參閱事件格線文件中的[建立自訂事件 - Azure 入口網站](../event-grid/custom-event-quickstart-portal.md)。

### <a name="generate-a-request"></a>產生要求

觸發會對您的 Web 應用程式端點產生 HTTP 流量的事件。  例如，如果您已建立 Blob 儲存體訂用帳戶，請上傳或刪除 Blob。 當要求出現在 Web 應用程式上時，請複製要求本文。

您會先收到訂用帳戶驗證要求；請忽略任何驗證要求，並複製事件要求。

![從 Web 應用程式複製要求本文](media/functions-bindings-event-grid/view-results.png)

### <a name="manually-post-the-request"></a>手動發佈要求

在本機執行您的 Event Grid 函式。

使用 [Postman](https://www.getpostman.com/) 或 [curl](https://curl.haxx.se/docs/httpscripting.html) 之類的工具建立 HTTP POST 要求：

* 設定 `Content-Type: application/json` 標頭。
* 設定 `aeg-event-type: Notification` 標頭。
* 將 RequestBin 資料貼到要求本文中。 
* 使用下列模式，發佈到 Event Grid 觸發程序函式的 URL：

```
http://localhost:7071/admin/extensions/EventGridExtensionConfig?functionName={functionname}
``` 

`functionName` 參數必須是 `FunctionName` 屬性中指定的名稱。

下列螢幕擷取畫面顯示 Postman 中的標頭和要求本文：

![Postman 中的標頭](media/functions-bindings-event-grid/postman2.png)

![Postman 中的要求本文](media/functions-bindings-event-grid/postman.png)

Event Grid 觸發程序函式會執行並顯示類似於下列範例的記錄：

![Event Grid 觸發程序函式記錄的範例](media/functions-bindings-event-grid/eg-output.png)

## <a name="local-testing-with-ngrok"></a>使用 ngrok 進行本機測試

在本機測試 Event Grid 觸發程序的另一種方式，是自動建立網際網路與您開發電腦之間的 HTTP 連線。 您可以使用名為 [ngrok](https://ngrok.com/) 的開放原始碼工具執行此作業：

3. [建立 ngrok 端點](#create-an-ngrok-endpoint)。
4. [執行 Event Grid 觸發程序函式](#run-the-event-grid-trigger-function)。
5. [建立會將事件傳送至 ngrok 端點的 Event Grid 訂用帳戶](#create-a-subscription)。
6. [觸發事件](#trigger-an-event)。

完成測試後，您可以藉由更新端點，在生產環境使用相同的訂用帳戶。 使用 [az eventgrid event-subscription update](https://docs.microsoft.com/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az-eventgrid-event-subscription-update) Azure CLI 命令。

### <a name="create-an-ngrok-endpoint"></a>建立 ngrok 端點

從 [ngrok](https://ngrok.com/) 下載 *ngrok.exe*，並使用下列命令加以執行：

```
ngrok http -host-header=localhost 7071
```

此時必須使用 -host-header 參數，因為函式執行階段在 localhost 執行時預期會有來自 localhost 的要求。 7071 是執行階段在本機執行時的預設連接埠號碼。

此命令會建立如下的輸出：

```
Session Status                online
Version                       2.2.8
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://263db807.ngrok.io -> localhost:7071
Forwarding                    https://263db807.ngrok.io -> localhost:7071

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

您的 Event Grid 訂用帳戶將會使用 `https://{subdomain}.ngrok.io` URL。

### <a name="run-the-event-grid-trigger-function"></a>執行 Event Grid 觸發程序函式

Event Grid 不會以特殊方式處理 ngrok URL，因此，在訂用帳戶建立時，您的函式必須在本機執行。 若非如此，則不會傳送驗證回應，且訂用帳戶的建立將會失敗。

### <a name="create-a-subscription"></a>建立訂用帳戶

建立您想要測試的 Event Grid 訂用帳戶類型，並為其提供您的 ngrok 端點。

將此端點模式使用於 Functions 1.x：
```
https://{subdomain}.ngrok.io/admin/extensions/EventGridExtensionConfig?functionName={functionname}
``` 
將此端點模式使用於 Functions 2.x：
```
https://{subdomain}.ngrok.io/runtime/webhooks/EventGridExtensionConfig?functionName={functionName}
``` 
`functionName` 參數必須是 `FunctionName` 屬性中指定的名稱。

以下是使用 Azure CLI 的範例：

```
az eventgrid event-subscription create --resource-id /subscriptions/aeb4b7cb-b7cb-b7cb-b7cb-b7cbb6607f30/resourceGroups/eg0122/providers/Microsoft.Storage/storageAccounts/egblobstor0122 --name egblobsub0126 --endpoint https://263db807.ngrok.io/admin/extensions/EventGridExtensionConfig?functionName=EventGridTrigger
```

如需如何建立訂用帳戶的相關資訊，請參閱本文前述的[建立訂用帳戶](#create-a-subscription)。

### <a name="trigger-an-event"></a>觸發事件

觸發會對您的 ngrok 端點產生 HTTP 流量的事件。  例如，如果您已建立 Blob 儲存體訂用帳戶，請上傳或刪除 Blob。

Event Grid 觸發程序函式會執行並顯示類似於下列範例的記錄：

![Event Grid 觸發程序函式記錄的範例](media/functions-bindings-event-grid/eg-output.png)

## <a name="use-an-http-trigger-as-an-event-grid-trigger"></a>以 HTTP 觸發程序作為 Event Grid 觸發程序

接收到的 Event Grid 事件會是 HTTP 要求，因此您可以使用 HTTP 觸發程序來處理事件，而不使用 Event Grid 觸發程序。 這麼做的可能原因之一，是對於叫用函式的端點 URL 想要有更充分的掌控。 另一個原因是當您需要接收 [CloudEvents 結構描述](../event-grid/cloudevents-schema.md)中的事件時。 事件格線觸發程序目前不支援 CloudEvents 結構描述。 本節中的範例示範事件格線結構描述和 CloudEvents 結構描述的解決方案。

如果您使用 HTTP 觸發程序，則必須撰寫程式碼來處理 Event Grid 觸發程序自動執行的作業：

* 將驗證回應傳送給[訂用帳戶驗證要求](../event-grid/security-authentication.md#webhook-event-delivery)。
* 為要求本文中包含的每個事件陣列元素分別叫用一次函式。

如需在本機叫用函式時或在 Azure 中執行函式時所使用的 URL 相關資訊，請參閱 [HTTP 觸發程序繫結參考文件](functions-bindings-http-webhook.md)

### <a name="event-grid-schema"></a>事件格線結構描述

HTTP 觸發程序的下列範例 C# 程式碼會模擬事件格線觸發程序的行為。 使用此範例示範以事件格線結構描述傳送的事件。

```csharp
[FunctionName("HttpTrigger")]
public static async Task<HttpResponseMessage> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")]HttpRequestMessage req,
    TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    var messages = await req.Content.ReadAsAsync<JArray>();

    // If the request is for subscription validation, send back the validation code.
    if (messages.Count > 0 && string.Equals((string)messages[0]["eventType"], 
        "Microsoft.EventGrid.SubscriptionValidationEvent", 
        System.StringComparison.OrdinalIgnoreCase))
    {
        log.Info("Validate request received");
        return req.CreateResponse<object>(new
        {
            validationResponse = messages[0]["data"]["validationCode"]
        });
    }

    // The request is not for subscription validation, so it's for one or more events.
    foreach (JObject message in messages)
    {
        // Handle one event.
        EventGridEvent eventGridEvent = message.ToObject<EventGridEvent>();
        log.Info($"Subject: {eventGridEvent.Subject}");
        log.Info($"Time: {eventGridEvent.EventTime}");
        log.Info($"Event data: {eventGridEvent.Data.ToString()}");
    }

    return req.CreateResponse(HttpStatusCode.OK);
}
```

HTTP 觸發程序的下列範例 JavaScript 程式碼會模擬事件格線觸發程序行為。 使用此範例示範以事件格線結構描述傳送的事件。

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    var messages = req.body;
    // If the request is for subscription validation, send back the validation code.
    if (messages.length > 0 && messages[0].eventType == "Microsoft.EventGrid.SubscriptionValidationEvent") {
        context.log('Validate request received');
        var code = messages[0].data.validationCode;
        context.res = { status: 200, body: { "ValidationResponse": code } };
    }
    else {
        // The request is not for subscription validation, so it's for one or more events.
        // Event Grid schema delivers events in an array.
        for (var i = 0; i < messages.length; i++) {
            // Handle one event.
            var message = messages[i];
            context.log('Subject: ' + message.subject);
            context.log('Time: ' + message.eventTime);
            context.log('Data: ' + JSON.stringify(message.data));
        }
    }
    context.done();
};
```

您的事件處理程式碼會透過 `messages` 陣列進入迴圈。

### <a name="cloudevents-schema"></a>CloudEvents 結構描述

HTTP 觸發程序的下列範例 C# 程式碼會模擬事件格線觸發程序的行為。  使用此範例示範以 CloudEvents 結構描述傳送的事件。

```csharp
[FunctionName("HttpTrigger")]
public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequestMessage req, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    var requestmessage = await req.Content.ReadAsStringAsync();
    var message = JToken.Parse(requestmessage);

    if (message.Type == JTokenType.Array)
    {
        // If the request is for subscription validation, send back the validation code.
        if (string.Equals((string)message[0]["eventType"],
        "Microsoft.EventGrid.SubscriptionValidationEvent",
        System.StringComparison.OrdinalIgnoreCase))
        {
            log.Info("Validate request received");
            return req.CreateResponse<object>(new
            {
                validationResponse = message[0]["data"]["validationCode"]
            });
        }
    }
    else
    {
        // The request is not for subscription validation, so it's for an event.
        // CloudEvents schema delivers one event at a time.
        log.Info($"Source: {message["source"]}");
        log.Info($"Time: {message["eventTime"]}");
        log.Info($"Event data: {message["data"].ToString()}");
    }

    return req.CreateResponse(HttpStatusCode.OK);
}
```

HTTP 觸發程序的下列範例 JavaScript 程式碼會模擬事件格線觸發程序行為。 使用此範例示範以 CloudEvents 結構描述傳送的事件。

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    var message = req.body;
    // If the request is for subscription validation, send back the validation code.
    if (message.length > 0 && message[0].eventType == "Microsoft.EventGrid.SubscriptionValidationEvent") {
        context.log('Validate request received');
        var code = message[0].data.validationCode;
        context.res = { status: 200, body: { "ValidationResponse": code } };
    }
    else {
        // The request is not for subscription validation, so it's for an event.
        // CloudEvents schema delivers one event at a time.
        var event = JSON.parse(message);
        context.log('Source: ' + event.source);
        context.log('Time: ' + event.eventTime);
        context.log('Data: ' + JSON.stringify(event.data));
    }
    context.done();
};
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [深入了解 Event Grid](../event-grid/overview.md)
