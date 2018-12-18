---
title: 教學課程：Azure Active Directory 與 Procore SSO 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Procore SSO 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 9818edd3-48c0-411d-b05a-3ec805eafb2e
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/07/2017
ms.author: jeedes
ms.openlocfilehash: ffc1e0d66761700cb0da1f998c2639afc5abb2c4
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49114051"
---
# <a name="tutorial-azure-active-directory-integration-with-procore-sso"></a>教學課程：Azure Active Directory 與 Procore SSO 整合

在本教學課程中，您會了解如何整合 Procore SSO 與 Azure Active Directory (Azure AD)。

Procore SSO 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Procore SSO 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Procore SSO (單一登入)
- 您可以在 Azure 管理入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Procore SSO 與 Azure AD 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Procore SSO 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Procore SSO
1. 設定並測試 Azure AD 單一登入

## <a name="adding-procore-sso-from-the-gallery"></a>從資源庫新增 Procore SSO
若要設定將 Procore SSO 整合到 Azure AD 中，您需要從資源庫將 Procore SSO 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Procore SSO，請執行下列步驟：**

1. 在 **[Azure 管理入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 按一下對話方塊頂端的 [新增] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Procore SSO**。

    ![建立 Azure AD 測試使用者](./media/procoresso-tutorial/tutorial_procoresso_search.png)

1. 在結果窗格中，選取 [Procore SSO]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/procoresso-tutorial/tutorial_procoresso_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Procore SSO 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Procore SSO 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Procore SSO 中相關使用者之間的連結關聯性。

建立此連結關聯性的方法，就是將 Azure AD 中**使用者名稱**的值指派為 Procore SSO 中 **Username** 的值。

若要設定及測試與 Procore SSO 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Procore SSO 測試使用者](#creating-a-procore-sso-test-user)** - 讓 Procore SSO 中對應的 Britta Simon 連結到她在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 管理入口網站中啟用 Azure AD 單一登入，並在您的 Procore SSO 中設定單一登入。

**若要設定與 Procore SSO 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 管理入口網站的 [Procore SSO] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，選取 [SAML 型登入] 作為 [模式]，以啟用單一登入。
 
    ![設定單一登入](./media/procoresso-tutorial/tutorial_procoresso_samlbase.png)

1. 在 [Procore SSO 網域和 URL] 區段中，使用者不需要執行任何步驟，因為應用程式已預先與 Azure 整合。

    ![設定單一登入](./media/procoresso-tutorial/tutorial_procoresso_url.png)

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/procoresso-tutorial/tutorial_procoresso_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/procoresso-tutorial/tutorial_general_400.png)

1. 在 [Procore SSO 組態] 區段上，按一下 [設定 Procore SSO] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/procoresso-tutorial/tutorial_procoresso_configure.png) 

1. 若要在 **Procore SSO** 端設定單一登入，請以系統管理員身分登入 procore 公司網站。

1. 從工具箱下拉式清單按一下 [管理] 以開啟 SSO 設定頁面。

    ![設定單一登入](./media/procoresso-tutorial/procore_tool_admin.png)

1. 如下所述貼上方塊中的值-

    ![設定單一登入](./media/procoresso-tutorial/procore_setting_admin.png)  

    a. 在 [單一登入簽發者 URL] 方塊中，貼上從 Azure 入口網站複製的 SAML 實體 ID。

    b. 在 [SAML 登入目標 URL] 方塊中，貼上從 Azure 入口網站複製的 SAML 單一登入服務 URL。

    c. 現在開啟上述從 Azure 入口網站下載的**中繼資料 XML**，然後複製名為 **X509Certificate**之標記中的憑證。 將複製的值貼至 [單一登入 x509 憑證] 方塊。

1. 按一下 [儲存變更]。

1. 在進行這些設定之後，您必須將透過它來登入 Procore 的 **網域名稱** (例如 **contoso.com**)，傳送給 [Procore 支援小組](https://support.procore.com/)，支援小組會為該網域啟動同盟 SSO。

<!--### Next steps

To ensure users can sign-in to Procore SSO after it has been configured to use Azure Active Directory, review the following tasks and topics:

- User accounts must be pre-provisioned into Procore SSO prior to sign-in. To set this up, see Provisioning.
 
- Users must be assigned access to Procore SSO in Azure AD to sign-in. To assign users, see Users.
 
- To configure access polices for Procore SSO users, see Access Policies.
 
- For additional information on deploying single sign-on to users, see [this article](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis#deploying-azure-ad-integrated-applications-to-users).-->


### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節目標是在 Azure 管理入口網站中建立名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 管理入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/procoresso-tutorial/create_aaduser_01.png) 

1. 移至 [使用者和群組]，然後按一下 [所有使用者] 以顯示使用者清單。
    
    ![建立 Azure AD 測試使用者](./media/procoresso-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。
 
    ![建立 Azure AD 測試使用者](./media/procoresso-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/procoresso-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-procore-sso-test-user"></a>建立 Procore SSO 測試使用者

請遵循下列步驟以在 Procore 端建立 Procore 測試使用者。

1. 以系統管理員身分登入您的 procore 公司網站。  

1. 從工具箱下拉式清單按一下 [目錄] 以開啟公司目錄頁面。

    ![設定單一登入](./media/procoresso-tutorial/Procore_sso_directory.png)

1. 按一下 [新增人員] 選項來開啟表單，然後輸入執行以下選項-

    ![設定單一登入](./media/procoresso-tutorial/Procore_user_add.png)

    a. 在 [名字] 文字方塊中，輸入使用者的名字，例如 **Britta**。

    b. 在 [姓氏] 文字方塊中，輸入使用者的姓氏，例如 **Simon**。

    c. 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件地址，例如 **BrittaSimon@contoso.com**。

    d. 將 [權限範本] 選取為 [稍後套用權限範本]。

    e. 按一下頁面底部的 [新增] 。

1. 檢查並更新新增之連絡人的詳細資料。

    ![設定單一登入](./media/procoresso-tutorial/Procore_user_check.png)

1. 按一下 [儲存並傳送邀請] \(如果透過郵件邀請是必要的)，或者按一下 [儲存] \(直接儲存) 以完成使用者註冊。
    
    ![設定單一登入](./media/procoresso-tutorial/Procore_user_save.png)  

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Procore SSO 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Procore SSO，請執行下列步驟：**

1. 在 Azure 管理入口網站中，開啟應用程式檢視，然後瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Procore SSO]。

    ![設定單一登入](./media/procoresso-tutorial/tutorial_procoresso_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

如果要測試您的單一登入設定，請開啟存取面板。 如需 [存取面板] 的詳細資訊，請參閱 [存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 當您按一下「存取面板」中的 Procore SSO 圖格時，應該會自動登入您的 Procore SSO 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/procoresso-tutorial/tutorial_general_01.png
[2]: ./media/procoresso-tutorial/tutorial_general_02.png
[3]: ./media/procoresso-tutorial/tutorial_general_03.png
[4]: ./media/procoresso-tutorial/tutorial_general_04.png

[100]: ./media/procoresso-tutorial/tutorial_general_100.png

[200]: ./media/procoresso-tutorial/tutorial_general_200.png
[201]: ./media/procoresso-tutorial/tutorial_general_201.png
[202]: ./media/procoresso-tutorial/tutorial_general_202.png
[203]: ./media/procoresso-tutorial/tutorial_general_203.png

