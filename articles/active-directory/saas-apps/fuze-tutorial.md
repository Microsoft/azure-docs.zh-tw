---
title: 教學課程：Azure Active Directory 與 Fuze 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Fuze 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 9780b4bf-1fd1-48c1-9ceb-f750225ae08a
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/31/2017
ms.author: jeedes
ms.openlocfilehash: 6a3fa8db14f2b155a471fd42d0d20ccf543fdbba
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49115750"
---
# <a name="tutorial-azure-active-directory-integration-with-fuze"></a>教學課程：Azure Active Directory 與 Fuze 整合

在本教學課程中，您會了解如何整合 Fuze 與 Azure Active Directory (Azure AD)。

Fuze 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Fuze 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Fuze (單一登入)
- 您可以在 Azure 管理入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Fuze 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Fuze 單一登入的訂用帳戶


> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。


若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。


## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Fuze
1. 設定並測試 Azure AD 單一登入


## <a name="adding-fuze-from-the-gallery"></a>從資源庫新增 Fuze
若要設定將 Fuze 整合到 Azure AD 中，您需要從資源庫將 Fuze 新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 Fuze，請執行下列步驟：**

1. 在 **[Azure 管理入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 按一下對話方塊頂端的 [新增] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Fuze**。

    ![建立 Azure AD 測試使用者](./media/fuze-tutorial/tutorial_fuze_000.png)

1. 在結果窗格中，選取 [Fuze]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/fuze-tutorial/tutorial_fuze_0001.png)


##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Fuze 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Fuze 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Fuze 中相關使用者之間的連結關聯性。

建立此連結關聯性的方法，是將 Azure AD 中**使用者名稱**的值，指派為 Fuze 中 **Username** 的值。

若要設定及測試與 Fuze 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Fuze 測試使用者](#creating-a-fuze-test-user)** - 在 Fuze 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表 Britta Simon 的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 管理入口網站中啟用 Azure AD 單一登入，並在您的 Fuze 中設定單一登入。

**若要設定與 Fuze 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 管理入口網站的 [Fuze] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，選取 [SAML 型登入] 做為 [模式]，以啟用單一登入。
 
    ![設定單一登入](./media/fuze-tutorial/tutorial_fuze_01.png)

1. 在 [Fuze 網域與 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/fuze-tutorial/tutorial_fuze_020.png)
    
    在 [登入 URL] 文字方塊中，將登入 URL 輸入為下列項目：`https://www.thinkingphones.com/jetspeed/portal/`

1.  按一下 [儲存]  按鈕。

    ![設定單一登入](./media/fuze-tutorial/tutorial_general_400.png)

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/fuze-tutorial/tutorial_fuze_05.png) 

1. 若要在 **Fuze** 端設定單一登入，您必須將已下載的**中繼資料 XML** 傳送給 [Fuze 支援小組](https://www.fuze.com/support)。 他們會進行設定，讓 SAML SSO 連線在兩端都正確設定。


### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節目標是在 Azure 管理入口網站中建立名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 管理入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/fuze-tutorial/create_aaduser_01.png) 

1. 移至 [使用者和群組]，然後按一下 [所有使用者] 以顯示使用者清單。
    
    ![建立 Azure AD 測試使用者](./media/fuze-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。
 
    ![建立 Azure AD 測試使用者](./media/fuze-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/fuze-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。 


### <a name="creating-a-fuze-test-user"></a>建立 Fuze 測試使用者

Fuze 應用程式僅在使用者佈建時完整支援，所以當他們登入時會自動建立。 如需任何其他詳細資料，請連絡 Fuze [支援](https://www.fuze.com/support)。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Fuze 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Fuze，請執行下列步驟：**

1. 在 Azure 管理入口網站中，開啟應用程式檢視，然後瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Fuze]。

    ![設定單一登入](./media/fuze-tutorial/tutorial_fuze_50.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    

### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Fuze 圖格時，應該會自動登入您的 Fuze 應用程式。


## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/fuze-tutorial/tutorial_general_01.png
[2]: ./media/fuze-tutorial/tutorial_general_02.png
[3]: ./media/fuze-tutorial/tutorial_general_03.png
[4]: ./media/fuze-tutorial/tutorial_general_04.png

[100]: ./media/fuze-tutorial/tutorial_general_100.png

[200]: ./media/fuze-tutorial/tutorial_general_200.png
[201]: ./media/fuze-tutorial/tutorial_general_201.png
[202]: ./media/fuze-tutorial/tutorial_general_202.png
[203]: ./media/fuze-tutorial/tutorial_general_203.png