---
title: 在 Azure Data Factory 管道中分支 | Microsoft Docs
description: 了解如何將活動分支和鏈結來控制 Azure Data Factory 中的資料流程。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: shlo
ms.openlocfilehash: b492635da55ae08f92b18dcf9c030cb23d4fa48c
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665061"
---
# <a name="branching-and-chaining-activities-in-a-data-factory-pipeline"></a>在 Data Factory 管道中將活動分支和鏈結
在本教學課程中，您會建立 Data Factory 管道來展示部分的控制流程功能。 這個管道只是簡單地從 Azure Blob 儲存體中的一個容器複製到相同儲存體帳戶中的另一個容器。 如果複製活動成功，您希望透過成功電子郵件傳送成功複製作業的詳細資料 (例如寫入的資料量)。 如果複製活動失敗，您希望透過失敗電子郵件傳送複製失敗的詳細資料 (例如錯誤訊息)。 在整個教學課程中，您會看到如何傳遞參數。

情節的高階概觀：![概觀](media/tutorial-control-flow/overview.png)

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立 Azure 儲存體連結服務。
> * 建立 Azure Blob 資料集
> * 建立包含複製活動和網路活動的管道
> * 將活動的輸出傳送至後續的活動
> * 利用參數傳遞和系統變數
> * 啟動管道執行
> * 監視管道和活動執行

本教學課程使用 .NET SDK。 您可以使用其他機制來與 Azure Data Factory 互動，請參閱目錄中的「快速入門」。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件

* **Azure 儲存體帳戶**。 您會使用 Blob 儲存體作為**來源**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)一文，按照步驟來建立帳戶。
* **Azure SQL Database**。 您會使用資料庫作為**接收**資料存放區。 如果您沒有 Azure SQL Database，請參閱[建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)一文，按照步驟建立資料庫。
* **Visual Studio** 2013、2015 或 2017。 本文中的逐步解說使用 Visual Studio 2017。
* **下載並安裝 [Azure .NET SDK](http://azure.microsoft.com/downloads/)**。
* 依照[這些指示](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)**在 Azure Active Directory 中建立應用程式**。 記下這些值，您在稍後的步驟中會用到：**應用程式識別碼**、**驗證金鑰**和**租用戶識別碼**。 依照相同文件中的指示，將應用程式指派給「**參與者**」角色。

### <a name="create-blob-table"></a>建立 Blob 資料表

1. 啟動 [記事本]。 複製下列文字，並在磁碟上儲存為 **input.txt** 檔案。

    ```
    John|Doe
    Jane|Doe
    ```
2. 使用 [Azure 儲存體總管](http://storageexplorer.com/)之類的工具建立 **adfv2branch** 容器，然後將 **input.txt** 檔案上傳至該容器。

## <a name="create-visual-studio-project"></a>建立 Visual Studio 專案

使用 Visual Studio 2015/2017 建立 C# .NET 主控台應用程式。

1. 啟動 **Visual Studio**。
2. 按一下 [檔案]，指向 [新增]，然後按一下 [專案]。 需要 .NET 4.5.2 版或更新版本。
3. 從右邊的專案類型清單中，選取 [Visual C#] -> [主控台應用程式 (.NET Framework)]。
4. 在 [名稱] 中輸入 **ADFv2BranchTutorial**。
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

2. 將這些靜態變數新增至 **Program 類別**。 將預留位置取代為您自己的值。 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

    ```csharp
        // Set variables
        static string tenantID = "<tenant ID>";
        static string applicationId = "<application ID>";
        static string authenticationKey = "<Authentication key for your application>";
        static string subscriptionId = "<Azure subscription ID>";
        static string resourceGroup = "<Azure resource group name>";

        static string region = "East US";
        static string dataFactoryName = "<Data factory name>";

        // Specify the source Azure Blob information
        static string storageAccount = "<Azure Storage account name>";
        static string storageKey = "<Azure Storage account key>";
        // confirm that you have the input.txt file placed in th input folder of the adfv2branch container. 
        static string inputBlobPath = "adfv2branch/input";
        static string inputBlobName = "input.txt";
        static string outputBlobPath = "adfv2branch/output";
        static string emailReceiver = "<specify email address of the receiver>";

        static string storageLinkedServiceName = "AzureStorageLinkedService";
        static string blobSourceDatasetName = "SourceStorageDataset";
        static string blobSinkDatasetName = "SinkStorageDataset";
        static string pipelineName = "Adfv2TutorialBranchCopy";

        static string copyBlobActivity = "CopyBlobtoBlob";
        static string sendFailEmailActivity = "SendFailEmailActivity";
        static string sendSuccessEmailActivity = "SendSuccessEmailActivity";
    
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
在 Program.cs 檔案中，建立 "CreateOrUpdateDataFactory" 函式：

```csharp
static Factory CreateOrUpdateDataFactory(DataFactoryManagementClient client)
{
    Console.WriteLine("Creating data factory " + dataFactoryName + "...");
    Factory resource = new Factory
    {
        Location = region
    };
    Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));

    Factory response;
    {
        response = client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, resource);
    }

    while (client.Factories.Get(resourceGroup, dataFactoryName).ProvisioningState == "PendingCreation")
    {
        System.Threading.Thread.Sleep(1000);
    }
    return response;
}
```



將下列程式碼新增至 **Main** 方法，以建立**資料處理站**。 

```csharp
Factory df = CreateOrUpdateDataFactory(client);
```

## <a name="create-an-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
在 Program.cs 檔案中，建立 "StorageLinkedServiceDefinition" 函式：

```csharp
static LinkedServiceResource StorageLinkedServiceDefinition(DataFactoryManagementClient client)
{
    Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");
    AzureStorageLinkedService storageLinkedService = new AzureStorageLinkedService
    {
        ConnectionString = new SecureString("DefaultEndpointsProtocol=https;AccountName=" + storageAccount + ";AccountKey=" + storageKey)
    };
    Console.WriteLine(SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings));
    LinkedServiceResource linkedService = new LinkedServiceResource(storageLinkedService, name:storageLinkedServiceName);
    return linkedService;
}
```
將下列程式碼新增至 **Main** 方法，以建立 **Azure 儲存體連結服務**。 若要深入了解支援的屬性和詳細資料，請參閱 [Azure Blob 連結服務屬性](connector-azure-blob-storage.md#linked-service-properties)。

```csharp
client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
```

## <a name="create-datasets"></a>建立資料集

在本節中，您會建立兩個資料集：一個作為來源，另一個作為接收。 

### <a name="create-a-dataset-for-source-azure-blob"></a>建立來源 Azure Blob 的資料集
將下列程式碼新增至 **Main** 方法，以建立 **Azure Blob 資料集**。 若要深入了解支援的屬性和詳細資料，請參閱 [Azure Blob 資料集屬性](connector-azure-blob-storage.md#dataset-properties)。

您可以定義資料集來代表 Azure Blob 中的來源資料。 此 Blob 資料集會參考您在前一個步驟中建立的 Azure 儲存體連結服務，並描述：

- 複製的來源 Blob 位置：**FolderPath** 和 **FileName**。
- 請注意 FolderPath 的參數用法。 “sourceBlobContainer” 是參數的名稱，而傳入管道執行中的值會取代運算式。 定義參數的語法是 `@pipeline().parameters.<parameterName>`

在 Program.cs 檔案中，建立 “SourceBlobDatasetDefinition” 函式

```csharp
static DatasetResource SourceBlobDatasetDefinition(DataFactoryManagementClient client)
{
    Console.WriteLine("Creating dataset " + blobSourceDatasetName + "...");
    AzureBlobDataset blobDataset = new AzureBlobDataset
    { 
        FolderPath = new Expression { Value = "@pipeline().parameters.sourceBlobContainer" },
        FileName = inputBlobName,
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        }
    };
    Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
    DatasetResource dataset = new DatasetResource(blobDataset, name:blobSourceDatasetName);
    return dataset;
}
```

### <a name="create-a-dataset-for-sink-azure-blob"></a>建立接收 Azure Blob 的資料集

在 Program.cs 檔案中，建立 “SourceBlobDatasetDefinition” 函式

```csharp
static DatasetResource SinkBlobDatasetDefinition(DataFactoryManagementClient client)
{
    Console.WriteLine("Creating dataset " + blobSinkDatasetName + "...");
    AzureBlobDataset blobDataset = new AzureBlobDataset
    {
        FolderPath = new Expression { Value = "@pipeline().parameters.sinkBlobContainer" },
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        }
    };
    Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
    DatasetResource dataset = new DatasetResource(blobDataset, name: blobSinkDatasetName);
    return dataset;
}
```

將下列程式碼新增至 **Main** 方法，以建立 Azure Blob 來源和接收資料集。 

```csharp
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));

client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));
```

## <a name="create-a-c-class-emailrequest"></a>建立 C# 類別：EmailRequest
在 C# 專案中，建立名為 **EmailRequest** 的類別。 這會定義傳送電子郵件時，管道在本文要求中會傳送哪些屬性。 在本教學課程中，管道會將四個屬性從管道傳送至電子郵件：

- **訊息**：電子郵件的本文。 如果複製成功，此屬性會包含執行的詳細資料 (寫入的資料數量)。 如果複製失敗，此屬性會包含錯誤的詳細資料。
- **資料處理站名稱**：資料處理站的名稱
- **管道名稱**：管道的名稱
- **接收者**：已傳遞的參數。 此屬性指定電子郵件的接收者。

```csharp
    class EmailRequest
    {
        [Newtonsoft.Json.JsonProperty(PropertyName = "message")]
        public string message;

        [Newtonsoft.Json.JsonProperty(PropertyName = "dataFactoryName")]
        public string dataFactoryName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "pipelineName")]
        public string pipelineName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "receiver")]
        public string receiver;

        public EmailRequest(string input, string df, string pipeline, string receiverName)
        {
            message = input;
            dataFactoryName = df;
            pipelineName = pipeline;
            receiver = receiverName;
        }
    }
```
## <a name="create-email-workflow-endpoints"></a>建立電子郵件工作流程端點
若要觸發傳送電子郵件，您需要使用 [Logic Apps](../logic-apps/logic-apps-overview.md) 來定義工作流程。 如需有關建立邏輯應用程式工作流程的詳細資訊，請參閱[如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

### <a name="success-email-workflow"></a>成功電子郵件工作流程 
建立名為 `CopySuccessEmail` 的邏輯應用程式工作流程。 將工作流程觸發程序定義為 `When an HTTP request is received`，並新增動作 `Office 365 Outlook – Send an email`。

![成功電子郵件工作流程](media/tutorial-control-flow/success-email-workflow.png)

針對您的要求觸發程序，在 `Request Body JSON Schema` 中填寫下列 JSON：

```json
{
    "properties": {
        "dataFactoryName": {
            "type": "string"
        },
        "message": {
            "type": "string"
        },
        "pipelineName": {
            "type": "string"
        },
        "receiver": {
            "type": "string"
        }
    },
    "type": "object"
}
```
這與您在上一節建立的 **EmailRequest** 類別一致。 

在邏輯應用程式設計工具中，您的要求看起來應該如下所示：

![邏輯應用程式設計工具 - 要求](media/tutorial-control-flow/logic-app-designer-request.png)

對於**傳送電子郵件**動作，利用要求本文 JSON 結構描述中傳遞的屬性，以自訂您要如何格式化電子郵件。 下列是一個範例：

![邏輯應用程式設計工具 - 傳送電子郵件動作](media/tutorial-control-flow/send-email-action.png)

請記下成功電子郵件工作流程的 HTTP Post 要求 URL：

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

## <a name="fail-email-workflow"></a>失敗電子郵件工作流程 
複製 **CopySuccessEmail**，並建立另一個 Logic Apps 工作流程 **CopyFailEmail**。 在要求觸發程序中，`Request Body JSON schema` 相同。 簡單地變更電子郵件的格式 (例如 `Subject`)，以調整為適合失敗電子郵件。 下列是一個範例：

![邏輯應用程式設計工具 - 失敗電子郵件工作流程](media/tutorial-control-flow/fail-email-workflow.png)

請記下失敗電子郵件工作流程的 HTTP Post 要求 URL：

```
//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

您現在應該有兩個工作流程 URL：

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000

//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```
## <a name="create-a-pipeline"></a>建立管線
將下列程式碼新增至 Main 方法，以建立具有複製活動和 dependsOn 屬性的管道。 在本教學課程中，此管道包含一個活動：複製活動，可接受 Blob 資料集作為來源，並接受另一個 Blob 資料集作為接收。 根據複製活動成功或失敗，它會呼叫不同的電子郵件工作。

在此管道中，您會使用下列功能：

- 參數
- 網路活動
- 活動相依性
- 使用一個活動的輸出作為後續活動的輸入

讓我們逐段分解下列管道：

```csharp

static PipelineResource PipelineDefinition(DataFactoryManagementClient client)
        {
            Console.WriteLine("Creating pipeline " + pipelineName + "...");
            PipelineResource resource = new PipelineResource
            {
                Parameters = new Dictionary<string, ParameterSpecification>
                {
                    { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                    { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                    { "receiver", new ParameterSpecification { Type = ParameterType.String } }

                },
                Activities = new List<Activity>
                {
                    new CopyActivity
                    {
                        Name = copyBlobActivity,
                        Inputs = new List<DatasetReference>
                        {
                            new DatasetReference
                            {
                                ReferenceName = blobSourceDatasetName
                            }
                        },
                        Outputs = new List<DatasetReference>
                        {
                            new DatasetReference
                            {
                                ReferenceName = blobSinkDatasetName
                            }
                        },
                        Source = new BlobSource { },
                        Sink = new BlobSink { }
                    },
                    new WebActivity
                    {
                        Name = sendSuccessEmailActivity,
                        Method = WebActivityMethod.POST,
                        Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/00000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000000",
                        Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                        DependsOn = new List<ActivityDependency>
                        {
                            new ActivityDependency
                            {
                                Activity = copyBlobActivity,
                                DependencyConditions = new List<String> { "Succeeded" }
                            }
                        }
                    },
                    new WebActivity
                    {
                        Name = sendFailEmailActivity,
                        Method =WebActivityMethod.POST,
                        Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000",
                        Body = new EmailRequest("@{activity('CopyBlobtoBlob').error.message}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                        DependsOn = new List<ActivityDependency>
                        {
                            new ActivityDependency
                            {
                                Activity = copyBlobActivity,
                                DependencyConditions = new List<String> { "Failed" }
                            }
                        }
                    }
                }
            };
            Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));
            return resource;
        }
```
將下列程式碼新增至 **Main** 方法，以建立管道：

```
client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));
```
### <a name="parameters"></a>參數
管道的第一個區段定義參數。 

- sourceBlobContainer - 管道中由來源 Blob 資料集所取用的參數。
- sinkBlobContainer – 管道中由接收 Blob 資料集所取用的參數
- 接收者 – 管線中的兩項 Web 活動會使用此參數，將成功或失敗電子郵件傳送給其電子郵件地址由此參數指定的接收者。


```csharp
Parameters = new Dictionary<string, ParameterSpecification>
    {
        { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "receiver", new ParameterSpecification { Type = ParameterType.String } }
    },
```
### <a name="web-activity"></a>網路活動
「網路活動」允許呼叫任何 REST 端點。 如需活動的詳細資訊，請參閱[網路活動](control-flow-web-activity.md)。 這個管道會使用「網路活動」來呼叫 Logic Apps 電子郵件工作流程。 您會建立兩個網路活動：其中一個會呼叫 **CopySuccessEmail** 工作流程，另一個會呼叫 **CopyFailWorkFlow**。

```csharp
        new WebActivity
        {
            Name = sendCopyEmailActivity,
            Method = WebActivityMethod.POST,
            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/12345",
            Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
            DependsOn = new List<ActivityDependency>
            {
                new ActivityDependency
                {
                    Activity = copyBlobActivity,
                    DependencyConditions = new List<String> { "Succeeded" }
                }
            }
        }
```
在 “Url” 屬性中，從您的 Logic Apps 工作流程貼上相應的要求 URL 端點。 在「本文」屬性中，傳遞 “EmailRequest” 類別的執行個體。 電子郵件要求包含下列屬性：

- 訊息 – 傳遞 `@{activity('CopyBlobtoBlob').output.dataWritten` 的值。 存取先前複製活動的屬性，並傳遞 dataWritten 的值。 對於失敗案例，請傳遞錯誤輸出，而不是 `@{activity('CopyBlobtoBlob').error.message`。
- 資料處理站名稱 – 傳遞 `@{pipeline().DataFactory}` 的值。這是系統變數，可讓您存取對應的資料處理站名稱。 如需系統變數的清單，請參閱[系統變數](control-flow-system-variables.md)一文。
- 管道名稱 - 傳遞 `@{pipeline().Pipeline}` 的值。 這也是系統變數，可讓您存取對應的管道名稱。 
- 接收者 – 傳遞 "\@pipeline().parameters.receiver") 的值。 存取管道參數。
 
此程式碼會建立新的活動相依性，取決於它接替的前一個複製活動而定。

## <a name="create-a-pipeline-run"></a>建立管線執行
將下列程式碼新增至 **Main** 方法，以**觸發管道執行**。

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};
 
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="main-class"></a>Main 類別 
最終的 Main 方法看起來應該如下所示。 建置並執行您的程式來觸發管道執行！

```csharp
// Authenticate and create a data factory management client
var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };

Factory df = CreateOrUpdateDataFactory(client);

client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));

client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));

Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};

CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
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
        //SaveToJson(SafeJsonConvert.SerializeObject(activityRuns.First().Output, client.SerializationSettings), "ActivityRunResult.json", folderForJsons);
    }
    else
        Console.WriteLine(activityRuns.First().Error);

    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>執行程式碼
建置並啟動應用程式，然後確認管線執行。
主控台會印出建立資料處理站、連結服務、資料集、管線和管線執行的進度。 然後會檢查管線執行狀態。 請等待出現複製活動執行詳細資料及讀取/寫入的資料大小。 然後，使用 Azure 儲存體總管之類的工具，檢查 Blob 已從 "inputBlobPath" 複製到 "outputBlobPath" (您在變數中指定)。

**範例輸出：**

```json
Creating data factory DFTutorialTest...
{
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "type": "AzureStorage",
  "typeProperties": {
    "connectionString": {
      "type": "SecureString",
      "value": "DefaultEndpointsProtocol=https;AccountName=***;AccountKey=***"
    }
  }
}
Creating dataset SourceStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sourceBlobContainer"
    },
    "fileName": "input.txt"
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating dataset SinkStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sinkBlobContainer"
    }
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating pipeline Adfv2TutorialBranchCopy...
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
            "type": "DatasetReference",
            "referenceName": "SourceStorageDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SinkStorageDataset"
          }
        ],
        "name": "CopyBlobtoBlob"
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').output.dataWritten}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendSuccessEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Succeeded"
            ]
          }
        ]
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').error.message}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendFailEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Failed"
            ]
          }
        ]
      }
    ],
    "parameters": {
      "sourceBlobContainer": {
        "type": "String"
      },
      "sinkBlobContainer": {
        "type": "String"
      },
      "receiver": {
        "type": "String"
      }
    }
  }
}
Creating pipeline run...
Pipeline run ID: 00000000-0000-0000-0000-0000000000000
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 20,
  "dataWritten": 20,
  "copyDuration": 4,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
}
{}

Press any key to exit...
```

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已執行下列步驟： 

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立 Azure 儲存體連結服務。
> * 建立 Azure Blob 資料集
> * 建立包含複製活動和網路活動的管道
> * 將活動的輸出傳送至後續的活動
> * 利用參數傳遞和系統變數
> * 啟動管道執行
> * 監視管道和活動執行

您現在可以繼續閱讀＜概念＞一節，以了解 Azure Data Factory 的詳細資訊。
> [!div class="nextstepaction"]
>[管線和活動](concepts-pipelines-activities.md)
