---
title: Microsoft Translator Text API 翻譯方法 | Microsoft Docs
titleSuffix: Cognitive Services
description: 使用 Microsoft Translator Text API 翻譯方法。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: d8d5e1e2fac747fa733f1d92c08008b7eac2a1bc
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370674"
---
# <a name="text-api-30-translate"></a>Text API 3.0：翻譯

翻譯文字。

## <a name="request-url"></a>要求 URL

將 `POST` 要求傳送至：

```HTTP
https://api.cognitive.microsofttranslator.com/translate?api-version=3.0
```

## <a name="request-parameters"></a>要求參數

在查詢字串上傳遞的要求參數如下：

<table width="100%">
  <th width="20%">查詢參數</th>
  <th>說明</th>
  <tr>
    <td>api-version</td>
    <td>必要參數。<br/>用戶端要求的 API 版本。 值必須是 `3.0`。</td>
  </tr>
  <tr>
    <td>from</td>
    <td>選擇性參數。<br/>指定輸入文字的語言。 使用 `translation` 範圍查閱[支援語言](.\v3-0-languages.md)，以尋找可用於翻譯的來源語言。 若未指定 `from` 參數，則會套用自動語言偵測來判斷來源語言。</td>
  </tr>
  <tr>
    <td>to</td>
    <td>必要參數。<br/>指定輸出文字的語言。 目標語言必須是 `translation` 範圍內包含的[支援語言](.\v3-0-languages.md)之一。 例如，使用 `to=de` 翻譯為德文。<br/>在查詢字串中重複參數，可能會同時翻譯為多種語言。 例如，使用 `to=de&to=it` 翻譯為德文和義大利文。</td>
  </tr>
  <tr>
    <td>textType</td>
    <td>選擇性參數。<br/>定義要翻譯的文字是純文字還是 HTML 文字。 任何 HTML 都需要是格式正確的完整項目。 可能的值為：`plain` (預設) 或 `html`。</td>
  </tr>
  <tr>
    <td>category</td>
    <td>選擇性參數。<br/>字串，指定翻譯的分類 (定義域)。 此參數用來從使用 [Custom Translator](../customization.md) 所建置的自訂系統取得翻譯。 預設值為：`general`。</td>
  </tr>
  <tr>
    <td>profanityAction</td>
    <td>選擇性參數。<br/>指定如何處理翻譯中的粗話。 可能的值為：`NoAction` (預設)、`Marked` 或 `Deleted`。 若要了解處理粗話的方式，請參閱[粗話處理](#handle-profanity)。</td>
  </tr>
  <tr>
    <td>profanityMarker</td>
    <td>選擇性參數。<br/>指定如何在翻譯中標示粗話。 可能的值為：`Asterisk` (預設) 或 `Tag`。 若要了解處理粗話的方式，請參閱[粗話處理](#handle-profanity)。</td>
  </tr>
  <tr>
    <td>includeAlignment</td>
    <td>選擇性參數。<br/>指定是否包括從來源文字到翻譯文字的對齊方式投影。 可能的值為：`true` 或 `false` (預設)。 </td>
  </tr>
  <tr>
    <td>includeSentenceLength</td>
    <td>選擇性參數。<br/>指定是否包括輸入文字和翻譯文字的句子界限。 可能的值為：`true` 或 `false` (預設)。</td>
  </tr>
  <tr>
    <td>suggestedFrom</td>
    <td>選擇性參數。<br/>指定無法識別輸入文字語言時的後援語言。 省略 `from` 參數時，會套用語言自動偵測。 若偵測失敗，則會採用 `suggestedFrom` 語言。</td>
  </tr>
  <tr>
    <td>fromScript</td>
    <td>選擇性參數。<br/>指定輸入文字的指令碼。</td>
  </tr>
  <tr>
    <td>toScript</td>
    <td>選擇性參數。<br/>指定翻譯文字的指令碼。</td>
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
    <td>*選擇性*。<br/>用於唯一識別要求之用戶端產生的 GUID。 若您使用名為 `ClientTraceId` 的查詢參數在查詢字串中包含追蹤識別碼，您就可以省略此標頭。</td>
  </tr>
</table> 

## <a name="request-body"></a>Request body

要求的本文是 JSON 陣列。 每個陣列項目都是字串屬性名為 `Text` 的 JSON 物件，其代表要翻譯的字串。

```json
[
    {"Text":"I would really like to drive your car around the block a few times."}
]
```

套用下列限制：

* 陣列最多可以有 25 個項目。
* 要求中包含的完整文字不能超過 5,000 個字元，包括空格。

## <a name="response-body"></a>Response body

成功的回應是輸入陣列的每個字串各有一個結果的 JSON 陣列。 結果物件包含下列屬性：

  * `detectedLanguage`：物件，透過下列屬性描述偵測到語言：

      * `language`：字串，代表偵測到語言的代碼。

      * `score`：浮點值，表示結果的信賴度。 分數介於 0 與 1 之間，低分數表示信賴度徧低。

    只有在要求自動偵測語言時，`detectedLanguage` 屬性才會存在於結果物件中。

  * `translations`：翻譯結果的陣列。 陣列大小符合透過 `to` 查詢參數指定的目標語言數。 陣列中的每個項目都包括：

    * `to`：字串，代表目標語言的語言代碼。

    * `text`：字串，提供翻譯文字。

    * `transliteration`：物件，提供 `toScript` 參數所指定指令碼中的翻譯文字。

      * `script`：字串，指定目標指令碼。   

      * `text`：字串，提供目標指令碼中的翻譯文字。

    若未進行音譯，則不會包括 `transliteration` 物件。

    * `alignment`：單一字串屬性名為 `proj` 的物件，以將輸入文字對應至翻譯文字。 只有在要求參數 `includeAlignment` 是 `true` 時，才會提供對齊方式資訊。 對齊方式會以下列格式的字串值傳回：`[[SourceTextStartIndex]:[SourceTextEndIndex]–[TgtTextStartIndex]:[TgtTextEndIndex]]`。  冒號區隔開始與結束索引、虛線區隔語言，而空格區隔字組。 一個字組可能符合另一種語言的零、一或多個字組，而且符合的字組可能未連續。 沒有對齊方式資訊可用時，對齊方式項目會空白。 如需範例和限制，請參閱[取得對齊資訊](#obtain-alignment-information)。

    * `sentLen`：物件，傳回輸入和輸出文字中的句子界限。

      * `srcSentLen`：整數陣列，代表輸入文字中句子的長度。 陣列長度就是句子數目，而值是每個句子的長度。

      * `transSentLen`：整數陣列，代表翻譯文字中句子的長度。 陣列長度就是句子數目，而值是每個句子的長度。

    只有在要求參數 `includeSentenceLength` 是 `true` 時，才會包括句子界限。

  * `sourceText`：具有名為 `text` 之單一字串屬性的物件，以提供來源語言之預設指令碼中的輸入文字。 只有在以不是語言一般指令碼的指令碼表示輸入時，才會有 `sourceText` 屬性。 例如，若輸入是以拉丁文指令碼寫入的阿拉伯文，則 `sourceText.text` 會是轉換成阿拉伯文指令碼的相同阿拉伯文文字。

[範例](#examples)一節提供 JSON 回應的範例。

## <a name="response-status-codes"></a>回應狀態碼

以下是要求傳回的可能 HTTP 狀態碼。 

<table width="100%">
  <th width="20%">狀態碼</th>
  <th>說明</th>
  <tr>
    <td>200</td>
    <td>成功。</td>
  </tr>
  <tr>
    <td>400</td>
    <td>其中一個查詢參數遺漏或無效。 請先更正要求參數，然後重試。</td>
  </tr>
  <tr>
    <td>401</td>
    <td>無法驗證此要求。 請確認認證已指定且有效。</td>
  </tr>
  <tr>
    <td>403</td>
    <td>此要求未經授權。 請查看詳細錯誤訊息。 這通常表示試用訂用帳戶提供的所有免費翻譯都已用完。</td>
  </tr>
  <tr>
    <td>429</td>
    <td>呼叫者傳送太多要求。</td>
  </tr>
  <tr>
    <td>500</td>
    <td>發生未預期的錯誤。 若錯誤仍然存在，請回報：失敗的日期和時間、來自回應標頭 `X-RequestId` 的要求識別碼，以及來自要求標頭 `X-ClientTraceId` 的用戶端識別碼。</td>
  </tr>
  <tr>
    <td>503</td>
    <td>暫時無法使用伺服器。 重試要求。 若錯誤仍然存在，請回報：失敗的日期和時間、來自回應標頭 `X-RequestId` 的要求識別碼，以及來自要求標頭 `X-ClientTraceId` 的用戶端識別碼。</td>
  </tr>
</table> 

## <a name="examples"></a>範例

### <a name="translate-a-single-input"></a>翻譯單一輸入

此範例示範如何將單一句子從英文翻譯成簡體中文。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

回應本文如下：

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    }
]
```

`translations` 陣列包括一個項目，以提供輸入中單一文字片段的翻譯。

### <a name="translate-a-single-input-with-language-auto-detection"></a>使用語言自動偵測翻譯單一輸入

此範例示範如何將單一句子從英文翻譯成簡體中文。 要求未指定輸入語言。 改為使用來源語言的自動偵測。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

回應本文如下：

```
[
    {
        "detectedLanguage": {"language": "en", "score": 1.0},
        "translations":[
            {"text": "你好, 你叫什么名字？", "to": "zh-Hans"}
        ]
    }
]
```
回應與前一個範例的回應類似。 因為要求語言自動偵測，所以回應也會包括針對輸入文字偵測到之語言的相關資訊。 

### <a name="translate-with-transliteration"></a>含轉換的翻譯

讓我們新增轉換以擴充前一個範例。 下列要求會要求以拉丁文指令碼撰寫的中文翻譯。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=zh-Latn" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

回應本文如下：

```
[
    {
        "detectedLanguage":{"language":"en","score":1.0},
        "translations":[
            {
                "text":"你好, 你叫什么名字？",
                "transliteration":{"text":"nǐ hǎo , nǐ jiào shén me míng zì ？","script":"Latn"},
                "to":"zh-Hans"
            }
        ]
    }
]
```

翻譯結果現在包括 `transliteration` 屬性，以使用拉丁文字元來提供翻譯文字。

### <a name="translate-multiple-pieces-of-text"></a>翻譯文字的多個片段

一次翻譯多個字串就像是在要求本文中指定字串陣列一樣。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}, {'Text':'I am fine, thank you.'}]"
```

---

回應本文如下：

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    },            
    {
        "translations":[
            {"text":"我很好，谢谢你。","to":"zh-Hans"}
        ]
    }
]
```

### <a name="translate-to-multiple-languages"></a>翻譯為多種語言

本範例示範如何使用一個要求將相同輸入翻譯為數種語言。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

回應本文如下：

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"},
            {"text":"Hallo, was ist dein Name?","to":"de"}
        ]
    }
]
```

### <a name="handle-profanity"></a>處理粗話

Translator 服務通常會在翻譯中保留存在於來源的粗話。 粗話程度以及產生不雅字眼的內容在文化特性之間會不同，因此目標語言中的粗話程度可能會加大或減少。

若您想要在翻譯中避免粗話，則不論來源文字中是否有粗話，都可以使用粗話篩選選項。 此選項可讓您選擇是否要查看刪除的粗話、是否要使用適當的標籤標示粗話 (可讓您選擇新增您自己的後續處理)，或您不想要採取任何動作。 接受的 `ProfanityAction` 值為 `Deleted`、`Marked` 和 `NoAction` (預設)。

<table width="100%">
  <th width="20%">ProfanityAction</th>
  <th>動作</th>
  <tr>
    <td>`NoAction`</td>
    <td>此為預設行為。 粗話會從來源傳遞到目標。<br/><br/>
    **範例來源 (日文)**：彼はジャッカスです。<br/>
    **範例翻譯 (英文)**：He is a jackass.
    </td>
  </tr>
  <tr>
    <td>`Deleted`</td>
    <td>將會從輸出中移除不雅字眼，但不予取代。<br/><br/>
    **範例來源 (日文)**：彼はジャッカスです。<br/>
    **範例翻譯 (英文)**：He is a.
    </td>
  </tr>
  <tr>
    <td>`Marked`</td>
    <td>不雅字眼會取代為輸出中的標記。 標記取決於 `ProfanityMarker` 參數。<br/><br/>
對於 `ProfanityMarker=Asterisk`，不雅字眼會取代為 `***`：<br/>
    **範例來源 (日文)**：彼はジャッカスです。<br/>
    **範例翻譯 (英文)**：He is a \*\*\*.<br/><br/>
對於 `ProfanityMarker=Tag`，不雅字眼會括上 XML 標籤 &lt;profanity&gt; 和 &lt;/profanity&gt;：<br/>
    **範例來源 (日文)**：彼はジャッカスです。<br/>
    **範例翻譯 (英文)**：He is a &lt;profanity&gt;jackass&lt;/profanity&gt;.
  </tr>
</table> 

例如︰

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de&profanityAction=Marked" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'This is a fucking good idea.'}]"
```

---

這會傳回：

```
[
    {
        "translations":[
            {"text":"Das ist eine *** gute Idee.","to":"de"}
        ]
    }
]
```

相較於：

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de&profanityAction=Marked&profanityMarker=Tag" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'This is a fucking good idea.'}]"
```

---

這個最後一個要求會傳回：

```
[
    {
        "translations":[
            {"text":"Das ist eine <profanity>verdammt</profanity> gute Idee.","to":"de"}
        ]
    }
]
```

### <a name="translate-content-with-markup-and-decide-whats-translated"></a>翻譯包含標記的內容，並決定翻譯內容

通常會翻譯包含標記的內容，例如 HTML 頁面的內容或 XML 文件的內容。 翻譯具有標籤的內容時，請包括 `textType=html` 查詢參數。 此外，排除特定內容不翻譯有時可能會十分有用。 您可以使用 `class=notranslate` 屬性，指定應該保留其原始語言的內容。 在下列範例中，不會翻譯第一個 `div` 項目內的內容，但會翻譯第二個 `div` 項目中的內容。

```
<div class="notranslate">This will not be translated.</div>
<div>This will be translated. </div>
```

以下是要說明的範例要求。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&textType=html" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'<div class=\"notranslate\">This will not be translated.</div><div>This will be translated.</div>'}]"
```

---

回應如下：

```
[
    {
        "translations":[
            {"text":"<div class=\"notranslate\">This will not be translated.</div><div>这将被翻译。</div>","to":"zh-Hans"}
        ]
    }
]
```

### <a name="obtain-alignment-information"></a>取得對齊方式資訊

若要收到對齊方式資訊，請在查詢字串上指定 `includeAlignment=true`。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=fr&includeAlignment=true" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'The answer lies in machine translation.'}]"
```

---

回應如下：

```
[
    {
        "translations":[
            {
                "text":"La réponse se trouve dans la traduction automatique.",
                "to":"fr",
                "alignment":{"proj":"0:2-0:1 4:9-3:9 11:14-11:19 16:17-21:24 19:25-40:50 27:37-29:38 38:38-51:51"}
            }
        ]
    }
]
```

對齊方式資訊開始於 `0:2-0:1`，表示來源文字中的前三個字元 (`The`) 對應至翻譯文字中的前兩個字元 (`La`)。

請注意下列限制：

* 只會傳回一部分語言組的對齊方式：
  - 從英文到任何其他語言；
  - 從任何其他語言到英文，但從簡體中文、繁體中文和拉脫維亞文到英文除外；
  - 從日文到韓文，或從韓文到日文。
* 若句子是已定義的翻譯，則您收不到對齊方式。 已定義的翻譯範例是 "This is a test"、"I love you" 和其他高頻率出現句子。

### <a name="obtain-sentence-boundaries"></a>取得句子界限

若要收到來源文字和翻譯文字中句子長度的相關資訊，請在查詢字串上指定 `includeSentenceLength=true`。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=fr&includeSentenceLength=true" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'The answer lies in machine translation. The best machine translation technology cannot always provide translations tailored to a site or users like a human. Simply copy and paste a code snippet anywhere.'}]"
```

---

回應如下：

```
[
    {
        "translations":[
            {
                "text":"La réponse se trouve dans la traduction automatique. La meilleure technologie de traduction automatique ne peut pas toujours fournir des traductions adaptées à un site ou des utilisateurs comme un être humain. Il suffit de copier et coller un extrait de code n’importe où.",
                "to":"fr",
                "sentLen":{"srcSentLen":[40,117,46],"transSentLen":[53,157,62]}
            }
        ]
    }
]
```

### <a name="translate-with-dynamic-dictionary"></a>使用動態字典翻譯

若您已經知道您想要套用至字組或片語的翻譯，則可以提供它作為要求內的標記。 動態字典僅適用於正確名稱和產品名稱這類複合名詞。

要提供的標記會使用下列語法。

``` 
<mstrans:dictionary translation=”translation of phrase”>phrase</mstrans:dictionary>
```

例如，請考慮使用英文句子 "The word wordomatic is a dictionary entry." 若要在翻譯中保留 _wordomatic_ 這個字，則會傳送要求：

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'The word <mstrans:dictionary translation=\"wordomatic\">word or phrase</mstrans:dictionary> is a dictionary entry.'}]"
```

結果如下：

```
[
    {
        "translations":[
            {"text":"Das Wort "wordomatic" ist ein Wörterbucheintrag.","to":"de"}
        ]
    }
]
```

此功能的運作方式與 `textType=text` 或 `textType=html` 相同。 應該謹慎使用此功能。 自訂翻譯的適當且最適合方式是使用 Custom Translator。 Custom Translator 會完全利用內容和統計機率。 若您有或可以建立內容中顯示您工作或片語的訓練資料，則會取得較好的效果。 [深入了解 Custom Translator](../customization.md)。
 





