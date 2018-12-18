---
title: Azure API 管理跨網域原則 | Microsoft Docs
description: 了解可在 Azure API 管理中使用的跨網域原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 7689d277-8abe-472a-a78c-e6d4bd43455d
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/28/2017
ms.author: apimpm
ms.openlocfilehash: 590831454e8a18678e357b4824eb35a717d1fee0
ms.sourcegitcommit: b854df4fc66c73ba1dd141740a2b348de3e1e028
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2017
ms.locfileid: "26129029"
---
# <a name="api-management-cross-domain-policies"></a>API 管理跨網域原則
本主題提供下列 API 管理原則的參考。 如需有關新增和設定原則的資訊，請參閱 [API 管理中的原則](http://go.microsoft.com/fwlink/?LinkID=398186)。  
  
##  <a name="CrossDomainPolicies"></a>跨網域原則  
  
-   [允許跨網域呼叫](api-management-cross-domain-policies.md#AllowCrossDomainCalls) - 將 API 設為可供 Adobe Flash 和 Microsoft Silverlight 瀏覽器型用戶端存取。  
-   [CORS](api-management-cross-domain-policies.md#CORS) - 將跨原始來源資源分享 (CORS) 支援加入至操作或 API，以允許來自瀏覽器型用戶端的跨網域呼叫。  
-   [JSONP](api-management-cross-domain-policies.md#JSONP) - 將 JSON 與補充的 (JSONP) 支援加入至操作或 API，以允許來自 JavaScript 瀏覽器型用戶端的跨網域呼叫。  
  
##  <a name="AllowCrossDomainCalls"></a>允許跨網域呼叫  
 使用 `cross-domain` 原則以將 API 設為可供 Adobe Flash 和 Microsoft Silverlight 瀏覽器型用戶端存取。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<cross-domain>  
   <!-Policy configuration is in the Adobe cross-domain policy file format,   
      see http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html-->  
</cross-domain>  
```  
  
### <a name="example"></a>範例  
  
```xml  
<cross-domain>  
    <cross-domain-policy>  
        <allow-http-request-headers-from domain='*' headers='*' />  
    </cross-domain-policy>  
</cross-domain>  
```  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|cross-domain|根元素。 子元素必須符合 [Adobe 跨網域原則檔案規格](http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)。|yes|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** inbound  
-   **原則範圍︰** 全域  
  
##  <a name="CORS"></a>CORS  
 `cors` 原則會將跨原始來源資源分享 (CORS) 支援加入至操作或 API，以允許來自瀏覽器型用戶端的跨網域呼叫。  
  
 CORS 可讓瀏覽器和伺服器互動，以決定是否允許特定的跨原始來源要求 (例如，從網頁上的 JavaScript 對其他網域提出的 XMLHttpRequests 呼叫)。 這比只允許相同原始來源的要求更有彈性，也比允許所有跨原始來源的要求更安全。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<cors allow-credentials="false|true">  
    <allowed-origins>  
        <origin>origin uri</origin>  
    </allowed-origins>  
    <allowed-methods preflight-result-max-age="number of seconds">  
        <method>http verb</method>  
    </allowed-methods>  
    <allowed-headers>  
        <header>header name</header>  
    </allowed-headers>  
    <expose-headers>  
        <header>header name</header>  
    </expose-headers>  
</cors>  
```  
  
### <a name="example"></a>範例  
 此範例示範如何支援事前要求，例如具有自訂標頭或 GET 和 POST 以外方法的要求。 若要支援自訂標頭和其他 HTTP 動詞命令，請使用 `allowed-methods` 和 `allowed-headers` 區段，如下列範例所示。  
  
```xml  
<cors allow-credentials="true">  
    <allowed-origins>  
        <!-- Localhost useful for development -->  
        <origin>http://localhost:8080/</origin>  
        <origin>http://example.com/</origin>  
    </allowed-origins>  
    <allowed-methods preflight-result-max-age="300">  
        <method>GET</method>  
        <method>POST</method>  
        <method>PATCH</method>  
        <method>DELETE</method>  
    </allowed-methods>  
    <allowed-headers>  
        <!-- Examples below show Azure Mobile Services headers -->  
        <header>x-zumo-installation-id</header>  
        <header>x-zumo-application</header>  
        <header>x-zumo-version</header>  
        <header>x-zumo-auth</header>  
        <header>content-type</header>  
        <header>accept</header>  
    </allowed-headers>  
    <expose-headers>  
        <!-- Examples below show Azure Mobile Services headers -->  
        <header>x-zumo-installation-id</header>  
        <header>x-zumo-application</header>  
    </expose-headers>  
</cors>  
```  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|cors|根元素。|yes|N/A|  
|allowed-origins|包含可說明跨網域要求之允許來源的 `origin` 元素。 `allowed-origins` 可包含指定了 `*` 以允許任何來源的單一 `origin` 元素，或一或多個包含 URI 的 `origin` 元素。|yes|N/A|  
|來源|值可以是 `*` 以允許所有來源，或是 URI 以指定單一來源。 URI 必須包含配置、主機和連接埠。|yes|如果 URI 中省略了連接埠，則會將連接埠 80 用於 HTTP，將連接埠 443 用於 HTTPS。|  
|allowed-methods|如果允許 GET 或 POST 以外的方法，則需要此元素。 包含指定了所支援 HTTP 動詞命令的 `method` 元素。|否|如果這個區段不存在，則會支援 GET 和 POST。|  
|method|指定 HTTP 動詞命令。|如果 `allowed-methods` 區段存在，則需要至少一個 `method` 元素。|N/A|  
|allowed-headers|此元素包含指定了可包含在要求中之標頭名稱的 `header` 元素。|否|N/A|  
|expose-headers|此元素包含指定了可供用戶端存取之標頭名稱的 `header` 元素。|否|N/A|  
|頁首|指定標頭名稱。|如果 `allowed-headers` 或 `expose-headers` 區段存在，則該區段中需要至少一個 `header` 元素。|N/A|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|allow-credentials|事前回應中的 `Access-Control-Allow-Credentials` 標頭會設定為這個屬性的值，並影響用戶端是否能夠在跨網域要求中提交認證。|否|false|  
|preflight-result-max-age|事前回應中的 `Access-Control-Max-Age` 標頭會設定為這個屬性的值，並影響使用者代理程式是否能夠快取事前回應。|否|0|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** 輸入  
-   **原則範圍︰** API、作業  
  
##  <a name="JSONP"></a>JSONP  
 `jsonp` 原則會將 JSON 與補充的 (JSONP) 支援加入至作業或 API，以允許來自 JavaScript 瀏覽器型用戶端的跨網域呼叫。 JSONP 是 JavaScript 程式中使用的方法，可從位於不同網域的伺服器要求資料。 JSONP 會略過大多數網頁瀏覽器中規定必須在相同網域內才能存取網頁的限制。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<jsonp callback-parameter-name="callback function name" />  
```  
  
### <a name="example"></a>範例  
  
```xml  
<jsonp callback-parameter-name="cb" />  
```  
  
 如果呼叫方法時未指定回呼參數 ?cb=XXX，則會傳回一般 JSON (沒有函式呼叫包裝函式)。  
  
 如果加上回呼參數 `?cb=XXX`，則會傳回 JSONP 結果，在回呼函數旁邊包裝原始的 JSON 結果，例如 `XYZ('<json result goes here>');`  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|jsonp|根元素。|yes|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|callback-parameter-name|跨網域 JavaScript 函數呼叫，開頭加上函數所在的完整網域名稱。|yes|N/A|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** 輸出  
-   **原則範圍︰** 全域、產品、API、作業  
  
## <a name="next-steps"></a>後續步驟

如需使用原則的詳細資訊，請參閱︰

+ [API 管理中的原則](api-management-howto-policies.md)
+ [轉換 API](transform-api.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)   