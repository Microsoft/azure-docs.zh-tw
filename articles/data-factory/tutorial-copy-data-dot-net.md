---
title: 將資料從 Azure Blob 儲存體複製到 SQL Database | Microsoft Docs
description: 本教學課程提供逐步指示，將資料從 Azure Blob 儲存體複製到 Azure SQL Database。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
ms.openlocfilehash: 780a9323c07af4754f5afd0ab758b882444a2154
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43107808"
---
# <a name="copy-data-from-azure-blob-to-azure-sql-database-using-azure-data-factory"></a>使用 Azure Data Factory 將資料從 Azure Blob 複製到 Azure SQL Database
在本教學課程中，您會建立 Data Factory 管道，將資料從 Azure Blob 儲存體複製到 Azure SQL Database。 本教學課程中的設定模式從以檔案為基礎的資料存放區複製到關聯式資料存放區。 如需支援作為來源和接收的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立 Azure 儲存體和 Azure SQL Database 連結服務。
> * 建立 Azure Blob 和 Azure SQL Database 資料集。
> * 建立包含複製活動的管道。
> * 啟動管道執行。
> * 監視管道和活動執行。

本教學課程使用 .NET SDK。 您可以使用其他機制來與 Azure Data Factory 互動，請參閱「快速入門」下的範例。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件

* **Azure 儲存體帳戶**。 您會使用 Blob 儲存體作為**來源**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)一文，按照步驟來建立帳戶。
* **Azure SQL Database**。 您會使用資料庫作為**接收**資料存放區。 如果您沒有 Azure SQL Database，請參閱[建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)一文，按照步驟建立資料庫。
* **Visual Studio** 2015 或 2017。 本文中的逐步解說使用 Visual Studio 2017。
* **下載並安裝 [Azure .NET SDK](http://azure.microsoft.com/downloads/)**。
* 依照[此指示](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)**在 Azure Active Directory 中建立應用程式**。 記下這些值，您在稍後的步驟中會用到：**應用程式識別碼**、**驗證金鑰**和**租用戶識別碼**。 遵循相同文件中的指示，將應用程式指派給「**參與者**」角色。

### <a name="create-a-blob-and-a-sql-table"></a>建立 Blob 和 SQL 資料表

現在，請執行下列步驟，以準備本教學課程的 Azure Blob 和 Azure SQL Database：

#### <a name="create-a-source-blob"></a>建立來源 Blob

1. 啟動 [記事本]。 複製下列文字，並在磁碟上儲存為 **inputEmp.txt** 檔案。

    ```
    John|Doe
    Jane|Doe
    ```

2. 使用 [Azure 儲存體總管](http://storageexplorer.com/)之類的工具建立 **adfv2tutorial** 容器，然後將 **inputEmp.txt** 檔案上傳至該容器。

#### <a name="create-a-sink-sql-table"></a>建立接收 SQL 資料表

1. 使用下列 SQL 指令碼，在您的 Azure SQL Database 中建立 **dbo.emp** 資料表。

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO

    CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
    ```

2. 允許 Azure 服務存取 SQL 伺服器。 確定**開啟** Azure SQL 伺服器的 [允許存取 Azure 服務] 設定，讓 Data Factory 服務可以將資料寫入您的 Azure SQL 伺服器。 若要確認並開啟此設定，請執行下列步驟：

    1. 按一下左邊的 [更多服務] 中樞，然後按一下 [SQL Server]。
    2. 選取您的伺服器，然後按一下 [設定] 下的 [防火牆]。
    3. 在 [防火牆設定] 頁面中，對 [允許存取 Azure 服務] 按一下 [開啟]。


## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

使用 Visual Studio 2015/2017 建立 C# .NET 主控台應用程式。

1. 啟動 **Visual Studio**。
2. 按一下 [檔案]，指向 [新增]，然後按一下 [專案]。
3. 從右邊的專案類型清單中，選取 [Visual C#] -> [主控台應用程式 (.NET Framework)]。 需要 .NET 4.5.2 版或更新版本。
4. 在 [名稱] 中輸入 **ADFv2Tutorial**。
5. 按一下 [確定]  以建立專案。

## <a name="install-nuget-packages"></a>安裝 NuGet 套件

1. 按一下 [工具] -> [NuGet 套件管理員] -> [套件管理員主控台]。
2. 在 **Package Manager Console** 中，執行下列命令以安裝封裝：

    ```
    Install-Package Microsoft.Azure.Management.DataFactory -Prerelease
    Install-Package Microsoft.Azure.Management.ResourceManager -Prerelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-data-factory-client"></a>建立資料處理站用戶端

1. 開啟 **Program.cs**，併入下列陳述式以將參考新增至命名空間。

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Rest;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.DataFactory;
    using Microsoft.Azure.Management.DataFactory.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```
    
2. 將下列程式碼新增至 **Main** 方法，以設定變數。 將預留位置取代為您自己的值。 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

    ```csharp
    // Set variables
    string tenantID = "<your tenant ID>";
    string applicationId = "<your application ID>";
    string authenticationKey = "<your authentication key for the application>";
    string subscriptionId = "<your subscription ID to create the factory>";
    string resourceGroup = "<your resource group to create the factory>";

    string region = "East US";
    string dataFactoryName = "<specify the name of a data factory to create. It must be globally unique.>";

    // Specify the source Azure Blob information
    string storageAccount = "<your storage account name to copy data>";
    string storageKey = "<your storage account key>";
    string inputBlobPath = "adfv2tutorial/";
    string inputBlobName = "inputEmp.txt";

    // Specify the sink Azure SQL Database information
    string azureSqlConnString = "Server=tcp:<your server name>.database.windows.net,1433;Database=<your database name>;User ID=<your username>@<your server name>;Password=<your password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30";
    string azureSqlTableName = "dbo.emp";

    string storageLinkedServiceName = "AzureStorageLinkedService";
    string sqlDbLinkedServiceName = "AzureSqlDbLinkedService";
    string blobDatasetName = "BlobDataset";
    string sqlDatasetName = "SqlDataset";
    string pipelineName = "Adfv2TutorialBlobToSqlCopy";
    ```

3. 將下列程式碼新增至 **Main** 方法，以建立 **DataFactoryManagementClient** 類別的執行個體。 您會使用此物件來建立資料處理站、連結服務、資料集和管道。 您也可以使用此物件來監視管線執行的詳細資料。

    ```csharp
    // Authenticate and create a data factory management client
    var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
    ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
    AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
    ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
    var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };
    ```

## <a name="create-a-data-factory"></a>建立 Data Factory

將下列程式碼新增至 **Main** 方法，以建立**資料處理站**。

```csharp
// Create a data factory
Console.WriteLine("Creating a data factory " + dataFactoryName + "...");
Factory dataFactory = new Factory
{
    Location = region,
    Identity = new FactoryIdentity()

};
client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
Console.WriteLine(SafeJsonConvert.SerializeObject(dataFactory, client.SerializationSettings));

while (client.Factories.Get(resourceGroup, dataFactoryName).ProvisioningState == "PendingCreation")
{
    System.Threading.Thread.Sleep(1000);
}
```

## <a name="create-linked-services"></a>建立連結的服務

在本教學課程中，您會建立兩個連結服務，分別是針對來源和接收：

### <a name="create-an-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務

將下列程式碼新增至 **Main** 方法，以建立 **Azure 儲存體連結服務**。 若要深入了解支援的屬性和詳細資料，請參閱 [Azure Blob 連結服務屬性](connector-azure-blob-storage.md#linked-service-properties)。

```csharp
// Create an Azure Storage linked service
Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");

LinkedServiceResource storageLinkedService = new LinkedServiceResource(
    new AzureStorageLinkedService
    {
        ConnectionString = new SecureString("DefaultEndpointsProtocol=https;AccountName=" + storageAccount + ";AccountKey=" + storageKey)
    }
);
client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, storageLinkedService);
Console.WriteLine(SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings));
```

### <a name="create-an-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務

將下列程式碼新增至 **Main** 方法，以建立 **Azure SQL Database 連結服務**。 若要深入了解支援的屬性和詳細資料，請參閱 [Azure SQL Database 連結服務屬性](connector-azure-sql-database.md#linked-service-properties)。

```csharp
// Create an Azure SQL Database linked service
Console.WriteLine("Creating linked service " + sqlDbLinkedServiceName + "...");

LinkedServiceResource sqlDbLinkedService = new LinkedServiceResource(
    new AzureSqlDatabaseLinkedService
    {
        ConnectionString = new SecureString(azureSqlConnString)
    }
);
client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, sqlDbLinkedServiceName, sqlDbLinkedService);
Console.WriteLine(SafeJsonConvert.SerializeObject(sqlDbLinkedService, client.SerializationSettings));
```

## <a name="create-datasets"></a>建立資料集

在本節中，您會建立兩個資料集：一個作為來源，另一個作為接收。 

### <a name="create-a-dataset-for-source-azure-blob"></a>建立來源 Azure Blob 的資料集

將下列程式碼新增至 **Main** 方法，以建立 **Azure Blob 資料集**。 若要深入了解支援的屬性和詳細資料，請參閱 [Azure Blob 資料集屬性](connector-azure-blob-storage.md#dataset-properties)。

您可以定義資料集來代表 Azure Blob 中的來源資料。 此 Blob 資料集會參考您在前一個步驟中建立的 Azure 儲存體連結服務，並描述：

- 複製的來源 Blob 位置：**FolderPath** 和 **FileName**；
- Blob 格式，指出如何剖析內容：**TextFormat** 及其設定 (例如，資料行分隔符號)。
- 資料結構，包括資料行名稱和資料型別，在此案例中是對應至接收 SQL 資料表。

```csharp
// Create a Azure Blob dataset
Console.WriteLine("Creating dataset " + blobDatasetName + "...");
DatasetResource blobDataset = new DatasetResource(
    new AzureBlobDataset
    {
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        },
        FolderPath = inputBlobPath,
        FileName = inputBlobName,
        Format = new TextFormat { ColumnDelimiter = "|" },
        Structure = new List<DatasetDataElement>
        {
            new DatasetDataElement
            {
                Name = "FirstName",
                Type = "String"
            },
            new DatasetDataElement
            {
                Name = "LastName",
                Type = "String"
            }
        }
    }
);
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobDatasetName, blobDataset);
Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
```

### <a name="create-a-dataset-for-sink-azure-sql-database"></a>建立接收 Azure SQL Database 的資料集

將下列程式碼新增至 **Main** 方法，以建立 **Azure SQL Database 資料集**。 若要深入了解支援的屬性和詳細資料，請參閱 [Azure SQL Database 資料集屬性](connector-azure-sql-database.md#dataset-properties)。

您可以定義資料集來代表 Azure SQL Database 中的接收資料。 此資料集會參考您在前一個步驟中建立的 Azure SQL Database 連結服務。 它也會指定 SQL 資料表，其中保存已複製的資料。 

```csharp
// Create a Azure SQL Database dataset
Console.WriteLine("Creating dataset " + sqlDatasetName + "...");
DatasetResource sqlDataset = new DatasetResource(
    new AzureSqlTableDataset
    {
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = sqlDbLinkedServiceName
        },
        TableName = azureSqlTableName
    }
);
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, sqlDatasetName, sqlDataset);
Console.WriteLine(SafeJsonConvert.SerializeObject(sqlDataset, client.SerializationSettings));
```

## <a name="create-a-pipeline"></a>建立管線

將下列程式碼新增至 **Main** 方法，以建立**具有複製活動的管線**。 在本教學課程中，此管道包含一個活動：複製活動，可接受 Blob 資料集作為來源，也接受 SQL 資料集作為接收。 若要深入了解複製活動的詳細資料，請參閱[複製活動概觀](copy-activity-overview.md)。

```csharp
// Create a pipeline with copy activity
Console.WriteLine("Creating pipeline " + pipelineName + "...");
PipelineResource pipeline = new PipelineResource
{
    Activities = new List<Activity>
    {
        new CopyActivity
        {
            Name = "CopyFromBlobToSQL",
            Inputs = new List<DatasetReference>
            {
                new DatasetReference()
                {
                    ReferenceName = blobDatasetName
                }
            },
            Outputs = new List<DatasetReference>
            {
                new DatasetReference
                {
                    ReferenceName = sqlDatasetName
                }
            },
            Source = new BlobSource { },
            Sink = new SqlSink { }
        }
    }
};
client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, pipeline);
Console.WriteLine(SafeJsonConvert.SerializeObject(pipeline, client.SerializationSettings));
```

## <a name="create-a-pipeline-run"></a>建立管道執行

將下列程式碼新增至 **Main** 方法，以**觸發管線執行**。

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="monitor-a-pipeline-run"></a>監視管道執行

1. 將下列程式碼新增至 **Main** 方法，以持續檢查管線執行的狀態，直到完成複製資料為止。

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, runResponse.RunId);
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

2. 將下列程式碼新增至 **Main** 方法，以取出複製活動執行的詳細資料，例如讀取/寫入的資料大小。

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    List<ActivityRun> activityRuns = client.ActivityRuns.ListByPipelineRun(
    resourceGroup, dataFactoryName, runResponse.RunId, DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)).ToList(); 
 
    if (pipelineRun.Status == "Succeeded")
    {
        Console.WriteLine(activityRuns.First().Output);
    }
    else
        Console.WriteLine(activityRuns.First().Error);
    
    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>執行程式碼

建置並啟動應用程式，然後確認管線執行。

在建立資料處理站、連結服務、資料集、管道和管道執行時，主控台會印出進度。 然後會檢查管道執行狀態。 請等待出現複製活動執行詳細資料及讀取/寫入的資料大小。 然後，使用 SSMS (SQL Server Management Studio) 或 Visual Studio 之類的工具，連線至目的地 Azure SQL Database，檢查資料是否已複製到您指定的資料表。

### <a name="sample-output"></a>範例輸出

```json
Creating a data factory AdfV2Tutorial...
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>"
      }
    }
  }
}
Creating linked service AzureSqlDbLinkedService...
{
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
      }
    }
  }
}
Creating dataset BlobDataset...
{
  "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": "adfv2tutorial/",
      "fileName": "inputEmp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "|"
      }
    },
    "structure": [
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureStorageLinkedService"
    }
  }
}
Creating dataset SqlDataset...
{
  "properties": {
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": "dbo.emp"
    },
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureSqlDbLinkedService"
    }
  }
}
Creating pipeline Adfv2TutorialBlobToSqlCopy...
{
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "inputs": [
          {
            "type": "DatasetReference",
            "referenceName": "BlobDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SqlDataset"
          }
        ],
        "name": "CopyFromBlobToSQL"
      }
    ]
  }
}
Creating pipeline run...
Pipeline run ID: 1cd03653-88a0-4c90-aabc-ae12d843e252
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 18,
  "dataWritten": 28,
  "rowsCopied": 2,
  "copyDuration": 2,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)",
  "usedDataIntegrationUnits": 2,
  "billedDuration": 2
}

Press any key to exit...
```


## <a name="next-steps"></a>後續步驟
在此範例中的管線會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 您已了解如何︰ 

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立 Azure 儲存體和 Azure SQL Database 連結服務。
> * 建立 Azure Blob 和 Azure SQL Database 資料集。
> * 建立包含複製活動的管道。
> * 啟動管道執行。
> * 監視管線和活動執行。


進入下列教學課程，以了解如何將資料從內部部署複製到雲端： 

> [!div class="nextstepaction"]
>[將資料從內部部署複製到雲端](tutorial-hybrid-copy-powershell.md)
