---
title: 教學課程：Azure Active Directory 與 LogicMonitor 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 LogicMonitor 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 496156c3-0e22-4492-b36f-2c29c055e087
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/02/2018
ms.author: jeedes
ms.openlocfilehash: a6bc220d15e720662eaa9605421e21ccb99892ab
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502340"
---
# <a name="tutorial-azure-active-directory-integration-with-logicmonitor"></a>教學課程：Azure Active Directory 與 LogicMonitor 整合

在本教學課程中，您將了解如何整合 LogicMonitor 與 Azure Active Directory (Azure AD)。

LogicMonitor 與 Azure AD 整合有下列優點：

- 您可以在 Azure AD 中控制可存取 LogicMonitor 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 LogicMonitor (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要進行 Azure AD 與 LogicMonitor 整合的設定，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 LogicMonitor 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 LogicMonitor
1. 設定並測試 Azure AD 單一登入

## <a name="adding-logicmonitor-from-the-gallery"></a>從資源庫新增 LogicMonitor
若要進行 LogicMonitor 與 Azure AD 整合的設定，您需要從資源庫將 LogicMonitor 新增至受控 SaaS 應用程式清單中。

**若要從資源庫新增 LogicMonitor，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **LogicMonitor**。

    ![建立 Azure AD 測試使用者](./media/logicmonitor-tutorial/tutorial_logicmonitor_search.png)

1. 在結果面板中，選取 [LogicMonitor]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/logicmonitor-tutorial/tutorial_logicmonitor_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您將以名為 "Britta Simon" 的測試使用者身分，設定及測試與 LogicMonitor 搭配運作的 Azure AD 單一登入。

為了讓單一登入正常運作，Azure AD 必須知道 Azure AD 使用者在 LogicMonitor 中的對應使用者是誰。 換句話說，必須在 Azure AD 使用者與 LogicMonitor 的相關使用者之間建立連結關聯性。

在 LogicMonitor 中，將 Azure AD 中 [使用者名稱] 的值指派為 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 LogicMonitor 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 LogicMonitor 測試使用者](#creating-a-logicmonitor-test-user)** - 使 LogicMonitor 中 Britta Simon 的對應使用者連結到該使用者在 Azure AD 中的代表身分。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您將在 Azure 入口網站中啟用 Azure AD 單一登入，並在 LogicMonitor 應用程式中設定單一登入。

**若要設定 LogicMonitor 的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [LogicMonitor] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/logicmonitor-tutorial/tutorial_logicmonitor_samlbase.png)

1. 在 [LogicMonitor 網域及 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/logicmonitor-tutorial/tutorial_logicmonitor_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<companyname>.logicmonitor.com`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<companyname>.logicmonitor.com`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [LogicMonitor 用戶端支援小組](https://www.logicmonitor.com/contact/)以取得這些值。 
 


1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/logicmonitor-tutorial/tutorial_logicmonitor_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/logicmonitor-tutorial/tutorial_general_400.png)

1. 以系統管理員身分登入您的 **LogicMonitor** 公司網站。

1. 在頂端的功能表中，按一下 [設定] 。
   
    ![設定](./media/logicmonitor-tutorial/ic790052.png "設定")

1. 在左側導覽列中按一下 [單一登入] 
   
    ![單一登入](./media/logicmonitor-tutorial/ic790053.png "單一登入")

1. 在 [單一登入 (SSO) 設定]  區段中，執行下列步驟：
   
    ![單一登入設定](./media/logicmonitor-tutorial/ic790054.png "單一登入設定")
   
    a. 選取 [啟用單一登入] 。

    b. 在 [預設角色指派] 中選取 [唯讀]。
   
    c. 在記事本中開啟下載的中繼資料檔，然後將檔案內容貼到 [識別提供者中繼資料]  文字方塊中。
   
    d. 按一下 [儲存變更] 。

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/logicmonitor-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/logicmonitor-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/logicmonitor-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/logicmonitor-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-logicmonitor-test-user"></a>建立 LogicMonitor 測試使用者

若要讓 Azure AD 使用者能夠登入，必須使用他們的 Azure Active Directory 使用者名稱將他們佈建到 LogicMonitor 應用程式。

**若要設定使用者佈建，請執行下列步驟：**

1. 以系統管理員身分登入您的 LogicMonitor 公司網站。

1. 在頂端的功能表中，按一下 [設定]，然後按一下 [角色與使用者]。
   
    ![角色和使用者](./media/logicmonitor-tutorial/ic790056.png "角色和使用者")

1. 按一下 [新增] 。

1. 在 [新增帳戶]  區段中，執行下列步驟：
   
    ![新增帳戶](./media/logicmonitor-tutorial/ic790057.png "新增帳戶")
   
    a. 在相關的文字方塊中輸入您想要佈建之 Azure Active Directory 使用者的 [使用者名稱]、[電子郵件]、[密碼] 及 [重新輸入密碼] 值。
   
    b. 選取 [角色]、[檢視權限] 及 [狀態]。
   
    c. 按一下 [提交] 。

>[!NOTE]
>您可以使用任何其他的 LogicMonitor 使用者帳戶建立工具或 LogicMonitor 提供的 API，佈建 Azure Active Directory 使用者帳戶。 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您將把 LogicMonitor 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 LogicMonitor，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [LogicMonitor]。

    ![設定單一登入](./media/logicmonitor-tutorial/tutorial_logicmonitor_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。
 
按一下存取面板中的 [LogicMonitor] 圖格，應會自動登入您的 LogicMonitor 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/logicmonitor-tutorial/tutorial_general_01.png
[2]: ./media/logicmonitor-tutorial/tutorial_general_02.png
[3]: ./media/logicmonitor-tutorial/tutorial_general_03.png
[4]: ./media/logicmonitor-tutorial/tutorial_general_04.png

[100]: ./media/logicmonitor-tutorial/tutorial_general_100.png

[200]: ./media/logicmonitor-tutorial/tutorial_general_200.png
[201]: ./media/logicmonitor-tutorial/tutorial_general_201.png
[202]: ./media/logicmonitor-tutorial/tutorial_general_202.png
[203]: ./media/logicmonitor-tutorial/tutorial_general_203.png

