---
title: 教學課程：異常偵測 Python
titlesuffix: Azure Cognitive Services
description: 探索會使用異常偵測 API 的 Python Notebook。 將原始資料點傳送到 API，並取得預期的值和異常點。
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.component: anomaly-detection
ms.topic: tutorial
ms.date: 05/01/2018
ms.author: chliang
ms.openlocfilehash: 98f83fbc4db8685526833c3d4f1d15903419ad3f
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48887746"
---
# <a name="tutorial-anomaly-detection-with-python-application"></a>教學課程：使用 Python 應用程式進行異常偵測

[!INCLUDE [PrivatePreviewNote](../../../../../includes/cognitive-services-anomaly-finder-private-preview-note.md)]

本教學課程說明如何使用以 Python 撰寫的異常偵測 API，以及如何使用常用程式庫，以視覺化方式呈現您的結果。 使用 Jupyter 執行本教學課程，然後利用您的訂用帳戶金鑰來試用您自己的資料。 若要了解如何開始使用互動式 Jupyter Notebook，請參考 [Jupyter 文件](http://jupyter.readthedocs.io/en/latest/index.html) \(英文\)。 

## <a name="prerequisites"></a>必要條件

### <a name="subscribe-to-anomaly-detection-and-get-a-subscription-key"></a>訂閱異常偵測並取得訂用帳戶金鑰 

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="download-the-example-code"></a>下載範例程式碼

1. 瀏覽至 [GitHub 中的教學課程 Notebook](https://github.com/MicrosoftAnomalyDetection/python-sample)。
2. 按一下綠色按鈕來複製或下載教學課程。 

## <a name="opening-the-tutorial-notebook-in-jupyter"></a>在 Jupyter 中開啟教學課程 Notebook

1. 開啟命令提示字元並移至 python-sample 資料夾。
2. 從命令提示字元執行 Jupyter notebook 命令，這將會啟動 Jupyter。
3. 在 Jupyter 視窗中，按一下 [Computer Vision API Example.ipynb] \(電腦視覺 API Example.ipynb\) 以開啟教學課程 Notebook。   

## <a name="running-the-tutorial"></a>執行教學課程

若要使用這個 Notebook，您需要有異常偵測 API 的訂用帳戶金鑰。 請造訪訂用帳戶頁面來註冊。 在 [登入] 頁面上，使用您的 Microsoft 帳戶登入，便能訂閱並取得您的金鑰。 完成註冊程序之後，將金鑰貼到下面重新產生之 Notebook 的變數區段中。 不論是主要金鑰還是次要金鑰都可運作。 請務必以引號括住金鑰來讓它變成字串。

```Python

            # Variables
            endpoint = 'https://api.labs.cognitive.microsoft.com/anomalyfinder/v1.0/anomalydetection'
            subscription_key = None #Here you have to paste your primary key

```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [REST API 參考資料](https://dev.labs.cognitive.microsoft.com/docs/services/anomaly-detection/operations/post-anomalydetection)
