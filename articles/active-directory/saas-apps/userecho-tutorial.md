---
title: 教學課程：Azure Active Directory 與 UserEcho 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 UserEcho 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: bedd916b-8f69-4b50-9b8d-56f4ee3bd3ed
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2017
ms.author: jeedes
ms.openlocfilehash: ecdd37db662c6861e35f80bfbf4ac8ff7e0d08c9
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39443259"
---
# <a name="tutorial-azure-active-directory-integration-with-userecho"></a>教學課程：Azure Active Directory 與 UserEcho 整合

在本教學課程中，您將了解如何整合 UserEcho 與 Azure Active Directory (Azure AD)。

將 UserEcho 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 UserEcho 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 UserEcho (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 UserEcho 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 UserEcho 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在這裡取得一個月試用：[試用供應項目](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 UserEcho
1. 設定並測試 Azure AD 單一登入

## <a name="adding-userecho-from-the-gallery"></a>從資源庫新增 UserEcho
若要設定將 UserEcho 整合到 Azure AD 中，您需要從資源庫將 UserEcho 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 UserEcho，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **UserEcho**。

    ![建立 Azure AD 測試使用者](./media/userecho-tutorial/tutorial_userecho_search.png)

1. 在結果窗格中，選取 [UserEcho]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/userecho-tutorial/tutorial_userecho_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 UserEcho 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 UserEcho 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 UserEcho 中的相關使用者之間建立連結關聯性。

在 UserEcho 中，將 Azure AD 中 [使用者名稱]的值指派為 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 UserEcho 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 UserEcho 測試使用者](#creating-a-userecho-test-user)** - 使 UserEcho 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 UserEcho 應用程式中設定單一登入。

**若要設定與 UserEcho 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [UserEcho] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_samlbase.png)

1. 在 [UserEcho 網域與 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<companyname>.userecho.com/`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<companyname>.userecho.com/saml/metadata/`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [UserEcho 用戶端支援小組](https://feedback.userecho.com/)以取得這些值。 

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/userecho-tutorial/tutorial_general_400.png)

1. 在 [UserEcho 組態] 區段上，按一下 [設定 UserEcho] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL 和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_configure.png) 

1. 在另一個瀏覽器視窗中，以系統管理員身分登入您的 UserEcho 公司網站。

1. 在頂端工具列中，按一下您的使用者名稱以展開功能表，然後按一下 [設定] 。
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_06.png) 

1. 按一下 [整合] 。
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_07.png) 

1. 按一下 [網站]，然後按一下 [單一登入 (SAML2)]。
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_08.png) 

1. 在 [單一登入 (SAML)]  頁面上，執行下列步驟：
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_09.png)
    
    a. 在 [已啟用 SAML] 選取 [是]。
    
    b. 將您從 Azure 入口網站複製的 [SAML 單一登入服務 URL]，貼到 [SAML SSO URL] 文字方塊中。
    
    c. 將您從 Azure 入口網站複製的「登出 URL」貼到 [遠端登出 URL] 文字方塊中。
    
    d. 在記事本中開啟下載的憑證，複製其內容，然後貼到 [X.509 憑證] 文字方塊中。
    
    e. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/userecho-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/userecho-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/userecho-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/userecho-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-userecho-test-user"></a>建立 UserEcho 測試使用者

本節的目標是要在 UserEcho 中建立一個名為 Britta Simon 的使用者。

**若要在 UserEcho 中建立名為 Britta Simon 的使用者，請執行下列步驟：**

1. 以系統管理員身分登入您的 UserEcho 公司網站。

1. 在頂端工具列中，按一下您的使用者名稱以展開功能表，然後按一下 [設定] 。
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_06.png)

1. 按一下 [使用者] 以展開 [使用者] 區段。
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_10.png)

1. 按一下 [使用者] 。
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_11.png)

1. 按一下 [邀請新使用者] 。
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_12.png)

1. 在 [邀請新使用者]  對話方塊中，執行下列步驟：
   
    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_13.png)

    a. 在 [名稱] 文字方塊中，輸入使用者的名稱，例如 Britta Simon。
    
    b.  在 [電子郵件] 文字方塊中，輸入像是 Brittasimon@contoso.com 的使用者電子郵件地址。
    
    c. 按一下 [邀請] 。

會傳送邀請給 Britta，邀請可讓她開始使用 UserEcho。 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 UserEcho 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 UserEcho，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [UserEcho] 。

    ![設定單一登入](./media/userecho-tutorial/tutorial_userecho_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。  

當您在存取面板中按一下 [UserEcho] 磚時，應該會自動登入您的 UserEcho 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/userecho-tutorial/tutorial_general_01.png
[2]: ./media/userecho-tutorial/tutorial_general_02.png
[3]: ./media/userecho-tutorial/tutorial_general_03.png
[4]: ./media/userecho-tutorial/tutorial_general_04.png

[100]: ./media/userecho-tutorial/tutorial_general_100.png

[200]: ./media/userecho-tutorial/tutorial_general_200.png
[201]: ./media/userecho-tutorial/tutorial_general_201.png
[202]: ./media/userecho-tutorial/tutorial_general_202.png
[203]: ./media/userecho-tutorial/tutorial_general_203.png

