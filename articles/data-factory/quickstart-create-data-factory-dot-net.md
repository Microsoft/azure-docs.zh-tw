---
title: 使用 .NET 建立 Azure 資料處理站 | Microsoft Docs
description: 建立 Azure 資料處理站，以將資料從 Azure Blob 儲存體中的某個位置複製到另一個位置。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 03/28/2018
ms.author: jingwang
ms.openlocfilehash: a7916a434552cbcb999f1e69c7a5bc2419f517fb
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43094337"
---
# <a name="create-a-data-factory-and-pipeline-using-net-sdk"></a>使用 .NET SDK 建立資料處理站和管線
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [目前的版本](quickstart-create-data-factory-dot-net.md)

本快速入門說明如何使用 .NET SDK 來建立 Azure 資料處理站。 在此資料處理站中建立的管線會將資料從 Azure Blob 儲存體中的一個資料夾**複製**到其他資料夾。 如需如何使用 Azure Data Factory **轉換**資料的教學課程，請參閱[教學課程︰使用 Spark 轉換資料](transform-data-using-spark.md)。 

> [!NOTE]
> 本文不提供 Data Factory 服務的詳細簡介。 如需 Azure Data Factory 服務簡介，請參閱 [Azure Data Factory 簡介](introduction.md)。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)] 

### <a name="visual-studio"></a>Visual Studio
本文中的逐步解說使用 Visual Studio 2017。 您也可以使用 Visual Studio 2013 或 2015。

### <a name="azure-net-sdk"></a>Azure .NET SDK
在您的電腦上下載並安裝 [Azure .NET SDK](http://azure.microsoft.com/downloads/)。

## <a name="create-an-application-in-azure-active-directory"></a>在 Azure Active Directory 中建立應用程式
依照[本文](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)各節中的指示執行下列工作： 

1. **建立 Azure Active Directory 應用程式**。 在 Azure Active Directory 建立應用程式，代表您要在本教學課程中建立的 .NET 應用程式。 針對登入 URL，您可以提供虛擬 URL，如文章中所示 (`https://contoso.org/exampleapp`)。
2. 取得**應用程式識別碼**和**驗證金鑰**，並記下您稍後在本教學課程中會用到的值。 
3. 取得**租用戶識別碼**並記下您稍後在本教學課程中會用到的值。
4. 將應用程式指派給訂用帳戶層級的**參與者**角色，讓應用程式可以在訂用帳戶中建立 Data Factory。

## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

使用 Visual Studio 2013/2015/2017 建立 C# .NET 主控台應用程式。

1. 啟動 **Visual Studio**。
2. 按一下 [檔案]，指向 [新增]，然後按一下 [專案]。
3. 從右邊的專案類型清單中，選取 [Visual C#] -> [主控台應用程式 (.NET Framework)]。 需要 .NET 4.5.2 版或更新版本。
4. 輸入 **ADFv2QuickStart** 作為名稱。
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
    string subscriptionId = "<your subscription ID where the data factory resides>";
    string resourceGroup = "<your resource group where the data factory resides>";
    string region = "East US 2";
    string dataFactoryName = "<specify the name of data factory to create. It must be globally unique.>";
    string storageAccount = "<your storage account name to copy data>";
    string storageKey = "<your storage account key>";
    // specify the container and input folder from which all files need to be copied to the output folder. 
    string inputBlobPath = "<the path to existing blob(s) to copy data from, e.g. containername/foldername>";
    //specify the contains and output folder where the files are copied
    string outputBlobPath = "<the blob path to copy data to, e.g. containername/foldername>";

    string storageLinkedServiceName = "AzureStorageLinkedService";  // name of the Azure Storage linked service
    string blobDatasetName = "BlobDataset";             // name of the blob dataset
    string pipelineName = "Adfv2QuickStartPipeline";    // name of the pipeline
    ```

3. 將下列程式碼新增至 **Main** 方法，以建立 **DataFactoryManagementClient** 類別的執行個體。 您會使用此物件來建立資料處理站、連結服務、資料集和管線。 您也可以使用此物件來監視管線執行的詳細資料。

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
Console.WriteLine("Creating data factory " + dataFactoryName + "...");
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

## <a name="create-a-linked-service"></a>建立連結的服務

將下列程式碼新增至 **Main** 方法，以建立 **Azure 儲存體連結服務**。

您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在此快速入門中，您只需要為複製來源與接收存放區建立一個 Azure 儲存體連結服務，在此範例中名為 "AzureStorageLinkedService"。

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

## <a name="create-a-dataset"></a>建立資料集

將下列程式碼新增至 **Main** 方法，以建立 **Azure Blob 資料集**。

您可以定義資料集來代表要從來源複製到接收的資料。 在此範例中，此 Blob 資料集會參考您在上一個步驟中建立的 Azure 儲存體連結服務。 資料集會使用一個參數，其值在取用資料集的活動中設定。 該參數用來建構 "folderPath"，指向資料位於/儲存所在的位置。

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
        FolderPath = new Expression { Value = "@{dataset().path}" },
        Parameters = new Dictionary<string, ParameterSpecification>
        {
            { "path", new ParameterSpecification { Type = ParameterType.String } }

        }
    }
);
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobDatasetName, blobDataset);
Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
```

## <a name="create-a-pipeline"></a>建立管線

將下列程式碼新增至 **Main** 方法，以建立**具有複製活動的管線**。

在此範例中，此管線包含一個活動，並使用兩個參數，輸入的 Blob 路徑和輸出 Blob 路徑。 這些參數的值是在觸發/執行管線時設定。 複製活動指的是在前一個步驟中建立作為輸入和輸出的同一 Blob 資料集。 將該資料集用作輸入資料集時，即會指定輸入路徑。 並且，將該資料集用作輸出資料集時，即會指定輸出路徑。 

```csharp
// Create a pipeline with a copy activity
Console.WriteLine("Creating pipeline " + pipelineName + "...");
PipelineResource pipeline = new PipelineResource
{
    Parameters = new Dictionary<string, ParameterSpecification>
    {
        { "inputPath", new ParameterSpecification { Type = ParameterType.String } },
        { "outputPath", new ParameterSpecification { Type = ParameterType.String } }
    },
    Activities = new List<Activity>
    {
        new CopyActivity
        {
            Name = "CopyFromBlobToBlob",
            Inputs = new List<DatasetReference>
            {
                new DatasetReference()
                {
                    ReferenceName = blobDatasetName,
                    Parameters = new Dictionary<string, object>
                    {
                        { "path", "@pipeline().parameters.inputPath" }
                    }
                }
            },
            Outputs = new List<DatasetReference>
            {
                new DatasetReference
                {
                    ReferenceName = blobDatasetName,
                    Parameters = new Dictionary<string, object>
                    {
                        { "path", "@pipeline().parameters.outputPath" }
                    }
                }
            },
            Source = new BlobSource { },
            Sink = new BlobSink { }
        }
    }
};
client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, pipeline);
Console.WriteLine(SafeJsonConvert.SerializeObject(pipeline, client.SerializationSettings));
```

## <a name="create-a-pipeline-run"></a>建立管線執行

將下列程式碼新增至 **Main** 方法，以**觸發管線執行**。

此程式碼也會以來源和接收 Blob 路徑的實際值，設定在管線中指定的 **inputPath** 和 **outputPath** 參數的值。

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> parameters = new Dictionary<string, object>
{
    { "inputPath", inputBlobPath },
    { "outputPath", outputBlobPath }
};
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, parameters: parameters).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="monitor-a-pipeline-run"></a>監視管線執行

1. 將下列程式碼新增至 **Main** 方法，以持續檢查狀態，直到完成複製資料為止。

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
        Console.WriteLine(activityRuns.First().Output);
    else
        Console.WriteLine(activityRuns.First().Error);
    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>執行程式碼

建置並啟動應用程式，然後確認管線執行。

主控台會印出建立資料處理站、連結服務、資料集、管線和管線執行的進度。 然後會檢查管線執行狀態。 請等待出現複製活動執行詳細資料及讀取/寫入的資料大小。 然後，使用 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)之類的工具，檢查 Blob 已從 "inputBlobPath" 複製到 "outputBlobPath" (您在變數中指定)。

### <a name="sample-output"></a>範例輸出： 
```json
Creating data factory SPv2Factory0907...
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
        "value": "DefaultEndpointsProtocol=https;AccountName=<storageAccountName>;AccountKey=<storageAccountKey>",
        "type": "SecureString"
      }
    }
  }
}
Creating dataset BlobDataset...
{
  "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@{dataset().path}",
        "type": "Expression"
      }
    },
    "linkedServiceName": {
      "referenceName": "AzureStorageLinkedService",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "path": {
        "type": "String"
      }
    }
  }
}
Creating pipeline Adfv2QuickStartPipeline...
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
            "type": "BlobSink"
          }
        },
        "inputs": [
          {
            "referenceName": "BlobDataset",
            "parameters": {
              "path": "@pipeline().parameters.inputPath"
            },
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "BlobDataset",
            "parameters": {
              "path": "@pipeline().parameters.outputPath"
            },
            "type": "DatasetReference"
          }
        ],
        "name": "CopyFromBlobToBlob"
      }
    ],
    "parameters": {
      "inputPath": {
        "type": "String"
      },
      "outputPath": {
        "type": "String"
      }
    }
  }
}
Creating pipeline run...
Pipeline run ID: 308d222d-3858-48b1-9e66-acd921feaa09
Checking pipeline run status...
Status: InProgress
Status: InProgress
Checking copy activity run details...
{
    "dataRead": 331452208,
    "dataWritten": 331452208,
    "copyDuration": 23,
    "throughput": 14073.209,
    "errors": [],
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (West US)",
    "usedDataIntegrationUnits": 2,
    "billedDuration": 23
}

Press any key to exit...
```

## <a name="verify-the-output"></a>驗證輸出
管道會自動在 adftutorial Blob 容器中建立輸出資料夾。 然後，它會將 emp.txt 檔案從輸入資料夾複製到輸出資料夾。 

1. 在 Azure 入口網站的 [adftutorial] 容器頁面上，按一下 [重新整理] 可查看輸出資料夾。 
    
    ![重新整理](media/quickstart-create-data-factory-dot-net/output-refresh.png)
2. 在資料夾清單中按一下 [輸出]。 
2. 確認 **emp.txt** 已複製到輸出資料夾。 

    ![重新整理](media/quickstart-create-data-factory-dot-net/output-file.png)

## <a name="clean-up-resources"></a>清除資源
若要以程式設計方式刪除資料處理站，請將下列程式碼新增至程式： 

```csharp
            Console.WriteLine("Deleting the data factory");
            client.Factories.Delete(resourceGroup, dataFactoryName);
```

## <a name="next-steps"></a>後續步驟
在此範例中的管線會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 瀏覽[教學課程](tutorial-copy-data-dot-net.md)以了解使用 Data Factory 的更多案例。 
