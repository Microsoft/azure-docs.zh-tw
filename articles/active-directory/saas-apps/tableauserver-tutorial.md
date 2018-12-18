---
title: 教學課程：Azure Active Directory 與 Tableau Server 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Tableau Server 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: c1917375-08aa-445c-a444-e22e23fa19e0
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/18/2017
ms.author: jeedes
ms.openlocfilehash: 78f58f28eb9c25e0b5f6869f7e2348b41780fb60
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39437061"
---
# <a name="tutorial-azure-active-directory-integration-with-tableau-server"></a>教學課程：Azure Active Directory 與 Tableau Server 整合

在本教學課程中，您將了解如何整合 Tableau Server 與 Azure Active Directory (Azure AD)。

Tableau Server 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Tableau Server 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Tableau Server (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Tableau Server 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Tableau Server 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Tableau Server
1. 設定並測試 Azure AD 單一登入

## <a name="adding-tableau-server-from-the-gallery"></a>從資源庫新增 Tableau Server
若要設定 Tableau Server 與 Azure AD 整合，您需要從資源庫將 Tableau Server 加入到受控 SaaS 應用程式清單中。

**如要從資源庫新增 Tableau Server，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Tableau Server**。

    ![建立 Azure AD 測試使用者](./media/tableauserver-tutorial/tutorial_tableauserver_search.png)

1. 在結果窗格中，選取 [Tableau Server]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/tableauserver-tutorial/tutorial_tableauserver_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Tableau Server 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Tableau Server 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Tableau Server 中的相關使用者之間建立連結關聯性。

在 Tableau Server 中，將 Azure AD 中 [使用者名稱] 的值指派為 [使用者名稱] 的值，以建立連結關聯性。

若要使用 Tableau Server 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Tableau Server 測試使用者](#creating-a-tableau-server-test-user)** - 使 Tableau Server 中 Britta Simon 的對應使用者連結到該使用者在 Azure AD 中的代表身分。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Tableau Server 應用程式中設定單一登入。

**若要設定透過 Tableau Server 使用 Azure AD 單一登入功能，請執行下列步驟：**

1. 在 Azure 入口網站的 [Tableau Server] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/tableauserver-tutorial/tutorial_tableauserver_samlbase.png)

1. 在 [Tableau Server 網域及 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/tableauserver-tutorial/tutorial_tableauserver_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://azure.<domain name>.link`
    
    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://azure.<domain name>.link`

    c. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://azure.<domain name>.link/wg/saml/SSO/index.html`
     
    > [!NOTE] 
    > 上述值並非真正的值。 稍後您將使用 [Tableau Server 設定] 頁面中實際的 URL 和識別碼來更新這些值。 

1. Tableau Server 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 下列螢幕擷取畫面顯示相同範例。
    
    ![設定單一登入](./media/tableauserver-tutorial/3.png)
    
1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱 | 屬性值 |
    | ---------------| --------------- |    
    | username | *user.mailnickname* |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/tableauserver-tutorial/tutorial_officespace_04.png)

    ![設定單一登入](./media/tableauserver-tutorial/tutorial_officespace_05.png)
    
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定]。


1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/tableauserver-tutorial/tutorial_tableauserver_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/tableauserver-tutorial/tutorial_general_400.png)
<CS>
1. 若要取得為應用程式設定的 SSO，您必須以系統管理員身分登入 Tableau Server 租用戶。
   
   a. 在 Tableau Server 組態中，按一下 [SAML] 索引標籤。
  
    ![設定單一登入](./media/tableauserver-tutorial/tutorial_tableauserver_001.png) 
  
   b. 選取 [使用 SAML 進行單一登入] 的核取方塊。
   
   c. Tableau Server 傳回 URL — Tableau Server 使用者將存取的 URL，例如 http://tableau_server。 不建議使用 http://localhost。 不支援使用包含結尾斜線的 URL (例如， http://tableau_server/)。 複製 [Tableau Server 傳回 URL]，並將其貼到 [Tableau Server 網域及 URL] 區段中的 Azure AD [單一登入 URL] 文字方塊。
   
   d. SAML 實體識別碼—實體識別碼可唯一識別安裝至 IdP 的 Tableau Server。 您可以依意願再次在這裡輸入 Tableau Server URL，但是它不一定是您的 Tableau Server URL。 複製 [SAML 實體識別碼]，並將其貼到 [Tableau Server 網域及 URL] 區段中的 Azure AD [識別碼] 文字方塊。
     
   e. 按一下 [匯出中繼資料檔案]，並在文字編輯器應用程式中將其開啟。 利用 Http Post 與索引 0 找出判斷提示取用者服務 URL，並複製 URL。 現在將其貼到 [Tableau Server 網域及 URL] 區段中的 Azure AD [回覆 URL] 文字方塊。
   
   f. 尋找從 Azure 入口網站下載的同盟中繼資料檔案，然後在 [SAML Idp 中繼資料檔案] 中將其上傳。
   
   g. 按一下 [Tableau Server 設定] 頁面中的 [確定] 按鈕。
   
    >[!NOTE] 
    >客戶必須在 [Tableau Server SAML SSO 設定] 中上傳任何憑證，系統將在 SSO 流程中忽略該憑證。
    >如果您需要在 Tableau Server 上設定 SAML 的協助，請參閱[設定 SAML](http://onlinehelp.tableau.com/current/server/en-us/config_saml.htm)。
    >
<CE>

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/tableauserver-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/tableauserver-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/tableauserver-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/tableauserver-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-tableau-server-test-user"></a>建立 Tableau Server 測試使用者

本節目標是在 Tableau Server 中建立名為 Britta Simon 的使用者。 您必須在 Tableau Server 中佈建所有使用者。 

使用者的使用者名稱應符合您在 Azure AD 自訂屬性 **username** 中設定的值。 透過正確的對應，應該可以有效整合， [設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)。

>[!NOTE]
>如果您需要以手動方式建立使用者，您必須連絡貴組織的 Tableau Server 系統管理員。
> 
> 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Tableau Server 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**如要將 Britta Simon 指派給 Tableau Server，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Tableau Server] 。

    ![設定單一登入](./media/tableauserver-tutorial/tutorial_tableauserver_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Tableau Server] 圖格時，應該會自動登入您的 Tableau Server 應用程式。
如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/tableauserver-tutorial/tutorial_general_01.png
[2]: ./media/tableauserver-tutorial/tutorial_general_02.png
[3]: ./media/tableauserver-tutorial/tutorial_general_03.png
[4]: ./media/tableauserver-tutorial/tutorial_general_04.png

[100]: ./media/tableauserver-tutorial/tutorial_general_100.png

[200]: ./media/tableauserver-tutorial/tutorial_general_200.png
[201]: ./media/tableauserver-tutorial/tutorial_general_201.png
[202]: ./media/tableauserver-tutorial/tutorial_general_202.png
[203]: ./media/tableauserver-tutorial/tutorial_general_203.png

