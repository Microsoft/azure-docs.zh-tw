---
title: 快速入門：偵測影像中的人臉 - 臉部 API cURL
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用臉部 API 搭配 cURL 偵測影像中的人臉。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: quickstart
ms.date: 05/10/2018
ms.author: pafarley
ms.openlocfilehash: da18b7b3666863742f987b2ee0da297c0838d266
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343056"
---
# <a name="quickstart-detect-faces-in-an-image-using-curl"></a>快速入門：使用 cURL 偵測影像中的人臉

在此快速入門中，您會使用臉部 API 來偵測影像中的人臉。

## <a name="prerequisites"></a>必要條件

您需要有訂用帳戶金鑰才能執行範例。 您可以從[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=face-api)取得免費的試用訂用帳戶金鑰。

## <a name="detect-faces-in-an-image"></a>偵測影像中的人臉

使用 [Face - Detect](https://westcentralus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) 方法，偵測影像中的人臉並傳回臉部屬性，包括：

* Face ID：數個臉部 API 案例中使用的唯一識別碼。
* 臉部矩形：左側、頂端、寬度和高度，表示影像中臉部的位置。
* 特徵點：27 點臉部特徵點陣列，指向臉部元件的重要位置。
* 臉部屬性，包括年齡、性別、笑容程度、頭部姿勢以及臉部毛髮。

若要執行範例，請執行下列步驟：

1. 開啟命令提示字元。
2. 將 `<Subscription Key>` 換成您的有效訂用帳戶金鑰。
3. 必要時，請將 URL (`https://westcentralus.api.cognitive.microsoft.com/face/v1.0/detect`) 變更為使用您取得訂用帳戶金鑰的位置。
4. (選擇性) 變更要分析的影像 (`"{\"url\":...`)。
5. 將程式碼貼入命令視窗中。
6. 執行命令。

### <a name="face---detect-request"></a>Face - Detect 要求

> [!NOTE]
> 您必須在 REST 呼叫中使用與用來取得訂用帳戶金鑰相同的位置。 例如，如果您已從 westus 取得訂用帳戶金鑰，請將下列 URL 中的 "westcentralus" 取代為 "westus"。

```shell
curl -H "Ocp-Apim-Subscription-Key: <Subscription Key>" "https://westcentralus.api.cognitive.microsoft.com/face/v1.0/detect?returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise" -H "Content-Type: application/json" --data-ascii "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}"
```

### <a name="face---detect-response"></a>Face - Detect 回應

成功的回應會以 JSON 的形式傳回。

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    },
    "faceAttributes": {
      "smile": 0,
      "headPose": {
        "pitch": 0,
        "roll": 0.1,
        "yaw": -32.9
      },
      "gender": "female",
      "age": 22.9,
      "facialHair": {
        "moustache": 0,
        "beard": 0,
        "sideburns": 0
      },
      "glasses": "NoGlasses",
      "emotion": {
        "anger": 0,
        "contempt": 0,
        "disgust": 0,
        "fear": 0,
        "happiness": 0,
        "neutral": 0.986,
        "sadness": 0.009,
        "surprise": 0.005
      },
      "blur": {
        "blurLevel": "low",
        "value": 0.06
      },
      "exposure": {
        "exposureLevel": "goodExposure",
        "value": 0.67
      },
      "noise": {
        "noiseLevel": "low",
        "value": 0
      },
      "makeup": {
        "eyeMakeup": true,
        "lipMakeup": true
      },
      "accessories": [],
      "occlusion": {
        "foreheadOccluded": false,
        "eyeOccluded": false,
        "mouthOccluded": false
      },
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
      }
    }
  }
]
```

## <a name="next-steps"></a>後續步驟

探索臉部 API，此 API 可用來偵測影像中的人臉、使用矩形標定臉部，以及傳回例如年齡和性別等屬性。

> [!div class="nextstepaction"]
> [臉部 API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (英文)
