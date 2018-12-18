---
title: Azure 認知服務中的 LUIS 應用程式特性
titleSuffix: Azure Cognitive Services
description: 將特性新增至語言模型，以針對如何辨識您想要標示或分類的輸入，提供相關提示。 特性可協助 LUIS 辨識意圖和實體。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 015679b6020e9d2a4d702f9d6e723ecd9499d8dc
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47034462"
---
# <a name="phrase-list-features-in-luis"></a>LUIS 中的片語清單功能

在機器學習中，「特性」是您的系統觀察到的資料特徵或屬性。 

將特性新增至語言模型，以針對如何辨識您想要標示或分類的輸入，提供相關提示。 特性可協助 LUIS 辨識意圖和實體，但特性不是意圖或實體本身。 相反地，替性可能會提供相關字詞的範例。  

## <a name="what-is-a-phrase-list-feature"></a>什麼是片語清單特性？
片語清單包含一組屬於相同類別的值 (字組或片語)，而且必須以類似的方式處理 (例如城市或產品的名稱)。 LUIS 對於其中一個值的認識也會自動套用到其他值。 這個清單不是相符文字的封閉[清單實體](luis-concept-entity-types.md#types-of-entities) \(全文相符項目\)。

片語清單會新增至應用程式網域的詞彙，作為向 LUIS 指示這些字的第二個信號。

## <a name="how-to-use-phrase-lists"></a>如何使用片語清單
在人力資源應用程式的[簡單實體教學課程](luis-quickstart-primary-and-secondary-data.md)中，應用程式會使用工作類型的 **Job** 片語類型，例如程式設計人員、屋頂工和秘書。 如果您將這其中一個值標示為機器學習的實體，LUIS 就會學習辨識其他值。 

片語清單可能可互換或不可互換。 「可交換」的片語清單適用於同義字的值，而「不可交換」的片語清單適用於不是同義字，但在應用程式中仍然需要其他信號的值。 

<a name="phrase-lists-help-identify-simple-exchangeable-entities"></a>
## <a name="phrase-lists-help-identify-simple-interchangeable-entities"></a>片語清單有助於識別簡單的可交換實體
可交換的片語清單是調整 LUIS 應用程式效能的好方法。 如果您的應用程式無法將語句預測為正確的意圖，或辨識實體，請思考語句是否包含不尋常的字組，或意義可能會模稜兩可的字組。 這些字組是要包含在片語清單中的理想候選項目。

## <a name="phrase-lists-help-identify-intents-by-better-understanding-context"></a>片語清單有助於以更加理解的內容來識別意圖
片語清單不會指示 LUIS 執行嚴格的比對，或一律標示片語清單中完全相同的所有字詞。 這只是一個提示。 例如，您可能有一個指出 "Patti" 和 "Selma" 都是名稱的片語清單，但 LUIS 仍可使用內容資訊來將它們認定為在 "Make a reservation for 2 at Patti's Diner for dinner" (在 Patti's Diner 預訂兩人份的晚餐) 和 "Find me driving directions to Selma, Georgia" (為我指引到 Selma, Georgia 的行車路線) 中有不同的意義。 

新增片語清單是將更多範例語句新增至意圖的替代方式。 

## <a name="an-interchangeable-phrase-list"></a>可交換的片語清單
在字詞或片語清單建立類別或群組時，使用可交換的片語清單。 範例為月份 (例如 "January"、"February"、"March") 或名稱 (例如 "John"、"Mary"、"Frank") 的清單。  這些清單是可交換的，因為，如果在片語中使用了不同的字詞，則會使用相同的意圖或實體來標示語句。 例如，如果 "show the calendar for January" (顯示一月份的行事曆) 的意圖與 "show the calendar for February" (顯示二月份的行事曆) 相同，則文字應該會位於可交換的清單上。 

## <a name="a-non-interchangeable-phrase-list"></a>不可交換的片語清單
針對可在您的網域中加以群組的非同義字詞或片語，使用不可交換的片語清單。 

例如，針對罕見、專用及外來的字詞，使用不可交換的片語清單。 LUIS 可能無法辨識罕見與專用的文字以及外來文字 (應用程式的文化特性之外)。 不可交換的設定表示這組罕見文字會構成 LUIS 應學習辨識的類別，但這些文字並不是同義字或可彼此交換的。

## <a name="when-to-use-phrase-lists-versus-list-entities"></a>何時使用片語清單與清單實體
雖然片語清單和清單實體都可能影響所有意圖的語句，但各自會以不同方式帶來影響。 使用片語清單來影響意圖預測分數。 使用清單實體來影響全文相符實體的擷取。 

### <a name="use-a-phrase-list"></a>使用片語清單
使用片語清單，LUIS 仍可將內容納入考量並且一般化，以識別類似但並未與清單中的項目完全相符的項目。 如果您希望 LUIS 應用程式能夠一般化並識別類別中的新項目，請使用字詞清單。 

當您想要能夠辨識實體的新執行個體時 (應可辨識新連絡人名稱的會議排程器，或應可辨識新產品的清查應用程式)，請使用另一種機器學習的實體類型，例如簡單或階層式實體。 然後建立字組和片語的片語清單，協助 LUIS 尋找其他類似於實體的字組。 這份清單會將額外的重要性新增至這些個字組的值，引導 LUIS 辨識實體範例。 

片語清單就像是網域專屬詞彙，有助於增強意圖和實體的理解品質。 片語清單的常見用法是專有名詞，例如城市名稱。 城市名稱可以是包含連字號或單引號的幾個字組。
 
### <a name="dont-use-a-phrase-list"></a>請勿使用片語清單 
清單實體會明確地定義實體可以採用的每個值，而且只會識別完全相符的值。 清單實體可能適用於下列應用程式：其中實體的所有執行個體均為已知且不會經常變更。 範例為餐廳菜單上不常變更的食物項目。 如果您需要全文相符的實體，請勿使用片語清單。 

## <a name="best-practices"></a>最佳作法
了解[最佳做法](luis-concept-best-practices.md)。

## <a name="next-steps"></a>後續步驟

請參閱[新增功能](luis-how-to-add-features.md)，深入了解如何將功能新增至 LUIS 應用程式。
