---
title: 快速入門：取得句子長度 - 翻譯工具文字、Java
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您可以使用認知服務的翻譯工具文字 API 搭配 Java，了解文字中句子的長度。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-text
ms.topic: quickstart
ms.date: 06/21/2018
ms.author: nolachar
ms.openlocfilehash: d0459b04bbd0dbbda3dab80623a39742a6f843b2
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46122615"
---
# <a name="quickstart-get-sentence-lengths-with-java"></a>快速入門：使用 Java 取得句子長度

在本快速入門中，您可以使用翻譯工具文字 API，了解文字中句子的長度。

## <a name="prerequisites"></a>必要條件

您將需要有 [JDK 7 或 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) (英文)，才能編譯和執行此程式碼。 若您有特別喜愛的 Java IDE，也可以拿來使用，但亦可使用文字編輯器。

若要使用翻譯工具文字 API，您也需要有訂用帳戶金鑰；請參閱[如何註冊翻譯工具文字 API](translator-text-how-to-signup.md)。

## <a name="breaksentence-request"></a>BreakSentence 要求

下列程式碼會使用 [BreakSentence](./reference/v3-0-break-sentence.md) 方法，將來源文字分割成句子。

1. 在您慣用的程式碼編輯器中，建立新的 Java 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 執行程式。

```java
import java.io.*;
import java.net.*;
import java.util.*;
import javax.net.ssl.HttpsURLConnection;

/*
 * Gson: https://github.com/google/gson
 * Maven info:
 *     groupId: com.google.code.gson
 *     artifactId: gson
 *     version: 2.8.1
 */
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

/* NOTE: To compile and run this code:
1. Save this file as BreakSentences.java.
2. Run:
    javac BreakSentences.java -cp .;gson-2.8.1.jar -encoding UTF-8
3. Run:
    java -cp .;gson-2.8.1.jar BreakSentences
*/

public class BreakSentences {

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace the subscriptionKey string value with your valid subscription key.
    static String subscriptionKey = "ENTER KEY HERE";

    static String host = "https://api.cognitive.microsofttranslator.com";
    static String path = "/breaksentence?api-version=3.0";

    static String text = "How are you? I am fine. What did you do today?";

    public static class RequestBody {
        String Text;

        public RequestBody(String text) {
            this.Text = text;
        }
    }

    public static String Post (URL url, String content) throws Exception {
        HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
        connection.setRequestMethod("POST");
        connection.setRequestProperty("Content-Type", "application/json");
        connection.setRequestProperty("Content-Length", content.length() + "");
        connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
        connection.setRequestProperty("X-ClientTraceId", java.util.UUID.randomUUID().toString());
        connection.setDoOutput(true);

        DataOutputStream wr = new DataOutputStream(connection.getOutputStream());
        byte[] encoded_content = content.getBytes("UTF-8");
        wr.write(encoded_content, 0, encoded_content.length);
        wr.flush();
        wr.close();

        StringBuilder response = new StringBuilder ();
        BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream(), "UTF-8"));
        String line;
        while ((line = in.readLine()) != null) {
            response.append(line);
        }
        in.close();

        return response.toString();
    }

    public static String BreakSentences () throws Exception {
        URL url = new URL (host + path);

        List<RequestBody> objList = new ArrayList<RequestBody>();
        objList.add(new RequestBody(text));
        String content = new Gson().toJson(objList);

        return Post(url, content);
    }

    public static String prettify(String json_text) {
        JsonParser parser = new JsonParser();
        JsonElement json = parser.parse(json_text);
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }

    public static void main(String[] args) {
        try {
            String response = BreakSentences ();
            System.out.println (prettify (response));
        }
        catch (Exception e) {
            System.out.println (e);
        }
    }
}
```

## <a name="breaksentence-response"></a>BreakSentence 回應

成功的回應會以 JSON 格式傳回，如下列範例所示：

```json
[
  {
    "detectedLanguage": {
      "language": "en",
      "score": 1.0
    },
    "sentLen": [
      13,
      11,
      22
    ]
  }
]
```

## <a name="next-steps"></a>後續步驟

瀏覽本快速入門及其他文件的範例程式碼，包括翻譯和音譯，以及 GitHub 上的其他「翻譯工具文字」專案範例。

> [!div class="nextstepaction"]
> [瀏覽 GitHub 上的 Java 範例](https://aka.ms/TranslatorGitHub?type=&language=java) (英文)
