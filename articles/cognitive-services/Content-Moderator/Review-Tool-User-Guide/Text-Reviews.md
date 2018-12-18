---
title: 在 Azure Content Moderator 中審查文字 | Microsoft Docs
description: 了解如何在 Content Moderator 中審查文字，以查看其分數和偵測到的標記。 請使用此資訊來判斷內容適當與否。
services: cognitive-services
author: sanjeev3
manager: mikemcca
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: article
ms.date: 02/03/2017
ms.author: sajagtap
ms.openlocfilehash: cb8774395b7374677e8de3b80630a2d4abf490f9
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367931"
---
# <a name="review-text"></a>檢閱文字

您可以透過 Azure Content Moderator，使用分數和偵測到的標記來審查文字。 請使用此資訊來判斷內容適當與否。 

## <a name="select-or-enter-the-text-to-review"></a>選取或輸入要審查的文字

在 Content Moderator 中，選取 [嘗試] 索引標籤。然後，選取 [文字] 選項即可移至文字審核開始畫面。 輸入任何文字，或針對自動化文字審核提交預設範例文字。 您最多可以輸入 1,024 個字元。

## <a name="get-ready-to-review-results"></a>準備審查結果

審查工具會先呼叫文字審核 API。 然後，它會使用偵測到的標記，產生文字審查。 審查工具會比對分數結果，引起您的小組關注。

## <a name="review-text-results"></a>審查文字結果

詳細的結果會出現在視窗中。 結果包含偵測到的標記以及文字審核 API 所傳回的字詞。 若要切換標記的選取狀態，請選取標記。 您也可使用您已建立的任何自訂標記。

![審查文字結果](images/3-review-text-2.png)
