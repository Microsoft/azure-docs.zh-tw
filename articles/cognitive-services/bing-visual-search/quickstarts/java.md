---
title: 快速入門：建立圖像式搜尋查詢 (Java) - Bing 圖像式搜尋
titleSuffix: Azure Cognitive Services
description: 如何將影像上傳到 Bing 圖像式搜尋 API，並取得影像的深入解析。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.technology: bing-visual-search
ms.topic: quickstart
ms.date: 5/16/2018
ms.author: scottwhi
ms.openlocfilehash: 56e1b943f03128fa6703a7b15bd0d6ade09089d6
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222619"
---
# <a name="quickstart-your-first-bing-visual-search-query-in-java"></a>快速入門：使用 Java 的第一個 Bing 圖像式搜尋查詢

Bing 圖像式搜尋 API 會傳回您所提供影像的相關資訊。 您可以使用影像的 URL、見解權杖，或上傳影像來提供影像。 如需這些選項的資訊，請參閱[什麼是 Bing 圖像式搜尋 API？](../overview.md) 本文將示範如何上傳影像。 在拍攝知名地標的照片並取回其相關資訊的行動裝置案例中，上傳影像可能很有用。 例如，見解可能包含關於地標的雜項。 

若您上傳本機影像，以下會顯示您必須包含在 POST 本文中的表單資料。 表單資料必須包含 Content-Disposition 標頭。 其 `name` 參數必須設定為 "image"，而 `filename` 參數可以設定為任何字串。 表單的內容是影像的二進位檔案。 您可以上傳的影像大小上限為 1 MB。 

```
--boundary_1234-abcd
Content-Disposition: form-data; name="image"; filename="myimagefile.jpg"

ÿØÿà JFIF ÖÆ68g-¤CWŸþ29ÌÄøÖ‘º«™æ±èuZiÀ)"óÓß°Î= ØJ9á+*G¦...

--boundary_1234-abcd--
```

本文包含簡單的主控台應用程式，它會傳送 Bing 圖像式搜尋 API 要求，並顯示 JSON 搜尋結果。 雖然此應用程式是以 Java 撰寫，但 API 是一種與任何程式語言相容的 RESTful Web 服務，可產生 HTTP 要求，並剖析 JSON。 


## <a name="prerequisites"></a>必要條件

您將需要有 [JDK 7 或 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，才能編譯和執行此程式碼。 若您有特別喜愛的 Java IDE 也可以使用，但是文字編輯器就夠了。

在本快速入門中，您可以使用[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)的訂用帳戶金鑰或付費的訂用帳戶金鑰。

## <a name="running-the-application"></a>執行應用程式

以下示範如何使用 Java 中的 MultipartEntityBuilder 來上傳影像。

若要執行此應用程式，請遵循下列步驟：

1. 下載或安裝 [gson 程式庫](https://github.com/google/gson)。 您也可以透過 Maven 取得。
2. 在您愛用的 IDE 或編輯器中建立新的 Java 專案。
3. 在名為 `VisualSearch.java` 的檔案中新增提供的程式碼。
4. 以您的訂用帳戶金鑰取代 `subscriptionKey` 值。
4. 以要上傳之影像的路徑取代 `imagePath` 值。
5. 執行程式。


```java
package uploadimage;

import java.util.*;
import java.io.*;

/*
 * Gson: https://github.com/google/gson
 * Maven info:
 *     groupId: com.google.code.gson
 *     artifactId: gson
 *     version: 2.8.2
 *
 * Once you have compiled or downloaded gson-2.8.2.jar, assuming you have placed it in the
 * same folder as this file (BingImageSearch.java), you can compile and run this program at
 * the command line as follows.
 *
 * javac BingImageSearch.java -classpath .;gson-2.8.2.jar -encoding UTF-8
 * java -cp .;gson-2.8.2.jar BingImageSearch
 */

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

// http://hc.apache.org/downloads.cgi (HttpComponents Downloads) HttpClient 4.5.5

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;


public class UploadImage2 {

    static String endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch";
    static String subscriptionKey = "<yoursubscriptionkeygoeshere";
    static String imagePath = "<pathtoyourimagetouploadgoeshere>";

    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();
        
        try {
            HttpEntity entity = MultipartEntityBuilder
                .create()
                .addBinaryBody("image", new File(imagePath))
                .build();

            HttpPost httpPost = new HttpPost(endpoint);
            httpPost.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);
            httpPost.setEntity(entity);
            HttpResponse response = httpClient.execute(httpPost);

            InputStream stream = response.getEntity().getContent();
            String json = new Scanner(stream).useDelimiter("\\A").next();

            System.out.println("\nJSON Response:\n");
            System.out.println(prettify(json));
        }
        catch (IOException e)
        {
            e.printStackTrace(System.out);
            System.exit(1);
        }
        catch (Exception e) {
            e.printStackTrace(System.out);
            System.exit(1);
        }
    }
    
    // pretty-printer for JSON; uses GSON parser to parse and re-serialize
    public static String prettify(String json_text) {
        JsonParser parser = new JsonParser();
        JsonObject json = parser.parse(json_text).getAsJsonObject();
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }
    
}
```

## <a name="next-steps"></a>後續步驟

[使用見解權杖取得影像的相關見解](../use-insights-token.md)  
[Bing 圖像式搜尋影像上傳教學課程](../tutorial-visual-search-image-upload.md)
[Bing 圖像式搜尋單頁應用程式教學課程](../tutorial-bing-visual-search-single-page-app.md)  
[Bing 圖像式搜尋概觀](../overview.md)  
[試試看](https://aka.ms/bingvisualsearchtryforfree)  
[取得免費試用的存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-visual-search-api)  
[Bing 圖像式搜尋 API 參考](https://aka.ms/bingvisualsearchreferencedoc)

