---
title: 教學課程：Azure Active Directory 與 Riskware 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Riskware 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 81866167-b163-4695-8978-fd29a25dac7a
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/27/2018
ms.author: jeedes
ms.openlocfilehash: 4c664fac99e93e94b46f5d917a63aa6530b695bd
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39437768"
---
# <a name="tutorial-azure-active-directory-integration-with-riskware"></a>教學課程：Azure Active Directory 與 Riskware 整合

在本教學課程中，您將了解如何整合 Riskware 與 Azure Active Directory (Azure AD)。

將 Riskware 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Riskware 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 Riskware (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Riskware 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Riskware 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Riskware
1. 設定並測試 Azure AD 單一登入

## <a name="adding-riskware-from-the-gallery"></a>從資源庫新增 Riskware
若要設定將 Riskware 整合到 Azure AD 中，您需要從資源庫將 Riskware 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Riskware，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Riskware**，從結果面板中選取 [Riskware]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Riskware](./media/riskware-tutorial/tutorial_riskware_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Riskware 來設定和測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Riskware 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Riskware 中的相關使用者之間建立連結關聯性。

若要設定及測試與 Riskware 搭配運作的 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Riskware 測試使用者](#create-a-riskware-test-user)** - 讓 Riskware 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 Riskware 應用程式中設定單一登入。

**若要設定與 Riskware 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Riskware] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/riskware-tutorial/tutorial_riskware_samlbase.png)

1. 在 [Riskware 網域及 URL] 區段上，執行下列步驟：

    ![[Riskware 網域及 URL] 單一登入資訊](./media/riskware-tutorial/tutorial_riskware_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰
    | 環境| URL 模式|
    |--|--|
    | UAT|  `https://riskcloud.net/uat?ccode=<COMPANYCODE>` |
    | PROD| `https://riskcloud.net/prod?ccode=<COMPANYCODE>` |
    | 示範| `https://riskcloud.net/demo?ccode=<COMPANYCODE>` |
    |||

    b. 在 [識別碼 (實體識別碼)] 文字方塊中，輸入 URL：
    | 環境| URL 模式|
    |--|--|
    | UAT| `https://riskcloud.net/uat` |
    | PROD| `https://riskcloud.net/prod` |
    | 示範| `https://riskcloud.net/demo` |
    |||

    > [!NOTE]
    > [登入 URL] 的值不是真正的值。 請使用實際的「登入 URL」來更新此值。 請連絡 [Riskware 用戶端支援小組](mailto:support@pansoftware.com.au)以取得此值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/riskware-tutorial/tutorial_riskware_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/riskware-tutorial/tutorial_general_400.png)

1. 在 [Riskware 設定] 區段上，按一下 [設定 Riskware] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL 和 SAML 單一登入服務 URL]。

    ![Riskware 設定](./media/riskware-tutorial/tutorial_riskware_configure.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Riskware 公司網站。

1. 按一下右上方的 [Maintenance] \(維護\) 以開啟維護頁面。

    ![Riskware 設定：維護](./media/riskware-tutorial/tutorial_riskware_maintain.png)

1. 在維護頁面中，按一下 [Authentication] \(驗證\)。

    ![Riskware 設定：驗證](./media/riskware-tutorial/tutorial_riskware_authen.png)

1. 在 [Authentication Configuration] \(驗證設定\) 頁面中，執行下列步驟：

    ![Riskware 設定：驗證設定](./media/riskware-tutorial/tutorial_riskware_config.png)

    a. 選取 [SAML] 作為驗證的 [Type] \(類型\)。

    b. 在 [Code]\(代碼\) 文字方塊中，輸入您的代碼，例如 AZURE_UAT。

    c. 在 [Description]\(描述\) 文字方塊中，輸入您的描述，例如 AZURE Configuration for SSO (AZURE 的 SSO 設定)。

    d. 在 [單一登入頁面] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    e. 在 [登出頁面] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    f. 在 [POST 表單欄位] 文字方塊中，輸入 POST 回應中包含 SAML 的欄位名稱，例如 SAMLResponse

    g. 在 [XML Identity Tag Name] \(XML 身分識別標記名稱\) 文字方塊中，輸入 SAML 回應中包含唯一識別碼的屬性，例如 NameID。

    h. 在記事本中開啟從 Azure 入口網站下載的 **Metadata Xml**，複製中繼資料檔中的憑證，然後將其貼到 [Certificate]\(憑證\) 文字方塊中

    i. 在 [Consumer URL]\(取用者 URL\) 文字方塊中，貼上您向支援小組取得的**回覆 URL** 值。

    j. 在 [簽發者] 文字方塊中，貼上您向支援小組取得的**識別碼**值。

    > [!Note]
    > 請連絡 [Riskware 用戶端支援小組](mailto:support@pansoftware.com.au)以取得這些值

    k. 勾選 [使用 POST] 核取方塊。

    l. 勾選 [使用 SAML 要求] 核取方塊。

    m. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/riskware-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/riskware-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/riskware-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/riskware-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-riskware-test-user"></a>建立 Riskware 測試使用者

若要讓 Azure AD 使用者能夠登入 Riskware，必須將這些使用者佈建到 Riskware。 在 Riskware 中，需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以安全性系統管理員身分登入 Riskware。

1. 按一下右上方的 [Maintenance] \(維護\) 以開啟維護頁面。 

    ![Riskware 設定：維護](./media/riskware-tutorial/tutorial_riskware_maintain.png)

1. 在維護頁面中，按一下 [People]\(人員\)。

    ![Riskware 設定：人員](./media/riskware-tutorial/tutorial_riskware_people.png)

1. 選取 [詳細資料] 索引標籤，然後執行下列步驟：

    ![Riskware 設定：詳細資料](./media/riskware-tutorial/tutorial_riskware_details.png)

    a. 選取 [Person Type]\(人員類型\)，例如 [Employee] \(員工\)。

    b. 在 [First Name]\(名字\) 文字方塊中，輸入使用者的名字，例如 **Britta**。

    c. 在 [Surname]\(姓氏\) 文字方塊中，輸入使用者的姓氏，例如 **Simon**。

1. 在 [安全性]  索引標籤上執行下列步驟：。

    ![Riskware 設定：安全性](./media/riskware-tutorial/tutorial_riskware_security.png)

    a. 在 [Authentication]\(驗證\) 區段下，選取您已設定的 [Authentication]\(驗證\) 模式，例如 [AZURE Configuration for SSO] \(AZURE 的 SSO 設定\)。

    b. 在 [Logon Details]\(登入詳細資料\) 區段下的 [User ID] \(使用者識別碼\) 文字方塊中，輸入使用者的電子郵件，例如 **brittasimon@contoso.com**。

    c. 在 [Password] \(密碼\) 文字方塊中，輸入使用者的密碼。

1. 在 [組織] 索引標籤上，執行下列步驟：

    ![Riskware 設定：組織](./media/riskware-tutorial/tutorial_riskware_org.png)

    a. 選取選項作為 **Level1** 組織。

    b. 在 [Person's Primary Workplace]\(人員的主要工作場所\) 區段下的 [Location]\(位置\) 文字方塊中，輸入您的位置。

    c. 在 [Employee]\(員工\) 區段下，選取 [Employee Status]\(員工狀態\)，例如 [Casual]\(臨時\)。

1. 按一下 [檔案] 。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Riskware 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Riskware，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Riskware]。

    ![應用程式清單中的 Riskware 連結](./media/riskware-tutorial/tutorial_riskware_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Riskware] 圖格時，應該就會自動登入 Riskware 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/riskware-tutorial/tutorial_general_01.png
[2]: ./media/riskware-tutorial/tutorial_general_02.png
[3]: ./media/riskware-tutorial/tutorial_general_03.png
[4]: ./media/riskware-tutorial/tutorial_general_04.png

[100]: ./media/riskware-tutorial/tutorial_general_100.png

[200]: ./media/riskware-tutorial/tutorial_general_200.png
[201]: ./media/riskware-tutorial/tutorial_general_201.png
[202]: ./media/riskware-tutorial/tutorial_general_202.png
[203]: ./media/riskware-tutorial/tutorial_general_203.png

