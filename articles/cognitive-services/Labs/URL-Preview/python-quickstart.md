---
title: 快速入門：專案 URL 預覽 Python
titlesuffix: Azure Cognitive Services
description: 快速開始使用專案 URL 預覽與 Python 的指令碼範例。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-url-preview
ms.topic: quickstart
ms.date: 03/29/2018
ms.author: rosh
ms.openlocfilehash: c4af701188c6453bd3c067b838519c32862af13d
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48867267"
---
# <a name="quickstart-url-preview-with-python"></a>快速入門：使用 Python 進行 URL 預覽

下列 Python 範例會建立 SwiftKey 網站的 URL 預覽： https://swiftkey.com/en。

## <a name="prerequisites"></a>必要條件

取得免費試用版[認知服務實驗室](https://aka.ms/answersearchsubscription)的存取金鑰

此範例使用 Python 3.6。

## <a name="code-scenario"></a>程式碼案例 

下列程式碼會建立 URL 預覽。
其實作的步驟如下：
1. 宣告變數以依主機及路徑指定端點。
2. 指定要預覽的查詢 URL，並新增查詢參數。  
3. 設定查詢參數。
4. 定義建立要求的搜尋函式，並新增 *Ocp-Apim-Subscription-Key* 標頭。
5. 設定 *Ocp-Apim-Subscription-Key* 標頭。 
6. 進行連線，並傳送要求。
7. 列印 JSON 結果。

此示範的完整程式碼如下：

````
import http.client, urllib.parse
import json

# Replace the subscriptionKey string value with your valid subscription key.
subscriptionKey = 'your-subscription-key'

host = 'api.labs.cognitive.microsoft.com'
path = '/urlpreview/v7.0/search'

query = 'https://SwiftKey.com'

params = '?q=' + urllib.parse.quote (query)

def get_preview ():
    headers = {'Ocp-Apim-Subscription-Key': subscriptionKey}
    conn = http.client.HTTPSConnection (host)
    conn.request ("GET", path + params, None, headers)
    response = conn.getresponse ()
    return response.read ()

result = get_preview ()
print (json.dumps(json.loads(result), indent=4))
````
## <a name="next-steps"></a>後續步驟
- [C# 快速入門](csharp.md)
- [JAVA 快速入門](java-quickstart.md)
- [JavaScript 快速入門](javascript.md)
- [Node URL 快速入門](node-quickstart.md)