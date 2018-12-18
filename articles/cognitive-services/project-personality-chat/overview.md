---
title: 什麼是專案特質交談？
titlesuffix: Azure Cognitive Services
description: 本文將概述 Azure 專案個人化聊天，這是可加強您 Bot 對話能力的雲端式 API。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: personality-chat
ms.topic: overview
ms.date: 05/07/2018
ms.author: nolachar
comment: As a bot developer, I want my bot to be able to handle small talk in a consistent tone so that my bot appears more complete and conversational.
ms.openlocfilehash: 865815862be4a2250347f782985e0bce87780197
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220529"
---
# <a name="what-is-project-personality-chat"></a>什麼是專案特質交談？

專案個人化聊天會藉由處理符合獨特選定特性的小型交談，來增強您 Bot 的對話能力。 個人化聊天會使用意圖分類器來識別常見的小型交談意圖，並產生與特性一致的回應。 根據意圖和信賴分數，Bot 就可以從已策劃的編輯基底中選擇最佳回應，或是使用深度類神經網路 (DNN) 來即時產生回應。 您可以選擇三個預設角色。 角色模型會傳回與所選特性最類似的回應。

您可以針對小型交談查詢設定自訂編輯。 這可以使用 Microsoft Bot Framework SDK 輕鬆地進行整合。 此 SDK 可節省您從頭撰寫查詢的時間和成本。

## <a name="getting-started-with-project-personality-chat"></a>開始使用專案個人化聊天

您可以瀏覽專案個人化聊天的實驗頁面，並與可用的示範項目交談，以及要求優先存取可用服務。
現在，您也可以透過 Microsoft Bot Framework SDK 將僅限自訂編輯的程式庫整合至您的 Bot 中。 <br>
[範例：在 Bot 中整合個人化聊天](https://github.com/Microsoft/BotBuilder-PersonalityChat/) <br>
[嘗試個人化聊天程式庫](https://github.com/Microsoft/BotBuilder-PersonalityChat/tree/master/CSharp)

## <a name="generating-responses-using-neural-networks"></a>使用類神經網路產生回應

個人化聊天會使用深度學習模型來學習一般對話模式，並產生回應。 回應產生模型是遞迴式類神經網路 (RNN)。 此模型在數以百萬計的對話中進行訓練，藉此了解及學習人類互動模式。 藉由學習到的語句結構模式，可形成新的查詢並產生回應。 產生這個新的回應後，模型會搜尋字組的「撰寫詞彙」，並挑選回應中第一個字組的最佳 N 個結果。 因為使用定向搜索，模型會針對每個所產生的第一個字組，繼續搜尋第二個字組的最佳 N 個結果。 直到模型挑選到「句子結束」(EOS) 的文字後，回應就會視為完成。 當模型有了所有回應後，它會根據完整回應的機率分數選擇前 N 個最佳回應。

模型會學習產生符合上下文且有正確「結構」的適當結果。 例如，有個問題類似「您現在要交談嗎？」 我們來多談一下合理的回覆結構：此回覆可能會以一些「是」或「否」和代名詞「我」之類的詮釋開頭。 「否」的回應比較容易包含禮貌性的陳述，以此類推。

系統會嘗試學習對話基礎結構的語言模型。 此結構應允許回應可部分受到外部條件 (例如主題、特性等) 的影響。  由於這些條件須從廣泛的模式中學習，因此適用於 (但不限於) 進行小型交談的應用程式。

![產生回應的序列到序列模型](./media/overview/sequence-to-sequence-model.png)

## <a name="personality-modeling"></a>特性模型化

 在任何資料導向的交談模型上，提供一致的「特性」一直是個挑戰。 「角色」的定義為在交談互動期間所經歷的特性。 角色可視為身分識別、語言行為和互動風格等元素的結合。 在個人化聊天的目前版本中，語言行為和互動風格是最重要的。

此模型會將每個個別的說話者表示為一個向量或內嵌項目。 它會對來自說話者的資訊進行編碼，而這些資訊會影響回應的內容和風格。 接著，此模型會根據不同說話者提供的交談內容，來學習說話者的表述。 做出類似回應的說話者會傾向有類似的內嵌項目，並位於向量空間中附近的位置。 如此一來，若訓練資料來自向量空間中位於附近的說話者，則此資料有助提升說話者模型的一般化能力。 模型會有效地把有類似表述的說話者聚集在向量空間中，以構成有「角色識別碼」的角色叢集。

針對預設角色，會使用屬性和已策劃的回應來尋找最相符的說話者叢集。 然後，此叢集會作為每個預設特性的角色識別碼。 只要採用一組 Bot/品牌回應，任何類型的特性都可進行持續自訂。 然後針對對話，精確地將個別的角色模擬成語言回應行為和其他主要特性等。

![使用說話者叢集建立角色模型](./media/overview/persona-modeling.png)

## <a name="small-talk-intent-understanding"></a>了解小型交談的意圖

個人化聊天具有意圖分類器，可確保回應符合小型交談的意圖。 這些分類器不會妨礙工作或資訊查詢。 高階交談分類器會判斷查詢屬於小型交談或閒聊意圖。 這會使用以語彙和語意為基礎的分類器並結合分數來決定。 這些意圖的訓練會用到交談式資料 (作為正面意圖範例) 和搜尋/工作查詢 (針對負面意圖範例)。

其他次要意圖的分類器會用來識別小型交談的子類別，以限制服務回應的小型交談類型，例如：問候語、祝賀、意見等等。 這些使用「交談深入結構化語意模型 (Convolutional Deep Structure Semantic Model，CDSSM)」的深入學習分類器會將所有查詢轉換為向量。 這些分類器會使用數萬個對子意圖執行的已策劃查詢來進行訓練。 然後分類器會透過尋找向量空間中最符合的項目，將查詢與已識別的現有意圖類別進行比對。

許多控制項都已在定位，協助防止不適合的回應，並根據 Zo 這類智慧型代理程式的知識建置。 專案特質交談預設為只回應可辨識的使用者意圖。 建議您測試專案特質交談是否適合您的情況。 若您看到任何需要進一步訓練的項目，則十分歡迎提供您的意見反應。
