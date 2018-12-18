---
title: 教學課程：Azure Active Directory 與 Nimblex 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Nimblex 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: d3e165a5-f062-4b50-ac0b-b400838e99cd
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/05/2018
ms.author: jeedes
ms.openlocfilehash: 7b5dc6d892741f63596589a48ad5d45891b14c21
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39040400"
---
# <a name="tutorial-azure-active-directory-integration-with-nimblex"></a>教學課程：Azure Active Directory 與 Nimblex 整合

在本教學課程中，您將了解如何整合 Nimblex 與 Azure Active Directory (Azure AD)。

Nimblex 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Nimblex 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Nimblex (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Nimblex 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Nimblex 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Nimblex
2. 設定並測試 Azure AD 單一登入

## <a name="adding-nimblex-from-the-gallery"></a>從資源庫新增 Nimblex
若要設定將 Nimblex 整合到 Azure AD 中，您必須從資源庫將 Nimblex 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Nimblex，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，鍵入 **Nimblex**，從結果面板中選取 [Nimblex]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Nimblex](./media/nimblex-tutorial/tutorial_nimblex_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以稱為 "Britta Simon" 的測試使用者身分，使用 Nimblex 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Nimblex 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Nimblex 中的相關使用者之間的連結關聯性。

若要設定及測試與 Nimblex 搭配使用的 Azure AD 單一登入，您需要完成下列構成元素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Nimblex 測試使用者](#create-a-nimblex-test-user)** - 使 Nimblex 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Nimblex 應用程式中設定單一登入。

**若要使用 Nimblex 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Nimblex] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/nimblex-tutorial/tutorial_nimblex_samlbase.png)

3. 在 [Nimblex 網域與 URL] 區段上，執行下列步驟：

    ![Nimblex 網域與 URL 單一登入資訊](./media/nimblex-tutorial/tutorial_nimblex_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<YOUR APPLICATION PATH>/Login.aspx`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<YOUR APPLICATION PATH>/`

    c. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<path-to-application>/SamlReply.aspx`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「單一登入 URL」、「識別碼」及「回覆 URL」來更新這些值。 請連絡 [Nimblex 用戶端支援小組](mailto:support@ebms.com.au)以取得這些值。

4. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/nimblex-tutorial/tutorial_nimblex_certificate.png) 

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/nimblex-tutorial/tutorial_general_400.png)

6. 在 [Nimblex 設定] 區段上，按一下 [設定 Nimblex] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![Nimblex 設定](./media/nimblex-tutorial/tutorial_nimblex_configure.png) 

7. 在不同的網頁瀏覽器視窗中，以安全性系統管理員身分登入 Nimblex。

9. 按一下頁面右上方的 [設定] 標誌。

    ![Nimblex 設定](./media/nimblex-tutorial/tutorial_nimblex_settings.png)

10. 在 [控制台] 頁面的 [安全性] 區段下，按一下 [單一登入]。

    ![Nimblex 設定](./media/nimblex-tutorial/tutorial_nimblex_single.png)

11. 在 [管理單一登入] 頁面上，選取您的執行個體名稱，然後按一下 [編輯]。

    ![Nimblex saml](./media/nimblex-tutorial/tutorial_nimblex_saml.png)

12. 在 [Edit SSO Provider] \(編輯 SSO 提供者\) 頁面上，執行下列步驟：

    ![Nimblex saml](./media/nimblex-tutorial/tutorial_nimblex_sso.png)

    a. 在 [描述] 文字方塊中，鍵入您的執行個體名稱。

    b. 在 [記事本] 中開啟從 Azure 入口網站下載的 Base-64 編碼憑證，複製其內容，然後貼到 [憑證] 方塊中。

    c. 在 [識別提供者 SSO目標 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 **SAML 單一登入服務 URL** 值。

    d. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/nimblex-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/nimblex-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/nimblex-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/nimblex-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-nimblex-test-user"></a>建立 Nimblex 測試使用者

本節的目標是要在 Nimblex 中建立名為 Britta Simon 的使用者。 Nimblex 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 嘗試存取 Nimblex 時，如果使用者還不存在，就會建立新使用者。

>[!Note]
>如果您需要手動建立使用者，請連絡 [Nimblex 用戶端支援小組](mailto:support@ebms.com.au)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Nimblex 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派到 Nimblex，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Nimblex]。

    ![應用程式清單中的 Nimblex 連結](./media/nimblex-tutorial/tutorial_nimblex_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Nimblex] 磚時，應該會自動登入 Nimblex 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/nimblex-tutorial/tutorial_general_01.png
[2]: ./media/nimblex-tutorial/tutorial_general_02.png
[3]: ./media/nimblex-tutorial/tutorial_general_03.png
[4]: ./media/nimblex-tutorial/tutorial_general_04.png

[100]: ./media/nimblex-tutorial/tutorial_general_100.png

[200]: ./media/nimblex-tutorial/tutorial_general_200.png
[201]: ./media/nimblex-tutorial/tutorial_general_201.png
[202]: ./media/nimblex-tutorial/tutorial_general_202.png
[203]: ./media/nimblex-tutorial/tutorial_general_203.png

