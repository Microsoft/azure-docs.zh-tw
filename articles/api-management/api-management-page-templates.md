---
title: Azure API 管理中的頁面範本 | Microsoft Docs
description: 了解如何使用「Azure API 管理」中的一組範本來自訂開發人員入口網站頁面的內容。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: e57df269-1019-4b74-b74d-53155b809d59
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/09/2018
ms.author: apimpm
ms.openlocfilehash: 882bb27c7692f4eaf628d26f6081b1e1015be81b
ms.sourcegitcommit: 9292e15fc80cc9df3e62731bafdcb0bb98c256e1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/10/2018
ms.locfileid: "27745001"
---
# <a name="page-templates-in-azure-api-management"></a>Azure API 管理中的頁面範本
「Azure API 管理」可讓您使用一組可設定開發人員入口網站頁面內容的範本，來自訂那些頁面的內容。 使用這些範本時，您可以運用 [DotLiquid](http://dotliquidmarkup.org/) 語法和您選擇的編輯器 (例如 [DotLiquid for Designers](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers))，以及一組提供的當地語系化[字串資源](api-management-template-resources.md#strings)、[字符資源](api-management-template-resources.md#glyphs)和[頁面控制項](api-management-page-controls.md)，依照您的想法自由靈活地設定頁面內容。  
  
 本節中的範本可讓您自訂開發人員入口網站中下列頁面的內容：登入、註冊及找不到頁面。  
  
-   [登入](#SignIn)  
  
-   [註冊](#SignUp)  
  
-   [找不到頁面](#PageNotFound)  
  
> [!NOTE]
>  下列文件中包含範例預設範本，但範本可能會因持續進行的改善而有變更。 您可以瀏覽至想要的個別範本，來檢視開發人員入口網站中的即時預設範本。 如需有關使用範本的詳細資訊，請參閱[如何使用範本自訂 API 管理開發人員入口網站](https://azure.microsoft.com/documentation/articles/api-management-developer-portal-templates/)。  
  
##  <a name="SignIn"></a> 登入  
 「登入」範本可讓您自訂開發人員入口網站中的登入頁面。  
  
 ![登入頁面](./media/api-management-page-templates/APIM-Sign-In-Page-Developer-Portal-Templates.png "APIM 登入頁面開發人員入口網站範本")  
  
### <a name="default-template"></a>預設範本  
  
```xml  
<h2 class="text-center">{% localized "SigninStrings|WebAuthenticationSigninTitle" %}</h2>  
{% if registrationEnabled == true %}  
<p class="text-center">{% localized "SigninStrings|WebAuthenticationNotAMember" %}</p>  
{% endif %}  
  
<div class="row center-block ap-idp-container">  
  <div class="col-md-6">  
    {% if registrationEnabled == true %}  
        <p>{% localized "SigninStrings|WebAuthenticationSigininWithPassword" %}</p>  
    <basic-SignIn></basic-SignIn>  
    {% endif %}  
  </div>  
  
    {% if registrationEnabled != true and providers.size == 0 %}  
        {% localized "ProviderInfoStrings|TextboxExternalIdentitiesDisabled" %}  
  {% else %}  
        {% if providers.size > 0 %}  
      <div class="col-md-6">  
            <div class="providers-list">  
                <p class="text-left">  
                {% if registrationEnabled == true %}  
                    {% localized "ProviderInfoStrings|TextboxExternalIdentitiesSigninInvitation" %}  
                {% else %}  
                    {% localized "ProviderInfoStrings|TextboxExternalIdentitiesSigninInvitationPrimary" %}  
                {% endif %}  
                </p>  
        <providers></providers>  
            </div>  
    </div>  
        {% endif %}  
    {% endif %}  
  
  {% if userRegistrationTermsEnabled == true %}  
    <div class="col-md-6">  
        <div id="terms" class="modal" role="dialog" tabindex="-1">  
            <div class="modal-dialog">  
                <div class="modal-content">  
                    <div class="modal-header">  
                        <h4 class="modal-title">{% localized "SigninResources|DialogHeadingTermsOfUse" %}</h4>  
                    </div>  
                    <div class="modal-body break-all">{{userRegistrationTerms}}</div>  
                    <div class="modal-footer">  
                        <button type="button" class="btn btn-default" data-dismiss="modal">{% localized "CommonStrings|ButtonLabelClose" %}</button>  
                    </div>  
                </div>  
            </div>  
        </div>  
        <p>{% localized "SigninResources|TextblockUserRegistrationTermsProvided" %}</p>  
    </div>  
    {% endif %}  
</div>  
```  
  
### <a name="controls"></a>控制  
 此範本可能會使用下列[頁面控制項](api-management-page-controls.md)。  
  
-   [basic-signin](api-management-page-controls.md#basic-signin)  
  
-   [提供者](api-management-page-controls.md#providers)  
  
### <a name="data-model"></a>資料模型  
 [使用者登入](api-management-template-data-model-reference.md#UseSignIn)實體。  
  
### <a name="sample-template-data"></a>範例範本資料  
  
```json  
{
    "Email": null,
    "Password": null,
    "ReturnUrl": null,
    "RememberMe": false,
    "RegistrationEnabled": true,
    "DelegationEnabled": false,
    "DelegationUrl": null,
    "SsoSignUpUrl": null,
    "AuxServiceUrl": "https://portal.azure.com/#resource/subscriptions/{subscription ID}/resourceGroups/Api-Default-West-US/providers/Microsoft.ApiManagement/service/contoso5",
    "Providers": [  
        {  
            "Properties": {  
                "AuthenticationType": "Aad",  
                "Caption": "Azure Active Directory"  
            },  
            "AuthenticationType": "Aad",  
            "Caption": "Azure Active Directory"  
        }  
        ],
    "UserRegistrationTerms": null,
    "UserRegistrationTermsEnabled": false
}
```  
  
##  <a name="SignUp"></a> 註冊  
 「註冊」範本可讓您自訂開發人員入口網站中的註冊頁面。  
  
 ![註冊頁面](./media/api-management-page-templates/APIM-Sign-Up-Page-Developer-Portal-Templates.png "APIM 註冊頁面開發人員入口網站範本")  
  
### <a name="default-template"></a>預設範本  
  
```xml  
<h2 class="text-center">{% localized "SignupStrings|PageTitleSignup" %}</h2>  
<p class="text-center">  
  {% localized "SignupStrings|WebAuthenticationAlreadyAMember" %} <a href="/signin">{% localized "SignupStrings|WebAuthenticationSigninNow" %}</a>  
</p>  
  
<div class="row center-block ap-idp-container">  
  <div class="col-md-6">  
    <p>{% localized "SignupStrings|WebAuthenticationCreateNewAccount" %}</p>  
    <sign-up></sign-up>  
  </div>  
</div>  
```  
  
### <a name="controls"></a>控制  
 此範本可能會使用下列[頁面控制項](api-management-page-controls.md)。  
  
-   [sign-up](api-management-page-controls.md#sign-up)  
  
### <a name="data-model"></a>資料模型  
 [使用者註冊](api-management-template-data-model-reference.md#UserSignUp)實體。  
  
### <a name="sample-template-data"></a>範例範本資料  
  
```json  
{  
    "PasswordConfirm": null,  
    "Password": null,  
    "PasswordVerdictLevel": 0,  
    "UserRegistrationTerms": null,  
    "UserRegistrationTermsOptions": 0,  
    "ConsentAccepted": false,  
    "Email": null,  
    "FirstName": null,  
    "LastName": null,  
    "UserData": null,  
    "NameIdentifier": null,  
    "ProviderName": null  
}  
```  
  
##  <a name="PageNotFound"></a> 找不到頁面  
 「找不到頁面」範本可讓您自訂開發人員入口網站中的「找不到頁面」頁面。  
  
 ![找不到頁面](./media/api-management-page-templates/APIM-Not-Found-Page-Developer-Portal-Templates.png "APIM 找不到頁面開發人員入口網站範本")  
  
### <a name="default-template"></a>預設範本  
  
```xml  
<h2>{% localized "NotFoundStrings|PageTitleNotFound" %}</h2>  
  
<h3>{% localized "NotFoundStrings|TitlePotentialCause" %}</h3>  
<ul>  
  <li>{% localized "NotFoundStrings|TextblockPotentialCauseOldLink" %}</li>  
  <li>{% localized "NotFoundStrings|TextblockPotentialCauseMisspelledUrl" %}</li>  
</ul>  
  
<h3>{% localized "NotFoundStrings|TitlePotentialSolution" %}</h3>  
<ul>  
  <li>{% localized "NotFoundStrings|TextblockPotentialSolutionRetype" %}</li>  
  <li>  
    {% capture textPotentialSolutionStartOver %}{% localized "NotFoundStrings|TextblockPotentialSolutionStartOver" %}{% endcapture %}  
    {% capture homeLink %}<a href="/">{% localized "NotFoundStrings|LinkLabelHomePage" %}</a>{% endcapture %}  
    {% assign replaceString = '{0}' %}  
  
    {{ textPotentialSolutionStartOver | replace : replaceString, homeLink }}  
  </li>  
</ul>  
  
<p>  
  {% capture textReportProblem %}{% localized "NotFoundStrings|TextReportProblem" %}{% endcapture %}  
  {% capture emailLink %}<a href="mailto:apimgmt@microsoft.com" target="_self" title="API Management Support">{% localized "NotFoundStrings|LinkLabelSendUsEmail" %}</a>{% endcapture %}  
  {% assign replaceString = '{0}' %}  
  
  {{ textReportProblem | replace : replaceString, emailLink }}  
</p>  
```  
  
### <a name="controls"></a>控制  
 此範本可能不使用任何[頁面控制項](api-management-page-controls.md)。  
  
### <a name="data-model"></a>資料模型  
  
|屬性|類型|說明|  
|--------------|----------|-----------------|  
|referenceCode|字串|因發生內部錯誤而顯示此頁面時所產生的代碼。|  
|errorCode|字串|因發生內部錯誤而顯示此頁面時所產生的代碼。|  
|emailBody|字串|因發生內部錯誤而顯示此頁面時所產生的電子郵件本文。|  
|requestedUrl|字串|找不到頁面時所要求的 URL。|  
|referrerUrl|字串|所要求 URL 的查閱者 URL。|  
  
### <a name="sample-template-data"></a>範例範本資料  
  
```json  
{  
    "referenceCode": null,  
    "errorCode": null,  
    "emailBody": null,  
    "requestedUrl": "https://contoso5.portal.azure-api.net:443/NotFoundPage?startEditTemplate=NotFoundPage",  
    "referrerUrl": "https://contoso5.portal.azure-api.net/signup?startEditTemplate=SignUpTemplate"  
}  
```

## <a name="next-steps"></a>後續步驟
如需有關使用範本的詳細資訊，請參閱[如何使用範本自訂 API 管理開發人員入口網站](api-management-developer-portal-templates.md)。
