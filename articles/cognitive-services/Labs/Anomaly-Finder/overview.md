---
title: 什麼是異常搜尋工具？ - Microsoft 認知服務 | Microsoft Docs
description: 使用異常搜尋工具中的進階演算法，可協助您識別時間序列資料中的異常，並回傳 Microsoft 認知服務中的資訊。
services: cognitive-services
author: tonyxing
ms.service: cognitive-services
ms.technology: anomaly-detection
ms.topic: article
ms.date: 04/19/2018
ms.author: tonyxing
ms.openlocfilehash: 1080bb0ad1d901a8b9a5ace4993d4e0d46924a03
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369150"
---
# <a name="what-is-anomaly-finder"></a>什麼是異常搜尋工具？

異常搜尋工具可讓您持續監看資料，並利用機器學習來偵測異常，這種機器學習已經適應您獨特的資料，無論針對什麼產業、情況或資料量，都能套用正確的統計模型。 異常搜尋工具 API 使用時間序列做為輸入，回傳資料點是否異常，得出期望值，並決定資料視覺化之上下限。 異常搜尋工具為預先建置之 AI 服務，使用上只要知道如何用 RESTful API 即可，不需要其他機器學習之專業知識。 這可讓開發變得簡單而且多元化，因為它可搭配任何時間序列資料，而且也可以整合於串流處理系統。 異常搜尋工具涵蓋廣泛的使用情況，例如用於應對詐騙、偷竊、市場變化及潛在業務事故，或是監看 IoT 裝置流量同時維持匿名的財務工具。 此解決方案也可以成為一種付費服務，讓終端客戶了解資料、花費、投資報酬率或使用者活動的變化。
嘗試使用異常搜尋工具 API 來更深入了解您的資料。 

了解您可以使用此 API 建置什麼內容：

* 學習依據時間序列中的歷史資料預測期望值
* 判斷資料點是否為歷史資料模式中的異常點
* 產生區段，以視覺化方式呈現「正常」值的範圍

![Anomaly_Finder](./media/anomaly_detection1.png) 

圖 1：偵測銷售營收中的異常

![Anomaly_Finder](./media/anomaly_detection2.png)

圖 2：偵測服務要求中的模式變更

## <a name="requirements"></a>需求

- 輸入時間序列的資料下限：對於沒有明確週期性的時間序列至少 13 個資料點，對於已知週期性的時間序列至少 4 個循環的資料點。 
- 資料完整性：時間序列資料點會按照相同的間隔分開，而且不會有任何遺漏的資料點。 

## <a name="identify-anomalies"></a>識別異常

異常偵測 API 會回傳是否有任何指定資料點異常的結果，並提供下列其他資訊
* Period - API 用來偵測異常點的週期性。
* WarningText - 可能出現的警告資訊。
* ExpectedValue - 學習型模型的預測值
* IsAnomaly - 判斷資料點是否為異常的結果
* IsAnomaly_Neg - 資料點低於預期值 (下降)
* IsAnomaly_Pos - 資料點高於預期值 (尖峰)
* UpperMargin - ExpectedValue 和 UpperMargin 的總和決定資料點仍屬於正常範圍的上限
* LowerMargin - (ExpectedValue - LowerMargin) 決定資料點仍屬於正常範圍的下限

> [!Note]
> UpperMargin 和 LowerMargin 可用來產生實際時間序列周圍的區段，以視覺化方式呈現正常值的範圍。 

## <a name="adjusting-lower-and-upper-bounds-in-post-processing-on-the-response"></a>調整回應的文章處理之中的上下限

異常偵測 API 會回傳資料點是否為異常的預設結果，而且可以從 ExpectedValue 和 UpperMargin/LowerMargin 計算上下限。 在大多數的情況下，這些預設值應該是適用的。 不過，某些情況需要與預設界限不同的上下限。 建議套用係數於 UpperMargin 或 LowerMargin 以調整動態界線。

### <a name="examples-with-1152-as-coefficiency"></a>係數為 1/1.5/2 的範例

![預設敏感度](./media/sensitivity_1.png)

![1.5 敏感度](./media/sensitivity_1.5.png)

![2 敏感度](./media/sensitivity_2.png)

含有範例資料的要求

[!INCLUDE [Request](./includes/request.md)]

範例 JSON 回應

[!INCLUDE [Response](./includes/response.md)]
