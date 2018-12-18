---
title: Python 快速入門 - 預測意圖 - LUIS
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您會了解如何使用 Python 來呼叫 LUIS 應用程式。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: e560aeffecf63f63966a49053e0f79d012b4a0a3
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038267"
---
# <a name="quickstart-get-intent-using-python"></a>快速入門：使用 Python 來取得意圖
在本快速入門中，會將將語句傳遞至 LUIS 端點，並取回意圖和實體。

[!include[Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-intro-para.md)]

## <a name="prerequisites"></a>必要條件

* [Python 3.6](https://www.python.org/downloads/) 或更新版本。
* [Visual Studio Code](https://code.visualstudio.com/)

[!include[Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-luis-repo-note.md)]

## <a name="get-luis-key"></a>取得 LUIS 金鑰

[!include[Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-get-key-para.md)]

## <a name="get-intent-with-browser"></a>使用瀏覽器取得意圖

[!include[Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-browser-para.md)]

## <a name="get-intent--programmatically"></a>以程式設計方式取得意圖

您可以使用 Python 來存取您在上一個步驟的瀏覽器視窗中看到的相同結果。

1. 將下列其中一個程式碼片段，複製到名為 `quickstart-call-endpoint.py` 的檔案：

   [!code-python[Console app code that calls a LUIS endpoint for Python 2.7](~/samples-luis/documentation-samples/quickstarts/analyze-text/python/2.x/quickstart-call-endpoint-2-7.py)]

   [!code-python[Console app code that calls a LUIS endpoint for Python 3.6](~/samples-luis/documentation-samples/quickstarts/analyze-text/python/3.x/quickstart-call-endpoint-3-6.py)]

2. 以您的 LUIS 端點金鑰，取代 `Ocp-Apim-Subscription-Key` 欄位的值。

3. 使用 `pip install requests` 安裝相依性。

4. 使用 `python ./quickstart-call-endpoint.py` 執行指令碼。 它會顯示您稍早在瀏覽器視窗中看到的相同 JSON。

## <a name="luis-keys"></a>LUIS 金鑰

[!include[Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-key-usage-para.md)]

## <a name="clean-up-resources"></a>清除資源
刪除 Python 檔案。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [新增表達方式](luis-get-started-python-add-utterance.md)