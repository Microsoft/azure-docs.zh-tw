---
title: 表情 API 常見問題集 | Microsoft Docs
description: 取得有關認知服務中表情 API 常見問題集的解答。
services: cognitive-services
author: anrothMSFT
manager: corncar
ms.service: cognitive-services
ms.component: emotion-api
ms.topic: article
ms.date: 01/26/2017
ms.author: anroth
ms.openlocfilehash: 8532d7c00fd8d7b01d84b5e55cb9bbc60241789c
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367971"
---
# <a name="emotion-api-frequently-asked-questions"></a>表情 API 常見問題集
 
> [!IMPORTANT]
> 影片 API 預覽將於 2017 年 10 月 30 日結束。 請試用新的[影片索引器 API 預覽](https://azure.microsoft.com/services/cognitive-services/video-indexer/)，以輕鬆地從影片中擷取見解，並增強內容探索體驗，像是偵測話語、臉部、人物及表情而得來的搜尋結果。 [深入了解](https://docs.microsoft.com/azure/cognitive-services/video-indexer/video-indexer-overview)。

### <a name="if-you-cant-find-answers-to-your-questions-in-this-faq-try-asking-the-emotion-api-community-on-stackoverflowhttpsstackoverflowcomquestionstaggedproject-oxfordormicrosoft-cognitive-or-contact-help-and-support-on-uservoicehttpscognitiveuservoicecom"></a>如果您在此常見問題集中找不到問題的解答，請嘗試在 [StackOverflow](https://stackoverflow.com/questions/tagged/project-oxford+or+microsoft-cognitive) 上詢問表情 API 社群，或連絡 [UserVoice 協助與支援](https://cognitive.uservoice.com/)。  

-----

**問題**：*哪些類型的影像可從表情 API 獲得最佳結果？*

**解答**：使用未遮擋、完全正面的臉部影像，可獲得最佳結果。 部分正面人臉部的可靠性會降低，而且在臉部旋轉 45 度或更多度的影像中，表情 API 可能無法辨識表情。

-----

**問題**：*表情 API 可識別多少表情？*

**解答**：表情 API 可辨識八個普遍接受的不同表情： 
* 快樂
* 憂傷
* 驚奇
* 憤怒
* 恐懼
* 蔑視
* 討厭 
* 中性 

-----

**問題**：*是否有任何方法可將即時影音串流傳遞至此 API 並同時取得結果？*

**解答**：使用以影像為基礎的表情 API，並且在每個畫面上呼叫它，或略過畫面以提升效能。  有視訊逐畫面分析範例可以使用。

-----

**問題**：我正在傳入二進位影像資料，但出現：「臉部影像無效」。*

**解答**：這表示演算法有剖析影像的問題。  
* 支援的輸入影像格式包括 JPEG、PNG、GIF (第一個畫面)、BMP。 
* 影像檔案大小不得大於 4MB
* 可偵測的臉部大小範圍是 36x36 到 4096x4096 像素。 無法偵測此範圍之外的臉部
* 某些臉部可能因技術挑戰而無法偵測，例如非常大的臉部角度 (頭部姿勢)、大型的阻擋物。 正面和接近正面的臉部能提供最佳結果

-----
