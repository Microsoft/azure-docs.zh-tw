---
title: 快速入門：偵測影像中的人臉 - 臉部 API、Java
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用臉部 API 搭配 Java 來偵測影像中的人臉。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: quickstart
ms.date: 05/10/2018
ms.author: pafarley
ms.openlocfilehash: 42a8b13b87623b92ee0fe92db4e6a0caff5c3a1b
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339110"
---
# <a name="quickstart-detect-faces-in-an-image-using-java"></a>快速入門：使用 Java 偵測影像中的人臉

在本快速入門中，您會使用臉部 API 來偵測影像中的人臉。

## <a name="prerequisites"></a>必要條件

您需要有訂用帳戶金鑰才能執行範例。 您可以從[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=face-api)取得免費的試用訂用帳戶金鑰。

## <a name="detect-faces-in-an-image"></a>偵測影像中的人臉

使用 [Face - Detect](https://westcentralus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) 方法，偵測影像中的人臉並傳回臉部屬性，包括：

* Face ID：數個臉部 API 案例中使用的唯一識別碼。
* 臉部矩形：左側、頂端、寬度和高度，表示影像中臉部的位置。
* 特徵點：27 點臉部特徵點陣列，指向臉部元件的重要位置。
* 臉部屬性，包括年齡、性別、笑容程度、頭部姿勢以及臉部毛髮。

若要執行範例，請執行下列步驟：

1. 在喜愛的 Java IDE 中，建立新的命令列應用程式。
2. 以下列程式碼取代主要類別 (保留任何 `package` 陳述式)。
3. 將 `<Subscription Key>` 換成您的有效訂用帳戶金鑰。
4. 必要時，請將 `uriBase` 值變更為使用您取得訂用帳戶金鑰的位置。
5. 將 Maven 存放庫中的這些通用程式庫下載到您專案中的 `lib` 目錄：
   * `org.apache.httpcomponents:httpclient:4.2.4`
   * `org.json:json:20170516`
6. 執行 'Main'。

### <a name="face---detect-request"></a>Face - Detect 要求

```java
// This sample uses Apache HttpComponents:
// http://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/
// https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/

import java.net.URI;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;

public class Main
{
    // Replace <Subscription Key> with your valid subscription key.
    private static final String subscriptionKey = "<Subscription Key>";

    // NOTE: You must use the same region in your REST call as you used to
    // obtain your subscription keys. For example, if you obtained your
    // subscription keys from westus, replace "westcentralus" in the URL
    // below with "westus".
    //
    // Free trial subscription keys are generated in the westcentralus region. If you
    // use a free trial subscription key, you shouldn't need to change this region.
    private static final String uriBase =
        "https://westcentralus.api.cognitive.microsoft.com/face/v1.0/detect";

    private static final String imageWithFaces =
        "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}";

    private static final String faceAttributes =
        "age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise";

    public static void main(String[] args)
    {
        HttpClient httpclient = new DefaultHttpClient();

        try
        {
            URIBuilder builder = new URIBuilder(uriBase);

            // Request parameters. All of them are optional.
            builder.setParameter("returnFaceId", "true");
            builder.setParameter("returnFaceLandmarks", "false");
            builder.setParameter("returnFaceAttributes", faceAttributes);

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

            // Request body.
            StringEntity reqEntity = new StringEntity(imageWithFaces);
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            if (entity != null)
            {
                // Format and display the JSON response.
                System.out.println("REST Response:\n");

                String jsonString = EntityUtils.toString(entity).trim();
                if (jsonString.charAt(0) == '[') {
                    JSONArray jsonArray = new JSONArray(jsonString);
                    System.out.println(jsonArray.toString(2));
                }
                else if (jsonString.charAt(0) == '{') {
                    JSONObject jsonObject = new JSONObject(jsonString);
                    System.out.println(jsonObject.toString(2));
                } else {
                    System.out.println(jsonString);
                }
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
        }
    }
}
```

### <a name="face---detect-response"></a>Face - Detect 回應

成功的回應會以 JSON 的形式傳回。

```json
[{
  "faceRectangle": {
    "top": 131,
    "left": 177,
    "width": 162,
    "height": 162
  },
  "faceAttributes": {
    "makeup": {
      "eyeMakeup": true,
      "lipMakeup": true
    },
    "facialHair": {
      "sideburns": 0,
      "beard": 0,
      "moustache": 0
    },
    "gender": "female",
    "accessories": [],
    "blur": {
      "blurLevel": "low",
      "value": 0.06
    },
    "headPose": {
      "roll": 0.1,
      "pitch": 0,
      "yaw": -32.9
    },
    "smile": 0,
    "glasses": "NoGlasses",
    "hair": {
      "bald": 0,
      "invisible": false,
      "hairColor": [
        {
          "color": "brown",
          "confidence": 1
        },
        {
          "color": "black",
          "confidence": 0.87
        },
        {
          "color": "other",
          "confidence": 0.51
        },
        {
          "color": "blond",
          "confidence": 0.08
        },
        {
          "color": "red",
          "confidence": 0.08
        },
        {
          "color": "gray",
          "confidence": 0.02
        }
      ]
    },
    "emotion": {
      "contempt": 0,
      "surprise": 0.005,
      "happiness": 0,
      "neutral": 0.986,
      "sadness": 0.009,
      "disgust": 0,
      "anger": 0,
      "fear": 0
    },
    "exposure": {
      "value": 0.67,
      "exposureLevel": "goodExposure"
    },
    "occlusion": {
      "eyeOccluded": false,
      "mouthOccluded": false,
      "foreheadOccluded": false
    },
    "noise": {
      "noiseLevel": "low",
      "value": 0
    },
    "age": 22.9
  },
  "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f"
}]
```

## <a name="next-steps"></a>後續步驟

了解如何建立 Android 應用程式，使用臉部服務來偵測影像中的人臉。 應用程式會顯示在每張臉的周圍繪出邊框的影像。

> [!div class="nextstepaction"]
> [教學課程：開始在 Android 中使用臉部 API](../Tutorials/FaceAPIinJavaForAndroidTutorial.md)
