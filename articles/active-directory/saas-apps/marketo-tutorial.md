---
title: 教學課程：Azure Active Directory 與 Marketo 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Marketo 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/13/2021
ms.author: jeedes
ms.openlocfilehash: f0bf99748363505e362d3c35e53a51be3a03e938
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98728677"
---
# <a name="tutorial-azure-active-directory-integration-with-marketo"></a>教學課程：Azure Active Directory 與 Marketo 整合

在本教學課程中，您會了解如何整合 Marketo 與 Azure Active Directory (Azure AD)。
Marketo 與 Azure AD 整合提供下列優點：

* 您可以在 Azure AD 中控制可存取 Marketo 的人員。
* 您可以讓使用者使用其 Azure AD 帳戶自動登入 Marketo (單一登入)。
* 您可以在 Azure 入口網站中集中管理您的帳戶。

## <a name="prerequisites"></a>Prerequisites

若要設定 Azure AD 與 Marketo 整合，您需要下列項目：

* Azure AD 訂用帳戶。 如果您沒有 Azure AD 環境，您可以在[這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月的試用帳戶
* 已啟用 Marketo 單一登入的訂用帳戶

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD 單一登入。

* Marketo 支援由 **IDP** 起始的 SSO

> [!NOTE]
> 此應用程式的識別碼是固定的字串值，因此一個租用戶中只能設定一個執行個體。

## <a name="adding-marketo-from-the-gallery"></a>從資源庫新增 Marketo

若要設定 Marketo 與 Azure AD 整合，您需要從資源庫將 Marketo 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]。
1. 在 [從資源庫新增] 區段的搜尋方塊中，輸入 **Marketo**。
1. 從結果面板中選取 [Marketo]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-marketo"></a>設定和測試 Marketo 的 Azure AD SSO

在本節中，您將以名為 **Britta Simon** 的測試使用者身分，設定及測試與 Marketo 搭配運作的 Azure AD 單一登入。
若要讓單一登入能夠運作，必須建立 Azure AD 使用者與 Marketo 中相關使用者之間的連結關聯性。

若要設定及測試與 Marketo 搭配運作的 Azure AD 單一登入，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 以 Britta Simon 測試 Azure AD SSO。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓使用者 Britta Simon 能夠使用 Azure AD SSO。
2. **[設定 Marketo SSO](#configure-marketo-sso)** - 在應用程式端設定 SSO 設定。
    1. **[建立 Marketo 測試使用者](#create-marketo-test-user)** - 使 Marketo 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
3. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [Marketo] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入] 頁面上，按一下 [基本 SAML 設定] 的鉛筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，輸入下列欄位的值：

    a. 在 [識別碼]  文字方塊中，輸入 URL：`https://saml.marketo.com/sp`

    b. 在 [回覆 URL]  文字方塊中，使用下列模式來輸入 URL：`https://login.marketo.com/saml/assertion/\<munchkinid\>`

    c. 在 [轉送狀態]  文字方塊中，使用下列模式輸入 URL：`https://<munchkinid>.marketo.com/`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的「回覆 URL」和「轉送狀態」來更新這些值。 請連絡[Marketo 用戶端支援小組](https://investors.marketo.com/contactus.cfm)以取得這些值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

5. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，按一下 [下載]  ，以依據您的需求從指定選項下載 [憑證 (Base64)]  ，並儲存在您的電腦上。

    ![憑證下載連結](common/certificatebase64.png)

6. 在 [設定 Marketo]  區段上，依據您的需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者 

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下頁面底部的 [新增] 。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Marketo 的存取權授與 B.Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Marketo]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-marketo-sso"></a>設定 Marketo SSO

1. 若要自動執行 Marketo 內的設定，您必須按一下 [安裝擴充功能] 來安裝「我的應用程式安全登入瀏覽器擴充功能」。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 Marketo] 便會將您導向到 Marketo 應用程式。 請從該處提供用以登入 Marketo 的管理員認證。 瀏覽器擴充功能會自動為您設定應用程式，並自動執行步驟 3 到 6。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 Marketo，請在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Marketo 公司網站。

1. 若要取得應用程式的 Munchkin 識別碼，請執行下列動作：
   
    a. 使用管理員認證登入 Marketo 應用程式。
   
    b. 按一下頂端瀏覽窗格中的 [管理]  按鈕。
   
    ![設定單一登入 1](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 瀏覽至 [整合] 功能表，然後按一下 [Munchkin]  連結。
   
    ![設定單一登入 2](./media/marketo-tutorial/tutorial_marketo_11.png)
   
    d. 複製螢幕上顯示的 Munchkin 識別碼，完成 Azure AD 設定精靈中的回覆 URL。
   
    ![設定單一登入 3](./media/marketo-tutorial/tutorial_marketo_12.png) 

2. 若要設定應用程式中的 SSO，請遵循下列步驟︰
   
    a. 使用管理員認證登入 Marketo 應用程式。
   
    b. 按一下頂端瀏覽窗格中的 [管理]  按鈕。
   
    ![設定單一登入 4](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 瀏覽至 [整合] 功能表，然後按一下 [單一登入]  。
   
    ![設定單一登入 5](./media/marketo-tutorial/tutorial_marketo_07.png) 
   
    d. 若要啟用 SAML 設定，按一下 [編輯]  按鈕。
   
    ![設定單一登入6](./media/marketo-tutorial/tutorial_marketo_08.png) 
   
    e. **已啟用** 單一登入設定。
   
    f. 在 [簽發者識別碼] 文字方塊中，貼上 [Azure AD 識別碼]。
   
    g. 在 [實體識別碼]  文字方塊中，輸入 URL `http://saml.marketo.com/sp`。
   
    h. 選取 [使用者識別碼位置] 做為 [名稱識別碼元素]  。
   
    ![設定單一登入 7](./media/marketo-tutorial/tutorial_marketo_09.png)
   
    > [!NOTE]
    > 如果您的使用者識別碼不是 UPN 值，則至 [屬性] 索引標籤中變更其值。
   
    i. 上傳您從 Azure AD 設定精靈下載的憑證。 **儲存** 設定。
   
    j. 編輯 [重新導向頁面] 設定。
   
    k. 在 [登入 URL]  文字方塊中，貼上 [登入 URL]  。
   
    l. 在 [登出 URL]  文字方塊中，貼上登出 URL  。
   
    m. 在 [錯誤 URL]  中，複製您的 **Marketo 執行個體 URL**，並按一下 [儲存]  按鈕以儲存設定。
   
    ![設定單一登入 8](./media/marketo-tutorial/tutorial_marketo_10.png)

3. 若要啟用使用者的 SSO，完成下列動作：
   
    a. 使用管理員認證登入 Marketo 應用程式。
   
    b. 按一下頂端瀏覽窗格中的 [管理]  按鈕。
   
    ![設定單一登入 9](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 瀏覽至 [安全性]  功能表，然後按一下 [登入設定]  。
   
    ![設定單一登入 10](./media/marketo-tutorial/tutorial_marketo_13.png)
   
    d. 勾選 [需要 SSO]  選項，並 [儲存]  設定。
   
    ![設定單一登入 11](./media/marketo-tutorial/tutorial_marketo_14.png)


### <a name="create-marketo-test-user"></a>建立 Marketo 測試使用者

在本節中，您要在 Marketo 中建立名為 Britta Simon 的使用者。 遵循以下步驟在 Marketo 平台中建立使用者。

1. 使用管理員認證登入 Marketo 應用程式。

2. 按一下頂端瀏覽窗格中的 [管理]  按鈕。
   
    ![測試使用者 1](./media/marketo-tutorial/tutorial_marketo_06.png) 

3. 瀏覽至 [安全性]  功能表，然後按一下 [使用者與角色] 
   
    ![測試使用者 2](./media/marketo-tutorial/tutorial_marketo_19.png)  

4. 按一下 [使用者] 索引標籤上的 [邀請新使用者]  連結
   
    ![測試使用者 3](./media/marketo-tutorial/tutorial_marketo_15.png) 

5. 在 [邀請新使用者] 精靈中填寫下列資訊。
   
    a. 在 [電子郵件]  文字方塊中輸入使用者的電子郵件地址。
   
    ![測試使用者 4](./media/marketo-tutorial/tutorial_marketo_16.png)
   
    b. 在 [名字]  文字方塊中輸入名字。
   
    c. 在 [姓氏]  文字方塊中輸入姓氏。
   
    d. 按 **[下一步]**

6. 在 [權限]  索引標籤中選取 [userRoles]  ，然後按一下 [下一步] 
   
    ![測試使用者 5](./media/marketo-tutorial/tutorial_marketo_17.png)
7. 按一下 [傳送]  按鈕，傳送使用者邀請
   
    ![測試使用者 6](./media/marketo-tutorial/tutorial_marketo_18.png)

8. 使用者會收到電子郵件通知，他必須按一下連結，並變更密碼才能啟動帳戶。 

### <a name="test-sso"></a>測試 SSO

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入您已設定 SSO 的 Marketo

* 您可以使用 Microsoft 的「我的應用程式」。 當您在我的應用程式中按一下 Marketo 圖格時，應該會自動登入您已設定 SSO 的 Marketo。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。

## <a name="next-steps"></a>後續步驟

設定 Marketo 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。