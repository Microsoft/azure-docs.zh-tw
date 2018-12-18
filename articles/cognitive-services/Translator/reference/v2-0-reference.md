---
title: Microsoft Translator Text API v2.0 參考 | Microsoft Docs
titleSuffix: Cognitive Services
description: Microsoft Translator Text API v2.0 參考文件。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 05/15/2018
ms.author: v-jansko
ms.openlocfilehash: e32e28608d2fecf27b61acff74af7eb6849f0ba1
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370354"
---
# <a name="translator-text-api-v20"></a>Translator Text API v2.0

> [!IMPORTANT]
> 這版 Translator Text API 已淘汱。 [檢視 Translator Text API v3文件](v3-0-reference.md)。

Microsoft Translator Text API V2 可以無縫整合到您的應用程式、網站、工具或其他解決方案，以提供多語系使用者體驗。 利用業界標準，它可以用於任何硬體平台，並搭配任何作業系統執行語言翻譯和其他語言相關作業，例如文字語言偵測或文字轉換語音。 如需 Microsoft Translator API 的詳細資訊，請按一下這裡。

## <a name="getting-started"></a>開始使用
若要存取 Microsoft Translator Text API，您需要[註冊 Microsoft Azure](../translator-text-how-to-signup.md)。

## <a name="authorization"></a>Authorization
所有 Microsoft Translator Text API 呼叫都需要驗證訂用帳戶金鑰。 API 支援兩種驗證模式：

* 使用存取權杖。 使用**步驟** 9 中所參考的訂用帳戶金鑰，以對授權服務提出 POST 要求來產生存取權杖。 如需詳細資料，請參閱權杖服務文件。 使用 Authorization 標頭或 access_token 查詢參數，將存取權杖傳遞給 Translator 服務。 存取權杖的有效時間為 10 分鐘。 每隔 10 分鐘取得新的存取權杖一次，並在這 10 分鐘內針對重複的要求持續使用相同的存取權杖。

* 直接使用訂用帳戶金鑰。 以值形式將要求所含之 `Ocp-Apim-Subscription-Key` 標頭中的訂用帳戶金鑰傳遞給 Translator API。 在此模式中，您不需要呼叫驗證權杖服務來產生存取權杖。

請考慮使用您的訂用帳戶金鑰和存取權杖作為應該隱藏不予檢視的祕密。

## <a name="profanity-handling"></a>粗話處理
Translator 服務通常會在翻譯中保留存在於來源的粗話。 粗話程度以及產生不雅字眼的內容在文化特性之間會不同，因此目標語言中的粗話程度可能會加大或減少。

若您想要在翻譯中避免粗話，則不論來源文字中是否有粗話，都可以針對支援它的方法使用粗話篩選選項。 此選項可讓您選擇要查看刪除的粗話或標記為適當的標籤，還是不採取任何動作。 接受的 `ProfanityAction` 值為 `NoAction` (預設)、Marked 和 `Deleted`。


|ProfanityAction    |動作 |範例來源 (日文)  |範例翻譯 (英文)  |
|:--|:--|:--|:--|
|NoAction   |預設值。 與未設定此選項相同。 粗話會從來源傳遞到目標。        |彼はジャッカスです。     |He is a jackass.   |
|Marked     |不雅字眼會以 XML 標籤 <profanity> 和 </profanity> 括住。     |彼はジャッカスです。 |He is a <profanity>jackass</profanity>.    |
|Deleted    |從輸出中移除不雅字眼，且不予取代。     |彼はジャッカスです。 |He is a.   |

    
## <a name="excluding-content-from-translation"></a>從翻譯中排除內容
翻譯具有 HTML (`contentType=text/html`) 這類標籤的內容時，從翻譯中排除特定內容有時十分有用。 您可以使用 `class=notranslate` 屬性，指定應該保留其原始語言的內容。 在下列範例中，不會翻譯第一個 `div` 項目內的內容，但會翻譯第二個 `div` 項目中的內容。

```HTML
<div class="notranslate">This will not be translated.</div>
<div>This will be translated. </div>
```

## <a name="get-translate"></a>GET /Translate

### <a name="implementation-notes"></a>實作附註
將文字字串從某種語言翻譯成另一種語言。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/Translate`。

**傳回值：** 字串，代表翻譯文字。

若您先前針對相同的來源句子使用 `AddTranslation` 或 `AddTranslationArray` 輸入評分等於或大於 5 的翻譯，則 `Translate` 只會傳回最適合您系統的選擇。 「相同的來源句子」表示除了大小寫、空格、標籤值以及句尾的標點符號之外完全相同 (100% 相符)。 若未儲存評分等於或大於 5 的評分，則 Microsoft Translator 會自動翻譯傳回的結果。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)

字串

回應內容類型：application/xml 

### <a name="parameters"></a>參數

|參數|值|說明    |參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid  |(空白)    |必要。 若使用 Authorization 或 Ocp-Apim-Subscription-Key 標頭，請將 appid 欄位空白，或是包括內含 "Bearer" + " " + "access_token" 的字串。|query|字串|
|text|(空白)   |必要。 字串，代表要翻譯的文字。 文字大小不得超過 10000 個字元。|query|字串|
|from|(空白)   |選用。 字串，代表翻譯文字的語言代碼。 例如，en 代表「英文」。|query|字串|
|to|(空白) |必要。 字串，代表目標翻譯文字的語言代碼。|query|字串|
|contentType|(空白)    |選用。 要翻譯文字的格式。 支援的格式為 text/plain (預設) 和 text/html。 任何 HTML 都需要格式正確的完整項目。|query|字串|
|category|(空白)   |選用。 字串，包含翻譯的分類 (網域)。 預設為 "general"。|query|字串|
|Authorization|(空白)  |若未指定 appid 欄位或 Ocp-Apim-Subscription-Key 標頭，則為必要項目。 授權權杖："Bearer" + " " + "access_token"。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)  |若未指定 appid 欄位或 Authorization 標頭，則為必要項目。|頁首|字串|


### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 X-MS-Trans-Info 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="post-translatearray"></a>POST /TranslateArray

### <a name="implementation-notes"></a>實作附註
使用 `TranslateArray` 方法來擷取多個來源文字的翻譯。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/TranslateArray`。

要求本文的格式應該如下：

```
<TranslateArrayRequest>
  <AppId />
  <From>language-code</From>
  <Options>
    <Category xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2" >string-value</Category>
    <ContentType xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">text/plain</ContentType>
    <ReservedFlags xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2" />
    <State xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2" >int-value</State>
    <Uri xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2" >string-value</Uri>
    <User xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2" >string-value</User>
  </Options>
  <Texts>
    <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/Arrays">string-value</string>
    <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/Arrays">string-value</string>
  </Texts>
  <To>language-code</To>
</TranslateArrayRequest>
```

`TranslateArrayRequest` 內的項目為：


* `appid`：必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。
* `from`：選擇性。 字串，代表來源翻譯文字的語言代碼。 若保留空白，則回應將會包括語言自動偵測的結果。
* `options`：選擇性。 `Options` 物件，包含下面列出的值。 它們都是選擇性項目，且預設為最常見設定。 指定的項目必須以字母順序列出。
    - `Category`：字串，包含翻譯的分類 (網域)。 預設為 `general`。
    - `ContentType`：要翻譯文字的格式。 支援的格式為 `text/plain` (預設)、`text/xml` 和 `text/html`。 任何 HTML 都需要格式正確的完整項目。
    - `ProfanityAction`：指定如何處理粗語，如上所示。 接受的 `ProfanityAction` 值為 `NoAction` (預設)、`Marked` 和 `Deleted`。
    - `State`：使用者狀態，協助建立要求與回應的關聯。 將在回應中傳回相同的內容。
    - `Uri`：依此 URI 篩選結果。 預設：`all`。
    - `User`：依此使用者篩選結果。 預設：`all`。
* `texts`：必要。 陣列，包含翻譯的文字。 所有字串都必須是相同的語言。 要翻譯之所有文字的總數不得超過 10000 個字元。 最大陣列項目數是 2000。
* `to`：必要。 字串，代表目標翻譯文字的語言代碼。

可以省略選擇性項目。 為 TranslateArrayRequest 直接子系的項目必須以字母順序列出。

TranslateArray 方法會接受 `Content-Type` 的 `application/xml` 或 `text/xml`。

**傳回值：**`TranslateArrayResponse` 陣列。 每個 `TranslateArrayResponse` 都會有下列項目：

* `Error`：指出發生其中一種情況時發生錯誤。 否則設定為 null。
* `OriginalSentenceLengths`：整數陣列，指出原始來源文字中每個句子的長度。 陣列長度，指出句子數目。
* `TranslatedText`：翻譯的文字。
* `TranslatedSentenceLengths`：整數陣列，指出翻譯文字中每個句子的長度。 陣列長度，指出句子數目。
* `State`：使用者狀態，協助建立要求與回應的關聯。 傳回與要求中相同的內容。

回應本文的格式如下。

```
<ArrayOfTranslateArrayResponse xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2"
  xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
  <TranslateArrayResponse>
    <From>language-code</From>
    <OriginalTextSentenceLengths xmlns:a="http://schemas.microsoft.com/2003/10/Serialization/Arrays">
      <a:int>int-value</a:int>
    </OriginalTextSentenceLengths>
    <State/>
    <TranslatedText>string-value</TranslatedText>
    <TranslatedTextSentenceLengths xmlns:a="http://schemas.microsoft.com/2003/10/Serialization/Arrays">
      <a:int>int-value</a:int>
    </TranslatedTextSentenceLengths>
  </TranslateArrayResponse>
</ArrayOfTranslateArrayResponse>
```

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
成功的回應會包括具有上述格式的 `TranslateArrayResponse` 陣列。

字串

回應內容類型：application/xml

### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|Authorization|(空白) |若未指定 appid 欄位或 Ocp-Apim-Subscription-Key 標頭，則為必要項目。 授權權杖："Bearer" + " " + "access_token"。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 appid 欄位或 Authorization 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼   |原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。 常見錯誤包括： <ul><li>陣列項目不可空白</li><li>分類無效</li><li>From 語言無效</li><li>To 語言無效</li><li>要求包含太多項目</li><li>From 語言不受支援</li><li>To 語言不受支援</li><li>翻譯要求有太多資料</li><li>HTML 的格式不正確</li><li>翻譯要求中傳遞的字串太多</li></ul>|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 X-MS-Trans-Info 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="post-getlanguagenames"></a>POST /GetLanguageNames

### <a name="implementation-notes"></a>實作附註
擷取下列語言的易記名稱：傳入為 `languageCodes` 參數的語言，以及使用傳遞之地區設定語言當地語系化的語言。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/GetLanguageNames`。

要求本文包含字串陣列，代表要擷取其易記名稱的 ISO 639-1 語言代碼。 例如︰

```
<ArrayOfstring xmlns:i="http://www.w3.org/2001/XMLSchema-instance"  xmlns="http://schemas.microsoft.com/2003/10/Serialization/Arrays">
    <string>zh</string>
    <string>en</string>
</ArrayOfstring>
```

**傳回值：** 字串陣列，包含 Translator 服務所支援且當地語系化為所要求語言的語言名稱。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
字串陣列，包含 Translator 服務所支援且當地語系化為所要求語言的語言名稱。

字串

回應內容類型：application/xml
 
### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)|必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|地區設定|(空白) |必要。 字串，代表與語言建立關聯的 ISO 639 兩個字母小寫文化特性代碼以及 ISO 3166 兩個字母大寫子文化特性代碼的組合，自行當地語系化語言名稱或 ISO 639 小寫文化特性代碼。|query|字串|
|Authorization|(空白)  |若未指定 appid 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)  |若未指定 appid 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 X-MS-Trans-Info 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="get-getlanguagesfortranslate"></a>GET /GetLanguagesForTranslate

### <a name="implementation-notes"></a>實作附註
取得語言代碼清單，代表 Translation 服務所支援的語言。  `Translate` 和 `TranslateArray` 可以翻譯這些語言的任兩種。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/GetLanguagesForTranslate`。

**傳回值：** 字串陣列，包含 Translator 服務所支援的語言代碼。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
字串陣列，包含 Translator 服務所支援的語言代碼。

字串

回應內容類型：application/xml
 
### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)|必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|Authorization|(空白)  |若未指定 `appid` 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 `appid` 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 X-MS-Trans-Info 中。|
|503|服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="get-getlanguagesforspeak"></a>GET /GetLanguagesForSpeak

### <a name="implementation-notes"></a>實作附註
擷取提供語音合成功能的語言種類。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/GetLanguagesForSpeak`。

**傳回值：** 字串陣列，包含 Translator 服務之語音合成所支援的語言代碼。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
字串陣列，包含 Translator 服務之語音合成所支援的語言代碼。

字串

回應內容類型：application/xml

### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)|必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|Authorization|(空白)|若未指定 `appid` 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 `appid` 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|
 
### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400|錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401|認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 `X-MS-Trans-Info` 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="get-speak"></a>GET /Speak

### <a name="implementation-notes"></a>實作附註
傳回以所需語言說出之傳入文字的 wave 或 mp3 串流。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/Speak`。

**傳回值：** 以所需語言說出之傳入文字的 wave 或 mp3 串流。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)

binary

回應內容類型：application/xml 

### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)|必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|text|(空白)   |必要。 字串，包含要針對 wave 串流說出之指定語言的一或多個句子。 要說出的文字大小不得超過 2000 個字元。|query|字串|
|語言|(空白)   |必要。 字串，代表支援用來說出文字的語言代碼。 程式碼必須存在於從 `GetLanguagesForSpeak` 方法所傳回的程式碼清單。|query|字串|
|format|(空白)|選用。 字串，指定 Content-Type 識別碼。 目前可以使用 `audio/wav` 和 `audio/mp3`。 預設值為 `audio/wav`。|query|字串|
|options|(空白)    |<ul><li>選用。 字串，指定合成語音的內容：<li>`MaxQuality` 和 `MinSize` 可用來指定音訊訊號品質。 使用 `MaxQuality`，您可以取得最高品質的語音，而使用 `MinSize`，您可以取得最小大小的語音。 預設為 `MinSize`。</li><li>`female` 和 `male` 可用來指定語音的所需性別。 預設值為 `female`。 使用分隔號 `|` to include multiple options. For example  `MaxQuality|Male`。</li></li></ul> |query|字串|
|Authorization|(空白)|若未指定 `appid` 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)  |若未指定 `appid` 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 `X-MS-Trans-Info` 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="get-detect"></a>GET /Detect

### <a name="implementation-notes"></a>實作附註
使用 `Detect` 方法，識別所選取一段文字的語言。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/Detect`。

**傳回值：** 字串，包含所指定文字的二字元語言代碼。 .

### <a name="response-class-status-200"></a>回應類別 (狀態 200)

字串

回應內容類型：application/xml

### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)  |必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|text|(空白)|必要。 字串，包含要識別其語言的一些文字。 文字大小不得超過 10000 個字元。|query| 字串|
|Authorization|(空白)|若未指定 `appid` 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|頁首|字串|
|Ocp-Apim-Subscription-Key  |(空白)    |若未指定 `appid` 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400|錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 `X-MS-Trans-Info` 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|


## <a name="post-detectarray"></a>POST /DetectArray

### <a name="implementation-notes"></a>實作附註
使用 `DetectArray` 方法，識別字串陣列的語言一次。 執行每個個別陣列項目的獨立偵測，並傳回陣列中每個資料列的結果。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/DetectArray`。

要求本文的格式應該如下。

```
<ArrayOfstring xmlns="http://schemas.microsoft.com/2003/10/Serialization/Arrays">
    <string>string-value-1</string>
    <string>string-value-2</string>
</ArrayOfstring>
```

文字大小不得超過 10000 個字元。

**傳回值：** 字串陣列，包含輸入陣列每個資料列的二字元語言代碼。

回應本文的格式如下。

```
<ArrayOfstring xmlns="http://schemas.microsoft.com/2003/10/Serialization/Arrays" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
  <string>language-code-1</string>
  <string>language-code-2</string>
</ArrayOfstring>
```

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
DetectArray 成功。 傳回字串陣列，包含輸入陣列每個資料列的二字元語言代碼。

字串

回應內容類型：application/xml
 
### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)|必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|Authorization|(空白)|若未指定 `appid` 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 `appid` 欄位或 Authorization 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 X-MS-Trans-Info 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="get-addtranslation"></a>GET /AddTranslation

### <a name="implementation-notes"></a>實作附註

> [!IMPORTANT]
> **淘汱附註：** 在 2018 年 1 月 31 日之後，此方法將不會接受新的句子提交，而且您會收到錯誤訊息。 如需共同作業翻譯函式的變更，請參閱本宣告。

將翻譯新增至翻譯記憶體。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/AddTranslation`。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)

字串

回應內容類型：application: xml
 
### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型   |
|:--|:--|:--|:--|:--|
|appid|(空白)|必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|originalText|(空白)|必要。 字串，包含要翻譯的來源文字。 字串長度上限為 1000 個字元。|query|字串|
|translatedText|(空白) |必要。 字串，包含目標語言的翻譯文字。 字串長度上限為 2000 個字元。|query|字串|
|from|(空白)   |必要。 字串，代表翻譯文字的語言代碼。 en = english, de = german etc...|query|字串|
|to|(空白)|必要。 字串，代表目標翻譯文字的語言代碼。|query|字串|
|rating|(空白) |選用。 整數，代表此字串的品質評分。 -10 與 10 之間的值。 預設值為 1。|query|integer|
|contentType|(空白)    |選用。 要翻譯文字的格式。 支援的格式為 "text/plain" 和 "text/html"。 任何 HTML 都需要格式正確的完整項目。   |query|字串|
|category|(空白)|選用。 字串，包含翻譯的分類 (網域)。 預設為 "general"。|query|字串|
|user|(空白)|必要。 用來追蹤提交建立者的字串。|query|字串|
|uri|(空白)|選用。 字串，包含此翻譯的內容位置。|query|字串|
|Authorization|(空白)|若未指定 appid 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。    |頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 `appid` 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|410|不再支援 AddTranslation。|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 X-MS-Trans-Info 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="post-addtranslationarray"></a>POST /AddTranslationArray

### <a name="implementation-notes"></a>實作附註

> [!IMPORTANT]
> **淘汱附註：** 在 2018 年 1 月 31 日之後，此方法將不會接受新的句子提交，而且您會收到錯誤訊息。 如需共同作業翻譯函式的變更，請參閱本宣告。

新增翻譯陣列，以新增翻譯記憶體。 這是 `AddTranslation` 的陣列版本。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/AddTranslationArray`。

要求本文的格式應該如下。

```
<AddtranslationsRequest>
  <AppId></AppId>
  <From>A string containing the language code of the source language</From>
  <Options>
    <Category xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">string-value</Category>
    <ContentType xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">text/plain</ContentType>
    <User xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">string-value</User>
    <Uri xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">string-value</Uri>
  </Options>
  <To>A string containing the language code of the target language</To>
  <Translations>
    <Translation xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">
      <OriginalText>string-value</OriginalText>
      <Rating>int-value</Rating>
      <Sequence>int-value</Sequence>
      <TranslatedText>string-value</TranslatedText>
    </Translation>
  </Translations>
</AddtranslationsRequest>
```

AddtranslationsRequest 項目內的項目為：

* `AppId`：必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。
* `From`：必要。 字串，包含來源語言的語言代碼。 必須是 `GetLanguagesForTranslate` 方法所傳回的其中一種語言。
* `To`：必要。 字串，包含目標語言的語言代碼。 必須是 `GetLanguagesForTranslate` 方法所傳回的其中一種語言。
* `Translations`：必要。 要新增至翻譯記憶體的翻譯陣列。 每個翻譯都必須包含：originalText、translatedText 和 rating。 每個 originalText 和 translatedText 的大小限制為 1000 個字元。 所有 originalText 和 translatedText 的總數不得超過 10000 個字元。 最大陣列項目數是 100。
* `Options`：必要。 一組選項，包括 Category、ContentType、Uri 和 User。 需要使用者。 Category、ContentType 和 Uri 是選擇性項目。 指定的項目必須以字母順序列出。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
AddTranslationArray 方法成功。 在 2018 年 1 月 31 日之後，將不會接受句子提交。 服務將會回應錯誤碼 410。

字串

回應內容類型：application/xml
 
### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|Authorization|(空白)|若未指定 appid 欄位或 Ocp-Apim-Subscription-Key 標頭，則為必要項目。 授權權杖："Bearer" + " " + "access_token"。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 appid 欄位或 Authorization 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|410    |不再支援 AddTranslation。|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 `X-MS-Trans-Info` 中。|
|503|服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="get-breaksentences"></a>GET /BreakSentences

### <a name="implementation-notes"></a>實作附註
將一段文字分成多個句子，並傳回包含每個句子中長度的陣列。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/BreakSentences`。

**傳回值：** 整數陣列，代表句子長度。 陣列長度就是句子數目，而值是每個句子的長度。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
整數陣列，代表句子長度。 陣列長度就是句子數目，而值是每個句子的長度。

integer

回應內容類型：application/xml 

### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)  |必要。 若使用 Authorization 或 Ocp-Apim-Subscription-Key 標頭，請將 appid 欄位空白，或是包括內含 "Bearer" + " " + "access_token" 的字串。|query| 字串|
|text|(空白)   |必要。 字串，代表要分割為句子的文字。 文字大小不得超過 10000 個字元。|query|字串|
|語言   |(空白)    |必要。 字串，代表輸入文字的語言代碼。|query|字串|
|Authorization|(空白)|若未指定 appid 欄位或 Ocp-Apim-Subscription-Key 標頭，則為必要項目。 授權權杖："Bearer" + " " + "access_token"。    |頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 appid 欄位或 Authorization 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400|錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401|認證無效|
|500|伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 X-MS-Trans-Info 中。|
|503|服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="post-gettranslations"></a>POST /GetTranslations

### <a name="implementation-notes"></a>實作附註
從存放區和 MT 引擎中擷取指定語言組的翻譯陣列。 GetTranslations 與 Translate 不同，因為它會傳回所有可用的翻譯。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/GetTranslations`。

要求的本文會包括具有下列格式的選擇性 TranslationOptions 物件。

```
<TranslateOptions xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">
  <Category>string-value</Category>
  <ContentType>text/plain</ContentType>
  <ReservedFlags></ReservedFlags>
  <State>int-value</State>
  <Uri>string-value</Uri>
  <User>string-value</User>
</TranslateOptions>
```

`TranslateOptions` 物件，包含下面列出的值。 它們都是選擇性項目，且預設為最常見設定。 指定的項目必須以字母順序列出。

* `Category`：字串，包含翻譯的分類 (網域)。 預設為 "general"。
* `ContentType`：唯一支援且為預設值的選項是 "text/plain"。
* `IncludeMultipleMTAlternatives`：布林值旗標，判斷是否應該從 MT 引擎傳回數個替代項目。 有效值為 true 和 false (區分大小寫)。 預設為 false，而且只包含 1 個替代項目。 將此旗標設定為 true 允許在翻譯中產生人造替代項目，以與共同作業翻譯架構 (CTF) 完全整合。 此功能允許傳回 CTF 中沒有替代項目之句子的替代項目，方法是從 n 最佳解碼器清單新增人造替代項目。
    - 評分：評分套用如下：1) 最佳自動翻譯的評分為 5。 2) 來自 CTF 的替代項目會反映檢閱者的授權單位，從 -10 到 +10。 3) 自動產生的 (n 最佳) 翻譯替代項目具有評分 0，而且相符程度 100。
    - 替代項目數目：傳回的替代項目數目最多為 maxTranslations，但可能更少。
    - 語言組：此功能不適用於簡體中文與繁體中文之間的翻譯 (雙向)。 這適用於所有其他 Microsoft Translator 支援的語言組。
* `State`：使用者狀態，協助建立要求與回應的關聯。 將在回應中傳回相同的內容。
* `Uri`：依此 URI 篩選結果。 若未設定任何值，則預設為 all。
* `User`：依此使用者篩選結果。 若未設定任何值，則預設為 all。

要求 `Content-Type` 應該是 `text/xml`。

**傳回值：** 回應的格式如下。

```
<GetTranslationsResponse xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2"
  xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
  <From>Two character language code</From>
  <State/>
  <Translations>
    <TranslationMatch>
      <Count>int-value</Count>
      <MatchDegree>int-value</MatchDegree>
      <MatchedOriginalText/>
      <Rating>int value</Rating>
      <TranslatedText>string-value</TranslatedText>
    </TranslationMatch>
  </Translations>
</GetTranslationsResponse>
```

這包括包含下列值的 `GetTranslationsResponse` 項目：

* `Translations`：找到的相符項陣列，儲存在 TranslationMatch (請參閱下面) 物件。 翻譯可能包括原始文字的些微變化 (模糊比對)。 將會排序翻譯：前面是 100% 相符項目，後面是模糊相符項目。
* `From`：若方法未指定 From 語言，則這會是自動語言偵測的結果。 否則，它會是指定 From 語言。
* `State`：使用者狀態，協助建立要求與回應的關聯。 包含 TranslateOptions 參數中指定的相同值。

TranslationMatch 物件由下列項目組成：

* `Error`：若特定輸入字串發生錯誤，則會儲存錯誤碼。 否則此欄位會空白。
* `MatchDegree`：系統會比對輸入句子與存放區 (包括未完全相符的項目)。  MatchDegree 指出輸入文字與存放區中找到之原始文字的相符程度。 所傳回值的範圍從 0 到 100，其中 0 是無相似度，而 100 是區分大小寫完全相符。
MatchedOriginalText：此結果相符的原始文字。 只有在相符的原始文字與輸入文字不同時才傳回。 用來傳回模糊比對的來源文字。 針對 Microsoft Translator 結果不予傳回。
* `Rating`：指出做出品質決策之人員的授權單位。 機器翻譯結果將會有評分 5。 匿名提供的翻譯一般會有 1 到 4 的評分，而授權提供的翻譯一般會有 6 到 10 的評分。
* `Count`：已選取具有此評分之這個翻譯的次數。 自動翻譯回應的值將會是 0。
* `TranslatedText`：翻譯的文字。

### <a name="response-class-status-200"></a>回應類別 (狀態 200)
具有上述格式的 `GetTranslationsResponse` 物件。

字串

回應內容類型：application/xml
 
### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|appid|(空白)|必要。 若使用 `Authorization` 或 `Ocp-Apim-Subscription-Key` 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。|query|字串|
|text|(空白)|必要。 字串，代表要翻譯的文字。 文字大小不得超過 10000 個字元。|query|字串|
|from|(空白)|必要。 字串，代表翻譯文字的語言代碼。|query|字串|
|to |(空白)    |必要。 字串，代表目標翻譯文字的語言代碼。|query|字串|
|maxTranslations|(空白)|必要。 整數，代表要傳回的最大翻譯數目。|query|integer|
|Authorization| (空白)|若未指定 `appid` 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|字串| 頁首|
|Ocp-Apim-Subscription-Key|(空白)  |若未指定 `appid` 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 `X-MS-Trans-Info` 中。|
|503|服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="post-gettranslationsarray"></a>POST /GetTranslationsArray

### <a name="implementation-notes"></a>實作附註
使用 `GetTranslationsArray` 方法來擷取多個來源文字的多個翻譯候選項目。

要求 URI 為 `https://api.microsofttranslator.com/V2/Http.svc/GetTranslationsArray`。

要求本文的格式應該如下。

```
<GetTranslationsArrayRequest>
  <AppId></AppId>
  <From>language-code</From>
  <MaxTranslations>int-value</MaxTranslations>
  <Options>
    <Category xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">string-value</Category>
    <ContentType xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">text/plain</ContentType>
    <ReservedFlags xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2"/>
    <State xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">int-value</State>
    <Uri xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">string-value</Uri>
    <User xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2">string-value</User>
  </Options>
  <Texts>
    <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/Arrays">string-value</string>
  </Texts>
  <To>language-code</To>
</GetTranslationsArrayRequest>
```

`GetTranslationsArrayRequest` 包括下列項目：

* `AppId`：必要。 若使用 Authorization 標頭，請將 appid 欄位空白，或是包括內含 `"Bearer" + " " + "access_token"` 的字串。
* `From`：必要。 字串，代表翻譯文字的語言代碼。
* `MaxTranslations`：必要。 整數，代表要傳回的最大翻譯數目。
* `Options`：選擇性。 Options 物件，包含下面列出的值。 它們都是選擇性項目，且預設為最常見設定。 指定的項目必須以字母順序列出。
    - Category`：字串，包含翻譯的分類 (網域)。 預設為 general。
    - `ContentType`：唯一支援且為預設值的選項是 text/plain。
    - `IncludeMultipleMTAlternatives`：布林值旗標，判斷是否應該從 MT 引擎傳回數個替代項目。 有效值為 true 和 false (區分大小寫)。 預設為 false，而且只包含 1 個替代項目。 將此旗標設定為 true 允許在翻譯中產生人造替代項目，以與共同作業翻譯架構 (CTF) 完全整合。 此功能允許傳回 CTF 中沒有替代項目之句子的替代項目，方法是從 n 最佳解碼器清單新增人造替代項目。
        - 評分：評分套用如下：1) 最佳自動翻譯的評分為 5。 2) 來自 CTF 的替代項目會反映檢閱者的授權單位，從 -10 到 +10。 3) 自動產生的 (n 最佳) 翻譯替代項目具有評分 0，而且相符程度 100。
        - 替代項目數目：傳回的替代項目數目最多為 maxTranslations，但可能更少。
        - 語言組：此功能不適用於簡體中文與繁體中文之間的翻譯 (雙向)。 這適用於所有其他 Microsoft Translator 支援的語言組。
* `State`：使用者狀態，協助建立要求與回應的關聯。 將在回應中傳回相同的內容。
* `Uri`：依此 URI 篩選結果。 若未設定任何值，則預設為 all。
* `User`：依此使用者篩選結果。 若未設定任何值，則預設為 all。
* `Texts`：必要。 陣列，包含翻譯的文字。 所有字串都必須是相同的語言。 要翻譯之所有文字的總數不得超過 10000 個字元。 最大陣列項目數是 10。
* `To`：必要。 字串，代表目標翻譯文字的語言代碼。

可以省略選擇性項目。 為 `GetTranslationsArrayRequest` 直接子系的項目必須以字母順序列出。

要求 `Content-Type` 應該是 `text/xml`。

**傳回值：** 回應的格式如下。

```
<ArrayOfGetTranslationsResponse xmlns="http://schemas.datacontract.org/2004/07/Microsoft.MT.Web.Service.V2" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
  <GetTranslationsResponse>
    <From>language-code</From>
    <State/>
    <Translations>
      <TranslationMatch>
        <Count>int-value</Count>
        <MatchDegree>int-value</MatchDegree>
        <MatchedOriginalText>string-value</MatchedOriginalText>
        <Rating>int-value</Rating>
        <TranslatedText>string-value</TranslatedText>
      </TranslationMatch>
      <TranslationMatch>
        <Count>int-value</Count>
        <MatchDegree>int-value</MatchDegree>
        <MatchedOriginalText/>
        <Rating>int-value</Rating>
        <TranslatedText>string-value</TranslatedText>
      </TranslationMatch>
    </Translations>
  </GetTranslationsResponse>
</ArrayOfGetTranslationsResponse>
```

每個 `GetTranslationsResponse` 項目都包含下列值：

* `Translations`：找到的相符項陣列，儲存在 `TranslationMatch` (請參閱下面) 物件。 翻譯可能包括原始文字的些微變化 (模糊比對)。 將會排序翻譯：前面是 100% 相符項目，後面是模糊相符項目。
* `From`：若方法未指定 `From` 語言，則這會是自動語言偵測的結果。 否則，它會是指定 From 語言。
* `State`：使用者狀態，協助建立要求與回應的關聯。 包含 `TranslateOptions` 參數中指定的相同值。

`TranslationMatch` 物件由下列項目組成：
* `Error`：若特定輸入字串發生錯誤，則會儲存錯誤碼。 否則此欄位會空白。
* `MatchDegree`：系統會比對輸入句子與存放區 (包括未完全相符的項目)。  `MatchDegree` 指出輸入文字與存放區中找到之原始文字的相符程度。 所傳回值的範圍從 0 到 100，其中 0 是無相似度，而 100 是區分大小寫完全相符。
* `MatchedOriginalText`：此結果相符的原始文字。 只有在相符的原始文字與輸入文字不同時才傳回。 用來傳回模糊比對的來源文字。 針對 Microsoft Translator 結果不予傳回。
* `Rating`：指出制定品質決策之人員的授權單位。 機器翻譯結果將會有評分 5。 匿名提供的翻譯一般會有 1 到 4 的評分，而授權提供的翻譯一般會有 6 到 10 的評分。
* `Count`：已選取具有此評分之這個翻譯的次數。 自動翻譯回應的值將會是 0。
* `TranslatedText`：翻譯的文字。


### <a name="response-class-status-200"></a>回應類別 (狀態 200)

字串

回應內容類型：application/xml
 
### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:--|:--|:--|:--|:--|
|Authorization  |(空白)    |若未指定 `appid` 欄位或 `Ocp-Apim-Subscription-Key` 標頭，則為必要項目。 授權權杖：`"Bearer" + " " + "access_token"`。|頁首|字串|
|Ocp-Apim-Subscription-Key|(空白)  |若未指定 `appid` 欄位或 `Authorization` 標頭，則為必要項目。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|
|:--|:--|
|400    |錯誤的要求。 檢查輸入參數和詳細錯誤回應。|
|401    |認證無效|
|500    |伺服器錯誤。 若錯誤持續發生，請讓我們知道。 請提供要求的大約日期和時間，並將要求識別碼包括在回應標頭 `X-MS-Trans-Info` 中。|
|503    |服務暫時無法使用。 請重試，並讓我們知道錯誤是否持續發生。|

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [遷移至 v3 Translator Text API](../migrate-to-v3.md)










