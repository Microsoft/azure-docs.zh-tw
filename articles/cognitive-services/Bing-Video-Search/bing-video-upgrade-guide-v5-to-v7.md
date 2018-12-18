---
title: 將 Bing 影片搜尋 API v5 升級至 v7
titlesuffix: Azure Cognitive Services
description: 識別您的應用程式有哪些部分需要更新，以使用第 7 版。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: conceptual
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: 967355d213f226711914e298a2fbf5676ebefe4f
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224455"
---
# <a name="video-search-api-upgrade-guide"></a>影片搜尋 API 升級指南

本升級指南會識別 Bing 影片搜尋 API 第 5 版和第 7 版之間的變更。 本指南可協助您識別出應用程式有哪些部分需要更新以使用第 7 版。

## <a name="breaking-changes"></a>重大變更

### <a name="endpoints"></a>端點

- 從 v5 變更為 v7 的端點版本號碼。 例如： `https://api.cognitive.microsoft.com/bing/v7.0/videos/search`。

### <a name="error-response-objects-and-error-codes"></a>錯誤回應物件和錯誤碼

- 所有失敗的要求應該立即在回應本文中包含 `ErrorResponse` 物件。

- 將下列欄位新增至 `Error` 物件。  
  - `subCode`&mdash;如果可能，將錯誤碼分割成離散貯體
  - `moreDetails`&mdash;`message` 欄位中所述之關於錯誤的其他資訊
   

- 將 v5 錯誤碼取代為下列可能的 `code` 和 `subCode` 值。

|代碼|子代碼|說明
|-|-|-
|ServerError|UnexpectedError<br/>ResourceError<br/>NotImplemented|只要出現任何子代碼條件，Bing 會傳回 ServerError。 如果 HTTP 狀態碼為 500，則回應會包含這些錯誤。
|InvalidRequest|ParameterMissing<br/>ParameterInvalidValue<br/>HttpNotAllowed<br/>Blocked|只要要求的任何部分無效，Bing 就會傳回 InvalidRequest。 例如缺少必要的參數或參數值無效。<br/><br/>如果錯誤是 ParameterMissing 或 ParameterInvalidValue，則 HTTP 狀態碼為 400。<br/><br/>如果錯誤是 HttpNotAllowed，則 HTTP 狀態碼為 410。
|RateLimitExceeded||每當您超過每秒查詢 (QPS) 或每月查詢 (QPM) 配額時，Bing 會傳回 RateLimitExceeded。<br/><br/>如果超過 QPS，Bing 會傳回 HTTP 狀態碼 429，如果超過 QPM，則會傳回 403。
|InvalidAuthorization|AuthorizationMissing<br/>AuthorizationRedundancy|當 Bing 無法驗證呼叫者時，Bing 會傳回 InvalidAuthorization。 例如，缺少 `Ocp-Apim-Subscription-Key` 標頭，或訂用帳戶金鑰無效。<br/><br/>如果您指定一個以上的驗證方法，則會出現備援。<br/><br/>如果錯誤是 InvalidAuthorization，則 HTTP 狀態碼為 401。
|InsufficientAuthorization|AuthorizationDisabled<br/>AuthorizationExpired|當呼叫者沒有資源存取權限時，Bing 會傳回 InsufficientAuthorization。 如果訂用帳戶金鑰已停用或已過期，則會發生此情況。 <br/><br/>如果錯誤是 InsufficientAuthorization，則 HTTP 狀態碼為 403。

- 以下會將先前的錯誤碼對應至新的代碼。 如果您已在 v5 錯誤碼上取得相依性，請視情況更新您的程式碼。

|第 5 版程式碼|第 7 版 code.subCode
|-|-
|RequestParameterMissing|InvalidRequest.ParameterMissing
RequestParameterInvalidValue|InvalidRequest.ParameterInvalidValue
ResourceAccessDenied|InsufficientAuthorization
ExceededVolume|RateLimitExceeded
ExceededQpsLimit|RateLimitExceeded
已停用|InsufficientAuthorization.AuthorizationDisabled
UnexpectedError|ServerError.UnexpectedError
DataSourceErrors|ServerError.ResourceError
AuthorizationMissing|InvalidAuthorization.AuthorizationMissing
HttpNotAllowed|InvalidRequest.HttpNotAllowed
UserAgentMissing|InvalidRequest.ParameterMissing
NotImplemented|ServerError.NotImplemented
InvalidAuthorization|InvalidAuthorization
InvalidAuthorizationMethod|InvalidAuthorization
MultipleAuthorizationMethod|InvalidAuthorization.AuthorizationRedundancy
ExpiredAuthorizationToken|InsufficientAuthorization.AuthorizationExpired
InsufficientScope|InsufficientAuthorization
Blocked|InvalidRequest.Blocked

### <a name="query-parameters"></a>查詢參數

- 將 `modulesRequested` 查詢參數重新命名為 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#modulesrequested)。  

### <a name="object-changes"></a>物件變更

- 將 [Videos](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos) 的 `nextOffsetAddCount` 欄位重新命名為 `nextOffset`。 使用 offset 的方式也會跟著變更。 先前，您已將 [offset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#offset) 查詢參數設定為 `nextOffset` 值，以在結果中加上前一個 offset 值和影片數目。 現在，您只需要將 `offset` 查詢參數設定為 `nextOffset` 值。  
  
- 已將 `relatedVideos` 欄位的資料類型從 `Video[]` 變更為 [VideosModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videosmodule) (請參閱 [VideoDetails](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videodetails))。

