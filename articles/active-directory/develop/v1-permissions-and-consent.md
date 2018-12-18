---
title: Azure Active Directory 中的權限 | Microsoft Docs
description: 了解 Azure Active Directory 中的權限及其使用方式。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 6c0dc122-2cd8-4d70-be5a-3943459d308e
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: jesakowi, justhu
ms.custom: aaddev
ms.openlocfilehash: 93bc3db2b7cf3002efc93f1e8006c5362eddab9f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46959966"
---
# <a name="permissions-and-consent-in-the-azure-active-directory-v10-endpoint"></a>Azure Active Directory v1.0 端點中的權限和同意

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

Azure Active Directory (Azure AD) 廣泛使用 OAuth 和 OpenID Connect (OIDC) 流程的權限。 當應用程式收到來自 Azure AD 的存取權杖時，它會納入宣告來描述應用程式對特定資源的權限。

*權限* (也稱為*範圍*) 因為只需要確認權杖包含應用程式所呼叫 API 的適當權限，所以會讓資源的授權變輕鬆。

## <a name="types-of-permissions"></a>權限的類型

Azure AD 定義兩種權限：

* **委派權限** - 供已有登入使用者的應用程式使用。 針對這些應用程式，使用者或系統管理員要同意應用程式所要求的權限，然後對應用程式委派權限，讓其在呼叫 API 時以登入使用者的身分行事。 視 API 而定，使用者不一定能直接同意 API，而會[需要系統管理員提供「管理員同意」](/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview#understanding-user-and-admin-consent)。
* **應用程式權限** - 供沒有登入使用者的應用程式在執行時使用；例如，當作背景服務或精靈來執行的應用程式。 應用程式權限通常很強大，允許跨使用者界限存取資料，或存取僅限系統管理員存取的資料，因此這種權限只能[由系統管理員同意](/azure/active-directory/develop/active-directory-v2-scopes#requesting-consent-for-an-entire-tenant)。

有效權限是應用程式向 API 提出要求時會具備的權限。 

* 就委派權限來說，應用程式的有效權限是應用程式 (透過同意) 所獲得授與的委派權限和目前已登入使用者的權限二者的最小權限交集。 應用程式絕對不會擁有已登入使用者權限以外的權限。 在組織內，已登入使用者的權限會由原則或是一或多個系統管理員角色的成員資格決定。 如需有關系統管理員角色的詳細資訊，請參閱[在 Azure AD 中指派系統管理員角色](../users-groups-roles/directory-assign-admin-roles.md)。
    例如，假設已經對應用程式授與在 Microsoft Graph 中的 `User.ReadWrite.All` 委派權限。 此權限名義上會對應用程式授與讀取及更新組織中每個使用者設定檔的權限。 如果已登入使用者是全域管理員，應用程式便能夠更新組織中每個使用者的設定檔。 不過，如果已登入使用者不是全域管理員角色，應用程式則只能更新該已登入使用者的設定檔。 應用程式有權代表其行事的使用者沒有這些權限，因此應用程式無法更新組織中其他使用者的設定檔。
* 就應用程式權限來說，應用程式的有效權限是權限所隱含的完整層級權限。 例如，具有 `User.ReadWrite.All` 應用程式權限的應用程式可以更新組織中每個使用者的設定檔。

## <a name="permission-attributes"></a>權限屬性
Azure AD 中的權限有許多屬性，可協助使用者、系統管理員或應用程式開發人員，就權限可授與什麼資源的存取權做出明智的決定。

> [!NOTE]
> 您可以使用 Azure 入口網站或 PowerShell，檢視 Azure AD 應用程式或服務主體所公開的權限。 請嘗試使用以下指令碼檢視 Microsoft Graph 所公開的權限。
> ```powershell
> Connect-AzureAD
> 
> # Get OAuth2 Permissions/delegated permissions
> (Get-AzureADServicePrincipal -filter "DisplayName eq 'Microsoft Graph'").OAuth2Permissions
> 
> # Get App roles/application permissions
> (Get-AzureADServicePrincipal -filter "DisplayName eq 'Microsoft Graph'").AppRoles
> ```

| 屬性名稱 | 說明 | 範例 |
| --- | --- | --- |
| `ID` | 是可唯一識別此權限的 GUID 值。 | 570282fd-fa5c-430d-a7fd-fc8dc98a9dca |
| `IsEnabled` | 指出此權限是否可供使用。 | true |
| `Type` | 指出此權限需要使用者同意還是系統管理員同意。 | 使用者 |
| `AdminConsentDescription` | 是在系統管理員同意體驗期間，向系統管理員顯示的說明 | 允許應用程式讀取使用者信箱中的電子郵件。 |
| `AdminConsentDisplayName` | 是在系統管理員同意體驗期間，向系統管理員顯示的易記名稱。 | 讀取使用者的郵件 |
| `UserConsentDescription` | 是在使用者同意體驗期間，向使用者顯示的說明。 |  允許應用程式讀取您信箱中的電子郵件。 |
| `UserConsentDisplayName` | 是在使用者同意體驗期間，向使用者顯示的易記名稱。 | 讀取您的郵件 |
| `Value` | 是在 OAuth 2.0 授權流程中用來識別權限的字串。 `Value` 也可與應用程式識別碼 URI 字串結合，以形成完整的權限名稱。 | `Mail.Read` |

## <a name="types-of-consent"></a>同意的類型

Azure AD 中的應用程式需仰賴同意，才能取得所需資源或 API 的存取權。 應用程式可能需要知道數種同意才能成功。 如果您要定義權限，您還必須了解使用者會如何取得應用程式或 API 的存取權。

* **靜態使用者同意** - 當您指定應用程式要與之互動的資源時，會在 [OAuth 2.0 授權流程](v1-protocols-oauth-code.md#request-an-authorization-code)期間自動發生此同意。 在靜態使用者同意案例中，應用程式必須已經指定它在 Azure 入口網站中的應用程式設定所需的所有權限。 如果使用者 (或系統管理員，視情況而定) 尚未授與此應用程式的同意，Azure AD 會在此時提示使用者同意。 

    深入了解如何註冊會要求存取一組靜態 API 的 Azure AD 應用程式。
* **動態使用者同意** - 這是 v2 Azure AD 應用程式模型的功能。 在此案例中，應用程式會要求一組它在 [v2 應用程式的 OAuth 2.0 授權流程](/azure/active-directory/develop/active-directory-v2-scopes#requesting-individual-user-consent)中需要的權限。 如果使用者尚未同意，系統會在此時提示他同意。 [深入了解動態同意](/azure/active-directory/develop/active-directory-v2-compare#incremental-and-dynamic-consent)。

    > [!IMPORTANT]
    > 動態同意很方便，但因為系統管理員同意體驗在同意時不知道這些權限，對於需要系統管理員同意的權限來說會是重大挑戰。 如果您需要系統管理員特有的權限，或是您的應用程式使用動態同意，您就必須在 Azure 入口網站中註冊所有權限 (不只是需要系統管理員同意的權限子集)。 這可讓租用戶管理員代表其所有的使用者表示同意。
  
* **系統管理員同意** - 當應用程式需要特定高權限的存取權時，就需要此同意。 系統管理員同意可確保在系統管理員授權應用程式或使用者存取組織中的高權限資料之前，有某些額外的控制能力。 [深入了解如何授與系統管理員同意](/azure/active-directory/develop/active-directory-v2-scopes#using-the-admin-consent-endpoint)。

## <a name="best-practices"></a>最佳作法

### <a name="client-best-practices"></a>用戶端最佳作法

- 只要求應用程式所需的權限。 如果應用程式有太多權限，會有遭到入侵時暴露使用者資料的風險。
- 根據您的應用程式支援的案例，選擇委派的權限或應用程式權限。
    - 如果要代表使用者進行呼叫，則一律使用委派的權限。
    - 如果應用程式為非互動式，且未代表任何特定使用者進行呼叫，則只能使用應用程式權限。 應用程式權限具有高度權限，應該只在絕對必要時使用。
- 使用以 V2.0 端點為基礎的應用程式時，一律會將靜態權限 (在您的應用程式註冊中指定的權限) 設定為您在執行階段要求的動態權限超集 (在程式碼中指定並且在授權要求中當作查詢參數中傳送的權限)，以便系統管理員同意等案例運作正確。

### <a name="resourceapi-best-practices"></a>資源/API 最佳做法

- 會公開 API 的資源應定義權限，明確指定它們所保護的資料或動作。 遵循此最佳做法有助於確保用戶端最後不會得到可存取非其所需資料的權限，而使用者也會知道他們同意了哪些資料。
- 資源應該明確地分開定義 `Read` 和 `ReadWrite` 權限。
- 資源應該要將任何允許跨使用者界限存取資料的權限，標示為 `Admin` 權限。
- 資源應遵循命名模式 `Subject.Permission[.Modifier]`，其中：
    - `Subject` 對應於可用的資料類型
    - `Permission` 對應於使用者可對該資料採取的動作
    - `Modifier` 選擇性地用來說明另一個權限的特製化
    
    例如︰
    * Mail.Read - 可讓使用者讀取郵件。
    * Mail.ReadWrite - 可讓使用者閱讀或撰寫郵件。
    * Mail.ReadWrite.All - 可讓系統管理員或使用者存取組織中的所有郵件。
