---
title: 針對 Azure Active Directory 中的企業應用程式自訂 SAML 權杖中發出的宣告 | Microsoft Docs
description: 了解如何針對 Azure Active Directory 中的企業應用程式自訂 SAML 權杖中發出的宣告
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: f1daad62-ac8a-44cd-ac76-e97455e47803
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/11/2018
ms.author: celested
ms.reviewer: jeedes
ms.custom: aaddev
ms.openlocfilehash: 80842f7e99ee0c58f1615892f3c3c4adf03119b6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956956"
---
# <a name="how-to-customize-claims-issued-in-the-saml-token-for-enterprise-applications-in-azure-ad"></a>如何：針對 Azure AD 中的企業應用程式自訂 SAML 權杖中發出的宣告

現在，Azure Active Directory 已支援大部分企業應用程式的單一登入，包括 Azure AD 資源庫中預先整合的應用程式，以及自訂應用程式。 當使用者利用 SAML 2.0 通訊協定來透過 Azure AD 向應用程式驗證時，Azure AD 會將權杖傳送給應用程式 (透過 HTTP POST)。 然後，應用程式會驗證並使用權杖將使用者登入，而不會提示輸入使用者名稱和密碼。 這些 SAML 權杖包含關於使用者的資訊片段 (稱為「宣告」)。

在身分識別交談中，「宣告」是身分識別提供者在為使用者發出的權杖中關於使用者說明的資訊。 在 [SAML 權杖](http://en.wikipedia.org/wiki/SAML_2.0)中，此資料通常包含在 SAML 屬性陳述式中。 使用者的唯一識別碼通常在 SAML Subject 中表示，也稱為「名稱識別碼」。

根據預設，Azure Active Directory 會發出 SAML 權杖給您的應用程式，其包含一個 NameIdentifier 宣告，具有使用者在 Azure AD 中的使用者名稱 (也稱為使用者主體名稱) 的值。 此值可唯一識別使用者。 SAML 權杖也包含額外的宣告，其包含使用者的電子郵件地址、名字和姓氏。

若要檢視或編輯在 SAML 權杖中對應用程式發出的宣告，請在 Azure 入口網站中開啟應用程式。 然後在應用程式的 [使用者屬性] 區段中，選取 [檢視和編輯所有其他使用者屬性] 核取方塊。

![使用者屬性區段][1]

編輯在 SAML 權杖中簽發的宣告有兩個可能原因：
* 應用程式是設計為要求不同的宣告 URI 組或宣告值。
* 應用程式已部署為要求 NameIdentifier 宣告必須是 Azure Active Directory 中儲存之 username (也稱為使用者主體名稱) 以外的項目。

您可以編輯任何預設的宣告值。 選取 SAML 權杖屬性資料表中的宣告資料列。 這會開啟 [編輯屬性] 區段，然後您可以編輯宣告名稱、值，以及與宣告相關聯的命名空間。

![編輯使用者屬性][2]

您也可以使用快顯功能表來移除宣告 (除了 NameIdentifier 以外)，按一下 [...] 圖示即可開啟該功能表。 使用 [新增屬性] 按鈕，也可以新增宣告。

![編輯使用者屬性][3]

## <a name="editing-the-nameidentifier-claim"></a>編輯 NameIdentifier 宣告
若要解決使用不同的使用者名稱部署應用程式的問題，請按一下 [使用者屬性] 區段中的 [使用者識別碼] 下拉式清單。 這個動作會提供包含數個不同選項的對話方塊：

![編輯使用者屬性][4]

在下拉式清單中，選取 [user.mail]，將 NameIdentifier 宣告設定為使用者在目錄中的電子郵件地址。 或者，選取 **user.onpremisessamaccountname** 設定為從內部部署 Azure AD 同步處理的使用者 SAM 帳戶名稱。

您也可以使用特殊的 **ExtractMailPrefix()** 函式，從電子郵件地址、SAM 帳戶名稱或使用者主體名稱中將網域尾碼移除。 這只會擷取使用者名稱的第一個部分 (例如，"joe_smith" 而不是 joe_smith@contoso.com)。

![編輯使用者屬性][5]

我們現在還新增了 **join()** 函式，以聯結已驗證的網域與使用者識別碼值。 當您選取 [使用者識別碼] 中的 join() 函式時，先選取像是電子郵件地址或使用者主體名稱的使用者識別碼，然後在第二個下拉式清單中選取已驗證的網域。 如果您選取包含已驗證網域的電子郵件地址，則 Azure AD 會從 joe_smith@contoso.com 的第一個值 joe_smith 中擷取使用者名稱，並將它與 contoso.onmicrosoft.com 附加。 請參閱下列範例：

![編輯使用者屬性][6]

## <a name="adding-claims"></a>新增宣告
新增宣告時，您可以指定屬性名稱 (不必根據 SAML 規格嚴格遵循 URI 模式)。 將值設定為儲存在目錄中的任何使用者屬性。

![新增使用者屬性][7]

例如，您必須傳送使用者在其組織內所屬的部門作為宣告 (例如，銷售)。 輸入應用程式預期的宣告名稱，然後選取 **user.department** 作為值。

> [!NOTE]
> 如果指定的使用者沒有針對選取的屬性儲存的值，則權杖中不會發出該宣告。

> [!TIP]
> 只有在使用 [Azure AD Connect 工具](../hybrid/whatis-hybrid-identity.md)從內部部署的 Active Directory 同步處理使用者資料時，才支援 **user.onpremisesecurityidentifier** 和 **user.onpremisesamaccountname**。

## <a name="restricted-claims"></a>受限制的宣告

SAML 有一些受限制的宣告。 如果您新增這些宣告，則 Azure AD 不會傳送這些宣告。 以下是受限制的 SAML 宣告集：

    | 宣告類型 (URI) |
    | ------------------- |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/expiration |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/expired |
    | http://schemas.microsoft.com/identity/claims/accesstoken |
    | http://schemas.microsoft.com/identity/claims/openid2_id |
    | http://schemas.microsoft.com/identity/claims/identityprovider |
    | http://schemas.microsoft.com/identity/claims/objectidentifier |
    | http://schemas.microsoft.com/identity/claims/puid |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier[MR1] |
    | http://schemas.microsoft.com/identity/claims/tenantid |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationinstant |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod |
    | http://schemas.microsoft.com/accesscontrolservice/2010/07/claims/identityprovider |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/groups |
    | http://schemas.microsoft.com/claims/groups.link |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/role |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/wids |
    | http://schemas.microsoft.com/2014/09/devicecontext/claims/iscompliant |
    | http://schemas.microsoft.com/2014/02/devicecontext/claims/isknown |
    | http://schemas.microsoft.com/2012/01/devicecontext/claims/ismanaged |
    | http://schemas.microsoft.com/2014/03/psso |
    | http://schemas.microsoft.com/claims/authnmethodsreferences |
    | http://schemas.xmlsoap.org/ws/2009/09/identity/claims/actor |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/samlissuername |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/confirmationkey |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/primarygroupsid |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/authorizationdecision |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/authentication |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/sid |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/denyonlyprimarygroupsid |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/denyonlyprimarysid |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/denyonlysid |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/denyonlywindowsdevicegroup |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsdeviceclaim |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsdevicegroup |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsfqbnversion |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/windowssubauthority |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsuserclaim |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/x500distinguishedname |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn |
    | http://schemas.microsoft.com/ws/2008/06/identity/claims/ispersistent |
    | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/privatepersonalidentifier |
    | http://schemas.microsoft.com/identity/claims/scope |

## <a name="next-steps"></a>後續步驟
* [Azure Active Directory 中的應用程式管理](../manage-apps/what-is-application-management.md)
* [設定對不在 Azure Active Directory 應用程式庫中的應用程式的單一登入](../manage-apps/configure-federated-single-sign-on-non-gallery-applications.md)
* [對 SAML 型單一登入進行疑難排解](howto-v1-debug-saml-sso-issues.md)

<!--Image references-->
[1]: ./media/active-directory-saml-claims-customization/user-attribute-section.png
[2]: ./media/active-directory-saml-claims-customization/edit-claim-name-value.png
[3]: ./media/active-directory-saml-claims-customization/delete-claim.png
[4]: ./media/active-directory-saml-claims-customization/user-identifier.png
[5]: ./media/active-directory-saml-claims-customization/extractemailprefix-function.png
[6]: ./media/active-directory-saml-claims-customization/join-function.png
[7]: ./media/active-directory-saml-claims-customization/add-attribute.png
