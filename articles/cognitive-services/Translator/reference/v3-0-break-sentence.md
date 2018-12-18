---
title: Microsoft 翻譯工具文字 API BreakSentence 方法 | Microsoft Docs
description: 使用 Microsoft 翻譯工具文字 API BreakSentence 方法。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: 8ce6644d21b397ea0e7f2e71e3c3a5a96638eec5
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370242"
---
# <a name="text-api-30-breaksentence"></a>文字 API 3.0：BreakSentence

識別句子界限在一段文字裡的位置。

## <a name="request-url"></a>要求 URL

將 `POST` 要求傳送至：

```HTTP
https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0
```

## <a name="request-parameters"></a>要求參數

在查詢字串上傳遞的要求參數為：

<table width="100%">
  <th width="20%">查詢參數</th>
  <th>說明</th>
  <tr>
    <td>api-version</td>
    <td>*必要查詢參數*。<br/>用戶端要求的 API 版本。 值必須為 `3.0`。</td>
  </tr>
  <tr>
    <td>語言</td>
    <td>*選擇性的查詢參數*。<br/>識別輸入文字語言的語言標記。 如果未指定代碼，將會套用自動語言偵測。</td>
  </tr>
  <tr>
    <td>script</td>
    <td>*選擇性的查詢參數*。<br/>識別輸入文字所使用指令碼的指令碼標記。 如果未指定指令碼，將會假設語言的預設指令碼。</td>
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
    <td>*選擇性*。<br/>用於識別唯一要求的 GUID，由用戶端產生。 請注意，如果您使用名為 `ClientTraceId` 的查詢參數在查詢字串中包含追蹤識別碼，您就可以省略此標頭。</td>
  </tr>
</table> 

## <a name="request-body"></a>Request body

要求的本文是 JSON 陣列。 每個陣列項目都是具有字串屬性 `Text` 的 JSON 物件。 會針對 `Text` 屬性的值計算句子界限。 具有一段文字的要求本文範例看起來像這樣：

```json
[
    { "Text": "How are you? I am fine. What did you do today?" }
]
```

套用下列限制：

* 陣列最多可以有 100 個項目。
* 陣列項目的文字值不能超過 10,000 個字元，包括空格。
* 要求中包含的完整文字不能超過 50,000 個字元，包括空格。
* 如果指定 `language` 查詢參數，則所有陣列項目必須都是相同的語言。 否則，語言自動偵測會個別套用至每個陣列項目。

## <a name="response-body"></a>Response body

成功的回應是輸入陣列的每個字串各有一個結果的 JSON 陣列。 結果物件包含下列屬性：

  * `sentLen`：整數陣列，代表文字項目中的句子長度。 陣列長度就是句子數目，而值是每個句子的長度。 

  * `detectedLanguage`：物件，透過下列屬性描述偵測到的語言：

     * `language`：代碼，代表偵測到的語言。

     * `score`：浮點值，表示結果的信賴度。 分數介於 0 與 1 之間，分數低表示信賴度偏低。
     
    請注意，只有在要求自動偵測語言時，`detectedLanguage` 屬性才會存在於結果物件中。

範例 JSON 回應如下：

```json
[
  {
    "sentenceLengths": [ 13, 11, 22 ]
    "detectedLanguage": {
      "language": "en",
      "score": 401
    },
  }
]
```

## <a name="response-headers"></a>回應標頭

<table width="100%">
  <th width="20%">headers</th>
  <th>說明</th>
  <tr>
    <td>X-RequestId</td>
    <td>服務產生的值，用於識別要求。 作為疑難排解之用。</td>
  </tr>
</table> 

## <a name="response-status-codes"></a>回應狀態碼

以下是要求可能會傳回的 HTTP 狀態碼。 

<table width="100%">
  <th width="20%">狀態碼</th>
  <th>說明</th>
  <tr>
    <td>200</td>
    <td>成功。</td>
  </tr>
  <tr>
    <td>400</td>
    <td>缺少其中一個查詢參數，或查詢參數無效。 請先修正要求參數再重試。</td>
  </tr>
  <tr>
    <td>401</td>
    <td>無法驗證要求。 請確認認證已指定且有效。</td>
  </tr>
  <tr>
    <td>403</td>
    <td>要求未經授權。 請查看詳細錯誤訊息。 這通常表示試用訂用帳戶提供的所有免費翻譯都已用完。</td>
  </tr>
  <tr>
    <td>429</td>
    <td>呼叫者傳送過多要求。</td>
  </tr>
  <tr>
    <td>500</td>
    <td>發生意外錯誤。 如果錯誤仍然存在，請回報以下資訊：失敗的日期和時間、回應標頭 `X-RequestId` 中的要求識別碼、要求標頭 `X-ClientTraceId` 中的用戶端識別碼。</td>
  </tr>
  <tr>
    <td>503</td>
    <td>暫時無法使用伺服器。 重試要求。 如果錯誤仍然存在，請回報以下資訊：失敗的日期和時間、回應標頭 `X-RequestId` 中的要求識別碼、要求標頭 `X-ClientTraceId` 中的用戶端識別碼。</td>
  </tr>
</table> 

## <a name="examples"></a>範例

下列範例會示範如何取得單一句子的句子界限。 服務會自動偵測句子的語言。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'How are you? I am fine. What did you do today?'}]"
```

---

