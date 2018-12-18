---
title: 如何搭配 Python 使用異常搜尋工具 API - Microsoft 認知服務 | Microsoft Docs
description: 取得資訊和程式碼範例，以協助您搭配 Python 快速開始使用認知服務中的異常搜尋工具。
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.technology: anomaly-detection
ms.topic: article
ms.date: 05/01/2018
ms.author: chliang
ms.openlocfilehash: c14916b0644edab613b298d6e71f8bbb9a6bb804
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369127"
---
# <a name="use-the-anomaly-finder-api-with-python"></a>搭配 Python 使用異常搜尋工具 API

本文提供資訊和程式碼範例，協助您快速開始搭配 Python 使用異常搜尋工具 API，以便完成對於時間序列資料取得異常結果的工作。

## <a name="prerequisites"></a>先決條件

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="getting-anomaly-points-with-anomaly-finder-api-using-python"></a>搭配 Python 使用異常搜尋工具 API 取得異常點 

[!INCLUDE [DataContract](../includes/datacontract.md)]

### <a name="example-of-time-series-data"></a>時間序列資料範例

時間序列資料點的範例如下所示。

[!INCLUDE [Request](../includes/request.md)]

### <a name="analyze-data-and-get-anomaly-points-python-example"></a>分析資料並取得異常點的 Python 範例

確定您已安裝 python3，然後建立名為 detect.py 的 python 可執行檔。 在 detect.py 中，您應該包含下列程式碼。 執行程式碼之前，請記得以您的有效訂用帳戶金鑰取代 `[YOUR_SUBSCRIPTION_KEY]` 值。
以您的資料點取代 `[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]`。

```python
import requests
import json


def detect(url, subscription_key, request_data):
    headers = {'Content-Type': 'application/json', 'Ocp-Apim-Subscription-Key': subscription_key}
    response = requests.post(url, data=json.dumps(request_data), headers=headers)
    if response.status_code == 200:
        return json.loads(response.content.decode("utf-8"))
    else:
        print(response.status_code)
        raise Exception(response.text)


sample_data = "[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]"
endpont = "https://api.labs.cognitive.microsoft.com/anomalyfinder/v1.0/anomalydetection"
subscription_key = "[YOUR_SUBSCRIPTION_KEY]"

result = detect(endpont, subscription_key, sample_data)
print(result)

```

### <a name="example-response"></a>範例回應

以 JSON 傳回成功的回應。 範例回應如下所示。
[!INCLUDE [Response](../includes/response.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Python 應用程式](../tutorials/python-tutorial.md)
