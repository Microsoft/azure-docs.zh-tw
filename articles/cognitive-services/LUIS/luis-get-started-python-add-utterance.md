---
title: Python 快速入門 - 變更模型及訓練 LUIS 應用程式
titleSuffix: Azure Cognitive Services
description: 在此 Python 快速入門中，會將範例語句新增至「家庭自動化」應用程式，並且訓練應用程式。 範例語句是對應到意圖的交談使用者文字。 您可以藉由提供與意圖相關的範例語句，教導 LUIS 哪些種類的使用者提供文字屬於哪種意圖。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: ff076954d51605935bb41488ec6a757c2f7806a1
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040171"
---
# <a name="quickstart-change-model-using-python"></a>快速入門：使用 Python 來變更模型

[!INCLUDE [Quickstart introduction for change model](../../../includes/cognitive-services-luis-qs-endpoint-intro-para.md)]

## <a name="prerequisites"></a>必要條件

[!INCLUDE [Quickstart prerequisites for changing model](../../../includes/cognitive-services-luis-qs-change-model-prereq.md)]
* [Python 3.6](https://www.python.org/downloads/) 或更新版本。
* [Visual Studio Code](https://code.visualstudio.com/)

[!INCLUDE [Code is available in LUIS-Samples Github repo](../../../includes/cognitive-services-luis-qs-change-model-luis-repo-note.md)]

## <a name="example-utterances-json-file"></a>範例語句 JSON 檔案

[!INCLUDE [Quickstart explanation of example utterance JSON file](../../../includes/cognitive-services-luis-qs-change-model-json-ex-utt.md)]

## <a name="create-quickstart-code"></a>建立快速入門程式碼

1. 將下列程式碼片段複製到名為 `add-utterances-3-6.py` 的檔案：

   [!code-python[Console app code that adds an utterance Python 3.6](~/samples-luis/documentation-samples/quickstarts/change-model/python/3.x/add-utterances-3-6.py)]

## <a name="run-code"></a>執行程式碼
使用 Python 3.6 從命令列執行應用程式。

### <a name="add-utterances-from-the-command-line"></a>從命令列新增語句

不使用任何引數來呼叫 add-utterance，會將語句新增至此應用程式，但不會加以訓練。

```CMD
> python add-utterances-3-6.py
```

當語句新增至模型時，就會傳回下列回應。  

```json
    "response": [
        {
            "value": {
                "UtteranceText": "go to seattle",
                "ExampleId": -5123383
            },
            "hasError": false
        },
        {
            "value": {
                "UtteranceText": "book a flight",
                "ExampleId": -169157
            },
            "hasError": false
        }
    ]
```

以下顯示成功訓練要求的結果：

```json
{
    "request": null,
    "response": {
        "statusId": 9,
        "status": "Queued"
    }
}
```

```JSON
Requested training status.
[
   {
      "modelId": "eb2f117c-e10a-463e-90ea-1a0176660acc",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "c1bdfbfc-e110-402e-b0cc-2af4112289fb",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "863023ec-2c96-4d68-9c44-34c1cbde8bc9",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "82702162-73ba-4ae9-a6f6-517b5244c555",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "37121f4c-4853-467f-a9f3-6dfc8cad2763",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "de421482-753e-42f5-a765-ad0a60f50d69",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "80f58a45-86f2-4e18-be3d-b60a2c88312e",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "c9eb9772-3b18-4d5f-a1e6-e0c31f91b390",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "2afec2ff-7c01-4423-bb0e-e5f6935afae8",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   },
   {
      "modelId": "95a81c87-0d7b-4251-8e07-f28d180886a1",
      "details": {
         "statusId": 0,
         "status": "Success",
         "exampleCount": 33,
         "trainingDateTime": "2017-11-20T18:09:11Z"
      }
   }
]
```

## <a name="clean-up-resources"></a>清除資源
當您完成快速入門時，請移除在本快速入門中建立的所有檔案。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [以程式設計方式建置 LUIS 應用程式](luis-tutorial-node-import-utterances-csv.md)