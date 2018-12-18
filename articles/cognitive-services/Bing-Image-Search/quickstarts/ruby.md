---
title: 快速入門：使用 Ruby 來執行影像搜尋 - Bing 影像搜尋 API
description: 使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 Ruby 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。
services: cognitive-services
documentationcenter: ''
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 8/20/2018
ms.author: aahi
ms.openlocfilehash: 4c2c91b42af46ba42bdda84d7b8b77987c7ea818
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297327"
---
# <a name="quickstart-send-search-queries-using-the-rest-api-and-ruby"></a>快速入門：使用 REST API 和 Ruby 傳送搜尋查詢

使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 Ruby 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。

雖然此應用程式是以 Ruby 撰寫的，但 API 是一種與大多數程式設計語言都相容的 RESTful Web 服務。

[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/ruby/Search/BingImageSearchv7.rb) 上有此範例的原始程式碼。
## <a name="prerequisites"></a>先決條件

* [最新版的 Ruby](https://www.ruby-lang.org/en/downloads/)。

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>建立應用程式並將其初始化

1. 將下列套件匯入至您的程式碼檔案中。

    ```ruby
    require 'net/https'
    require 'uri'
    require 'json'
    ```

2. 建立適用於 API 端點、影像 API 搜尋路徑、您訂用帳戶金鑰及搜尋字詞的變數。

    ```ruby
    uri  = "https://api.cognitive.microsoft.com"
    path = "/bing/v7.0/images/search"
    term = "puppies"
    ```

## <a name="format-and-make-an-api-request"></a>製作 API 要求格式並提出 API 要求

使用上一個步驟中的變數來製作適用於 API 要求的搜尋 URL 格式。 然後傳送要求。

```ruby
uri = URI(uri + path + "?q=" + URI.escape(term))


request = Net::HTTP::Get.new(uri)
request['Ocp-Apim-Subscription-Key'] = accessKey

response = Net::HTTP.start(uri.host, uri.port, :use_ssl => uri.scheme == 'https') do |http|
    http.request(request)
end
```

## <a name="process-and-print-the-json"></a>處理及列印 JSON

收到回應之後，您便可以剖析 JSON 並從中取得值。 例如，第一個結果的縮圖 URL，以及所傳回影像的總數。

```ruby
response.each_header do |key, value|
    # header names are lowercased
    if key.start_with?("bingapis-") or key.start_with?("x-msedge-") then
        puts key + ": " + value
    end
end

parsed_json = JSON.parse(response.body)
total_returned_images = parsed_json["totalEstimatedMatches"]
first_result = parsed_json["value"][0]["thumbnailUrl"]

puts "total number of returned matches: #{total_returned_images}"
puts "Url to the thumbnail of the first returned search result: #{first_result}"
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
