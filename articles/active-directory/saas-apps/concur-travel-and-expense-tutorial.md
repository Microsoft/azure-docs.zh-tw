---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 Concur Travel and Expense 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Concur Travel and Expense 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/21/2020
ms.author: jeedes
ms.openlocfilehash: 56eab06bd1afd18bfd4e23b9acb0b44d7bd1f80b
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98737027"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-concur-travel-and-expense"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 Concur Travel and Expense 整合

在此教學課程中，您將了解如何整合 Concur Travel and Expense 與 Azure Active Directory (Azure AD)。 在整合 Concur Travel and Expense 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 Concur Travel and Expense 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Concur Travel and Expense。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* Concur Travel and Expense 訂用帳戶。
* Concur 使用者帳戶下的「公司系統管理員」角色。 您可以前往 [Concur SSO 自助服務工具](https://www.concursolutions.com/nui/authadmin/ssoadmin)來測試您是否擁有正確的存取權。 如果您沒有存取權，請連絡 Concur 的支援或實作專案經理。 

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會設定和測試 Azure AD SSO。

* Concur Travel and Expense 支援 **IDP** 和 **SP** 起始的 SSO
* Concur Travel and Expense 可同時支援在生產和實作環境中測試 SSO 

> [!NOTE]
> 在下列三個區域中，此應用程式的識別碼是都是固定字串值：美國、EMEA (歐洲、中東與非洲) 和中國。 因此，在一個租用戶中，只能為每個區域設定一個執行個體。 

## <a name="adding-concur-travel-and-expense-from-the-gallery"></a>從資源庫新增 Concur Travel and Expense

若要設定將 Concur Travel and Expense 整合到 Azure AD 中，您需要從資源庫將 Concur Travel and Expense 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增] 區段的搜尋方塊中輸入 [Concur Travel and Expense]。
1. 從結果面板選取 [Concur Travel and Expense]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-concur-travel-and-expense"></a>設定及測試 Concur Travel and Expense 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 Concur Travel and Expense 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Concur Travel and Expense 中相關使用者之間的連結關聯性。

若要設定及測試與 Concur Travel and Expense 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Concur Travel and Expense SSO](#configure-concur-travel-and-expense-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 Concur Travel and Expense 測試使用者](#create-concur-travel-and-expense-test-user)** - 使 Concur Travel and Expense 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [Concur Travel and Expense] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，已預先以 **IDP** 起始的模式設定好應用程式，並已經為 Azure 預先填入必要的 URL。 使用者必須按一下 [儲存]  按鈕，才能儲存設定。

    > [!NOTE]
    > [識別碼 (實體識別碼)] 和 [回復 URL] (判斷提示取用者服務 URL) 均為區域特有。 請根據 Concur 實體的資料中心來選取。 如果您不知道 Concur 實體的資料中心，請連絡 Concur 支援服務。 

5. 在 [以 SAML 設定單一登入] 頁面上，按一下 [使用者屬性] 的編輯/畫筆圖示，以編輯設定。 [唯一的使用者識別碼] 必須符合 Concur 的使用者 login_id。 通常來說，您應該將 **user.userprincipalname** 變更為 **user.mail**。

    ![編輯使用者屬性](common/edit-attribute.png)

6. 在 [以 SAML 設定單一登入] 頁面上的 [SAML 簽署憑證] 區段中，尋找 [同盟中繼資料 XML]，然後選取 [下載] 以下載中繼資料，並將其儲存在電腦上。

    ![憑證下載連結](common/metadataxml.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下頁面底部的 [新增]  。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Concur Travel and Expense 的存取權授與 B.Simon，讓她能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Concur Travel and Expense]。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。

1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-concur-travel-and-expense-sso"></a>設定 Concur Travel and Expense SSO

1. 若要自動執行 Concur Travel and Expense 內的設定，您必須按一下 [安裝擴充功能] 來安裝「我的應用程式安全登入瀏覽器擴充功能」。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 Concur Travel and Expense] 便會將您導向至 Concur Travel and Expense 應用程式。 請從該處提供用以登入 Concur Travel and Expense 的管理員認證。 瀏覽器延伸模組將會自動為您設定應用程式，並自動執行步驟 3 到 7。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 Concur Travel and Expense，您必須在不同的網頁瀏覽器視窗中，將下載的 [同盟中繼資料 XML] 上傳至 [Concur SSO 自助式工具](https://www.concursolutions.com/nui/authadmin/ssoadmin)，然後以系統管理員身分登入您的 Concur Travel and Expense 公司網站。

1. 按一下 [新增] 。
1. 輸入 IdP 的自訂名稱，例如 "Azure AD (US)"。 
1. 按一下 [上傳 XML 檔案]，然後附上您先前下載的 **同盟中繼資料 XML**。
1. 按一下 [新增中繼資料] 以儲存變更。

    ![Concur SSO 自助服務工具的螢幕擷取畫面](./media/concur-travel-and-expense-tutorial/add-metadata-concur-self-service-tool.png)

### <a name="create-concur-travel-and-expense-test-user"></a>建立 Concur Travel and Expense 測試使用者

在本節中，您要在 Concur Travel and Expense 中建立名為 B.Simon 的使用者。 請與 Concur 支援小組合作，在 Concur Travel and Expense 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。 

> [!NOTE]
> B.Simon 的 Concur 登入識別碼必須符合 B.Simon 在 Azure AD 的唯一識別碼。 例如，如果 B.Simon 的 Azure AD 唯一識別碼為 `B.Simon@contoso.com`。 B.Simon 的 Concur 登入識別碼也必須為 `B.Simon@contoso.com`。 

## <a name="configure-concur-mobile-sso"></a>設定 Concur Mobile SSO
若要啟用 Concur Mobile SSO，您必須對 Concur 支援小組提供 **使用者存取 URL**。 請遵循下列步驟從 Azure AD 取得 **使用者存取 URL**：
1. 移至 [企業應用程式]
1. 按一下 [Concur Travel and Expense]
1. 按一下 [屬性]
1. 複製 **使用者存取 URL** 並將此 URL 提供給 Concur 支援人員

> [!NOTE]
> 您無法使用自助服務選項來設定 SSO，因此請與 Concur 支援小組合作以啟用 Mobile SSO。 

## <a name="test-sso"></a>測試 SSO 

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

#### <a name="sp-initiated"></a>SP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至您可以在其中起始登入流程的 Concur Travel and Expense 登入 URL。

* 直接移至 Concur Travel and Expense 登入 URL，然後從該處起始登入流程。

#### <a name="idp-initiated"></a>IDP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入您已設定 SSO 的 Concur Travel and Expense

您也可以使用 Microsoft「我的應用程式」，以任何模式測試應用程式。 當您按一下「我的應用程式」中的 [Concur Travel and Expense] 圖格時，如果是在 SP 模式中設定，您會重新導向至 [應用程式登入] 頁面來起始登入流程，如果在 IDP 模式中設定，則應該會自動登入已設定 SSO 的 Concur Travel and Expense。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。

## <a name="next-steps"></a>後續步驟

設定 Concur Travel and Expense 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。