---
title: 快速入門：使用 Java 呼叫端點 - Bing 自訂搜尋
titlesuffix: Azure Cognitive Services
description: 本快速入門說明如何使用 Java 呼叫自訂的 Bing 搜尋端點，以從您的自訂搜尋執行個體要求搜尋結果。
services: cognitive-services
author: brapel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: quickstart
ms.date: 05/07/2018
ms.author: v-brapel
ms.openlocfilehash: ba1eb9799bc95dede081c6f1a6d972896c126da6
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48814105"
---
# <a name="quickstart-call-bing-custom-search-endpoint-java"></a>快速入門：呼叫 Bing 自訂搜尋端點 (Java)

此快速入門說明如何使用 Java 呼叫 Bing 自訂搜尋端點，以從您的自訂搜尋執行個體要求搜尋結果。 

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，您需要：

- 已經可以使用的自訂搜尋執行個體。 請參閱[建立您的第一個 Bing 自訂搜尋執行個體](quick-start.md)。
- 未安裝 [Java](https://www.java.com)。
- 訂用帳戶金鑰。 您可以在啟用[免費試用版](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search)時取得訂用帳戶金鑰，或者可以從 Azure 儀表板使用付費訂用帳戶金鑰 (請參閱[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account))。    

## <a name="run-the-code"></a>執行程式碼

若要執行此範例，請遵循下列步驟：

1. 使用您所選擇的 Java IDE 建立套件。  
  
2. 在套件中建立名為 CustomSrchJava.java 的檔案，然後將下列程式碼複製到其中。 將 **YOUR-SUBSCRIPTION-KEY** 與 **YOUR-CUSTOM-CONFIG-ID** 取代為您的訂用帳戶金鑰與設定識別碼。  
  
    ```java
    import java.io.InputStream;
    import java.net.URL;
    import java.net.URLEncoder;
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map;
    import java.util.Scanner;
    
    import javax.net.ssl.HttpsURLConnection;
    
    import com.google.gson.Gson;
    import com.google.gson.GsonBuilder;
    import com.google.gson.JsonObject;
    import com.google.gson.JsonParser;
    
    public class CustomSrchJava {       
        static String host = "https://api.cognitive.microsoft.com";
        static String path = "/bingcustomsearch/v7.0/search";
        static String subscriptionKey = "YOUR-SUBSCRIPTION-KEY"; 
        static String customConfigId = "YOUR-CUSTOM-CONFIG-ID";  
    
        static String searchTerm = "Microsoft";  // Replace with search term specific to your search scenario.
    
        public static SearchResults SearchWeb (String searchQuery) throws Exception {
            // construct URL of search request (endpoint + query string)
            URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchTerm, "UTF-8") + "&CustomConfig=" + customConfigId);
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
                System.out.println("Invalid Custom Search subscription key!");
                System.out.println("Please paste yours into the source code.");
                System.exit(1);
            }
    
            try {
                System.out.println("Searching your slice of the Web for: " + searchTerm);
    
                SearchResults result = SearchWeb(searchTerm);
    
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
    ```  
  
4. 執行程式。
    
## <a name="next-steps"></a>後續步驟
- [設定託管的 UI 體驗](./hosted-ui.md)
- [使用裝飾標記醒目提示文字](./hit-highlighting.md)
- [頁面網頁](./page-webpages.md)