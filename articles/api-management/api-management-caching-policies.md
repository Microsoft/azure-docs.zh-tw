---
title: Azure API 管理快取原則 | Microsoft Docs
description: 了解可在 Azure API 管理中使用的快取原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 8147199c-24d8-439f-b2a9-da28a70a890c
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/27/2017
ms.author: apimpm
ms.openlocfilehash: f3734304bdcc4b3f0944ebf568094595eea01a4e
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39214895"
---
# <a name="api-management-caching-policies"></a>API 管理快取原則
本主題提供下列 API 管理原則的參考。 如需有關新增和設定原則的資訊，請參閱 [API 管理中的原則](http://go.microsoft.com/fwlink/?LinkID=398186)。  
  
##  <a name="CachingPolicies"></a>快取原則  
  
-   回應快取原則  
  
    -   [從快取中取得](api-management-caching-policies.md#GetFromCache) - 執行快取查閱並傳回有效的快取回應 (如果有的話)。  
    -   [儲存至快取](api-management-caching-policies.md#StoreToCache) - 根據指定的快取控制組態來快取回應。  
  
-   值快取原則  

    -   [從快取取得值](#GetFromCacheByKey) - 依金鑰擷取快取的項目。 
    -   [儲存快取中的值](#StoreToCacheByKey) -依金鑰儲存快取中的項目。 
    -   [移除快取中的值](#RemoveCacheByKey) - 依金鑰移除快取中的項目。  
  
##  <a name="GetFromCache"></a>從快取中取得  
 使用 `cache-lookup` 原則來執行快取查閱並傳回有效的快取回應 (如果有的話)。 此原則可於回應內容在一段期間維持靜態時套用。 回應快取可降低加諸於後端 Web 伺服器的頻寬和處理需求，並縮短 API 取用者所感受的延遲時間。  
  
> [!NOTE]
>  此原則必須有對應的[儲存至快取](api-management-caching-policies.md#StoreToCache)原則。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<cache-lookup vary-by-developer="true | false" vary-by-developer-groups="true | false" downstream-caching-type="none | private | public" must-revalidate="true | false" allow-private-response-caching="@(expression to evaluate)">  
  <vary-by-header>Accept</vary-by-header>  
  <!-- should be present in most cases -->  
  <vary-by-header>Accept-Charset</vary-by-header>  
  <!-- should be present in most cases -->  
  <vary-by-header>Authorization</vary-by-header>  
  <!-- should be present when allow-private-response-caching is "true"-->  
  <vary-by-header>header name</vary-by-header>  
  <!-- optional, can repeated several times -->  
  <vary-by-query-parameter>parameter name</vary-by-query-parameter>  
  <!-- optional, can repeated several times -->  
</cache-lookup>  
```  
  
### <a name="examples"></a>範例  
  
#### <a name="example"></a>範例  
  
```xml  
<policies>  
    <inbound>  
        <base />  
        <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="none" must-revalidate="true">  
            <vary-by-query-parameter>version</vary-by-query-parameter>  
        </cache-lookup>           
    </inbound>  
    <outbound>  
        <cache-store duration="seconds" />  
        <base />          
    </outbound>  
</policies>  
```  
  
#### <a name="example-using-policy-expressions"></a>使用原則運算式的範例  
 此範例說明如何設定 API 管理回應快取期間，使其符合備用服務之 `Cache-Control` 指示詞所指定的後端服務回應快取。 如需設定和使用此原則的示範，請觀賞 [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/)，快轉到 25:25。  
  
```xml  
<!-- The following cache policy snippets demonstrate how to control API Management reponse cache duration with Cache-Control headers sent by the backend service. -->  
  
<!-- Copy this snippet into the inbound section -->  
<cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="public" must-revalidate="true" >  
  <vary-by-header>Accept</vary-by-header>  
  <vary-by-header>Accept-Charset</vary-by-header>  
</cache-lookup>  
  
<!-- Copy this snippet into the outbound section. Note that cache duration is set to the max-age value provided in the Cache-Control header received from the backend service or to the deafult value of 5 min if none is found  -->  
<cache-store duration="@{  
    var header = context.Response.Headers.GetValueOrDefault("Cache-Control","");  
    var maxAge = Regex.Match(header, @"max-age=(?<maxAge>\d+)").Groups["maxAge"]?.Value;  
    return (!string.IsNullOrEmpty(maxAge))?int.Parse(maxAge):300;  
  }"  
 />  
```  
  
 如需詳細資訊，請參閱[原則運算式](api-management-policy-expressions.md)和[內容變數](api-management-policy-expressions.md#ContextVariables)。  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|cache-lookup|根元素。|是|  
|vary-by-header|根據指定標頭 (例如 Accept、Accept-Charset、Accept-Encoding、Accept-Language、Authorization、Expect、From、Host、If-Match) 的值開始快取回應。|否|  
|vary-by-query-parameter|根據指定查詢參數的值開始快取回應。 輸入單一或多個參數。 使用分號作為分隔符號。 如果未指定任何參數，則會使用所有查詢參數。|否|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|allow-private-response-caching|當設定為 `true` 時，可快取包含 Authorization 標頭的要求。|否|false|  
|downstream-caching-type|此屬性必須設為下列其中一個值。<br /><br /> -   none - 不允許下游快取。<br />-   private - 允許下游私人快取。<br />-   public - 允許私人和共用下游快取。|否|None|  
|must-revalidate|當下游快取啟用時，此屬性會開啟或關閉閘道回應中的 `must-revalidate` 快取控制指示詞。|否|true|  
|vary-by-developer|設定為 `true` 可按照開發人員索引鍵來快取回應。|是||  
|vary-by-developer-groups|設定為 `true` 可按照使用者角色來快取回應。|是||  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** inbound  
-   **原則範圍︰** API、作業、產品  
  
##  <a name="StoreToCache"></a>儲存至快取  
 `cache-store` 原則會根據指定的快取設定來快取回應。 此原則可於回應內容在一段期間維持靜態時套用。 回應快取可降低加諸於後端 Web 伺服器的頻寬和處理需求，並縮短 API 取用者所感受的延遲時間。  
  
> [!NOTE]
>  此原則必須有對應的[從快取中取得](api-management-caching-policies.md#GetFromCache)原則。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<cache-store duration="seconds" />  
```  
  
### <a name="examples"></a>範例  
  
#### <a name="example"></a>範例  
  
```xml  
<policies>  
    <inbound>  
        <base />  
          <cache-lookup vary-by-developer="true | false" vary-by-developer-groups="true | false" downstream-caching-type="none | private | public" must-revalidate="true | false">  
                <vary-by-query-parameter>parameter name</vary-by-query-parameter> <!-- optional, can repeated several times -->  
        </cache-lookup>  
    </inbound>  
    <outbound>  
        <base />  
            <cache-store duration="3600" />  
    </outbound>  
</policies>  
```  
  
#### <a name="example-using-policy-expressions"></a>使用原則運算式的範例  
 此範例說明如何設定 API 管理回應快取期間，使其符合備用服務之 `Cache-Control` 指示詞所指定的後端服務回應快取。 如需設定和使用此原則的示範，請觀賞 [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/)，快轉到 25:25。  
  
```xml  
<!-- The following cache policy snippets demonstrate how to control API Management reponse cache duration with Cache-Control headers sent by the backend service. -->  
  
<!-- Copy this snippet into the inbound section -->  
<cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="public" must-revalidate="true" >  
  <vary-by-header>Accept</vary-by-header>  
  <vary-by-header>Accept-Charset</vary-by-header>  
</cache-lookup>  
  
<!-- Copy this snippet into the outbound section. Note that cache duration is set to the max-age value provided in the Cache-Control header received from the backend service or to the deafult value of 5 min if none is found  -->  
<cache-store duration="@{  
    var header = context.Response.Headers.GetValueOrDefault("Cache-Control","");  
    var maxAge = Regex.Match(header, @"max-age=(?<maxAge>\d+)").Groups["maxAge"]?.Value;  
    return (!string.IsNullOrEmpty(maxAge))?int.Parse(maxAge):300;  
  }"  
 />  
```  
  
 如需詳細資訊，請參閱[原則運算式](api-management-policy-expressions.md)和[內容變數](api-management-policy-expressions.md#ContextVariables)。  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|cache-store|根元素。|是|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|duration|快取項目的存留時間，以秒為單位進行指定。|是|N/A|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** 輸出    
-   **原則範圍︰** API、作業、產品  
  
##  <a name="GetFromCacheByKey"></a>從快取取得值  
 使用 `cache-lookup-value` 原則以執行依索引鍵的快取查閱，並傳回快取的值。 金鑰可以具有任意字串值，而且通常會使用原則運算式來提供。  
  
> [!NOTE]
>  此原則必須有對應的[儲存快取中的值](#StoreToCacheByKey)原則。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<cache-lookup-value key="cache key value"   
    default-value="value to use if cache lookup resulted in a miss"   
    variable-name="name of a variable looked up value is assigned to" />  
```  
  
### <a name="example"></a>範例  
 如需此原則的詳細資訊和範例，請參閱[在 Azure API 管理中自訂快取](https://azure.microsoft.com/documentation/articles/api-management-sample-cache-by-key/)。  
  
```xml  
<cache-lookup-value  
    key="@("userprofile-" + context.Variables["enduserid"])"    
    variable-name="userprofile" />  
  
```  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|cache-lookup-value|根元素。|是|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|default-value|當快取索引鍵查閱沒有結果時，要指派給變數的值。 如果未指定此屬性，則會指派 `null`。|否|`null`|  
|索引鍵|要在查閱中使用的快取索引鍵值。|是|N/A|  
|variable-name|查閱成功時，要將查閱到的值指派到之[內容變數](api-management-policy-expressions.md#ContextVariables)的名稱。 如果查閱沒有結果，會對變數指派 `default-value` 屬性的值，如果省略 `default-value` 屬性，則會指派 `null`。|是|N/A|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** 輸入、輸出、後端、錯誤  
-   **原則範圍︰** 全域、API、作業、產品  
  
##  <a name="StoreToCacheByKey"></a>儲存快取中的值  
 `cache-store-value` 會依索引鍵執行快取儲存。 金鑰可以具有任意字串值，而且通常會使用原則運算式來提供。  
  
> [!NOTE]
>  此原則必須有對應的[從快取取得值](#GetFromCacheByKey)原則。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<cache-store-value key="cache key value" value="value to cache" duration="seconds" />  
```  
  
### <a name="example"></a>範例  
 如需此原則的詳細資訊和範例，請參閱[在 Azure API 管理中自訂快取](https://azure.microsoft.com/documentation/articles/api-management-sample-cache-by-key/)。  
  
```xml  
<cache-store-value  
    key="@("userprofile-" + context.Variables["enduserid"])"  
    value="@((string)context.Variables["userprofile"])" duration="100000" />  
  
```  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|cache-store-value|根元素。|是|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|duration|會針對所提供的持續時間值來快取值，以秒為單位進行指定。|是|N/A|  
|索引鍵|用來做為值儲存依據的快取索引鍵。|是|N/A|  
|value|要快取的值。|是|N/A|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** 輸入、輸出、後端、錯誤  
-   **原則範圍︰** 全域、API、作業、產品  
  
###  <a name="RemoveCacheByKey"></a>移除快取中的值  
`cache-remove-value` 會刪除依其索引鍵所識別的快取項目。 金鑰可以具有任意字串值，而且通常會使用原則運算式來提供。  
  
#### <a name="policy-statement"></a>原則陳述式  
  
```xml  
  
<cache-remove-value key="cache key value"/>  
  
```  
  
#### <a name="example"></a>範例  
  
```xml  
  
<cache-remove-value key="@("userprofile-" + context.Variables["enduserid"])"/>  
  
```  
  
#### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|cache-remove-value|根元素。|是|  
  
#### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|索引鍵|要從快取中移除之先前快取值的索引鍵。|是|N/A|  
  
#### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** 輸入、輸出、後端、錯誤  
-   **原則範圍︰** 全域、API、作業、產品  

## <a name="next-steps"></a>後續步驟

如需使用原則的詳細資訊，請參閱︰

+ [API 管理中的原則](api-management-howto-policies.md)
+ [轉換 API](transform-api.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)   
