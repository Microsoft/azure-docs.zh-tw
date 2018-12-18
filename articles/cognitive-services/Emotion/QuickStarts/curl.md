---
title: 快速入門：辨識影像中的臉部表情 - 表情 API，cURL
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您搭配 cURL 快速開始使用表情 API。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: emotion-api
ms.topic: quickstart
ms.date: 05/23/2017
ms.author: anroth
ROBOTS: NOINDEX
ms.openlocfilehash: dfdaa89c9d29e419539f385f601dc7f264bf838e
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48237070"
---
# <a name="quickstart-build-an-app-to-recognize-emotions-on-faces-in-an-image"></a>快速入門：建置應用程式以辨識影像中的臉部表情。

> [!IMPORTANT]
> 表情 API 將於 2019 年 2 月 15 日淘汰。 表情辨識功能現已公開推出，是[臉部 API](https://docs.microsoft.com/azure/cognitive-services/face/) 的一部分。

本文提供資訊和程式碼範例，以協助您搭配 cURL 快速開始使用[表情 API 辨識方法](https://westus.dev.cognitive.microsoft.com/docs/services/5639d931ca73072154c1ce89/operations/563b31ea778daf121cc3a5fa)來辨識出影像中一或多個人員所表現出來的表情。

## <a name="prerequisite"></a>必要條件
* 從[這裡](https://azure.microsoft.com/try/cognitive-services/)取得免費的訂用帳戶金鑰

## <a name="recognize-emotions-curl-example-request"></a>辨識表情 cURL 範例要求

> [!NOTE]
> 您必須在 REST 呼叫中使用與用來取得訂用帳戶金鑰相同的位置。 例如，如果您已從 westcentralus 取得訂用帳戶金鑰，請將下列 URL 中的 "westus" 取代為 "westcentralus"。

```json
@ECHO OFF

curl -v -X POST "https://westus.api.cognitive.microsoft.com/emotion/v1.0/recognize"
-H "Content-Type: application/json"
-H "Ocp-Apim-Subscription-Key: {subscription key}"

--data-ascii "{body}"
```

## <a name="recognize-emotions-sample-response"></a>辨識表情範例回應
呼叫若成功，則會傳回一系列的臉部資料，以及其相關聯的表情分數，依臉部方框大小以遞減方式排序。 空的回應表示沒有偵測到任何臉部。 表情項目包含下列欄位：
* faceRectangle - 臉部於影像中的矩形位置。
* scores - 影像中每個臉部的表情分數。

```json
application/json
[
  {
    "faceRectangle": {
      "left": 68,
      "top": 97,
      "width": 64,
      "height": 97
    },
    "scores": {
      "anger": 0.00300731952,
      "contempt": 5.14648448E-08,
      "disgust": 9.180124E-06,
      "fear": 0.0001912825,
      "happiness": 0.9875571,
      "neutral": 0.0009861537,
      "sadness": 1.889955E-05,
      "surprise": 0.008229999
    }
  }
]
