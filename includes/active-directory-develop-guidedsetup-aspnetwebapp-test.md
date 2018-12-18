---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/19/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: 3a141bcde75872f2384aedf982ffef5cba9666a3
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843031"
---
## <a name="test-your-code"></a>測試您的程式碼

若要在 Visual Studio 中測試您的應用程式，請按 **F5** 來執行專案。 瀏覽器會開啟至 http://<span></span>localhost:{port} 位置，且您會看到 [使用 Microsoft 帳戶登入] 按鈕。 選取按鈕以啟動登入程序。

當您準備好執行測試時，請使用 Microsoft Azure Active Directory (Azure AD) 帳戶 (公司或學校帳戶) 或個人 Microsoft 帳戶 (<span>live.</span>com 或 <span>outlook.</span>com) 帳戶登入。

![使用 Microsoft 登入](media/active-directory-develop-guidedsetup-aspnetwebapp-test/aspnetbrowsersignin.png)
<br/><br/>
![登入您的 Microsoft 帳戶](media/active-directory-develop-guidedsetup-aspnetwebapp-test/aspnetbrowsersignin2.png)

#### <a name="view-application-results"></a>檢視應用程式結果
登入之後，系統會將使用者重新導向至您網站的首頁。 首頁就是您在 Microsoft 應用程式註冊入口網站的應用程式註冊資訊中指定的 HTTPS URL。 首頁包含歡迎訊息「\<使用者>，您好」、登出連結，以及用來檢視使用者宣告的連結。 使用者宣告的連結會瀏覽至您稍早建立的「宣告」控制器。

### <a name="browse-to-see-the-users-claims"></a>瀏覽以查看使用者的宣告
若要查看使用者的宣告，請選取瀏覽至控制器檢視的連結，僅可供已驗證的使用者使用。

#### <a name="view-the-claims-results"></a>檢視宣告結果
瀏覽至控制器檢視之後，您應該就會看到一個資料表，包含使用者的基本屬性：

|屬性 |值 |說明 |
|---|---|---|
|**名稱** |使用者的全名 | 使用者的名字和姓氏。
|**使用者名稱** |user<span>@domain.com</span> | 用來識別使用者的使用者名稱。
|**主旨** |主體 |用來跨網站唯一識別使用者的字串。|
|**租用戶識別碼** |Guid | 唯一代表使用者 Azure AD 組織的 **guid**。|

此外，您應該會看到驗證要求中所有宣告的資料表。 如需詳細資訊，請參閱 [Azure AD ID 權杖中的宣告清單](https://docs.microsoft.com/azure/active-directory/develop/active-directory-token-and-claims)。


### <a name="test-access-to-a-method-that-has-an-authorize-attribute-optional"></a>對具有 Authorize 屬性 (選用) 的方法進行存取測試
若要利用匿名使用者身分測試以 `Authorize` 屬性保護之控制器的存取權限，請遵循以下步驟：
1. 選取連結以將使用者登出，並完成登出程序。
2. 在您的瀏覽器中輸入 http://<span></span>localhost:{port}/claims，存取以 `Authorize` 屬性保護的控制器。

#### <a name="expected-results-after-access-to-a-protected-controller"></a>存取受保護控制站之後的預期結果
系統會提示您使用受保護的控制器檢視進行驗證。

## <a name="advanced-options"></a>進階選項

<!--start-collapse-->
### <a name="protect-your-entire-website"></a>保護您的整個網站
若要保護整個網站，請在 **Global.asax** 檔案中，將 `AuthorizeAttribute` 屬性新增至 `Application_Start` 方法中的 `GlobalFilters` 篩選：

```csharp
GlobalFilters.Filters.Add(new AuthorizeAttribute());
```
<!--end-collapse-->

### <a name="restrict-who-can-sign-in-to-your-application"></a>限制誰可以登入您的應用程式
根據預設，當您建置依照本指南建立的應用程式時，將能夠接受使用個人帳戶 (包括 outlook.com、live.com 和其他帳戶)，以及來自已整合 Azure Active Directory 之公司或組織的公司與學校帳戶登入。 這是 SaaS 應用程式的建議選項。

若要限制使用者登入應用程式的存取權限，可使用多個選項：

#### <a name="option-1-restrict-users-from-only-one-organizations-active-directory-instance-to-sign-in-to-your-application-single-tenant"></a>選項 1：限制只來自一個組織之 Active Directory 執行個體的使用者，登入您的應用程式 (單一租用戶)

這個選項是「LOB 應用程式」的常見案例：如果您想要讓應用程式只接受特定 Azure Active Directory 執行個體的成員帳戶登入 (包括該執行個體的「來賓帳戶」)，請執行下列動作：

1. 在 **web.config** 檔案中，將 `Tenant` 參數的值從 `Common` 變更為組織的租用戶名稱 (如 `contoso.onmicrosoft.com`)。
2. 在您的 [OWIN Startup 類別](#configure-the-authentication-pipeline) 中，將 `ValidateIssuer` 引數設為 `true`。

#### <a name="option-2-restrict-access-to-your-application-to-users-in-a-specific-list-of-organizations"></a>選項 2：將應用程式的存取限制為特定組織清單內的使用者

您可以將登入存取限制為僅限允許組織清單內之 Azure AD 組織中的使用者帳戶：
1. 在您的 [OWIN Startup 類別](#configure-the-authentication-pipeline) 中，將 `ValidateIssuer` 引數設為 `true`。
2. 將 `ValidIssuers` 參數的值設為允許組織的清單。

#### <a name="option-3-use-a-custom-method-to-validate-issuers"></a>選項 3：使用自訂方法來驗證簽發者
您可以實作自訂方法，使用 **IssuerValidator** 參數來驗證簽發者。 如需此參數使用方式的詳細資訊，請閱讀 MSDN 上的 [TokenValidationParameters 類別](https://msdn.microsoft.com/library/system.identitymodel.tokens.tokenvalidationparameters.aspx)。

[!INCLUDE [Help and support](./active-directory-develop-help-support-include.md)]
