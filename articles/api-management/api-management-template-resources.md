---
title: Azure API 管理中的範本資源 | Microsoft Docs
description: 了解在 Azure API 管理的開發人員入口網站範本中可使用的資源類型。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 51a1b4c6-a9fd-4524-9e0e-03a9800c3e94
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/09/2017
ms.author: apimpm
ms.openlocfilehash: 212e7ea7bb2ffea63c7ba210195df0da38aa8f0a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22983783"
---
# <a name="azure-api-management-template-resources"></a>Azure API 管理中的範本資源
Azure API 管理提供下列類型的資源，用於開發人員入口網站的範本。  
  
-   [字串資源](#strings)  
  
-   [圖像資源](#glyphs)  
  
##  <a name="strings"></a> 字串資源  
 Azure API 管理提供一整套完整的字串資源，可在開發人員入口網站中使用。 這些資源已當地語系化，提供 API 管理支援的所有語言版本。 預設範本集將這些資源使用在頁首、標籤、及在開發人員入口網站中會顯示的任何恆常字串。 若要在範本中使用字串資源，需提供資源字串前置詞，後面接著字串名稱，如下列範例所示。  
  
```  
{% localized "Prefix|Name" %}  
  
```  
  
 下列範例取自產品清單範本，會在頁面頂端顯示**產品**。  
  
```  
<h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>  
  
```  
  
 關於開發人員入口網站範本中可使用的字串資源，請參閱下列表格。 請使用表格名稱做為該表格中字串資源的前置詞。  
  
-   [ApisStrings](#ApisStrings)  
  
-   [ApplicationListStrings](#ApplicationListStrings)  
  
-   [AppDetailsStrings](#AppDetailsStrings)  
  
-   [AppStrings](#AppStrings)  
  
-   [CommonResources](#CommonResources)  
  
-   [CommonStrings](#CommonStrings)  
  
-   [Documentation](#Documentation)  
  
-   [ErrorPageStrings](#ErrorPageStrings)  
  
-   [IssuesStrings](#IssuesStrings)  
  
-   [NotFoundStrings](#NotFoundStrings)  
  
-   [ProductDetailsStrings](#ProductDetailsStrings)  
  
-   [ProductsStrings](#ProductsStrings)  
  
-   [ProviderInfoStrings](#ProviderInfoStrings)  
  
-   [SigninResources](#SigninResources)  
  
-   [SigninStrings](#SigninStrings)  
  
-   [SignupStrings](#SignupStrings)  
  
-   [SubscriptionListStrings](#SubscriptionListStrings)  
  
-   [SubscriptionStrings](#SubscriptionStrings)  
  
-   [UpdateProfileStrings](#UpdateProfileStrings)  
  
-   [UserProfile](#UserProfile)  
  
###  <a name="ApisStrings"></a> ApisStrings  
  
|Name|文字|  
|----------|----------|  
|PageTitleApis|API|  
  
###  <a name="AppDetailsStrings"></a> AppDetailsStrings  
  
|Name|文字|  
|----------|----------|  
|WebApplicationsDetailsTitle|應用程式預覽|  
|WebApplicationsRequirementsHeader|需求|  
|WebApplicationsScreenshotAlt|螢幕擷取畫面|  
|WebApplicationsScreenshotsHeader|螢幕擷取畫面|  
  
###  <a name="ApplicationListStrings"></a> ApplicationListStrings  
  
|Name|文字|  
|----------|----------|  
|WebDevelopersAppDeleteConfirmation|確定要移除應用程式嗎?|  
|WebDevelopersAppNotPublished|未發佈|  
|WebDevelopersAppNotSubminted|未提交|  
|WebDevelopersAppTableCategoryHeader|類別|  
|WebDevelopersAppTableNameHeader|Name|  
|WebDevelopersAppTableStateHeader|狀況|  
|WebDevelopersEditLink|Edit|  
|WebDevelopersRegisterAppLink|註冊應用程式|  
|WebDevelopersRemoveLink|移除|  
|WebDevelopersSubmitLink|提交|  
|WebDevelopersYourApplicationsHeader|您的應用程式|  
  
###  <a name="AppStrings"></a> AppStrings  
  
|Name|文字|  
|----------|----------|  
|WebApplicationsHeader|[應用程式]|  
  
###  <a name="CommonResources"></a> CommonResources  
  
|Name|文字|  
|----------|----------|  
|NoItemsToDisplay|找不到結果。|  
|GeneralExceptionMessage|某些項目不正確。 可能是暫時性的問題或有錯誤。 請再試一次。|  
|GeneralJsonExceptionMessage|某些項目不正確。 可能是暫時性的問題或有錯誤。 請重新載入頁面，然後再試一次。|  
|ConfirmationMessageUnsavedChanges|有一些變更未儲存。 確定要取消並捨棄變更嗎?|  
|AzureActiveDirectory|Azure Active Directory|  
|HttpLargeRequestMessage|HTTP 要求本文太大。|  
  
###  <a name="CommonStrings"></a> CommonStrings  
  
|Name|文字|  
|----------|----------|  
|ButtonLabelCancel|取消|  
|ButtonLabelSave|儲存|  
|GeneralExceptionMessage|某些項目不正確。 可能是暫時性的問題或有錯誤。 請再試一次。|  
|NoItemsToDisplay|沒有可顯示的項目。|  
|PagerButtonLabelFirst|第一頁|  
|PagerButtonLabelLast|最後一頁|  
|PagerButtonLabelNext|下一頁|  
|PagerButtonLabelPrevious|上一頁|  
|PagerLabelPageNOfM|{0} 頁/共 {1\} 頁|  
|PasswordTooShort|密碼太短|  
|EmailAsPassword|請勿使用您的電子郵件做為您的密碼|  
|PasswordSameAsUserName|密碼不可包含您的使用者名稱|  
|PasswordTwoCharacterClasses|使用不同的字元類別|  
|PasswordTooManyRepetitions|太多重複|  
|PasswordSequenceFound|您的密碼包含連續數字|  
|PagerLabelPageSize|頁面大小|  
|CurtainLabelLoading|正在載入...|  
|TablePlaceholderNothingToDisplay|選取的期間和範圍中沒有資料|  
|ButtonLabelClose|關閉|  
  
###  <a name="Documentation"></a> Documentation  
  
|Name|文字|  
|----------|----------|  
|WebDocumentationInvalidHeaderErrorMessage|無效的標頭 '{0}'|  
|WebDocumentationInvalidRequestErrorMessage|無效的要求 URL|  
|TextboxLabelAccessToken|存取權杖 *|  
|DropdownOptionPrimaryKeyFormat|主要-{0}|  
|DropdownOptionSecondaryKeyFormat|次要-{0}|  
|WebDocumentationSubscriptionKeyText|您的訂用帳戶金鑰|  
|WebDocumentationTemplatesAddHeaders|新增必要的 HTTP 標頭|  
|WebDocumentationTemplatesBasicAuthSample|基本授權範例|  
|WebDocumentationTemplatesCurlForBasicAuth|基本授權請使用：--user {username}:{password}|  
|WebDocumentationTemplatesCurlValuesForPath|指定路徑參數的值 (顯示為 {...})、您的訂用帳戶金鑰和查詢參數的值|  
|WebDocumentationTemplatesDeveloperKey|指定您的訂用帳戶金鑰|  
|WebDocumentationTemplatesJavaApache|這個範例會使用 HTTP 元件中的 Apache HTTP 用戶端 (http://hc.apache.org/httpcomponents-client-ga/)|  
|WebDocumentationTemplatesOptionalParams|視需要指定選用參數的值|  
|WebDocumentationTemplatesPhpPackage|這個範例會使用 HTTP_Request2 套件。 (如需詳細資訊︰ http://pear.php.net/package/HTTP_Request2)|  
|WebDocumentationTemplatesPythonValuesForPath|視需要指定路徑參數的值 (顯示為 {...}) 和要求本文|  
|WebDocumentationTemplatesRequestBody|指定要求本文|  
|WebDocumentationTemplatesRequiredParams|指定下列必要參數的值|  
|WebDocumentationTemplatesValuesForPath|指定路徑參數的值 (顯示為 {...})|  
|OAuth2AuthorizationEndpointDescription|授權端點會用於與資源擁有者互動，並獲得授權授與。|  
|OAuth2AuthorizationEndpointName|授權端點|  
|OAuth2TokenEndpointDescription|用戶端使用的權杖端點，需出示其授權授與或重新整理權杖以取得存取權杖。|  
|OAuth2TokenEndpointName|權杖端點|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_Description|<p\>         用戶端會將資源擁有者的使用者代理程式導向授權端點以起始流程。  用戶端會加入其用戶端識別碼、要求的範圍、本機狀態和重新導向 URI，當授權伺服器被授與存取 (或被拒絕存取)，會將使用者代理程式送回此 URI。     </p\>     <p\>         授權伺服器會透過使用者代理程式驗證資源擁有者，然後表明資源擁有者要授與或拒絕用戶端的存取要求。     </p\>     <p\>         假設資源擁有者授與存取，授權伺服器會使用稍早提供的 URI 將使用者代理程式重新導向回用戶端 (在要求中或在用戶端註冊時)。  重新導向 URI 包含授權碼和用戶端稍早提供的任何本機狀態。     </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_ErrorDescription|<p\>     如果使用者拒絕存取要求或是要求無效，會在重新導向中新增以下參數以通知用戶端︰</p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_Name|授權要求|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_RequestDescription|<p\>         用戶端應用程式必須將使用者傳送給授權端點，才能起始 OAuth 程序。          在授權端點，使用者會先驗證，然後授與或拒絕對應用程式的存取。     </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_ResponseDescription|<p\>     假設資源擁有者授與存取，授權伺服器會使用稍早提供的 URI 將使用者代理程式重新導向回用戶端 (在要求中或在用戶端註冊時)。  重新導向 URI 包含授權碼和用戶端稍早提供的任何本機狀態。 </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_Description|<p\>  用戶端會加入在先前步驟收到的授權碼，向授權伺服器的權杖端點要求存取權杖。  當提出要求時，用戶端要向授權伺服器驗證。  用戶端會加入用於取得授權碼的重新導向 URI，以便驗證。 </p\> <p\>     授權伺服器會驗證用戶端、驗證授權碼，並確保收到的重新導向 URI 和步驟 (C) 用於重新導向用戶端所使用的 URI 相符。  如果正確，授權伺服器會以存取權杖回應，或以重新整理權杖回應 (選擇性)。 </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_ErrorDescription|<p\>     如果要求用戶端驗證失敗或不正確，授權伺服器會回應 HTTP 400 (不正確的要求) 狀態碼 (除非另有指定)，並在回應時加入以下參數。 </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_RequestDescription|<p\>   用戶端對權杖端點提出要求，做法是在 HTTP 要求實體本文中使用 "application/x-www-form-urlencoded" 格式的 UTF-8 字元編碼傳送下列參數。 </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_ResponseDescription|<p\>  授權伺服器發出存取權杖和選擇性的重新整理權杖，並在包含 200 (確定) 狀態碼的 HTTP 回應的實體本文中加入下列參數，建構成一個回應。 </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_Description|<p\>  用戶端向授權伺服器驗證，並向權杖端點要求存取權杖。 </p\> <p\>  授權伺服器會驗證用戶端，如果正確，會發出存取權杖。 </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_ErrorDescription|<p\>     如果要求後用戶端驗證失敗或要求不正確，授權伺服器會回應 HTTP 400 (不正確的要求) 狀態碼 (除非另有指定)，並在回應時加入以下參數。 </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_RequestDescription|<p\>   用戶端對權杖端點提出要求，做法是在 HTTP 要求實體本文中使用 "application/x-www-form-urlencoded" 格式的 UTF-8 字元編碼傳送下列參數。 </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_ResponseDescription|<p\>  如果存取權杖要求有效且獲得授權，授權伺服器會發出存取權杖和選擇性的重新整理權杖，並在包含 200 (確定) 狀態碼的 HTTP 回應的實體本文中加入下列參數，建構成一個回應。 </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_Description|<p\>   用戶端會將資源擁有者的使用者代理程式導向授權端點以起始流程。  用戶端會加入用戶端識別碼、要求的範圍、本機狀態和重新導向 URI，當授權伺服器被授與存取 (或被拒絕存取)，會將使用者代理程式送回此 URI。 </p\> <p\>        授權伺服器會透過使用者代理程式驗證資源擁有者，然後表明資源擁有者要授與或拒絕用戶端的存取要求。 </p\> <p\>        假設資源擁有者授與存取，授權伺服器會使用稍早提供的 URI 將使用者代理程式重新導向回用戶端。  重新導向 URI 會將存取權杖包含在 URI 片段中。 </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_ErrorDescription|<p\>     如果資源擁有者拒絕存取要求，或是要求因為遺失或重新導向 URI 無效以外的原因而失敗，授權伺服器會在 "application/x-www-form-urlencoded" 格式的重新導向 URI 的片段元件中加入下列參數，藉此通知用戶端。 </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_RequestDescription|<p\>     用戶端應用程式必須將使用者傳送給授權端點，才能起始 OAuth 程序。      在授權端點，使用者會先驗證，然後授與或拒絕對應用程式的存取。 </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_ResponseDescription|<p\>    如果資源擁有者授與存取要求，授權伺服器會在 "application/x-www-form-urlencoded" 格式的重新導向 URI 的片段元件中加入下列參數，藉此方式發出存取權杖並提供給用戶端。 </p\>|  
|OAuth2Flow_ObtainAuthorization_AuthorizationCodeGrant_Description|授權碼流程最適合能夠維護其認證機密性的用戶端 (例如，使用 PHP、Java、Python、Ruby、ASP.NET 等實作的 Web 伺服器應用程式)。|  
|OAuth2Flow_ObtainAuthorization_AuthorizationCodeGrant_Name|授權碼授與|  
|OAuth2Flow_ObtainAuthorization_ClientCredentialsGrant_Description|用戶端認證流程最適合用戶端 (您的應用程式) 會在受保護資源的控制下要求對資源之存取權的情況。 用戶端會被視為資源擁有者，因此不需要任何使用者互動。|  
|OAuth2Flow_ObtainAuthorization_ClientCredentialsGrant_Name|用戶端認證授與|  
|OAuth2Flow_ObtainAuthorization_ImplicitGrant_Description|隱含流程最適合無法維護其認證機密性 (已知用於操作特定重新導向 URI) 的用戶端。 這些用戶端通常是在使用指令碼語言 (如 JavaScript ) 的瀏覽器中實作。|  
|OAuth2Flow_ObtainAuthorization_ImplicitGrant_Name|隱含授與|  
|OAuth2Flow_ObtainAuthorization_ResourceOwnerPasswordCredentialsGrant_Description|資源擁有者密碼認證流程最適合資源擁有者與用戶端 (您的應用程式，例如裝置作業系統、高特殊權限的應用程式) 之間有信任關係的情況。 此流程很適合能夠取得資源擁有者的認證 (使用者名稱和密碼，通常會使用互動的形式) 的用戶端。|  
|OAuth2Flow_ObtainAuthorization_ResourceOwnerPasswordCredentialsGrant_Name|資源擁有者密碼認證授與|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_Description|<p\>   資源擁有者會為用戶端提供自己的使用者名稱和密碼。 </p\> <p\> 用戶端會加入從資源擁有者那裡收到的認證，向授權伺服器的權杖端點要求存取權杖。  當提出要求時，用戶端要向授權伺服器驗證。 </p\> <p\>     授權伺服器會驗證用戶端及驗證資源擁有者，如果正確，會發出存取權杖。 </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_ErrorDescription|<p\>  如果要求後用戶端驗證失敗或要求不正確，授權伺服器會回應 HTTP 400 (不正確的要求) 狀態碼 (除非另有指定)，並在回應時加入以下參數。 </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_RequestDescription|<p\>    用戶端對權杖端點提出要求，做法是在 HTTP 要求實體本文中使用 "application/x-www-form-urlencoded" 格式的 UTF-8 字元編碼傳送下列參數。 </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_ResponseDescription|<p\>   如果存取權杖要求有效且獲得授權，授權伺服器會發出存取權杖和選擇性的重新整理權杖，並在包含 200 (確定) 狀態碼的 HTTP 回應的實體本文中加入下列參數，建構成一個回應。 </p\>|  
|OAuth2Step_AccessTokenRequest_Name|存取權杖要求|  
|OAuth2Step_AuthorizationRequest_Name|授權要求|  
|OAuth2AccessToken_AuthorizationCodeGrant_TokenResponse|「必要」。 由授權伺服器發出的存取權杖。|  
|OAuth2AccessToken_ClientCredentialsGrant_TokenResponse|「必要」。 由授權伺服器發出的存取權杖。|  
|OAuth2AccessToken_ImplicitGrant_AuthorizationResponse|「必要」。 由授權伺服器發出的存取權杖。|  
|OAuth2AccessToken_ResourceOwnerPasswordCredentialsGrant_TokenResponse|「必要」。 由授權伺服器發出的存取權杖。|  
|OAuth2ClientId_AuthorizationCodeGrant_AuthorizationRequest|「必要」。 用戶端識別碼。|  
|OAuth2ClientId_AuthorizationCodeGrant_TokenRequest|如果用戶端沒有向授權伺服器驗證，則為「必要」。|  
|OAuth2ClientId_ImplicitGrant_AuthorizationRequest|「必要」。 用戶端識別碼。|  
|OAuth2Code_AuthorizationCodeGrant_AuthorizationResponse|「必要」。 由授權伺服器產生的授權碼。|  
|OAuth2Code_AuthorizationCodeGrant_TokenRequest|「必要」。 從授權伺服器那裡收到的授權碼。|  
|OAuth2ErrorDescription_AuthorizationCodeGrant_AuthorizationErrorResponse|「選用」。 人類可閱讀的 ASCII 文字，提供其他資訊。|  
|OAuth2ErrorDescription_AuthorizationCodeGrant_TokenErrorResponse|「選用」。 人類可閱讀的 ASCII 文字，提供其他資訊。|  
|OAuth2ErrorDescription_ClientCredentialsGrant_TokenErrorResponse|「選用」。 人類可閱讀的 ASCII 文字，提供其他資訊。|  
|OAuth2ErrorDescription_ImplicitGrant_AuthorizationErrorResponse|「選用」。 人類可閱讀的 ASCII 文字，提供其他資訊。|  
|OAuth2ErrorDescription_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|「選用」。 人類可閱讀的 ASCII 文字，提供其他資訊。|  
|OAuth2ErrorUri_AuthorizationCodeGrant_AuthorizationErrorResponse|「選用」。 識別人類可閱讀網頁的 URI，網頁中包含錯誤的相關資訊。|  
|OAuth2ErrorUri_AuthorizationCodeGrant_TokenErrorResponse|「選用」。 識別人類可閱讀網頁的 URI，網頁中包含錯誤的相關資訊。|  
|OAuth2ErrorUri_ClientCredentialsGrant_TokenErrorResponse|「選用」。 識別人類可閱讀網頁的 URI，網頁中包含錯誤的相關資訊。|  
|OAuth2ErrorUri_ImplicitGrant_AuthorizationErrorResponse|「選用」。 識別人類可閱讀網頁的 URI，網頁中包含錯誤的相關資訊。|  
|OAuth2ErrorUri_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|「選用」。 識別人類可閱讀網頁的 URI，網頁中包含錯誤的相關資訊。|  
|OAuth2Error_AuthorizationCodeGrant_AuthorizationErrorResponse|「必要」。 下列 ASCII 錯誤碼的其中一個︰invalid_request、unauthorized_client、access_denied、unsupported_response_type、invalid_scope、server_error、temporarily_unavailable。|  
|OAuth2Error_AuthorizationCodeGrant_TokenErrorResponse|「必要」。 下列 ASCII 錯誤碼的其中一個invalid_request、invalid_client、invalid_grant、unauthorized_client、unsupported_grant_type、invalid_scope。|  
|OAuth2Error_ClientCredentialsGrant_TokenErrorResponse|「必要」。 下列 ASCII 錯誤碼的其中一個invalid_request、invalid_client、invalid_grant、unauthorized_client、unsupported_grant_type、invalid_scope。|  
|OAuth2Error_ImplicitGrant_AuthorizationErrorResponse|「必要」。 下列 ASCII 錯誤碼的其中一個︰invalid_request、unauthorized_client、access_denied、unsupported_response_type、invalid_scope、server_error、temporarily_unavailable。|  
|OAuth2Error_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|「必要」。 下列 ASCII 錯誤碼的其中一個invalid_request、invalid_client、invalid_grant、unauthorized_client、unsupported_grant_type、invalid_scope。|  
|OAuth2ExpiresIn_AuthorizationCodeGrant_TokenResponse|「建議使用」。 存取權杖的有效期 (以秒為單位)。|  
|OAuth2ExpiresIn_ClientCredentialsGrant_TokenResponse|「建議使用」。 存取權杖的有效期 (以秒為單位)。|  
|OAuth2ExpiresIn_ImplicitGrant_AuthorizationResponse|「建議使用」。 存取權杖的有效期 (以秒為單位)。|  
|OAuth2ExpiresIn_ResourceOwnerPasswordCredentialsGrant_TokenResponse|「建議使用」。 存取權杖的有效期 (以秒為單位)。|  
|OAuth2GrantType_AuthorizationCodeGrant_TokenRequest|「必要」。 此值必須設為 "authorization_code"。|  
|OAuth2GrantType_ClientCredentialsGrant_TokenRequest|「必要」。 此值必須設為 "client_credentials"。|  
|OAuth2GrantType_ResourceOwnerPasswordCredentialsGrant_TokenRequest|「必要」。 此值必須設為 "password"。|  
|OAuth2Password_ResourceOwnerPasswordCredentialsGrant_TokenRequest|「必要」。 資源擁有者的密碼。|  
|OAuth2RedirectUri_AuthorizationCodeGrant_AuthorizationRequest|「選用」。 重新導向端點 URI 必須是絕對 URI。|  
|OAuth2RedirectUri_AuthorizationCodeGrant_TokenRequest|如果授權要求中包含 "redirect_uri" 參數則為「必要」，且其值「必須」相同。|  
|OAuth2RedirectUri_ImplicitGrant_AuthorizationRequest|「選用」。 重新導向端點 URI 必須是絕對 URI。|  
|OAuth2RefreshToken_AuthorizationCodeGrant_TokenResponse|「選用」。 重新整理權杖，可用來取得新的存取權杖。|  
|OAuth2RefreshToken_ClientCredentialsGrant_TokenResponse|「選用」。 重新整理權杖，可用來取得新的存取權杖。|  
|OAuth2RefreshToken_ResourceOwnerPasswordCredentialsGrant_TokenResponse|「選用」。 重新整理權杖，可用來取得新的存取權杖。|  
|OAuth2ResponseType_AuthorizationCodeGrant_AuthorizationRequest|「必要」。 此值必須設為 "code"。|  
|OAuth2ResponseType_ImplicitGrant_AuthorizationRequest|「必要」。 此值必須設為 "token"。|  
|OAuth2Scope_AuthorizationCodeGrant_AuthorizationRequest|「選用」。 存取要求的範圍。|  
|OAuth2Scope_AuthorizationCodeGrant_TokenResponse|如果與用戶端要求的範圍一樣則為「選用」，否則為「必要」。|  
|OAuth2Scope_ClientCredentialsGrant_TokenRequest|「選用」。 存取要求的範圍。|  
|OAuth2Scope_ClientCredentialsGrant_TokenResponse|如果與用戶端要求的範圍一樣則為「選用」，否則為「必要」。|  
|OAuth2Scope_ImplicitGrant_AuthorizationRequest|「選用」。 存取要求的範圍。|  
|OAuth2Scope_ImplicitGrant_AuthorizationResponse|如果與用戶端要求的範圍一樣則為「選用」，否則為「必要」。|  
|OAuth2Scope_ResourceOwnerPasswordCredentialsGrant_TokenRequest|「選用」。 存取要求的範圍。|  
|OAuth2Scope_ResourceOwnerPasswordCredentialsGrant_TokenResponse|如果與用戶端要求的範圍一樣則為「選用」，否則為「必要」。|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationErrorResponse|如果用戶端授權要求中有 "state" 參數，則為「必要」。  收到來自用戶端的確切值。|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationRequest|「建議使用」。 不透明值，用戶端用來維持在要求與回呼之間的狀態。  授權伺服器將使用者代理程式重新導向傳回給用戶端時，會加入此值。  「應該」使用此參數來防止跨網站偽造要求。|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationResponse|如果用戶端授權要求中有 "state" 參數，則為「必要」。  收到來自用戶端的確切值。|  
|OAuth2State_ImplicitGrant_AuthorizationErrorResponse|如果用戶端授權要求中有 "state" 參數，則為「必要」。  收到來自用戶端的確切值。|  
|OAuth2State_ImplicitGrant_AuthorizationRequest|「建議使用」。 不透明值，用戶端用來維持在要求與回呼之間的狀態。  授權伺服器將使用者代理程式重新導向傳回給用戶端時，會加入此值。  「應該」使用此參數來防止跨網站偽造要求。|  
|OAuth2State_ImplicitGrant_AuthorizationResponse|如果用戶端授權要求中有 "state" 參數，則為「必要」。  收到來自用戶端的確切值。|  
|OAuth2TokenType_AuthorizationCodeGrant_TokenResponse|「必要」。 發出的權杖類型。|  
|OAuth2TokenType_ClientCredentialsGrant_TokenResponse|「必要」。 發出的權杖類型。|  
|OAuth2TokenType_ImplicitGrant_AuthorizationResponse|「必要」。 發出的權杖類型。|  
|OAuth2TokenType_ResourceOwnerPasswordCredentialsGrant_TokenResponse|「必要」。 發出的權杖類型。|  
|OAuth2UserName_ResourceOwnerPasswordCredentialsGrant_TokenRequest|「必要」。 資源擁有者的使用者名稱。|  
|OAuth2UnsupportedTokenType|不支援 '{0}' 權杖類型。|  
|OAuth2InvalidState|來自授權伺服器的回應無效|  
|OAuth2GrantType_AuthorizationCode|授權碼|  
|OAuth2GrantType_Implicit|隱含|  
|OAuth2GrantType_ClientCredentials|用戶端認證|  
|OAuth2GrantType_ResourceOwnerPassword|資源擁有者的密碼|  
|WebDocumentation302Code|302 已找到|  
|WebDocumentation400Code|400 (不正確的要求)|  
|OAuth2SendingMethod_AuthHeader|驗證標頭|  
|OAuth2SendingMethod_QueryParam|查詢參數|  
|OAuth2AuthorizationServerGeneralException|透過 {0} 授權存取時發生錯誤|  
|OAuth2AuthorizationServerCommunicationException|無法建立連接至授權伺服器的 HTTP 連線，或連線已意外關閉。|  
|WebDocumentationOAuth2GeneralErrorMessage|發生未預期的錯誤。|  
|AuthorizationServerCommunicationException|發生授權伺服器通訊的例外狀況。 請連絡系統管理員。|  
|TextblockSubscriptionKeyHeaderDescription|訂用帳戶金鑰，可讓您存取此 API。 可在您的<a href='/developer'\>設定檔</a\>中找到。|  
|TextblockOAuthHeaderDescription|自 <i\>{0}</i\> 取得的 OAuth 2.0 存取權杖。 支援的授與類型︰<i\>{1}</i\>。|  
|TextblockContentTypeHeaderDescription|傳送至 API 的本文媒體類型。|  
|ErrorMessageApiNotAccessible|您嘗試呼叫的 API 目前無法使用。 請從<a href="/issues"\>這裡</a\>連絡 API 發行者。|  
|ErrorMessageApiTimedout|您嘗試呼叫的 API 所花的時間比正常收到回應的時間更久。 請從<a href="/issues"\>這裡</a\>連絡 API 發行者。|  
|BadRequestParameterExpected|「預期收到 '{0}' 參數」|  
|TooltipTextDoubleClickToSelectAll|按兩下以全部選取。|  
|TooltipTextHideRevealSecret|顯示/隱藏|  
|ButtonLinkOpenConsole|試試看|  
|SectionHeadingRequestBody|Request body|  
|SectionHeadingRequestParameters|要求參數|  
|SectionHeadingRequestUrl|要求 URL|  
|SectionHeadingResponse|Response|  
|SectionHeadingRequestHeaders|要求標頭|  
|FormLabelSubtextOptional|選用|  
|SectionHeadingCodeSamples|程式碼範例|  
|TextblockOpenidConnectHeaderDescription|自 <i\>{0}</i\> 取得的 OpenID 連接識別碼。 支援的授與類型︰<i\>{1}</i\>。|  
  
###  <a name="ErrorPageStrings"></a> ErrorPageStrings  
  
|Name|文字|  
|----------|----------|  
|LinkLabelBack|上一步|  
|LinkLabelHomePage|首頁|  
|LinkLabelSendUsEmail|傳送電子郵件|  
|PageTitleError|很抱歉，提供要求的頁面時發生問題|  
|TextblockPotentialCauseIntermittentIssue|這是間歇性的資料存取問題，可能已經不存在。|  
|TextblockPotentialCauseOldLink|您點選的連結可能太舊，無法再指向正確的位置。|  
|TextblockPotentialCauseTechnicalProblem|可能是我們這一端發生技術問題。|  
|TextblockPotentialSolutionRefresh|請嘗試重新整理頁面。|  
|TextblockPotentialSolutionStartOver|自我們的 {0} 從頭開始。|  
|TextblockPotentialSolutionTryAgain|移至 {0}，然後再試一次您執行的動作。|  
|TextReportProblem|{0} 描述問題出在哪裡，我們會盡快查看。|  
|TitlePotentialCause|可能的原因|  
|TitlePotentialSolution|可能只是暫時性的問題，可嘗試一些動作|  
  
###  <a name="IssuesStrings"></a> IssuesStrings  
  
|Name|文字|  
|----------|----------|  
|WebIssuesIndexTitle|問題|  
|WebIssuesNoActiveSubscriptions|您沒有作用中的訂用帳戶。 您必須訂閱產品才會回報問題。|  
|WebIssuesNotSignin|您尚未登入。 請至 {0} 回報問題或張貼註解。|  
|WebIssuesReportIssueButton|報告問題|  
|WebIssuesSignIn|登入|  
|WebIssuesStatusReportedBy|狀態︰{0} &#124;由 {1} 報告|  
  
###  <a name="NotFoundStrings"></a> NotFoundStrings  
  
|Name|文字|  
|----------|----------|  
|LinkLabelHomePage|首頁|  
|LinkLabelSendUsEmail|傳送電子郵件|  
|PageTitleNotFound|抱歉，我們找不到您搜尋的頁面|  
|TextblockPotentialCauseMisspelledUrl|如果您是手動輸入 URL，您可能拼錯 URL。|  
|TextblockPotentialCauseOldLink|您點選的連結可能太舊，無法再指向正確的位置。|  
|TextblockPotentialSolutionRetype|請嘗試重新輸入 URL。|  
|TextblockPotentialSolutionStartOver|自我們的 {0} 從頭開始。|  
|TextReportProblem|{0} 描述問題出在哪裡，我們會盡快查看。|  
|TitlePotentialCause|可能的原因|  
|TitlePotentialSolution|可能的解決辦法|  
  
###  <a name="ProductDetailsStrings"></a> ProductDetailsStrings  
  
|Name|文字|  
|----------|----------|  
|WebProductsAgreement|訂閱 {0} 產品即表示我同意 `<a data-toggle='modal' href='#legal-terms'\>Terms of Use</a\>`。|  
|WebProductsLegalTermsLink|使用條款|  
|WebProductsSubscribeButton|訂閱|  
|WebProductsUsageLimitsHeader|使用限制|  
|WebProductsYouAreNotSubscribed|您已訂閱此產品。|  
|WebProductsYouRequestedSubscription|您要求訂閱此產品。|  
|ErrorYouNeedtoAgreeWithLegalTerms|您必須同意使用條款才可以繼續。|  
|ButtonLabelAddSubscription|加入訂閱|  
|LinkLabelChangeSubscriptionName|變更|  
|ButtonLabelConfirm|確認|  
|TextblockMultipleSubscriptionsCount|您有 {0} 個此產品的訂用帳戶:|  
|TextblockSingleSubscriptionsCount|您有 {0} 個此產品的訂用帳戶：|  
|TextblockSingleApisCount|此產品有 {0} 個 API:|  
|TextblockMultipleApisCount|此產品有 {0} 個 API:|  
|TextblockHeaderSubscribe|訂閱產品|  
|TextblockSubscriptionDescription|已建立新的訂用帳戶如下:|  
|TextblockSubscriptionLimitReached|已達到訂用帳戶限制。|  
  
###  <a name="ProductsStrings"></a> ProductsStrings  
  
|Name|文字|  
|----------|----------|  
|PageTitleProducts|產品|  
  
###  <a name="ProviderInfoStrings"></a> ProviderInfoStrings  
  
|Name|文字|  
|----------|----------|  
|TextboxExternalIdentitiesDisabled|目前系統管理員已停用登入。|  
|TextboxExternalIdentitiesSigninInvitation|或者，以此登入|  
|TextboxExternalIdentitiesSigninInvitationPrimary|以此登入:|  
  
###  <a name="SigninResources"></a> SigninResources  
  
|Name|文字|  
|----------|----------|  
|PrincipalNotFound|找不到主體或簽章無效|  
|ErrorSsoAuthenticationFailed|SSO 驗證失敗|  
|ErrorSsoAuthenticationFailedDetailed|提供的權杖無效或無法驗證簽章。|  
|ErrorSsoTokenInvalid|SSO 權杖無效|  
|ValidationErrorSpecificEmailAlreadyExists|電子郵件 ' {0} ' 已註冊|  
|ValidationErrorSpecificEmailInvalid|電子郵件 '{0}' 無效|  
|ValidationErrorPasswordInvalid|密碼無效。 請更正錯誤，然後再試一次。|  
|PropertyTooShort|{0} 太短|  
|WebAuthenticationAddresserEmailInvalidErrorMessage|電子郵件地址無效。|  
|ValidationMessageNewPasswordConfirmationRequired|確認新密碼|  
|ValidationErrorPasswordConfirmationRequired|確認密碼為空白|  
|WebAuthenticationEmailChangeNotice|已將變更確認電子郵件傳送到 {0}。 請遵循其中的指示確認您的新電子郵件地址。 如果您的收件匣在接下來的幾分鐘內沒有收到電子郵件，請檢查垃圾郵件資料夾。|  
|WebAuthenticationEmailChangeNoticeHeader|已成功處理您的電子郵件變更要求|  
|WebAuthenticationEmailChangeNoticeTitle|已要求電子郵件變更|  
|WebAuthenticationEmailHasBeenRevertedNotice|您的電子郵件已經存在。 已還原要求|  
|ValidationErrorEmailAlreadyExists|電子郵件已經存在|  
|ValidationErrorEmailInvalid|電子郵件地址無效|  
|TextboxLabelEmail|電子郵件|  
|ValidationErrorEmailRequired|電子郵件為必填欄位。|  
|WebAuthenticationErrorNoticeHeader|Error|  
|WebAuthenticationFieldLengthErrorMessage|{0} 的長度上限是 {1}|  
|TextboxLabelEmailFirstName|名字|  
|ValidationErrorFirstNameRequired|名字為必填欄位。|  
|ValidationErrorFirstNameInvalid|名字無效|  
|NoticeInvalidInvitationToken|請注意，確認連結只在 48 小時內有效。 如果您仍在此時間範圍內時，請確定您的連結正確。 如果您的連結已過期，請重複您嘗試確認的動作。|  
|NoticeHeaderInvalidInvitationToken|無效的邀請權杖|  
|NoticeTitleInvalidInvitationToken|確認錯誤|  
|WebAuthenticationLastNameInvalidErrorMessage|姓氏無效|  
|TextboxLabelEmailLastName|姓氏|  
|ValidationErrorLastNameRequired|姓氏為必填欄位。|  
|WebAuthenticationLinkExpiredNotice|傳送給您的確認連結已過期。 `<a href={0}?token={1}>Resend confirmation email.</a\>`|  
|NoticePasswordResetLinkInvalidOrExpired|您的密碼重設連結無效或已過期。|  
|WebAuthenticationLinkExpiredNoticeTitle|已傳送連結|  
|WebAuthenticationNewPasswordLabel|新密碼|  
|ValidationMessageNewPasswordRequired|新密碼為必填欄位。|  
|TextboxLabelNotificationsSenderEmail|通知的寄件者電子郵件|  
|TextboxLabelOrganizationName|組織名稱|  
|WebAuthenticationOrganizationRequiredErrorMessage|組織名稱為空白|  
|WebAuthenticationPasswordChangedNotice|已成功更新您的密碼|  
|WebAuthenticationPasswordChangedNoticeTitle|已更新密碼|  
|WebAuthenticationPasswordCompareErrorMessage|密碼不符|  
|WebAuthenticationPasswordConfirmLabel|確認密碼|  
|ValidationErrorPasswordInvalidDetailed|密碼太弱。|  
|WebAuthenticationPasswordLabel|密碼|  
|ValidationErrorPasswordRequired|密碼為必填欄位。|  
|WebAuthenticationPasswordResetSendNotice|已將變更密碼確認電子郵件傳送到 {0}。 請遵循電子郵件中的指示以繼續您的密碼變更程序。|  
|WebAuthenticationPasswordResetSendNoticeHeader|已成功處理您的密碼重設要求|  
|WebAuthenticationPasswordResetSendNoticeTitle|已要求密碼重設|  
|WebAuthenticationRequestNotFoundNotice|找不到要求|  
|WebAuthenticationSenderEmailRequiredErrorMessage|通知的寄件者電子郵件為空白|  
|WebAuthenticationSigninPasswordLabel|請輸入密碼確認變更|  
|WebAuthenticationSignupConfirmNotice|已將註冊確認電子郵件傳送到 {0}。<br /\>請遵循電子郵件中的指示啟用您的帳戶。<br /\>如果您的收件匣在接下來的幾分鐘內沒有收到電子郵件，請檢查垃圾郵件資料夾。|  
|WebAuthenticationSignupConfirmNoticeHeader|已成功建立您的帳戶|  
|WebAuthenticationSignupConfirmNoticeRepeatHeader|已再次傳送註冊確認電子郵件|  
|WebAuthenticationSignupConfirmNoticeTitle|已建立帳戶|  
|WebAuthenticationTokenRequiredErrorMessage|權杖為空白|  
|WebAuthenticationUserAlreadyRegisteredNotice|擁有這封電子郵件的使用者已經在系統中註冊。 如果您忘記密碼，請嘗試將它還原，或連絡我們的支援小組。|  
|WebAuthenticationUserAlreadyRegisteredNoticeHeader|使用者已經註冊|  
|WebAuthenticationUserAlreadyRegisteredNoticeTitle|已經註冊|  
|ButtonLabelChangePassword|變更密碼|  
|ButtonLabelChangeAccountInfo|變更帳戶資訊|  
|ButtonLabelCloseAccount|關閉帳戶|  
|WebAuthenticationInvalidCaptchaErrorMessage|輸入的文字與圖片上的文字不符。 請再試一次。|  
|ValidationErrorCredentialsInvalid|電子郵件或密碼無效。 請更正錯誤，然後再試一次。|  
|WebAuthenticationRequestIsNotValid|要求無效|  
|WebAuthenticationUserIsNotConfirm|嘗試登入之前，請確認您的註冊。|  
|WebAuthenticationInvalidEmailFormated|電子郵件無效: {0}|  
|WebAuthenticationUserNotFound|找不到使用者|  
|WebAuthenticationTenantNotRegistered|您的帳戶屬於未獲得此入口網站存取授權的 Azure Active Directory 租用戶。|  
|WebAuthenticationAuthenticationFailed|驗證失敗。|  
|WebAuthenticationGooglePlusNotEnabled|驗證失敗。 如果您已授權應用程式，請連絡系統管理員以確定 Google 驗證的設定正確。|  
|ValidationErrorAllowedTenantIsRequired|允許的租用戶為必填欄位|  
|ValidationErrorTenantIsNotValid|Azure Active Directory 租用戶 '{0}' 無效。|  
|WebAuthenticationActiveDirectoryTitle|Azure Active Directory|  
|WebAuthenticationLoginUsingYourProvider|使用您的 {0} 帳戶登入|  
|WebAuthenticationUserLimitNotice|您已達到允許的使用者數目上限。 請 `<a href="mailto:{0}"\>contact the administrator</a\>` 升級其服務，然後重新啟用使用者註冊。|  
|WebAuthenticationUserLimitNoticeHeader|已停用使用者註冊|  
|WebAuthenticationUserLimitNoticeTitle|已停用使用者註冊|  
|WebAuthenticationUserRegistrationDisabledNotice|系統管理員已停用使用者註冊。 請使用外部識別提供者登入。|  
|WebAuthenticationUserRegistrationDisabledNoticeHeader|已停用使用者註冊|  
|WebAuthenticationUserRegistrationDisabledNoticeTitle|已停用使用者註冊|  
|WebAuthenticationSignupPendingConfirmationNotice|在我們完成您的帳戶建立之前，需要先驗證您的電子郵件地址。 我們已傳送電子郵件到 {0}。 請遵循電子郵件內的指示來啟動您的帳戶。 如果在接下來的幾分鐘內沒有收到電子郵件，請檢查垃圾郵件資料夾。|  
|WebAuthenticationSignupPendingConfirmationAccountFoundNotice|我們發現電子郵件地址 {0} 尚未確認帳戶。 為了完成您的帳戶建立，我們需要先驗證您的電子郵件地址。 我們已傳送電子郵件到 {0}。 請遵循電子郵件內的指示來啟動您的帳戶。 如果在接下來的幾分鐘內沒有收到電子郵件，請檢查垃圾郵件資料夾|  
|WebAuthenticationSignupConfirmationAlmostDone|快要完成了|  
|WebAuthenticationSignupConfirmationEmailSent|我們已傳送電子郵件到 {0}。 請遵循電子郵件內的指示來啟動您的帳戶。 如果在接下來的幾分鐘內沒有收到電子郵件，請檢查垃圾郵件資料夾。|  
|WebAuthenticationEmailSentNotificationMessage|已成功將電子郵件傳送到 {0}|  
|WebAuthenticationNoAadTenantConfigured|服務未設定 Azure Active Directory 租用戶。|  
|CheckboxLabelUserRegistrationTermsConsentRequired|我同意 `<a data-toggle="modal" href="#" data-target="#terms"\>Terms of Use</a\>`。|  
|TextblockUserRegistrationTermsProvided|請檢閱 `<a data-toggle="modal" href="#" data-target="#terms"\>Terms of Use.</a\>`|  
|DialogHeadingTermsOfUse|使用條款|  
|ValidationMessageConsentNotAccepted|您必須同意使用條款才可以繼續。|  
  
###  <a name="SigninStrings"></a> SigninStrings  
  
|Name|文字|  
|----------|----------|  
|WebAuthenticationForgotPassword|忘記密碼了嗎?|  
|WebAuthenticationIfAdministrator|如果您是系統管理員，您必須登入 `<a href="{0}"\>here</a\>`。|  
|WebAuthenticationNotAMember|還不是會員嗎? `<a href="/signup"\>Sign up now</a\>`|  
|WebAuthenticationRemember|在這台電腦上記住我|  
|WebAuthenticationSigininWithPassword|以您的使用者名稱和密碼登入|  
|WebAuthenticationSigninTitle|登入|  
|WebAuthenticationSignUpNow|立即註冊|  
  
###  <a name="SignupStrings"></a> SignupStrings  
  
|Name|文字|  
|----------|----------|  
|PageTitleSignup|註冊|  
|WebAuthenticationAlreadyAMember|已經是會員嗎?|  
|WebAuthenticationCreateNewAccount|建立新的 API 管理帳戶|  
|WebAuthenticationSigninNow|立即登入|  
|ButtonLabelSignup|註冊|  
  
###  <a name="SubscriptionListStrings"></a> SubscriptionListStrings  
  
|Name|文字|  
|----------|----------|  
|SubscriptionCancelConfirmation|確定要取消輸入這個訂用帳戶嗎?|  
|SubscriptionRenewConfirmation|確定要續訂這個訂用帳戶嗎?|  
|WebDevelopersManageSubscriptions|管理訂用帳戶|  
|WebDevelopersPrimaryKey|主索引鍵|  
|WebDevelopersRegenerateLink|重新產生|  
|WebDevelopersSecondaryKey|次要索引鍵|  
|ButtonLabelShowKey|顯示|  
|ButtonLabelRenewSubscription|續訂|  
|WebDevelopersSubscriptionReqested|已於 {0} 要求|  
|WebDevelopersSubscriptionRequestedState|已要求|  
|WebDevelopersSubscriptionTableNameHeader|Name|  
|WebDevelopersSubscriptionTableStateHeader|狀態|  
|WebDevelopersUsageStatisticsLink|分析報告|  
|WebDevelopersYourSubscriptions|您的訂用帳戶|  
|SubscriptionPropertyLabelRequestedDate|已於此日期要求|  
|SubscriptionPropertyLabelStartedDate|已於此日期開始|  
|PageTitleRenameSubscription|重新命名訂用帳戶|  
|SubscriptionPropertyLabelName|訂用帳戶名稱|  
  
###  <a name="SubscriptionStrings"></a> SubscriptionStrings  
  
|Name|文字|  
|----------|----------|  
|SectionHeadingCloseAccount|想要關閉您的帳戶嗎?|  
|PageTitleDeveloperProfile|設定檔|  
|ButtonLabelHideKey|隱藏|  
|ButtonLabelRegenerateKey|重新產生|  
|InformationMessageKeyWasRegenerated|確定要重新產生此索引鍵嗎?|  
|ButtonLabelShowKey|顯示|  
  
###  <a name="UpdateProfileStrings"></a> UpdateProfileStrings  
  
|Name|文字|  
|----------|----------|  
|ButtonLabelUpdateProfile|更新設定檔|  
|PageTitleUpdateProfile|變更帳戶資訊|  
  
###  <a name="UserProfile"></a> UserProfile  
  
|Name|文字|  
|----------|----------|  
|ButtonLabelChangeAccountInfo|變更帳戶資訊|  
|ButtonLabelChangePassword|變更密碼|  
|ButtonLabelCloseAccount|關閉帳戶|  
|TextboxLabelEmail|電子郵件|  
|TextboxLabelEmailFirstName|名字|  
|TextboxLabelEmailLastName|姓氏|  
|TextboxLabelNotificationsSenderEmail|通知的寄件者電子郵件|  
|TextboxLabelOrganizationName|組織名稱|  
|SubscriptionStateActive|作用中|  
|SubscriptionStateCancelled|已取消|  
|SubscriptionStateExpired|已過期|  
|SubscriptionStateRejected|已拒絕|  
|SubscriptionStateRequested|已要求|  
|SubscriptionStateSuspended|暫止|  
|DefaultSubscriptionNameTemplate|{0}  (預設值)|  
|SubscriptionNameTemplate|開發人員存取 #{0}|  
|TextboxLabelSubscriptionName|訂用帳戶名稱|  
|ValidationMessageSubscriptionNameRequired|訂用帳戶名稱不可為空白。|  
|ApiManagementUserLimitReached|您已達到允許的使用者數目上限。 請升級至更高的定價層。|  
  
##  <a name="glyphs"></a> 圖像資源  
 API 管理的開發人員入口網站範本可以使用 [Bootstrap 中的 Glyphicons](http://getbootstrap.com/components/#glyphicons) 中的圖像。 這一套圖像包含超過 250 個 [Glyphicon](http://glyphicons.com/) Halfling 集字型格式的圖像。 若要使用此圖像集中的圖像，使用下列語法。  
  
```html  
<span class="glyphicon glyphicon-user">  
```  
  
 如需圖像的完整清單，請參閱 [Bootstrap 中的 Glyphicons](http://getbootstrap.com/components/#glyphicons)。

## <a name="next-steps"></a>後續步驟
如需有關使用範本的詳細資訊，請參閱[如何使用範本自訂 API 管理開發人員入口網站](api-management-developer-portal-templates.md)。
