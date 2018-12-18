---
title: 快速入門：Bing 拼字檢查 API，Java
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用 Bing 拼字檢查 API。
services: cognitive-services
author: v-jaswel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-spell-check
ms.topic: quickstart
ms.date: 09/14/2017
ms.author: v-jaswel
ms.openlocfilehash: 00e0b7db5bfc8b763d9b16524bd783601d1ec4d8
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48800990"
---
# <a name="quickstart-for-bing-spell-check-api-with-java"></a>搭配使用 Bing 拼字檢查 API 與 Java 的快速入門 

本文示範如何使用以 Java 撰寫的 [Bing 拼字檢查 API](https://azure.microsoft.com/services/cognitive-services/spell-check/)。 拼字檢查 API 會傳回未能辨識的字詞清單，附帶建議的替代項目。 一般而言，您會將文字提交給此 API，然後在文字中進行建議的替代，或者向應用程式使用者顯示建議的替代項目，讓他們決定是否要進行替代。 本文顯示如何傳送其中包含文字 "Hollo, wrld!" 的要求。 建議的替代項目是 "Hello" 和 "world"。

## <a name="prerequisites"></a>必要條件

您將需要有 [JDK 7 或 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，才能編譯和執行此程式碼。 若您有特別喜愛的 Java IDE 也可以使用，但是文字編輯器就夠了。

您必須有具備 **Bing 拼字檢查 API v7** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 [免費試用版](https://azure.microsoft.com/try/cognitive-services/#lang)即足以供本快速入門使用。 您必須要有啟用免費試用版時所提供的存取金鑰，或者您可以從 Azure 儀表板使用付費訂用帳戶金鑰。

## <a name="get-spell-check-results"></a>取得拼字檢查結果

1. 在您偏好的 IDE 中建立新的 JAVA 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 執行程式。

```java
import java.io.*;
import java.net.*;
import javax.net.ssl.HttpsURLConnection;

public class HelloWorld {

    static String host = "https://api.cognitive.microsoft.com";
    static String path = "/bing/v7.0/spellcheck";

    // NOTE: Replace this example key with a valid subscription key.
    static String key = "ENTER KEY HERE";

    static String mkt = "en-US";
    static String mode = "proof";
    static String text = "Hollo, wrld!";

    public static void check () throws Exception {
        String params = "?mkt=" + mkt + "&mode=" + mode;
        URL url = new URL(host + path + params);
        HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
        connection.setRequestMethod("POST");
        connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
        connection.setRequestProperty("Content-Length", "" + text.length() + 5);
        connection.setRequestProperty("Ocp-Apim-Subscription-Key", key);
        connection.setDoOutput(true);

        DataOutputStream wr = new DataOutputStream(connection.getOutputStream());
        wr.writeBytes("text=" + text);
        wr.flush();
        wr.close();

        BufferedReader in = new BufferedReader(
        new InputStreamReader(connection.getInputStream()));
        String line;
        while ((line = in.readLine()) != null) {
            System.out.println(line);
        }
        in.close();
    }

    public static void main(String[] args) {
        try {
            check ();
        }
        catch (Exception e) {
            System.out.println (e);
        }
    }
}
```

**回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
   "_type": "SpellCheck",
   "flaggedTokens": [
      {
         "offset": 0,
         "token": "Hollo",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "Hello",
               "score": 0.9115257530801
            },
            {
               "suggestion": "Hollow",
               "score": 0.858039839213461
            },
            {
               "suggestion": "Hallo",
               "score": 0.597385084464481
            }
         ]
      },
      {
         "offset": 7,
         "token": "wrld",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "world",
               "score": 0.9115257530801
            }
         ]
      }
   ]
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 拼字檢查教學課程](../tutorials/spellcheck.md)

## <a name="see-also"></a>另請參閱

- [Bing 拼字檢查概觀](../proof-text.md)
- [Bing 拼字檢查 API v7 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference)
