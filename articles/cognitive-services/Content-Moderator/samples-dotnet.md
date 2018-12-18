---
title: 程式碼範例 - Content Moderator (.NET)
description: 在應用程式中使用 Content Moderator
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: sample
ms.date: 01/10/2018
ms.author: sajagtap
ms.openlocfilehash: c5465a2de0924037eb93b9de652e93e85f166737
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223146"
---
# <a name="net-sdk-samples"></a>.NET SDK 範例

以下清單包含使用 Azure Content Moderator SDK for .NET 建置之程式碼範例的連結。

- **協助程式程式庫**：[建立可供在其他範例中使用的 Content Moderator 用戶端](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ModeratorHelper/Clients.cs)。 請參閱[快速入門](content-moderator-helper-quickstart-dotnet.md)。

## <a name="moderation"></a>審核

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

## <a name="review"></a>審核

- **影像作業**：[啟動一個會掃描並建立審核項目的審核作業](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ImageJobs/Program.cs)。 請參閱[快速入門](moderation-jobs-quickstart-dotnet.md)。
- **影像審核**：[建立人在迴路 (human-in-the-loop) 的審核項目](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/ImageReviews/Program.cs)。 請參閱[快速入門](moderation-reviews-quickstart-dotnet.md)。
- **影片審核**：[建立人在迴路 (human-in-the-loop) 的影片審核項目](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/VideoReviews/Program.cs)。 請參閱[快速入門](video-reviews-quickstart-dotnet.md)
- **影片文字記錄審核**：[建立人在迴路 (human-in-the-loop) 的影片文字記錄審核項目](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/ContentModerator/VideoTranscriptReviews/Program.cs)。請參閱[快速入門](video-reviews-quickstart-dotnet.md)

請參閱位於 [GitHub 上的 Content Moderator .NET 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)的所有 .NET 範例 \(英文\)。
