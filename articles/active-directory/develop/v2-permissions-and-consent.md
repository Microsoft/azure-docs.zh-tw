---
title: Azure Active Directory v2.0 範圍、權限及同意 | Microsoft Docs
description: Azure AD v2.0 端點中的授權說明，包括範圍、權限及同意。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 8f98cbf0-a71d-4e34-babf-e644ad9ff423
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: hirsin, jesakowi, justhu
ms.custom: aaddev
ms.openlocfilehash: da8eebb2fc6b87b8916e944495679b45aa34dbf2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960323"
---
# <a name="permissions-and-consent-in-the-azure-active-directory-v20-endpoint"></a>Azure Active Directory v2.0 端點中的權限和同意

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

與 Microsoft 身分識別平台整合的應用程式，會遵循可讓使用者和系統管理員控制資料存取方式的授權模型。 v2.0 端點的授權模型實作已更新，並且變更了應用程式必須與 Microsoft 身分識別平台互動的方式。 本文涵蓋此授權模型的基本概念，包括範圍、權限及同意。

> [!NOTE]
> v2.0 端點並未支援所有的案例和功能。 若要判斷您是否應該使用 v2.0 端點，請參閱 [v2.0 限制](active-directory-v2-limitations.md)。

## <a name="scopes-and-permissions"></a>範圍和權限

Microsoft 身分識別平台會實作 [OAuth 2.0](active-directory-v2-protocols.md) 授權通訊協定。 OAuth 2.0 是一種可讓協力廠商應用程式代表使用者存取 Web 主控資源的方法。 任何與 Microsoft 身分識別平台整合的 Web 主控資源都具有資源識別碼 (或稱為「應用程式識別碼 URI」)。 例如，Microsoft 的 Web 主控資源包括：

* Microsoft Graph：`https://graph.microsoft.com`
* Office 365 郵件 API：`https://outlook.office.com`
* Azure AD Graph：`https://graph.windows.net`

> [!NOTE]
> 強烈建議您使用 Microsoft Graph，而不要使用 Azure AD Graph、Office 365 郵件 API 等工具。

這也適用於已與 Microsoft 身分識別平台整合的第三方資源。 任何這些資源也都可以定義一組權限，可用來進一步細分該資源的功能。 例如，[Microsoft Graph](https://graph.microsoft.com) 除了別的之外，還定義了權限來執行下列工作：

* 讀取使用者的行事曆
* 寫入使用者的行事曆
* 以使用者身分傳送郵件

藉由定義這些類型的權限，資源可以更精細地掌控其資料及 API 功能的公開方式。 第三方應用程式可向使用者和系統管理員要求這些權限，且必須在他們核准要求後，應用程式才可存取資料或代表使用者執行動作。 透過將資源的功能切割成較小的權限集，便可將協力廠商應用程式建置成只要求它們執行其功能所需的特定權限。 使用者和系統管理員可確切知道應用程式可存取哪些資料，因而能夠更確信應用程式的行為不具惡意企圖。 開發人員應一律遵守最低權限的概念，而僅就其應用程式運作所需的程度要求權限。

在 OAuth 中，這些類型的權限也稱為「範圍」。 一般也常直接稱之為「權限」。 權限在 Microsoft 身分識別平台中會以字串值表示。 繼續討論 Microsoft Graph 範例，每個權限的字串值如下：

* 使用 `Calendars.Read` 來讀取使用者的行事曆
* 使用 `Calendars.ReadWrite` 來寫入使用者的行事曆
* 使用 `Mail.Send` 來以使用者身分傳送郵件

應用程式最常用來要求這些權限的方式，是在對 v2.0 授權端點的要求中指定範圍。 不過，較高的特定權限只能透過系統管理員的同意而授與，且通常會使用[系統管理員同意端點](v2-permissions-and-consent.md#admin-restricted-scopes)來要求/授與。 繼續閱讀以深入了解。

## <a name="permission-types"></a>權限類型

Microsoft 身分識別平台支援兩種類型的權限：**委派權限**和**應用程式權限**。

- **委派權限**供已有登入使用者的應用程式使用。 針對這些應用程式，使用者或系統管理員要同意應用程式所要求的權限，然後對應用程式委派權限，讓其在呼叫目標資源時以登入使用者的身分行事。 有些委派權限可由非系統管理使用者同意，但有些較高的特定權限則需要[系統管理員的同意](v2-permissions-and-consent.md#admin-restricted-scopes)。  

- **應用程式權限**供沒有登入使用者的應用程式在執行時使用；例如，當作背景服務或精靈來執行的應用程式。  應用程式權限只能[由系統管理員同意](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant)。 

_有效權限_是應用程式向目標資源提出要求時所將具備的權限。 請務必了解為您的應用程式授與的委派權限和應用程式權限有何差異，並了解應用程式在呼叫目標資源時所具備的有效權限。

- 就委派權限來說，應用程式的_有效權限_是應用程式 (透過同意) 所獲得授與的委派權限和目前已登入使用者的權限二者的最小權限交集。 應用程式絕對不會擁有已登入使用者權限以外的權限。 在組織內，已登入使用者的權限會由原則或是一或多個系統管理員角色的成員資格決定。 如需有關系統管理員角色的詳細資訊，請參閱[在 Azure Active Directory 中指派系統管理員角色](../users-groups-roles/directory-assign-admin-roles.md)。
  例如，假設已經對您的應用程式授與 _User.ReadWrite.All_ 委派權限。 此權限名義上會對應用程式授與讀取及更新組織中每個使用者設定檔的權限。 如果已登入使用者是全域管理員，應用程式便能夠更新組織中每個使用者的設定檔。 不過，如果已登入使用者不是全域管理員角色，應用程式則只能更新該已登入使用者的設定檔。 應用程式有權代表其行事的使用者沒有這些權限，因此應用程式無法更新組織中其他使用者的設定檔。
  
- 就應用程式權限來說，應用程式的_有效權限_將是權限所隱含的完整層級權限。 例如，具有 _User.ReadWrite.All_ 應用程式權限的應用程式可以更新組織中每個使用者的設定檔。 

## <a name="openid-connect-scopes"></a>OpenId Connect 範圍

v2.0 的 OpenID Connect 實作有一些定義妥善但不會套用至特定資源的範圍：`openid`、`email`、`profile` 和 `offline_access`。

### <a name="openid"></a>openid

如果應用程式使用 [OpenID Connect](active-directory-v2-protocols.md) 來執行登入，它就必須要求 `openid` 範圍。 `openid` 範圍會在工作帳戶同意頁面上顯示為「將您登入」權限，而在個人 Microsoft 帳戶同意頁面上會顯示為「檢視您的設定檔並使用您的 Microsoft 帳戶連接到應用程式和服務」權限。 有了此權限之後，應用程式便能夠以 `sub` 宣告的形式接收使用者的唯一識別碼。 它也會為應用程式提供 UserInfo 端點的存取權。 `openid` 範圍也可在 v2.0 權杖端點用來取得識別碼權杖，而該權杖可用來保護應用程式不同元件之間的 HTTP 呼叫。

### <a name="email"></a>電子郵件

`email` 範圍可以與 `openid` 範圍及任何其他範圍搭配使用。 它會以 `email` 宣告的形式為應用程式提供使用者主要電子郵件地址的存取權。 只有當電子郵件地址與使用者帳戶關聯 (兩者並不一定會關聯) 時，`email` 宣告才會包含在權杖中。 如果它使用 `email` 範圍，您的應用程式就應該做好準備，以處理權杖中沒有 `email` 宣告的情況。

### <a name="profile"></a>設定檔

`profile` 範圍可以與 `openid` 範圍及任何其他範圍搭配使用。 它會為應用程式提供大量使用者相關資訊的存取權。 它能存取的資訊包括但不限於使用者的名字、姓氏、慣用使用者名稱及物件識別碼。 如需特定使用者之識別碼權杖中可用的設定檔宣告完整清單，請參閱 [`id_tokens` 參考](id-tokens.md)。

### <a name="offlineaccess"></a>offline_access

[`offline_access` 範圍](http://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess)可延長您應用程式代表使用者存取資源的時間。 在工作帳戶同意畫面上，此範圍會顯示為「隨時存取您的資料」權限。 在個人 Microsoft 帳戶同意頁面上，它會顯示為「隨時存取您的資訊」權限。 當使用者核准 `offline_access` 範圍時，您的應用程式將可從 v2.0 權杖端點收到重新整理權杖。 重新整理權杖是長期權杖。 您的應用程式可以在舊存取權杖到期時取得新的存取權杖。

如果您的應用程式未要求 `offline_access` 範圍，則不會收到重新整理權杖。 這意謂著當您在 [OAuth 2.0 授權碼流程](active-directory-v2-protocols.md)中兌換授權碼時，您只會從 `/token` 端點收到存取權杖。 存取權杖的有效期短。 存取權杖的有效期通常在一小時內。 屆時，您的應用程式將必須把使用者重新導向回 `/authorize` 端點，以擷取新的授權碼。 在此重新導向期間，視應用程式的類型而定，使用者可能需要重新輸入其認證或重新同意權限。

如需有關如何取得及使用重新整理權杖的詳細資訊，請參閱 [v2.0 通訊協定參考](active-directory-v2-protocols.md)。

## <a name="requesting-individual-user-consent"></a>要求個別使用者同意

在 [OpenID Connect 或 OAuth 2.0](active-directory-v2-protocols.md) 授權要求中，應用程式可以使用 `scope` 查詢參數來要求它所需的權限。 例如，當使用者登入應用程式時，應用程式會傳送如以下範例的要求 (加入分行符號是為了增加可讀性)：

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendars.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

`scope` 參數是應用程式所要求的委派權限清單 (以空格分隔)。 藉由將權限值附加至資源的識別碼 (應用程式識別碼 URI)，即可指出每個權限。 在要求範例中，應用程式需要權限來讀取使用者的行事曆，以及以使用者身分傳送郵件。

在使用者輸入其認證之後，v2.0 端點會檢查是否有相符的 「使用者同意」記錄。 如果使用者過去未曾對所要求的任何權限表示同意，且系統管理員也未代表整個組織同意這些權限，v2.0 端點就會要求使用者授與所要求的權限。

![工作帳戶同意](./media/v2-permissions-and-consent/work_account_consent.png)

當使用者核准權限要求時，系統就會記錄同意，使用者在後續登入應用程式時將不需要重新表示同意。

## <a name="requesting-consent-for-an-entire-tenant"></a>要求整個租用戶的同意

通常，當組織購買應用程式的授權或訂用帳戶時，都會想要主動設定應用程式以供組織的所有成員使用。 在此過程中，系統管理員可以授與同意，讓應用程式代表租用戶中的任何使用者執行動作。 如果系統管理員為整個租用戶授與同意，該組織的使用者便不會看見應用程式的同意頁面。

若要為租用戶中的所有使用者要求委派權限的同意，您的應用程式可以使用系統管理員同意端點。

此外，應用程式必須使用管理員同意端點來要求應用程式權限。

## <a name="admin-restricted-permissions"></a>受管理員限制的權限

Microsoft 生態系統中的某些高特權權限可以設定為「受系統管理員限制」。 這類權限的範例包括：

* 使用 `User.Read.All` 讀取所有使用者的完整設定檔
* 使用 `Directory.ReadWrite.All` 將資料寫入組織的目錄
* 使用 `Groups.Read.All` 讀取組織目錄中的所有群組

雖然取用者使用者可以為應用程式授與這類資料的存取權，但組織使用者會受到限制，而無法授與同一組機密公司資料的存取權。 如果您的應用程式向組織使用者要求這其中一個權限的存取權，使用者將會收到錯誤訊息，指出他們未獲授權而無法對您應用程式的權限表示同意。

如果您的應用程式需要存取組織的受系統管理員限制範圍，您應該同樣使用系統管理員同意端點，直接向公司系統管理員要求權限，接下來將會說明。

如果應用程式要求高權限的委派權限，且系統管理員透過管理員同意端點授與了這些權限，則會為租用戶中的所有使用者授與同意。

如果應用程式要求應用程式權限，且系統管理員透過管理員同意端點授與了這些權限，則不會代表任何特定使用者完成此授與。 此時會*直接*為用戶端應用程式授與權限。 通常只有精靈服務以及在背景中執行的其他非互動式應用程式會使用這些類型的權限。

## <a name="using-the-admin-consent-endpoint"></a>使用系統管理員同意端點

如果公司的系統管理員在使用您的應用程式時被導向至授權端點，Microsoft 身分識別平台將會偵測使用者的角色，並詢問他們是否要代表整個租用戶同意您所要求的權限。 不過，還有如果您想要主動要求由系統管理員代表整個租用戶授與權限，您也可以使用專用的管理員同意端點。 在要求應用程式權限時也必須使用此端點 (此權限無法使用授權端點來要求)。

如果您依照這些步驟，您的應用程式便能為租用戶中所有的使用者要求權限，包括受管理員限制的範圍。 這是高權限作業，只有在您的案例有需要時才應執行。

若要查看實作這些步驟的程式碼範例，請參閱[受系統管理員限制的範圍範例](https://github.com/Azure-Samples/active-directory-dotnet-admin-restricted-scopes-v2)。

### <a name="request-the-permissions-in-the-app-registration-portal"></a>在應用程式註冊入口網站中要求權限

管理員同意不接受範圍參數，因此所要求的任何權限都必須以靜態方式定義在應用程式的註冊中。 一般情況下的最佳做法是，確定以靜態方式為指定的應用程式定義的權限，是應用程式將以動態方式/累加方式要求的權限超集。

若要為應用程式設定以靜態方式要求的權限清單： 
1. 在[應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)中移至您的應用程式，或[建立應用程式](quickstart-v2-register-an-app.md) (如果您尚未這麼做)。
2. 找出 [Microsoft Graph 權限] 區段，然後新增您應用程式所需的權限。
3. [儲存] 應用程式註冊。

### <a name="recommended-sign-the-user-in-to-your-app"></a>建議︰將使用者登入您的應用程式

通常，當您建置使用系統管理員同意端點的應用程式時，應用程式會需要一個可供系統管理員核准應用程式權限的頁面或檢視。 此頁面可以是應用程式註冊流程的一部分、應用程式設定的一部分，或是專用的「連接」流程。 在許多情況下，應用程式只在使用者利用工作或學校 Microsoft 帳戶登入之後顯示此「連接」檢視是很合理的。

將使用者登入應用程式時，您可以先識別系統管理員所屬的組織，然後再要求他們核准必要的權限。 雖然這並非絕對必要，但這麼做可協助您為組織使用者建立更直覺式的體驗。 若要讓使用者登入，請遵循我們的 [v2.0 通訊協定教學課程](active-directory-v2-protocols.md)。

### <a name="request-the-permissions-from-a-directory-admin"></a>向目錄管理員要求權限

當您已準備好向組織的系統管理員要求權限時，您可以將使用者重新導向到 v2.0「系統管理員同意端點」。

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

```
// Pro tip: Try pasting the below request in a browser!
```

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| 參數 | 條件 | 說明 |
| --- | --- | --- |
| `tenant` | 必要 | 您想要要求權限的目錄租用戶。 可以提供 GUID 或易記的名稱格式，或是一般會參考使用 "common" (如範例所示)。 |
| `client_id` | 必要 | [應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)指派給您應用程式的「應用程式識別碼」。 |
| `redirect_uri` | 必要 |您想要傳送回應以供應用程式處理的重新導向 URI。 它必須與您在應用程式註冊入口網站中註冊的其中一個重新導向 URI 完全相符。 |
| `state` | 建議 | 同樣會隨權杖回應傳回之要求中所包含的值。 它可以是您想要的任何內容的字串。 請在驗證要求出現之前，先使用此狀態在應用程式中將使用者狀態的相關資訊 (例如他們之前所在的網頁或檢視) 編碼。 |

此時，Azure AD 會要求租用戶系統管理員登入來完成要求。 系統會請系統管理員核准您在應用程式註冊入口網站中，為您應用程式要求的所有權限。

#### <a name="successful-response"></a>成功回應

如果系統管理員為您的應用程式核准權限，則成功的回應看起來會像這樣︰

```
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| 參數 | 說明 |
| --- | --- | --- |
| `tenant` | 將應用程式所要求的權限授與應用程式的目錄租用戶 (採用 GUID 格式)。 |
| `state` | 一個包含在要求中而將一併在權杖回應中傳回的值。 它可以是您想要的任何內容的字串。 此狀態用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |
| `admin_consent` | 將設定為 **true**。 |

#### <a name="error-response"></a>錯誤回應

如果系統管理員沒有為您的應用程式核准權限，則失敗的回應看起來會像這樣︰

```
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| 參數 | 說明 |
| --- | --- | --- |
| `error` |用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| `error_description` |可協助開發人員識別錯誤根本原因的特定錯誤訊息。 |

從系統管理員同意端點收到成功回應之後，您的應用程式便已取得它所要求的權限。 接著，您可以針對您想要的資源要求權杖。

## <a name="using-permissions"></a>使用權限

在使用者同意您的應用程式的權限之後，您的應用程式即可取得存取權杖，而這些權杖代表您的應用程式存取資源的權限。 一個存取權杖只能用於一個單一資源，但存取權杖內部所編碼的是您應用程式針對該資源已獲得的每項權限。 若要取得存取權杖，您的應用程式可以對 v2.0 權杖端點提出要求，像這樣：

```
POST common/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/json

{
    "grant_type": "authorization_code",
    "client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
    "scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
    "code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq..."
    "redirect_uri": "https://localhost/myapp",
    "client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
}
```

您可以在對資源提出的 HTTP 要求中使用產生的存取權杖。 它會可靠地向資源指出您的應用程式具有可執行特定工作的適當權限。 

如需有關 OAuth 2.0 通訊協定及如何取得存取權杖的詳細資訊，請參閱 [v2.0 端點通訊協定參考](active-directory-v2-protocols.md)。

## <a name="troubleshooting"></a>疑難排解

如果您或應用程式的使用者在同意程序中發現非預期的錯誤，請參閱本文以取得疑難排解步驟：[對應用程式執行同意時出現非預期的錯誤](../manage-apps/application-sign-in-unexpected-user-consent-error.md)。
