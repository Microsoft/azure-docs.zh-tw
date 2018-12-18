---
title: 快速入門：使用 C# 來執行影像搜尋 - Bing 影像搜尋 API
titleSuffix: Azure Cognitive Services
description: 使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並從 JSON 回應檢視搜尋結果。 這個簡單的 C# 應用程式會將 HTTP 影像搜尋查詢傳送給 API，並顯示所傳回第一個影像的 URL。
services: cognitive-services
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 9/07/2018
ms.author: aahi
ms.openlocfilehash: 897e380092b029855ac6c986c1126ca4b2d657a9
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46296633"
---
# <a name="quickstart-send-search-queries-using-the-bing-image-search-api-and-c"></a>快速入門：使用 Bing 影像搜尋 API 和 C# 來傳送搜尋查詢

使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並從 JSON 回應檢視搜尋結果。 這個簡單的 C# 應用程式會將 HTTP 影像搜尋查詢傳送給 API，並顯示所傳回第一個影像的 URL。

雖然此應用程式是以 C# 撰寫的，但 API 是一種與大多數程式設計語言都相容的 RESTful Web 服務。

[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Search/BingImageSearchv7Quickstart.cs) 上有此範例的原始程式碼，其中還有其他錯誤處理和註釋。

## <a name="prerequisites"></a>先決條件

* 任何一版 [Visual Studio 2017](https://www.visualstudio.com/downloads/)。
* [Json.NET](https://www.newtonsoft.com/json) 架構 (以 NuGet 套件形式提供)。
* 如果您使用 Linux/MacOS，則可以使用 [Mono](http://www.mono-project.com/)來執行此應用程式。

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-a-project"></a>建立專案並將其初始化

1. 在 Visual Studio 中建立一個名為 `BingSearchApisQuickStart` 的新主控台解決方案。 然後將下列命名空間新增至主要程式碼檔案。

    ```csharp
    using System;
    using System.Net;
    using System.IO;
    using System.Collections.Generic;
    using Newtonsoft.Json.Linq;
    ```

2. 建立適用於 API 端點、您訂用帳戶金鑰及搜尋字詞的變數。

    ```csharp
    //...
    namespace BingSearchApisQuickstart
    {
        class Program
        {
        // Replace the this string with your valid access key.
        const string subscriptionKey = "enter your key here";
        const string uriBase = "https://api.cognitive.microsoft.com/bing/v7.0/images/search";
        const string searchTerm = "tropical ocean";
    //...
    ```

## <a name="create-a-struct-to-format-the-bing-image-search-response"></a>建立結構來製作 Bing 影像搜尋回應的格式

定義 `SearchResult` 結構以包含影像搜尋結果及 JSON 標頭資訊。

```csharp
    namespace BingSearchApisQuickstart
    {
        class Program
        {
        //...
            struct SearchResult
            {
                public String jsonResult;
                public Dictionary<String, String> relevantHeaders;
            }
//...
```

## <a name="create-a-method-to-send-search-requests"></a>建立方法來傳送搜尋要求

建立一個名為 `BingImageSearch` 的方法來執行對 API 的呼叫，並將傳回類型設定為稍早建立的 `SearchResult` 結構。

```csharp
//...
namespace BingSearchApisQuickstart
{
    //...
    class Program
    {
        //...
        static SearchResult BingImageSearch(string searchTerm)
        {
        }
//...
```

## <a name="create-and-handle-an-image-search-request"></a>建立及處理影像搜尋要求

在 `BingImageSearch` 方法中，執行下列步驟。

1. 建構搜尋要求的 URI。 請注意，搜尋字詞 `toSearch` 必須先格式化，才能附加到字串。

    ```csharp
    static SearchResult BingImageSearch(string toSearch){

        var uriQuery = uriBase + "?q=" + Uri.EscapeDataString(toSearch);
    //...
    ```

2. 執行 Web 要求並取得 JSON 字串形式的回應。

    ```csharp
    WebRequest request = WebRequest.Create(uriQuery);
    request.Headers["Ocp-Apim-Subscription-Key"] = subscriptionKey;
    HttpWebResponse response = (HttpWebResponse)request.GetResponseAsync().Result;
    string json = new StreamReader(response.GetResponseStream()).ReadToEnd();
    ```

3. 建立搜尋結果物件並擷取 Bing HTTP 標頭。 然後傳回 `searchResult`。

    ```csharp
    // Create the result object for return
    var searchResult = new SearchResult()
    {
        jsonResult = json,
        relevantHeaders = new Dictionary<String, String>()
    };

    // Extract Bing HTTP headers
    foreach (String header in response.Headers)
    {
        if (header.StartsWith("BingAPIs-") || header.StartsWith("X-MSEdge-"))
            searchResult.relevantHeaders[header] = response.Headers[header];
    }
    return searchResult;
    ```

## <a name="process-and-view-the-response"></a>處理及檢視回應

1. 在主要方法中，呼叫 `BingImageSearch()` 並儲存傳回的回應。 然後將 JSON 還原序列化成物件。

    ```csharp
    SearchResult result = BingImageSearch(searchTerm);
    //deserialize the JSON response from the Bing Image Search API
    dynamic jsonObj = Newtonsoft.Json.JsonConvert.DeserializeObject(result.jsonResult);
    ```

2. 從 `jsonObj` 取得第一個傳回的影像，並列印出影像的標題和 URL。
    ```csharp
    var firstJsonObj = jsonObj["value"][0];
    Console.WriteLine("Title for the first image result: " + firstJsonObj["name"]+"\n");
    //After running the application, copy the output URL into a browser to see the image.
    Console.WriteLine("URL for the first image result: " + firstJsonObj["webSearchUrl"]+"\n");
    ```  

3. 務必從應用程式的程式碼中移除您的訂用帳戶金鑰。

## <a name="json-response"></a>JSON 回應

來自「Bing 影像搜尋 API」的回應會以 JSON 形式傳回。 此範例回應已截斷而只顯示單一結果。

```json
{
"_type":"Images",
"instrumentation":{
    "_type":"ResponseInstrumentation"
},
"readLink":"images\/search?q=tropical ocean",
"webSearchUrl":"https:\/\/www.bing.com\/images\/search?q=tropical ocean&FORM=OIIARP",
"totalEstimatedMatches":842,
"nextOffset":47,
"value":[
    {
        "webSearchUrl":"https:\/\/www.bing.com\/images\/search?view=detailv2&FORM=OIIRPO&q=tropical+ocean&id=8607ACDACB243BDEA7E1EF78127DA931E680E3A5&simid=608027248313960152",
        "name":"My Life in the Ocean | The greatest WordPress.com site in ...",
        "thumbnailUrl":"https:\/\/tse3.mm.bing.net\/th?id=OIP.fmwSKKmKpmZtJiBDps1kLAHaEo&pid=Api",
        "datePublished":"2017-11-03T08:51:00.0000000Z",
        "contentUrl":"https:\/\/mylifeintheocean.files.wordpress.com\/2012\/11\/tropical-ocean-wallpaper-1920x12003.jpg",
        "hostPageUrl":"https:\/\/mylifeintheocean.wordpress.com\/",
        "contentSize":"897388 B",
        "encodingFormat":"jpeg",
        "hostPageDisplayUrl":"https:\/\/mylifeintheocean.wordpress.com",
        "width":1920,
        "height":1200,
        "thumbnail":{
        "width":474,
        "height":296
        },
        "imageInsightsToken":"ccid_fmwSKKmK*mid_8607ACDACB243BDEA7E1EF78127DA931E680E3A5*simid_608027248313960152*thid_OIP.fmwSKKmKpmZtJiBDps1kLAHaEo",
        "insightsMetadata":{
        "recipeSourcesCount":0,
        "bestRepresentativeQuery":{
            "text":"Tropical Beaches Desktop Wallpaper",
            "displayText":"Tropical Beaches Desktop Wallpaper",
            "webSearchUrl":"https:\/\/www.bing.com\/images\/search?q=Tropical+Beaches+Desktop+Wallpaper&id=8607ACDACB243BDEA7E1EF78127DA931E680E3A5&FORM=IDBQDM"
        },
        "pagesIncludingCount":115,
        "availableSizesCount":44
        },
        "imageId":"8607ACDACB243BDEA7E1EF78127DA931E680E3A5",
        "accentColor":"0050B2"
    }
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 影像搜尋單頁應用程式教學課程](../tutorial-bing-image-search-single-page-app.md) (英文)

## <a name="see-also"></a>另請參閱

* [什麼是 Bing 影像搜尋？](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)  
* [試用線上互動式示範](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [取得免費認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-image-search-api)  
* [Azure 認知服務文件](https://docs.microsoft.com/azure/cognitive-services)
* [Bing 影像搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
