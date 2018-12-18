---
title: 利用 OCR 擷取文字 - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 使用電腦視覺 API 以光學字元辨識 (OCR) 擷取文字的相關概念。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: conceptual
ms.date: 08/29/2018
ms.author: pafarley
ms.openlocfilehash: 52b6265722d5cfbf8baf54e1785ace627aa0892f
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341662"
---
# <a name="extracting-text-with-ocr"></a>利用 OCR 擷取文字

電腦視覺中的光學字元辨識 (OCR) 技術可偵測影像中的文字內容，並將識別出來的文字詞擷取到電腦可讀取的字元資料流中。 您可以將其結果用於搜尋和其他多種用途上，像是醫療記錄、保全和銀行業務等。 它會自動偵測語言。 OCR 讓使用者只需拍攝文字相片而不必謄寫文字，省時又方便。

OCR 支援 25 種語言。 這些語言是：阿拉伯文、簡體中文、繁體中文、捷克文、丹麥文、荷蘭文、英文、芬蘭文、法文、德文、希臘文、匈牙利文、義大利文、日文、韓文、挪威文、波蘭文、葡萄牙文、羅馬尼亞文、俄文、塞爾維亞文 (斯拉夫文和英文)、斯洛伐克文、西班牙文、瑞典文和土耳其文。

如有需要，OCR 會沿著水平影像軸以度為單位校正已辨識文字的旋轉角度。 OCR 會提供每個字的框架座標，如下圖所示。

![OCR 概觀](./Images/vision-overview-ocr.png)

## <a name="ocr-requirements"></a>OCR 需求

電腦視覺可以使用 OCR 從符合下列需求的影像擷取文字：

* 必須以 JPEG、PNG、GIF 或 BMP 格式呈現的影像
* 輸入影像的大小必須介於 50 x 50 與 4200 x 4200 像素之間


輸入影像可以旋轉 90 度的任何倍數，並可輔以不超過 40 度的小角度旋轉。

## <a name="improving-ocr-accuracy"></a>改善 OCR 正確性

文字辨識的準確性取決於影像品質。 在下列情況下可能會導致不正確的讀取：

* 影像模糊。
* 手寫或草寫的文字。
* 藝術字型樣式。
* 文字太小。
* 複雜的背景、陰影、文字反光或透視失真。
* 文字開頭處的大寫字母過大或遺漏
* 文字加上了下標、上標或刪除線。

### <a name="ocr-limitations"></a>OCR 限制

在主要由文字構成的相片上，部分辨識的文字可能會造成誤判。 在某些相片 (特別是沒有任何文字的相片) 上，精確度可能會隨著影像的類型而有極大差異。

## <a name="next-steps"></a>後續步驟

深入了解[辨識印刷和手寫文字](concept-recognizing-text.md)的概念。
