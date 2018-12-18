---
title: 教學課程：電腦視覺 API Python
titlesuffix: Azure Cognitive Services
description: 了解如何透過使用 Jupyter Notebook，將「電腦視覺 API」與 Python 搭配使用。 使用常用的程式庫以視覺方式呈現您的結果。
services: cognitive-services
author: KellyDF
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: tutorial
ms.date: 02/25/2017
ms.author: kefre
ms.openlocfilehash: 046250d3d2142badaac35490eff27bcac220fea9
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344892"
---
# <a name="tutorial-computer-vision-api-python"></a>教學課程：電腦視覺 API Python

本教學課程說明如何使用以 Python 撰寫的「電腦視覺 API」，以及如何使用一些常用程式庫將您的結果以視覺化方式呈現。 使用 Jupyter 來執行教學課程。 若要了解如何開始使用互動式 Jupyter Notebook，請參考：[Jupyter 文件](http://jupyter.readthedocs.io/en/latest/index.html) \(英文\)。 

## <a name="open-the-tutorial-notebook-in-jupyter"></a>在 Jupyter 中開啟教學課程 Notebook 

1. 瀏覽至 [GitHub 中的教學課程筆記本](https://github.com/Microsoft/Cognitive-Vision-Python)。 
2. 按一下綠色按鈕來複製或下載教學課程。 
3. 開啟命令提示字元，然後移至 _Cognitive-Vision-Python-master\Jupyter Notebook_資料夾。 
4. 從命令提示字元執行 **jupyter notebook** 命令。 這會啟動 Jupyter。
5. 在 Jupyter 視窗中，按一下 [Computer Vision API Example.ipynb] \(電腦視覺 API Example.ipynb\) 以開啟教學課程 Notebook 

## <a name="run-the-tutorial"></a>執行教學課程

若要使用這個 Notebook，您將需要有「電腦視覺 API」的訂用帳戶金鑰。 請瀏覽[訂用帳戶頁面](https://azure.microsoft.com/try/cognitive-services/)來註冊。 在 [登入] 頁面上，使用您的 Microsoft 帳戶進行登入，如此就能夠訂閱並取得免費金鑰。 完成註冊程序之後，將金鑰貼到下面重新產生之 Notebook 的變數區段中。 不論是主要金鑰還是次要金鑰都可運作。 請務必以引號括住金鑰來讓它變成字串。

```python
# Variables

_url = 'https://westcentralus.api.cognitive.microsoft.com/vision/v1/analyses'
_key = None #Here you have to paste your primary key
_maxNumRetries = 10
```
