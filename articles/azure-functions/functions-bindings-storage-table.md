---
title: Azure Functions 的 Azure 資料表儲存體繫結
description: 了解如何在 Azure Functions 中使用 Azure 資料表繫結。
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
ms.openlocfilehash: 5dcd37b4aa8ddd976a463ac82b470b169a91c15e
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666836"
---
# <a name="azure-table-storage-bindings-for-azure-functions"></a>Azure Functions 的 Azure 資料表儲存體繫結

本文說明如何在 Azure Functions 中使用 Azure 資料表儲存體繫結。 Azure Functions 支援 Azure 資料表儲存體的輸入和輸出繫結。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs](http://www.nuget.org/packages/Microsoft.Azure.WebJobs) NuGet 套件 2.x 版中提供資料表儲存體繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Table) GitHub 存放庫中。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) NuGet 套件 3.x 版中提供資料表儲存體繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="input"></a>輸入

使用 Azure 資料表儲存體輸入繫結以讀取 Azure 儲存體帳戶中的資料表。

## <a name="input---example"></a>輸入 - 範例

請參閱特定語言的範例：

* [C# 讀取單一實體](#input---c-example---one-entity)
* [C# 繫結至 IQueryable](#input---c-example---iqueryable)
* [C# 繫結至 CloudTable](#input---c-example---cloudtable)
* [C# 指令碼讀取單一實體](#input---c-script-example---one-entity)
* [C# 指令碼繫結至 IQueryable](#input---c-script-example---iqueryable)
* [C# 指令碼繫結至 CloudTable](#input---c-script-example---cloudtable)
* [F#](#input---f-example)
* [JavaScript](#input---javascript-example)
* [Java](#input---java-example)

### <a name="input---c-example---one-entity"></a>輸入 - C# 範例 - 一個實體

下列範例示範可讀取單一資料表資料列的 [C# 函式](functions-dotnet-class-library.md)。 

列索引鍵值 "{queueTrigger}" 表示資料列索引鍵來自佇列訊息字串。

```csharp
public class TableStorage
{
    public class MyPoco
    {
        public string PartitionKey { get; set; }
        public string RowKey { get; set; }
        public string Text { get; set; }
    }

    [FunctionName("TableInput")]
    public static void TableInput(
        [QueueTrigger("table-items")] string input, 
        [Table("MyTable", "MyPartition", "{queueTrigger}")] MyPoco poco, 
        TraceWriter log)
    {
        log.Info($"PK={poco.PartitionKey}, RK={poco.RowKey}, Text={poco.Text}");
    }
}
```

### <a name="input---c-example---iqueryable"></a>輸入 - C# 範例 - IQueryable

下列範例示範可讀取多個資料表資料列的 [C# 函式](functions-dotnet-class-library.md)。 請注意，`MyPoco` 類別衍生自 `TableEntity`。

```csharp
public class TableStorage
{
    public class MyPoco : TableEntity
    {
        public string Text { get; set; }
    }

    [FunctionName("TableInput")]
    public static void TableInput(
        [QueueTrigger("table-items")] string input, 
        [Table("MyTable", "MyPartition")] IQueryable<MyPoco> pocos, 
        TraceWriter log)
    {
        foreach (MyPoco poco in pocos)
        {
            log.Info($"PK={poco.PartitionKey}, RK={poco.RowKey}, Text={poco.Text}");
        }
    }
}
```

### <a name="input---c-example---cloudtable"></a>輸入 - C# 範例 - CloudTable

[Functions V2 執行階段](functions-versions.md)不支援 `IQueryable`。 替代方式是使用 Azure 儲存體 SDK，藉此利用 `CloudTable` 方法參數來讀取資料表。 以下為查詢 Azure Functions 記錄資料表的 2.x 函式範例：

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.WindowsAzure.Storage.Table;
using System;
using System.Threading.Tasks;

namespace FunctionAppCloudTable2
{
    public class LogEntity : TableEntity
    {
        public string OriginalName { get; set; }
    }
    public static class CloudTableDemo
    {
        [FunctionName("CloudTableDemo")]
        public static async Task Run(
            [TimerTrigger("0 */1 * * * *")] TimerInfo myTimer, 
            [Table("AzureWebJobsHostLogscommon")] CloudTable cloudTable,
            TraceWriter log)
        {
            log.Info($"C# Timer trigger function executed at: {DateTime.Now}");

            TableQuery<LogEntity> rangeQuery = new TableQuery<LogEntity>().Where(
                TableQuery.CombineFilters(
                    TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, 
                        "FD2"),
                    TableOperators.And,
                    TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, 
                        "t")));

            // Execute the query and loop through the results
            foreach (LogEntity entity in 
                await cloudTable.ExecuteQuerySegmentedAsync(rangeQuery, null))
            {
                log.Info(
                    $"{entity.PartitionKey}\t{entity.RowKey}\t{entity.Timestamp}\t{entity.OriginalName}");
            }
        }
    }
}
```

如需如何使用 CloudTable 的詳細資訊，請參閱[開始使用 Azure 資料表儲存體](../cosmos-db/table-storage-how-to-use-dotnet.md)。

如果您嘗試繫結至 `CloudTable`，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

### <a name="input---c-script-example---one-entity"></a>輸入 - C# 指令碼範例 - 一個實體

下列範例所示範的是使用繫結之 *function.json* 檔案，以及 [C# 指令碼](functions-reference-csharp.md)程式碼中的資料表輸入繫結。 此函式會使用佇列觸發程序來讀取單一資料表列。 

*function.json* 檔案會指定 `partitionKey` 和 `rowKey`。 `rowKey` 值 "{queueTrigger}" 表示資料列索引鍵來自佇列訊息字串。

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
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

[設定](#input---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```csharp
public static void Run(string myQueueItem, Person personEntity, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    log.Info($"Name in Person entity: {personEntity.Name}");
}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}
```

### <a name="input---c-script-example---iqueryable"></a>輸入 - C# 指令碼範例 - IQueryable

下列範例所示範的是使用繫結之 *function.json* 檔案，以及 [C# 指令碼](functions-reference-csharp.md)程式碼中的資料表輸入繫結。 此函式會讀取佇列訊息中指定之分割區索引鍵的實體。

以下是 *function.json* 檔案：

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
      "name": "tableBinding",
      "type": "table",
      "connection": "MyStorageConnectionAppSetting",
      "tableName": "Person",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

[設定](#input---configuration)章節會說明這些屬性。

C# 指令碼程式碼會新增對「Azure 儲存體 SDK」的參考，讓實體類型可以衍生自 `TableEntity`：

```csharp
#r "Microsoft.WindowsAzure.Storage"
using Microsoft.WindowsAzure.Storage.Table;

public static void Run(string myQueueItem, IQueryable<Person> tableBinding, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    foreach (Person person in tableBinding.Where(p => p.PartitionKey == myQueueItem).ToList())
    {
        log.Info($"Name: {person.Name}");
    }
}

public class Person : TableEntity
{
    public string Name { get; set; }
}
```

### <a name="input---c-script-example---cloudtable"></a>輸入 - C# 指令碼範例 - CloudTable

[Functions V2 執行階段](functions-versions.md)不支援 `IQueryable`。 替代方式是使用 Azure 儲存體 SDK，藉此利用 `CloudTable` 方法參數來讀取資料表。 以下為查詢 Azure Functions 記錄資料表的 2.x 函式範例：

```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 */1 * * * *"
    },
    {
      "name": "cloudTable",
      "type": "table",
      "connection": "AzureWebJobsStorage",
      "tableName": "AzureWebJobsHostLogscommon",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

```csharp
#r "Microsoft.WindowsAzure.Storage"
using Microsoft.WindowsAzure.Storage.Table;
using System;
using System.Threading.Tasks;

public static async Task Run(TimerInfo myTimer, CloudTable cloudTable, TraceWriter log)
{
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");

    TableQuery<LogEntity> rangeQuery = new TableQuery<LogEntity>().Where(
    TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, 
            "FD2"),
        TableOperators.And,
        TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, 
            "a")));

    // Execute the query and loop through the results
    foreach (LogEntity entity in 
    await cloudTable.ExecuteQuerySegmentedAsync(rangeQuery, null))
    {
        log.Info(
            $"{entity.PartitionKey}\t{entity.RowKey}\t{entity.Timestamp}\t{entity.OriginalName}");
    }
}

public class LogEntity : TableEntity
{
    public string OriginalName { get; set; }
}
```

如需如何使用 CloudTable 的詳細資訊，請參閱[開始使用 Azure 資料表儲存體](../cosmos-db/table-storage-how-to-use-dotnet.md)。

如果您嘗試繫結至 `CloudTable`，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

### <a name="input---f-example"></a>輸入 - F# 範例

下列範例所示範的是使用繫結之 *function.json* 檔案和 [F# 指令碼](functions-reference-fsharp.md)程式碼中的資料表輸入繫結。 此函式會使用佇列觸發程序來讀取單一資料表列。 

*function.json* 檔案會指定 `partitionKey` 和 `rowKey`。 `rowKey` 值 "{queueTrigger}" 表示資料列索引鍵來自佇列訊息字串。

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
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

[設定](#input---configuration)章節會說明這些屬性。

以下是 F# 程式碼：

```fsharp
[<CLIMutable>]
type Person = {
  PartitionKey: string
  RowKey: string
  Name: string
}

let Run(myQueueItem: string, personEntity: Person) =
    log.Info(sprintf "F# Queue trigger function processed: %s" myQueueItem)
    log.Info(sprintf "Name in Person entity: %s" personEntity.Name)
```

### <a name="input---javascript-example"></a>輸入 - JavaScript 範例

下列範例所示範的是使用繫結的 *function.json* 檔案和 [JavaScript 指令碼](functions-reference-node.md)程式碼中的資料表輸入繫結。 此函式會使用佇列觸發程序來讀取單一資料表列。 

*function.json* 檔案會指定 `partitionKey` 和 `rowKey`。 `rowKey` 值 "{queueTrigger}" 表示資料列索引鍵來自佇列訊息字串。

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
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

[設定](#input---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);
    context.log('Person entity name: ' + context.bindings.personEntity.Name);
    context.done();
};
```

### <a name="input---java-example"></a>輸入 - Java 範例

下列範例顯示由 HTTP 觸發的函式，其會傳回資料表儲存體中指定資料分割內的項目總計數。

```java
@FunctionName("getallcount")
public int run(
   @HttpTrigger(name = "req",
                 methods = {"get"},
                 authLevel = AuthorizationLevel.ANONYMOUS) Object dummyShouldNotBeUsed,
   @TableInput(name = "items",
                tableName = "mytablename",  partitionKey = "myparkey",
                connection = "myconnvarname") MyItem[] items
) {
    return items.length;
}
```


## <a name="input---attributes"></a>輸入 - 屬性
 
在 [C# 類別庫](functions-dotnet-class-library.md)中，使用下列屬性來設定資料表輸入繫結：

* [TableAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/TableAttribute.cs)

  屬性的建構函式採用資料表名稱、分割區索引鍵以及資料列索引鍵。 它可以用於 out 參數或函式的傳回值，如下列範例所示：

  ```csharp
  [FunctionName("TableInput")]
  public static void Run(
      [QueueTrigger("table-items")] string input, 
      [Table("MyTable", "Http", "{queueTrigger}")] MyPoco poco, 
      TraceWriter log)
  {
      ...
  }
  ```

  您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

  ```csharp
  [FunctionName("TableInput")]
  public static void Run(
      [QueueTrigger("table-items")] string input, 
      [Table("MyTable", "Http", "{queueTrigger}", Connection = "StorageConnectionAppSetting")] MyPoco poco, 
      TraceWriter log)
  {
      ...
  }
  ```

  如需完整範例，請參閱[輸入 - C# 範例](#input---c-example)。

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  提供另一種方式來指定要使用的儲存體帳戶。 建構函式採用的是內含儲存體連接字串的應用程式設定名稱。 屬性可以套用在參數、方法或類別層級。 下列範例所示範的是類別層級與方法層級：

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("TableInput")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ...
  }
  ```

要使用的儲存體帳戶按以下順序決定：

* `Table` 屬性的 `Connection` 內容。
* `StorageAccount` 屬性套用至與 `Table` 屬性相同的參數。
* `StorageAccount` 屬性套用至該函式。
* `StorageAccount` 屬性套用至該類別。
* 函數應用程式 (「AzureWebJobsStorage」應用程式設定) 的預設儲存體帳戶。

## <a name="input---java-annotations"></a>輸入 - Java 註釋

在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值來自資料表儲存體的參數使用 `@TableInput` 註釋。  此註釋可以搭配原生 Java 類型、POJO 或使用「選擇性」<T>的可為 Null 值一起使用。 

## <a name="input---configuration"></a>輸入 - 組態

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Table` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 `table`。 當您在 Azure 入口網站中建立繫結時，會自動設定此屬性。|
|**direction** | n/a | 必須設為 `in`。 當您在 Azure 入口網站中建立繫結時，會自動設定此屬性。 |
|**name** | n/a | 代表函式程式碼中的資料表或實體的變數名稱。 | 
|**tableName** | **TableName** | 資料表的名稱。| 
|**partitionKey** | **PartitionKey** |選用。 要讀取之資料表實體的分割區索引鍵。 若要了解如何使用該屬性，請參閱[使用方式](#input---usage)一節。| 
|**rowKey** |**RowKey** | 選用。 要讀取之資料表實體的資料列索引鍵。 若要了解如何使用該屬性，請參閱[使用方式](#input---usage)一節。| 
|**take** |**Take** | 選用。 要在 JavaScript 中讀取的實體數目上限。 若要了解如何使用該屬性，請參閱[使用方式](#input---usage)一節。| 
|**filter** |**Filter** | 選用。 用於在 JavaScript 中輸入資料表的 OData 篩選運算式。 若要了解如何使用該屬性，請參閱[使用方式](#input---usage)一節。| 
|**連接** |**連接** | 應用程式設定的名稱包含要用於此繫結的儲存體連接字串。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為「MyStorage」，則函式執行階段會尋找名稱為「AzureWebJobsMyStorage」的應用程式設定。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="input---usage"></a>輸入 - 使用方式

資料表儲存體輸入繫結支援下列案例：

* **讀取 C# 或 C# 指令碼中的一個資料列**

  請設定 `partitionKey` 和 `rowKey`。 使用方法參數 `T <paramName>` 存取資料表資料。 在 C# 指令碼中，`paramName` 是 *function.json* 之 `name` 屬性中指定的值。 `T` 通常是實作 `ITableEntity` 的型別或衍生自 `TableEntity`。 此案例中不使用 `filter` 和 `take` 屬性。 

* **讀取 C# 或 C# 指令碼中的一或多個資料列**

  使用方法參數 `IQueryable<T> <paramName>` 存取資料表資料。 在 C# 指令碼中，`paramName` 是 *function.json* 之 `name` 屬性中指定的值。 `T` 通常是實作 `ITableEntity` 的型別或衍生自 `TableEntity`。 您可以使用 `IQueryable` 方法，以執行任何所需的篩選條件。 此案例中不使用 `partitionKey`、`rowKey`、`filter` 和 `take` 屬性。  

  > [!NOTE]
  > [Functions V2 執行階段](functions-versions.md)不支援 `IQueryable`。 替代方式是使用 Azure 儲存體 SDK，藉以[使用 CloudTable paramName 方法參數](https://stackoverflow.com/questions/48922485/binding-to-table-storage-in-v2-azure-functions-using-cloudtable) \(英文\) 來讀取資料表。 如果您嘗試繫結至 `CloudTable`，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

* **讀取 JavaScript 中的一或多個資料列**

  請設定 `filter` 和 `take` 屬性。 請勿設定 `partitionKey` 或 `rowKey`。 使用 `context.bindings.<name>` 來存取輸入資料表實體 (或多個實體)。 還原序列化的物件具有 `RowKey` 和 `PartitionKey` 屬性。

## <a name="output"></a>輸出

使用 Azure 資料表儲存體輸出繫結以寫入 Azure 儲存體帳戶中的資料表。

> [!NOTE]
> 此輸出繫結不支援更新現有的實體。 [從 Azure 儲存體 SDK](https://docs.microsoft.com/azure/cosmos-db/table-storage-how-to-use-dotnet#replace-an-entity) 使用 `TableOperation.Replace` 作業來更新現有的實體。   

## <a name="output---example"></a>輸出 - 範例

請參閱特定語言的範例：

* [C#](#output---c-example)
* [C# 指令碼 (.csx)](#output---c-script-example)
* [F#](#output---f-example)
* [JavaScript](#output---javascript-example)

### <a name="output---c-example"></a>輸出 - C# 範例

下列範例示範 [C# 函式](functions-dotnet-class-library.md)，該函式會使用 HTTP 觸發程序來寫入單一資料表資料列。 

```csharp
public class TableStorage
{
    public class MyPoco
    {
        public string PartitionKey { get; set; }
        public string RowKey { get; set; }
        public string Text { get; set; }
    }

    [FunctionName("TableOutput")]
    [return: Table("MyTable")]
    public static MyPoco TableOutput([HttpTrigger] dynamic input, TraceWriter log)
    {
        log.Info($"C# http trigger function processed: {input.Text}");
        return new MyPoco { PartitionKey = "Http", RowKey = Guid.NewGuid().ToString(), Text = input.Text };
    }
}
```

### <a name="output---c-script-example"></a>輸出 - C# 指令碼範例

下列範例所示範的是使用繫結之 *function.json* 檔案，以及 [C# 指令碼](functions-reference-csharp.md)程式碼中的資料表輸出繫結。 函式會寫入多個資料表實體。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[設定](#output---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```csharp
public static void Run(string input, ICollector<Person> tableBinding, TraceWriter log)
{
    for (int i = 1; i < 10; i++)
        {
            log.Info($"Adding Person entity {i}");
            tableBinding.Add(
                new Person() { 
                    PartitionKey = "Test", 
                    RowKey = i.ToString(), 
                    Name = "Name" + i.ToString() }
                );
        }

}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}

```

### <a name="output---f-example"></a>輸出 - F# 範例

下列範例所示範的是使用繫結之 *function.json* 檔案，以及 [F# 指令碼](functions-reference-fsharp.md)程式碼中的資料表輸出繫結。 函式會寫入多個資料表實體。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[設定](#output---configuration)章節會說明這些屬性。

以下是 F# 程式碼：

```fsharp
[<CLIMutable>]
type Person = {
  PartitionKey: string
  RowKey: string
  Name: string
}

let Run(input: string, tableBinding: ICollector<Person>, log: TraceWriter) =
    for i = 1 to 10 do
        log.Info(sprintf "Adding Person entity %d" i)
        tableBinding.Add(
            { PartitionKey = "Test"
              RowKey = i.ToString()
              Name = "Name" + i.ToString() })
```

### <a name="output---javascript-example"></a>輸出 - JavaScript 範例

下列範例所示範的是使用繫結之 *function.json* 檔案，以及 [JavaScript 函式](functions-reference-node.md)中的資料表輸出繫結。 函式會寫入多個資料表實體。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[設定](#output---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context) {

    context.bindings.tableBinding = [];

    for (var i = 1; i < 10; i++) {
        context.bindings.tableBinding.push({
            PartitionKey: "Test",
            RowKey: i.toString(),
            Name: "Name " + i
        });
    }
    
    context.done();
};
```

## <a name="output---attributes"></a>輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [TableAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/TableAttribute.cs)。

屬性的建構函式採用資料表名稱。 它可以用於 `out` 參數或函式的傳回值，如下列範例所示：

```csharp
[FunctionName("TableOutput")]
[return: Table("MyTable")]
public static MyPoco TableOutput(
    [HttpTrigger] dynamic input, 
    TraceWriter log)
{
    ...
}
```

您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

```csharp
[FunctionName("TableOutput")]
[return: Table("MyTable", Connection = "StorageConnectionAppSetting")]
public static MyPoco TableOutput(
    [HttpTrigger] dynamic input, 
    TraceWriter log)
{
    ...
}
```

如需完整範例，請參閱[輸出 - C# 範例](#output---c-example)。

您可以使用 `StorageAccount` 屬性來指定類別、方法或參數層級的儲存體帳戶。 如需詳細資訊，請參閱[輸入 - 屬性](#input---attributes)。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Table` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設為 `table`。 當您在 Azure 入口網站中建立繫結時，會自動設定此屬性。|
|**direction** | n/a | 必須設為 `out`。 當您在 Azure 入口網站中建立繫結時，會自動設定此屬性。 |
|**name** | n/a | 函式程式碼中所使用的變數名稱，代表資料表或實體。 設為 `$return` 以參考函式傳回值。| 
|**tableName** |**TableName** | 資料表的名稱。| 
|**partitionKey** |**PartitionKey** | 要寫入之資料表實體的分割區索引鍵。 若要了解如何使用該屬性，請參閱[使用方式](#output---usage)一節。| 
|**rowKey** |**RowKey** | 要寫入之資料表實體的資料列索引鍵。 若要了解如何使用該屬性，請參閱[使用方式](#output---usage)一節。| 
|**連接** |**連接** | 應用程式設定的名稱包含要用於此繫結的儲存體連接字串。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如，如果您將 `connection` 設定為「MyStorage」，則函式執行階段會尋找名稱為「AzureWebJobsMyStorage」的應用程式設定。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>輸出 - 使用方式

資料表儲存體輸出繫結支援下列案例：

* **以任何語言寫入單一資料列**

  在 C# 和 C# 指令碼中，使用方法參數 (例如 `out T paramName`) 或函式傳回值，藉此存取輸出資料表實體。 在 C# 指令碼中，`paramName` 是 *function.json* 之 `name` 屬性中指定的值。 如果分割區索引鍵和資料列索引鍵是由 *function.json* 檔案或 `Table` 屬性提供，`T` 可以是任何可序列化的型別。 否則，`T` 必須包含 `PartitionKey` 和 `RowKey` 屬性的型別。 在此案例中，`T` 通常會實作 `ITableEntity` 或衍生自 `TableEntity`，但不一定如此。

* **寫入 C# 或 C# 指令碼中的一或多個資料列**

  在 C# 和 C# 指令碼中，使用方法參數 `ICollector<T> paramName` 或 `IAsyncCollector<T> paramName` 存取輸出資料表實體。 在 C# 指令碼中，`paramName` 是 *function.json* 之 `name` 屬性中指定的值。 `T` 指定了您想要新增之實體的結構描述。 一般而言，`T` 會衍生自 `TableEntity` 或實作 `ITableEntity`，但不一定如此。 *function.json* 中的分割區索引鍵或資料列索引鍵值，或 `Table` 屬性建構函式不會在此案例中使用。

  替代方式是藉由使用 Azure 儲存體 SDK，利用 `CloudTable` 方法參數來寫入資料表。 如果您嘗試繫結至 `CloudTable`，並出現錯誤訊息，請確定您已參考[正確的儲存體 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。 如需繫結至 `CloudTable` 的程式碼範例，請參閱本文稍早之適用於 [C#](#input---c-example---cloudtable) 或 [C# 指令碼](#input---c-script-example---cloudtable)的輸入繫結範例。

* **寫入 JavaScript 中的一或多個資料列**

  在 JavaScript 函式中，會使用 `context.bindings.<name>` 來存取資料表輸出。

## <a name="exceptions-and-return-codes"></a>例外狀況和傳回碼

| 繫結 | 參考 |
|---|---|
| 資料表 | [資料表錯誤碼](https://docs.microsoft.com/rest/api/storageservices/fileservices/table-service-error-codes) |
| Bob、資料表、佇列 | [儲存體錯誤碼](https://docs.microsoft.com/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Bob、資料表、佇列 | [疑難排解](https://docs.microsoft.com/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
