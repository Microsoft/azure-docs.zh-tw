---
title: 使用 Azure 媒體內容仲裁來偵測潛在的成人和猥褻內容 | Microsoft Docs
description: 影片仲裁可協助您偵測影片中潛在的成人和猥褻內容。
services: media-services
documentationcenter: ''
author: sanjeev3
manager: mikemcca
editor: ''
ms.assetid: a245529f-3150-4afc-93ec-e40d8a6b761d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/06/2018
ms.author: sajagtap
ms.openlocfilehash: e44308f38a138c0e186e41fc8310f8b480cd4e09
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783427"
---
# <a name="use-azure-media-content-moderator-to-detect-possible-adult-and-racy-content"></a>使用 Azure 媒體內容仲裁來偵測潛在的成人和猥褻內容

## <a name="overview"></a>概觀
**Azure 媒體內容仲裁**媒體處理器 (MP) 可讓您針對影片使用機器輔助仲裁。 例如，您可能會想要偵測影片中潛在的成人和猥褻內容，並由您的人力仲裁小組檢閱這些被標記的內容。

**Azure 媒體內容仲裁** MP 目前為預覽功能。

本文提供有關 **Azure 媒體內容仲裁**的詳細資料，並示範如何搭配適用於 .NET 的媒體服務 SDK 來使用它。

## <a name="content-moderator-input-files"></a>內容仲裁輸入檔案
影片檔案。 目前支援下列格式：MP4、MOV 及 WMV。

## <a name="content-moderator-output-files"></a>內容仲裁輸出檔案
JSON 格式的仲裁輸出包含自動偵測的擷取畫面和主要畫面格。 系統會傳回包含是否有潛在成人或猥褻內容之信賴分數的主要畫面格。 它們也會包含一個布林值旗標，指示是否建議檢閱。 系統會依據成人和猥褻分數的內部閾值，指派值給檢閱建議旗標。

## <a name="elements-of-the-output-json-file"></a>輸出 JSON 檔案的元素

作業會產生包含中繼資料的 JSON 輸出檔，這些中繼資料會包含已偵測之擷取畫面和主要畫面格的相關內容，以及描述畫面中是否包含成人或猥褻內容。

輸出 JSON 包括下列元素：

### <a name="root-json-elements"></a>根 JSON 元素

| 元素 | 說明 |
| --- | --- |
| version |內容仲裁的版本。 |
| timescale |影片每秒的「刻度」數目。 |
| Offset |時間戳記的時間位移。 在 1.0 版的影片 API 中，此值永遠是 0。 此值未來可能會變更。 |
| framerate |影片的每秒畫面格數。 |
| width |輸出影片畫面格的寬度 (以像素為單位)。|
| height |輸出影片畫面格的高度 (以像素為單位)。|
| totalDuration |輸入影片的持續時間 (以刻度為單位)。 |
| [fragments](#fragments-json-elements) |中繼資料會被分成稱為「片段」的不同區段。 每個片段都是一個自動偵測的擷取畫面，包含開始、持續時間、間隔數字及事件。 |

### <a name="fragments-json-elements"></a>Fragments JSON 元素

|元素|說明|
|---|---|
| start |第一個事件的開始時間 (以「刻度」為單位)。 |
| duration |片段的長度 (以「刻度」為單位)。 |
| interval |片段內每個事件項目的間隔 (以「刻度」為單位)。 |
| [events](#events-json-elements) |每個事件都代表一段畫面剪輯，而每段畫面剪輯都包含該持續時間內偵測和追蹤的主要畫面格。 它是包含事件的陣列。 外部陣列代表一個單位的時間間隔。 內部陣列包含在那個時間點所發生的 0 或多個事件。|

### <a name="events-json-elements"></a>Events JSON 元素

|元素|說明|
|---|---|
| reviewRecommended | `true` 或 `false` 需視 **adultScore** 或 **racyScore** 是否超出內部閾值而定。 |
| adultScore | 是否有潛在成人內容的信賴分數，範圍為 0.00 到 0.99。 |
| racyScore | 是否有潛在猥褻內容的信賴分數，範圍為 0.00 到 0.99。 |
| index | 從第一個畫面格索引到最後一個畫面格索引的範圍畫格索引。 |
| timestamp | 畫面格位置 (以刻度為單位)。 |
| shotIndex | 父系擷取畫面的索引。 |


## <a name="content-moderation-quickstart-and-sample-output"></a>內容仲裁快速入門和範例輸出

### <a name="task-configuration-preset"></a>工作組態 (預設)
以 **Azure 媒體內容仲裁**建立工作時，您必須指定設定預設值。 下列設定預設值僅適用於內容仲裁。

    {
      "version":"2.0"
    }

### <a name="net-code-sample"></a>.NET 程式碼範例

以下 .NET 程式碼範例使用媒體服務 .NET SDK 來執行內容仲裁作業。 它會使用媒體服務資產作為輸入，其中包含要仲裁的影片。
請參閱[內容仲裁影片快速入門](../../cognitive-services/Content-Moderator/video-moderation-api.md)，以取得完整的原始程式碼和 Visual Studio 專案。


```csharp
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

For the full source code and the Visual Studio project, check out the [Content Moderator video quickstart](../../cognitive-services/Content-Moderator/video-moderation-api.md).

### JSON output

The following example of a Content Moderator JSON output was truncated.

> [!NOTE]
> Location of a keyframe in seconds = timestamp/timescale

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
```

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>相關連結
[Azure 媒體服務分析概觀](media-services-analytics-overview.md)

[Azure 媒體分析示範](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

## <a name="next-steps"></a>後續步驟

深入了解內容仲裁的[影片仲裁和檢閱解決方案](../../cognitive-services/Content-Moderator/video-moderation-human-review.md)。

從[影片仲裁快速入門](../../cognitive-services/Content-Moderator/video-moderation-api.md)取得完整的原始程式碼和 Visual Studio 專案。 

了解如何從仲裁的輸出產生[影片檢閱](../../cognitive-services/Content-Moderator/video-reviews-quickstart-dotnet.md)，以及以 .NET 產生[仲裁文字記錄](../../cognitive-services/Content-Moderator/video-transcript-reviews-quickstart-dotnet.md)。

查看詳細的.NET [影片仲裁並檢閱教學課程](../../cognitive-services/Content-Moderator/video-transcript-moderation-review-tutorial-dotnet.md)。 
