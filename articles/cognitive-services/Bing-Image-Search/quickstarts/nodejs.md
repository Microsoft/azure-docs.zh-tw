---
title: 快速入門：使用 Node.js 來執行影像搜尋 - Bing 影像搜尋 API
titleSuffix: Azure Cognitive Services
description: 使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 JavaScript 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。
services: cognitive-services
documentationcenter: ''
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 8/20/2018
ms.author: aahi
ms.openlocfilehash: 1584b3e0a1e1c560d42b5f8320d0e15ad6242918
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46294589"
---
# <a name="quickstart-send-search-queries-using-the-bing-image-search-rest-api-and-nodejs"></a>快速入門：使用 Bing 影像搜尋 REST API 和 Node.js 來傳送搜尋查詢

使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 JavaScript 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。

雖然此應用程式是以 JavaScript 撰寫並在 Node.js 中執行的，但 API 是一種與大多數程式設計語言都相容的 RESTful Web 服務。

[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingImageSearchv7Quickstart.js) 上有此範例的原始程式碼，其中還有其他錯誤處理和註釋。

## <a name="prerequisites"></a>先決條件

* 最新版的 [Node.js](https://nodejs.org/en/download/)。

* [JavaScript 要求程式庫](https://github.com/request/request)
[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>建立應用程式並將其初始化

1. 在您最愛的 IDE 或編輯器中建立新的 JavaScript 檔案，並設定嚴謹度 和 https 需求。

    ```javascript
    'use strict';
    let https = require('https');
    ```

2. 建立適用於 API 端點、影像 API 搜尋路徑、您訂用帳戶金鑰及搜尋字詞的變數。
    ```javascript
    let subscriptionKey = 'enter key here';
    let host = 'api.cognitive.microsoft.com';
    let path = '/bing/v7.0/images/search';
    let term = 'tropical ocean';
    ```

## <a name="construct-the-search-request-and-query"></a>建構搜尋要求和查詢。

1. 使用上一個步驟中的變數來製作適用於 API 要求的搜尋 URL 格式。 請注意，您的搜尋字詞必須先進行 URL 編碼，才能傳送給 API。

    ```javascript
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path + '?q=' + encodeURIComponent(search),
        headers : {
        'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };
    ```

2. 使用要求程式庫將查詢傳送給 API。 下一節中將會定義 `response_handler`。
    ```javascript
    let req = https.request(request_params, response_handler);
    req.end();
    ```

## <a name="handle-and-parse-the-response"></a>處理及剖析回應

1. 定義一個名為 `response_handler` 的函式，此函式會接受 HTTP 呼叫 `response` 作為參數。 在此函式內，執行下列步驟：

    1. 定義一個變數來包含 JSON 回應本文。  
        ```javascript
        let response_handler = function (response) {
            let body = '';
        };
        ```

    2. 當呼叫 **data** 旗標時，儲存回應本文
        ```javascript
        response.on('data', function (d) {
            body += d;
        });
        ```

    3. 當發出 **end** 旗標信號時，即可處理 JSON，並可列印影像的 URL，以及所傳回影像的總數。

        ```javascript
        response.on('end', function () {
            let firstImageResult = imageResults.value[0];
            console.log(`Image result count: ${imageResults.value.length}`);
            console.log(`First image thumbnail url: ${firstImageResult.thumbnailUrl}`);
            console.log(`First image web search url: ${firstImageResult.webSearchUrl}`);
         });
        ```

## <a name="json-response"></a>JSON 回應

來自「Bing 影像搜尋 API」的回應會以 JSON 形式傳回。 此範例回應已截斷而只顯示單一結果。

```json
{
"_type":"Images",
"instrumentation":{
    "_type":"ResponseInstrumentation"
},
"readLink":"images\/search?q=tropical ocean",
"webSearchUrl":"https:\/\/www.bing.com\/images\/search?q=tropical ocean&FORM=OIIARP",
"totalEstimatedMatches":842,
"nextOffset":47,
"value":[
    {
        "webSearchUrl":"https:\/\/www.bing.com\/images\/search?view=detailv2&FORM=OIIRPO&q=tropical+ocean&id=8607ACDACB243BDEA7E1EF78127DA931E680E3A5&simid=608027248313960152",
        "name":"My Life in the Ocean | The greatest WordPress.com site in ...",
        "thumbnailUrl":"https:\/\/tse3.mm.bing.net\/th?id=OIP.fmwSKKmKpmZtJiBDps1kLAHaEo&pid=Api",
        "datePublished":"2017-11-03T08:51:00.0000000Z",
        "contentUrl":"https:\/\/mylifeintheocean.files.wordpress.com\/2012\/11\/tropical-ocean-wallpaper-1920x12003.jpg",
        "hostPageUrl":"https:\/\/mylifeintheocean.wordpress.com\/",
        "contentSize":"897388 B",
        "encodingFormat":"jpeg",
        "hostPageDisplayUrl":"https:\/\/mylifeintheocean.wordpress.com",
        "width":1920,
        "height":1200,
        "thumbnail":{
        "width":474,
        "height":296
        },
        "imageInsightsToken":"ccid_fmwSKKmK*mid_8607ACDACB243BDEA7E1EF78127DA931E680E3A5*simid_608027248313960152*thid_OIP.fmwSKKmKpmZtJiBDps1kLAHaEo",
        "insightsMetadata":{
        "recipeSourcesCount":0,
        "bestRepresentativeQuery":{
            "text":"Tropical Beaches Desktop Wallpaper",
            "displayText":"Tropical Beaches Desktop Wallpaper",
            "webSearchUrl":"https:\/\/www.bing.com\/images\/search?q=Tropical+Beaches+Desktop+Wallpaper&id=8607ACDACB243BDEA7E1EF78127DA931E680E3A5&FORM=IDBQDM"
        },
        "pagesIncludingCount":115,
        "availableSizesCount":44
        },
        "imageId":"8607ACDACB243BDEA7E1EF78127DA931E680E3A5",
        "accentColor":"0050B2"
    }
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 影像搜尋單頁應用程式教學課程](../tutorial-bing-image-search-single-page-app.md) (英文)

## <a name="see-also"></a>另請參閱

* [什麼是 Bing 影像搜尋？](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)  
* [試用線上互動式示範](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [取得免費認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-image-search-api)  
* [Azure 認知服務文件](https://docs.microsoft.com/azure/cognitive-services)
* [Bing 影像搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
