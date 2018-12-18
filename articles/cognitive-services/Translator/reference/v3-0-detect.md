---
title: Microsoft 翻譯工具文字 API 偵測方法 | Microsoft Docs
description: 使用 Microsoft 翻譯工具文字 API 偵測方法。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: 7e81e91230e1ada4423d77d22134b1b64df65d9d
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370235"
---
# <a name="text-api-30-detect"></a>文字 API 3.0：偵測

識別一段文字的語言。

## <a name="request-url"></a>要求 URL

將 `POST` 要求傳送至：

```HTTP
https://api.cognitive.microsofttranslator.com/detect?api-version=3.0
```

## <a name="request-parameters"></a>要求參數

在查詢字串上傳遞的要求參數為：

<table width="100%">
  <th width="20%">查詢參數</th>
  <th>說明</th>
  <tr>
    <td>api-version</td>
    <td>*必要參數*。<br/>用戶端要求的 API 版本。 值必須是 `3.0`。</td>
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
    <td>*必要的要求標頭*。<br/>指定酬載的內容類型。 可能的值為：`application/json`。</td>
  </tr>
  <tr>
    <td>Content-Length</td>
    <td>*必要的要求標頭*。<br/>要求本文的長度。</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*選擇性*。<br/>用於唯一識別要求之用戶端產生的 GUID。 請注意，若您使用名為 `ClientTraceId` 的查詢參數在查詢字串中包含追蹤識別碼，您就可以省略此標頭。</td>
  </tr>
</table> 

## <a name="request-body"></a>Request body

要求的本文是 JSON 陣列。 每個陣列項目都是字串屬性名為 `Text` 的 JSON 物件。 語言偵測會套用到 `Text` 屬性的值。 範例回應本文應如下所示：

```json
[
    { "Text": "Ich würde wirklich gern Ihr Auto um den Block fahren ein paar Mal." }
]
```

套用下列限制：

* 陣列最多可以有 100 個項目。
* 陣列項目的文字值不能超過 10,000 個字元，包括空格。
* 要求中包含的完整文字不能超過 50,000 個字元，包括空格。

## <a name="response-body"></a>Response body

成功的回應是輸入陣列的每個字串各有一個結果的 JSON 陣列。 結果物件包含下列屬性：

  * `language`：代碼，代表偵測到的語言。

  * `score`：浮點值，表示結果的信賴度。 分數介於 0 與 1 之間，低分數表示信賴度徧低。

  * `isTranslationSupported`：布林值，若偵測到的語言是文字翻譯支援的語言之一，則為 true。

  * `isTransliterationSupported`：布林值，若偵測到的語言是轉換支援的語言之一，則為 true。
  
  * `alternatives`：陣列，包含其他可能語言。 陣列的每個項目是具有上方所列相同屬性的另一個物件：`language`、`score`、`isTranslationSupported` 和 `isTransliterationSupported`。

範例 JSON 回應如下：

```json
[
  {
    "language": "de",
    "score": 0.92,
    "isTranslationSupported": true,
    "isTransliterationSupported": false,
    "alternatives": [
      {
        "language": "pt",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      },
      {
        "language": "sk",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      }
    ]
  }
]
```

## <a name="response-headers"></a>回應標頭

<table width="100%">
  <th width="20%">headers</th>
  <th>說明</th>
  <tr>
    <td>X-RequestId</td>
    <td>用於識別要求之服務產生的值。 其用於進行疑難排解。</td>
  </tr>
</table> 

## <a name="response-status-codes"></a>回應狀態碼

以下是要求傳回之可能的 HTTP 狀態碼。 

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

以下範例示範如何擷取文字翻譯支援的語言。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/detect?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'What language is this text written in?'}]"
```

---
