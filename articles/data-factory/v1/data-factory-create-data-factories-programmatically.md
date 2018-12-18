---
title: 使用 Azure .NET SDK 建立資料管線 |Microsoft Docs
description: 了解如何使用 Data Factory .NET SDK，以程式設計方式建立、監視和管理 Azure Data Factory。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: b0a357be-3040-4789-831e-0d0a32a0bda5
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: de892afee57b9a39b841f6cfc93f8470d831c2a6
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045927"
---
# <a name="create-monitor-and-manage-azure-data-factories-using-azure-data-factory-net-sdk"></a>使用 Azure Data Factory .NET SDK 來建立、監視及管理 Azure Data Factory
> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[複製活動教學課程](../quickstart-create-data-factory-dot-net.md)。 

## <a name="overview"></a>概觀
您可以使用 Data Factory .NET SDK，以程式設計方式建立、監視及管理 Azure Data Factory 本文包含指導您建立範例 .NET 主控台應用程式的逐步解說，此應用程式將會建立並監視 Data Factory。 

> [!NOTE]
> 這篇文章並未涵蓋所有的 Data Factory .NET API。 如需適用於 Data Factory 之 .NET API 的完整文件，請參閱 [Data Factory .NET API 參考](/dotnet/api/index?view=azuremgmtdatafactories-4.12.1)。 

## <a name="prerequisites"></a>先決條件
* Visual Studio 2012、2013 或 2015
* 下載並安裝 [Azure .NET SDK](http://azure.microsoft.com/downloads/)。
* Azure PowerShell。 按照 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 一文中的指示操作，在您的電腦上安裝 Azure PowerShell。 您可以使用 Azure PowerShell 建立 Azure Active Directory 應用程式。

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
    $azureAdApplication = New-AzureRmADApplication -DisplayName "ADFDotNetWalkthroughApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfdotnetwalkthroughapp.org/example" -Password "Pass@word1"
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
在逐步解說中，您可以建立具有管線的資料處理站，其中包含複製活動。 複製活動會將資料從 Azure Blob 儲存體中的資料夾，複製到相同 Blob 儲存體中的另一個資料夾。 

複製活動會在 Azure Data Factory 中執行資料移動。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱 [資料移動活動](data-factory-data-movement-activities.md) 文章。

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
4. 以下列內容取代專案中 **App.config** 檔案的內容： 
    
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
5. 在 App.Config 檔案中，以您自己的值更新**&lt;應用程式識別碼&gt;**、**&lt;密碼&gt;**、**&lt;訂用帳戶識別碼&gt;****&lt;租用戶識別碼&gt;** 的值。
6. 將下列 **using** 陳述式新增至專案的 **Program.cs**檔案。

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

    //IMPORTANT: specify the name of Azure resource group here
    string resourceGroupName = "ADFTutorialResourceGroup";

    //IMPORTANT: the name of the data factory must be globally unique.
    // Therefore, update this value. For example:APITutorialFactory05122017
    string dataFactoryName = "APITutorialFactory";

    TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader().Result);

    Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
    ```

   > [!IMPORTANT]
   > 以您的 Azure 資源群組名稱取代 **resourceGroupName** 的值。 若要建立資源群組，請使用 [New-AzureResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) Cmdlet。
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
9. 將下列會建立**輸入和輸出資料集**的程式碼新增至 **Main** 方法中。

    輸入 Blob 的 **FolderPath** 是設定為 **adftutorial/**，其中的 **adftutorial** 是 Blob 儲存體中容器的名稱。 如果 Azure Blob 儲存體中沒有此容器，請以下列名稱建立容器： **adftutorial** ，並將文字檔上傳至容器。

    輸出 Blob 的 FolderPath 是設定為：**adftutorial/apifactoryoutput/{Slice}**，其中的 **Slice** 是根據 **SliceStart** (每個配量的開始日期時間) 的值自動計算而得。

    ```csharp
    // create input and output datasets
    Console.WriteLine("Creating input and output datasets");
    string Dataset_Source = "DatasetBlobSource";
    string Dataset_Destination = "DatasetBlobDestination";
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Source,
            Properties = new DatasetProperties()
            {
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
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Destination,
            Properties = new DatasetProperties()
            {
    
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/apifactoryoutput/{Slice}",
                    PartitionedBy = new Collection<Partition>()
                    {
                        new Partition()
                        {
                            Name = "Slice",
                            Value = new DateTimePartitionValue()
                            {
                                Date = "SliceStart",
                                Format = "yyyyMMdd-HH"
                            }
                        }
                    }
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
10. 將下列會**建立並啟用管線**的程式碼新增至 **Main** 方法中。 此管線有一個 **CopyActivity**，它以 **BlobSource** 為來源，**BlobSink** 為接收器。

    複製活動會在 Azure Data Factory 中執行資料移動。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱 [資料移動活動](data-factory-data-movement-activities.md) 文章。

    ```csharp
    // create a pipeline
    Console.WriteLine("Creating a pipeline");
    DateTime PipelineActivePeriodStartTime = new DateTime(2014, 8, 9, 0, 0, 0, 0, DateTimeKind.Utc);
    DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
    string PipelineName = "PipelineBlobSample";
    
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
                        Name = "BlobToBlob",
                        Inputs = new List<ActivityInput>()
                        {
                            new ActivityInput()
                {
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
    
                },
            }
        }
    });
    ```
12. 將下列程式碼加入 **Main** 方法中，以取得輸出資料集的資料配量狀態。 預期此範例中只有一個配量。

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
13. **(選用)** 將下列取得資料配量之執行詳細資料的程式碼新增到 **Main** 方法中。

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
        });
    
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
14. 將 **Main** 方法所使用的下列 Helper 方法新增至 **Program** 類別中。 此方法會顯示一個對話方塊，讓您提供用來登入 Azure 入口網站的**使用者名稱**和**密碼**。

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

15. 在 [方案總管] 中展開 **DataFactoryAPITestApp** 專案，以滑鼠右鍵按一下 [參考]，然後按一下 [加入參考]。 選取 `System.Configuration` 組件的核取方塊，然後按一下 [確定]。
15. 建置主控台應用程式。 按一下功能表上的 [建置]，再按一下 [建置方案]。
16. 確認您 Azure Blob 儲存體之 adftutorial 容器中至少有一個檔案。 如果沒有，請在「記事本」中以下列內容建立 Emp.txt 檔案，然後將它上傳至 adftutorial 容器。

    ```
    John, Doe
    Jane, Doe
    ```
17. 按一下功能表上的 [偵錯] -> [開始偵錯]，執行範例。 當您看到 [取得資料配量的執行詳細資料]，請等待數分鐘再按 **ENTER**。
18. 使用 Azure 入口網站確認 Data Factory： **APITutorialFactory** 是使用下列成品所建立：
    * 連結服務：**AzureStorageLinkedService**
    * 資料集：**DatasetBlobSource** 和 **DatasetBlobDestination**。
    * 管線： **PipelineBlobSample**
19. 確認輸出檔案已建立於 **adftutorial** 容器的 **apifactoryoutput** 資料夾中。

## <a name="get-a-list-of-failed-data-slices"></a>取得失敗資料配量的清單 

```csharp
// Parse the resource path
var ResourceGroupName = "ADFTutorialResourceGroup";
var DataFactoryName = "DataFactoryAPITestApp";

var parameters = new ActivityWindowsByDataFactoryListParameters(ResourceGroupName, DataFactoryName);
parameters.WindowState = "Failed";
var response = dataFactoryManagementClient.ActivityWindows.List(parameters);
do
{
    foreach (var activityWindow in response.ActivityWindowListResponseValue.ActivityWindows)
    {
        var row = string.Join(
            "\t",
            activityWindow.WindowStart.ToString(),
            activityWindow.WindowEnd.ToString(),
            activityWindow.RunStart.ToString(),
            activityWindow.RunEnd.ToString(),
            activityWindow.DataFactoryName,
            activityWindow.PipelineName,
            activityWindow.ActivityName,
            string.Join(",", activityWindow.OutputDatasets));
        Console.WriteLine(row);
    }

    if (response.NextLink != null)
    {
        response = dataFactoryManagementClient.ActivityWindows.ListNext(response.NextLink, parameters);
    }
    else
    {
        response = null;
    }
}
while (response != null);
```

## <a name="next-steps"></a>後續步驟
請參閱下列範例以建立管線，該管線使用 .NET SDK (從 Azure Blob 儲存體將資料複製到 Azure SQL Database)： 

- [建立管線以將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-activity-tutorial-using-dotnet-api.md)
