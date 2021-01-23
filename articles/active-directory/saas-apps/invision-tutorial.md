---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 InVision 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 InVision 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/24/2020
ms.author: jeedes
ms.openlocfilehash: ec35917ca18064d58279d8ed2b3fb1f0e83a88fc
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98736929"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-invision"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 InVision 整合

在本教學課程中，您將了解如何整合 InVision 與 Azure Active Directory (Azure AD)。 在整合 InVision 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 InVision 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 InVision。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 InVision 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* InVision 支援由 **SP 和 IDP** 起始的 SSO

## <a name="adding-invision-from-the-gallery"></a>從資源庫新增 InVision

若要進行將 InVision 整合到 Azure AD 中的設定，您必須從資源庫將 InVision 新增至受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增]  區段的搜尋方塊中，輸入 **InVision**。
1. 從結果面板中選取 [InVision]  ，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-invision"></a>設定和測試 InVision 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 InVision 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 InVision 中相關使用者之間的連結關聯性。

若要設定及測試與 InVision 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 InVision SSO](#configure-invision-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 InVision 測試使用者](#create-invision-test-user)** - 使 InVision 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [InVision] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 設定]  區段上，如果您想要以 **IDP** 起始模式設定應用程式，請輸入下列欄位的值：

    a. 在 [識別碼]  文字方塊中，使用下列模式來輸入 URL：`https://<SUBDOMAIN>.invisionapp.com`

    b. 在 [回覆 URL]  文字方塊中，使用下列模式來輸入 URL：`https://<SUBDOMAIN>.invisionapp.com//sso/auth`

1. 如果您想要以 **SP** 起始模式設定應用程式，請按一下 [設定其他 URL]  ，然後執行下列步驟：

    在 [登入 URL]  文字方塊中，以下列模式輸入 URL︰`https://<SUBDOMAIN>.invisionapp.com`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的「識別碼」、「回覆 URL」及「登入 URL」來更新這些值。 請連絡 [InVision 用戶端支援小組](mailto:support@invisionapp.com)以取得這些值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

1. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，尋找 [憑證 (Base64)]  並選取 [下載]  ，以下載憑證並將其儲存在電腦上。

    ![憑證下載連結](common/certificatebase64.png)

1. 在 [設定 InVision]  區段上，根據您的需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)

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

在本節中，您會將 InVision 的存取權授與 B.Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [InVision]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。

1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-invision-sso"></a>設定 InVision SSO

1. 若要自動執行 InVision 內的設定，您必須按一下 [安裝擴充功能] 來安裝「我的應用程式安全登入瀏覽器擴充功能」。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 InVision] 便會將您導向至 InVision 應用程式。 請從該處提供用以登入 InVision 的管理員認證。 瀏覽器擴充功能會自動為您設定應用程式，並自動執行步驟 3 到 6。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 InVision，請在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 InVision 公司網站。

1. 按一下 [小組]  並選取 [設定]  。

    ![此螢幕擷取畫面顯示已選取 [設定] 的 [小組] 索引標籤。](./media/invision-tutorial/config1.png)

1. 向下捲動至 [單一登入]  ，然後按一下 [變更]  。

    ![此螢幕擷取畫面顯示 [單一登入] 的 [變更] 按鈕。](./media/invision-tutorial/config3.png)

1. 在 [單一登入]  頁面上，執行下列步驟：

    ![此螢幕擷取畫面顯示此步驟中用來輸入值的 [單一登入] 頁面。](./media/invision-tutorial/config4.png)

    a. 將 [<帳戶名稱> 的每個成員都需要 SSO]  變更為 [開啟]  。

    b. 在 [名稱]  文字方塊中輸入名稱，例如 `azureadsso`。

    c. 在 [登入 URL]  文字方塊中輸入 [登入 URL] 值。

    d. 在 [登出 URL]  文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL]  值。

    e. 在 [SAML 憑證]  文字方塊中，將下載的 **憑證 (Base64)** 以記事本開啟，並複製內容，然後將其貼至 [SAML 憑證] 文字方塊中。

    f. 在 [名稱識別碼格式]  文字方塊中，使用 `Unspecified` 作為 [名稱識別碼格式]  。

    g. 從 [雜湊演算法]  的下拉式清單中，選取 [SHA-256]  。

    h. 為 [SSO 按鈕標籤]  輸入適當的名稱。

    i. 將 [允許 Just-in-Time 佈建]  設為 [開啟]。

    j. 按一下 [更新]  。

### <a name="create-invision-test-user"></a>建立 InVision 測試使用者

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入 InVision 網站。

1. 按一下 [小組]  並選取 [人員]  。

    ![此螢幕擷取畫面顯示已選取 [人員] 的 [小組] 索引標籤。](./media/invision-tutorial/config2.png)

1. 按一下 [+ 圖示]  以新增使用者。

    ![此螢幕擷取畫面顯示用來新增使用者的 [+] 圖示。](./media/invision-tutorial/user1.png)

1. 輸入使用者的電子郵件地址，然後按 [下一步]  。

    ![此螢幕擷取畫面顯示可以在其中輸入地址的 [邀請至] 對話方塊。](./media/invision-tutorial/user2.png)

1. 確認電子郵件地址，然後按一下 [邀請]  。

    ![此螢幕擷取畫面顯示可以在其中選取 [邀請] 以繼續進行的 [邀請] 對話方塊。](./media/invision-tutorial/user3.png)

## <a name="test-sso"></a>測試 SSO

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

#### <a name="sp-initiated"></a>SP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至您可以在其中起始登入流程的 InVision 登入 URL。

* 直接移至 InVision 登入 URL，然後從該處起始登入流程。

#### <a name="idp-initiated"></a>IDP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入您已設定 SSO 的 InVision

您也可以使用 Microsoft「我的應用程式」，以任何模式測試應用程式。 當您按一下「我的應用程式」中的 [InVision] 圖格時，如果是在 SP 模式中設定，您會重新導向至 [應用程式登入] 頁面來起始登入流程，如果在 IDP 模式中設定，則應該會自動登入已設定 SSO 的 InVision。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。

## <a name="next-steps"></a>後續步驟

設定 InVision 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。