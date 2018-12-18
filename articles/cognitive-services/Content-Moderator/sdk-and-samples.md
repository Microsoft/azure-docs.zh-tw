---
title: SDK 和範例 - Content Moderator、Python、Java、Node.js 和 .NET
titlesuffix: Azure Cognitive Services
description: 取得 Content Moderator 的 SDK 和範例
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: sample
ms.date: 02/27/2018
ms.author: sajagtap
ms.openlocfilehash: a57f6a312b00d7ec3d927c6fda319f1de8663c9c
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220443"
---
# <a name="content-moderator-sdks-and-samples"></a>Content Moderator SDK 和範例

## <a name="sdks-for-python-java-nodejs-and-net"></a>Python、Java、Node.js 和 .NET 的 SDK

- [Content Moderator SDK for Python](https://pypi.python.org/pypi/azure-cognitiveservices-vision-contentmoderator)
- [Content Moderator SDK for Java](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-cognitiveservices-contentmoderator%22)
- [Content Moderator SDK for Node.js](https://www.npmjs.com/package/azure-cognitiveservices-contentmoderator)
- [Content Moderator SDK for .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/)

## <a name="net-sdk-samples"></a>.NET SDK 範例

以下清單包含使用 Azure Content Moderator SDK for .NET 建置之程式碼範例的連結。

- **協助程式程式庫**：[建立可供在其他範例中使用的 Content Moderator 用戶端](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ModeratorHelper/Clients.cs)。 請參閱[快速入門](content-moderator-helper-quickstart-dotnet.md)。

### <a name="moderation"></a>審核 
- **影像審核**：[評估影像中是否有成人或猥褻內容、文字及臉部](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ImageModeration/Program.cs)。 請參閱[快速入門](image-moderation-quickstart-dotnet.md)。
- **自訂影像**：[使用自訂影像清單進行審核](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ImageListManagement/Program.cs)。 請參閱[快速入門](image-lists-quickstart-dotnet.md)。

> [!NOTE]
> 上限是 **5 個影像清單**，其中每個清單**不可超過 10,000 個影像**。
>

- **文字審核**：[過濾粗話和個人識別資訊 (PII) 的文字](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/TextModeration/Program.cs)。 請參閱[快速入門](text-moderation-quickstart-dotnet.md)。
- **自訂字詞**：[使用自訂字詞清單進行審核](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/TermListManagement/Program.cs)。 請參閱[快速入門](term-lists-quickstart-dotnet.md)。

> [!NOTE]
> 上限是 **5 個字詞清單**，其中每個清單**不可超過 10,000 個字詞**。
>

- **影片審核**：[掃描影片中是否有成人和猥褻內容，並取得結果](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/VideoModeration/Program.cs)。 請參閱[快速入門](video-moderation-api.md)。

### <a name="review"></a>審核
- **影像作業**：[啟動一個會掃描並建立審核項目的審核作業](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ImageJobs/Program.cs)。 請參閱[快速入門](moderation-jobs-quickstart-dotnet.md)。
- **影像審核**：[建立人在迴路 (human-in-the-loop) 的審核項目](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ImageReviews/Program.cs)。 請參閱[快速入門](moderation-reviews-quickstart-dotnet.md)。
- **影片審核**：[建立人在迴路 (human-in-the-loop) 的影片審核項目](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/VideoReviews/Program.cs)。 請參閱[快速入門](video-reviews-quickstart-dotnet.md)
- **影片文字記錄審核**：[建立人在迴路 (human-in-the-loop) 的影片文字記錄審核項目](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/VideoTranscriptReviews/Program.cs)。請參閱[快速入門](video-reviews-quickstart-dotnet.md)

請參閱位於 [GitHub 上的 Content Moderator .NET 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)的所有 .NET 範例 \(英文\)。

## <a name="rest-api-samples-in-c"></a>C# 中的 REST API 範例

- [影像仲裁](https://github.com/MicrosoftContentModerator/ContentModerator-API-Samples/tree/master/ImageModeration)
- [文字仲裁](https://github.com/MicrosoftContentModerator/ContentModerator-API-Samples/tree/master/TextModeration)
- [影片仲裁](https://github.com/MicrosoftContentModerator/ContentModerator-API-Samples/tree/master/VideoModeration)
- [影像檢閱](https://github.com/MicrosoftContentModerator/ContentModerator-API-Samples/tree/master/ImageReviews)
- [影像作業](https://github.com/MicrosoftContentModerator/ContentModerator-API-Samples/tree/master/ImageJob)

如需這些範例的逐步解說，請參閱[隨選網路研討會](https://info.microsoft.com/cognitive-services-content-moderator-ondemand.html)。

## <a name="tutorials"></a>教學課程
- [電子商務目錄審核](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration)。 請參閱[教學課程](ecommerce-retail-catalog-moderation.md)。
- [Facebook 內容審核](https://github.com/MicrosoftContentModerator/samples-fbPageModeration)。 請參閱[教學課程](facebook-post-moderation.md)。
- [影片及文字記錄的審核和檢閱解決方案](https://github.com/MicrosoftContentModerator/VideoReviewConsoleApp)。請參閱[教學課程](video-transcript-moderation-review-tutorial-dotnet.md)

## <a name="on-demand-webinars"></a>隨選網路研討會
- [使用 Content Moderator 進行大規模的電腦輔助內容審核](https://info.microsoft.com/cognitive-services-content-moderator-ondemand.html)
