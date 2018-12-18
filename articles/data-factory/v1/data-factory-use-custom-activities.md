---
title: 在 Azure 資料處理站管線中使用自訂活動
description: 了解如何建立自訂活動，並在 Azure 資料處理站管線中使用這些活動。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.assetid: 8dd7ba14-15d2-4fd9-9ada-0b2c684327e9
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: douglasl
robots: noindex
ms.openlocfilehash: 580dd5bf6a7e905927189f4b1ae42ab49a1cbc80
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45730707"
---
# <a name="use-custom-activities-in-an-azure-data-factory-pipeline"></a>在 Azure 資料處理站管線中使用自訂活動
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-use-custom-activities.md)
> * [第 2 版 (目前的版本)](../transform-data-using-dotnet-custom-activity.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的自訂活動](../transform-data-using-dotnet-custom-activity.md)。

您可以在 Azure Data Factory 管線中使用兩種活動。

- [資料移動活動](data-factory-data-movement-activities.md)，可在[支援的來源與接收資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)之間移動資料。
- 使用 Azure HDInsight、Azure Batch 及 Azure Machine Learning 等計算服務來轉換資料的[資料轉換活動](data-factory-data-transformation-activities.md)。 

如果要移動資料至/自 Data Factory 不支援的資料存放區，利用自己的資料移動邏輯建立**自訂活動**，然後在管線中使用活動。 同樣地，若要以 Data Factory 不支援的方法轉換/處理資料，可以利用自己的資料轉換邏輯建立自訂活動，然後在管線中使用活動。 

您可以設定自訂活動以在虛擬機器的 **Azure Batch** 集區上執行。 當使用 Azure Batch 時，您只可以使用現有的 Azure Batch 集區。

下列逐步解說提供建立自訂 .NET 活動以及在管線中使用自訂活動的逐步指示。 本逐步解說使用 **Azure Batch** 連結服務。 

> [!IMPORTANT]
> - 您不能使用自訂活動中的資料管理閘道來存取內部部署資料來源。 目前在 Data Factory 中，[資料管理閘道](data-factory-data-management-gateway.md)只支援複製活動和預存程序活動。   

## <a name="walkthrough-create-a-custom-activity"></a>逐步解說：建立自訂活動
### <a name="prerequisites"></a>必要條件
* Visual Studio 2012/2013/2015
* 下載並安裝 [Azure .NET SDK](https://azure.microsoft.com/downloads/)

### <a name="azure-batch-prerequisites"></a>Azure Batch 的必要條件
在逐步解說中，您會將 Azure Batch 當作計算資源使用來執行自訂 .NET 活動。 **Azure Batch** 是一項平台服務，可用於在雲端有效地執行大規模的平行和高效能運算 (HPC) 應用程式。 Azure Batch 可排程要在**一組受控虛擬機器**上執行的計算密集型工作，而且可以調整計算資源以符合工作的需求。 請參閱 [Azure Batch 基本知識][batch-technical-overview]文章，以取得 Azure Batch 服務的詳細概觀。

在教學課程中，建立含 VM 集區的 Azure Batch 帳戶。 步驟如下：

1. 使用 **Azure 入口網站** 建立 [Azure Batch 帳戶](http://portal.azure.com)。 請參閱[建立和管理 Azure Batch 帳戶][batch-create-account]一文以取得指示。
2. 記下 Azure Batch 帳戶名稱、帳戶金鑰、URI，以及集區名稱。 您需要它們來建立 Azure Batch 連結服務。
    1. 在 Azure Batch 帳戶首頁上，您會看到一串 URL 為下列格式︰`https://myaccount.westus.batch.azure.com`。 在此範例中，**myaccount** 是 Azure Batch 帳戶的名稱。 您在連結服務的定義中使用之 URI 是不含帳戶名稱的 URL。 例如： `https://<region>.batch.azure.com` 。
    2. 在左窗格上按一下 [金鑰]，然後複製**主要存取金鑰**。
    3. 若要使用現有的集區，在功能表上按一下 [集區]，記下集區的**識別碼**。 如果您沒有現有的集區，請移至下一個步驟。     
2. 建立 **Azure Batch 集區**。

   1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下左側功能標中的 [瀏覽]，然後按一下 [Batch 帳戶]。
   2. 選取您的 Azure Batch 帳戶，以開啟 [Batch 帳戶]  刀鋒視窗。
   3. 按一下 [集區]  圖格。
   4. 在 [集區]  刀鋒視窗中，按一下工具列上的 [新增] 按鈕以新增集區。
      1. 輸入集區的識別碼 (集區識別碼)。 請注意 **集區的識別碼**；您在建立 Data Factory 解決方案時需要它。
      2. 指定作業系統系列設定的 **Windows Server 2012 R2** 。
      3. 選取 **節點定價層**。
      4. 輸入 **2** 做為 [目標專用] 設定的值。
      5. 輸入 **2** 做為 [每個節點的工作上限] 設定的值。
   5. 按一下 [確定]  以建立集區。
   6. 記下集區的**識別碼**。 



### <a name="high-level-steps"></a>高階步驟
以下是您在此逐步解說中執行的兩個高階步驟︰ 

1. 建立包含簡單資料轉換/處理邏輯的自訂活動。
2. 透過使用自訂活動的管線建立 Azure Data Factory。

### <a name="create-a-custom-activity"></a>建立自訂活動
若要建立 .NET 自訂活動，您必須利用實作 **IDotNetActivity** 介面的類別建立 **.NET 類別庫**專案。 這個介面只有一個方法： [Execute](https://msdn.microsoft.com/library/azure/mt603945.aspx) ，其簽章為：

```csharp
public IDictionary<string, string> Execute(
        IEnumerable<LinkedService> linkedServices,
        IEnumerable<Dataset> datasets,
        Activity activity,
        IActivityLogger logger)
```


此方法會採用四個參數：

- **linkedServices**。 這個屬性是活動之輸入/輸出資料集所參考的資料存放區連結服務的可列舉清單。   
- **資料集**。 這個屬性是活動之輸入/輸出資料集的可列舉清單。 您可以使用這個參數取得輸入和輸出資料集定義的位置和結構描述。
- **活動**。 這個屬性表示目前的活動。 它可以用來存取與自訂活動相關聯的延伸屬性。 如需詳細資訊，請參閱[存取延伸屬性](#access-extended-properties)。
- **logger**。 此物件可讓您撰寫會呈現為管線的使用者記錄檔的偵錯註解。

此方法會傳回未來可用來將自訂活動鏈結在一起的字典。 尚未實作這項功能，因此會從方法傳回空的字典。  

### <a name="procedure"></a>程序
1. 建立 **.NET 類別庫** 專案。
   <ol type="a">
     <li>啟動 <b>Visual Studio 2017</b> 或 <b>Visual Studio 2015</b> 或 <b>Visual Studio 2013</b> 或 <b>Visual Studio 2012</b>。</li>
     <li>按一下 [檔案]<b></b>，指向 [新增]<b></b>，然後按一下 [專案]<b></b>。</li>
     <li>展開 [範本]<b></b>，然後選取 [Visual C#]<b></b>。 在此逐步解說中，您使用 C# 中，但您可以使用任何 .NET 語言來開發自訂活動。</li>
     <li>從右邊的專案類型清單中選取 [類別庫]<b></b>。 在 VS 2017 中，選擇 <b>類別庫 (.NET Framework)</b> </li>
     <li>針對 [名稱]<b></b> 輸入 <b>MyDotNetActivity</b>。</li>
     <li>在 [位置]<b></b> 選取 <b>C:\ADFGetStarted</b>。</li>
     <li>按一下 [確定] <b></b> 以建立專案。</li>
   </ol>
   
2. 按一下 [**工具**]，指向 [**NuGet 封裝管理員**]，然後按一下 [**封裝管理員主控台**]。

3. 在 [封裝管理員主控台] 中，執行下列命令以匯入 **Microsoft.Azure.Management.DataFactories**。

    ```PowerShell
    Install-Package Microsoft.Azure.Management.DataFactories
    ```
4. 將 **Azure 儲存體** NuGet 封裝匯入專案中。

    ```PowerShell
    Install-Package WindowsAzure.Storage -Version 4.3.0
    ```

    > [!IMPORTANT]
    > Data Factory 服務啟動程式需要 4.3 版的 WindowsAzure.Storage。 如果您在自訂活動專案中新增新版的 Azure 儲存體組件的參考，當活動執行時會看到錯誤。 若要解決這個錯誤，請參閱[ Appdomain 隔離](#appdomain-isolation)小節。 
5. 將下列 **using** 陳述式加入至專案的原始程式檔。

    ```csharp

    // Comment these lines if using VS 2017
    using System.IO;
    using System.Globalization;
    using System.Diagnostics;
    using System.Linq;
    // --------------------

    // Comment these lines if using <= VS 2015
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    // ---------------------

    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Runtime;

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
6. 將 **namespace** 的名稱變更為 **MyDotNetActivityNS**。

    ```csharp
    namespace MyDotNetActivityNS
    ```
7. 將類別的名稱變更為 **MyDotNetActivity**，並從 **IDotNetActivity** 介面衍生它，如下列程式碼片段所示：

    ```csharp
    public class MyDotNetActivity : IDotNetActivity
    ```
8. 對 **MyDotNetActivity** 類別實作 (新增) **IDotNetActivity** 介面的 **Execute** 方法，並將下列範例程式碼複製到方法。

    下列範例會在每個與資料配量相關聯之 Blob 中計算搜尋詞彙 ("Microsoft") 的出現次數。

    ```csharp
    /// <summary>
    /// Execute method is the only method of IDotNetActivity interface you must implement.
    /// In this sample, the method invokes the Calculate method to perform the core logic.  
    /// </summary>
    
    public IDictionary<string, string> Execute(
        IEnumerable<LinkedService> linkedServices,
        IEnumerable<Dataset> datasets,
        Activity activity,
        IActivityLogger logger)
    {
        // get extended properties defined in activity JSON definition
        // (for example: SliceStart)
        DotNetActivity dotNetActivity = (DotNetActivity)activity.TypeProperties;
        string sliceStartString = dotNetActivity.ExtendedProperties["SliceStart"];
    
        // to log information, use the logger object
        // log all extended properties            
        IDictionary<string, string> extendedProperties = dotNetActivity.ExtendedProperties;
        logger.Write("Logging extended properties if any...");
        foreach (KeyValuePair<string, string> entry in extendedProperties)
        {
            logger.Write("<key:{0}> <value:{1}>", entry.Key, entry.Value);
        }
    
        // linked service for input and output data stores
        // in this example, same storage is used for both input/output
        AzureStorageLinkedService inputLinkedService;

        // get the input dataset
        Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);
    
        // declare variables to hold type properties of input/output datasets
        AzureBlobDataset inputTypeProperties, outputTypeProperties;
        
        // get type properties from the dataset object
        inputTypeProperties = inputDataset.Properties.TypeProperties as AzureBlobDataset;
    
        // log linked services passed in linkedServices parameter
        // you will see two linked services of type: AzureStorage
        // one for input dataset and the other for output dataset 
        foreach (LinkedService ls in linkedServices)
            logger.Write("linkedService.Name {0}", ls.Name);
    
        // get the first Azure Storate linked service from linkedServices object
        // using First method instead of Single since we are using the same
        // Azure Storage linked service for input and output.
        inputLinkedService = linkedServices.First(
            linkedService =>
            linkedService.Name ==
            inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
            as AzureStorageLinkedService;
    
        // get the connection string in the linked service
        string connectionString = inputLinkedService.ConnectionString;
    
        // get the folder path from the input dataset definition
        string folderPath = GetFolderPath(inputDataset);
        string output = string.Empty; // for use later.
    
        // create storage client for input. Pass the connection string.
        CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
        CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();
    
        // initialize the continuation token before using it in the do-while loop.
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
    
            // Calculate method returns the number of occurrences of
            // the search term (“Microsoft”) in each blob associated
               // with the data slice. definition of the method is shown in the next step.
    
            output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");
    
        } while (continuationToken != null);
    
        // get the output dataset using the name of the dataset matched to a name in the Activity output collection.
        Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

        // get type properties for the output dataset
        outputTypeProperties = outputDataset.Properties.TypeProperties as AzureBlobDataset;
    
        // get the folder path from the output dataset definition
        folderPath = GetFolderPath(outputDataset);

        // log the output folder path   
        logger.Write("Writing blob to the folder: {0}", folderPath);
    
        // create a storage object for the output blob.
        CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
        // write the name of the file.
        Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));
    
        // log the output file name
        logger.Write("output blob URI: {0}", outputBlobUri.ToString());

        // create a blob and upload the output text.
        CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
        logger.Write("Writing {0} to the output blob", output);
        outputBlob.UploadText(output);
    
        // The dictionary can be used to chain custom activities together in the future.
        // This feature is not implemented yet, so just return an empty dictionary.  
    
        return new Dictionary<string, string>();
    }
    ```
9. 新增下列 Helper 方法： 

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

        // get type properties of the dataset   
        AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
        if (blobDataset == null)
        {
            return null;
        }
    
        // return the folder path found in the type properties
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
    
        // get type properties of the dataset
        AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
        if (blobDataset == null)
        {
            return null;
        }
    
        // return the blob/file name in the type properties
        return blobDataset.FileName;
    }
    
    /// <summary>
    /// Iterates through each blob (file) in the folder, counts the number of instances of search term in the file,
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

    GetFolderPath 方法會將路徑傳回資料集所指向的資料夾，而 GetFileName 方法會傳回資料集指向的 blob/檔案名稱。 如果您的 havefolderPath 定義使用如 {Year}、{Month}、{Day} 等變數，方法會以未將變數取代為執行階段值的形式傳回字串。 如需存取 SliceStart、SliceEnd 等的詳細資料，請參閱 [存取延伸屬性](#access-extended-properties) 一節。    

    ```JSON
    "name": "InputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "file.txt",
            "folderPath": "adftutorial/inputfolder/",
    ```

    Calculate 方法會在輸入檔案 (資料夾中的 blob) 計算關鍵字 Microsoft 的執行個體數目。 搜尋詞彙 ("Microsoft") 已在程式碼中硬式編碼。
10. 編譯專案。 按一下功能表中的 [建置]，然後按一下 [建置方案]。

    > [!IMPORTANT]
    > 將 .NET Framework 4.5.2 版設定為您專案的目標架構：在專案上按一下滑鼠右鍵，然後按一下 [屬性] 來設定目標架構。 Data Factory 不支援針對 .NET Framework 4.5.2 版之後的版本編譯的自訂活動。

11. 啟動 [Windows 檔案總管]，瀏覽至 **bin\debug** 或 **bin\release** 資料夾，視建置類型而定。
12. 建立 zip 檔案 **MyDotNetActivity.zip**，檔案中包含 <project folder>\bin\Debug 資料夾中的所有二進位檔。 新增 **MyDotNetActivity.pdb** 檔案，讓您可以取得額外的詳細資訊，例如如果有失敗時，原始程式碼中引起問題的程式碼行號。 

    > [!IMPORTANT]
    > 自訂活動之 zip 檔案中的所有檔案都必須位於 **最上層** 且不包含任何子資料夾。

    ![二進位輸出檔案](./media/data-factory-use-custom-activities/Binaries.png)
14. 如果名為 **customactivitycontainer** 的 Blob 容器不存在，請自行建立。 
15. 將 MyDotNetActivity.zip 作為 blob，上傳至 AzureStorageLinkedService 所參照之**一般用途** Azure Blob 儲存體 (而不是經常性/非經常性 Blob 儲存體) 中的 customactivitycontainer。  

> [!IMPORTANT]
> 如果您將這個 .NET 活動專案加入 Visual Studio 中包含 Data Factory 專案的方案，並從 Data Factory 應用程式專案加入 .NET 活動的參考，您就不需要執行最後兩個步驟，也就是建立 zip 檔案，和手動上傳到一般用途 Azure Blob 儲存體。 當您使用 Visual Studio 發佈 Data Factory 實體時，發佈程序會自動完成這些步驟。 如需詳細資訊，請參閱[Visual Studio 中的 Data Factory 專案](#data-factory-project-in-visual-studio)一節。

## <a name="create-a-pipeline-with-custom-activity"></a>建立具有自訂活動的管線
您已建立自訂活動，並將二進位檔的 zip 檔案上傳至**一般用途** Azure 儲存體帳戶中的 blob 容器。 在本節中，您將透過使用自訂活動的管線建立 Azure Data Factory。

自訂活動的輸入資料集代表 blob 儲存體中 adftutorial 容器之 customactivityinput 資料夾中的輸入 blob (檔案)。 活動的輸出資料集代表 blob 儲存體中 adftutorial 容器之 customactivityinput 資料夾中的輸出 blob。

以下列內容建立 **file.txt** 檔案，然後將它上傳至 **adftutorial** 容器的 **customactivityinput** 資料夾。 建立 adftutorial 容器 (如果尚未存在)。 

```
test custom activity Microsoft test custom activity Microsoft
```

即使資料夾有 2 個以上的檔案，輸入資料夾還是會對應至 Azure Data Factory 中的配量。 管線處理每個配量時，自訂活動會為該配量逐一查看輸入資料夾中的所有 blob。

您會看到 adftutorial\customactivityoutput 資料夾中的一個輸出檔案具有 1 行或更多行 (和輸入資料夾中的 blob 數目相同)：

```
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2016-11-16-00/file.txt.
```


以下是您會在此節中執行的步驟：

1. 建立 **Data Factory**。
2. 建立自訂活動執行所在之 Azure Batch VM 集區的連結服務，以及容納輸入/輸出 Blob 之 Azure 儲存體的**連結服務**。
3. 建立輸入和輸出**資料集**，代表自訂活動的輸入和輸出。
4. 建立使用自訂活動的**管線**。

> [!NOTE]
> 建立 **file.txt** 並上傳到 Blob 容器 (如果您尚未完成)。 請參閱上一節中的指示。   

### <a name="step-1-create-the-data-factory"></a>步驟 1：建立 Data Factory
1. 登入 Azure 入口網站之後，執行下列步驟：
   1. 在左功能表中，按一下 [建立資源]。
   2. 按一下 [新增] 刀鋒視窗中的 [資料 + 分析]。
   3. 按一下 [資料分析] 刀鋒視窗上的 [Data Factory]。
   
    ![新增 Azure Data Factory 功能表](media/data-factory-use-custom-activities/new-azure-data-factory-menu.png)
2. 在 [新增 Data Factory] 刀鋒視窗中，輸入 **CustomActivityFactor** 做為 [名稱]。 Azure Data Factory 的名稱在全域必須是唯一的。 如果您收到錯誤：**Data Factory 名稱 “CustomActivityFactory” 無法使用**，請變更 Data Factory 名稱 (例如 **yournameCustomActivityFactory**)，然後試著重新建立。

    ![新增 Azure Data Factory 刀鋒視窗](media/data-factory-use-custom-activities/new-azure-data-factory-blade.png)
3. 按一下 [資源群組名稱] ，並選取現有的資源群組，或建立一個群組。
4. 請確認您使用的是要在其中建立 Data Factory 的正確**訂用帳戶**和**區域**。
5. 按一下 [新增 Data Factory] 刀鋒視窗上的 [建立]。
6. 您會看到 Data Factory 建立在 Azure 入口網站的 [儀表板]  中。
7. 在 Data Factory 成功建立後，您會看到 Data Factory 刀鋒視窗，顯示 Data Factory 的內容。
    
    ![Data Factory 刀鋒視窗](media/data-factory-use-custom-activities/data-factory-blade.png)

### <a name="step-2-create-linked-services"></a>步驟 2：建立連結服務
連結服務會將資料存放區或計算服務連結至 Azure Data Factory。 在此步驟中，您會將 Azure 儲存體帳戶和 Azure Batch 帳戶連結到 Data Factory。

#### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
1. 按一下 **CustomActivityFactory** 的 [DATA FACTORY] 刀鋒視窗上的 [作者和部署] 圖格。 您會看到 [Data Factory 編輯器]。
2. 在命令列上按一下 [新增資料儲存區]，然後選擇 [Azure 儲存體]。 在編輯器中，您應該會看到用來建立 Azure 儲存體連結服務的 JSON 指令碼。
    
    ![新增資料存放區 - Azure 儲存體](media/data-factory-use-custom-activities/new-data-store-menu.png)
3. 以您的 Azure 儲存體帳戶名稱取代`<accountname>`，並以 Azure 儲存體帳戶的存取金鑰取代`<accountkey>`。 若要了解如何取得儲存體存取金鑰，請參閱 [檢視、複製和重新產生儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)

    ![Azure 儲存體類型的連結服務](media/data-factory-use-custom-activities/azure-storage-linked-service.png)
4. 按一下命令列的 [部署]  ，部署連結服務。

#### <a name="create-azure-batch-linked-service"></a>建立 Azure Batch 連結服務
1. 在 [Data Factory 編輯器] 中，按一下工具列上的 [...**其他]**，按一下 [新增計算]，然後從功能表選取 [Azure 批次]。

    ![[新增計算]-[Azure 批次]](media/data-factory-use-custom-activities/new-azure-compute-batch.png)
2. 對 JSON 指令碼進行下列變更：

   1. 指定 **accountName** 屬性的 Azure Batch 帳戶名稱。 [Azure Batch 帳戶] 刀鋒視窗中的 **URL** 格式如下：`http://accountname.region.batch.azure.com`。 如果是 JSON 中的 **batchUri** 屬性，您需要移除 URL 中的 `accountname.`，並為 JSON 屬性使用 `accountname``accountName`。
   2. 指定 **accessKey** 屬性的 Azure Batch 帳戶金鑰 。
   3. 針對為滿足 **poolName** 屬性之必要條件而建立的集區指定名稱。 您也可以指定該集區的 ID，而非集區名稱。
   4. 指定 **batchUri** 屬性的 Azure Batch URI。 範例： `https://westus.batch.azure.com`.  
   5. 指定 **AzureStorageLinkedService** for the **linkedServiceName** 屬性的 Azure Batch 帳戶名稱。

        ```json
        {
         "name": "AzureBatchLinkedService",
         "properties": {
           "type": "AzureBatch",
           "typeProperties": {
             "accountName": "myazurebatchaccount",
             "batchUri": "https://westus.batch.azure.com",
             "accessKey": "<yourbatchaccountkey>",
             "poolName": "myazurebatchpool",
             "linkedServiceName": "AzureStorageLinkedService"
           }
         }
        }
        ```

       針對 **poolName** 屬性，您也可以指定該集區的 ID，而非集區名稱。

    

### <a name="step-3-create-datasets"></a>步驟 3：建立資料集
在此步驟中，您會建立資料集來代表輸入和輸出資料。

#### <a name="create-input-dataset"></a>建立輸入資料集
1. 在 Data Factory 的 [編輯器] 中，依序按一下下拉式功能表中的 **[...其他]**，按一下 [新增資料集]，然後從下拉式功能表選取 [Azure Blob 儲存體]。
2. 使用下列 JSON 程式碼片段取代右窗格中的 JSON：

    ```json
    {
     "name": "InputDataset",
     "properties": {
         "type": "AzureBlob",
         "linkedServiceName": "AzureStorageLinkedService",
         "typeProperties": {
             "folderPath": "adftutorial/customactivityinput/",
             "format": {
                 "type": "TextFormat"
             }
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

   您稍後會在本逐步解說建立管線，開始時間為：2016-11-16T00:00:00Z，而結束時間為：2016-11-16T05:00:00Z。 排程為每小時產生，因此會有 5 個輸入/輸出配量 (在 **00**:00:00 -> **05**:00:00 之間)。

   輸入資料集的 **frequency** 和 **interval** 設定為 **Hour** 和 **1**，這表示每小時皆可使用輸入配量。 在此範例中，它是 intputfolder 中的相同檔案 (file.txt)。

   以下是每個配量的開始時間，由上述 JSON 程式碼片段中的 SliceStart 系統變數代表。
3. 按一下工具列的 [部署]，建立並部署 **InputDataset**。 確認您在編輯器的標題列看到  [已成功建立資料表] 訊息。

#### <a name="create-an-output-dataset"></a>建立輸出資料表
1. 在 [Data Factory 編輯器] 中，按一下命令列上的 [...**其他]**，按一下 [新增資料集]，然後選取 [Azure Blob 儲存體]。
2. 使用下列 JSON 指令碼取代右窗格中的 JSON 指令碼：

    ```JSON
    {
        "name": "OutputDataset",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "{slice}.txt",
                "folderPath": "adftutorial/customactivityoutput/",
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

     輸出位置是 **adftutorial/customactivityoutput/** ，而輸出檔案名稱是 yyyy-MM-dd-HH.txt ，其中 yyyy-MM-dd-HH 是產生配量的年、月、日、時。 如需詳細資訊，請參閱[開發人員參考資料][adf-developer-reference]。

    為每個輸入配量產生輸出 blob/檔案。 以下是為每個配量命名輸出檔案的方式。 所有的輸出檔案會產生於一個輸出資料夾：**adftutorial\customactivityoutput**。

   | 配量 | 開始時間 | 輸出檔案 |
   |:--- |:--- |:--- |
   | 1 |2016-11-16T00:00:00 |2016-11-16-00.txt |
   | 2 |2016-11-16T01:00:00 |2016-11-16-01.txt |
   | 3 |2016-11-16T02:00:00 |2016-11-16-02.txt |
   | 4 |2016-11-16T03:00:00 |2016-11-16-03.txt |
   | 5 |2016-11-16T04:00:00 |2016-11-16-04.txt |

    請記得輸入資料夾中的所有檔案都是包含上述開始時間之配量的一部分。 處理此配量時，自訂活動會掃描每個檔案，並利用搜尋詞彙 (“Microsoft”) 的出現次數在輸出檔案中產生資料行。 如果 inputfolder 中有三個檔案，每小時的配量會有三行在輸出檔案中：2016-11-16-00.txt、2016-11-16:01:00:00.txt 等等。
3. 若要部署 **OutputDataset**，按一下命令列上的 [部署]。

### <a name="create-and-run-a-pipeline-that-uses-the-custom-activity"></a>建立並執行使用自訂活動的管線
1. 在 [Data Factory 編輯器] 中，按一下工具列上的 [...**其他]**，接著選擇命令列上的 [新增管線]。 
2. 使用下列 JSON 指令碼取代右窗格中的 JSON︰

    ```JSON
    {
      "name": "ADFTutorialPipelineCustom",
      "properties": {
        "description": "Use custom activity",
        "activities": [
          {
            "Name": "MyDotNetActivity",
            "Type": "DotNetActivity",
            "Inputs": [
              {
                "Name": "InputDataset"
              }
            ],
            "Outputs": [
              {
                "Name": "OutputDataset"
              }
            ],
            "LinkedServiceName": "AzureBatchLinkedService",
            "typeProperties": {
              "AssemblyName": "MyDotNetActivity.dll",
              "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
              "PackageLinkedService": "AzureStorageLinkedService",
              "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
              "extendedProperties": {
                "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"
              }
            },
            "Policy": {
              "Concurrency": 2,
              "ExecutionPriorityOrder": "OldestFirst",
              "Retry": 3,
              "Timeout": "00:30:00",
              "Delay": "00:00:00"
            }
          }
        ],
        "start": "2016-11-16T00:00:00Z",
        "end": "2016-11-16T05:00:00Z",
        "isPaused": false
      }
    }
    ```

    請注意下列幾點：

   * **Concurrency** 設定為 **2**，因此 Azure Batch 集區中會有 2 部 VM 以平行方式處理 2 個配量。
   * activities 區段中有一個活動，它的類型是： **DotNetActivity**。
   * **AssemblyName** 設定為 DLL 的名稱：**MyDotnetActivity.dll**。
   * **EntryPoint** 設定為 **MyDotNetActivityNS.MyDotNetActivity**。
   * **PackageLinkedService** 設定為 **AzureStorageLinkedService**，它會指向包含自訂活動 zip 檔案的 Blob 儲存體。 如果您將不同的 Azure 儲存體帳戶用於輸入/輸出檔案和自訂活動 zip 檔案，您可以建立另一個 Azure 儲存體連結服務。 本文假設您使用相同的 Azure 儲存體帳戶。
   * **PackageFile** 設定為 **customactivitycontainer/MyDotNetActivity.zip**。 其格式為：containerforthezip/nameofthezip.zip。
   * 自訂活動會採用 **InputDataset** 做為輸入和 **OutputDataset** 做為輸出。
   * 自訂活動的 linkedServiceName 屬性會指向 **AzureBatchLinkedService**，這會告知 Azure Data Factory 自訂活動必須在 Azure Batch VM 上執行。
   * **isPaused** 屬性預設為 **false**。 在此範例中，管線會立即執行，因為配量已在過去開始。 您可以將此屬性設為 true，以暫停管線，並將其設回 false，以重新啟動。
   * **start** 時間和 **end**時間相差 **5** 小時，而配量會每小時產生，因此管線會產生 5 個配量。
3. 若要部署管線，按一下命令列上的 [部署]。

### <a name="monitor-the-pipeline"></a>監視管線
1. 在 Azure 入口網站的 [Data Factory] 刀鋒視窗中，按一下 [圖表] 。

    ![[圖表] 圖格](./media/data-factory-use-custom-activities/DataFactoryBlade.png)
2. 在 [圖表] 檢視中，現在按一下 [OutputDataset]。

    ![圖表檢視](./media/data-factory-use-custom-activities/diagram.png)
3. 您應該會看到這五個輸出配量都處於就緒狀態。 如果沒有處於就緒狀態，則他們尚未產生。 

   ![輸出配量](./media/data-factory-use-custom-activities/OutputSlices.png)
4. 確認 Blob 儲存體的 **adftutorial** 容器中已產生輸出檔案。

   ![自訂活動的輸出][image-data-factory-ouput-from-custom-activity]
5. 如果您開啟輸出檔，您應該會看到類似下面的輸出：

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2016-11-16-00/file.txt.
    ```
6. 使用 [Azure 入口網站][azure-preview-portal]或 Azure PowerShell Cmdlet 來監視您的 Data Factory、管線和資料集。 在可從入口網站下載的記錄檔中 (尤其是 user-0.log)，您可以從程式碼中的 **ActivityLogger** ，或使用 Cmdlet，以查看自訂活動的訊息。

   ![從自訂活動下載記錄檔][image-data-factory-download-logs-from-custom-activity]

如需有關監視資料集和管線的詳細步驟，請參閱 [監視和管理管線](data-factory-monitor-manage-pipelines.md) 。      

## <a name="data-factory-project-in-visual-studio"></a>Visual Studio 中的 Data Factory 專案  
您可以使用 Visual Studio (而不是使用 Azure 入口網站) 來建立並發佈 Data Factory 實體。 如需使用 Visual Studio 建立並發佈 Data Factory 實體的詳細資訊，請參閱[使用 Visual Studio 建置您的第一個管線](data-factory-build-your-first-pipeline-using-vs.md)和[從 Azure Blob 複製資料到 Azure SQL](data-factory-copy-activity-tutorial-using-visual-studio.md)一文。

如果您在 Visual Studio 中建立 Data Factory 專案，請執行下列額外步驟︰
 
1. 將 Data Factory 專案新增至包含自訂活動專案的 Visual Studio 解決方案。 
2. 從 Data Factory 專案將參考新增至 .NET 活動專案。 以滑鼠右鍵按一下 Data Factory 專案，指向 [新增]，然後按一下 [參考]。 
3. 在 [新增參考] 對話方塊中，選取 **MyDotNetActivity** 專案，然後按一下 [確定]。
4. 建置及發佈解決方案。

    > [!IMPORTANT]
    > 當您發佈 Data Factory 實體時，系統會自動為您建立 zip 檔，並且上傳至 blob 容器：customactivitycontainer。 如果 blob 容器不存在，也會自動建立。  


## <a name="data-factory-and-batch-integration"></a>Data Factory 和 Batch 整合
Data Factory 服務會在 Azure Batch 中建立作業，其名為：**adf-poolname: job-xxx**。 按一下左側功能表中的 [作業]。 

![Azure Data Factory - Batch 作業](media/data-factory-use-custom-activities/data-factory-batch-jobs.png)

配量的每個活動執行都會建立一個作業。 如果有五個配量就緒可供處理，此作業中會建立五個作業。 如果 Batch 集區中有多個運算節點，可以平行執行兩個或多個配量。 如果每個計算結點的最大工作設為 > 1，您也可以在相同的計算中執行多個配量。

![Azure Data Factory - Batch 作業工作](media/data-factory-use-custom-activities/data-factory-batch-job-tasks.png)

下圖說明 Azure Data Factory 與 Batch 工作之間的關聯性。

![Data Factory & Batch](./media/data-factory-use-custom-activities/DataFactoryAndBatch.png)

## <a name="troubleshoot-failures"></a>疑難排解失敗
疑難排解包含一些基本技術：

1. 如果您看到下列錯誤，您可能正在使用經常性/非經常性 Blob 儲存體，而非使用一般用途的 Azure Blob 儲存體。 將 zip 檔案上傳至**一般用途的 Azure 儲存體帳戶**。 
 
    ```
    Error in Activity: Job encountered scheduling error. Code: BlobDownloadMiscError Category: ServerError Message: Miscellaneous error encountered while downloading one of the specified Azure Blob(s).
    ``` 
2. 如果您看到下列錯誤，請確認 CS 檔案中的類別名稱符合您在管線 JSON 中為 **EntryPoint** 屬性指定的名稱。 在逐步解說中，類別的名稱是︰MyDotNetActivity，而 JSON 中的 EntryPoint 是︰MyDotNetActivityNS.**MyDotNetActivity**。

    ```
    MyDotNetActivity assembly does not exist or doesn't implement the type Microsoft.DataFactories.Runtime.IDotNetActivity properly
    ```

   如果名稱相符，請確認所有二進位檔皆位於 zip 檔案的 **根資料夾** 中。 也就是說，當您開啟 zip 檔案，您應該會在根資料夾中看到所有檔案，而非在任何子資料夾中看到。   
3. 如果輸入配量不是設定為 [就緒] ，請確認輸入資料夾結構正確，且在於輸入資料夾中有 **file.txt**。
3. 在自訂活動的 **Execute** 方法中，使用可協助您針對問題進行疑難排解的 **IActivityLogger** 物件記錄資訊。 記錄的訊息會顯示在使用者記錄檔 (一或多個名為 user-0.log、user-1.log、user-2.log 等的檔案) 中。

   在 [OutputDataset] 刀鋒視窗中，按一下配量，以查看該配量的 [資料配量] 刀鋒視窗。 您會看到該配量的 [活動執行]。 您會看到一個為該配量執行的活動。 如果您按一下命令列中的 [執行]，您可以為相同的配量啟動另一個活動執行。

   當您按一下活動執行，您會看到包含記錄檔清單的 [活動執行詳細資料]  刀鋒視窗。 您會在 user_0.log 檔案中看到記錄的訊息。 發生錯誤時，您會看到三個活動執行，因為管線/活動 JSON 中的重試計數設定為 3。 當您按一下活動執行，您會看到您可以檢閱的記錄檔來疑難排解錯誤。

   在記錄檔清單中，按一下 [user-0.log] **IActivityLogger.Write**方法的結果。 在右窗格中的是使用 **IActivityLogger.Write** 方法的結果。 如果您無法看到所有訊息，請檢查是否有名為 user_1.log、user_2.log 等等多個記錄檔。否則，程式碼可能會在最後一個記錄的訊息之後失敗。

   此外，檢查 **system-0.log** 是否有任何系統錯誤訊息和例外狀況。
4. 在 zip 檔案中包含 **PDB** 檔案，錯誤詳細資料才會在錯誤發生時包含**呼叫堆疊**等資訊。
5. 自訂活動之 zip 檔案中的所有檔案都必須位於 **最上層** 且不包含任何子資料夾。
6. 確認 **assemblyName** (MyDotNetActivity.dll)、**entryPoint** (MyDotNetActivityNS.MyDotNetActivity)、**packageFile** (customactivitycontainer/MyDotNetActivity.zip) 和 **packageLinkedService** (應指向包含 zip 檔案的**一般用途** Azure blob 儲存體) 都設為正確的值。
7. 如果您修正錯誤，並想要重新處理配量，請以滑鼠右鍵按一下 [OutputDataset] 刀鋒視窗中的配量，然後按一下 [執行]。
8. 如果您看到下列錯誤，您使用的 Azure 儲存體封裝的版本即 > 4.3.0。 Data Factory 服務啟動程式需要 4.3 版的 WindowsAzure.Storage。 若您必須使用更新版本的 Azure 儲存體組件，請參閱 [Appdomain 隔離](#appdomain-isolation)小節，以取得因應措施。 

    ```
    Error in Activity: Unknown error in module: System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.TypeLoadException: Could not load type 'Microsoft.WindowsAzure.Storage.Blob.CloudBlob' from assembly 'Microsoft.WindowsAzure.Storage, Version=4.3.0.0, Culture=neutral, 
    ```

    如果您可以使用版本 4.3.0 的 Azure 儲存體封裝，請移除對版本 > 4.3.0 Azure.Storage 封裝的現有參考。 接著，從 NuGet Package Manager Console 執行下列命令。 

    ```PowerShell
    Install-Package WindowsAzure.Storage -Version 4.3.0
    ```

    建置專案。 從 bin\Debug 資料夾刪除版本 > 4.3.0 的 Azure.Storage 組件。 以二進位檔和 PDB 檔案建立 zip 檔案。 以 blob 容器 (customactivitycontainer) 中的 zip 檔案取代舊的 zip 檔案。 重新執行失敗的配量 (以滑鼠右鍵按一下配量並按一下 [執行])。   
8. 自訂活動不會使用來自您套件的 **app.config** 檔案。 因此，如果您的程式碼會從組態檔讀取任何連接字串，則在執行階段沒有作用。 最佳做法是使用 Azure Batch 將所有祕密存放在 **Azure KeyVault** 中、使用以憑證為基礎的服務主體來保護 **keyvault**，然後將憑證發佈至 Azure Batch 集區。 接著，.NET 自訂活動便可以在執行階段從 KeyVault 存取密碼。 這是一般解決方案，而且可以擴展至任何類型的密碼，不僅限於連接字串。

   此外，也有較簡單的因應措施 (但並非最佳做法)︰您可以建立一個帶有連接字串設定的 **Azure SQL 連結服務** 、建立一個使用該連結服務的資料集，然後將該資料集以虛擬輸入資料集的形式鏈結至自訂 .NET 活動。 接著，您便可以在自訂活動程式碼中存取連結服務的連接字串。  

## <a name="update-custom-activity"></a>更新自訂活動
如果您更新自訂活動的程式碼，請建置它，並將包含新二進位檔案的 zip 檔案上傳至 Blob 儲存體。

## <a name="appdomain-isolation"></a>Appdomain 隔離
請參閱 [跨 AppDomain 範例](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/CrossAppDomainDotNetActivitySample)，此範例示範如何建立不受 Data Factory 啟動器之組件版本 (例如：WindowsAzure.Storage v4.3.0、Newtonsoft.Json v6.0.x 等) 限制的自訂活動。

## <a name="access-extended-properties"></a>存取延伸屬性
您可以在活動 JSON 中宣告延伸屬性，如下列範例所示：

```JSON
"typeProperties": {
  "AssemblyName": "MyDotNetActivity.dll",
  "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
  "PackageLinkedService": "AzureStorageLinkedService",
  "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
  "extendedProperties": {
    "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))",
    "DataFactoryName": "CustomActivityFactory"
  }
},
```


範例中有兩個延伸屬性：**SliceStart** 和 **DataFactoryName**。 SliceStart 的值以 SliceStart 系統變數為基礎。 如需支援的系統變數清單，請參閱 [系統變數](data-factory-functions-variables.md) 。 DataFactoryName 值為硬式編碼為 CustomActivityFactory。

若要以 **Execute** 方法存取這些延伸屬性，請使用類似以下的程式碼：

```csharp
// to get extended properties (for example: SliceStart)
DotNetActivity dotNetActivity = (DotNetActivity)activity.TypeProperties;
string sliceStartString = dotNetActivity.ExtendedProperties["SliceStart"];

// to log all extended properties                               
IDictionary<string, string> extendedProperties = dotNetActivity.ExtendedProperties;
logger.Write("Logging extended properties if any...");
foreach (KeyValuePair<string, string> entry in extendedProperties)
{
    logger.Write("<key:{0}> <value:{1}>", entry.Key, entry.Value);
}
```

## <a name="auto-scaling-of-azure-batch"></a>Azure Batch 的自動調整
您也可以建立具有 **自動調整** 功能的 Azure Batch 集區。 例如，您可以用 0 專用 VM 和依據暫止工作數目自動調整的公式，建立 Azure Batch 集區。 

這裡的範例公式會有下列行為：當集區一開始建立時，它會以 1 部 VM 啟動。 $PendingTasks 計量會定義執行中 + 作用中 (已排入佇列) 狀態的工作數目。  公式會尋找過去 180 秒內的平均擱置中工作數目，並據以設定 TargetDedicated。 它會確保 TargetDedicated 一律不會超過 25 部 VM。 因此，當提交新工作時，集區會自動成長而且工作會完成，VM 會依序成為可用，而且自動調整規模功能會壓縮那些 VM。 您可以視需要調整 startingNumberOfVMs 及 maxNumberofVMs。

自動調整公式：

``` 
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
```

如需詳細資訊，請參閱 [自動調整 Azure Batch 集區中的計算節點](../../batch/batch-automatic-scaling.md) 。

如果集區使用預設的 [autoScaleEvaluationInterval](https://msdn.microsoft.com/library/azure/dn820173.aspx)，Batch 服務在執行自訂活動之前，可能需要 15-30 分鐘的時間準備 VM。  如果集區使用不同的 autoScaleEvaluationInterval，Batch 服務可能需要 autoScaleEvaluationInterval + 10 分鐘。


## <a name="create-a-custom-activity-by-using-net-sdk"></a>使用 .NET SDK 建立自訂活動
在這篇文章的逐步解說中，您會使用 Azure 入口網站建立具有管線 (使用自訂活動) 的資料處理站。 下列程式碼會示範如何改為使用 .NET SDK 建立資料處理站。 您可以在[使用 .NET API 建立具有複製活動的管線](data-factory-copy-activity-tutorial-using-dotnet-api.md)一文中找到使用 SDK 以程式設計方式建立管線的詳細資訊。 

```csharp
using System;
using System.Configuration;
using System.Collections.ObjectModel;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure;
using Microsoft.Azure.Management.DataFactories;
using Microsoft.Azure.Management.DataFactories.Models;
using Microsoft.Azure.Management.DataFactories.Common.Models;

using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Collections.Generic;

namespace DataFactoryAPITestApp
{
    class Program
    {
        static void Main(string[] args)
        {
            // create data factory management client

            // TODO: replace ADFTutorialResourceGroup with the name of your resource group.
            string resourceGroupName = "ADFTutorialResourceGroup";

            // TODO: replace APITutorialFactory with a name that is globally unique. For example: APITutorialFactory04212017
            string dataFactoryName = "APITutorialFactory";

            TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
                ConfigurationManager.AppSettings["SubscriptionId"],
                GetAuthorizationHeader().Result);

            Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

            DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);

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
                            // TODO: Replace <accountname> and <accountkey> with name and key of your Azure Storage account.
                            new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>")
                        )
                    }
                }
            );

            // create a linked service for output data store: Azure SQL Database
            Console.WriteLine("Creating Azure Batch linked service");
            client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new LinkedServiceCreateOrUpdateParameters()
                {
                    LinkedService = new LinkedService()
                    {
                        Name = "AzureBatchLinkedService",
                        Properties = new LinkedServiceProperties
                        (
                            // TODO: replace <batchaccountname> and <yourbatchaccountkey> with name and key of your Azure Batch account
                            new AzureBatchLinkedService("<batchaccountname>", "https://westus.batch.azure.com", "<yourbatchaccountkey>", "myazurebatchpool", "AzureStorageLinkedService")
                        )
                    }
                }
            );

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
                            LinkedServiceName = "AzureStorageLinkedService",
                            TypeProperties = new AzureBlobDataset()
                            {
                                FolderPath = "adftutorial/customactivityinput/",
                                Format = new TextFormat()
                            },
                            External = true,
                            Availability = new Availability()
                            {
                                Frequency = SchedulePeriod.Hour,
                                Interval = 1,
                            },

                            Policy = new Policy() { }
                        }
                    }
                });

            Console.WriteLine("Creating output dataset of type: Azure Blob");
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
                                FileName = "{slice}.txt",
                                FolderPath = "adftutorial/customactivityoutput/",
                                PartitionedBy = new List<Partition>()
                                {
                                    new Partition()
                                    {
                                        Name = "slice",
                                        Value = new DateTimePartitionValue()
                                        {
                                            Date = "SliceStart",
                                            Format = "yyyy-MM-dd-HH"
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

            Console.WriteLine("Creating a custom activity pipeline");
            DateTime PipelineActivePeriodStartTime = new DateTime(2017, 3, 9, 0, 0, 0, 0, DateTimeKind.Utc);
            DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
            string PipelineName = "ADFTutorialPipelineCustom";

            client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new PipelineCreateOrUpdateParameters()
                {
                    Pipeline = new Pipeline()
                    {
                        Name = PipelineName,
                        Properties = new PipelineProperties()
                        {
                            Description = "Use custom activity",

                            // Initial value for pipeline's active period. With this, you won't need to set slice status
                            Start = PipelineActivePeriodStartTime,
                            End = PipelineActivePeriodEndTime,
                            IsPaused = false,

                            Activities = new List<Activity>()
                            {
                                new Activity()
                                {
                                    Name = "MyDotNetActivity",
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
                                    LinkedServiceName = "AzureBatchLinkedService",
                                    TypeProperties = new DotNetActivity()
                                    {
                                        AssemblyName = "MyDotNetActivity.dll",
                                        EntryPoint = "MyDotNetActivityNS.MyDotNetActivity",
                                        PackageLinkedService = "AzureStorageLinkedService",
                                        PackageFile = "customactivitycontainer/MyDotNetActivity.zip",
                                        ExtendedProperties = new Dictionary<string, string>()
                                        {
                                            { "SliceStart", "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"}
                                        }
                                    },
                                    Policy = new ActivityPolicy()
                                    {
                                        Concurrency = 2,
                                        ExecutionPriorityOrder = "OldestFirst",
                                        Retry = 3,
                                        Timeout = new TimeSpan(0,0,30,0),
                                        Delay = new TimeSpan()
                                    }
                                }
                            }
                        }
                    }
                });
        }

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
    }
}
```

## <a name="debug-custom-activity-in-visual-studio"></a>在 Visual Studio 中針對自訂活動進行偵錯
GitHub 上的 [Azure Data Factory - 本機環境](https://github.com/gbrueckl/Azure.DataFactory.LocalEnvironment) 範例包含可讓您在 Visual Studio 內針對自訂 .NET 活動進行偵錯的工具。  


## <a name="sample-custom-activities-on-github"></a>GitHub 上的範例自訂活動
| 範例 | 自訂活動的工作內容 |
| --- | --- |
| [HTTP 資料下載程式](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/HttpDataDownloaderSample)。 |使用 Data Factory 的自訂 C# 活動，從 HTTP 端點將資料下載到 Azure Blob 儲存體。 |
| [Twitter 情感分析範例](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/TwitterAnalysisSample-CustomC%23Activity) |叫用 Azure ML 模型，執行情感分析、評分、預測等等。 |
| [執行 R 指令碼](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample)。 |在已安裝 R 的 HDInsight 叢集上執行 RScript.exe 來叫用 R 指令碼。 |
| [跨 AppDomain.NET 活動](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/CrossAppDomainDotNetActivitySample) |使用非 Data Factory 啟動器所使用的組件版本 |
| [重新處理 Azure Analysis Services 中的模型](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/AzureAnalysisServicesProcessSample) |  重新處理 Azure Analysis Services 中的模型。 |

[batch-net-library]: ../../batch/batch-dotnet-get-started.md
[batch-create-account]: ../../batch/batch-account-create-portal.md
[batch-technical-overview]: ../../batch/batch-technical-overview.md
[batch-get-started]: ../../batch/batch-dotnet-get-started.md
[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[data-factory-introduction]: data-factory-introduction.md
[azure-powershell-install]: https://github.com/Azure/azure-sdk-tools/releases


[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908
[cmdlet-reference]: http://go.microsoft.com/fwlink/?LinkId=517456

[new-azure-batch-account]: https://msdn.microsoft.com/library/mt125880.aspx
[new-azure-batch-pool]: https://msdn.microsoft.com/library/mt125936.aspx
[azure-batch-blog]: http://blogs.technet.com/b/windowshpc/archive/2014/10/28/using-azure-powershell-to-manage-azure-batch-account.aspx

[nuget-package]: http://go.microsoft.com/fwlink/?LinkId=517478
[adf-developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908
[azure-preview-portal]: https://portal.azure.com/

[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[hivewalkthrough]: data-factory-data-transformation-activities.md

[image-data-factory-ouput-from-custom-activity]: ./media/data-factory-use-custom-activities/OutputFilesFromCustomActivity.png

[image-data-factory-download-logs-from-custom-activity]: ./media/data-factory-use-custom-activities/DownloadLogsFromCustomActivity.png
