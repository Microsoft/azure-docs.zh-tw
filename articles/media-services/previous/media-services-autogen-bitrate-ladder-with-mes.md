---
title: 使用 Azure 媒體編碼器標準自動產生位元速率階梯 | Microsoft Docs
description: 本主題說明如何使用媒體編碼器標準 (MES) 根據輸入解析度和位元速率自動產生位元速率階梯。 永遠不會超過輸入解析度和位元速率。 例如，如果輸入是 720p 3Mbps，則輸出會維持在最多 720p，且速率啟動低於 3Mbps。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/10/2017
ms.author: juliako
ms.openlocfilehash: 80f76f413ec2c267ba8fb93514480e168563f470
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783367"
---
#  <a name="use-azure-media-encoder-standard-to-auto-generate-a-bitrate-ladder"></a>使用 Azure 媒體編碼器標準自動產生位元速率階梯

## <a name="overview"></a>概觀

本文章說明如何使用媒體編碼器標準 (MES) 根據解析度和位元速率自動產生輸入位元速率階梯 (位元速率解析組)。 自動產生的預設值絕對不會超過輸入解析度和位元速率。 例如，如果輸入是 720p 3 Mbps，則輸出會維持在最多 720p，且速率啟動低於 3 Mbps。

### <a name="encoding-for-streaming-only"></a>只針對串流編碼

如果您打算只針對串流編碼來源視訊，則您在建立編碼工作時，應該使用 "Adaptive Streaming" (彈性資料流) 預設值。 當使用 **Adaptive Streaming** (彈性資料流) 預設值時，MES 編碼器會智慧地覆蓋位元速率階梯。 不過，您將無法控制編碼成本，因為服務會決定要使用多少層以及哪種解析度。 由於本文章結尾使用 **Adaptive Streaming** (彈性資料流) 進行編碼的結果，您可以看到 MES 產生的輸出層範例。 輸出資產包含 MP4 檔案，其中的音訊和視訊為非交錯格式。

### <a name="encoding-for-streaming-and-progressive-download"></a>針對串流和漸進式下載編碼

如果您打算針對串流以及漸進式下載要產生的 MP4 檔案編碼，則您在建立編碼工作時，應該使用 "Content Adaptive Multiple Bitrate MP4" (內容自適性多位元速率 MP4) 預設值。 使用 **Content Adaptive Multiple Bitrate MP4** (內容自適性多位元速率 MP4) 預設值時，MES 編碼器會套用與先前相同的編碼邏輯，但現在輸出資產會包含 MP4 檔案，其中的音訊和視訊為交錯格式。 您可以使用這些 MP4 檔案的其中一個 (例如最高位元速率的版本) 作為漸進式下載檔案。

## <a id="encoding_with_dotnet"></a>使用媒體服務 .NET SDK 進行編碼

下列程式碼範例使用媒體服務 .NET SDK 執行下列工作：

- 建立編碼工作。
- 取得對 Media Encoder Standard 編碼器的參考
- 將編碼工作新增至作業，並指定使用**彈性資料流**預設值。 
- 建立包含已編碼資產的輸出資產。
- 加入事件處理常式來檢查工作進度。
- 提交作業。

#### <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 

#### <a name="example"></a>範例

```
using System;
using System.Configuration;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;

namespace AdaptiveStreamingMESPresest
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

            // Get an uploaded asset.
            var asset = _context.Assets.FirstOrDefault();

            // Encode and generate the output using the "Adaptive Streaming" preset.
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

            // Create a task
            ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
            processor,
            "Adaptive Streaming",
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

## <a id="output"></a>輸出

由於編碼與**彈性資料流**預設值，本區段會顯示 MES 所產生的輸出層範例。 

### <a name="example-1"></a>範例 1
高度 "1080" 和畫面播放速率 "29.970" 的來源會產生 6 個視訊層︰

|層次|高度|寬度|位元速率 (kbps)|
|---|---|---|---|
|1|1080|1920|6780|
|2|720|1280|3520|
|3|540|960|2210|
|4|360|640|1150|
|5|270|480|720|
|6|180|320|380|

### <a name="example-2"></a>範例 2
高度 "720" 和畫面播放速率 "23.970" 的來源會產生 5 個視訊層︰

|層次|高度|寬度|位元速率 (kbps)|
|---|---|---|---|
|1|720|1280|2940|
|2|540|960|1850|
|3|360|640|960|
|4|270|480|600|
|5|180|320|320|

### <a name="example-3"></a>範例 3
高度 "360" 和畫面播放速率 "29.970" 的來源會產生 3 個視訊層︰

|層次|高度|寬度|位元速率 (kbps)|
|---|---|---|---|
|1|360|640|700|
|2|270|480|440|
|3|180|320|230|
## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[媒體服務編碼概觀](media-services-encode-asset.md)

