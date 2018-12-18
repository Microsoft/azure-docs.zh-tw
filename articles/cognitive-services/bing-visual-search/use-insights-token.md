---
title: 使用深入解析權杖 - Bing 圖像式搜尋
titleSuffix: Azure Cognitive Services
description: 說明如何搭配 Bing 圖像式搜尋 API 使用影像的深入解析權杖取得影像的相關深入解析。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.technology: bing-visual-search
ms.topic: conceptual
ms.date: 5/16/2018
ms.author: scottwhi
ms.openlocfilehash: e853545e07709d82fa1e3b1143392e6d6012f6f6
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227583"
---
# <a name="using-an-insights-token-to-get-insights-about-an-image"></a>使用見解權杖取得影像的相關見解

Bing 圖像式搜尋 API 會傳回您所提供影像的相關資訊。 您可以使用影像的 URL、見解權杖，或上傳影像來提供影像。 如需這些選項的資訊，請參閱[什麼是 Bing 圖像式搜尋 API？](overview.md) 本文將示範如何使用見解權杖。 如需示範如何上傳影像以取得見解的範例，請參閱快速入門 ([C#](quickstarts\csharp.md) | [Java](quickstarts\java.md) | [Node.js](quickstarts\nodejs.md) | [Python](quickstarts\python.md))。


若您將影像權杖或 URL 傳送至圖像式搜尋，以下會顯示您必須包含在 POST 本文中的表單資料。 表單資料必須包含 Content-Disposition 標頭，而且其 `name` 參數必須設定為 "knowledgeRequest"。 如需 `imageInfo` 物件的詳細資料，請參閱[要求](#the-request)。

```json
{
    "imageInfo" : {
        "url" : "",
        "imageInsightsToken" : "",
        "cropArea" : {
            "top" : 0.1,
            "left" : 0.5,
            "right" : 0.9,
            "bottom" : 0.9
        }
    },
    "knowledgeRequest" : {
      "filters" : {
        "site" : ""
      }
    }
}
```

本文中的範例將示範如何使用見解權杖。 您可以從 /images/search API 回應中的影像物件取得見解權杖。 如需取得見解權杖的資訊，請參閱 [Bing 影像搜尋 API](../Bing-Image-Search/overview.md)。

```
--boundary_1234-abcd
Content-Disposition: form-data; name="knowledgeRequest"

{
    "imageInfo" : {
        "imageInsightsToken" : "ccid_tmaGQ2eU*mid_D12339146CFEDF3D40...",
    }
}

--boundary_1234-abcd--
```


如需使用見解權杖的範例，請參閱 [C#](#using-csharp) | [Java](#using-java) | [Node.js](#using-nodejs) | [Python](#using-python)。

<a name="using-csharp" />

## <a name="using-c"></a>使用 C#

### <a name="prerequisites"></a>必要條件

您將需要 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 以在 Windows 上執行此程式碼。 (可使用免費的 Community Edition)。

在本快速入門中，您可以使用[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)的訂用帳戶金鑰或付費的訂用帳戶金鑰。

## <a name="running-the-application"></a>執行應用程式

若要執行此應用程式，請遵循下列步驟：

1. 在 Visual Studio 中建立新的主控台解決方案。
1. 以本快速入門中所示的程式碼取代 `Program.cs` 的內容。
2. 以您的訂用帳戶金鑰取代 `accessKey` 值。
2. 以 /images/search 回應中的見解權杖取代 `insightsToken` 值。
3. 執行程式。

```csharp
using System;
using System.Text;
using System.Net;
using System.IO;
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using System.Threading;

namespace VisualSearchInsightsToken
{

    class Program
    {
        // Replace the accessKey string value with your valid subscription key.
        const string accessKey = "<yoursubscriptionkeygoeshere>";

        const string uriBase = "https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch";

        // Update with an insights token from an image object that the /images/search API endpoint returns.
        static string insightsToken = @"ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ";

        static string BoundaryTemplate = "batch_{0}";

        static void Main(string[] args)
        {
            try { 
                Console.WriteLine("Getting image insights using insights token: " + insightsToken);

                var knowledgeRequest = "{\"imageInfo\" : {\"imageInsightsToken\" : \"" + insightsToken + "\"}}";
                var boundary = string.Format(BoundaryTemplate, Guid.NewGuid());

                var json = BingVisualSearch(knowledgeRequest, boundary, uriBase, accessKey);

                Console.WriteLine("\nJSON Response:\n");
                Console.WriteLine(JsonPrettyPrint(json));

                Console.Write("\nPress Enter to exit ");
                Console.ReadLine();
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }

        }


        /// <summary>
        /// Calls the Bing visual search endpoint and returns the JSON response with insights.
        /// </summary>
        static string BingVisualSearch(string knowledgeRequest, string boundary, string uri, string subscriptionKey)
        {
            var requestMessage = new HttpRequestMessage(HttpMethod.Post, uri);
            requestMessage.Headers.Add("Ocp-Apim-Subscription-Key", accessKey);

            var content = new MultipartFormDataContent(boundary);
            content.Add(new StringContent(knowledgeRequest, Encoding.UTF8, "application/json"), "knowledgeRequest");
            requestMessage.Content = content;

            var httpClient = new HttpClient();

            Task<HttpResponseMessage> httpRequest = httpClient.SendAsync(requestMessage, HttpCompletionOption.ResponseContentRead, CancellationToken.None);
            HttpResponseMessage httpResponse = httpRequest.Result;
            HttpStatusCode statusCode = httpResponse.StatusCode;
            HttpContent responseContent = httpResponse.Content;

            string json = null;

            if (responseContent != null)
            {
                Task<String> stringContentsTask = responseContent.ReadAsStringAsync();
                json = stringContentsTask.Result;
            }


            return json;
        }


        /// <summary>
        /// Formats the given JSON string by adding line breaks and indents.
        /// </summary>
        /// <param name="json">The raw JSON string to format.</param>
        /// <returns>The formatted JSON string.</returns>
        static string JsonPrettyPrint(string json)
        {
            if (string.IsNullOrEmpty(json))
                return string.Empty;

            json = json.Replace(Environment.NewLine, "").Replace("\t", "");

            StringBuilder sb = new StringBuilder();
            bool quote = false;
            bool ignore = false;
            char last = ' ';
            int offset = 0;
            int indentLength = 2;

            foreach (char ch in json)
            {
                switch (ch)
                {
                    case '"':
                        if (!ignore) quote = !quote;
                        break;
                    case '\\':
                        if (quote && last != '\\') ignore = true;
                        break;
                }

                if (quote)
                {
                    sb.Append(ch);
                    if (last == '\\' && ignore) ignore = false;
                }
                else
                {
                    switch (ch)
                    {
                        case '{':
                        case '[':
                            sb.Append(ch);
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', ++offset * indentLength));
                            break;
                        case '}':
                        case ']':
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', --offset * indentLength));
                            sb.Append(ch);
                            break;
                        case ',':
                            sb.Append(ch);
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', offset * indentLength));
                            break;
                        case ':':
                            sb.Append(ch);
                            sb.Append(' ');
                            break;
                        default:
                            if (quote || ch != ' ') sb.Append(ch);
                            break;
                    }
                }
                last = ch;
            }

            return sb.ToString().Trim();
        }
    }
}
```

<a name="using-java" />

## <a name="using-java"></a>使用 Java

### <a name="prerequisites"></a>必要條件

您將需要有 [JDK 7 或 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，才能編譯和執行此程式碼。 若您有特別喜愛的 Java IDE 也可以使用，但是文字編輯器就夠了。

在本快速入門中，您可以使用[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)的訂用帳戶金鑰或付費的訂用帳戶金鑰。

## <a name="running-the-application"></a>執行應用程式

若要執行此應用程式，請遵循下列步驟：

1. 下載或安裝 [gson 程式庫](https://github.com/google/gson)。 您也可以透過 Maven 取得。
2. 在您愛用的 IDE 或編輯器中建立新的 Java 專案。
3. 在名為 `VisualSearch.java` 的檔案中新增提供的程式碼。
4. 以您的訂用帳戶金鑰取代 `subscriptionKey` 值。
5. 執行程式。

```java
package insightstoken;

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


public class InsightsToken {

    
    static String endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch";
    static String subscriptionKey = "<yoursubscriptionkeygoeshere>";

    // To get an insights, call the /images/search endpoint. Get the token from
    // the imageInsightsToken field in the Image object.
    static String insightsToken = "ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ";

    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();
        
        String knowledgeRequest = "{\"imageInfo\" : {\"imageInsightsToken\" : \"" + insightsToken + "\"}}";

        try {
            HttpEntity entity = MultipartEntityBuilder
                .create()
                .addTextBody("knowledgeRequest", knowledgeRequest, ContentType.create("application/json"))
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


<a name="using-nodejs" />

## <a name="using-nodejs"></a>使用 Node.js

### <a name="prerequisites"></a>必要條件

您需要有 [Node.js 6](https://nodejs.org/en/download/)，才能執行此程式碼。

在本快速入門中，您可以使用[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)的訂用帳戶金鑰或付費的訂用帳戶金鑰。

## <a name="running-the-application"></a>執行應用程式

若要執行此應用程式，請遵循下列步驟：

1. 建立專案的資料夾 (或使用您最愛的 IDE 或編輯器)。
2. 從命令提示字元或終端機，巡覽至您剛剛建立的資料夾。
3. 安裝 request 模組：  
  ```  
  npm install request  
  ```  
3. 安裝 form-data 模組：  
  ```  
  npm install form-data  
  ```  
4. 建立名為 GetVisualInsights.js 的檔案，並在其中新增下列程式碼。
5. 以您的訂用帳戶金鑰取代 `subscriptionKey` 值。
7. 執行程式。  
  ```
  node GetVisualInsights.js
  ```

```javascript
var request = require('request');
var FormData = require('form-data');

var baseUri = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch';
var subscriptionKey = '<yoursubscriptionkeygoeshere>';

// To get an insights, call the /images/search endpoint. Get the token from
// the imageInsightsToken field in the Image object.
var insightsToken = "ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ";

var knowledgeRequest = {
    "imageInfo" : {
        "imageInsightsToken" : insightsToken
    }
};

var form = new FormData();
form.append('knowledgeRequest', JSON.stringify(knowledgeRequest));

form.getLength(function(err, length){
  if (err) {
    return requestCallback(err);
  }

  var r = request.post(baseUri, requestCallback);
  r._form = form; 
  r.setHeader('Ocp-Apim-Subscription-Key', subscriptionKey);
});

function requestCallback(err, res, body) {
    console.log(JSON.stringify(JSON.parse(body), null, '  '))
}
```


<a name="using-python" />

## <a name="using-python"></a>使用 Python


### <a name="prerequisites"></a>必要條件

您需要 [Python 3](https://www.python.org/) 以執行此程式碼。

在本快速入門中，您可以使用[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)的訂用帳戶金鑰或付費的訂用帳戶金鑰。

## <a name="running-the-walkthrough"></a>執行逐步解說

若要執行此應用程式，請遵循下列步驟：

1. 在您最愛的 IDE 或編輯器中，建立新的 Python 專案。
2. 建立名為 visualsearch.py 的檔案，並新增本快速入門中所示的程式碼。
3. 以您的訂用帳戶金鑰取代 `SUBSCRIPTION_KEY` 值。
4. 執行程式。


```python
"""Bing Visual Search example"""

# Download and install Python at https://www.python.org/
# Run the following in a command console window
# pip3 install requests

import requests, json

BASE_URI = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch'

SUBSCRIPTION_KEY = '<yoursubscriptionkeygoeshere>'

HEADERS = {'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY}

# To get an insights, call the /images/search endpoint. Get the token from
# the imageInsightsToken field in the Image object.
insightsToken = 'ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ'

formData = '{"imageInfo":{"imageInsightsToken":"' + insightsToken + '"}}'


file = {'knowledgeRequest' : (None, formData)}

def main():
    
    try:
        response = requests.post(BASE_URI, headers=HEADERS, files=file)
        response.raise_for_status()
        print_json(response.json())

    except Exception as ex:
        raise ex


def print_json(obj):
    """Print the object as json"""
    print(json.dumps(obj, sort_keys=True, indent=2, separators=(',', ': ')))



# Main execution
if __name__ == '__main__':
    main()
```

## <a name="next-steps"></a>後續步驟

[Bing 圖像式搜尋單頁應用程式教學課程](tutorial-bing-visual-search-single-page-app.md)  
[Bing 圖像式搜尋概觀](overview.md)  
[試試看](https://aka.ms/bingvisualsearchtryforfree)  
[取得免費試用的存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-visual-search-api)  
[Bing 圖像式搜尋 API 參考](https://aka.ms/bingvisualsearchreferencedoc)
