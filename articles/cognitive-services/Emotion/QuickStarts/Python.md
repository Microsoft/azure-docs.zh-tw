---
title: 快速入門：辨識影像中的臉部表情 - 表情 API，Python
description: 取得資訊和程式碼範例，以協助您搭配 Python 快速開始使用表情 API。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: emotion-api
ms.topic: quickstart
ms.date: 02/05/2018
ms.author: anroth
ROBOTS: NOINDEX
ms.openlocfilehash: c7611628918cf40800d173dc9404b0948b9a68a4
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48236562"
---
# <a name="quickstart-build-an-app-to-recognize-emotions-on-faces-in-an-image"></a>快速入門：建置應用程式以辨識影像中的臉部表情。

> [!IMPORTANT]
> 表情 API 將於 2019 年 2 月 15 日淘汰。 表情辨識功能現已公開推出，是[臉部 API](https://docs.microsoft.com/azure/cognitive-services/face/) 的一部分。 

本逐步解說提供資訊和程式碼範例，以協助您搭配 Python 快速開始使用[表情 API 辨識方法](https://westus.dev.cognitive.microsoft.com/docs/services/5639d931ca73072154c1ce89/operations/563b31ea778daf121cc3a5fa) \(英文\) 來辨識出影像中一或多個人員所表現出來的表情。

您可以按一下 [launch Binder] \(啟動 Binder\) 徽章，在 [MyBinder](https://mybinder.org) \(英文\) 上以 Jupyter Notebook 執行此範例：[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=EmotionAPI.ipynb)


## <a name="prerequisite"></a>必要條件
從[這裡](https://azure.microsoft.com/try/cognitive-services/)取得免費的訂用帳戶金鑰

## <a name="running-the-walkthrough"></a>執行逐步解說
若要繼續進行本逐步解說，請以您稍早取得的 API 金鑰取代 `subscription_key`。


```python
subscription_key = None
assert subscription_key
```

接下來，確認服務 URL 對應至您在設定 API 金鑰時所使用的區域。 如果您使用的是試用金鑰，則不需進行任何變更。


```python
emotion_recognition_url = "https://westus.api.cognitive.microsoft.com/emotion/v1.0/recognize"
```

本逐步解說會使用磁碟所儲存的影像。 您也可以使用能透過可公開存取的 URL 來取得的影像。 如需詳細資訊，請參閱 [REST API 文件](https://westus.dev.cognitive.microsoft.com/docs/services/5639d931ca73072154c1ce89/operations/563b31ea778daf121cc3a5fa)。

因為影像資料會作為要求本文的一部分來傳遞，請注意您需要將 `Content-Type` 標頭設定為 `application/octet-stream`。 如果您透過 URL 傳入影像，請記得將標頭設定為：
```python
header = {'Ocp-Apim-Subscription-Key': subscription_key }
```
建立包含 URL 的字典：
```python
data = {'url': image_url}
```
並使用下列命令將該字典傳遞至 `requests` 程式庫：
```python
requests.post(emotion_recognition_url, headers=headers, json=image_data)
```

先從[表情 API](https://azure.microsoft.com/services/cognitive-services/emotion/) 網站下載幾個影像範例。


```bash
%%bash
mkdir -p images
curl -Ls https://aka.ms/csnb-emotion-1 -o images/emotion_1.jpg
curl -Ls https://aka.ms/csnb-emotion-2 -o images/emotion_2.jpg
```


```python
image_path = "images/emotion_1.jpg"
image_data = open(image_path, "rb").read()
```


```python
import requests
headers  = {'Ocp-Apim-Subscription-Key': subscription_key, "Content-Type": "application/octet-stream" }
response = requests.post(emotion_recognition_url, headers=headers, data=image_data)
response.raise_for_status()
analysis = response.json()
analysis
```




    [{'faceRectangle': {'height': 162, 'left': 130, 'top': 141, 'width': 162},
      'scores': {'anger': 9.29041e-06,
       'contempt': 0.000118981574,
       'disgust': 3.15619363e-05,
       'fear': 0.000589638,
       'happiness': 0.06630674,
       'neutral': 0.00555004273,
       'sadness': 7.44669524e-06,
       'surprise': 0.9273863}}]



所傳回的 JSON 物件包含已辨識出的臉部週框以及所偵測到的表情。 每個表情都會與介於 0 到 1 的分數相關聯，分數越高越表示這是表情。

下列幾行程式碼已使用 `matplotlib` 程式庫偵測到影像中臉部的表情。 為了讓版面清楚，此處只顯示前三個表情。


```python
%matplotlib inline
import matplotlib.pyplot as plt
from matplotlib.patches import Rectangle
from io import BytesIO
from PIL import Image

plt.figure(figsize=(5,5))

image  = Image.open(BytesIO(image_data))
ax     = plt.imshow(image, alpha=0.6)

for face in analysis:
    fr = face["faceRectangle"]
    em = face["scores"]
    origin = (fr["left"], fr["top"])
    p = Rectangle(origin, fr["width"], fr["height"], fill=False, linewidth=2, color='b')
    ax.axes.add_patch(p)
    ct = "\n".join(["{0:<10s}{1:>.4f}".format(k,v) for k, v in sorted(list(em.items()),key=lambda r: r[1], reverse=True)][:3])
    plt.text(origin[0], origin[1], ct, fontsize=20)    
_ = plt.axis("off")
```

接著顯示的 `annotate_image` 函式可用來在給定影像檔在檔案系統上所在路徑的情況下，於影像檔之上重疊表情。 其依據的是稍早所示用於呼叫表情 API 的程式碼。


```python
def annotate_image(image_path):    
    image_data = open(image_path, "rb").read()
    headers  = {'Ocp-Apim-Subscription-Key': subscription_key, "Content-Type": "application/octet-stream" }
    response = requests.post(emotion_recognition_url, headers=headers, data=image_data)
    response.raise_for_status()
    analysis = response.json()

    plt.figure()

    image  = Image.open(image_path)
    ax     = plt.imshow(image, alpha=0.6)

    for face in analysis:
        fr = face["faceRectangle"]
        em = face["scores"]
        origin = (fr["left"], fr["top"])
        p = Rectangle(origin, fr["width"], fr["height"], fill=False, linewidth=2, color='b')
        ax.axes.add_patch(p)
        ct = "\n".join(["{0:<10s}{1:>.4f}".format(k,v) for k, v in sorted(list(em.items()),key=lambda r: r[1], reverse=True)][:3])
        plt.text(origin[0], origin[1], ct, fontsize=20, va="bottom")    
    _ = plt.axis("off")
```

最後，您可以對影像檔呼叫 `annotate_image` 函式，如下列程式行所示：


```python
annotate_image("images/emotion_2.jpg")
```
