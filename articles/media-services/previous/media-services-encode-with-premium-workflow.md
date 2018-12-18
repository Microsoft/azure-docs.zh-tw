---
title: 利用 Media Encoder Premium Workflow 進行進階編碼 | Microsoft Docs
description: 了解如何使用 Media Encoder Premium Workflow 進行編碼。 程式碼範例以 C# 撰寫，並使用 Media Services SDK for .NET。
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
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: 9b341b244d53993699dfc9096a86305def82cad7
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783477"
---
# <a name="advanced-encoding-with-media-encoder-premium-workflow"></a>使用 Media Encoder Premium Workflow 進行進階編碼
> [!NOTE]
> 本文中討論的媒體編碼器高階工作流程媒體處理器無法在中國使用。
>
>

如有進階編碼器的問題，請傳送電子郵件到 mepd@microsoft.com。

## <a name="overview"></a>概觀
Microsoft Azure 媒體服務推出 **Media Encoder Premium Workflow** 媒體處理器。 此處理器為高階隨選工作流程提供先進的編碼功能。

以下主題說明 **Media Encoder Premium Workflow**的相關詳細資料：

* [Media Encoder Premium Workflow 支援的格式](media-services-premium-workflow-encoder-formats.md) – 討論 **Media Encoder Premium Workflow**支援的檔案格式和轉碼器。
* [Azure 隨選媒體編碼器的概觀和比較](media-services-encode-asset.md)比較**媒體編碼器高階工作流程**和**媒體編碼器標準**的編碼功能。

本文示範如何使用 .NET 以**媒體編碼器高階工作流程**進行編碼。

**Media Encoder Premium Workflow** 的編碼工作需要個別的組態檔，這稱為工作流程檔案。 這些檔案具有 .workflow 副檔名，並且使用 [工作流程設計工具](media-services-workflow-designer.md) 工具建立。

您也可以在 [這裡](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows)取得預設的工作流程檔案。 資料夾也包含這些檔案的說明。

工作流程檔案必須上傳至媒體服務帳戶做為資產，而且應將此資產傳遞給編碼工作。

## <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 

## <a name="encoding-example"></a>編碼範例

下列範例示範如何使用 **Media Encoder Premium Workflow**進行編碼。

會執行下列步驟：

1. 建立資產並上傳工作流程檔案。
2. 建立資產並上傳來源媒體檔案。
3. 取得 “Media Encoder Premium Workflow” 媒體處理器。
4. 建立工作 (Job) 和工作 (Task)。

    在大部分情況下，工作的組態字串是空的 (如下列範例)。 在一些進階的案例 (會要求您動態設定執行階段屬性) 情況下，您可以提供 XML 字串給編碼工作。 這類案例的範例包括：建立疊加、平行或循序的媒體編結、顯示字幕。
5. 將兩個輸入資產加入工作 (Task)。

    1. 第一個 – 工作流程資產。
    2. 第二個 – 視訊資產。

    >[!NOTE]
    >新增媒體資產前，必須先將工作流程資產新增至工作 (Task)。
   這項工作 (Task) 的組態字串應該是空的。
   
6. 提交編碼工作 (Job)。

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MediaServices.Client;

namespace MediaEncoderPremiumWorkflowSample
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

        private static readonly string _workflowFilePath =
            Path.GetFullPath(_supportFiles + @"\H264 Progressive Download MP4.workflow");

        private static readonly string _singleMP4InputFilePath =
            Path.GetFullPath(_supportFiles + @"\BigBuckBunny.mp4");

        static void Main(string[] args)
        {

            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            var workflowAsset = CreateAssetAndUploadSingleFile(_workflowFilePath);
            var videoAsset = CreateAssetAndUploadSingleFile(_singleMP4InputFilePath);
            IAsset outputAsset = CreateEncodingJob(workflowAsset, videoAsset);

        }

        static public IAsset CreateAssetAndUploadSingleFile(string singleFilePath)
        {
            var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();
            var asset = _context.Assets.Create(assetName, AssetCreationOptions.None);

            var fileName = Path.GetFileName(singleFilePath);

            var assetFile = asset.AssetFiles.Create(fileName);

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return asset;
        }

        static public IAsset CreateEncodingJob(IAsset workflow, IAsset video)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Premium Workflow encoding job");
            // Get a media processor reference, and pass to it the name of the
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Premium Workflow");

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("Premium Workflow encoding task",
                processor,
                "",
                TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(workflow);
            task.InputAssets.Add(video); // we add one asset
                                         // Add an output asset to contain the results of the job.
                                         // This output is specified as AssetCreationOptions.None, which
                                         // means the output asset is not encrypted.
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new
                    EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish.
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
            progressJobTask.Wait();

            // If job state is Error the event handling
            // method for job progress should log errors.  Here we check
            // for error state and exit if needed.
            if (job.State == JobState.Error)
            {
                throw new Exception("\nExiting method due to job error.");
            }

            return job.OutputMediaAssets[0];
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

如有進階編碼器的問題，請傳送電子郵件到 mepd@microsoft.com。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
