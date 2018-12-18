---
title: 專案 URL 預覽參考 - Microsoft 認知服務 | Microsoft Docs
description: 專案 URL 預覽端點參考。
services: cognitive-services
author: mikedodaro
manager: rosh
ms.service: cognitive-services
ms.technology: project-url-preview
ms.topic: article
ms.date: 03/29/2018
ms.author: rosh, v-gedod
ms.openlocfilehash: 46c011d62b6ae51f5f7d292345e6ece0e27a8541
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37865870"
---
# <a name="project-url-preview-v7-reference"></a>專案 URL 預覽 v7 參考

URL 預覽支援部落格文章、論壇討論、預覽頁面等 Web 資源的簡短說明。此 URL 可以是任何類型的網際網路資源，例如網頁、新聞、映像或影片。 查詢必須是帶有 http 或 https 配置的絕對 URL；不支援相對 URL 或其他配置，例如 ftp://。

使用 URL 預覽的應用程式會以 URL 將 Web 要求傳送至端點，以在查詢參數中預覽。  要求必須包含 *Ocp-Apim-Subscription-Key* 標頭。   

可以針對預覽資訊剖析 JSON 回應：名稱、資源描述、*isFamilyFriendly*，以及可存取代表圖片和完整線上資源的連結。

僅可使用來自 URL 預覽的資料，以在終端使用者於社交媒體、聊天機器人或類似產品上進行 URL 分享時，顯示以超連結連自其來源網站的預覽程式碼片段縮圖影像。 您不得複製、儲存或快取從專案 URL 預覽收到的任何資料。 若您收到要求停用您從網站或內容擁有者所接收的預覽，則一律必須接受。

## <a name="endpoint"></a>端點
若要要求 URL 預覽結果，請將要求傳送至下列端點。 使用標頭和 URL 參數以進一步定義規格。

端點 GET： 
````
https://api.labs.cognitive.microsoft.com/urlpreview/v7.0/search?q=queryURL

````

要求必須使用 HTTPS 通訊協定，並包含下列查詢參數：

 q - 識別欲預覽之 URL 的查詢 

下列章節提供關於影響搜尋結果之回應物件、查詢參數和標頭的技術詳細資料。 
  
關於要求所應包含之標頭的相關資訊，請參閱[標頭](#headers)。  
  
關於要求所應包含之查詢參數的相關資訊，請參閱[查詢參數](#query-parameters)。  
  
關於回應所包含之 JSON 物件的相關資訊，請參閱[回應物件](#response-objects)。

查詢 URL 長度上限是 2,048 個字元。 若要確保您的 URL 長度不超過限制，查詢參數的最大長度應該小於 1,500 個字元。 如果 URL 超過 2,048 個字元，則伺服器會傳回「404 找不到」。  

如需使用允許和顯示結果方式的相關資訊，請參閱[使用和顯示需求](use-display-requirements.md)。 

> [!NOTE]
> 某些對其他搜尋 API 有意義的要求標頭不會影響 URL 預覽
> - Pragma – 呼叫者無法控制 URL 預覽是否使用快取
> - 使用者代理程式 – 目前，URL 預覽 API 不會對源自於 PC、筆記型電腦或行動裝置的呼叫分別提供不同回應。

> 此外，某些參數目前對 URL 預覽 API 沒有意義，但可能在將來用於改善全球化。 
 
## <a name="headers"></a>headers  
以下是要求和回應可能包含的標頭。  
  
|頁首|說明|  
|------------|-----------------|   
|<a name="market" />BingAPIs-Market|回應標頭。<br /><br /> 要求所使用的市場。 格式為 \<languageCode\>-\<countryCode\>。 例如：en-US。|  
|<a name="traceid" />BingAPIs-TraceId|回應標頭。<br /><br /> 包含要求詳細資料之記錄項目的識別碼。 發生錯誤時，會擷取這個識別碼。 如果您無法判定並解決問題，請將此識別碼與其他資訊一併提供給支援小組。|  
|<a name="subscriptionkey" />Ocp-Apim-Subscription-Key|必要的要求標頭。<br /><br /> 您在[認知服務](https://www.microsoft.com/cognitive-services/)中註冊此服務時收到的訂用帳戶金鑰。|  
|<a name="clientid" />X-MSEdge-ClientID|選擇性要求和回應標頭。<br /><br /> Bing 使用此標頭在 Bing API 所有呼叫之間為使用者提供一致的行為。 Bing 經常推出新功能和改善項目，且會以用戶端識別碼作為不同測試版指派流量的金鑰。 如果在多個要求中，未讓使用者使用相同的用戶端識別碼，Bing 可能會將使用者指派至多個衝突的測試版。 若指派給多個衝突的測試版，可能會導致使用者體驗不一致。 例如若第二個要求與第一個要求指派的測試版不同，可能會產生意料外的體驗。 此外，Bing 可以使用用戶端識別碼將 Web 結果調整為該用戶端識別碼的搜尋記錄，為使用者提供更豐富的體驗。<br /><br /> Bing 也會使用此標頭分析用戶端識別碼產生的活動，協助改善結果的順位。 相關性改進功能有助於 Bing API 提供更高品質的結果，進而為 API 取用者提供更高的點擊率。<br /><br />以下是適用於此標頭的基本使用規則。<br /><ul><li>在裝置上使用您應用程式的每個使用者必須具有 Bing 產生的唯一用戶端識別碼。<br /><br/>如果您未在要求中包含此標頭，Bing 會產生一個識別碼，並於 X-MSEdge-ClientID 回應標頭傳回該識別碼。 不應在要求中包含此標頭的唯一時機是使用者初次在該裝置上使用您的應用程式時。<br /><br/></li><li>您的應用程式每次為該裝置上的該名使用者提出 Bing API 請求時，均需使用此用戶端識別碼。<br /><br/></li><li>**注意：** 您必須確保此用戶端識別碼不能連結到任何可辨識身份的使用者帳戶資訊。</li><br/><li>保留用戶端識別碼。 若要在瀏覽器應用程式中保留識別碼，請使用永續性的 HTTP Cookie，以確保在所有工作階段均使用該識別碼。 請勿使用工作階段 Cookie。 若為其他應用程式 (例如行動裝置應用程式)，請使用裝置本身的永久儲存裝置保留識別碼。<br /><br/>使用者下次在該裝置上使用您的應用程式時，會取得您保留的用戶端識別碼。</li></ul><br /> **注意：** Bing 回應可能包含或可能不包含此標頭。 如果回應包含此標頭，請擷取用戶端識別碼，並將其用於該裝置上使用者的所有後續 Bing 要求。<br /><br /> **注意：** 如果您包含 X-MSEdge-ClientID，則不得在要求中加入 Cookie。|  
|<a name="clientip" />X-MSEdge-ClientIP|選擇性要求標頭。<br /><br /> 用戶端裝置的 IPv4 或 IPv6 位址。 IP 位址用於探索使用者的位置。 Bing 使用位置資訊來判斷安全搜尋行為。<br /><br />  請勿混淆位址 (例如，藉由將最後一個八位元變更為 0)。 混淆位址會導致位置不在裝置的實際位置附近，這可能會造成 Bing 產生錯誤結果。|  
<br /><br /></li></ul>   

## <a name="query-parameters"></a>查詢參數  
要求可能含有下列查詢參數。 請參閱必要參數的必要資料行。 您必須對查詢參數進行 URL 編碼。 查詢必須是帶有 http 或 https 配置的絕對 URL；不支援相對 URL 或其他配置，例如 ftp://
  
  
|Name|值|類型|必要|  
|----------|-----------|----------|--------------|  
|<a name="mkt" />mkt|結果所來自的市場。 <br /><br />如需可能的市場值清單，請參閱[市場代碼](#market-codes)。<br /><br /> **注意：** URL 預覽 API 目前僅支援美國地理位置和英文。<br /><br />|字串|yes|  
|<a name="query" />q|要預覽的 URL|字串|yes|  
|<a name="responseformat" />responseFormat|要用於回應的媒體類型。 以下是可能的值 (不區分大小寫)。<br /><ul><li>JSON</li><li>JSONLD</li></ul><br /> 預設值是 JSON。 如需回應所包含之 JSON 物件的相關資訊，請參閱[回應物件](#response-objects)。<br /><br />  如果您指定 JsonLd，回應本文會有內含搜尋結果的 JSON-LD 物件。 如需 JSON-LD 的相關資訊，請參閱 [JSON-LD](http://json-ld.org/)。|字串|否|
|<a name="safesearch"/>safeSearch|禁止非法成人內容或盜版內容，錯誤代碼為 400，並且不會傳回 *isFamilyFriendly* 旗標。 <p>對於合法的成人內容，會出現以下行為。 狀態碼傳回 200，而且 *isFamilyFriendly* 旗標設為 false。<ul><li>safeSearch=strict：不會傳回標題、描述、URL 以及映像。</li><li>safeSearch=moderate；取得標題、URL 以及描述，但不會有描述性的影像。</li><li>safeSearch=off；取得所有回應物件/元素 – 標題、URL、描述、影像。</li></ul> |字串|不需要。 </br> 預設值為 safeSearch=strict。| 

## <a name="response-objects"></a>回應物件  
回應結構描述是 [WebPage] 或 ErrorResponse，如同在 Web 搜尋 API 中一樣。 如果要求失敗，則最上層的物件是 [ErrorResponse](#errorresponse) 物件。


|Object|說明|  
|------------|-----------------|  
|[網頁](#webpage)|包含預覽屬性的最上層 JSON 物件。|  

  
### <a name="error"></a>Error  
定義發生的錯誤。  
  
|元素|說明|類型|  
|-------------|-----------------|----------|  
|<a name="error-code" />code|識別錯誤類別的錯誤碼。 如需可能的代碼清單，請參閱[錯誤碼](#error-codes)。|字串|  
|<a name="error-message" />message|錯誤的描述。|字串|  
|<a name="error-moredetails" />moreDetails|提供關於錯誤之其他資訊的描述。|字串|  
|<a name="error-parameter" />parameter|造成錯誤之要求的查詢參數。|字串|  
|<a name="error-subcode" />subCode|識別錯誤的錯誤碼。 例如，如果 `code`是 InvalidRequest，則 `subCode` 可能是 ParameterInvalid 或 ParameterInvalidValue。 |字串|  
|<a name="error-value" />value|非有效的查詢參數值。|字串|  
  

### <a name="errorresponse"></a>ErrorResponse  
要求失敗時，回應包含的最上層物件。  
  
|Name|值|類型|  
|----------|-----------|----------|  
|_type|類型提示。|字串|  
|<a name="errors" />errors|說明要求失敗原因的錯誤清單。|[錯誤](#error)[]|   
  

### <a name="webpage"></a>網頁  
定義預覽網頁的相關資訊。  
  
|Name|值|類型|  
|----------|-----------|----------|
|name|頁面標題，不一定是 HTML 標題|字串|
|url|實際搜耙的 URL (要求可能已遵循重新導向)|字串|  
|說明|頁面和內容的簡短描述|字串|  
|isFamilyFriendly|對於 Web 索引中的項目最準確；即時擷取僅會根據 URL 進行此偵測動作，而非頁面內容|布林值|
|primaryImageOfPage/contentUrl|要包含在預覽中的代表圖片 URL|字串| 


### <a name="identifiable"></a>可識別
|Name|值|類型|  
|-------------|-----------------|----------|
|id|資源識別碼|字串|
 

## <a name="error-codes"></a>錯誤碼

以下是要求可能會傳回的 HTTP 狀態碼。  
  
|狀態碼|說明|  
|-----------------|-----------------|  
|200|成功。|  
|400|缺少其中一個查詢參數或無效。| 
|400|ServerError、subCode ResourceError：無法觸達要求的 URL|
|400|ServerError、subCode ResourceError：要求的 URL 未傳回成功代碼 (如果其傳回 HTTP 404 也包括在內)|
|400|InvalidRequest、subCode Blocked：要求的 URL 可能包含成人內容，且已遭封鎖| 
|401|缺少訂用帳戶金鑰或無效。|  
|403|使用者已通過身分驗證 (例如已使用有效的訂用帳戶金鑰)，但並未擁有所要求的資源的權限。<br /><br /> 如果呼叫者超過其每月查詢配額，Bing 可能也會傳回此狀態。|  
|410|要求所用的是 HTTP 而非 HTTPS 通訊協定。 HTTPS 是唯一支援的通訊協定。|  
|429|呼叫者超過其每秒查詢配額。|  
|500|未預期的伺服器錯誤。|

如果要求失敗，回應會包含 [ErrorResponse](#errorresponse) 物件，內有說明錯誤原因的 [Error](#error) 物件清單。 如果錯誤與參數有關，則 `parameter` 欄位會識別有問題的參數。 如果錯誤與參數值有關，則 `value` 欄位會識別無效的值。

```json
{
  "_type": "ErrorResponse", 
  "errors": [
    {
      "code": "InvalidRequest", 
      "subCode": "ParameterMissing", 
      "message": "Required parameter is missing.", 
      "parameter": "q" 
    }
  ]
}

{
  "_type": "ErrorResponse", 
  "errors": [
    {
      "code": "InvalidAuthorization", 
      "subCode": "AuthorizationMissing", 
      "message": "Authorization is required.", 
      "moreDetails": "Subscription key is not recognized."
    }
  ]
}
```

以下是可能的錯誤碼和子錯誤碼值。

|代碼|子代碼|說明
|-|-|-
|ServerError|UnexpectedError<br/>ResourceError<br/>NotImplemented|HTTP 狀態碼為 500。
|InvalidRequest|ParameterMissing<br/>ParameterInvalidValue<br/>HttpNotAllowed<br/>Blocked|只要要求的任何部分無效，Bing 就會傳回 InvalidRequest。 例如缺少必要的參數或參數值無效。<br/><br/>如果錯誤是 ParameterMissing 或 ParameterInvalidValue，則 HTTP 狀態碼為 400。<br/><br/>如果您使用的是 HTTP 通訊協定，而不是 HTTPS，Bing 會傳回 HttpNotAllowed，且 HTTP 狀態碼為 410。
|RateLimitExceeded|沒有子代碼|每當您超過每秒查詢 (QPS) 或每月查詢 (QPM) 配額時，Bing 會傳回 RateLimitExceeded。<br/><br/>如果您超過 QPS，Bing 會傳回 HTTP 狀態碼 429，如果您超過 QPM，Bing 會傳回 403。
|InvalidAuthorization|AuthorizationMissing<br/>AuthorizationRedundancy|當 Bing 無法驗證呼叫者時，Bing 會傳回 InvalidAuthorization。 例如，缺少 `Ocp-Apim-Subscription-Key` 標頭，或訂用帳戶金鑰無效。<br/><br/>如果您指定一個以上的驗證方法，則會出現備援。<br/><br/>如果錯誤是 InvalidAuthorization，則 HTTP 狀態碼為 401。
|InsufficientAuthorization|AuthorizationDisabled<br/>AuthorizationExpired|當呼叫者沒有資源存取權限時，Bing 會傳回 InsufficientAuthorization。 如果訂用帳戶金鑰已停用或已過期，則會發生此情況。 <br/><br/>如果錯誤是 InsufficientAuthorization，則 HTTP 狀態碼為 403。

## <a name="next-steps"></a>後續步驟
- [C# 快速入門](csharp.md)
- [JAVA 快速入門](java-quickstart.md)
- [JavaScript 快速入門](javascript.md)
- [Node 快速入門](node-quickstart.md)
- [Python 快速入門](python-quickstart.md)

