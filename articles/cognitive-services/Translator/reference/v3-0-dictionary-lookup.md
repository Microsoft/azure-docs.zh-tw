---
title: Microsoft 翻譯工具文字 API 字典查閱方法 | Microsoft Docs
description: 使用 Microsoft 翻譯工具文字 API 字典查閱方法。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: 5a186f60dc099b095c00056d965aa92618c2c708
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37868080"
---
# <a name="text-api-30-dictionary-lookup"></a>文字 API 3.0：字典查閱

提供字組和少數慣用語的替代翻譯。 每個翻譯都有一個詞性和一份反向翻譯清單。 反向翻譯可讓使用者了解內容的翻譯。 [字典範例](.\v3-0-dictionary-examples.md)作業允許進一步向下鑽研，以查看每組翻譯配對的範例用法。

## <a name="request-url"></a>要求 URL

將 `POST` 要求傳送至：

```HTTP
https://api.cognitive.microsofttranslator.com/dictionary/lookup?api-version=3.0
```

## <a name="request-parameters"></a>要求參數

在查詢字串上傳遞的要求參數為：

<table width="100%">
  <th width="20%">查詢參數</th>
  <th>說明</th>
  <tr>
    <td>api-version</td>
    <td>必要參數。<br/>用戶端要求的 API 版本。 值必須為 `3.0`。</td>
  </tr>
  <tr>
    <td>from</td>
    <td>必要參數。<br/>指定輸入文字的語言。 來源語言必須是 `dictionary` 範圍內包含的[支援語言](.\v3-0-languages.md)之一。</td>
  </tr>
  <tr>
    <td>to</td>
    <td>必要參數。<br/>指定輸出文字的語言。 目標語言必須是 `dictionary` 範圍內包含的[支援語言](.\v3-0-languages.md)之一。</td>
  </tr>
</table>

要求標頭包括：

<table width="100%">
  <th width="20%">headers</th>
  <th>說明</th>
  <tr>
    <td>_一個授權_<br/>_標頭_</td>
    <td>必要的要求標頭。<br/>請參閱[可用的驗證選項](./v3-0-reference.md#authentication)。</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>必要的要求標頭。<br/>指定承載的內容類型。 可能的值為：`application/json`。</td>
  </tr>
  <tr>
    <td>Content-Length</td>
    <td>必要的要求標頭。<br/>要求本文的長度。</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*選擇性*。<br/>用於識別唯一要求的 GUID，由用戶端產生。 若您使用名為 `ClientTraceId` 的查詢參數在查詢字串中包含追蹤識別碼，您就可以省略此標頭。</td>
  </tr>
</table> 

## <a name="request-body"></a>Request body

要求的本文是 JSON 陣列。 每個陣列項目都是字串屬性名為 `Text` 的 JSON 物件，其表示要查閱的字詞。

```json
[
    {"Text":"fly"}
]
```

套用下列限制：

* 陣列最多可以有 10 個項目。
* 陣列項目的文字值不能超過 100 個字元，包括空格。

## <a name="response-body"></a>Response body

成功的回應是輸入陣列的每個字串各有一個結果的 JSON 陣列。 結果物件包含下列屬性：

  * `normalizedSource`：字串，指定來源字詞的標準化形式。 例如，若要求是 "JOHN"，標準化形式將是 "john"。 這個欄位的內容會變成[查閱範例](.\v3-0-dictionary-examples.md)的輸入。
    
  * `displaySource`：字串，以最適合終端使用者顯示的形式指定來源字詞。 例如，若輸入為 "JOHN"，顯示形式會反映慣用的名稱拼字："John"。 

  * `translations`：清單，包含來源字詞的翻譯。 清單的每個項目是具有下列屬性的物件：

    * `normalizedTarget`：字串，指定這個字詞在目標語言中的的標準化形式。 這個值應該作為[查閱範例](.\v3-0-dictionary-examples.md)的輸入。

    * `displayTarget`：字串，以目標語言和最適合終端使用者顯示的形式指定字詞。 一般來說，這僅在大小寫方面不同於 `normalizedTarget`。 例如，對於 "Juan" 之類的專有名詞，其 `normalizedTarget = "juan"` 且 `displayTarget = "Juan"`。

    * `posTag`：字串，將這個字詞與詞性標籤建立關聯。

        | 標籤名稱 | 說明  |
        |----------|--------------|
        | ADJ      | 形容詞   |
        | ADV      | 副詞      |
        | CONJ     | 連接詞 |
        | DET      | 限定詞  |
        | MODAL    | 動詞        |
        | NOUN     | 名詞        |
        | PREP     | 介系詞 |
        | PRON     | 代名詞     |
        | VERB     | 動詞        |
        | OTHER    | 其他        |

        作為實作注意事項，這些標籤是透過標記英文端的詞性來決定，然後針對每個來源/目標配對採用最常見的標籤。 因此，若人們經常將西班牙文字組翻譯成英文中的不同詞性標籤，則標籤最後可能會發生錯誤 (相對於西班牙文字組)。

    * `confidence`：介於 0.0 與 1.0 之間的值，表示該翻譯配對的「信賴度」(或者可能更準確地說，「訓練資料中的機率」)。 某個來源字組的信賴度分數總和不一定加總為 1.0。 

    * `prefixWord`：字串，指定要顯示為翻譯前置詞的字組。 目前在具有性別限定詞的語言中，這是名詞的性別限定詞。 例如，西班牙文字組 "mosca" 的前置詞是 "la"，因為 "mosca" 是西班牙文中的女性名詞。 這只取決於翻譯，而非來源。 若沒有前置詞，它將會是空字串。
    
    * `backTranslations`：清單，包含目標的「反向翻譯」。 例如，目標可翻譯成的來源字組。 此清單保證包含所要求的來源字組 (例如，若所要查閱的來源字組是 "fly"，則保證 "fly" 將會在 `backTranslations` 清單中)。 不過，不保證它位在第一個位置，通常也不會在該位置。 `backTranslations` 清單的每個項目都是以下列屬性描述的物件：

        * `normalizedText`：字串，指定來源字詞的標準化形式，其為目標的反向翻譯。 這個值應該作為[查閱範例](.\v3-0-dictionary-examples.md)的輸入。        

        * `displayText`：字串，以最適合終端使用者顯示的形式指定其為目標反向翻譯的來源字詞。

        * `numExamples`：整數，代表可供此翻譯配對使用的範例數目。 您必須透過個別呼叫[查閱範例](.\v3-0-dictionary-examples.md)來擷取實際範例。 此數目主要是為了方便在 UX 中顯示。 比方說，若範例數目大於零，使用者介面可能會新增反向翻譯的超連結；若沒有任何範例，則會以純文字格式顯示反向翻譯。 請注意，[查閱範例](.\v3-0-dictionary-examples.md)呼叫所傳回的實際範例數目可能會小於 `numExamples`，因為可能會即時套用其他篩選來移除「不良」的範例。
        
        * `frequencyCount`：整數，代表這個翻譯配對在資料中的頻率。 此欄位的主要用途是為使用者介面提供排序反向翻譯的方法，讓最常用的字詞成為第一個。

    > [!NOTE]
    > 若所要查詢的字詞不存在於字典中，回應會是 200 (良好)，但 `translations` 清單卻是空清單。

## <a name="examples"></a>範例

此範例示範如何查閱英文字詞 `fly` 的西班牙文替代翻譯。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/dictionary/lookup?api-version=3.0&from=en&to=es" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'fly'}]"
```

---

回應本文 (為了清楚緣故已縮減) 如下：

```
[
    {
        "normalizedSource":"fly",
        "displaySource":"fly",
        "translations":[
            {
                "normalizedTarget":"volar",
                "displayTarget":"volar",
                "posTag":"VERB",
                "confidence":0.4081,
                "prefixWord":"",
                "backTranslations":[
                    {"normalizedText":"fly","displayText":"fly","numExamples":15,"frequencyCount":4637},
                    {"normalizedText":"flying","displayText":"flying","numExamples":15,"frequencyCount":1365},
                    {"normalizedText":"blow","displayText":"blow","numExamples":15,"frequencyCount":503},
                    {"normalizedText":"flight","displayText":"flight","numExamples":15,"frequencyCount":135}
                ]
            },
            {
                "normalizedTarget":"mosca",
                "displayTarget":"mosca",
                "posTag":"NOUN",
                "confidence":0.2668,
                "prefixWord":"",
                "backTranslations":[
                    {"normalizedText":"fly","displayText":"fly","numExamples":15,"frequencyCount":1697},
                    {"normalizedText":"flyweight","displayText":"flyweight","numExamples":0,"frequencyCount":48},
                    {"normalizedText":"flies","displayText":"flies","numExamples":9,"frequencyCount":34}
                ]
            },
            //
            // ...list abbreviated for documentation clarity
            //
        ]
    }
]
```

此範例顯示有效字典配對不存在要進行查閱的字詞時，會發生什麼事。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/dictionary/lookup?api-version=3.0&from=en&to=es" -H "X-ClientTraceId: 875030C7-5380-40B8-8A03-63DACCF69C11" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'fly123456'}]"
```

---

因為在字典中找不到此字詞，回應本文會包含空白的 `translations` 清單。

```
[
    {
        "normalizedSource":"fly123456",
        "displaySource":"fly123456",
        "translations":[]
    }
]
```
