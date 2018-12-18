---
title: 教學課程：將 Azure Active Directory 與 Kiteworks 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Kiteworks 之間的單一登入功能。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: f7984aaf-ab1f-4a85-9646-a9523f5275d9
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2017
ms.author: jeedes
ms.openlocfilehash: 88531ee1a98eefdf3ee85b4308150f10417858d0
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39420218"
---
# <a name="tutorial-azure-active-directory-integration-with-kiteworks"></a>教學課程：將 Azure Active Directory 與 Kiteworks 整合

在本教學課程中，您將了解如何整合 Kiteworks 與 Azure Active Directory (Azure AD)。

將 Kiteworks 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Kiteworks 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Kiteworks (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Kiteworks 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Kiteworks 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Kiteworks
1. 設定並測試 Azure AD 單一登入

## <a name="adding-kiteworks-from-the-gallery"></a>從資源庫新增 Kiteworks
若要設定將 Kiteworks 整合到 Azure AD 中，您需要從資源庫將 Kiteworks 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Kiteworks，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Kiteworks**。

    ![建立 Azure AD 測試使用者](./media/kiteworks-tutorial/tutorial_kiteworks_search.png)

1. 在結果窗格中，選取 [Kiteworks]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/kiteworks-tutorial/tutorial_kiteworks_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Kiteworks 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Kiteworks 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Kiteworks 中的相關使用者之間建立連結關聯性。

在 Kiteworks 中，將 Azure AD 中 [使用者名稱] 的值指派為 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 Kiteworks 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Kiteworks 測試使用者](#creating-a-kiteworks-test-user)** - 使 Kiteworks 中 Britta Simon 的對應使用者連結到該使用者在 Azure AD 中的代表身分。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Kiteworks 應用程式中設定單一登入。

**若要設定與 Kiteworks 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Kiteworks] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_samlbase.png)

1. 在 [Kiteworks 網域及 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<subdomain>.kiteworks.com`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<subdomain>.kiteworks.com/sp/module.php/saml/sp/saml2-acs.php/sp-sso`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Kiteworks 用戶端支援小組](http://accellion.com/support)以取得這些值。 
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/kiteworks-tutorial/tutorial_general_400.png)

1. 在 [Kiteworks 組態] 區段上，按一下 [設定 Kiteworks] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_configure.png) 

1. 以系統管理員身分登入 Kiteworks 公司網站。

1. 在頂端的工具列中，按一下 [設定] 。
   
    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_06.png) 

1. 在 [驗證和授權] 區段中，按一下 [SSO 設定]。 
   
    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_07.png)
 
1. 在 [SSO 設定] 頁面上，執行下列步驟：
   
    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_09.png)   

    a. 選取 [透過 SSO 驗證]。

    b. 選取 [起始 AuthnRequest]。

    c. 在 [IDP 實體識別碼] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。 

    d. 在 [單一登入服務 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    e. 在 [單一登出服務 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    f. 在 [記事本] 中開啟下載的憑證，複製其內容，然後貼到 [RSA 公開金鑰憑證] 文字方塊中。
 
    g. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/kiteworks-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/kiteworks-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/kiteworks-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/kiteworks-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-kiteworks-test-user"></a>建立 Kiteworks 測試使用者

本節的目標是要在 Kiteworks 中建立一個名為 Britta Simon 的使用者。

Kiteworks 支援預設啟用的 Just-in-Time 佈建。 在這一節沒有您需要進行的動作項目。 嘗試存取 Kitewors 時，如果使用者還不存在，就會建立新使用者。

>[!NOTE]
>如果您需要手動建立使用者，您需要連絡 [Kiteworks 支援小組](http://accellion.com/support)。
 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Kiteworks 的存取權授與 Britta Simon，讓 Britta Simon 能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Kiteworks，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Kiteworks] 。

    ![設定單一登入](./media/kiteworks-tutorial/tutorial_kiteworks_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。  

當您在存取面板中按一下 Kiteworks 磚時，應該會自動登入 Kiteworks 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/kiteworks-tutorial/tutorial_general_01.png
[2]: ./media/kiteworks-tutorial/tutorial_general_02.png
[3]: ./media/kiteworks-tutorial/tutorial_general_03.png
[4]: ./media/kiteworks-tutorial/tutorial_general_04.png

[100]: ./media/kiteworks-tutorial/tutorial_general_100.png

[200]: ./media/kiteworks-tutorial/tutorial_general_200.png
[201]: ./media/kiteworks-tutorial/tutorial_general_201.png
[202]: ./media/kiteworks-tutorial/tutorial_general_202.png
[203]: ./media/kiteworks-tutorial/tutorial_general_203.png

