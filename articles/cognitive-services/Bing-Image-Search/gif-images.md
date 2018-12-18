---
title: 取得 .gif 影像 - Microsoft 認知服務 | Microsoft Docs
description: 如何使用 Bing 影像搜尋 API 來取得更多 .gif 的相關資訊。
services: cognitive-services
author: MikeDodaro
manager: rosh
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: article
ms.date: 04/24/2018
ms.author: rosh, v-gedod
ms.openlocfilehash: 10e922b0cd15868bfe8f09b3846c76a368052e69
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370295"
---
# <a name="search-for-gif-images"></a>搜尋 .gif 影像
Bing 影像搜尋 API 可讓您同時跨整個網路搜尋最相關的 .gif 影像。  開發人員可以在各種交談情節中整合吸引人的 GIF。 

下列 URL 是用於動畫 .gif 影像的查詢。
````
https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=interesting&imageType=AnimatedGif&mkt=en-us
````
[q](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/bing-images-api-v7-reference#query) 參數指定搜尋字詞。  上述查詢還使用 [imageType](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/bing-images-api-v7-reference#imagetype) 篩選參數來指定 `animatedGif`。

若要查看結果的範例，請使用下列 URL 來搜尋 bing.com。
````
https://www.bing.com/images/search?q=interesting&qft=%20filterui%3Aphoto-animatedgif 

````
## <a name="query-parameters"></a>查詢參數

如需查詢參數和選項的詳細資訊，請參閱[影像搜尋 API 參考](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/bing-images-api-v7-reference#query-parameters)。 範例位在[使用 Java 搜尋動畫 GIF 的範例](#gifExample)標題底下。

## <a name="tips-and-suggestions"></a>祕訣與建議

- 您可以指定 [maxFileSize](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/bing-images-api-v7-reference#maxfilesize) 和 [minFileSize](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/bing-images-api-v7-reference#minfilesize) 參數。 建議您設定 maxFileSize=2000000，因為大部分的 GIF 在我們的索引中都低於 2 MB。  若有頻寬疑慮 (例如在行動電話案例中)，這也有助於控制資料大小。
- 若要協助改善認知的效能，請在載入來源 URL 之前先載入縮圖。  
- 若為尚未擁有使用者查詢的首次執行或登陸頁面體驗，請嘗試從[趨勢影像 API](trending-images.md) 使用我們的趨勢 GIF 搜尋來協助。
- [safeSearch](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/bing-images-api-v7-reference#safesearch) 參數有三個設定。  `strict` 選項會封鎖成人內容。 
- 如需支援之語言和位置的完整清單，請參閱 [mkt](supported-countries-markets.md)。
- *AnimatedGifHttps* 只會傳回來自 Https 位址的動畫 GIF 影像。 基於安全性，許多應用程式需要透過 Https 連線至外部網頁連結。 例如，Apple App Store 需要透過 HTTPS 連線至 Web 服務，這會在傳輸過程中加密使用者資料以維護安全。

<a name="gifExample" />
## <a name="example-search-for-animated-gif-using-java"></a>使用 Java 搜尋動畫 GIF 的範例

下列 URL 會搜尋動畫 .gif 影像：`q=interesting`
````
https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=interesting&imageType=AnimatedGif&mkt=en-us

````
如下列範例所示，此 URL 查詢需要 [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/bing-images-api-v7-reference#headers) 標頭。

下列 Java 範例會建置並傳送要求。

````
package gifSearch;
import java.net.*;
import java.util.*;
import java.io.*;
import javax.net.ssl.HttpsURLConnection;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

/*
 * Gson: https://github.com/google/gson
 * Maven info:
 *     groupId: com.google.code.gson
 *     artifactId: gson
 *     version: 2.8.1
 *
 * Once you have compiled or downloaded gson-2.8.1.jar, assuming you have placed it in the
 * same folder as this file (BingImageSearch.java), you can compile and run this program at
 * the command line as follows.
 *
 * javac GIFsearch.java -classpath .;gson-2.8.1.jar -encoding UTF-8
 * java -cp .;gson-2.8.1.jar GIFsearch
 */


public class GIFsearch {

    // Replace the subscriptionKey string value with your valid subscription key.
    static String subscriptionKey = "YOUR-ACCESS-KEY";

    static String host = "https://api.cognitive.microsoft.com";
    static String path = "/bing/v7.0/images/search";

    static String searchTerm = "interesting";

    public static SearchResults SearchImages (String searchQuery) throws Exception {
        // construct URL of search request (endpoint + query string)
        URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8") + "&imageType=AnimatedGif&mkt=en-us");
        HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
        connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

        // receive JSON body
        InputStream stream = connection.getInputStream();
        String response = new Scanner(stream).useDelimiter("\\A").next();

        // construct result object for return
        SearchResults results = new SearchResults(new HashMap<String, String>(), response);

        // extract Bing-related HTTP headers
        Map<String, List<String>> headers = connection.getHeaderFields();
        for (String header : headers.keySet()) {
            if (header == null) continue;      // may have null key
            if (header.startsWith("BingAPIs-") || header.startsWith("X-MSEdge-")) {
                results.relevantHeaders.put(header, headers.get(header).get(0));
            }
        }

        stream.close();
        return results;
    }

    // pretty-printer for JSON; uses GSON parser to parse and re-serialize
    public static String prettify(String json_text) {
        JsonParser parser = new JsonParser();
        JsonObject json = parser.parse(json_text).getAsJsonObject();
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }

    public static void main (String[] args) {
        if (subscriptionKey.length() != 32) {
            System.out.println("Invalid Bing Search API subscription key!");
            System.out.println("Please paste yours into the source code.");
            System.exit(1);
        }

        try {
            System.out.println("Searching the Web for: " + searchTerm);

            SearchResults result = SearchImages(searchTerm);

            System.out.println("\nRelevant HTTP Headers:\n");
            for (String header : result.relevantHeaders.keySet())
                System.out.println(header + ": " + result.relevantHeaders.get(header));

            System.out.println("\nJSON Response:\n");
            System.out.println(prettify(result.jsonResponse));
        }
        catch (Exception e) {
            e.printStackTrace(System.out);
            System.exit(1);
        }
    }

}

//Container class for search results encapsulates relevant headers and JSON data
class SearchResults{
 HashMap<String, String> relevantHeaders;
 String jsonResponse;
 SearchResults(HashMap<String, String> headers, String json) {
     relevantHeaders = headers;
     jsonResponse = json;
 }
}

````

## <a name="results"></a>結果
程式碼會取得下列結果作為 JSON 物件：

```json
    {
      "webSearchUrl": "https://www.bing.com/images/search?view\u003ddetai...",
      "name": "Very Interesting GIF - Thats Very Interesting - ...",
      "thumbnailUrl": "https://tse1.mm.bing.net/th?id\u003dOIP.yJX6Vz345JPK...",
      "datePublished": "2017-03-12T01:35:00.0000000Z",
      "contentUrl": "https://media.contoso.co/images/c895fa573df8e493ca8d0dec7d93b/raw",
      "hostPageUrl": "https://www.contoso.co/view/thats-very-interesting-christi...",
      "contentSize": "1295633 B",
      "encodingFormat": "animatedgif",
      "hostPageDisplayUrl": "https://www.contoso.co/view/thats-very-christian...",
      "width": 440,
      "height": 186,
      "thumbnail": {
        "width": 474,
        "height": 200
      },
      "imageInsightsToken": "ccid_yJX6Vz34*mid_9FF0FFA42AADA1357F042443D2103B40EA...",
      "insightsMetadata": {
        "recipeSourcesCount": 0,
        "bestRepresentativeQuery": {
          "text": "That\u0027s Very Interesting",
          "displayText": "That\u0027s Very Interesting",
          "webSearchUrl": "https://www.bing.com/images/search?q\u003dThat..."
        },
        "pagesIncludingCount": 19,
        "availableSizesCount": 2
      },
      "imageId": "9FF0FFA42AADA1357F042443D21030EAAA225F",
      "accentColor": "62452D"
    },

```

## <a name="next-steps"></a>後續步驟
- [C# 快速入門](quickstarts/csharp.md)
- [教學課程：影像搜尋單頁應用程式](tutorial-bing-image-search-single-page-app.md)