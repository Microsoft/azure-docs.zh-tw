---
title: 教學課程：Azure Active Directory 與 Dome9 Arc 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Dome9 Arc 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 4c12875f-de71-40cb-b9ac-216a805334e5
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/22/2018
ms.author: jeedes
ms.openlocfilehash: 0b6730d6c365484dc19273069785e6824218113e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387849"
---
# <a name="tutorial-azure-active-directory-integration-with-dome9-arc"></a>教學課程：Azure Active Directory 與 Dome9 Arc 整合

在本教學課程中，您將了解如何整合 Dome9 Arc 與 Azure Active Directory (Azure AD)。

Dome9 Arc 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Dome9 Arc 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Dome9 Arc (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Dome9 Arc 整合，您需要下列項目：

- Azure AD 訂用帳戶
- Dome9 Arc 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Dome9 Arc
2. 設定並測試 Azure AD 單一登入

## <a name="adding-dome9-arc-from-the-gallery"></a>從資源庫新增 Dome9 Arc

若要設定將 Dome9 Arc 整合到 Azure AD 中，您必須從資源庫將 Dome9 Arc 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Dome9 Arc，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Dome9 Arc**，從結果面板中選取 [Dome9 Arc]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Dome9 Arc](./media/dome9arc-tutorial/tutorial_dome9arc_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以稱為 "Britta Simon" 的測試使用者身分，使用 Dome9 Arc 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Dome9 Arc 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Dome9 Arc 中相關使用者之間的連結關聯性。

若要使用 Dome9 Arc 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Dome9 Arc 測試使用者](#create-a-dome9-arc-test-user)** - 使 Dome9 Arc 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Dome9 Arc 應用程式中設定單一登入。

**若要使用 Dome9 Arc 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Dome9 Arc] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/dome9arc-tutorial/tutorial_dome9arc_samlbase.png)

3. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Dome9 Arc 網域和 URL] 區段上執行下列步驟：

    ![Dome9 Arc 網域與 URL 單一登入資訊](./media/dome9arc-tutorial/tutorial_dome9arc_url.png)

    a. 在 [識別碼] 文字方塊中，輸入 URL：`https://secure.dome9.com/`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://secure.dome9.com/sso/saml/yourcompanyname`

    > [!NOTE]
    > 您將在 dome9 管理入口網站中選取您的公司名稱值，本教學課程稍後會說明這一點。

4. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![Dome9 Arc 網域與 URL 單一登入資訊](./media/dome9arc-tutorial/tutorial_dome9arc_url1.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://secure.dome9.com/sso/saml/<yourcompanyname>`
 
    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的回覆 URL 與登入 URL 更新這些值。 請連絡 [Dome9 Arc 用戶端支援小組](https://dome9.com/about/contact-us/)以取得這些值。 

5. Dome9 Arc 軟體應用程式預期要有特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。

    ![設定單一登入屬性](./media/dome9arc-tutorial/tutorial_dome9arc_attribute.png)

6. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱  | 屬性值 | 
    | --------------- | --------------- | 
    | memberof | user.assignedroles |
    
    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入新增屬性](./media/dome9arc-tutorial/tutorial_dome9_04.png)

    ![設定單一登入編輯屬性](./media/dome9arc-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 按一下 [確定] 。
    
    > [!NOTE]
    > 請參閱此[連結](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-app-role-management)，以了解如何設定及安裝應用程式的角色。

7. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/dome9arc-tutorial/tutorial_dome9arc_certificate.png) 

8. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/dome9arc-tutorial/tutorial_general_400.png)

9. 在 [Dome9 Arc 設定] 區段上，按一下 [設定 Dome9 Arc] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![Dome9 Arc 設定](./media/dome9arc-tutorial/tutorial_dome9arc_configure.png) 

10. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Dome9 Arc 公司網站。

11. 按一下右上角的 [設定檔設定]，然後按一下 [帳戶設定]。 

    ![Dome9 Arc 設定](./media/dome9arc-tutorial/configure1.png)

12. 瀏覽至 [SSO]，然後按一下 [啟用]。

    ![Dome9 Arc 設定](./media/dome9arc-tutorial/configure2.png)

13. 在 [SSO 設定] 區段中，執行下列步驟：

    ![Dome9 Arc 設定](./media/dome9arc-tutorial/configure3.png)

    a. 在 [帳戶識別碼] 文字方塊中輸入公司名稱。 在 Azure 入口網站 URL 一節所述的回覆 URL 中將使用這個值。

    b. 在 [簽發者] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    c. 在 [IdP 端點 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    d. 在記事本中開啟您下載的 Base64 編碼的憑證，將其內容複製到剪貼簿，然後貼到 [X.509 憑證] 文字方塊。

    e. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/dome9arc-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/dome9arc-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/dome9arc-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/dome9arc-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-dome9-arc-test-user"></a>建立 Dome9 Arc 測試使用者

若要讓 Azure AD 使用者能夠登入 Dome9 Arc，必須將他們佈建到應用程式。 Dome9 Arc 支援 Just-In-Time 佈建；但是，若要正常佈建，使用者必須選取特定**角色**，並將該角色指派給使用者。

   >[!Note]
   >關於**角色**建立和其他詳細資料，請連絡 [Dome9 Arc 用戶端支援小組](https://dome9.com/about/contact-us/)。

**若要手動佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 Dome9 Arc 公司網站。

2. 按一下 [使用者和角色]，然後按一下 [使用者]。

    ![新增員工](./media/dome9arc-tutorial/user1.png)

3. 按一下 [新增使用者]。

    ![新增員工](./media/dome9arc-tutorial/user2.png)

4. 在 [建立使用者]  區段中，執行下列步驟：

    ![新增員工](./media/dome9arc-tutorial/user3.png)

    a. 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件，例如 Brittasimon@contoso.com。

    b. 在 [名字] 文字方塊中，輸入使用者的名字，例如 Britta。

    c. 在 [姓氏] 文字方塊中，輸入使用者的姓氏，例如 Simon。

    d. 將 [SSO 使用者] 設定為 [開啟]。

    e. 按一下 [建立]。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Dome9 Arc 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Dome9 Arc，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Dome9 Arc]。

    ![應用程式清單中的 Dome9 Arc 連結](./media/dome9arc-tutorial/tutorial_dome9arc_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Dome9 Arc 圖示時，應該會自動登入 Dome9 Arc 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/dome9arc-tutorial/tutorial_general_01.png
[2]: ./media/dome9arc-tutorial/tutorial_general_02.png
[3]: ./media/dome9arc-tutorial/tutorial_general_03.png
[4]: ./media/dome9arc-tutorial/tutorial_general_04.png

[100]: ./media/dome9arc-tutorial/tutorial_general_100.png

[200]: ./media/dome9arc-tutorial/tutorial_general_200.png
[201]: ./media/dome9arc-tutorial/tutorial_general_201.png
[202]: ./media/dome9arc-tutorial/tutorial_general_202.png
[203]: ./media/dome9arc-tutorial/tutorial_general_203.png

