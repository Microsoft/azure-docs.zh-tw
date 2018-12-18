---
title: 快速入門：使用領域模型 - REST、Python - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將搭配使用電腦視覺 API 與 Python，使用領域模型來識別影像中的名人和地標。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: v-deken
ms.openlocfilehash: 93027e2f9cd3a9b0e9c6ef261b8af876022632a4
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45632444"
---
# <a name="quickstart-use-a-domain-model-using-the-rest-api-and-python-in-computer-vision"></a>快速入門：在電腦視覺中利用 REST API 和 Python 來使用領域模型

在本快速入門中，您將利用電腦視覺的 REST API，使用領域模型來識別遠端儲存之影像中的地標或名人 (選擇性)。 使用[辨識特定領域內容](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e200) \(英文\) 方法，您可以套用特定領域的模型來辨識影像中的內容。

您可以使用 [MyBinder](https://mybinder.org) (英文) 上的 Jupyter Notebook，以逐步方式執行本快速入門。 若要啟動 Binder，請選取下列按鈕：

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=VisionAPI.ipynb)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services)。

## <a name="prerequisites"></a>先決條件

- 如果您想要在本機執行此範例，就必須安裝 [Python](https://www.python.org/downloads/)。
- 您必須擁有適用於電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-landmarks-sample"></a>建立並執行地標範例

若要建立並執行地標範例，請執行下列步驟：

1. 將下列程式碼複製到文字編輯器。
1. 視需要在程式碼中進行下列變更：
    1. 將 `subscription_key` 的值取代為您的訂用帳戶金鑰。
    1. 如有需要，請在您取得訂用帳戶金鑰的 Azure 區域中，將 `vision_base_url` 的值取代為電腦視覺資源的端點 URL。
    1. (選擇性) 將 `image_url` 的值取代為您要在其中偵測地標之不同影像的 URL。
1. 將程式碼儲存為 `.py` 副檔名的檔案。 例如： `get-landmarks.py`。
1. 開啟命令提示字元視窗。
1. 出現提示時，使用 `python` 命令來執行範例。 例如： `python get-landmarks.py`。

```python
import requests
# If you are using a Jupyter notebook, uncomment the following line.
#%matplotlib inline
import matplotlib.pyplot as plt
from PIL import Image
from io import BytesIO

# Replace <Subscription Key> with your valid subscription key.
subscription_key = "<Subscription Key>"
assert subscription_key

# You must use the same region in your REST call as you used to get your
# subscription keys. For example, if you got your subscription keys from
# westus, replace "westcentralus" in the URI below with "westus".
#
# Free trial subscription keys are generated in the westcentralus region.
# If you use a free trial subscription key, you shouldn't need to change
# this region.
vision_base_url = "https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/"

landmark_analyze_url = vision_base_url + "models/landmarks/analyze"

# Set image_url to the URL of an image that you want to analyze.
image_url = "https://upload.wikimedia.org/wikipedia/commons/f/f6/" + \
    "Bunker_Hill_Monument_2005.jpg"

headers = {'Ocp-Apim-Subscription-Key': subscription_key}
params  = {'model': 'landmarks'}
data    = {'url': image_url}
response = requests.post(
    landmark_analyze_url, headers=headers, params=params, json=data)
response.raise_for_status()

# The 'analysis' object contains various fields that describe the image. The
# most relevant landmark for the image is obtained from the 'result' property.
analysis = response.json()
assert analysis["result"]["landmarks"] is not []
print(analysis)
landmark_name = analysis["result"]["landmarks"][0]["name"].capitalize()

# Display the image and overlay it with the landmark name.
image = Image.open(BytesIO(requests.get(image_url).content))
plt.imshow(image)
plt.axis("off")
_ = plt.title(landmark_name, size="x-large", y=-0.1)
```

## <a name="examine-the-response-for-the-landmarks-sample"></a>檢查地標範例的回應

成功的回應會以 JSON 的形式傳回。 範例網站會在命令提示字元視窗中剖析並顯示成功的回應，如下列範例所示：

```json
{
  "result": {
    "landmarks": [
      {
        "name": "Bunker Hill Monument",
        "confidence": 0.9768505096435547
      }
    ]
  },
  "requestId": "659a10cd-44bb-44db-9147-a295b853b2b8",
  "metadata": {
    "height": 1600,
    "width": 1200,
    "format": "Jpeg"
  }
}
```

## <a name="create-and-run-the-celebrities-sample"></a>建立並執行名人範例

若要建立並執行地標範例，請執行下列步驟：

1. 將下列程式碼複製到文字編輯器。
1. 視需要在程式碼中進行下列變更：
    1. 將 `subscription_key` 的值取代為您的訂用帳戶金鑰。
    1. 如有需要，請在您取得訂用帳戶金鑰的 Azure 區域中，將 `vision_base_url` 的值取代為電腦視覺資源的端點 URL。
    1. (選擇性) 將 `image_url` 的值取代為您要在其中偵測名人之不同影像的 URL。
1. 將程式碼儲存為 `.py` 副檔名的檔案。 例如： `get-celebrities.py`。
1. 開啟命令提示字元視窗。
1. 出現提示時，使用 `python` 命令來執行範例。 例如： `python get-celebrities.py`。

```python
import requests
# If you are using a Jupyter notebook, uncomment the following line.
#%matplotlib inline
import matplotlib.pyplot as plt
from PIL import Image
from io import BytesIO

# Replace <Subscription Key> with your valid subscription key.
subscription_key = "<Subscription Key>"
assert subscription_key

vision_base_url = "https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/"

celebrity_analyze_url = vision_base_url + "models/celebrities/analyze"

# Set image_url to the URL of an image that you want to analyze.
image_url = "https://upload.wikimedia.org/wikipedia/commons/d/d9/" + \
    "Bill_gates_portrait.jpg"

headers = {'Ocp-Apim-Subscription-Key': subscription_key}
params  = {'model': 'celebrities'}
data    = {'url': image_url}
response = requests.post(
    celebrity_analyze_url, headers=headers, params=params, json=data)
response.raise_for_status()

# The 'analysis' object contains various fields that describe the image. The
# most relevant celebrity for the image is obtained from the 'result' property.
analysis = response.json()
assert analysis["result"]["celebrities"] is not []
print(analysis)
celebrity_name = analysis["result"]["celebrities"][0]["name"].capitalize()

# Display the image and overlay it with the celebrity name.
image = Image.open(BytesIO(requests.get(image_url).content))
plt.imshow(image)
plt.axis("off")
_ = plt.title(celebrity_name, size="x-large", y=-0.1)
```

## <a name="examine-the-response-for-the-celebrities-sample"></a>檢查名人範例的回應

成功的回應會以 JSON 的形式傳回。 範例網站會在命令提示字元視窗中剖析並顯示成功的回應，如下列範例所示：


```json
{
  "result": {
    "celebrities": [
      {
        "faceRectangle": {
          "top": 123,
          "left": 156,
          "width": 187,
          "height": 187
        },
        "name": "Bill Gates",
        "confidence": 0.9993845224380493
      }
    ]
  },
  "requestId": "f14ec1d0-62d4-4296-9ceb-6b5776dc2020",
  "metadata": {
    "height": 521,
    "width": 550,
    "format": "Jpeg"
  }
}
```

## <a name="clean-up-resources"></a>清除資源

不再需要檔案時，請針對這兩個範例刪除檔案。

## <a name="next-steps"></a>後續步驟

探索使用「電腦視覺」在影像中執行光學字元辨識 (OCR)、建立智慧型裁剪縮圖以及偵測、分類、標記和描述視覺特徵 (包括臉部) 的 Python 應用程式。 若要快速地試驗電腦視覺 API，請嘗試使用 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console) \(英文\)。

> [!div class="nextstepaction"]
> [電腦視覺 API Python 教學課程](../Tutorials/PythonTutorial.md)
