---
title: 了解 LUIS 所傳回的預測分數
titleSuffix: Azure Cognitive Services
description: 了解 LUIS 中的預測分數意義
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: dc19151e82f44159533050e60ba2f1c6d65466fe
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032218"
---
# <a name="prediction-score"></a>預測分數
預測分數表示 LUIS 對預測結果的信賴程度。 

預測分數通常介於零 (0) 到一 (1) 之間。 高信賴度 LUIS 分數的範例是 0.99。 低信賴度的範例是 0.01。 

|分數值|信賴度|
|--|--|
|1|明確相符|
|0.99|高信賴度|
|0.01|低信賴度|
|0|明確不相符|

當語句產生低信賴度分數時，LUIS 會在 [LUIS](luis-reference-regions.md) 網站的 [意圖] 頁面上，將所識別的**已標示的意圖**以紅色框線標示來醒目提示。 

![分數差異](./media/luis-concept-score/score-discrepancy.png)

## <a name="top-scoring-intent"></a>最高分的意圖
每個語句預測都會傳回一個最高分的意圖。 這是預測分數的數值比較。 最高的兩個分數之間可能差異很小。 LUIS 並不會指出此微小差異，而只會傳回分數。  

如果您在意最高分數的微小差異，則應該傳回所有意圖的分數。 您可以將以單字選擇和排列來表示其差異的語句新增至該兩個意圖，或是讓 LUIS 呼叫應用程式 (例如 Chatbot) 透過程式設計方式選擇如何處理這兩個最高分意圖。 

兩個分數過於接近的意圖可能會因為非決定性訓練而反轉。 第一高分可能會變成第二高分，而第二高分可能會變成第一高分。 若要避免此問題，可一一對該語句的兩個高分意圖新增範例語句，而其中需具有可區分這兩個意圖的文字選擇和內容。 兩個意圖應具有相同數目的範例語句。 若要避免因為訓練而發生反轉，區隔語句的經驗法則是分數上要有 15% 的差異。

## <a name="return-prediction-score-for-all-intents"></a>傳回所有意圖的預測分數
測試或端點結果可以包含所有意圖。 若要進行此設定，請在[端點](https://aka.ms/v1-endpoint-api-docs)上以 `verbose=true` 查詢字串名稱/值組來設定。 

## <a name="review-intents-with-similar-scores"></a>檢閱具有相似分數的意圖
檢閱所有意圖的分數是一個很好的方式，不僅可確認所識別的意圖正確，也可確認下一個所識別意圖的分數對語句而言明顯一致較低。 

如果有多個意圖的預測分數相近，則 LUIS 可能會根據語句的內容，在意圖之間做切換。 若要修正此問題，請繼續將具有更廣泛內容差異的語句新增至每個意圖。   

## <a name="e-exponent-notation"></a>E (指數) 標記法

預測分數可以使用指數標記法，「顯示成」高於 0-1 的範圍，例如 `9.910309E-07`。 此分數表示一個非常**小**的數字。

|E 標記法分數 |實際分數|
|--|--|
|9.910309E-07|.0000009910309|

## <a name="differences-with-predictions"></a>預測上的差異
當您在不同應用程式中為相同的模型進行定型，而分數卻不再相同時，這是因為定型中有隨機性元素。 其次，當語句與多個意圖有任何重疊時，即意謂著相同語句的最高分意圖會根據定型變更。

如果您的 Chatbot 需要有特定的 LUIS 分數，才能指出對某個意圖的信賴度，您就應該改為使用兩個最高分意圖之間的分數差異。 這樣可以在定型中提供變化的彈性。 

## <a name="punctuation"></a>標點符號
標點符號在 LUIS 中是個別的語彙基元。 結尾包含句點的語句與未包含句點的語句是兩個不同的語句，而且可能會收到兩個不同的預測。 請確定模型會在[範例語句](luis-concept-utterance.md) (含標點符號和不含標點符號) 或在更容易使用特殊語法來忽略標點符號的 [patterns}(luis-concept-patterns.md) 中處理標點符號：`I am applying for the {Job} position[.]`

## <a name="next-steps"></a>後續步驟

請參閱[新增實體](luis-how-to-add-entities.md)，以深入了解如何將實體新增至 LUIS 應用程式。