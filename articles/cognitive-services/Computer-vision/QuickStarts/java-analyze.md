---
title: 快速入門：分析遠端影像 - REST、Java - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用電腦視覺 API 搭配 Java 分析影像。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: 6dae10262c40a127cb24dd40f0d5389c0cdd7c20
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340013"
---
# <a name="quickstart-analyze-a-remote-image-using-the-rest-api-and-java-in-computer-vision"></a>快速入門：在電腦視覺中使用 REST API 與 Java 分析遠端影像

在此快速入門中，您將使用電腦視覺的 REST API，來分析遠端儲存的影像以擷取視覺功能。 您可以使用[分析影像](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa)方法，根據影像內容來擷取視覺功能。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services) 。

## <a name="prerequisites"></a>必要條件

- 您必須已安裝 [Java&trade; Platform, Standard Edition Development Kit 7 或 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) (JDK 7 或 8)。
- 您必須有電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-sample-application"></a>建立並執行範例應用程式

若要建立並執行範例，請執行下列步驟：

1. 在您最愛的 IDE 或編輯器中建立新的 Java 專案。 如果此選項可用，請從命令列應用程式範本建立 Java 專案。
1. 將下列程式庫匯入您的 Java 專案中。 如果您使用 Maven，則會提供每個程式庫的 Maven 座標。
   - [Apache HTTP 用戶端](https://hc.apache.org/downloads.cgi) (org.apache.httpcomponents:httpclient:4.5.5)
   - [Apache HTTP 核心](https://hc.apache.org/downloads.cgi) (org.apache.httpcomponents:httpcore:4.4.9)
   - [JSON 程式庫](https://github.com/stleary/JSON-java) (org.json:json:20180130)
1. 將下列 `import` 陳述式新增到包含適用於您應用程式之公用類別的 `Main` 檔案。  

   ```java
   import java.net.URI;
   import org.apache.http.HttpEntity;
   import org.apache.http.HttpResponse;
   import org.apache.http.client.methods.HttpPost;
   import org.apache.http.entity.StringEntity;
   import org.apache.http.client.utils.URIBuilder;
   import org.apache.http.impl.client.CloseableHttpClient;
   import org.apache.http.impl.client.HttpClientBuilder;
   import org.apache.http.util.EntityUtils;
   import org.json.JSONObject;
   ```

1. 將 `Main` 公用類別取代為下列程式碼，然後視需要在程式碼中進行下列變更：
   1. 將 `subscriptionKey` 的值取代為您的訂用帳戶金鑰。
   1. 如有需要，請從您取得訂用帳戶金鑰的 Azure 區域，將 `uriBase` 的值取代為[分析影像](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa)方法的端點 URL。
   1. (選擇性) 將 `imageToAnalyze` 的值取代為您要分析之不同影像的 URL。
1. 儲存並建置 Java 專案。
1. 如果您使用 IDE，請執行 `Main`。 否則，請開啟命令提示字元視窗，然後使用 `java` 命令執行已編譯的類別。 例如： `java Main`。

```java
public class Main {
    // **********************************************
    // *** Update or verify the following values. ***
    // **********************************************

    // Replace <Subscription Key> with your valid subscription key.
    private static final String subscriptionKey = "<Subscription Key>";

    // You must use the same Azure region in your REST API method as you used to
    // get your subscription keys. For example, if you got your subscription keys
    // from the West US region, replace "westcentralus" in the URL
    // below with "westus".
    //
    // Free trial subscription keys are generated in the West Central US region.
    // If you use a free trial subscription key, you shouldn't need to change
    // this region.
    private static final String uriBase =
            "https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/analyze";

    private static final String imageToAnalyze =
            "https://upload.wikimedia.org/wikipedia/commons/" +
                    "1/12/Broadway_and_Times_Square_by_night.jpg";

    public static void main(String[] args) {
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();

        try {
            URIBuilder builder = new URIBuilder(uriBase);

            // Request parameters. All of them are optional.
            builder.setParameter("visualFeatures", "Categories,Description,Color");
            builder.setParameter("language", "en");

            // Prepare the URI for the REST API method.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

            // Request body.
            StringEntity requestEntity =
                    new StringEntity("{\"url\":\"" + imageToAnalyze + "\"}");
            request.setEntity(requestEntity);

            // Call the REST API method and get the response entity.
            HttpResponse response = httpClient.execute(request);
            HttpEntity entity = response.getEntity();

            if (entity != null) {
                // Format and display the JSON response.
                String jsonString = EntityUtils.toString(entity);
                JSONObject json = new JSONObject(jsonString);
                System.out.println("REST Response:\n");
                System.out.println(json.toString(2));
            }
        } catch (Exception e) {
            // Display error message.
            System.out.println(e.getMessage());
        }
    }
}
```

## <a name="examine-the-response"></a>檢查回應

成功的回應會以 JSON 的形式傳回。 範例應用程式會在主控台視窗中剖析並顯示成功的回應，如下列範例所示：

```json
REST Response:

{
  "metadata": {
    "width": 1826,
    "format": "Jpeg",
    "height": 2436
  },
  "color": {
    "dominantColorForeground": "Brown",
    "isBWImg": false,
    "accentColor": "B74314",
    "dominantColorBackground": "Brown",
    "dominantColors": ["Brown"]
  },
  "requestId": "bbffe1a1-4fa3-4a6b-a4d5-a4964c58a811",
  "description": {
    "captions": [{
      "confidence": 0.8241405091548035,
      "text": "a group of people on a city street filled with traffic at night"
    }],
    "tags": [
      "outdoor",
      "building",
      "street",
      "city",
      "busy",
      "people",
      "filled",
      "traffic",
      "many",
      "table",
      "car",
      "group",
      "walking",
      "bunch",
      "crowded",
      "large",
      "night",
      "light",
      "standing",
      "man",
      "tall",
      "umbrella",
      "riding",
      "sign",
      "crowd"
    ]
  },
  "categories": [{
    "score": 0.625,
    "name": "outdoor_street"
  }]
}
```

## <a name="clean-up-resources"></a>清除資源

不再需要 Java 專案時請將它刪除，包括已編譯的類別與匯入的程式庫。

## <a name="next-steps"></a>後續步驟

探索使用「電腦視覺」在影像中執行光學字元辨識 (OCR)、建立智慧型裁剪縮圖以及偵測、分類、標記和描述視覺特徵 (包括臉部) 的 Java Swing 應用程式。 若要快速地試驗電腦視覺 API，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console)。

> [!div class="nextstepaction"]
> [電腦視覺 API Java 教學課程](../Tutorials/java-tutorial.md)
