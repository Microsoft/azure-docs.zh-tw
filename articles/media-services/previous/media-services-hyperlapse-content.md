---
title: Hyperlapse Media 檔案與 Azure 媒體超縮時攝影 | Microsoft Docs
description: Azure Media Hyperlapse 能夠利用第一人稱視角或運動攝影的內容，來建立流暢的縮時影片。 本主題說明如何使用 Media Indexer。
services: media-services
documentationcenter: ''
author: asolanki
manager: johndeu
editor: ''
ms.assetid: 37d54db6-9cf3-4ae9-b3c6-0d29c744e965
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/28/2018
ms.author: adsolank
ms.openlocfilehash: ed64a616538ed4699abc03225a2dcf27d164521f
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783167"
---
# <a name="hyperlapse-media-files-with-azure-media-hyperlapse"></a>Hyperlapse Media 檔案與 Azure Media Hyperlapse
Azure Media Hyperlapse 是可以使用第一人稱視角或運動攝影機內容建立流暢縮時攝影影片的「媒體處理器 (MP)」。  Azure 媒體服務的雲端型 Microsoft Hyperlapse 與 [Microsoft Research 的桌面 Hyperlapse Pro 和手機型 Hyperlapse Mobile](http://aka.ms/hyperlapse)相似，它運用大規模的 Azure 媒體服務媒體處理平台來水平調整，並平行化大量的 Hyperlapse 處理。

> [!IMPORTANT]
> Microsoft Hyperlapse 是專門使用移動中攝影機拍攝第一人稱視角內容而設計。 雖然攝影機位置固定的內容也可以運作，但 Azure 媒體 Hyperlapse 媒體處理器無法保證其他類型內容的效能及品質。
> 
> 

Azure 媒體 Hyperlapse 工作接受輸入 MP4、MOV 或 WMV 資產檔案連同組態檔，以指定影片中要縮時的畫面及其速度 (例如前 10,000 個畫面速度為 2x)。  輸出是輸入影片經過穩定和縮時轉譯的成果。

## <a name="hyperlapse-an-asset"></a>將資產進行 Hyperlapse 處理
首先您需要上傳要輸入 Azure 媒體服務的檔案。  若要深入了解上傳和管理內容的相關概念，請閱讀 [內容管理文章](media-services-portal-vod-get-started.md)。

### <a id="configuration"></a>Hyperlapse 的預設組態
一旦內容在媒體服務帳戶中，您將需要建構預設組態。  下表說明使用者指定的欄位：

| 欄位 | 說明 |
| --- | --- |
| StartFrame |開始進行 Microsoft Hyperlapse 處理的畫面。 |
| NumFrames |要處理的畫面數目。 |
| 速度 |輸入影片要加速的設定因素。 |

以下是符合標準的 JSON 和 XML 格式組態檔：

**XML 預設：**
```xml
    <?xml version="1.0" encoding="utf-16"?>
    <Preset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
        <Sources>
            <Source StartFrame="0" NumFrames="10000" />
        </Sources>
        <Options>
            <Speed>12</Speed>
        </Options>
    </Preset>
```

**JSON 預設：**
```json
    {
        "Version":1.0,
        "Sources": [
            {
                "StartFrame":0,
                "NumFrames":2147483647
            }
        ],
        "Options": {
            "Speed":1,
            "Stabilize":false
        }
    }
```

### <a id="sample_code"></a> 包含 AMS .NET SDK 的 Microsoft Hyperlapse
下列方法會將媒體檔案上傳為資產，並使用 Azure 媒體 Hyperlapse 媒體處理器建立工作。

> [!NOTE]
> 為了使程式碼可以運作，您應該已經具備在名稱為「context」之範圍內的 CloudMediaContext。  若要深入了解，請參閱 [內容管理文章](media-services-dotnet-get-started.md)。
> 
> [!NOTE]
> 字串引數 "hyperConfig" 預期為上述符合標準的 JSON 或 XML 格式的預設組態。
> 
> 

```csharp
        static bool RunHyperlapseJob(string input, string output, string hyperConfig)
        {
            // create asset with input file
            IAsset asset = context
            .Assets
            .CreateAssetAndUploadSingleFile(input, "My Hyperlapse Input", AssetCreationOptions.None);

            // grab instances of Azure Media Hyperlapse MP
            IMediaProcessor mp = context
            .MediaProcessors
            .GetLatestMediaProcessorByName("Azure Media Hyperlapse");

            // create Job with Hyperlapse task
            IJob job = context
            .Jobs
            .Create(String.Format("Hyperlapse {0}", input));

            if (String.IsNullOrEmpty(hyperConfig))
            {
            // config cannot be empty
            return false;
            }

            hyperConfig = File.ReadAllText(hyperConfig);

            ITask hyperlapseTask = job.Tasks.AddNew("Hyperlapse task",
            mp,
            hyperConfig,
            TaskOptions.None);
            hyperlapseTask.InputAssets.Add(asset);
            hyperlapseTask.OutputAssets.AddNew("Hyperlapse output",
            AssetCreationOptions.None);

            job.Submit();

            // Create progress printing and querying tasks
            Task progressPrintTask = new Task(() =>
            {

            IJob jobQuery = null;
            do
            {
                var progressContext = context;
                jobQuery = progressContext.Jobs
                .Where(j => j.Id == job.Id)
                .First();
                Console.WriteLine(string.Format("{0}\t{1}\t{2}",
                DateTime.Now,
                jobQuery.State,
                jobQuery.Tasks[0].Progress));
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
                return false;                  
            }

        DownloadAsset(job.OutputMediaAssets.First(), output);
        return true;
    }

    static void DownloadAsset(IAsset asset, string outputDirectory)
    {
        foreach (IAssetFile file in asset.AssetFiles)
        {
            file.Download(Path.Combine(outputDirectory, file.Name));
        }
    }


    static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
    {
        IAsset asset = context.Assets.Create(assetName, options);

        var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
        assetFile.Upload(filePath);

        return asset;
    }

    static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
    {
        var processor = context.MediaProcessors
        .Where(p => p.Name == mediaProcessorName)
        .ToList()
        .OrderBy(p => new Version(p.Version))
        .LastOrDefault();

        if (processor == null)
            throw new ArgumentException(string.Format("Unknown media processor",
                                                       mediaProcessorName));

        return processor;
    }
```

### <a id="file_types"></a>支援的檔案類型
* MP4
* MOV
* WMV

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>相關連結
[Azure 媒體服務分析概觀](media-services-analytics-overview.md)

[Azure 媒體分析示範](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

