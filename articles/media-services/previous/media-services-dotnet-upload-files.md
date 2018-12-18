---
title: 使用 .NET 將檔案上傳至媒體服務帳戶 | Microsoft Docs
description: 了解如何建立並上傳資產，以將媒體內容移至媒體服務中。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: c9c86380-9395-4db8-acea-507c52066f73
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/21/2018
ms.author: juliako
ms.openlocfilehash: 9edfa8ea0c9e469d09cef7ddbd1c7edda4484b47
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42444624"
---
# <a name="upload-files-into-a-media-services-account-using-net"></a>使用 .NET 將檔案上傳至媒體服務帳戶
> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-upload-files.md)
> * [REST](media-services-rest-upload-files.md)
> * [入口網站](media-services-portal-upload-files.md)
> 
> 

在媒體服務中，您可以將數位檔案上傳 (或內嵌) 到資產。 **資產**實體可以包含視訊、音訊、影像、縮圖集合、文字播放軌及隱藏式輔助字幕檔案 (以及這些檔案的相關中繼資料)。上傳檔案之後，您的內容會安全地儲存在雲端，以進一步進行處理和串流處理。

資產中的檔案稱為 **資產檔案**。 **AssetFile** 執行個體和實際媒體檔是兩個不同的物件。 AssetFile 執行個體包含媒體檔案的相關中繼資料，而媒體檔案包含實際的媒體內容。

> [!NOTE]
> 您必須考量下列事項：
> 
> * 建置串流內容的 URL (例如， http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters) 時，媒體服務會使用 IAssetFile.Name 屬性的值。基於這個理由，不允許 percent-encoding。 **Name** 屬性的值不能有下列任何[百分比編碼保留字元](http://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)：!*'();:@&=+$,/?%#[]"。 而且，副檔名只能有一個 '.'。
> * 名稱長度不應超過 260 個字元。
> * 對於在媒體服務處理檔案，支援的檔案大小有上限。 請參閱[這篇](media-services-quotas-and-limitations.md)文章，以取得有關檔案大小限制的詳細資料。
> * 對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限，例如，要長時間維持就地 (非上載原則) 的定位器原則，您應該使用相同的原則識別碼。 如需詳細資訊，請參閱[本篇文章](media-services-dotnet-manage-entities.md#limit-access-policies)。
> 

建立資產時，您可以指定下列加密選項：

* **None** - 不使用加密。 這是預設值。 使用此選項時，您的內容在傳輸或儲存體中靜止時不會受到保護。
  如果您計劃使用漸進式下載傳遞 MP4，請使用此選項： 
* **CommonEncryption** - 如果您上傳的內容已經受到一般加密或 PlayReady DRM (例如，受到 PlayReady DRM 保護的 Smooth Streaming) 的加密保護，請使用此選項。
* **EnvelopeEncrypted** - 如果您上傳以 AES 加密的 HLS，請使用此選項。 請注意，檔案必須已由 Transform Manager 編碼和加密。
* **StorageEncrypted** - 使用 AES-256 位元加密對您的內容進行本機加密，接著上傳到已靜止加密儲存的 Azure 儲存體。 以儲存體加密保護的資產會自動解除加密並在編碼前放置在加密的檔案系統中，並且會在上傳為新輸出資產之前選擇性地重新編碼。 儲存體加密的主要使用案例是讓您可以使用強式加密來保護磁碟中靜止的高品質輸入媒體檔。
  
    媒體服務可為您的資產提供磁碟上的儲存體加密，而不是在線上加密，例如數位版權管理員 (DRM)。
  
    如果您的資產是儲存體加密，必須設定資產傳遞原則。 如需詳細資訊，請參閱[設定資產傳遞原則](media-services-dotnet-configure-asset-delivery-policy.md)。

如果您指定使用 **CommonEncrypted** 選項或 **EnvelopeEncypted** 選項加密資產，則需要建立資產與 **ContentKey** 的關聯。 如需詳細資訊，請參閱 [如何建立 ContentKey](media-services-dotnet-create-contentkey.md)。 

如果您指定使用 **StorageEncrypted** 選項來加密資產，則 Media Services SDK for .NET 會建立資產的 **StorateEncrypted** 和 **ContentKey**。

本文顯示如何使用 Media Services .NET SDK 以及 Media Services .NET SDK 擴充功能，以將檔案上傳到媒體服務資產。

## <a name="upload-a-single-file-with-media-services-net-sdk"></a>使用媒體服務 .NET SDK 上傳單一檔案
以下程式碼使用 .NET 來上傳一個檔案。 AccessPolicy 與定位器會由所上傳的函式建立並終結。 

```csharp
        static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
        {
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, assetCreationOptions);

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return inputAsset;
        }
```


## <a name="upload-multiple-files-with-media-services-net-sdk"></a>使用媒體服務 .NET SDK 上傳多個檔案
下列程式碼將說明如何建立資產並上傳多個檔案。

此程式碼會執行下列動作：

* 使用上一個步驟中所定義的 CreateEmptyAsset 方法，來建立空白資產。
* 建立 **AccessPolicy** 執行個體，以定義存取資產所需的權限和規定期間。
* 建立可用來存取資產的 **Locator** 執行個體。
* 建立 **BlobTransferClient** 執行個體。 此類型代表在 Azure Blob 上作業的用戶端。 在此範例中，用戶端會監視上傳進度。 
* 逐一列舉所指定目錄中的檔案，並建立每個檔案的 **AssetFile** 執行個體。
* 使用 **UploadAsync** 方法，將檔案上傳到媒體服務。 

> [!NOTE]
> 使用 UploadAsync 方法確保未封鎖呼叫，並且平行上傳檔案。
> 
> 

```csharp
        static public IAsset CreateAssetAndUploadMultipleFiles(AssetCreationOptions assetCreationOptions, string folderPath)
        {
            var assetName = "UploadMultipleFiles_" + DateTime.UtcNow.ToString();

            IAsset asset = _context.Assets.Create(assetName, assetCreationOptions);

            var accessPolicy = _context.AccessPolicies.Create(assetName, TimeSpan.FromDays(30),
                                                                AccessPermissions.Write | AccessPermissions.List);

            var locator = _context.Locators.CreateLocator(LocatorType.Sas, asset, accessPolicy);

            var blobTransferClient = new BlobTransferClient();
            blobTransferClient.NumberOfConcurrentTransfers = 20;
            blobTransferClient.ParallelTransferThreadCount = 20;

            blobTransferClient.TransferProgressChanged += blobTransferClient_TransferProgressChanged;

            var filePaths = Directory.EnumerateFiles(folderPath);

            Console.WriteLine("There are {0} files in {1}", filePaths.Count(), folderPath);

            if (!filePaths.Any())
            {
                throw new FileNotFoundException(String.Format("No files in directory, check folderPath: {0}", folderPath));
            }

            var uploadTasks = new List<Task>();
            foreach (var filePath in filePaths)
            {
                var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
                Console.WriteLine("Created assetFile {0}", assetFile.Name);

                // It is recommended to validate AssetFiles before upload. 
                Console.WriteLine("Start uploading of {0}", assetFile.Name);
                uploadTasks.Add(assetFile.UploadAsync(filePath, blobTransferClient, locator, CancellationToken.None));
            }

            Task.WaitAll(uploadTasks.ToArray());
            Console.WriteLine("Done uploading the files");

            blobTransferClient.TransferProgressChanged -= blobTransferClient_TransferProgressChanged;

            locator.Delete();
            accessPolicy.Delete();

            return asset;
        }

    static void  blobTransferClient_TransferProgressChanged(object sender, BlobTransferProgressChangedEventArgs e)
    {
        if (e.ProgressPercentage > 4) // Avoid startup jitter, as the upload tasks are added.
        {
            Console.WriteLine("{0}% upload competed for {1}.", e.ProgressPercentage, e.LocalFile);
        }
    }
```


上傳大量資產時，請考慮下列項目：

* 為每個執行緒建立新的 **CloudMediaContext** 物件。 **CloudMediaContext** 類別不具備執行緒安全。
* 將 NumberOfConcurrentTransfers 從預設值 2 增加為較高的值 (例如 5)。 設定此屬性會影響所有 **CloudMediaContext**執行個體。 
* 將 ParallelTransferThreadCount 保持為預設值 10。

## <a id="ingest_in_bulk"></a>使用媒體服務 .NET SDK 大量擷取資產
上傳大型資產檔案可能會在建立資產期間造成瓶頸。 大量內嵌資產或「大量內嵌」包含透過上傳程序來解除結合資產建立。 若要使用大量內嵌方式，請建立可描述資產及其相關檔案的資訊清單 (IngestManifest)。 然後使用您選擇的上傳方法，將相關的檔案上傳至資訊清單的 Blob 容器。 Microsoft Azure 媒體服務會監看與資訊清單相關聯的 Blob 容器。 將檔案上傳至 Blob 容器之後，Microsoft Azure 媒體服務會根據資訊清單 (IngestManifestAsset) 中的資產組態來完成資產建立。

若要建立新的 IngestManifest，請呼叫 CloudMediaContext 上 IngestManifests 集合所公開的 Create 方法。 此方法使用您提供的資訊清單名稱來建立新的 IngestManifest。

```csharp
    IIngestManifest manifest = context.IngestManifests.Create(name);
```

建立會與大量 IngestManifest 相關聯的資產。 對資產設定想要的加密選項，以進行大量內嵌。

```csharp
    // Create the assets that will be associated with this bulk ingest manifest
    IAsset destAsset1 = _context.Assets.Create(name + "_asset_1", AssetCreationOptions.None);
    IAsset destAsset2 = _context.Assets.Create(name + "_asset_2", AssetCreationOptions.None);
```

IngestManifestAsset 會建立資產與大量 IngestManifest 的關聯，以進行大量內嵌。 它也會建立構成每個資產之 AssetFile 的關聯。 若要建立 IngestManifestAsset，請在伺服器內容上使用 Create 方法。

下列範例示範如何新增兩個新的 IngestManifestAsset，以建立先前建立之兩個資產與大量內嵌資訊清單的關聯。 每個 IngestManifestAsset 也會關聯在大量內嵌期間針對每個資產所上傳的一組檔案。  

```csharp
    string filename1 = _singleInputMp4Path;
    string filename2 = _primaryFilePath;
    string filename3 = _singleInputFilePath;

    IIngestManifestAsset bulkAsset1 =  manifest.IngestManifestAssets.Create(destAsset1, new[] { filename1 });
    IIngestManifestAsset bulkAsset2 =  manifest.IngestManifestAssets.Create(destAsset2, new[] { filename2, filename3 });
```

您可以使用任何高速用戶端應用程式，而高速用戶端應用程式可以將資產檔案上傳至 IngestManifest 之 **IIngestManifest.BlobStorageUriForUpload** 屬性所提供的 Blob 儲存體容器 URI。 

下列程式碼說明如何使用 .NET SDK 上傳資產檔案。

```csharp
    static void UploadBlobFile(string containerName, string filename)
    {
        Task copytask = new Task(() =>
        {
            var storageaccount = new CloudStorageAccount(new StorageCredentials(_storageAccountName, _storageAccountKey), true);
            CloudBlobClient blobClient = storageaccount.CreateCloudBlobClient();
            CloudBlobContainer blobContainer = blobClient.GetContainerReference(containerName);

            string[] splitfilename = filename.Split('\\');
            var blob = blobContainer.GetBlockBlobReference(splitfilename[splitfilename.Length - 1]);

            using (var stream = System.IO.File.OpenRead(filename))
                blob.UploadFromStream(stream);

            lock (consoleWriteLock)
            {
                Console.WriteLine("Upload for {0} completed.", filename);
            }
        });

        copytask.Start();
    }
```

下列程式碼範例顯示上傳本文中所用範例之資產檔案的程式碼：

```csharp
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename1);
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename2);
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename3);
```

輪詢 **IngestManifest** 的 Statistics 屬性，即可判斷與 **IngestManifest** 相關聯之所有資產的大量內嵌進度。 若要更新進度資訊，每次輪詢 Statistics 屬性時，都必須使用新的 **CloudMediaContext** 。

下列範例示範如何依 **Id**輪詢 IngestManifest。

```csharp
    static void MonitorBulkManifest(string manifestID)
    {
       bool bContinue = true;
       while (bContinue)
       {
          CloudMediaContext context = GetContext();
          IIngestManifest manifest = context.IngestManifests.Where(m => m.Id == manifestID).FirstOrDefault();

          if (manifest != null)
          {
             lock(consoleWriteLock)
             {
                Console.WriteLine("\nWaiting on all file uploads.");
                Console.WriteLine("PendingFilesCount  : {0}", manifest.Statistics.PendingFilesCount);
                Console.WriteLine("FinishedFilesCount : {0}", manifest.Statistics.FinishedFilesCount);
                Console.WriteLine("{0}% complete.\n", (float)manifest.Statistics.FinishedFilesCount / (float)(manifest.Statistics.FinishedFilesCount + manifest.Statistics.PendingFilesCount) * 100);

                if (manifest.Statistics.PendingFilesCount == 0)
                {
                   Console.WriteLine("Completed\n");
                   bContinue = false;
                }
             }

             if (manifest.Statistics.FinishedFilesCount < manifest.Statistics.PendingFilesCount)
                Thread.Sleep(60000);
          }
          else // Manifest is null
             bContinue = false;
       }
    }
```


## <a name="upload-files-using-net-sdk-extensions"></a>使用 .NET SDK 延伸模組上傳檔案
下列範例顯示如何使用 .NET SDK 擴充功能上傳單一檔案。 在此情況下，會使用 **CreateFromFile** 方法，但也會提供非同步版本 (**CreateFromFileAsync**)。 **CreateFromFile** 方法可讓您指定檔案名稱、加密選項和回呼，以報告檔案的上傳進度。

```csharp
    static public IAsset UploadFile(string fileName, AssetCreationOptions options)
    {
        IAsset inputAsset = _context.Assets.CreateFromFile(
            fileName,
            options,
            (af, p) =>
            {
                Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Asset {0} created.", inputAsset.Id);

        return inputAsset;
    }
```

下列範例會呼叫 UploadFile 函數，並指定儲存體加密做為資產建立選項。  

```csharp
    var asset = UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.StorageEncrypted);
```

## <a name="next-steps"></a>後續步驟

您現在可以將上傳的資產編碼。 如需詳細資訊，請參閱 [為資產編碼](media-services-portal-encode.md)。

您也可以使用 Azure Functions，以根據在所設定容器到達的檔案來觸發編碼作業。 如需詳細資訊，請參閱[此範例](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ )。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-step"></a>後續步驟
您已將資產上傳至媒體服務，現在請移至 [如何取得媒體處理器][How to Get a Media Processor]一文。

[How to Get a Media Processor]: media-services-get-media-processor.md

