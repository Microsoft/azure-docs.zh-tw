---
title: 教學課程：使用 .NET API 建立具有複製活動的管線 | Microsoft Docs
description: 在本教學課程中，您會使用 .NET API，建立具有複製活動的 Azure Data Factory 管線。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 58fc4007-b46d-4c8e-a279-cb9e479b3e2b
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 991dc661c40f96a1c167821d76c01ea62d62dc52
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43089063"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-net-api"></a>教學課程：使用 .NET API 建立具有複製活動的管線
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [複製精靈](data-factory-copy-data-wizard-tutorial.md)
> * [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Azure Resource Manager 範本](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[複製活動教學課程](../quickstart-create-data-factory-dot-net.md)。 

在本文中，您會了解如何使用 [.NET API](https://portal.azure.com) 建立資料處理站，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫。 如果您不熟悉 Azure Data Factory，請先詳閱 [Azure Data Factory 簡介](data-factory-introduction.md)一文，再進行本教學課程。   

在本教學課程中，您可以建立包含一個活動的管線：複製活動。 複製活動會將資料從支援的資料存放區複製到支援的接收資料存放區。 如需作為來源和接收區支援的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。

一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[管線中的多個活動](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。 

> [!NOTE] 
> 如需適用於 Data Factory 之 .NET API 的完整文件，請參閱 [Data Factory .NET API 參考](/dotnet/api/index?view=azuremgmtdatafactories-4.12.1)。
> 
> 本教學課程中的資料管線會將資料從來源資料存放區，複製到目的地資料存放區。 如需如何使用 Azure Data Factory 轉換資料的教學課程，請參閱[教學課程︰使用 Hadoop 叢集建置管線來轉換資料](data-factory-build-your-first-pipeline.md)。

## <a name="prerequisites"></a>必要條件
* 請檢閱 [教學課程概觀和必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) ，以取得本教學課程的概觀並完成 **必要** 步驟。
* Visual Studio 2012、2013 或 2015
* 下載並安裝 [Azure .NET SDK](http://azure.microsoft.com/downloads/)
* Azure PowerShell。 按照 [如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps) 一文中的指示操作，在您的電腦上安裝 Azure PowerShell。 您可以使用 Azure PowerShell 建立 Azure Active Directory 應用程式。

### <a name="create-an-application-in-azure-active-directory"></a>在 Azure Active Directory 中建立應用程式
建立 Azure Active Directory 應用程式、建立該應用程式的服務主體，然後將其指派給 **Data Factory 參與者** 角色。

1. 啟動 **PowerShell**。
2. 執行下列命令並輸入您用來登入 Azure 入口網站的使用者名稱和密碼。

    ```PowerShell
    Connect-AzureRmAccount
    ```
3. 執行下列命令以檢視此帳戶的所有訂用帳戶。

    ```PowerShell
    Get-AzureRmSubscription
    ```
4. 執行下列命令以選取您要使用的訂用帳戶。 以您的 Azure 訂用帳戶名稱取代 **&lt;NameOfAzureSubscription**&gt;。

    ```PowerShell
    Get-AzureRmSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzureRmContext
    ```

   > [!IMPORTANT]
   > 請記下此命令輸出中的 **SubscriptionId** 和 **TenantId**。

5. 在 PowerShell 中執行以下命令，建立名為 **ADFTutorialResourceGroup** 的 Azure 資源群組。

    ```PowerShell
    New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

    如果資源群組已存在，您可指定是否要更新 (Y) 或予以保留 (N)。

    如果使用不同的資源群組，您必須以資源群組的名稱取代本教學課程中的 ADFTutorialResourceGroup。
6. 建立 Azure Active Directory 應用程式。

    ```PowerShell
    $azureAdApplication = New-AzureRmADApplication -DisplayName "ADFCopyTutotiralApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfcopytutorialapp.org/example" -Password "Pass@word1"
    ```

    如果您收到下列錯誤，請指定不同的 URL 並再次執行此命令。
    
    ```PowerShell
    Another object with the same value for property identifierUris already exists.
    ```
7. 建立 AD 服務主體。

    ```PowerShell
    New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId
    ```
8. 對 **Data Factory 參與者** 角色新增服務主體。

    ```PowerShell
    New-AzureRmRoleAssignment -RoleDefinitionName "Data Factory Contributor" -ServicePrincipalName $azureAdApplication.ApplicationId.Guid
    ```
9. 取得應用程式識別碼。

    ```PowerShell
    $azureAdApplication 
    ```
    記下輸出的應用程式識別碼 (applicationID)。

您應會從這些步驟取得下列四個值︰

* 租用戶識別碼
* 訂用帳戶識別碼
* 應用程式識別碼
* 密碼 (在第一個命令中指定)

## <a name="walkthrough"></a>逐步介紹
1. 使用 Visual Studio 2012/2013/2015 建立 C# .NET 主控台應用程式。
   1. 啟動 **Visual Studio** 2012/2013/2015。
   2. 按一下 [檔案]，指向 [新增]，然後按一下 [專案]。
   3. 展開 [範本]，然後選取 [Visual C#]。 在此逐步解說中，您使用的是 C#，但您可以使用任何 .NET 語言。
   4. 從右邊的專案類型清單中選取 [主控台應用程式]  。
   5. 在 [名稱] 中輸入 **DataFactoryAPITestApp** 。
   6. 在 [位置] 中選取 **C:\ADFGetStarted**。
   7. 按一下 [確定]  以建立專案。
2. 按一下 [**工具**]，指向 [**NuGet 封裝管理員**]，然後按一下 [**封裝管理員主控台**]。
3. 在 [Package Manager Console] 中，輸入下列命令：
   1. 執行以下命令安裝 Data Factory 套件：`Install-Package Microsoft.Azure.Management.DataFactories`
   2. 執行下列命令安裝 Azure Active Directory 套件 (您在程式碼中使用 Active Directory API)︰`Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.19.208020213`
4. 將下列 **appSetttings** 區段新增 **App.config** 檔案。 以下 Helper 方法會使用這些設定： **Microsoft.identitymodel.waad.preview.graph.graphinterface**。

    以您自己的值取代**&lt;應用程式識別碼&gt;**、**&lt;密碼&gt;**、**&lt;訂用帳戶識別碼&gt;****&lt;租用戶識別碼&gt;** 的值。

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <appSettings>
            <add key="ActiveDirectoryEndpoint" value="https://login.microsoftonline.com/" />
            <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
            <add key="WindowsManagementUri" value="https://management.core.windows.net/" />

            <add key="ApplicationId" value="your application ID" />
            <add key="Password" value="Password you used while creating the AAD application" />
            <add key="SubscriptionId" value= "Subscription ID" />
            <add key="ActiveDirectoryTenantId" value="Tenant ID" />
        </appSettings>
    </configuration>
    ```

5. 將下列 **using** 陳述式加入專案的原始程式檔 (Program.cs) 中。

    ```csharp
    using System.Configuration;
    using System.Collections.ObjectModel;
    using System.Threading;
    using System.Threading.Tasks;

    using Microsoft.Azure;
    using Microsoft.Azure.Management.DataFactories;
    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Common.Models;

    using Microsoft.IdentityModel.Clients.ActiveDirectory;

    ```

6. 將下列會建立 **DataPipelineManagementClient** 類別執行個體的程式碼新增至 **Main** 方法中。 您會使用此物件來建立 Data Factory、連結的服務、輸入和輸出資料集，以及管線。 您也會使用此物件來監視執行階段的資料集配量。

    ```csharp
    // create data factory management client
    string resourceGroupName = "ADFTutorialResourceGroup";
    string dataFactoryName = "APITutorialFactory";

    TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader().Result);

    Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
    ```

   > [!IMPORTANT]
   > 以您的 Azure 資源群組名稱取代 **resourceGroupName** 的值。
   >
   > 將 Data Factory 的名稱 (dataFactoryName) 更新成唯一的名稱。 Data Factory 的名稱必須是全域唯一的名稱。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md) 主題，以了解 Data Factory 成品的命名規則。

7. 將下列會建立 **data Factory** 的程式碼新增至 **Main** 方法中。

    ```csharp
    // create a data factory
    Console.WriteLine("Creating a data factory");
    client.DataFactories.CreateOrUpdate(resourceGroupName,
        new DataFactoryCreateOrUpdateParameters()
        {
            DataFactory = new DataFactory()
            {
                Name = dataFactoryName,
                Location = "westus",
                Properties = new DataFactoryProperties()
            }
        }
    );
    ```

    資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 例如，「複製活動」會從來源將資料複製到目的地資料存放區，HDInsight Hive 活動則是執行 Hive 指令碼來轉換輸入資料，以產生輸出資料。 讓我們在這個步驟中開始建立 Data Factory。
8. 將下列會建立 **Azure 儲存體**的程式碼新增至 **Main** 方法中。

   > [!IMPORTANT]
   > 以 Azure 儲存體帳戶的名稱和金鑰取代 **storageaccountname** 和 **accountkey**。

    ```csharp
    // create a linked service for input data store: Azure Storage
    Console.WriteLine("Creating Azure Storage linked service");
    client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new LinkedServiceCreateOrUpdateParameters()
        {
            LinkedService = new LinkedService()
            {
                Name = "AzureStorageLinkedService",
                Properties = new LinkedServiceProperties
                (
                    new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<accountkey>")
                )
            }
        }
    );
    ```

    您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本教學課程中，您不會使用任何計算服務，例如 Azure HDInsight 或 Azure Data Lake Analytics。 您可以使用兩種類型的資料存放區：Azure 儲存體 (來源) 和 Azure SQL Database (目的地)。 

    因此，您可以建立名為 AzureStorageLinkedService 和 AzureSqlLinkedService 的兩個連結服務︰類型為 AzureStorage 和 AzureSqlDatabase。  

    AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 此儲存體帳戶是您在其中建立容器並將資料上傳為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一部分的帳戶。
9. 將下列會建立 **Azure SQL 連結服務**的程式碼新增至 **Main** 方法中。

   > [!IMPORTANT]
   > 以您的 Azure SQL 伺服器名稱、資料庫名稱、使用者和密碼取代 **servername**、**databasename**、**username** **password**。

    ```csharp
    // create a linked service for output data store: Azure SQL Database
    Console.WriteLine("Creating Azure SQL Database linked service");
    client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new LinkedServiceCreateOrUpdateParameters()
        {
            LinkedService = new LinkedService()
            {
                Name = "AzureSqlLinkedService",
                Properties = new LinkedServiceProperties
                (
                    new AzureSqlDatabaseLinkedService("Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30")
                )
            }
        }
    );
    ```

    AzureSqlLinkedService 會將 Azure SQL Database 連結至資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 emp 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。
10. 將下列會建立**輸入和輸出資料集**的程式碼新增至 **Main** 方法中。

    ```csharp
    // create input and output datasets
    Console.WriteLine("Creating input and output datasets");
    string Dataset_Source = "InputDataset";
    string Dataset_Destination = "OutputDataset";

    Console.WriteLine("Creating input dataset of type: Azure Blob");
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,

    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Source,
            Properties = new DatasetProperties()
            {
                Structure = new List<DataElement>()
                {
                    new DataElement() { Name = "FirstName", Type = "String" },
                    new DataElement() { Name = "LastName", Type = "String" }
                },
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/",
                    FileName = "emp.txt"
                },
                External = true,
                Availability = new Availability()
                {
                    Frequency = SchedulePeriod.Hour,
                    Interval = 1,
                },

                Policy = new Policy()
                {
                    Validation = new ValidationPolicy()
                    {
                        MinimumRows = 1
                    }
                }
            }
        }
    });

    Console.WriteLine("Creating output dataset of type: Azure SQL");
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new DatasetCreateOrUpdateParameters()
        {
            Dataset = new Dataset()
            {
                Name = Dataset_Destination,
                Properties = new DatasetProperties()
                {
                    Structure = new List<DataElement>()
                    {
                        new DataElement() { Name = "FirstName", Type = "String" },
                        new DataElement() { Name = "LastName", Type = "String" }
                    },
                    LinkedServiceName = "AzureSqlLinkedService",
                    TypeProperties = new AzureSqlTableDataset()
                    {
                        TableName = "emp"
                    },
                    Availability = new Availability()
                    {
                        Frequency = SchedulePeriod.Hour,
                        Interval = 1,
                    },
                }
            }
        });
    ```
    
    在上一個步驟中，您已建立可將 Azure 儲存體帳戶和 Azure SQL Database 連結至資料處理站的連結服務。 在此步驟中，您會定義名為 InputDataset 和 OutputDataset 的兩個資料集，它們分別代表 AzureStorageLinkedService 和 AzureSqlLinkedService 所參照資料存放區中儲存的輸入和輸出資料。

    Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集 (InputDataset) 會指定包含輸入資料的容器和資料夾。  

    同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集 (OututDataset) 會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。

    在此步驟中，您將在 AzureStorageLinkedService 連結服務所代表的 Azure 儲存體中，建立名為 InputDataset 的資料集，該資料集會指向 Blob 容器 (adftutorial) 根資料夾中的 Blob 檔案 (emp.txt)。 如果您未指定 (或跳過) fileName 的值，則輸入資料夾中所有 Blob 資料都會複製到目的地。 在本教學課程中，您可指定 fileName 的值。    

    在此步驟中，您會建立名為 **OutputDataset**的輸出資料集。 此資料集指向 Azure SQL Database 中 **AzureSqlLinkedService**所代表的 SQL 資料表。
11. 將下列會**建立並啟用管線**的程式碼新增至 **Main** 方法中。 在此步驟中您會建立管線，其中含有使用 **InputDataset** 作為輸入和使用 **OutputDataset** 作為輸出的**複製活動**。

    ```csharp
    // create a pipeline
    Console.WriteLine("Creating a pipeline");
    DateTime PipelineActivePeriodStartTime = new DateTime(2017, 5, 11, 0, 0, 0, 0, DateTimeKind.Utc);
    DateTime PipelineActivePeriodEndTime = new DateTime(2017, 5, 12, 0, 0, 0, 0, DateTimeKind.Utc);
    string PipelineName = "ADFTutorialPipeline";

    client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new PipelineCreateOrUpdateParameters()
        {
            Pipeline = new Pipeline()
            {
                Name = PipelineName,
                Properties = new PipelineProperties()
                {
                    Description = "Demo Pipeline for data transfer between blobs",

                    // Initial value for pipeline's active period. With this, you won't need to set slice status
                    Start = PipelineActivePeriodStartTime,
                    End = PipelineActivePeriodEndTime,

                    Activities = new List<Activity>()
                    {
                        new Activity()
                        {
                            Name = "BlobToAzureSql",
                            Inputs = new List<ActivityInput>()
                            {
                                new ActivityInput() {
                                    Name = Dataset_Source
                                }
                            },
                            Outputs = new List<ActivityOutput>()
                            {
                                new ActivityOutput()
                                {
                                    Name = Dataset_Destination
                                }
                            },
                            TypeProperties = new CopyActivity()
                            {
                                Source = new BlobSource(),
                                Sink = new BlobSink()
                                {
                                    WriteBatchSize = 10000,
                                    WriteBatchTimeout = TimeSpan.FromMinutes(10)
                                }
                            }
                        }
                    }
                }
            }
        });
    ```

    請注意下列幾點：
   
    - 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。 在 Data Factory 解決方案中，您也可以使用[資料轉換活動](data-factory-data-transformation-activities.md)。
    - 活動的輸入設定為 **InputDataset**，活動的輸出則設定為 **OutputDataset**。 
    - 在 **typeProperties** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。 如需複製活動作為來源和接收器支援的資料存放區完整清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 若要了解如何使用特定支援的資料存放區作為來源/接收器，請按一下資料表中的連結。  
   
    目前，驅動排程的是輸出資料集。 在本教學課程中，輸出資料集設定成一小時產生一次配量。 管線具有相隔一天 (也就是 24 小時) 的開始時間和結束時間。 因此，管線會產生輸出資料集的 24 個配量。
12. 將下列程式碼加入 **Main** 方法中，以取得輸出資料集的資料配量狀態。 在此範例中只預期有配量。

    ```csharp
    // Pulling status within a timeout threshold
    DateTime start = DateTime.Now;
    bool done = false;

    while (DateTime.Now - start < TimeSpan.FromMinutes(5) && !done)
    {
        Console.WriteLine("Pulling the slice status");        
        // wait before the next status check
        Thread.Sleep(1000 * 12);

        var datalistResponse = client.DataSlices.List(resourceGroupName, dataFactoryName, Dataset_Destination,
            new DataSliceListParameters()
            {
                DataSliceRangeStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString(),
                DataSliceRangeEndTime = PipelineActivePeriodEndTime.ConvertToISO8601DateTimeString()
            });

        foreach (DataSlice slice in datalistResponse.DataSlices)
        {
            if (slice.State == DataSliceState.Failed || slice.State == DataSliceState.Ready)
            {
                Console.WriteLine("Slice execution is done with status: {0}", slice.State);
                done = true;
                break;
            }
            else
            {
                Console.WriteLine("Slice status is: {0}", slice.State);
            }
        }
    }
    ```

13. 將下列會取得資料配量之執行詳細資料的程式碼加入 **Main** 方法中。

    ```csharp
    Console.WriteLine("Getting run details of a data slice");

    // give it a few minutes for the output slice to be ready
    Console.WriteLine("\nGive it a few minutes for the output slice to be ready and press any key.");
    Console.ReadKey();

    var datasliceRunListResponse = client.DataSliceRuns.List(
            resourceGroupName,
            dataFactoryName,
            Dataset_Destination,
            new DataSliceRunListParameters()
            {
                DataSliceStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString()
            }
        );

    foreach (DataSliceRun run in datasliceRunListResponse.DataSliceRuns)
    {
        Console.WriteLine("Status: \t\t{0}", run.Status);
        Console.WriteLine("DataSliceStart: \t{0}", run.DataSliceStart);
        Console.WriteLine("DataSliceEnd: \t\t{0}", run.DataSliceEnd);
        Console.WriteLine("ActivityId: \t\t{0}", run.ActivityName);
        Console.WriteLine("ProcessingStartTime: \t{0}", run.ProcessingStartTime);
        Console.WriteLine("ProcessingEndTime: \t{0}", run.ProcessingEndTime);
        Console.WriteLine("ErrorMessage: \t{0}", run.ErrorMessage);
    }

    Console.WriteLine("\nPress any key to exit.");
    Console.ReadKey();
    ```

14. 將 **Main** 方法所使用的下列 Helper 方法新增至 **Program** 類別中。

    > [!NOTE] 
    > 當您複製並貼上下列程式碼時，請確定複製的程式碼位於與 Main 方法相同的層級。

    ```csharp
    public static async Task<string> GetAuthorizationHeader()
    {
        AuthenticationContext context = new AuthenticationContext(ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] + ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
        ClientCredential credential = new ClientCredential(
            ConfigurationManager.AppSettings["ApplicationId"],
            ConfigurationManager.AppSettings["Password"]);
        AuthenticationResult result = await context.AcquireTokenAsync(
            resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
            clientCredential: credential);

        if (result != null)
            return result.AccessToken;

        throw new InvalidOperationException("Failed to acquire token");
    }
    ```

15. 在 [方案總管] 中展開專案 (DataFactoryAPITestApp)，以滑鼠右鍵按一下 [參考]，然後按一下 [新增參考]。 選取 **System.Configuration** 組件的核取方塊。 然後按一下 [確定]。
16. 建置主控台應用程式。 按一下功能表上的 [建置]，再按一下 [建置方案]。
17. 確認您 Azure Blob 儲存體之 **adftutorial** 容器中至少有一個檔案。 如果沒有，請在「記事本」中以下列內容建立 **Emp.txt** 檔案，然後將它上傳至 adftutorial 容器。

    ```
    John, Doe
    Jane, Doe
    ```
18. 按一下功能表上的 [偵錯] -> [開始偵錯]，執行範例。 當您看到 [取得資料配量的執行詳細資料]，請等待數分鐘再按 **ENTER**。
19. 使用 Azure 入口網站確認 Data Factory： **APITutorialFactory** 是使用下列成品所建立：
   * 連結服務：**LinkedService_AzureStorage**
   * 資料集︰**InputDataset** 和 **OutputDataset**。
   * 管線： **PipelineBlobSample**
20. 確認在指定 Azure SQL Database 的 **emp** 資料表中建立兩筆員工記錄。

## <a name="next-steps"></a>後續步驟
如需適用於 Data Factory 之 .NET API 的完整文件，請參閱 [Data Factory .NET API 參考](/dotnet/api/index?view=azuremgmtdatafactories-4.12.1)。

在本教學課程中，您可使用 Azure Blob 儲存體作為來源資料存放區以及使用 Azure SQL Database 作為複製作業的目的地資料存放區。 下表提供複製活動所支援作為來源或目的地的資料存放區清單： 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

若要深入了解如何從資料存放區雙向複製資料，請按一下資料表中資料存放區的連結。

