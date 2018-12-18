---
title: 教學課程：Azure Active Directory 與 Adobe Sign 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Adobe Sign 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: f9385723-8fe7-4340-8afb-1508dac3e92b
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/26/2018
ms.author: jeedes
ms.openlocfilehash: d5cdc2ec0c6cfcf52f84629485d0dd879fbf6fa2
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39053993"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-sign"></a>教學課程：Azure Active Directory 與 Adobe Sign 整合

在本教學課程中，您會了解如何將 Adobe Sign 與 Azure Active Directory (Azure AD) 整合。

將 Adobe Sign 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Adobe Sign 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 Adobe Sign (單一登入)。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需有關 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Adobe Sign 整合，您需要：

- Azure AD 訂用帳戶
- 已啟用 Adobe Sign 單一登入功能的訂用帳戶

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Adobe Sign。
2. 設定並測試 Azure AD 單一登入。

## <a name="add-adobe-sign-from-the-gallery"></a>從資源庫新增 Adobe Sign
若要設定將 Adobe Sign 整合到 Azure AD 中，您需要從資源庫將 Adobe Sign 新增到受控 SaaS 應用程式清單。

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 圖示的螢幕擷取畫面][1]

2. 瀏覽至 [企業應用程式] > [所有應用程式]。

    ![Azure Active Directory 功能表的螢幕擷取畫面，其中已醒目提示 [企業應用程式] 和 [所有應用程式]][2]
    
3. 若要新增應用程式，請選取對話方塊頂端的 [新增應用程式]。

    ![對話方塊頂端 [新增應用程式] 的螢幕擷取畫面][3]

4. 在搜尋方塊中，輸入 **Adobe Sign**。

    ![搜尋方塊的螢幕擷取畫面](./media/adobe-echosign-tutorial/tutorial_adobesign_search.png)

5. 在結果面板中，選取 [Adobe Sign]，然後選取 [新增]。

    ![結果面板的螢幕擷取畫面](./media/adobe-echosign-tutorial/tutorial_adobesign_addfromgallery.png)

##  <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Adobe Sign 來設定和測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須能夠辨識 Azure AD 使用者與 Adobe Sign 中相關使用者之間的連結關聯性。

若要建立連結關聯性，請在 Adobe Sign 中，指派 Azure AD 中 [使用者名稱] 的值作為 [Username] 的值。

若要設定及測試與 Adobe Sign 搭配運作的 Azure AD 單一登入，請完成下列基本工作：

1. [設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)，讓您的使用者能夠使用此功能。
2. [建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
3. [建立 Adobe Sign 測試使用者](#creating-an-adobe-sign-test-user) - 讓 Adobe Sign 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
4. [指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. [測試單一登入](#testing-single-sign-on) - 驗證設定是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 Adobe Sign 應用程式中設定單一登入。

1. 在 Azure 入口網站的 [Adobe Sign] 應用程式整合頁面上，選取 [單一登入]。

    ![Adobe Sign 應用程式整合頁面的螢幕擷取畫面，其中已醒目提示 [單一登入]][4]

2. 在 [單一登入] 對話方塊上，選取 [SAML 登入] 作為 [模式]，以啟用單一登入。

    ![[單一登入] 對話方塊的螢幕擷取畫面，其中已醒目提示 [模式] 方塊](./media/adobe-echosign-tutorial/tutorial_adobesign_samlbase.png)

3. 在 [Adobe Sign 網域及 URL] 區段中，執行下列步驟：

    ![[Adobe Sign 網域及 URL] 區段的螢幕擷取畫面](./media/adobe-echosign-tutorial/tutorial_adobesign_url.png)

    a. 在 [登入 URL] 文字方塊中，以下列模式輸入 URL︰`https://<companyname>.echosign.com/`

    b. 在 [識別碼] 文字方塊中，以下列模式輸入 URL：`https://<companyname>.echosign.com`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Adobe Sign 用戶端支援小組](https://helpx.adobe.com/in/contact/support.html)以取得這些值。

4. 在 [SAML 簽署憑證] 區段中，選取 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![[SAML 簽署憑證] 區段的螢幕擷取畫面](./media/adobe-echosign-tutorial/tutorial_adobesign_certificate.png) 

5. 選取 [ **儲存**]。

    ![[儲存] 按鈕的螢幕擷取畫面](./media/adobe-echosign-tutorial/tutorial_general_400.png)

6. 在 [Adobe Sign 設定] 區段上，選取 [設定 Adobe Sign] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL]、[SAML 實體識別碼]，以及 [SAML 單一登入服務 URL]。

    ![[Adobe Sign 設定] 區段的螢幕擷取畫面，其中已醒目提示 [設定 Adobe Sign]](./media/adobe-echosign-tutorial/tutorial_adobesign_configure.png)

7. 設定之前，請先連絡 [Adobe Sign Client 支援小組](https://helpx.adobe.com/in/contact/support.html)，以將您的網域列入 Adobe Sign 的白名單中。 以下是新增網域的方式：

    a. [Adobe Sign Client 支援小組](https://helpx.adobe.com/in/contact/support.html)會傳送隨機產生的權杖給您。 您網域的權杖會如下所示：**adobe-sign-verification= xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx**

    b. 在 DNS 文字記錄中發佈驗證權杖，然後通知 [Adobe Sign Client 支援小組](https://helpx.adobe.com/in/contact/support.html)。
    
    > [!NOTE]
    > 這可能需要幾天或更長的時間。 請注意，DNS 傳播延遲意謂著在 DNS 中發佈的值可能長達一或多個小時無法顯示。 您的 IT 系統管理員應該知道如何在 DNS 文字記錄中發佈此權杖。
    
    c. 當您於發佈權杖後透過支援票證通知 [Adobe Sign Client 支援小組](https://helpx.adobe.com/in/contact/support.html)時，他們會驗證該網域並將它新增至您的帳戶。
    
    d. 通常在 DNS 記錄上發佈權杖的步驟如下：

    * 登入您的網域帳戶
    * 尋找用於更新 DNS 記錄的頁面。 此頁面可能名為 [DNS 管理]、[名稱伺服器管理] 或 [進階設定]。
    * 尋找您網域的 TXT 記錄。
    * 以 Adobe 提供的完整權杖值新增 TXT 記錄。
    * 儲存您的變更。

8. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Adobe Sign 公司網站。

9. 在 [SAML] 功能表中，選取 [Account Settings] \(帳戶設定\) > [SAML Settings] \(SAML 設定\)。
   
    ![Adobe Sign [SAML Settings] \(SAML 設定\) 頁面的螢幕擷取畫面](./media/adobe-echosign-tutorial/ic789520.png "Account (\帳戶\)")

10. 在 [SAML 設定] 區段中，執行下列步驟：
  
    ![[SAML Settings] \(SAML 設定\) 的螢幕擷取畫面](./media/adobe-echosign-tutorial/ic789521.png "SAML Settings \(SAML 設定\)")
   
    a. 在 [SAML Mode] \(SAML 模式\) 底下，選取 [SAML Mandatory] \(SAML 強制\)。
   
    b. 選取 [Allow Echosign Account Administrators to log in using their Echosign Credentials] \(允許 EchoSign 帳戶管理員使用其 EchoSign 認證登入\)。
   
    c. 在 [User Creation] \(使用者建立\) 底下，選取 [Automatically add users authenticated through SAML] \(透過 SAML 自動新增已驗證的使用者\)。

    d. 將您從 Azure 入口網站複製的 [SAML 實體識別碼] 貼到 [Entity ID/Issuer URL] \(實體識別碼/簽發者 URL\) 文字方塊中。
    
    e. 將您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 貼到 [Login URL/SSO Endpoint] 文字方塊中。
   
    f. 將您從 Azure 入口網站複製的 [登出 URL] 貼到 [Logout URL/SLO Endpoint] \(登出 URL/SLO 端點\) 文字方塊中。

    g. 在「記事本」中開啟所下載的**憑證 (Base64)** 檔案。 將其內容複製到剪貼簿，然後貼到 [IdP Certificate] \(IdP 憑證\) 文字方塊中。

    h. 選取 [Save Changes] \(儲存變更\)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![Azure 入口網站測試使用者名稱的螢幕擷取畫面][100]

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 圖示。

    ![Azure AD 圖示的螢幕擷取畫面](./media/adobe-echosign-tutorial/create_aaduser_01.png) 

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後選取 [所有使用者]。
    
    ![Azure AD 功能表的螢幕擷取畫面，其中已醒目提示 [使用者和群組] 及 [所有使用者]](./media/adobe-echosign-tutorial/create_aaduser_02.png) 

3. 若要開啟 [使用者] 對話方塊，請選取 [新增]。
 
    ![[所有使用者] 的螢幕擷取畫面，其中已醒目提示 [新增] 選項](./media/adobe-echosign-tutorial/create_aaduser_03.png) 

4. 在 [使用者] 對話方塊上，執行下列步驟：
 
    ![[使用者] 對話方塊的螢幕擷取畫面](./media/adobe-echosign-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的電子郵件地址。

    c. 選取 [顯示密碼]，然後記下 [密碼] 的值。

    d. 選取 [建立] 。
 
### <a name="create-an-adobe-sign-test-user"></a>建立 Adobe Sign 測試使用者

若要讓 Azure AD 使用者能夠登入 Adobe Sign，必須將這些使用者佈建到 Adobe Sign。 這是手動工作。

>[!NOTE]
>您可以使用任何其他 Adobe Sign 使用者帳戶建立工具或 Adobe Sign 提供的 API，來佈建 Azure AD 使用者帳戶。 

1. 以系統管理員身分登入您的 **Adobe Sign** 公司網站。

2. 在頂端的功能表中，選取 [Account] \(帳戶\)。 接著，在左側窗格中，選取 [Users & Groups] \(使用者和群組\) > [Create a new user] \(建立新使用者\)。
   
    ![Adobe Sign 公司網站的螢幕擷取畫面，其中已醒目提示 [Account] \(帳戶\)、[Users &Groups] \(使用者和群組\) 及 [Create a new user] \(建立新使用者\)](./media/adobe-echosign-tutorial/ic789524.png "Account \(帳戶\)")
   
3. 在 [建立新的使用者] 區段中，執行下列步驟：
   
    ![[Create a new user] \(建立新使用者\) 區段的螢幕擷取畫面](./media/adobe-echosign-tutorial/ic789525.png "Create User \(建立使用者\)")
   
    a. 針對您想要佈建的有效 Azure AD 帳戶，將帳戶的 [Email Address] \(電子郵件地址\)、[First Name] \(名字\) 及 [Last Name] \(姓氏\) 輸入至相關文字方塊中。
   
    b. 選取 [Create User] \(建立使用者\)。

>[!NOTE]
>Azure Active Directory 帳戶持有者會收到一封電子郵件，其中包含一個用來確認帳戶以使其生效的連結。 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Adobe Sign 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![Azure 入口網站單一登入的螢幕擷取畫面][200] 

1. 在 Azure 入口網站中，開啟應用程式檢視。 接著，瀏覽至目錄檢視，移至 [企業應用程式]，然後選取 [所有應用程式]。

    ![Azure 入口網站應用程式檢視的螢幕擷取畫面，其中已醒目提示 [企業應用程式] 和 [所有應用程式]][201] 

2. 在應用程式清單中，選取 [Adobe Sign]。

    ![應用程式清單的螢幕擷取畫面，其中已醒目提示 [Adobe Sign]](./media/adobe-echosign-tutorial/tutorial_adobesign_app.png) 

3. 在左側功能表中，選取 [使用者和群組]。

    ![功能表的螢幕擷取畫面，其中已醒目提示 [使用者和群組]][202] 

4. 選取 [新增] 。 接著，在 [新增指派] 區段中，選取 [使用者和群組]。

    ![[使用者和群組] 頁面及 [新增指派] 的螢幕擷取畫面][203]

5. 在 [使用者和群組] 對話方塊的使用者清單中，選取 [Britta Simon]。

6. 在 [使用者和群組] 對話方塊中，按一下 [選取]。

7. 在 [新增指派] 對話方塊中，選取 [指派]。
    
### <a name="test-single-sign-on"></a>測試單一登入

當您在「存取面板」中選取 [Adobe Sign] 圖格時，應該就會自動登入 Adobe Sign 應用程式。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/adobe-echosign-tutorial/tutorial_general_01.png
[2]: ./media/adobe-echosign-tutorial/tutorial_general_02.png
[3]: ./media/adobe-echosign-tutorial/tutorial_general_03.png
[4]: ./media/adobe-echosign-tutorial/tutorial_general_04.png

[100]: ./media/adobe-echosign-tutorial/tutorial_general_100.png

[200]: ./media/adobe-echosign-tutorial/tutorial_general_200.png
[201]: ./media/adobe-echosign-tutorial/tutorial_general_201.png
[202]: ./media/adobe-echosign-tutorial/tutorial_general_202.png
[203]: ./media/adobe-echosign-tutorial/tutorial_general_203.png
