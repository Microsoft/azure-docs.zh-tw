---
title: 仲裁文字 - Content Moderator (.NET)
titlesuffix: Azure Cognitive Services
description: 如何使用 Content Moderator SDK for .NET 來仲裁文字
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 09/10/2018
ms.author: sajagtap
ms.openlocfilehash: 6db97231b2dc8905f1a3b3a6bc63580e9f371f84
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227260"
---
# <a name="moderate-text-using-net"></a>使用 .NET 來仲裁文字

本文提供資訊和範例程式碼，可協助您開始使用 [Content Moderator SDK for .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 來執行下列操作：

- 使用字詞型篩選來偵測文字中可能的粗話
- 使用機器學習型模型模型將[文字分類](text-moderation-api.md#classification)成三種類別。
- 偵測個人識別資訊 (PII)，例如美國和英國電話號碼、電子郵件地址及美國郵寄地址。
- 將文字正規化並自動校正錯字

本文假設您已經熟悉 Visual Studio 和 C#。

## <a name="sign-up-for-content-moderator-services"></a>註冊 Content Moderator 服務

您必須有訂用帳戶金鑰，才能透過 REST API 或 SDK 使用 Content Moderator 服務。
請參考[快速入門](quick-start.md)，以了解如何取得金鑰。

## <a name="create-your-visual-studio-project"></a>建立 Visual Studio 專案

1. 將一個新的 [主控台應用程式 (.NET Framework)] 專案新增到您的解決方案。

   在範例程式碼中，將專案命名為 **TextModeration**。

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

將下列靜態欄位新增至 Program.cs 中的 **Program** 類別。

    /// <summary>
    /// The name of the file that contains the text to evaluate.
    /// </summary>
    /// <remarks>You will need to create an input file and update this path
    /// accordingly. Relative paths are relative to the execution directory.</remarks>
    private static string TextFile = "TextFile.txt";

    /// <summary>
    /// The name of the file to contain the output from the evaluation.
    /// </summary>
    /// <remarks>Relative paths are relative to the execution directory.</remarks>
    private static string OutputFile = "TextModerationOutput.txt";

我們已使用下列文字來產生本快速入門的輸出：

> [!NOTE]
> 以下範例文字中的無效社會安全號碼是刻意安排的。 目的是要傳達範例輸入和輸出格式。

    Is this a grabage or crap email abcdef@abcd.com, phone: 6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.
    These are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 
    0800 820 3300. Also, 999-99-9999 looks like a social security number (SSN).

## <a name="add-code-to-load-and-evaluate-the-input-text"></a>新增程式碼以載入和評估輸入文字

將以下程式碼新增至 **Main** 方法。

    // Load the input text.
    string text = File.ReadAllText(TextFile);
    Console.WriteLine("Screening {0}", TextFile);

    text = text.Replace(System.Environment.NewLine, " ");

    // Save the moderation results to a file.
    using (StreamWriter outputWriter = new StreamWriter(OutputFile, false))
    {
        // Create a Content Moderator client and evaluate the text.
        using (var client = Clients.NewClient())
        {
            // Screen the input text: check for profanity, classify the text into three categories,
                // do autocorrect text, and check for personally identifying
                // information (PII)
                outputWriter.WriteLine("Autocorrect typos, check for matching terms, PII, and classify.");
                var screenResult =
                    client.TextModeration.ScreenText("eng", "text/plain", text, true, true, null, true);
                outputWriter.WriteLine(
                        JsonConvert.SerializeObject(screenResult, Formatting.Indented));
        }
        outputWriter.Flush();
        outputWriter.Close();
    }

> [!NOTE]
> 您的 Content Moderator 服務金鑰會有每秒要求數目 (RPS) 的速率限制。如果超出此限制，SDK 就會擲回錯誤碼為 429 的例外狀況。
>
> 使用免費層金鑰時，要求速率限制為每秒一個要求。

## <a name="run-the-program-and-review-the-output"></a>執行程式並檢閱輸出

程式的範例輸出 (寫入至記錄檔) 為：

    Autocorrect typos, check for matching terms, PII, and classify.
    {
    "OriginalText": "\"Is this a grabage or crap email abcdef@abcd.com, phone: 6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052. These are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 0800 820 3300. Also, 999-99-9999 looks like a social security number (SSN).\"",
    "NormalizedText": "\" Is this a garbage or crap email abide@ abed. com, phone: 6657789887, IP: 255. 255. 255. 255, 1 Microsoft Way, Redmond, WA 98052. These are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 0800 820 3300. Also, 999-99-9999 looks like a social security number ( SSN) . \"",
    "AutoCorrectedText": "\" Is this a garbage or crap email abide@ abed. com, phone: 6657789887, IP: 255. 255. 255. 255, 1 Microsoft Way, Redmond, WA 98052. These are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 0800 820 3300. Also, 999-99-9999 looks like a social security number ( SSN) . \"",
    "Misrepresentation": null,
    
    "Classification": {
            "Category1": {
            "Score": 1.5113095059859916E-06
            },
            "Category2": {
            "Score": 0.12747249007225037
            },
            "Category3": {
            "Score": 0.98799997568130493
            },
            "ReviewRecommended": true
    },
    "Status": {
        "Code": 3000,
        "Description": "OK",
        "Exception": null
    },
    "PII": {
            "Email": [
                {
                "Detected": "abcdef@abcd.com",
                "SubType": "Regular",
                "Text": "abcdef@abcd.com",
                "Index": 33
                }
                ],
            "IPA": [
                {
                "SubType": "IPV4",
                "Text": "255.255.255.255",
                "Index": 73
                }
                ],
            "Phone": [
                {
                "CountryCode": "US",
                "Text": "6657789887",
                "Index": 57
                },
                {
                "CountryCode": "US",
                "Text": "870 608 4000",
                "Index": 211
                },
                {
                "CountryCode": "UK",
                "Text": "+44 870 608 4000",
                "Index": 207
                },
                {
                "CountryCode": "UK",
                "Text": "0344 800 2400",
                "Index": 227
                },
                {
            "   CountryCode": "UK",
                "Text": "0800 820 3300",
                "Index": 244
                }
                ],
             "Address": [{
                 "Text": "1 Microsoft Way, Redmond, WA 98052",
                 "Index": 89
                    }]
        },
    "Language": "eng",
    "Terms": [
        {
            "Index": 22,
            "OriginalIndex": 22,
            "ListId": 0,
            "Term": "crap"
        }
    ],
    "TrackingId": "9392c53c-d11a-441d-a874-eb2b93d978d3"
    }

## <a name="next-steps"></a>後續步驟

針對這個及其他適用於 .NET 的 Content Moderator 快速入門取得 [Content Moderator .NET SDK](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 和 [Visual Studio 解決方案](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)，並開始進行您的整合。
