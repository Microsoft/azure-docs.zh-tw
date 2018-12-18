---
title: 如何使用 Azure 媒體服務執行即時串流，以使用 .NET 建立多位元速率串流 | Microsoft Docs
description: 本教學課程將逐步引導您使用 .NET SDK 建立通道，以接收單一位元速率即時串流，並將其編碼為多位元速率串流。
services: media-services
documentationcenter: ''
author: anilmur
manager: cfowler
editor: ''
ms.assetid: 4df5e690-ff63-47cc-879b-9c57cb8ec240
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 12/09/2017
ms.author: juliako;anilmur
ms.openlocfilehash: 946de42c499182f8785866f362677779b105760d
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33940818"
---
# <a name="how-to-perform-live-streaming-using-azure-media-services-to-create-multi-bitrate-streams-with-net"></a>如何使用 Azure 媒體服務執行即時串流，以使用 .NET 建立多位元速率串流
> [!div class="op_single_selector"]
> * [入口網站](media-services-portal-creating-live-encoder-enabled-channel.md)
> * [.NET](media-services-dotnet-creating-live-encoder-enabled-channel.md)
> * [REST API](https://docs.microsoft.com/rest/api/media/operations/channel)
> 
> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)。
> 
> 

## <a name="overview"></a>概觀
本教學課程將逐步引導您建立 **通道** ，可接收單一位元速率的即時串流，並將其編碼為多位元速率串流。

如需為即時編碼啟用之通道相關的詳細概念資訊，請參閱 [使用 Azure 媒體服務的即時串流，以建立多位元速率串流](media-services-manage-live-encoder-enabled-channels.md)。

## <a name="common-live-streaming-scenario"></a>常見即時串流案例
下列步驟描述當我們建立一般即時資料流應用程式時，會涉及到的各種工作。

> [!NOTE]
> 目前，即時事件的最大建議持續時間是 8 小時。 如果您需要較長的時間來執行通道，請連絡 amslived@microsoft.com。
> 
> 

1. 將攝影機連接到電腦。 啟動和設定可使用下列其中一種通訊協定輸出單一位元速率串流的內部部署即時編碼器：RTMP 或 Smooth Streaming。 如需詳細資訊，請參閱 [Azure 媒體服務 RTMP 支援和即時編碼器](http://go.microsoft.com/fwlink/?LinkId=532824)。

    此步驟也可以在您建立通道之後執行。

2. 建立並啟動通道。
3. 擷取通道內嵌 URL。

    內嵌 URL 可供即時編碼器用來傳送串流到通道。

4. 擷取通道預覽 URL。

    使用此 URL 來確認您的通道會正確接收即時串流。

5. 建立資產。
6. 如果您想要在播放期間動態加密資產，請執行下列動作：
7. 建立內容金鑰。
8. 設定內容金鑰的授權原則。
9. 設定資產傳遞原則 (供動態封裝和動態加密使用)。
10. 建立程式，並指定使用您所建立的資產。
11. 藉由建立 OnDemand 定位器，發行與程式相關聯的資產。

    >[!NOTE]
    >建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 您想要串流內容的串流端點必須處於 [執行中] 狀態。 

12. 當您準備好開始串流和封存時，請啟動程式。
13. 即時編碼器會收到啟動公告的信號 (選擇性)。 公告會插入輸出串流中。
14. 每當您想要停止串流處理和封存事件時，請停止程式。
15. 刪除程式 (並選擇性地刪除資產)。

## <a name="what-youll-learn"></a>您將學到什麼
本文示範如何使用媒體服務 .NET SDK 在通道上執行不同的作業和程式。 因為許多作業會長時間執行，所以會使用管理長時間執行作業的 .NET API。

本文示範如何執行下列動作：

1. 建立並啟動通道。 使用長時間執行的 API。
2. 取得通道內嵌 (輸入) 端點。 此端點應該提供給可以傳送單一位元速率即時串流的編碼器。
3. 取得預覽端點。 此端點可用來預覽您的串流。
4. 建立用來儲存內容的資產。 資產傳遞原則也應該另外設定，如此範例中所示。
5. 建立程式，並指定使用稍早建立的資產。 啟動程式。 使用長時間執行的 API。
6. 建立資產的定位器，讓內容發行，並且可以串流至用戶端。
7. 顯示和隱藏 slate。 啟動和停止公告。 使用長時間執行的 API。
8. 清除您的通道和所有相關聯的資源。

## <a name="prerequisites"></a>先決條件
需要有下列項目，才能完成教學課程。

* 一個 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)。 您將獲得能用來試用 Azure 付費服務的額度。 即使在額度用完後，您仍可保留帳戶，並使用免費的 Azure 服務和功能，例如 Azure App Service 中的 Web Apps 功能。
* 媒體服務帳戶。 若要建立媒體服務帳戶，請參閱 [建立帳戶](media-services-portal-create-account.md)。
* Visual Studio 2010 SP1 (Professional、Premium、Ultimate 或 Express) 或較新版本。
* 您必須使用媒體服務 .NET SDK 3.2.0.0 版或更新版本。
* 網路攝影機以及可以傳送單一位元速率即時串流的編碼器。

## <a name="considerations"></a>考量
* 目前，即時事件的最大建議持續時間是 8 小時。 如果您需要較長的時間來執行通道，請連絡 amslived@Microsoft.com。
* 對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限，例如，要長時間維持就地 (非上載原則) 的定位器原則，您應該使用相同的原則識別碼。 如需詳細資訊，請參閱[本篇文章](media-services-dotnet-manage-entities.md#limit-access-policies)。

## <a name="download-sample"></a>下載範例

您可以從[這裡](https://azure.microsoft.com/documentation/samples/media-services-dotnet-encode-live-stream-with-ams-clear/)下載本文所述的範例。

## <a name="set-up-for-development-with-media-services-sdk-for-net"></a>設定使用媒體服務 SDK for.NET 的開發

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 

## <a name="code-example"></a>程式碼範例

```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using System.IO;
using System.Linq;
using System.Net;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;

namespace EncodeLiveStreamWithAmsClear
{
    class Program
    {
        private const string ChannelName = "channel001";
        private const string AssetlName = "asset001";
        private const string ProgramlName = "program001";

        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            IChannel channel = CreateAndStartChannel();

            // The channel's input endpoint:
            string ingestUrl = channel.Input.Endpoints.FirstOrDefault().Url.ToString();

            Console.WriteLine("Intest URL: {0}", ingestUrl);


            // Use the previewEndpoint to preview and verify 
            // that the input from the encoder is actually reaching the Channel. 
            string previewEndpoint = channel.Preview.Endpoints.FirstOrDefault().Url.ToString();

            Console.WriteLine("Preview URL: {0}", previewEndpoint);

            // When Live Encoding is enabled, you can now get a preview of the live feed as it reaches the Channel. 
            // This can be a valuable tool to check whether your live feed is actually reaching the Channel. 
            // The thumbnail is exposed via the same end-point as the Channel Preview URL.
            string thumbnailUri = new UriBuilder
            {
                Scheme = Uri.UriSchemeHttps,
                Host = channel.Preview.Endpoints.FirstOrDefault().Url.Host,
                Path = "thumbnails/input.jpg"
            }.Uri.ToString();

            Console.WriteLine("Thumbain URL: {0}", thumbnailUri);

            // Once you previewed your stream and verified that it is flowing into your Channel, 
            // you can create an event by creating an Asset, Program, and Streaming Locator. 
            IAsset asset = CreateAndConfigureAsset();

            IProgram program = CreateAndStartProgram(channel, asset);

            ILocator locator = CreateLocatorForAsset(program.Asset, program.ArchiveWindowLength);

            // You can use slates and ads only if the channel type is Standard.  
            StartStopAdsSlates(channel);

            // Once you are done streaming, clean up your resources.
            Cleanup(channel);
        }

        public static IChannel CreateAndStartChannel()
        {
            var channelInput = CreateChannelInput();
            var channePreview = CreateChannelPreview();
            var channelEncoding = CreateChannelEncoding();

            ChannelCreationOptions options = new ChannelCreationOptions
            {
                EncodingType = ChannelEncodingType.Standard,
                Name = ChannelName,
                Input = channelInput,
                Preview = channePreview,
                Encoding = channelEncoding
            };

            Log("Creating channel");
            IOperation channelCreateOperation = _context.Channels.SendCreateOperation(options);
            string channelId = TrackOperation(channelCreateOperation, "Channel create");

            IChannel channel = _context.Channels.Where(c => c.Id == channelId).FirstOrDefault();

            Log("Starting channel");
            var channelStartOperation = channel.SendStartOperation();
            TrackOperation(channelStartOperation, "Channel start");

            return channel;
        }

        /// <summary>
        /// Create channel input, used in channel creation options. 
        /// </summary>
        /// <returns></returns>
        private static ChannelInput CreateChannelInput()
        {
            return new ChannelInput
            {
                StreamingProtocol = StreamingProtocol.FragmentedMP4,
                AccessControl = new ChannelAccessControl
                {
                    IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                        Name = "TestChannelInput001",
                        Address = IPAddress.Parse("0.0.0.0"),
                        SubnetPrefixLength = 0
                        }
                    }
                }
            };
        }

        /// <summary>
        /// Create channel preview, used in channel creation options. 
        /// </summary>
        /// <returns></returns>
        private static ChannelPreview CreateChannelPreview()
        {
            return new ChannelPreview
            {
                AccessControl = new ChannelAccessControl
                {
                    IPAllowList = new List<IPRange>
                {
                    new IPRange
                    {
                    Name = "TestChannelPreview001",
                    Address = IPAddress.Parse("0.0.0.0"),
                    SubnetPrefixLength = 0
                    }
                }
                }
            };
        }

        /// <summary>
        /// Create channel encoding, used in channel creation options. 
        /// </summary>
        /// <returns></returns>
        private static ChannelEncoding CreateChannelEncoding()
        {
            return new ChannelEncoding
            {
                SystemPreset = "Default720p",
                IgnoreCea708ClosedCaptions = false,
                AdMarkerSource = AdMarkerSource.Api
            };
        }

        /// <summary>
        /// Create an asset and configure asset delivery policies.
        /// </summary>
        /// <returns></returns>
        public static IAsset CreateAndConfigureAsset()
        {
            IAsset asset = _context.Assets.Create(AssetlName, AssetCreationOptions.None);

            IAssetDeliveryPolicy policy =
            _context.AssetDeliveryPolicies.Create("Clear Policy",
            AssetDeliveryPolicyType.NoDynamicEncryption,
            AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.SmoothStreaming | AssetDeliveryProtocol.Dash, null);

            asset.DeliveryPolicies.Add(policy);

            return asset;
        }

        /// <summary>
        /// Create a Program on the Channel. You can have multiple Programs that overlap or are sequential;
        /// however each Program must have a unique name within your Media Services account.
        /// </summary>
        /// <param name="channel"></param>
        /// <param name="asset"></param>
        /// <returns></returns>
        public static IProgram CreateAndStartProgram(IChannel channel, IAsset asset)
        {
            IProgram program = channel.Programs.Create(ProgramlName, TimeSpan.FromHours(3), asset.Id);
            Log("Program created", program.Id);

            Log("Starting program");
            var programStartOperation = program.SendStartOperation();
            TrackOperation(programStartOperation, "Program start");

            return program;
        }

        /// <summary>
        /// Create locators in order to be able to publish and stream the video.
        /// </summary>
        /// <param name="asset"></param>
        /// <param name="ArchiveWindowLength"></param>
        /// <returns></returns>
        public static ILocator CreateLocatorForAsset(IAsset asset, TimeSpan ArchiveWindowLength)
        {
            // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.            
            var locator = _context.Locators.CreateLocator
            (
                LocatorType.OnDemandOrigin,
                asset,
                _context.AccessPolicies.Create
                (
                    "Live Stream Policy",
                    ArchiveWindowLength,
                    AccessPermissions.Read
                )
            );

            return locator;
        }

        /// <summary>
        /// Perform operations on slates.
        /// </summary>
        /// <param name="channel"></param>
        public static void StartStopAdsSlates(IChannel channel)
        {
            int cueId = new Random().Next(int.MaxValue);
            var path = Path.GetFullPath(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"..\\..\\SlateJPG\\DefaultAzurePortalSlate.jpg"));

            Log("Creating asset");
            var slateAsset = _context.Assets.Create("Slate test asset " + DateTime.Now.ToString("yyyy-MM-dd HH-mm"), AssetCreationOptions.None);
            Log("Slate asset created", slateAsset.Id);

            Log("Uploading file");
            var assetFile = slateAsset.AssetFiles.Create("DefaultAzurePortalSlate.jpg");
            assetFile.Upload(path);
            assetFile.IsPrimary = true;
            assetFile.Update();

            Log("Showing slate");
            var showSlateOpeartion = channel.SendShowSlateOperation(TimeSpan.FromMinutes(1), slateAsset.Id);
            TrackOperation(showSlateOpeartion, "Show slate");

            Log("Hiding slate");
            var hideSlateOperation = channel.SendHideSlateOperation();
            TrackOperation(hideSlateOperation, "Hide slate");

            Log("Starting ad");
            var startAdOperation = channel.SendStartAdvertisementOperation(TimeSpan.FromMinutes(1), cueId, false);
            TrackOperation(startAdOperation, "Start ad");

            Log("Ending ad");
            var endAdOperation = channel.SendEndAdvertisementOperation(cueId);
            TrackOperation(endAdOperation, "End ad");

            Log("Deleting slate asset");
            slateAsset.Delete();
        }

        /// <summary>
        /// Clean up resources associated with the channel.
        /// </summary>
        /// <param name="channel"></param>
        public static void Cleanup(IChannel channel)
        {
            IAsset asset;
            if (channel != null)
            {
                foreach (var program in channel.Programs)
                {
                    asset = _context.Assets.Where(se => se.Id == program.AssetId)
                                .FirstOrDefault();

                    Log("Stopping program");
                    var programStopOperation = program.SendStopOperation();
                    TrackOperation(programStopOperation, "Program stop");

                    program.Delete();

                    if (asset != null)
                    {
                        Log("Deleting locators");
                        foreach (var l in asset.Locators)
                            l.Delete();

                        Log("Deleting asset");
                        asset.Delete();
                    }
                }

                Log("Stopping channel");
                var channelStopOperation = channel.SendStopOperation();
                TrackOperation(channelStopOperation, "Channel stop");

                Log("Deleting channel");
                var channelDeleteOperation = channel.SendDeleteOperation();
                TrackOperation(channelDeleteOperation, "Channel delete");
            }
        }

        /// <summary>
        /// Track long running operations.
        /// </summary>
        /// <param name="operation"></param>
        /// <param name="description"></param>
        /// <returns></returns>
        public static string TrackOperation(IOperation operation, string description)
        {
            string entityId = null;
            bool isCompleted = false;

            Log("starting to track ", null, operation.Id);
            while (isCompleted == false)
            {
                operation = _context.Operations.GetOperation(operation.Id);
                isCompleted = IsCompleted(operation, out entityId);
                System.Threading.Thread.Sleep(TimeSpan.FromSeconds(30));
            }
            // If we got here, the operation succeeded.
            Log(description + " in completed", operation.TargetEntityId, operation.Id);

            return entityId;
        }

        /// <summary> 
        /// Checks if the operation has been completed. 
        /// If the operation succeeded, the created entity Id is returned in the out parameter.
        /// </summary> 
        /// <param name="operationId">The operation Id.</param> 
        /// <param name="channel">
        /// If the operation succeeded, 
        /// the entity Id associated with the sucessful operation is returned in the out parameter.</param>
        /// <returns>Returns false if the operation is still in progress; otherwise, true.</returns> 
        private static bool IsCompleted(IOperation operation, out string entityId)
        {
            bool completed = false;

            entityId = null;

            switch (operation.State)
            {
                case OperationState.Failed:
                    // Handle the failure. 
                    // For example, throw an exception. 
                    // Use the following information in the exception: operationId, operation.ErrorMessage.
                    Log("operation failed", operation.TargetEntityId, operation.Id);
                    break;
                case OperationState.Succeeded:
                    completed = true;
                    entityId = operation.TargetEntityId;
                    break;
                case OperationState.InProgress:
                    completed = false;
                    Log("operation in progress", operation.TargetEntityId, operation.Id);
                    break;
            }
            return completed;
        }

        private static void Log(string action, string entityId = null, string operationId = null)
        {
            Console.WriteLine(
            "{0,-21}{1,-51}{2,-51}{3,-51}",
            DateTime.Now.ToString("yyyy'-'MM'-'dd HH':'mm':'ss"),
            action,
            entityId ?? string.Empty,
            operationId ?? string.Empty);
        }
    }
}
```

## <a name="next-step"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]


