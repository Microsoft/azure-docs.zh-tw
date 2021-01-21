---
title: 教學課程：Azure Active Directory 與 Displayr 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Displayr 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: jeedes
ms.openlocfilehash: 3cb6ee3162c70d2d07c4868ae90ecc54bd489966
ms.sourcegitcommit: a0c1d0d0906585f5fdb2aaabe6f202acf2e22cfc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98622486"
---
# <a name="tutorial-integrate-displayr-with-azure-active-directory"></a>教學課程：整合 Displayr 與 Azure Active Directory

在本教學課程中，您會了解如何整合 Displayr 與 Azure Active Directory (Azure AD)。 在整合 Displayr 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 Displayr 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Displayr。
* 在 Azure 入口網站集中管理您的帳戶。

若要深入了解 SaaS 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>Prerequisites

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Displayr 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。 Displayr 支援 **SP** 起始的 SSO。

## <a name="adding-displayr-from-the-gallery"></a>從資源庫新增 Displayr

若要設定將 Displayr 整合到 Azure AD 中，您需要從資源庫將 Displayr 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 [Azure 入口網站](https://portal.azure.com)。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory]  服務。
1. 巡覽至 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增]  區段的搜尋方塊中輸入 **Displayr**。
1. 從結果面板選取 [Displayr]  ，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

以名為 **Britta Simon** 的測試使用者，設定及測試與 Displayr 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Displayr 中相關使用者之間的連結關聯性。

若要設定及測試與 Displayr 搭配運作的 Azure AD SSO，請完成下列建置組塊：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** ，讓您的使用者能夠使用此功能。
2. **[設定 Displayr](#configure-displayr)** 以在應用程式端設定 SSO 設定。
3. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** ，以使用 Britta Simon 測試 Azure AD 單一登入。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** ，讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[建立 Displayr 測試使用者](#create-displayr-test-user)** ，使 Displayr 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
6. **[測試 SSO](#test-sso)** ，以驗證組態是否能運作。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 [Azure 入口網站](https://portal.azure.com/)的 [Displayr]  應用程式整合頁面上，尋找 [管理]  區段並選取 [單一登入]  。
1. 在 [選取單一登入方法]  頁面上，選取 [SAML]  。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，執行下列步驟：

    a. 在 [登入 URL]  文字方塊中，以下列模式輸入 URL︰`https://<YOURDOMAIN>.displayr.com`

    b. 在 [識別碼 (實體識別碼)]  文字方塊中，使用下列模式輸入 URL：`<YOURDOMAIN>.displayr.com`
    
    c. 在 [ **回復 URL** ] 文字方塊中，輸入 `https://app.displayr.com/Login/ProcessSamlResponse` 。
    
    d. 按一下 [檔案]  。

    >[!NOTE]
    >這些都不是真正的值。 請使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Displayr 用戶端支援小組](mailto:support@displayr.com)以取得這些值。 您也可以參考 Azure 入口網站中 [基本 SAML 設定] 區段所示的模式。

1. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，尋找 [憑證 (Base64)]  並選取 [下載]  以下載憑證並將其儲存在電腦上。

   ![憑證下載連結](common/certificatebase64.png)

1. Displayr 應用程式需要特定格式的 SAML 判斷提示，需要您加入自訂屬性對應到您的 SAML token 屬性設定。 以下螢幕擷取畫面顯示預設屬性清單。 按一下 [編輯]  圖示以開啟 [使用者屬性] 對話方塊。

   ![顯示 [使用者屬性] 區段的螢幕擷取畫面，其中醒目提示 [編輯] 圖示。](common/edit-attribute.png)

1. 除了以上屬性外，Displayr 應用程式還需要在 SAML 回應中傳回更多屬性。 在 [群組宣告 (預覽)]  對話方塊的 [使用者屬性與宣告]  區段中，執行下列步驟：

   a. 按一下 [ **新增群組** 宣告]。

      ![螢幕擷取畫面，顯示已選取設定的 [群組宣告 (預覽)] 視窗。](./media/displayr-tutorial/config05.png)

   b. 選取選項按鈕清單中的 [所有群組]  。

   c. 選取 [群組識別碼]  的 [來源屬性]  。

   d. 勾選 [自訂群組宣告的名稱]  。

   e. 勾選 [以角色宣告名義發出群組]  。

   f. 按一下 [檔案]  。

1. 在 [設定 Displayr]  區段上，依據您的需求複製適當的 URL。

   ![複製組態 URL](common/copy-configuration-urls.png)

### <a name="configure-displayr"></a>設定 Displayr

1. 若要自動執行 Displayr 內的設定，您必須按一下 [安裝延伸模組]  來安裝「我的應用程式安全登入瀏覽器延伸模組」  。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 Displayr]  便會將您導向到 Displayr 應用程式。 請從該處提供用以登入 Displayr 的管理員認證。 瀏覽器擴充功能會自動為您設定應用程式，並自動執行步驟 3 到 6。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 Displayr，請開啟新的網頁瀏覽器視窗，並以系統管理員身分登入 Displayr 公司網站，然後執行下列步驟：

4. 按一下 [設定]  ，然後瀏覽至 [帳戶]  。

    ![顯示已選取 [設定] 圖示和 [帳戶] 的螢幕擷取畫面。](./media/displayr-tutorial/config01.png)

5. 從頂端功能表切換至 [設定]  ，然後向下捲動頁面，再按一下 [設定單一登入 (SAML)]  。

    ![顯示已選取 [設定] 索引標籤和已選取 [設定單一登入 (SAML)] 動作的螢幕擷取畫面。](./media/displayr-tutorial/config02.png)

6. 在 [單一登入 (SAML)]  頁面上，執行下列步驟：

    ![組態](./media/displayr-tutorial/config03.png)

    a. 勾選 [啟用單一登入 (SAML)]  方塊。

    b. 從 Azure AD 的 [基本 SAML 設定]  區段複製實際的 [識別碼]  值，然後將其貼至 [簽發者]  文字方塊。

    c. 在 [登入 URL]  文字方塊中，貼上您從 Azure 入口網站複製的 [登入 URL]  值。

    d. 在 [登出 URL]  文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL]  值。

    e. 在記事本中開啟「憑證 (Base64)」，複製其內容，然後貼到 [憑證]  文字方塊中。

    f. [群組對應]  為選擇性。

    g. 按一下 [檔案] 。  

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您會在 Azure 入口網站中建立名稱為 Britta Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `Britta Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `BrittaSimon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下頁面底部的 [新增]  。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Displayr 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Displayr]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。

   ![[使用者和群組] 連結](common/users-groups-blade.png)

1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。

    ![[新增使用者] 連結](common/add-assign-user.png)

1. 在 [使用者和群組]  對話方塊中，從使用者清單選取 **Britta Simon**，然後按一下畫面底部的 [選取]  按鈕。
1. 如果您在 SAML 判斷提示中需要任何角色值，請在 [選取角色] 對話方塊的清單中為使用者選取適當的角色，然後按一下畫面底部的 [選取] 按鈕。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

### <a name="create-displayr-test-user"></a>建立 Displayr 測試使用者

若要讓 Azure AD 使用者能夠登入 Displayr，必須將這些使用者佈建到 Displayr。 在 Displayr 中，需手動進行佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入 Displayr。

2. 按一下 [設定]  ，然後瀏覽至 [帳戶]  。

    ![顯示已選取 [設定 (cog)] 圖示和 [帳戶] 的螢幕擷取畫面。](./media/displayr-tutorial/config01.png)

3. 從頂端功能表切換至 [設定]  ，然後向下捲動頁面至 [使用者]  區段，再按一下 [新增使用者]  。

    ![顯示 [設定] 索引標籤的螢幕擷取畫面，其中醒目提示 [使用者] 並且已選取 [新增使用者] 按鈕。](./media/displayr-tutorial/config07.png)

4. 在 [新增使用者]  頁面上，執行下列步驟：

    ![Displayr 設定](./media/displayr-tutorial/config06.png)

    a. 在 [名稱]  文字方塊中，輸入使用者的姓名，例如 **Brittasimon**。

    b. 在 [電子郵件]  文字方塊中，輸入使用者的電子郵件，例如 `Brittasimon@contoso.com`。

    c. 選取適當的 [群組成員資格]  。

    d. 按一下 [檔案]  。

### <a name="test-sso"></a>測試 SSO

當您在存取面板中選取 [Displayr] 圖格時，應該會自動登入您設定 SSO 的 Displayr。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/my-apps-portal-end-user-access.md)。

## <a name="additional-resources"></a>其他資源

- [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](./tutorial-list.md)

- [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

- [什麼是 Azure Active Directory 中的條件式存取？](../conditional-access/overview.md)
