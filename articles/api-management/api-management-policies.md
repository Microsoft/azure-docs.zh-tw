---
title: Azure API 管理原則 | Microsoft Docs
description: 了解可在「Azure API 管理」中使用的原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/19/2017
ms.author: apimpm
ms.openlocfilehash: f617abc164761b4839b188822585eaa4eb0aacd6
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33936890"
---
# <a name="api-management-policies"></a>API 管理原則
本節提供下列「API 管理」原則的參考。 如需有關新增和設定原則的資訊，請參閱 [API 管理中的原則](api-management-howto-policies.md)。  
  
 原則是系統的強大功能，可讓發行者透過組態來變更 API 的行為。 原則是「陳述式」的集合，會在 API 的要求或回應上循序執行。 常見的「陳述式」包括從 XML 至 JSON 的格式轉換，以及利用呼叫速率限制來限制開發人員傳入的呼叫數量。 還有許多現成的原則可用。  
  
 如果原則不另行指定，則可以在任何 API 管理原則中，使用原則運算式做為屬性值或文字值。 某些原則是以原則運算式為基礎，例如[控制流程](api-management-advanced-policies.md#choose)和[設定變數](api-management-advanced-policies.md#set-variable)原則。 如需詳細資訊，請參閱[進階原則](api-management-advanced-policies.md#AdvancedPolicies)和[原則運算式](api-management-policy-expressions.md)。  
  
##  <a name="ProxyPolicies"></a> 原則  
  
-   [存取限制原則](api-management-access-restriction-policies.md#AccessRestrictionPolicies)  
    -   [檢查 HTTP 標頭](api-management-access-restriction-policies.md#CheckHTTPHeader) - 強制必須存在和/或強制採用 HTTP 標頭的值。  
    -   [依訂閱限制呼叫率](api-management-access-restriction-policies.md#LimitCallRate) - 以訂閱為單位，限制呼叫率以避免 API 使用量暴增。  
    -   [依金鑰限制呼叫率](api-management-access-restriction-policies.md#LimitCallRateByKey) - 以金鑰為單位，限制呼叫率以避免 API 使用量暴增。  
    -   [限制呼叫端 IP](api-management-access-restriction-policies.md#RestrictCallerIPs) - 篩選 (允許/拒絕) 來自特定 IP 位址和/或位址範圍的呼叫。  
    -   [依訂閱設定使用量配額](api-management-access-restriction-policies.md#SetUsageQuota) - 以訂閱為單位，讓您可以強制採用可續訂或有存留期呼叫量與 (或) 頻寬配額。  
    -   [依金鑰設定使用量配額](api-management-access-restriction-policies.md#SetUsageQuotaByKey) - 以金鑰為單位，讓您可以強制採用可續訂或有存留期呼叫量與 (或) 頻寬配額。  
    -   [驗證 JWT](api-management-access-restriction-policies.md#ValidateJWT) - 強制擷取自指定 HTTP 標頭或指定查詢參數的 JWT 必須存在且有效。  
-   [進階原則](api-management-advanced-policies.md#AdvancedPolicies)  
    -   [控制流程](api-management-advanced-policies.md#choose) - 根據布林值運算式的評估，有條件地套用原則陳述式。  
    -   [轉寄要求](api-management-advanced-policies.md#ForwardRequest) - 將要求轉寄至後端服務。  
    -   [記錄至事件中樞](api-management-advanced-policies.md#log-to-eventhub) - 將訊息以指定的格式傳送至「記錄器」實體所定義的訊息目標。  
    -   [重試](api-management-advanced-policies.md#Retry) - 重試已括住的原則陳述式執行，直到符合條件為止。 系統會在指定的時間間隔重複執行，直到指定的重試計數為止。  
    -   [傳回回應](api-management-advanced-policies.md#ReturnResponse) - 中止管線執行，並將指定的回應直接傳回呼叫者。  
    -   [傳送單向要求](api-management-advanced-policies.md#SendOneWayRequest) - 將要求傳送到指定的 URL，無須等待回應。  
    -   [傳送要求](api-management-advanced-policies.md#SendRequest) - 將要求傳送到指定的 URL。  
    -   [設定變數](api-management-advanced-policies.md#set-variable) - 將值保存在具名的 context 變數中以供日後存取。  
    -   [設定要求方法](api-management-advanced-policies.md#SetRequestMethod) - 允許您變更要求的 HTTP 方法。  
    -   [設定狀態碼](api-management-advanced-policies.md#SetStatus) - 將 HTTP 狀態碼變更為指定的值。  
    -   [追蹤](api-management-advanced-policies.md#Trace) - 將字串新增至 [API 檢查器](https://azure.microsoft.com/documentation/articles/api-management-howto-api-inspector/)輸出。  
    -   [等候](api-management-advanced-policies.md#Wait) - 等候括住的 [Send 要求](api-management-advanced-policies.md#SendRequest)、[從快取取得值](api-management-caching-policies.md#GetFromCacheByKey)或[控制流程](api-management-advanced-policies.md#choose)原則完成後再繼續進行。  
-   [驗證原則](api-management-authentication-policies.md#AuthenticationPolicies)  
    -   [使用基本驗證進行驗證](api-management-authentication-policies.md#Basic) - 使用基本驗證來驗證後端服務。  
    -   [使用用戶端憑證進行驗證](api-management-authentication-policies.md#ClientCertificate) - 使用用戶端憑證來驗證後端服務。  
-   [快取原則](api-management-caching-policies.md#CachingPolicies)  
    -   [從快取中取得](api-management-caching-policies.md#GetFromCache) - 執行快取查閱並傳回有效的快取回應 (如果有的話)。  
    -   [儲存至快取](api-management-caching-policies.md#StoreToCache) - 根據指定的快取控制組態來快取回應。  
    -   [從快取取得值](api-management-caching-policies.md#GetFromCacheByKey) - 依金鑰擷取快取的項目。  
    -   [儲存快取中的值](api-management-caching-policies.md#StoreToCacheByKey) -依金鑰儲存快取中的項目。  
    -   [移除快取中的值](api-management-caching-policies.md#RemoveCacheByKey) - 依金鑰移除快取中的項目。  
-   [跨網域原則](api-management-cross-domain-policies.md#CrossDomainPolicies)  
    -   [允許跨網域呼叫](api-management-cross-domain-policies.md#AllowCrossDomainCalls) - 將 API 設為可供 Adobe Flash 和 Microsoft Silverlight 瀏覽器型用戶端存取。  
    -   [CORS](api-management-cross-domain-policies.md#CORS) - 將跨原始來源資源分享 (CORS) 支援加入至操作或 API，以允許來自瀏覽器型用戶端的跨網域呼叫。  
    -   [JSONP](api-management-cross-domain-policies.md#JSONP) - 將 JSON 與補充的 (JSONP) 支援加入至操作或 API，以允許來自 JavaScript 瀏覽器型用戶端的跨網域呼叫。  
-   [轉換原則](api-management-transformation-policies.md#TransformationPolicies)  
    -   [將 JSON 轉換成 XML](api-management-transformation-policies.md#ConvertJSONtoXML) - 將要求或回應內文從 JSON 轉換成 XML。  
    -   [將 XML 轉換成 JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) - 將要求或回應內文從 XML 轉換成 JSON。  
    -   [在內文中尋找並取代字串](api-management-transformation-policies.md#Findandreplacestringinbody) - 尋找要求或回應子字串，並將它取代為其他子字串。  
    -   [遮罩內容中的 URL](api-management-transformation-policies.md#MaskURLSContent) - 重寫 (遮罩) 回應內文的連結，使其經由閘道器指向同等的連結。  
    -   [設定後端服務](api-management-transformation-policies.md#SetBackendService) - 變更傳入要求的後端服務。  
    -   [設定本文](api-management-transformation-policies.md#SetBody) - 設定傳入和傳出要求的訊息本文。  
    -   [設定 HTTP 標頭](api-management-transformation-policies.md#SetHTTPheader) - 指派值給現有的回應及/或要求標頭，或加入新的回應及/或要求標頭。  
    -   [設定查詢字串參數](api-management-transformation-policies.md#SetQueryStringParameter) - 新增、取代值或刪除要求查詢字串參數。  
    -   [重寫 URL](api-management-transformation-policies.md#RewriteURL) - 將要求 URL 從公用格式轉換成 Web 服務所需的格式。  
    -   [使用 XSLT 轉換 XML](api-management-transformation-policies.md#XSLTransform) - 將 XSL 轉換套用至要求或回應主體中的 XML。  



## <a name="next-steps"></a>後續步驟
如需使用原則的詳細資訊，請參閱︰

+ [API 管理中的原則](api-management-howto-policies.md)
+ [轉換 API](transform-api.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)   
