---
title: 臉部 API 服務常見問題集 | Microsoft Docs
description: 以下是有關臉部 API 服務最熱門問題的解答。
services: cognitive-services
author: SteveMSFT
manager: corncar
ms.service: cognitive-services
ms.component: face-api
ms.topic: article
ms.date: 01/26/2017
ms.author: sbowles
ms.openlocfilehash: da2f75deef8a8beea3ba23b6a39eb6d2fe104b54
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367959"
---
# <a name="face-api-frequently-asked-questions"></a>臉部 API 常見問題集
### <a name="if-you-cant-find-answers-to-your-questions-in-this-faq-try-asking-the-face-api-community-on-stackoverflowhttpsstackoverflowcomquestionstaggedproject-oxfordormicrosoft-cognitive-or-contact-help-and-support-on-uservoicehttpscognitiveuservoicecom"></a>如果您在此常見問題集中找不到問題的解答，請嘗試在 [StackOverflow](https://stackoverflow.com/questions/tagged/project-oxford+or+microsoft-cognitive) 上詢問臉部 API 社群，或連絡 [UserVoice 協助與支援](https://cognitive.uservoice.com/)。

-----
**問題**：哪些因素可能降低臉部 API 在辨識、驗證或尋找相似方面的精確度？

**解答**：通常與人類難以識別某人的情況相同，包括：
* 阻擋一或兩個眼睛的障礙物
* 刺眼的光線，例如嚴重背光
* 改變髮型或臉毛
* 因年紀而改變
* 極端的臉部表情 (例如尖叫)

在這類具挑戰性的情況下，臉部 API 通常很成功，但精確度可能會降低。 若要讓辨識功能梗強固並解決這些挑戰，請使用包含各種角度和光線的相片訓練您的人員。

-----
**問題**：我正在傳入二進位影像資料，但收到「臉部影像無效」錯誤。

**解答**：這表示演算法有剖析影像的問題。 原因包括：
* 支援的輸入影像格式包括 JPEG、PNG、GIF (第一個畫面)、BMP。
* 影像檔案大小不得大於 4MB
* 可偵測的臉部大小範圍是 36x36 到 4096x4096 像素。 無法偵測此範圍之外的臉部
* 某些臉部可能因技術挑戰而無法偵測，例如非常大的臉部角度 (頭部姿勢)、大型的阻擋物。 正面和接近正面的臉部能提供最佳結果

-----
