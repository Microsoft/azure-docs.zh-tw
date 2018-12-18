---
title: 以 .NET 開始使用 Azure 資料表儲存體和 Azure Cosmos DB 資料表 API
description: 使用 Azure 資料表儲存體或 Azure Cosmos DB 資料表 API 將結構化資料儲存在雲端。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: dotnet
ms.topic: sample
ms.date: 08/17/2018
ms.author: sngun
ms.openlocfilehash: 00bc950493e7b4eaad4a2165779f5c8151070aee
ms.sourcegitcommit: 3856c66eb17ef96dcf00880c746143213be3806a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48042572"
---
# <a name="get-started-with-azure-table-storage-and-the-azure-cosmos-db-table-api-using-net"></a>以 .NET 開始使用 Azure 資料表儲存體和 Azure Cosmos DB 資料表 API
[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

您可以使用 Azure 資料表儲存體或 Azure Cosmos DB 資料表 API 將結構化的 NoSQL 資料儲存在雲端中，以提供具有無結構描述設計的索引鍵/屬性存放區。 由於資料表儲存體和 Azure Cosmos DB 資料表 API 並無結構描述，因此可輕易隨著應用程式發展需求改寫資料。 相較於類似資料量的傳統 SQL，對許多類型的應用程式而言，資料表儲存體資料和 Azure Cosmos DB 資料表 API 可快速存取且符合成本效益，通常可降低成本。

您可以使用資料表儲存體或 Azure Cosmos DB 資料表 API 來儲存具彈性的資料集，例如 Web 應用程式的使用者資料、通訊錄、裝置資訊，以及服務所需的其他中繼資料類型。 您可以在資料表中儲存任意數目的實體，且儲存體帳戶或資料表 API 帳戶可包含任意數目的資料表，最高可達儲存體帳戶或資料表 API 帳戶的容量限制。

### <a name="about-this-sample"></a>關於此範例
此範例示範如何在常見的 Azure 資料表儲存體和資料表 API 案例中使用[適用於 .NET 的 Microsoft Azure CosmosDB 表格文件庫](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table)。 套件名稱表示其適用於 Azure Cosmos DB，但該套件也同時適用於 Azure Cosmos DB 資料表 API 和 Azure 資料表儲存體，兩個服務皆具有唯一的端點。 在這些案例探索中會使用 C# 範例，可說明如何執行下列動作：
* 建立和刪除表格
* 插入、更新和刪除資料列
* 查詢資料表

## <a name="prerequisites"></a>必要條件

您需要下列項目才能成功完成此範例︰

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)
* [適用於 .NET 的 Azure 儲存體通用程式庫 (預覽)](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/)。 - 在生產環境中受到支援的必要預覽套件。 
* [適用於 .NET 的 Microsoft Azure CosmosDB 表格文件庫](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table) - 此文件庫目前僅適用於 .NET Standard，尚不適用於 .NET Core。
* [適用於.NET 的 Azure 設定管理員](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
* [Azure 儲存體帳戶](../storage/common/storage-quickstart-create-account.md)

[!INCLUDE [storage-dotnet-client-library-version-include](../../includes/storage-dotnet-client-library-version-include.md)]

### <a name="more-samples"></a>更多範例
如需使用表格儲存體的其他範例，請參閱 [在 .NET 中開始使用 Azure 表格儲存體](https://azure.microsoft.com/documentation/samples/storage-table-dotnet-getting-started/)。 您可以下載範例應用程式並加以執行，或瀏覽 GitHub 上的程式碼。

## <a name="create-an-azure-service-account"></a>建立 Azure 服務帳戶
[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

### <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶
* 建立您第一個 Azure 儲存體帳戶最簡單的方法，就是使用 [Azure 入口網站](https://portal.azure.com)。 若要深入了解，請參閱 [建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)。

* 您也可以使用 [Azure PowerShell](../storage/common/storage-powershell-guide-full.md)、[Azure CLI](../storage/common/storage-azure-cli.md)，或 [Storage Resource Provider Client Library for .NET](/dotnet/api/microsoft.azure.management.storage) 來建立 Azure 儲存體帳戶。

* 如果您不想在此時建立儲存體帳戶，也可以使用 Azure 儲存體模擬器在本機環境中執行並測試您的程式碼。 如需詳細資訊，請參閱 [使用 Azure 儲存體模擬器進行開發和測試](../storage/common/storage-use-emulator.md)。

### <a name="create-an-azure-cosmos-db-table-api-account"></a>建立 Azure Cosmos DB 表格 API 帳戶
[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

### <a name="create-a-windows-console-application-project"></a>建立 Windows 主控台應用程式專案
在 Visual Studio 中，建立新的 Windows 主控台應用程式。 下列步驟示範如何在 Visual Studio 2017 中建立主控台應用程式。 這些步驟類似其他 Visual Studio 版本中的步驟。

1. 選取 [檔案] > [新增] > [專案]。
2. 選取 [已安裝] > [Visual C#] > [Windows 傳統桌面]。
3. 選取 **主控台應用程式 (.NET Framework)**。
4. 在 [名稱] 欄位中，輸入應用程式的名稱。
5. 選取 [確定] 。

此範例中的所有程式碼範例均可新增至您主控台應用程式的 `Program.cs` 檔案中的 `Main()` 方法。

您可以在任何類型的 .NET 應用程式 (包括 Azure 雲端服務或 Web 應用程式和桌面與行動應用程式) 中使用 Azure CosmosDB 表格文件庫。 在本指南中，為求簡化，我們會使用主控台應用程式。

### <a name="install-the-required-nuget-packages"></a>安裝必要的 NuGet 套件
您必須在您的專案中參考下列三個建議套件，才能完成此範例︰

* [適用於 .NET 的 Azure 儲存體通用程式庫 (預覽)](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common)。 - 使用低於或等於 9.0.0.1 (<= 9.0.0.1) 的版本。

* [適用於 .NET 的 Microsoft Azure Cosmos DB 表格文件庫](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table)。 此套件可供以程式設計方式存取 Azure 表格儲存體帳戶或 Azure Cosmos DB 表格 API 帳戶中的資料資源。 此文件庫目前僅適用於 .NET Standard，尚不適用於 .NET Core。

* [適用於 .NET 的 Microsoft Azure Configuration Manager 程式庫](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)︰此套件提供一個類別，無論您的應用程式於何處執行，均可用來剖析組態檔中的連接字串。

若要取得 NuGet 套件，請遵循下列步驟：

1. 在 [方案總管] 中以滑鼠右鍵按一下專案，然後選擇 [管理 NuGet 套件]。
2. 線上搜尋 "Microsoft.Azure.Storage.Common"，選擇 <= 9.0.0.1 的版本，然後選取 [安裝] 以安裝適用於 .NET 的 Azure 儲存體通用程式庫 (預覽) 及其相依性。 請確認已勾選 [包含發行前版本] 方塊，因為這是預覽套件。
3. 線上搜尋 "Microsoft.Azure.CosmosDB.Table"，然後選取 [安裝] 以安裝 Microsoft Azure CosmosDB 表格文件庫。
4. 線上搜尋 "WindowsAzure.ConfigurationManager"，然後選取 [安裝] 以安裝 Microsoft Azure Configuration Manager Library。

> [!NOTE]
> 適用於 .NET 的儲存體通用程式庫中的 ODataLib 相依性現由 ODataLib 套件解析，該套件可在 NuGet 上取得而非由 WCF 資料服務提供。 您可以直接下載 ODataLib 程式庫，或是由您的程式碼專案透過 NuGet 參照這些程式庫。 Storage Client Library 使用的特定 ODataLib 封裝有 [OData](http://nuget.org/packages/Microsoft.Data.OData/)、[Edm](http://nuget.org/packages/Microsoft.Data.Edm/)，以及 [Spatial](http://nuget.org/packages/System.Spatial/)。 這些程式庫雖由 Azure 表格儲存體類別使用，它們同時也是使用儲存體通用程式庫進行程式設計的必要相依項目。
> 
> 

> [!TIP]
> 已經熟悉 Azure 資料表儲存體的開發人員以往可能會使用 [WindowsAzure.Storage](https://www.nuget.org/packages/WindowsAzure.Storage/) 套件。 建議所有新的資料表應用程式使用 [Azure 儲存體通用程式庫](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common)和 [Azure Cosmos DB 表格文件庫](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table)，但是 WindowsAzure.Storage 套件仍然受到支援。 如果您使用 WindowsAzure.Storage 程式庫，在您使用的陳述式中包含 Microsoft.WindowsAzure.Storage.Table。
>
>

### <a name="determine-your-target-environment"></a>決定您的目標環境
有三個環境選項可供您執行本指南中的範例︰

* 您可以對雲端中的 Azure 儲存體帳戶執行您的程式碼。 
* 您可以對雲端中的 Azure Cosmos DB 帳戶執行您的程式碼。
* 您可以對 Azure 儲存體模擬器執行您的程式碼。 儲存體模擬器是模擬雲端中 Azure 儲存體帳戶的本機環境。 模擬器是一個免費選項，在開發您的應用程式時可用於測試和偵錯您的程式碼。 模擬器會使用已知的帳戶和金鑰。 如需詳細資訊，請參閱[使用 Azure 儲存體模擬器進行開發和測試](../storage/common/storage-use-emulator.md)。

如果您要選擇以雲端中的儲存體帳戶為目標，請從 Azure 入口網站複製您的儲存體帳戶的主要存取金鑰。 如需詳細資訊，請參閱[儲存體帳戶存取金鑰](../storage/common/storage-account-manage.md#access-keys)。

> [!NOTE]
> 您可以選擇以儲存體模擬器為目標，以避免產生與 Azure 儲存體相關聯的任何費用。 不過，如果您選擇以雲端中的 Azure 儲存體帳戶為目標，則執行此範例的費用可以忽略不計。
> 
> 

如果您選擇以 Azure Cosmos DB 帳戶為目標，請從 Azure 入口網站複製表格 API 帳戶的主要存取金鑰。 如需詳細資訊，請參閱[更新連接字串](create-table-dotnet.md#update-your-connection-string)。

### <a name="configure-your-storage-connection-string"></a>設定儲存體連接字串
適用於 .NET 的 Azure 儲存體通用程式庫可支援使用儲存體連接字串，來設定存取儲存服務時所需的端點與認證。 在組態檔中維護儲存體連接字串是最佳方式。 

如需有關連接字串的詳細資訊，請參閱[設定 Azure 儲存體的連接字串](../storage/common/storage-configure-connection-string.md)。

> [!NOTE]
> 帳戶金鑰類似於儲存體帳戶的根密碼。 請務必小心保護您的儲存體帳戶金鑰。 請避免轉發給其他使用者、進行硬式編碼，或將它儲存在其他人可以存取的純文字檔案。 如果您認為金鑰可能遭到破解，請使用 Azure 入口網站重新產生金鑰。
> 

若要設定連接字串，請從 Visual Studio 中的 [方案總管] 開啟 `app.config` 檔案。 如下所示，加入 `<appSettings>` 元素的內容。 以您的帳戶名稱取代 `account-name`，並以您的帳戶存取金鑰取代 `account-key`：

```xml
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6.1" />
    </startup>
    <appSettings>
        <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key" />
    </appSettings>
</configuration>
```

例如，如果您使用 Azure 儲存體帳戶，您的組態設定會類似於：

```xml
<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=<account-key>" />
```

如果您使用 Azure Cosmos DB 帳戶，您的組態設定會類似於：

```xml
<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=tableapiacct;AccountKey=<account-key>;TableEndpoint=https://tableapiacct.table.cosmosdb.azure.com:443/;" />
```

若要以儲存體模擬器為目標，您可以使用對應到已知帳戶名稱和金鑰的捷徑。 在此情況下，您的連接字串設定會是︰

```xml
<add key="StorageConnectionString" value="UseDevelopmentStorage=true;" />
```

### <a name="add-using-directives"></a>新增 using 指示詞
將下列 using 指示詞新增至 `Program.cs` 檔案的開頭處：

```csharp
using Microsoft.Azure; // Namespace for CloudConfigurationManager
using Microsoft.Azure.Storage; // Namespace for StorageAccounts
using Microsoft.Azure.CosmosDB.Table; // Namespace for Table storage types
```

### <a name="parse-the-connection-string"></a>解析連接字串
[!INCLUDE [storage-cloud-configuration-manager-include](../../includes/storage-cloud-configuration-manager-include.md)]

### <a name="create-the-table-service-client"></a>建立表格服務用戶端
[CloudTableClient][dotnet_CloudTableClient] 類別可讓您擷取表格儲存體中儲存的資料表。 以下是建立表格服務用戶端的其中一種方式：

```csharp
// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
```

您現在可以開始撰寫程式碼，以讀取表格儲存體的資料並將資料寫入其中。

## <a name="create-a-table"></a>建立資料表
此範例說明如何建立尚不存在的資料表：

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Retrieve a reference to the table.
CloudTable table = tableClient.GetTableReference("people");

// Create the table if it doesn't exist.
table.CreateIfNotExists();
```

## <a name="add-an-entity-to-a-table"></a>將實體新增至資料表
使用衍生自 [TableEntity][dotnet_TableEntity] 的自訂類別，將實體對應至 C# 物件。 若要將實體新增至資料表，請建立一個類別來定義實體的屬性。 下列程式碼會定義一個使用客戶名字作為資料列索引鍵、並使用姓氏作為資料分割索引鍵的實體類別。 系統會在資料表中以實體的資料分割和資料列索引鍵共同針對實體進行唯一識別。 查詢具有相同分割區索引鍵的實體，其速度快於查詢具有不同分割區索引鍵的實體，但使用不同的資料分割索引鍵可提供更佳的延展性或平行作業。 要儲存在資料表中的實體都必須屬於支援的類型，例如衍生自 [TableEntity][dotnet_TableEntity] 類別。 您想要儲存在資料表中的實體屬性必須是該類型的公用屬性，而且同時支援值的取得和設定。 此外，您的實體類型「必須」  公開無參數建構函式。

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

包含實體的資料表作業會透過您先前在＜建立資料表＞一節建立的 [CloudTable][dotnet_CloudTable] 物件執行。 要執行的作業是以 [TableOperation][dotnet_TableOperation] 物件代表。 下列程式碼範例示範如何依序建立 [CloudTable][dotnet_CloudTable] 物件及 **CustomerEntity** 物件。 為了準備作業，已建立 [TableOperation][dotnet_TableOperation] 物件以將客戶實體插入資料表。 最後，其呼叫了 [CloudTable][dotnet_CloudTable].[Execute][dotnet_CloudTable_Execute] 來執行作業。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable object that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Create a new customer entity.
CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
customer1.Email = "Walter@contoso.com";
customer1.PhoneNumber = "425-555-0101";

// Create the TableOperation object that inserts the customer entity.
TableOperation insertOperation = TableOperation.Insert(customer1);

// Execute the insert operation.
table.Execute(insertOperation);
```

## <a name="insert-a-batch-of-entities"></a>插入一批實體
您可以在單一寫入作業中，插入實體批次至資料表。 以下是批次作業的其他一些注意事項：

* 您可以在同一批次作業中執行更新、刪除和插入。
* 單一批次作業最多可包含 100 個實體。
* 單一批次作業中的所有實體必須具有相同的資料分割索引鍵。
* 雖然可以單一批次作業的形式來執行查詢，但該查詢必須是批次中的唯一作業。

下列程式碼範例會建立兩個實體物件，並使用 [Insert][dotnet_TableBatchOperation_Insert] 方法將每個物件新增至 [TableBatchOperation][dotnet_TableBatchOperation]。 然後會呼叫 [CloudTable][dotnet_CloudTable].[ExecuteBatch][dotnet_CloudTable_ExecuteBatch] 以執行作業。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable object that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

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
table.ExecuteBatch(batchOperation);
```

## <a name="retrieve-all-entities-in-a-partition"></a>擷取資料分割中的所有實體
若要向資料表查詢資料分割中的所有實體，請使用 [TableQuery][dotnet_TableQuery] 物件。 下列程式碼範例會指定篩選器來篩選出資料分割索引鍵為 'Smith' 的實體。 此範例會將查詢結果中每個實體的欄位列印至主控台。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable object that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Construct the query operation for all customer entities where PartitionKey="Smith".
TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

// Print the fields for each customer.
foreach (CustomerEntity entity in table.ExecuteQuery(query))
{
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
}
```

## <a name="retrieve-a-range-of-entities-in-a-partition"></a>擷取資料分割中某個範圍的實體
如果您不想要查詢資料分割中的所有實體，可結合資料分割索引鍵篩選器與資料列索引鍵篩選器來指定範圍。 下列程式碼範例會使用兩個篩選器來取得資料分割 'Smith' 中，資料列索引鍵 (名字) 是以字母 'E' 前之字母為開頭的所有實體，然後會列印查詢結果。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable object that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Create the table query.
TableQuery<CustomerEntity> rangeQuery = new TableQuery<CustomerEntity>().Where(
    TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"),
        TableOperators.And,
        TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "E")));

// Loop through the results, displaying information about the entity.
foreach (CustomerEntity entity in table.ExecuteQuery(rangeQuery))
{
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
}
```

## <a name="retrieve-a-single-entity"></a>擷取單一實體
您可以撰寫查詢來擷取單一特定實體。 下列程式碼會使用 [TableOperation][dotnet_TableOperation] 來指定客戶 'Ben Smith'。 這個方法只會傳回一個實體而不是集合，而且所傳回的 [TableResult][dotnet_TableResult].[Result][dotnet_TableResult_Result] 值是 **CustomerEntity** 物件。 若要從資料表服務中擷取單一實體，最快的方法是在查詢中同時指定資料分割索引鍵和資料列索引鍵。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable object that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the retrieve operation.
TableResult retrievedResult = table.Execute(retrieveOperation);

// Print the phone number of the result.
if (retrievedResult.Result != null)
{
    Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
}
else
{
    Console.WriteLine("The phone number could not be retrieved.");
}
```

## <a name="replace-an-entity"></a>取代實體
若要更新實體，從資料表服務擷取實體、修改實體物件，然後將變更儲存回資料表服務。 下列程式碼會變更現有客戶的電話號碼。 此程式碼不會呼叫 [Insert][dotnet_TableOperation_Insert]，而是使用 [Replace][dotnet_TableOperation_Replace]。 [Replace][dotnet_TableOperation_Replace] 會完全取代伺服器上的實體，但如果伺服器上的實體自擷取後產生變化，作業就會失敗。 如此會造成失敗，是為了防止應用程式意外覆寫該應用程式的其他元件在擷取後到更新前的這段期間所做的變更。 正確處理此失敗的方式為重新擷取實體、進行變更 (如果仍然有效)，然後再執行一次 [Replace][dotnet_TableOperation_Replace] 作業。 下一節將示範如何覆寫此行為。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable object that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the operation.
TableResult retrievedResult = table.Execute(retrieveOperation);

// Assign the result to a CustomerEntity object.
CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

if (updateEntity != null)
{
    // Change the phone number.
    updateEntity.PhoneNumber = "425-555-0105";

    // Create the Replace TableOperation.
    TableOperation updateOperation = TableOperation.Replace(updateEntity);

    // Execute the operation.
    table.Execute(updateOperation);

    Console.WriteLine("Entity updated.");
}
else
{
    Console.WriteLine("Entity could not be retrieved.");
}
```

## <a name="insert-or-replace-an-entity"></a>插入或取代實體
如果從伺服器擷取的實體自擷取後發生變化，[Replace][dotnet_TableOperation_Replace] 作業將失敗。 此外，您必須先從伺服器擷取實體，[Replace][dotnet_TableOperation_Replace] 作業才會成功。 不過有時候，您不知道實體是否存在於伺服器上且其中所儲存的值是否無關。 您的更新應該將其全部覆寫。 若要達到此目標，您可以使用 [InsertOrReplace][dotnet_TableOperation_InsertOrReplace] 作業。 此作業會插入實體 (如果其目前並不存在) 或取代實體 (如果其已存在)，不論上次是何時更新。

在下列程式碼範例中，'Fred Jones' 的客戶實體會建立並插入 'people' 資料表中。 接下來，我們使用 [InsertOrReplace][dotnet_TableOperation_InsertOrReplace] 作業，將具有相同資料分割索引鍵 (Jones) 和資料列索引鍵 (Fred) 的實體儲存到伺服器，但這次使用不同的 PhoneNumber 屬性值。 因為我們使用 [InsertOrReplace][dotnet_TableOperation_InsertOrReplace]，其所有的屬性值都會被取代。 不過，如果 'Fred Jones' 實體尚未存在於資料表中，則會予以插入。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable object that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Create a customer entity.
CustomerEntity customer3 = new CustomerEntity("Jones", "Fred");
customer3.Email = "Fred@contoso.com";
customer3.PhoneNumber = "425-555-0106";

// Create the TableOperation object that inserts the customer entity.
TableOperation insertOperation = TableOperation.Insert(customer3);

// Execute the operation.
table.Execute(insertOperation);

// Create another customer entity with the same partition key and row key.
// We've already created a 'Fred Jones' entity and saved it to the
// 'people' table, but here we're specifying a different value for the
// PhoneNumber property.
CustomerEntity customer4 = new CustomerEntity("Jones", "Fred");
customer4.Email = "Fred@contoso.com";
customer4.PhoneNumber = "425-555-0107";

// Create the InsertOrReplace TableOperation.
TableOperation insertOrReplaceOperation = TableOperation.InsertOrReplace(customer4);

// Execute the operation. Because a 'Fred Jones' entity already exists in the
// 'people' table, its property values will be overwritten by those in this
// CustomerEntity. If 'Fred Jones' didn't already exist, the entity would be
// added to the table.
table.Execute(insertOrReplaceOperation);
```

## <a name="query-a-subset-of-entity-properties"></a>查詢實體屬性的子集
一項資料表查詢可以只擷取實體的少數屬性而非所有屬性。 這項稱為「投射」的技術可減少頻寬並提高查詢效能 (尤其是對大型實體而言)。 下列程式碼中的查詢只會傳回資料表中各實體的電子郵件地址。 這是使用 [DynamicTableEntity][dotnet_DynamicTableEntity] 以及 [EntityResolver][dotnet_EntityResolver] 的查詢所完成的。 您可以在[更新插入和查詢投影簡介的部落格文章][blog_post_upsert]中進一步了解投影。 儲存體模擬器並不支援投影，因此此程式碼只有在資料表服務中使用帳戶時才會執行。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Define the query, and select only the Email property.
TableQuery<DynamicTableEntity> projectionQuery = new TableQuery<DynamicTableEntity>().Select(new string[] { "Email" });

// Define an entity resolver to work with the entity after retrieval.
EntityResolver<string> resolver = (pk, rk, ts, props, etag) => props.ContainsKey("Email") ? props["Email"].StringValue : null;

foreach (string projectedEmail in table.ExecuteQuery(projectionQuery, resolver, null, null))
{
    Console.WriteLine(projectedEmail);
}
```

## <a name="delete-an-entity"></a>刪除實體
使用更新實體時所展示的相同方法，輕鬆地在擷取實體後將其刪除。 下列程式碼會擷取並刪除客戶實體。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Create a retrieve operation that expects a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the operation.
TableResult retrievedResult = table.Execute(retrieveOperation);

// Assign the result to a CustomerEntity.
CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

// Create the Delete TableOperation.
if (deleteEntity != null)
{
    TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

    // Execute the operation.
    table.Execute(deleteOperation);

    Console.WriteLine("Entity deleted.");
}
else
{
    Console.WriteLine("Could not retrieve the entity.");
}
```

## <a name="delete-a-table"></a>刪除資料表
最後，下列程式碼範例會從儲存體帳戶刪除資料表。 已刪除的資料表在刪除後一段時間內，將無法重新建立。

```csharp
// Retrieve the storage account from the connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the table client.
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

// Create the CloudTable that represents the "people" table.
CloudTable table = tableClient.GetTableReference("people");

// Delete the table it if exists.
table.DeleteIfExists();
```

## <a name="retrieve-entities-in-pages-asynchronously"></a>以非同步方式擷取頁面中的實體
如果您要讀取大量實體，且您想要在擷取實體時處理/顯示實體，但不想等待它們全部傳回，您可以使用分割查詢來擷取實體。 這個範例示範如何使用 Async-Await 模式，在您等候大量的結果集傳回時亦不會妨礙執行作業，以便在頁面中傳回結果。 如需在 .NET 中使用 Async-Awaitt 模式的詳細資訊，請參閱 [Async 和 Await (C# 和 Visual Basic) 的非同步程式設計](https://msdn.microsoft.com/library/hh191443.aspx)。

```csharp
// Initialize a default TableQuery to retrieve all the entities in the table.
TableQuery<CustomerEntity> tableQuery = new TableQuery<CustomerEntity>();

// Initialize the continuation token to null to start from the beginning of the table.
TableContinuationToken continuationToken = null;

do
{
    // Retrieve a segment (up to 1,000 entities).
    TableQuerySegment<CustomerEntity> tableQueryResult =
        await table.ExecuteQuerySegmentedAsync(tableQuery, continuationToken);

    // Assign the new continuation token to tell the service where to
    // continue on the next iteration (or null if it has reached the end).
    continuationToken = tableQueryResult.ContinuationToken;

    // Print the number of rows retrieved.
    Console.WriteLine("Rows retrieved {0}", tableQueryResult.Results.Count);

// Loop until a null continuation token is received, indicating the end of the table.
} while(continuationToken != null);
```

## <a name="next-steps"></a>後續步驟
現在您已經了解表格儲存體的基本概念，請參考下列連結以了解更複雜的儲存體工作：

* [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md) 是一個免費的獨立應用程式，可讓您在 Windows、MacOS 和 Linux 上以視覺化方式處理 Azure 儲存體資料。
* 如需更多表格儲存體範例，請參閱 [在 .NET 中開始使用 Azure 表格儲存體](https://azure.microsoft.com/documentation/samples/storage-table-dotnet-getting-started/)
* 如需可用 API 的完整詳細資訊，請檢視資料表服務參考文件：
* [Storage Client Library for .NET 參考資料](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
* [REST API 參考資料](http://msdn.microsoft.com/library/azure/dd179355)
* 了解如何使用 [Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/wiki)
* 如需了解 Azure 中的其他資料儲存選項，請檢視更多功能指南。
* [以 .NET 開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md) 以儲存非結構化資料。
* [使用 .NET (C#) 連接到 SQL Database ](../sql-database/sql-database-develop-dotnet-simple.md) 以儲存關聯式資料。

[Download and install the Azure SDK for .NET]: /develop/net/
[Creating an Azure Project in Visual Studio]: http://msdn.microsoft.com/library/azure/ee405487.aspx

[blog_post_upsert]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx

[dotnet_api_ref]: https://msdn.microsoft.com/library/azure/mt347887.aspx
[dotnet_CloudTableClient]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.cloudtableclient.aspx
[dotnet_CloudTable]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.cloudtable.aspx
[dotnet_CloudTable_Execute]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.cloudtable.execute.aspx
[dotnet_CloudTable_ExecuteBatch]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.cloudtable.executebatch.aspx
[dotnet_DynamicTableEntity]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.dynamictableentity.aspx
[dotnet_EntityResolver]: https://msdn.microsoft.com/library/jj733144.aspx
[dotnet_TableBatchOperation]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tablebatchoperation.aspx
[dotnet_TableBatchOperation_Insert]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tablebatchoperation.insert.aspx
[dotnet_TableEntity]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tableentity.aspx
[dotnet_TableOperation]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tableoperation.aspx
[dotnet_TableOperation_Insert]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tableoperation.insert.aspx
[dotnet_TableOperation_InsertOrReplace]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tableoperation.insertorreplace.aspx
[dotnet_TableOperation_Replace]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tableoperation.replace.aspx
[dotnet_TableQuery]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tablequery.aspx
[dotnet_TableResult]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tableresult.aspx
[dotnet_TableResult_Result]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tableresult.result.aspx
