---
title: 教學課程：Azure Active Directory 與 Slack 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Slack 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.assetid: ffc5e73f-6c38-4bbb-876a-a7dd269d4e1c
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2018
ms.author: jeedes
ms.openlocfilehash: 8f79926d0d4729c6ad939bc604e9eb885dbe9f03
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421236"
---
# <a name="tutorial-azure-active-directory-integration-with-slack"></a>教學課程：Azure Active Directory 與 Slack 整合

在本教學課程中，您會了解如何整合 Slack 與 Azure Active Directory (Azure AD)。

Slack 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Slack
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Slack (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Slack 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Slack 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Slack
1. 設定並測試 Azure AD 單一登入

## <a name="adding-slack-from-the-gallery"></a>從資源庫新增 Slack
若要設定將 Slack 整合到 Azure AD 中，您需要從資源庫將 Slack 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Slack，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在 [搜尋方塊] 中，輸入 **Slack**。

    ![建立 Azure AD 測試使用者](./media/slack-tutorial/tutorial_slack_search.png)

1. 在結果窗格中，選取 [Slack]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/slack-tutorial/tutorial_slack_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Slack 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Slack 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Slack 中的相關使用者之間建立連結關聯性。

在 Slack 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Slack 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Slack 測試使用者](#creating-a-slack-test-user)** - 使 Slack 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Slack 應用程式中設定單一登入。

**若要使用 Slack 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Slack] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/slack-tutorial/tutorial_slack_samlbase.png)

1. 在 [Slack 網域與 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/slack-tutorial/tutorial_slack_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<companyname>.slack.com`

    b. 在 [識別碼] 文字方塊中，輸入 URL：`https://slack.com`

    > [!NOTE] 
    > 這不是真正的值。 您必須使用實際的「登入 URL」來更新此值。 請連絡 [Slack 支援小組](https://slack.com/help/contact)以取得此值。
     
1. Slack 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。
    
    ![設定單一登入](./media/slack-tutorial/tutorial_slack_attribute.png)

    > [!NOTE] 
    > 如果您使用者獲指派的**電子郵件地址**不具備 Office365 授權，SAML 權杖中就不會出現 **User.Email** 宣告。 在這些情況下，建議您改用 **user.userprincipalname** 作為 **User.Email** 屬性值來對應成**唯一識別碼**。

1. 在 [單一登入] 對話方塊內的 [使用者屬性] 區段中，選取 [user.mail] 當作 [使用者識別碼]，並在下表顯示的每個列上執行下列步驟：
    
    | 屬性名稱 | 屬性值 |
    | --- | --- |
    | first_name | user.givenname |
    | last_name | user.surname |
    | User.Email | user.mail |  
    | User.Username | user.userprincipalname |

    a. 按一下 [屬性] 以開啟 [編輯屬性] 對話方塊，然後執行下列步驟：

    ![設定單一登入](./media/slack-tutorial/tutorial_slack_attribute1.png)

    a. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    b. 在 [值] 清單中，選取該資料列所顯示的屬性值。

    c. 讓 [命名空間] 保持空白。

    d. 按一下 [檔案] &gt; [新增] &gt; [專案] 

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/slack-tutorial/tutorial_slack_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/slack-tutorial/tutorial_general_400.png)

1. 在 [Slack 組態] 區段上，按一下 [設定 Slack] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體識別碼] 和 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/slack-tutorial/tutorial_slack_configure.png)

1. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Slack 公司網站。

1. 瀏覽至 [Microsoft Azure AD]，然後移至 [小組設定]。

     ![在應用程式端設定單一登入](./media/slack-tutorial/tutorial_slack_001.png)

1. 在 [小組設定] 區段中，按一下 [驗證] 索引標籤，然後按一下 [變更設定]。

    ![在應用程式端設定單一登入](./media/slack-tutorial/tutorial_slack_002.png)

1. 在 [SAML 驗證設定]  對話方塊上，執行下列步驟：

    ![在應用程式端設定單一登入](./media/slack-tutorial/tutorial_slack_003.png)

    a.  在 [SAML 2.0 端點]\(HTTP\) 文字方塊中，貼上您從 Azure 入口網站複製的 **「SAML 單一登入服務 URL」** 值。

    b.  在 [識別提供者簽發者] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    c.  在記事本中開啟您下載的憑證檔，將其內容複製到剪貼簿上，然後貼到 [公開憑證] 文字方塊中。

    d. 針對您的 Slack 小組適當地設定上述三個設定。 如需這些這設定的詳細資訊，請參閱這裡的 **Slack 的 SSO 設定指南**。 `https://get.slack.help/hc/articles/220403548-Guide-to-single-sign-on-with-Slack%60`

    e.  按一下 [儲存組態] 。

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/slack-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/slack-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。

    ![建立 Azure AD 測試使用者](./media/slack-tutorial/create_aaduser_03.png)

1. 在 [使用者]  對話頁面上，執行下列步驟：

    ![建立 Azure AD 測試使用者](./media/slack-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="creating-a-slack-test-user"></a>建立 Slack 測試使用者

本節目標是在 Slack 中建立名為 Britta Simon 的使用者。 Slack 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 當您嘗試存取 Slack 時，如果 Slack 還沒有使用者，它就會建立新的使用者。 Slack 也支援自動使用者佈建，您可以在[這裡](slack-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

> [!NOTE]
> 如果您需要手動建立使用者，就需要連絡 [Slack 支援小組](https://slack.com/help/contact)。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Slack 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200]

**若要將 Britta Simon 指派到 Slack，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Slack]。

    ![設定單一登入](./media/slack-tutorial/tutorial_slack_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Slack 圖格時，應該會自動登入您的 Slack 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](slack-provisioning-tutorial.md)


<!--Image references-->

[1]: ./media/slack-tutorial/tutorial_general_01.png
[2]: ./media/slack-tutorial/tutorial_general_02.png
[3]: ./media/slack-tutorial/tutorial_general_03.png
[4]: ./media/slack-tutorial/tutorial_general_04.png

[100]: ./media/slack-tutorial/tutorial_general_100.png

[200]: ./media/slack-tutorial/tutorial_general_200.png
[201]: ./media/slack-tutorial/tutorial_general_201.png
[202]: ./media/slack-tutorial/tutorial_general_202.png
[203]: ./media/slack-tutorial/tutorial_general_203.png
