---
title: 快速入門：使用 Python 執行搜尋 - Bing Web 搜尋 API
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將學習如何使用 Python 來第一次呼叫 Bing Web 搜尋 API，並接收 JSON 回應。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: quickstart
ms.date: 8/16/2018
ms.author: erhopf
ms.openlocfilehash: 22dc88d2b924587818f7767105872f01f2e43faf
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46129143"
---
# <a name="quickstart-use-python-to-call-the-bing-web-search-api"></a>快速入門：使用 Python 來呼叫 Bing Web 搜尋 API  

本快速入門可讓您在 10 分鐘內完成第一次呼叫 Bing Web 搜尋 API，並接收 JSON 回應。  

[!INCLUDE [bing-web-search-quickstart-signup](../../../../includes/bing-web-search-quickstart-signup.md)]

此範例在 [MyBinder](https://mybinder.org) (英文) 上以 Jupyter Notebook 形式執行。 按一下「啟動 Binder」徽章：

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingWebSearchAPI.ipynb)

## <a name="define-variables"></a>定義變數

將 `subscription_key` 值換成您的 Azure 帳戶中有效的訂用帳戶金鑰。

```python
subscription_key = "YOUR_ACCESS_KEY"
assert subscription_key
```

宣告 Bing Web 搜尋 API 端點。 如果您遇到任何授權錯誤，請對照 Azure 儀表板中的 Bing 搜尋端點，仔細檢查此值。

```python
search_url = "https://api.cognitive.microsoft.com/bing/v7.0/search"
```

請自行取代 `search_term` 的值來自訂搜尋查詢。

```python
search_term = "Azure Cognitive Services"
```

## <a name="make-a-request"></a>發出要求

下列區塊會使用 `requests` 程式庫呼叫 Bing Web 搜尋 API，並以 JSON 物件的形式傳回結果。 API 金鑰會透過 `headers` 字典傳入，而搜尋字詞和查詢參數則會透過 `params` 字典傳入。 如需選項和參數的完整清單，請參閱 [Bing Web 搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference) (英文) 文件。

```python
import requests

headers = {"Ocp-Apim-Subscription-Key" : subscription_key}
params  = {"q": search_term, "textDecorations":True, "textFormat":"HTML"}
response = requests.get(search_url, headers=headers, params=params)
response.raise_for_status()
search_results = response.json()
```

## <a name="format-and-display-the-response"></a>格式化並顯示回應

`search_results` 物件包含搜尋結果和中繼資料，例如相關的查詢和網頁。 此程式碼會使用 `IPython.display` 程式庫來格式化回應，並顯示在瀏覽器中。

```python
from IPython.display import HTML

rows = "\n".join(["""<tr>
                       <td><a href=\"{0}\">{1}</a></td>
                       <td>{2}</td>
                     </tr>""".format(v["url"],v["name"],v["snippet"]) \
                  for v in search_results["webPages"]["value"]])
HTML("<table>{0}</table>".format(rows))
```

## <a name="sample-code-on-github"></a>GitHub 上的程式碼範例

如果您想要在本機執行此程式碼，完整的[範例在 GitHub 上](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingWebSearchv7.js) (英文)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing Web 搜尋單頁應用程式教學課程](../tutorial-bing-web-search-single-page-app.md)

[!INCLUDE [bing-web-search-quickstart-see-also](../../../../includes/bing-web-search-quickstart-see-also.md)]
