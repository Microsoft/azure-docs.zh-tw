---
title: 快速入門：Bing 新聞搜尋 SDK，C#
titleSuffix: Azure Cognitive Services
description: 設定 Bing 新聞搜尋 SDK 主控台應用程式。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: quickstart
ms.date: 01/30/2018
ms.author: v-gedod
ms.openlocfilehash: 416557b11ebef953411fb6fabcddb72d08dcb5af
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48802979"
---
# <a name="quickstart-bing-news-search-sdk-with-c"></a>快速入門：搭配使用 Bing 新聞搜尋 SDK 與 C#

Bing 新聞搜尋 SDK 包含用於新聞查詢以及剖析結果的 REST API 功能。 

[C# Bing 新聞搜尋 SDK 範例的原始程式碼](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingNewsSearch)可從 Git Hub 取得。

## <a name="application-dependencies"></a>應用程式相依性

若要使用 Bing 新聞搜尋 SDK 來設定主控台應用程式，請在 Visual Studio 中瀏覽至 [方案總管] 中的 `Manage NuGet Packages` 選項。  新增 `Microsoft.Azure.CognitiveServices.Search.NewsSearch` 套件。

安裝 [NuGet 新聞搜尋 SDK 套件](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.NewsSearch/1.2.0)也會安裝相依性，包括：
* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

## <a name="news-search-client"></a>新聞搜尋用戶端
若要建立 `NewsSearchAPI` 用戶端的執行個體，請新增 using 指示詞：
```
using Microsoft.Azure.CognitiveServices.Search.NewsSearch;

```
然後，具現化用戶端：
```
var client = new NewsSearchAPI(new ApiKeyServiceClientCredentials("YOUR-ACCESS-KEY"));


```
使用用戶端以利用查詢文字搜尋：
```
var newsResults = client.News.SearchAsync(query: "Quantum  Computing", market: "en-us", count: 10).Result;
Console.WriteLine("Search news for query \"Quantum  Computing\" with market and count");

```
剖析先前查詢結果中所傳回的新聞：
```
if (newsResults.Value.Count > 0)
{
    var firstNewsResult = newsResults.Value.First();

    Console.WriteLine($"TotalEstimatedMatches value: {newsResults.TotalEstimatedMatches}");
    Console.WriteLine($"News result count: {newsResults.Value.Count}");
    Console.WriteLine($"First news name: {firstNewsResult.Name}");
    Console.WriteLine($"First news url: {firstNewsResult.Url}");
    Console.WriteLine($"First news description: {firstNewsResult.Description}");
    Console.WriteLine($"First news published time: {firstNewsResult.DatePublished}");
    Console.WriteLine($"First news provider: {firstNewsResult.Provider.First().Name}");
}

else
{
    Console.WriteLine("Couldn't find news results!");
}

```
## <a name="complete-console-application"></a>完成主控台應用程式

下列主控台應用程式會執行先前定義的查詢，並搜尋新聞中是否有 "Quantum Computing"。 要求包括 `market` 和 `count` 參數。 程式碼會驗證結果數目，並印出第一個新聞結果的 `totalEstimatedMatches`、`name`、`url`、`description`、`published time` 以及 `provider` 的 `name`。

```
using System;
using System.Linq;
using Microsoft.Azure.CognitiveServices.Search.NewsSearch;

namespace NewsSrchSDK
{
    class Program
    {
        static void Main(string[] args)
        {
            var client = new NewsSearchAPI(new ApiKeyServiceClientCredentials("YOUR-ACCESS-KEY"));

            try
            {
                var newsResults = client.News.SearchAsync(query: "Quantum  Computing", market: "en-us", count: 10).Result;
                Console.WriteLine("Search news for query \"Quantum  Computing\" with market and count");

                if (newsResults == null)
                {
                    Console.WriteLine("Didn't see any news result data..");
                }
                else
                {
                    if (newsResults.Value.Count > 0)
                    {
                        var firstNewsResult = newsResults.Value.First();

                        Console.WriteLine($"TotalEstimatedMatches value: {newsResults.TotalEstimatedMatches}");
                        Console.WriteLine($"News result count: {newsResults.Value.Count}");
                        Console.WriteLine($"First news name: {firstNewsResult.Name}");
                        Console.WriteLine($"First news url: {firstNewsResult.Url}");
                        Console.WriteLine($"First news description: {firstNewsResult.Description}");
                        Console.WriteLine($"First news published time: {firstNewsResult.DatePublished}");
                        Console.WriteLine($"First news provider: {firstNewsResult.Provider.First().Name}");
                    }
                    else
                    {
                        Console.WriteLine("Couldn't find news results!");
                    }
                }
            }

            catch (Exception ex)
            {
                Console.WriteLine("Encountered exception. " + ex.Message);
            }

            // Include these methods to run queries explained under headings following.
            // NewsSearchWithFilters(client);
            // NewsCategory(client);
            // TrendingTopics(client);

            Console.WriteLine("Any key to exit...");
            Console.ReadKey();
        }

    }
}

```
## <a name="recent-news-freshness-and-sortby-parameters"></a>最新 news、freshness 和 sortBy 參數
下列程式碼使用 `freshness` 和 `sortBy` 參數來搜尋最新新聞中是否有 "Artificial Intelligence"。 它會驗證結果數目，並印出第一個新聞結果的 `totalEstimatedMatches`、`name`、`url`、`description`、`published time` 和提供者 `name`。
```
        public static void NewsSearchWithFilters(NewsSearchAPI client)
        {
            try
            {
                var newsResults = client.News.SearchAsync(query: "Artificial Intelligence", market: "en-us", freshness: "Week", sortBy: "Date").Result;
                Console.WriteLine("Search most recent news for query \"Artificial Intelligence\" with freshness and sortBy");

                if (newsResults == null)
                {
                    Console.WriteLine("Didn't see any news result data..");
                }
                else
                {
                    if (newsResults.Value.Count > 0)
                    {
                        var firstNewsResult = newsResults.Value.First();

                        Console.WriteLine($"\r\nTotalEstimatedMatches value: {newsResults.TotalEstimatedMatches}");
                        Console.WriteLine($"News result count: {newsResults.Value.Count}");
                        Console.WriteLine($"First news name: {firstNewsResult.Name}");
                        Console.WriteLine($"First news url: {firstNewsResult.Url}");
                        Console.WriteLine($"First news description: {firstNewsResult.Description}");
                        Console.WriteLine($"First news published time: {firstNewsResult.DatePublished}");
                        Console.WriteLine($"First news provider: {firstNewsResult.Provider.First().Name}");
                    }
                    else
                    {
                        Console.WriteLine("Couldn't find news results!");
                    }
                }
            }

            catch (Exception ex)
            {
                Console.WriteLine("Encountered exception. " + ex.Message);
            }
        }

```

## <a name="category-news-safe-search"></a>分類新聞、安全搜尋
下列程式碼使用安全搜尋，搜尋分類新聞中是否有電影和電視娛樂。  它會驗證結果數目，並印出第一個新聞結果的 `category`、`name`、`url`、`description`、`published time` 和提供者 `name`。
```
        public static void NewsCategory(NewsSearchAPI client)
        {
            try
            {
                var newsResults = client.News.CategoryAsync(category: "Entertainment_MovieAndTV", market: "en-us", safeSearch: "strict").Result;
                Console.WriteLine("Search category news for movie and TV entertainment with safe search");

                if (newsResults == null)
                {
                    Console.WriteLine("Didn't see any news result data..");
                }
                else
                {
                    if (newsResults.Value.Count > 0)
                    {
                        var firstNewsResult = newsResults.Value.First();

                        Console.WriteLine($"\r\nNews result count: {newsResults.Value.Count}");
                        Console.WriteLine($"First news category: {firstNewsResult.Category}");
                        Console.WriteLine($"First news name: {firstNewsResult.Name}");
                        Console.WriteLine($"First news url: {firstNewsResult.Url}");
                        Console.WriteLine($"First news description: {firstNewsResult.Description}");
                        Console.WriteLine($"First news published time: {firstNewsResult.DatePublished}");
                        Console.WriteLine($"First news provider: {firstNewsResult.Provider.First().Name}");
                    }
                    else
                    {
                        Console.WriteLine("Couldn't find news results!");
                    }
                }
            }

            catch (Exception ex)
            {
                Console.WriteLine("Encountered exception. " + ex.Message);
            }
        }

```
## <a name="trending-topics"></a>趨勢主題
下列程式碼會在 Bing 中搜尋新聞趨勢主題。 它會驗證結果數目，並印出第一個新聞結果的 `name`、`text of query`、`webSearchUrl`、`newsSearchUrl` 和 `image.Url`。
```
        public static void TrendingTopics(NewsSearchAPI client)
        {
            try
            {
                var trendingTopics = client.News.TrendingAsync(market: "en-us").Result;
                Console.WriteLine("Search news trending topics in Bing");

                if (trendingTopics == null)
                {
                    Console.WriteLine("Didn't see any news trending topics..");
                }
                else
                {
                    if (trendingTopics.Value.Count > 0)
                    {
                        var firstTopic = trendingTopics.Value.First();

                        Console.WriteLine($"\r\nTrending topics count: {trendingTopics.Value.Count}");
                        Console.WriteLine($"First topic name: {firstTopic.Name}");
                        Console.WriteLine($"First topic query: {firstTopic.Query.Text}");
                        Console.WriteLine($"First topic image url: {firstTopic.Image.Url}");
                        Console.WriteLine($"First topic webSearchUrl: {firstTopic.WebSearchUrl}");
                        Console.WriteLine($"First topic newsSearchUrl: {firstTopic.NewsSearchUrl}");
                    }
                    else
                    {
                        Console.WriteLine("Couldn't find news trending topics!");
                    }
                }
            }

            catch (Exception ex)
            {
                Console.WriteLine("Encountered exception. " + ex.Message);
            }
        }

```

## <a name="next-steps"></a>後續步驟

[認知服務 .NET SDK 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)