---
title: 使用 Azure 媒體分析 OCR 將文字數位化 | Microsoft Docs
description: Azure 媒體分析 OCR (光學字元辨識) 可讓您將視訊檔中的文字內容轉換成可編輯、可搜尋的數位文字。  這可讓您從媒體的視訊訊號自動擷取有意義的中繼資料。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 307c196e-3a50-4f4b-b982-51585448ffc6
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: 4a7a31b4e0069d2c94a4f109248d7b02c0b03faa
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785897"
---
# <a name="use-azure-media-analytics-to-convert-text-content-in-video-files-into-digital-text"></a>使用 Azure 媒體分析以將視訊檔案中的文字內容轉換為數位文字
## <a name="overview"></a>概觀
如果您需要擷取視訊檔案的文字內容，並產生可編輯、可搜尋的數位文字，您應該使用 Azure 媒體分析 OCR (光學字元辨識)。 此 Azure 媒體處理器會偵測視訊檔案的文字內容並產生文字檔案，以供您使用。 OCR 可讓您從媒體的視訊訊號自動擷取有意義的中繼資料。

搭配搜尋引擎使用時，您可以輕易地依文字編製媒體的索引，並增強探索內容的能力。 這在具有大量文字的視訊 (例如視訊錄製或投影片簡報的螢幕擷取) 中非常實用。 Azure OCR 媒體處理器已針對數位文字進行最佳化。

**Azure 媒體 OCR** 媒體處理器目前為預覽版。

本文章提供有關 **Azure 媒體 OCR** 的詳細資料，並示範如何搭配適用於 .NET 的媒體服務 SDK 來使用它。 如需詳細資訊和範例，請參閱[此部落格](https://azure.microsoft.com/blog/announcing-video-ocr-public-preview-new-config/)。

## <a name="ocr-input-files"></a>OCR 輸入檔案
影片檔案。 目前支援下列格式：MP4、MOV 及 WMV。

## <a name="task-configuration"></a>工作組態
工作組態 (預設)。 使用 **Azure 媒體 OCR** 建立工作時，您必須使用 JSON 或 XML 來指定組態預設。 

>[!NOTE]
>OCR 引擎只會接受高度/寬度兩者在最小 40 像素到最大 32000 像素的影像區域為有效的輸入。
>

### <a name="attribute-descriptions"></a>屬性描述
| 屬性名稱 | 說明 |
| --- | --- |
|AdvancedOutput| 如果您將 AdvancedOutput 設為 true，JSON 輸出就會包含每一個文字的位置資料 (除了片語和區域)。 如果您不想要查看這些詳細資料，請將旗標設定為 false。 預設值為 False。 如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/azure-media-ocr-simplified-output/)。|
| 語言 |(選擇性) 說明要尋找的文字語言。 下列其中一種︰AutoDetect (預設值)、Arabic、ChineseSimplified、ChineseTraditional、Czech Danish、Dutch、English、Finnish、French、German、Greek、Hungarian、Italian、Japanese、Korean、Norwegian、Polish、Portuguese、Romanian、Russian、SerbianCyrillic、SerbianLatin、Slovak、Spanish、Swedish、Turkish。 |
| TextOrientation |(選擇性) 說明要尋找的文字方向。  "Left" 表示所有字母頂端都會指向左邊。  預設文字 (像是可在書本中找到的文字) 的方向為 "Up"。  下列其中一種︰AutoDetect (預設值)、Up、Right、Down、Left。 |
| TimeInterval |(選擇性) 說明取樣率。  預設值為每 1/2 秒。<br/>JSON 格式 – HH:mm:ss.SSS (預設值 00:00:00.500)<br/>XML 格式 – W3C XSD 持續時間基本型別 (預設值 PT0.5) |
| DetectRegions |(選擇性) DetectRegion 物件的陣列，指定在其中偵測文字的視訊畫面格內的區域。<br/>DetectRegion 物件是由下列四個整數值組成︰<br/>左 – 像素的左邊界<br/>上 – 像素的上邊界<br/>寬度 – 以像素為單位的區域寬度<br/>高度 – 以像素為單位的區域高度 |

#### <a name="json-preset-example"></a>JSON 預設範例

```json
    {
        "Version":1.0, 
        "Options": 
        {
            "AdvancedOutput":"true",
            "Language":"English", 
            "TimeInterval":"00:00:01.5",
            "TextOrientation":"Up",
            "DetectRegions": [
                    {
                       "Left": 10,
                       "Top": 10,
                       "Width": 100,
                       "Height": 50
                    }
             ]
        }
    }
```

#### <a name="xml-preset-example"></a>XML 預設範例

```xml
    <?xml version=""1.0"" encoding=""utf-16""?>
    <VideoOcrPreset xmlns:xsi=""http://www.w3.org/2001/XMLSchema-instance"" xmlns:xsd=""http://www.w3.org/2001/XMLSchema"" Version=""1.0"" xmlns=""http://www.windowsazure.com/media/encoding/Preset/2014/03"">
      <Options>
         <AdvancedOutput>true</AdvancedOutput>
         <Language>English</Language>
         <TimeInterval>PT1.5S</TimeInterval>
         <DetectRegions>
             <DetectRegion>
                   <Left>10</Left>
                   <Top>10</Top>
                   <Width>100</Width>
                   <Height>50</Height>
            </DetectRegion>
       </DetectRegions>
       <TextOrientation>Up</TextOrientation>
      </Options>
    </VideoOcrPreset>
```

## <a name="ocr-output-files"></a>OCR 輸出檔案
OCR 媒體處理器的輸出是 JSON 檔案。

### <a name="elements-of-the-output-json-file"></a>輸出 JSON 檔案的元素
視訊 OCR 輸出會在可於視訊上找到的字元中提供時間分段資料。  您可以使用屬性 (例如語言或方向)，確實琢磨您有興趣進行分析的文字。 

輸出包含下列屬性：

| 元素 | 說明 |
| --- | --- |
| 時幅 |影片每秒的「刻度」數目 |
| Offset |時間戳記的時間位移。 在版本 1.0 的影片 API 中，這永遠會是 0。 |
| 畫面播放速率 |影片的每秒畫面格數 |
| width |視訊寬度 (以像素為單位) |
| height |視訊高度 (以像素為單位) |
| 片段 |在視訊中，將中繼資料切割為以時間為基礎的區塊陣列 |
| start |片段的開始時間 (以「刻度」為單位) |
| duration |片段的長度 (以「刻度」為單位) |
| interval |指定片段內每個事件的間隔 |
| events |包含區域的陣列 |
| region |物件，代表偵測到的單字或片語 |
| 語言 |區域內偵測到的文字語言 |
| orientation |區域內偵測到的文字方向 |
| lines |區域內偵測到的文字行陣列 |
| text |實際的文字 |

### <a name="json-output-example"></a>JSON 輸出範例
下列輸出範例包含一般視訊資訊和數個視訊片段。 每個視訊片段都包含 OCR MP 使用語言及其文字方向偵測到的每個區域。 區域也包含這個區域中的每個文字行，以及該行的文字、該行的位置和該行中每個單字的資訊 (單字內容、位置和信賴度)。 以下是範例，而我在其中放入了一些註解。

```json
    {
        "version": 1, 
        "timescale": 90000, 
        "offset": 0, 
        "framerate": 30, 
        "width": 640, 
        "height": 480,  // general video information
        "fragments": [
            {
                "start": 0, 
                "duration": 180000, 
                "interval": 90000,  // the time information about this fragment
                "events": [
                    [
                       { 
                            "region": { // the detected region array in this fragment 
                                "language": "English",  // region language
                                "orientation": "Up",  // text orientation
                                "lines": [  // line information array in this region, including the text and the position
                                    {
                                        "text": "One Two", 
                                        "left": 10, 
                                        "top": 10, 
                                        "right": 210, 
                                        "bottom": 110, 
                                        "word": [  // word information array in this line
                                            {
                                                "text": "One", 
                                                "left": 10, 
                                                "top": 10, 
                                                "right": 110, 
                                                "bottom": 110, 
                                                "confidence": 900
                                            }, 
                                            {
                                                "text": "Two", 
                                                "left": 110, 
                                                "top": 10, 
                                                "right": 210, 
                                                "bottom": 110, 
                                                "confidence": 910
                                            }
                                        ]
                                    }
                                ]
                            }
                        }
                    ]
                ]
            }
        ]
    }
```

## <a name="net-sample-code"></a>.NET 範例程式碼

下列程式將示範如何：

1. 建立資產並將媒體檔案上傳到資產。
2. 使用 OCR 設定/預設檔案建立作業。
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

namespace OCR
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

            // Run the OCR job.
            var asset = RunOCRJob(@"C:\supportFiles\OCR\presentation.mp4",
                                        @"C:\supportFiles\OCR\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\OCR\Output");
        }

        static IAsset RunOCRJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                "My OCR Input Asset",
                AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My OCR Job");

            // Get a reference to Azure Media OCR.
            string MediaProcessorName = "Azure Media OCR";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My OCR Task",
                processor,
                configuration,
                TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My OCR Output Asset", AssetCreationOptions.None);

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

