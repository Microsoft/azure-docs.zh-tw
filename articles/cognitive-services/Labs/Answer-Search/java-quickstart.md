---
title: 快速入門：專案答案搜尋 Java
titlesuffix: Azure Cognitive Services
description: 在 Java 中開始使用專案答案搜尋。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-answer-search
ms.topic: quickstart
ms.date: 04/13/2018
ms.author: rosh
ms.openlocfilehash: 1e10ecf685aaf5b60b94a5e9e0d8e118f45e5168
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48867020"
---
# <a name="quickstart-project-answer-search-query-in-java"></a>快速入門：Java 中的專案答案搜尋查詢
本文使用 Java 示範如何在 Azure 上使用屬於 Microsoft 認知服務的 Bing 答案搜尋 API。 API 是一種與任何程式語言相容的 REST Web 服務，可產生 HTTP 要求及剖析 JSON。
 
範例程式碼會使用 Java 和基本的外部相依性。  您也可以使用 Mono 在 Linux 或 Mac OS X 上加以執行。

## <a name="prerequisites"></a>必要條件

取得免費試用版[認知服務實驗室](https://aka.ms/answersearchsubscription)的存取金鑰

## <a name="request"></a>要求 

下列程式碼會建立 `WebRequest`、設定存取金鑰標頭，並新增 "Gibraltar" 的查詢字串。  接著，它會傳送要求，並為字串指派包含 JSON 文字的回應。

````
    static String host = "https://api.labs.cognitive.microsoft.com";
    static String path = "/answerSearch/v7.0/search";

    // construct URL of search request (endpoint + query string)

    URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8") + &mkt=en-us");
    HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
    connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

    // receive JSON body
    InputStream stream = connection.getInputStream();
    String response = new Scanner(stream).useDelimiter("\\A").next();

    // construct result object for return
    SearchResults results = new SearchResults(new HashMap<String, String>(), response);
````

## <a name="complete-code"></a>完整程式碼

Bing 答案搜尋 API 會從 Bing 搜尋引擎傳回結果。
1. 下載或安裝 gson 程式庫。
2. 在您最愛的 IDE 或編輯器中建立新的 Java 專案。
3. 新增下方提供的程式碼。
4. 將 subscriptionKey 值取代為您訂用帳戶的有效存取金鑰。
5. 執行程式。

````
package knowledgeAPI;
import java.io.InputStream;
import java.net.*;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Scanner;

import javax.net.ssl.HttpsURLConnection;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

public class KnowledgeSrch {

        // Replace the subscriptionKey string value with your valid subscription key.
        static String subscriptionKey = "YOUR-ACCESS-KEY";

        static String host = "https://api.labs.cognitive.microsoft.com";
        static String path = "/answerSearch/v7.0/search";

        static String searchTerm = "Gibraltar";

        public static SearchResults SearchKnowledge (String searchQuery) throws Exception {

            URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8") + "&mkt=en-us");
            
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
            try {
                System.out.println("Searching the Web for: " + searchTerm);

                SearchResults result = SearchKnowledge(searchTerm);

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

    // Container class for search results encapsulates relevant headers and JSON data
    class SearchResults{
        HashMap<String, String> relevantHeaders;
        String jsonResponse;
        SearchResults(HashMap<String, String> headers, String json) {
            relevantHeaders = headers;
            jsonResponse = json;
        }
}

````

## <a name="next-steps"></a>後續步驟
- [C# 快速入門](c-sharp-quickstart.md)
- [Java 快速入門](java-quickstart.md)
- [Node 快速入門](node-quickstart.md)