---
title: 教學課程：在 .NET 中仲裁影片及文字記錄 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 如何使用 Content Moderator 在 .NET 中審核影片及文字記錄。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: tutorial
ms.date: 1/27/2018
ms.author: sajagtap
ms.openlocfilehash: 12f03352373bebecb74b9dd8d31470ac337f5e71
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227566"
---
# <a name="tutorial-video-and-transcript-moderation"></a>教學課程：影片及文字記錄仲裁

Content Moderator 的影片 API 可讓您審核影片，以及在人工審核工具中建立影片檢閱。 

這篇詳細的教學課程可協助您了解如何使用機器輔助審核和建立人機互動 (human-in-the-loop) 檢閱，來建置完整的影片及文字記錄審核解決方案。

下載本教學課程適用的 [C# 主控台應用程式](https://github.com/MicrosoftContentModerator/VideoReviewConsoleApp)。 主控台應用程式會使用 SDK 和相關的套件來執行下列工作：

- 壓縮輸入影片以加快處理速度
- 審核影片以取得包含見解的截圖和畫面
- 使用畫面時間戳記來建立縮圖 (影像)
- 提交時間戳記和縮圖來建立影片檢閱
- 使用媒體索引器 API 將影片語音轉換成文字 (文字記錄)
- 使用文字審核服務來審核文字記錄
- 將審核過的文字記錄新增至影片檢閱

## <a name="sample-program-outputs"></a>程式輸出範例

進行下一步之前，讓我們來看看程式的以下輸出範例：

- [主控台輸出](#program-output)
- [影片檢閱](#video-review-default-view)
- [文字記錄檢閱](#video-review-transcript-view)

## <a name="prerequisites"></a>必要條件

1. 註冊 [Content Moderator 審核工具](https://contentmoderator.cognitive.microsoft.com/)網站，並[建立自訂標記](Review-Tool-User-Guide/tags.md)，C# 主控台應用程式會從程式碼內指派這些標記。 自訂標記如下列畫面所示。

  ![影片審核自訂標記](images/video-tutorial-custom-tags.png)

1. 若要執行範例應用程式，您需要 Azure 帳戶和 Azure 媒體服務帳戶。 此外，您必須有 Content Moderator 個人預覽版的存取權。 最後，您需要 Azure Active Directory 驗證的認證。 如需取得此資訊的詳細資訊，請參閱[影片審核 API 快速入門](video-moderation-api.md)。

1. 編輯 `App.config` 檔案，並新增 Active Directory 租用戶名稱、服務端點和 `#####` 指示的訂用帳戶金鑰。 您需要下列資訊：

|Key|說明|
|-|-|
|`AzureMediaServiceRestApiEndpoint`|Azure 媒體服務 (AMS) API 的端點|
|`ClientSecret`|Azure 媒體服務的訂用帳戶金鑰|
|`ClientId`|Azure 媒體服務的用戶端識別碼|
|`AzureAdTenantName`|代表您組織的 Active Directory 租用戶名稱|
|`ContentModeratorReviewApiSubscriptionKey`|Content Moderator 檢閱 API 的訂用帳戶金鑰|
|`ContentModeratorApiEndpoint`|Content Moderator API 的端點|
|`ContentModeratorTeamId`|Content Moderator 小組 API|

## <a name="getting-started"></a>開始使用

`Program.cs` 中的 `Program` 類別是影片審核應用程式的主要進入點。

### <a name="methods-of-class-program"></a>類別程式的方法

|方法|說明|
|-|-|
|`Main`|剖析命令列、收集使用者輸入及開始處理。|
|`ProcessVideo`|壓縮、上傳、審核及建立影片檢閱。|
|`CreateVideoStreamingRequest`|建立上傳影片用的串流|
|`GetUserInputs`|收集使用者輸入；沒有命令列選項時會使用此項目|
|`Initialize`|初始化審核程序所需的物件|

### <a name="the-main-method"></a>Main 方法

`Main()` 是執行開始的位置，因此可以從這裡開始了解審核程序。

    static void Main(string[] args)
    {
        if (args.Length == 0)
        {
            string videoPath = string.Empty;
            GetUserInputs(out videoPath);
            Initialize();
            AmsConfigurations.logFilePath = Path.Combine(Path.GetDirectoryName(videoPath), "log.txt");
            try
            {
                ProcessVideo(videoPath).Wait();
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
        else
        {
            DirectoryInfo directoryInfo = new DirectoryInfo(args[0]);
            if (args.Length == 2)
                bool.TryParse(args[1], out generateVtt);
            Initialize();
            AmsConfigurations.logFilePath = Path.Combine(args[0], "log.txt");
            var files = directoryInfo.GetFiles("*.mp4", SearchOption.AllDirectories);
            foreach (var file in files)
            {
                try
                {
                    ProcessVideo(file.FullName).Wait();
                }
                catch (Exception ex)
                {
                    Console.WriteLine(ex.Message);
                }
            }
        }
    }

`Main()` 會處理下列命令列引數：

- 目錄路徑，目錄中包含要提交以進行審核的 MPEG-4 影片檔案。 此目錄和其子目錄中的所有 `*.mp4` 檔案都會提交以進行審核。
- (選擇性) 布林值 (true/false) 旗標，指示用於審核音訊時是否要產生文字記錄。

如果沒有命令列引數，`Main()` 會呼叫 `GetUserInputs()`。 此方法會提示使用者輸入單一影片檔的路徑，並指定是否應產生文字記錄。

> [!NOTE]
> 主控台應用程式會使用 [Azure 媒體索引器 API](https://docs.microsoft.com/azure/media-services/media-services-process-content-with-indexer2) 從已上傳視訊的音軌中產生文字記錄。結果會以 WebVTT 格式提供。 如需有關此格式的詳細資訊，請參閱 [Web 影片文字播放軌格式](https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API)。

### <a name="initialize-and-processvideo-methods"></a>Initialize 和 ProcessVideo 方法

不論程式的選項來自命令列或互動式使用者輸入，`Main()` 接下來會呼叫 `Initialize()` 來建立下列執行個體：

|類別|說明|
|-|-|
|`AMSComponent`|提交影片檔案以進行審核前，先壓縮影片檔案。|
|`AMSconfigurations`|應用程式組態資料的介面，位於 `App.config`。|
|`VideoModerator`| 使用 AMS SDK 執行上傳、編碼、加密及審核|
|`VideoReviewApi`|管理 Content Moderator 服務中的視訊檢閱|

這些類別 (除了簡單明瞭的 `AMSConfigurations` 以外) 會在本教學課程的後續章節中更詳細地討論。

最後，針對每個影片檔案呼叫 `ProcessVideo()`，可一次處理一個影片檔案。

    private static async Task ProcessVideo(string videoPath)
    {
        Stopwatch sw = new Stopwatch();
        sw.Start();
        Console.ForegroundColor = ConsoleColor.White;
        Console.WriteLine("\nVideo compression process started...");

        var compressedVideoPath = amsComponent.CompressVideo(videoPath);
        if (string.IsNullOrWhiteSpace(compressedVideoPath))
        {
            Console.ForegroundColor = ConsoleColor.Red;
            Console.WriteLine("Video Compression failed.");
        }

        Console.WriteLine("\nVideo compression process completed...");

        UploadVideoStreamRequest uploadVideoStreamRequest = CreateVideoStreamingRequest(compressedVideoPath);
        UploadAssetResult uploadResult = new UploadAssetResult();

        if (generateVtt)
        {
            uploadResult.GenerateVTT = generateVtt;
        }
        Console.WriteLine("\nVideo moderation process started...");

        if (!videoModerator.CreateAzureMediaServicesJobToModerateVideo(uploadVideoStreamRequest, uploadResult))
        {
            Console.ForegroundColor = ConsoleColor.Red;
            Console.WriteLine("Video Review process failed.");
        }

        Console.WriteLine("\nVideo moderation process completed...");
        Console.WriteLine("\nVideo review process started...");
        string reviewId = await videoReviewApi.CreateVideoReviewInContentModerator(uploadResult);
        Console.WriteLine("\nVideo review successfully completed...");
        sw.Stop();
        Console.WriteLine("\nTotal Elapsed Time: {0}", sw.Elapsed);
        using (var stw = new StreamWriter(AmsConfigurations.logFilePath, true))
        {
            stw.WriteLine("Video File Name: " + Path.GetFileName(videoPath));
            stw.WriteLine($"ReviewId: {reviewId}");
            stw.WriteLine("Total Elapsed Time: {0}", sw.Elapsed);
        }
    }


`ProcessVideo()` 方法相當簡單。 它會依序執行下列作業：

- 壓縮影片
- 將影片上傳至 Azure 媒體服務資產
- 建立 AMS 作業以審核影片
- 在 Content Moderator 中建立影片檢閱

下列各節將更仔細考量一些由 `ProcessVideo()` 叫用的個別程序。 

## <a name="compressing-the-video"></a>壓縮影片

為了將網路流量降到最低，應用程式會將影片檔案轉換為 H.264 (MPEG-4 AVC) 格式，並將其調整為最大寬度 640 像素。 建議您使用 H.264 轉碼器，因為它有很高的效率 (壓縮率)。 壓縮會使用免費的 `ffmpeg` 命令列工具來完成，其隨附於 Visual Studio 解決方案的 `Lib` 資料夾中。 輸入檔案可以使用 `ffmpeg` 支援的任何格式，包括最常使用的影片檔案格式和轉碼器。

> [!NOTE]
> 當您使用命令列選項啟動程式時，您會指定包含影片檔案的目錄，以提交檔案來進行審核。 此目錄中副檔名為 `.mp4` 的所有檔案皆會進入處理程序。 若要處理其他副檔名，請更新 `Program.cs` 中的 `Main()` 方法，讓其包含所需的副檔名。

壓縮單一影片檔案的程式碼是 `AMSComponent.cs` 中的 `AmsComponent` 類別。 負責這項功能的方法是 `CompressVideo()`，如下所示。

    public string CompressVideo(string videoPath)
    {
        string ffmpegBlobUrl;
        if (!ValidatePreRequisites())
        {
            Console.WriteLine("Configurations check failed. Please cross check the configurations!");
            throw new Exception();
        }

        if (File.Exists(_configObj.FfmpegExecutablePath))
        {
            ffmpegBlobUrl = this._configObj.FfmpegExecutablePath;
        }
        else
        {
            Console.WriteLine("ffmpeg.exe is missing. Please check the Lib folder");
            throw new Exception();
        }

        string videoFilePathCom = videoPath.Split('.')[0] + "_c.mp4";
        ProcessStartInfo processStartInfo = new ProcessStartInfo();
        processStartInfo.WindowStyle = ProcessWindowStyle.Hidden;
        processStartInfo.FileName = ffmpegBlobUrl;
        processStartInfo.Arguments = "-i \"" + videoPath + "\" -vcodec libx264 -n -crf 32 -preset veryfast -vf scale=640:-1 -c:a aac -aq 1 -ac 2 -threads 0 \"" + videoFilePathCom + "\"";
        var process = Process.Start(processStartInfo);
        process.WaitForExit();
        process.Close();
        return videoFilePathCom;
    }

程式碼會執行下列步驟：

- 檢查 `App.config` 中的組態，確定其包含所有必要資料
- 檢查並確定 `ffmpeg` 二進位檔已存在
- 藉由在檔案基底名稱上加上 `_c.mp4` 來建置輸出檔案的名稱 (例如 `Example.mp4` -> `E>xample_c.mp4`)
- 建置命令列字串來執行轉換
- 使用命令列開始 `ffmpeg` 處理程序
- 等候影片進行處理

> [!NOTE]
> 如果您知道影片已使用 H.264 進行壓縮，而且有適當的規格，則您可以重新撰寫 `CompressVideo()` 以略過壓縮。

方法會傳回壓縮後輸出檔案的檔案名稱。

## <a name="uploading-and-moderating-the-video"></a>上傳及審核影片

影片必須儲存在 Azure 媒體服務，才能由內容審核服務進行處理。 `Program.cs` 中的 `Program` 類別具有 `CreateVideoStreamingRequest()` 簡短方法，其傳回的物件代表用來上傳影片的串流要求。

    private static UploadVideoStreamRequest CreateVideoStreamingRequest(string compressedVideoFilePath)
    {
        return
            new UploadVideoStreamRequest
            {
                VideoStream = File.ReadAllBytes(compressedVideoFilePath),
                VideoName = Path.GetFileName(compressedVideoFilePath),
                EncodingRequest = new EncodingRequest()
                {
                    EncodingBitrate = AmsEncoding.AdaptiveStreaming
                },
                VideoFilePath = compressedVideoFilePath
            };
    }

產生的 `UploadVideoStreamRequest` 物件會定義於 `UploadVideoStreamRequest.cs` (以及位於 `UploadVideoRequest.cs` 中的其父代 `UploadVideoRequest`)。 這些類別不會在此顯示；這些類別是簡短類別，而且只會用來保存壓縮的影片資料和相關資訊。 另一個僅限資料的類別 `UploadAssetResult` (`UploadAssetResult.cs`) 會用來保存上傳程序的結果。 現在應該就可以了解 `ProcessVideo()` 中的這幾行：

    UploadVideoStreamRequest uploadVideoStreamRequest = CreateVideoStreamingRequest(compressedVideoPath);
    UploadAssetResult uploadResult = new UploadAssetResult();

    if (generateVtt)
    {
        uploadResult.GenerateVTT = generateVtt;
    }
    Console.WriteLine("\nVideo moderation process started...");

    if (!videoModerator.CreateAzureMediaServicesJobToModerateVideo(uploadVideoStreamRequest, uploadResult))
    {
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine("Video Review process failed.");
    }

這幾行會執行下列工作：

- 建立 `UploadVideoStreamRequest` 來上傳壓縮的影片
- 設定要求的 `GenerateVTT` 旗標 (如果使用者已要求文字記錄)
- 呼叫 `CreateAzureMediaServicesJobToModerateVideo()` 以執行上傳並接收結果

## <a name="deep-dive-into-video-moderation"></a>深入探討影片審核

`CreateAzureMediaServicesJobToModerateVideo()` 方法位於 `VideoModerator.cs`，其包含大量與 Azure 媒體服務互動的程式碼。 下列擷取區段中會顯示方法的原始程式碼。

    public bool CreateAzureMediaServicesJobToModerateVideo(UploadVideoStreamRequest uploadVideoRequest, UploadAssetResult uploadResult)
    {
        asset = CreateAsset(uploadVideoRequest);
        uploadResult.VideoName = uploadVideoRequest.VideoName;
        // Encoding the asset , Moderating the asset, Generating transcript in parallel
        IAsset encodedAsset = null;
        //Creates the job for the tasks.
        IJob job = this._mediaContext.Jobs.Create("AMS Review Job");

        //Adding encoding task to job.
        ConfigureEncodeAssetTask(uploadVideoRequest.EncodingRequest, job);

        ConfigureContentModerationTask(job);

        //adding transcript task to job.
        if (uploadResult.GenerateVTT)
        {
            ConfigureTranscriptTask(job);
        }

        Stopwatch timer = new Stopwatch();
        timer.Start();
        //submit and execute job.
        job.Submit();
        job.GetExecutionProgressTask(new CancellationTokenSource().Token).Wait();
        timer.Stop();
        using (var sw = new StreamWriter(AmsConfigurations.logFilePath, true))
        {
            sw.WriteLine("AMS Job Elapsed Time: {0}", timer.Elapsed);
        }

        if (job.State == JobState.Error)
        {
            throw new Exception("Video moderation has failed due to AMS Job error.");
        }

        UploadAssetResult result = uploadResult;
        encodedAsset = job.OutputMediaAssets[0];
        result.ModeratedJson = GetCmDetail(job.OutputMediaAssets[1]);
        // Check for valid Moderated JSON
        var jsonModerateObject = JsonConvert.DeserializeObject<VideoModerationResult>(result.ModeratedJson);

        if (jsonModerateObject == null)
        {
            return false;
        }
        if (uploadResult.GenerateVTT)
        {
            GenerateTranscript(job.OutputMediaAssets.Last());
        }

        uploadResult.StreamingUrlDetails = PublishAsset(encodedAsset);
        string downloadUrl = GenerateDownloadUrl(asset, uploadVideoRequest.VideoName);
        uploadResult.StreamingUrlDetails.DownloadUri = downloadUrl;
        uploadResult.VideoName = uploadVideoRequest.VideoName;
        uploadResult.VideoFilePath = uploadVideoRequest.VideoFilePath;
        return true;
    }

此程式碼會執行下列工作：

- 建立 AMS 作業以完成處理程序
- 新增用來編碼影片檔案、審核影片檔案及產生文字記錄的工作
- 提交作業上、上傳檔案及開始處理程序
- 擷取審核結果、文字記錄 (若有要求的話) 及其他資訊

## <a name="sample-video-moderation-response"></a>影片審核回應的範例

影片審核作業的結果 (請參閱[影片審核快速入門](video-moderation-api.md)) 是以 JSON 資料結構包含審核結果。 這些結果包含影片內的片段分解 (截圖)，每個片段分解都包含事件 (剪輯)，而其中用來進行檢閱的主要畫面會以旗標標示。 每個主要畫面都會以包含成人或不雅內容的可能性來加以評分。 下列範例顯示 JSON 回應：

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

如果設定 `GenerateVTT` 旗標，則也會產生影片中的音訊文字記錄。

> [!NOTE]
> 主控台應用程式會使用 [Azure 媒體索引器 API](https://docs.microsoft.com/azure/media-services/media-services-process-content-with-indexer2) 從已上傳視訊的音軌中產生文字記錄。結果會以 WebVTT 格式提供。 如需有關此格式的詳細資訊，請參閱 [Web 影片文字播放軌格式](https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API)。


## <a name="creating-the-human-in-the-loop-review"></a>建立人機互動檢閱

審核程序會傳回影片中的主要畫面清單，以及其音軌的文字記錄。 下一個步驟是在 Content Moderator 審核工具中建立審核項目，以進行人工審核。 返回 `Program.cs` 中的 `ProcessVideo()` 方法，您會看到對 `CreateVideoReviewInContentModerator()` 方法的呼叫。 此方法位於 `VideoReviewAPI.cs` 中的 `videoReviewApi` 類別，如下所示。

    public async Task<string> CreateVideoReviewInContentModerator(UploadAssetResult uploadAssetResult)
    {
    
        string reviewId = string.Empty;
        List<ProcessedFrameDetails> frameEntityList = framegenerator.CreateVideoFrames(uploadAssetResult);
        string path = uploadAssetResult.GenerateVTT == true ? this._amsConfig.FfmpegFramesOutputPath + Path.GetFileNameWithoutExtension (uploadAssetResult.VideoName) + "_aud_SpReco.vtt" : "";
        TranscriptScreenTextResult screenTextResult = new TranscriptScreenTextResult();
        
    if (File.Exists(path))
        {
            screenTextResult = await GenerateTextScreenProfanity(reviewId, path, frameEntityList);
            uploadAssetResult.Category1TextScore = screenTextResult.Category1Score;
            uploadAssetResult.Category2TextScore = screenTextResult.Category2Score;
            uploadAssetResult.Category3TextScore = screenTextResult.Category3Score;
            uploadAssetResult.Category1TextTag = screenTextResult.Category1Tag;
            uploadAssetResult.Category2TextTag = screenTextResult.Category2Tag;
            uploadAssetResult.Category3TextTag = screenTextResult.Category3Tag;
        }
        
        var reviewVideoRequestJson = CreateReviewRequestObject(uploadAssetResult, frameEntityList);
        if (string.IsNullOrWhiteSpace(reviewVideoRequestJson))
        {
            throw new Exception("Video review process failed in CreateVideoReviewInContentModerator");
        }
        
        reviewId = JsonConvert.DeserializeObject<List<string>>(ExecuteCreateReviewApi(reviewVideoRequestJson).Result).FirstOrDefault();
        frameEntityList = framegenerator.GenerateFrameImages(frameEntityList, uploadAssetResult, reviewId);
        await CreateAndPublishReviewInContentModerator(uploadAssetResult, frameEntityList, reviewId, path, screenTextResult);
        return reviewId;
    
    }

`CreateVideoReviewInContentModerator()` 會呼叫數種其他方法來執行下列工作：

> [!NOTE]
> 主控台應用程式會使用 [FFmpeg](https://ffmpeg.org/) 程式庫來產生縮圖。 這些縮圖 (影像) 會對應至[影片審核輸出](#sample-video-moderation-response)中的畫面時間戳記。

|Task|方法|檔案|
|-|-|-|
|從影片中擷取主要畫面，並建立其縮圖影像|`CreateVideoFrames()`<br>`GenerateFrameImages()`|`FrameGeneratorServices.cs`|
|瀏覽文字記錄 (如果有的話)，以找出成人或不雅音訊|`GenerateTextScreenProfanity()`| `VideoReviewAPI.cs`|
|準備並提交影片檢閱要求，以進行人力檢查|`CreateReviewRequestObject()`<br> `ExecuteCreateReviewApi()`<br>`CreateAndPublishReviewInContentModerator()`|`VideoReviewAPI.cs`|

## <a name="video-review-default-view"></a>影片檢閱的預設檢視

下列畫面會顯示先前步驟的結果。

![影片檢閱的預設檢視](images/video-tutorial-default-view.PNG)

## <a name="transcript-generation"></a>文字記錄的產生

到目前為止，本教學課程中出現的程式碼都著重於視覺內容。 語音內容檢閱是個別且選用的處理程序，它會使用從音訊產生的文字記錄 (如前面所述)。 現在該來看看文字記錄如何在檢閱程序中建立及使用。 產生文字記錄的工作會落在 [Azure 媒體索引器](https://docs.microsoft.com/azure/media-services/media-services-index-content)服務上。

此應用程式會執行下列工作：

|Task|方法|檔案|
|-|-|-|
|判斷是否要產生文字記錄|`Main()`<br>`GetUserInputs()`|`Program.cs`|
|如果要，請提交轉譯作業作為審核的一部分|`ConfigureTranscriptTask()`|`VideoModerator.cs`|
|取得文字記錄的本機複本|`GenerateTranscript()`|`VideoModerator.cs`|
|以旗標標示包含不適當音訊的影片畫面|`GenerateTextScreenProfanity()`<br>`TextScreen()`|`VideoReviewAPI.cs`|
|將結果新增至檢閱|`UploadScreenTextResult()`<br>`ExecuteAddTranscriptSupportFile()`|`VideoReviewAPI.cs`|

### <a name="task-configuration"></a>工作組態

讓我們直接跳到提交轉譯作業。 `CreateAzureMediaServicesJobToModerateVideo()` (已描述過) 會呼叫 `ConfigureTranscriptTask()`。

    private void ConfigureTranscriptTask(IJob job)
    {
        string mediaProcessorName = _amsConfigurations.MediaIndexer2MediaProcessor;
        IMediaProcessor processor = _mediaContext.MediaProcessors.GetLatestMediaProcessorByName(mediaProcessorName);

        string configuration = File.ReadAllText(_amsConfigurations.MediaIndexerConfigurationJson);
        ITask task = job.Tasks.AddNew("AudioIndexing Task", processor, configuration, TaskOptions.None);
        task.InputAssets.Add(asset);
        task.OutputAssets.AddNew("AudioIndexing Output Asset", AssetCreationOptions.None);
    }

文字記錄工作的組態會從 `MediaIndexerConfig.json` 檔案中讀取，此檔案位在解決方案的 `Lib` 資料夾。 AMS 資產會針對組態檔案和轉譯程序的輸出來建立。 當 AMS 作業執行時，此工作會從影片檔案的音軌中建立文字記錄。

> [!NOTE]
> 範例應用程式只會辨識英文 (美國) 的語音。

### <a name="transcript-generation"></a>文字記錄的產生

文字記錄會發佈為 AMS 資產。 若要瀏覽文字記錄以找出不當內容，應用程式會從 Azure 媒體服務中下載該資產。 `CreateAzureMediaServicesJobToModerateVideo()` 會呼叫 `GenerateTranscript()` 來擷取檔案，如下所示。

    public bool GenerateTranscript(IAsset asset)
    {
        try
        {
            var outputFolder = this._amsConfigurations.FfmpegFramesOutputPath;
            IAsset outputAsset = asset;
            IAccessPolicy policy = null;
            ILocator locator = null;
            policy = _mediaContext.AccessPolicies.Create("My 30 days readonly policy", TimeSpan.FromDays(360), AccessPermissions.Read);
            locator = _mediaContext.Locators.CreateLocator(LocatorType.Sas, outputAsset, policy, DateTime.UtcNow.AddMinutes(-5));
            DownloadAssetToLocal(outputAsset, outputFolder);
            locator.Delete();
            return true;
        }
        catch
        {   //TODO:  Logging
            Console.WriteLine("Exception occured while generating index for video.");
            throw;
        }
    }

經過一些必要的 AMS 設定後，會藉由呼叫 `DownloadAssetToLocal()` 泛型函式來執行實際的下載，此函式會將 AMS 資產複製到本機檔案。

## <a name="transcript-moderation"></a>文字記錄審核

取得文字記錄後，即可進行瀏覽和檢閱。 建立檢閱是 `CreateVideoReviewInContentModerator()` 的範圍，它會呼叫 `GenerateTextScreenProfanity()` 來執行這項作業。 接著，這個方法會呼叫包含大部分功能的 `TextScreen()`。 

`TextScreen()` 會執行下列工作：

- 剖析文字記錄以取得時間戳記和標題
- 提交每個標題以進行文字審核
- 以旗標標示可能有不當語音內容的畫面

讓我們更仔細地檢查這幾個工作：

### <a name="initialize-the-code"></a>初始化程式碼

首先，初始化所有變數和集合。

    private async Task<TranscriptScreenTextResult> TextScreen(string filepath, List<ProcessedFrameDetails> frameEntityList)
    {
        List<TranscriptProfanity> profanityList = new List<TranscriptProfanity>();
        string responseContent = string.Empty;
        HttpResponseMessage response;
        bool category1Tag = false;
        bool category2Tag = false;
        bool category3Tag = false;
        double category1Score = 0;
        double category2Score = 0;
        double category3Score = 0;
        List<string> vttLines = File.ReadAllLines(filepath).Where(line => !line.Contains("NOTE Confidence:") && line.Length > 0).ToList();
        StringBuilder sb = new StringBuilder();
        List<CaptionScreentextResult> csrList = new List<CaptionScreentextResult>();
        CaptionScreentextResult captionScreentextResult = new CaptionScreentextResult() { Captions = new List<string>() };

        // Code from the next sections in the tutorial
    

### <a name="parse-the-transcript-for-captions"></a>剖析標題的文字記錄

接下來，剖析 VTT 格式化的標題文字記錄，以取得標題和時間戳記。 審核工具會將這些標題顯示在影片檢閱畫面上的 [文字記錄] 索引標籤中。 時間戳記會用來同步標題與相對應的影片畫面。

        // Code from the previous section(s) in the tutorial

        //
        // Parse the transcript
        //
        foreach (var line in vttLines.Skip(1))
        {
                if (line.Contains("-->"))
                {
                    if (sb.Length > 0)
                    {
                        captionScreentextResult.Captions.Add(sb.ToString());
                        sb.Clear();
                    }
                    if (captionScreentextResult.Captions.Count > 0)
                    {
                        csrList.Add(captionScreentextResult);
                        captionScreentextResult = new CaptionScreentextResult() { Captions = new List<string>() };
                    }
                    string[] times = line.Split(new string[] { "-->" }, StringSplitOptions.RemoveEmptyEntries);
                    string startTimeString = times[0].Trim();
                    string endTimeString = times[1].Trim();
                    int startTime = (int)TimeSpan.ParseExact(startTimeString, @"hh\:mm\:ss\.fff", CultureInfo.InvariantCulture).TotalMilliseconds;
                    int endTime = (int)TimeSpan.ParseExact(endTimeString, @"hh\:mm\:ss\.fff", CultureInfo.InvariantCulture).TotalMilliseconds;
                    captionScreentextResult.StartTime = startTime;
                    captionScreentextResult.EndTime = endTime;
                }
                else
                {
                    sb.Append(line);
                }
                if (sb.Length + line.Length > 1024)
                {
                    captionScreentextResult.Captions.Add(sb.ToString());
                    sb.Clear();
                }
            }
            if (sb.Length > 0)
            {
                captionScreentextResult.Captions.Add(sb.ToString());
            }
            if (captionScreentextResult.Captions.Count > 0)
            {
                csrList.Add(captionScreentextResult);
            }

            // Code from the following section in the quickstart

### <a name="moderate-captions-with-the-text-moderation-service"></a>使用文字審核服務來審核標題

接下來，我們會以 Content Moderator 的文字 API 來掃描剖析的文字標題。

> [!NOTE]
> Content Moderator 服務金鑰會有每秒要求數目 (RPS) 的速率限制。 如果您超出此限制，SDK 就會擲回錯誤碼為 429 的例外狀況。 
>
> 免費層金鑰有一個 RPS 速率限制。

    //
    // Moderate the captions or cues
    //
    int waitTime = 1000;
    foreach (var csr in csrList)
    {
                bool captionAdultTextTag = false;
                bool captionRacyTextTag = false;
                bool captionOffensiveTextTag = false;
                bool retry = true;

                foreach (var caption in csr.Captions)
                {
                    while (retry)
                    {
                        try
                        {
                            System.Threading.Thread.Sleep(waitTime);
                            var lang = await CMClient.TextModeration.DetectLanguageAsync("text/plain", caption);
                            var oRes = await CMClient.TextModeration.ScreenTextWithHttpMessagesAsync(lang.DetectedLanguageProperty, "text/plain", caption, null, null, null, true);
                            response = oRes.Response;
                            responseContent = await response.Content.ReadAsStringAsync();
                            retry = false;
                        }
                        catch (Exception e)
                        {
                            if (e.Message.Contains("429"))
                            {
                                Console.WriteLine($"Moderation API call failed. Message: {e.Message}");
                                waitTime = (int)(waitTime * 1.5);
                                Console.WriteLine($"wait time: {waitTime}");
                            }
                            else
                            {
                                retry = false;
                                Console.WriteLine($"Moderation API call failed. Message: {e.Message}");
                            }
                        }
                    }
                    var jsonTextScreen = JsonConvert.DeserializeObject<TextScreen>(responseContent);
                    if (jsonTextScreen != null)
                    {
                        TranscriptProfanity transcriptProfanity = new TranscriptProfanity();
                        transcriptProfanity.TimeStamp = "";
                        List<Terms> transcriptTerm = new List<Terms>();
                        if (jsonTextScreen.Terms != null)
                        {
                            foreach (var term in jsonTextScreen.Terms)
                            {
                                var profanityobject = new Terms
                                {
                                    Term = term.Term,
                                    Index = term.Index
                                };
                                transcriptTerm.Add(profanityobject);
                            }
                            transcriptProfanity.Terms = transcriptTerm;
                            profanityList.Add(transcriptProfanity);
                        }
                        if (jsonTextScreen.Classification.Category1.Score > _amsConfig.Category1TextThreshold) captionAdultTextTag = true;
                        if (jsonTextScreen.Classification.Category2.Score > _amsConfig.Category2TextThreshold) captionRacyTextTag = true;
                        if (jsonTextScreen.Classification.Category3.Score > _amsConfig.Category3TextThreshold) captionOffensiveTextTag = true;
                        if (jsonTextScreen.Classification.Category1.Score > _amsConfig.Category1TextThreshold) category1Tag = true;
                        if (jsonTextScreen.Classification.Category2.Score > _amsConfig.Category2TextThreshold) category2Tag = true;
                        if (jsonTextScreen.Classification.Category3.Score > _amsConfig.Category3TextThreshold) category3Tag = true;
                        category1Score = jsonTextScreen.Classification.Category1.Score > category1Score ? jsonTextScreen.Classification.Category1.Score : category1Score;
                        category2Score = jsonTextScreen.Classification.Category2.Score > category2Score ? jsonTextScreen.Classification.Category2.Score : category2Score;
                        category3Score = jsonTextScreen.Classification.Category3.Score > category3Score ? jsonTextScreen.Classification.Category3.Score : category3Score;
                    }
                    foreach (var frame in frameEntityList.Where(x => x.TimeStamp >= csr.StartTime && x.TimeStamp <= csr.EndTime))
                    {
                        frame.IsAdultTextContent = captionAdultTextTag;
                        frame.IsRacyTextContent = captionRacyTextTag;
                        frame.IsOffensiveTextContent = captionOffensiveTextTag;
                    }
                }
            }
            TranscriptScreenTextResult screenTextResult = new TranscriptScreenTextResult()
            {
                TranscriptProfanity = profanityList,
                Category1Tag = category1Tag,
                Category2Tag = category2Tag,
                Category3Tag = category3Tag,
                Category1Score = category1Score,
                Category2Score = category2Score,
                Category3Score = category3Score
            };
            return screenTextResult;
    }

### <a name="breaking-down-the-text-moderation-step"></a>細分文字審核步驟

`TextScreen()` 是內容充實的方法，所以讓我們來分解它。

1. 首先，此方法會一行一行地讀取文字記錄檔案。 它會忽略空白行和其中有信賴分數的 `NOTE` 行。 它會從檔案中的「提示」擷取時間戳記和文字項目。 「提示」表示來自音軌的文字，並且包含開始和結束時間。 提示會以包含 `-->` 字串的時間戳記行開頭。 其後方會跟著一行或多行文字。

1. `CaptionScreentextResult` 的執行個體 (定義於`TranscriptProfanity.cs`) 會用來保存從每個提示剖析的資訊。  當偵測到新的時間戳記行時，或達到最大文字長度 1024 個字元時，新的 `CaptionScreentextResult` 會新增至 `csrList`。 

1. 方法接著會將每個提示提交給文字審核 API。 它會呼叫 `Microsoft.Azure.CognitiveServices.ContentModerator` 組件中定義的 `ContentModeratorClient.TextModeration.DetectLanguageAsync()` 和 `ContentModeratorClient.TextModeration.ScreenTextWithHttpMessagesAsync()`。 為避免有速率限制，方法會在提交每個提示前先暫停一秒。

1. 從文字審核服務接收結果之後，方法接著會分析這些結果，以查看其是否符合信賴閾值。 在 `App.config` 中，這些值會建立為 `OffensiveTextThreshold`、`RacyTextThreshold` 和 `AdultTextThreshold`。 最後，也會儲存不當字詞本身。 提示時間範圍內的所有畫面都會標示為包含具攻擊性、不雅及/或成人文字。

1. `TextScreen()` 會傳回 `TranscriptScreenTextResult` 執行個體，其中包含整個影片的文字審核結果。 此物件包含各種不當內容的旗標和分數，以及所有不當字詞的清單。 呼叫端 `CreateVideoReviewInContentModerator()`會呼叫 `UploadScreenTextResult()` 來將此資訊附加至檢閱，因此可供人類檢閱者使用。
 
## <a name="video-review-transcript-view"></a>影片檢視的文字記錄檢視

下列畫面顯示文字記錄結果的產生和審核步驟。

![影片審核文字記錄檢視](images/video-tutorial-transcript-view.PNG)

## <a name="program-output"></a>程式輸出

下列來自程式的命令列輸出會顯示已完成的各種工作。 此外，審核結果 (JSON 格式) 和語音文字記錄可從原始影片檔案所在的目錄中取得。

    Microsoft.ContentModerator.AMSComponentClient
    Enter the fully qualified local path for Uploading the video :
    "Your File Name.MP4"
    Generate Video Transcript? [y/n] : y
    
    Video compression process started...
    Video compression process completed...
    
    Video moderation process started...
    Video moderation process completed...
    
    Video review process started...
    Video Frames Creation inprogress...
    Frames(83) created successfully.
    Review Created Successfully and the review Id 201801va8ec2108d6e043229ba7a9e6373edec5
    Video review successfully completed...
    
    Total Elapsed Time: 00:05:56.8420355


## <a name="next-steps"></a>後續步驟

[下載 Visual Studio 解決方案](https://github.com/MicrosoftContentModerator/VideoReviewConsoleApp)、範例檔案和必要的程式庫以用於此教學課程，並開始進行您的整合作業。
