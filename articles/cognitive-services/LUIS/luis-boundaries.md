---
title: Language Understanding (LUIS) 的界線和限制
titleSuffix: Azure Cognitive Services
description: 本文包含 Azure 認知服務 Language Understanding (LUIS) 的已知限制。 LUIS 句有數個界線領域。 模型界線可控制 LUIS 中的意圖、實體和特性。 以金鑰類型為基礎的配額限制。 鍵盤組合可控制 LUIS 網站。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: b7a2d1554d668b4000b87b210a69b290def9b19e
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035431"
---
# <a name="luis-boundaries"></a>LUIS 界線
LUIS 句有數個界線領域。 第一個是[模型界線](#model-boundaries)，其控制 LUIS 中的意圖、實體和功能。 第二個領域是以金鑰類型為基礎的[配額限制](#key-limits)。 第三個界線領域是用來控制 LUIS 網站的[鍵盤組合](#keyboard-controls)。 第四個領域是 LUIS 撰寫網站和 LUIS [端點](luis-glossary.md#endpoint) API 之間的[世界區域對應](luis-reference-regions.md)。 


## <a name="model-boundaries"></a>模型界線

|領域|限制|
|--|:--|--|
| [應用程式名稱][luis-get-started-create-app] | *預設字元上限 |
| [批次測試][batch-testing]| 10 個資料集，每個資料集 1000 個語句|
| **[複合](./luis-concept-entity-types.md)|100 個，最多 10 個子系 |
| 明確清單 | 每個應用程式 50 個|
| **[階層式](./luis-concept-entity-types.md) |100 個，最多 10 個子系 |
| [意圖][intents]|每個應用程式 500 個<br>[發送型](https://aka.ms/dispatch-tool) \(英文\) 應用程式具有相對應的 500 個發送來源|
| [清單實體](./luis-concept-entity-types.md) | 父系：50 個項目，子系：20,000 個項目。 正式名稱為*預設字元上限。同義值沒有長度限制。 |
| [模式](luis-concept-patterns.md)|每個應用程式 500 個模式。<br>模式的長度上限為 400 個字元。<br>每個模式 3 個 pattern.any 實體<br>模式中最多有 2 個巢狀選擇性文字|
| [Pattern.any](./luis-concept-entity-types.md)|每個應用程式 100 個，每個模式 3 個 pattern.any 實體 |
| [片語清單][phrase-list]|10 個片語清單，每個清單 5,000 個項目|
| [預先建置實體](./luis-prebuilt-entities.md) | 沒有限制|
| [規則運算式實體](./luis-concept-entity-types.md)|20 個實體<br>每個規則運算式實體模式 具有 500 個字元的上限|
| [角色](luis-concept-roles.md)|每個應用程式 300 個角色。 每個實體 10 個角色|
| **[簡單](./luis-concept-entity-types.md)| 100 個實體|
| [語句][utterances] | 500 個字元|
| [語句][utterances] | 每個應用程式 15,000 個|
| [版本](luis-concept-version.md)| 沒有限制 |
| [版本名稱][luis-how-to-manage-versions] | 10 個字元，限制為英數字元和句號 (.) |

*預設字元上限為 50 個字元。 

**簡單、階層式及複合實體的總計數不能超過 100 個。 階層式實體、複合實體、簡單實體，以及階層式子系實體的總計數不能超過 330 個。 

## <a name="intent-and-entity-naming"></a>意圖和實體命名
請不要在意圖和實體名稱中使用下列字元：

|Character|名稱|
|--|--|
|`{`|左大括號|
|`}`|右大括號|
|`[`|左方括號|
|`]`|右方括號|
|`\`|反斜線|

## <a name="key-limits"></a>金鑰限制
撰寫金鑰針對撰寫及端點具有不同的限制。 LUIS 服務端點金鑰僅針對端點查詢有效。

|Key|編寫|端點|目的|
|--|--|--|--|
|撰寫/入門|1 百萬個/月，5 個/秒|1 千個/月，5 個/秒|撰寫 LUIS 應用程式|
|[訂用帳戶][pricing] - F0 - 免費層 |無效|1 萬個/月，5 個/秒|查詢 LUIS 端點|
|[訂用帳戶][pricing] - S0 - 基本層|無效|50 個/秒|查詢 LUIS 端點|
|[情感分析整合](luis-how-to-publish-app.md#enable-sentiment-analysis)|無效|不收費|新增情感資訊，包括關鍵片語資料擷取 |
|語音整合|無效|$5.50 美元/1 千個端點要求|將口語語句轉換成文字語句並傳回 LUIS 結果|

## <a name="keyboard-controls"></a>鍵盤控制項

|鍵盤輸入 | 說明 | 
|--|--|
|Ctrl+E|在語句清單上的權杖和實體之間切換|

## <a name="website-sign-in-time-period"></a>網站登入時段

您登入存取的時間為 **60 分鐘**。 在此時段之後，您將會收到這個錯誤。 您需要再次登入。

[luis-get-started-create-app]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app
[batch-testing]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-test#batch-testing
[intents]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-intent
[phrase-list]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-feature
[utterances]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-utterance
[luis-how-to-manage-versions]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-how-to-manage-versions
[pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
<!-- TBD: fix this link -->
[speech-to-intent-pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
