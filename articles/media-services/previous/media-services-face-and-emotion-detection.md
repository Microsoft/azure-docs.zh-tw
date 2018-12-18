---
title: 使用 Azure 媒體分析偵測臉部和情緒 | Microsoft Docs
description: 本主題示範如何使用 Azure 媒體分析偵測臉部和情緒。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 5ca4692c-23f1-451d-9d82-cbc8bf0fd707
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2017
ms.author: milanga;juliako;
ms.openlocfilehash: 859e75819f96edd527fceb143faf8357738ce80e
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33784457"
---
# <a name="detect-face-and-emotion-with-azure-media-analytics"></a>使用 Azure 媒體分析偵測臉部和情緒
## <a name="overview"></a>概觀
**Azure 媒體臉部偵測器** 媒體處理器 (MP) 能讓您透過臉部表情針對對象的參與情形做出計算、追蹤移動，甚至進行量測。 此服務包含兩個功能： 

* **臉部偵測**
  
    臉部偵測能夠找出並追蹤影片中的臉部。 可以同時追蹤多個臉部，隨著對象移動持續進行追蹤，並將時間和位置的中繼資料以 JSON 檔案的格式回傳。 追蹤期間，服務會在人員於螢幕上四處移動時，嘗試為他們的臉部賦予相同的識別碼，就算他們被擋住或暫時離開畫面也一樣。
  
  > [!NOTE]
  > 此服務並不會執行臉部辨識。 臉部離開畫面或被擋住太久的人員，將會在回來時被賦予新的識別碼。
  > 
  > 
* **情緒偵測**
  
    「情緒偵測」是臉部偵測媒體處理器的選擇性元件，它會根據已偵測的臉部，傳回多個情緒屬性的分析，包括快樂、悲傷、恐懼、憤怒等等。 

**Azure 媒體臉部偵測器** MP 目前為預覽功能。

本文提供有關 **Azure 媒體臉部偵測器**的詳細資料，並示範如何搭配適用於 .NET 的媒體服務 SDK 來使用它。

## <a name="face-detector-input-files"></a>臉部偵測器輸入檔案
影片檔案。 目前支援下列格式：MP4、MOV 及 WMV。

## <a name="face-detector-output-files"></a>臉部偵測器輸出檔案
臉部偵測器和追蹤 API 能提供高精確度的臉部位置偵測和追蹤功能，並能在單一影片中偵測到最多 64 個人類臉部。 正面的臉部能提供最佳結果，而側面的臉部和較小的臉部 (小於或等於 24x24 像素) 可能就無法取得相同的精確度。

已偵測並追蹤的臉部將會搭配能以像素為單位表示臉部在影像中位置的座標 (左側、上方、寬度和高度)，以及表示對該人員進行追蹤的臉部識別碼。 臉部識別碼很容易在正面臉部長時間於畫面中遺失或重疊的情況下重設，導致某些人員被指派多個識別碼。

## <a id="output_elements"></a>輸出 JSON 檔案的元素

[!INCLUDE [media-services-analytics-output-json](../../../includes/media-services-analytics-output-json.md)]

臉部偵測器使用分散 (中繼資料可以分解為以時間為基礎的區塊，讓您可以只下載需要的部分) 及分割 (可以在事件過於龐大的情況下對事件進行分解) 的技術。 某些簡單的計算可以協助您轉換資料。 例如，如果事件是從 6300 (刻度) 開始，並擁有 2997 (刻度/每秒) 的時幅，以及 29.97 (畫面/每秒) 的畫面播放速率，則：

* 開始/時幅 = 2.1 秒
* 秒數 x 畫面播放速率 = 63 格畫面

## <a name="face-detection-input-and-output-example"></a>臉部偵測輸入和輸出範例
### <a name="input-video"></a>輸入影片
[輸入影片](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

### <a name="task-configuration-preset"></a>工作設定 (預設)
以 **Azure 媒體臉部偵測器**建立工作時，您必須指定設定預設值。 下列設定預設值僅適用於臉部偵測。

```json
    {
      "version":"1.0",
      "options":{
          "TrackingMode": "Fast"
      }
    }
```

#### <a name="attribute-descriptions"></a>屬性描述
| 屬性名稱 | 說明 |
| --- | --- |
| Mode |Fast：較快的處理速度，但較不精確 (預設)。|

### <a name="json-output"></a>JSON 輸出
下列為 JSON 輸出受到截斷的範例。

```json
    {
    "version": 1,
    "timescale": 30000,
    "offset": 0,
    "framerate": 29.97,
    "width": 1280,
    "height": 720,
    "fragments": [
        {
        "start": 0,
        "duration": 60060
        },
        {
        "start": 60060,
        "duration": 60060,
        "interval": 1001,
        "events": [
            [
            {
                "id": 0,
                "x": 0.519531,
                "y": 0.180556,
                "width": 0.0867188,
                "height": 0.154167
            }
            ],
            [
            {
                "id": 0,
                "x": 0.517969,
                "y": 0.181944,
                "width": 0.0867188,
                "height": 0.154167
            }
            ],
            [
            {
                "id": 0,
                "x": 0.517187,
                "y": 0.183333,
                "width": 0.0851562,
                "height": 0.151389
            }
            ],
```


## <a name="emotion-detection-input-and-output-example"></a>情緒偵測輸入和輸出範例
### <a name="input-video"></a>輸入影片
[輸入影片](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

### <a name="task-configuration-preset"></a>工作設定 (預設)
以 **Azure 媒體臉部偵測器**建立工作時，您必須指定設定預設值。 下列設定預設值指定以情緒偵測為基礎建立 JSON。

```json
    {
      "version": "1.0",
      "options": {
        "aggregateEmotionWindowMs": "987",
        "mode": "aggregateEmotion",
        "aggregateEmotionIntervalMs": "342"
      }
    }
```


#### <a name="attribute-descriptions"></a>屬性描述
| 屬性名稱 | 說明 |
| --- | --- |
| Mode |Faces：僅臉部偵測。<br/>PerFaceEmotion：將每個臉部偵測的情緒單獨傳回。<br/>AggregateEmotion：傳回該畫面中所有臉部的平均情緒值。 |
| AggregateEmotionWindowMs |在已選取 AggregateEmotion 模式時使用。 指定要用來產生每個彙總結果之影片的長度，以毫秒為單位。 |
| AggregateEmotionIntervalMs |在已選取 AggregateEmotion 模式時使用。 指定產生彙總結果的頻率。 |

#### <a name="aggregate-defaults"></a>彙總預設值
以下為彙總窗口和間隔設定的建議值。 AggregateEmotionWindowMs 應該超過 AggregateEmotionIntervalMs。

|| 預設 | 最大 | 最小 |
|--- | --- | --- | --- |
| AggregateEmotionWindowMs |0.5 |2 |0.25|
| AggregateEmotionIntervalMs |0.5 |1 |0.25|

### <a name="json-output"></a>JSON 輸出
彙總情緒的 JSON 輸出 (已截斷)：

```json
    {
     "version": 1,
     "timescale": 30000,
     "offset": 0,
     "framerate": 29.97,
     "width": 1280,
     "height": 720,
     "fragments": [
       {
         "start": 0,
         "duration": 60060,
         "interval": 15015,
         "events": [
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               }
             }
           ]
         ]
       },
       {
         "start": 60060,
         "duration": 60060,
         "interval": 15015,
         "events": [
           [
             {
               "windowFaceDistribution": {
                 "neutral": 1,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
                 "contempt": 0
               },
               "windowMeanScores": {
                 "neutral": 0.688541,
                 "happiness": 0.0586323,
                 "surprise": 0.227184,
                 "sadness": 0.00945675,
                 "anger": 0.00592107,
                 "disgust": 0.00154993,
                 "fear": 0.00450447,
                 "contempt": 0.0042109
               }
             }
           ],
           [
             {
               "windowFaceDistribution": {
                 "neutral": 1,
                 "happiness": 0,
                 "surprise": 0,
                 "sadness": 0,
                 "anger": 0,
                 "disgust": 0,
                 "fear": 0,
```

## <a name="limitations"></a>限制
* 支援的輸入影片格式包括 MP4、MOV 及 WMV。
* 可偵測的臉部大小範圍是 24x24 到 2048x2048 像素。 位於此範圍之外的臉部將無法被偵測。
* 針對每個影片，傳回臉部的最大數目為 64。
* 某些臉部可能因技術挑戰而無法偵測，例如非常大的臉部角度 (頭部姿勢)，以及大型的阻擋物。 正面和接近正面的臉部能提供最佳結果。

## <a name="net-sample-code"></a>.NET 範例程式碼

下列程式將示範如何：

1. 建立資產並將媒體檔案上傳到資產。
2. 根據包含下列 JSON 預設值的組態檔，建立帶有臉部偵測工作的作業： 

    ```json
            {
                "version": "1.0"
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

namespace FaceDetection
{
    class Program
    {
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

            // Run the FaceDetection job.
            var asset = RunFaceDetectionJob(@"C:\supportFiles\FaceDetection\BigBuckBunny.mp4",
                                        @"C:\supportFiles\FaceDetection\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\FaceDetection\Output");
        }

        static IAsset RunFaceDetectionJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                "My Face Detection Input Asset",
                AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My Face Detection Job");

            // Get a reference to Azure Media Face Detector.
            string MediaProcessorName = "Azure Media Face Detector";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My Face Detection Task",
                processor,
                configuration,
                TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My Face Detectoion Output Asset", AssetCreationOptions.None);

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
[Azure 媒體服務分析概觀](media-services-analytics-overview.md)

[Azure 媒體分析示範](http://amslabs.azurewebsites.net/demos/Analytics.html)

