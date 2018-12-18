---
title: 教學課程：辨識影像中的臉部表情 - 表情 API，Python
titlesuffix: Azure Cognitive Services
description: 使用 Jupyter Notebook 來了解如何搭配 Python 使用表情 API。 使用常用的程式庫以視覺方式呈現您的結果。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: emotion-api
ms.topic: tutorial
ms.date: 05/23/2017
ms.author: anroth
ROBOTS: NOINDEX
ms.openlocfilehash: 31e346cd9a3f43f8181ebee4474ae6c9ee2cc6fc
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48237852"
---
# <a name="tutorial-use-the-emotion-api-with-a-jupyter-notebook--python"></a>教學課程：搭配 Jupyter Notebook 和 Python 使用表情 API。

> [!IMPORTANT]
> 表情 API 將於 2019 年 2 月 15 日淘汰。 表情辨識功能現已公開推出，是[臉部 API](https://docs.microsoft.com/azure/cognitive-services/face/) 的一部分。 

為了讓您能輕鬆開始使用表情 API，以下連結的 Jupyter Notebook 會示範如何在 Python 中使用該 API，以及如何使用一些常用的程式庫以視覺方式呈現您的結果。

[GitHub 中筆記本的連結](https://github.com/Microsoft/Cognitive-Emotion-Python/blob/master/Jupyter%20Notebook/Emotion%20Analysis%20Example.ipynb) \(英文\)

### <a name="using-the-jupyter-notebook"></a>使用 Jupyter Notebook

若要以互動方式使用該筆記本，您必須複製它，然後在 Jupyter 中執行。 若要了解如何開始使用互動式 Jupyter Notebook，請遵循以下網址中的指示： http://jupyter.readthedocs.org/en/latest/install.html \(英文\)。

若要使用這個筆記本，您需要有表情 API 的訂用帳戶金鑰。 請造訪[訂用帳戶頁面](https://azure.microsoft.com/try/cognitive-services/)來註冊。 在 [登入] 頁面上，請使用您的 Microsoft 帳戶登入，便能訂閱並取得免費的金鑰。 完成註冊程序之後，請將金鑰貼至以下所示的變數區段中。 不論是主要金鑰還是次要金鑰都可運作。

```
Python Example

#Variables

_url = 'https://westus.api.cognitive.microsoft.com/emotion/v1.0/recognize'
_key = None #Here you have to paste your primary key
_maxNumRetries = 10

```
