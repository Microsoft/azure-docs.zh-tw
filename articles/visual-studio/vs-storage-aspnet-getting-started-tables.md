---
title: 開始使用 Azure 資料表儲存體和 Visual Studio 已連線的服務 (ASP.NET) | Microsoft Docs
description: 在使用 Visual Studio 已連線的服務連接到儲存體帳戶之後，如何在 Visual Studio ASP.NET 專案中開始使用 Azure 資料表儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: af81a326-18f4-4449-bc0d-e96fba27c1f8
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 12/21/2016
ms.author: ghogen
ms.openlocfilehash: 39456380769e1c3b790d2bbc6fdf9c04c983d054
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38531384"
---
# <a name="get-started-with-azure-table-storage-and-visual-studio-connected-services-aspnet"></a>開始使用 Azure 資料表儲存體和 Visual Studio 已連線的服務 (ASP.NET)
[!INCLUDE [storage-try-azure-tools-tables](../../includes/storage-try-azure-tools-tables.md)]

## <a name="overview"></a>概觀

Azure 資料表儲存體可讓您儲存大量的結構化資料。 此服務是一個 NoSQL 資料存放區，接受來自 Azure 雲端內外經過驗證的呼叫。 Azure 資料表很適合儲存結構化、非關聯式資料。

本教學課程說明如何使用 Azure 表格儲存體實體撰寫一些常見案例的 ASP.NET 程式碼。 這些案例包括建立資料表，以及新增、查詢和刪除資料表實體。 

## <a name="prerequisites"></a>先決條件

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)
* [Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md#create-a-storage-account)

[!INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../includes/vs-storage-aspnet-getting-started-create-azure-account.md)]

[!INCLUDE [storage-development-environment-include](../../includes/vs-storage-aspnet-getting-started-setup-dev-env.md)]

### <a name="create-an-mvc-controller"></a>建立 MVC 控制器 

1. 在 [方案總管] 中，用滑鼠右鍵按一下 [控制器]，然後從內容功能表中選取 [新增] > [控制器]。

    ![將控制器新增至 ASP.NET MVC 應用程式](./media/vs-storage-aspnet-getting-started-tables/add-controller-menu.png)

1. 在 [Add Scaffold] 對話方塊中，按一下 [MVC 5 Controller - Empty]然後選取 [新增]。

    ![指定 MVC 控制器類型](./media/vs-storage-aspnet-getting-started-tables/add-controller.png)

1. 在 [新增控制器] 對話方塊中，將控制器命名為 TablesController，然後選取 [新增]。

    ![命名 MVC 控制器](./media/vs-storage-aspnet-getting-started-tables/add-controller-name.png)

1. 將下列 using 指示詞新增至 `TablesController.cs` 檔案：

    ```csharp
    using Microsoft.Azure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Table;
    ```

### <a name="create-a-model-class"></a>建立模型類別

本文中的許多範例使用 **TableEntity**衍生的類別，稱為 **CustomerEntity**。 下列步驟會引導您宣告這個類別做為模型類別︰

1. 在 [方案總管] 中，用滑鼠右鍵按一下 [模型]，然後從內容功能表中選取 [新增] > [類別]。

1. 在 [新增項目] 對話方塊中，將類別命名為 **CustomerEntity**。

1. 開啟 `CustomerEntity.cs` 檔案，並新增下列 using 指示詞：

    ```csharp
    using Microsoft.WindowsAzure.Storage.Table;
    ```

1. 修改類別，以便在完成時，這個類別會如下列程式碼所示宣告。 類別使用客戶名字作為資料列索引鍵、並使用姓氏作為資料分割索引鍵的，宣告一個稱為 **CustomerEntity** 的實體類別。

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
    }
    ```

## <a name="create-a-table"></a>建立資料表

下列步驟說明如何建立資料表：

> [!NOTE]
> 
> 本章節假設您已完成[設定開發環境](#set-up-the-development-environment)中的步驟。 

1. 開啟 `TablesController.cs` 檔案。

1. 新增名為 **CreateTable** 的方法，其會傳回 **ActionResult**。

    ```csharp
    public ActionResult CreateTable()
    {
        // The code in this section goes here.

        return View();
    }
    ```

1. 在 **CreateTable** 方法內，取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼從 Azure 服務組態中取得儲存體連接字串和儲存體帳戶資訊：(將 &lt;storage-account-name> 變更為您要存取之 Azure 儲存體帳戶的名稱。)
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
    ```

1. 取得 **CloudTableClient** 物件，代表表格服務用戶端。
   
    ```csharp
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```

1. 取得 **CloudTable** 物件，代表所需資料表名稱的參考。 **CloudTableClient.GetTableReference** 方法不會進行對表格儲存體的要求。 無論資料表是否存在都會傳回參考。 
   
    ```csharp
    CloudTable table = tableClient.GetTableReference("TestTable");
    ```

1. 呼叫 **CloudTable.CreateIfNotExists** 方法來建立資料表 (如果尚不存在)。 如果容器不存在且已成功建立，則 **CloudTable.CreateIfNotExists** 方法會傳回 **true**。 否則，會傳回 **false**。    

    ```csharp
    ViewBag.Success = table.CreateIfNotExists();
    ```

1. 使用資料表的名稱更新 **ViewBag**。

    ```csharp
    ViewBag.TableName = table.Name;
    ```

1. 在 [方案總管] 中，展開 [檢視] 資料夾、用滑鼠右鍵按一下 [資料表]，然後從內容功能表中選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 **CreateTable**，然後選取 [新增]。

1. 開啟 `CreateTable.cshtml` 並加以修改，以便其如下列程式碼片段所示：

    ```csharp
    @{
        ViewBag.Title = "Create Table";
    }
    
    <h2>Create Table results</h2>

    Creation of @ViewBag.TableName @(ViewBag.Success == true ? "succeeded" : "failed")
    ```

1. 在 [方案總管] 中，展開 **檢視-> 共用** 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Create table", "CreateTable", "Tables")</li>
    ```

1. 執行應用程式，並選取 **Create table** 來查看類似下列螢幕擷取畫面的結果︰
  
    ![建立資料表](./media/vs-storage-aspnet-getting-started-tables/create-table-results.png)

    如前所述，僅當資料表不存在且已建立時，**CloudTable.CreateIfNotExists** 方法才會傳回 **true**。 因此，如果您在資料表已存在時執行應用程式，此方法會傳回 **false**。 若要多次執行應用程式，您必須先刪除資料表後，才能再次執行應用程式。 可以透過完成 **CloudTable.Delete** 方法來刪除資料表。 您也可以使用 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)或 [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)來刪除資料表。  

## <a name="add-an-entity-to-a-table"></a>將實體新增至資料表

使用衍生自 **TableEntity** 的自訂類別，將實體對應至 C\# 物件。 若要將實體新增至資料表，請建立一個類別來定義實體的屬性。 在本節中，您將了解如何定義一個使用客戶名字作為資料列索引鍵、並使用姓氏作為資料分割索引鍵的實體類別。 實體的資料分割索引鍵和資料列索引鍵共同唯一識別資料表中的實體。 查詢具有相同分割區索引鍵的實體，其速度快於查詢具有不同分割區索引鍵的實體，但使用不同的資料分割索引鍵可提供更佳的延展性或平行作業。 應該儲存在資料表服務中的任何屬性，都必須是公開設定和擷取值之支援類型的公用屬性。
實體類別必須宣告公用的無參數建構函式。

> [!NOTE]
> 
> 本章節假設您已完成[設定開發環境](#set-up-the-development-environment)中的步驟。

1. 開啟 `TablesController.cs` 檔案。

1. 新增下列指示詞，以便 `TablesController.cs` 檔案中的程式碼可以存取 **CustomerEntity** 類別︰

    ```csharp
    using StorageAspnet.Models;
    ```

1. 新增名為 **AddEntity** 的方法，其會傳回 **ActionResult**。

    ```csharp
    public ActionResult AddEntity()
    {
        // The code in this section goes here.

        return View();
    }
    ```

1. 在 **AddEntity** 方法內，取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼從 Azure 服務組態中取得儲存體連接字串和儲存體帳戶資訊：(將 &lt;storage-account-name> 變更為您要存取之 Azure 儲存體帳戶的名稱。)
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
    ```

1. 取得 **CloudTableClient** 物件，代表表格服務用戶端。
   
    ```csharp
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```

1. 取得 **CloudTable** 物件，其代表您要新增新實體的資料表之參考。 
   
    ```csharp
    CloudTable table = tableClient.GetTableReference("TestTable");
    ```

1. 具現化及初始化 **CustomerEntity** 類別。

    ```csharp
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.Email = "Walter@contoso.com";
    ```

1. 建立 **TableOperation** 物件，以插入客戶實體。

    ```csharp
    TableOperation insertOperation = TableOperation.Insert(customer1);
    ```

1. 呼叫 **CloudTable.Execute** 方法以執行插入作業。 您可以檢查 **TableResult.HttpStatusCode** 屬性，以確認作業的結果。 狀態碼 2xx 表示已成功處理用戶端所要求的動作。 例如，成功插入新實體會造成 HTTP 狀態碼 204，這表示已成功處理作業且伺服器未傳回任何內容。

    ```csharp
    TableResult result = table.Execute(insertOperation);
    ```

1. 使用資料表名稱以及插入作業的結果更新 **ViewBag**。

    ```csharp
    ViewBag.TableName = table.Name;
    ViewBag.Result = result.HttpStatusCode;
    ```

1. 在 [方案總管] 中，展開 [檢視] 資料夾、用滑鼠右鍵按一下 [資料表]，然後從內容功能表中選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 **AddEntity**，然後選取 [新增]。

1. 開啟 `AddEntity.cshtml` 並加以修改，以便其如下列程式碼片段所示：

    ```csharp
    @{
        ViewBag.Title = "Add entity";
    }
    
    <h2>Add entity results</h2>

    Insert of entity into @ViewBag.TableName @(ViewBag.Result == 204 ? "succeeded" : "failed")
    ```
1. 在 [方案總管] 中，展開 **檢視-> 共用** 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Add entity", "AddEntity", "Tables")</li>
    ```

1. 執行應用程式，並選取 **Add entity** 來查看類似下列螢幕擷取畫面的結果︰
  
    ![新增實體](./media/vs-storage-aspnet-getting-started-tables/add-entity-results.png)

    您可以確認實體是由章節[取得單一實體](#get-a-single-entity)中的步驟所新增。 您也可以使用 [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)檢視資料表中的所有實體。

## <a name="add-a-batch-of-entities-to-a-table"></a>將一批實體新增至資料表

除了能夠[一次將一個實體新增至一個資料表](#add-an-entity-to-a-table)，您也可以分批新增實體。 在批次中新增實體可減少程式碼與 Azure 資料表服務之間的來回行程次數。 下列步驟說明如何使用單一插入作業將多個實體新增至資料表：

> [!NOTE]
> 
> 本章節假設您已完成[設定開發環境](#set-up-the-development-environment)中的步驟。

1. 開啟 `TablesController.cs` 檔案。

1. 新增名為 **AddEntity** 的方法，其會傳回 **ActionResult**。

    ```csharp
    public ActionResult AddEntities()
    {
        // The code in this section goes here.

        return View();
    }
    ```

1. 在 **AddEntity** 方法內，取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼從 Azure 服務組態中取得儲存體連接字串和儲存體帳戶資訊：(將 &lt;storage-account-name> 變更為您要存取之 Azure 儲存體帳戶的名稱。)
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
    ```

1. 取得 **CloudTableClient** 物件，代表表格服務用戶端。
   
    ```csharp
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```

1. 取得 **CloudTable** 物件，其代表您要新增新實體的資料表之參考。 
   
    ```csharp
    CloudTable table = tableClient.GetTableReference("TestTable");
    ```

1. 根據[將實體新增至資料表](#add-an-entity-to-a-table)章節中呈現的 **CustomerEntity** 模型類別具現化某些客戶物件。

    ```csharp
    CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
    customer1.Email = "Jeff@contoso.com";

    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.Email = "Ben@contoso.com";
    ```

1. 取得 **TableBatchOperation** 物件。

    ```csharp
    TableBatchOperation batchOperation = new TableBatchOperation();
    ```

1. 將實體新增至批次插入作業物件。

    ```csharp
    batchOperation.Insert(customer1);
    batchOperation.Insert(customer2);
    ```

1. 呼叫 **CloudTable.ExecuteBatch** 方法以執行批次插入作業。   

    ```csharp
    IList<TableResult> results = table.ExecuteBatch(batchOperation);
    ```

1. **CloudTable.ExecuteBatch** 方法會傳回一份 **TableResult** 物件，可以檢查其中每個 **TableResult** 物件來判定各項操作的成功或失敗。 例如，將清單傳遞到檢視，讓檢視顯示每個作業的結果。 
 
    ```csharp
    return View(results);
    ```

1. 在 [方案總管] 中，展開 [檢視] 資料夾、用滑鼠右鍵按一下 [資料表]，然後從內容功能表中選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 **AddEntity**，然後選取 [新增]。

1. 開啟 `AddEntities.cshtml` 並加以修改，以便其如下列所示。

    ```csharp
    @model IEnumerable<Microsoft.WindowsAzure.Storage.Table.TableResult>
    @{
        ViewBag.Title = "AddEntities";
    }
    
    <h2>Add-entities results</h2>
    
    <table border="1">
        <tr>
            <th>First name</th>
            <th>Last name</th>
            <th>HTTP result</th>
        </tr>
        @foreach (var result in Model)
        {
        <tr>
            <td>@((result.Result as StorageAspnet.Models.CustomerEntity).RowKey)</td>
            <td>@((result.Result as StorageAspnet.Models.CustomerEntity).PartitionKey)</td>
            <td>@result.HttpStatusCode</td>
        </tr>
        }
    </table>
    ```

1. 在 [方案總管] 中，展開 **檢視-> 共用** 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Add entities", "AddEntities", "Tables")</li>
    ```

1. 執行應用程式，並選取 **Add entity** 來查看類似下列螢幕擷取畫面的結果︰
  
    ![新增實體](./media/vs-storage-aspnet-getting-started-tables/add-entities-results.png)

    您可以確認實體是由章節[取得單一實體](#get-a-single-entity)中的步驟所新增。 您也可以使用 [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)檢視資料表中的所有實體。

## <a name="get-a-single-entity"></a>取得單一實體

本節將說明如何使用實體的資料列索引鍵和資料分割索引鍵，從資料表取得單一實體。 

> [!NOTE]
> 
> 本節假設您已完成[設定開發環境](#set-up-the-development-environment)中的步驟，並使用來自[將一批實體新增至資料表](#add-a-batch-of-entities-to-a-table)的資料。 

1. 開啟 `TablesController.cs` 檔案。

1. 新增名為 **GetSingle** 的方法，其會傳回 **ActionResult**。

    ```csharp
    public ActionResult GetSingle()
    {
        // The code in this section goes here.

        return View();
    }
    ```

1. 在 **GetSingle** 方法內，取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼從 Azure 服務組態中取得儲存體連接字串和儲存體帳戶資訊：(將 &lt;storage-account-name> 變更為您要存取之 Azure 儲存體帳戶的名稱。)
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
    ```

1. 取得 **CloudTableClient** 物件，代表表格服務用戶端。
   
    ```csharp
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```

1. 取得 **CloudTable** 物件，其代表您要擷取實體的資料表之參考。 
   
    ```csharp
    CloudTable table = tableClient.GetTableReference("TestTable");
    ```

1. 建立擷取作業物件，其採用衍生自 **TableEntity** 的實體物件。 第一個參數是 partitionKey，而第二個參數是 rowKey。 使用[將一批實體新增至資料表](#add-a-batch-of-entities-to-a-table)一節中顯示的 **CustomerEntity** 類別和資料，下列程式碼片段可查詢資料表中是否有 *partitionKey* 值為 "Smith" 且 rowKey 值為 "Ben"的 **CustomerEntity** 實體：

    ```csharp
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");
    ```

1. 執行擷取作業。   

    ```csharp
    TableResult result = table.Execute(retrieveOperation);
    ```

1. 將結果傳遞至顯示的檢視。

    ```csharp
    return View(result);
    ```

1. 在 [方案總管] 中，展開 [檢視] 資料夾、用滑鼠右鍵按一下 [資料表]，然後從內容功能表中選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 **GetSingle**，然後選取 [新增]。

1. 開啟 `GetSingle.cshtml` 並加以修改，以便其如下列程式碼片段所示：

    ```csharp
    @model Microsoft.WindowsAzure.Storage.Table.TableResult
    @{
        ViewBag.Title = "GetSingle";
    }
    
    <h2>Get Single results</h2>
    
    <table border="1">
        <tr>
            <th>HTTP result</th>
            <th>First name</th>
            <th>Last name</th>
            <th>Email</th>
        </tr>
        <tr>
            <td>@Model.HttpStatusCode</td>
            <td>@((Model.Result as StorageAspnet.Models.CustomerEntity).RowKey)</td>
            <td>@((Model.Result as StorageAspnet.Models.CustomerEntity).PartitionKey)</td>
            <td>@((Model.Result as StorageAspnet.Models.CustomerEntity).Email)</td>
        </tr>
    </table>
    ```

1. 在 [方案總管] 中，展開 **檢視-> 共用** 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Get single", "GetSingle", "Tables")</li>
    ```

1. 執行應用程式，並選取 **Get Single** 來查看類似下列螢幕擷取畫面的結果︰
  
    ![取得單一](./media/vs-storage-aspnet-getting-started-tables/get-single-results.png)

## <a name="get-all-entities-in-a-partition"></a>取得資料分割中的所有實體

如[將實體加入至資料表](#add-an-entity-to-a-table)章節中所述，資料分割和資料列索引鍵的組合唯一識別資料表中的實體。 相較於查詢具有不同資料分割索引鍵的實體，查詢具有相同資料分割索引鍵的實體速度會較快。 本節將說明如何從指定的資料分割中查詢所有實體的資料表。  

> [!NOTE]
> 
> 本節假設您已完成[設定開發環境](#set-up-the-development-environment)中的步驟，並使用來自[將一批實體新增至資料表](#add-a-batch-of-entities-to-a-table)的資料。 

1. 開啟 `TablesController.cs` 檔案。

1. 新增名為 **GetPartition** 的方法，其會傳回 **ActionResult**。

    ```csharp
    public ActionResult GetPartition()
    {
        // The code in this section goes here.

        return View();
    }
    ```

1. 在 **GetPartition** 方法內，取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼從 Azure 服務組態中取得儲存體連接字串和儲存體帳戶資訊：(將 &lt;storage-account-name> 變更為您要存取之 Azure 儲存體帳戶的名稱。)
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
    ```

1. 取得 **CloudTableClient** 物件，代表表格服務用戶端。
   
    ```csharp
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```

1. 取得 **CloudTable** 物件，其代表您要擷取實體的資料表之參考。 
   
    ```csharp
    CloudTable table = tableClient.GetTableReference("TestTable");
    ```

1. 具現化 **TableQuery** 物件，以在 **Where** 子句中指定查詢。 使用 **CustomerEntity** 類別和[將資料表新增至實體批次](#add-a-batch-of-entities-to-a-table)章節中顯示的資料，下列程式碼片段會查詢所有實體的資料表，其中 **PartitionKey** (客戶的姓氏) 的值為 "Smith":

    ```csharp
    TableQuery<CustomerEntity> query = 
        new TableQuery<CustomerEntity>()
        .Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));
    ```

1. 在迴圈內呼叫 **CloudTable.ExecuteQuerySegmented** 方法，以傳遞您在上一個步驟中具現化的查詢物件。  **CloudTable.ExecuteQuerySegmented** 方法會傳回 **TableContinuationToken** 物件 - 若為 **null**，表示再也沒有要擷取的實體。 在迴圈內，使用另一個迴圈來逐一查看所傳回的實體。 在下列程式碼範例中，每個傳回的實體都會新增至清單。 一旦迴圈結束後，清單會傳遞至檢視顯示︰ 

    ```csharp
    List<CustomerEntity> customers = new List<CustomerEntity>();
    TableContinuationToken token = null;
    do
    {
        TableQuerySegment<CustomerEntity> resultSegment = table.ExecuteQuerySegmented(query, token);
        token = resultSegment.ContinuationToken;

        foreach (CustomerEntity customer in resultSegment.Results)
        {
            customers.Add(customer);
        }
    } while (token != null);

    return View(customers);
    ```

1. 在 [方案總管] 中，展開 [檢視] 資料夾、用滑鼠右鍵按一下 [資料表]，然後從內容功能表中選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 **GetPartition**，然後選取 [新增]。

1. 開啟 `GetPartition.cshtml` 並加以修改，以便其如下列程式碼片段所示：

    ```csharp
    @model IEnumerable<StorageAspnet.Models.CustomerEntity>
    @{
        ViewBag.Title = "GetPartition";
    }
    
    <h2>Get Partition results</h2>
    
    <table border="1">
        <tr>
            <th>First name</th>
            <th>Last name</th>
            <th>Email</th>
        </tr>
        @foreach (var customer in Model)
        {
        <tr>
            <td>@(customer.RowKey)</td>
            <td>@(customer.PartitionKey)</td>
            <td>@(customer.Email)</td>
        </tr>
        }
    </table>
    ```

1. 在 [方案總管] 中，展開 **檢視-> 共用** 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Get partition", "GetPartition", "Tables")</li>
    ```

1. 執行應用程式，並選取 **Get Partition** 來查看類似下列螢幕擷取畫面的結果︰
  
    ![取得資料分割](./media/vs-storage-aspnet-getting-started-tables/get-partition-results.png)

## <a name="delete-an-entity"></a>刪除實體

本節說明如何從資料表刪除實體。

> [!NOTE]
> 
> 本節假設您已完成[設定開發環境](#set-up-the-development-environment)中的步驟，並使用來自[將一批實體新增至資料表](#add-a-batch-of-entities-to-a-table)的資料。 

1. 開啟 `TablesController.cs` 檔案。

1. 新增名為 **DeleteEntity** 的方法，其會傳回 **ActionResult**。

    ```csharp
    public ActionResult DeleteEntity()
    {
        // The code in this section goes here.

        return View();
    }
    ```

1. 在 **DeleteEntity** 方法內，取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼從 Azure 服務組態中取得儲存體連接字串和儲存體帳戶資訊：(將 &lt;storage-account-name> 變更為您要存取之 Azure 儲存體帳戶的名稱。)
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
    ```

1. 取得 **CloudTableClient** 物件，代表表格服務用戶端。
   
    ```csharp
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    ```

1. 取得 **CloudTable** 物件，其代表您要刪除實體的資料表之參考。 
   
    ```csharp
    CloudTable table = tableClient.GetTableReference("TestTable");
    ```

1. 建立刪除作業物件，其採用衍生自 **TableEntity** 的實體物件。 在此案例中，我們使用[將一批實體新增至資料表](#add-a-batch-of-entities-to-a-table)一節中出現的 **CustomerEntity** 類別和資料。 實體的 **ETag** 必須設為有效的值。  

    ```csharp
    TableOperation deleteOperation = 
        TableOperation.Delete(new CustomerEntity("Smith", "Ben") { ETag = "*" } );
    ```

1. 執行刪除作業。   

    ```csharp
    TableResult result = table.Execute(deleteOperation);
    ```

1. 將結果傳遞至顯示的檢視。

    ```csharp
    return View(result);
    ```

1. 在 [方案總管] 中，展開 [檢視] 資料夾、用滑鼠右鍵按一下 [資料表]，然後從內容功能表中選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 **DeleteEntity**，然後選取 [新增]。

1. 開啟 `DeleteEntity.cshtml` 並加以修改，以便其如下列程式碼片段所示：

    ```csharp
    @model Microsoft.WindowsAzure.Storage.Table.TableResult
    @{
        ViewBag.Title = "DeleteEntity";
    }
    
    <h2>Delete Entity results</h2>
    
    <table border="1">
        <tr>
            <th>First name</th>
            <th>Last name</th>
            <th>HTTP result</th>
        </tr>
        <tr>
            <td>@((Model.Result as StorageAspnet.Models.CustomerEntity).RowKey)</td>
            <td>@((Model.Result as StorageAspnet.Models.CustomerEntity).PartitionKey)</td>
            <td>@Model.HttpStatusCode</td>
        </tr>
    </table>

    ```

1. 在 [方案總管] 中，展開 **檢視-> 共用** 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Delete entity", "DeleteEntity", "Tables")</li>
    ```

1. 執行應用程式，並選取 **Delete entity** 來查看類似下列螢幕擷取畫面的結果︰
  
    ![取得單一](./media/vs-storage-aspnet-getting-started-tables/delete-entity-results.png)

## <a name="next-steps"></a>後續步驟
如需了解 Azure 中的其他資料儲存選項，請檢視更多功能指南。

  * [開始使用 Azure Blob 儲存體和 Visual Studio 已連接服務 (ASP.NET)](../storage/vs-storage-aspnet-getting-started-blobs.md)
  * [開始使用 Azure 佇列儲存體和 Visual Studio 已連線的服務 (ASP.NET)](../storage/vs-storage-aspnet-getting-started-queues.md)
