---
title: 使用 .NET 透過媒體編碼器標準為資產編碼 | Microsoft Docs
description: 本文說明如何使用 .NET 來使用媒體編碼器標準為資產編碼。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 03431b64-5518-478a-a1c2-1de345999274
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako;anilmur
ms.openlocfilehash: 2964be45c98350fb1f3c82f25716943493979240
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783447"
---
# <a name="encode-an-asset-with-media-encoder-standard-using-net"></a>使用 .NET 透過 Media Encoder Standard 為資產編碼
編碼工作是媒體服務中最常見的處理作業。 您建立編碼工作以將媒體檔案從一種編碼轉換成另一種編碼。 編碼時，您可以使用媒體服務內建的 Media Encoder。 您也可以使用媒體服務合作夥伴提供的編碼器；第三方編碼器可透過 Azure Marketplace 取得。 

本文說明如何使用 .NET 透過媒體編碼器標準 (MES) 將您的資產編碼。 媒體編碼器標準是使用[這裡](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409)所描述的其中一個編碼器預設來進行設定。

建議一律將來源檔編碼為調適性位元速率 MP4 集，然後使用[動態封裝](media-services-dynamic-packaging-overview.md)將該集合轉換為所要的格式。 

如果您的輸出資產是儲存體加密，必須設定資產傳遞原則。 如需詳細資訊，請參閱[設定資產傳遞原則](media-services-dotnet-configure-asset-delivery-policy.md)。

> [!NOTE]
> MES 會產生一個輸出檔案，其名稱包含輸入檔案名稱的前 32 個字元。 名稱是以預設檔案中指定的名稱為基礎。 例如，"FileName": "{Basename}_{Index}{Extension}"。 {Basename} 會由輸入檔案名稱的前 32 個字元取代。
> 
> 

### <a name="mes-formats"></a>MES 格式
[格式和轉碼器](media-services-media-encoder-standard-formats.md)

### <a name="mes-presets"></a>MES 預設值
媒體編碼器標準是使用[這裡](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409)所描述的其中一個編碼器預設來進行設定。

### <a name="input-and-output-metadata"></a>輸入和輸出中繼資料
如果您使用 MES 為輸入資產 (或資產) 編碼，在該編碼工作完成時，您便能取得輸出資產。 輸出資產包含視訊、音訊、縮圖、資訊清單等等，依據您所使用的編碼預設格式而定。

輸出資產也包含隨附關於輸入資產中繼資料的檔案。 中繼資料 XML 檔案的名稱具備下列格式︰<asset_id>_metadata.xml (例如：41114ad3-eb5e-4c57-8d92-5354e2b7d4a4_metadata.xml)，其中的 <asset_id> 是輸入資產的 AssetId 值。 [這裡](media-services-input-metadata-schema.md)說明了此輸入中繼資料 XML 的結構描述。

輸出資產也包含隨附關於輸出資產中繼資料的檔案。 中繼資料 XML 檔案的名稱具備下列格式︰<source_file_name>_manifest.xml (例如：BigBuckBunny_manifest.xml)。 [這裡](media-services-output-metadata-schema.md)說明了此輸出中繼資料 XML 的結構描述。

如果您想要檢查這兩個中繼資料檔案的任一個，可以建立 SAS 定位器並將檔案下載到您的本機電腦。 您可以找到關於如何建立 SAS 定位器的範例，並且下載使用媒體服務 .NET SDK 擴充功能的檔案。

## <a name="download-sample"></a>下載範例
您可以從[這裡 (英文)](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/) 取得並執行示範如何使用 MES 進行編碼的範例。

## <a name="net-sample-code"></a>.NET 範例程式碼

下列程式碼範例使用媒體服務 .NET SDK 執行下列工作：

* 建立編碼工作。
* 取得對 Media Encoder Standard 編碼器的參考
* 指定要使用[彈性資料流](media-services-autogen-bitrate-ladder-with-mes.md)預設值。 
* 將單一編碼工作加入工作。 
* 指定要編碼的輸入資產。
* 建立包含已編碼資產的輸出資產。
* 加入事件處理常式來檢查工作進度。
* 提交作業。

#### <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 

#### <a name="example"></a>範例 

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;

namespace MediaEncoderStandardSample
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

        private static readonly string _supportFiles =
            Path.GetFullPath(@"../..\Media");

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

            // Create a task with the encoding details, using a string preset.
            // In this case "Adaptive Streaming" preset is used.
            ITask task = job.Tasks.AddNew("My encoding task",
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

## <a name="advanced-encoding-features-to-explore"></a>可探索的進階編碼功能
* [如何產生縮圖](media-services-dotnet-generate-thumbnail-with-mes.md)
* [在編碼期間產生縮圖](media-services-dotnet-generate-thumbnail-with-mes.md#example-of-generating-a-thumbnail-while-encoding)
* [在編碼期間裁剪影片](media-services-crop-video.md)
* [自訂編碼預設](media-services-custom-mes-presets-with-dotnet.md)
* [使用影像在影片上進行重疊或加上浮水印](media-services-advanced-encoding-with-mes.md#overlay)

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟
[如何使用媒體編碼器標準搭配 .NET 產生縮圖](media-services-dotnet-generate-thumbnail-with-mes.md)
[媒體服務編碼概觀](media-services-encode-asset.md)

