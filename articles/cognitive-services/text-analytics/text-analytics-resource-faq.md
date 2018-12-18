---
title: 關於文字分析 API 的常見問題集 - Azure 認知服務 | Microsoft Docs
description: 取得 Azure 上 Microsoft 認知服務文字分析 API 常見問題的解答。
services: cognitive-services
author: HeidiSteen
manager: cgronlun
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: conceptual
ms.date: 3/07/2018
ms.author: heidist
ms.openlocfilehash: bf82899b4317f0f5ce0f6ca5096dccef7cddd931
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370370"
---
# <a name="frequently-asked-questions-faq-about-the-text-analytics-api"></a>關於文字分析 API 的常見問題集 (FAQ)

 尋找 Azure 上 Microsoft 認知服務文字分析 API 相關概念、程式碼和案例常見問題的解答。

## <a name="can-text-analytics-identify-sarcasm"></a>文字分析是否可以識別嘲笑挖苦？

分析是針對正面與負面人氣，而不是心情偵測。

情感分析一定會有某種程度的不精確，但若沒有隱藏意義或內容潛在含意，則此模型會最實用。 諷刺、嘲笑挖苦、幽默和非常類似的內容依賴文化內容和規範來傳達意圖。 這類型內容在分析時的挑戰最高。 一般而言，針對其意義非常類似的內容，分析器所產生的指定分數與人為主觀評定之間會有最大相依性。

## <a name="can-i-add-my-own-training-data-or-models"></a>我是否可以新增自己的訓練資料或模型？

否，模型已預先訓練。 唯一可對上傳資料執行的作業為計分、關鍵片語擷取和語言偵測。 我們不會裝載自訂模型。 若您想要建立及裝載自訂 Machine Learning 模型，請考慮使用 [Microsoft R Server 中的機器學習服務功能](https://docs.microsoft.com/r-server/r/concept-what-is-the-microsoftml-package)。

## <a name="can-i-request-additional-languages"></a>我是否可以要求其他語言？

情感分析和關鍵片語擷取適用於[某些選取的語言](text-analytics-supported-languages.md)。 自然語言處理非常複雜，而且需要大量測試，才能發行新功能。 基於這個理由，我們會避免預先宣告支援，以確保不會有人採用需要更多時間成熟的相依性和功能。 

若要協助我們排列接下來要支援哪些語言的優先順序，請在 [UserVoice](https://cognitive.uservoice.com/forums/555922-text-analytics) 投票支持特定語言。 

## <a name="why-does-key-phrase-extraction-return-some-words-but-not-others"></a>為什麼關鍵片語擷取會傳回某些文字而非其他文字？

關鍵片語擷取會排除不必要的文字和單獨的形容詞。 形容詞與名詞組合 (例如 "spectacular views" 或 "foggy weather") 會一起傳回。

一般而言，輸出是由句子的名詞和受詞所組成。 輸出會依重要性順序列出，第一個片語最重要。 重要性的測量方式為提到特定概念的次數，或該項目與文字中其他項目的關聯。

## <a name="why-does-output-vary-given-identical-inputs"></a>為什麼相同的輸入會有不同的輸出？

模型和演算法的改善若為主要變更則會宣佈，若為次要更新則會以安靜模式匯集到服務中。 過了一段時間，您可能會發現相同的文字輸入產生不同的人氣分數或關鍵片語輸出。 這是在雲端中使用受控機器學習服務資源的正常蓄意結果。

## <a name="next-steps"></a>後續步驟

您的問題是否與缺少特性或功能相關？ 請考慮在我們的 [UserVoice 網站](https://cognitive.uservoice.com/forums/555922-text-analytics)上要求或對它投票。

## <a name="see-also"></a>另請參閱

 [StackOverflow：文字分析 API](https://stackoverflow.com/questions/tagged/text-analytics-api)   
 [StackOverflow：認知服務](http://stackoverflow.com/questions/tagged/microsoft-cognitive)
