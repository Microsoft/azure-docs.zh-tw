---
title: 教學課程：使用 C# 擷取影像詳細資料 - Bing 影像搜尋 API
titleSuffix: Azure Cognitive Services
description: 使用此文章建立 C# 應用程式，以使用 Bing 影像搜尋 API 來擷取影像詳細資料。
services: cognitive-services
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: tutorial
ms.date: 9/14/2018
ms.author: aahi
ms.openlocfilehash: 96d011a04c97d309409062a286bdd7a17db9cda5
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297651"
---
# <a name="tutorial-extract-image-details-using-the-bing-image-search-api-and-c"></a>教學課程：使用 Bing 影像搜尋 API 和 C# 擷取影像詳細資料

有多個[端點](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/image-search-endpoint)可透過 Bing 影像搜尋 API 來取得。 `/details` 端點會接受含有影像的 POST 要求，並可傳回有關該影像的各種不同詳細資料。 這個 C# 應用程式會使用此 API 傳送影像，並顯示由 Bing 傳回的詳細資料，也就是 JSON 物件，如下所示：

![[JSON 結果]](media/cognitive-services-bing-images-api/jsonResult.jpg)

本教學課程說明如何：

> [!div class="checklist"]
> * 在 `POST` 要求中使用影像搜尋 `/details` 端點
> * 指定要求的標頭
> * 使用 URL 參數來指定結果
> * 上傳影像資料並傳送 `POST` 要求
> * 將 JSON 結果列印到主控台

此範例的原始程式碼位於 [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/BingGetSimilarImages.cs)。

## <a name="prerequisites"></a>先決條件

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/) 的任意版本。

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="construct-an-image-details-search-request"></a>建構影像詳細資料搜尋要求

以下是 `/details` 端點，此端點會接受要求主體中含有影像資料的 POST 要求。
```
https://api.cognitive.microsoft.com/bing/v7.0/images/details
```

建構搜尋要求 URL 時，`modules` 參數會遵循前一個端點，並指定結果將包含的詳細資料類型：

* `modules=All`
* `modules=RecognizedEntities` (影像中可見的人員或地點)

在 POST 要求中指定 `modules=All`，以取得包含下列各項的 JSON 文字：

* `bestRepresentativeQuery` - Bing 查詢，其傳回的影像類似於所上傳的影像
* `detectedObjects` - 在影像中找到的物件
* `image` - 適用於影像的中繼資料
* `imageInsightsToken` - 適用於稍後從影像中取得 `RecognizedEntities` (影像中顯示的人員或位置) 之 GET 要求的權杖。
* `imageTags` - 適用於影像的標記
* `pagesIncluding` - 包含影像的網頁
* `relatedSearches` - 根據影像中的詳細資料所進行的搜尋。
* `visuallySimilarImages` - Web 上的類似影像。

指定 POST 要求中的 `modules=RecognizedEntities`，以便只取得 `imageInsightsToken`，其可在後續 GET 要求中用來識別影像中的人員或位置。

## <a name="create-a-webclient-object-and-set-headers-for-the-api-request"></a>建立 WebClient 物件並設定 API 要求的標頭

建立 `WebClient` 物件，並設定標頭。 Bing 搜尋 API 的所有要求都需要 `Ocp-Apim-Subscription-Key`。 上傳影像的 `POST` 要求也必須指定 `ContentType: multipart/form-data`。

```javascript
WebClient client = new WebClient();
client.Headers["Ocp-Apim-Subscription-Key"] = accessKey;
client.Headers["ContentType"] = "multipart/form-data";
```

## <a name="upload-the-image-and-display-the-results"></a>上傳影像並顯示結果

`WebClient` 類別的 `UpLoadFile()` 方法會針對 `POST` 要求來將資料格式化，包括將 `RequestStream` 格式化，然後呼叫 `HttpWebRequest`。

利用 `/details` 端點和要上傳的影像檔呼叫 `WebClient.UpLoadFile()`。 使用 JSON 回應來將 `SearchResult` 結構的執行個體初始化，並儲存回應。

```javascript        
const string uriBase = "https://api.cognitive.microsoft.com/bing/v7.0/images/details";
// The image to upload. Replace with your file and path.
const string imageFile = "your-image.jpg";
byte[] resp = client.UploadFile(uriBase + "?modules=All", imageFile);
var json = System.Text.Encoding.Default.GetString(resp);
// Create result object for return
var searchResult = new SearchResult()
{
    jsonResult = json,
    relevantHeaders = new Dictionary<String, String>()
};
```
此 JSON 回應接著可列印到主控台。

## <a name="use-an-image-insights-token-in-a-request"></a>在要求中使用影像深入解析權杖

若要將傳回的 `ImageInsightsToken` 與 `POST` 的結果搭配使用，您可以將它加入至 `GET` 要求。 例如︰

```
https://api.cognitive.microsoft.com/bing/v7.0/images/details?InsightsToken="bcid_A2C4BB81AA2C9EF8E049C5933C546449*ccid_osS7gaos*mid_BF7CC4FC4A882A3C3D56E644685BFF7B8BACEAF2
```

如果影像中有可辨識的人員或地點，此要求會傳回其相關資訊。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在單頁 Web 應用程式中顯示影像與搜尋選項](tutorial-bing-image-search-single-page-app.md)

## <a name="see-also"></a>另請參閱

* [Bing 影像搜尋 API 參考](//docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
