---
title: 快速入門：使用 Python 來執行影像搜尋 - Bing 影像搜尋 API
description: 使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 Python 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。
services: cognitive-services
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 8/20/2018
ms.author: aahi
ms.openlocfilehash: 44cc556e68234fb9957c01fa9f04861293e96e6a
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46298161"
---
# <a name="quickstart-send-search-queries-using-the-rest-api-and-python"></a>快速入門：使用 REST API 和 Python 傳送搜尋查詢

使用本快速入門來進行您對「Bing 影像搜尋 API」的第一次呼叫，並接收 JSON 回應。 這個簡單的 Python 應用程式會將搜尋查詢傳送給 API，並顯示原始結果。

雖然此應用程式是以 Python 撰寫的，但 API 是一種與大多數程式設計語言都相容的 RESTful Web 服務。

您可以按一下 [launch Binder] \(啟動 Binder\) 徽章，在 [MyBinder](https://mybinder.org) \(英文\) 上以 Jupyter Notebook 執行此範例：

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingImageSearchAPI.ipynb)


此外，[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingImageSearchv7.py) 上有此範例的原始程式碼。

## <a name="prerequisites"></a>先決條件

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="running-the-quickstart"></a>執行快速入門

若要開始進行，請將 `subscription_key` 設定成 Bing API 服務的有效訂用帳戶金鑰。

```python
subscription_key = None
assert subscription_key
```

接下來，確認 `search_url` 端點正確無誤。 在撰寫此文章時，Bing 搜尋 API 只使用一個端點。 如果您遇到授權錯誤，請對照 Azure 儀表板中的 Bing 搜尋端點以仔細檢查此值。


```python
search_url = "https://api.cognitive.microsoft.com/bing/v7.0/images/search"
```

設定 `search_term` 尋找小狗的影像。


```python
search_term = "puppies"
```

下列區塊使用 Python 中的 `requests` 程式庫，呼叫 Bing 搜尋 API 並將結果以 JSON 物件的形式傳回。 觀察我們透過 `headers` 字典傳入 API 金鑰，然後透過 `params` 字典搜尋字詞。 若要查看可用來篩選搜尋結果的選項完整清單，請參閱 [REST API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference) \(英文\) 文件。


```python
import requests

headers = {"Ocp-Apim-Subscription-Key" : subscription_key}
params  = {"q": search_term, "license": "public", "imageType": "photo"}
response = requests.get(search_url, headers=headers, params=params)
response.raise_for_status()
search_results = response.json()
```

`search_results` 物件包含實際的影像，以及豐富的中繼資料，例如相關項目。 例如，下列程式碼可以擷取前 16 個結果的縮圖 URL。


```python
thumbnail_urls = [img["thumbnailUrl"] for img in search_results["value"][:16]]
```

接著，使用 `PIL` 程式庫來下載縮圖影像，並使用 `matplotlib` 程式庫以 $4 \times 4$ 的格線呈現這些影像。


```python
%matplotlib inline
import matplotlib.pyplot as plt
from PIL import Image
from io import BytesIO

f, axes = plt.subplots(4, 4)
for i in range(4):
    for j in range(4):
        image_data = requests.get(thumbnail_urls[i+4*j])
        image_data.raise_for_status()
        image = Image.open(BytesIO(image_data.content))        
        axes[i][j].imshow(image)
        axes[i][j].axis("off")
plt.show()
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
