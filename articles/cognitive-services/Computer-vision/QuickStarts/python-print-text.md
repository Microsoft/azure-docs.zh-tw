---
title: 快速入門：擷取印刷文字 (OCR) - REST、Python - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將搭配使用電腦視覺 API 與 Python 來擷取影像中的印刷文字。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: e206f3e716bed558c8041b5714433cecbf9500ac
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49345108"
---
# <a name="quickstart-extract-printed-text-ocr-using-the-rest-api-and-python-in-computer-vision"></a>快速入門：在電腦視覺中使用 REST API 和 Python 擷取印刷文字 (OCR)

在本快速入門中，您將使用電腦視覺的 REST API，利用光學字元辨識 (OCR) 來擷取影像中的印刷文字。 使用 [OCR](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fc) 方法，您可以偵測影像中的印刷文字，然後將辨識出的字元擷取到電腦可使用的字元資料流中。

您可以使用 [MyBinder](https://mybinder.org) (英文) 上的 Jupyter Notebook，以逐步方式執行本快速入門。 若要啟動 Binder，請選取下列按鈕：

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=VisionAPI.ipynb)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services) 。

## <a name="prerequisites"></a>必要條件

- 如果您想要在本機執行此範例，就必須安裝 [Python](https://www.python.org/downloads/)。
- 您必須擁有適用於電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-sample"></a>建立並執行範例

若要建立並執行範例，請執行下列步驟：

1. 將下列程式碼複製到文字編輯器。
1. 視需要在程式碼中進行下列變更：
    1. 將 `subscription_key` 的值取代為您的訂用帳戶金鑰。
    1. 如有需要，請在您取得訂用帳戶金鑰的 Azure 區域中，將 `vision_base_url` 的值取代為電腦視覺資源的端點 URL。
    1. (選擇性) 將 `image_url` 的值取代為您要從中擷取印刷文字之不同影像的 URL。
1. 將程式碼儲存為副檔名為 `.py` 的檔案。 例如： `get-printed-text.py`。
1. 開啟命令提示字元視窗。
1. 出現提示時，使用 `python` 命令執行範例。 例如： `python get-printed-text.py`。

```python
import requests
# If you are using a Jupyter notebook, uncomment the following line.
#%matplotlib inline
import matplotlib.pyplot as plt
from matplotlib.patches import Rectangle
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

ocr_url = vision_base_url + "ocr"

# Set image_url to the URL of an image that you want to analyze.
image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/" + \
    "Atomist_quote_from_Democritus.png/338px-Atomist_quote_from_Democritus.png"

headers = {'Ocp-Apim-Subscription-Key': subscription_key}
params  = {'language': 'unk', 'detectOrientation': 'true'}
data    = {'url': image_url}
response = requests.post(ocr_url, headers=headers, params=params, json=data)
response.raise_for_status()

analysis = response.json()

# Extract the word bounding boxes and text.
line_infos = [region["lines"] for region in analysis["regions"]]
word_infos = []
for line in line_infos:
    for word_metadata in line:
        for word_info in word_metadata["words"]:
            word_infos.append(word_info)
word_infos

# Display the image and overlay it with the extracted text.
plt.figure(figsize=(5, 5))
image = Image.open(BytesIO(requests.get(image_url).content))
ax = plt.imshow(image, alpha=0.5)
for word in word_infos:
    bbox = [int(num) for num in word["boundingBox"].split(",")]
    text = word["text"]
    origin = (bbox[0], bbox[1])
    patch  = Rectangle(origin, bbox[2], bbox[3], fill=False, linewidth=2, color='y')
    ax.axes.add_patch(patch)
    plt.text(origin[0], origin[1], text, fontsize=20, weight="bold", va="top")
plt.axis("off")
```

## <a name="examine-the-response"></a>檢查回應

成功的回應會以 JSON 的形式傳回。 範例網站會在命令提示字元視窗中剖析並顯示成功的回應，如下列範例所示：

```json
{
  "language": "en",
  "orientation": "Up",
  "textAngle": 0,
  "regions": [
    {
      "boundingBox": "21,16,304,451",
      "lines": [
        {
          "boundingBox": "28,16,288,41",
          "words": [
            {
              "boundingBox": "28,16,288,41",
              "text": "NOTHING"
            }
          ]
        },
        {
          "boundingBox": "27,66,283,52",
          "words": [
            {
              "boundingBox": "27,66,283,52",
              "text": "EXISTS"
            }
          ]
        },
        {
          "boundingBox": "27,128,292,49",
          "words": [
            {
              "boundingBox": "27,128,292,49",
              "text": "EXCEPT"
            }
          ]
        },
        {
          "boundingBox": "24,188,292,54",
          "words": [
            {
              "boundingBox": "24,188,292,54",
              "text": "ATOMS"
            }
          ]
        },
        {
          "boundingBox": "22,253,297,32",
          "words": [
            {
              "boundingBox": "22,253,105,32",
              "text": "AND"
            },
            {
              "boundingBox": "144,253,175,32",
              "text": "EMPTY"
            }
          ]
        },
        {
          "boundingBox": "21,298,304,60",
          "words": [
            {
              "boundingBox": "21,298,304,60",
              "text": "SPACE."
            }
          ]
        },
        {
          "boundingBox": "26,387,294,37",
          "words": [
            {
              "boundingBox": "26,387,210,37",
              "text": "Everything"
            },
            {
              "boundingBox": "249,389,71,27",
              "text": "else"
            }
          ]
        },
        {
          "boundingBox": "127,431,198,36",
          "words": [
            {
              "boundingBox": "127,431,31,29",
              "text": "is"
            },
            {
              "boundingBox": "172,431,153,36",
              "text": "opinion."
            }
          ]
        }
      ]
    }
  ]
}
```

## <a name="clean-up-resources"></a>清除資源

不再需要檔案時，請將它刪除。

## <a name="next-steps"></a>後續步驟

探索使用「電腦視覺」在影像中執行光學字元辨識 (OCR)、建立智慧型裁剪縮圖以及偵測、分類、標記和描述視覺特徵 (包括臉部) 的 Python 應用程式。 若要快速地試驗電腦視覺 API，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console)。

> [!div class="nextstepaction"]
> [電腦視覺 API Python 教學課程](../Tutorials/PythonTutorial.md)
