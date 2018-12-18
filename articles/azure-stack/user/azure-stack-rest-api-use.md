---
title: 使用 Azure Stack API | Microsoft Docs
description: 了解如何從 Azure 擷取驗證，以便對 Azure Stack 提出 API 要求。
services: azure-stack
documentationcenter: ''
author: cblackuk
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/02/2018
ms.author: mabrigg
ms.reviewer: thoroet
ms.openlocfilehash: 3b89564bf17a9884640b51faa1c3966dce93f89a
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37346785"
---
<!--  cblackuk and charliejllewellyn. This is a community contribution by cblackuk-->

# <a name="use-the-azure-stack-api"></a>使用 Azure Stack API

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用應用程式開發介面 (API)，將在 Azure Stack 雲端中新增 VM 之類的作業自動化。

使用 API 時，您的用戶端必須向 Microsoft Azure 登入端點進行驗證。 端點會傳送權杖，以用於每個傳送至 Azure Stack API 之要求的標頭中 Microsoft Azure 使用 Oauth 2.0。

本文提供使用 **cURL** 公用程式建立 Azure Stack 要求的範例。 cURL 應用程式是用於傳送資料且具有程式庫的命令列工具。 這些範例將逐步引導您進行擷取權杖以存取 Azure Stack API 的流程。 大部分的程式設計語言都會提供 Oauth 2.0 程式庫，其具有健全的權杖管理，並會處理重新整理權杖之類的工作。

檢閱搭配一般 REST 用戶端 (例如 **cURL**) 使用 Azure Stack REST API 的完整流程，可協助您了解基本要求，並顯示您可預期會在回應承載中接收到的內容。

本文不會探索適用於擷取權杖的所有選項，例如互動式登入或建立專用的應用程式識別碼。 若要取得有關這些主題的資訊，請參閱 [Azure REST API 參考](https://docs.microsoft.com/rest/api/)。

## <a name="get-a-token-from-azure"></a>從 Azure 取得權杖

建立要求本文並使用內容類型 x-www-form-urlencoded 加以格式化，以取得存取權杖。 將您的要求 POST 到 Azure REST 驗證和登入端點。

### <a name="uri"></a>URI

```bash  
POST https://login.microsoftonline.com/{tenant id}/oauth2/token
```

**租用戶識別碼**可能是：

 - 您的租用戶網域，例如 `fabrikam.onmicrosoft.com`
 - 您的租用戶識別碼，例如 `8eaed023-2b34-4da1-9baa-8bc8c9d6a491`
 - 租用戶獨立金鑰的預設值：`common`

### <a name="post-body"></a>張貼本文

```bash  
grant_type=password
&client_id=1950a258-227b-4e31-a9cf-717495945fc2
&resource=https://contoso.onmicrosoft.com/4de154de-f8a8-4017-af41-df619da68155
&username=admin@fabrikam.onmicrosoft.com
&password=Password123
&scope=openid
```

針對每個值：

 - grant_type  
    您將使用的驗證配置類型。 在此範例中，值是 `password`

 - **資源**  
    權杖存取的資源。 您可以藉由查詢 Azure Stack 管理中繼資料端點來尋找資源。 查看 **audiences** 區段

 - **Azure Stack 管理端點**  
    ```
    https://management.{region}.{Azure Stack domain}/metadata/endpoints?api-version=2015-01-01
    ```

  > [!NOTE]  
  > 如果您是嘗試存取租用戶 API 的系統管理員，請務必使用租用戶端點，例如：`https://adminmanagement.{region}.{Azure Stack domain}/metadata/endpoints?api-version=2015-01-011`  

  例如，使用 Azure Stack 開發套件作為端點：

    ```bash
    curl 'https://management.local.azurestack.external/metadata/endpoints?api-version=2015-01-01'
    ```

  回應：

  ```
  {
  "galleryEndpoint":"https://adminportal.local.azurestack.external:30015/",
  "graphEndpoint":"https://graph.windows.net/",
  "portalEndpoint":"https://adminportal.local.azurestack.external/",
  "authentication":{
      "loginEndpoint":"https://login.windows.net/",
      "audiences":["https://contoso.onmicrosoft.com/4de154de-f8a8-4017-af41-df619da68155"]
      }
  }
  ```

### <a name="example"></a>範例

  ```
  https://contoso.onmicrosoft.com/4de154de-f8a8-4017-af41-df619da68155
  ```

  client_id

  這個值會硬式編碼為預設值：

  ```
  1950a258-227b-4e31-a9cf-717495945fc2
  ```

  可供特定案例使用的替代選項：

  
  | Application | ApplicationID |
  | --------------------------------------- |:-------------------------------------------------------------:|
  | LegacyPowerShell | 0a7bdc5c-7b57-40be-9939-d4c5fc7cd417 |
  | PowerShell | 1950a258-227b-4e31-a9cf-717495945fc2 |
  | WindowsAzureActiveDirectory | 00000002-0000-0000-c000-000000000000 |
  | VisualStudio | 872cd9fa-d31f-45e0-9eab-6e460a02d1f1 |
  | AzureCLI | 04b07795-8ddb-461a-bbee-02f9e1bf7b46 |

  **username**

  例如，Azure Stack AAD 帳戶：

  ```
  azurestackadmin@fabrikam.onmicrosoft.com
  ```

  **password**

  Azure Stack AAD 系統管理員密碼。

### <a name="example"></a>範例

要求：

```
curl -X "POST" "https://login.windows.net/fabrikam.onmicrosoft.com/oauth2/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
--data-urlencode "client_id=1950a258-227b-4e31-a9cf-717495945fc2" \
--data-urlencode "grant_type=password" \
--data-urlencode "username=admin@fabrikam.onmicrosoft.com" \
--data-urlencode 'password=Password12345' \
--data-urlencode "resource=https://contoso.onmicrosoft.com/4de154de-f8a8-4017-af41-df619da68155"
```

回應：

```
{
  "token_type": "Bearer",
  "scope": "user_impersonation",
  "expires_in": "3599",
  "ext_expires_in": "0",
  "expires_on": "1512574780",
  "not_before": "1512570880",
  "resource": "https://contoso.onmicrosoft.com/4de154de-f8a8-4017-af41-df619da68155",
  "access_token": "eyJ0eXAiOi...truncated for readability..."
}
```

## <a name="api-queries"></a>API 查詢

一旦取得存取權杖之後，您需要將它當作標頭新增至每個 API 要求。 若要這樣做，您必須建立標頭**授權**且值為：`Bearer <access token>`。 例如︰

要求：

```bash  
curl -H "Authorization: Bearer eyJ0eXAiOi...truncated for readability..." 'https://adminmanagement.local.azurestack.external/subscriptions?api-version=2016-05-01'
```

回應：

```bash  
offerId : /delegatedProviders/default/offers/92F30E5D-F163-4C58-8F02-F31CFE66C21B
id : /subscriptions/800c4168-3eb1-406b-a4ca-919fe7ee42e8
subscriptionId : 800c4168-3eb1-406b-a4ca-919fe7ee42e8
tenantId : 9fea4606-7c07-4518-9f3f-8de9c52ab628
displayName : Default Provider Subscription
state : Enabled
subscriptionPolicies : @{locationPlacementId=AzureStack}
```

### <a name="url-structure-and-query-syntax"></a>URL 結構和查詢語法

一般要求 URI，由下列項目所組成：{URI 配置} :// {URI 主機} / {資源路徑} ? {query-string}

- **URI 配置**：  
此 URI 表示用來傳送要求的通訊協定。 例如，`http` 或 `https`。
- **URI 主機**：  
此主機會指定 REST 服務端點裝載所在伺服器的網域名稱或 IP 位址，例如 `graph.microsoft.com` 或 `adminmanagement.local.azurestack.external`。
- **資源路徑**：  
此路徑會指定資源或資源集合，其中可能包含服務用來判斷要選取哪些資源的多個區段。 例如：`beta/applications/00003f25-7e1f-4278-9488-efc7bac53c4a/owners` 可用來查詢應用程式集合中特定應用程式的擁有者清單。
- **查詢字串**：  
此字串提供其他簡單參數，例如 API 版本或資源選取準則。

## <a name="azure-stack-request-uri-construct"></a>Azure Stack 要求 URI 建構

```
{URI-scheme} :// {URI-host} / {subscription id} / {resource group} / {provider} / {resource-path} ? {OPTIONAL: filter-expression} {MANDATORY: api-version}
```

### <a name="uri-syntax"></a>URI 語法

```
https://adminmanagement.local.azurestack.external/{subscription id}/resourcegroups/{resource group}/providers/{provider}/{resource-path}?{api-version}
```

### <a name="query-uri-example"></a>查詢 URI 範例

```
https://adminmanagement.local.azurestack.external/subscriptions/800c4168-3eb1-406b-a4ca-919fe7ee42e8/resourcegroups/system.local/providers/microsoft.infrastructureinsights.admin/regionhealths/local/Alerts?$filter=(Properties/State eq 'Active') and (Properties/Severity eq 'Critical')&$orderby=Properties/CreatedTimestamp desc&api-version=2016-05-01"
```

## <a name="next-steps"></a>後續步驟

如需使用 Azure RESTful 端點的詳細資訊，請參閱 [Azure REST API 參考](https://docs.microsoft.com/rest/api/) \(英文\)。
