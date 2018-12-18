---
title: LUIS 應用程式中的語句
titleSuffix: Azure Cognitive Services
description: 語句是應用程式需要解譯的使用者輸入。 收集您認為使用者會輸入的片語。 納入意義相同但以不同單字長度和單字位置建構的語句。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 39c99cc35f4c2549efc9c20af0680b77483325c5
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038930"
---
# <a name="utterances-in-luis"></a>LUIS 中的語句

**語句**是應用程式需要解譯的使用者輸入。 若要將 LUIS 定型以從中擷取意圖和實體，務必要針對每個意圖擷取各種不同的輸入。 主動學習或持續將新語句定型的程序，對 LUIS 提供的機器學習智慧非常重要。

收集您認為使用者會輸入的片語。 納入意義相同但以不同單字長度和單字位置建構的語句。 

## <a name="how-to-choose-varied-utterances"></a>如何選擇各種語句
當您剛開始為 LUIS 模型[新增範例語句](luis-how-to-add-example-utterances.md)時，請將以下幾個原則謹記在心。

### <a name="utterances-arent-always-well-formed"></a>語句不見得格式正確
它可能是一個句子，如「幫我預訂飛往巴黎的機票」，或句子片段，如「預訂」或「巴黎的班機」。  使用者常發生拼字錯誤。 在規劃應用程式時，請考慮是否要先檢查使用者輸入的拼字，再將它傳遞至 LUIS。 [Bing 拼字檢查 API][BingSpellCheck] 與 LUIS 整合。 當您發行 LUIS 應用程式時，可將它與 Bing 拼字檢查 API 的外部金鑰相關聯。 如果未檢查使用者語句的拼字，您應該針對包含錯字與拼字錯誤的語句將 LUIS 定型。

### <a name="use-the-representative-language-of-the-user"></a>使用使用者的代表語言
選擇語句時，請注意您認為是常見的字詞或片語，對於用戶端應用程式的一般使用者而言可能並不常見。 他們可能不具領域經驗。 使用只有當使用者是專家時才會用的字詞與片語時，請小心謹慎。

### <a name="choose-varied-terminology-as-well-as-phrasing"></a>選擇不同的詞彙以及片語
您會發現即使努力建立各種的句子模式，有些詞彙仍會不斷重複。

以下面這些範例語句為例：

|範例語句|
|--|
|如何取得電腦？|
|何處取得電腦？|
|我想要取得電腦，如何著手呢？|
|我何時可擁有電腦？| 

這裡不變的核心字詞為「電腦」。 它們可稱為桌上型電腦、膝上型電腦、工作站，甚至只稱為機器。 LUIS 可運用智慧從上下文推斷出同義字，但當在建立用於定型的語句時，最好還是更改用字。

## <a name="example-utterances-in-each-intent"></a>每個意圖的範例語句
每個意圖都必須至少要有 10 到 15 個範例語句。 如果是沒有任何範例語句的意圖，則無法將 LUIS 定型。 如果是有一個或少數範例語句的意圖，LUIS 會無法準確地預測該意圖。 

## <a name="add-small-groups-of-10-15-utterances-for-each-authoring-iteration"></a>對每個製作的反覆項目都少量新增多組 10-15 個語句
請不要在模型的每個反覆項目中新增大量語句。 新增數十個語句。 再次[定型](luis-how-to-train.md)、[發佈](luis-how-to-publish-app.md)及[測試](luis-interactive-test.md)。  

LUIS 會利用精挑細選的語句來建置有效的模型。 新增太多語句只會導致產生混淆，並沒有用。  

最好從少量語句開始，然後[檢閱端點語句](luis-how-to-review-endoint-utt.md)，以正確地預測意圖和擷取實體。

## <a name="ignoring-words-and-punctuation"></a>忽略單字和標點符號
如果您想要忽略範例語句中的特定單字或標點符號，請搭配「忽略」語法使用[模式](luis-concept-patterns.md#pattern-syntax)。 

## <a name="training-utterances"></a>將語句定型
定型不具確定性：版本或應用程式間的語句預測會稍微不同。

## <a name="testing-utterances"></a>測試語句 

開發人員應該透過將語句傳送至端點，以實際流量測試其 LUIS 應用程式。 這些語句可用來改善使用[檢閱語句](luis-how-to-review-endoint-utt.md)的意圖和實體效能。 使用 LUIS 網站測試窗格提交的測試，不會透過端點傳送，因此也不會提供給主動學習。 

## <a name="review-utterances"></a>檢閱語句
在將模型定型、發佈及接收[端點](luis-glossary.md#endpoint)查詢之後，請[檢閱 LUIS 所建議的語句](luis-how-to-review-endoint-utt.md)。 LUIS 會選取對意圖或實體而言分數低的端點語句。 

## <a name="best-practices"></a>最佳作法
檢閱[最佳作法](luis-concept-best-practices.md)以深入了解。

## <a name="next-steps"></a>後續步驟
如需將 LUIS 應用程式定型以了解使用者語句的詳細資訊，請參閱[新增範例語句](luis-how-to-add-example-utterances.md)。

[BingSpellCheck]: https://docs.microsoft.com/azure/cognitive-services/bing-spell-check/proof-text