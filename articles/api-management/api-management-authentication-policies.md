---
title: Azure API 管理驗證原則 | Microsoft Docs
description: 了解「Azure API 管理」中可用的驗證原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 061702a7-3a78-472b-a54a-f3b1e332490d
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/27/2017
ms.author: apimpm
ms.openlocfilehash: 4d13d9dbea9da9db5bfe9a9af85fdbf9eab1ae84
ms.sourcegitcommit: b854df4fc66c73ba1dd141740a2b348de3e1e028
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2017
ms.locfileid: "26127754"
---
# <a name="api-management-authentication-policies"></a>API 管理驗證原則
本主題提供下列「API 管理」原則的參考。 如需有關新增和設定原則的資訊，請參閱 [API 管理中的原則](http://go.microsoft.com/fwlink/?LinkID=398186)。  

##  <a name="AuthenticationPolicies"></a> 驗證原則  
  
-   [使用基本驗證進行驗證](api-management-authentication-policies.md#Basic) - 使用基本驗證來驗證後端服務。  
  
-   [使用用戶端憑證進行驗證](api-management-authentication-policies.md#ClientCertificate) - 使用用戶端憑證來驗證後端服務。  
  
##  <a name="Basic"></a> 使用基本驗證進行驗證  
 使用 `authentication-basic` 原則以利用「基本」驗證向後端服務進行驗證。 此原則會將「HTTP 授權」標頭有效地設定為與此原則中所提供認證對應的值。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<authentication-basic username="username" password="password" />  
```  
  
### <a name="example"></a>範例  
  
```xml  
<authentication-basic username="testuser" password="testpassword" />  
```  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|authentication-basic|根元素。|yes|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|username|指定「基本驗證」認證的使用者名稱。|yes|N/A|  
|password|指定「基本驗證」認證的密碼。|yes|N/A|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** inbound  
  
-   **原則範圍︰** API  
  
##  <a name="ClientCertificate"></a> 使用用戶端憑證進行驗證  
 使用 `authentication-certificate` 原則以利用用戶端憑證向後端服務進行驗證。 憑證必須先[安裝到 API 管理中](http://go.microsoft.com/fwlink/?LinkID=511599)且會以其指紋作為識別。  
  
### <a name="policy-statement"></a>原則陳述式  
  
```xml  
<authentication-certificate thumbprint="thumbprint" />  
```  
  
### <a name="example"></a>範例  
  
```xml  
<authentication-certificate thumbprint="....." />  
```  
  
### <a name="elements"></a>元素  
  
|Name|說明|必要|  
|----------|-----------------|--------------|  
|authentication-certificate|根元素。|yes|  
  
### <a name="attributes"></a>屬性  
  
|Name|說明|必要|預設值|  
|----------|-----------------|--------------|-------------|  
|thumbprint|用戶端憑證的指紋。|yes|N/A|  
  
### <a name="usage"></a>使用量  
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。  
  
-   **原則區段︰** inbound  
  
-   **原則範圍︰** API  
  

## <a name="next-steps"></a>後續步驟
如需使用原則的詳細資訊，請參閱︰

+ [API 管理中的原則](api-management-howto-policies.md)
+ [轉換 API](transform-api.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)   
