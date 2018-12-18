---
title: 快速入門：使用適用於 C# 的 Bing Web 搜尋 SDK
titleSuffix: Azure Cognitive Services
description: Bing Web 搜尋 SDK 可讓您輕鬆地將 Bing Web 搜尋整合到 C# 應用程式。 在本快速入門中，您將了解如何具現化用戶端、傳送要求，以及列印回應。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: quickstart
ms.date: 08/16/2018
ms.author: erhopf
ms.openlocfilehash: 04a70c2afbd499d3965e23487cd49540b38e08af
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318651"
---
# <a name="quickstart-use-the-bing-web-search-sdk-for-c"></a>快速入門：使用適用於 C# 的 Bing Web 搜尋 SDK

Bing Web 搜尋 SDK 可讓您輕鬆地將 Bing Web 搜尋整合到 C# 應用程式。 在本快速入門中，您將了解如何具現化用戶端、傳送要求，以及列印回應。

[!INCLUDE [bing-web-search-quickstart-signup](../../../includes/bing-web-search-quickstart-signup.md)]

## <a name="prerequisites"></a>必要條件

以下是執行本快速入門之前的幾個必備項目：

* [Visual Studio](https://visualstudio.microsoft.com/downloads/) 或
* [Visual Studio Code 2017](https://code.visualstudio.com/download) (英文)
  * [C# for Visual Studio Code](https://visualstudio.microsoft.com/downloads/)
  * [NuGet 套件管理員](https://github.com/jmrog/vscode-nuget-package-manager) (英文)
* [.Net Core SDK](https://www.microsoft.com/net/download) (英文)

## <a name="create-a-project-and-install-dependencies"></a>建立專案並安裝相依性

> [!TIP]
> 從 [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/) 取得最新程式碼作為 Visual Studio 解決方案。

第一個步驟是建立新的主控台專案。 關於設定主控台專案，如需協助，請參閱 [Hello World - 您的第一個程式 (C# 程式設計手冊)](https://docs.microsoft.com/dotnet/csharp/programming-guide/inside-a-program/hello-world-your-first-program)。 若要在您的應用程式中使用 Bing Web 搜尋 SDK，您必須使用 NuGet 套件管理員安裝 `Microsoft.Azure.CognitiveServices.Search.WebSearch`。

[Web 搜尋 SDK 套件](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.WebSearch/1.2.0) (英文) 也會安裝：

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

## <a name="declare-dependencies"></a>宣告相依性

在 Visual Studio 或 Visual Studio Code 中開啟您的專案，並匯入這些相依性：

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Azure.CognitiveServices.Search.WebSearch;
using Microsoft.Azure.CognitiveServices.Search.WebSearch.Models;
using System.Linq;
```

## <a name="create-project-scaffolding"></a>建立專案 Scaffolding

當您建立新的主控台專案時，應該就已建立應用程式的命名空間和類別。 您的程式看起來應該像這樣：

```csharp
namespace WebSearchSDK
{
    class YOUR_PROGRAM
    {

        // The code in the following sections goes here.

    }
}
```

在下列章節中，我們會在此類別內建置我們的範例應用程式。

## <a name="construct-a-request"></a>建構要求

此程式碼會建構搜尋查詢。

```csharp
public static void WebResults(WebSearchAPI client)
{
    try
    {
        var webData = client.Web.Search(query: "Yosemite National Park");
        Console.WriteLine("Searching for \"Yosemite National Park\"");

        // Code for handling responses is provided in the next section...

    }
    catch (Exception ex)
    {
        Console.WriteLine("Encountered exception. " + ex.Message);
    }
}
```

## <a name="handle-the-response"></a>處理回應

接下來，讓我們新增一些程式碼來剖析回應並列印結果。 若第一個網頁、影像、新聞文章和影片存在於回應物件中，則會列印其 `name` 和 `url`。

```csharp
if (webData?.WebPages?.Value?.Count > 0)
{
    // find the first web page
    var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

    if (firstWebPagesResult != null)
    {
        Console.WriteLine("Webpage Results # {0}", webData.WebPages.Value.Count);
        Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
        Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
    }
    else
    {
        Console.WriteLine("Didn't find any web pages...");
    }
}
else
{
    Console.WriteLine("Didn't find any web pages...");
}

/*
 * Images
 * If the search response contains images, the first result's name
 * and url are printed.
 */
if (webData?.Images?.Value?.Count > 0)
{
    // find the first image result
    var firstImageResult = webData.Images.Value.FirstOrDefault();

    if (firstImageResult != null)
    {
        Console.WriteLine("Image Results # {0}", webData.Images.Value.Count);
        Console.WriteLine("First Image result name: {0} ", firstImageResult.Name);
        Console.WriteLine("First Image result URL: {0} ", firstImageResult.ContentUrl);
    }
    else
    {
        Console.WriteLine("Didn't find any images...");
    }
}
else
{
    Console.WriteLine("Didn't find any images...");
}

/*
 * News
 * If the search response contains news articles, the first result's name
 * and url are printed.
 */
if (webData?.News?.Value?.Count > 0)
{
    // find the first news result
    var firstNewsResult = webData.News.Value.FirstOrDefault();

    if (firstNewsResult != null)
    {
        Console.WriteLine("\r\nNews Results # {0}", webData.News.Value.Count);
        Console.WriteLine("First news result name: {0} ", firstNewsResult.Name);
        Console.WriteLine("First news result URL: {0} ", firstNewsResult.Url);
    }
    else
    {
        Console.WriteLine("Didn't find any news articles...");
    }
}
else
{
    Console.WriteLine("Didn't find any news articles...");
}

/*
 * Videos
 * If the search response contains videos, the first result's name
 * and url are printed.
 */
if (webData?.Videos?.Value?.Count > 0)
{
    // find the first video result
    var firstVideoResult = webData.Videos.Value.FirstOrDefault();

    if (firstVideoResult != null)
    {
        Console.WriteLine("\r\nVideo Results # {0}", webData.Videos.Value.Count);
        Console.WriteLine("First Video result name: {0} ", firstVideoResult.Name);
        Console.WriteLine("First Video result URL: {0} ", firstVideoResult.ContentUrl);
    }
    else
    {
        Console.WriteLine("Didn't find any videos...");
    }
}
else
{
    Console.WriteLine("Didn't find any videos...");
}
```

## <a name="declare-the-main-method"></a>宣告 main 方法

在此應用程式中，main 方法包含可具現化用戶端、驗證 `subscriptionKey` 及呼叫 `WebResults` 的程式碼。 請務必輸入 Azure 帳戶的有效訂用帳戶金鑰再繼續。

```csharp
static void Main(string[] args)
{
    var client = new WebSearchAPI(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

    WebResults(client);

    Console.WriteLine("Press any key to exit...");
    Console.ReadKey();
}
```

## <a name="run-the-application"></a>執行應用程式

讓我們執行應用程式！

```console
dotnet run
```

## <a name="define-functions-and-filter-results"></a>定義函式和篩選結果

既然您已完成第一次呼叫 Bing Web 搜尋 API，讓我們看看一些函式，以突顯 SDK 在改善查詢和篩選結果方面的實用性。 每個函式都可以新增至上一節所建立的 C# 應用程式。

### <a name="limit-the-number-of-results-returned-by-bing"></a>限制 Bing 所傳回的結果數目

這個範例會使用 `count` 和 `offset` 參數，來限制「西雅圖最佳餐廳」傳回的結果數目。 第一個結果的 `name` 和 `URL` 會列印出來。

1. 將此程式碼新增至主控台專案：
    ```csharp
    public static void WebResultsWithCountAndOffset(WebSearchAPI client)
    {
        try
        {
            var webData = client.Web.SearchAsync(query: "Best restaurants in Seattle", offset: 10, count: 20).Result;
            Console.WriteLine("\r\nSearching for \" Best restaurants in Seattle \"");

            if (webData?.WebPages?.Value?.Count > 0)
            {
                var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

                if (firstWebPagesResult != null)
                {
                    Console.WriteLine("Web Results #{0}", webData.WebPages.Value.Count);
                    Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
                    Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
                }
                else
                {
                    Console.WriteLine("Couldn't find first web result!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any Web data..");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```
2. 將 `WebResultsWithCountAndOffset` 新增至 `main`：
    ```csharp
    static void Main(string[] args)
    {
        var client = new WebSearchAPI(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        WebResults(client);
        // Search with count and offset...
        WebResultsWithCountAndOffset(client);  

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```
3. 執行應用程式。

### <a name="filter-for-news"></a>篩選新聞

這個範例會使用 `response_filter` 參數來篩選搜尋結果。 傳回的搜尋結果僅限於有關 "Microsoft" 的新聞文章。 第一個結果的 `name` 和 `URL` 會列印出來。

1. 將此程式碼新增至主控台專案：
    ```csharp
    public static void WebSearchWithResponseFilter(WebSearchAPI client)
    {
        try
        {
            IList<string> responseFilterstrings = new List<string>() { "news" };
            var webData = client.Web.SearchAsync(query: "Microsoft", responseFilter: responseFilterstrings).Result;
            Console.WriteLine("\r\nSearching for \" Microsoft \" with response filter \"news\"");

            if (webData?.News?.Value?.Count > 0)
            {
                var firstNewsResult = webData.News.Value.FirstOrDefault();

                if (firstNewsResult != null)
                {
                    Console.WriteLine("News Results #{0}", webData.News.Value.Count);
                    Console.WriteLine("First news result name: {0} ", firstNewsResult.Name);
                    Console.WriteLine("First news result URL: {0} ", firstNewsResult.Url);
                }
                else
                {
                    Console.WriteLine("Couldn't find first News results!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any News data..");
            }

        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```
2. 將 `WebResultsWithCountAndOffset` 新增至 `main`：
    ```csharp
    static void Main(string[] args)
    {
        var client = new WebSearchAPI(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        WebResults(client);
        // Search with count and offset...
        WebResultsWithCountAndOffset(client);  
        // Search with news filter...
        WebSearchWithResponseFilter(client);

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```
3. 執行應用程式。

### <a name="use-safe-search-answer-count-and-the-promote-filter"></a>使用安全搜尋、回應計數和宣傳篩選條件

這個範例使用 `answer_count`、`promote` 和 `safe_search` 參數，來篩選 "音樂影片" 的搜尋結果。 第一個結果的 `name` 和 `URL` 會出現。

1. 將此程式碼新增至主控台專案：
    ```csharp
    public static void WebSearchWithAnswerCountPromoteAndSafeSearch(WebSearchAPI client)
    {
        try
        {
            IList<string> promoteAnswertypeStrings = new List<string>() { "videos" };
            var webData = client.Web.SearchAsync(query: "Music Videos", answerCount: 2, promote: promoteAnswertypeStrings, safeSearch: SafeSearch.Strict).Result;
            Console.WriteLine("\r\nSearching for \"Music Videos\"");

            if (webData?.Videos?.Value?.Count > 0)
            {
                var firstVideosResult = webData.Videos.Value.FirstOrDefault();

                if (firstVideosResult != null)
                {
                    Console.WriteLine("Video Results # {0}", webData.Videos.Value.Count);
                    Console.WriteLine("First Video result name: {0} ", firstVideosResult.Name);
                    Console.WriteLine("First Video result URL: {0} ", firstVideosResult.ContentUrl);
                }
                else
                {
                    Console.WriteLine("Couldn't find videos results!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any data..");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```
2. 將 `WebResultsWithCountAndOffset` 新增至 `main`：
    ```csharp
    static void Main(string[] args)
    {
        var client = new WebSearchAPI(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        WebResults(client);
        // Search with count and offset...
        WebResultsWithCountAndOffset(client);  
        // Search with news filter...
        WebSearchWithResponseFilter(client);
        // Search with answer count, promote, and safe search
        WebSearchWithAnswerCountPromoteAndSafeSearch(client);

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```
3. 執行應用程式。

## <a name="clean-up-resources"></a>清除資源

此專案使用完畢時，請務必從應用程式的程式碼中移除訂用帳戶金鑰。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [認知服務 Node.js SDK 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/) (英文)
