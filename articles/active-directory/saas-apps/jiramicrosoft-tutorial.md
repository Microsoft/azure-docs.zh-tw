---
title: 教學課程：Azure Active Directory 與 JIRA SAML SSO by Microsoft 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 JIRA SAML SSO by Microsoft 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 4b663047-7f88-443b-97bd-54224b232815
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/30/2018
ms.author: jeedes
ms.openlocfilehash: cc87985404ef8c9ee625f32b359e6ac1a29e73ae
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140900"
---
# <a name="tutorial-azure-active-directory-integration-with-jira-saml-sso-by-microsoft"></a>教學課程：Azure Active Directory 與 JIRA SAML SSO by Microsoft 整合

在本教學課程中，您會了解如何整合 JIRA SAML SSO by Microsoft 與 Azure Active Directory (Azure AD)。

JIRA SAML SSO by Microsoft 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 JIRA SAML SSO by Microsoft 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 JIRA SAML SSO by Microsoft (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="description"></a>說明

使用 Microsoft Azure Active Directory 帳戶搭配 Atlassian JIRA 伺服器來啟用單一登入。 如此一來，所有組織使用者即可使用 Azure AD 認證來登入 JIRA 應用程式。 此外掛程式使用 SAML 2.0 來進行同盟。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 JIRA SAML SSO by Microsoft 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 應該在 Windows 64 位元版本上安裝及設定 JIRA Core 和 JIRA Software 6.0 到 7.8 或 JIRA Service Desk 3.0 到 3.2
- JIRA 伺服器已啟用 HTTPS
- 請注意下一節提及支援的 JIRA 外掛程式版本。
- JIRA 伺服器可從網際網路連上，特別是連線至 Azure AD 登入頁面來進行驗證，且應該能夠接收來自 Azure AD 的權杖
- JIRA 中已設定管理員認證
- JIRA 中已停用 WebSudo
- JIRA 伺服器應用程式中建立的測試使用者

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用 JIRA 的生產環境。 先在應用程式的開發或預備環境中測試整合，然後使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在這裡取得一個月試用：[試用供應項目](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="supported-versions-of-jira"></a>支援的 JIRA 版本

*   JIRA Core 和 JIRA Software：6.0 到 7.8
*   JIRA Service Desk 3.0 到 3.2
*   JIRA 也支援 5.2。 如需詳細資訊，請按一下[適用於 JIRA 5.2 的 Microsoft Azure Active Directory 單一登入](jira52microsoft-tutorial.md)

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 JIRA SAML SSO by Microsoft
2. 設定並測試 Azure AD 單一登入

## <a name="adding-jira-saml-sso-by-microsoft-from-the-gallery"></a>從資源庫新增 JIRA SAML SSO by Microsoft
如要設定將 JIRA SAML SSO by Microsoft 整合到 Azure AD 中，您需要從資源庫將 JIRA SAML SSO by Microsoft 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 JIRA SAML SSO by Microsoft，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **JIRA SAML SSO by Microsoft**，從結果面板中選取 [JIRA SAML SSO by Microsoft]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 JIRA SAML SSO by Microsoft](./media/jiramicrosoft-tutorial/tutorial_singlesign-onforjira_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 JIRA SAML SSO by Microsoft 來設定和測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 JIRA SAML SSO by Microsoft 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 JIRA SAML SSO by Microsoft 中的相關使用者之間，建立連結關聯性。

若要設定及測試與 JIRA SAML SSO by Microsoft 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 JIRA SAML SSO by Microsoft 測試使用者](#create-a-jira-saml-sso-by-microsoft-test-user)** - 讓 JIRA SAML SSO by Microsoft 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 JIRA SAML SSO by Microsoft 應用程式中設定單一登入。

**若要設定與 JIRA SAML SSO by Microsoft 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [JIRA SAML SSO by Microsoft] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/jiramicrosoft-tutorial/tutorial_singlesign-onforjira_samlbase.png)

3. 在 [JIRA SAML SSO by Microsoft 網域及 URL] 區段中，執行下列步驟：

    ![JIRA SAML SSO by Microsoft 網域及 URL 單一登入資訊](./media/jiramicrosoft-tutorial/tutorial_singlesign-onforjira_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<domain:port>/plugins/servlet/saml/auth`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<domain:port>/`

    c. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<domain:port>/plugins/servlet/saml/auth`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 如果連接埠是具名 URL，則為選擇性。 在設定 Jira 外掛程式 (本教學課程稍後會說明) 期間會收到這些值。

4. 在 [SAML 簽署憑證] 區段中，按一下「複製」按鈕複製「應用程式同盟中繼資料 URL」，並將它貼到 [記事本]。

    ![設定單一登入](./media/jiramicrosoft-tutorial/tutorial_metadataurl.png)

5. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/jiramicrosoft-tutorial/tutorial_general_400.png)

6. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 JIRA 執行個體。

7. 將滑鼠停留在 cog 上，然後按一下 [附加元件]。

    ![設定單一登入](./media/jiramicrosoft-tutorial/addon1.png)

8. 從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=56506)下載外掛程式。 使用 [上傳附加元件] 功能表來手動上傳 Microsoft 所提供的外掛程式。 外掛程式下載受到 [Microsoft 服務合約](https://www.microsoft.com/servicesagreement/)的規範。

    ![設定單一登入](./media/jiramicrosoft-tutorial/addon12.png)

9. 安裝外掛程式之後，它會出現在 [管理附加元件] 區段的 [使用者安裝的附加元件] 區段中。 按一下 [設定] 來設定新的外掛程式。

    ![設定單一登入](./media/jiramicrosoft-tutorial/addon13.png)

10. 在設定頁面上執行下列步驟：

    ![設定單一登入](./media/jiramicrosoft-tutorial/addon52.png)

    > [!TIP]
    > 請確定只有一個對應至應用程式的憑證，解析中繼資料時就不會發生錯誤。 如果有多個憑證，則解析中繼資料時，管理員會遇到錯誤。

    a. 在 [中繼資料 URL] 文字方塊中，貼上您從 Azure 入口網站複製的**應用程式同盟中繼資料 Url**值，然後按一下 [解析] 按鈕。 這樣會讀取 IdP 中繼資料 URL 並填入所有欄位資訊。

    b. 複製 [識別碼]、[回覆 URL] 和 [登入 URL] 值，然後在 Azure 入口網站的 [JIRA SAML SSO by Microsoft 網域及 URL] 中，分別貼到 [識別碼]、[回覆 URL] 和 [登入 URL] 文字方塊。

    c. 在 [登入按鈕名稱] 中，輸入您的組織要讓使用者在登入畫面上看到的按鈕名稱。

    d. 在 [SAML 使用者識別碼位置] 中，選取 [使用者識別碼在 Subject 陳述式的 NameIdentifier 元素中] 或 [使用者識別碼在 Attribute 元素中]。  此識別碼必須為 JIRA 使用者識別碼。如果使用者識別碼不符，系統將不會允許使用者登入。

    > [!Note]
    > 預設 SAML 使用者識別碼位置是名稱識別碼。 您可以將它變更為屬性選項，並輸入適當的屬性名稱。

    e. 如果您選取 [使用者識別碼在 Attribute 元素中] 選項，請在 [屬性名稱] 文字方塊中，輸入需要使用者識別碼的屬性名稱。

    f. 如果您使用同盟網域 (例如 ADFS 等) 搭配 Azure AD，請按一下 [啟用主領域探索] 選項，並設定 [網域名稱]。

    g. 在 [網域名稱] 中，如果是使用以 ADFS 為基礎的登入，請在此輸入網域名稱。

    h. 如果您想要在使用者登出 JIRA 時登出 Azure AD，請勾選 [啟用單一登出]。

    i. 按一下 [儲存] 按鈕以儲存設定。

    > [!NOTE]
    > 如需有關安裝和疑難排解的詳細資訊，請瀏覽 [MS JIRA SSO 連接器管理指南](../ms-confluence-jira-plugin-adminguide.md)，另外也有[常見問題集](../ms-confluence-jira-plugin-faq.md)可協助您

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/jiramicrosoft-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/jiramicrosoft-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/jiramicrosoft-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/jiramicrosoft-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-jira-saml-sso-by-microsoft-test-user"></a>建立 JIRA SAML SSO by Microsoft 測試使用者

若要讓 Azure AD 使用者能夠登入 JIRA 內部部署伺服器，必須將他們佈建到 JIRA SAML SSO by Microsoft。 對於 JIRA SAML SSO by Microsoft，佈建是手動工作。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 JIRA 內部部署伺服器。

2. 將滑鼠停留在 cog 上，然後按一下 [使用者管理]。

    ![新增員工](./media/jiramicrosoft-tutorial/user1.png)

3. 系統會將您重新導向至 [系統管理員存取] 頁面，以輸入**密碼**，然後按一下 [確認] 按鈕。

    ![新增員工](./media/jiramicrosoft-tutorial/user2.png)

4. 在 [使用者管理] 索引標籤區段下，按一下 [建立使用者]。

    ![新增員工](./media/jiramicrosoft-tutorial/user3.png) 

5. 在 [建立新的使用者] 對話方塊中，執行下列步驟：

    ![新增員工](./media/jiramicrosoft-tutorial/user4.png) 

    a. 在 [電子郵件地址] 文字方塊中，輸入像是 Brittasimon@contoso.com 的使用者電子郵件地址。

    b. 在 [全名] 文字方塊中，輸入像是 Britta Simon 的使用者全名。

    c. 在 [使用者名稱] 文字方塊中，輸入像是 Brittasimon@contoso.com 的使用者電子郵件。

    d. 在 [密碼] 文字方塊中，輸入使用者的密碼。

    e. 按一下 [建立使用者]。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 JIRA SAML SSO by Microsoft 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 JIRA SAML SSO by Microsoft，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [JIRA SAML SSO by Microsoft]。

    ![應用程式清單中的 JIRA SAML SSO by Microsoft 連結](./media/jiramicrosoft-tutorial/tutorial_singlesign-onforjira_app.png)

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [JIRA SAML SSO by Microsoft] 圖格時，應該會自動登入您的 JIRA SAML SSO by Microsoft 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/msaadssojira-tutorial/tutorial_general_01.png
[2]: ./media/msaadssojira-tutorial/tutorial_general_02.png
[3]: ./media/msaadssojira-tutorial/tutorial_general_03.png
[4]: ./media/msaadssojira-tutorial/tutorial_general_04.png

[100]: ./media/msaadssojira-tutorial/tutorial_general_100.png

[200]: ./media/msaadssojira-tutorial\tutorial_general_200.png
[201]: ./media/msaadssojira-tutorial\tutorial_general_201.png
[202]: ./media/msaadssojira-tutorial\tutorial_general_202.png
[203]: ./media/msaadssojira-tutorial\tutorial_general_203.png