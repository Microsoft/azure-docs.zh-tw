---
title: 教學課程：Azure Active Directory 與 Kanbanize 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Kanbanize 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: b436d2f6-bfa5-43fd-a8f9-d2144dc25669
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2018
ms.author: jeedes
ms.openlocfilehash: 746eaadcdb9a588087367c4c70237922cf0f14bf
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39057830"
---
# <a name="tutorial-azure-active-directory-integration-with-kanbanize"></a>教學課程：Azure Active Directory 與 Kanbanize 整合

在本教學課程中，您將了解如何整合 Kanbanize 與 Azure Active Directory (Azure AD)。

Kanbanize 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Kanbanize 的人員。
- 您可以讓您的使用者使用其 Azure AD 帳戶自動登入 Kanbanize (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Kanbanize 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Kanbanize 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Kanbanize
2. 設定並測試 Azure AD 單一登入

## <a name="adding-kanbanize-from-the-gallery"></a>從資源庫新增 Kanbanize
若要設定 Kanbanize 與 Azure AD 整合，您需要從資源庫將 Kanbanize 新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 Kanbanize，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，鍵入 **Kanbanize**，從結果面板中選取 **Kanbanize**，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Kanbanize](./media/kanbanize-tutorial/tutorial_kanbanize_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Kanbanize 來設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Kanbanize 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Kanbanize 中的相關使用者之間建立連結關聯性。

若要使用 Kanbanize 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Kanbanize 測試使用者](#create-a-kanbanize-test-user)** - 讓 Kanbanize 中對應的 Britta Simon 連結到使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Kanbanize 應用程式中設定單一登入。

**若要使用 Kanbanize 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Kanbanize] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/kanbanize-tutorial/tutorial_kanbanize_samlbase.png)

3. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Kanbanize 網域和 URL] 區段上執行下列步驟：

    ![Kanbanize 網域和 URL 單一登入資訊](./media/kanbanize-tutorial/tutorial_kanbanize_url.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<subdomain>.kanbanize.com/`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<subdomain>.kanbanize.com/saml/acs`

    c. 按一下 [顯示進階 URL 設定]。

    d.  在 [轉送狀態] 文字方塊中，輸入 URL：`/ctrl_login/saml_login`

    e. 如果您想要以 **SP** 起始模式設定應用程式，請在 [登入 URL] 文字方塊中使用下列模式輸入 URL：`https://<subdomain>.kanbanize.com`
     
    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 請連絡 [Kanbanize 用戶端支援小組](mailto:support@ms.kanbanize.com)以取得這些值。 

5. Kanbanize 應用程式需要特定格式的 SAML 判斷提示，因此您需要新增可對應到 SAML 權杖屬性設定的自訂屬性。 以下螢幕擷取畫面顯示上述的範例。 [使用者識別碼] 的預設值是 **user.userprincipalname**，但是 Kanbanize 會預期它是與使用者電子郵件地址對應的值。 對此您可以使用清單中的 **user.mail** 屬性，或者根據組織組態使用適當的屬性值
    
    ![設定單一登入](./media/kanbanize-tutorial/tutorial_Kanbanize_attribute.png)

6. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/kanbanize-tutorial/tutorial_kanbanize_certificate.png) 

7. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/kanbanize-tutorial/tutorial_general_400.png)
    
8. 在 [Kanbanize 組態] 區段上，按一下 [設定 Kanbanize] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![Kanbanize 組態](./media/kanbanize-tutorial/tutorial_kanbanize_configure.png)

9. 在不同的 Web 瀏覽器視窗中，以安全性系統管理員身分登入 Kanbanize。 

10. 移至頁面的右上方，按一下 [設定] 標誌。

    ![Kanbanize 設定](./media/kanbanize-tutorial/tutorial_kanbanize_set.png)

11. 從功能表左側的 [系統管理面板] 頁面上，按一下 [整合]，然後再啟用 [單一登入]。 

    ![Kanbanize 整合](./media/kanbanize-tutorial/tutorial_kanbanize_admin.png)

12. 在 [整合] 區段下，按一下 [設定] 來開啟 [單一登入整合] 頁面。

    ![Kanbanize 組態](./media/kanbanize-tutorial/tutorial_kanbanize_config.png)

13. 在 [單一登入整合] 頁面的 [設定] 底下，執行下列步驟：

    ![Kanbanize 整合](./media/kanbanize-tutorial/tutorial_kanbanize_save.png)

    a. 在 [IdP 實體識別碼] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    b. 在 [IdP 登入端點] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    c. 在 [IdP 登出端點] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    d. 在 [電子郵件的屬性名稱] 文字方塊中，輸入此值 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

    e. 在 [名字的屬性名稱] 文字方塊中，輸入此值 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    f. 在 [姓氏的屬性名稱] 文字方塊中，輸入此值 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname` 
    > [!Note]
    > 您可以從 Azure 入口網站的 [使用者屬性] 區段中，結合命名空間與個別屬性的名稱值來取得這些值。

    g. 在「記事本」中開啟從 Azure 入口網站下載的 Base 64 編碼憑證，複製其內容 (不含開始和結束標記)，然後貼到 [Idp X.509 憑證] 方塊中。

    h. 勾選 [使用 SSO 和 Kanbanize 啟用登入]。
    
    i. 按一下 [儲存設定] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/kanbanize-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/kanbanize-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/kanbanize-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/kanbanize-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-kanbanize-test-user"></a>建立 Kanbanize 測試使用者

本節目標是在 Kanbanize 中建立名為 Britta Simon 的使用者。 Kanbanize 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 嘗試存取 Kanbanize 時若尚無使用者，則會建立新使用者。

>[!Note]
>如果您需要手動建立使用者，請連絡 [Kanbanize 用戶端支援小組](mailto:support@ms.kanbanize.com)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Kanbanize 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Kanbanize，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Kanbanize]。

    ![應用程式清單中的 Kanbanize 連結](./media/kanbanize-tutorial/tutorial_kanbanize_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Kanbanize] 圖格時，應該會自動登入您的 Kanbanize 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/kanbanize-tutorial/tutorial_general_01.png
[2]: ./media/kanbanize-tutorial/tutorial_general_02.png
[3]: ./media/kanbanize-tutorial/tutorial_general_03.png
[4]: ./media/kanbanize-tutorial/tutorial_general_04.png

[100]: ./media/kanbanize-tutorial/tutorial_general_100.png

[200]: ./media/kanbanize-tutorial/tutorial_general_200.png
[201]: ./media/kanbanize-tutorial/tutorial_general_201.png
[202]: ./media/kanbanize-tutorial/tutorial_general_202.png
[203]: ./media/kanbanize-tutorial/tutorial_general_203.png

