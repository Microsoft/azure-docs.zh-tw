---
title: 教學課程：Azure Active Directory 與 TimeOffManager 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 TimeOffManager 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 3685912f-d5aa-4730-ab58-35a088fc1cc3
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2017
ms.author: jeedes
ms.openlocfilehash: e7f0f6bb778dedeea61b74b5ca0c2edbadd5279b
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39430282"
---
# <a name="tutorial-azure-active-directory-integration-with-timeoffmanager"></a>教學課程：Azure Active Directory 與 TimeOffManager 整合

在本教學課程中，您將了解如何整合 TimeOffManager 與 Azure Active Directory (Azure AD)。

TimeOffManager 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中管控可存取 TimeOffManager 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 TimeOffManager (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 TimeOffManager 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 TimeOffManager 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 TimeOffManager
1. 設定和測試 Azure AD 單一登入

## <a name="add-timeoffmanager-from-the-gallery"></a>從資源庫新增 TimeOffManager
若要設定將 TimeOffManager 整合到 Azure AD 中，您需要從資源庫將 TimeOffManager 新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 TimeOffManager，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **TimeOffManager**，從結果面板中選取 [TimeOffManager]，然後按一下 [新增] 按鈕以新增應用程式。

    ![從資源庫新增](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_addfromgallery.png)

##  <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 TimeOffManager 設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 TimeOffManager 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 TimeOffManager 中的相關使用者之間建立連結關聯性。

在 TimeOffManager 中，將 Azure AD 中 [使用者名稱] 的值指派為 **Username** 的值，以建立連結關聯性。

若要使用 TimeOffManager 來設定並測試 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 TimeOffManager 測試使用者](#create-a-timeoffmanager-test-user)** - 使 TimeOffManager 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 TimeOffManager 應用程式中設定單一登入。

**若要使用 TimeOffManager 設定 Azure AD 單一登入功能，請執行下列步驟：**

1. 在 Azure 入口網站的 [TimeOffManager] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![SAML 型登入](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_samlbase.png)

1. 在 [TimeOffManager 網域與 URL] 區段中，執行下列動作：

     ![[TimeOffManager 網域與 URL] 區段](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_url.png)

    在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://www.timeoffmanager.com/cpanel/sso/consume.aspx?company_id=<companyid>`

    > [!NOTE] 
    > 這不是真實的值。 請使用實際的「回覆 URL」來更新此值。 您可以從 [單一登入設定] 頁面取得此值，稍後會在本教學課程中加以說明，或連絡 [TimeOffManager 支援小組](https://www.purelyhr.com/contact-us)。
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![[SAML 簽署憑證] 區段](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_certificate.png) 

1. 本節的目的是要說明如何依據 SAML 通訊協定來使用同盟，讓使用者能夠用自己在 Azure AD 中的帳戶驗證至 TimeOffManger。
    
    您的 TimeOffManger 應用程式需要特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應加入 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示上述的範例。

    ![saml 權杖屬性](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_attrb.png "saml 權杖屬性")
    
    | 屬性名稱 | 屬性值 |
    | --- | --- |
    | Firstname |User.givenname |
    | lastname |User.surname |
    | 電子郵件 |User.mail |
    
    a.  針對上表中的每個資料列，按一下 [新增使用者屬性] 。
    
    ![saml 權杖屬性](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_addattrb.png "saml 權杖屬性")
    
    ![saml 權杖屬性](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_addattrb1.png "saml 權杖屬性")
    
    b.  在 [屬性名稱]  文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    c.  在 [屬性值] 文字方塊中，選取該資料列所顯示的屬性值。
    
    d.  按一下 [確定] 。
    
1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/timeoffmanager-tutorial/tutorial_general_400.png)

1. 在 [TimeOffManager 設定] 區段上，按一下 [設定 TimeOffManager] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![[TimeOffManager 設定] 區段](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 TimeOffManager 公司網站。

1. 移至 [帳戶] \> [帳戶選項] \> [單一登入設定]。
   
   ![單一登入設定](./media/timeoffmanager-tutorial/ic795917.png "單一登入設定")
1. 在 [單一登入設定]  區段中，執行下列步驟：
   
   ![單一登入設定](./media/timeoffmanager-tutorial/ic795918.png "單一登入設定")
   
   a. 在記事本中開啟您的 base-64 編碼的憑證，將它的內容複製到您的剪貼簿，然後將整個憑證貼到 [X.509 憑證]  文字方塊中。
   
   b. 在 [IdP 簽發者] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。
   
   c. 在 [IdP 端點 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。
   
   d. 在 [強制 SAML] 選取 [否]。
   
   e. 在 [自動建立使用者] 選取 [是]。
   
   f. 在 [登出 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。
   
   g. 按一下 [儲存變更]。

1. 在 [單一登入設定] 頁面中，複製 [判斷提示取用者服務 URL] 的值，並將其貼在 Azure 入口網站之 [TimeOffManager 網域與 URL] 區段下的 [回覆 URL] 文字方塊中。 

      ![單一登入設定](./media/timeoffmanager-tutorial/ic795915.png "單一登入設定")

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/timeoffmanager-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![[使用者和群組] --> [所有使用者]](./media/timeoffmanager-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![[新增] 按鈕](./media/timeoffmanager-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![使用者對話方塊頁面](./media/timeoffmanager-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-timeoffmanager-test-user"></a>建立 TimeOffManager 測試使用者

若要讓 Azure AD 使用者可以登入 TimeOffManager，則必須將他們佈建到 TimeOffManager。  

TimeOffManager 支援即時使用者佈建。 沒有您適用的動作項目。  

在第一次登入時使用單一登入，便會自動加入使用者。

>[!NOTE]
>您可以使用任何其他的 TimeOffManager 使用者帳戶建立工具或 TimeOffManager 提供的 API 來佈建 Azure AD 使用者帳戶。
> 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 TimeOffManager 的存取權授予 Britta Simon，讓 Britta Simon 能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 TimeOffManager，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [TimeOffManager]。

    ![應用程式清單中的 TimeOffManager](./media/timeoffmanager-tutorial/tutorial_timeoffmanager_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 TimeOffManager 圖格時，應該會自動登入您的 TimeOffManager 應用程式。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/timeoffmanager-tutorial/tutorial_general_01.png
[2]: ./media/timeoffmanager-tutorial/tutorial_general_02.png
[3]: ./media/timeoffmanager-tutorial/tutorial_general_03.png
[4]: ./media/timeoffmanager-tutorial/tutorial_general_04.png

[100]: ./media/timeoffmanager-tutorial/tutorial_general_100.png

[200]: ./media/timeoffmanager-tutorial/tutorial_general_200.png
[201]: ./media/timeoffmanager-tutorial/tutorial_general_201.png
[202]: ./media/timeoffmanager-tutorial/tutorial_general_202.png
[203]: ./media/timeoffmanager-tutorial/tutorial_general_203.png

