---
title: 快速入門：使用 .NET 啟動仲裁作業 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 如何使用 Azure Content Moderator SDK for .NET 起始仲裁作業。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: sajagtap
ms.openlocfilehash: 6045d6daf2abace6e2b38bd6fd6e22516e3a60a0
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227418"
---
# <a name="quickstart-start-moderation-jobs-using-net"></a>快速入門：使用 .NET 啟動仲裁作業

本文提供資訊和範例程式碼，可協助您開始使用 [Content Moderator SDK for .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 來執行下列操作：
 
- 啟動審核作業以掃描和建立人工審核者的檢閱
- 取得擱置中檢閱的狀態
- 追蹤並取得檢閱的最終狀態
- 將結果提交至回呼 URL

本文假設您已經熟悉 Visual Studio 和 C#。

## <a name="sign-up-for-content-moderator"></a>註冊 Content Moderator

您必須有訂用帳戶金鑰，才能透過 REST API 或 SDK 使用 Content Moderator 服務。
請參考[快速入門](quick-start.md)，以了解如何取得金鑰。

## <a name="sign-up-for-a-review-tool-account-if-not-completed-in-the-previous-step"></a>如果未在上一個步驟中完成審核工具帳戶的註冊，請於此時註冊

如果您是從 Azure 入口網站取得 Content Moderator，也請[註冊審核工具帳戶](https://contentmoderator.cognitive.microsoft.com/)，並建立審核小組。 您需要小組識別碼和審核工具才能呼叫審核 API，以在審核工具中啟動作業及檢視審核項目。

## <a name="ensure-your-api-key-can-call-the-review-api-for-review-creation"></a>請確定您的 API 金鑰可呼叫審核 API 以建立審核項目

完成前述步驟後，如果您是從 Azure 入口網站開始作業的，您可能會獲得兩個 Content Moderator 金鑰。 

如果您打算在 SDK 範例中使用 Azure 提供的 API 金鑰，請依照[搭配使用 Azure 金鑰與審核 API](review-tool-user-guide/credentials.md#use-the-azure-account-with-the-review-tool-and-review-api) 一節中說明的步驟操作，以允許應用程式呼叫審核 API 並建立審核項目。

如果您使用審核工具所產生的免費試用版金鑰，則您的審核工具帳戶已知悉金鑰，因此不需執行額外的步驟。

## <a name="define-a-custom-moderation-workflow"></a>定義自訂審核工作流程

審核作業會使用 API 掃描內容，並使用**工作流程**來判斷是否要建立檢閱。
雖然檢閱工具包含預設工作流程，不過讓我們針對本快速入門[定義自訂工作流程](Review-Tool-User-Guide/Workflows.md)。

您會在啟動審核作業的程式碼中使用工作流程的名稱。

## <a name="create-your-visual-studio-project"></a>建立 Visual Studio 專案

1. 將一個新的 [主控台應用程式 (.NET Framework)] 專案新增到您的解決方案。

   在範例程式碼中，將專案命名為 **CreateReviews**。

1. 選取此專案作為解決方案的單一啟始專案。

### <a name="install-required-packages"></a>安裝必要的套件

安裝下列 NuGet 封裝：

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>更新程式的 using 陳述式

修改程式的 using 陳述式。

    using Microsoft.Azure.CognitiveServices.ContentModerator;
    using Microsoft.CognitiveServices.ContentModerator;
    using Microsoft.CognitiveServices.ContentModerator.Models;
    using Newtonsoft.Json;
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Threading;

### <a name="create-the-content-moderator-client"></a>建立 Content Moderator 用戶端

新增下列程式碼，為您的訂用帳戶建立 Content Moderator 用戶端。

> [!IMPORTANT]
> 以您的區域識別碼和訂用帳戶訂用帳戶的值更新 **AzureRegion** 和 **CMSubscriptionKey** 欄位。


    /// <summary>
    /// Wraps the creation and configuration of a Content Moderator client.
    /// </summary>
    /// <remarks>This class library contains insecure code. If you adapt this 
    /// code for use in production, use a secure method of storing and using
    /// your Content Moderator subscription key.</remarks>
    public static class Clients
    {
        /// <summary>
        /// The region/location for your Content Moderator account, 
        /// for example, westus.
        /// </summary>
        private static readonly string AzureRegion = "YOUR API REGION";

        /// <summary>
        /// The base URL fragment for Content Moderator calls.
        /// </summary>
        private static readonly string AzureBaseURL =
            $"https://{AzureRegion}.api.cognitive.microsoft.com";

        /// <summary>
        /// Your Content Moderator subscription key.
        /// </summary>
        private static readonly string CMSubscriptionKey = "YOUR API KEY";

        /// <summary>
        /// Returns a new Content Moderator client for your subscription.
        /// </summary>
        /// <returns>The new client.</returns>
        /// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
        /// When you have finished using the client,
        /// you should dispose of it either directly or indirectly. </remarks>
        public static ContentModeratorClient NewClient()
        {
            // Create and initialize an instance of the Content Moderator API wrapper.
            ContentModeratorClient client = new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey));

            client.Endpoint = AzureBaseURL;
            return client;
        }
    }

### <a name="initialize-application-specific-settings"></a>將應用程式特定的設定初始化

將下列內容和靜態欄位新增至 Program.cs 中的 **Program** 類別。

> [!NOTE]
> 您可以將 TeamName 常數設定為建立 Content Moderator 訂用帳戶時所使用的名稱。 您可以從 [Content Moderator 網站](https://westus.contentmoderator.cognitive.microsoft.com/)擷取 TeamName。
> 登入後，請從 [設定] (齒輪) 功能表選取 [認證]。
>
> 小組名稱會是 [API] 區段中 [Id] 欄位的值。


    /// <summary>
    /// The moderation job will use this workflow that you defined earlier.
    /// See the quickstart article to learn how to setup custom workflows.
    /// </summary>
    private const string WorkflowName = "OCR";
    
    /// <summary>
    /// The name of the team to assign the job to.
    /// </summary>
    /// <remarks>This must be the team name you used to create your 
    /// Content Moderator account. You can retrieve your team name from
    /// the Content Moderator web site. Your team name is the Id associated 
    /// with your subscription.</remarks>
    private const string TeamName = "***";

    /// <summary>
    /// The URL of the image to create a review job for.
    /// </summary>
    private const string ImageUrl =
        "https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png";

    /// <summary>
    /// The name of the log file to create.
    /// </summary>
    /// <remarks>Relative paths are relative to the execution directory.</remarks>
    private const string OutputFile = "OutputLog.txt";

    /// <summary>
    /// The number of seconds to delay after a review has finished before
    /// getting the review results from the server.
    /// </summary>
    private const int latencyDelay = 45;

    /// <summary>
    /// The callback endpoint for completed reviews.
    /// </summary>
    /// <remarks>Reviews show up for reviewers on your team. 
    /// As reviewers complete reviews, results are sent to the
    /// callback endpoint using an HTTP POST request.</remarks>
    private const string CallbackEndpoint = "";

## <a name="add-code-to-auto-moderate-create-a-review-and-get-the-job-details"></a>新增程式碼來自動審核、建立檢閱和取得作業詳細資料

> [!Note]
> 實務上，您可以將回呼 URL **CallbackEndpoint** 設定為負責 (透過 HTTP POST 要求) 接收手動檢閱結果的 URL。

一開始，請先將以下程式碼新增至 **Main** 方法。

    using (TextWriter writer = new StreamWriter(OutputFile, false))
    {
        using (var client = Clients.NewClient())
        {
            writer.WriteLine("Create review job for an image.");
            var content = new Content(ImageUrl);
        
            // The WorkflowName contains the name of the workflow defined in the online review tool.
            // See the quickstart article to learn more.
            var jobResult = client.Reviews.CreateJobWithHttpMessagesAsync(
                    TeamName, "image", "contentID", WorkflowName, "application/json", content, CallbackEndpoint);

            // Record the job ID.
            var jobId = jobResult.Result.Body.JobIdProperty;

            // Log just the response body from the returned task.
            writer.WriteLine(JsonConvert.SerializeObject(
                jobResult.Result.Body, Formatting.Indented));

            Thread.Sleep(2000);
            writer.WriteLine();

            writer.WriteLine("Get review job status.");
            var jobDetails = client.Reviews.GetJobDetailsWithHttpMessagesAsync(
                    TeamName, jobId);

            // Log just the response body from the returned task.
            writer.WriteLine(JsonConvert.SerializeObject(
                    jobDetails.Result.Body, Formatting.Indented));

            Console.WriteLine();
            Console.WriteLine("Perform manual reviews on the Content Moderator site.");
            Console.WriteLine("Then, press any key to continue.");
            Console.ReadKey();

            Console.WriteLine();
            Console.WriteLine($"Waiting {latencyDelay} seconds for results to propagate.");
            Thread.Sleep(latencyDelay * 1000);

            writer.WriteLine("Get review details.");
            jobDetails = client.Reviews.GetJobDetailsWithHttpMessagesAsync(
            TeamName, jobId);

            // Log just the response body from the returned task.
            writer.WriteLine(JsonConvert.SerializeObject(
            jobDetails.Result.Body, Formatting.Indented));
        }
        writer.Flush();
        writer.Close();
    }

> [!NOTE]
> 您的 Content Moderator 服務金鑰會有每秒要求數目 (RPS) 的速率限制。 如果您超出此限制，SDK 就會擲回錯誤碼為 429 的例外狀況。 
>
> 免費層金鑰有一個 RPS 速率限制。

## <a name="run-the-program-and-review-the-output"></a>執行程式並檢閱輸出

您會在主控台中看到下列輸出範例：

    Perform manual reviews on the Content Moderator site.
    Then, press any key to continue.

請登入 Content Moderator 檢閱工具，以查看擱置中的影像檢閱。

使用 [下一步] 按鈕以提交影像。

![給人工審核者的影像檢閱](images/ocr-sample-image.PNG)

## <a name="see-the-sample-output-in-the-log-file"></a>查看記錄檔中的輸出範例

> [!NOTE]
> 在輸出檔案中，**Teamname**、**ContentId**、**CallBackEndpoint** 和 **WorkflowId** 字串會反映您稍早所使用的值。

    Create moderation job for an image.
    {
        "JobId": "2018014caceddebfe9446fab29056fd8d31ffe"
    }

    Get review details.
    {
        "Id": "2018014caceddebfe9446fab29056fd8d31ffe",
        "TeamName": "some team name",
        "Status": "InProgress",
        "WorkflowId": "OCR",
        "Type": "Image",
        "CallBackEndpoint": "",
        "ReviewId": "",
        "ResultMetaData": [],
        "JobExecutionReport": [
        {
            "Ts": "2018-01-07T00:38:26.7714671",
            "Msg": "Successfully got hasText response from Moderator"
        },
        {
            "Ts": "2018-01-07T00:38:26.4181346",
            "Msg": "Getting hasText from Moderator"
        },
        {
            "Ts": "2018-01-07T00:38:25.5122828",
            "Msg": "Starting Execution - Try 1"
        }
        ]
    }


## <a name="your-callback-url-if-provided-receives-this-response"></a>若有提供回呼 URL，則會收到此回應。

您會看到如下列範例所示的回應：

> [!NOTE]
> 在回呼回應中，**ContentId** 和 **WorkflowId** 字串會反映您稍早所使用的值。

    {
        "JobId": "2018014caceddebfe9446fab29056fd8d31ffe",
        "ReviewId": "201801i28fc0f7cbf424447846e509af853ea54",
        "WorkFlowId": "OCR",
        "Status": "Complete",
        "ContentType": "Image",
        "CallBackType": "Job",
        "ContentId": "contentID",
        "Metadata": {
            "hastext": "True",
            "ocrtext": "IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n",
            "imagename": "contentID"
        }
    }


## <a name="next-steps"></a>後續步驟

針對這個及其他適用於 .NET 的 Content Moderator 快速入門取得 [Content Moderator .NET SDK](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 和 [Visual Studio 解決方案](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)，並開始進行您的整合。
