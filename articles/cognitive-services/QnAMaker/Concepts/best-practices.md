---
title: 最佳做法 - QnA Maker
titlesuffix: Azure Cognitive Services
description: 使用下列最佳做法可改善您的知識庫，並為您的應用程式/聊天機器人使用者提供更好的結果。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/24/2018
ms.author: tulasim
ms.openlocfilehash: cb171a666a4a54660a3bf54b8f26aed23f60d249
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036298"
---
# <a name="best-practices"></a>最佳作法
[知識庫開發生命週期](../Concepts/development-lifecycle-knowledge-base.md)可引導您了解如何從頭到尾管理知識庫。 使用下列最佳做法可改善您的知識庫，並為您的應用程式/聊天機器人使用者提供更好的結果。

## <a name="extraction"></a>擷取
QnA Maker 服務會持續改進從內容中擷取 QnA 的演算法，並擴充支援的檔案和 HTML 格式清單。 請遵循[指導方針](../Concepts/data-sources-supported.md)，以根據您的文件類型擷取資料。 

一般情況下，常見問題集頁面應獨立存在，而不會與其他資訊結合。 產品手冊應有清楚的標題，且最好有索引頁面。 

## <a name="chit-chat"></a>閒聊
將閒聊新增至您的 Bot，輕而易舉地讓 Bot 變得更健談且吸引人。 您可以在建立知識庫時，針對 3 個預先定義的特質輕鬆地新增閒聊，並隨時加以變更。 了解如何[將閒聊新增至您的 KB](../How-To/chit-chat-knowledge-base.md)。 

### <a name="choosing-a-personality"></a>選擇特質
有 3 個預先定義的特質可支援閒聊： 

|特質|
|--|
|專業人員|
|朋友|
|卡通人物|

回應的範圍從正式到非正式和不敬。 您應該選取最符合您希望 Bot 所用語調的特質。 您可以檢視資料集，並選擇其中一個資料集作為 Bot 的基礎，然後自訂回應。 

### <a name="edit-bot-specific-questions"></a>編輯 Bot 特有問題
有一些屬於閒聊資料集的 Bot 特有問題，而且已填入泛型回答。 變更下列回答，充分反映您的 Bot 詳細資料。 

建議讓下列閒聊 QnA 更加明確：

* 你是誰？
* 您該怎麼辦？
* 你幾歲？
* 誰建立了你？
* 您好
   

## <a name="rankingscoring"></a>排名/評分
請務必充分利用 QnA Maker 所支援的排名功能。 這樣將可提高指定的使用者查詢獲得適當回應的可能性。

### <a name="choosing-a-threshold"></a>選擇閾值
預設信賴分數的閾值為 50，不過您可以根據需求為您的知識庫變更該閾值。 每個知識庫各有不同，因此您應該測試並選擇最適合您知識庫的閾值。 深入了解[信賴分數](../Concepts/confidence-score.md)。 


### <a name="add-alternate-questions"></a>新增替代問題
[替代問題](../How-To/edit-knowledge-base.md)可提高使用者查詢產生相符項目的可能性。 在同樣的問題可用多種方式提問時，替代問題就有其效用。 這可以包括文句結構和字組樣式的變更。

|原始查詢|替代查詢|變更| 
|--|--|--|
|Is parking available?|Do you have car park?|文句結構|
 |Hi|Yo<br>Hey there!|字組樣式或俚語|

### <a name="use-metadata-filters"></a>使用中繼資料篩選條件
[中繼資料](../How-To/edit-knowledge-base.md)可增加根據篩選條件縮小使用者查詢結果範圍的能力。 即使查詢相同，知識庫解答也可能根據中繼資料標記而不同。 例如，如果餐廳分店的地點不同 (亦即，中繼資料為「地點：西雅圖」和「地點：雷德蒙」的不同)，「停車場在哪裡」就可能有不同的回答。

### <a name="use-synonyms"></a>使用同義字
同義字在英文中可獲得某種程度的支援，使用[文字變異形式](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fd)可為關鍵字新增採用不同格式的同義字。 同義字應新增於 QnA Maker 服務層級上，而由服務中所有的知識庫所共用。

|原始字組|同義字|
|--|--|
|buy|purchase<br>netbanking<br>net banking|

### <a name="use-distinct-words-to-differentiate-questions"></a>使用相異字詞區隔問題
會比對使用者查詢與知識庫中所含問題的 QnA Maker 比對和排名演算法，在以每個問題分別處理不同的需求時，可獲得最佳效果。 若在不同問題間有相同的字組重複出現，就比較難以用這些字組為指定的使用者查詢選擇正確的解答。 

例如，您對於下列問題可能會有兩個不同的 QnA：

|QnA|
|--|
|where is the parking *location*|
|where is the atm *location*|

因為這兩個 QnA 是以非常類似的字組表達，此相似性可能會對以 *"where is the `<x>` location"* 這類方式表達的許多使用者查詢，造成非常類似的分數。 相反地，試著避免使用在知識庫中出現於許多問題的字組 (像是 "location")，以 *"where is the parking lot"* 和 *"where is the atm"* 之類的查詢來清楚區分。 


## <a name="collaborate"></a>共同作業
QnA Maker 可讓使用者對知識庫進行[共同作業](../How-to/collaborate-knowledge-base.md)。 使用者需要有 Azure QnA Maker 資源群組的存取權，才能存取知識庫。 有些組織可能想要將知識庫的編輯和維護委外處理，但同時仍能夠保護其 Azure 資源的存取權。 若要完成此「編輯者-核准者」模型，可以在不同的訂用帳戶中設定兩個相同的 [QnA Maker 服務](../How-to/set-up-qnamaker-service-azure.md)，並選取其中一個用於編輯測試週期。 測試完成之後，就會使用[匯入-匯出](../Tutorials/migrate-knowledge-base.md)程序將知識庫內容轉移至核准者的 QnA Maker 服務，由他來執行最終的知識庫發佈和端點更新。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [編輯知識庫](../How-to/edit-knowledge-base.md)
