---
title: 影片仲裁 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 使用影片仲裁掃描有無可能的成人和猥褻內容。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 02/02/2018
ms.author: sajagtap
ms.openlocfilehash: 9b6240ef0883e1a523c50c9b641065deb2669d53
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226530"
---
# <a name="video-moderation"></a>影片仲裁

今日，線上檢視器會在熱門的區域社交媒體網站之間產生數十億個影片觀看次數，而且不斷在增加。 藉由套用機器學習服務以預測成人和猥褻內容，您可以降低仲裁工作的成本。

## <a name="sign-up-for-the-content-moderator-media-processor-preview"></a>註冊 Content Moderator 媒體處理器 (預覽)

### <a name="create-a-free-azure-account"></a>建立 Azure 免費帳戶

若您還沒有 Azure 帳戶，請[從這裡開始](https://azure.microsoft.com/free/)免費建立一個。

### <a name="create-an-azure-media-services-account"></a>建立 Azure 媒體服務帳戶

內容仲裁的影片功能是以 Azure 媒體服務 (AMS) 中公開預覽**媒體處理器**的形式免費提供。

請在您的 Azure 訂用帳戶中[建立 Azure 媒體服務帳戶](https://docs.microsoft.com/azure/media-services/media-services-portal-create-account)。

### <a name="get-azure-active-directory-credentials"></a>取得 Azure Active Directory 認證

   1. 閱讀 [Azure 媒體服務入口網站文章](https://docs.microsoft.com/azure/media-services/media-services-portal-get-started-with-aad)，以了解如何使用 Azure 入口網站來取得您的 Azure AD 驗證認證。
   1. 閱讀 [Azure 媒體服務 .NET 文章](https://docs.microsoft.com/azure/media-services/media-services-dotnet-get-started-with-aad)，以了解如何搭配 .NET SDK 使用您的 Azure Active Directory 認證。

   > [!NOTE]
   > 本快速入門中的範例程式碼使用這兩篇文章中所述的**服務主體驗證**方法。

一旦您取得 AMS 認證，即可透過兩個方法來嘗試內容仲裁媒體處理器。

## <a name="use-azure-media-services-explorer"></a>使用 Azure 媒體服務總管

使用互動式 [Azure 媒體服務 (AMS) 總管](https://azure.microsoft.com/blog/managing-media-workflows-with-the-new-azure-media-services-explorer-tool/)瀏覽您的 AMS 帳戶、上傳影片，並使用內容仲裁媒體處理器進行掃描。 從 GitHub [將它下載並安裝](https://github.com/Azure/Azure-Media-Services-Explorer/releases)，然後使用 AMS SDK [瀏覽原始程式碼](http://github.com/Azure/Azure-Media-Services-Explorer)以深入分析。

![Azure 媒體服務總管與內容仲裁](images/ams-explorer-content-moderator.PNG)

## <a name="net-quickstart-with-visual-studio-and-c"></a>使用 Visual Studio 和 C# 的 .NET 快速入門

1. 將一個新的 [主控台應用程式 (.NET Framework)] 專案新增到您的解決方案。

   在範例程式碼中，將專案命名為 **VideoModeration**。

1. 選取此專案作為解決方案的單一啟始專案。

### <a name="install-required-packages"></a>安裝必要的套件

安裝 [NuGet](https://www.nuget.org/) 上提供的下列 NuGet 套件。

- windowsazure.mediaservices
- windowsazure.mediaservices.extensions

### <a name="update-the-programs-using-statements"></a>更新程式的 using 陳述式

修改程式的 using 陳述式。

    using System;
    using System.Linq;
    using System.Threading.Tasks;
    using Microsoft.WindowsAzure.MediaServices.Client;
    using System.IO;
    using System.Threading;
    using Microsoft.WindowsAzure.Storage.Blob;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using System.Collections.Generic;


### <a name="initialize-application-specific-settings"></a>將應用程式特定的設定初始化

將下列靜態欄位新增至 Program.cs 中的 **Program** 類別。

    // declare constants and globals
    private static CloudMediaContext _context = null;
    private static CloudStorageAccount _StorageAccount = null;

    // Azure Media Services (AMS) associated Storage Account, Key, and the Container that has 
    // a list of Blobs to be processed.
    static string STORAGE_NAME = "YOUR AMS ASSOCIATED BLOB STORAGE NAME";
    static string STORAGE_KEY = "YOUR AMS ASSOCIATED BLOB STORAGE KEY";
    static string STORAGE_CONTAINER_NAME = "YOUR BLOB CONTAINER FOR VIDEO FILES";

    private static StorageCredentials _StorageCredentials = null;

    // Azure Media Services authentication. See the quickstart for how to get these.
    private const string AZURE_AD_TENANT_NAME = "microsoft.onmicrosoft.com";
    private const string CLIENT_ID = "YOUR CLIENT ID"
    private const string CLIENT_SECRET = "YOUR CLIENT SECRET";

    // REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".      
    private const string REST_API_ENDPOINT = "YOUR API ENDPOINT";

    // Content Moderator Media Processor Nam
    private const string MEDIA_PROCESSOR = "Azure Media Content Moderator";

    // Input and Output files in the current directory of the executable
    private const string INPUT_FILE = "VIDEO FILE NAME";
    private const string OUTPUT_FOLDER = "";

### <a name="create-a-preset-file-json"></a>建立預設檔案 (json)

在具有版本號碼的目前目錄中建立 JSON 檔案。

    private static readonly string CONTENT_MODERATOR_PRESET_FILE = "preset.json";
    //Example file content:
    //        {
    //             "version": "2.0"
    //        }
    private static readonly string CONTENT_MODERATOR_PRESET_FILE = "preset.json";

### <a name="add-the-following-code-to-the-main-method"></a>將下列程式碼新增至 main 方法

若您的影片位於 Blob 儲存體中，main 方法會先建立 Azure 媒體內容，再建立 Azure 儲存體內容。
其餘程式碼會從本機資料夾、blob 或 Azure 儲存體容器中的多個 blob 掃描影片。
您可以透過將其他程式碼行註解化，來嘗試所有選項。

    // Create Azure Media Context
    CreateMediaContext();

    // Create Storage Context
    CreateStorageContext();

    // Use a file as the input.
    // IAsset asset = CreateAssetfromFile();
    
    // -- OR ---
    
    // Or a blob as the input
    // IAsset asset = CreateAssetfromBlob((CloudBlockBlob)GetBlobsList().First());

    // Then submit the asset to Content Moderator
    // RunContentModeratorJob(asset);

    //-- OR ----

    // Just run the content moderator on all blobs in a list (from a Blob Container)
    RunContentModeratorJobOnBlobs();

### <a name="add-the-code-to-create-an-azure-media-context"></a>新增程式碼以建立 Azure 媒體內容

    /// <summary>
    /// Creates a media context from azure credentials
    /// </summary>
    static void CreateMediaContext()
    {
        // Get Azure AD credentials
        var tokenCredentials = new AzureAdTokenCredentials(AZURE_AD_TENANT_NAME,
            new AzureAdClientSymmetricKey(CLIENT_ID, CLIENT_SECRET),
            AzureEnvironments.AzureCloudEnvironment);

        // Initialize an Azure AD token
        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        // Create a media context
        _context = new CloudMediaContext(new Uri(REST_API_ENDPOINT), tokenProvider);
    }

### <a name="add-the-code-to-create-an-azure-storage-context"></a>新增程式碼以建立 Azure 儲存體內容
您可以使用從儲存體認證建立的儲存體內容，以存取您的 Blob 儲存體。

    /// <summary>
    /// Creates a storage context from the AMS associated storage name and key
    /// </summary>
    static void CreateStorageContext()
    {
        // Get a reference to the storage account associated with a Media Services account. 
        if (_StorageCredentials == null)
        {
            _StorageCredentials = new StorageCredentials(STORAGE_NAME, STORAGE_KEY);
        }
        _StorageAccount = new CloudStorageAccount(_StorageCredentials, false);
    }

### <a name="add-the-code-to-create-azure-media-assets-from-local-file-and-blob"></a>新增程式碼以從本機檔案和 blob 建立 Azure 媒體資產
內容仲裁媒體處理器會針對 Azure 媒體服務平台中的**資產**執行工作。
這些方法可從本機檔案或相關聯的 blob 建立資產。

    /// <summary>
    /// Creates an Azure Media Services Asset from the video file
    /// </summary>
    /// <returns>Asset</returns>
    static IAsset CreateAssetfromFile()
    {
        return _context.Assets.CreateFromFile(INPUT_FILE, AssetCreationOptions.None); ;
    }

    /// <summary>
    /// Creates an Azure Media Services asset from your blog storage
    /// </summary>
    /// <param name="Blob"></param>
    /// <returns>Asset</returns>
    static IAsset CreateAssetfromBlob(CloudBlockBlob Blob)
    {
        // Create asset from the FIRST blob in the list and return it
        return _context.Assets.CreateFromBlob(Blob, _StorageCredentials, AssetCreationOptions.None);
    }

### <a name="add-the-code-to-scan-a-collection-of-videos-as-blobs-within-a-container"></a>新增程式碼以掃描容器中的影片集合 (以 blob 表示)

    /// <summary>
    /// Runs the Content Moderator Job on all Blobs in a given container name
    /// </summary>
    static void RunContentModeratorJobOnBlobs()
    {
        // Get the reference to the list of Blobs. See the following method.
        var blobList = GetBlobsList();

        // Iterate over the Blob list items or work on specific ones as needed
        foreach (var sourceBlob in blobList)
        {
            // Create an Asset
            IAsset asset = _context.Assets.CreateFromBlob((CloudBlockBlob)sourceBlob,
                               _StorageCredentials, AssetCreationOptions.None);
            asset.Update();

            // Submit to Content Moderator
            RunContentModeratorJob(asset);
        }
    }

    /// <summary>
    /// Get all blobs in your container
    /// </summary>
    /// <returns></returns>
    static IEnumerable<IListBlobItem> GetBlobsList()
    {
        // Get a reference to the Container within the Storage Account
        // that has the files (blobs) for moderation
        CloudBlobClient CloudBlobClient = _StorageAccount.CreateCloudBlobClient();
        CloudBlobContainer MediaBlobContainer = CloudBlobClient.GetContainerReference(STORAGE_CONTAINER_NAME);

        // Get the reference to the list of Blobs 
        var blobList = MediaBlobContainer.ListBlobs();
        return blobList;
    }

### <a name="add-the-method-to-run-the-content-moderator-job"></a>新增方法以執行內容仲裁工作

    /// <summary>
    /// Run the Content Moderator job on the designated Asset from local file or blob storage
    /// </summary>
    /// <param name="asset"></param>
    static void RunContentModeratorJob(IAsset asset)
    {
        // Grab the presets
        string configuration = File.ReadAllText(CONTENT_MODERATOR_PRESET_FILE);

        // grab instance of Azure Media Content Moderator MP
        IMediaProcessor mp = _context.MediaProcessors.GetLatestMediaProcessorByName(MEDIA_PROCESSOR);

        // create Job with Content Moderator task
        IJob job = _context.Jobs.Create(String.Format("Content Moderator {0}",
                asset.AssetFiles.First() + "_" + Guid.NewGuid()));

        ITask contentModeratorTask = job.Tasks.AddNew("Adult and racy classifier task",
                mp, configuration,
                TaskOptions.None);
        contentModeratorTask.InputAssets.Add(asset);
        contentModeratorTask.OutputAssets.AddNew("Adult and racy classifier output",
            AssetCreationOptions.None);

        job.Submit();


        // Create progress printing and querying tasks
        Task progressPrintTask = new Task(() =>
        {
            IJob jobQuery = null;
            do
            {
                var progressContext = _context;
                jobQuery = progressContext.Jobs
                .Where(j => j.Id == job.Id)
                    .First();
                    Console.WriteLine(string.Format("{0}\t{1}",
                    DateTime.Now,
                    jobQuery.State));
                    Thread.Sleep(10000);
             }
             while (jobQuery.State != JobState.Finished &&
             jobQuery.State != JobState.Error &&
             jobQuery.State != JobState.Canceled);
        });
        progressPrintTask.Start();

        Task progressJobTask = job.GetExecutionProgressTask(
        CancellationToken.None);
        progressJobTask.Wait();

        // If job state is Error, the event handling 
        // method for job progress should log errors.  Here we check 
        // for error state and exit if needed.
        if (job.State == JobState.Error)
        {
            ErrorDetail error = job.Tasks.First().ErrorDetails.First();
            Console.WriteLine(string.Format("Error: {0}. {1}",
            error.Code,
            error.Message));
        }

        DownloadAsset(job.OutputMediaAssets.First(), OUTPUT_FOLDER);
    }

### <a name="add-a-couple-of-helper-functions"></a>新增兩個協助程式函式

這些方法會從 Azure 媒體服務資產下載內容仲裁輸出檔案 (JSON)，並協助追蹤仲裁工作的狀態，讓程式可將執行中狀態記錄至主控台。

    static void DownloadAsset(IAsset asset, string outputDirectory)
    {
        foreach (IAssetFile file in asset.AssetFiles)
        {
            file.Download(Path.Combine(outputDirectory, file.Name));
        }
    }

    // event handler for Job State
    static void StateChanged(object sender, JobStateChangedEventArgs e)
    {
        Console.WriteLine("Job state changed event:");
        Console.WriteLine("  Previous state: " + e.PreviousState);
        Console.WriteLine("  Current state: " + e.CurrentState);
        switch (e.CurrentState)
        {
            case JobState.Finished:
                Console.WriteLine();
                Console.WriteLine("Job finished.");
                break;
            case JobState.Canceling:
            case JobState.Queued:
            case JobState.Scheduled:
            case JobState.Processing:
                Console.WriteLine("Please wait...\n");
                break;
            case JobState.Canceled:
                Console.WriteLine("Job is canceled.\n");
                break;
            case JobState.Error:
                Console.WriteLine("Job failed.\n");
                break;
            default:
                break;
        }
    }

### <a name="run-the-program-and-review-the-output"></a>執行程式並檢閱輸出

內容仲裁工作完成後，分析 JSON 回應。 其中包含下列項目：

- 影片資訊摘要
- 以 "**fragments**" 表示的**畫面**
- 以 "**events**" 表示**主要畫面格**，其 **reviewRecommended" (= true 或 false)"** 旗標視**成人**和**猥褻**分數而定
- **start**、**duration**、**totalDuration** 和 **timestamp** 是以「刻度」為單位。 除以 **timescale** 以取得秒數。
 
> [!NOTE]

> - `adultScore` 代表可能有在特定情況下被視為明顯色情或成人內容存在，以及其預測分數。
> - `racyScore` 代表可能有在特定情況下被視為具性暗示或成人內容存在，以及其預測分數。
> - `adultScore` 和 `racyScore` 介於 0 到 1 之間。 分數越高，模型預測為適用該類別的可能性就越高。 此預覽版功能倚賴統計模型，而不是手動編碼的結果。 建議您使用自己的內容進行測試，以判斷每個類別如何符合您的需求。
> - `reviewRecommended` 會是 true 或 false，視內部分數閾值而定。 客戶應該評估是要使用此值，還是根據其內容原則決定自訂閾值。
>

    {
    "version": 2,
    "timescale": 90000,
    "offset": 0,
    "framerate": 50,
    "width": 1280,
    "height": 720,
    "totalDuration": 18696321,
    "fragments": [
    {
      "start": 0,
      "duration": 18000
    },
    {
      "start": 18000,
      "duration": 3600,
      "interval": 3600,
      "events": [
        [
          {
            "reviewRecommended": false,
            "adultScore": 0.00001,
            "racyScore": 0.03077,
            "index": 5,
            "timestamp": 18000,
            "shotIndex": 0
          }
        ]
      ]
    },
    {
      "start": 18386372,
      "duration": 119149,
      "interval": 119149,
      "events": [
        [
          {
            "reviewRecommended": true,
            "adultScore": 0.00000,
            "racyScore": 0.91902,
            "index": 5085,
            "timestamp": 18386372,
            "shotIndex": 62
          }
        ]
      ]
    }
    ]
    }

## <a name="next-steps"></a>後續步驟

了解如何從仲裁輸出產生[影片檢閱](video-reviews-quickstart-dotnet.md)。

將[文字記錄仲裁](video-transcript-moderation-review-tutorial-dotnet.md)新增至您的影片檢閱。

查看有關如何建置[完整影片和文字記錄仲裁解決方案](video-transcript-moderation-review-tutorial-dotnet.md)的詳細教學課程。

[下載 Visual Studio 解決方案](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)，以取得這個及其他適用於 .NET 的內容仲裁快速入門。
