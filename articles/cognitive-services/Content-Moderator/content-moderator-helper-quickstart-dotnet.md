---
title: 快速入門：Content Moderator SDK for .NET 協助程式方法
titlesuffix: Azure Cognitive Services
description: 如何使用 Azure Content Moderator SDK for .NET 傳回 Content Moderator 用戶端
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: quickstart
ms.date: 01/04/2018
ms.author: sajagtap
ms.openlocfilehash: 0c167de3af8aea6820962e7cc8d0c21f5ba61807
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223262"
---
# <a name="quickstart-helper-code-to-return-a-content-moderator-client"></a>快速入門：用於傳回 Content Moderator 用戶端的協助程式程式碼

本文提供資訊和範例程式碼，可協助您開始使用 Content Moderator SDK for .NET 執行下列操作，為您的訂用帳戶建立 Content Moderator 用戶端。

本節中的其他快速入門會使用此程式庫。

本文假設您已經熟悉 Visual Studio 和 C#。

> [!IMPORTANT]
> 此類別庫包含僅供示範的程式碼。
> 如果您改寫此程式碼以供在生產環境中使用，請使用安全的方法來儲存及使用 Content Moderator 訂用帳戶金鑰。

## <a name="sign-up-for-content-moderator-services"></a>註冊 Content Moderator 服務

您必須有訂用帳戶金鑰，才能透過 REST API 或 SDK 使用 Content Moderator 服務。
請參考[快速入門](quick-start.md)，以了解如何取得金鑰。

## <a name="create-your-visual-studio-project"></a>建立 Visual Studio 專案

1. 建立新的**類別庫 (.NET Framework)** 專案。

   在範例程式碼中，我將專案命名為 **ModeratorHelper**。

1. 新增 **System.Configuration** Framework 組件的參考。

### <a name="install-required-packages"></a>安裝必要的套件

安裝下列 NuGet 封裝：

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Newtonsoft.Json

### <a name="create-the-content-moderator-client"></a>建立 Content Moderator 用戶端

使用下列程式碼取代 ModeratorHelper.cs 檔案的內容：

    using Microsoft.CognitiveServices.ContentModerator;

    namespace ModeratorHelper
    {
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
        private static readonly string AzureRegion = "myRegion";

        /// <summary>
        /// The base URL fragment for Content Moderator calls.
        /// </summary>
        private static readonly string AzureBaseURL =
            $"{AzureRegion}.api.cognitive.microsoft.com";

        /// <summary>
        /// Your Content Moderator subscription key.
        /// </summary>
        private static readonly string CMSubscriptionKey = "myKey";

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

            client.BaseUrl = AzureBaseURL;
            return client;
        }
    }
    }


> [!IMPORTANT]
> 以您的區域識別碼和訂用帳戶訂用帳戶的值更新 **AzureRegion** 和 **CMSubscriptionKey** 欄位。

您現在有辦法為您的訂用帳戶快速建立 Content Moderator 用戶端。

## <a name="next-steps"></a>後續步驟

針對這個及其他適用於 .NET 的 Content Moderator 快速入門，[下載 Visual Studio 解決方案](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)，並開始進行您的整合。
