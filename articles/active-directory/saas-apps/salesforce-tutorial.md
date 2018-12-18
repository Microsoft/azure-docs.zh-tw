---
title: 教學課程：Azure Active Directory 與 Salesforce 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Salesforce 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: d2d7d420-dc91-41b8-a6b3-59579e043b35
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2018
ms.author: jeedes
ms.openlocfilehash: 2f87c4a15ac21241b3304d1fdf0a5bd0ae715615
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39430200"
---
# <a name="tutorial-azure-active-directory-integration-with-salesforce"></a>教學課程：Azure Active Directory 與 Salesforce 整合

在本教學課程中，您會了解如何整合 Salesforce 與 Azure Active Directory (Azure AD)。

Salesforce 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Salesforce 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Salesforce (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Salesforce 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Salesforce 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Salesforce
1. 設定並測試 Azure AD 單一登入

## <a name="adding-salesforce-from-the-gallery"></a>從資源庫新增 Salesforce
若要設定 Salesforce 與 Azure AD 的整合作業，您必須從資源庫將 Salesforce 新增至受控 SaaS 應用程式清單。

**若要從資源庫新增 Salesforce，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Salesforce**，從結果面板中選取 [Salesforce]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Salesforce](./media/salesforce-tutorial/tutorial_salesforce_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Salesforce 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Salesforce 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Salesforce 中的相關使用者之間建立連結關聯性。

在 Salesforce 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Salesforce 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Salesforce 測試使用者](#create-a-salesforce-test-user)** - 使 Salesforce 中 Britta Simon 的對應身分連結到該使用者在 Azure AD 中的代表身分。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Salesforce 應用程式中設定單一登入。

**若要使用 Salesforce 設定 Azure AD 單一登入功能，請執行下列步驟：**

1. 在 Azure 入口網站的 [Salesforce] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/salesforce-tutorial/tutorial_salesforce_samlbase.png)

1. 在 [Salesforce 網域及 URL] 區段中，執行下列步驟：

    ![Salesforce 網域和 URL 單一登入資訊](./media/salesforce-tutorial/tutorial_salesforce_url.png)

    a. 在 [登入 URL] 文字方塊中，以下列模式輸入值：

    企業帳戶： `https://<subdomain>.my.salesforce.com`

    開發人員帳戶： `https://<subdomain>-dev-ed.my.salesforce.com`

    b. 在 [識別碼] 文字方塊中，使用下列模式將值輸入：

    企業帳戶： `https://<subdomain>.my.salesforce.com`

    開發人員帳戶： `https://<subdomain>-dev-ed.my.salesforce.com`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Salesforce 用戶端支援小組](https://help.salesforce.com/support)以取得這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/salesforce-tutorial/tutorial_salesforce_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/salesforce-tutorial/tutorial_general_400.png)

1. 在 [Salesforce 組態] 區段上，按一下 [設定 Salesforce] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![Salesforce 設定](./media/salesforce-tutorial/tutorial_salesforce_configure.png) 

1. 在瀏覽器中開啟新索引標籤，登入您的 Salesforce 系統管理員帳戶。

1. 按一下分頁右上角**設定圖示**底下的 [設定]。

    ![設定單一登入](./media/salesforce-tutorial/configure1.png)

1. 在瀏覽窗格中向下捲動至 [設定]，按一下 [識別] 以展開相關的區段。 然後按一下 [單一登入設定]。

    ![設定單一登入](./media/salesforce-tutorial/sf-admin-sso.png)

1. 在 [單一登入設定] 頁面上，按一下 [編輯] 按鈕。

    ![設定單一登入](./media/salesforce-tutorial/sf-admin-sso-edit.png)
    
    > [!NOTE]
    > 如果您的 Salesforce 帳戶無法啟用單一登入設定，您可能需要連絡 [Salesforce 用戶端支援小組](https://help.salesforce.com/support)。 

1. 選取 [啟用 SAML]，然後按一下 [儲存]。

      ![設定單一登入](./media/salesforce-tutorial/sf-enable-saml.png)
1. 若要設定 SAML 單一登入設定，請按一下 [新增] 。

    ![設定單一登入](./media/salesforce-tutorial/sf-admin-sso-new.png)

1. 在 [SAML 單一登入設定編輯]  頁面上，進行下列設定：

    ![設定單一登入](./media/salesforce-tutorial/sf-saml-config.png)

    a. 在 [名稱]  欄位中，為此組態輸入易記的名稱。 提供 [名稱] 的值會自動填入 [API 名稱] 文字方塊。

    b. 在 [簽發者] 欄位中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    c. 在 [實體識別碼] 文字方塊中，使用下列形式輸入您的 Salesforce 網域名稱：

      * 企業帳戶： `https://<subdomain>.my.salesforce.com`
      * 開發人員帳戶： `https://<subdomain>-dev-ed.my.salesforce.com`

    d. 若要上傳 [識別提供者憑證]，按一下 [選擇檔案] 以瀏覽及選取您從 Azure 入口網站下載的憑證檔案。

    e. 作為 **SAML 識別類型**，選擇下列其中一個選項：

      * 如果使用者的 Salesforce 使用者名稱傳入 SAML 判斷提示，選取 [判斷提示包含使用者的 Salesforce 使用者名稱]

      * 如果使用者物件的同盟識別碼傳入 SAML 判斷提示，選取 [判斷提示包含使用者物件的同盟識別碼]

      * 如果使用者物件的使用者識別碼傳入 SAML 判斷提示，選取 [判斷提示包含使用者物件的使用者識別碼]

    f. 對於 [SAML 身分識別位置]，請選取 [身分識別位於 Subject 陳述式的 NameIdentifier 元素中]。

    g. 對於 [服務提供者起始的要求繫結]，請選取 [HTTP 重新導向]。

    h. 在 [識別提供者登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [單一登入服務 URL] 值

    i. 最後，按一下 [儲存]  套用您的 SAML 單一登入設定。

1. 在 Salesforce 的左方導覽窗格中，按一下 [公司設定] 以展開相關的區段，然後按一下 [我的網域]。

    ![設定單一登入](./media/salesforce-tutorial/sf-my-domain.png)

1. 向下捲動至 [驗證組態] 區段，然後按一下 [編輯] 按鈕。

    ![設定單一登入](./media/salesforce-tutorial/sf-edit-auth-config.png)

1. 在 [驗證組態] 區段中，核取 [AzureSSO] 作為 SAML SSO 設定的 [驗證服務]，然後按一下 [儲存]。

    ![設定單一登入](./media/salesforce-tutorial/sf-auth-config.png)

    > [!NOTE]
    > 如果選取了一個以上的驗證服務，當使用者嘗試啟動單一登入到您的 Salesforce 環境時，將會提示使用者選取登入時想要使用的驗證服務。 如果您不想發生這種情形，您應該**不要核取其他所有的驗證服務**。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/salesforce-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/salesforce-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/salesforce-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/salesforce-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-salesforce-test-user"></a>建立 Salesforce 測試使用者

本節會在 Salesforce 中建立名為 Britta Simon 的使用者。 Salesforce 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 如果 Salesforce 中還沒有該使用者，當您嘗試存取 Salesforce 時，就會建立新的使用者。 Salesforce 也支援自動使用者佈建，您可以在[這裡](salesforce-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Salesforce 的存取權授與 Britta Simon，讓 Britta Simon 能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Salesforce，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

1. 在應用程式清單中，選取 [Salesforce]。

    ![應用程式清單中的 Salesforce 連結](./media/salesforce-tutorial/tutorial_salesforce_app.png)

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Salesforce] 圖格時，應該會自動登入您的 Salesforce 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](salesforce-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/salesforce-tutorial/tutorial_general_01.png
[2]: ./media/salesforce-tutorial/tutorial_general_02.png
[3]: ./media/salesforce-tutorial/tutorial_general_03.png
[4]: ./media/salesforce-tutorial/tutorial_general_04.png

[100]: ./media/salesforce-tutorial/tutorial_general_100.png

[200]: ./media/salesforce-tutorial/tutorial_general_200.png
[201]: ./media/salesforce-tutorial/tutorial_general_201.png
[202]: ./media/salesforce-tutorial/tutorial_general_202.png
[203]: ./media/salesforce-tutorial/tutorial_general_203.png
