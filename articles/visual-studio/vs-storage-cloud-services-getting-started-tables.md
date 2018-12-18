---
title: 開始使用資料表儲存體和 Visual Studio 已連接服務 (雲端服務) | Microsoft Docs
description: 在使用 Visual Studio 已連接服務連接到儲存體帳戶之後，如何在 Visual Studio 雲端服務專案中開始使用 Azure 資料表儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: a3a11ed8-ba7f-4193-912b-e555f5b72184
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ms.openlocfilehash: d88e8e85613faa24213b6e12b5ba4f30e3d84f74
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31795148"
---
# <a name="getting-started-with-azure-table-storage-and-visual-studio-connected-services-cloud-services-projects"></a>開始使用 Azure 資料表儲存體和 Visual Studio 已連接服務 (雲端服務專案)
[!INCLUDE [storage-try-azure-tools-tables](../../includes/storage-try-azure-tools-tables.md)]

## <a name="overview"></a>概觀
本文說明如何在您使用 Visual Studio 的 [新增連接的服務]  對話方塊建立或參考雲端服務專案中的 Azure 儲存體帳戶之後，開始在 Visual Studio 中使用 Azure 表格儲存體。 [ **新增連接的服務** ] 作業會安裝適當的 NuGet 封裝，以存取專案中的 Azure 儲存體，並將儲存體帳戶的連接字串新增至您的專案組態檔。

Azure 資料表儲存體服務可讓您儲存大量的結構化資料。 此服務是一個 NoSQL 資料存放區，接受來自 Azure 雲端內外經過驗證的呼叫。 Azure 資料表很適合儲存結構化、非關聯式資料。

若要開始，首先您必須在儲存體帳戶中建立資料表。 我們將說明如何使用程式碼建立 Azure 資料表，以及如何執行基本的資料表和實體作業，例如新增、修改、讀取和讀取資料表實體。 這些範例均以 C\# 程式碼撰寫，並使用 [Microsoft Azure Storage client library for .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx)。

**注意：** 有一些 API會以非同步方式對 Azure 儲存體執行呼叫。 如需詳細資訊，請參閱 [使用 Async 及 Await 進行非同步程式設計](http://msdn.microsoft.com/library/hh191443.aspx) 。 以下程式碼假設使用非同步程式設計方法。

* 如需以程式設計方式處理資料表的詳細資訊，請參閱 [以 .NET 開始使用 Azure 資料表儲存體](../storage/storage-dotnet-how-to-use-tables.md) 。
* 如需 Azure 儲存體的一般資訊，請參閱 [儲存體文件](https://azure.microsoft.com/documentation/services/storage/) 。
* 如需 Azure 雲端服務的一般資訊，請參閱 [雲端服務文件](https://azure.microsoft.com/documentation/services/cloud-services/) 。
* 若需要如何編寫 ASP.NET 應用程式的詳細資訊，請參閱 [ASP.NET](http://www.asp.net) 。

## <a name="access-tables-in-code"></a>在程式碼中存取資料表
若要存取雲端服務專案中的資料表，您需要將下列項目併入至存取 Azure 資料表儲存體的任何 C# 原始程式檔。

1. 請確定 C# 檔案頂端的命名空間宣告包含這些 **using** 陳述式。
   
        using Microsoft.Framework.Configuration;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Table;
        using System.Threading.Tasks;
        using LogLevel = Microsoft.Framework.Logging.LogLevel;
2. 取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼，從 Azure 服務組態取得儲存體連接字串和儲存體帳戶資訊。
   
         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage account name>
         _AzureStorageConnectionString"));
   > [!NOTE]
   > 請在下列範例中的程式碼前面使用上述所有程式碼。
   > 
   > 
3. 取得 **CloudTableClient** 物件，以參考您儲存體帳戶中的資料表物件。
   
         // Create the table client.
         CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
4. 取得 **CloudTable** 參考物件，以參考特定資料表與實體。
   
        // Get a reference to a table named "peopleTable".
        CloudTable peopleTable = tableClient.GetTableReference("peopleTable");

## <a name="create-a-table-in-code"></a>在程式碼中建立資料表
若要建立 Azure 資料表，只要依照＜存取程式碼中的資料表＞一節所述，在取得 **CloudTable** 物件後加入 **CreateIfNotExistsAsync** 呼叫即可。

    // Create the CloudTable if it does not exist.
    await peopleTable.CreateIfNotExistsAsync();

## <a name="add-an-entity-to-a-table"></a>將實體新增至資料表
若要將實體新增至資料表，請建立一個類別來定義實體的屬性。 下列程式碼會定義稱為 **CustomerEntity** 的實體類別，其使用客戶名字作為資料列索引鍵，並使用姓氏作為資料分割索引鍵。

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

實體相關的資料表作業會使用您稍早在＜在程式碼中存取資料表＞中建立的 **CloudTable** 物件執行。 **TableOperation** 物件要執行的操作。 下列程式碼範例顯示如何建立 **CloudTable** 物件及 **CustomerEntity** 物件。 為了準備這項操作，其建立了 **TableOperation** ，以便在資料表中插入客戶實體。 最後會呼叫 **CloudTable.ExecuteAsync**來執行此作業。

    // Create a new customer entity.
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.Email = "Walter@contoso.com";
    customer1.PhoneNumber = "425-555-0101";

    // Create the TableOperation that inserts the customer entity.
    TableOperation insertOperation = TableOperation.Insert(customer1);

    // Execute the insert operation.
    await peopleTable.ExecuteAsync(insertOperation);


## <a name="insert-a-batch-of-entities"></a>插入一批實體
您可以在單一寫入操作中將多個項目插入至資料表。 下列程式碼範例會建立兩個實體物件 ("Jeff Smith" 和 "Ben Smith")，並使用 Insert 方法將這兩個物件加入 **TableBatchOperation** 物件中，然後再呼叫 **CloudTable.ExecuteBatchAsync** 啟動作業。

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

## <a name="get-all-of-the-entities-in-a-partition"></a>取得資料分割中的所有實體
若要向資料表查詢資料分割中的所有實體，請使用 **TableQuery** 物件。 下列程式碼範例會指定篩選器來篩選出資料分割索引鍵為 'Smith' 的實體。 此範例會將查詢結果中每個實體的欄位列印至主控台。

    // Construct the query operation for all customer entities where PartitionKey="Smith".
    TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>()
        .Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

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

    return View();


## <a name="get-a-single-entity"></a>取得單一實體
您可以撰寫查詢來取得單一特定實體。 下列程式碼使用 **TableOperation** 物件來指定名為 'Ben Smith' 的客戶。 這個方法只會傳回一個實體而不是集合，而且所傳回的 **TableResult.Result** 值是 **CustomerEntity** 物件。 若要從「資料表」  服務中擷取單一實體，最快的方法是在查詢中同時指定資料分割索引鍵和資料列索引鍵。

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the retrieve operation.
    TableResult retrievedResult = await peopleTable.ExecuteAsync(retrieveOperation);

    // Print the phone number of the result.
    if (retrievedResult.Result != null)
       Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
    else
       Console.WriteLine("The phone number could not be retrieved.");

## <a name="delete-an-entity"></a>刪除實體
找到實體之後，您可以刪除它。 下列程式碼會尋找名為 "Ben Smith" 的客戶實體，如果找到，就會刪除它。

    // Create a retrieve operation that expects a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = peopleTable.Execute(retrieveOperation);

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

## <a name="next-steps"></a>後續步驟
[!INCLUDE [vs-storage-dotnet-tables-next-steps](../../includes/vs-storage-dotnet-tables-next-steps.md)]

