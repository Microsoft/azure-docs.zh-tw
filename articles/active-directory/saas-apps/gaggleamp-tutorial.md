---
title: 教學課程：Azure Active Directory 與 GaggleAMP 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 GaggleAMP 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 9cc1a4b7-964b-406b-9e0c-05cb1a7c9856
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/06/2018
ms.author: jeedes
ms.openlocfilehash: 828dd1e1dcef900a7105143088f6782032b4f22e
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39436507"
---
# <a name="tutorial-azure-active-directory-integration-with-gaggleamp"></a>教學課程：Azure Active Directory 與 GaggleAMP 整合

在本教學課程中，您將了解如何整合 GaggleAMP 與 Azure Active Directory (Azure AD)。

GaggleAMP 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 GaggleAMP 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 GaggleAMP (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 GaggleAMP 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 GaggleAMP 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 GaggleAMP
1. 設定並測試 Azure AD 單一登入

## <a name="adding-gaggleamp-from-the-gallery"></a>從資源庫新增 GaggleAMP
若要設定將 GaggleAMP 整合到 Azure AD 中，您需要從資源庫將 GaggleAMP 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 GaggleAMP，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **GaggleAMP**。

    ![建立 Azure AD 測試使用者](./media/gaggleamp-tutorial/tutorial_gaggleamp_search.png)

1. 在結果窗格中，選取 [GaggleAMP]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/gaggleamp-tutorial/tutorial_gaggleamp_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 GaggleAMP 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 GaggleAMP 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 GaggleAMP 中相關使用者之間的連結關聯性。

在 GaggleAMP 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 GaggleAMP 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 GaggleAMP 測試使用者](#creating-a-gaggleamp-test-user)** - 在 GaggleAMP 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 GaggleAMP 應用程式中設定單一登入。

**若要設定與 GaggleAMP 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [GaggleAMP] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/gaggleamp-tutorial/tutorial_gaggleamp_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [GaggleAMP 網域與 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/gaggleamp-tutorial/tutorial_gaggleamp_url.png)

     在 [識別碼] 文字方塊中，輸入 URL：`https://accounts.gaggleamp.com/auth/saml/callback`

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![設定單一登入](./media/gaggleamp-tutorial/tutorial_gaggleamp_url1.png)

     在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://gaggleamp.com/i/<customerid>`

    > [!NOTE]
    > [登入 URL] 的值不是真正的值。 使用實際的「登入 URL」來更新此值。 請連絡 [GaggleAMP 用戶端支援小組](mailto:sales@gaggleamp.com)以取得此值。
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/gaggleamp-tutorial/tutorial_gaggleamp_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/gaggleamp-tutorial/tutorial_general_400.png)

1. 在 [GaggleAMP 組態] 區段上，按一下 [設定 GaggleAMP] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/gaggleamp-tutorial/tutorial_gaggleamp_configure.png) 

1. 在另一個瀏覽器執行個體中，瀏覽至 Gaggle 支援小組為您建立的 SAML SSO 頁面 (例如：*https://accounts.gaggleamp.com/saml_configurations/oXH8sQcP79dOzgFPqrMTyw/edit*)。

1. 在您的 [SAML SSO]  頁面上，執行下列步驟：  
   
    ![GaggleAMP 單一登入](./media/gaggleamp-tutorial/tutorial_gaggleamp_06.png)

    a. 從 [識別提供者] 下拉式功能表中，選取 [其他]。
    
    b. 在 [識別提供者簽發者] 文字方塊中，貼上您從 Azure 入口網站複製的 [簽發者 URL] 值。
    
    c. 在 [識別提供者單一登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [單一登入服務 URL] 值。
    
    d. 在記事本中開啟您下載的**憑證 (Base64)** 檔案，將其內容複製到剪貼簿，然後貼到 [X.509 憑證] 文字方塊中。
    
    e. 按一下 [檔案] 。

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/gaggleamp-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/gaggleamp-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/gaggleamp-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/gaggleamp-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-gaggleamp-test-user"></a>建立 GaggleAMP 測試使用者

本節的目標是要在 GaggleAMP 中建立名為 Britta Simon 的使用者。 GaggleAMP 支援預設啟用的 Just-In-Time 佈建。

在這一節沒有您需要進行的動作項目。 嘗試存取 GaggleAMP 時，如果使用者尚未存在，就會建立新使用者。 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 GaggleAMP 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 GaggleAMP，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [GaggleAMP] 。

    ![設定單一登入](./media/gaggleamp-tutorial/tutorial_gaggleamp_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。

當您在「存取面板」中按一下 [GaggleAMP] 磚時，應該會自動登入您的 GaggleAMP 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/gaggleamp-tutorial/tutorial_general_01.png
[2]: ./media/gaggleamp-tutorial/tutorial_general_02.png
[3]: ./media/gaggleamp-tutorial/tutorial_general_03.png
[4]: ./media/gaggleamp-tutorial/tutorial_general_04.png

[100]: ./media/gaggleamp-tutorial/tutorial_general_100.png

[200]: ./media/gaggleamp-tutorial/tutorial_general_200.png
[201]: ./media/gaggleamp-tutorial/tutorial_general_201.png
[202]: ./media/gaggleamp-tutorial/tutorial_general_202.png
[203]: ./media/gaggleamp-tutorial/tutorial_general_203.png
