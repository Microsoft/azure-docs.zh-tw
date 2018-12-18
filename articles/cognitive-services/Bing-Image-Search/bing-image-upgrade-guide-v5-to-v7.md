---
title: 將 Bing 影像搜尋 API v5 升級為 v7 | Microsoft Docs
description: 識別您的應用程式有哪些部分需要更新，以使用第 7 版。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: 7F78B91F-F13B-40A4-B8A7-770FDB793F0F
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: article
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: b4d785eafe9ced6fb12e2dac3282dfd286849eb6
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368139"
---
# <a name="image-search-api-upgrade-guide"></a>影像搜尋 API 升級指南

本升級指南會識別 Bing 影像搜尋 API 第 5 版和第 7 版之間的變更。 本指南可協助您識別出應用程式有哪些部分需要更新以使用第 7 版。

## <a name="breaking-changes"></a>重大變更

### <a name="endpoints"></a>端點

- 從 v5 變更為 v7 的端點版本號碼。 例如，https://api.cognitive.microsoft.com/bing/\*\*v7.0**/images/search。

### <a name="error-response-objects-and-error-codes"></a>錯誤回應物件和錯誤碼

- 所有失敗的要求應該立即在回應本文中包含 `ErrorResponse` 物件。

- 將下列欄位新增至 `Error` 物件。  
  - `subCode`&mdash;如果可能，將錯誤碼分割成離散貯體
  - `moreDetails`&mdash;`message` 欄位中所述之關於錯誤的其他資訊
   

- 將 v5 錯誤碼取代為下列可能的 `code` 和 `subCode` 值。

|代碼|SubCode|說明
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

- 將 `modulesRequested` 查詢參數重新命名為 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modules)。  
  
- 將 Annotations 重新命名為 Tags。 請參閱 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modules) 查詢參數上的 Tags。  

- 將支援的 ShoppingSources 篩選值市場清單變更為僅限 en-US。 請參閱 [imageType](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagetype)。  
 

### <a name="image-insights-changes"></a>影像深入解析變更
  
- 將 [ImagesInsights](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imageinsightsresponse) 的 `annotations` 欄位重新命名為 `imageTags`。  
  
- 將 `AnnotationModule` 物件重新命名為 [ImageTagsModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagetagsmodule)。  
  
- 將 `Annotation` 物件重新命名為 [Tag](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#tag)，並移除 `confidence` 欄位。  
  
- 將 [Image](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#image) 物件的 `insightsSourcesSummary` 欄位重新命名為 `insightsMetadata`。  
  
- 將 `InsightsSourcesSummary` 物件重新命名為 [InsightsMetadata](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightsmetadata)。  
  
- 新增 `https://api.cognitive.microsoft.com/bing/v7.0/images/details` 端點。 使用此端點來要求影像深入解析，而不是使用 /images/search 端點。 請參閱[影像深入解析](./image-insights.md)。 
  
- 下列查詢參數目前只有搭配 `/images/details` 端點才有效。  
  
    -   [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightstoken)  
    -   [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modules)  
    -   [imgUrl](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imgurl)  
    -   [cab](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cab)  
    -   [cal](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cal)  
    -   [car](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#car)  
    -   [cat](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cat)  
    -   [ct](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#ct)  
  
- 將 `ImageInsightsResponse` 物件重新命名為 [ImageInsights](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imageinsights)。  
  
- 變更 [ImageInsights](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imageinsights) 物件中下列欄位的資料類型。  
  
    -   將 `relatedCollections` 欄位的類型從 `ImageGallery[]` 變更為 [RelatedCollectionsModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#relatedcollectionsmodule)。  
  
    -   將 `pagesIncluding` 欄位的類型從 `Image[]` 變更為 [ImagesModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagesmodule)。  
  
    -   將 `relatedSearches` 欄位的類型從 `Query[]` 變更為 [RelatedSearchesModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#relatedsearchesmodule)。  
  
    -   將 `recipes` 欄位的類型從 `Recipe[]` 變更為 [RecipesModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#recipesmodule)。  
  
    -   將 `visuallySimilarImages` 欄位的類型從 `Image[]` 變更為 [ImagesModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagesmodule)。  
  
    -   將 `visuallySimilarProducts` 欄位的類型從 `ProductSummaryImage[]` 變更為 [ImagesModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagesmodule)。  
  
    -   移除 `ProductSummaryImage` 物件，並將產品相關欄位移至 [Image](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#image) 物件中。 只有在影像屬於影像深入解析回應中的視覺相似產品時，`Image` 物件才會包含產品相關欄位。  
  
    -   將 `recognizedEntityGroups` 欄位的類型從 `RecognizedEntityGroup[]` 變更為 [RecognizedEntitiesModule](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#recognizedentitiesmodule)。  
  
-   將 [ImageInsights](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imageinsightsresponse) 的 `categoryClassification` 欄位重新命名為 `annotations`，並將其類型變更為 `AnnotationsModule`。  

### <a name="images-answer"></a>影像回應

-   從 [Images](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#images) 中移除 displayShoppingSourcesBadges 和 displayRecipeSourcesBadges 欄位。  
  
-   將 [Images](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#images) 的 `nextOffsetAddCount` 欄位重新命名為 `nextOffset`。 使用 offset 的方式也會跟著變更。 先前，您已將 [offset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#offset) 查詢參數設定為 `nextOffsetAddCount` 值，以在結果中加上前一個 offset 值和影像數目。 現在，您會將 `offset` 設定為 `nextOffset` 值。  
    
  
## <a name="non-breaking-changes"></a>非中斷性變更

### <a name="query-parameters"></a>查詢參數
  
- 將 Transparent 新增為可能的 [imageType](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagetype) 篩選值。 Transparent 篩選只會傳回具有透明背景的影像。

- 將 Any 新增為可能的 [license](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#license) 篩選值。 Any 篩選只會傳回授權下的影像。
  
- 新增 [maxFileSize](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#maxfilesize) 和 [minFileSize](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#minfilesize) 查詢參數。 您可以使用這些篩選來傳回檔案大小範圍內的影像。  
  
- 新增 [maxHeight](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#maxheight)、[minHeight](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#minheight)、[maxWidth](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#maxwidth)、[minWidth](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#minwidth) 查詢參數。 您可以使用這些篩選來傳回高度與寬度範圍內的影像。  

### <a name="object-changes"></a>物件變更
  
- 將 `description` 和 `lastUpdated` 欄位新增至 [Offer](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#offer) 物件。  
  
- 將 `name` 欄位新增至 [ImageGallery](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagegallery) 物件。  
  
- 將 `similarTerms` 新增至 [Images](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#images) 物件。 此欄位會包含與使用者查詢字串有類似意思的字詞清單。  
