---
title: LUIS 中的資料變動概念 - Language Understanding
titleSuffix: Azure Cognitive Services
description: 了解如何在於 Language Understanding (LUIS) 中進行預測之前變更資料
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 1aad540086764b1e2315d3b3e195c55ba5931e07
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036045"
---
# <a name="data-alterations"></a>資料變動
LUIS 提供可在預測之前或預測期間操作語句的方法。 這些包括修正拼字，以及修正重新建置 datetimeV2 的時區問題。 

## <a name="correct-spelling-errors-in-utterance"></a>校正語句中的拼字錯誤
LUIS 使用 [Bing 拼字檢查 API V7](https://azure.microsoft.com/services/cognitive-services/spell-check/) 來校正語句中的拼字錯誤。 LUIS 需要與該服務相關的金鑰。 請建立金鑰，然後在[端點](https://aka.ms/luis-endpoint-apis)新增該金鑰作為查詢字串參數。 

您也可以在 [Test] \(測試\) 面板中[輸入金鑰](luis-interactive-test.md#view-bing-spell-check-corrections-in-test-panel)來更正拼字錯誤。 金鑰會以工作階段變數的形式保留在 [Test] \(測試\) 面板的瀏覽器中。 請在您想要校正拼字的每個瀏覽器工作階段中，將金鑰新增至 [Test] \(測試\) 面板。 

在測試面板及端點的金鑰使用會計入[金鑰使用量](https://azure.microsoft.com/pricing/details/cognitive-services/spellcheck-api/)配額中。 LUIS 實作「Bing 拼字檢查」的文字長度限制。 

端點必須有兩個參數，才能讓拼字校正運作：

|參數|值|
|--|--|
|`spellCheck`|布林值|
|`bing-spell-check-subscription-key`|[Bing 拼字檢查 API V7](https://azure.microsoft.com/services/cognitive-services/spell-check/) 端點金鑰|

當 [Bing 拼字檢查 API V7](https://azure.microsoft.com/services/cognitive-services/spell-check/) 偵測到錯誤時，系統會將原始語句和校正後語句及預測一起從端點傳回。

```JSON
{
  "query": "Book a flite to London?",
  "alteredQuery": "Book a flight to London?",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 0.780123
  },
  "entities": []
}
```
 
### <a name="whitelist-words"></a>將字組列入白名單
LUIS 中使用的 Bing 拼字檢查 API 不支援要在拼字檢查變動期間忽略的字組白名單。 如果您需要將字組或縮寫列入白名單，請在將語句傳送至 LUIS 進行意圖預測之前，利用白名單處理用戶端應用程式中的語句。

## <a name="change-time-zone-of-prebuilt-datetimev2-entity"></a>變更預先建置 datetimeV2 實體的時區
當 LUIS 應用程式使用預先建置的 datetimeV2 實體時，可以在預測回應中傳回日期時間值。 要求的時區會用來判斷要傳回的正確日期時間。 如果要求來自 Bot 或另一個集中式應用程式，請在其抵達 LUIS 之前，先更正 LUIS 使用的時區。 

### <a name="endpoint-querystring-parameter"></a>端點查詢字串參數
更正時區的方式是使用 `timezoneOffset` 參數將使用者時區新增至[端點](https://aka.ms/luis-endpoint-apis)。 `timezoneOffset` 的值應該是用以變更時間的正數或負數 (以分鐘為單位)。  

|參數|值|
|--|--|
|`timezoneOffset`|正數或負數 (以分鐘為單位)|

### <a name="daylight-savings-example"></a>日光節約範例
如果您需要讓傳回的預先建置 datetimeV2 針對日光節約時間進行調整，則應該針對[端點](https://aka.ms/luis-endpoint-apis)查詢，使用 `timezoneOffset` 查詢字串參數搭配以分鐘為單位的 +/- 值。

增加 60 分鐘： 

https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q=Turn the lights on?**timezoneOffset=60**&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}

減去 60 分鐘： 

https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q=Turn the lights on?**timezoneOffset=-60**&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}

## <a name="c-code-determines-correct-value-of-timezoneoffset"></a>C# 程式碼可判斷 timezoneOffset 的正確值
下列 C# 程式碼使用 [TimeZoneInfo](https://docs.microsoft.com/dotnet/api/system.timezoneinfo?view=netframework-4.7.1) 類別的 [FindSystemTimeZoneById](https://docs.microsoft.com/dotnet/api/system.timezoneinfo.findsystemtimezonebyid?view=netframework-4.7.1#examples) 方法，根據系統時間判斷正確的 `timezoneOffset`：

```CSharp
// Get CST zone id
TimeZoneInfo targetZone = TimeZoneInfo.FindSystemTimeZoneById("Central Standard Time");

// Get local machine's value of Now
DateTime utcDatetime = DateTime.UtcNow;

// Get Central Standard Time value of Now
DateTime cstDatetime = TimeZoneInfo.ConvertTimeFromUtc(utcDatetime, targetZone);

// Find timezoneOffset
int timezoneOffset = (int)((cstDatetime - utcDatetime).TotalMinutes);
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用此教學課程來校正拼字錯誤](luis-tutorial-bing-spellcheck.md)