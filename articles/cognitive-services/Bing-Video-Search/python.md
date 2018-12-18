---
title: 快速入門：Bing 影片搜尋 (Python)
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用 Bing 影片搜尋 API。
services: cognitive-services
author: v-jerkin
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: quickstart
ms.date: 9/21/2017
ms.author: v-jerkin
ms.openlocfilehash: 797eb476aa3386949b08efb957edf48a97e40d6b
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220005"
---
# <a name="quickstart-bing-video-search-api-with-python"></a>快速入門：使用 Python 的 Bing 影片搜尋 API

本逐步解說示範如何在 Azure 上使用屬於 Microsoft 認知服務的 Bing 影片搜尋 API。 請參閱 [API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference) \(英文\) 了解 API 的技術詳細資料。

您可以按一下 [launch Binder] \(啟動 Binder\) 徽章，在 [MyBinder](https://mybinder.org) \(英文\) 上以 Jupyter Notebook 執行此範例： 

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingVideoSearchAPI.ipynb)


## <a name="prerequisites"></a>必要條件
您必須有具備 **Bing 搜尋 API** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 [免費試用版](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)即足以供本快速入門使用。 您必須要有啟用免費試用版時所提供的存取金鑰，或者您可以從 Azure 儀表板使用付費訂用帳戶金鑰。

## <a name="running-the-walkthrough"></a>執行逐步解說

首先將 Bing API 服務的 `subscription_key` 設定為您的 API 金鑰。


```python
subscription_key = None
assert subscription_key
```

接下來，確認 `search_url` 端點正確無誤。 在撰寫本文時，Bing 搜尋 API 只使用一個端點。 如果您遇到授權錯誤，請對照 Azure 儀表板中的 Bing 搜尋端點以仔細檢查此值。


```python
search_url = "https://api.cognitive.microsoft.com/bing/v7.0/videos/search"
```

設定 `search_term` 尋找小貓的影片


```python
search_term = "kittens"
```

下列區塊使用 Python 中的 `requests` 程式庫，呼叫 Bing 搜尋 API 並將結果以 JSON 物件的形式傳回。 觀察我們透過 `headers` 字典傳入 API 金鑰，然後透過 `params` 字典搜尋字詞。 若要查看可用來篩選搜尋結果的選項完整清單，請參閱 [REST API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference) \(英文\) 文件。


```python
import requests

headers = {"Ocp-Apim-Subscription-Key" : subscription_key}
params  = {"q": search_term, "count":5, "pricing": "free", "videoLength":"short"}
response = requests.get(search_url, headers=headers, params=params)
response.raise_for_status()
search_results = response.json()
```

`search_results` 物件包含相關的影片以及豐富的中繼資料。 若要檢視其中一個影片，請使用其 `embedHtml` 屬性並將它插入 `IFrame`。


```python
from IPython.display import HTML
HTML(search_results["value"][0]["embedHtml"].replace("autoplay=1","autoplay=0"))
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [將影片分頁](paging-videos.md)
> [調整大小和裁剪縮圖影像](resize-and-crop-thumbnails.md)

## <a name="see-also"></a>另請參閱 

 [搜尋網路上的影片](search-the-web.md) (英文) [Bing 影片搜尋](https://azure.microsoft.com/services/cognitive-services/bing-video-search-api/)
