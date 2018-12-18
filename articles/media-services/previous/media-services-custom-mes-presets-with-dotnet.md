---
title: 自訂媒體編碼器標準預設 | Microsoft Docs
description: 本主題說明如何透過自訂 Media Encoder Standard 工作預設值執行進階編碼。 本主題說明如何使用媒體服務 .NET SDK 建立編碼工作與作業。 本主題也會說明如何提供自訂預設值給編碼作業。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: ec95392f-d34a-4c22-a6df-5274eaac445b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: 4bdfdb5bd5362d5a8039ca31d498d122843df2a7
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783067"
---
# <a name="customizing-media-encoder-standard-presets"></a>自訂媒體編碼器標準預設

## <a name="overview"></a>概觀

本文章說明如何透過使用自訂預設的媒體編碼器標準 (MES) 執行進階編碼。 本文章使用 .NET 建立編碼工作與執行此工作的作業。  

本文章說明如何採取 [H264 多重位元速率 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) 預設值來自訂預設，並減少圖層數目。 [自訂媒體編碼器標準預設](media-services-advanced-encoding-with-mes.md)文章示範可用於執行進階編碼工作的自訂預設。

## <a id="customizing_presets"></a> 自訂 MES 預設值

### <a name="original-preset"></a>原始預設

將 [H264 多重位元速率 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) 文章中定義的 JSON 儲存於一些副檔名為 .json 的檔案中。 例如，**CustomPreset_JSON.json**。

### <a name="customized-preset"></a>自訂的預設

開啟 **CustomPreset_JSON.json** 檔案，並從 **H264Layers** 移除前三層，您的檔案看起來就會像這樣。

```json 
    {  
      "Version": 1.0,  
      "Codecs": [  
        {  
          "KeyFrameInterval": "00:00:02",  
          "H264Layers": [  
            {  
              "Profile": "Auto",  
              "Level": "auto",  
              "Bitrate": 1000,  
              "MaxBitrate": 1000,  
              "BufferWindow": "00:00:05",  
              "Width": 640,  
              "Height": 360,  
              "BFrames": 3,  
              "ReferenceFrames": 3,  
              "AdaptiveBFrame": true,  
              "Type": "H264Layer",  
              "FrameRate": "0/1"  
            },  
            {  
              "Profile": "Auto",  
              "Level": "auto",  
              "Bitrate": 650,  
              "MaxBitrate": 650,  
              "BufferWindow": "00:00:05",  
              "Width": 640,  
              "Height": 360,  
              "BFrames": 3,  
              "ReferenceFrames": 3,  
              "AdaptiveBFrame": true,  
              "Type": "H264Layer",  
              "FrameRate": "0/1"  
            },  
            {  
              "Profile": "Auto",  
              "Level": "auto",  
              "Bitrate": 400,  
              "MaxBitrate": 400,  
              "BufferWindow": "00:00:05",  
              "Width": 320,  
              "Height": 180,  
              "BFrames": 3,  
              "ReferenceFrames": 3,  
              "AdaptiveBFrame": true,  
              "Type": "H264Layer",  
              "FrameRate": "0/1"  
            }  
          ],  
          "Type": "H264Video"  
        },  
        {  
          "Profile": "AACLC",  
          "Channels": 2,  
          "SamplingRate": 48000,  
          "Bitrate": 128,  
          "Type": "AACAudio"  
        }  
      ],  
      "Outputs": [  
        {  
          "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",  
          "Format": {  
            "Type": "MP4Format"  
          }  
        }  
      ]  
    }  
```

## <a id="encoding_with_dotnet"></a>使用媒體服務 .NET SDK 進行編碼

下列程式碼範例使用媒體服務 .NET SDK 執行下列工作：

- 建立編碼工作。
- 取得對 Media Encoder Standard 編碼器的參考
- 載入您在上一節建立的自訂 JSON 預設。 
  
        // Load the JSON from the local file.
        string configuration = File.ReadAllText(fileName);  

- 將編碼工作新增至作業。 
- 指定要編碼的輸入資產。
- 建立包含已編碼資產的輸出資產。
- 加入事件處理常式來檢查工作進度。
- 提交作業。
   
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

namespace CustomizeMESPresests
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

        private static readonly string _mediaFiles =
            Path.GetFullPath(@"../..\Media");

        private static readonly string _singleMP4File =
            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Get an uploaded asset.
            var asset = _context.Assets.FirstOrDefault();

            // Encode and generate the output using custom presets.
            EncodeToAdaptiveBitrateMP4Set(asset);

            Console.ReadLine();
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Media Encoder Standard Job");
            // Get a media processor reference, and pass to it the name of the 
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Load the XML (or JSON) from the local file.
            string configuration = File.ReadAllText("CustomPreset_JSON.json");

            // Create a task
            ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
            processor,
            configuration,
            TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(asset);
            // Add an output asset to contain the results of the job. 
            // This output is specified as AssetCreationOptions.None, which 
            // means the output asset is not encrypted. 
            task.OutputAssets.AddNew("Output asset",
            AssetCreationOptions.None);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }

        private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);
            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
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
                    break;
                default:
                    break;
            }
        }

        private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
            ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

            return processor;
        }

    }
}
```

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[媒體服務編碼概觀](media-services-encode-asset.md)

