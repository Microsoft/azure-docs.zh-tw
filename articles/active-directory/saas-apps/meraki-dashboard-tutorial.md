---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 Meraki Dashboard 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Meraki Dashboard 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/28/2020
ms.author: jeedes
ms.openlocfilehash: f635a4c4c6e0b1dcb4d4842d3cddb337d2b26407
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98735149"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-meraki-dashboard"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 Meraki Dashboard 整合

在本教學課程中，您會了解如何整合 Meraki Dashboard 與 Azure Active Directory (Azure AD)。 在整合 Meraki Dashboard 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 Meraki Dashboard 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Meraki Dashboard。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Meraki Dashboard 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* Meraki Dashboard 支援由 **IDP** 起始的 SSO

> [!NOTE]
> 此應用程式的識別碼是固定的字串值，因此一個租用戶中只能設定一個執行個體。

## <a name="adding-meraki-dashboard-from-the-gallery"></a>從資源庫新增 Meraki Dashboard

若要進行將 Meraki Dashboard 整合到 Azure AD 的設定，您必須從資源庫將 Meraki Dashboard 新增至受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增]  區段的搜尋方塊中輸入 **Meraki Dashboard**。
1. 從結果面板選取 [Meraki Dashboard]  ，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-meraki-dashboard"></a>設定及測試 Meraki Dashboard 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 Meraki Dashboard 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Meraki Dashboard 中相關使用者之間的連結關聯性。

若要設定及測試與 Meraki 儀表板搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Meraki Dashboard SSO](#configure-meraki-dashboard-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 Meraki Dashboard 測試使用者](#create-meraki-dashboard-test-user)** - 使 Meraki Dashboard 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 **Meraki 儀表板** 應用程式整合頁面上，尋找 **管理** 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，執行下列步驟：
     
    在 [回覆 URL]  文字方塊中，以下列模式輸入 URL：`https://n27.meraki.com/saml/login/m9ZEgb/< UNIQUE ID >`

    > [!NOTE]
    > [回覆 URL] 不是真實的值。 使用實際的回覆 URL 值來更新值，稍後會在本教學課程中說明。

1. 按一下 [儲存]  按鈕。

1. Meraki Dashboard 應用程式需要特定格式的 SAML 判斷提示，而您需要將自訂屬性對應新增到 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/default-attributes.png)

1. 除了上述屬性外，Meraki Dashboard 應用程式還需要在 SAML 回應中多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。
    
    | 名稱 | 來源屬性|
    | ---------------| --------- |
    | `https://dashboard.meraki.com/saml/attributes/username` | user.userprincipalname |
    | `https://dashboard.meraki.com/saml/attributes/role` | user.assignedroles |

    > [!NOTE]
    > 若要了解如何在 Azure AD 中設定角色，請參閱[此文章](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui--preview)。

1. 在 [SAML 簽署憑證]  區段中，按一下 [編輯]  按鈕以開啟 [SAML 簽署憑證]  對話方塊。

    ![編輯 SAML 簽署憑證](common/edit-certificate.png)

1. 在 [SAML 簽署憑證]  區段中，複製 [指紋值]  並儲存在您的電腦上。

    ![複製指紋值](common/copy-thumbprint.png)

1. 在 [設定 Meraki Dashboard]  區段上，複製 [登出 URL] 值並將其儲存在您的電腦上。

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

在本節中，您會將 Meraki Dashboard 的存取權授與 B.Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Meraki Dashboard]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。

    ![使用者角色](./media/meraki-dashboard-tutorial/user-role.png)

    > [!NOTE]
    > **選取角色** 選項將會停用，而預設角色是所選使用者的 USER。

1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-meraki-dashboard-sso"></a>設定 Meraki Dashboard SSO

1. 若要自動執行 Meraki Dashboard 內的設定，您必須按一下 [安裝擴充功能] 來安裝「我的應用程式安全登入瀏覽器擴充功能」。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 Meraki Dashboard] 便會將您導向至 Meraki Dashboard 應用程式。 請從該處提供用以登入 Meraki Dashboard 的管理員認證。 瀏覽器延伸模組將會自動為您設定應用程式，並自動執行步驟 3 到 7。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 Meraki Dashboard，請在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Meraki Dashboard 公司網站。

1. 瀏覽至 [組織]   -> [設定]  。

    ![Meraki Dashboard 設定索引標籤](./media/meraki-dashboard-tutorial/configure-1.png)

1. 在 [驗證] 之下，將 [SAML SSO]  變更為 [已啟用 SAML SSO]  。

    ![Meraki Dashboard 驗證](./media/meraki-dashboard-tutorial/configure-2.png)

1. 按一下 [新增 SAML IdP]  。

    ![Meraki Dashboard 新增 SAML IdP](./media/meraki-dashboard-tutorial/configure-3.png)

1. 將您從 Azure 入口網站複製的 [指紋]  值，貼到 [X.590 憑證 SHA1 指紋]  文字方塊中。 然後按一下 [儲存]  。 儲存之後，取用者 URL 就會顯示。 複製 [取用者 URL] 值，並將其貼到 Azure 入口網站中 [基本 SAML 設定]  區段中的 [回覆 URL]  文字方塊。

    ![Meraki Dashboard 設定](./media/meraki-dashboard-tutorial/configure-4.png)

### <a name="create-meraki-dashboard-test-user"></a>建立 Meraki Dashboard 測試使用者

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入 Meraki Dashboard 網站。

1. 瀏覽至 [組織]   -> [系統管理員]  。

    ![Meraki Dashboard 系統管理員](./media/meraki-dashboard-tutorial/user-1.png)

1. 在 [SAML 管理員角色] 區段中，按一下 [新增 SAML 角色]  按鈕。

    ![Meraki Dashboard 新增 SAML 角色按鈕](./media/meraki-dashboard-tutorial/user-2.png)

1. 輸入角色 **meraki_full_admin**，將 [組織存取]  標示為 [完整]  ，然後按一下 [建立角色]  。 針對 **meraki_readonly_admin** 重複此程序，這次會將 [組織存取]  標示為 [唯讀]  方塊。
 
    ![Meraki Dashboard 建立使用者](./media/meraki-dashboard-tutorial/user-3.png)

## <a name="test-sso"></a>測試 SSO 

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入已設定 SSO 的 Meraki 儀表板

* 您可以使用 Microsoft 的「我的應用程式」。 在我的應用程式中按一下 Meraki Dashboard 圖格時，應該會自動登入您設定 SSO 的 Meraki 儀表板。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。


## <a name="next-steps"></a>後續步驟

設定 Meraki Dashboard 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。
