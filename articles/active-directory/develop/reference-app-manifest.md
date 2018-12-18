---
title: 了解 Azure Active Directory 應用程式資訊清單 | Microsoft Docs
description: 詳細說明 Azure Active Directory 應用程式資訊清單；此資訊清單代表應用程式在 Azure AD 租用戶中的身分識別組態，可用來協助進行 OAuth 授權、同意體驗等等。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 4804f3d4-0ff1-4280-b663-f8f10d54d184
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.custom: aaddev
ms.reviewer: sureshja
ms.openlocfilehash: 74bcd1e795186eba652d2da986c290e1989dd1d1
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041569"
---
# <a name="azure-active-directory-app-manifest"></a>Azure Active Directory 應用程式資訊清單

與 Azure Active Directory (Azure AD) 整合的應用程式必須使用 Azure AD 租用戶來註冊。 選取 [Azure Active Directory] 底下的 [應用程式註冊]，選擇您要設定的應用程式，然後選取 [資訊清單]，即可在 [Azure 入口網站](https://portal.azure.com)中設定應用程式。

## <a name="manifest-reference"></a>資訊清單參考

> [!NOTE]
> 如果您看不到 [描述] 之後的 [範例值] 資料行，請將瀏覽器視窗放到最大並捲動/撥動，直到您看到 [範例值] 資料行為止。

>[!div class="mx-tdBreakAll"]
>[!div class="mx-tdCol2BreakAll"]

| Key  | 值類型 | 說明  | 範例值 |
|---------|---------|---------|---------|
| `accessTokenAcceptedVersion` | 可為 null 的 Int32 | 指定目前 API 資源可接受的存取權杖版本。 可能的值為 1、2、null。 預設值是 null，系統會將此值視為 2。 | `2` |
| `allowPublicClient` | 布林值 | 指定後援應用程式類型。 根據預設，Azure AD 會從 replyUrlsWithType 推斷應用程式類型。 在某些情況中，Azure AD 無法判斷用戶端應用程式類型 (例如 [ROPC](https://tools.ietf.org/html/rfc6749#section-4.3) 流程，其會發生沒有 URL 重新導向的 HTTP 要求)。 在這些情況下，Azure AD 會根據這個屬性的值來解譯應用程式類型。 如果此值設為 true，後援應用程式類型就會設定為公用用戶端，例如在行動裝置上執行的已安裝應用程式。 預設值為 false，這表示後援應用程式類型是機密用戶端，例如 Web 應用程式。 | `false` |
| `appId` | 識別碼字串 | 針對由 Azure AD 指派給應用程式的應用程式，指定唯一識別碼。 | `"601790de-b632-4f57-9523-ee7cb6ceba95"` |
| `appRoles` | 陣列類型 | 指定應用程式可以宣告的角色集合。 這些角色可以指派給使用者、群組或服務主體。 如需更多範例和資訊，請參閱[在您的應用程式中新增應用程式角色，並且在權杖中接收這些角色](howto-add-app-roles-in-azure-ad-apps.md) | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;"allowedMemberTypes": [<br>&emsp;&nbsp;&nbsp;&nbsp;"User"<br>&nbsp;&nbsp;&nbsp;],<br>&nbsp;&nbsp;&nbsp;"description":"Read-only access to device information",<br>&nbsp;&nbsp;&nbsp;"displayName":"Read Only",<br>&nbsp;&nbsp;&nbsp;"id":guid,<br>&nbsp;&nbsp;&nbsp;"isEnabled":true,<br>&nbsp;&nbsp;&nbsp;"value":"ReadOnly"<br>&nbsp;&nbsp;}<br>]</code>  |
| `groupMembershipClaims` | 字串 | 此為位元遮罩，可設定應用程式所需使用者或 OAuth 2.0 存取權杖中所發出的 `groups` 宣告。 位元遮罩值如下：<br>0：無<br>1：安全性群組和 Azure AD 角色<br>2：已保留<br>4：已保留<br>將位元遮罩設為 7，則可以取得登入使用者所屬的所有安全性群組、通訊群組及 Azure AD 目錄角色。 | `1` |
| `optionalClaims` | 字串 | 由此特定應用程式的安全性權杖服務在權杖中傳回的選擇性宣告。 如需詳細資訊，請參閱[選用宣告](active-directory-optional-claims.md)。 | `null` |
| `id` | 識別碼字串 | 目錄中應用程式的唯一識別碼。 此識別碼不是用來在任何通訊協定交易中識別應用程式的識別碼。 它是用來參考目錄查詢中的物件。 | `"f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd"` |
| `identifierUris` | 字串陣列 | 當應用程式為多租用戶時，專門在 Web 應用程式的 Azure AD 租用戶或經過驗證的自訂網域中，用於識別 Web 應用程式的使用者定義 URI。 | <code>[<br>&nbsp;&nbsp;"https://MyRegistererdApp"<br>]</code> |
| `informationalUrls` | 字串 | 指定應用程式服務條款和隱私權聲明的連結。 使用者會透過使用者同意體驗看到服務條款和隱私權聲明。 如需詳細資訊，請參閱[如何：新增 Azure AD 註冊應用程式的服務條款和隱私權聲明](howto-add-terms-of-service-privacy-statement.md)。 | <code>{<br>&nbsp;&nbsp;&nbsp;"marketing":"https://MyRegisteredApp/marketing",<br>&nbsp;&nbsp;&nbsp;"privacy":"https://MyRegisteredApp/privacystatement",<br>&nbsp;&nbsp;&nbsp;"support":"https://MyRegisteredApp/support",<br>&nbsp;&nbsp;&nbsp;"termsOfService":"https://MyRegisteredApp/termsofservice"<br>}</code> |
| `keyCredentials` | 陣列類型 | 保留對應用程式指定認證、字串型的共用密碼以及 X.509 憑證的參考。 要求存取權杖時會使用這些認證 (當應用程式作為用戶端，而不是資源時)。 | <code>[<br>&nbsp;{<br>&nbsp;&nbsp;&nbsp;"customKeyIdentifier":null,<br>&nbsp;&nbsp;&nbsp;"endDate":"2018-09-13T00:00:00Z",<br>&nbsp;&nbsp;&nbsp;"keyId":"\<guid>",<br>&nbsp;&nbsp;&nbsp;"startDate":"2017-09-12T00:00:00Z",<br>&nbsp;&nbsp;&nbsp;"type":"AsymmetricX509Cert",<br>&nbsp;&nbsp;&nbsp;"usage":"Verify",<br>&nbsp;&nbsp;&nbsp;"value":null<br>&nbsp;&nbsp;}<br>]</code> |
| `knownClientApplications` | 陣列類型 | 如果您有包含兩個組件 (用戶端應用程式及自訂 Web API 應用程式) 的解決方案，則用來統合同意。 如果您將此值輸入為用戶端應用程式的 appID ，則使用者只需同意用戶端應用程式一次。 Azure AD 會知道同意用戶端表示隱含地同意 Web API，並會自動同時佈建用戶端與 Web API 的服務主體。 用戶端與 Web API 應用程式都必須在相同的租用戶中註冊。 | `[GUID]` |
| `logoUrl` | 字串 | 此為唯讀值，會指向入口網站中已上傳標誌的 CDN URL。 | `https://MyRegisteredAppLogo` |
| `logoutUrl` | 字串 | 用於登出應用程式的 URL。 | `https://MyRegisteredAppLogout` |
| `name` | 字串 | 應用程式的顯示名稱。 | `MyRegisteredApp` |
| `oauth2AllowImplicitFlow` | 布林值 | 指定此 Web 應用程式可否要求 OAuth2.0 隱含流程存取權杖。 預設值為 false。 此旗標用於瀏覽器型應用程式，例如 JavaScript 單頁應用程式。 若要進一步了解，請在目錄中輸入 `OAuth 2.0 implicit grant flow` 並查看隱含流程相關主題。 | `false` |
| `oauth2AllowIdTokenImplicitFlow` | 布林值 | 指定此 Web 應用程式可否要求 OAuth2.0 隱含流程識別碼權杖。 預設值為 false。 此旗標用於瀏覽器型應用程式，例如 JavaScript 單頁應用程式。 | `false` |
| `oauth2Permissions` | 陣列類型 | 指定 Web API (資源) 應用程式公開給用戶端應用程式的 OAuth 2.0 權限範圍集合。 這些權限範圍可能會在同意過程中授與用戶端應用程式。 | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;"adminConsentDescription":"Allow the app to access resources on behalf of the signed-in user.",<br>&nbsp;&nbsp;&nbsp;"adminConsentDisplayName":"Access resource1",<br>&nbsp;&nbsp;&nbsp;"id":"\<guid>",<br>&nbsp;&nbsp;&nbsp;"isEnabled":true,<br>&nbsp;&nbsp;&nbsp;"type":"User",<br>&nbsp;&nbsp;&nbsp;"userConsentDescription":"Allow the app to access resource1 on your behalf.",<br>&nbsp;&nbsp;&nbsp;"userConsentDisplayName":"Access resources",<br>&nbsp;&nbsp;&nbsp;"value":"u| 指定 Web API (資源) 應用程式公開給用戶端應用程式的 OAuth 2.0 權限範圍集合。 這些權限範圍可能會在同意過程中授與用戶端應用程式。 ser_impersonation"<br>&nbsp;&nbsp;}<br>]</code> |
| `oauth2RequiredPostResponse` | 布林值 | 指定 Azure AD 在 OAuth 2.0 權杖要求期間是否允許 POST 要求 (相對於 GET 要求)。 預設值為 false，亦即指定只允許 GET 要求。 | `false` |
| `parentalControlSettings` | 字串 | `countriesBlockedForMinors` 指定會對未成年人封鎖此應用程式的國家/地區。<br>`legalAgeGroupRule` 指定會套用到應用程式使用者的法定年齡群組規則。 可以設定為 `Allow`、`RequireConsentForPrivacyServices`、`RequireConsentForMinors`、`RequireConsentForKids` 或 `BlockMinors`。  | <code>{<br>&nbsp;&nbsp;&nbsp;"countriesBlockedForMinors":[],<br>&nbsp;&nbsp;&nbsp;"legalAgeGroupRule":"Allow"<br>} </code> |
| `passwordCredentials` | 陣列類型 | 請參閱 `keyCredentials` 屬性的描述。 | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;"customKeyIdentifier":null,<br>&nbsp;&nbsp;&nbsp;"endDate":"2018-10-19T17:59:59.6521653Z",<br>&nbsp;&nbsp;&nbsp;"keyId":"\<guid>",<br>&nbsp;&nbsp;&nbsp;"startDate":"2016-10-19T17:59:59.6521653Z",<br>&nbsp;&nbsp;&nbsp;"value":null<br>&nbsp;&nbsp;&nbsp;}<br>] </code> |
| `preAuthorizedApplications` | 陣列類型 | 列出需要隱含同意的應用程式和要求的權限。 需要管理員已向應用程式提供同意。 preAuthorizedApplications 不需要使用者同意要求的權限。 PreAuthorizedApplications 中所列的權限不需要使用者同意。 不過，preAuthorizedApplications 中未列出的任何其他要求權限則需要使用者同意。 | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;"appId": "abcdefg2-000a-1111-a0e5-812ed8dd72e8",<br>&nbsp;&nbsp;&nbsp;&nbsp;"permissionIds": [<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"8748f7db-21fe-4c83-8ab5-53033933c8f1"<br>&nbsp;&nbsp;&nbsp;&nbsp;]<br>&nbsp;&nbsp;}<br>]</code> |
| `replyUrlsWithType` | 字串陣列 | 傳回權杖時，此多值屬性會保留 Azure AD 接受作為目的地的已註冊 redirect_uri 值清單。 每個 URI 值皆應包含相關聯的應用程式類型值。 支援的類型值為：`Web`、`InstalledClient`。 | <code>"replyUrlsWithType":&nbsp;[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"url":&nbsp;"https://localhost:4400/services/office365/redirectTarget.html",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"type":&nbsp;"InstalledClient"&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;}<br>]</code> |
| `requiredResourceAccess` | 陣列類型 | 在動態同意的情況下，`requiredResourceAccess` 可對使用靜態同意的使用者，推動系統管理員同意體驗和使用者同意體驗。 不過，這不會推動一般情況的使用者同意體驗。<br>`resourceAppId` 是應用程式所需存取資源的唯一識別碼。 此值應該等於目標資源應用程式上宣告的 appId。<br>`resourceAccess` 是一個陣列，其中列出應用程式向指定的資源要求的 OAuth2.0 權限範圍和應用程式角色。 包含所指定資源的 `id` 和 `type` 值。 | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;"resourceAppId":"00000002-0000-0000-c000-000000000000",<br>&nbsp;&nbsp;&nbsp;&nbsp;"resourceAccess":[<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"id":"311a71cc-e848-46a1-bdf8-97ff7156d8e6",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"type":"Scope"<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;]<br>&nbsp;&nbsp;}<br>] </code> |
| `samlMetadataUrl` | 字串 | 應用程式 SAML 中繼資料的 URL。 | `https://MyRegisteredAppSAMLMetadata` |
| `signInUrl` | 字串 | 指定應用程式首頁的 URL。 | `https://MyRegisteredApp` |
| `signInAudience` | 字串 | 指定目前的應用程式支援哪些 Microsoft 帳戶。 支援的值包括：<ul><li>**AzureADMyOrg** - 使用者具有本組織 Azure AD 租用戶中的 Microsoft 公司或學校帳戶 (也就是單一租用戶)</li><li>**AzureADMultipleOrgs** - 使用者具有任何組織 Azure AD 租用戶中的 Microsoft 公司或學校帳戶 (也就是多租用戶)</li> <li>**AzureADandPersonalMicrosoftAccount** - 使用者具有個人 Microsoft 帳戶，或任何組織 Azure AD 租用戶中的公司或學校帳戶</li></ul> | `AzureADandPersonalMicrosoftAccount` |
| `tags` | 字串陣列 | 可用來分類及識別應用程式的自訂字串。 | <code>[<br>&nbsp;&nbsp;"ProductionApp"<br>]</code> |

## <a name="next-steps"></a>後續步驟

* 如需有關應用程式的「應用程式」和「服務主體」物件之間關係的詳細資訊，請參閱 [Azure AD 中的應用程式和服務主體物件](app-objects-and-service-principals.md)。
* 如需核心 Azure Active Directory (AD) 開發人員概念的一些定義，請參閱 [Azure AD 開發人員詞彙](developer-glossary.md)。

使用下列意見區段來提供意見反應，以協助改善及塑造我們的內容。

<!--article references -->
[AAD-APP-OBJECTS]:app-objects-and-service-principals.md
[AAD-DEVELOPER-GLOSSARY]:developer-glossary.md
[AAD-GROUPS-FOR-AUTHORIZATION]: http://www.dushyantgill.com/blog/2014/12/10/authorization-cloud-applications-using-ad-groups/
[ADD-UPD-RMV-APP]:quickstart-v1-integrate-apps-with-azure-ad.md
[APPLICATION-ENTITY]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity
[APPLICATION-ENTITY-APP-ROLE]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type
[APPLICATION-ENTITY-OAUTH2-PERMISSION]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type
[AZURE-PORTAL]: https://portal.azure.com
[DEV-GUIDE-TO-AUTH-WITH-ARM]: http://www.dushyantgill.com/blog/2015/05/23/developers-guide-to-auth-with-azure-resource-manager-api/
[GRAPH-API]: active-directory-graph-api.md
[IMPLICIT-GRANT]:v1-oauth2-implicit-grant-flow.md
[INTEGRATING-APPLICATIONS-AAD]: https://azure.microsoft.com/documentation/articles/active-directory-integrating-applications/
[O365-PERM-DETAILS]: https://msdn.microsoft.com/office/office365/HowTo/application-manifest
[O365-SERVICE-DAEMON-APPS]: https://msdn.microsoft.com/office/office365/howto/building-service-apps-in-office-365
[RBAC-CLOUD-APPS-AZUREAD]: http://www.dushyantgill.com/blog/2014/12/10/roles-based-access-control-in-cloud-applications-using-azure-ad/
