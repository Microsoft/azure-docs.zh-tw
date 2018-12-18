---
title: 教學課程：Azure Active Directory 與 Pingboard 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Pingboard 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 28acce3e-22a0-4a37-8b66-6e518d777350
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/15/2018
ms.author: jeedes
ms.openlocfilehash: 794e3f6fe568d76f0687caa36709185f2a538270
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39436095"
---
# <a name="tutorial-azure-active-directory-integration-with-pingboard"></a>教學課程：Azure Active Directory 與 Pingboard 整合

在本教學課程中，您會了解如何整合 Pingboard 與 Azure Active Directory (Azure AD)。

Pingboard 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Pingboard 的人員
- 您可以讓使用者利用自己的 Azure AD 帳戶，來自動登入 Pingboard (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Pingboard 與 Azure AD 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Pingboard 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Pingboard
1. 設定並測試 Azure AD 單一登入

## <a name="adding-pingboard-from-the-gallery"></a>從資源庫新增 Pingboard
若要設定將 Pingboard 整合到 Azure AD 中，您需要從資源庫將 Pingboard 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Pingboard，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式][2]

1. 按一下對話方塊頂端的 [新增] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Pingboard**、從結果面板中選取 [Pingboard]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Pingboard](./media/pingboard-tutorial/tutorial_pingboard_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Pingboard 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Pingboard 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Pingboard 中相關使用者之間的連結關聯性。

建立此連結關聯性的方法是將 Azure AD 中**使用者名稱**的值指定為 Pingboard 中 **Username** 的值。

若要設定及測試與 Pingboard 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Pingboard 測試使用者](#create-a-pingboard-test-user)** - 在 Pingboard 中建立 Britta Simon 的對應項目，且該項目必須與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Pingboard 應用程式中設定單一登入。

**若要使用 Pingboard 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Pingboard] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1.  在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/pingboard-tutorial/tutorial_pingboard_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Pingboard 網域和 URL] 區段上執行下列步驟：

    ![Pingboard 網域與 URL 單一登入資訊 IDP](./media/pingboard-tutorial/tutorial_pingboard_url.png)

    a. 在 [識別碼] 文字方塊中，以下列形式輸入值：`http://app.pingboard.com/sp`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<entity-id>.pingboard.com/auth/saml/consume`

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]：

    ![Pingboard 網域與 URL 單一登入資訊 SP](./media/pingboard-tutorial/tutorial_pingboard_sp_initiated01.png)

     在 [登入 URL] 文字方塊中，以下列模式輸入 URL：`https://<sub-domain>.pingboard.com/sign_in`

    > [!NOTE]
    > 請注意，這些不是真正的值。 使用實際的回覆 URL 與登入 URL 更新這些值。 請連絡 [Pingboard 用戶端支援小組](https://support.pingboard.com/)以取得這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![Pingboard 中繼資料 xml](./media/pingboard-tutorial/tutorial_pingboard_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/pingboard-tutorial/tutorial_general_400.png)

1. 若要在 Pingboard 端上設定 SSO，請開啟新的瀏覽器視窗，然後登入 Pingboard 帳戶。 您必須是 Pingboard 管理員才能設定單一登入。

1. 從頂端功能表中選取 [應用程式] > [整合]

    ![設定單一登入](./media/pingboard-tutorial/Pingboard_integration.png)

1. 在 [整合] 頁面上，尋找 [Azure Active Directory] 圖格，然後按一下它。

    ![Pingboard 單一登入整合](./media/pingboard-tutorial/Pingboard_aad.png)

1. 在接下來的強制回應中按一下 [設定]

    ![Pingboard 設定按鈕](./media/pingboard-tutorial/Pingboard_configure.png)

1. 在下列頁面上，您會注意到「Azure SSO 整合已啟用」。 在記事本中開啟下載的中繼資料 XML 檔案，並且將內容貼上到 **IDP 中繼資料**。

    ![Pingboard SSO 組態畫面](./media/pingboard-tutorial/Pingboard_sso_configure.png)

1. 系統會驗證檔案，如果所有項目都正確，則會立即啟用單一登入。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕](./media/pingboard-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/pingboard-tutorial/create_aaduser_02.png)

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。

    ![[新增] 按鈕](./media/pingboard-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：

    ![[使用者] 對話方塊](./media/pingboard-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-pingboard-test-user"></a>建立 Pingboard 測試使用者

本節目標是在 Pingboard 中建立名為 Britta Simon 的使用者。 Pingboard 支援自動使用者佈建，該功能預設為啟用。 在[這裡](pingboard-provisioning-tutorial.md)可以找到更多關於如何設定自動使用者佈建的詳細資料。

**如果您需要手動建立使用者，請執行下列步驟：**

1. 以系統管理員身分登入您的 Pingboard 公司網站。

1. 按一下 [目錄] 頁面上的 [新增員工] 按鈕。

    ![新增員工](./media/pingboard-tutorial/create_testuser_add.png)

1. 在 [新增員工] 對話方塊頁面上，執行下列步驟：

    ![邀請人員](./media/pingboard-tutorial/create_testuser_name.png)

    a. 在 [全名] 文字方塊中，輸入使用者的全名，例如 **Britta Simon**。

    b. 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件地址，例如 **brittasimon@contoso.com**。

    c. 在 [職稱] 文字方塊中，輸入 Britta Simon 的職稱。

    d. 在 [位置] 下拉式清單中，選取 Britta Simon 的位置。

    e. 按一下 [新增] 。

1. 確認畫面會顯示，供確認新增使用者。

    ![確認](./media/pingboard-tutorial/create_testuser_confirm.png)

    > [!NOTE]
    > Azure Active Directory 帳戶的持有者會收到一封電子郵件，並依照連結在啟用其帳戶前進行確認。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Pingboard 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 Pingboard，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Pingboard]。

    ![應用程式清單中的 Pingboard 連結](./media/pingboard-tutorial/tutorial_pingboard_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

當您在「存取面板」中按一下 Pingboard 圖格時，應該會自動登入您的 Pingboard 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](pingboard-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/pingboard-tutorial/tutorial_general_01.png
[2]: ./media/pingboard-tutorial/tutorial_general_02.png
[3]: ./media/pingboard-tutorial/tutorial_general_03.png
[4]: ./media/pingboard-tutorial/tutorial_general_04.png

[100]: ./media/pingboard-tutorial/tutorial_general_100.png

[200]: ./media/pingboard-tutorial/tutorial_general_200.png
[201]: ./media/pingboard-tutorial/tutorial_general_201.png
[202]: ./media/pingboard-tutorial/tutorial_general_202.png
[203]: ./media/pingboard-tutorial/tutorial_general_203.png