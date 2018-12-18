---
title: 教學課程：Azure Active Directory 與 Workplace by Facebook 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Workplace by Facebook 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 30f2ee64-95d3-44ef-b832-8a0a27e2967c
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/30/2018
ms.author: jeedes
ms.openlocfilehash: 1f83dd64c7f6773ddb8956e6ebbc37b8c55aacec
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39423866"
---
# <a name="tutorial-azure-active-directory-integration-with-workplace-by-facebook"></a>教學課程：Azure Active Directory 與 Workplace by Facebook 整合

在本教學課程中，您會了解如何將 Workplace by Facebook 與 Azure Active Directory (Azure AD) 整合。

將 Workplace by Facebook 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Workplace by Facebook 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Workplace by Facebook (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Workplace by Facebook 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Workplace by Facebook 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

> [!NOTE]
> Facebook 有兩項產品 Workplace Standard (免費) 和 Workplace Premium (付費)。 任何 Workplace Premium 租用戶都可以設定 SCIM 和 SSO 整合，而無須任何其他費用或授權。 Workplace Standard 執行個體中並未提供 SSO 和 SCIM。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Workplace by Facebook
1. 設定並測試 Azure AD 單一登入

## <a name="adding-workplace-by-facebook-from-the-gallery"></a>從資源庫新增 Workplace by Facebook
若要設定將 Workplace by Facebook 整合到 Azure AD 中，您需要從資源庫將 Workplace by Facebook 新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 Workplace by Facebook，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Workplace by Facebook**。

    ![建立 Azure AD 測試使用者](./media/workplacebyfacebook-tutorial/tutorial_workplacebyfacebook_search.png)

1. 在結果面板中，選取 [Workplace by Facebook]，然後按一下 [新增] 按鈕以新增該應用程式。

    ![建立 Azure AD 測試使用者](./media/workplacebyfacebook-tutorial/tutorial_workplacebyfacebook_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Workplace by Facebook 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Workplace by Facebook 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Workplace by Facebook 中的相關使用者之間建立連結關聯性。

建立此連結關聯性的方法，就是將 Azure AD 中**使用者名稱**的值指派為 Workplace by Facebook 中 **Username** 的值。

若要設定及測試與 Workplace by Facebook 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[設定重新驗證頻率](#configuring-reauthentication-frequency)** - 可設定要提示進行 SAML 檢查的 Workplace。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Workplace by Facebook 測試使用者](#creating-a-workplace-by-facebook-test-user)** - 在 Workplace by Facebook 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Workplace by Facebook 應用程式中設定單一登入。

**若要設定透過 Workplace by Facebook 使用 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Workplace by Facebook] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/workplacebyfacebook-tutorial/tutorial_workplacebyfacebook_samlbase.png)

1. 在 [Workplace by Facebook 網域與 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/workplacebyfacebook-tutorial/tutorial_workplacebyfacebook_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<instancename>.facebook.com`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://www.facebook.com/company/<instanceID>`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請參閱 [Workplace 公司儀表板] 的 [驗證] 頁面，以查看您 Workplace 社群的正確值。 

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/workplacebyfacebook-tutorial/tutorial_workplacebyfacebook_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/workplacebyfacebook-tutorial/tutorial_general_400.png)

1. 在 [Workplace by Facebook 設定] 區段中，按一下 [設定 Workplace by Facebook] 可開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/workplacebyfacebook-tutorial/config.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Workplace by Facebook 公司網站。
  
   > [!NOTE] 
   > 為將參數傳遞給 Azure AD，Workplace 可利用大小最多 2.5 KB 的查詢字串，作為 SAML 驗證流程的一部分。

1. 在 [公司儀表板] 中，請移至 [驗證] 索引標籤。

1. 在 [SAML 驗證] 下，從下拉式清單中選取 [僅限 SSO]。

1. 將您從 Azure 入口網站的 [Workplace by Facebook 設定] 區段中複製的值輸入對應的欄位中：

    *   在 [SAML URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [單一登入服務 URL] 的值。
    *   在 [SAML 簽發者 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。
    *   在 [SAML 登出重新導向] \(選擇性) 中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。
    *   在從 Azure 入口網站下載的記事本檔案中開啟您的 **base-64 編碼憑證**，將憑證的內容複製到剪貼簿，再貼到 [SAML 憑證] 文字方塊。

1. 您將需要輸入 [SAML 設定] 區段下所列的對象 URL、收件者 URL 和 ACS (判斷提示取用者服務) URL。

1. 捲動到區段的底部，然後按一下 [測試 SSO] 按鈕。 快顯視窗中的這個結果隨即顯示，並會出現 Azure AD 登入頁面。 如往常輸入您的認證以進行驗證。 

    **疑難排解：** 確保從 Azure AD 傳回的電子郵件地址與您用來登入的 Workplace 帳戶相同。

1. 一旦測試已順利完成後，捲動到頁面底部，然後按一下 [儲存] 按鈕。

1. 現在，使用 Workplace 的所有使用者都將會看到進行驗證的 Azure AD 登入頁面。

1. **SAML 登出重新導向 (選擇性)** - 

    您可以選擇性地選擇設定 SAML 登出 URL，可用來指向 Azure AD 的登出頁面。 當啟用並設定此設定時，就不會再將使用者導向至 Workplace 登出頁面。 相反地，系統會將使用者重新導向至 [SAML 登出重新導向] 設定中所新增的 URL。

### <a name="configuring-reauthentication-frequency"></a>設定重新驗證頻率

您可以將 Workplace 設定為每一天、每三天、每一週、每兩週、每一個月或一律不會提示進行 SAML 檢查。

> [!NOTE] 
>行動應用程式的 SAML 檢查最小值是設定為一週。

您也可以使用下列按鈕來強制重設所有使用者的 SAML：所有使用者現在需要 SAML 驗證。


### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/workplacebyfacebook-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/workplacebyfacebook-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/workplacebyfacebook-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/workplacebyfacebook-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-workplace-by-facebook-test-user"></a>建立 Workplace by Facebook 測試使用者

本節會在 Workplace by Facebook 中建立名為 Britta Simon 的使用者。 Workplace by Facebook 支援預設啟用的 Just-In-Time 佈建。

在這一節沒有您需要進行的動作。 如果使用者不存在於 Workplace by Facebook 中，當您嘗試存取 Workplace by Facebook 時，就會建立一個新的。

>[!Note]
>如果您需要手動建立使用者，請連絡 [Workplace by Facebook 用戶端支援小組](https://workplace.fb.com/faq/)

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Workplace by Facebook 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Workplace by Facebook，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Workplace by Facebook]。

    ![設定單一登入](./media/workplacebyfacebook-tutorial/tutorial_workplacebyfacebook_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

如果要測試您的單一登入設定，請開啟存取面板。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。


## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](workplacebyfacebook-provisioning-tutorial.md)



<!--Image references-->

[1]: ./media/workplacebyfacebook-tutorial/tutorial_general_01.png
[2]: ./media/workplacebyfacebook-tutorial/tutorial_general_02.png
[3]: ./media/workplacebyfacebook-tutorial/tutorial_general_03.png
[4]: ./media/workplacebyfacebook-tutorial/tutorial_general_04.png

[100]: ./media/workplacebyfacebook-tutorial/tutorial_general_100.png

[200]: ./media/workplacebyfacebook-tutorial/tutorial_general_200.png
[201]: ./media/workplacebyfacebook-tutorial/tutorial_general_201.png
[202]: ./media/workplacebyfacebook-tutorial/tutorial_general_202.png
[203]: ./media/workplacebyfacebook-tutorial/tutorial_general_203.png
