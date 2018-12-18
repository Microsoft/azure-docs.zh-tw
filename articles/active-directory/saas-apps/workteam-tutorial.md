---
title: 教學課程：Azure Active Directory 與 Workteam 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Workteam 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 41df17a1-ba69-414f-8ec3-11079b030df6
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/28/2018
ms.author: jeedes
ms.openlocfilehash: 8d6ca6395e4f5e1aca361c56e21afc4e6bd1fc0c
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43132425"
---
# <a name="tutorial-azure-active-directory-integration-with-workteam"></a>教學課程：Azure Active Directory 與 Workteam 整合

在本教學課程中，您將了解如何整合 Workteam 與 Azure Active Directory (Azure AD)。

Workteam 與 Azure AD 的整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Workteam 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Workteam (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Workteam 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Workteam 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Workteam
2. 設定並測試 Azure AD 單一登入

## <a name="adding-workteam-from-the-gallery"></a>從資源庫新增 Workteam
若要進行將 Workteam 整合至 Azure AD 的設定，您需要從資源庫將 Workteam 新增至受控 SaaS 應用程式清單。

**若要從資源庫新增 Workteam，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中輸入 **Workteam**，從結果面板中選取 **Workteam**，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Workteam](./media/workteam-tutorial/tutorial_workteam_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Workteam 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Workteam 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Workteam 中的相關使用者之間建立連結關聯性。

若要設定並測試與 Workteam 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Workteam 測試使用者](#create-a-workteam-test-user)** - 使 Workteam 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Workteam 應用程式中設定單一登入。

**若要設定與 Workteam 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站中的 **Workteam** 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/workteam-tutorial/tutorial_workteam_samlbase.png)

3. 在 [Workteam 網域和 URL] 區段中，使用者不需要執行任何步驟，因為應用程式已預先與 Azure 整合。

    ![Workteam 網域與 URL 單一登入資訊](./media/workteam-tutorial/tutorial_workteam_url.png)

4. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![Workteam 網域與 URL 單一登入資訊](./media/workteam-tutorial/tutorial_workteam_url1.png)

    在 [登入 URL] 文字方塊中，輸入 URL：`https://app.workte.am`
     
5. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/workteam-tutorial/tutorial_workteam_certificate.png) 

6. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/workteam-tutorial/tutorial_general_400.png)
    
7. 在 [Workteam 設定] 區段上按一下 [設定 Workteam]，以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體識別碼] 和 [SAML 單一登入服務 URL]。

    ![Workteam 設定](./media/workteam-tutorial/tutorial_workteam_configure.png) 

8. 在不同的網頁瀏覽器視窗中，以安全性系統管理員身分登入 Workteam。

9. 在右上角按一下**設定檔標誌**，然後按一下 [組織設定]。 

    ![Workteam 設定](./media/workteam-tutorial/tutorial_workteam_settings.png)

10. 在 [驗證] 區段下方，按一下 [設定標誌]。

     ![Workteam azure](./media/workteam-tutorial/tutorial_workteam_azure.png)

11. 在 [SAML設定]  頁面上，執行下列步驟：

     ![Workteam saml](./media/workteam-tutorial/tutorial_workteam_saml.png)

    a. 選取 [SAML IdP] 作為 **AD Azure**。

    b. 在 [SAML 單一登入服務 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    c. 在 [SAML 實體識別碼] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼]值。

    d. 在 [記事本] 中開啟從 Azure 入口網站下載的 **Base-64 編碼憑證**，複製其內容，然後貼到 [SAML 簽署憑證 (Base64)] 方塊中。

    e. 按一下 [確定]。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/workteam-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/workteam-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/workteam-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/workteam-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-workteam-test-user"></a>建立 Workteam 測試使用者

為使 Azure AD 使用者登入 Workteam，必須將他們佈建至 Workteam。 在 Workteam 中，需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以安全性系統管理員身分登入 Workteam。

2. 在 [組織設定] 頁面正上方按一下 [使用者]，然後按一下 [新增使用者]。

    ![Workteam 使用者](./media/workteam-tutorial/tutorial_workteam_user.png)

3. 在 [新增員工] 頁面上，執行下列步驟：

    ![Workteam 的新使用者](./media/workteam-tutorial/tutorial_workteam_newuser.png)

    a. 在 [名稱] 文字方塊中，輸入使用者的名字，例如 **Brittasimon**。

    b. 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件，例如 **Brittasimon@contoso.com**。

    c. 按一下 [確定]。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Asana 的存取權授與 Britta Simon，讓她能夠使用 Workteam 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Workteam，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 **Workteam**。

    ![應用程式清單中的 Workteam 連結](./media/workteam-tutorial/tutorial_workteam_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Workteam] 圖格時，您應該會自動登入 Workteam 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/workteam-tutorial/tutorial_general_01.png
[2]: ./media/workteam-tutorial/tutorial_general_02.png
[3]: ./media/workteam-tutorial/tutorial_general_03.png
[4]: ./media/workteam-tutorial/tutorial_general_04.png

[100]: ./media/workteam-tutorial/tutorial_general_100.png

[200]: ./media/workteam-tutorial/tutorial_general_200.png
[201]: ./media/workteam-tutorial/tutorial_general_201.png
[202]: ./media/workteam-tutorial/tutorial_general_202.png
[203]: ./media/workteam-tutorial/tutorial_general_203.png

