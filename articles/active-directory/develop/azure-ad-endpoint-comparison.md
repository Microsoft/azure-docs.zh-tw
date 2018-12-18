---
title: 比較 Azure AD v2.0 端點與 v1.0 端點 | Microsoft Docs
description: 了解 Azure AD v2.0 端點與 v1.0 端點之間的差異
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 5060da46-b091-4e25-9fa8-af4ae4359b6c
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/27/2018
ms.author: andret
ms.reviewer: hirsin, andret
ms.custom: aaddev
ms.openlocfilehash: b75b31ddfc77be5ed651e7b8484e41a4ae73d8d8
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406527"
---
# <a name="comparing-the-azure-ad-v20-endpoint-with-the-v10-endpoint"></a>比較 Azure AD v2.0 端點與 v1.0 端點

在開發新的應用程式時，務必知道 v1.0 與 v2.0 端點之間的差異。 以下是主要的差異，以及 v2.0 端點的一些現有限制。

> [!NOTE]
> v2.0 端點並非支援每個 Azure Active Directory (Azure AD) 案例和功能。 若要判斷是否應該使用 v2.0 端點，請閱讀相關的 [v2.0 限制](#limitations)。

## <a name="who-can-sign-in"></a>誰可以登入

![誰可以登入 v1.0 與 v2.0 端點](media/azure-ad-endpoint-comparison/who-can-sign-in.png)

* v1.0 端點只允許使用公司和學校帳戶登入您的應用程式 (Azure AD)

* V2.0 端點則允許使用公司和學校帳戶 (Azure AD) 及個人帳戶 (MSA) (hotmail.com、outlook.com、msn.com) 進行登入。

* 對於設定為[單一租用戶](single-and-multi-tenant-apps.md)的應用程式，或對於設定為指向租用戶專用端點的多租用戶應用程式 (`https://login.microsoftonline.com/{TenantId_or_Name}`)，V1.0 與 v2.0 端點也都接受 Azure AD 目錄的[來賓使用者](https://docs.microsoft.com/azure/active-directory/b2b/what-is-b2b)登入。

V2.0 端點可讓您撰寫應用程式，以接受來自個人及公司和學校帳戶的登入，並且讓您能夠撰寫與帳戶完全無關的應用程式。 例如，如果您的應用程式呼叫 [Microsoft Graph](https://graph.microsoft.io)，則有一些額外的功能和資料可供公司帳戶使用，例如他們的 SharePoint 網站或目錄資料。 但在許多動作 (例如[讀取使用者的郵件](https://graph.microsoft.io/docs/api-reference/v1.0/resources/message)) 中，相同的程式碼可以存取個人及公司和學校帳戶的電子郵件。

對於 v2.0 端點，您可以使用單一程式庫 (MSAL) 來進入消費者、教育和企業的世界。

 Azure AD v1.0 端點只接受來自公司和學校帳戶的登入。

## <a name="incremental-and-dynamic-consent"></a>增量和動態同意

使用 Azure AD v1.0 端點的應用程式必須事先指定其所需的 OAuth 2.0 權限，例如：

![權限註冊 UI](./media/azure-ad-endpoint-comparison/app_reg_permissions.png)

直接在應用程式註冊上設定的權限是**靜態的**。 雖然應用程式的靜態權限定義於 Azure 入口網站中並使程式碼好用又簡單，但是開發人員可能會面臨一些問題：

* 應用程式必須知道在應用程式建立時可能會需要的所有權限。 隨著時間新增權限有所困難。

* 應用程式必須事先知道可能會存取的所有資源。 很難建立能夠存取任意數目的資源的應用程式。

* 應用程式必須在使用者第一次登入時要求可能會需要的權限。 在某些情況下，這會導致冗長的權限清單，而讓終端使用者在初始登入時打消核准應用程式存取權的念頭。

使用 v2.0 端點，您可以在 Azure 入口網站中略過應用程式註冊資訊中靜態定義的權限，並指定您的應用程式在執行階段**動態**需要的權限，而不管應用程式註冊資訊中靜態定義的權限為何。

若要這樣做，在要求存取權杖時於 `scope` 參數中納入新的範圍，即可指定您的應用程式在應用程式階段中任何指定的時間點所需的範圍，而不需要在應用程式註冊資訊中預先定義範圍。

如果使用者還不同意新增至要求的新範圍，則系統會提示他們只能同意新的權限。 若要深入了解，您可以參閱 [權限、同意和範圍](v2-permissions-and-consent.md)。

允許應用程式透過 `scope` 參數動態要求權限，讓開發人員可以完全掌控使用者的體驗。 如果您希望，也可以選擇將您的同意體驗提前，並在一個初始授權要求中要求所有的權限。 或者，如果您的應用程式需要大量的權限，您可以選擇在嘗試使用您的應用程式的某些功能時，以遞增方式向使用者收集這些權限。

請注意，管理員代表組織行使的同意仍會使用針對該應用程式註冊的靜態權限，因此如果您需要管理員代表整個組織行使同意，建議您為使用 v2.0 端點的應用程式設定這些權限。 這可減少組織系統管理員設定應用程式所需的週期

## <a name="scopes-not-resources"></a>範圍，而非資源

若為使用 v1.0 端點的應用程式，應用程式能以**資源**或權杖收件者的方式運作。 資源可以定義它所了解的許多**範圍**或 **oAuth2Permissions**，讓用戶端應用程式得以針對特定一組範圍要求該資源的權杖。 請考慮以 Azure AD Graph API 做為資源的範例：

* 資源識別碼，或`AppID URI`：`https://graph.windows.net/`

* 範圍，或 `OAuth2Permissions`：`Directory.Read`、`Directory.Write` 等。

這一切都適用於 v2.0 端點。 應用程式仍可做為資源、定義範圍並依據 URI 識別。 用戶端應用程式仍可要求存取這些範圍。 不過，用戶端要求那些權限的方式已改變。 對於 v1.0 端點，Azure AD 的 OAuth 2.0 授權要求可能如下所示：

```text
GET https://login.microsoftonline.com/common/oauth2/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&resource=https://graph.windows.net/
...
```

其中 **resource** 參數指出用戶端應用程式要求授權的資源。 Azure AD 根據 Azure 入口網站中的靜態設定計算應用程式所需的權限，並據以發出權杖。 若為使用 v2.0 端點的應用程式，相同的 OAuth 2.0 授權要求如下所示：

```text
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&scope=https://graph.windows.net/directory.read%20https://graph.windows.net/directory.write
...
```

其中 **scope** 參數指出用戶端應用程式授權的資源和權限。 所需的資源仍存在於要求中，它只會包含在 scope 參數的每個值中。 以此方式使用 scope 參數可讓 v2.0 端點更符合 OAuth 2.0 規格，並且更貼近常見的業界作法。 它也可以讓應用程式執行先前所述的[增量同意](#incremental-and-dynamic-consent)。

## <a name="well-known-scopes"></a>知名的範圍

### <a name="offline-access"></a>離線存取

使用 v2.0 端點的應用程式可能需要針對應用程式使用新的已知權限：`offline_access` 範圍。 如果應用程式需要長期代表使用者存取資源，則所有應用程式都需要要求此權限，即使使用者可能不會主動使用此應用程式亦然。 在同意對話方塊中， `offline_access` 範圍會對使用者顯示為 [隨時存取您的資料]，而使用者必須加以同意。 要求 `offline_access` 權限可讓您的 Web 應用程式從 v2.0 端點獲取 OAuth 2.0 refresh_token。 重新整理權杖屬於長效權杖，可用來交換新的 OAuth 2.0 存取權杖以延長存取期間。

如果您的應用程式未要求 `offline_access` 範圍，則不會收到重新整理權杖。 這表示當您在 OAuth 2.0 授權碼流程中兌換授權碼時，只會從 `/token` 端點接收回一個存取權杖。 該存取權杖會短時間維持有效 (通常是一小時)，但最後終將過期。 屆時，您的應用程式必須將使用者重新導向回到 `/authorize` 端點以擷取新的授權碼。 在此重新導向期間，視應用程式的類型而定，使用者或許不需要再次輸入其認證或重新同意權限。

若要深入了解 OAuth 2.0、`refresh_tokens` 和 `access_tokens`，請參閱 [v2.0 通訊協定參考](active-directory-v2-protocols.md)。

### <a name="openid-profile-and-email"></a>OpenID、設定檔和電子郵件

過去，搭配 Azure Active Directory 的最基本 OpenID Connect 登入流程，會在產生的 id_token 中提供大量的使用者相關資訊。 id_token 中的宣告可以包含使用者的名稱、慣用使用者名稱、電子郵件地址、物件識別碼等等。

現在會限制 `openid` 範圍可提供應用程式存取權的資訊。 `openid` 範圍只允許您的應用程式將使用者登入，並接收使用者的應用程式特定識別碼。 如果您想要取得有關應用程式中使用者的個人資料，您的應用程式便必須向使用者要求其他權限。 有兩個新的範圍 (`email` 和 `profile` 範圍) 將可讓您要求其他權限。

`email` 範圍可讓您的應用程式透過 id_token 中的 `email` 宣告來存取使用者的主要電子郵件地址。 `profile` 範圍可以讓您的應用程式存取使用者的所有其他基本資訊 – 其名稱、慣用的使用者名稱、物件識別碼等等。

這可讓您以最低洩漏的方式編碼應用程式 – 您只可以向使用者要求應用程式執行其作業所需的資訊集。 如需這些範圍的詳細資訊，請參閱 [v2.0 範圍參考](v2-permissions-and-consent.md)。

## <a name="token-claims"></a>權杖宣告

v2.0 端點所簽發權杖中的宣告將不會與正式運作之 Azure AD 端點所簽發的權杖一樣。 移轉至新服務的應用程式不應該假設特定的宣告將存在於 id_tokens 或 access_tokens 中。 如需 v2.0 端點中所用不同權杖類型的詳細說明，請參閱[存取權杖](access-tokens.md)參考和 [`id_token` 參考](id-tokens.md)

## <a name="limitations"></a>限制

使用 v2.0 時有一些要注意的限制。

當您建置與 Microsoft 身分識別平台整合的應用程式時，您必須判斷 v2.0 端點和驗證通訊協定是否符合您的需求。 v1.0 端點和平台仍受到完整的支援，而且在某些方面比 v2.0 的功能更豐富。 不過，v2.0 為開發人員[帶來極大的好處](azure-ad-endpoint-comparison.md)。

以下是我們在這個時間點針對開發人員所提供的簡化建議：

* 如果您必須在應用程式中支援個人 Microsoft 帳戶，請使用 v2.0。 但在這樣做之前，請務必了解本文中討論的限制。

* 如果您的應用程式只需要支援 Microsoft 公司和學校帳戶，請不要使用 v2.0。 請改為參閱 [v1.0 指南](azure-ad-developers-guide.md)。

v2.0 端點會進化到排除此處所列的限制，屆時您只需要使用 v2.0 端點即可。 在此同時，請使用本文來判斷 v2.0 端點是否適合您。 我們將持續更新本文，以反映 v2.0 端點目前的狀態。 請記得回來重新評估您對 v2.0 功能的需求。

### <a name="restrictions-on-app-types"></a>應用程式類型的限制

v2.0 端點目前不支援下列類型的應用程式。 如需所支援應用程式類型的描述，請參閱 [v2.0 中的應用程式類型](v2-app-types.md)。

#### <a name="standalone-web-apis"></a>獨立的 Web API

您可以使用 v2.0 端點來[建置使用 OAuth 2.0 保護的 Web API](v2-app-types.md#web-apis)。 不過，該 Web API 只能從具有相同「應用程式識別碼」的應用程式接收權杖。 您無法從具有不同「應用程式識別碼」的用戶端存取 Web API。 該用戶端無法要求或取得您 Web API 的權限。

若要了解如何建置 Web API 以接受來自具有相同「應用程式識別碼」之用戶端的權杖，請參閱[開始使用 v2.0](v2-overview.md#getting-started) 一節中的 v2.0 端點 Web API 範例。

### <a name="restrictions-on-app-registrations"></a>應用程式註冊的限制

目前，針對您想要與 v2.0 端點整合的每個應用程式，您都必須在新的 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)中建立應用程式註冊。 現有的 Azure AD 或 Microsoft 帳戶應用程式與 v2.0 端點不相容。 在「應用程式註冊入口網站」以外的任何入口網站註冊的應用程式也與 v2.0 端點不相容。

此外，您在[應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)中建立的應用程式註冊還需注意下列幾點︰

* 每個「應用程式識別碼」只允許兩個應用程式密碼。

* 由使用個人 Microsoft 帳戶之使用者所註冊的應用程式註冊，只能透過單一開發人員帳戶來檢視及管理。 不能在多位開發人員之間共用。 如果您要和多位開發人員共用您的應用程式註冊，可以使用 Azure AD 帳戶登入註冊入口網站來建立應用程式。

* 所允許的重新導向 URL 的格式有幾項限制。 如需有關重新導向 URL 的詳細資訊，請參閱下一節。

### <a name="restrictions-on-redirect-urls"></a>重新導向 URL 的限制

針對已在「應用程式註冊入口網站」中註冊的應用程式，僅限使用一組受限的重新導向 URL 值。 Web 應用程式和服務的重新導向 URL 必須以配置 `https` 開頭，而且所有重新導向 URL 值必須共用單一 DNS 網域。 例如，您不能註冊具有下列其中一個重新導向 URL 的 Web 應用程式：

* `https://login-east.contoso.com`  
* `https://login-west.contoso.com`

註冊系統會將現有重新導向 URL 的整個 DNS 名稱與您要新增之重新導向 URL 的 DNS 名稱做比較。 如果下列任一條件成立，新增 DNS 名稱的要求就會失敗︰  

* 新重新導向 URL 的整個 DNS 名稱與現有重新導向 URL 的 DNS 名稱不相符。

* 新重新導向 URL 的整個 DNS 名稱不是現有重新導向 URL 的子網域。

例如，如果應用程式具有下列重新導向 URL：

`https://login.contoso.com`

您可以在其中新增，就像這樣：

`https://login.contoso.com/new`

在此情況下，DNS 名稱會完全相符。 或者，您可以這樣做：

`https://new.login.contoso.com`

在此情況下，您參照的是 login.contoso.com 的 DNS 子網域。 如果您希望應用程式以 `login-east.contoso.com` 和 `login-west.contoso.com` 作為重新導向 URL，您必須依下列序新增這些重新導向 URL︰

`https://contoso.com`  
`https://login-east.contoso.com`  
`https://login-west.contoso.com`  

您可以新增後面兩個，因為它們是第一個重新導向 URL (contoso.com) 的子網域。 即將推出的版本將會移除這項限制。

也請注意，一個特定應用程式只可以有 20 個回覆 URL。

若要了解如何在「應用程式註冊入口網站」中註冊應用程式，請參閱[如何使用 v2.0 端點註冊 App](quickstart-v2-register-an-app.md)。

### <a name="restrictions-on-libraries-and-sdks"></a>程式庫和 SDK 的限制

目前對 v2.0 端點提供的程式庫支援相當有限。 如果您想要在實際執行應用程式中使用 v2.0 端點，您有下列選項：

* 如果您要建置 Web 應用程式，您可以安心地使用 Microsoft 正式運作的伺服器端中介軟體來執行登入和權杖驗證。 這些包括了適用於 ASP.NET 的 OWIN Open ID Connect 中介軟體和 Node.js Passport 外掛程式。 如需使用 Microsoft 中介軟體的程式碼範例，請參閱[開始使用 v2.0](v2-overview.md#getting-started) 一節。

* 如果您正在建置傳統型或行動應用程式，可以使用其中一個預覽 Microsoft Authentication Library (MSAL)。 這些程式庫處於支援生產環境的預覽階段，因此可以在生產應用程式中安全地使用。 您可以在[驗證程式庫參考](reference-v2-libraries.md)中，閱讀更多和預覽條款與可用程式庫相關的詳細資訊。

* 針對 Microsoft 程式庫未包含的其他平台，您可以透過直接在應用程式程式碼中傳送和接收通訊協定訊息，來與 v2.0 端點整合。 v2.0 OpenID Connect 和 OAuth 通訊協定[有明確的文件記載](active-directory-v2-protocols.md)，可協助您執行這種整合。

* 最後，您可以使用開放原始碼 OpenID Connect 和 OAuth 程式庫來與 v2.0 端點整合。 v2.0 通訊協定應該不需進行重大變更，即可與許多開放原始碼通訊協定程式庫相容。 這類的程式庫的可用性會依語言和平台而有所不同。 [Open ID Connect](http://openid.net/connect/) 和 [OAuth 2.0](http://oauth.net/2/) 網站保有一份常用的實作清單。 如需詳細資訊，以及已用 v2.0 端點測試的開放原始碼用戶端程式庫和範例的清單，請參閱 [Azure Active Directory v2.0 和驗證程式庫](reference-v2-libraries.md)。

* 基於參考目的，適用於 v2.0 常見端點的 `.well-known` 端點是 `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`。  使用您的租用戶識別碼來取代 `common`，以取得您租用戶特有的資料。  

### <a name="restrictions-on-protocols"></a>通訊協定的限制

v2.0 端點不支援 SAML 或 WS-同盟，只支援 Open ID Connect 和 OAuth 2.0。 v2.0 端點中並未納入 OAuth 通訊協定的所有功能與特性。

v2.0 端點中目前「並未提供」或「並未支援」下列通訊協定功能與特性：

* 只有在已設定選擇性宣告，以及範圍是要求中指定的 scope=email 時，才會傳回 `email` 宣告。 不過，除此之外，變更為 v2.0 端點的行為已更新為進一步遵循 Open ID Connect 和 OAuth2.0 標準。

* v2.0 端點不支援在識別碼權杖中發出角色或群組宣告。

* v2.0 端點不支援 [OAuth 2.0 資源擁有者密碼認證授與 (英文)](https://tools.ietf.org/html/rfc6749#section-4.3)。

若要進一步了解 v2.0 端點中支援的通訊協定功能範圍，請參閱我們的 [OpenID Connect 與 OAuth 2.0 通訊協定參考](active-directory-v2-protocols.md)。

#### <a name="saml-restrictions"></a>SAML 限制

如果您已在 Windows 應用程式中使用 Active Directory Authentication Library (ADAL)，您可能已利用使用「安全性聲明標記語言」(SAML) 判斷提示授與的 Windows 整合式驗證。 有了此授與之後，同盟 Azure AD 租用戶的使用者便可向其內部部署 Active Directory 執行個體以無訊息方式進行驗證，而不需輸入認證。 目前 v2.0 端點不支援 SAML 判斷提示授與。
