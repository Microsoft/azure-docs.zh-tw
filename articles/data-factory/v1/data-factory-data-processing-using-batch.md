---
title: 使用 Data Factory 和 Batch 來處理大型資料集 | Microsoft Docs
description: 說明如何使用 Azure Batch 的平行處理功能來處理 Azure Data Factory 管線中的大量資料。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 688b964b-51d0-4faa-91a7-26c7e3150868
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 511a0122b37f5f34097da38a645790d99212ad18
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45737408"
---
# <a name="process-large-scale-datasets-by-using-data-factory-and-batch"></a>使用 Data Factory 和 Batch 來處理大型資料集
> [!NOTE]
> 本文適用於第 1 版 Azure Data Factory (正式運作版)。 如果您使用目前版本的 Data Factory 服務，請參閱 [Data Factory 中的自訂活動](../transform-data-using-dotnet-custom-activity.md)。

本文說明範例解決方案的架構，此解決方案能以自動且排程的方式移動和處理大型資料集。 本文也提供如何使用 Data Factory 和 Azure Batch 來實作解決方案的端對端逐步解說。

這篇文章比一般文章還長，因為它包含整個範例解決方案的逐步解說。 如果您不熟悉 Batch 和 Data Factory，您可以了解這兩項服務以及它們如何搭配運作。 如果您有這些服務的一些相關知識，而要設計解決方案或建立解決方案架構，您可以將焦點放在本文的[架構小節](#architecture-of-sample-solution)。 如果您要開發原型或解決方案，則可以試著依照[逐步解說](#implementation-of-sample-solution)中的逐步指示進行操作。 歡迎您對此內容以及您如何使用它提出看法。

讓我們先看看 Data Factory 和 Batch 服務如何協助您處理雲端中的大型資料集。     

## <a name="why-azure-batch"></a>為何使用 Azure Batch？
 您可以使用 Batch 在雲端有效率地執行大規模的平行和高效能運算 (HPC) 應用程式。 這是一項平台服務，可將計算密集型工作排定在一組受控虛擬機器 (VM) 上執行。 它可以自動調整計算資源來符合作業的需求。

在使用 Batch 服務時，您可以定義用來大規模平行執行應用程式的 Azure 計算資源。 您可以視需要執行作業，或執行已排定的作業。 您不需要手動建立、設定及管理 HPC 叢集、個別 VM、虛擬網路或複雜的作業和工作排程基礎結構。

 如果您不熟悉 Batch，下列文章將可協助您了解本文所述解決方案的架構/實作：   

* [Batch 的基本概念](../../batch/batch-technical-overview.md)
* [Batch 功能概觀](../../batch/batch-api-basics.md)

(選擇性) 若要深入了解 Batch，請參閱 [Batch 的學習路徑](https://azure.microsoft.com/documentation/learning-paths/batch/)。

## <a name="why-azure-data-factory"></a>為何使用 Azure Data Factory？
Data Factory 是雲端架構資料整合服務，用來協調以及自動移動和轉換資料。 您可以使用 Data Factory 來建立受控資料管線，以將資料從內部部署和雲端資料存放區移至集中式資料存放區。 Azure Blob 儲存體便是一個例子。 您可以使用 Data Factory 透過 Azure HDInsight 和 Azure Machine Learning 之類的服務來處理/轉換資料。 您也可以排定讓資料管線以排程方式執行 (例如每小時、每天及每週)。 您一眼就能監視及管理管線來找出問題點並採取行動。

  如果您不熟悉 Data Factory，下列文章將可協助您了解本文所述解決方案的架構/實作：  

* [Data Factory 服務簡介](data-factory-introduction.md)
* [建置第一個資料管線](data-factory-build-your-first-pipeline.md)   

(選擇性) 若要深入了解 Data Factory，請參閱 [Data Factory 的學習路徑](https://azure.microsoft.com/documentation/learning-paths/data-factory/)。

## <a name="data-factory-and-batch-together"></a>Data Factory 和 Batch 一起使用
Data Factory 包含內建的活動。 例如，使用「複製」活動可將資料從來源資料存放區複製/移動到目的地資料存放區。 使用 Hive 活動可透過 Azure 上的 Hadoop 叢集 (HDInsight) 來處理資料。 如需支援的轉換活動清單，請參閱[資料轉換活動](data-factory-data-transformation-activities.md)。

您也可以建立自訂的 .NET 活動，運用自己的邏輯來移動或處理資料。 您可以在 HDInsight 叢集上或 VM 的批次集區上執行這些活動。 當您使用 Batch 時，可以設定讓集區根據您提供的公式自動調整大小 (根據工作負載來新增或移除 VM)。     

## <a name="architecture-of-a-sample-solution"></a>範例解決方案的架構
  本文所述的架構適用於簡單解決方案。 它也與複雜的案例相關，例如依金融服務、映像處理和轉譯以及基因分析進行的風險模型建立。

下圖說明 Data Factory 如何協調資料移動與處理。 此圖也說明 Batch 如何平行處理資料。 請下載及列印此圖以便參考 (11 x 17 英吋或 A3 大小)。 若要存取此圖以便列印，請參閱[使用 Batch 和 Data Factory 來進行的 HPC 與資料協調流程](http://go.microsoft.com/fwlink/?LinkId=717686)。

[![大規模資料處理圖表](./media/data-factory-data-processing-using-batch/image1.png)](http://go.microsoft.com/fwlink/?LinkId=717686)

下列清單提供程序中的基本步驟。 此解決方案包含用來建置端對端解決方案的程式碼和說明。

* **為 Batch 設定計算節點 (VM) 集區**。 您可以指定節點數目和每個節點的大小。

* **建立 Data Factory 執行個體**並為其設定實體，這些實體代表 Blob 儲存體、Batch 計算服務、輸入/輸出資料，以及具有會移動和轉換資料之活動的工作流程/管線。

* **在 Data Factory 管線中建立自訂 .NET 活動**。 活動係指在 Batch 集區上執行的使用者程式碼。

* **將大量輸入資料以 Blob 形式儲存在 Azure 儲存體中**。 資料會分成邏輯配量 (通常依時間來分)。

* **Data Factory 將要以平行方式處理的資料複製到次要位置**。

* **Data Factory 使用 Batch 所配置的集區來執行自訂活動**。 Data Factory 可以同時執行多個活動。 每個活動各處理某個配量的資料。 結果會儲存在儲存體中。

* **Data Factory 將最終結果移至第三個位置**，以透過應用程式進行散發，或由其他工具進行進一步處理。

## <a name="implementation-of-the-sample-solution"></a>範例解決方案的實作
範例解決方案是刻意設計成簡單的解決方案。 其設計目的是要說明如何將 Data Factory 與 Batch 搭配使用來處理資料集。 此解決方案會計算搜尋詞彙 "Microsoft" 在以時間序列組織的輸入檔案中出現的次數。 然後會將此計數輸出至輸出檔案。

**時間**：如果您熟悉 Azure、Data Factory 和 Batch 的基本概念，並已符合下列先決條件，此解決方案將需要 1 到 2 小時的時間來完成。

### <a name="prerequisites"></a>必要條件
#### <a name="azure-subscription"></a>Azure 訂用帳戶
如果您沒有 Azure 訂用帳戶，您可以快速建立免費的試用帳戶。 如需詳細資訊，請參閱[免費試用](https://azure.microsoft.com/pricing/free-trial/)。

#### <a name="azure-storage-account"></a>Azure 儲存體帳戶
在本教學課程中，您會使用儲存體帳戶來儲存資料。 如果您沒有儲存體帳戶，請參閱[建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md)。 範例解決方案會使用 Blob 儲存體。

#### <a name="azure-batch-account"></a>Azure Batch 帳戶
使用 [Azure 入口網站](http://portal.azure.com/)來建立儲存體帳戶。 如需詳細資訊，請參閱[建立及管理 Batch 帳戶](../../batch/batch-account-create-portal.md)。 請記下 Batch 帳戶名稱和帳戶金鑰。 您也可以使用 [New-AzureRmBatchAccount](https://docs.microsoft.com/powershell/module/azurerm.batch/new-azurermbatchaccount) Cmdlet 來建立 Batch 帳戶。 如需有關如何使用此 Cmdlet 的指示，請參閱 [開始使用 Batch PowerShell Cmdlet](../../batch/batch-powershell-cmdlets-get-started.md)。

此範例解決方案會使用 Batch (透過資料處理站管線間接使用)，以平行方式在計算節點集區 (受控 VM 集合) 上處理資料。

#### <a name="azure-batch-pool-of-virtual-machines"></a>Azure Batch 虛擬機器集區
建立一個至少含有 2 個計算節點的 Batch 集區。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取左側功能標中的 [瀏覽]，然後選取 [Batch 帳戶]。

1. 選取您的 Batch 帳戶以開啟 [Batch 帳戶]  刀鋒視窗。

1. 選取 [集區] 圖格。

1. 在 [集區] 刀鋒視窗上，選取工具列上的 [新增] 按鈕以新增集區。

   a. 輸入集區的識別碼 (**集區識別碼**)。 請記下集區的識別碼。 在建立資料處理站解決方案時會需要它。

   b. 為 [作業系統系列] 設定指定 [Windows Server 2012 R2]。

   c. 選取 **節點定價層**。

   d. 輸入 **2** 作為 [目標專用] 設定的值。

   e. 輸入 **2** 作為 [每個節點的工作數上限] 設定的值。

   f. 選取 [確定] 以建立集區。

#### <a name="azure-storage-explorer"></a>Azure 儲存體總管
您將使用 [Azure 儲存體總管 6](https://azurestorageexplorer.codeplex.com/) 或 [CloudXplorer](http://clumsyleaf.com/products/cloudxplorer) (來自 ClumsyLeaf Software) 來檢查和更改您儲存體專案中的資料。 您也可以檢查和更改雲端所裝載應用程式之記錄中的資料。

1. 建立名為 **mycontainer** 且具有私人存取權 (無匿名存取權) 的容器。

1. 如果您使用 CloudXplorer，請建立具有下列結構的資料夾和子資料夾：

   ![資料夾和子資料夾結構](./media/data-factory-data-processing-using-batch/image3.png)

   `Inputfolder` 和 `outputfolder` 是 `mycontainer` 中的最上層資料夾。 `inputfolder` 資料夾具有含日期時間戳記 (YYYY-MM-DD-HH) 的子資料夾。

   如果您使用「儲存體總管」，在下一個步驟中，您將上傳具有下列名稱的檔案：`inputfolder/2015-11-16-00/file.txt`、`inputfolder/2015-11-16-01/file.txt` 等等。 此步驟會自動建立資料夾。

1. 在您的電腦上建立內容中含有關鍵字 **Microsoft** 的文字檔 **file.txt**。 例如 "test custom activity Microsoft test custom activity Microsoft"。

1. 將檔案上傳至 Blob 儲存體中的下列輸入資料夾：

   ![輸入資料夾](./media/data-factory-data-processing-using-batch/image4.png)

   如果您使用「儲存體總管」，請將 **file.txt** 檔案上傳至 [mycontainer]。 選取工具列上的 [複製]以建立 Blob 複本。 在 [複製 Blob] 對話方塊中，將**目的地 Blob 名稱**變更為 `inputfolder/2015-11-16-00/file.txt`。 重複此步驟以建立 `inputfolder/2015-11-16-01/file.txt`、`inputfolder/2015-11-16-02/file.txt`、`inputfolder/2015-11-16-03/file.txt`、`inputfolder/2015-11-16-04/file.txt` 等等。 此動作會自動建立資料夾。

1. 建立名為 `customactivitycontainer` 的另一個容器。 將自訂活動 zip 檔案上傳至此容器。

#### <a name="visual-studio"></a>Visual Studio
安裝 Visual Studio 2012 或更新版本，以建立要在資料處理站解決方案中使用的自訂 Batch 活動。

### <a name="high-level-steps-to-create-the-solution"></a>建立解決方案的高階步驟
1. 建立包含資料處理邏輯的自訂活動。

1. 建立使用自訂活動的資料處理站。

### <a name="create-the-custom-activity"></a>建立自訂活動
資料處理站自訂活動是此範例解決方案的核心。 此範例解決方案會使用 Batch 來執行自訂活動。 如需了解如何開發自訂活動並在資料處理站管線中使用自訂活動，請參閱[在資料處理站管線中使用自訂活動](data-factory-use-custom-activities.md)。

若要建立可在資料處理站管線中使用的 .NET 自訂活動，您需利用實作 IDotNetActivity 介面的類別來建立 .NET 類別庫專案。 這個介面只有一個方法：Execute。 以下是該方法的簽章：

```csharp
public IDictionary<string, string> Execute(
            IEnumerable<LinkedService> linkedServices,
            IEnumerable<Dataset> datasets,
            Activity activity,
            IActivityLogger logger)
```

此方法有幾個您必須了解的關鍵元件：

* 此方法會採用四個參數：

  * **linkedServices**。 此參數是一個可列舉的已連結服務清單，這些服務會將輸入/輸出資料來源 (例如 Blob 儲存體) 連結到資料處理站。 在此範例中，只有一個類型為「Azure 儲存體」的已連結服務，同時用於輸入和輸出。
  * **資料集**。 此參數是可列舉的資料集清單。 您可以使用這個參數取得輸入和輸出資料集定義的位置和結構描述。
  * **活動**。 此參數代表目前的計算實體。 在此例中為 Batch 服務。
  * **logger**。 您可以使用記錄器來撰寫會呈現為管線之「使用者」記錄的偵錯註解。
* 此方法會傳回未來可用來將自訂活動鏈結在一起的字典。 目前尚未實作此功能，因此只會從方法傳回空的字典。

#### <a name="procedure-create-the-custom-activity"></a>程序：建立自訂活動
1. 在 Visual Studio 中建立 .NET 類別庫專案。

   a. 啟動 Visual Studio 2012/2013/2015。

   b. 選取 [檔案] > [新增] > [專案]。

   c. 展開 [範本]，然後選取 [Visual C#]**\#**。 在此逐步解說中，您使用 C\# 中，但您可以使用任何 .NET 語言來開發自訂活動。

   d. 從右邊的專案類型清單中選取 [類別庫]。

   e. 針對 [名稱] 輸入 **MyDotNetActivity**。

   f. 在 [位置] 中選取 **C:\\ADF**。 建立 [ADF] 資料夾 (如果不存在)。

   g. 選取 [確定] 以建立專案。

1. 選取 [工具] > [NuGet 套件管理員] > [套件管理員主控台]。

1. 在 [套件管理員主控台] 中，執行下列命令以匯入 Microsoft.Azure.Management.DataFactories：

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactories
    ```
1. 將 **Azure 儲存體** NuGet 套件匯入到專案中。 您會需要此套件，因為您會在此範例中使用「Blob 儲存體」API：

    ```powershell
    Install-Package Azure.Storage
    ```
1. 將下列 using 指示詞新增至專案中的原始程式檔：

    ```csharp
    using System.IO;
    using System.Globalization;
    using System.Diagnostics;
    using System.Linq;
    
    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Runtime;
    
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
1. 將命名空間的名稱變更為 **MyDotNetActivityNS**。

    ```csharp
    namespace MyDotNetActivityNS
    ```
1. 將類別的名稱變更為 **MyDotNetActivity**，並從 **IDotNetActivity** 介面衍生它，如下所示：

    ```csharp
    public class MyDotNetActivity : IDotNetActivity
    ```
1. 對 **MyDotNetActivity** 類別實作 (新增) **IDotNetActivity** 介面的 **Execute** 方法。 將下列範例程式碼複製到此方法。 如需此方法中所使用之邏輯的說明，請參閱 [Execute 方法](#execute-method) 一節。

    ```csharp
    /// <summary>
    /// The Execute method is the only method of IDotNetActivity interface you must implement.
    /// In this sample, the method invokes the Calculate method to perform the core logic.  
    /// </summary>
    public IDictionary<string, string> Execute(
       IEnumerable<LinkedService> linkedServices,
       IEnumerable<Dataset> datasets,
       Activity activity,
       IActivityLogger logger)
    {
    
       // Declare types for the input and output data stores.
       AzureStorageLinkedService inputLinkedService;
    
       Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);
    
       foreach (LinkedService ls in linkedServices)
           logger.Write("linkedService.Name {0}", ls.Name);
    
       // Use the First method instead of Single because we are using the same
       // Azure Storage linked service for input and output.
       inputLinkedService = linkedServices.First(
           linkedService =>
           linkedService.Name ==
           inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
           as AzureStorageLinkedService;
    
       string connectionString = inputLinkedService.ConnectionString; // To create an input storage client.
       string folderPath = GetFolderPath(inputDataset);
       string output = string.Empty; // for use later.
    
       // Create the storage client for input. Pass the connection string.
       CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
       CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();
    
       // Initialize the continuation token before using it in the do-while loop.
       BlobContinuationToken continuationToken = null;
       do
       {   // get the list of input blobs from the input storage client object.
           BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
                                    true,
                                    BlobListingDetails.Metadata,
                                    null,
                                    continuationToken,
                                    null,
                                    null);
    
           // The Calculate method returns the number of occurrences of
           // the search term "Microsoft" in each blob associated
           // with the data slice.
           //
           // The definition of the method is shown in the next step.
           output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");
    
       } while (continuationToken != null);
    
       // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
       Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);
    
       folderPath = GetFolderPath(outputDataset);
    
       logger.Write("Writing blob to the folder: {0}", folderPath);
    
       // Create a storage object for the output blob.
       CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
       // Write the name of the file.
       Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));
    
       logger.Write("output blob URI: {0}", outputBlobUri.ToString());
       // Create a blob and upload the output text.
       CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
       logger.Write("Writing {0} to the output blob", output);
       outputBlob.UploadText(output);
    
       // The dictionary can be used to chain custom activities together in the future.
       // This feature is not implemented yet, so just return an empty dictionary.
       return new Dictionary<string, string>();
    }
    ```
1. 將下列協助程式方法加入至類別。 這些方法可用 **Execute** 方法來叫用。 最重要的是，**Calculate** 方法會隔離逐一查看每個 Blob 的程式碼。

    ```csharp
    /// <summary>
    /// Gets the folderPath value from the input/output dataset.
    /// </summary>
    private static string GetFolderPath(Dataset dataArtifact)
    {
       if (dataArtifact == null || dataArtifact.Properties == null)
       {
           return null;
       }
    
       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }
    
       return blobDataset.FolderPath;
    }
    
    /// <summary>
    /// Gets the fileName value from the input/output dataset.
    /// </summary>
    
    private static string GetFileName(Dataset dataArtifact)
    {
       if (dataArtifact == null || dataArtifact.Properties == null)
       {
           return null;
       }
    
       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }
    
       return blobDataset.FileName;
    }
    
    /// <summary>
    /// Iterates through each blob (file) in the folder, counts the number of instances of the search term in the file,
    /// and prepares the output text that is written to the output blob.
    /// </summary>
    
    public static string Calculate(BlobResultSegment Bresult, IActivityLogger logger, string folderPath, ref BlobContinuationToken token, string searchTerm)
    {
       string output = string.Empty;
       logger.Write("number of blobs found: {0}", Bresult.Results.Count<IListBlobItem>());
       foreach (IListBlobItem listBlobItem in Bresult.Results)
       {
           CloudBlockBlob inputBlob = listBlobItem as CloudBlockBlob;
           if ((inputBlob != null) && (inputBlob.Name.IndexOf("$$$.$$$") == -1))
           {
               string blobText = inputBlob.DownloadText(Encoding.ASCII, null, null, null);
               logger.Write("input blob text: {0}", blobText);
               string[] source = blobText.Split(new char[] { '.', '?', '!', ' ', ';', ':', ',' }, StringSplitOptions.RemoveEmptyEntries);
               var matchQuery = from word in source
                                where word.ToLowerInvariant() == searchTerm.ToLowerInvariant()
                                select word;
               int wordCount = matchQuery.Count();
               output += string.Format("{0} occurrences(s) of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
           }
       }
       return output;
    }
    ```
    GetFolderPath 方法會將路徑傳回資料集所指向的資料夾，而 GetFileName 方法會傳回資料集指向的 blob/檔案名稱。

    ```csharp

    "name": "InputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "fileName": "file.txt",
            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
    ```

    Calculate 方法會計算輸入檔案 (資料夾中的 Blob) 中關鍵字 "Microsoft" 的執行個體數目。 搜尋詞彙 "Microsoft" 是已在程式碼中硬式編碼的詞彙。

1. 編譯專案。 從功能表中選取 [建置]，然後選取 [建置方案]。

1. 啟動「Windows 檔案總管」，然後移至 **bin\\debug** 或 **bin\\release** 資料夾。 資料夾選擇取決於建置類型。

1. 建立 zip 檔案 **MyDotNetActivity.zip**，檔案中包含 **\\bin\\Debug** 資料夾中的所有二進位檔。 您可以包含 MyDotNetActivity.**pdb** 檔案，以便取得額外的詳細資料，例如在失敗發生時，原始程式碼中引起問題的程式碼行號。

   ![[bin\Debug] 資料夾清單](./media/data-factory-data-processing-using-batch/image5.png)

1. 將 **MyDotNetActivity.zip** 以 Blob 形式上傳至 ADFTutorialDataFactory 中 StorageLinkedService 已連結服務所使用之 Blob 儲存體中的 Blob 容器 `customactivitycontainer`。 建立 Blob 容器 `customactivitycontainer` (如果尚未存在)。

#### <a name="execute-method"></a>Execute 方法
本節會提供有關 Execute 方法中程式碼的更多詳細資料。

1. 逐一查看輸入集合的成員可在 [Microsoft.WindowsAzure.Storage.Blob](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.blob.aspx) 命名空間中找到。 若要逐一查看 Blob 集合，您必須使用 **BlobContinuationToken** 類別。 基本上，您必須搭配使用 do-while 迴圈和權杖做為結束迴圈的機制。 如需詳細資訊，請參閱[從 .NET 使用 Blob 儲存體](../../storage/blobs/storage-dotnet-how-to-use-blobs.md)。 基本迴圈如下所示：

    ```csharp
    // Initialize the continuation token.
    BlobContinuationToken continuationToken = null;
    do
    {
    // Get the list of input blobs from the input storage client object.
    BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
    
                         true,
                                   BlobListingDetails.Metadata,
                                   null,
                                   continuationToken,
                                   null,
                                   null);
    // Return a string derived from parsing each blob.
    
     output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");
    
    } while (continuationToken != null);

    ```
   如需詳細資訊，請參閱 [ListBlobsSegmented](https://msdn.microsoft.com/library/jj717596.aspx) 方法的文件。

1. 以邏輯方式逐一查看 blob 集的程式碼會在 do-while 迴圈中執行。 在 **Execute** 方法中，執行 do-while 迴圈會將 blob 清單傳遞至名為 **Calculate** 的方法。 此方法會傳回名為 **output** 的字串變數，也就是在區段中逐一查看所有 blob 的結果。

   它會傳回搜尋詞彙 "Microsoft" 在傳遞給 **Calculate** 方法之 Blob 中出現的次數。

    ```csharp
    output += string.Format("{0} occurrences of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
    ```
1. 在 **Calculate** 方法執行完成後，系統必須將它寫入至新的 Blob。 針對所處理的每一組 Blob，系統都可以在新的 Blob 中寫入結果。 若要寫入新的 blob，請先找到輸出資料集。

    ```csharp
    // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
    Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);
    ```
1. 程式碼也會呼叫 Helper 方法 **GetFolderPath** 來擷取資料夾路徑 (儲存體容器名稱)。

    ```csharp
    folderPath = GetFolderPath(outputDataset);
    ```
   GetFolderPath 會將 DataSet 物件轉換成 AzureBlobDataSet (具有一個名為 FolderPath 的屬性)。

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
    
    return blobDataset.FolderPath;
    ```
1. 程式碼會呼叫 **GetFileName** 方法來擷取檔案名稱 (blob 名稱)。 此程式碼與先前用來取得資料夾路徑的程式碼類似。

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
    
    return blobDataset.FileName;
    ```
1. 藉由建立 URI 物件寫入檔案的名稱。 URI 建構函式使用 **BlobEndpoint** 屬性傳回容器名稱。 新增資料夾路徑和檔案名稱以建構輸出 blob URI。  

    ```csharp
    // Write the name of the file.
    Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));
    ```
1. 在寫入檔案名稱之後，您便可以從 **Calculate** 方法將輸出字串寫入至新的 Blob：

    ```csharp
    // Create a blob and upload the output text.
    CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
    logger.Write("Writing {0} to the output blob", output);
    outputBlob.UploadText(output);
    ```

### <a name="create-the-data-factory"></a>建立 Data Factory
在[建立自訂活動](#create-the-custom-activity)一節中，您已建立自訂活動，並將包含二進位檔和 PDB 檔案的 zip 檔案上傳到 Blob 容器。 在本節中，您將建立一個含有使用自訂活動之管線的資料處理站。

自訂活動的輸入資料集代表 blob 儲存體中輸入資料夾 (`mycontainer\\inputfolder`) 的 blob (檔案)。 活動的輸出資料集代表 blob 儲存體中輸出資料夾 (`mycontainer\\outputfolder`) 的輸出 blob。

將一或多個檔案放置到輸入資料夾中：

```
mycontainer -\> inputfolder
    2015-11-16-00
    2015-11-16-01
    2015-11-16-02
    2015-11-16-03
    2015-11-16-04
```

例如，將一個含有下列內容的檔案 (file.txt) 放置到每個資料夾中：

```
test custom activity Microsoft test custom activity Microsoft
```

即使資料夾有兩個以上的檔案，每個輸入資料夾還是會與資料處理站中的一個配量對應。 管線處理每個配量時，自訂活動會為該配量逐一查看輸入資料夾中的所有 blob。

您可看到五個具有相同內容的輸出檔案。 例如，處理 2015-11-16-00 資料夾中的檔案所產生的輸出檔案包含下列內容：

```
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
```

如果您將多個具有相同內容的檔案 (file.txt、file2.txt、file3.txt) 放置到輸入資料夾中，就會在輸出檔案中看見以下內容。 每個資料夾 (2015-11-16-00 等) 分別對應至此範例中的配量，即使資料夾有多個輸入檔案亦然。

```csharp
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file2.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file3.txt.
```

輸出檔案現在會有三行，與配量相關聯的資料夾 (2015-11-16-00) 中的每個輸入檔案 (blob) 各一行。

每個活動執行都會建立一個工作。 在此範例中，管線中只有一個活動。 由管線處理配量時，自訂活動會在 Batch 上執行以處理配量。 由於有 5 個配量 (每個配量可以有多個 Blob 或檔案)，因此會在 Batch 中建立 5 個工作。 當工作在 Batch 上執行時，實際上就是自訂活動在執行。

下列逐步解說將提供其他詳細資料。

#### <a name="step-1-create-the-data-factory"></a>步驟 1：建立 Data Factory
1. 登入 [Azure 入口網站](https://portal.azure.com/)之後，執行下列步驟：

   a. 選取左側功能表上的 [新增]。

   b. 選取 [新增] 刀鋒視窗上的 [資料 + 分析]。

   c. 選取 [資料分析] 刀鋒視窗上的 [資料處理站]。

1. 在 [新增資料處理站] 刀鋒視窗上，輸入 **CustomActivityFactory** 作為名稱。 資料處理站的名稱必須是全域唯一的名稱。 如果您收到「資料處理站名稱 CustomActivityFactory 無法使用」錯誤，請變更資料處理站名稱。 例如，使用 yournameCustomActivityFactory，然後重新建立資料處理站。

1. 選取 [資源群組名稱]，然後選取現有的資源群組，或建立一個資源群組。

1. 確認您要在其中建立資料處理站的訂用帳戶和區域正確。

1. 選取 [新增資料處理站] 刀鋒視窗上的 [建立]。

1. 系統會在入口網站的儀表板中建立資料處理站。

1. 成功建立資料處理站之後，您會看到 [資料處理站] 頁面，當中會顯示資料處理站的內容。

   ![Data Factory 頁面](./media/data-factory-data-processing-using-batch/image6.png)

#### <a name="step-2-create-linked-services"></a>步驟 2：建立連結服務
已連結的服務會將資料存放區或計算服務連結至資料處理站。 在此步驟中，您會將儲存體帳戶和 Batch 帳戶連結至資料處理站。

#### <a name="create-an-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
1. 選取 **CustomActivityFactory** 之 [資料處理站] 刀鋒視窗上的 [編寫及部署] 圖格。 隨即會出現「Data Factory 編輯器」。

1. 選取命令列上的 [新增資料存放區]，然後選擇 [Azure 儲存體]。 隨即會出現您在編輯器中用來建立儲存體已連結服務的 JSON 指令碼。

   ![新增資料存放區](./media/data-factory-data-processing-using-batch/image7.png)

1. 將**帳戶名稱**取代成您儲存體帳戶的名稱。 將**帳戶金鑰**取代成儲存體帳戶的存取金鑰。 若要了解如何取得儲存體存取金鑰，請參閱[檢視、複製和重新產生儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)。

1. 選取命令列上的 [部署] 以部署連結服務。

   ![部署](./media/data-factory-data-processing-using-batch/image8.png)

#### <a name="create-an-azure-batch-linked-service"></a>建立 Azure Batch 已連結服務
在此步驟中，您將為您的 Batch 帳戶建立用來執行資料處理站自訂活動的已連結服務。

1. 選取命令列上的 [新增計算]，然後選擇 [Azure Batch]。 隨即會出現您在編輯器中用來建立 Batch 已連結服務的 JSON 指令碼。

1. 在 JSON 指令碼中：

   a. 將**帳戶名稱**取代成您 Batch 帳戶的名稱。

   b. 將**存取金鑰**取代成 Batch 帳戶的存取金鑰。

   c. 針對 **poolName** 屬性輸入集區的識別碼。 您可以針對此屬性指定集區名稱或集區識別碼。

   d. 針對 **batchUri** JSON 屬性，輸入 Batch URI。

      > [!IMPORTANT]
      > [Batch 帳戶] 刀鋒視窗中的 URL 格式如下：\<accountname\>.\<region\>.batch.azure.com。 針對 JSON 指令碼中的 **batchUri** 屬性，您必須從該 URL 中移除 a88"accountname."**。 例如 `"batchUri": "https://eastus.batch.azure.com"`。
      >
      >

      ![[Batch 帳戶] 刀鋒視窗](./media/data-factory-data-processing-using-batch/image9.png)

      針對 **poolName** 屬性，您也可以指定集區的識別碼，而非集區的名稱。

      > [!NOTE]
      > 與支援 HDInsight 的情況不同，Data Factory 服務針對 Batch 並不支援隨選選項。 在資料處理站中，您只能使用自己的 Batch 集區。
      >
      >
   
   e. 指定作業系統系列設定的 **StorageLinkedService** for the **StorageLinkedService** 。 您已在前述步驟中建立此連結服務。 此儲存體會做為檔案和記錄檔的預備區域。

1. 選取命令列上的 [部署] 以部署連結服務。

#### <a name="step-3-create-datasets"></a>步驟 3：建立資料集
在此步驟中，您會建立資料集來代表輸入和輸出資料。

#### <a name="create-the-input-dataset"></a>建立輸入資料集
1. 在「Data Factory 編輯器」中，選取工具列上的 [新增資料集]按鈕。 從下拉式清單中選取 [Azure Blob 儲存體]。

1. 使用下列 JSON 指令碼片段取代右窗格中的 JSON 指令碼：

    ```json
    {
       "name": "InputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
               "format": {
                   "type": "TextFormat"
               },
               "partitionedBy": [
                   {
                       "name": "Year",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "yyyy"
                       }
                   },
                   {
                       "name": "Month",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "MM"
                       }
                   },
                   {
                       "name": "Day",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "dd"
                       }
                   },
                   {
                       "name": "Hour",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "HH"
                       }
                   }
               ]
           },
           "availability": {
               "frequency": "Hour",
               "interval": 1
           },
           "external": true,
           "policy": {}
       }
    }
    ```

    您稍後會在本逐步解說中建立一個開始時間為 2015-11-16T00:00:00Z 而結束時間為 2015-11-16T05:00:00Z 的管線。 此管線排定在每小時產生資料，因此會有 5 個輸入/輸出配量 (在 **00**:00:00 -\> **05**:00:00 之間)。

    輸入資料集的 **frequency** 和 **interval** 已設定為 **Hour** 和 **1**，這意謂著每小時都會有輸入配量。

    上面 JSON 程式碼片段中的 **SliceStart** 系統變數代表每個配量的開始時間。 以下是每個配量的開始時間。

    | **配量** | **開始時間**          |
    |-----------|-------------------------|
    | 1         | 2015-11-16T**00**:00:00 |
    | 2         | 2015-11-16T**01**:00:00 |
    | 3         | 2015-11-16T**02**:00:00 |
    | 4         | 2015-11-16T**03**:00:00 |
    | 5         | 2015-11-16T**04**:00:00 |

    **FolderPath** 是使用配量開始時間 (**SliceStart**) 的年、月、日和小時部分來計算的。 以下是輸入資料夾對應至配量的方式。

    | **配量** | **開始時間**          | **輸入資料夾**  |
    |-----------|-------------------------|-------------------|
    | 1         | 2015-11-16T**00**:00:00 | 2015-11-16-**00** |
    | 2         | 2015-11-16T**01**:00:00 | 2015-11-16-**01** |
    | 3         | 2015-11-16T**02**:00:00 | 2015-11-16-**02** |
    | 4         | 2015-11-16T**03**:00:00 | 2015-11-16-**03** |
    | 5         | 2015-11-16T**04**:00:00 | 2015-11-16-**04** |

1. 選取工具列上的 [部署]，以建立並部署 **InputDataset** 資料表。

#### <a name="create-the-output-dataset"></a>建立輸出資料集
在此步驟中，您會建立類型為 AzureBlob 的另一個資料集來代表輸出資料。

1. 在「Data Factory 編輯器」中，選取工具列上的 [新增資料集]按鈕。 從下拉式清單中選取 [Azure Blob 儲存體]。

1. 使用下列 JSON 指令碼片段取代右窗格中的 JSON 指令碼：

    ```json
    {
       "name": "OutputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "fileName": "{slice}.txt",
               "folderPath": "mycontainer/outputfolder",
               "partitionedBy": [
                   {
                       "name": "slice",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "yyyy-MM-dd-HH"
                       }
                   }
               ]
           },
           "availability": {
               "frequency": "Hour",
               "interval": 1
           }
       }
    }
    ```

    為每個輸入配量產生輸出 blob/檔案。 以下是為每個配量命名輸出檔案的方式。 所有輸出檔案都會在一個輸出資料夾 `mycontainer\\outputfolder` 中產生。

    | **配量** | **開始時間**          | **輸出檔案**       |
    |-----------|-------------------------|-----------------------|
    | 1         | 2015-11-16T**00**:00:00 | 2015-11-16-**00.txt** |
    | 2         | 2015-11-16T**01**:00:00 | 2015-11-16-**01.txt** |
    | 3         | 2015-11-16T**02**:00:00 | 2015-11-16-**02.txt** |
    | 4         | 2015-11-16T**03**:00:00 | 2015-11-16-**03.txt** |
    | 5         | 2015-11-16T**04**:00:00 | 2015-11-16-**04.txt** |

    請記住，輸入資料夾 (例如 [2015-11-16-00]) 中的所有檔案都是開始時間為 2015-11-16-00 之配量的一部分。 處理此配量時，自訂活動會掃描每個檔案，然後利用搜尋詞彙 "Microsoft" 的出現次數在輸出檔案中產生一行。 如果資料夾 [2015-11-16-00] 中有三個檔案，輸出檔案 2015-11-16-00.txt 中就會有三行。

1. 選取工具列上的 [部署]，以建立並部署 **OutputDataset**。

#### <a name="step-4-create-and-run-the-pipeline-with-a-custom-activity"></a>步驟 4：建立並執行具有自訂活動的管線
在此步驟中，您會建立具有一個活動的管線，也就是您先前建立的自訂活動。

> [!IMPORTANT]
> 如果您尚未將 **file.txt** 上傳至 Blob 容器中的輸入資料夾，請先執行此動作，再建立管線。 在管線 JSON 中，**isPaused** 屬性會設定為 false，以便讓管線立即執行，因為 **start** 日期是在過去。
>
>

1. 在「Data Factory 編輯器」中，選取命令列上的 [新增管線]。 如果您沒有看到命令，請選取省略符號來顯示它。

1. 使用下列 JSON 指令碼片段取代右窗格中的 JSON 指令碼：

    ```json
    {
       "name": "PipelineCustom",
       "properties": {
           "description": "Use custom activity",
           "activities": [
               {
                   "type": "DotNetActivity",
                   "typeProperties": {
                       "assemblyName": "MyDotNetActivity.dll",
                       "entryPoint": "MyDotNetActivityNS.MyDotNetActivity",
                       "packageLinkedService": "AzureStorageLinkedService",
                       "packageFile": "customactivitycontainer/MyDotNetActivity.zip"
                   },
                   "inputs": [
                       {
                           "name": "InputDataset"
                       }
                   ],
                   "outputs": [
                       {
                           "name": "OutputDataset"
                       }
                   ],
                   "policy": {
                       "timeout": "00:30:00",
                       "concurrency": 5,
                       "retry": 3
                   },
                   "scheduler": {
                       "frequency": "Hour",
                       "interval": 1
                   },
                   "name": "MyDotNetActivity",
                   "linkedServiceName": "AzureBatchLinkedService"
               }
           ],
           "start": "2015-11-16T00:00:00Z",
           "end": "2015-11-16T05:00:00Z",
           "isPaused": false
      }
    }
    ```
   請注意下列幾點：

   * 管線中只有一個活動，且其類型為 **DotNetActivity**。
   * **AssemblyName** 已設定為 DLL **MyDotNetActivity.dll** 的名稱。
   * **EntryPoint** 設定為 **MyDotNetActivityNS.MyDotNetActivity**。 在您的程式碼中，它基本上是 \<namespace\>.\<classname\>。
   * **PackageLinkedService** 已設為 **StorageLinkedService**，這會指向包含自訂活動 zip 檔案的 Blob 儲存體。 如果您針對輸入/輸出檔案和自訂活動 zip 檔案使用不同的儲存體帳戶，就必須建立另一個儲存體已連結服務。 本文假設您使用相同的儲存體帳戶。
   * **PackageFile** 設定為 **customactivitycontainer/MyDotNetActivity.zip**。 其格式為 \<containerforthezip\>/\<nameofthezip.zip\>。
   * 自訂活動會採用 **InputDataset** 做為輸入和 **OutputDataset** 做為輸出。
   * 自訂活動的 **linkedServiceName** 屬性會指向 **AzureBatchLinkedService**，這可讓 Data Factory 知道自訂活動必須在 Batch 上執行。
   * **並行** 設定很重要。 如果您使用預設值 1，則即使 Batch 集區中有兩個以上的計算節點，系統仍會逐一處理配量。 因此，您將無法利用 Batch 的平行處理功能。 如果您將 **concurrency** 設定為更大的值 (例如 2)，即表示可以同時處理兩個配量 (對應至 Batch 中的兩個工作)。 在此情況下，會同時運用 Batch 集區中的兩個 VM。 請適當地設定 concurrency 屬性。
   * 根據預設，無論何時，一個工作 (配量) 都只會在一個 VM 上執行。 Batch 集區的 [每個 VM 的工作數上限] 預設是設定為 1。 為了符合先決條件，您在建立集區時已將此屬性設定為 2。 因此，可以在 VM 上同時執行兩個資料處理站配量。
    - **isPaused** 屬性預設是設定為 false。 在此範例中，管線會立即執行，因為配量已在過去開始。 您可以將此屬性設定為 **true** 以暫停管線，然後將其設定回 **false** 以重新啟動。
    -   **start** 和 **end** 時間相差 5 小時。 配量的產生頻率是每小時一次，因此管線會產生 5 個配量。

1. 選取命令列上的 [部署] 以部署管線。

#### <a name="step-5-test-the-pipeline"></a>步驟 5：測試管線
在此步驟中，您會將檔案放置在輸入資料夾中，以測試管線。 首先，在每個輸入資料夾中放置一個檔案來測試管線。

1. 在 Azure 入口網站的 [資料處理站] 刀鋒視窗中，選取 [圖表]。

   ![圖表](./media/data-factory-data-processing-using-batch/image10.png)

1. 在 [圖表] 檢視中，按兩下輸入資料集 [InputDataset]。

   ![InputDataset](./media/data-factory-data-processing-using-batch/image11.png)

1. 隨即會出現 [InputDataset] 刀鋒視窗，內含全部 5 個已就緒的配量。 請留意每個配量的 [配量開始時間] 和 [配量結束時間]。

   ![輸入配量的開始和結束時間](./media/data-factory-data-processing-using-batch/image12.png)

1. 在 [圖表] 檢視中，選取 [OutputDataset]。

1. 如果已產生 5 個輸出配量，這 5 個配量的狀態就會顯示為 [就緒]。

   ![輸出配量的開始和結束時間](./media/data-factory-data-processing-using-batch/image13.png)

1. 使用入口網站來檢視與配量關聯的工作，並查看每個配量在哪個 VM 上執行。 如需詳細資訊，請參閱 [Data Factory 和 Batch 整合](#data-factory-and-batch-integration)一節。

1. 輸出檔案會出現在您 Blob 儲存體內 `outputfolder` 中的 `mycontainer` 底下。

   ![儲存體中的輸出檔案](./media/data-factory-data-processing-using-batch/image15.png)

   列出的輸出檔案有 5 個，每個輸入配量各一個輸出檔案。 每個輸出檔案都有類似下列輸出的內容：

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
    ```
   下圖說明資料處理站配量如何對應至 Batch 中的工作。 在此範例中，一個配量只有一個執行。

   ![配量對應圖](./media/data-factory-data-processing-using-batch/image16.png)

1. 現在，嘗試在一個資料夾中放置多個檔案來進行操作。 使用與資料夾 [2015-11-06-01] 中 file.txt 相同的內容，來建立檔案 **file2.txt**、**file3.txt**、**file4.txt** 和 **file5.txt**。

1. 在輸出資料夾中，刪除輸出檔案 **2015-11-16-01.txt**。

1. 在 [OutputDataset] 刀鋒視窗上，於 [配量開始時間] 設定為 **11/16/2015 01:00:00 AM** 的配量上按一下滑鼠右鍵。 選取 [執行] 以重新執行/重新處理配量。 現在，配量就會有 5 個檔案，而不是 1 個檔案。

    ![執行](./media/data-factory-data-processing-using-batch/image17.png)

1. 在配量執行且其狀態變成 [就緒] 之後，驗證此配量之輸出檔案 (**2015-11-16-01.txt**) 中的內容。 此輸出檔案會出現在您 Blob 儲存體內 `outputfolder` 中的 `mycontainer` 底下。 配量的每個檔案應該都有一行。

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file2.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file3.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file4.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file5.txt.
    ```

> [!NOTE]
> 如果您未先刪除輸出檔案 2015-11-16-01.txt，即嘗試以 5 個輸入檔案進行操作，就會看到有一行來自先前的配量執行，而有五行來自目前的配量執行。 內容預設會附加至已存在的輸出檔案。
>
>

#### <a name="data-factory-and-batch-integration"></a>Data Factory 和 Batch 整合
Data Factory 服務會在 Batch 中建立一個名為 `adf-poolname:job-xxx` 的作業。

![Batch 作業](media/data-factory-data-processing-using-batch/data-factory-batch-jobs.png)

配量的每個活動執行都會在作業中建立一個工作。 如果可供處理的配量有 10 個，作業中就會建立 10 個工作。 如果您在集區中有多個計算結點，您可以同時執行多個配量。 如果每個計算結點的工作數上限設定為大於 1，便可在同一個計算上執行多個配量。

此範例中有 5 個配量，因此 Batch 中有 5 個工作。 只要將資料處理站之管線 JSON 中的 **concurrency** 設定為 **5**，並將含有 **2** 個 VM 之 Batch 集區中的 [每個 VM 的工作數上限] 設定為 **2**，工作便可快速執行。 (請檢查工作的開始和結束時間)。

使用入口網站來檢視與配量關聯的 Batch 作業及其工作，並查看每個配量在哪個 VM 上執行。

![Batch 作業工作](media/data-factory-data-processing-using-batch/data-factory-batch-job-tasks.png)

### <a name="debug-the-pipeline"></a>偵錯管線
偵錯包含幾個基本技巧。

1. 如果輸入配量不是設定為 [就緒]，請確認輸入資料夾結構正確，並且輸入資料夾中有 file.txt。

   ![輸入資料夾結構](./media/data-factory-data-processing-using-batch/image3.png)

1. 在自訂活動的 **Execute** 方法中，使用可協助您針對問題進行疑難排解的 **IActivityLogger** 物件記錄資訊。 記錄的訊息會顯示在 user\_0.log 檔案中。

   在 [OutputDataset] 刀鋒視窗上，選取配量以查看該配量的 [資料配量] 刀鋒視窗。 在 [活動回合] 底下，您會看到該配量有一個活動回合。 如果您選取命令列中的 [執行]，便可針對同一個配量啟動另一個活動回合。

   當您選取活動回合時，會看到含有記錄檔清單的 [活動回合詳細資料] 刀鋒視窗。 您會在 user\_0.log 檔案中看到已記錄的訊息。 發生錯誤時，您會看到三個活動執行，因為管線/活動 JSON 中的重試計數設定為 3。 當您選取活動回合時，會看到可供您檢閱來針對錯誤進行疑難排解的記錄檔。

   ![[OutputDataset] 和 [資料配量] 刀鋒視窗](./media/data-factory-data-processing-using-batch/image18.png)

   在記錄檔清單中，選取 [user-0.log]。 在右面板中，會出現使用 **IActivityLogger.Write** 方法的結果。

   ![[活動回合詳細資料] 刀鋒視窗](./media/data-factory-data-processing-using-batch/image19.png)

   請檢查 system-0.log 是否有任何系統錯誤訊息和例外狀況。

    ```
    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Loading assembly file MyDotNetActivity...
    
    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Creating an instance of MyDotNetActivityNS.MyDotNetActivity from assembly file MyDotNetActivity...
    
    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Executing Module
    
    Trace\_T\_D\_12/6/2015 1:43:38 AM\_T\_D\_\_T\_D\_Information\_T\_D\_0\_T\_D\_Activity e3817da0-d843-4c5c-85c6-40ba7424dce2 finished successfully
    ```
1. 在 zip 檔案中包含 **PDB** 檔案，如此錯誤詳細資料才會包含錯誤發生時的呼叫堆疊等資訊。

1. 自訂活動之 zip 檔案中的所有檔案都必須位於最上層，且沒有任何子資料夾。

   ![自訂活動 zip 檔案清單](./media/data-factory-data-processing-using-batch/image20.png)

1. 確定 **assemblyName** (MyDotNetActivity.dll)、**entryPoint** (MyDotNetActivityNS.MyDotNetActivity)、**packageFile** (customactivitycontainer/MyDotNetActivity.zip) 和 **packageLinkedService** (應指向包含 zip 檔案的 Blob 儲存體) 都已設定為正確的值。

1. 如果您已修正某個錯誤，而想要重新處理配量，請在 [OutputDataset] 刀鋒視窗中的該配量上按一下滑鼠右鍵，然後選取 [執行]。

   ![[OutputDataset] 刀鋒視窗的 [執行] 選項](./media/data-factory-data-processing-using-batch/image21.png)

   > [!NOTE]
   > 在 Blob 儲存體中會有一個名為 `adfjobs`.的容器。 系統不會自動刪除此容器，但您可在完成解決方案測試之後，放心地刪除此容器。 同樣地，資料處理站解決方案也會建立一個名為 `adf-\<pool ID/name\>:job-0000000001` 的 Batch 作業。 您可以在測試解決方案之後刪除此作業 (如果您要的話)。
   >
   >
1. 自訂活動不會使用來自您套件的 **app.config** 檔案。 因此，如果您的程式碼會從組態檔讀取任何連接字串，在執行階段就會無法運作。 使用 Batch 時，最佳做法是將所有祕密都存放在 Azure Key Vault 中。 然後使用憑證型服務主體來保護 Key Vault，再將憑證散發至 Batch 集區。 .NET 自訂活動將可在執行階段從 Key Vault 存取祕密。 這個一般解決方案可以擴展至任何類型的祕密，而不僅限於連接字串。

    有一個較簡單的因應措施，但並非最佳做法。 您可以建立一個具有連接字串設定的 SQL Database 已連結服務。 接著，建立一個使用該已連結服務的資料集，然後將資料集以虛擬輸入資料集的形式鏈結至自訂 .NET 活動。 接著，您便可以在自訂活動程式碼中存取連結服務的連接字串。 這樣應該可以在執行階段正常運作。  

#### <a name="extend-the-sample"></a>擴充範例
您可以延伸這個範例來深入了解 Data Factory 和 Batch 功能。 例如，若要處理不同時間範圍的配量，請執行下列步驟：

1. 在 `inputfolder` 中新增下列子資料夾：2015-11-16-05、2015-11-16-06、201-11-16-07、2011-11-16-08 及 2015-11-16-09。 將輸入檔案放在這些資料夾中。 將管線的結束時間從 `2015-11-16T05:00:00Z` 變更為 `2015-11-16T10:00:00Z`。 在 [圖表] 檢視中，按兩下 [InputDataset]，並確認輸入配量已就緒。 按兩下 [OutputDataset] 以查看輸出配量的狀態。 如果它們的狀態為 [就緒]，則請查看輸出檔案的輸出資料夾。

1. 提高或降低 **concurrency** 設定的值，以了解它對您解決方案 (尤其是在 Batch 上進行的處理) 效能的影響。 如需有關 **concurrency** 設定的詳細資訊，請參閱＜步驟 4：建立並執行具有自訂活動的管線＞。

1. 建立 [每個 VM 的工作數上限] 較低/較高的集區。 若要使用您建立的新集區，請更新資料處理站解決方案中的 Batch 已連結服務。 如需有關 [每個 VM 的工作數上限] 設定的詳細資訊，請參閱＜步驟 4：建立並執行具有自訂活動的管線＞。

1. 建立具有**自動調整**功能的 Batch 集區。 自動調整 Batch 集區中的計算節點係指動態調整應用程式所使用的處理能力。 

    這裡的範例公式會產生下列行為。 一開始建立集區時，會從一個 VM 開始。 $PendingTasks 計量會定義處於執行中和作用中 (已排入佇列) 狀態的工作數目。 公式會尋找過去 180 秒內的平均擱置中工作數目，並據以設定 TargetDedicated。 它會確保 TargetDedicated 一律不會超過 25 部 VM。 集區會隨著新工作的提交而自動成長。 隨著工作完成，VM 會逐一變成可用，而自動調整功能將會縮減這些 VM。 您可以視需要調整 startingNumberOfVMs 和 maxNumberofVMs。
 
    自動調整公式：

    ``` 
    startingNumberOfVMs = 1;
    maxNumberofVMs = 25;
    pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
    pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
    $TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
    ```

   如需詳細資訊，請參閱[自動調整 Batch 集區中的計算節點](../../batch/batch-automatic-scaling.md)。

   如果集區使用預設的 [autoScaleEvaluationInterval](https://msdn.microsoft.com/library/azure/dn820173.aspx)，Batch 服務在執行自訂活動之前，可能需要 15 到 30 分鐘的時間來準備 VM。 如果集區使用不同的 autoScaleEvaluationInterval，則 Batch 服務所需的時間可能為 autoScaleEvaluationInterval 加 10 分鐘。

1. 在範例解決方案中，**Execute** 方法會叫用可處理輸入資料配量以產生輸出資料配量的 **Calculate** 方法。 您可以自行撰寫方法來處理輸入資料，然後在 **Execute** 方法中呼叫您的方法來取代呼叫 **Calculate** 方法。

### <a name="next-steps-consume-the-data"></a>後續步驟：取用資料
處理資料之後，您可以使用 Power BI 之類的線上工具來取用資料。 以下連結可協助您了解 Power BI，以及如何在 Azure 中加以使用：

* [在 Power BI 中探索資料集](https://powerbi.microsoft.com/documentation/powerbi-service-get-data/)
* [開始使用 Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-getting-started/)
* [重新整理 Power BI 中的資料](https://powerbi.microsoft.com/documentation/powerbi-refresh-data/)
* [Azure 和 Power BI：基本概觀](https://powerbi.microsoft.com/documentation/powerbi-azure-and-power-bi/)

## <a name="references"></a>參考
* [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/)

  * [Data Factory 服務簡介](data-factory-introduction.md)
  * [開始使用 Data Factory](data-factory-build-your-first-pipeline.md)
  * [在 Data Factory 管線中使用自訂活動](data-factory-use-custom-activities.md)
* [Azure Batch](https://azure.microsoft.com/documentation/services/batch/)

  * [Batch 的基本概念](../../batch/batch-technical-overview.md)
  * [Batch 功能概觀](../../batch/batch-api-basics.md)
  * [在 Azure 入口網站中建立和管理 Batch 帳戶](../../batch/batch-account-create-portal.md)
  * [開始使用適用於 .NET 的 Batch 用戶端程式庫](../../batch/quick-run-dotnet.md)

[batch-explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[batch-explorer-walkthrough]: http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx
