---
title: 電腦視覺快速入門摘要
titleSuffix: Azure Cognitive Services
description: 在這些快速入門中，您會使用電腦視覺 API，來分析影像、建立縮圖，以及擷取印刷文字和手寫文字。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: e44eb52323a93bddd629f3591cdbfbf021768629
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344082"
---
# <a name="quickstart-summary"></a>快速入門：摘要

這些快速入門提供了資訊和程式碼範例，可協助您快速開始使用「電腦視覺」服務。

範例會對電腦視覺 API 進行直接 HTTP 呼叫。 請參閱 *SDK 快速入門*一節，以取得使用電腦視覺用戶端程式庫的範例，其中提供了包裝 HTTP 呼叫的便利方法。

若要快速地試驗「電腦視覺 API」，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console) (英文)。

您可以使用「電腦視覺」服務來完成下列工作：

* 分析遠端影像
* 分析本機影像
* 偵測名人和地標 (領域模型)
* 以智慧方式產生縮圖
* 從影像偵測及擷取印刷文字 (OCR)
* 從影像偵測及擷取手寫的文字

每個範例中的程式碼都很類似。 不過，它們各自著重在不同的「電腦視覺」功能，以及與服務交換資料的不同技術，例如：

* _產生縮圖_會在回應主體中，以位元組陣列的形式傳回影像。
* _分析本機影像_會要求影像以位元組陣列的形式，包含在要求中。
* _擷取手寫文字_會要求兩個呼叫以擷取文字。

## <a name="summary"></a>總結

| 快速入門               | 要求參數                          | Response          |
| ------------------------ | ------------------------------------------- | ----------------  |
| 分析遠端影像   | visualFeatures=Categories,Description,Color | JSON 字串       |
| 分析本機影像    | data=image_data (位元組陣列)                | JSON 字串       |
| 偵測名人       | model=celebrities                           | JSON 字串       |
| 產生縮圖     | width=200&height=150&smartCropping=true     | 位元組陣列        |
| 擷取列印的文字     | language=unk&detectOrientation=true         | JSON 字串       |
| 擷取手寫的文字 | handwriting=true                            | URL，JSON 字串  |

## <a name="next-steps"></a>後續步驟

探索用來分析影像、偵測名人和地標、建立縮圖及擷取印刷和手寫文字的「電腦視覺 API」。

> [!div class="nextstepaction"]
> [探索電腦視覺 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44) (英文)
