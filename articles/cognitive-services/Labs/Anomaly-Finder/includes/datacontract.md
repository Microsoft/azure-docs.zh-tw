---
title: 包含檔案
description: 包含檔案
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.component: anomaly-finder
ms.topic: include
ms.date: 04/13/2018
ms.author: chliang
ms.custom: include file
ms.openlocfilehash: 9280790f6692096a0b3909c9d1dfab2e94a8c0d7
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48904485"
---
使用[異常搜尋 API](https://labs.cognitive.microsoft.com/en-us/project-anomaly-finder)，您可以將 JSON 格式的時間序列資料上傳至 API 端點，然後讀取 API 回應的結果。 您可以上傳時間序列資料，每個資料點包括：  
* Timestamp - 資料點的時間戳記。 確定它使用 UTC 日期時間字串，例如 "2017-08-01T00:00:00Z"
* Value - 該資料點的量值

結果包括：
* Period - API 用來偵測異常點的週期性
* WarningText - 可能出現的警告資訊
* ExpectedValue - 學習型模型的預測值
* IsAnomaly - 資料點是否比預測值要高或低，為異常的結果 (尖峰或下降)
* IsAnomaly_Neg - 資料點低於預測值 (下降)
* IsAnomaly_Pos - 資料點高於預測值 (尖峰)
* UpperMargin - ExpectedValue 和 UpperMargin 的總和決定資料點仍屬於正常範圍的上限
* LowerMargin - (ExpectedValue - LowerMargin) 決定資料點仍屬於正常範圍的下限

您可以在[這裡](../apiref.md)找到資料合約的詳細資料。

