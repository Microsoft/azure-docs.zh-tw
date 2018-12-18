---
title: Microsoft Translator Text API 字典範例方法 | Microsoft Docs
description: 使用 Microsoft Translator Text API 字典範例方法。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: 9960f3be42090edaec1df935d70e4c1a0d25b691
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370239"
---
# <a name="text-api-30-dictionary-examples"></a>文字 API 3.0：字典範例

提供範例以說明字典中的字詞在內容中的使用方式。 此作業會與[字典查閱](.\v3-0-dictionary-lookup.md)搭配使用。

## <a name="request-url"></a>要求 URL

將 `POST` 要求傳送至：

```HTTP
https://api.cognitive.microsofttranslator.com/dictionary/examples?api-version=3.0
```

## <a name="request-parameters"></a>要求參數

在查詢字串上傳遞的要求參數如下：

<table width="100%">
  <th width="20%">查詢參數</th>
  <th>說明</th>
  <tr>
    <td>api-version</td>
    <td>*必要參數*。<br/>用戶端要求的 API 版本。 值必須是 `3.0`。</td>
  </tr>
  <tr>
    <td>from</td>
    <td>*必要參數*。<br/>指定輸入文字的語言。 來源語言必須是 `dictionary` 範圍內包含的[支援語言](.\v3-0-languages.md)之一。</td>
  </tr>
  <tr>
    <td>to</td>
    <td>*必要參數*。<br/>指定輸出文字的語言。 目標語言必須是 `dictionary` 範圍內包含的[支援語言](.\v3-0-languages.md)之一。</td>
  </tr>
</table>

要求標頭包括：

<table width="100%">
  <th width="20%">headers</th>
  <th>說明</th>
  <tr>
    <td>_一個授權_<br/>_標頭_</td>
    <td>*必要的要求標頭*。<br/>請參閱[可用的驗證選項](./v3-0-reference.md#authentication)。</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>*必要的要求標頭*。<br/>指定承載的內容類型。 可能的值為：`application/json`。</td>
  </tr>
  <tr>
    <td>Content-Length</td>
    <td>*必要的要求標頭*。<br/>要求本文的長度。</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*選擇性*。<br/>用戶端產生用以唯一識別要求的 GUID。 如果您使用名為 `ClientTraceId` 的查詢參數在查詢字串中包含追蹤識別碼，您就可以省略此標頭。</td>
  </tr>
</table> 

## <a name="request-body"></a>Request body

要求的本文是 JSON 陣列。 每個陣列元素都是具有下列屬性的 JSON 物件：

  * `Text`：一個字串，指定要查閱的字詞。 此屬性應為先前[字典查閱](.\v3-0-dictionary-lookup.md)要求的反向翻譯中的 `normalizedText` 欄位值。 它也可以是 `normalizedSource` 欄位的值。

  * `Translation`：一個字串，指定[字典查閱](.\v3-0-dictionary-lookup.md)作業先前傳回的翻譯文字。 此屬性應為[字典查閱](.\v3-0-dictionary-lookup.md)回應的 `translations` 清單中包含的 `normalizedTarget` 欄位值。 服務會傳回特定的來源-目標字組配對的範例。

範例如下：

```json
[
    {"Text":"fly", "Translation":"volar"}
]
```

套用下列限制：

* 陣列最多可以有 10 個元素。
* 陣列元素的文字值不能超過 100 個字元，包括空格。

## <a name="response-body"></a>Response body

成功的回應是輸入陣列的每個字串各有一個結果的 JSON 陣列。 結果物件包含下列屬性：

  * `normalizedSource`：一個字串，指定來源字詞的標準化形式。 一般而言，此屬性應等同於要求本文中位於相符清單索引上的 `Text` 欄位值。
    
  * `normalizedTarget`：一個字串，指定目標字詞的標準化形式。 一般而言，此屬性應等同於要求本文中位於相符清單索引上的 `Translation` 欄位值。
  
  * `examples`：(來源字詞、目標字詞) 配對的範例清單。 清單的每個元素是具有下列屬性的物件：

    * `sourcePrefix`：要在 `sourceTerm` 的值之前串連以形成完整範例的字串。 請勿加入空格字元，因為此字元會在必要時自動加入。 此值可以是空字串。

    * `sourceTerm`：與查閱的實際字詞等同的字串。 此字串可透過 `sourcePrefix` 和 `sourceSuffix` 來新增，以形成完整範例。 其值會分隔以便標示於使用者介面中，例如藉由粗體來標示。

    * `sourceSuffix`：要在 `sourceTerm` 的值之後串連以形成完整範例的字串。 請勿加入空格字元，因為此字元會在必要時自動加入。 此值可以是空字串。

    * `targetPrefix`：一個類似於 `sourcePrefix` 但用於目標的字串。

    * `targetTerm`：一個類似於 `sourceTerm` 但用於目標的字串。

    * `targetSuffix`：一個類似於 `sourceSuffix` 但用於目標的字串。

    > [!NOTE]
    > 如果字典中沒有範例，回應會是 200 (良好)，但 `examples` 清單卻是空白清單。

## <a name="examples"></a>範例

此範例說明如何查閱為英文字詞 `fly` 及其西班牙文翻譯 `volar` 所做配對的範例。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/dictionary/examples?api-version=3.0&from=en&to=es" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'fly', 'Translation':'volar'}]"
```

---

回應本文 (為了清楚起見已縮減) 如下：

```
[
    {
        "normalizedSource":"fly",
        "normalizedTarget":"volar",
        "examples":[
            {
                "sourcePrefix":"They need machines to ",
                "sourceTerm":"fly",
                "sourceSuffix":".",
                "targetPrefix":"Necesitan máquinas para ",
                "targetTerm":"volar",
                "targetSuffix":"."
            },      
            {
                "sourcePrefix":"That should really ",
                "sourceTerm":"fly",
                "sourceSuffix":".",
                "targetPrefix":"Eso realmente debe ",
                "targetTerm":"volar",
                "targetSuffix":"."
            },
            //
            // ...list abbreviated for documentation clarity
            //
        ]
    }
]
```
