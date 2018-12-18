---
title: 在 Azure API 管理中自訂快取
description: 了解如何在 Azure API 管理中依索引鍵快取項目
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 772bc8dd-5cda-41c4-95bf-b9f6f052bc85
ms.service: api-management
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/15/2016
ms.author: apimpm
ms.openlocfilehash: 838850d38c9df51fabcf620831371bed401e9492
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29376026"
---
# <a name="custom-caching-in-azure-api-management"></a>在 Azure API 管理中自訂快取
Azure API 管理服務以資源 URL 做為索引鍵，內建對 [HTTP 回應的快取](api-management-howto-cache.md) 的支援。 可以在要求標頭中使用 `vary-by` 屬性修改索引鍵。 這很適合用於快取整個 HTTP 回應 (也稱為｢表示法｣)，但是有時候也很適合用於只快取部分表示法。 新的 [cache-lookup-value](https://msdn.microsoft.com/library/azure/dn894086.aspx#GetFromCacheByKey) 和 [cache-store-value](https://msdn.microsoft.com/library/azure/dn894086.aspx#StoreToCacheByKey) 原則提供了可儲存及擷取原則定義中任意資料的能力。 這個能力也讓先前推出的 [send-request](https://msdn.microsoft.com/library/azure/dn894085.aspx#SendRequest) 原則更有價值，因為您現在可以快取外部服務的回應。

## <a name="architecture"></a>架構
API 管理服務使用共用的個別租用戶資料快取，所以，當您相應增加為多個單位時，您仍可以存取相同的快取資料。 不過，使用多區域部署時，在每個區域內有獨立的快取。 切勿將快取視為資料存放區，資料存放區是部分資訊片段的唯一來源。 如果您這麼做，之後又決定要採用多區域部署，擁有旅遊使用者的客戶可能會失去該快取資料的存取權。

## <a name="fragment-caching"></a>片段快取
在某些案例中，傳回的回應包含的某些資料不但判斷代價昂貴，而且還會保留一段合理的長時間。 以航空公司建立的服務為例，服務提供航班訂位、航班狀態等相關資訊。如果使用者是航空公司集點方案的會員，他們也會擁有與其目前狀態和累積里程數有關的資訊。 這些使用者相關資訊可能儲存在不同的系統中，但有可能需要包含在航班狀態和訂位相關的傳回回應中。 這可以用稱為｢片段快取｣的程序做到。 可從原始伺服器傳回主要的表示法，並使用某種權杖來指示要將使用者相關資訊插入何處。 

考慮下列來自後端 API 的 JSON 回應。

```json
{
  "airline" : "Air Canada",
  "flightno" : "871",
  "status" : "ontime",
  "gate" : "B40",
  "terminal" : "2A",
  "userprofile" : "$userprofile$"
}  
```

在 `/userprofile/{userid}` 的次要資源看起來像，

```json
{ "username" : "Bob Smith", "Status" : "Gold" }
```

為了決定要包含的適當使用者資訊，API 管理需要找出使用者是誰。 此機制依存於實作。 例如，我使用 `JWT` 權杖的 `Subject` 宣告。 

```xml
<set-variable
  name="enduserid"
  value="@(context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1].AsJwt()?.Subject)" />
```

API 管理會將 `enduserid` 值儲存在內容變數中，供後續使用。 下一個步驟是判斷先前的要求是否已擷取使用者資訊並將其儲存在快取中。 為此，API 管理會使用 `cache-lookup-value` 原則。

```xml
<cache-lookup-value
key="@("userprofile-" + context.Variables["enduserid"])"
variable-name="userprofile" />
```

如果回應索引鍵值的快取中沒有任何項目，則不會建立 `userprofile` 內容變數。 API 管理會使用 `choose` 控制流程原則來檢查查閱是否成功。

```xml
<choose>
    <when condition="@(!context.Variables.ContainsKey("userprofile"))">
        <!-- If the userprofile context variable doesn’t exist, make an HTTP request to retrieve it.  -->
    </when>
</choose>
```

如果 `userprofile` 內容變數不存在，則 API 管理會發出 HTTP 要求加以擷取。

```xml
<send-request
  mode="new"
  response-variable-name="userprofileresponse"
  timeout="10"
  ignore-error="true">

  <!-- Build a URL that points to the profile for the current end-user -->
  <set-url>@(new Uri(new Uri("https://apimairlineapi.azurewebsites.net/UserProfile/"),
      (string)context.Variables["enduserid"]).AbsoluteUri)
  </set-url>
  <set-method>GET</set-method>
</send-request>
```

API 管理會使用 `enduserid` 來建構使用者設定檔資源的 URL。 一旦得到回應，API 管理就會提取回應中的本文，並將它存回內容變數中。

```xml
<set-variable
    name="userprofile"
    value="@(((IResponse)context.Variables["userprofileresponse"]).Body.As<string>())" />
```

為了避免 API 管理需要再次發出此 HTTP 要求，當相同使用者發出另一個要求時，您可以指定將使用者設定檔儲存在快取中。

```xml
<cache-store-value
    key="@("userprofile-" + context.Variables["enduserid"])"
    value="@((string)context.Variables["userprofile"])" duration="100000" />
```

API 管理會使用它原先嘗試擷取值時使用的相同索引鍵，將值儲存在快取中。 API 管理選擇的值儲存持續時間，應基於資訊變更的經常性以及使用者對過期資訊的容忍度。 

重要的是要了解，從快取擷取仍然是程序外的網路要求，並可能使要求的時間增加數十毫秒。 但若判定使用者設定檔資訊因為需要執行資料庫查詢或從多個後端彙總資訊，而會花費更多時間，其優點就會顯現出來。

此程序的最後一個步驟，是以使用者設定檔資訊更新傳回的回應。

```xml
<!-- Update response body with user profile-->
<find-and-replace
    from='"$userprofile$"'
    to="@((string)context.Variables["userprofile"])" />
```

您可以選擇在權杖中使用引號，這樣一來，即使取代沒有發生，回應仍是有效的 JSON。  

將這些步驟全部結合在一起後，最終結果是看起來像以下這個的原則。

```xml
<policies>
    <inbound>
        <!-- How you determine user identity is application dependent -->
        <set-variable
          name="enduserid"
          value="@(context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1].AsJwt()?.Subject)" />

        <!--Look for userprofile for this user in the cache -->
        <cache-lookup-value
          key="@("userprofile-" + context.Variables["enduserid"])"
          variable-name="userprofile" />

        <!-- If API Management doesn’t find it in the cache, make a request for it and store it -->
        <choose>
            <when condition="@(!context.Variables.ContainsKey("userprofile"))">
                <!-- Make HTTP request to get user profile -->
                <send-request
                  mode="new"
                  response-variable-name="userprofileresponse"
                  timeout="10"
                  ignore-error="true">

                   <!-- Build a URL that points to the profile for the current end-user -->
                    <set-url>@(new Uri(new Uri("https://apimairlineapi.azurewebsites.net/UserProfile/"),(string)context.Variables["enduserid"]).AbsoluteUri)</set-url>
                    <set-method>GET</set-method>
                </send-request>

                <!-- Store response body in context variable -->
                <set-variable
                  name="userprofile"
                  value="@(((IResponse)context.Variables["userprofileresponse"]).Body.As<string>())" />

                <!-- Store result in cache -->
                <cache-store-value
                  key="@("userprofile-" + context.Variables["enduserid"])"
                  value="@((string)context.Variables["userprofile"])"
                  duration="100000" />
            </when>
        </choose>
        <base />
    </inbound>
    <outbound>
        <!-- Update response body with user profile-->
        <find-and-replace
              from='"$userprofile$"'
              to="@((string)context.Variables["userprofile"])" />
        <base />
    </outbound>
</policies>
```

此快取方法主要用於 HTML 是在伺服器端組成的網站，讓 HTML 可以轉譯成單一頁面。 它也適用於用戶端無法執行用戶端 HTTP 快取的 API，或是不想將此責任放在用戶端的情況。

此種片段快取也可以在使用 Redis 快取伺服器的後端 Web 伺服器上進行，不過，當快取的片段來自不同後端而非主要回應時，使用 API 管理服務來執行這項工作便很實用。

## <a name="transparent-versioning"></a>透明的版本設定
隨時支援多個不同實作版本的 API 是常見的做法。 例如，為了支援不同的環境 (開發、測試、生產等)，或為了支援舊版的 API 讓 API 取用者有時間移轉至較新版本。 

其中一種做法，是將取用者目前想使用的 API 版本儲存在取用者的設定檔資料中，並呼叫適當的後端 URL，而不需要用戶端開發人員將 URL 從 `/v1/customers` 變更為 `/v2/customers`。 若要判斷特定用戶端應呼叫的正確後端 URL，必須查詢某些組態資料。 透過快取這項組態資料，API 管理可盡量減少執行此查閱的效能減損。

第一個步驟是判斷用來設定所需版本的識別碼。 在此範例中，我選擇將版本關聯至產品訂用帳戶金鑰。 

```xml
<set-variable name="clientid" value="@(context.Subscription.Key)" />
```

然後，API 管理會執行快取查閱，以查看是否已擷取所需的用戶端版本。

```xml
<cache-lookup-value
key="@("clientversion-" + context.Variables["clientid"])"
variable-name="clientversion" />
```

接著，API 管理會確認該版本是否不在快取中。

```xml
<choose>
    <when condition="@(!context.Variables.ContainsKey("clientversion"))">
```

如果 API 管理未看到該版本，表示 API 管理已加以擷取。

```xml
<send-request
    mode="new"
    response-variable-name="clientconfiguresponse"
    timeout="10"
    ignore-error="true">
            <set-url>@(new Uri(new Uri(context.Api.ServiceUrl.ToString() + "api/ClientConfig/"),(string)context.Variables["clientid"]).AbsoluteUri)</set-url>
            <set-method>GET</set-method>
</send-request>
```

從回應中擷取回應本文文字。

```xml
<set-variable
      name="clientversion"
      value="@(((IResponse)context.Variables["clientconfiguresponse"]).Body.As<string>())" />
```

將它存回快取中供日後使用。

```xml
<cache-store-value
      key="@("clientversion-" + context.Variables["clientid"])"
      value="@((string)context.Variables["clientversion"])"
      duration="100000" />
```

最後，更新後端 URL 以選取用戶端所需的服務版本。

```xml
<set-backend-service
      base-url="@(context.Api.ServiceUrl.ToString() + "api/" + (string)context.Variables["clientversion"] + "/")" />
```

完整的原則如下所示：

```xml
<inbound>
    <base />
    <set-variable name="clientid" value="@(context.Subscription.Key)" />
    <cache-lookup-value key="@("clientversion-" + context.Variables["clientid"])" variable-name="clientversion" />

    <!-- If API Management doesn’t find it in the cache, make a request for it and store it -->
    <choose>
        <when condition="@(!context.Variables.ContainsKey("clientversion"))">
            <send-request mode="new" response-variable-name="clientconfiguresponse" timeout="10" ignore-error="true">
                <set-url>@(new Uri(new Uri(context.Api.ServiceUrl.ToString() + "api/ClientConfig/"),(string)context.Variables["clientid"]).AbsoluteUri)</set-url>
                <set-method>GET</set-method>
            </send-request>
            <!-- Store response body in context variable -->
            <set-variable name="clientversion" value="@(((IResponse)context.Variables["clientconfiguresponse"]).Body.As<string>())" />
            <!-- Store result in cache -->
            <cache-store-value key="@("clientversion-" + context.Variables["clientid"])" value="@((string)context.Variables["clientversion"])" duration="100000" />
        </when>
    </choose>
    <set-backend-service base-url="@(context.Api.ServiceUrl.ToString() + "api/" + (string)context.Variables["clientversion"] + "/")" />
</inbound>
```

讓 API 取用者能夠直接控制用戶端所存取的後端版本，而無須更新和重新部署用戶端，是一種簡潔的解決方式，能夠解決許多 API 版本設定問題。

## <a name="tenant-isolation"></a>租用戶隔離
在大型、多租用戶的部署中，有些公司會在後端硬體的不同部署上建立不同的租用戶群組。 這樣可以降低後端硬體問題所影響的客戶數目。 也可以將新的軟體版本分階段推動。 在理想情況下，對 API 取用者來說這個後端架構應該是透明的。 其做法類似於透明的版本設定，因為它根據相同的後端 URL 操作技巧，使用每個 API 金鑰的組態狀態。  

您將會傳回將租用戶與指派之硬體群組建立關聯的識別碼，而不是傳回每個訂用帳戶金鑰的 API 慣用的版本。 這個識別碼可以用來建構適當的後端 URL。

## <a name="summary"></a>總結
能夠自由地使用 Azure API 管理快取來儲存任何種類的資料，可讓您有效率地存取組態資料，而這些資料可能會影響輸入要求的處理方式。 上述快取也可以用來儲存資料片段，此種片段可以增加從後端 API 傳回的回應。
