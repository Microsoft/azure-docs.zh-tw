---
title: 教學課程：偵測並框出影像中的人臉 - 臉部 API、Python
titleSuffix: Azure Cognitive Services
description: 了解如何搭配 Python SDK 使用臉部 API 來偵測影像中的人臉。
services: cognitive-services
author: SteveMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: tutorial
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: 6cc3ac25d2196c0275b445503b79b9ac06a791d3
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127732"
---
# <a name="tutorial-detect-and-frame-faces-with-the-face-api-and-python"></a>教學課程：使用臉部 API 和 Python 偵測並框出人臉 

在本教學課程中，您將了解如何透過 Python SDK 叫用臉部 API 來偵測影像中的人類。

## <a name="prerequisites"></a>必要條件

若要使用教學課程，您必須執行下列動作：

- 安裝 Python 2.7+ 或 Python 3.5+。
- 安裝 pip。
- 安裝適用於臉部 API 的 Python SDK，如下所示：

```bash
pip install cognitive_face
```

- 取得 Microsoft 認知服務的[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/)。 您可以在本教學課程中使用主要或次要金鑰 (請注意，若要使用任何臉部 API，您必須擁有有效的訂用帳戶金鑰)。

## <a name="detect-a-face-in-an-image"></a>偵測影像中的人臉

```python
import cognitive_face as CF

KEY = '<Subscription Key>'  # Replace with a valid subscription key (keeping the quotes in place).
CF.Key.set(KEY)

BASE_URL = 'https://westus.api.cognitive.microsoft.com/face/v1.0/'  # Replace with your regional Base URL
CF.BaseUrl.set(BASE_URL)

# You can use this example JPG or replace the URL below with your own URL to a JPEG image.
img_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
faces = CF.face.detect(img_url)
print(faces)
```

以下是範例結果。 它是所偵測臉部的 `list`。 清單中的每個項目都是 `dict` 執行個體，其中 `faceId` 是所偵測臉部的唯一識別碼，而 `faceRectangle` 描述所偵測臉部的位置。 臉部識別碼會在 24 小時內到期。

```python
[{u'faceId': u'68a0f8cf-9dba-4a25-afb3-f9cdf57cca51', u'faceRectangle': {u'width': 89, u'top': 66, u'height': 89, u'left': 446}}]
```

## <a name="draw-rectangles-around-the-faces"></a>在臉部周圍繪製矩形

您可以使用從上一個命令收到的 json 座標，在影像上繪製矩形以視覺方式表示每張臉。 您需要 `pip install Pillow` 才能使用 `PIL` 影像處理模組。  在檔案的最上方，新增下列項目：

```python
import requests
from io import BytesIO
from PIL import Image, ImageDraw
```

然後，在 `print(faces)` 後面，將下列內容加入您的指令碼：

```python
#Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary['faceRectangle']
    left = rect['left']
    top = rect['top']
    bottom = left + rect['height']
    right = top + rect['width']
    return ((left, top), (bottom, right))

#Download the image from the url
response = requests.get(img_url)
img = Image.open(BytesIO(response.content))

#For each face returned use the face rectangle and draw a red box.
draw = ImageDraw.Draw(img)
for face in faces:
    draw.rectangle(getRectangle(face), outline='red')

#Display the image in the users default image browser.
img.show()
```

## <a name="further-exploration"></a>進一步探索

為了協助您進一步探索臉部 API，本教學課程提供 GUI 範例。 若要執行它，請先安裝 [wxPython](https://wxpython.org/pages/downloads/)，然後執行下列命令。

```bash
git clone https://github.com/Microsoft/Cognitive-Face-Python.git
cd Cognitive-Face-Python
python sample
```

## <a name="summary"></a>總結

在本教學課程中，您已了解透過叫用 Python SDK 使用臉部 API 的基本程序。 如需 API 詳細資料的詳細資訊，請參閱操作說明和 [API Reference](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (API 參考)。

## <a name="related-topics"></a>相關主題

- [在 CSharp 中開始使用臉部 API](FaceAPIinCSharpTutorial.md)
- [在適用於 Android 的 Java 中開始使用臉部 API](FaceAPIinJavaForAndroidTutorial.md)
