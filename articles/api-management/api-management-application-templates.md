---
title: Azure API 管理中的應用程式範本 | Microsoft Docs
description: 了解如何在「Azure API 管理」中自訂開發人員入口網站中的「應用程式」頁面內容。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: f3122c4d-e10e-4cdf-977b-36e8f4133fc8
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/09/2017
ms.author: apimpm
ms.openlocfilehash: 40a4d159afff4db78e27f89b448e21b1d94cd378
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22982633"
---
# <a name="application-templates-in-azure-api-management"></a>Azure API 管理中的應用程式範本
「Azure API 管理」可讓您使用一組可設定開發人員入口網站頁面內容的範本，來自訂那些頁面的內容。 使用這些範本時，您可以運用 [DotLiquid](http://dotliquidmarkup.org/) 語法和您選擇的編輯器 (例如 [DotLiquid for Designers](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers))，以及一組提供的當地語系化[字串資源](api-management-template-resources.md#strings)、[字符資源](api-management-template-resources.md#glyphs)和[頁面控制項](api-management-page-controls.md)，依照您的想法自由靈活地設定頁面內容。  
  
 本節的範本可讓您自訂開發人員入口網站中「應用程式」頁面的內容。  
  
-   [應用程式清單](#ProductList)  
  
-   [應用程式](#Application)  
  
> [!NOTE]
>  下列文件中包含範例預設範本，但範本可能會因持續進行的改善而有變更。 您可以瀏覽至想要的個別範本，來檢視開發人員入口網站中的即時預設範本。 如需有關使用範本的詳細資訊，請參閱[如何使用範本自訂 API 管理開發人員入口網站](https://azure.microsoft.com/documentation/articles/api-management-developer-portal-templates/)。  
  
##  <a name="ProductList"></a> 應用程式清單  
 「應用程式清單」範本可讓您自訂開發人員入口網站中應用程式清單頁面的主體。  
  
 ![應用程式清單頁面開發人員入口網站範本](./media/api-management-application-templates/APIM-Application-List-Page-Developer-Portal-Templates.png "APIM 應用程式清單頁面開發人員入口網站範本")  
  
### <a name="default-template"></a>預設範本  
  
```xml  
<div class="row">  
    <div class="col-md-9">  
        <h2>{% localized "AppStrings|WebApplicationsHeader" %}</h2>  
    </div>  
</div>  
<div class="row">  
    <div class="col-md-12">  
    {% if applications.size > 0 %}  
        <ul class="list-unstyled">  
        {% for app in applications %}  
            <li>  
            {% if app.application.icon.url != "" %}  
                <aside>  
                    <a href="/applications/details/{{app.application.id}}"><img src="{{app.application.icon.url}}" alt="App Icon" /></a>  
                </aside>  
            {% endif %}  
                <h3><a href="/applications/details/{{app.application.id}}">{{app.application.title}}</a></h3>  
                {{app.application.description}}  
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
  
### <a name="data-model"></a>資料模型  
  
|屬性|類型|說明|  
|--------------|----------|-----------------|  
|分頁|[分頁](api-management-template-data-model-reference.md#Paging)實體。|應用程式集合的分頁資訊。|  
|[應用程式]|[應用程式](api-management-template-data-model-reference.md#Application)實體的集合。|目前使用者可看見的應用程式。|  
|CategoryName|字串|應用程式的類別。|  
  
### <a name="sample-template-data"></a>範例範本資料  
  
```json  
{  
    "Paging": {  
        "Page": 1,  
        "PageSize": 10,  
        "TotalItemCount": 1,  
        "ShowAll": false,  
        "PageCount": 1  
    },  
    "Applications": [  
        {  
            "Application": {  
                "Id": "5702b96fb16653124c8f9ba8",  
                "Title": "Contoso Calculator",  
                "Description": "A simple online calculator.",  
                "Url": null,  
                "Version": null,  
                "Requirements": "Free application with no requirements.",  
                "State": 2,  
                "RegistrationDate": "2016-04-04T18:59:00",  
                "CategoryId": 5,  
                "DeveloperId": "5702b5b0b16653124c8f9ba4",  
                "Attachments": [  
                    {  
                        "UniqueId": "a58af001-e6c3-45fd-8bc9-c60a1875c3f6",  
                        "Url": "https://apimgmtst65gdjvjrgdbfhr4.blob.core.windows.net/content/applications/a58af001-e6c3-45fd-8bc9-c60a1875c3f6.png",  
                        "Type": "Icon",  
                        "ContentType": "image/png"  
                    },  
                    {  
                        "UniqueId": "2b4fa5dd-00ff-4a8f-b1b7-51e715849ede",  
                        "Url": "https://apimgmtst65gdjvjrgdbfhr4.blob.core.windows.net/content/applications/2b4fa5dd-00ff-4a8f-b1b7-51e715849ede.png",  
                        "Type": "Screenshot",  
                        "ContentType": "image/png"  
                    }  
                ],  
                "Icon": {  
                    "UniqueId": "a58af001-e6c3-45fd-8bc9-c60a1875c3f6",  
                    "Url": "https://apimgmtst65gdjvjrgdbfhr4.blob.core.windows.net/content/applications/a58af001-e6c3-45fd-8bc9-c60a1875c3f6.png",  
                    "Type": "Icon",  
                    "ContentType": "image/png"  
                }  
            },  
            "CategoryName": "Finance"  
        }  
    ]  
}  
```  
  
##  <a name="Application"></a> 應用程式  
 「應用程式」範本可讓您自訂開發人員入口網站中應用程式頁面的主體。  
  
 ![應用程式頁面開發人員入口網站範本](./media/api-management-application-templates/APIM-Application-Page-Developer-Portal-Templates.png "APIM 應用程式頁面開發人員入口網站範本")  
  
### <a name="default-template"></a>預設範本  
  
```xml  
<h2>{{title}}</h2>  
{% if icon.url != "" %}  
<aside class="applications_aside">  
  <div class="image-placeholder">  
    <img src="{{icon.url}}" alt="Application Icon" />  
  </div>  
</aside>  
{% endif %}  
  
<article>  
  {% if url != "" %}  
    <a target="_blank" href="{{url}}">{{url}}</a>  
  {% endif %}  
  
  <p>{{description}}</p>  
  
  {% if requirements != null %}  
  <h3>{% localized "AppDetailsStrings|WebApplicationsRequirementsHeader" %}</h3>  
  <p>{{requirements}}</p>  
  {% endif %}  
  
  {% if attachments.size > 0 %}  
  <h3>{% localized "AppDetailsStrings|WebApplicationsScreenshotsHeader" %}</h3>  
    {% for screenshot in attachments %}  
      {% if screenshot.type != "Icon" %}  
      <a href="{{screenshot.url}}" data-lightbox="example-set">  
          <img src="/Developer/Applications/Thumbnail?url={{screenshot.url}}" alt='{% localized "AppDetailsStrings|WebApplicationsScreenshotAlt" %}' />  
      </a>  
      {% endif %}  
    {% endfor %}  
  {% endif %}  
</article>  
  
```  
  
### <a name="controls"></a>控制  
 `Application` 範本不允許使用任何[頁面控制項](api-management-page-controls.md)。  
  
### <a name="data-model"></a>資料模型  
 [應用程式](api-management-template-data-model-reference.md#Application)實體。  
  
### <a name="sample-template-data"></a>範例範本資料  
  
```json  
{  
    "Id": "5702b96fb16653124c8f9ba8",  
    "Title": "Contoso Calculator",  
    "Description": "A simple online calculator.",  
    "Url": null,  
    "Version": null,  
    "Requirements": "Free application with no requirements.",  
    "State": 2,  
    "RegistrationDate": "2016-04-04T18:59:00",  
    "CategoryId": 5,  
    "DeveloperId": "5702b5b0b16653124c8f9ba4",  
    "Attachments": [  
        {  
            "UniqueId": "a58af001-e6c3-45fd-8bc9-c60a1875c3f6",  
            "Url": "https://apimgmtst3aybshdqqcqrle4.blob.core.windows.net/content/applications/a58af001-e6c3-45fd-8bc9-c60a1875c3f6.png",  
            "Type": "Icon",  
            "ContentType": "image/png"  
        },  
        {  
            "UniqueId": "2b4fa5dd-00ff-4a8f-b1b7-51e715849ede",  
            "Url": "https://apimgmtst3aybshdqqcqrle4.blob.core.windows.net/content/applications/2b4fa5dd-00ff-4a8f-b1b7-51e715849ede.png",  
            "Type": "Screenshot",  
            "ContentType": "image/png"  
        }  
    ],  
    "Icon": {  
        "UniqueId": "a58af001-e6c3-45fd-8bc9-c60a1875c3f6",  
        "Url": "https://apimgmtst3aybshdqqcqrle4.blob.core.windows.net/content/applications/a58af001-e6c3-45fd-8bc9-c60a1875c3f6.png",  
        "Type": "Icon",  
        "ContentType": "image/png"  
    }  
}  
```

## <a name="next-steps"></a>後續步驟
如需有關使用範本的詳細資訊，請參閱[如何使用範本自訂 API 管理開發人員入口網站](api-management-developer-portal-templates.md)。