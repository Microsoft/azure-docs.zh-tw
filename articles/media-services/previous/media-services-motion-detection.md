---
title: 使用 Azure 媒體分析偵測動作 | Microsoft Docs
description: 「Azure 媒體動作偵測器」媒體處理器 (MP) 能讓您在冗長且平淡的影片中，有效識別出感興趣的區段。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2017
ms.author: milanga;juliako;
ms.openlocfilehash: 8488b968fe2ab823479d70a98ba86be97b28f67d
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783617"
---
# <a name="detect-motions-with-azure-media-analytics"></a>使用 Azure 媒體分析偵測動作
## <a name="overview"></a>概觀
**Azure 媒體動作偵測器** 媒體處理器 (MP) 能讓您在冗長且平淡的影片中，有效識別出感興趣的區段。 動作偵測可以用來在靜態攝影機資料上識別影片中有動作發生的區段。 它會產生 JSON 檔案，其中包含具有時間戳記的中繼資料，以及發生事件的週框區域。

如果將此技術用於監視器的影片輸出，它將能把動作分類為相關事件及誤判情況 (例如陰影或光源變更)。 這能讓您在不用檢視無止境的不相關事件之下，便能從攝影機輸出產生安全性警示，並從長時間的監視影片中擷取感興趣的片段。

**Azure 媒體動作偵測器** MP 目前為預覽功能。

本文提供有關 **Azure 媒體動作偵測器**的詳細資料，並示範如何搭配適用於 .NET 的媒體服務 SDK 來使用它。

## <a name="motion-detector-input-files"></a>動作偵測器輸入檔案
影片檔案。 目前支援下列格式：MP4、MOV 及 WMV。

## <a name="task-configuration-preset"></a>工作組態 (預設)
以 **Azure 媒體動作偵測器**建立工作時，您必須指定設定預設值。 

### <a name="parameters"></a>參數
您可以使用以下參數：

| Name | 選項 | 說明 | 預設值 |
| --- | --- | --- | --- |
| sensitivityLevel |字串：'low'、'medium'、'high' |設定要報告哪些動作的敏感度等級。 調整該項目可調整誤判的數目。 |'medium' |
| frameSamplingValue |正整數 |設定演算法的執行頻率。 1 等於每個畫面，2 表示每 2 個畫面，依此類推。 |1 |
| detectLightChange |布林值：'true'、'false' |設定是否要在結果中報告光源變化 |'False' |
| mergeTimeThreshold |Xs-time: Hh:mm:ss<br/>範例：00:00:03 |指定要將 2 個事件合併及回報為 1 個事件的動作事件時間間隔。 |00:00:00 |
| detectionZones |偵測區域的陣列︰<br/>- 偵測區域是 3 個或更多個點的陣列<br/>- 點是介於 0 到 1 之間的 x 和 y 座標。 |描述要使用的多邊形偵測區域清單。<br/>結果會使用區域為識別碼報告，第一個為 'id':0 |涵蓋整個畫面的單一區域。 |

### <a name="json-example"></a>JSON 範例

```json
    {
      "version": "1.0",
      "options": {
        "sensitivityLevel": "medium",
        "frameSamplingValue": 1,
        "detectLightChange": "False",
        "mergeTimeThreshold":
        "00:00:02",
        "detectionZones": [
          [
            {"x": 0, "y": 0},
            {"x": 0.5, "y": 0},
            {"x": 0, "y": 1}
           ],
          [
            {"x": 0.3, "y": 0.3},
            {"x": 0.55, "y": 0.3},
            {"x": 0.8, "y": 0.3},
            {"x": 0.8, "y": 0.55},
            {"x": 0.8, "y": 0.8},
            {"x": 0.55, "y": 0.8},
            {"x": 0.3, "y": 0.8},
            {"x": 0.3, "y": 0.55}
          ]
        ]
      }
    }
```

## <a name="motion-detector-output-files"></a>動作偵測器輸出檔案
動作偵測作業會在輸出資產中傳回 JSON 檔案，該檔案會在影片中描述動作警示及類別。 檔案包含在影片中所偵測到動作之時間和持續時間的資訊。

動作偵測器 API 會在物件於固定背景的影片 (例如監視影片) 中移動時提供指示器。 動作偵測器已針對降低誤判 (例如光源和陰影變化) 進行調整。 目前演算法的限制包括夜視影片、半透明物件及小型物件。

### <a id="output_elements"></a>輸出 JSON 檔案的元素
> [!NOTE]
> 在最新版本中，輸出 JSON 格式已變更，而且對某些客戶來說可能代表著重大變更。
> 
> 

下表說明輸出 JSON 的元素。

| 元素 | 說明 |
| --- | --- |
| 版本 |這是指影片 API 的版本。 目前版本為 2。 |
| 時幅 |影片每秒的「刻度」數目。 |
| Offset |時間戳記的時間位移 (以「刻度」為單位)。 在版本 1.0 的影片 API 中，這永遠會是 0。 在我們於未來將支援的案例中，此值可能會變更。 |
| 畫面播放速率 |影片的每秒畫面格數。 |
| 寬度，高度 |指的是影片的寬度和高度 (以像素為單位)。 |
| Start |開始的時間戳記 (以「刻度」為單位)。 |
| Duration |事件的長度 (以「刻度」為單位)。 |
| 間隔 |事件中每個項目的間隔 (以「刻度」為單位)。 |
| 活動 |每個事件片段皆包含在該持續期間內所偵測到的動作。 |
| 類型 |在目前的版本中，針對一般動作，這永遠會是「2」。 此標籤可讓影片 API 在未來的版本中能夠彈性地為動作進行分類。 |
| RegionID |如前文所述，這在此版本中將永遠會是 0。 此標籤可讓影片 API 在未來的版本中能夠彈性地在各個區域中尋找動作。 |
| 區域 |指的是影片中您會關心其中所發生之動作的區塊。 <br/><br/>-「識別碼」表示區域區塊 – 在此版本中只有一個，識別碼為 0。 <br/>-「類型」代表您關心動作的區域形狀。 目前支援「矩形」和「多邊形」。<br/> 如果您指定「矩形」，區域的維度將以 X、Y、寬度及高度表示。 X 和 Y 座標代表以標準化縮放 (0.0 到 1.0) 顯示之區域左上角的 XY 座標。 寬度和高度代表以標準化縮放 (0.0 到 1.0) 顯示之區域的大小。 在目前版本中，X、Y、寬度和高度一律固定為 0, 0 和 1, 1。 <br/>如果您指定「多邊形」，區域的維度將以點表示。 <br/> |
| 片段 |中繼資料會被分成稱為「片段」的不同區段。 每個片段皆包含開始、持續時間、間隔數字及事件。 沒有事件的片段，代表在該片段的開始時間和持續時間之間沒有偵測到任何動作。 |
| 括號 [] |每個括號皆代表事件中的單一間隔。 如果間隔顯示空白括號，便代表沒有偵測到動作。 |
| 位置 |這是位在事件下的新項目，它能列出動作的發生位置。 它比偵測區域更明確。 |

下列 JSON 範例顯示輸出：

```json
    {
      "version": 2,
      "timescale": 23976,
      "offset": 0,
      "framerate": 24,
      "width": 1280,
      "height": 720,
      "regions": [
        {
          "id": 0,
          "type": "polygon",
          "points": [{'x': 0, 'y': 0},
            {'x': 0.5, 'y': 0},
            {'x': 0, 'y': 1}]
        }
      ],
      "fragments": [
        {
          "start": 0,
          "duration": 226765
        },
        {
          "start": 226765,
          "duration": 47952,
          "interval": 999,
          "events": [
            [
              {
                "type": 2,
                "typeName": "motion",
                "locations": [
                  {
                    "x": 0.004184,
                    "y": 0.007463,
                    "width": 0.991667,
                    "height": 0.985185
                  }
                ],
                "regionId": 0
              }
            ],
```

## <a name="limitations"></a>限制
* 支援的輸入影片格式包括 MP4、MOV 及 WMV。
* 動作偵測已針對固定背景的影片最佳化。 演算法會專注在降低誤判之上，例如光源變化和陰影。
* 某些動作可能會因技術挑戰而無法偵測，例如夜視影片、半透明物件及小型物件。

## <a name="net-sample-code"></a>.NET 範例程式碼

下列程式將示範如何：

1. 建立資產並將媒體檔案上傳到資產。
2. 根據包含下列 JSON 預設值的設定檔案，建立執行影片動作偵測工作的作業： 
   
    ```json
            {
            "Version": "1.0",
            "Options": {
                "SensitivityLevel": "medium",
                "FrameSamplingValue": 1,
                "DetectLightChange": "False",
                "MergeTimeThreshold":
                "00:00:02",
                "DetectionZones": [
                [
                    {"x": 0, "y": 0},
                    {"x": 0.5, "y": 0},
                    {"x": 0, "y": 1}
                ],
                [
                    {"x": 0.3, "y": 0.3},
                    {"x": 0.55, "y": 0.3},
                    {"x": 0.8, "y": 0.3},
                    {"x": 0.8, "y": 0.55},
                    {"x": 0.8, "y": 0.8},
                    {"x": 0.55, "y": 0.8},
                    {"x": 0.3, "y": 0.8},
                    {"x": 0.3, "y": 0.55}
                ]
                ]
            }
            }
    ```

3. 下載輸出 JSON 檔案。 

#### <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 

#### <a name="example"></a>範例

```csharp

using System;
using System.Configuration;
using System.IO;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;
using System.Threading.Tasks;

namespace VideoMotionDetection
{
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

        // Field for service context.
        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Run the VideoMotionDetection job.
            var asset = RunVideoMotionDetectionJob(@"C:\supportFiles\VideoMotionDetection\BigBuckBunny.mp4",
                                        @"C:\supportFiles\VideoMotionDetection\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\VideoMotionDetection\Output");
        }

        static IAsset RunVideoMotionDetectionJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                "My Video Motion Detection Input Asset",
                AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My Video Motion Detection Job");

            // Get a reference to Azure Media Motion Detector.
            string MediaProcessorName = "Azure Media Motion Detector";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My Video Motion Detection Task",
                processor,
                configuration,
                TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My Video Motion Detectoion Output Asset", AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish.
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);

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
                return null;
            }

            return job.OutputMediaAssets[0];
        }

        static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
        {
            IAsset asset = _context.Assets.Create(assetName, options);

            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);

            return asset;
        }

        static void DownloadAsset(IAsset asset, string outputDirectory)
        {
            foreach (IAssetFile file in asset.AssetFiles)
            {
                file.Download(Path.Combine(outputDirectory, file.Name));
            }
        }

        static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors
                .Where(p => p.Name == mediaProcessorName)
                .ToList()
                .OrderBy(p => new Version(p.Version))
                .LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor",
                                                           mediaProcessorName));

            return processor;
        }

        static private void StateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);

            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished.");
                    Console.WriteLine();
                    break;
                case JobState.Canceling:
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    Console.WriteLine("Please wait...\n");
                    break;
                case JobState.Canceled:
                case JobState.Error:
                    // Cast sender as a job.
                    IJob job = (IJob)sender;
                    // Display or log error details as needed.
                    // LogJobStop(job.Id);
                    break;
                default:
                    break;
            }
        }
    }
}
```

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>相關連結
[Azure 媒體服務動作偵測器部落格](https://azure.microsoft.com/blog/motion-detector-update/)

[Azure 媒體服務分析概觀](media-services-analytics-overview.md)

[Azure 媒體分析示範](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

