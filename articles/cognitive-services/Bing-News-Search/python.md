---
title: 快速入門：Bing 新聞搜尋 API，Python
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用 Bing 新聞搜尋 API。
services: cognitive-services
author: v-jerkin
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: quickstart
ms.date: 9/21/2017
ms.author: v-jerkin
ms.openlocfilehash: 583b304a742d9abfd799442c9aa2999ad6783a34
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48803540"
---
# <a name="quickstart-for-bing-news-search-api-with-python"></a>Bing 新聞搜尋 API 的 Python 快速入門
本逐步解說示範呼叫 Bing 新聞搜尋 API，以及對產生的 JSON 物件進行後處理的簡單範例。 如需詳細資訊，請參閱 [Bing 新聞搜尋文件](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference) \(英文\)。  

您可以按一下 [launch Binder] \(啟動 Binder\) 徽章，在 [MyBinder](https://mybinder.org) \(英文\) 上以 Jupyter Notebook 執行此範例： 

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingNewsSearchAPI.ipynb)

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
search_url = "https://api.cognitive.microsoft.com/bing/v7.0/news/search"
```

設定 `search_term` 尋找關於 Microsoft 的新聞文章。


```python
search_term = "Microsoft"
```

下列區塊使用 Python 中的 `requests` 程式庫，呼叫 Bing 搜尋 API 並將結果以 JSON 物件的形式傳回。 觀察我們透過 `headers` 字典傳入 API 金鑰，然後透過 `params` 字典搜尋字詞。 若要查看可用來篩選搜尋結果的選項完整清單，請參閱 [REST API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference) \(英文\) 文件。


```python
import requests

headers = {"Ocp-Apim-Subscription-Key" : subscription_key}
params  = {"q": search_term, "textDecorations": True, "textFormat": "HTML"}
response = requests.get(search_url, headers=headers, params=params)
response.raise_for_status()
search_results = response.json()
```

`search_results` 物件包含相關的新文章以及豐富的中繼資料。 例如，下列程式碼會擷取文章的描述。


```python
descriptions = [article["description"] for article in search_results["value"]]
```

這些說明可以再轉譯為以**粗體**醒目提示搜尋關鍵字的資料表。


```python
from IPython.display import HTML
rows = "\n".join(["<tr><td>{0}</td></tr>".format(desc) for desc in descriptions])
HTML("<table>"+rows+"</table>")
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [將新聞分頁](paging-news.md)
> [使用裝飾標記醒目提示文字](hit-highlighting.md)

## <a name="see-also"></a>另請參閱 

 [搜尋網路上的新聞](search-the-web.md)  
 [試試看](https://azure.microsoft.com/services/cognitive-services/bing-news-search-api/)
