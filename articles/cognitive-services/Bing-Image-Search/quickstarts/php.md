---
title: 快速入門：使用 PHP 來執行影像搜尋 - Bing 影像搜尋 API
titleSuffix: Azure Cognitive Services
description: 使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 此文章中的簡單應用程式會傳送搜尋查詢，並顯示原始結果。
services: cognitive-services
documentationcenter: ''
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 9/07/2018
ms.author: aahi
ms.openlocfilehash: ac3de25773249119535ac6f3bd063ff6b9b7831e
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295512"
---
# <a name="quickstart-send-search-queries-using-the-bing-image-search-rest-api-and-php"></a>快速入門：使用 Bing 影像搜尋 REST API 和 PHP 來傳送搜尋查詢

使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 此文章中的簡單應用程式會傳送搜尋查詢，並顯示原始結果。

雖然此應用程式是以 PHP 撰寫，但 API 是一種與任何程式語言相容的 RESTful Web 服務，可產生 HTTP 要求，並剖析 JSON。

[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/java/Search/BingImageSearchv7.java) 上有此範例的原始程式碼。

## <a name="prerequisites"></a>先決條件

* [PHP 5.6.x 或更新版本](http://php.net/downloads.php)。

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>建立應用程式並將其初始化

若要執行此應用程式，請遵循下列步驟。

1. 確定 `php.ini` 檔案中已啟用安全 HTTP 支援。 在 Windows 上，此檔案位於 `C:\windows`。
2. 在您最愛的 IDE 或編輯器中建立新的 PHP 專案。
3. 定義 API 端點、您的訂用帳戶金鑰及搜尋字詞。

    ```php
    $endpoint = 'https://api.cognitive.microsoft.com/bing/v7.0/images/search';
    // Replace the accessKey string value with your valid access key.
    $accessKey = 'enter key here';
    $term = 'tropical ocean';
    ```
## <a name="construct-and-perform-a-http-request"></a>建構及執行 HTTP 要求

1. 使用上一個步驟中的變數來準備對「影像搜尋 API」的 HTTP 要求。

    ```php
    $headers = "Ocp-Apim-Subscription-Key: $key\r\n";
    $options = array ( 'http' => array (
                            'header' => $headers,
                            'method' => 'GET' ));
    ```
2. 執行 Web 要求並取得 JSON 回應。

    ```php
    $context = stream_context_create($options);
    $result = file_get_contents($url . "?q=" . urlencode($query), false, $context);
    ```

## <a name="process-and-print-the-json"></a>處理及列印 JSON

處理及列印所傳回的 JSON 回應。

    ```php
    $headers = array();
        foreach ($http_response_header as $k => $v) {
            $h = explode(":", $v, 2);
            if (isset($h[1]))
                if (preg_match("/^BingAPIs-/", $h[0]) || preg_match("/^X-MSEdge-/", $h[0]))
                    $headers[trim($h[0])] = trim($h[1]);
        }
        return array($headers, $result);
    ```

## <a name="sample-json-response"></a>範例 JSON 回應

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
