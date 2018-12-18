---
title: 教學課程：Azure Active Directory 與 Vidyard 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Vidyard 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: bed7df23-6e13-4e7c-b4cc-53ed4804664d
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/22/2018
ms.author: jeedes
ms.openlocfilehash: 871942db15d6a3cff45584e33b2191e21d2281a0
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39426450"
---
# <a name="tutorial-azure-active-directory-integration-with-vidyard"></a>教學課程：Azure Active Directory 與 Vidyard 整合

在本教學課程中，您將了解如何整合 Vidyard 與 Azure Active Directory (Azure AD)。

將 Vidyard 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Vidyard 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 Vidyard (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Vidyard 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Vidyard 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Vidyard
1. 設定並測試 Azure AD 單一登入

## <a name="adding-vidyard-from-the-gallery"></a>從資源庫新增 Vidyard
若要設定將 Vidyard 整合到 Azure AD 中，您需要從資源庫將 Vidyard 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Vidyard，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Vidyard**，從結果面板中選取 [Vidyard]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Vidyard](./media/vidyard-tutorial/tutorial_vidyard_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Vidyard 來設定和測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Vidyard 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Vidyard 中的相關使用者之間建立連結關聯性。

若要設定及測試與 Vidyard 搭配運作的 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Vidyard 測試使用者](#create-a-vidyard-test-user)** - 讓 Vidyard 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 Vidyard 應用程式中設定單一登入。

**若要設定與 Vidyard 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Vidyard] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/vidyard-tutorial/tutorial_vidyard_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Vidyard 網域及 URL] 區段上執行下列步驟：

    ![[Vidyard 網域及 URL] 單一登入資訊](./media/vidyard-tutorial/tutorial_vidyard_url2.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://secure.vidyard.com/sso/saml/<unique id>/metadata`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://secure.vidyard.com/sso/saml/<unique id>/consume`

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![[Vidyard 網域及 URL] 單一登入資訊](./media/vidyard-tutorial/tutorial_vidyard_url1.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://secure.vidyard.com/sso/saml/<unique id>/login`

    > [!NOTE]
    > 這些都不是真正的值。 您將會使用實際的「識別碼」、「回覆 URL」及「登入 URL」來更新值，稍後會在本教學課程中說明

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/vidyard-tutorial/tutorial_vidyard_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/vidyard-tutorial/tutorial_general_400.png)

1. 在 [Vidyard 設定] 區段上，按一下 [設定 Vidyard] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![Vidyard 設定](./media/vidyard-tutorial/tutorial_vidyard_configure.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Vidyard Software 公司網站。

1. 從 Vidyard 儀表板中，選取 [Group] \(群組\) > [Security] \(安全性\)

    ![Vidyard 設定](./media/vidyard-tutorial/configure1.png)

1. 按一下 [New Profile] \(新增設定檔\) 索引標籤。

    ![Vidyard 設定](./media/vidyard-tutorial/configure2.png)

1. 在 [SAML Configuration] \(SAML 設定\) 區段中，執行下列步驟：

    ![Vidyard 設定](./media/vidyard-tutorial/configure3.png)

    a. 請在 [Profile Name] \(設定檔名稱\) 文字方塊中，輸入一般設定檔名稱。

    b. 複製 [SSO 使用者登入頁面] 值，然後將其貼到 Azure 入口網站上 [Vidyard 網域及 URL] 區段的 [登入 URL] 文字方塊中。

    c. 複製 [ACS URL] 值，然後將其貼到 Azure 入口網站上 [Vidyard 網域及 URL] 區段的 [回覆 URL] 文字方塊中。

    d. 複製 [簽發者/中繼資料 URL] 值，然後將其貼到 Azure 入口網站上 [Vidyard 網域及 URL] 區段的 [識別碼] 文字方塊中。

    e. 在「記事本」中開啟您從 Azure 入口網站下載的憑證，然後將其貼到 [X.509 Certificate] \(X.509 憑證\) 文字方塊中。

    f. 在 [SAML Endpoint URL] \(SAML 端點 URL\) 文字方塊中，貼上從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    g. 按一下 [確認]。

1. 從 [Single Sign On] \(單一登入\) 索引標籤中，選取現有設定檔旁邊的 [Assign] \(指派\)

    ![Vidyard 設定](./media/vidyard-tutorial/configure4.png)

    > [!NOTE]
    > 建立 SSO 設定檔之後，請將它指派給使用者將透過 Azure 要求存取權的任何群組。 如果使用者不在他們被指派的群組內，Vidyard 將會自動建立使用者帳戶，並即時指派其角色。

1. 選取您的組織群組，此群組會顯示在 [Groups Available to Assign] \(可指派的群組\) 中。

    ![Vidyard 設定](./media/vidyard-tutorial/configure5.png)

1. 您可以在 [Groups Currently Assigned] \(目前指派的群組\) 底下看到所指派的群組。 根據您的組織選取群組的角色，然後按一下 [Confirm] \(確認\)。

    ![Vidyard 設定](./media/vidyard-tutorial/configure6.png)

    > [!NOTE]
    > 如需詳細資訊，請參閱[此文件](https://knowledge.vidyard.com/saml-single-sign-on-authentication/saml-based-single-sign-on-sso-in-vidyard)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/vidyard-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/vidyard-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/vidyard-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/vidyard-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-vidyard-test-user"></a>建立 Vidyard 測試使用者

本節的目標是要在 Vidyard 中建立名為 Britta Simon 的使用者。 Vidyard 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 嘗試存取 Vidyard 時，如果使用者還不存在，就會建立新使用者。
>[!Note]
>如果您需要手動建立使用者，請連絡 [Vidyard 支援小組](mailto:support@vidyard.com)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Vidyard 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Vidyard，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Vidyard]。

    ![應用程式清單中的 Vidyard 連結](./media/vidyard-tutorial/tutorial_vidyard_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 Vidyard 圖格時，應該就會自動登入 Vidyard 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/vidyard-tutorial/tutorial_general_01.png
[2]: ./media/vidyard-tutorial/tutorial_general_02.png
[3]: ./media/vidyard-tutorial/tutorial_general_03.png
[4]: ./media/vidyard-tutorial/tutorial_general_04.png

[100]: ./media/vidyard-tutorial/tutorial_general_100.png

[200]: ./media/vidyard-tutorial/tutorial_general_200.png
[201]: ./media/vidyard-tutorial/tutorial_general_201.png
[202]: ./media/vidyard-tutorial/tutorial_general_202.png
[203]: ./media/vidyard-tutorial/tutorial_general_203.png

