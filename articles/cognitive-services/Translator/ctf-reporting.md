---
title: Microsoft Translator Collaborative Translation Framework (CTF) 報告
description: 如何使用 Collaborative Translation Framework (CTF) 報告。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.component: translator-text
ms.topic: article
ms.date: 12/14/2017
ms.author: v-jansko
ms.openlocfilehash: cefc630a82a56703ba4942bcad18f6e0a38b1ee5
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368658"
---
# <a name="how-to-use-collaborative-translation-framework-ctf-reporting"></a>如何使用 Collaborative Translation Framework (CTF) 報告

> [!NOTE]
> 此方法已被取代。 Translator Text API V3.0 中不提供此功能。

> 先前在 Translator Text API V2.0 中提供的 Collaborative Translations Framework (CTF) 自 2018 年 2 月 1 日起已被取代。 AddTranslation 和 AddTranslationArray 函式可讓使用者透過 Collaborative Translation Framework 啟用校正功能。 在 2018 年 1 月 31 日之後，這兩個函式便不接受新的句子提交，而使用者則會收到錯誤訊息。 這些函式已被淘汰，而不會被取代。 

>Translator Hub API 中有提供類似的功能，可讓您使用自己的術語和樣式來建置自訂翻譯系統，然後可以在 Translator Text API 中使用「分類 ID」來叫用它。 Translator Hub：[https://hub.microsofttranslator.com](https://hub.microsofttranslator.com). Translator Hub API：[https://hub.microsofttranslator.com/swagger](https://hub.microsofttranslator.com/swagger).

「Collaborative Translation Framework (CTF) 報告 API」會傳回統計資料和 CTF 存放區中的實際內容。 此 API 與 GetTranslations() 方法不同，因為它：
* 只會從您的帳戶 (appId 或 Azure Marketplace 帳戶) 傳回翻譯後的內容及其總計數。
* 會傳回翻譯後的內容及其總計數，但不會要求比對來源句子。
* 不會傳回自動翻譯 (機器翻譯)。

## <a name="endpoint"></a>端點
「CTF 報告 API」的端點是 http://api.microsofttranslator.com/v2/beta/ctfreporting.svc
                        

## <a name="methods"></a>方法
| Name |    說明|
|:---|:---|
| GetUserTranslationCounts 方法 | 取得使用者所建立的翻譯計數。 |
| GetUserTranslations 方法 | 擷取使用者所建立的翻譯。 |

這些方法可讓您：
* 擷取您帳戶識別碼底下可供下載的一組完整使用者翻譯和校正。
* 取得頻繁參與者的清單。 確定 AddTranslation() 中提供的使用者名稱正確。
* 建置使用者介面 (UI)，此使用者介面可允許受信任的使用者查看所有可用候選項目，必要時，還可根據 URI 前置詞，將範圍限制在您網站的某個部分。

> [!NOTE]
> 這兩種方法的執行速度都相當慢且成本高昂。 建議您儘量不要使用它們。

## <a name="getusertranslationcounts-method"></a>GetUserTranslationCounts 方法

此方法會取得使用者所建立的翻譯計數。 它可提供依 uriPrefix、from、to、user、minRating 及 maxRating 要求參數分組的翻譯計數清單。

**語法**

> [!div class="tabbedCodeSnippets"]
```cs
UserTranslationCount[]GetUserTranslationCounts(
           string appId,
           string uriPrefix,
           string from,
           string to,
           int? minRating,
           int? maxRating,
           string user, 
           string category
           DateTime? minDateUtc,
           DateTime? maxDateUtc,
           int? skip,
           int? take);
```

**參數**

| 參數 | 說明 |
|:---|:---|
| appId | **必要** 如果使用 Authorization 標頭，請將 appid 欄位保留空白，否則請指定一個包含 "Bearer" + " " + 存取權杖的字串。|
| uriPrefix | **選擇性** 一個包含翻譯 URI 前置詞的字串。|
| from | **選擇性** 一個代表翻譯文字之語言代碼的字串。 |
| to | **選擇性** 一個代表文字之目標翻譯語言代碼的字串。|
| minRating| **選擇性** 一個代表翻譯後文字之品質評分下限的整數值。 有效值介於 -10 到 10 之間。 預設值為 1。|
| maxRating| **選擇性** 一個代表翻譯後文字之品質評分上限的整數值。 有效值介於 -10 到 10 之間。 預設值為 1。|
| user | **選擇性** 一個可根據提交項目建立者來篩選結果的字串。 |
| category| **選擇性** 一個包含翻譯之分類或領域的字串。 此參數僅支援預設選項 general。|
| minDateUtc| **選擇性** 您想要開始擷取翻譯的日期。 此日期必須是 UTC 格式。 |
| maxDateUtc| **選擇性** 您想要停止擷取翻譯的日期。 此日期必須是 UTC 格式。 |
| skip| **選擇性** 您想要在頁面上略過的結果數目。 例如，如果您想要略過前 20 列結果並從第 21 個結果記錄開始檢視，請為此參數指定 20。 此參數的預設值為 0。|
| take |  您想要擷取的結果數目。 每個要求的數目上限為 100。 預設值為 100。|

> [!NOTE]
> skip 和 take 要求參數可讓您將大量結果記錄分頁。

**傳回值**

結果集會包含 **UserTranslationCount** 的陣列。 每個 UserTranslationCount 都有下列元素：

| 欄位 | 說明 |
|:---|:---|
| Count| 所擷取的結果數目|
| 從 | 來源語言|
| Rating| 提交者在 AddTranslation() 方法呼叫中套用的評分|
| 至| 目標語言|
| Uri| AddTranslation() 方法呼叫中套用的 URI|
| 使用者| 使用者名稱|

**例外狀況**

| 例外狀況 | 訊息 | 條件 |
|:---|:---|:---|
| ArgumentOutOfRangeException | '**maxDateUtc**' 參數必須大於或等於 '**minDateUtc**'。| **maxDateUtc** 參數的值小於 **minDateUtc** 參數的值。|
| TranslateApiException | IP 超出配額。| <ul><li>已達到每分鐘的要求數目限制。</li><li>要求大小維持限制在 10000 個字元。</li><li>每小時和每日配額會限制 Microsoft Translator API 將接受的字元數目。</li></ul>|
| TranslateApiException | AppId 超出配額。| 應用程式識別碼超出每小時或每日配額。|

> [!NOTE]
> 配額將會調整以確保服務之所有使用者之間的公平性。

**檢視 GitHub 上的程式碼範例**
* [C#](https://github.com/MicrosoftTranslator/Documentation-Code-TextAPI/blob/master/ctf/ctf-getusertranslationcounts-example-csharp.md)
* [PHP](https://github.com/MicrosoftTranslator/Documentation-Code-TextAPI/blob/master/ctf/ctf-getusertranslationcounts-example-php.md)

## <a name="getusertranslations-method"></a>GetUserTranslations 方法

此方法會擷取使用者所建立的翻譯。 它可提供依 uriPrefix、from、to、user、minRating 及 maxRating 要求參數分組的翻譯。

**語法**

> [!div class="tabbedCodeSnippets"]
```cs
UserTranslation[] GetUserTranslations (
            string appId,
            string uriPrefix,
            string from,
            string to,
            int? minRating,
            int? maxRating,
            string user, 
            string category
            DateTime? minDateUtc,
            DateTime? maxDateUtc,
            int? skip,
            int? take); 
```

**參數**

| 參數 | 說明 |
|:---|:---|
| appId | **必要** 如果使用 Authorization 標頭，請將 appid 欄位保留空白，否則請指定一個包含 "Bearer" + " " + 存取權杖的字串。|
| uriPrefix| **選擇性** 一個包含翻譯 URI 前置詞的字串。|
| from| **選擇性** 一個代表翻譯文字之語言代碼的字串。|
| to| **選擇性** 一個代表文字之目標翻譯語言代碼的字串。|
| minRating| **選擇性** 一個代表翻譯後文字之品質評分下限的整數值。 有效值介於 -10 到 10 之間。 預設值為 1。|
| maxRating| **選擇性** 一個代表翻譯後文字之品質評分上限的整數值。 有效值介於 -10 到 10 之間。 預設值為 1。|
| user| **選擇性。一個可根據提交項目建立者來篩選結果的字串**|
| category| **選擇性** 一個包含翻譯之分類或領域的字串。 此參數僅支援預設選項 general。| 
| minDateUtc| **選擇性** 您想要開始擷取翻譯的日期。 此日期必須是 UTC 格式。| 
| maxDateUtc| **選擇性** 您想要停止擷取翻譯的日期。 此日期必須是 UTC 格式。|
| skip| **選擇性** 您想要在頁面上略過的結果數目。 例如，如果您想要略過前 20 列結果並從第 21 個結果記錄開始檢視，請為此參數指定 20。 此參數的預設值為 0。|
| take|  您想要擷取的結果數目。 每個要求的數目上限為 100。 預設值為 50。|

> [!NOTE]
> skip 和 take 要求參數可讓您將大量結果記錄分頁。

**傳回值**

結果集會包含 **UserTranslation** 的陣列。 每個 UserTranslation 都有下列元素：

| 欄位 | 說明 |
|:---|:---|
| CreatedDateUtc| 使用 AddTranslation() 之項目的建立日期|
| 從| 來源語言|
| OriginalText| 提交要求時所使用的來源語言文字|
|Rating |提交者在 AddTranslation() 方法呼叫中套用的評分|
|至|    目標語言|
|TranslatedText|    AddTranslation() 方法呼叫中所提交的翻譯|
|Uri|   AddTranslation() 方法呼叫中套用的 URI|
|使用者   |使用者名稱|

**例外狀況**

| 例外狀況 | 訊息 | 條件 |
|:---|:---|:---|
| ArgumentOutOfRangeException | '**maxDateUtc**' 參數必須大於或等於 '**minDateUtc**'。| **maxDateUtc** 參數的值小於 **minDateUtc** 參數的值。|
| TranslateApiException | IP 超出配額。| <ul><li>已達到每分鐘的要求數目限制。</li><li>要求大小維持限制在 10000 個字元。</li><li>每小時和每日配額會限制 Microsoft Translator API 將接受的字元數目。</li></ul>|
| TranslateApiException | AppId 超出配額。| 應用程式識別碼超出每小時或每日配額。|

> [!NOTE]
> 配額將會調整以確保服務之所有使用者之間的公平性。

**檢視 GitHub 上的程式碼範例**
* [C#](https://github.com/MicrosoftTranslator/Documentation-Code-TextAPI/blob/master/ctf/ctf-getusertranslations-example-csharp.md)
* [PHP](https://github.com/MicrosoftTranslator/Documentation-Code-TextAPI/blob/master/ctf/ctf-getusertranslations-example-php.md)


















