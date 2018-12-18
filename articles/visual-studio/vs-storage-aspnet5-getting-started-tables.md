---
title: 如何開始使用資料表儲存體和 Visual Studio 已連接服務 (ASP.NET Core) | Microsoft Docs
description: 在使用 Visual Studio 已連接服務連接到儲存體帳戶之後，如何在 Visual Studio 的 ASP.NET Core 專案中開始使用 Azure 資料表儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: c3c451d1-71ff-4222-a348-c41c98a02b85
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 11/14/2017
ms.author: ghogen
ms.openlocfilehash: 0cb2e04d788bce2d3a5f6bc46632b9ae18b6467f
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39112911"
---
# <a name="how-to-get-started-with-azure-table-storage-and-visual-studio-connected-services"></a>開始使用 Azure 資料表儲存體和 Visual Studio 連線的服務

[!INCLUDE [storage-try-azure-tools-tables](../../includes/storage-try-azure-tools-tables.md)]

本文說明如何在您使用 Visual Studio 的**已連線的服務**功能建立或參考 ASP.NET Core 專案中的 Azure 儲存體帳戶之後，開始在 Visual Studio 使用 Azure 資料表儲存體。 **已連線的服務**作業會安裝適當的 NuGet 套件，以存取專案中的 Azure 儲存體，並將儲存體帳戶的連接字串新增至您的專案設定檔。 (如需 Azure 儲存體的一般資訊，請參閱[儲存體文件](https://azure.microsoft.com/documentation/services/storage/)。)

Azure 資料表儲存體服務可讓您儲存大量的結構化資料。 此服務是一個 NoSQL 資料存放區，接受來自 Azure 雲端內外經過驗證的呼叫。 Azure 資料表很適合儲存結構化、非關聯式資料。 如需其他有關使用 Azure 資料表儲存體的一般資訊，請參閱 [以 .NET 開始使用 Azure 資料表儲存體](../storage/storage-dotnet-how-to-use-tables.md)。

若要開始，請先在儲存體帳戶中建立 Azure 資料表。 本文章將接著示範如何以 C# 建立資料表，以及如何執行基本的資料表作業，例如新增、修改、讀取和移除資料表項目。  程式碼使用適用於 .NET 的 Azure 儲存體用戶端程式庫。 如需 ASP.NET 的詳細資訊，請參閱 [ASP.NET](http://www.asp.net)。

某些 Azure 儲存體 API 是非同步的，本文章中的程式碼假設我們正在使用非同步方法。 如需詳細資訊，請參閱[非同步程式設計](https://docs.microsoft.com/dotnet/csharp/async)。

## <a name="access-tables-in-code"></a>在程式碼中存取資料表

若要存取 ASP.NET Core 專案中的資料表，您需要將下列項目包含在存取 Azure 資料表儲存體的任何 C# 原始程式檔中。

1. 加入必要的 `using` 陳述式：

    ```csharp
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Table;
    using System.Threading.Tasks;
    ```

1. 取得代表儲存體帳戶資訊的 `CloudStorageAccount` 物件。 使用以下程式碼，並使用您的儲存體帳戶名稱和帳戶金鑰，您可以在 appSettings.json 的儲存體連接字串中找到它們：

    ```csharp
        CloudStorageAccount storageAccount = new CloudStorageAccount(
            new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials(
                "<name>", "<account-key>"), true);
    ```

1. 取得 `CloudTableClient` 物件，以參考儲存體帳戶中的資料表物件：

    ```csharp
    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```

1. 取得 `CloudTable`參考物件，以參考特定資料表與實體：

    ```csharp
    // Get a reference to a table named "peopleTable"
    CloudTable peopleTable = tableClient.GetTableReference("peopleTable");
    ```

## <a name="create-a-table-in-code"></a>在程式碼中建立資料表

若要建立 Azure 資料表，請建立非同步方法並在其中呼叫 `CreateIfNotExistsAsync()`：

```csharp
async void CreatePeopleTableAsync()
{
    // Create the CloudTable if it does not exist
    await peopleTable.CreateIfNotExistsAsync();
}
```
    
## <a name="add-an-entity-to-a-table"></a>將實體新增至資料表

若要將實體新增至資料表，請建立一個類別來定義實體的屬性。 下列程式碼會定義一個使用客戶名字作為資料列索引鍵、並使用姓氏作為資料分割索引鍵的實體類別 `CustomerEntity`。

```csharp
public class CustomerEntity : TableEntity
{
    public CustomerEntity(string lastName, string firstName)
    {
        this.PartitionKey = lastName;
        this.RowKey = firstName;
    }

    public CustomerEntity() { }

    public string Email { get; set; }

    public string PhoneNumber { get; set; }
}
```

實體相關的資料表作業會使用您稍早在[在程式碼中存取資料表](#access-tables-in-code)中所建立的 `CloudTable` 物件。 `TableOperation` 物件代表要執行的作業。 下列程式碼範例顯示如何建立 `CloudTable` 物件及 `CustomerEntity` 物件。 為了準備這項操作，其建立了 `TableOperation`，以便在資料表中插入客戶實體。 最後，其呼叫了 `CloudTable.ExecuteAsync` 來執行作業。

```csharp
// Create a new customer entity.
CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
customer1.Email = "Walter@contoso.com";
customer1.PhoneNumber = "425-555-0101";

// Create the TableOperation that inserts the customer entity.
TableOperation insertOperation = TableOperation.Insert(customer1);

// Execute the insert operation.
await peopleTable.ExecuteAsync(insertOperation);
```

## <a name="insert-a-batch-of-entities"></a>插入一批實體

您可以在單一寫入操作中將多個項目插入至資料表。 下列程式碼範例會建立兩個實體物件 ("Jeff Smith" 和 "Ben Smith")，並使用 `Insert` 方法將這兩個物件加入 `TableBatchOperation` 物件中，然後再呼叫 `CloudTable.ExecuteBatchAsync` 啟動作業。

```csharp
// Create the batch operation.
TableBatchOperation batchOperation = new TableBatchOperation();

// Create a customer entity and add it to the table.
CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
customer1.Email = "Jeff@contoso.com";
customer1.PhoneNumber = "425-555-0104";

// Create another customer entity and add it to the table.
CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
customer2.Email = "Ben@contoso.com";
customer2.PhoneNumber = "425-555-0102";

// Add both customer entities to the batch insert operation.
batchOperation.Insert(customer1);
batchOperation.Insert(customer2);

// Execute the batch operation.
await peopleTable.ExecuteBatchAsync(batchOperation);
```

## <a name="get-all-of-the-entities-in-a-partition"></a>取得資料分割中的所有實體

若要向資料表查詢資料分割中的所有實體，請使用 `TableQuery` 物件。 下列程式碼範例會指定篩選器來篩選出資料分割索引鍵為 'Smith' 的實體。 此範例會將查詢結果中每個實體的欄位列印至主控台。

```csharp
// Construct the query operation for all customer entities where PartitionKey="Smith".
TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

// Print the fields for each customer.
TableContinuationToken token = null;
do
{
    TableQuerySegment<CustomerEntity> resultSegment = await peopleTable.ExecuteQuerySegmentedAsync(query, token);
    token = resultSegment.ContinuationToken;

    foreach (CustomerEntity entity in resultSegment.Results)
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
    }
} while (token != null);
```

## <a name="get-a-single-entity"></a>取得單一實體

您可以撰寫查詢來取得單一特定實體。 下列程式碼使用 `TableOperation` 物件來指定名為 'Ben Smith' 的客戶。 此方法只會傳回一個實體而非一個集合，且傳回值在 `TableResult.Result` 中是一個 `CustomerEntity` 物件。 若要從 `Table`服務中擷取單一實體，最快的方法是在查詢中同時指定資料分割索引鍵和資料列索引鍵。

```csharp
// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the retrieve operation.
TableResult retrievedResult = await peopleTable.ExecuteAsync(retrieveOperation);

// Print the phone number of the result.
if (retrievedResult.Result != null)
   Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
else
   Console.WriteLine("The phone number could not be retrieved.");
```

## <a name="delete-an-entity"></a>刪除實體

找到實體之後，您可以刪除它。 下列程式碼會尋找及刪除名為 "Ben Smith" 的客戶實體：

```csharp
// Create a retrieve operation that expects a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the operation.
TableResult retrievedResult = await peopleTable.ExecuteAsync(retrieveOperation);

// Assign the result to a CustomerEntity object.
CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

// Create the Delete TableOperation and then execute it.
if (deleteEntity != null)
{
   TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

   // Execute the operation.
   await peopleTable.ExecuteAsync(deleteOperation);

   Console.WriteLine("Entity deleted.");
}

else
   Console.WriteLine("Couldn't delete the entity.");
```

## <a name="next-steps"></a>後續步驟
[!INCLUDE [vs-storage-dotnet-tables-next-steps](../../includes/vs-storage-dotnet-tables-next-steps.md)]
