---
title: 適用於 Functions 2.x 的 Azure Cosmos DB 繫結
description: 了解如何在 Azure Functions 中使用 Azure Cosmos DB 觸發程序和繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.service: azure-functions; cosmos-db
ms.devlang: multiple
ms.topic: reference
ms.date: 11/21/2017
ms.author: glenga
ms.openlocfilehash: 0726bd3ded0618d7fb45a589c21325717da7a5cf
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319025"
---
# <a name="azure-cosmos-db-bindings-for-azure-functions-2x"></a>適用於 Azure Functions 2.x 的 Azure Cosmos DB 繫結

> [!div class="op_single_selector" title1="Select the version of the Azure Functions runtime you are using: "]
> * [第 1 版](functions-bindings-cosmosdb.md)
> * [第 2 版](functions-bindings-cosmosdb-v2.md)

本文說明如何在 Azure Functions 2.x 中使用 [Azure Cosmos DB](..\cosmos-db\serverless-computing-database.md) 繫結。 Azure Functions 支援適用於 Azure Cosmos DB 的觸發程序、輸入和輸出繫結。

> [!NOTE]
> 此文章適用於 [Azure Functions 2.x 版](functions-versions.md)。  如需如何在 Functions 1.x 中使用這些繫結的詳細資訊，請參閱[適用於 Azure Functions 1.x 的 Azure Cosmos DB 繫結](functions-bindings-cosmosdb.md)。
>
> 這個繫結最初命名為 DocumentDB。 在 Functions 2.x 版本中，觸發程序、繫結和套件已重新命名為 Cosmos DB。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="supported-apis"></a>支援的 API

[!INCLUDE [SQL API support only](../../includes/functions-cosmosdb-sqlapi-note.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.CosmosDB](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB) NuGet 套件，版本 3.x 中提供適用於 Functions 2.x 版本的 Azure Cosmos DB 繫結。 繫結的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="trigger"></a>觸發程序

Azure Cosmos DB 觸發程序會使用 [Azure Cosmos DB 變更摘要](../cosmos-db/change-feed.md)，跨分割區接聽插入項目及變更。 變更摘要會發行插入和更新，而非刪除。

## <a name="trigger---example"></a>觸發程序 - 範例

請參閱特定語言的範例：

* [C#](#trigger---c-example)
* [C# 指令碼 (.csx)](#trigger---c-script-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---java-example)

[略過觸發程序範例](#trigger---attributes)

### <a name="trigger---c-example"></a>觸發程序 - C# 範例

以下範例顯示當指定的資料庫和集合中具備插入項目或更新時，系統叫用的 [C# 函式](functions-dotnet-class-library.md)。

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;

namespace CosmosDBSamplesV2
{
    public static class CosmosTrigger
    {
        [FunctionName("CosmosTrigger")]
        public static void Run([CosmosDBTrigger(
            databaseName: "ToDoItems",
            collectionName: "Items",
            ConnectionStringSetting = "CosmosDBConnection",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]IReadOnlyList<Document> documents, 
            TraceWriter log)
        {
            if (documents != null && documents.Count > 0)
            {
                log.Info($"Documents modified: {documents.Count}");
                log.Info($"First document Id: {documents[0].Id}");
            }
        }
    }
}
```

[略過觸發程序範例](#trigger---attributes)

### <a name="trigger---c-script-example"></a>觸發程序 - C# 指令碼範例

下列範例示範 function.json 檔案中的 Cosmos DB 觸發程序繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 修改 Cosmos DB 記錄時，函式會寫入記錄訊息。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

以下是 C# 指令碼程式碼：
 
```cs 
    #r "Microsoft.Azure.Documents.Client"
    
    using System;
    using Microsoft.Azure.Documents;
    using System.Collections.Generic;
    

    public static void Run(IReadOnlyList<Document> documents, TraceWriter log)
    {
      log.Verbose("Documents modified " + documents.Count);
      log.Verbose("First document Id " + documents[0].Id);
    }
```

[略過觸發程序範例](#trigger---attributes)

### <a name="trigger---javascript-example"></a>觸發程序 - JavaScript 範例

下列範例示範的是使用繫結之 function.json 檔案，以及 [JavaScript 函式](functions-reference-node.md)中的 Cosmos DB 觸發程序繫結。 修改 Cosmos DB 記錄時，函式會寫入記錄訊息。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

以下是 JavaScript 程式碼：

```javascript
    module.exports = function (context, documents) {
      context.log('First document Id modified : ', documents[0].id);

      context.done();
    }
```

### <a name="trigger---java-example"></a>觸發程序 - Java 範例

下列範例示範的是使用繫結的 function.json 檔案，以及 [Java 函式](functions-reference-java.md)中的 Cosmos DB 觸發程序繫結。 當指定的資料庫和集合中有插入或更新時，就會涉及此函式。

```json
{
    "type": "cosmosDBTrigger",
    "name": "items",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "AzureCosmosDBConnection",
    "databaseName": "ToDoList",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": false
}
```

以下是 Java 程式碼：

```java
    @FunctionName("cosmosDBMonitor")
    public void cosmosDbProcessor(
        @CosmosDBTrigger(name = "items",
            databaseName = "ToDoList",
            collectionName = "Items",
            leaseCollectionName = "leases",
            reateLeaseCollectionIfNotExists = true,
            connectionStringSetting = "AzureCosmosDBConnection") String[] items,
            final ExecutionContext context ) {
                context.getLogger().info(items.length + "item(s) is/are changed.");
            }
```


在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值來自 Cosmos DB 的參數使用 `@CosmosDBTrigger` 註釋。  此註釋可以搭配原生 Java 類型、POJO 或使用選擇性 <T> 的可為 Null 值使用。 

## <a name="trigger---c-attributes"></a>觸發程序 - C# 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [CosmosDBTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/Trigger/CosmosDBTriggerAttribute.cs) 屬性。

屬性的建構函式可接受資料庫名稱和集合名稱。 如需這些設定及其他您可以設定之屬性的相關資訊，請參閱[觸發程序 - 組態](#trigger---configuration)。 以下是方法簽章中的 `CosmosDBTrigger` 屬性範例：

```csharp
    [FunctionName("DocumentUpdates")]
    public static void Run(
        [CosmosDBTrigger("database", "collection", ConnectionStringSetting = "myCosmosDB")]
    IReadOnlyList<Document> documents,
        TraceWriter log)
    {
        ...
    }
```

如需完整範例，請參閱[觸發程序 - C# 範例](#trigger---c-example)。


## <a name="trigger---configuration"></a>觸發程式 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `CosmosDBTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** || 必須設為 `cosmosDBTrigger`。 |
|**direction** || 必須設為 `in`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此參數。 |
|**name** || 函式程式碼中使用的變數名稱，代表有變更的文件清單。 | 
|**connectionStringSetting**|**ConnectionStringSetting** | 應用程式設定的名稱，包含用來連接到要監視之 Azure Cosmos DB 帳戶的連接字串。 |
|**databaseName**|**DatabaseName**  | 含有要監視之集合的 Azure Cosmos DB 資料庫名稱。 |
|**collectionName** |**CollectionName** | 要監視的集合名稱。 |
|**leaseConnectionStringSetting** | **LeaseConnectionStringSetting** | (選擇性) 應用程式設定的名稱，包含連至保存租用集合之服務的連接字串。 如果未設定，會使用 `connectionStringSetting` 值。 在入口網站中建立繫結時，會自動設定此參數。 租用集合的連接字串必須具有寫入權限。|
|**leaseDatabaseName** |**LeaseDatabaseName** | (選擇性) 保存用來儲存租用之集合的資料庫名稱。 如果未設定，會使用 `databaseName` 設定的值。 在入口網站中建立繫結時，會自動設定此參數。 |
|**leaseCollectionName** | **LeaseCollectionName** | (選擇性) 用來儲存租用的集合名稱。 如果未設定，會使用 `leases` 值。 |
|**createLeaseCollectionIfNotExists** | **CreateLeaseCollectionIfNotExists** | (選擇性) 設為 `true` 時，如果租用集合尚未存在，即會自動加以建立。 預設值為 `false`。 |
|**leasesCollectionThroughput**| **LeasesCollectionThroughput**| (選擇性) 定義要在建立租用集合時指派的要求單位數。 只有在將 `createLeaseCollectionIfNotExists` 設為 `true` 時才會使用此設定。 使用入口網站建立繫結時，會自動設定此參數。
|**leaseCollectionPrefix**| **LeaseCollectionPrefix**| (選擇性) 如果設定，將會為此函式對建立於租用集合中的租用加上前置詞，而有效地讓兩個不同的 Azure Functions 藉由使用不同的前置詞來共用相同的租用。
|**feedPollDelay**| **FeedPollDelay**| (選擇性) 如果設定，將會以毫秒為單位定義在目前所有的變更都清空後，每次輪詢分割區以了解摘要上是否有新變更時所要延遲的時間。 預設值為 5000 (5 秒)。
|**leaseAcquireInterval**| **LeaseAcquireInterval**| (選擇性) 如果設定，將會以毫秒為單位定義啟動工作以計算分割區是否平均分散到已知主機執行個體的間隔。 預設值為 13000 (13 秒)。
|**leaseExpirationInterval**| **LeaseExpirationInterval**| (選擇性) 如果設定，將會以毫秒為單位定義租用代表分割區的間隔。 未在此間隔內更新的租用將會過期，且分割區的擁有權會移轉給另一個執行個體。 預設值為 60000 (60 秒)。
|**leaseRenewInterval**| **LeaseRenewInterval**| (選擇性) 如果設定，將會以毫秒為單位定義目前由執行個體保有之分割區的所有租用所適用的更新間隔。 預設值為 17000 (17 秒)。
|**checkpointFrequency**| **CheckpointFrequency**| (選擇性) 如果設定，將會以毫秒為單位定義租用檢查點的間隔。 預設值為一律在成功呼叫函式之後。
|**maxItemsPerInvocation**| **MaxItemsPerInvocation**| (選擇性) 如果設定，將會自訂每個函式呼叫所接收的項目數量上限。

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>觸發程序 - 使用方式

觸發程序需要第二個集合，用來在分割區上儲存「租用」。 要監視的集合和包含租用的集合必須可供觸發程序用來運作。

>[!IMPORTANT]
> 如果針對同一個集合設定多個函式使用 Cosmos DB 觸發程序，則每個函式都應該使用專用的租用集合，或為每個函式指定不同的 `LeaseCollectionPrefix`。 否則，將只會觸發其中一個函式。 如需前置詞的相關資訊，請參閱[組態區段](#trigger---configuration)。

觸發程序不會指示是否更新或插入文件，只是提供文件本身。 如果您需要以不同方式處理更新和插入，您可以透過實作插入或更新的時間戳記欄位執行。

## <a name="input"></a>輸入

Azure Cosmos DB 輸入繫結會使用 SQL API 來擷取一或多個 Azure Cosmos DB 文件，並傳遞給函式的輸入參數。 您可以叫用函式的觸發程序作為基礎來判斷文件識別碼或查詢參數。 

## <a name="input---examples"></a>輸入 - 範例

請指定 ID 指，以讀取單一文件的方式參閱您慣用語言的範例：

* [C#](#input---c-examples)
* [C# 指令碼 (.csx)](#input---c-script-examples)
* [JavaScript](#input---javascript-examples)
* [F#](#input---f-examples)
* [Java](#input---java-examples)

[跳過輸入範例](#input---attributes)

### <a name="input---c-examples"></a>輸入 - C# 範例

本區段包含下列範例：

* [佇列觸發程序，從 JSON 中查閱識別碼](#queue-trigger-look-up-id-from-json-c)
* [HTTP 觸發程序，從查詢字串中查閱識別碼](#http-trigger-look-up-id-from-query-string-c)
* [HTTP 觸發程序，從路由資料中查閱識別碼](#http-trigger-look-up-id-from-route-data-c)
* [HTTP 觸發程序，使用 SqlQuery 從路由資料中查閱識別碼](#http-trigger-look-up-id-from-route-data-using-sqlquery-c)
* [HTTP 觸發程序，使用 SqlQuery 取得多個文件](#http-trigger-get-multiple-docs-using-sqlquery-c)
* [HTTP 觸發程序，使用 DocumentClient 取得多個文件](#http-trigger-get-multiple-docs-using-documentclient-c)

範例會參考簡單的 `ToDoItem` 類型：

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

[跳過輸入範例](#input---attributes)

#### <a name="queue-trigger-look-up-id-from-json-c"></a>佇列觸發程序，從 JSON 中查閱識別碼 (C#)

下列範例示範會顯示擷取單一文件的 [C# 函式](functions-dotnet-class-library.md)。 函式會由包含 JSON 物件的佇列訊息觸發。 佇列觸發程序會將 JSON 剖析至名為 `ToDoItemLookup` 的 JSON，其中包含要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItemLookup
    {
        public string ToDoItemId { get; set; }
    }
}
```

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromJSON
    {
        [FunctionName("DocByIdFromJSON")]
        public static void Run(
            [QueueTrigger("todoqueueforlookup")] ToDoItemLookup toDoItemLookup,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection", 
                Id = "{ToDoItemId}")]ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info($"C# Queue trigger function processed Id={toDoItemLookup?.ToDoItemId}");

            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
        }
    }
}
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-look-up-id-from-query-string-c"></a>HTTP 觸發程序，從查詢字串中查閱識別碼 (C#)

下列範例示範會顯示擷取單一文件的 [C# 函式](functions-dotnet-class-library.md)。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用查詢字串指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromQueryString
    {
        [FunctionName("DocByIdFromQueryString")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection", 
                Id = "{Query.id}")] ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
            return new OkResult();
        }
    }
}
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-c"></a>HTTP 觸發程序，從路由資料中查閱識別碼 (C#)

下列範例示範會顯示擷取單一文件的 [C# 函式](functions-dotnet-class-library.md)。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用路由資料指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromRouteData
    {
        [FunctionName("DocByIdFromRouteData")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", 
                Route = "todoitems/{id}")]HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection", 
                Id = "{id}")] ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
            return new OkResult();
        }
    }
}
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-using-sqlquery-c"></a>HTTP 觸發程序，使用 SqlQuery 從路由資料中查閱識別碼 (C#)

下列範例示範會顯示擷取單一文件的 [C# 函式](functions-dotnet-class-library.md)。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用路由資料指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。 

範例會顯示如何在 `SqlQuery` 參數中使用繫結運算式。 您可以如範例所示，將路由資料傳遞至 `SqlQuery` 參數，但目前[您無法傳遞查詢字串值](https://github.com/Azure/azure-functions-host/issues/2554#issuecomment-392084583)。


```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromRouteDataUsingSqlQuery
    {
        [FunctionName("DocByIdFromRouteDataUsingSqlQuery")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", 
                Route = "todoitems2/{id}")]HttpRequest req,
            [CosmosDB("ToDoItems", "Items", 
                ConnectionStringSetting = "CosmosDBConnection", 
                SqlQuery = "select * from ToDoItems r where r.id = {id}")]
                IEnumerable<ToDoItem> toDoItems,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.Info(toDoItem.Description);
            }
            return new OkResult();
        }
    }
}
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-sqlquery-c"></a>HTTP 觸發程序，使用 SqlQuery 取得多個文件 (C#)

下列範例示範會顯示擷取文件清單的 [C# 函式](functions-dotnet-class-library.md)。 函式是由 HTTP 要求所觸發。 查詢會在 `SqlQuery` 屬性內容中指定。

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;

namespace CosmosDBSamplesV2
{
    public static class DocsBySqlQuery
    {
        [FunctionName("DocsBySqlQuery")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection", 
                SqlQuery = "SELECT top 2 * FROM c order by c._ts desc")]
                IEnumerable<ToDoItem> toDoItems,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");
            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.Info(toDoItem.Description);
            }
            return new OkResult();
        }
    }
}

```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-documentclient-c"></a>HTTP 觸發程序，使用 DocumentClient 取得多個文件 (C#)

下列範例示範會顯示擷取文件清單的 [C# 函式](functions-dotnet-class-library.md)。 函式是由 HTTP 要求所觸發。 程式碼會使用由 Azure Cosmos DB 繫結提供的 `DocumentClient` 執行個體來讀取文件清單。 `DocumentClient` 執行個體也會用來進行寫入作業。

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System;
using System.Linq;
using System.Threading.Tasks;

namespace CosmosDBSamplesV2
{
    public static class DocsByUsingDocumentClient
    {
        [FunctionName("DocsByUsingDocumentClient")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", 
                Route = null)]HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            var searchterm = req.Query["searchterm"];
            if (string.IsNullOrWhiteSpace(searchterm))
            {
                return (ActionResult)new NotFoundResult();
            }

            Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");

            log.Info($"Searching for: {searchterm}");

            IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
                .Where(p => p.Description.Contains(searchterm))
                .AsDocumentQuery();

            while (query.HasMoreResults)
            {
                foreach (ToDoItem result in await query.ExecuteNextAsync())
                {
                    log.Info(result.Description);
                }
            }
            return new OkResult();
        }
    }
}
```

[跳過輸入範例](#input---attributes)

### <a name="input---c-script-examples"></a>輸入 - C# 指令碼範例

本區段包含下列範例：

* [佇列觸發程序，從字串中查閱識別碼](#queue-trigger-look-up-id-from-string-c-script)
* [佇列觸發程序，使用 SqlQuery 取得多個文件](#queue-trigger-get-multiple-docs-using-sqlquery-c-script)
* [HTTP 觸發程序，從查詢字串中查閱識別碼](#http-trigger-look-up-id-from-query-string-c-script)
* [HTTP 觸發程序，從路由資料中查閱識別碼](#http-trigger-look-up-id-from-route-data-c-script)
* [HTTP 觸發程序，使用 SqlQuery 取得多個文件](#http-trigger-get-multiple-docs-using-sqlquery-c-script)
* [HTTP 觸發程序，使用 DocumentClient 取得多個文件](#http-trigger-get-multiple-docs-using-documentclient-c-script)

HTTP 觸發程序範例會參考簡單的 `ToDoItem` 類型：

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

[跳過輸入範例](#input---attributes)

#### <a name="queue-trigger-look-up-id-from-string-c-script"></a>佇列觸發程序，從字串中查閱識別碼 (C# 指令碼)

下列範例示範 function.json 檔案中的 Cosmos DB 輸入繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 函式會讀取單一文件，並更新文件的文字值。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "inputDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger}",
    "partitionKey": "{partition key value}",
    "connectionStringSetting": "MyAccount_COSMOSDB",     
    "direction": "in"
}
```
[設定](#input---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```cs
    using System;

    // Change input document contents using Azure Cosmos DB input binding 
    public static void Run(string myQueueItem, dynamic inputDocument)
    {   
      inputDocument.text = "This has changed.";
    }
```

[跳過輸入範例](#input---attributes)

#### <a name="queue-trigger-get-multiple-docs-using-sqlquery-c-script"></a>佇列觸發程序，使用 SqlQuery 取得多個文件 (C# 指令碼)

下列範例顯示 *function.json* 檔案中的 Azure Cosmos DB 輸入繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 函式會使用佇列觸發程序來自訂查詢參數，以擷取 SQL 查詢所指定的多份文件。

佇列觸發程序會提供參數 `departmentId`。 `{ "departmentId" : "Finance" }` 的佇列訊息會傳回財務部門的所有記錄。 

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

[設定](#input---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```csharp
    public static void Run(QueuePayload myQueueItem, IEnumerable<dynamic> documents)
    {   
        foreach (var doc in documents)
        {
            // operate on each document
        }    
    }

    public class QueuePayload
    {
        public string departmentId { get; set; }
    }
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-look-up-id-from-query-string-c-script"></a>HTTP 觸發程序，從查詢字串中查閱識別碼 (C# 指令碼)

下列範例會顯示擷取單一文件的 [C# 指令碼函式](functions-reference-csharp.md)。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用查詢字串指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}"
    }
  ],
  "disabled": true
}
```

以下是 C# 指令碼程式碼：

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
         log.Info($"ToDo item not found");
    }
    else
    {
        log.Info($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-c-script"></a>HTTP 觸發程序，從路由資料中查閱識別碼 (C# 指令碼)

下列範例會顯示擷取單一文件的 [C# 指令碼函式](functions-reference-csharp.md)。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用路由資料指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}"
    }
  ],
  "disabled": false
}
```

以下是 C# 指令碼程式碼：

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
         log.Info($"ToDo item not found");
    }
    else
    {
        log.Info($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-sqlquery-c-script"></a>HTTP 觸發程序，使用 SqlQuery 取得多個文件 (C# 指令碼)

下列範例會顯示擷取文件清單的 [C# 指令碼函式](functions-reference-csharp.md)。 函式是由 HTTP 要求所觸發。 查詢會在 `SqlQuery` 屬性內容中指定。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "sqlQuery": "SELECT top 2 * FROM c order by c._ts desc"
    }
  ],
  "disabled": false
}
```

以下是 C# 指令碼程式碼：

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, IEnumerable<ToDoItem> toDoItems, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    foreach (ToDoItem toDoItem in toDoItems)
    {
        log.Info(toDoItem.Description);
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-documentclient-c-script"></a>HTTP 觸發程序，使用 DocumentClient 取得多個文件 (C# 指令碼)

下列範例會顯示擷取文件清單的 [C# 指令碼函式](functions-reference-csharp.md)。 函式是由 HTTP 要求所觸發。 程式碼會使用由 Azure Cosmos DB 繫結提供的 `DocumentClient` 執行個體來讀取文件清單。 `DocumentClient` 執行個體也會用來進行寫入作業。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "client",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "inout"
    }
  ],
  "disabled": false
}
```

以下是 C# 指令碼程式碼：

```cs
#r "Microsoft.Azure.Documents.Client"

using System.Net;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, DocumentClient client, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");
    string searchterm = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "searchterm", true) == 0)
        .Value;

    if (searchterm == null)
    {
        return req.CreateResponse(HttpStatusCode.NotFound);
    }

    log.Info($"Searching for word: {searchterm} using Uri: {collectionUri.ToString()}");
    IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
        .Where(p => p.Description.Contains(searchterm))
        .AsDocumentQuery();

    while (query.HasMoreResults)
    {
        foreach (ToDoItem result in await query.ExecuteNextAsync())
        {
            log.Info(result.Description);
        }
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[跳過輸入範例](#input---attributes)

### <a name="input---javascript-examples"></a>輸入 - JavaScript 範例

本章節包含下列範例，藉由從各種來源中指定識別碼值，讀取單一文件：

* [佇列觸發程序，從 JSON 中查閱識別碼](#queue-trigger-look-up-id-from-string-javascript)
* [HTTP 觸發程序，從查詢字串中查閱識別碼](#http-trigger-look-up-id-from-query-string-javascript)
* [HTTP 觸發程序，從路由資料中查閱識別碼](#http-trigger-look-up-id-from-route-data-javascript)
* [佇列觸發程序，使用 SqlQuery 取得多個文件](#queue-trigger-get-multiple-docs-using-sqlquery-javascript)

[跳過輸入範例](#input---attributes)

#### <a name="queue-trigger-look-up-id-from-json-javascript"></a>佇列觸發程序，從 JSON 中查閱識別碼 (JavaScript)

下列範例示範 function.json 檔案中的 Cosmos DB 輸入繫結，以及使用此繫結的 [JavaScript 指令碼函式](functions-reference-node.md)。 函式會讀取單一文件，並更新文件的文字值。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "inputDocumentIn",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger_payload_property}",
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",     
    "direction": "in"
},
{
    "name": "inputDocumentOut",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": false,
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```
[設定](#input---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
    // Change input document contents using Azure Cosmos DB input binding, using context.bindings.inputDocumentOut
    module.exports = function (context) {   
    context.bindings.inputDocumentOut = context.bindings.inputDocumentIn;
    context.bindings.inputDocumentOut.text = "This was updated!";
    context.done();
    };
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-look-up-id-from-query-string-javascript"></a>HTTP 觸發程序，從查詢字串中查閱識別碼 (JavaScript)

下列範例會顯示擷取單一文件的 [JavaScript 函式](functions-reference-node.md)。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用查詢字串指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}"
    }
  ],
  "disabled": true
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

[跳過輸入範例](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-javascript"></a>HTTP 觸發程序，從路由資料中查閱識別碼 (JavaScript)

下列範例會顯示擷取單一文件的 [JavaScript 函式](functions-reference-node.md)。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用查詢字串指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 `ToDoItem` 文件。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}"
    }
  ],
  "disabled": false
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

[跳過輸入範例](#input---attributes)

#### <a name="queue-trigger-get-multiple-docs-using-sqlquery-javascript"></a>佇列觸發程序，使用 SqlQuery 取得多個文件 (JavaScript)

下列範例顯示 *function.json* 檔案中的 Azure Cosmos DB 輸入繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。 函式會使用佇列觸發程序來自訂查詢參數，以擷取 SQL 查詢所指定的多份文件。

佇列觸發程序會提供參數 `departmentId`。 `{ "departmentId" : "Finance" }` 的佇列訊息會傳回財務部門的所有記錄。 

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

[設定](#input---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
    module.exports = function (context, input) {    
        var documents = context.bindings.documents;
        for (var i = 0; i < documents.length; i++) {
            var document = documents[i];
            // operate on each document
        }       
        context.done();
    };
```

[跳過輸入範例](#input---attributes)

<a name="infsharp"></a>

### <a name="input---f-examples"></a>輸入 - F# 範例

下列範例示範 function.json 檔案中的 Cosmos DB 觸發程序繫結，以及使用此繫結的 [F# 函式](functions-reference-fsharp.md)。 函式會讀取單一文件，並更新文件的文字值。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "inputDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger}",
    "connectionStringSetting": "MyAccount_COSMOSDB",     
    "direction": "in"
}
```

[設定](#input---configuration)章節會說明這些屬性。

以下是 F# 程式碼：

```fsharp
    (* Change input document contents using Azure Cosmos DB input binding *)
    open FSharp.Interop.Dynamic
    let Run(myQueueItem: string, inputDocument: obj) =
    inputDocument?text <- "This has changed."
```

此範例需要一個指定 `FSharp.Interop.Dynamic` 和 `Dynamitey` NuGet 相依性的 `project.json` 檔案︰

```json
{
    "frameworks": {
        "net46": {
            "dependencies": {
                "Dynamitey": "1.0.2",
                "FSharp.Interop.Dynamic": "3.0.0"
            }
        }
    }
}
```

若要新增 `project.json` 檔案，請參閱 [F# 封裝管理](functions-reference-fsharp.md#package)。

### <a name="input---java-examples"></a>輸入 - Java 範例

下列範例示範會擷取單一文件的 Java 函式。 函式會由 HTTP 要求觸發，該 HTTP 要求會使用查詢字串指定要查閱的識別碼。 該識別碼會用來從指定的資料庫和集合中，擷取 ToDoItem 文件。

以下是 Java 程式碼：

```java
@FunctionName("getItem")
public String cosmosDbQueryById(
    @HttpTrigger(name = "req",
                  methods = {HttpMethod.GET},
                  authLevel = AuthorizationLevel.ANONYMOUS) Optional<String> dummy,
    @CosmosDBInput(name = "database",
                      databaseName = "ToDoList",
                      collectionName = "Items",
                      leaseCollectionName = "",
                      id = "{Query.id}"
                      connectionStringSetting = "AzureCosmosDBConnection") Optional<String> item,
    final ExecutionContext context
 ) {
    return item.orElse("Not found");
 }
 ```

在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值來自 Cosmos DB 的函式參數使用 `@CosmosDBInput` 註釋。  此註釋可以搭配原生 Java 類型、POJO 或使用「選擇性」<T>的可為 Null 值一起使用。 

## <a name="input---attributes"></a>輸入 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [CosmosDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs) 屬性。

屬性的建構函式可接受資料庫名稱和集合名稱。 如需這些設定及其他您可以設定之屬性的相關資訊，請參閱[下列組態區段](#input---configuration)。 

## <a name="input---configuration"></a>輸入 - 組態

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `CosmosDB` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type**     || 必須設為 `cosmosDB`。        |
|**direction**     || 必須設為 `in`。         |
|**name**     || 代表函式中之文件的繫結參數名稱。  |
|**databaseName** |**DatabaseName** |包含文件的資料庫。        |
|**collectionName** |**CollectionName** | 包含文件的集合名稱。 |
|**id**    | **Id** | 要擷取之文件的識別碼。 此屬性支援[繫結運算式](functions-triggers-bindings.md#binding-expressions-and-patterns)。 請勿同時設定 **id** 和 **sqlQuery** 屬性。 如果您未設定其中一個，就會擷取整個集合。 |
|**sqlQuery**  |**SqlQuery**  | 用來擷取多份文件的 Azure Cosmos DB SQL 查詢。 屬性會支援執行階段繫結，如此範例所示：`SELECT * FROM c where c.departmentId = {departmentId}`。 請勿同時設定 **id** 和 **sqlQuery** 屬性。 如果您未設定其中一個，就會擷取整個集合。|
|**connectionStringSetting**     |**ConnectionStringSetting**|包含 Azure Cosmos DB 連接字串的應用程式設定名稱。        |
|**partitionKey**|**PartitionKey**|指定分割區索引鍵值進行查閱。 可能包含繫結參數。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="input---usage"></a>輸入 - 使用方式

在 C# 和 F# 函式中，當函式順利結束時，系統會自動保留透過具名輸入參數對輸入文件所做的任何變更。 

在 JavaScript 函數中，不會在函數結束時自動執行更新。 請改用 `context.bindings.<documentName>In` 和 `context.bindings.<documentName>Out` 來進行更新。 請參閱 [JavaScript 範例](#input---javascript-example)。

## <a name="output"></a>輸出

Azure Cosmos DB 輸出繫結可讓您使用 SQL API，將新的文件寫入 Azure Cosmos DB 資料庫。 

## <a name="output---examples"></a>輸出 - 範例

請參閱特定語言的範例：

* [C#](#output---c-examples)
* [C# 指令碼 (.csx)](#output---c-script-examples)
* [JavaScript](#output---javascript-examples)
* [F#](#output---f-examples)
* [Java](#output---java-example)

另請參閱使用 `DocumentClient` 的[輸入範例](#input---c-examples)。

[跳過輸出範例](#output---attributes)

### <a name="ouput---c-examples"></a>輸出 - C# 範例

本區段包含下列範例：

* 佇列觸發程序，寫入一份文件
* 佇列觸發程序，使用 IAsyncCollector 寫入文件

範例會參考簡單的 `ToDoItem` 類型：

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

[跳過輸出範例](#output---attributes)

#### <a name="queue-trigger-write-one-doc-c"></a>佇列觸發程序，寫入一份文件 (C#)

下列範例顯示 [C# 函式](functions-dotnet-class-library.md)，它使用佇列儲存體之訊息中提供的資料，將文件新增至資料庫。

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System;

namespace CosmosDBSamplesV2
{
    public static class WriteOneDoc
    {
        [FunctionName("WriteOneDoc")]
        public static void Run(
            [QueueTrigger("todoqueueforwrite")] string queueMessage,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]out dynamic document,
            TraceWriter log)
        {
            document = new { Description = queueMessage, id = Guid.NewGuid() };

            log.Info($"C# Queue trigger function inserted one row");
            log.Info($"Description={queueMessage}");
        }
    }
}
```

[跳過輸出範例](#output---attributes)

#### <a name="queue-trigger-write-docs-using-iasynccollector-c"></a>佇列觸發程序，使用 IAsyncCollector 寫入文件 (C#)

下列範例顯示 [C# 函式](functions-dotnet-class-library.md)，它會使用佇列訊息 JSON 中提供的資料，將文件集合新增至資料庫中。

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Threading.Tasks;

namespace CosmosDBSamplesV2
{
    public static class WriteDocsIAsyncCollector
    {
        [FunctionName("WriteDocsIAsyncCollector")]
        public static async Task Run(
            [QueueTrigger("todoqueueforwritemulti")] ToDoItem[] toDoItemsIn,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]
                IAsyncCollector<ToDoItem> toDoItemsOut,
            TraceWriter log)
        {
            log.Info($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

            foreach (ToDoItem toDoItem in toDoItemsIn)
            {
                log.Info($"Description={toDoItem.Description}");
                await toDoItemsOut.AddAsync(toDoItem);
            }
        }
    }
}
```

[跳過輸出範例](#output---attributes)

### <a name="output---c-script-examples"></a>輸出 - C# 指令碼範例

本區段包含下列範例：

* 佇列觸發程序，寫入一份文件
* 佇列觸發程序，使用 IAsyncCollector 寫入文件

[跳過輸出範例](#output---attributes)

#### <a name="queue-trigger-write-one-doc-c-script"></a>佇列觸發程序，寫入一份文件 (C# 指令碼)

下列範例顯示 *function.json* 檔案中的 Azure Cosmos DB 輸出繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式會使用可接收 JSON 佇列的佇列輸入繫結，其格式如下︰

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

此函式會針對每一筆記錄建立下列格式的 Azure Cosmos DB 文件：

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "employeeDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connectionStringSetting": "MyAccount_COSMOSDB",     
    "direction": "out"
}
```

[設定](#output---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```cs
    #r "Newtonsoft.Json"

    using Microsoft.Azure.WebJobs.Host;
    using Newtonsoft.Json.Linq;

    public static void Run(string myQueueItem, out object employeeDocument, TraceWriter log)
    {
      log.Info($"C# Queue trigger function processed: {myQueueItem}");

      dynamic employee = JObject.Parse(myQueueItem);

      employeeDocument = new {
        id = employee.name + "-" + employee.employeeId,
        name = employee.name,
        employeeId = employee.employeeId,
        address = employee.address
      };
    }
```

#### <a name="queue-trigger-write-docs-using-iasynccollector"></a>佇列觸發程序，使用 IAsyncCollector 寫入文件

如果要建立多個文件，您可以繫結至 `ICollector<T>` 或 `IAsyncCollector<T>`，`T` 表示其中一種支援的類型。

此範例會參考簡單的 `ToDoItem` 類型：

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

以下是 function.json 檔案：

```json
{
  "bindings": [
    {
      "name": "toDoItemsIn",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "todoqueueforwritemulti",
      "connectionStringSetting": "AzureWebJobsStorage"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItemsOut",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

以下是 C# 指令碼程式碼：

```cs
using System;

public static async Task Run(ToDoItem[] toDoItemsIn, IAsyncCollector<ToDoItem> toDoItemsOut, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

    foreach (ToDoItem toDoItem in toDoItemsIn)
    {
        log.Info($"Description={toDoItem.Description}");
        await toDoItemsOut.AddAsync(toDoItem);
    }
}
```

[跳過輸出範例](#output---attributes)

### <a name="output---javascript-examples"></a>輸出 - JavaScript 範例

下列範例顯示 function.json 檔案中的 Azure Cosmos DB 輸出繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式會使用可接收 JSON 佇列的佇列輸入繫結，其格式如下︰

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

此函式會針對每一筆記錄建立下列格式的 Azure Cosmos DB 文件：

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "employeeDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connectionStringSetting": "MyAccount_COSMOSDB",     
    "direction": "out"
}
```

[設定](#output---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
    module.exports = function (context) {

      context.bindings.employeeDocument = JSON.stringify({ 
        id: context.bindings.myQueueItem.name + "-" + context.bindings.myQueueItem.employeeId,
        name: context.bindings.myQueueItem.name,
        employeeId: context.bindings.myQueueItem.employeeId,
        address: context.bindings.myQueueItem.address
      });

      context.done();
    };
```

[跳過輸出範例](#output---attributes)

### <a name="output---f-examples"></a>輸出 - F# 範例

下列範例顯示 function.json 檔案中的 Azure Cosmos DB 輸出繫結，以及使用此繫結的 [F# 函式](functions-reference-fsharp.md)。 此函式會使用可接收 JSON 佇列的佇列輸入繫結，其格式如下︰

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

此函式會針對每一筆記錄建立下列格式的 Azure Cosmos DB 文件：

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "name": "employeeDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connectionStringSetting": "MyAccount_COSMOSDB",     
    "direction": "out"
}
```
[設定](#output---configuration)章節會說明這些屬性。

以下是 F# 程式碼：

```fsharp
    open FSharp.Interop.Dynamic
    open Newtonsoft.Json

    type Employee = {
      id: string
      name: string
      employeeId: string
      address: string
    }

    let Run(myQueueItem: string, employeeDocument: byref<obj>, log: TraceWriter) =
      log.Info(sprintf "F# Queue trigger function processed: %s" myQueueItem)
      let employee = JObject.Parse(myQueueItem)
      employeeDocument <-
        { id = sprintf "%s-%s" employee?name employee?employeeId
          name = employee?name
          employeeId = employee?employeeId
          address = employee?address }
```

此範例需要一個指定 `FSharp.Interop.Dynamic` 和 `Dynamitey` NuGet 相依性的 `project.json` 檔案︰

```json
{
    "frameworks": {
        "net46": {
          "dependencies": {
            "Dynamitey": "1.0.2",
            "FSharp.Interop.Dynamic": "3.0.0"
           }
        }
    }
}
```

若要新增 `project.json` 檔案，請參閱 [F# 封裝管理](functions-reference-fsharp.md#package)。

## <a name="output---java-examples"></a>輸出 - Java 範例

下列範例示範的 Java 函式使用佇列儲存體中的訊息資料，將文件新增至資料庫。

```java
@FunctionName("getItem")
@CosmosDBOutput(name = "database", databaseName = "ToDoList", collectionName = "Items", connectionStringSetting = "AzureCosmosDBConnection")
public String cosmosDbQueryById(
     @QueueTrigger(name = "msg", queueName = "myqueue-items", connection = "AzureWebJobsStorage") String message,
     final ExecutionContext context
)  {
     return "{ id: " + System.currentTimeMillis() + ", Description: " + message + " }";
   }
```

在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對即將寫入至 Cosmos DB 的參數使用 `@CosmosDBOutput` 註釋。  註解參數類型應該是 OutputBinding<T>，其中 T 是原生 Java 類型或 POJO。


## <a name="output---attributes"></a>輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [CosmosDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/master/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs) 屬性。

屬性的建構函式可接受資料庫名稱和集合名稱。 如需這些設定及其他您可以設定之屬性的相關資訊，請參閱[輸出 - 組態](#output---configuration)。 以下是方法簽章中的 `CosmosDB` 屬性範例：

```csharp
    [FunctionName("QueueToDocDB")]        
    public static void Run(
        [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
        [CosmosDB("ToDoList", "Items", Id = "id", ConnectionStringSetting = "myCosmosDB")] out dynamic document)
    {
        ...
    }
```

如需完整範例，請參閱[輸出 - C# 範例](#output---c-example)。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `CosmosDB` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type**     || 必須設為 `cosmosDB`。        |
|**direction**     || 必須設為 `out`。         |
|**name**     || 代表函式中之文件的繫結參數名稱。  |
|**databaseName** | **DatabaseName**|包含其中將建立文件之集合的資料庫。     |
|**collectionName** |**CollectionName**  | 包含其中將建立文件之集合的名稱。 |
|**createIfNotExists**  |**CreateIfNotExists**    | 一個布林值，用來指出當集合不存在時，是否要建立集合。 預設是 false，因為會使用保留的輸送量來建立新集合，可能會涉及成本。 如需詳細資訊，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/cosmos-db/)。  |
|**partitionKey**|**PartitionKey** |當 `CreateIfNotExists` 為 true 時，定義所建立集合的分割區索引鍵路徑。|
|**collectionThroughput**|**CollectionThroughput**| 當 `CreateIfNotExists` 為 true 時，定義所建立集合的[輸送量](../cosmos-db/set-throughput.md)。|
|**connectionStringSetting**    |**ConnectionStringSetting** |包含 Azure Cosmos DB 連接字串的應用程式設定名稱。        |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>輸出 - 使用方式

根據預設，當您在函式中寫入輸出參數時，會在資料庫中建立文件。 這份文件已自動產生 GUID 作為文件識別碼。 您可以藉由在傳遞至輸出參數的 JSON 物件中指定 `id` 屬性，來指定輸出文件的文件識別碼。 

> [!Note]  
> 當您指定現有文件的識別碼時，新的輸出文件會覆寫現有文件。 

## <a name="exceptions-and-return-codes"></a>例外狀況和傳回碼

| 繫結 | 參考 |
|---|---|
| CosmosDB | [CosmosDB 錯誤碼](https://docs.microsoft.com/rest/api/cosmos-db/http-status-codes-for-cosmosdb) |

## <a name="next-steps"></a>後續步驟

* [深入了解使用 Cosmos DB 的無伺服器資料庫](..\cosmos-db\serverless-computing-database.md)
* [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

<!---
> [!div class="nextstepaction"]
> [Go to a quickstart that uses a Cosmos DB trigger](functions-create-cosmos-db-triggered-function.md)
--->
