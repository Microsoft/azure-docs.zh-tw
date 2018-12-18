---
title: 教學課程：Azure Active Directory 與 Kantega SSO for Bitbucket 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Kantega SSO for Bitbucket 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: c41cdaaf-0441-493c-94c7-569615b7b1ab
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2017
ms.author: jeedes
ms.openlocfilehash: a81ea48937927e13141642d70093bc322196b2cc
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39434987"
---
# <a name="tutorial-azure-active-directory-integration-with-kantega-sso-for-bitbucket"></a>教學課程：Azure Active Directory 與 Kantega SSO for Bitbucket 整合

在本教學課程中，您會了解如何整合 Kantega SSO for Bitbucket 與 Azure Active Directory (Azure AD)。

Kantega SSO for Bitbucket 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Kantega SSO for Bitbucket 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Kantega SSO for Bitbucket (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Kantega SSO for Bitbucket 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Kantega SSO for Bitbucket 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Kantega SSO for Bitbucket
1. 設定並測試 Azure AD 單一登入

## <a name="adding-kantega-sso-for-bitbucket-from-the-gallery"></a>從資源庫新增 Kantega SSO for Bitbucket
若要設定將 Kantega SSO for Bitbucket 整合到 Azure AD 中，您需要從資源庫將 Kantega SSO for Bitbucket 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Kantega SSO for Bitbucket，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Kantega SSO for Bitbucket**。

    ![建立 Azure AD 測試使用者](./media/kantegassoforbitbucket-tutorial/tutorial_kantegassoforbitbucket_search.png)

1. 在結果面板中，選取 [Kantega SSO for Bitbucket]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/kantegassoforbitbucket-tutorial/tutorial_kantegassoforbitbucket_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Kantega SSO for Bitbucket 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Kantega SSO for Bitbucket 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Kantega SSO for Bitbucket 中的相關使用者之間，建立連結關聯性。

在 Kantega SSO for Bitbucket 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Kantega SSO for Bitbucket 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Kantega SSO for Bitbucket 測試使用者](#creating-a-kantega-sso-for-bitbucket-test-user)** - 使 Kantega SSO for Bitbucket 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Kantega SSO for Bitbucket 應用程式中設定單一登入。

**若要設定與 Kantega SSO for Bitbucket 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Kantega SSO for Bitbucket] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/tutorial_kantegassoforbitbucket_samlbase.png)

1. 在 **IDP** 起始模式下，在 [Kantega SSO for Bitbucket 網域及 URL] 區段中執行下列步驟：

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/tutorial_kantegassoforbitbucket_url1.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

1. 在 **SP** 起始模式下，勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/tutorial_kantegassoforbitbucket_url2.png)
    
    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 在設定 Bitbucket 外掛程式 (本教學課程稍後會說明) 期間會收到這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/tutorial_kantegassoforbitbucket_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/tutorial_general_400.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Bitbucket 系統管理入口網站。

1. 按一下齒輪，然後按一下 [尋找新的附加元件]。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon1.png)

1. 搜尋 **Kantega SSO for Bitbucket SAML & Kerberos**，然後按一下 [安裝] 按鈕以安裝新的 SAML 外掛程式。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon2.png)

1. 外掛程式會開始安裝。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon31.png)

1. 當安裝完成時。 按一下 [關閉] 。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon33.png)

1.  按一下 [管理] 。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon34.png)
    
1. 按一下 [設定] 來設定新的外掛程式。 

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon35.png)

1. 在 [SAML] 區段中。 從 [新增識別提供者] 下拉式清單中，選取 [Azure Active Directory]\(Azure AD\)。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon4.png)

1. 選取 [基本] 作為訂用帳戶層級。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon5.png)

1. 在 [應用程式屬性] 區段中，執行下列步驟：

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon6.png)

    a. 複製 [應用程式識別碼 URI] 值，然後在 Azure 入口網站的 [Kantega SSO for Bitbucket 網域及 URL] 區段上，使用此 URI 作為 [識別碼]、[回覆 URL] 和 [登入 URL]。

    b. 按 [下一步] 。

1. 在 [中繼資料匯入] 區段中，執行下列步驟：

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon7.png)

    a. 選取 [我的電腦上的中繼資料檔案]，上傳您從 Azure 入口網站下載的中繼資料檔案。

    b. 按 [下一步] 。

1. 在 [名稱和 SSO 位置] 區段中，執行下列步驟：

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon8.png)

    a. 在 [識別提供者名稱] 文字方塊中，新增識別提供者的名稱 (例如 Azure AD)。

    b. 按 [下一步] 。

1. 確認簽署憑證，然後按 [下一步]。   

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon9.png)

1. 在 [Bitbucket 使用者帳戶] 區段中，執行下列步驟：

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon10.png)

    a. 選取 [如果需要，在 Bitbucket 的內部目錄中建立使用者]，然後輸入使用者群組的適當名稱 (可以是多個 群組，以逗號分隔)。

    b. 按 [下一步] 。

1. 按一下 [完成] 。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon11.png)

1. 在 [Azure AD 的已知網域] 區段中，執行下列步驟：  

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/addon12.png)

    a. 從頁面的左面板中，選取 [已知網域]。

    b. 在 [已知網域] 文字方塊中，輸入網域名稱。

    c. 按一下 [檔案] 。  

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/kantegassoforbitbucket-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/kantegassoforbitbucket-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/kantegassoforbitbucket-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/kantegassoforbitbucket-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-kantega-sso-for-bitbucket-test-user"></a>建立 Kantega SSO for Bitbucket 測試使用者

若要讓 Azure AD 使用者能夠登入 Bitbucket，必須將他們佈建到 Bitbucket。 在 Kantega SSO for Bitbucket 中，佈建是手動工作。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 Bitbucket 公司網站。

1. 按一下設定圖示。

    ![新增員工](./media/kantegassoforbitbucket-tutorial/user1.png) 

1. 在 [管理] 索引標籤區段下，按一下 [使用者]。

    ![新增員工](./media/kantegassoforbitbucket-tutorial/user2.png)

1. 按一下 [建立使用者]。

    ![新增員工](./media/kantegassoforbitbucket-tutorial/user3.png)   

1. 在 [建立使用者] 對話頁面上，執行下列步驟：

    ![新增員工](./media/kantegassoforbitbucket-tutorial/user4.png) 

    a. 在 [使用者名稱] 文字方塊中，輸入像是 Brittasimon@contoso.com 的使用者電子郵件。
    
    b. 在 [全名] 文字方塊中，輸入像是 Britta Simon 的使用者全名。
    
    c. 在 [電子郵件地址] 文字方塊中，輸入像是 Brittasimon@contoso.com 的使用者電子郵件地址。

    d. 在 [密碼] 文字方塊中，輸入使用者的密碼。  

    e. 在 [確認密碼] 文字方塊中，再次輸入使用者的密碼。

    f. 按一下 [建立使用者]。   

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Kantega SSO for Bitbucket 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Kantega SSO for Bitbucket，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Kantega SSO for Bitbucket]。

    ![設定單一登入](./media/kantegassoforbitbucket-tutorial/tutorial_kantegassoforbitbucket_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Kantega SSO for Bitbucket 圖格時，應該會自動登入您的 Kantega SSO for Bitbucket 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_01.png
[2]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_02.png
[3]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_03.png
[4]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_04.png

[100]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_100.png

[200]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_200.png
[201]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_201.png
[202]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_202.png
[203]: ./media/kantegassoforbitbucket-tutorial/tutorial_general_203.png

