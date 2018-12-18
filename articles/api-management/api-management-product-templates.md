---
title: Azure API 管理中的產品範本 | Microsoft Docs
description: 了解如何在「Azure API 管理」開發人員入口網站中自訂產品頁面的內容。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 49f9254c-4c5f-4ed4-9c8d-798f44e805ee
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/09/2017
ms.author: apimpm
ms.openlocfilehash: dae757231d8f2ff7fcd8e032d941c0fa9f192796
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22983343"
---
# <a name="product-templates-in-azure-api-management"></a>Azure API 管理中的產品範本
「Azure API 管理」可讓您使用一組可設定開發人員入口網站頁面內容的範本，來自訂那些頁面的內容。 使用這些範本時，您可以運用 [DotLiquid](http://dotliquidmarkup.org/) 語法和您選擇的編輯器 (例如 [DotLiquid for Designers](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers))，以及一組提供的當地語系化[字串資源](api-management-template-resources.md#strings)、[字符資源](api-management-template-resources.md#glyphs)和[頁面控制項](api-management-page-controls.md)，依照您的想法自由靈活地設定頁面內容。  
  
 本節的範本可讓您自訂開發人員入口網站中產品頁面的內容。  
  
-   [產品清單](#ProductList)  
  
-   [產品](#Product)  
  
> [!NOTE]
>  下列文件中包含範例預設範本，但範本可能會因持續進行的改善而有變更。 您可以瀏覽至想要的個別範本，來檢視開發人員入口網站中的即時預設範本。 如需有關使用範本的詳細資訊，請參閱[如何使用範本自訂 API 管理開發人員入口網站](https://azure.microsoft.com/documentation/articles/api-management-developer-portal-templates/)。  
  
##  <a name="ProductList"></a>產品清單  
 **清單**範本可讓您自訂開發人員入口網站中產品清單頁面的主體。  
  
 ![產品清單](./media/api-management-product-templates/APIM_ProductsListTemplatePage.png "APIM_ProductsListTemplatePage")  
  
### <a name="default-template"></a>預設範本  
  
```xml  
<search-control></search-control>  
<div class="row">  
    <div class="col-md-9">  
        <h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>  
    </div>  
</div>  
<div class="row">  
    <div class="col-md-12">  
    {% if products.size > 0 %}  
    <ul class="list-unstyled">  
    {% for product in products %}  
        <li>  
            <h3><a href="/products/{{product.id}}">{{product.title}}</a></h3>  
            {{product.description}}  
        </li>     
    {% endfor %}  
    </ul>  
    <paging-control></paging-control>  
    {% else %}  
    {% localized "CommonResources|NoItemsToDisplay" %}  
    {% endif %}  
    </div>  
</div>  
```  
  
### <a name="controls"></a>控制  
 `Product list` 範本可能會使用下列[頁面控制項](api-management-page-controls.md)。  
  
-   [paging-control](api-management-page-controls.md#paging-control)  
  
-   [search-control](api-management-page-controls.md#search-control)  
  
### <a name="data-model"></a>資料模型  
  
|屬性|類型|說明|  
|--------------|----------|-----------------|  
|分頁|[分頁](api-management-template-data-model-reference.md#Paging)實體。|產品集合的分頁資訊。|  
|篩選|[篩選](api-management-template-data-model-reference.md#Filtering)實體。|產品清單頁面的篩選資訊。|  
|產品|[產品](api-management-template-data-model-reference.md#Product)實體的集合。|目前使用者可看見的產品。|  
  
### <a name="sample-template-data"></a>範例範本資料  
  
```json  
{  
    "Paging": {  
        "Page": 1,  
        "PageSize": 10,  
        "TotalItemCount": 2,  
        "ShowAll": false,  
        "PageCount": 1  
    },  
    "Filtering": {  
        "Pattern": null,  
        "Placeholder": "Search products"  
    },  
    "Products": [  
        {  
            "Id": "56f9445ffaf7560049060001",  
            "Title": "Starter",  
            "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",  
            "Terms": "",  
            "ProductState": 1,  
            "AllowMultipleSubscriptions": false,  
            "MultipleSubscriptionsCount": 1  
        },  
        {  
            "Id": "56f9445ffaf7560049060002",  
            "Title": "Unlimited",  
            "Description": "Subscribers have completely unlimited access to the API. Administrator approval is required.",  
            "Terms": null,  
            "ProductState": 1,  
            "AllowMultipleSubscriptions": false,  
            "MultipleSubscriptionsCount": 1  
        }  
    ]  
}  
```  
  
##  <a name="Product"></a>產品  
 **產品**範本可讓您自訂開發人員入口網站的產品頁面主體。  
  
 ![開發人員入口網站的產品頁面](./media/api-management-product-templates/APIM_ProductPage.png "APIM_ProductPage")  
  
### <a name="default-template"></a>預設範本  
  
```xml  
<h2>{{Product.Title}}</h2>  
<p>{{Product.Description}}</p>  
  
{% assign replaceString0 = '{0}' %}  
  
{% if Limits and Limits.size > 0 %}  
<h3>{% localized "ProductDetailsStrings|WebProductsUsageLimitsHeader"%}</h3>  
<ul>  
  {% for limit in Limits %}  
  <li>{{limit.DisplayName}}</li>  
  {% endfor %}  
</ul>  
{% endif %}  
  
{% if apis.size > 0 %}  
<p>  
  <b>  
    {% if apis.size == 1 %}  
    {% capture apisCountText %}{% localized "ProductDetailsStrings|TextblockSingleApisCount" %}{% endcapture %}  
    {% else %}  
    {% capture apisCountText %}{% localized "ProductDetailsStrings|TextblockMultipleApisCount" %}{% endcapture %}  
    {% endif %}  
  
    {% capture apisCount %}{{apis.size}}{% endcapture %}  
    {{ apisCountText | replace : replaceString0, apisCount }}  
  </b>  
</p>  
  
<ul>  
  {% for api in Apis %}  
  <li>  
    <a href="/docs/services/{{api.Id}}">{{api.Name}}</a>  
  </li>  
  {% endfor %}  
</ul>  
{% endif %}  
  
{% if subscriptions.size > 0 %}  
<p>  
  <b>  
    {% if subscriptions.size == 1 %}  
    {% capture subscriptionsCountText %}{% localized "ProductDetailsStrings|TextblockSingleSubscriptionsCount" %}{% endcapture %}  
    {% else %}  
    {% capture subscriptionsCountText %}{% localized "ProductDetailsStrings|TextblockMultipleSubscriptionsCount" %}{% endcapture %}  
    {% endif %}  
  
    {% capture subscriptionsCount %}{{subscriptions.size}}{% endcapture %}  
    {{ subscriptionsCountText | replace : replaceString0, subscriptionsCount }}  
  </b>  
</p>  
  
<ul>  
  {% for subscription in subscriptions %}  
  <li>  
    <a href="/developer#{{subscription.Id}}">{{subscription.DisplayName}}</a>  
  </li>  
  {% endfor %}  
</ul>  
{% endif %}  
{% if CannotAddBecauseSubscriptionNumberLimitReached %}  
<b>{% localized "ProductDetailsStrings|TextblockSubscriptionLimitReached" %}</b>  
{% elsif CannotAddBecauseMultipleSubscriptionsNotAllowed == false %}  
<subscribe-button></subscribe-button>  
{% endif %}  
```  
  
### <a name="controls"></a>控制  
 `Product list` 範本可能會使用下列[頁面控制項](api-management-page-controls.md)。  
  
-   [subscribe-button](api-management-page-controls.md#subscribe-button)  
  
### <a name="data-model"></a>資料模型  
  
|屬性|類型|說明|  
|--------------|----------|-----------------|  
|產品|[產品](api-management-template-data-model-reference.md#Product)|指定的產品。|  
|IsDeveloperSubscribed|布林值|目前的使用者是否已訂閱此產品。|  
|SubscriptionState|number|訂用帳戶的狀態。 可能的狀態為：<br /><br /> -   `0 - suspended` – 訂用帳戶已遭封鎖，而且訂閱者無法呼叫產品的任何 API。<br />-   `1 - active` – 訂用帳戶是作用中狀態。<br />-   `2 - expired` – 訂用帳戶已達到期日，因此已停用。<br />-   `3 - submitted` – 開發人員已提出訂用帳戶要求，但尚未進行核准或拒絕。<br />-   `4 - rejected` – 系統管理員已拒絕訂用帳戶要求。<br />-   `5 - cancelled` – 開發人員或系統管理員已取消訂用帳戶。|  
|限制|array|此屬性已被取代而不應該使用。|  
|DelegatedSubscriptionEnabled|布林值|此訂用帳戶是否已啟用[委派](https://azure.microsoft.com/documentation/articles/api-management-howto-setup-delegation/)。|  
|DelegatedSubscriptionUrl|字串|如果已啟用委派，則是所委派的訂用帳戶 URL。|  
|IsAgreed|布林值|如果產品有條款，目前的使用者是否已同意條款。|  
|訂用帳戶|[訂用帳戶摘要](api-management-template-data-model-reference.md#SubscriptionSummary)實體的集合。|產品的訂用帳戶。|  
|Apis|[API](api-management-template-data-model-reference.md#API) 實體的集合。|此產品中的 API。|  
|CannotAddBecauseSubscriptionNumberLimitReached|布林值|就訂用帳戶限制而言，目前的使用者是否有資格訂閱此產品。|  
|CannotAddBecauseMultipleSubscriptionsNotAllowed|布林值|就是否允許多個訂用帳戶而言，目前的使用者是否有資格訂閱此產品。|  
  
### <a name="sample-template-data"></a>範例範本資料  
  
```json  
{  
    "Product": {  
        "Id": "56f9445ffaf7560049060001",  
        "Title": "Starter",  
        "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",  
        "Terms": "",  
        "ProductState": 1,  
        "AllowMultipleSubscriptions": false,  
        "MultipleSubscriptionsCount": 1  
    },  
    "IsDeveloperSubscribed": true,  
    "SubscriptionState": 1,  
    "Limits": [],  
    "DelegatedSubscriptionEnabled": false,  
    "DelegatedSubscriptionUrl": null,  
    "IsAgreed": false,  
    "Subscriptions": [  
        {  
            "Id": "56f9445ffaf7560049070001",  
            "DisplayName": "Starter  (default)"  
        }  
    ],  
    "Apis": [  
        {  
            "id": "56f9445ffaf7560049040001",  
            "name": "Echo API",  
            "description": null,  
            "serviceUrl": "http://echoapi.cloudapp.net/api",  
            "path": "echo",  
            "protocols": [  
                2  
            ],  
            "authenticationSettings": null,  
            "subscriptionKeyParameterNames": null  
        }  
    ],  
    "CannotAddBecauseSubscriptionNumberLimitReached": false,  
    "CannotAddBecauseMultipleSubscriptionsNotAllowed": true  
}  
```

## <a name="next-steps"></a>後續步驟
如需有關使用範本的詳細資訊，請參閱[如何使用範本自訂 API 管理開發人員入口網站](api-management-developer-portal-templates.md)。