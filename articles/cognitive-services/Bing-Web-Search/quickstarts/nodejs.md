---
title: 快速入門：使用 Node.js 執行搜尋 - Bing Web 搜尋 API
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將學習如何使用 Node.js 來第一次呼叫 Bing Web 搜尋 API，並接收 JSON 回應。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: quickstart
ms.date: 9/26/2018
ms.author: erhopf
ms.openlocfilehash: debaa63adeb97063d0ea42e1da36352dc2c9c4e7
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405851"
---
# <a name="quickstart-use-nodejs-to-call-the-bing-web-search-api"></a>快速入門：使用 Node.js 來呼叫 Bing Web 搜尋 API  

本快速入門可讓您在 10 分鐘內完成第一次呼叫 Bing Web 搜尋 API，並接收 JSON 回應。

[!INCLUDE [bing-web-search-quickstart-signup](../../../../includes/bing-web-search-quickstart-signup.md)]

## <a name="prerequisites"></a>必要條件

以下是執行本快速入門之前的幾個必備項目：

* [Node.js 6](https://nodejs.org/en/download/) (英文) 或更新版本
* 訂用帳戶金鑰

## <a name="create-a-project-and-declare-required-modules"></a>建立專案，並宣告所需的模組

在您最愛的 IDE 或編輯器中建立新的 Node.js 專案。
然後，將下列程式碼片段複製到您專案內名為 `search.js` 的檔案中。

```javascript
// Use this simple app to query the Bing Web Search API and get a JSON response.
// Usage: node search.js "your query".
const https = require('https')
```

## <a name="set-the-subscription-key"></a>設定訂用帳戶金鑰

此程式碼片段會使用 `AZURE_SUBSCRIPTION_KEY` 環境變數來儲存您的訂用帳戶金鑰，此作法可有效避免在部署程式碼時意外洩露您的金鑰。 [按一下這裡](https://azure.microsoft.com/try/cognitive-services/my-apis/?apiSlug=search-api-v7)可查閱您的訂用帳戶金鑰。

如果您不熟悉環境變數的使用方式，或想要盡快執行此應用程式，您可以將 `process.env['AZURE_SUBSCRIPTION_KEY']` 取代為您設定為字串的訂用帳戶金鑰。

```javascript
const SUBSCRIPTION_KEY = process.env['AZURE_SUBSCRIPTION_KEY']
if (!SUBSCRIPTION_KEY) {
  throw new Error('AZURE_SUBSCRIPTION_KEY is not set.')
}
```

## <a name="create-a-function-to-make-the-request"></a>建立用來提出要求的函式

此函式將提出安全的 GET 要求，將搜尋查詢儲存為路徑中的查詢參數。 `encodeURIComponent` 可用來逸出無效字元，而訂用帳戶金鑰會傳入標頭中。 回呼所接收的[回應](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse)會訂閱用以彙總 JSON 主體的 `data` 事件、會記錄任何問題的 `error` 事件，和可據以得知何時應將訊息視為完整的 `end` 事件。 完成後，應用程式會列印有關的標頭和訊息本文。 您可以根據自己的喜好設定來調整色彩和設定深度，深度為 `1` 時可適當摘要回應。

```javascript
function bingWebSearch(query) {
  https.get({
    hostname: 'api.cognitive.microsoft.com',
    path:     '/bing/v7.0/search?q=' + encodeURIComponent(query),
    headers:  { 'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY },
  }, res => {
    let body = ''
    res.on('data', part => body += part)
    res.on('end', () => {
      for (var header in res.headers) {
        if (header.startsWith("bingapis-") || header.startsWith("x-msedge-")) {
          console.log(header + ": " + res.headers[header])
        }
      }
      console.log('\nJSON Response:\n')
      console.dir(JSON.parse(body), { colors: false, depth: null })
    })
    res.on('error', e => {
      console.log('Error: ' + e.message)
      throw e
    })
  })
}
```

## <a name="get-the-query"></a>取得查詢

我們將查看程式的引數以尋找查詢。 第一個引數是節點的路徑，第二個是我們的檔案名稱，第三個是您的查詢。 如果查詢不存在，則會使用預設查詢「Microsoft 認知服務」。

```javascript
const query = process.argv[2] || 'Microsoft Cognitive Services'
```

## <a name="make-a-request-and-print-the-response"></a>提出要求並列印回應

現在，所有定義皆已完成，接著我們將呼叫函式！

```javascript
bingWebSearch(query)
```

## <a name="put-it-all-together"></a>組合在一起

最後一步是執行您的程式碼：`node search.js "<your query>"`。

如果想要將您的程式碼與我們的程式碼做比較，以下是完整的程式：

```javascript
const https = require('https')
const SUBSCRIPTION_KEY = process.env['AZURE_SUBSCRIPTION_KEY']
if (!SUBSCRIPTION_KEY) {
  throw new Error('Missing the AZURE_SUBSCRIPTION_KEY environment varable')
}
function bingWebSearch(query) {
  https.get({
    hostname: 'api.cognitive.microsoft.com',
    path:     '/bing/v7.0/search?q=' + encodeURIComponent(query),
    headers:  { 'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY },
  }, res => {
    let body = ''
    res.on('data', part => body += part)
    res.on('end', () => {
      for (var header in res.headers) {
        if (header.startsWith("bingapis-") || header.startsWith("x-msedge-")) {
          console.log(header + ": " + res.headers[header])
        }
      }
      console.log('\nJSON Response:\n')
      console.dir(JSON.parse(body), { colors: false, depth: null })
    })
    res.on('error', e => {
      console.log('Error: ' + e.message)
      throw e
    })
  })
}
const query = process.argv[2] || 'Microsoft Cognitive Services'
bingWebSearch(query)
```

## <a name="sample-response"></a>範例回應

來自 Bing Web 搜尋 API 的回應會以 JSON 格式傳回。 本範例回應已截斷而只顯示單一結果。

```json
{
  "_type": "SearchResponse",
  "queryContext": {
    "originalQuery": "Microsoft Cognitive Services"
  },
  "webPages": {
    "webSearchUrl": "https://www.bing.com/search?q=Microsoft+cognitive+services",
    "totalEstimatedMatches": 22300000,
    "value": [
      {
        "id": "https://api.cognitive.microsoft.com/api/v7/#WebPages.0",
        "name": "Microsoft Cognitive Services",
        "url": "https://www.microsoft.com/cognitive-services",
        "displayUrl": "https://www.microsoft.com/cognitive-services",
        "snippet": "Knock down barriers between you and your ideas. Enable natural and contextual interaction with tools that augment users' experiences via the power of machine-based AI. Plug them in and bring your ideas to life.",
        "deepLinks": [
          {
            "name": "Face API",
            "url": "https://azure.microsoft.com/services/cognitive-services/face/",
            "snippet": "Add facial recognition to your applications to detect, identify, and verify faces using a Face API from Microsoft Azure. ... Cognitive Services; Face API;"
          },
          {
            "name": "Text Analytics",
            "url": "https://azure.microsoft.com/services/cognitive-services/text-analytics/",
            "snippet": "Cognitive Services; Text Analytics API; Text Analytics API . Detect sentiment, ... you agree that Microsoft may store it and use it to improve Microsoft services, ..."
          },
          {
            "name": "Computer Vision API",
            "url": "https://azure.microsoft.com/services/cognitive-services/computer-vision/",
            "snippet": "Extract the data you need from images using optical character recognition and image analytics with Computer Vision APIs from Microsoft Azure."
          },
          {
            "name": "Emotion",
            "url": "https://www.microsoft.com/cognitive-services/emotion-api",
            "snippet": "Cognitive Services Emotion API - microsoft.com"
          },
          {
            "name": "Bing Speech API",
            "url": "https://azure.microsoft.com/services/cognitive-services/speech/",
            "snippet": "Add speech recognition to your applications, including text to speech, with a speech API from Microsoft Azure. ... Cognitive Services; Bing Speech API;"
          },
          {
            "name": "Get Started for Free",
            "url": "https://azure.microsoft.com/services/cognitive-services/",
            "snippet": "Add vision, speech, language, and knowledge capabilities to your applications using intelligence APIs and SDKs from Cognitive Services."
          }
        ]
      }
    ]
  },
  "relatedSearches": {
    "id": "https://api.cognitive.microsoft.com/api/v7/#RelatedSearches",
    "value": [
      {
        "text": "microsoft bot framework",
        "displayText": "microsoft bot framework",
        "webSearchUrl": "https://www.bing.com/search?q=microsoft+bot+framework"
      },
      {
        "text": "microsoft cognitive services youtube",
        "displayText": "microsoft cognitive services youtube",
        "webSearchUrl": "https://www.bing.com/search?q=microsoft+cognitive+services+youtube"
      },
      {
        "text": "microsoft cognitive services search api",
        "displayText": "microsoft cognitive services search api",
        "webSearchUrl": "https://www.bing.com/search?q=microsoft+cognitive+services+search+api"
      },
      {
        "text": "microsoft cognitive services news",
        "displayText": "microsoft cognitive services news",
        "webSearchUrl": "https://www.bing.com/search?q=microsoft+cognitive+services+news"
      },
      {
        "text": "ms cognitive service",
        "displayText": "ms cognitive service",
        "webSearchUrl": "https://www.bing.com/search?q=ms+cognitive+service"
      },
      {
        "text": "microsoft cognitive services text analytics",
        "displayText": "microsoft cognitive services text analytics",
        "webSearchUrl": "https://www.bing.com/search?q=microsoft+cognitive+services+text+analytics"
      },
      {
        "text": "microsoft cognitive services toolkit",
        "displayText": "microsoft cognitive services toolkit",
        "webSearchUrl": "https://www.bing.com/search?q=microsoft+cognitive+services+toolkit"
      },
      {
        "text": "microsoft cognitive services api",
        "displayText": "microsoft cognitive services api",
        "webSearchUrl": "https://www.bing.com/search?q=microsoft+cognitive+services+api"
      }
    ]
  },
  "rankingResponse": {
    "mainline": {
      "items": [
        {
          "answerType": "WebPages",
          "resultIndex": 0,
          "value": {
            "id": "https://api.cognitive.microsoft.com/api/v7/#WebPages.0"
          }
        }
      ]
    },
    "sidebar": {
      "items": [
        {
          "answerType": "RelatedSearches",
          "value": {
            "id": "https://api.cognitive.microsoft.com/api/v7/#RelatedSearches"
          }
        }
      ]
    }
  }
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing Web 搜尋單頁應用程式教學課程](../tutorial-bing-web-search-single-page-app.md)

[!INCLUDE [bing-web-search-quickstart-see-also](../../../../includes/bing-web-search-quickstart-see-also.md)]
