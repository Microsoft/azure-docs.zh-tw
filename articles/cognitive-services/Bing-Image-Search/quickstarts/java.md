---
title: 快速入門：使用 Java 來執行影像搜尋 - Bing 影像搜尋 API
titleSuffix: Azure Cognitive Services
description: 使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 Java 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。
services: cognitive-services
documentationcenter: ''
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 8/20/2018
ms.author: aahi
ms.openlocfilehash: 5e002de273d50de9095c6e58b8e1a9fb9a05ed4f
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295337"
---
# <a name="quickstart-send-search-queries-using-the-bing-image-search-rest-api-and-java"></a>快速入門：使用 Bing 影像搜尋 REST API 和 Java 來傳送搜尋查詢

使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 Java 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。

雖然此應用程式是以 Java 撰寫的，但 API 是一種與大多數程式設計語言都相容的 RESTful Web 服務。

[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/java/Search/BingImageSearchv7Quickstart.java) 上有此範例的原始程式碼，其中還有其他錯誤處理和註釋。

## <a name="prerequisites"></a>先決條件

* [Java 開發套件 (JDK) 7 或 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

* [Gson 程式庫](https://github.com/google/gson)

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-a-project"></a>建立專案並將其初始化

1. 在您最愛的 IDE 或編輯器中建立新的 Java 專案，並匯入下列程式庫。

    ```java
    import java.net.*;
    import java.util.*;
    import java.io.*;
    import javax.net.ssl.HttpsURLConnection;
    import com.google.gson.Gson;
    import com.google.gson.GsonBuilder;
    import com.google.gson.JsonObject;
    import com.google.gson.JsonParser;
    ```

2. 建立適用於 API 端點、您訂用帳戶金鑰及搜尋字詞的變數。

    ```java
    static String subscriptionKey = "enter key here";
    static String host = "https://api.cognitive.microsoft.com";
    static String path = "/bing/v7.0/images/search";
    static String searchTerm = "tropical ocean";
    ```

## <a name="construct-the-search-request-and-query"></a>建構搜尋要求和查詢

1. 使用上一個步驟中的變數來製作適用於 API 要求的搜尋 URL 格式。 請注意，您的搜尋字詞必須先進行 URL 編碼，才能附加至要求。

    ```java
    // construct the search request URL (in the form of endpoint + query string)
    URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8"));
    HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
    connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
    ```

## <a name="receive-and-process-the-json-response"></a>接收及處理 JSON 回應

1. 接收來自「Bing 影像搜尋 API」的 JSON 回應，並建構結果物件。

    ```java
    // receive JSON body
    InputStream stream = connection.getInputStream();
    String response = new Scanner(stream).useDelimiter("\\A").next();
    // construct result object for return
    SearchResults results = new SearchResults(new HashMap<String, String>(), response);
    ```
2. 將 Bing 相關的 HTTP 標頭與 JSON 本文分開
    ```java
    // extract Bing-related HTTP headers
    Map<String, List<String>> headers = connection.getHeaderFields();
    for (String header : headers.keySet()) {
        if (header == null) continue;      // may have null key
        if (header.startsWith("BingAPIs-") || header.startsWith("X-MSEdge-")) {
            results.relevantHeaders.put(header, headers.get(header).get(0));
        }
    }
    ```

3. 關閉資料流並剖析回應。 取得所傳回搜尋結果的總數，以及第一個影像結果的縮圖 URL。

    ```java
    stream.close();
    JsonParser parser = new JsonParser();
    JsonObject json = parser.parse(result.jsonResponse).getAsJsonObject();
    //get the first image result from the JSON object, along with the total
    //number of images returned by the Bing Image Search API.
    String total = json.get("totalEstimatedMatches").getAsString();
    JsonArray results = json.getAsJsonArray("value");
    JsonObject first_result = (JsonObject)results.get(0);
    String resultURL = first_result.get("thumbnailUrl").getAsString();
    ```
4. 記得從應用程式程式碼中移除您的訂用帳戶金鑰。

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
