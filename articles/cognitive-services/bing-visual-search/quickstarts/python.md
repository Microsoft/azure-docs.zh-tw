---
title: 快速入門：建立圖像式搜尋查詢 (Python) - Bing 圖像式搜尋
titleSuffix: Azure Cognitive Services
description: 示範如何將影像上傳到 Bing 圖像式搜尋 API，並取回影像的相關見解。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-visual-search
ms.topic: quickstart
ms.date: 5/16/2018
ms.author: scottwhi
ms.openlocfilehash: 16d3d0ddf77e37e32cc50961a3870b820ac2748e
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48884236"
---
# <a name="quickstart-your-first-bing-visual-search-query-in-python"></a>快速入門：使用 Python 的第一個 Bing 圖像式搜尋查詢

Bing 圖像式搜尋 API 會傳回您所提供影像的相關資訊。 您可以使用影像的 URL、見解權杖，或上傳影像來提供影像。 如需這些選項的資訊，請參閱[什麼是 Bing 圖像式搜尋 API？](../overview.md) 本文將示範如何上傳影像。 在拍攝知名地標的照片並取回其相關資訊的行動裝置案例中，上傳影像可能很有用。 例如，見解可能包含關於地標的雜項。 

若您上傳本機影像，以下會顯示您必須包含在 POST 本文中的表單資料。 表單資料必須包含 Content-Disposition 標頭。 其 `name` 參數必須設定為 "image"，而 `filename` 參數可以設定為任何字串。 表單的內容是影像的二進位檔案。 您可以上傳的影像大小上限為 1 MB。 

```
--boundary_1234-abcd
Content-Disposition: form-data; name="image"; filename="myimagefile.jpg"

ÿØÿà JFIF ÖÆ68g-¤CWŸþ29ÌÄøÖ‘º«™æ±èuZiÀ)"óÓß°Î= ØJ9á+*G¦...

--boundary_1234-abcd--
```

本文包含簡單的主控台應用程式，它會傳送 Bing 圖像式搜尋 API 要求，並顯示 JSON 搜尋結果。 雖然此應用程式是以 Python 撰寫，但 API 是一種與任何程式語言相容的 RESTful Web 服務，可產生 HTTP 要求，並剖析 JSON。 

## <a name="prerequisites"></a>必要條件

您需要 [Python 3](https://www.python.org/) 以執行此程式碼。

在本快速入門中，您可以使用[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)的訂用帳戶金鑰或付費的訂用帳戶金鑰。

## <a name="running-the-walkthrough"></a>執行逐步解說

若要執行此應用程式，請遵循下列步驟：

1. 在您最愛的 IDE 或編輯器中，建立新的 Python 專案。
2. 建立名為 visualsearch.py 的檔案，並新增本快速入門中所示的程式碼。
3. 以您的訂用帳戶金鑰取代 `SUBSCRIPTION_KEY` 值。
3. 以要上傳之影像的路徑取代 `imagePath` 值。
4. 執行程式。



以下示範如何使用 Python 中的多部分表單資料來傳送訊息。

```python
"""Bing Visual Search upload image example"""

# Download and install Python at https://www.python.org/
# Run the following in a command console window
# pip3 install requests

import requests, json


BASE_URI = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch'

SUBSCRIPTION_KEY = '<yoursubscriptionkeygoeshere>'

HEADERS = {'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY}

imagePath = '<pathtoyourimagetouploadgoeshere>'

file = {'image' : ('myfile', open(imagePath, 'rb'))}

def main():
    
    try:
        response = requests.post(BASE_URI, headers=HEADERS, files=file)
        response.raise_for_status()
        print_json(response.json())

    except Exception as ex:
        raise ex


def print_json(obj):
    """Print the object as json"""
    print(json.dumps(obj, sort_keys=True, indent=2, separators=(',', ': ')))



# Main execution
if __name__ == '__main__':
    main()
```


## <a name="next-steps"></a>後續步驟

[使用見解權杖取得影像的相關見解](../use-insights-token.md)  
[Bing 圖像式搜尋影像上傳教學課程](../tutorial-visual-search-image-upload.md)
[Bing 圖像式搜尋單頁應用程式教學課程](../tutorial-bing-visual-search-single-page-app.md)  
[Bing 圖像式搜尋概觀](../overview.md)  
[試試看](https://aka.ms/bingvisualsearchtryforfree)  
[取得免費試用的存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-visual-search-api)  
[Bing 圖像式搜尋 API 參考](https://aka.ms/bingvisualsearchreferencedoc)
