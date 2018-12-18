---
title: Bing 搜尋 SDK | Microsoft Docs
titleSuffix: Microsoft Cognitive Services
description: 搜尋 Web 之應用程式的 Bing 搜尋 SDK。
services: cognitive-services
author: mikedodaro
manager: rosh
ms.assetid: ''
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: article
ms.date: 1/24/2018
ms.author: v-gedod
ms.openlocfilehash: 4a40ea665e153536d2322706b455598902ce41eb
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370434"
---
# <a name="bing-search-sdk"></a>Bing 搜尋 SDK
Bing 新聞搜尋 API 範例包括下列情節：
1. 使用 `market` 和 `count` 參數查詢新聞中的搜尋項目、驗證結果數目，並印出第一個新聞結果的 `totalEstimatedMatches`、名稱、URL、描述、已發行時間和提供者名稱。
2. 使用 `freshness` 和 `sortBy` 參數查詢最新新聞中的搜尋項目、驗證結果數目，並印出第一個新聞結果的 `totalEstimatedMatches`、URL、描述、已發行時間和提供者名稱。
3. 使用安全搜尋查詢分類新聞中的 `movie` 和 `TV entertainment`、驗證結果數目，並印出第一個新聞結果的的分類、名稱、URL、描述、已發行時間和提供者名稱。
4. 在 Bing 中查詢新聞趨勢主題、驗證結果數目，並印出第一個新聞結果的名稱、查詢文字、`webSearchUrl`、`newsSearchUrl` 和影像 URL。

Bing 搜尋 SDK 透過下列程式設計語言更輕易地存取 Web 搜尋功能：
* 開始使用 [.NET 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
    * [NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.NewsSearch/1.2.0)
    * 另請參閱 [.NET 程式庫](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/CognitiveServices/dataPlane/Search/BingNewsSearch)，以了解定義和相依性。
* 開始使用 [Node.js 範例](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples) 
    * 另請參閱 [Node.js 程式庫](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/newsSearch)，以了解定義和相依性。
* 開始使用 [Java 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples) 
    * 另請參閱 [Java 程式庫](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingNewsSearch)，以了解定義和相依性。
* 開始使用 [Python 範例](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples) 
    * 另請參閱 [Python 程式庫](https://github.com/Azure/azure-sdk-for-python/tree/master/azure-cognitiveservices-search-newssearch)，以了解定義和相依性。

每種語言的 SDK 範例都會包括讀我檔案，內含必要條件和安裝/執行範例的詳細資料。