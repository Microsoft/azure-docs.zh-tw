---
title: 翻譯工具語音 API 語言方法
titleSuffix: Azure Cognitive Services
description: 翻譯工具語音 API 語言方法。
services: cognitive-services
author: Jann-Skotdal
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: conceptual
ms.date: 05/18/18
ms.author: v-jansko
ms.openlocfilehash: b7005811898df9132be6bc199e26f6c6dc358618
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49345198"
---
# <a name="translator-speech-api-languages"></a>翻譯工具語音 API：語言

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-translator-speech-deprecation-note.md)]

翻譯工具語音持續擴充其服務所支援的語言清單。 使用此 API 來探索目前可和翻譯工具語音服務搭配使用的語言集合。

示範使用 API 取得可用語言的程式碼範例，可在 [Microsoft Translator Github 網站](https://github.com/MicrosoftTranslator) \(英文\) 取得。

## <a name="implementation-notes"></a>實作附註

GET /languages 

廣泛的語言集合可供繕寫語音、翻譯繕寫的文字，以及產生翻譯的合成語音。

用戶端使用 `scope` 查詢參數來定義其感興趣的語言集合。

* **語音轉換文字：** 使用查詢參數 `scope=speech` 來擷取可供將語音繕寫成文字的語言集合。
* **文字翻譯：** 使用查詢參數 `scope=text` 來擷取可供翻譯繕寫之文字的語言集合。
* **文字轉換語音：** 使用查詢參數 `scope=tts` 來擷取可供將翻譯的文字合成回語音的語言和語音集合。

用戶端可以透過指定所選的逗號分隔清單，同時擷取多個集合。 例如： `scope=speech,text,tts`。

成功的回應是 JSON 物件，其中的屬性包含每個所要求集合。

```
{
    "speech": {
        //... Set of languages supported for speech-to-text
    },
    "text": {
        //... Set of languages supported for text translation
    },
    "tts": {
        //... Set of languages supported for text-to-speech
    }
}
```

因為用戶端可以使用 `scope` 查詢參數來選取應該要傳回哪些支援的語言集合，因此實際回應可能只包含上方顯示所有屬性的子集。

隨每個屬性提供的值如下所示。

### <a name="speech-to-text-speech"></a>語音轉換文字 (speech)

與語音轉換文字屬性 (`speech`) 相關聯的值是 (索引鍵，值) 組字典。 每個索引鍵都識別語音轉換文字支援的語言。 索引鍵是用戶端傳遞至 API 的識別碼。 與索引鍵相關聯的值是具有以下屬性的物件：

* `name`：語言的顯示名稱。
* `language`：相關書寫語言的語言標籤。 請參閱下面的＜文字翻譯＞。
範例如下：

```
{
    "speech": {
        "en-US": { name: "English", language: "en" }
    }
}
```

### <a name="text-translation-text"></a>文字翻譯 (text)

與 `text` 屬性相關聯的值也是字典，其中每個索引鍵都識別文字翻譯支援的語言。 與索引鍵相關聯的值描述該語言：

* `name`：語言的顯示名稱。
* `dir`：方向，對於從右至左的語言是 `rtl`，或對於從左至右的語言是 `ltr`。

範例如下：

```
{
    "text": {
        "en": { name: "English", dir: "ltr" }
    }
}
```

### <a name="text-to-speech-tts"></a>文字轉換語音 (tts)

與文字轉換語音屬性 (tts) 相關聯的值也是字典，其中每個索引鍵都識別支援的語音。 語音物件的屬性如下：

* `displayName`：語音的顯示名稱。
* `gender`：語音的性別 (男性或女性)。
* `locale`：語音的語言標籤，包含主要語言子標籤和區域子標籤。
* `language`：相關書寫語言的語言標籤。
* `languageName`：語言的顯示名稱。
* `regionName`：此語言之區域的顯示名稱。

範例如下：

```
{
    "tts": {
        "en-US-Zira": {
            "displayName": "Zira",
            "gender": "female",
            "locale": "en-US",
            "languageName": "English",
            "regionName": "United States",
            "language": "en"
        }
}
```

### <a name="localization"></a>當地語系化
服務針對所有在文字翻譯中支援的語言，傳回 'Accept-Language' 標頭的語言中的所有名稱。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
描述支援之語言集合的物件。

ModelExample 值： 

Langagues { speech (object, optional), text (object, optional), tts (object, optional) }

### <a name="headers"></a>headers

|頁首|說明|類型|
:--|:--|:--|
X-RequestId|伺服器產生用來識別要求的值，並作為疑難排解之用。|字串|

### <a name="parameters"></a>參數

|參數|說明|參數類型|資料類型|
|:--|:--|:--|:--|
|api-version    |用戶端要求的 API 版本。 允許的值包括：`1.0`。|query|字串|
|scope  |要傳回至用戶端的支援語言或語音集合。 此參數是以關鍵字的逗號分隔清單來指定。 可使用以下關鍵字：<ul><li>`speech`：提供支援繕寫語音的語言集合。</li><li>`tts`：提供支援文字語音轉換的語音集合。</li><li>`text`：提供支援翻譯文字的語言集合。</li></ul>如果沒有指定值，則 `scope` 的值預設為 `text`。|query|字串|
|X-ClientTraceId    |用來追蹤要求的用戶端產生 GUID。 為了協助對問題進行疑難排解，用戶端應該隨著每個要求提供新的值並予以記錄。|頁首|字串|
|Accept-Language    |回應中的某些欄位是語言或區域名稱。 使用此參數來定義傳回這些名稱的語言。 語言是透過提供語式正確的 BCP 47 語言標記來指定的。 從使用 `text` 範圍傳回的語言識別碼清單選取標籤。 對於不支援的語言，會以英文提供其名稱。<br/>舉例而言，值 `fr` 可要求傳回法文名稱，使用值 `zh-Hant` 則可要求繁體中文的名稱。|頁首|字串|
    
### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400|不正確的要求。 請檢查輸入參數，以確保參數有效。 回應物件包括錯誤的更詳細描述。|
|429|太多要求。|
|500|發生錯誤。 若錯誤仍然存在，請使用用戶端追蹤識別碼 (X-ClientTraceId) 或要求識別碼 (X-RequestId) 予以回報。|
|503|暫時無法使用伺服器。 請重試要求。 若錯誤仍然存在，請使用用戶端追蹤識別碼 (X-ClientTraceId) 或要求識別碼 (X-RequestId) 予以回報。|
