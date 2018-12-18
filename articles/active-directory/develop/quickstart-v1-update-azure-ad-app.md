---
title: 整合應用程式與 Azure Active Directory
description: 了解如何在 Azure Active Directory (Azure AD) 中更新應用程式。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.custom: aaddev
ms.reviewer: lenalepa, sureshja
ms.openlocfilehash: f311f951e09e064b8eac779b1082c666fe029479
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977231"
---
# <a name="quickstart-update-an-application-in-azure-active-directory"></a>快速入門：在 Azure Active Directory 中更新應用程式

已在 Azure Active Directory (Azure AD) 註冊應用程式的企業開發人員和軟體即服務 (SaaS) 提供者，可能需要設定您的應用程式來存取其他資源 (如 Web API)，以便讓這些資源供其他組織使用等等。

在本快速入門中，您將了解各種設定或更新應用程式以符合您自己或其他的組織要求或需求的方式。

## <a name="prerequisites"></a>必要條件

若要開始，請確定您已完成下列步驟：

* 閱讀 [Azure AD 同意架構](consent-framework.md)概觀，當建置其他使用者或應用程式需要使用的應用程式時，這些是務必了解的重要資訊。
* 有一個已在 Azure AD 註冊應用程式的租用戶。
  * 如果您還沒有租用戶， [了解如何取得租用戶](quickstart-create-new-tenant.md)。
  * 如果您沒有在您的租用戶中註冊應用程式，請[了解如何新增應用程式至 Azure AD](quickstart-v1-integrate-apps-with-azure-ad.md)。

## <a name="configure-a-client-application-to-access-web-apis"></a>設定用戶端應用程式以存取 Web API

為了讓 Web/機密用戶端應用程式能夠參與需要驗證的授權授與流程 (及取得存取權杖)，用戶端必須建立安全的認證。 Azure 入口網站支援的預設驗證方法為用戶端識別碼 + 祕密金鑰。 本節會討論使用用戶端的認證提供秘密金鑰所需的設定步驟。

同意架構要先確保用戶端取得必要的權限授與 (根據所要求的權限)，用戶端才可以存取資源應用程式所公開的 Web API (例如 Microsoft Graph API)。 根據預設，所有應用程式都可以從 **Azure Active Directory** (圖形 API) 和 Azure 傳統部署模型選擇權限。 依預設也會選取[圖形 API「登入及讀取使用者設定檔」權限](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes#PermissionScopeDetails)。 如果您的用戶端將會註冊在擁有已訂閱 Office 365 之帳戶的租用戶中，則也可選取 SharePoint 與 Exchange Online 的 Web API 和權限。 針對每一個所要的 Web API，您可以從兩種權限類型中做選擇：

- 應用程式權限：您的用戶端應用程式本身需要直接存取 Web API (沒有使用者內容)。 這種類型的權限需要系統管理員的同意，而且無法供原生用戶端應用程式使用。
- 委派權限：您的用戶端應用程式需要存取 Web API 作為已登入的使用者，但其存取權受到選取權限的限制。 這種類型的權限可由使用者授與，除非權限需要系統管理員的同意。

  > [!NOTE]
  > 新增對應用程式的委派權限並不會自動將同意授與租用戶內的使用者。 使用者仍必須手動同意在執行階段新增的委派權限，除非者系統管理員代表所有使用者授與同意。

### <a name="add-application-credentials-or-permissions-to-access-web-apis"></a>新增用來存取 Web API 的應用程式認證或權限

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 如果您的帳戶可讓您存取多個租用戶，請在右上角選取您的帳戶，並將您的入口網站工作階段設定為想要的 Azure AD 租用戶。
3. 在左側導覽窗格中，依序選取 [Azure Active Directory] 服務和 [應用程式註冊]，然後尋找/選取您要設定的應用程式。

   ![更新應用程式的註冊](./media/quickstart-v1-integrate-apps-with-azure-ad/update-app-registration.png)

4. 系統會將您帶往應用程式的主要註冊頁面，其中包含應用程式的 [設定] 頁面。 若要新增您 Web 應用程式的認證：
  1. 選取 [設定] 頁面上的 [金鑰] 區段。
  2. 若要新增憑證：
    - 選取 [上傳公開金鑰]。
    - 選取您想要上傳的檔案。 檔案必須是下列其中一種檔案類型：.cer、.pem、.crt。
  - 若要新增密碼：
    - 為金鑰新增描述。
    - 選取持續時間。
    - 選取 [ **儲存**]。 在您儲存組態變更之後，最右邊的資料行會包含金鑰值。 **請務必複製此金鑰**，以供在用戶端應用程式的程式碼中使用，因為您一旦離開此頁面就無法再存取此金鑰。

5. 若要新增權限以從用戶端存取資源 API
  1. 在 [設定] 頁面上，選取 [必要權限]，然後選取 [新增]。
  1. 選取 [選取 API] 以選取您想要挑選的資源類型。
  1. 瀏覽可用的 API 清單，或使用搜尋方塊在公開 Web API 的目錄中從可用的資源應用程式中選取。 選取您感興趣的資源，然後按一下 [選取]。
  1. 在 [啟用存取] 頁面中，選取您的應用程式在存取 API 時需要的應用程式權限及/或委派的權限。
   
  ![更新應用程式的註冊 - 權限 API](./media/quickstart-v1-integrate-apps-with-azure-ad/update-app-registration-settings-permissions-api.png)

  ![更新應用程式的註冊 - 權限 perms](./media/quickstart-v1-integrate-apps-with-azure-ad/update-app-registration-settings-permissions-perms.png)

6. 完成後，選取 [啟用存取] 頁面上的 [選取] 按鈕，然後選取 [新增 API 存取] 頁面上的 [完成] 按鈕。 您會回到 [所需的權限] 頁面，其中新的資源已新增到 API 清單。

## <a name="configuring-a-resource-application-to-expose-web-apis"></a>設定資源應用程式以公開 Web API

您可以開發 Web API，並藉由公開存取[範圍](developer-glossary.md#scopes)和[角色](developer-glossary.md#roles)，使其可供用戶端應用程式使用。 正確設定的 web API 即可供使用，就像其他 Microsoft web API 一樣，包括 Graph API 和 Office 365 API。 存取範圍和角色會透過[應用程式的資訊清單](developer-glossary.md#application-manifest) (此為代表應用程式身分識別組態的 JSON 檔案) 公開。

下一節會示範如何藉由修改資源應用程式資訊清單來公開存取範圍。

### <a name="add-access-scopes-to-your-resource-application"></a>新增資源應用程式的存取範圍

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 如果您的帳戶可讓您存取多個租用戶，請在右上角選取您的帳戶，並將您的入口網站工作階段設定為想要的 Azure AD 租用戶。
3. 在左側導覽窗格中，依序選取 [Azure Active Directory] > [應用程式註冊]，然後尋找/選取您要設定的應用程式。

   ![更新應用程式的註冊](./media/quickstart-v1-integrate-apps-with-azure-ad/update-app-registration.png)

4. 系統會將您帶往應用程式的主要註冊頁面，以開啟應用程式的 [設定] 頁面。 從應用程式的 [註冊] 頁面按一下 [資訊清單]，以切換至 [編輯資訊清單] 頁面。 Web 式的資訊清單編輯器隨即開啟，以供您在入口網站內**編輯**資訊清單。 或者，您也可以按一下 [下載] 並在本機編輯，然後使用 [上傳] 以將其重新套用到您的應用程式。
5. 在此範例中，我們會藉由將下列 JSON 元素新增至 `oauth2Permissions` 集合，從而在資源/API 上公開稱為 `Employees.Read.All` 的新範圍。 在註冊期間，預設會提供現有的 `user_impersonation` 範圍。 `user_impersonation` 可讓用戶端應用程式依據登入使用者的身分識別要求資源存取權限。 請務必在現有 `user_impersonation` 範圍元素後方加上逗號，並將屬性值變更為符合您的資源需求。 

  ```json
  {
    "adminConsentDescription": "Allow the application to have read-only access to all Employee data.",
    "adminConsentDisplayName": "Read-only access to Employee records",
    "id": "2b351394-d7a7-4a84-841e-08a6a17e4cb8",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allow the application to have read-only access to your Employee data.",
    "userConsentDisplayName": "Read-only access to your Employee records",
    "value": "Employees.Read.All"
  }
  ```

  > [!NOTE]
  > `id` 的值必須以程式設計的方式產生，或使用 GUID 產生工具 (例如 [guidgen](https://msdn.microsoft.com/library/ms241442%28v=vs.80%29.aspx)) 產生。 `id` 代表由 Web API 所公開之範圍的唯一識別碼。 一旦為用戶端設定了適當的 Web API 存取權限，Azure AD 便會對用戶端發出 OAuth2.0 存取權杖。 用戶端在呼叫 Web API 時會出示存取權杖，此權杖的範圍 (scp) 宣告已設定為其應用程式註冊中所要求的權限。
  >
  > 稍後您可以視需要公開其他範圍。 請考慮您的 Web API 可能會公開多個與各種不同功能相關聯的範圍。 在執行階段，您的資源可藉由評估所收到之 OAuth 2.0 存取權杖中的範圍 (`scp`) 宣告，來控制 Web API 的存取。

6. 完成時，按一下 [儲存]。 您的 Web API 現在已設定為可供目錄中的其他應用程式使用。

  ![更新應用程式的註冊](./media/quickstart-v1-integrate-apps-with-azure-ad/update-app-registration-manifest.png)

### <a name="verify-the-web-api-is-exposed-to-other-applications-in-your-tenant"></a>確認已向租用戶中的其他應用程式公開 Web API

1. 返回您的 Azure AD 租用戶，再次選取 [應用程式註冊]，然後尋找/選取您想要設定的用戶端應用程式。

   ![更新應用程式的註冊](./media/quickstart-v1-integrate-apps-with-azure-ad/update-app-registration.png)

2. 重複您在[設定用戶端應用程式以存取 Web API](#configure-a-client-application-to-access-web-apis) 所進行過的步驟 5。 當您進入 [選取 API] 步驟，請在 [搜尋] 欄位中輸入資源的應用程式名稱來搜尋資源，然後按一下 [選取]。

3. 在 [啟用存取] 頁面上，您應該會看到可供用戶端權限要求使用的新範圍。

  ![新的權限顯示出來](./media/quickstart-v1-integrate-apps-with-azure-ad/update-app-registration-settings-permissions-perms-newscopes.png)

### <a name="more-on-the-application-manifest"></a>應用程式資訊清單的詳細資料

應用程式資訊清單可作為更新應用程式實體的一種機制，其可定義 Azure AD 應用程式身分識別設定的所有屬性，包括我們所討論的 API 存取範圍。 如需應用程式實體和其結構描述的詳細資訊，請參閱[圖形 API 應用程式實體文件](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity)。 該文章包含可用來指定您 API 權限之應用程式實體成員的完整參考資訊，包括：  

- appRoles 成員，此成員為 [AppRole](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type) 實體的集合，可用來定義 Web API 的[應用程式權限](developer-glossary.md#permissions)。 
- oauth2Permissions 成員，此成員為 [OAuth2Permission](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type) 實體的集合，可用來定義 Web API 的[委派的權限](developer-glossary.md#permissions)。

如需一般的應用程式資訊清單概念的詳細資訊，請參閱 [Azure AD 應用程式資訊清單](reference-app-manifest.md)。

## <a name="accessing-the-azure-ad-graph-and-office-365-via-microsoft-graph-apis"></a>透過 Microsoft Graph API 存取 Azure AD Graph 和 Office 365 API  

如前文所述，除了在自己的應用程式上公開/存取 API，您也可以註冊用戶端應用程式以存取 Microsoft 資源所公開的 API。 Microsoft Graph API (在入口網站的資源/API 清單中稱為「Microsoft Graph」) 適用於所有已向 Azure AD 註冊的應用程式。 如果您在包含已註冊 Office 365 訂用帳戶之帳戶的租用戶中註冊用戶端應用程式，您也可以存取各種 Office 365 資源所公開的範圍。

如需 Microsoft Graph API 所公開之範圍的完整討論，請參閱 [Microsoft Graph 權限參考](https://developer.microsoft.com/en-us/graph/docs/concepts/permissions_reference)一文。

> [!NOTE]
> 由於目前的限制，如果原生用戶端應用程式使用「存取組織的目錄」權限，它們只能呼叫 Azure AD 圖形 API。 這項限制不適用於 web 應用程式。

## <a name="configuring-multi-tenant-applications"></a>設定多租用戶應用程式

在 Azure AD 註冊應用程式時，您可以只讓組織中的使用者存取您的應用程式。 或者，您也可能想讓外部組織的使用者可以存取您的應用程式。 這兩個應用程式類型稱為單一租用戶和多租用戶應用程式。 本節會討論如何修改單一租用戶應用程式的組態，使其成為多組用戶應用程式。

請務必注意單一租用戶與多租用戶應用程式之間的差異：  

- 單一租用戶應用程式適合在一個組織中使用。 它通常是由企業開發人員撰寫的企業營運 (LoB) 應用程式。 單一租用戶應用程式只能由所擁有之帳戶位於和應用程式註冊相同之租用戶中的使用者來存取。 因此，它只需要佈建到一個目錄中。
- 多租用戶應用程式適合在許多組織中使用。 這種應用程式稱為軟體即服務 (SaaS) Web 應用程式，通常是由獨立軟體廠商 (ISV) 所撰寫出來的。 多租用戶應用程式必須佈建在使用者需要存取的每個租用戶中。 至於應用程式註冊所在之租用戶以外的租用戶，則需要使用者或系統管理員同意才能註冊這些租用戶。 請注意，原生用戶端應用程式預設有多個租用戶，因為其安裝於資源擁有者的裝置上。 如需同意架構的詳細資訊，請參閱前面的[同意架構的概觀](#overview-of-the-consent-framework)一節。

讓應用程式具有多租用戶性質需要同時變更應用程式註冊以及 Web 應用程式本身。 下列幾節會說明這兩項。

### <a name="changing-the-application-registration-to-support-multi-tenant"></a>變更應用程式註冊以支援多租用戶

如果您正在撰寫想要提供給組織外的客戶或合作夥伴使用的應用程式，您必須在 Azure 入口網站中更新應用程式定義。

> [!IMPORTANT]
> Azure AD 會要求多租用戶應用程式的應用程式識別碼 URI 必須是全域唯一的。 「應用程式識別碼 URI」是其中一種可在通訊協定訊息中識別應用程式的方式。 在單一租用戶應用程式中，只要該租用戶內有唯一的應用程式識別碼 URI 就已足夠。 就多租用戶應用程式而言，該 URI 則必須具全域唯一性，Azure AD 才能在所有租用戶中找到該應用程式。
> 系統會透過要求「應用程式識別碼 URI」必須具有與已驗證的 Azure AD 租用戶網域相符的主機名稱，來強制執行全域唯一性。 例如，如果租用戶的名稱是 contoso.onmicrosoft.com，則有效的應用程式識別碼 URI 會是 https://contoso.onmicrosoft.com/myapp。 如果租用戶的已驗證網域是 contoso.com，則有效的應用程式識別碼 URI 也會是 https://contoso.com/myapp。 如果「應用程式識別碼 URI」沒有按照這個模式，將應用程式設定成多租用戶時就會失敗。

若要讓外部使用者能夠存取您的應用程式：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 如果您的帳戶可讓您存取多個帳戶，請在右上角按一下您的帳戶，並將您的入口網站工作階段設定為想要的 Azure AD 租用戶。
3. 在左側導覽窗格中，依序按一下 [Azure Active Directory] 服務和 [應用程式註冊]，然後尋找/按一下您要設定的應用程式。 系統會將您帶往應用程式的主要註冊頁面，以開啟應用程式的 [設定] 頁面。
4. 在 [設定] 頁面中，按一下 [屬性]，並將 [多租用戶] 變更為 [是]。

進行變更之後，其他組織中的使用者和系統管理員就能夠授與其使用者登入您應用程式的能力，讓您的應用程式能夠存取這些組織的租用戶所保護的資源。

### <a name="changing-the-application-to-support-multi-tenant"></a>變更應用程式以支援多租用戶

多租用戶應用程式支援極度仰賴 Azure AD 同意架構。 同意機制可讓其他租用戶的使用者，對應用程式授與使用者的租用戶所保護之資源的存取權。 這項體驗稱為「使用者同意」。

您的 Web 應用程式可能還會提供：

- 讓系統管理員「註冊我的公司」的能力。 此體驗稱為「系統管理員同意」，可讓系統管理員有能力代替其組織中的*所有使用者*授與同意。 只有使用屬於全域管理員角色的帳戶進行驗證的使用者可以提供系統管理員同意；其他人會收到錯誤。
- 使用者的註冊體驗。 它應該會對使用者提供 [註冊] 按鈕，以將瀏覽器重新導向至 Azure AD OAuth2.0 `/authorize` 端點或 OpenID Connect `/userinfo` 端點。 這些端點可讓應用程式藉由檢查 id_token 取得新使用者的相關資訊。 在註冊階段後，使用者將會看到類似[同意架構的概觀](#overview-of-the-consent-framework)一節所示的同意提示。

如需有關要支援多租用戶存取和登入/註冊體驗所需進行之應用程式變更的詳細資訊，請參閱：

- [如何使用多租用戶應用程式模式登入任何 Azure Active Directory (AD) 使用者](howto-convert-app-to-be-multi-tenant.md)
- [多租用戶程式碼範例](https://azure.microsoft.com/documentation/samples/?service=active-directory&term=multi-tenant)的清單。
- [快速入門：在 Azure AD 中將公司商標新增至登入頁面](../fundamentals/customize-branding.md)

## <a name="enabling-oauth-20-implicit-grant-for-single-page-applications"></a>啟用單頁應用程式的 OAuth 2.0 隱含授權

單頁應用程式 (SPA) 的結構，通常會設計成具有在瀏覽器中執行且使用大量 JavaScript 的前端，此前端會呼叫應用程式的 Web API 後端以執行其商務邏輯。 針對裝載於 Azure AD 中的 SPA，您會使用 OAuth 2.0 隱含授權驗證具備 Azure AD 的使用者，並取得您可以使用的權杖以保護從應用程式的 JavaScript 用戶端到其後端 web API 的呼叫。

使用者授與同意權之後，這個相同的驗證通訊協定可用來取得權杖以保護用戶端和其他為應用程式設定之 web API 資源之間的呼叫。 若要深入了解隱含授權授與，並協助您決定其是否適合您的應用程式案例，請參閱 [了解 Azure Active Directory 中的 OAuth2 隱含授與流程](v1-oauth2-implicit-grant-flow.md)。

根據預設，應用程式的 OAuth 2.0 隱含授權會停用。 您可以在應用程式的[應用程式資訊清單](reference-app-manifest.md)中設定 `oauth2AllowImplicitFlow` 值，為應用程式啟用 OAuth 2.0 隱含授與。

### <a name="to-enable-oauth-20-implicit-grant"></a>啟用 OAuth 2.0 隱含授權

> [!NOTE]
> 如需如何編輯應用程式資訊清單的詳細資訊，請務必先檢閱上一節[設定資源應用程式以公開 Web API](#configuring-a-resource-application-to-expose-web-apis)。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 如果您的帳戶可讓您存取多個帳戶，請在右上角按一下您的帳戶，並將您的入口網站工作階段設定為想要的 Azure AD 租用戶。
3. 在左側導覽窗格中，依序按一下 [Azure Active Directory] 服務和 [應用程式註冊]，然後尋找/按一下您要設定的應用程式。 系統會將您帶往應用程式的主要註冊頁面，以開啟應用程式的 [設定] 頁面。
4. 從應用程式的 [註冊] 頁面按一下 [資訊清單]，以切換至 [編輯資訊清單] 頁面。 Web 式的資訊清單編輯器隨即開啟，以供您在入口網站內**編輯**資訊清單。 找到 "oauth2AllowImplicitFlow" 並將其值設為 "true"。 預設會設定為 "false"。
   
  ```json
  "oauth2AllowImplicitFlow": true,
  ```
5. 儲存更新後的資訊清單。 儲存之後，您的 Web API 現在已設定為使用 OAuth 2.0 隱含授權來驗證使用者。

## <a name="next-steps"></a>後續步驟

針對使用 Azure AD v1.0 端點的應用程式，深入了解這些其他相關的應用程式管理快速入門：
- [將應用程式新增到 Azure AD](quickstart-v1-integrate-apps-with-azure-ad.md)
- [從 Azure AD 移除應用程式](quickstart-v1-remove-azure-ad-app.md)

若要了解代表已註冊的應用程式和它們之間關係的兩個 Azure AD 物件，請參閱[應用程式物件和服務主體物件](app-objects-and-service-principals.md)。

若要了解使用 Azure Active Directory 開發應用程式時應使用的商標指導方針，請參閱[應用程式的商標指導方針](howto-add-branding-in-azure-ad-apps.md)。