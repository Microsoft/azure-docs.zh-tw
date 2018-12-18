---
title: 使用 LUIS 建置應用程式的最佳做法 - Language Understanding
titleSuffix: Azure Cognitive Services
description: 了解 LUIS 最佳做法以獲得最佳結果。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 511e6c732613cc577644365e38b271135659f2d3
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042239"
---
# <a name="best-practices-for-building-a-language-understanding-app-with-cognitive-services"></a>使用認知服務建置語言理解應用程式的最佳做法
使用應用程式撰寫程序來建置您的 LUIS 應用程式。 

* 建置語言模型
* 新增一些定型範例語句 (每一意圖 10-15 個)
* 發佈 
* 從端點測試 
* 新增功能

[發佈](luis-how-to-publish-app.md)您的應用程式之後，請使用新增功能、發佈及從端點測試的撰寫循環。 請勿以新增更多範例語句的方式來開始下一個撰寫循環。 這並不會讓 LUIS 藉由真實世界使用者語句學習您的模型。 

為了讓 LUIS 能有效率地進行學習，請勿擴充語句，直到目前這組範例和端點語句傳回確信的高預測分數為止。 請使用主動式學習、[模式](luis-concept-patterns.md)及[片語清單](luis-concept-feature.md)來提升分數。 

## <a name="do-and-dont"></a>建議事項和避免事項
以下清單包含 LUIS 應用程式的最佳做法：

|建議事項|避免事項|
|--|--|
|[定義不同的意圖](#do-define-distinct-intents) |[將許多範例語句新增至意圖](#dont-add-many-example-utterances-to-intents) |
|[找出每個意圖的太攏統與太特定之間的最佳點](#do-find-sweet-spot-for-intents)|[使用 LUIS 作為定型平台](#dont-use-luis-as-a-training-platform)|
|[以反覆方式建置您的應用程式](#do-build-the-app-iteratively)|[新增許多相同格式的範例語句而忽略其他格式](#dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats)|
|[在稍後的反覆項目中新增片語清單和模式](#do-add-phrase-lists-and-patterns-in-later-iterations)|[混合意圖和實體的定義](#dont-mix-the-definition-of-intents-and-entities)|
|[將範例語句新增至 None 意圖](#do-add-example-utterances-to-none-intent)|[建立含有所有可能值的片語清單](#dont-create-phrase-lists-with-all-the-possible-values)|
|[利用主動式學習的建議功能](#do-leverage-the-suggest-feature-for-active-learning)|[新增許多模式](#dont-add-many-patterns)|
|[監視您應用程式的效能](#do-monitor-the-performance-of-your-app)|[新增每一個範例語句都進行定型和發佈](#dont-train-and-publish-with-every-single-example-utterance)|

## <a name="do-define-distinct-intents"></a>請務必定義不同的意圖
請確定每個意圖的詞彙都僅適用於該意圖，而未與不同的意圖重疊。 例如，如果您想要有一個處理旅遊安排 (例如航班和飯店) 的應用程式，則可以選擇讓這些安排成為語句內具有特定資料實體的個別意圖或相同意圖。

如果兩個意圖之間的詞彙相同，請將意圖結合，然後使用實體。 

請思考一下以下的範例語句：

```
Book a flight
Book a hotel
```

"Book a flight" 和 "Book a hotel" 使用相同的 "book a "詞彙。 這會產生重疊，因此它應該是相同意圖，但具有擷取不同航班和飯店單字的實體。 

## <a name="do-find-sweet-spot-for-intents"></a>請務必找出意圖的最佳點
請使用來自 LUIS 的預測資料來判斷您的意圖是否重疊。 重疊的意圖會混淆 LUIS。 結果會造成最高分的意圖太接近另一個意圖。 由於 LUIS 不會每次都使用完全相同的資料路徑來進行定型，因此重疊的意圖有可能在定型中成為第一或第二。 您會希望每個意圖的語句分數都儘量拉開，以便避免發生此情況。 良好的意圖區別應該每次都產生預期的最高分意圖。 
 
## <a name="do-build-the-app-iteratively"></a>請務必以反覆方式建置應用程式
保留一個未用來作為[範例語句](luis-concept-utterance.md)或端點語句的個別「盲性」測試集。 持續針對測試集改善應用程式。 調適測試集以反映真實的使用者語句。 使用此盲性測試集來評估每個反覆項目。 

開發人員應該有三組資料。 第一組是用來建置模型的範例語句。 第二組用來在端點測試模型。 第三組是在[批次測試](luis-how-to-batch-test.md)中使用的盲性測試資料。 這最後一組既不用來進行應用程式定型，也不會在端點上傳送。  

## <a name="do-add-phrase-lists-and-patterns-in-later-iterations"></a>請務必在稍後的反覆項目中新增片語清單和模式
[片語清單](luis-concept-feature.md)可讓您定義與應用程式定義域相關的單字字典。 請在您的片語清單中植入一些單字，然後使用建議功能，讓 LUIS 知道詞彙中的更多單字。 請勿將每個單字都新增到詞彙中，因為片語清單並非完全相符項目。 

來自端點的真實使用者語句彼此非常相似，可能顯露單字選擇和位置的模式。 [模式](luis-concept-patterns.md)功能會搭配規則運算式採用此單字選擇和位置，以改善預測準確性。 模式中的規則運算式可考量您打算忽略的單字和標點符號，同時仍可比對模式。 

請針對標點符號使用模式的選擇性語法，以便忽略標點符號。

在您的應用程式收到端點要求之前，請勿套用這些做法，因為那樣會扭曲信賴度。  

## <a name="do-add-example-utterances-to-none-intent"></a>請務必將範例語句新增至 None 意圖
這是後援意圖，應用程式外的所有項目都以此表示。 請在您 LUIS 應用程式的其餘部分，每 10 個範例語句中，便將一個範例語句新增至 None 意圖。

## <a name="do-leverage-the-suggest-feature-for-active-learning"></a>請務必利用主動式學習的建議功能
請定期使用[主動式學習](luis-how-to-review-endoint-utt.md)的**檢閱端點語句**，而不要將更多範例語句新增至意圖。 由於應用程式會不斷接收端點語句，因此這份清單會不斷成長並變更。

## <a name="do-monitor-the-performance-of-your-app"></a>請務必監視您應用程式的效能
請使用測試集來監視預測準確性。 

## <a name="dont-add-many-example-utterances-to-intents"></a>請勿將許多範例語句新增至意圖
發佈應用程式之後，請只從反覆程序中的主動式學習新增語句。 如果語句太類似，則請新增模式。 

## <a name="dont-use-luis-as-a-training-platform"></a>請勿使用 LUIS 作為定型平台
LUIS 是語言模型定義域特定的。 並非用來作為一般定型平台。 

## <a name="dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats"></a>請勿新增許多相同格式的範例語句而忽略其他格式
LUIS 會預期意圖的語句中有所變化。 語句可以在改變的同時仍保有相同的整體意義。 變化可以包括語句長度、單字選擇，以及單字位置。 

|請勿使用相同的格式|請務必使用變化格式|
|--|--|
|購買一張到西雅圖的機票<br>購買一張到巴黎的機票<br>購買一張到奧蘭多的機票|購買 1 張到西雅圖的機票<br>預約兩個下週一到巴黎的紅眼航班機位<br>我想要預訂 3 張到奧蘭多的春假機票|

第二欄使用不同的動詞 (購買、預約、預訂)、不同的量詞 (1、兩、3) 及不同的單字排列，但全都具有購買旅遊機票的相同意圖。 

## <a name="dont-mix-the-definition-of-intents-and-entities"></a>請勿混合意圖和實體的定義
請針對您 Bot 將執行的任何動作建立意圖。 使用實體作為促成該動作的參數。 

針對將預訂航班的 Chatbot，請建立 **BookFlight** 意圖。 請勿為每個航空公司或每個目的地都建立意圖。 請使用這些資料片段作為[實體](luis-concept-entity-types.md)，然後在範例語句中標示它們。 

## <a name="dont-create-phrase-lists-with-all-the-possible-values"></a>請勿建立含有所有可能值的片語清單
請在[片語清單](luis-concept-feature.md)中提供一些範例，但不要提供每個單字。 LUIS 會將內容一般化並納入考量。 

## <a name="dont-add-many-patterns"></a>請勿新增許多模式
請勿新增太多[模式](luis-concept-patterns.md)。 LUIS 旨在透過更少的範例快速學習。 請勿不必要地讓系統多載。

## <a name="dont-train-and-publish-with-every-single-example-utterance"></a>請勿對每一個範例語句都進行定型和發佈
請新增 10 或 15 個語句之後，再進行定型和發佈。 這將可讓您了解對預測準確性的影響。 新增單一語句對分數可能不會有明顯的影響。 

## <a name="next-steps"></a>後續步驟

* 了解如何在 LUIS 應用程式中[規劃您的應用程式](luis-how-plan-your-app.md)。