---
title: Microsoft 翻譯工具文字 API 語言方法 | Microsoft Docs
description: 使用 Microsoft 翻譯工具文字 API 語言方法。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: 93c06218a560faf439f05903438d021b372ce257
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369691"
---
# <a name="text-api-30-languages"></a>文字 API 3.0：語言

取得文字 API 其他作業目前支援的語言集合。 

## <a name="request-url"></a>要求 URL

將 `GET` 要求傳送至：
```HTTP
https://api.cognitive.microsofttranslator.com/languages?api-version=3.0
```

## <a name="request-parameters"></a>要求參數

在查詢字串上傳遞的要求參數為：

<table width="100%">
  <th width="20%">查詢參數</th>
  <th>說明</th>
  <tr>
    <td>api-version</td>
    <td>*必要參數*。<br/>用戶端要求的 API 版本。 值必須為 `3.0`。</td>
  </tr>
  <tr>
    <td>scope</td>
    <td>*選擇性參數*。<br/>以逗號分隔的名稱列表，用於定義要傳回的語言群組。 允許的群組名稱為：`translation`、`transliteration` 和 `dictionary`。 如果未提供任何範圍，則會傳回所有群組，相當於傳遞 `scope=translation,transliteration,dictionary`。 若想要決定哪一組支援語言適合您的案例使用，請參閱[回應物件](#response-body)的說明。</td>
  </tr>
</table> 

要求標頭為：

<table width="100%">
  <th width="20%">headers</th>
  <th>說明</th>
  <tr>
    <td>Accept-Language</td>
    <td>*選擇性要求標頭*。<br/>用於使用者介面字串的語言。 回應中的某些欄位是語言名稱或區域名稱。 您可以使用此參數來定義傳回這些名稱的語言。 提供語式正確的的 BCP 47 語言標記即可指定語言。 舉例而言，值 `fr` 可要求傳回法文名稱，使用值 `zh-Hant` 則可要求繁體中文的名稱。<br/>未指定目標語言或無法進行當地語系化時，會提供英文名稱。
    </td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*選擇性要求標頭*。<br/>用於識別唯一要求的 GUID，由用戶端產生。</td>
  </tr>
</table> 

不需要驗證便能取得語言資源。

## <a name="response-body"></a>Response body

用戶端使用 `scope` 查詢參數來定義其感興趣的語言群組。

* `scope=translation` 提供支援的語言，可將文字從某種語言翻譯成另一種語言；

* `scope=transliteration` 提供將某種語言的文字從某個字集轉換成另一個字集的功能；

* `scope=dictionary` 為要傳回資料的 `Dictionary` 作業提供語言組合。

用戶端可以指定以逗號分隔的名稱清單，藉此同時擷取多個群組。 例如，`scope=translation,transliteration,dictionary` 會傳回所有群組的支援語言。

成功的回應為 JSON 物件，每個所要求的群組都有一個屬性：

```json
{
    "translation": {
        //... set of languages supported to translate text (scope=translation)
    },
    "transliteration": {
        //... set of languages supported to convert between scripts (scope=transliteration)
    },
    "dictionary": {
        //... set of languages supported for alternative translations and examples (scope=dictionary)
    }
}
```

各屬性值如下所示。

* `translation` 屬性

  `translation` 屬性的值是 (金鑰、值) 組的字典。 每個金鑰均為 BCP 47 語言標記。 金鑰可識別文字翻譯的目標語言或來源語言。 與金鑰相關聯的值是 JSON 物件，其屬性描述了該語言：

  * `name`：顯示 `Accept-Language` 標頭所要求的地區設定語言名稱。

  * `nativeName`：以該語言的原文顯示語言名稱。

  * `dir`：文字閱讀方向，`rtl` 代表從右至左的語言，`ltr` 代表從左至右的語言。

  範例如下：
          
  ```json
  {
    "translation": {
      ...
      "fr": {
        "name": "French",
        "nativeName": "Français",
        "dir": "ltr"
      },
      ...
    }
  }
  ```

* `transliteration` 屬性

  `transliteration` 屬性的值是 (金鑰、值) 組的字典。 每個金鑰均為 BCP 47 語言標記。 金鑰可識別出能在字集間轉換的文字的語言。 與金鑰相關聯的值是 JSON 物件，其屬性描述了該語言及其支援的字集：

  * `name`：顯示 `Accept-Language` 標頭所要求的地區設定語言名稱。

  * `nativeName`：以該語言的原文顯示語言名稱。

  * `scripts`：要轉換的字集清單。 `scripts` 清單的每個元素均有屬性：

    * `code`：識別字集的代碼。

    * `name`：顯示 `Accept-Language` 標頭所要求的地區設定字集名稱。

    * `nativeName`：以該語言的原文顯示語言名稱。

    * `dir`：文字閱讀方向，`rtl` 代表從右至左的語言，`ltr` 代表從左至右的語言。

    * `toScripts`：可用於轉換文字的目標字集清單。 `toScripts` 清單的每個元素均有 `code`、`name`、`nativeName` 和 `dir` 屬性，如先前所述。

  範例如下：

  ```json
  {
    "transliteration": {
      ...
      "ja": {
        "name": "Japanese",
        "nativeName": "日本語",
        "scripts": [
          {
            "code": "Jpan",
            "name": "Japanese",
            "nativeName": "日本語",
            "dir": "ltr",
            "toScripts": [
              {
                "code": "Latn",
                "name": "Latin",
                "nativeName": "ラテン語",
                "dir": "ltr"
              }
            ]
          },
          {
            "code": "Latn",
            "name": "Latin",
            "nativeName": "ラテン語",
            "dir": "ltr",
            "toScripts": [
              {
                "code": "Jpan",
                "name": "Japanese",
                "nativeName": "日本語",
                "dir": "ltr"
              }
            ]
          }
        ]
      },
      ...
    }
  }
  ```

* `dictionary` 屬性

  `dictionary` 屬性的值是 (金鑰、值) 組的字典。 每個金鑰均為 BCP 47 語言標記。 此金鑰可識別出可提供替代翻譯和反向翻譯的語言。 此值為 JSON 物件，描述了可用翻譯的來源語言和目標語言：

  * `name`：顯示 `Accept-Language` 標頭所要求的地區設定來源語言名稱。

  * `nativeName`：以該語言的原文顯示語言名稱。

  * `dir`：文字閱讀方向，`rtl` 代表從右至左的語言，`ltr` 代表從左至右的語言。

  * `translations`：具有替代翻譯的語言清單，以及以來源語言表示的查詢範例。 `translations` 清單的每個元素均有屬性：

    * `name`：顯示 `Accept-Language` 標頭所要求的地區設定目標語言名稱。

    * `nativeName`：以目標語言的原文顯示目標語言名稱。

    * `dir`：文字閱讀方向，`rtl` 代表從右至左的語言，`ltr` 代表從左至右的語言。
    
    * `code`：識別目標語言的語言代碼。

  範例如下：

  ```json
  "es": {
    "name": "Spanish",
    "nativeName": "Español",
    "dir": "ltr",
    "translations": [
      {
        "name": "English",
        "nativeName": "English",
        "dir": "ltr",
        "code": "en"
      }
    ]
  },
  ```

如果 API 版本沒有任何變更，回應物件的結構也不會變更。 至於相同版本的 API，因為 Microsoft Translator 會持續增加其服務所支援的語言清單，可用的語言清單可能會隨時間變更。

支援的語言清單不會經常變更。 若要節省網路頻寬並改善回應能力，用戶端應用程式可考慮快取處理語言資源和對應的實體標記 (`ETag`)。 然後，用戶端應用程式可以定期 (例如每隔 24 小時一次) 查詢服務，以擷取最新的支援語言集。 傳遞 `If-None-Match` 標頭欄位中目前的 `ETag` 值，可讓服務達到最佳化的回應。 如果資源未經修改，服務會傳回狀態碼 304 和空的回應本文。

## <a name="response-headers"></a>回應標頭

<table width="100%">
  <th width="20%">headers</th>
  <th>說明</th>
  <tr>
    <td>ETag</td>
    <td>所要求之支援語言群組的實體標記目前的值。 為了使後續要求更有效率，用戶端可以在 `If-None-Match` 標頭欄位中傳送 `ETag` 值。
    </td>
  </tr>
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
    <td>304</td>
    <td>自要求標頭 `If-None-Match` 指定的版本以來，資源未經修改。</td>
  </tr>
  <tr>
    <td>400</td>
    <td>缺少其中一個查詢參數或無效。 請先修正要求參數再重試。</td>
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

以下範例顯示如何擷取支援文字轉換的語言。

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl "https://api.cognitive.microsofttranslator.com/languages?api-version=3.0&scope=translation"
```

---
