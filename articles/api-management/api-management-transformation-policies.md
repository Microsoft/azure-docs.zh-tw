---
title: Azure API 管理中的轉換原則 | Microsoft Docs
description: 了解 Azure API 管理中可使用的轉換原則。
services: api-management
documentationcenter: ''
author: miaojiang
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/27/2017
ms.author: apimpm
ms.openlocfilehash: 3eb9d6851c30f11980d47d4e48b158217e41995d
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2018
ms.locfileid: "30233780"
---
# <a name="api-management-transformation-policies"></a>API 管理轉換原則
本主題提供下列 API 管理原則的參考。 如需有關新增和設定原則的資訊，請參閱 [API 管理中的原則](http://go.microsoft.com/fwlink/?LinkID=398186)。

##  <a name="TransformationPolicies"></a> 轉換原則

-   [將 JSON 轉換成 XML](api-management-transformation-policies.md#ConvertJSONtoXML) - 將要求或回應內文從 JSON 轉換成 XML。

-   [將 XML 轉換成 JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) - 將要求或回應內文從 XML 轉換成 JSON。

-   [在內文中尋找並取代字串](api-management-transformation-policies.md#Findandreplacestringinbody) - 尋找要求或回應子字串，並將它取代為其他子字串。

-   [遮罩內容中的 URL](api-management-transformation-policies.md#MaskURLSContent) - 重寫 (遮罩) 回應內文的連結，使其經由閘道器指向同等的連結。

-   [設定後端服務](api-management-transformation-policies.md#SetBackendService) - 變更傳入要求的後端服務。

-   [設定本文](api-management-transformation-policies.md#SetBody) - 設定傳入和傳出要求的訊息本文。

-   [設定 HTTP 標頭](api-management-transformation-policies.md#SetHTTPheader) - 指派值給現有的回應及/或要求標頭，或加入新的回應及/或要求標頭。

-   [設定查詢字串參數](api-management-transformation-policies.md#SetQueryStringParameter) - 新增、取代值或刪除要求查詢字串參數。

-   [重寫 URL](api-management-transformation-policies.md#RewriteURL) - 將要求 URL 從公用格式轉換成 Web 服務所需的格式。

-   [使用 XSLT 轉換 XML](api-management-transformation-policies.md#XSLTransform) - 將 XSL 轉換套用至要求或回應本文中的 XML。

##  <a name="ConvertJSONtoXML"></a> 將 JSON 轉換成 XML
 `json-to-xml` 原則將要求或回應本文從 JSON 轉換成 XML。

### <a name="policy-statement"></a>原則陳述式

```xml
<json-to-xml apply="always | content-type-json" consider-accept-header="true | false" parse-date="true | false"/>
```

### <a name="example"></a>範例

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <json-to-xml apply="always" consider-accept-header="false" parse-date="false"/>
    </outbound>
</policies>
```

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|json-to-xml|根元素。|yes|

### <a name="attributes"></a>屬性

|Name|說明|必要|預設值|
|----------|-----------------|--------------|-------------|
|apply|此屬性必須設為下列其中一個值。<br /><br /> -   always - 一律套用轉換。<br />-   content-type-json - 只有當回應中的 Content-type 標頭指出 JSON 存在時才轉換。|yes|N/A|
|consider-accept-header|此屬性必須設為下列其中一個值。<br /><br /> -   true - 如果在要求的 Accept 標頭中要求 JSON，才套用轉換。<br />-   false - 一律套用轉換。|否|true|
|剖析日期|設定為 `false` 時，只會在轉換期間複製日期值|否|true|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、錯誤

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="ConvertXMLtoJSON"></a> 將 XML 轉換成 JSON
 `xml-to-json` 原則將要求或回應本文從 XML 轉換成 JSON。 此原則可用於將架構在「僅使用 XML 的後端 Web 服務」上的 API 現代化。

### <a name="policy-statement"></a>原則陳述式

```xml
<xml-to-json kind="javascript-friendly | direct" apply="always | content-type-xml" consider-accept-header="true | false"/>
```

### <a name="example"></a>範例

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <xml-to-json kind="direct" apply="always" consider-accept-header="false" />
    </outbound>
</policies>
```

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|xml-to-json|根元素。|yes|

### <a name="attributes"></a>屬性

|Name|說明|必要|預設值|
|----------|-----------------|--------------|-------------|
|kind|此屬性必須設為下列其中一個值。<br /><br /> -   javascript-friendly - 轉換後的 JSON 有 JavaScript 開發人員熟悉的格式。<br />-   direct -  | 轉換後的 JSON 可反映原始 XML 文件的結構。|yes|N/A|
|apply|此屬性必須設為下列其中一個值。<br /><br /> -   always - 一律轉換。<br />-   content-type-xml - 只有當回應中的 Content-type 標頭指出 XML 存在時才轉換。|yes|N/A|
|consider-accept-header|此屬性必須設為下列其中一個值。<br /><br /> -   true - 如果在要求的 Accept 標頭中要求 XML，才套用轉換。<br />-   false - 一律套用轉換。|否|true|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、錯誤

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="Findandreplacestringinbody"></a> 在本文中尋找並取代字串
 `find-and-replace` 原則會尋找要求或回應子字串，並取代為不同的子字串。

### <a name="policy-statement"></a>原則陳述式

```xml
<find-and-replace from="what to replace" to="replacement" />
```

### <a name="example"></a>範例

```xml
<find-and-replace from="notebook" to="laptop" />
```

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|find-and-replace|根元素。|yes|

### <a name="attributes"></a>屬性

|Name|說明|必要|預設值|
|----------|-----------------|--------------|-------------|
|from|要搜尋的字串。|yes|N/A|
|to|取代字串。 指定零長度的取代字串可移除搜尋字串。|yes|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="MaskURLSContent"></a> 遮罩內容中的 URL
 `redirect-content-urls` 原則會重寫 (遮罩) 回應本文中的連結，使其經由閘道器指向同等的連結。 使用在輸出區段中，用以重新撰寫回應本文連結，使其指向閘道。 使用在輸入區段中則效果相反。

> [!NOTE]
>  此原則不會變更任何標頭值，如 `Location` 標頭。 若要變更標頭值，請使用 [set-header](api-management-transformation-policies.md#SetHTTPheader) 原則。

### <a name="policy-statement"></a>原則陳述式

```xml
<redirect-content-urls />
```

### <a name="example"></a>範例

```xml
<redirect-content-urls />
```

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|redirect-content-urls|根元素。|yes|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="SetBackendService"></a> 設定後端服務
 使用 `set-backend-service` 原則將傳入要求重新導向至不同的後端，而不是 API 設定中為該作業指定的後端。 此原則會將傳入要求中的後端服務基底 URL 變更為原則中指定的 URL。

### <a name="policy-statement"></a>原則陳述式

```xml
<set-backend-service base-url="base URL of the backend service" />
```

### <a name="example"></a>範例

```xml
<policies>
    <inbound>
        <choose>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2013-05")">
                <set-backend-service base-url="http://contoso.com/api/8.2/" />
            </when>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2014-03")">
                <set-backend-service base-url="http://contoso.com/api/9.1/" />
            </when>
        </choose>
        <base />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
在這個範例中，設定後端服務原則會根據查詢字串中傳遞的版本值，將要求路由傳送至不同於 API 指定的後端服務。

一開始的後端服務基底 URL 是來自 API 設定。 因此要求 URL `https://contoso.azure-api.net/api/partners/15?version=2013-05&subscription-key=abcdef` 變成 `http://contoso.com/api/10.4/partners/15?version=2013-05&subscription-key=abcdef`，其中 `http://contoso.com/api/10.4/` 是 API 設定中指定的後端服務 URL。

套用 [<choose\>](api-management-advanced-policies.md#choose) 原則陳述式後，後端服務基底 URL 可能再次變更，根據版本要求查詢參數的值變為 `http://contoso.com/api/8.2` 或 `http://contoso.com/api/9.1`。 例如，如果其值為 `"2013-15"`，最終的要求 URL 會變成 `http://contoso.com/api/8.2/partners/15?version=2013-05&subscription-key=abcdef`。

如果需要進一步轉換要求，可使用其他[轉換原則](api-management-transformation-policies.md#TransformationPolicies)。 例如，將要求路由傳送到版本特定後端之後要移除版本查詢參數，可使用[設定查詢字串參數](api-management-transformation-policies.md#SetQueryStringParameter)原則移除現在變得多餘的版本屬性。

### <a name="example"></a>範例

```xml
<policies>
    <inbound>
        <set-backend-service backend-id="my-sf-service" sf-partition-key="@(context.Request.Url.Query.GetValueOrDefault("userId","")" sf-replica-type="primary" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
在此範例中，原則會將要求傳送至 Service Fabric 後端，使用 userId 查詢字串作為資料分割索引鍵，以及使用資料分割的主要複本。

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|set-backend-service|根元素。|yes|

### <a name="attributes"></a>屬性

|Name|說明|必要|預設值|
|----------|-----------------|--------------|-------------|
|base-url|新的後端服務基底 URL。|否|N/A|
|backend-id|要傳送至的後端識別碼。|否|N/A|
|sf-partition-key|僅適用於後端為 Service Fabric 服務並使用 'backend-id' 指定時。 用於從名稱解析服務解析特定資料分割。|否|N/A|
|sf-replica-type|僅適用於後端為 Service Fabric 服務並使用 'backend-id' 指定時。 控制要求應移至資料分割的主要或次要複本。 |否|N/A|
|sf-resolve-condition|僅適用於後端為 Service Fabric 服務時。 識別新的解析是否必須重複呼叫 Service Fabric 後端的條件。|否|N/A|
|sf-service-instance-name|僅適用於後端為 Service Fabric 服務時。 允許在執行階段變更服務執行個體。 |否|N/A|
|sf-listener-name|僅適用於後端為 Service Fabric 服務並使用 ‘backend-id’ 指定時。 Service Fabric Reliable Services 可讓您在服務中建立多個接聽程式。 後端 Reliable Service 有多個接聽程式時，可使用此屬性來選取特定的接聽程式。 如果未指定此屬性，API 管理會嘗試使用沒有名稱的接聽程式。 沒有名稱的接聽程式通常適用於只有一個接聽程式的 Reliable Services。 |否|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、後端

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="SetBody"></a> 設定本文
 使用 `set-body` 原則來設定傳入和傳出要求的訊息本文。 若要存取訊息本文，您可以使用 `context.Request.Body` 屬性或 `context.Response.Body`，取決於原則是在輸入或輸出區段中。

> [!IMPORTANT]
>  請注意，根據預設，當您使用 `context.Request.Body` 或 `context.Response.Body` 存取訊息本文，原始訊息本文會遺失，且必須在運算式中傳回本文加以設置。 若要保留本文內容，存取訊息時將 `preserveContent` 參數設為 `true`。 如果 `preserveContent` 設為 `true`，且運算式傳回不同的本文，則會使用傳回的本文。
>
>  使用 `set-body` 原則時請注意下列事項。
>
>  -   如果您使用 `set-body` 原則來傳回新的或更新的本文，您不需要將 `preserveContent` 設為 `true`，因為您已明確地提供新的本文內容。
> -   在輸入管線中保留回應的內容沒有意義，因為尚無回應。
> -   在輸出管線中保留要求的內容沒有意義，因為此時要求已傳送至後端。
> -   如果在沒有任何訊息本文時使用此原則，例如輸入 GET，會擲回例外狀況。

 如需詳細資訊，請參閱[內容變數](api-management-policy-expressions.md#ContextVariables)表中 `context.Request.Body`、`context.Response.Body`、`IMessage` 的部分。

### <a name="policy-statement"></a>原則陳述式

```xml
<set-body>new body value as text</set-body>
```

### <a name="examples"></a>範例

#### <a name="literal-text-example"></a>常值文字範例

```xml
<set-body>Hello world!</set-body>
```

#### <a name="example-accessing-the-body-as-a-string-note-that-we-are-preserving-the-original-request-body-so-that-we-can-access-it-later-in-the-pipeline"></a>以字串存取本文的範例。 請注意，我們會保留原始要求本文，以在稍後於管線中存取它。

```xml
<set-body>
@{ 
    string inBody = context.Request.Body.As<string>(preserveContent: true); 
    if (inBody[0] =='c') { 
        inBody[0] = 'm'; 
    } 
    return inBody; 
}
</set-body>
```

#### <a name="example-accessing-the-body-as-a-jobject-note-that-since-we-are-not-reserving-the-original-request-body-accesing-it-later-in-the-pipeline-will-result-in-an-exception"></a>以 JObject 存取本文的範例。 請注意，由於我們不會保留原始要求本文，若在稍後於管線中存取它，將會導致例外狀況。

```xml
<set-body> 
@{ 
    JObject inBody = context.Request.Body.As<JObject>(); 
    if (inBody.attribute == <tag>) { 
        inBody[0] = 'm'; 
    } 
    return inBody.ToString(); 
} 
</set-body>

```

#### <a name="filter-response-based-on-product"></a>根據產品篩選回應
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

### <a name="using-liquid-templates-with-set-body"></a>使用 Liquid 範本搭配設定本文
您可以設定讓 `set-body` 原則使用 [Liquid](https://shopify.github.io/liquid/basics/introduction/) 範本化語言來轉換要求或回應的本文。 如果您需要完全重設訊息的格式，這會非常有效。

> [!IMPORTANT]
> `set-body` 原則中使用的 Liquid 實作是在「 C# 模式」下設定。 這在進行篩選之類的操作時尤其重要。 舉例來說，使用日期篩選需要使用 Pascal 大小寫慣例和 C# 日期格是，例如：
>
> {{body.foo.startDateTime| Date:"yyyyMMddTHH:mm:ddZ"}}

> [!IMPORTANT]
> 為了使用 Liquid 範本來正確地繫結至 XML 主體，請使用 `set-header` 原則將 Content-Type 設定為 application/xml、text/xml (或任何結尾為 +xml 的類型)；如果是 JSON 主體，則必須是 application/json、text/json (或任何結尾為 +json 的類型)。

#### <a name="convert-json-to-soap-using-a-liquid-template"></a>使用 Liquid 範本將 JSON 轉換成 SOAP
```xml
<set-body template="liquid">
    <soap:Envelope xmlns="http://tempuri.org/" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
        <soap:Body>
            <GetOpenOrders>
                <cust>{{body.getOpenOrders.cust}}</cust>
            </GetOpenOrders>
        </soap:Body>
    </soap:Envelope>
</set-body>
```

#### <a name="tranform-json-using-a-liquid-template"></a>使用 Liquid 範本來轉換 JSON
```xml
{
"order": {
    "id": "{{body.customer.purchase.identifier}}",
    "summary": "{{body.customer.purchase.orderShortDesc}}"
    }
}
```

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|set-body|根元素。 包含本文文字或會傳回本文的運算式。|yes|

### <a name="properties"></a>properties

|Name|說明|必要|預設值|
|----------|-----------------|--------------|-------------|
|template|用來變更設定本文原則將在其中執行的範本化模式。 目前唯一支援的值為：<br /><br />- liquid - 設定本文原則將會使用 Liquid 範本化引擎 |否|liquid|

為了存取要求與回應的相關資訊，Liquid 範本可以繫結至具有下列屬性的內容物件： <br />
<pre>context.
    Request.
        Url
        Method
        OriginalMethod
        OriginalUrl
        IpAddress
        MatchedParameters
        HasBody
        ClientCertificates
        Headers

    Response.
        StatusCode
        Method
        Headers
Url.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString

OriginalUrl.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString
</pre>



### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="SetHTTPheader"></a> 設定 HTTP 標頭
 `set-header` 原則會指派值給現有的回應及/或要求標頭，或加入新的回應及/或要求標頭。

 將 HTTP 標頭清單插入至 HTTP 訊息中。 放在輸入管線中時，此原則會為傳遞至目標服務的要求設定 HTTP 標頭。 放在輸出管線中時，此原則會為傳送至閘道器用戶端的回應設定 HTTP 標頭。

### <a name="policy-statement"></a>原則陳述式

```xml
<set-header name="header name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple headers with the same name add additional value elements-->
</set-header>
```

### <a name="examples"></a>範例

#### <a name="example"></a>範例

```xml
<set-header name="some header name" exists-action="override">
    <value>20</value>
</set-header>
```

#### <a name="forward-context-information-to-the-backend-service"></a>將內容資訊轉寄到後端服務
 這個範例示範如何在 API 層級套用，以提供後端服務的內容資訊。 如需設定和使用此原則的示範，請觀賞 [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/)，快轉到 10:30。 12:10 處是在開發人員入口網站中呼叫作業的示範，您可以看到「原則」的運作。

```xml
<!-- Copy this snippet into the inbound element to forward some context information, user id and the region the gateway is hosted in, to the backend service for logging or evaluation -->
<set-header name="x-request-context-data" exists-action="override">
  <value>@(context.User.Id)</value>
  <value>@(context.Deployment.Region)</value>
</set-header>
```

 如需詳細資訊，請參閱[原則運算式](api-management-policy-expressions.md)和[內容變數](api-management-policy-expressions.md#ContextVariables)。

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|set-header|根元素。|yes|
|value|指定要設定之標頭的值。 若多個標頭有相同名稱，請額外加入 `value` 元素。|yes|

### <a name="properties"></a>properties

|Name|說明|必要|預設值|
|----------|-----------------|--------------|-------------|
|exists-action|指定當已指定標頭時要採取的動作。 此屬性必須具有下列其中一個值。<br /><br /> -   override - 取代現有標頭的值。<br />-   skip - 不取代現有的標頭值。<br />-   append - 將值附加至現有標頭值之後。<br />-   delete - 移除要求中的標頭。<br /><br /> 設為 `override` 時，編列多個相同名稱的項目會導致根據所有項目來設定標頭 (列出多次)；只有列出的值才會設定在結果中。|否|override|
|name|指定要設定之標頭的名稱。|yes|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出、後端、錯誤

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="SetQueryStringParameter"></a> 設定查詢字串參數
 `set-query-parameter` 原則會新增、取代值或刪除要求查詢字串參數。 可用來傳遞後端服務所需的查詢參數，為選擇性或從未存在於要求中。

### <a name="policy-statement"></a>原則陳述式

```xml
<set-query-parameter name="param name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple parameters with the same name add additional value elements-->
</set-query-parameter>
```

### <a name="examples"></a>範例

#### <a name="example"></a>範例

```xml

<set-query-parameter>
  <parameter name="api-key" exists-action="skip">
    <value>12345678901</value>
  </parameter>
  <!-- for multiple parameters with the same name add additional value elements -->
</set-query-parameter>

```

#### <a name="forward-context-information-to-the-backend-service"></a>將內容資訊轉寄到後端服務
 這個範例示範如何在 API 層級套用，以提供後端服務的內容資訊。 如需設定和使用此原則的示範，請觀賞 [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/)，快轉到 10:30。 12:10 處是在開發人員入口網站中呼叫作業的示範，您可以看到「原則」的運作。

```xml
<!-- Copy this snippet into the inbound element to forward a piece of context, product name in this example, to the backend service for logging or evaluation -->
<set-query-parameter name="x-product-name" exists-action="override">
  <value>@(context.Product.Name)</value>
</set-query-parameter>

```

 如需詳細資訊，請參閱[原則運算式](api-management-policy-expressions.md)和[內容變數](api-management-policy-expressions.md#ContextVariables)。

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|set-query-parameter|根元素。|yes|
|value|指定要設定之查詢參數的值。 若多個查詢參數有相同名稱，請額外加入 `value` 元素。|yes|

### <a name="properties"></a>properties

|Name|說明|必要|預設值|
|----------|-----------------|--------------|-------------|
|exists-action|指定當已指定查詢參數時要採取的動作。 此屬性必須具有下列其中一個值。<br /><br /> -   override - 取代現有參數的值。<br />-   skip - 不取代現有的查詢參數值。<br />-   append - 將值附加至現有查詢參數值之後。<br />-   delete - 移除要求中的查詢參數。<br /><br /> 設為 `override` 時，編列多個相同名稱的項目會導致根據所有項目來設定查詢參數 (列出多次)；只有列出的值才會設定在結果中。|否|override|
|name|指定要設定之查詢參數的名稱。|yes|N/A|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、後端

-   **原則範圍︰** 全域、產品、API、作業

##  <a name="RewriteURL"></a> 重寫 URL
 `rewrite-uri` 原則會將要求 URL 從公用格式轉換成 Web 服務所需的格式，如以下範例中所示。

-   公用 URL - `http://api.example.com/storenumber/ordernumber`

-   要求 URL - `http://api.example.com/v2/US/hardware/storenumber&ordernumber?City&State`

 應該將一般人及/或瀏覽器可理解的 URL 轉換成 Web 服務所需的 URL 格式時，可使用此原則。 只有在公開替代 URL 格式時需要套用此原則；例如，簡潔 URL、RESTful URL、使用者易記 URL 或符合 SEO 的 URL 等單純的結構式 URL 格式，不含查詢字串，只包含資源的路徑 (在配置和授權後面)。 這樣做通常是基於美觀、實用性或搜尋引擎最佳化 (SEO) 目的。

> [!NOTE]
>  使用此原則只能加入查詢字串參數。 無法在重寫 URL 中加入額外的範本路徑參數。

### <a name="policy-statement"></a>原則陳述式

```xml
<rewrite-uri template="uri template" copy-unmatched-params="true | false" />
```

### <a name="example"></a>範例

```xml
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/v2/US/hardware/{storenumber}&{ordernumber}?City=city&State=state" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put?c=d -->
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" copy-unmatched-params="false" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put -->
```

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|rewrite-uri|根元素。|yes|

### <a name="attributes"></a>屬性

|屬性|說明|必要|預設值|
|---------------|-----------------|--------------|-------------|
|template|含有任何查詢字串參數的實際 Web 服務 URL。 使用運算式時，整個值必須是運算式。|yes|N/A|
|copy-unmatched-params|指定當連入要求中查詢參數不存在於原始 URL 範本時，是否要將它新增到由重寫範本所定義的 URL|否|true|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** inbound

-   **原則範圍︰** 產品、API、作業

##  <a name="XSLTransform"></a> 使用 XSLT 轉換 XML
 `Transform XML using an XSLT` 原則會將 XSL 轉換套用至要求或回應本文中的 XML。

### <a name="policy-statement"></a>原則陳述式

```xml
<xsl-transform>
    <parameter name="User-Agent">@(context.Request.Headers.GetValueOrDefault("User-Agent","non-specified"))</parameter>
    <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        <xsl:output method="xml" indent="yes" />
        <xsl:param name="User-Agent" />
        <xsl:template match="* | @* | node()">
            <xsl:copy>
                <xsl:if test="self::* and not(parent::*)">
                    <xsl:attribute name="User-Agent">
                        <xsl:value-of select="$User-Agent" />
                    </xsl:attribute>
                </xsl:if>
                <xsl:apply-templates select="* | @* | node()" />
            </xsl:copy>
        </xsl:template>
    </xsl:stylesheet>
  </xsl-transform>
```

### <a name="example"></a>範例

```xml
<policies>
  <inbound>
      <base />
  </inbound>
  <outbound>
      <base />
      <xsl-transform>
        <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
            <xsl:output omit-xml-declaration="yes" method="xml" indent="yes" />
            <!-- Copy all nodes directly-->
            <xsl:template match="node()| @*|*">
                <xsl:copy>
                    <xsl:apply-templates select="@* | node()|*" />
                </xsl:copy>
            </xsl:template>
        </xsl:stylesheet>
    </xsl-transform>
  </outbound>
</policies>
```

### <a name="elements"></a>元素

|Name|說明|必要|
|----------|-----------------|--------------|
|xsl-transform|根元素。|yes|
|參數|用於定義轉換中使用的變數|否|
|xsl:stylesheet|根樣式表元素。 遵循 [XSLT 規格](http://www.w3.org/TR/xslt)標準定義的所有的元素和屬性|yes|

### <a name="usage"></a>使用量
 此原則可用於下列原則[區段](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[範圍](http://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)。

-   **原則區段︰** 輸入、輸出

-   **原則範圍︰** 全域、產品、API、作業

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱下列主題：

+ [API 管理中的原則](api-management-howto-policies.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)
