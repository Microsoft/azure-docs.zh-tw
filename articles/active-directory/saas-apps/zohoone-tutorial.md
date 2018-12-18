---
title: 教學課程：Azure Active Directory 與 Zoho One 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Zoho One 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: bbc3038c-0d8b-45dd-9645-368bd3d01a0f
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/25/2018
ms.author: jeedes
ms.openlocfilehash: 81e86df270a7286426363c26a0e8a87b99082428
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39438268"
---
# <a name="tutorial-azure-active-directory-integration-with-zoho-one"></a>教學課程：Azure Active Directory 與 Zoho One 整合

在本教學課程中，您將了解如何整合 Zoho One 與 Azure Active Directory (Azure AD)。

將 Zoho One 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Zoho One 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 Zoho One (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Zoho One 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Zoho One 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Zoho One
1. 設定並測試 Azure AD 單一登入

## <a name="adding-zoho-one-from-the-gallery"></a>從資源庫新增 Zoho One
若要設定將 Zoho One 整合到 Azure AD 中，您需要從資源庫將 Zoho One 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Zoho One，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Zoho One**，從結果面板中選取 [Zoho One]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Zoho One](./media/zohoone-tutorial/tutorial_zohoone_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Zoho One 來設定和測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Zoho One 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Zoho One 中的相關使用者之間建立連結關聯性。

若要設定及測試與 Zoho One 搭配運作的 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Zoho One 測試使用者](#create-a-zoho-one-test-user)** - 讓 Zoho One 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 Zoho One 應用程式中設定單一登入。

**若要設定與 Zoho One 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Zoho One] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/zohoone-tutorial/tutorial_zohoone_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Zoho One 網域及 URL] 區段上執行下列步驟：

    ![[Zoho One 網域及 URL] 單一登入資訊](./media/zohoone-tutorial/tutorial_zohoone_url.png)

    a. 在 [識別碼 (實體識別碼)] 文字方塊中，輸入 URL：`one.zoho.com`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://accounts.zoho.com/samlresponse/<saml-identifier>`

    c. 按一下 [顯示進階 URL 設定]。

    d. 在 [回覆狀態] 文字方塊中，輸入 URL：`https://one.zoho.com`

1. 如果您想要以 **SP** 起始模式設定應用程式，請執行下列步驟：

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://accounts.zoho.com/samlauthrequest/<domain_name>?serviceurl=https://one.zoho.com`
     
    > [!NOTE] 
    > 上述 [回覆 URL] 和 [登入 URL] 值並非真正的值。 您將會使用實際的「回覆 URL」與「登入 URL」來更新值，稍後會在本教學課程中說明。 

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/zohoone-tutorial/tutorial_zohoone_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/zohoone-tutorial/tutorial_general_400.png)
    
1. 在 [Zoho One 組態] 區段上，按一下 [設定 Zoho One] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL 和 SAML 單一登入服務 URL]。

    ![Zoho One 設定](./media/zohoone-tutorial/tutorial_zohoone_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Zoho One 公司網站。

1. 在 [Organization]\(組織\) 索引標籤上，按一下 [SAML Authentication]\(SAML 驗證\) 下的 [Setup]\(設定\)。

    ![Zoho One：組織](./media/zohoone-tutorial/tutorial_zohoone_setup.png)

1. 在快顯頁面上，執行下列步驟：

    ![Zoho One：登入](./media/zohoone-tutorial/tutorial_zohoone_save.png)

    a. 在 [Sign-in URL] \(登入 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    b. 在 [Sign-out URL] \(登出 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    c. 按一下 [Browse]\(瀏覽\) 以上傳您從 Azure 入口網站下載的**憑證 (Base64)**。

    d. 按一下 [檔案] 。

1. 儲存 [SAML 驗證] 設定之後，複製 [SAML 識別碼] 值，然後在 Azure 入口網站上 [Zoho One 網域及 URL] 區段底下的 [回覆 URL] 中使用此值。

    ![Zoho One：SAML](./media/zohoone-tutorial/tutorial_zohoone_samlidenti.png)

1. 移至 [網域] 索引標籤，然後按一下 [新增網域]。

    ![Zoho One：網域](./media/zohoone-tutorial/tutorial_zohoone_domain.png)

1. 在 [新增網域] 頁面上，執行下列步驟：

    ![Zoho One：新增網域](./media/zohoone-tutorial/tutorial_zohoone_adddomain.png)

    a. 在 [網域名稱] 文字方塊中，輸入網域，例如 **contoso.com**。

    b. 按一下 [新增] 。

    >[!Note]
    >新增網域之後，請依照[這些](https://www.zoho.com/one/help/admin-guide/domain-verification.html)步驟來確認您的網域。 確認網域之後，請在 Azure 入口網站上 [Zoho One 網域及 URL] 的 [登入 URL] 中使用您的網域名稱。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/zohoone-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/zohoone-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/zohoone-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/zohoone-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-zoho-one-test-user"></a>建立 Zoho One 測試使用者

若要讓 Azure AD 使用者能夠登入 Zoho One，必須將這些使用者佈建到 Zoho One。 在 Zoho One 中，需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以安全性系統管理員身分登入 Zoho One。

1. 在 [Users]\(使用者\) 索引標籤上，按一下**使用者標誌**。

    ![Zoho One：使用者](./media/zohoone-tutorial/tutorial_zohoone_users.png)

1. 在 [新增使用者] 頁面上，執行下列步驟：

    ![Zoho One：新增使用者](./media/zohoone-tutorial/tutorial_zohoone_adduser.png)
    
    a. 在 [Name] \(名稱\) 文字方塊中，輸入使用者的名稱，例如 **Britta Simon**。
    
    b. 在 [電子郵件地址] 文字方塊中，輸入使用者的電子郵件，例如 **brittasimon@contoso.com**。

    >[!Note]
    >從網域清單中選取已驗證的網域。

    c. 按一下 [新增] 。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Zoho One 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Zoho One，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Zoho One]。

    ![應用程式清單中的 Zoho One 連結](./media/zohoone-tutorial/tutorial_zohoone_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Zoho One] 圖格時，應該就會自動登入 Zoho One 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/zohoone-tutorial/tutorial_general_01.png
[2]: ./media/zohoone-tutorial/tutorial_general_02.png
[3]: ./media/zohoone-tutorial/tutorial_general_03.png
[4]: ./media/zohoone-tutorial/tutorial_general_04.png

[100]: ./media/zohoone-tutorial/tutorial_general_100.png

[200]: ./media/zohoone-tutorial/tutorial_general_200.png
[201]: ./media/zohoone-tutorial/tutorial_general_201.png
[202]: ./media/zohoone-tutorial/tutorial_general_202.png
[203]: ./media/zohoone-tutorial/tutorial_general_203.png

