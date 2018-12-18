---
title: 使用 .NET 傳遞點播內容入門 | Microsoft Docs
description: 本教學課程會逐步完成使用 .NET 實作含 Azure 媒體服務的點播內容傳遞應用程式。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 388b8928-9aa9-46b1-b60a-a918da75bd7b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: hero-article
ms.date: 12/10/2017
ms.author: juliako
ms.openlocfilehash: 12a6f731dfb1c106c28d18caa95710751736629c
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39434579"
---
# <a name="get-started-with-delivering-content-on-demand-using-net-sdk"></a>使用 .NET SDK 傳遞點播內容入門
[!INCLUDE [media-services-selector-get-started](../../../includes/media-services-selector-get-started.md)]

本教學課程會逐步引導您使用 Azure 媒體服務 .NET SDK 實作含 Azure 媒體服務 (AMS) 應用程式的基本點播視訊 (VoD) 內容傳遞服務。

## <a name="prerequisites"></a>必要條件

需要有下列項目，才能完成教學課程：

* 一個 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* 媒體服務帳戶。 若要建立媒體服務帳戶，請參閱[如何建立媒體服務帳戶](media-services-portal-create-account.md)。
* .NET Framework 4.0 或更新版本。
* Visual Studio。

本教學課程內容包括以下工作：

1. 啟動串流端點 (使用 Azure 入口網站)。
2. 建立和設定 Visual Studio 專案。
3. 連線到媒體服務帳戶。
2. 上傳視訊檔案。
3. 將來源檔案編碼為一組自適性 MP4 檔案。
4. 發佈資產並取得串流和漸進式下載 URL。  
5. 播放您的內容。

## <a name="overview"></a>概觀
本教學課程會逐步完成使用 Azure Media Services (AMS) SDK for .NET 實作點播視訊 (VoD) 內容傳遞應用程式。

教學課程中介紹基本的媒體服務工作流程，以及媒體服務開發最常用的程式設計物件和必要工作。 完成本教學課程時，您將能夠串流或漸進式下載您已上傳、編碼和下載的範例媒體檔案。

### <a name="ams-model"></a>AMS 模型

下列影像顯示針對媒體服務 OData 模型開發 VoD 應用程式時一些最常用的物件。

按一下影像可以完整大小檢視。  

<a href="./media/media-services-dotnet-get-started/media-services-overview-object-model.png" target="_blank"><img src="./media/media-services-dotnet-get-started/media-services-overview-object-model-small.png"></a> 

您可以[在此](https://media.windows.net/API/$metadata?api-version=2.15)檢視整個模型。  

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>使用 Azure 入口網站開始串流端點

使用 Azure 媒體服務時，其中一個最常見的案例是透過自適性串流提供影片。 媒體服務提供動態封裝，這讓您以媒體服務即時支援的串流格式 (MPEG DASH、HLS、Smooth Streaming) 提供自適性 MP4 編碼內容，而不必儲存這些串流格式個別的預先封裝版本。

>[!NOTE]
>建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。

若要啟動串流端點，請執行下列作業︰

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 [設定] 視窗中，按一下 [串流端點]。
3. 按一下預設串流端點。

    [預設串流端點詳細資料] 視窗隨即出現。

4. 按一下 [啟動] 圖示。
5. 按一下 [儲存] 按鈕以儲存您的變更。

## <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

1. 設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 
2. 建立新的資料夾 (資料夾可在本機磁碟機上任意處)，並複製您想要編碼和串流處理或漸進式下載的 .mp4 檔案。 在此範例中，使用 "C:\VideoFiles" 路徑。

## <a name="connect-to-the-media-services-account"></a>連線到媒體服務帳戶

搭配使用媒體服務與 .NET 時，您必須將 **CloudMediaContext** 類別用於大部分的媒體服務程式設計工作：連線到媒體服務帳戶；建立、更新、存取和刪除下列物件：資產、資產檔案、工作、存取原則、定位器等。

以下列程式碼覆寫預設 Program 類別：此程式碼示範如何讀取 App.config 檔案中的連線值，以及如何建立 **CloudMediaContext** 物件來連線到媒體服務。 如需詳細資訊，請參閱[連線至媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。

務必更新您的媒體檔案的檔案名稱和路徑。

**Main** 函數會呼叫未來將在此區段中定義的方法。

> [!NOTE]
> 在您新增本文稍後定義知所有函式的定義以前，將會出現編譯錯誤。

```csharp
    class Program
    {
        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
        try
        {
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Add calls to methods defined in this section.
            // Make sure to update the file name and path to where you have your media file.
            IAsset inputAsset =
            UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.None);

            IAsset encodedAsset =
            EncodeToAdaptiveBitrateMP4s(inputAsset, AssetCreationOptions.None);

            PublishAssetGetURLs(encodedAsset);
        }
        catch (Exception exception)
        {
            // Parse the XML error message in the Media Services response and create a new
            // exception with its content.
            exception = MediaServicesExceptionParser.Parse(exception);

            Console.Error.WriteLine(exception.Message);
        }
        finally
        {
            Console.ReadLine();
        }
        }
```

## <a name="create-a-new-asset-and-upload-a-video-file"></a>建立新資產並上傳視訊檔案

在媒體服務中，您可以將數位檔案上傳 (或內嵌) 到資產。 **資產**實體可以包含視訊、音訊、影像、縮圖集合、文字播放軌及隱藏式輔助字幕檔案 (以及這些檔案的相關中繼資料)。上傳檔案之後，您的內容會安全地儲存在雲端，以進一步進行處理和串流處理。 資產中的檔案稱為 **資產檔案**。

下面所定義的 **UploadFile** 方法會呼叫 **CreateFromFile** (定義於 .NET SDK 延伸模組中)。 **CreateFromFile** 會建立要在其中上傳指定來源檔案的新資產。

**CreateFromFile** 方法會採用 **AssetCreationOptions**，以讓您指定下列其中一個資產建立選項：

* **None** - 不使用加密。 這是預設值。 請注意，使用此選項時，您的內容在傳輸或儲存體中靜止時不會受到保護。
  如果您計劃使用漸進式下載傳遞 MP4，請使用此選項。
* **StorageEncrypted** - 請使用此選項來利用進階加密標準 (AES) 256 位元加密，對您的純文字內容進行本機加密，然後會將它上傳到已靜止加密儲存的 Azure 儲存體。 以儲存體加密保護的資產會自動解除加密並在編碼前放置在加密的檔案系統中，並且會在上傳為新輸出資產之前選擇性地重新編碼。 儲存體加密的主要使用案例是讓您可以使用強式加密來保護磁碟中靜止的高品質輸入媒體檔。
* **CommonEncryptionProtected** - 如果您要上傳已經使用一般加密或 PlayReady DRM (例如使用 PlayReady DRM 保護的 Smooth Streaming) 加密及保護的內容，請使用這個選項。
* **EnvelopeEncryptionProtected** – 如果您要上傳使用 AES 加密的 HLS，請使用這個選項。 請注意，檔案必須已由 Transform Manager 編碼和加密。

**CreateFromFile** 方法也可讓您指定回呼，以報告檔案的上傳進度。

在下列範例中，我們將資產選項指定為 **None** 。

將下列方法新增至 Program 類別。

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

## <a name="encode-the-source-file-into-a-set-of-adaptive-bitrate-mp4-files"></a>將來源檔案編碼為一組調適性位元速率 MP4 檔案
將資產內嵌到媒體服務之後，可以先將媒體編碼、轉碼多工處理、加上浮水印等，再傳遞給用戶端。 這些活動會針對多個背景角色執行個體排定和執行，以確保高效能與可用性。 這些活動稱為作業，每個作業包含對資產檔案執行實際工作的不可部分完成的工作。

如稍早所提及，使用 Azure 媒體服務時，其中一個最常見的案例是將調適性位元速率串流傳遞給用戶端。 媒體服務可以以下列其中一種格式動態封裝一組可調位元速率 MP4 檔案：HTTP 即時串流 (HLS)、Smooth Streaming 和 MPEG DASH。

若要利用動態封裝功能，您必須將您的夾層 (來源) 檔編碼或轉換為一組調適性位元速率 MP4 檔案或調適性位元速率 Smooth Streaming 檔案。  

下列程式碼顯示如何提交編碼工作。 此工作包含一項作業，指定使用 **媒體編碼器標準**，將夾層檔轉碼為一組調適性位元速率 MP4。 此程式碼會提交工作，並等到工作完成。

工作完成之後，就可以串流處理資產，或漸進式下載轉碼後所建立的 MP4 檔案。

將下列方法新增至 Program 類別。

```csharp
    static public IAsset EncodeToAdaptiveBitrateMP4s(IAsset asset, AssetCreationOptions options)
    {

        // Prepare a job with a single task to transcode the specified asset
        // into a multi-bitrate asset.

        IJob job = _context.Jobs.CreateWithSingleTask(
            "Media Encoder Standard",
            "Adaptive Streaming",
            asset,
            "Adaptive Bitrate MP4",
            options);

        Console.WriteLine("Submitting transcoding job...");


        // Submit the job and wait until it is completed.
        job.Submit();

        job = job.StartExecutionProgressTask(
            j =>
            {
                Console.WriteLine("Job state: {0}", j.State);
                Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
            },
            CancellationToken.None).Result;

        Console.WriteLine("Transcoding job finished.");

        IAsset outputAsset = job.OutputMediaAssets[0];

        return outputAsset;
    }
```

## <a name="publish-the-asset-and-get-urls-for-streaming-and-progressive-download"></a>發佈資產並取得串流和漸進式下載 URL。

若要串流處理或下載資產，您必須先建立定位器來「發佈」它。 定位器提供對於資產中包含之檔案的存取。 媒體服務支援兩種類型的定位器：OnDemandOrigin 定位器，用於串流媒體 (例如，MPEG DASH、HLS 或 Smooth Streaming) 和存取簽章 (SAS) 定位器，用來下載媒體檔案。

### <a name="some-details-about-url-formats"></a>URL 格式的相關詳細資料

建立定位器之後，您便可以建立用來串流或下載檔案的 URL。 本教學課程中的範例會輸出您可貼在適當瀏覽器中的 URL。 本節只會提供不同格式外觀的簡短範例。

#### <a name="a-streaming-url-for-mpeg-dash-has-the-following-format"></a>MPEG DASH 的串流 URL 具有下列格式：

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest **(format=mpd-time-csf)**

#### <a name="a-streaming-url-for-hls-has-the-following-format"></a>HLS 的串流 URL 具有下列格式：

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest **(format=m3u8-aapl)**

#### <a name="a-streaming-url-for-smooth-streaming-has-the-following-format"></a>Smooth Streaming 的串流 URL 具有下列格式：

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest


#### <a name="a-sas-url-used-to-download-files-has-the-following-format"></a>用來下載檔案的 SAS URL 具有下列格式：

{blob 容器名稱}/{資產名稱}/{檔案名稱}/{SAS 簽章}

Media Services .NET SDK 延伸模組提供便利的協助程式方法，來傳回所發佈資產的格式化 URL。

下列程式碼使用 .NET SDK 延伸模組來建立定位器、取得串流，以及漸進式下載 URL。 此程式碼也會顯示如何將檔案下載到本機資料夾。

將下列方法新增至 Program 類別。

```csharp
    static public void PublishAssetGetURLs(IAsset asset)
    {
        // Publish the output asset by creating an Origin locator for adaptive streaming,
        // and a SAS locator for progressive download.

        _context.Locators.Create(
            LocatorType.OnDemandOrigin,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));

        _context.Locators.Create(
            LocatorType.Sas,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));


        IEnumerable<IAssetFile> mp4AssetFiles = asset
                .AssetFiles
                .ToList()
                .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Get the Smooth Streaming, HLS and MPEG-DASH URLs for adaptive streaming,
        // and the Progressive Download URL.
        Uri smoothStreamingUri = asset.GetSmoothStreamingUri();
        Uri hlsUri = asset.GetHlsUri();
        Uri mpegDashUri = asset.GetMpegDashUri();

        // Get the URls for progressive download for each MP4 file that was generated as a result
        // of encoding.
        List<Uri> mp4ProgressiveDownloadUris = mp4AssetFiles.Select(af => af.GetSasUri()).ToList();


        // Display  the streaming URLs.
        Console.WriteLine("Use the following URLs for adaptive streaming: ");
        Console.WriteLine(smoothStreamingUri);
        Console.WriteLine(hlsUri);
        Console.WriteLine(mpegDashUri);
        Console.WriteLine();

        // Display the URLs for progressive download.
        Console.WriteLine("Use the following URLs for progressive download.");
        mp4ProgressiveDownloadUris.ForEach(uri => Console.WriteLine(uri + "\n"));
        Console.WriteLine();

        // Download the output asset to a local folder.
        string outputFolder = "job-output";
        if (!Directory.Exists(outputFolder))
        {
            Directory.CreateDirectory(outputFolder);
        }

        Console.WriteLine();
        Console.WriteLine("Downloading output asset files to a local folder...");
        asset.DownloadToFolder(
            outputFolder,
            (af, p) =>
            {
                Console.WriteLine("Downloading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Output asset files available at '{0}'.", Path.GetFullPath(outputFolder));
    }
```

## <a name="test-by-playing-your-content"></a>播放您的內容以進行測試

執行上一節中所定義的程式之後，主控台視窗中會顯示與下面類似的 URL。

調適性串流 URL：

Smooth Streaming

    http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest

HLS

    http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)

MPEG DASH

    http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)

漸進式下載 URL (音訊和視訊)。

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z


若要串流處理視訊，請將您的 URL 貼在 [Azure 媒體服務播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)的 [URL] 文字方塊中。

若要測試漸進式下載，請將 URL 貼入瀏覽器 (例如，Internet Explorer、Chrome 或 Safari)。

如需詳細資訊，請參閱下列主題：

- [使用現有播放器來播放您的內容](media-services-playback-content-with-existing-players.md)
- [開發視訊播放程式應用程式](media-services-develop-video-players.md)
- [透過 DASH.js 將 MPEG-DASH 彈性資料流視訊嵌入到 HTML5 應用程式](media-services-embed-mpeg-dash-in-html5.md)

## <a name="download-sample"></a>下載範例
下列程式碼範例包含您在本教學課程中建立的程式碼︰[範例](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/)。

## <a name="next-steps"></a>後續步驟

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]



<!-- Anchors. -->


<!-- URLs. -->
[Web Platform Installer]: http://go.microsoft.com/fwlink/?linkid=255386
[Portal]: http://portal.azure.com/
