---
title: 教學課程：Azure Active Directory 與 Marketo 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Marketo 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: b88c45f5-d288-4717-835c-ca965add8735
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/09/2017
ms.author: jeedes
ms.openlocfilehash: bd647601249e22942596e78b66d0322857f3eaa4
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39448123"
---
# <a name="tutorial-azure-active-directory-integration-with-marketo"></a>教學課程：Azure Active Directory 與 Marketo 整合

在本教學課程中，您會了解如何整合 Marketo 與 Azure Active Directory (Azure AD)。

Marketo 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Marketo 的人員
- 您可以讓使用者利用自己的 Azure AD 帳戶，來自動登入 Marketo (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Marketo 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Marketo 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Marketo
1. 設定並測試 Azure AD 單一登入

## <a name="adding-marketo-from-the-gallery"></a>從資源庫新增 Marketo
若要設定 Marketo 與 Azure AD 整合，您需要從資源庫將 Marketo 新增到受控 SaaS 應用程式清單。

**若要從資源庫加入 Marketo，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Marketo**。

    ![建立 Azure AD 測試使用者](./media/marketo-tutorial/tutorial_marketo_search.png)

1. 在結果面板中，選取 [Marketo]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/marketo-tutorial/tutorial_marketo_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您將以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Marketo 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Marketo 與 Azure AD 中互相對應的使用者。 換句話說，必須要建立某位 Azure AD 使用者與 Marketo 中相關使用者之間的連結關聯性。

在 Marketor 中，將 Azure AD 中 [使用者名稱]的值指派為 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 Marketo 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Marketo 測試使用者](#creating-a-marketo-test-user)** - 使 Marketo 中 Britta Simon 的對應使用者連結到該使用者在 Azure AD 中的代表身分。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您將在 Azure 入口網站中啟用 Azure AD 單一登入，並在 Marketo 應用程式中設定單一登入。

**若要使用 Marketo 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Marketo] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_samlbase.png)

1. 在 [Marketo 網域及 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_url.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://saml.marketo.com/sp`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://login.marketo.com/saml/assertion/\<munchkinid\>`

    > [!NOTE] 
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 更新這些值。 請連絡 [Marketo 支援小組](http://investors.marketo.com/contactus.cfm)以取得這些值。
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/marketo-tutorial/tutorial_general_400.png)

1. 在 [Marketo 設定] 區段中，按一下 [設定 Marketo] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_configure.png) 

1. 若要取得您的應用程式的 Munchkin 識別碼，請使用系統管理員認證登入 Marketo，執行以下動作︰
   
    a. 使用管理員認證登入 Marketo 應用程式。
   
    b. 按一下頂端瀏覽窗格中的 [管理] 按鈕。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 瀏覽至 [整合] 功能表，然後按一下 [Munchkin] 連結。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_11.png)
   
    d. 複製螢幕上顯示的 Munchkin 識別碼，完成 Azure AD 設定精靈中的回覆 URL。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_12.png) 

1. 若要設定應用程式中的 SSO，請遵循下列步驟︰
   
    a. 使用管理員認證登入 Marketo 應用程式。
   
    b. 按一下頂端瀏覽窗格中的 [管理] 按鈕。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 瀏覽至 [整合] 功能表，然後按一下 [單一登入] 。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_07.png) 
   
    d. 若要啟用 SAML 設定，按一下 [編輯]按鈕。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_08.png) 
   
    e. **已啟用**單一登入設定。
   
    f. 在 [簽發者識別碼] 文字方塊中，貼上 [SAML 實體識別碼]。
   
    g. 在 [實體識別碼] 文字方塊中，輸入 URL `http://saml.marketo.com/sp`。
   
    h. 選取 [使用者識別碼位置] 做為 [名稱識別碼元素]。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_09.png)
   
    > [!NOTE]
    > 如果您的使用者識別碼不是 UPN 值，則至 [屬性] 索引標籤中變更其值。
   
    i. 上傳您從 Azure AD 設定精靈下載的憑證。 **儲存**設定。
   
    j. 編輯 [重新導向頁面] 設定。
   
    k. 在 [登入 URL] 文字方塊中，貼上 **SAML 單一登入服務 URL**。
   
    l. 在 [登出 URL] 文字方塊中，貼上登出 URL。
   
    m. 在 [錯誤 URL] 中，複製您的 **Marketo 執行個體 URL**，並按一下 [儲存] 按鈕以儲存設定。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_10.png)

1. 若要啟用使用者的 SSO，完成下列動作：
   
    a. 使用管理員認證登入 Marketo 應用程式。
   
    b. 按一下頂端瀏覽窗格中的 [管理] 按鈕。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 瀏覽至 [安全性] 功能表，然後按一下 [登入設定]。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_13.png)
   
    d. 勾選 [需要 SSO] 選項，並 [儲存]設定。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_14.png)

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/marketo-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/marketo-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/marketo-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/marketo-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-marketo-test-user"></a>建立 Marketo 測試使用者

在本節中，您要在 Marketo 中建立名為 Britta Simon 的使用者。 遵循以下步驟在 Marketo 平台中建立使用者。

1. 使用管理員認證登入 Marketo 應用程式。

1. 按一下頂端瀏覽窗格中的 [管理] 按鈕。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_06.png) 

1. 瀏覽至 [安全性] 功能表，然後按一下 [使用者與角色]
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_19.png)  

1. 按一下 [使用者] 索引標籤上的 [邀請新使用者]  連結
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_15.png) 

1. 在 [邀請新使用者] 精靈中填寫下列資訊。
   
    a. 在 [電子郵件] 文字方塊中輸入使用者的電子郵件地址。
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_16.png)
   
    b. 在 [名字] 文字方塊中輸入名字。
   
    c. 在 [姓氏] 文字方塊中輸入姓氏。
   
    d. 按一下 [下一步] 

1. 在 [權限]  索引標籤中選取 [userRoles]，然後按一下 [下一步]
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_17.png)
1. 按一下 [傳送] 按鈕，傳送使用者邀請
   
    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_18.png)

1. 使用者會收到電子郵件通知，他必須按一下連結，並變更密碼才能啟動帳戶。 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您將把 Marketo 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 Marketo，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Marketo] 。

    ![設定單一登入](./media/marketo-tutorial/tutorial_marketo_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Marketo] 圖格時，應該會自動登入您的 Marketo 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/marketo-tutorial/tutorial_general_01.png
[2]: ./media/marketo-tutorial/tutorial_general_02.png
[3]: ./media/marketo-tutorial/tutorial_general_03.png
[4]: ./media/marketo-tutorial/tutorial_general_04.png

[100]: ./media/marketo-tutorial/tutorial_general_100.png

[200]: ./media/marketo-tutorial/tutorial_general_200.png
[201]: ./media/marketo-tutorial/tutorial_general_201.png
[202]: ./media/marketo-tutorial/tutorial_general_202.png
[203]: ./media/marketo-tutorial/tutorial_general_203.png

