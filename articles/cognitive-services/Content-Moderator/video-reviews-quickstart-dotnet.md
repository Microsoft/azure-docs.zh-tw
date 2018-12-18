---
title: 使用 .NET 來建立影片審核項目 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 如何使用 Content Moderator SDK for .NET 來建立影片審核項目
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 01/18/2018
ms.author: sajagtap
ms.openlocfilehash: 284ee24bbb0a15d107acf85e2d58072a0ecbbc6e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219035"
---
# <a name="create-video-reviews-using-net"></a>使用 .NET 來建立影片審核項目

本文提供資訊和範例程式碼，可協助您[以 C# 快速開始使用 Content Moderator SDK](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 來執行下列操作：

- 建立給人工仲裁者的影片審核項目
- 將畫面新增至審核項目
- 取得審核項目的畫面 
- 取得審核項目的狀態和詳細資料
- 發佈審核項目

## <a name="prerequisites"></a>必要條件

本文假設您已[仲裁影片 (請參閱快速入門)](video-moderation-api.md)，而且具有回應資料。 您需要使用它來為人工仲裁者建立以畫面為基礎的審核項目。

本文同時也假設您已經熟悉 Visual Studio 和 C#。

## <a name="sign-up-for-content-moderator"></a>設定 Content Moderator

您必須有訂用帳戶金鑰，才能透過 REST API 或 SDK 使用 Content Moderator 服務。
請參考[快速入門](quick-start.md)，以了解如何取得金鑰。

## <a name="sign-up-for-a-review-tool-account-if-not-completed-in-the-previous-step"></a>如果未在上一個步驟中完成審核工具帳戶的註冊，請於此時註冊

如果您是從 Azure 入口網站取得 Content Moderator，也請[註冊審核工具帳戶](https://contentmoderator.cognitive.microsoft.com/)，並建立審核小組。 您需要小組識別碼和審核工具才能呼叫審核 API，以在審核工具中啟動作業及檢視審核項目。

## <a name="ensure-your-api-key-can-call-the-review-api-for-review-creation"></a>請確定您的 API 金鑰可呼叫審核 API 以建立審核項目

完成前述步驟後，如果您是從 Azure 入口網站開始作業的，您可能會獲得兩個 Content Moderator 金鑰。 

如果您打算在 SDK 範例中使用 Azure 提供的 API 金鑰，請依照[搭配使用 Azure 金鑰與審核 API](review-tool-user-guide/credentials.md#use-the-azure-account-with-the-review-tool-and-review-api) 一節中說明的步驟操作，以允許應用程式呼叫審核 API 並建立審核項目。

如果您使用審核工具所產生的免費試用版金鑰，則您的審核工具帳戶已知悉金鑰，因此不需執行額外的步驟。

### <a name="prepare-your-video-and-the-video-frames-for-review"></a>準備您的影片和影片畫面以供審核

要審核的影片和範例影片畫面必須在線上發佈，因為您需要它們的 URL。

> [!NOTE]
> 此程式會使用來自隨機成人/猥褻分數影片的手動儲存螢幕擷取畫面，以說明審核 API 的使用方式。 在真實世界情況中，您會使用[影片仲裁輸出](video-moderation-api.md#run-the-program-and-review-the-output)來建立影像及指派分數。 

針對影片，您需要一個串流端點，以便讓審核工具在播放程式檢視中播放影片。

![影片示範縮圖](images/ams-video-demo-view.PNG)

- 複製這個 [Azure 媒體服務示範](https://aka.ms/azuremediaplayer?url=https%3A%2F%2Famssamples.streaming.mediaservices.windows.net%2F91492735-c523-432b-ba01-faba6c2206a2%2FAzureMediaServicesPromo.ism%2Fmanifest)頁面上的 **URL** 作為資訊清單 URL。

針對影片畫面 (影像)，請使用下列影像：

![影片畫面縮圖 1](images/ams-video-frame-thumbnails-1.PNG) | ![影片畫面縮圖 2](images/ams-video-frame-thumbnails-2.PNG) | ![影片畫面縮圖 3](images/ams-video-frame-thumbnails-3.PNG) |
| :---: | :---: | :---: |
[畫面 1](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG) | [畫面 2](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG) | [畫面 3](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG) |

## <a name="create-your-visual-studio-project"></a>建立 Visual Studio 專案

1. 將一個新的 [主控台應用程式 (.NET Framework)] 專案新增到您的解決方案。

1. 將專案命名為 **VideoReviews**。

1. 選取此專案作為解決方案的單一啟始專案。

### <a name="install-required-packages"></a>安裝必要的套件

安裝下列適用於 TermLists 專案的 NuGet 套件。

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Microsoft.Rest.ClientRuntime.Azure
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>更新程式的 using 陳述式

修改程式的 using 陳述式，如下所示。

    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Threading;
    using Microsoft.Azure.CognitiveServices.ContentModerator;
    using Microsoft.CognitiveServices.ContentModerator;
    using Microsoft.CognitiveServices.ContentModerator.Models;
    using Newtonsoft.Json;


### <a name="add-private-properties"></a>新增私有屬性

將下列私有屬性新增至 VideoReviews 命名空間、Program 類別。

依指示取代這些屬性的範例值。


    namespace VideoReviews
    {
        class Program
        {
            // NOTE: Replace this example location with the location for your Content Moderator account.
            /// <summary>
            /// The region/location for your Content Moderator account, 
            /// for example, westus.
            /// </summary>
            private static readonly string AzureRegion = "YOUR CONTENT MODERATOR REGION";

            // NOTE: Replace this example key with a valid subscription key.
            /// <summary>
            /// Your Content Moderator subscription key.
            /// </summary>
            private static readonly string CMSubscriptionKey = "YOUR CONTENT MODERATOR KEY";

            // NOTE: Replace this example team name with your Content Moderator team name.
            /// <summary>
            /// The name of the team to assign the job to.
            /// </summary>
            /// <remarks>This must be the team name you used to create your 
            /// Content Moderator account. You can retrieve your team name from
            /// the Content Moderator web site. Your team name is the Id associated 
            /// with your subscription.</remarks>
            private const string TeamName = "YOUR CONTENT MODERATOR TEAM ID";

            /// <summary>
            /// The base URL fragment for Content Moderator calls.
            /// </summary>
            private static readonly string AzureBaseURL =
                $"{AzureRegion}.api.cognitive.microsoft.com";

            /// <summary>
            /// The minimum amount of time, in milliseconds, to wait between calls
            /// to the Content Moderator APIs.
            /// </summary>
            private const int throttleRate = 2000;


### <a name="create-content-moderator-client-object"></a>建立 Content Moderator 用戶端物件

將下列方法定義新增至 VideoReviews 命名空間、Program 類別。

    /// <summary>
    /// Returns a new Content Moderator client for your subscription.
    /// </summary>
    /// <returns>The new client.</returns>
    /// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
    /// When you have finished using the client,
    /// you should dispose of it either directly or indirectly. </remarks>
    public static ContentModeratorClient NewClient()
    {
        return new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey))
        {
            Endpoint = AzureBaseURL
        };
    }

## <a name="create-a-video-review"></a>建立影片審核項目

使用 **ContentModeratorClient.Reviews.CreateVideoReviews** 來建立影片審核項目。 如需詳細資訊，請參閱 [API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4) \(英文\)。

**CreateVideoReviews** 具有下列必要參數：
1. 一個包含 MIME 類型的字串，這應該是 "application/json"。 
1. 您的 Content Moderator 小組名稱。
1. 一個 **IList<CreateVideoReviewsBodyItem>** 物件。 每個 **CreateVideoReviewsBodyItem** 物件皆代表一個影片審核項目。 這個快速入門會一次建立一個審核項目。

**CreateVideoReviewsBodyItem** 具有數個屬性。 您必須至少設定下列屬性：
- **Content**。 要審核的影片 URL。
- **ContentId**。 要指派給影片審核項目的識別碼。
- **Status**。 請將值設定為 "Unpublished"。 如果您未設定此屬性，則會預設為 "Pending"，意謂著已發佈影片審核項目而正等待人工審核。 發佈影片審核項目之後，您就無法再將影片畫面、文字記錄或文字記錄仲裁結果新增至該審核項目。

> [!NOTE]
> **CreateVideoReviews** 會傳回 IList<string>。 這些字串中每個都包含影片審核項目的識別碼。 這些識別碼是 GUID 且與 **ContentId** 屬性的值不同。 

將下列方法定義新增至 VideoReviews 命名空間、Program 類別。

    /// <summary>
    /// Create a video review. For more information, see the API reference:
    /// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4 
    /// </summary>
    /// <param name="client">The Content Moderator client.</param>
    /// <param name="id">The ID to assign to the video review.</param>
    /// <param name="content">The URL of the video to review.</param>
    /// <returns>The ID of the video review.</returns>
    private static string CreateReview(ContentModeratorClient client, string id, string content)
    {
        Console.WriteLine("Creating a video review.");

        List<CreateVideoReviewsBodyItem> body = new List<CreateVideoReviewsBodyItem>() {
            new CreateVideoReviewsBodyItem
            {
                Content = content,
                ContentId = id,
                /* Note: to create a published review, set the Status to "Pending".
                However, you cannot add video frames or a transcript to a published review. */
                Status = "Unpublished",
            }
        };

        var result = client.Reviews.CreateVideoReviews("application/json", TeamName, body);

        Thread.Sleep(throttleRate);

        // We created only one review.
        return result[0];
    }

> [!NOTE]
> 您的 Content Moderator 服務金鑰會有每秒要求數目 (RPS) 的速率限制。如果超出此限制，SDK 就會擲回錯誤碼為 429 的例外狀況。 
>
> 免費層金鑰有一個 RPS 速率限制。

## <a name="add-video-frames-to-the-video-review"></a>將影片畫面新增至影片審核項目

您可以使用 **ContentModeratorClient.Reviews.AddVideoFrameUrl** (如果影片畫面裝載於線上) 或 **ContentModeratorClient.Reviews.AddVideoFrameStream** (如果影片畫面裝載於本機)，將影片畫面新增至影片審核項目。 本快速入門指南假設您的影片畫面裝載於線上，因此會使用 **AddVideoFrameUrl**。 如需詳細資訊，請參閱 [API 參考](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd) \(英文\)。

**AddVideoFrameUrl** 具有下列必要參數：
1. 一個包含 MIME 類型的字串，這應該是 "application/json"。
1. 您的 Content Moderator 小組名稱。
1. **CreateVideoReviews** 所傳回的影片審核項目識別碼。
1. 一個 **IList<VideoFrameBodyItem>** 物件。 每個 **VideoFrameBodyItem** 物件皆代表一個影片審核項目。

**VideoFrameBodyItem** 具有下列屬性：
- **Timestamp**。 一個包含從影片中擷取影片畫面之時間 (以秒為單位) 的字串。
- **FrameImage**。 影片畫面的 URL。
- **中繼資料**： 一個 IList<VideoFrameBodyItemMetadataItem>。 **VideoFrameBodyItemMetadataItem** 就是一個索引鍵/值組。 有效的索引鍵包括：
- **reviewRecommended**。 如果建議對影片畫面進行人工審核，則為 True。
- **adultScore**。 一個對影片畫面中成人內容的嚴重性進行評等的值 (0 到 1)。
- **a**。 如果影片包含成人內容，則為 True。
- **racyScore**。 一個對影片畫面中猥褻內容的嚴重性進行評等的值 (0 到 1)。
- **r**。 如果影片畫面包含猥褻內容，則為 True。
- **ReviewerResultTags**。 一個 IList<VideoFrameBodyItemReviewerResultTagsItem>。 **VideoFrameBodyItemReviewerResultTagsItem** 就是一個索引鍵/值組。 應用程式可以使用這些標籤來組織影片畫面。

> [!NOTE]
> 本快速入門會為 **adultScore** 和 **racyScore** 屬性產生隨機值。 在生產環境應用程式中，您會從部署成「Azure 媒體服務」的[影片仲裁服務](video-moderation-api.md)取得這些值。

將下列方法定義新增至 VideoReviews 命名空間、Program 類別。

    <summary>
    /// Create a video frame to add to a video review after the video review is created.
    /// </summary>
    /// <param name="url">The URL of the video frame image.</param>
    /// <returns>The video frame.</returns>
    private static VideoFrameBodyItem CreateFrameToAddToReview(string url, string timestamp_seconds)
    {
        // We generate random "adult" and "racy" scores for the video frame.
        Random rand = new Random();

        var frame = new VideoFrameBodyItem
        {
            // The timestamp is measured in milliseconds. Convert from seconds.
            Timestamp = (int.Parse(timestamp_seconds) * 1000).ToString(),
            FrameImage = url,

            Metadata = new List<VideoFrameBodyItemMetadataItem>
            {
                new VideoFrameBodyItemMetadataItem("reviewRecommended", "true"),
                new VideoFrameBodyItemMetadataItem("adultScore", rand.NextDouble().ToString()),
                new VideoFrameBodyItemMetadataItem("a", "false"),
                new VideoFrameBodyItemMetadataItem("racyScore", rand.NextDouble().ToString()),
                new VideoFrameBodyItemMetadataItem("r", "false")
            },

            ReviewerResultTags = new List<VideoFrameBodyItemReviewerResultTagsItem>()
            {
                new VideoFrameBodyItemReviewerResultTagsItem("tag1", "value1")
            }
        };

        return frame;
    }

    /// <summary>
    /// Add a video frame to the indicated video review. For more information, see the API reference:
    /// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd
    /// </summary>
    /// <param name="client">The Content Moderator client.</param>
    /// <param name="review_id">The video review ID.</param>
    /// <param name="url">The URL of the video frame image.</param>
    static void AddFrame(ContentModeratorClient client, string review_id, string url, string timestamp_seconds)
    {
        Console.WriteLine("Adding a frame to the review with ID {0}.", review_id);

        var frames = new List<VideoFrameBodyItem>()
        {
            CreateFrameToAddToReview(url, timestamp_seconds)
        };
            
        client.Reviews.AddVideoFrameUrl("application/json", TeamName, review_id, frames);

        Thread.Sleep(throttleRate);
    

## <a name="get-video-frames-for-video-review"></a>取得影片審核項目的影片畫面

您可以使用 **ContentModeratorClient.Reviews.GetVideoFrames** 來取得影片審核項目的影片畫面。 **GetVideoFrames** 具有下列必要參數：
1. 您的 Content Moderator 小組名稱。
1. **CreateVideoReviews** 所傳回的影片審核項目識別碼。
1. 要取得之第一個影片畫面的從零開始索引。
1. 要取得的影片畫面數目。

將下列方法定義新增至 VideoReviews 命名空間、Program 類別。

    /// <summary>
    /// Get the video frames assigned to the indicated video review.  For more information, see the API reference:
    /// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7ba43e7151f0b10d45200
    /// </summary>
    /// <param name="client">The Content Moderator client.</param>
    /// <param name="review_id">The video review ID.</param>
    static void GetFrames(ContentModeratorClient client, string review_id)
    {
        Console.WriteLine("Getting frames for the review with ID {0}.", review_id);

        Frames result = client.Reviews.GetVideoFrames(TeamName, review_id, 0);
        Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

        Thread.Sleep(throttleRate);
    }

## <a name="get-video-review-information"></a>取得審核項目資訊

您可以使用 **ContentModeratorClient.Reviews.GetReview** 來取得影片審核項目的資訊。 **GetReview** 具有下列必要參數：
1. 您的 Content Moderator 小組名稱。
1. **CreateVideoReviews** 所傳回的影片審核項目識別碼。

將下列方法定義新增至 VideoReviews 命名空間、Program 類別。

    /// <summary>
    /// Get the information for the indicated video review. For more information, see the reference API:
    /// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2
    /// </summary>
    /// <param name="client">The Content Moderator client.</param>
    /// <param name="review_id">The video review ID.</param>
    private static void GetReview(ContentModeratorClient client, string review_id)
    {
        Console.WriteLine("Getting the status for the review with ID {0}.", review_id);

        var result = client.Reviews.GetReview(ModeratorHelper.Clients.TeamName, review_id);
        Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

        Thread.Sleep(throttleRate);
    }

## <a name="publish-video-review"></a>發佈影片審核項目

您可以使用 **ContentModeratorClient.Reviews.PublishVideoReview** 來發佈影片審核項目。 **PublishVideoReview** 具有下列必要參數：
1. 您的 Content Moderator 小組名稱。
1. **CreateVideoReviews** 所傳回的影片審核項目識別碼。

將下列方法定義新增至 VideoReviews 命名空間、Program 類別。

    /// <summary>
    /// Publish the indicated video review. For more information, see the reference API:
    /// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7bb29e7151f0b10d45201
    /// </summary>
    /// <param name="client">The Content Moderator client.</param>
    /// <param name="review_id">The video review ID.</param>
    private static void PublishReview(ContentModeratorClient client, string review_id)
    {
        Console.WriteLine("Publishing the review with ID {0}.", review_id);
        client.Reviews.PublishVideoReview(TeamName, review_id);
        Thread.Sleep(throttleRate);
    }

## <a name="putting-it-all-together"></a>總整理

將 **Main** 方法定義新增至 VideoReviews 命名空間、Program 類別。 最後，結束 Program 類別和 VideoReviews 命名空間。

    static void Main(string[] args)
    {
        using (ContentModeratorClient client = NewClient())
        {
            // Create a review with the content pointing to a streaming endpoint (manifest)
            var streamingcontent = "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest";
            string review_id = CreateReview(client, "review1", streamingcontent);

            var frame1_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG";
            var frame2_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG";
            var frame3_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG";

            // Add the frames from 17, 64, and 144 seconds.
            AddFrame(client, review_id, frame1_url, "17");
            AddFrame(client, review_id, frame2_url, "64");
            AddFrame(client, review_id, frame3_url, "144");

            // Get frames information and show
            GetFrames(client, review_id);
            GetReview(client, review_id);

            // Publish the review
            PublishReview(client, review_id);

            Console.WriteLine("Open your Content Moderator Dashboard and select Review > Video to see the review.");
            Console.WriteLine("Press any key to close the application.");
            Console.ReadKey();
        }
    }

## <a name="run-the-program-and-review-the-output"></a>執行程式並檢閱輸出
當您執行應用程式時，會看到以下幾行的輸出：

    Creating a video review.
    Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
    Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
    Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
    Getting frames for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
    {
        "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
        "VideoFrames": [
        {
            "Timestamp": "17000",
            "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_17000.PNG",
            "Metadata": [
            {
                "Key": "reviewRecommended",
                "Value": "true"
            },
            {
                "Key": "adultScore",
                "Value": "0.808312381528463"
            },
            {
                "Key": "a",
                "Value": "false"
            },
            {
                "Key": "racyScore",
                "Value": "0.846378884206702"
            },
            {
                "Key": "r",
                "Value": "false"
            }
            ],
            "ReviewerResultTags": [
            {
                "Key": "tag1",
                "Value": "value1"
            }
        ]
        },
        {
            "Timestamp": "64000",
            "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_64000.PNG",
            "Metadata": [
            {
                "Key": "reviewRecommended",
                "Value": "true"
            },
            {
                "Key": "adultScore",
                "Value": "0.576078300166912"
            },
            {
                "Key": "a",
                "Value": "false"
            },
            {
                "Key": "racyScore",
                "Value": "0.244768953064815"
            },
            {
                "Key": "r",
                "Value": "false"
            }
            ],
            "ReviewerResultTags": [
            {
                "Key": "tag1",
                "Value": "value1"
            }
        ]
        },
        {
            "Timestamp": "144000",
            "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_144000.PNG",
            "Metadata": [
            {
                "Key": "reviewRecommended",
                "Value": "true"
            },
            {
                "Key": "adultScore",
                "Value": "0.664480847150311"
            },
            {
                "Key": "a",
                "Value": "false"
            },
            {
                "Key": "racyScore",
                "Value": "0.933817870418456"
            },
            {
                "Key": "r",
                "Value": "false"
            }
            ],
            "ReviewerResultTags": [
            {
                "Key": "tag1",
                "Value": "value1"
            }
            ]
        }
        ]
    }
    
    Getting the status for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
    {
        "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
        "SubTeam": "public",
        "Status": "UnPublished",
        "ReviewerResultTags": [],
        "CreatedBy": "testreview6",
        "Metadata": [
        {
            "Key": "FrameCount",
            "Value": "3"
        }
        ],
        "Type": "Video",
        "Content": "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
        "ContentId": "review1",
        "CallbackEndpoint": null
    }

    Publishing the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
    Open your Content Moderator Dashboard and select Review > Video to see the review.
    Press any key to close the application.

## <a name="check-out-your-video-review"></a>查看您的影片審核項目

最後，您會在 Content Moderator 審核工具帳戶中的 [審核]>[影片] 畫面上看到影片審核項目。

![給人工仲裁者的影片審核項目](images/ams-video-review.PNG)

## <a name="next-steps"></a>後續步驟

針對這個及其他適用於 .NET 的 Content Moderator 快速入門取得 [Content Moderator .NET SDK](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 和 [Visual Studio 解決方案](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)。

了解如何將[文字記錄仲裁](video-transcript-moderation-review-tutorial-dotnet.md)新增至影片審核項目。 

查看有關如何開發[完整影片審查解決方案](video-transcript-moderation-review-tutorial-dotnet.md)的詳細教學課程。
