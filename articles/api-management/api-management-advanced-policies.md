---
title: Azure API 管理進階原則 | Microsoft Docs
description: 了解可在 Azure API 管理中使用的進階原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/28/2017
ms.author: apimpm
ms.openlocfilehash: dcd4d28341e766baeaf6d581a69312cc33a0282a
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2018
ms.locfileid: "30233821"
---
# <a name="api-management-advanced-policies"></a>API 管理進階原則
本主題提供下列 API 管理原則的參考。 如需有關新增和設定原則的資訊，請參閱 [API 管理中的原則](http://go.microsoft.com/fwlink/?LinkID=398186)。

##  <a name="AdvancedPolicies"></a>進階原則

-   [控制流程](api-management-advanced-policies.md#choose) - 根據布林值[運算式](api-management-policy-expressions.md)的評估結果，有條件地套用原則陳述式。
-   [轉寄要求](#ForwardRequest) - 將要求轉寄至後端服務。
-   [限制並行](#LimitConcurrency) - 防止超過指定之要求數目同時執行括住的原則。
-   [記錄至事件中樞](#log-to-eventhub) - 將指定格式的訊息傳送至記錄器實體所定義的事件中樞。
-   [Mock 回應](#mock-response) - 中止管線執行，並將模擬回應直接傳回給呼叫者。
-   [重試](#Retry) - 重試已括住的原則陳述式執行，直到符合條件為止。 系統會在指定的時間間隔重複執行，直到指定的重試計數為止。
-   [傳回回應](#ReturnResponse) - 中止管線執行，並將指定的回應直接傳回呼叫者。
-   [傳送單向要求](#SendOneWayRequest) - 將要求傳送到指定的 URL，無須等待回應。
-   [傳送要求](#SendRequest) - 將要求傳送到指定的 URL。
-   [設定 HTTP Proxy](#SetHttpProxy) - 可讓您透過 HTTP Proxy 路由轉送要求。
-   [設定要求方法](#SetRequestMethod) - 允許您變更要求的 HTTP 方法。
-   [設定狀態碼](#SetStatus) - 將 HTTP 狀態碼變更為指定的值。
-   [設定變數](api-management-advanced-policies.md#set-variable) - 保存具名[內容](api-management-policy-expressions.md#ContextVariables)變數中的值，供日後存取使用。
-   [追蹤](#Trace) - 將字串新增至 [API 檢查器](https://azure.microsoft.com/documentation/articles/api-management-howto-api-inspector/)輸出。
-   [等候](#Wait) - 等候括住的 [Send 要求](api-management-advanced-policies.md#SendRequest)、[取得快取的值](api-management-caching-policies.md#GetFromCacheByKey)或[控制流程](api-management-advanced-policies.md#choose)原則完成後再繼續。

##  <a name="choose"></a>控制流程
 `choose` 原則會根據布林運算式 (類似於 if-then-else 或程式語言中的參數建構) 的評估結果套用括住的原則陳述式。

###  <a name="ChoosePolicyStatement"></a>原則陳述式

```xml
<choose>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <otherwise>
        <!— one or more policy statements to be applied if none of the above conditions are true  -->
</otherwise>
</choose>
```

 控制流程原則至少必須包含一個 `<when/>` 元素。 `<otherwise/>` 則是選擇性元素。 `<when/>` 元素中的條件會依其在原則內的出現順序進行評估。 系統會套用條件屬性等於 `true` 的第一個 `<when/>` 元素內所括住的原則陳述式。 如果所有 `<when/>` 元素的條件屬性都是 `false`，則會套用 `<otherwise/>` 元素內所括住的原則 (如果有的話)。

### <a name="examples"></a>範例

####  <a name="ChooseExample"></a>範例
 下列範例會示範 [set-variable](api-management-advanced-policies.md#set-variable) 原則和兩個控制流程原則。

 設定變數原則位於 inbound 區段中，並且會在 `User-Agent` 要求標頭包含文字 `iPad` 或 `iPhone` 時，建立設為 true 的 `isMobile` 布林[內容](api-management-policy-expressions.md#ContextVariables)變數。

 第一個控制流程原則也位於 inbound 區段中，並且會按照條件，根據 `isMobile` 內容變數的值套用兩個[設定查詢字串參數](api-management-transformation-policies.md#SetQueryStringParameter)原則的其中一個。

 第二個控制流程原則位於 outbound 區段中，並且會按照條件，在 `isMobile` 設為 `true` 時套用[將 XML 轉換為 JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) 原則。

```xml
<policies>
    <inbound>
        <set-variable name="isMobile" value="@(context.Request.Headers["User-Agent"].Contains("iPad") || context.Request.Headers["User-Agent"].Contains("iPhone"))" />
        <base />
        <choose>
            <when condition="@(context.Variables.GetValueOrDefault<bool>("isMobile"))">
                <set-query-parameter name="mobile" exists-action="override">
                    <value>true</value>
                </set-query-parameter>
            </when>
            <otherwise>
                <set-query-parameter name="mobile" exists-action="override">
                    <value>false</value>
                </set-query-parameter>
            </otherwise>
        </choose>
    </inbound>
    <outbound>
        <base />
        <choose>
            <when condition="@(context.Variables.GetValueOrDefault<bool>("isMobile"))">
                <xml-to-json kind="direct" apply="always" consider-accept-header="false"/>
            </when>
        </choose>
    </outbound>
</policies>
```

#### <a name="example"></a>範例
 這個範例示範如何在使用 `Starter` 產品時，移除「從後端服務收到的回應」中的資料元素，藉此執行內容篩選。 如需設定和使用此原則的示範，請觀賞 [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/)，快轉到 34:30。 從 31:50 處開始觀賞用於本示範的 [The Dark Sky Forecast API](https://developer.forecast.io/) 概觀。

```xml
<!-- Copy this snippet into the outbound section to remove a number of data elements from the response received from the backend service based on the name of the api product -->
<choose>
  <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
    <set-body>@{
        var response = context.Response.Body.As<JObject>();
        foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
          response.Property (key).Remove ();
        }
        return response.ToString();
      }
    </set-body>
  </when>
</choose>
```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|choose|根元素。|yes|
|when|要用於 `choose` 原則之 `if` 或 `ifelse` 組件的條件。 如果 `choose` 原則有多個 `when` 區段，則會依序評估這些區段。 一旦 when 元素的 `condition` 評估為 `true` 後，就不會再評估後面的 `when` 條件。|yes|
|otherwise|內含沒有任何 `when` 條件評估為 `true` 時，所要使用的原則片段。|否|

### <a name="attributes"></a>屬性

|屬性|說明|必要|
|---------------|-----------------|--------------|
|condition="Boolean expression &#124; Boolean constant"|所包含之 `when` 原則陳述式受到評估時，所要評估的布林運算式或常數。|yes|

###  <a name="ChooseUsage"></a>使用方式
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="ForwardRequest"></a>轉送要求
 `forward-request` 原則會將內送要求轉送給要求[內容](api-management-policy-expressions.md#ContextVariables)中指定的後端服務。 後端服務 URL 會指定於 API [設定](https://azure.microsoft.com/documentation/articles/api-management-howto-create-apis/#configure-api-settings)中，而且可以使用[設定後端服務](api-management-transformation-policies.md)原則加以變更。

> [!NOTE]
>  移除此原則會導致要求不會轉送到後端服務中，而且當 inbound 區段中的原則一順利完成，就會立即評估 outbound 區段中的原則。

### <a name="policy-statement"></a>原則陳述式

```xml
<forward-request timeout="time in seconds" follow-redirects="true | false"/>
```

### <a name="examples"></a>範例

#### <a name="example"></a>範例
 下列 API 層級原則會將所有要求轉送至後端服務，且逾時間隔為 60 秒。

```xml
<!-- api level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <forward-request timeout="60"/>
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>範例
 此作業層級原則使用 `base` 元素，從父 API 層級範圍繼承後端原則。

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <base/>
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>範例
 此作業層級原則會明確地將所有要求轉送至後端服務 (逾時值為 120)，且不會繼承父 API 層級的後端原則。

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <forward-request timeout="120"/>
        <!-- effective policy. note the absence of <base/> -->
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>範例
 此作業層級原則不會將要求轉送到後端服務。

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <!-- no forwarding to backend -->
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|forward-request|根元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|timeout="integer"|以秒為單位的逾時間隔，後端服務的呼叫在經過此間隔後便會失敗。|否|300 秒|
|follow-redirects="true &#124; false"|指定來自後端服務的重新導向會由閘道遵循或傳回給呼叫者。|否|false|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 後端
-   **原則範圍：** 所有範圍

##  <a name="LimitConcurrency"></a> 限制並行
 `limit-concurrency` 原則可防止超過指定之要求數目在指定的時間執行括住的原則。 超出該數目時，新的要求會立即失敗，並產生「429 太多要求」狀態碼。

###  <a name="LimitConcurrencyStatement"></a>原則陳述式

```xml
<limit-concurrency key="expression" max-count="number">
        <!— nested policy statements -->
</limit-concurrency>
```

### <a name="examples"></a>範例

#### <a name="example"></a>範例
 下列範例示範如何根據內容變數的值，限制轉送至後端的要求數目。

```xml
<policies>
  <inbound>…</inbound>
  <backend>
    <limit-concurrency key="@((string)context.Variables["connectionId"])" max-count="3">
      <forward-request timeout="120"/>
    <limit-concurrency/>
  </backend>
  <outbound>…</outbound>
</policies>
```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|limit-concurrency|根元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|--------------|
|索引鍵|字串。 允許的運算式。 指定並行範圍。 可由多個原則共用。|yes|N/A|
|max-count|整數。 指定允許輸入原則的要求數目上限。|yes|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="log-to-eventhub"></a>記錄到事件中樞
 `log-to-eventhub` 原則會將指定格式的訊息傳送至記錄器實體所定義的事件中樞。 此原則正如其名，可用來儲存選取的要求或回應內容資訊，以進行線上或離線分析。

> [!NOTE]
>  如需如何設定事件中樞和記錄事件的逐步指南，請參閱[如何使用 Azure 事件中樞記錄 API 管理事件](https://azure.microsoft.com/documentation/articles/api-management-howto-log-event-hubs/)。

### <a name="policy-statement"></a>原則陳述式

```xml
<log-to-eventhub logger-id="id of the logger entity" partition-id="index of the partition where messages are sent" partition-key="value used for partition assignment">
  Expression returning a string to be logged
</log-to-eventhub>

```

### <a name="example"></a>範例
 任何字串皆可做為值來記錄到事件中樞內。 在此範例中，所有傳入呼叫的日期和時間、部署服務名稱、要求識別碼、IP 位址和作業名稱，都會記錄到以 `contoso-logger` 識別碼所註冊的事件中樞記錄器內。

```xml
<policies>
  <inbound>
    <log-to-eventhub logger-id ='contoso-logger'>
      @( string.Join(",", DateTime.UtcNow, context.Deployment.ServiceName, context.RequestId, context.Request.IpAddress, context.Operation.Name) )
    </log-to-eventhub>
  </inbound>
  <outbound>
  </outbound>
</policies>
```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|log-to-eventhub|根元素。 此元素的值是要記錄至事件中樞的字串。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|
|---------------|-----------------|--------------|
|logger-id|向 API 管理服務註冊之記錄器的識別碼。|yes|
|partition-id|指定訊息傳送目的地的資料分割索引。|選用。 如果使用 `partition-key`，就不能使用這個屬性。|
|partition-key|指定在傳送訊息時，用來指派資料分割的值。|選用。 如果使用 `partition-id`，就不能使用這個屬性。|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="mock-response"></a> Mock 回應
如名稱所示，`mock-response` 是用來模擬 API 和作業。 它會中止正常的管線執行，並將模擬回應傳回給呼叫者。 原則一律會嘗試傳回最高精確度的回應。 它偏好使用回應內容範例 (若可使用)。 當提供結構描述而無提供範例時，它會從結構描述產生範例回應。 如果沒有找到範例或結構描述，則會傳回沒有內容的回應。

### <a name="policy-statement"></a>原則陳述式

```xml
<mock-response status-code="code" content-type="media type"/>

```

### <a name="examples"></a>範例

```xml
<!-- Returns 200 OK status code. Content is based on an example or schema, if provided for this
status code. First found content type is used. If no example or schema is found, the content is empty. -->
<mock-response/>

<!-- Returns 200 OK status code. Content is based on an example or schema, if provided for this
status code and media type. If no example or schema found, the content is empty. -->
<mock-response status-code='200' content-type='application/json'/>
```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|mock-response|根元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|--------------|
|status-code|指定回應狀態碼，且用來選取對應範例或結構描述。|否|200|
|Content-Type|指定 `Content-Type` 回應標頭值，且用來選取對應範例或結構描述。|否|None|

### <a name="usage"></a>使用方式
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、錯誤

-   **原則範圍：** 所有範圍

##  <a name="Retry"></a>重試
 `retry` 原則會執行其子原則一次，然後重試子原則的執行，直到重試 `condition` 變成 `false` 或重試 `count` 用盡。

### <a name="policy-statement"></a>原則陳述式

```xml

<retry
    condition="boolean expression or literal"
    count="number of retry attempts"
    interval="retry interval in seconds"
    max-interval="maximum retry interval in seconds"
    delta="retry interval delta in seconds"
    first-fast-retry="boolean expression or literal">
        <!-- One or more child policies. No restrictions -->
</retry>

```

### <a name="example"></a>範例
 下列範例會使用指數重試演算法來重試要求轉送最多十次。 因為 `first-fast-retry` 設定為 false，所以所有重試嘗試都會使用指數重試演算法。

```xml

<retry
    condition="@(context.Response.StatusCode == 500)"
    count="10"
    interval="10"
    max-interval="100"
    delta="10"
    first-fast-retry="false">
        <forward-request />
</retry>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|retry|根元素。 可包含其他任何原則來做為其子元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|condition|布林常值或[運算式](api-management-policy-expressions.md)，指定應停止 (`false`) 還是繼續 (`true`) 重試。|yes|N/A|
|count|正數，指定要嘗試的重試次數上限。|yes|N/A|
|interval|以秒為單位的正數，指定重試嘗試之間的等待間隔。|yes|N/A|
|max-interval|以秒為單位的正數，指定重試嘗試之間的最大等待間隔。 此屬性可用來實作指數重試演算法。|否|N/A|
|delta|以秒為單位的正數，指定等待間隔的增量。 此屬性可用來實作線性和指數的重試演算法。|否|N/A|
|first-fast-retry|如果設為 `true`，則會立即執行第一次的重試嘗試。|否|`false`|

> [!NOTE]
>  當只有指定 `interval` 時，會執行**固定**間隔的重試。
> 當只有指定 `interval` 和 `delta` 時，會使用**線性**的間隔重試演算法，其中，重試之間的等待時間會根據下列公式來進行計算：`interval + (count - 1)*delta`。
> 當指定了 `interval`、`max-interval` 和 `delta` 時，則會套用**指數**的間隔重試演算法，其中，重試之間的等待時間會根據下列公式，以指數方式從 `interval` 的值增加到 `max-interval` 的值：`min(interval + (2^count - 1) * random(delta * 0.8, delta * 1.2), max-interval)`。

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。 請注意，此原則會繼承子原則的使用方式限制。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="ReturnResponse"></a>傳回回應
 `return-response` 原則會中止管線的執行，並將預設或自訂的回應傳回給呼叫者。 預設回應是 `200 OK`，且沒有本文。 透過內容變數或原則陳述式即可指定自訂回應。 若同時提供兩者，原則陳述式會先修改內容變數中包含的回應，再傳回給呼叫者。

### <a name="policy-statement"></a>原則陳述式

```xml
<return-response response-variable-name="existing context variable">
  <set-header/>
  <set-body/>
  <set-status/>
</return-response>

```

### <a name="example"></a>範例

```xml
<return-response>
   <set-status code="401" reason="Unauthorized"/>
   <set-header name="WWW-Authenticate" exists-action="override">
      <value>Bearer error="invalid_token"</value>
   </set-header>
</return-response>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|return-response|根元素。|yes|
|set-header|[set-header](api-management-transformation-policies.md#SetHTTPheader) 原則陳述式。|否|
|set-body|[set-body](api-management-transformation-policies.md#SetBody) 原則陳述式。|否|
|set-status|[set-status](api-management-advanced-policies.md#SetStatus) 原則陳述式。|否|

### <a name="attributes"></a>屬性

|屬性|說明|必要|
|---------------|-----------------|--------------|
|response-variable-name|所參考的內容變數名稱，其參考來源為 (舉例來說) 上游 [send-request](api-management-advanced-policies.md#SendRequest) 原則，且包含 `Response` 物件|選用。|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="SendOneWayRequest"></a>傳送單向要求
 `send-one-way-request` 原則會將所提供的要求傳送到指定的 URL，無須等待回應。

### <a name="policy-statement"></a>原則陳述式

```xml
<send-one-way-request mode="new | copy">
  <url>...</url>
  <method>...</method>
  <header name="" exists-action="override | skip | append | delete">...</header>
  <body>...</body>
  <authentication-certificate thumbprint="thumbprint" />
</send-one-way-request>

```

### <a name="example"></a>範例
 此一相同原則會舉例說明如何使用 `send-one-way-request` 原則，以在 HTTP 回應碼大於或等於 500 時，將訊息傳送至 Slack 聊天室。 如需此範例的詳細資訊，請參閱[使用來自 Azure API 管理服務的外部服務](https://azure.microsoft.com/documentation/articles/api-management-sample-send-request/)。

```xml
<choose>
    <when condition="@(context.Response.StatusCode >= 500)">
      <send-one-way-request mode="new">
        <set-url>https://hooks.slack.com/services/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</set-url>
        <set-method>POST</set-method>
        <set-body>@{
                return new JObject(
                        new JProperty("username","APIM Alert"),
                        new JProperty("icon_emoji", ":ghost:"),
                        new JProperty("text", String.Format("{0} {1}\nHost: {2}\n{3} {4}\n User: {5}",
                                                context.Request.Method,
                                                context.Request.Url.Path + context.Request.Url.QueryString,
                                                context.Request.Url.Host,
                                                context.Response.StatusCode,
                                                context.Response.StatusReason,
                                                context.User.Email
                                                ))
                        ).ToString();
            }</set-body>
      </send-one-way-request>
    </when>
</choose>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|send-one-way-request|根元素。|yes|
|url|要求的 URL。|mode=copy 時為 [否]；否則為 [是]。|
|method|要求的 HTTP 方法。|mode=copy 時為 [否]；否則為 [是]。|
|頁首|要求標頭。 若有多個要求標頭，請使用多個 header 元素。|否|
|body|要求本文。|否|
|authentication-certificate|[用於用戶端驗證的憑證](api-management-authentication-policies.md#ClientCertificate)|否|


### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|mode="string"|判斷這是新要求還是現行要求的複本。 在輸出模式中，mode=copy 不會初始化要求本文。|否|新增|
|name|指定要設定之標頭的名稱。|yes|N/A|
|exists-action|指定當已指定標頭時要採取的動作。 此屬性必須具有下列其中一個值。<br /><br /> -   override - 取代現有標頭的值。<br />-   skip - 不取代現有的標頭值。<br />-   append - 將值附加至現有標頭值之後。<br />-   delete - 移除要求中的標頭。<br /><br /> 設為 `override` 時，編列多個相同名稱的項目會導致根據所有項目來設定標頭 (列出多次)；只有列出的值才會設定在結果中。|否|override|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="SendRequest"></a>傳送要求
 `send-request` 原則會將所提供的要求傳送至指定的 URL，等候時間不會超過所設定的逾時值。

### <a name="policy-statement"></a>原則陳述式

```xml
<send-request mode="new|copy" response-variable-name="" timeout="60 sec" ignore-error
="false|true">
  <set-url>...</set-url>
  <set-method>...</set-method>
  <set-header name="" exists-action="override|skip|append|delete">...</set-header>
  <set-body>...</set-body>
  <authentication-certificate thumbprint="thumbprint" />
</send-request>

```

### <a name="example"></a>範例
 此範例會顯示向授權伺服器確認參考權杖的方法。 如需此範例的詳細資訊，請參閱[使用來自 Azure API 管理服務的外部服務](https://azure.microsoft.com/documentation/articles/api-management-sample-send-request/)。

```xml
<inbound>
  <!-- Extract Token from Authorization header parameter -->
  <set-variable name="token" value="@(context.Request.Headers.GetValueOrDefault("Authorization","scheme param").Split(' ').Last())" />

  <!-- Send request to Token Server to validate token (see RFC 7662) -->
  <send-request mode="new" response-variable-name="tokenstate" timeout="20" ignore-error="true">
    <set-url>https://microsoft-apiappec990ad4c76641c6aea22f566efc5a4e.azurewebsites.net/introspection</set-url>
    <set-method>POST</set-method>
    <set-header name="Authorization" exists-action="override">
      <value>basic dXNlcm5hbWU6cGFzc3dvcmQ=</value>
    </set-header>
    <set-header name="Content-Type" exists-action="override">
      <value>application/x-www-form-urlencoded</value>
    </set-header>
    <set-body>@($"token={(string)context.Variables["token"]}")</set-body>
  </send-request>

  <choose>
        <!-- Check active property in response -->
        <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
            <!-- Return 401 Unauthorized with http-problem payload -->
            <return-response>
                <set-status code="401" reason="Unauthorized" />
                <set-header name="WWW-Authenticate" exists-action="override">
                    <value>Bearer error="invalid_token"</value>
                </set-header>
            </return-response>
        </when>
    </choose>
  <base />
</inbound>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|send-request|根元素。|yes|
|url|要求的 URL。|mode=copy 時為 [否]；否則為 [是]。|
|method|要求的 HTTP 方法。|mode=copy 時為 [否]；否則為 [是]。|
|頁首|要求標頭。 若有多個要求標頭，請使用多個 header 元素。|否|
|body|要求本文。|否|
|authentication-certificate|[用於用戶端驗證的憑證](api-management-authentication-policies.md#ClientCertificate)|否|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|mode="string"|判斷這是新要求還是現行要求的複本。 在輸出模式中，mode=copy 不會初始化要求本文。|否|新增|
|response-variable-name="string"|如果不存在，則會使用 `context.Response`。|否|N/A|
|timeout="integer"|以秒為單位的逾時間隔，URL 的呼叫在經過此間隔後便會失敗。|否|60|
|ignore-error|如果為 true，則要求會導致錯誤︰<br /><br /> -   如果已指定 response-variable-name，它會包含 null 值。<br />-   如果未指定 response-variable-name，則不會更新 context.Request。|否|false|
|name|指定要設定之標頭的名稱。|yes|N/A|
|exists-action|指定當已指定標頭時要採取的動作。 此屬性必須具有下列其中一個值。<br /><br /> -   override - 取代現有標頭的值。<br />-   skip - 不取代現有的標頭值。<br />-   append - 將值附加至現有標頭值之後。<br />-   delete - 移除要求中的標頭。<br /><br /> 設為 `override` 時，編列多個相同名稱的項目會導致根據所有項目來設定標頭 (列出多次)；只有列出的值才會設定在結果中。|否|override|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="SetHttpProxy"></a> 設定 HTTP Proxy
 `proxy` 原則可讓您透過 HTTP Proxy 將要求路由轉送至後端。 在閘道與 Proxy 之間僅支援 HTTP (不是 HTTPS)。 僅限基本和 NTLM 驗證。

### <a name="policy-statement"></a>原則陳述式

```xml
<proxy url="http://hostname-or-ip:port" username="username" password="password" />

```

### <a name="example"></a>範例
請注意，其中使用[屬性](api-management-howto-properties.md)作為 username 和 password 的值，以避免將機密資訊儲存在原則文件中。

```xml
<proxy url="http://192.168.1.1:8080" username={{username}} password={{password}} />

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|proxy|根元素|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|url="string"|http://host:port 格式的 Proxy URL。|yes|N/A|
|username="string"|用於向 Proxy 驗證的使用者名稱。|否|N/A|
|password="string"|用於向 Proxy 驗證的密碼。|否|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** inbound

-   **原則範圍：** 所有範圍

##  <a name="SetRequestMethod"></a>設定要求方法
 `set-method` 原則允許您變更要求的 HTTP 要求方法。

### <a name="policy-statement"></a>原則陳述式

```xml
<set-method>METHOD</set-method>

```

### <a name="example"></a>範例
 這個使用 `set-method` 原則的相同原則會舉例說明如何在 HTTP 回應碼大於或等於 500 時，將訊息傳送至 Slack 聊天室。 如需此範例的詳細資訊，請參閱[使用來自 Azure API 管理服務的外部服務](https://azure.microsoft.com/documentation/articles/api-management-sample-send-request/)。

```xml
<choose>
    <when condition="@(context.Response.StatusCode >= 500)">
      <send-one-way-request mode="new">
        <set-url>https://hooks.slack.com/services/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</set-url>
        <set-method>POST</set-method>
        <set-body>@{
                return new JObject(
                        new JProperty("username","APIM Alert"),
                        new JProperty("icon_emoji", ":ghost:"),
                        new JProperty("text", String.Format("{0} {1}\nHost: {2}\n{3} {4}\n User: {5}",
                                                context.Request.Method,
                                                context.Request.Url.Path + context.Request.Url.QueryString,
                                                context.Request.Url.Host,
                                                context.Response.StatusCode,
                                                context.Response.StatusReason,
                                                context.User.Email
                                                ))
                        ).ToString();
            }</set-body>
      </send-one-way-request>
    </when>
</choose>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|set-method|根元素。 元素的值會指定 HTTP 方法。|yes|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、錯誤

-   **原則範圍：** 所有範圍

##  <a name="SetStatus"></a>設定狀態碼
 `set-status` 原則會將 HTTP 狀態碼設為指定值。

### <a name="policy-statement"></a>原則陳述式

```xml
<set-status code="" reason=""/>

```

### <a name="example"></a>範例
 此範例會說明如何在授權權杖無效時傳回 401 回應。 如需詳細資訊，請參閱[使用來自 Azure API 管理服務的外部服務](https://azure.microsoft.com/documentation/articles/api-management-sample-send-request/)

```xml
<choose>
  <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
    <return-response response-variable-name="existing response variable">
      <set-status code="401" reason="Unauthorized" />
      <set-header name="WWW-Authenticate" exists-action="override">
        <value>Bearer error="invalid_token"</value>
      </set-header>
    </return-response>
  </when>
</choose>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|set-status|根元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|code="integer"|要傳回的 HTTP 狀態碼。|yes|N/A|
|reason="string"|狀態碼傳回原因的描述。|yes|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸出、後端、錯誤
-   **原則範圍：** 所有範圍

##  <a name="set-variable"></a>設定變數
 `set-variable` 原則會宣告[內容](api-management-policy-expressions.md#ContextVariables)變數，並對其指派透過[運算式](api-management-policy-expressions.md)或字串常值指定的值。 包含常值的運算式會轉換成字串，且值的類型為 `System.String`。

###  <a name="set-variablePolicyStatement"></a>原則陳述式

```xml
<set-variable name="variable name" value="Expression | String literal" />
```

###  <a name="set-variableExample"></a>範例
 下列範例會示範 inbound 區段中的設定變數原則。 此設定變數原則會在 `User-Agent` 要求標頭包含文字 `iPad` 或 `iPhone` 時，建立設為 true 的 `isMobile` 布林[內容](api-management-policy-expressions.md#ContextVariables)變數。

```xml
<set-variable name="IsMobile" value="@(context.Request.Headers["User-Agent"].Contains("iPad") || context.Request.Headers["User-Agent"].Contains("iPhone"))" />
```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|set-variable|根元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|
|---------------|-----------------|--------------|
|name|變數的名稱。|yes|
|value|變數的值。 此值可為運算式或常值。|yes|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤
-   **原則範圍：** 所有範圍

###  <a name="set-variableAllowedTypes"></a>允許的類型
 `set-variable` 原則中使用的運算式必須傳回下列其中一個基本類型。

-   System.Boolean
-   System.SByte
-   System.Byte
-   System.UInt16
-   System.UInt32
-   System.UInt64
-   System.Int16
-   System.Int32
-   System.Int64
-   System.Decimal
-   System.Single
-   System.Double
-   System.Guid
-   System.String
-   System.Char
-   System.DateTime
-   System.TimeSpan
-   System.Byte?
-   System.UInt16?
-   System.UInt32?
-   System.UInt64?
-   System.Int16?
-   System.Int32?
-   System.Int64?
-   System.Decimal?
-   System.Single?
-   System.Double?
-   System.Guid?
-   System.String?
-   System.Char?
-   System.DateTime?

##  <a name="Trace"></a>追蹤
 `trace` 原則會將字串新增至 [API 檢查器](https://azure.microsoft.com/documentation/articles/api-management-howto-api-inspector/)輸出。 此原則只會在觸發追蹤時執行，也就是 `Ocp-Apim-Trace` 要求標頭存在且設為 `true` 以及 `Ocp-Apim-Subscription-Key` 要求標頭存在且含有與管理帳戶相關聯的有效金鑰時。

### <a name="policy-statement"></a>原則陳述式

```xml

<trace source="arbitrary string literal"/>
    <!-- string expression or literal -->
</trace>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|trace|根元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|來源|對追蹤檢視器有意義，並指定了訊息來源的字串常值。|yes|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍：** 所有範圍

##  <a name="Wait"></a>等候
 `wait` 原則會以平行方式執行其直屬子原則，並等候其所有或其中一個直屬子原則完成後再完成。 等候原則可擁有做為其直屬子原則[傳送要求](api-management-advanced-policies.md#SendRequest)、[從快取取得值](api-management-caching-policies.md#GetFromCacheByKey)和[控制流程](api-management-advanced-policies.md#choose)原則。

### <a name="policy-statement"></a>原則陳述式

```xml
<wait for="all|any">
  <!--Wait policy can contain send-request, cache-lookup-value,
        and choose policies as child elements -->
</wait>

```

### <a name="example"></a>範例
 在下列範例中，有兩個 `choose` 原則做為 `wait` 原則的直屬子原則。 在這些 `choose` 原則中，每一個都會以平行方式執行。 每個 `choose` 原則都會嘗試擷取快取的值。 如果有快取遺漏，便會呼叫後端服務以提供值。 在此範例中，因為 `for` 屬性設為 `all`，所以要等到其所有直屬子原則完成後，`wait` 原則才會完成。   在此範例中，內容變數 (`execute-branch-one`、`value-one`、`execute-branch-two` 和 `value-two`) 會在此範例原則的範圍之外進行宣告。

```xml
<wait for="all">
  <choose>
    <when condition="@((bool)context.Variables["execute-branch-one="])">
      <cache-lookup-value key="key-one" variable-name="value-one" />
      <choose>
        <when condition="@(!context.Variables.ContainsKey("value-one="))">
          <send-request mode="new" response-variable-name="value-one">
            <set-url>https://backend-one</set-url>
            <set-method>GET</set-method>
          </send-request>
        </when>
      </choose>
    </when>
  </choose>
  <choose>
    <when condition="@((bool)context.Variables["execute-branch-two="])">
      <cache-lookup-value key="key-two" variable-name="value-two" />
      <choose>
        <when condition="@(!context.Variables.ContainsKey("value-two="))">
          <send-request mode="new" response-variable-name="value-two">
            <set-url>https://backend-two</set-url>
            <set-method>GET</set-method>
          </send-request>
        </when>
      </choose>
    </when>
  </choose>
</wait>

```

### <a name="elements"></a>元素

|元素|說明|必要|
|-------------|-----------------|--------------|
|wait|根元素。 只能包含做為子元素 `send-request`、`cache-lookup-value` 和 `choose` 原則。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|for|決定 `wait` 原則是要等候所有直屬子原則完成或只等候一個完成。 允許的值包括：<br /><br /> -   `all` - 等候所有直屬子原則完成<br />-   any - 等候任一直屬子原則完成。 第一個直屬子原則完成後，`wait` 原則便會完成，並終止執行任何其他直屬子原則。|否|所有|

### <a name="usage"></a>使用量

此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端
-   **原則範圍：** 所有範圍

## <a name="next-steps"></a>後續步驟

如需使用原則的詳細資訊，請參閱︰
+ [API 管理中的原則](api-management-howto-policies.md)
+ [原則運算式](api-management-policy-expressions.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)
