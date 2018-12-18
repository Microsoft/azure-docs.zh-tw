---
title: 連出驗證 - Azure 排程器
description: 了解如何針對 Azure 排程器設定或移除連出驗證
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam
ms.assetid: 6707f82b-7e32-401b-a960-02aae7bb59cc
ms.topic: article
ms.date: 08/15/2016
ms.openlocfilehash: 88f2fe0781bad4b652826b6a8d1961dd39b063e1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993323"
---
# <a name="outbound-authentication-for-azure-scheduler"></a>Azure 排程器的連出驗證

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 將會取代 Azure 排程器，Azure 排程器將會被淘汰。 若要排定作業，請[改為嘗試 Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md)。 

Azure 排程器作業可能必須呼叫要求驗證的服務，例如其他 Azure 服務、Salesforce.com、Facebook 與安全自訂網站。 被呼叫的服務可以決定排程器作業是否能存取要求的資源。 

排程器支援下列驗證方法： 

* *用戶端憑證*驗證 (當使用 SSL/TLS 用戶端憑證時)
* *基本*驗證
* *Active Directory OAuth* 驗證

## <a name="add-or-remove-authentication"></a>新增或移除驗證

* 若要將驗證新增到排程器作業，當您建立或更新作業時，請將 `authentication` JavaScript 物件標記法 (JSON) 子元素新增到 `request` 元素。 

  回應一律不會傳回透過 `authentication` 物件中的 PUT、PATCH 或 POST 要求傳遞到排程器服務的祕密。 
  回應會將祕密資訊設定為 Null 或可能會使用代表已驗證之實體的公用權杖。 

* 若要將驗證從排程器作業移除，請明確地在作業上執行 PUT 或 PATCH 要求，然後將 `authentication` 物件設定為 Null。 回應將不會包然任何驗證屬性。

## <a name="client-certificate"></a>用戶端憑證

### <a name="request-body---client-certificate"></a>要求主體 - 用戶端憑證

使用 `ClientCertificate` 模型新增驗證時，請在要求主體中指定這些額外元素。  

| 元素 | 必要 | 說明 |
|---------|----------|-------------|
| **authentication** (父元素) | 要用於使用 SSL 用戶端憑證的驗證物件 |
| **type** | 是 | 驗證類型。 若為 SSL 用戶端憑證，值為 `ClientCertificate`。 |
| **pfx** | 是 | PFX 檔案的 base64 編碼內容 |
| **password** | 是 | 用於存取 PFX 檔案的密碼 |
||| 

### <a name="response-body---client-certificate"></a>回應主體 - 用戶端憑證 

當傳送具有驗證資訊的要求時，回應包含下列驗證元素。

| 元素 | 說明 | 
|---------|-------------| 
| **authentication** (父元素) | 要用於使用 SSL 用戶端憑證的驗證物件 |
| **type** | 驗證類型。 若為 SSL 用戶端憑證，值為 `ClientCertificate`。 |
| **certificateThumbprint** |憑證的指紋 |
| **certificateSubjectName** |憑證的主體辨別名稱 |
| **certificateExpiration** | 憑證的到期日 |
||| 

### <a name="sample-rest-request---client-certificate"></a>範例 REST 要求 - 用戶端憑證

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "type": "clientcertificate",
          "password": "password",
          "pfx": "pfx key"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled"
  }
}
```

### <a name="sample-rest-response---client-certificate"></a>範例 REST 回應 - 用戶端憑證

```json
HTTP/1.1 200 OKCache-Control: no-cache
Pragma: no-cache
Content-Length: 858
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 56c7b40e-721a-437e-88e6-f68562a73aa8
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 1075219e-e879-4030-bc81-094e54fbabce
x-ms-routing-request-id: WESTUS:20160316T190424Z:1075219e-e879-4030-bc81-094e54fbabce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:04:23 GMT

{
  "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
  "type": "Microsoft.Scheduler/jobCollections/jobs",
  "name": "southeastasiajc/httpjob",
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "certificateThumbprint": "88105CG9DF9ADE75B835711D899296CB217D7055",
          "certificateExpiration": "2021-01-01T07:00:00Z",
          "certificateSubjectName": "CN=Scheduler Mgmt",
          "type": "ClientCertificate"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled",
    "status": {
      "nextExecutionTime": "2016-03-16T19:05:00Z",
      "executionCount": 0,
      "failureCount": 0,
      "faultedCount": 0
    }
  }
}
```

## <a name="basic"></a>基本

### <a name="request-body---basic"></a>要求主體 - 基本

使用 `Basic` 模型新增驗證時，請在要求主體中指定這些額外元素。

| 元素 | 必要 | 說明 |
|---------|----------|-------------|
| **authentication** (父元素) | 用於使用基本驗證的驗證物件 | 
| **type** | 是 | 驗證類型。 若為基本驗證，值為 `Basic`。 | 
| **username** | 是 | 要驗證的使用者名稱 | 
| **password** | 是 | 要驗證的密碼 |
|||| 

### <a name="response-body---basic"></a>回應主體 - 基本

當傳送具有驗證資訊的要求時，回應包含下列驗證元素。

| 元素 | 說明 | 
|---------|-------------|
| **authentication** (父元素) | 用於使用基本驗證的驗證物件 |
| **type** | 驗證類型。 針對基本驗證，值為 `Basic`。 |
| **username** | 已驗證的使用者名稱 |
||| 

### <a name="sample-rest-request---basic"></a>範例 REST 要求 - 基本

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 562
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "type": "basic",
          "username": "user",
          "password": "password"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled"
  }
}
```

### <a name="sample-rest-response---basic"></a>範例 REST 回應 - 基本

```json
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 701
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: a2dcb9cd-1aea-4887-8893-d81273a8cf04
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 7816f222-6ea7-468d-b919-e6ddebbd7e95
x-ms-routing-request-id: WESTUS:20160316T190506Z:7816f222-6ea7-468d-b919-e6ddebbd7e95
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:05:06 GMT

{  
   "id":"/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type":"Microsoft.Scheduler/jobCollections/jobs",
   "name":"southeastasiajc/httpjob",
   "properties":{  
      "startTime":"2015-05-14T14:10:00Z",
      "action":{  
         "request":{  
            "uri":"https://mywebserviceendpoint.com",
            "method":"GET",
            "headers":{  
               "x-ms-version":"2013-03-01"
            },
            "authentication":{  
               "username":"user1",
               "type":"Basic"
            }
         },
         "type":"Http"
      },
      "recurrence":{  
         "frequency":"Minute",
         "endTime":"2016-04-10T08:00:00Z",
         "interval":1
      },
      "state":"Enabled",
      "status":{  
         "nextExecutionTime":"2016-03-16T19:06:00Z",
         "executionCount":0,
         "failureCount":0,
         "faultedCount":0
      }
   }
}
```

## <a name="active-directory-oauth"></a>Active Directory OAuth

### <a name="request-body---active-directory-oauth"></a>要求主體 - Active Directory OAuth 

使用 `ActiveDirectoryOAuth` 模型新增驗證時，請在要求主體中指定這些額外元素。

| 元素 | 必要 | 說明 |
|---------|----------|-------------|
| **authentication** (父元素) | 是 | 用於使用 ActiveDirectoryOAuth 驗證的驗證物件 |
| **type** | 是 | 驗證類型。 若為 ActiveDirectoryOAuth 驗證，值為 `ActiveDirectoryOAuth`。 |
| **tenant** | 是 | Azure AD 租用戶的租用戶識別碼。 若要尋找 Azure AD 租用戶的租用戶識別碼，請在 Azure PowerShell 中執行 `Get-AzureAccount`。 |
| **audience** | 是 | 此值會設定為 `https://management.core.windows.net/`。 | 
| **clientId** | 是 | Azure AD 應用程式的用戶端識別碼 | 
| **secret** | 是 | 要求權杖之用戶端的祕密 | 
|||| 

### <a name="response-body---active-directory-oauth"></a>回應主體 - Active Directory OAuth

當傳送具有驗證資訊的要求時，回應包含下列驗證元素。

| 元素 | 說明 |
|---------|-------------|
| **authentication** (父元素) | 用於使用 ActiveDirectoryOAuth 驗證的驗證物件 |
| **type** | 驗證類型。 若為 ActiveDirectoryOAuth 驗證，值為 `ActiveDirectoryOAuth`。 | 
| **tenant** | Azure AD 租用戶的租用戶識別碼 |
| **audience** | 此值會設定為 `https://management.core.windows.net/`。 |
| **clientId** | Azure AD 應用程式的用戶端識別碼 |
||| 

### <a name="sample-rest-request---active-directory-oauth"></a>範例 REST 要求 - Active Directory OAuth

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 757
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "tenant":"microsoft.onmicrosoft.com",
          "audience":"https://management.core.windows.net/",
          "clientId":"dc23e764-9be6-4a33-9b9a-c46e36f0c137",
          "secret": "G6u071r8Gjw4V4KSibnb+VK4+tX399hkHaj7LOyHuj5=",
          "type":"ActiveDirectoryOAuth"
        }
      },
      "type": "Http"
    },
    "recurrence": {
      "frequency": "Minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "Enabled"
  }
}
```

### <a name="sample-rest-response---active-directory-oauth"></a>範例 REST 回應 - Active Directory OAuth

```json
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 885
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 86d8e9fd-ac0d-4bed-9420-9baba1af3251
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
x-ms-routing-request-id: WESTUS:20160316T191003Z:5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:10:02 GMT

{
   "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type": "Microsoft.Scheduler/jobCollections/jobs",
   "name": "southeastasiajc/httpjob",
   "properties": {
      "startTime": "2015-05-14T14:10:00Z",
      "action": {  
         "request": {
            "uri": "https://mywebserviceendpoint.com",
            "method": "GET",
            "headers": {  
               "x-ms-version": "2013-03-01"
            },
            "authentication": {  
               "tenant": "microsoft.onmicrosoft.com",
               "audience": "https://management.core.windows.net/",
               "clientId": "dc23e764-9be6-4a33-9b9a-c46e36f0c137",
               "type": "ActiveDirectoryOAuth"
            }
         },
         "type": "Http"
      },
      "recurrence": {  
         "frequency": "minute",
         "endTime": "2016-04-10T08:00:00Z",
         "interval": 1
      },
      "state": "Enabled",
      "status": {  
         "lastExecutionTime": "2016-03-16T19:10:00.3762123Z",
         "nextExecutionTime": "2016-03-16T19:11:00Z",
         "executionCount": 5,
         "failureCount": 5,
         "faultedCount": 1
      }
   }
}
```

## <a name="see-also"></a>另請參閱

* [Azure 排程器是什麼？](scheduler-intro.md)
* [Azure 排程器概念、術語及實體階層](scheduler-concepts-terms.md)
* [Azure 排程器限制、預設值和錯誤碼](scheduler-limits-defaults-errors.md)
* [Azure 排程器 REST API](https://msdn.microsoft.com/library/mt629143)
* [Azure 排程器 PowerShell Cmdlet 參考](scheduler-powershell-reference.md)
