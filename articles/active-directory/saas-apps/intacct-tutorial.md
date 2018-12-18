---
title: 教學課程：Azure Active Directory 與 Intacct 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Intacct 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 92518e02-a62c-4b1b-a8e9-2803eb2b49ac
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/23/2017
ms.author: jeedes
ms.openlocfilehash: d834ca75085878350e257cc1c50e60fc1bf28484
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39447334"
---
# <a name="tutorial-azure-active-directory-integration-with-intacct"></a>教學課程：Azure Active Directory 與 Intacct 整合

在本教學課程中，您會了解如何整合 Intacct 與 Azure Active Directory (Azure AD)。

Intacct 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Intacct 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Intacct (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Intacct 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Intacct 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Intacct
1. 設定並測試 Azure AD 單一登入

## <a name="adding-intacct-from-the-gallery"></a>從資源庫新增 Intacct
若要設定將 Intacct 整合到 Azure AD 中，您需要從資源庫將 Intacct 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Intacct，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Intacct**。

    ![建立 Azure AD 測試使用者](./media/intacct-tutorial/tutorial_intacct_search.png)

1. 在結果窗格中，選取 [Intacct]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/intacct-tutorial/tutorial_intacct_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Intacct 設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Intacct 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Intacct 中相關使用者之間的連結關聯性。

在 Intacct 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Intacct 搭配運作的 Azure AD 單一登入，您需要完成下列建構元素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Intacct 測試使用者](#creating-an-intacct-test-user)** - 在 Intacct 中建立 Britta Simon 的對應項目，且該項目必須與 Azure AD 中代表 Britta Simon 的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Intacct 應用程式中設定單一登入。

**若要設定與 Intacct 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Intacct] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/intacct-tutorial/tutorial_intacct_samlbase.png)

1. 在 [Intacct 網域及 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/intacct-tutorial/tutorial_intacct_url.png)

    在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：
    | |
    |--|
    | `https://<companyname>.intacct.com/ia/acct/sso_response.phtml`|
    | `https://www.intacct.com/ia/acct/sso_response.phtml` |

    > [!NOTE] 
    > 這不是真實的值。 請使用實際的「回覆 URL」來更新此值。 請連絡 [Intacct 支援小組](https://us.intacct.com/support)以取得這個值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/intacct-tutorial/tutorial_intacct_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/intacct-tutorial/tutorial_general_400.png)

1. 在 [Intacct 組態] 區段上，按一下 [設定 Intacct] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/intacct-tutorial/tutorial_intacct_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Intacct 公司網站。

1. 按一下 [公司] 索引標籤，然後按一下 [公司資訊]。

    ![公司](./media/intacct-tutorial/ic790037.png "公司")

1. 按一下 [安全性] 索引標籤，然後按一下 [編輯]。

    ![安全性](./media/intacct-tutorial/ic790038.png "安全性")

1. 在 [單一登入 (SSO)]  區段中，執行下列步驟：

    ![單一登入](./media/intacct-tutorial/ic790039.png "單一登入")

    a. 選取 [啟用單一登入] 。

    b. 在 [身分識別提供者類型]，選取 **SAML 2.0**。

    c. 在 [簽發者 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。
   
    d. 在 [登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    e. 在記事本中開啟您的 **base-64** 編碼的憑證，將其內容複製到您的剪貼簿，然後貼到 [憑證] 方塊中。
   
    f. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/intacct-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/intacct-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/intacct-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/intacct-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-an-intacct-test-user"></a>建立 Intacct 測試使用者

若要設定 Azure AD 使用者以便讓他們能夠登入 Intacct，則必須將他們佈建到 Intacct。 在 Intacct 中，佈建是手動工作。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 登入您的 **Intacct** 租用戶。

1. 按一下 [公司] 索引標籤，然後按一下 [使用者]。

    ![使用者](./media/intacct-tutorial/ic790041.png "使用者")
1. 按一下 [新增] 索引標籤。

    ![新增](./media/intacct-tutorial/ic790042.png "新增")
1. 在 [使用者資訊]  區段中，執行下列步驟：

    ![使用者資訊](./media/intacct-tutorial/ic790043.png "使用者資訊")

    a. 在 [使用者資訊] 區段中，為您要佈建的 Azure AD 帳戶輸入 [使用者識別碼]、[姓氏]、[名字]、[電子郵件地址]、[職稱] 和 [電話]。

    b. 選取您要佈建之 Azure AD 帳戶的 [系統管理員權限]。
   
    c. 按一下 [檔案] 。 Azure AD 帳戶的持有者會收到一封電子郵件，並依照連結在啟用其帳戶前進行確認。

>[!NOTE]
>若要佈建 Azure AD 使用者帳戶，您可以使用 Intacct 所提供的其他 Intacct 使用者帳戶建立工具或 API。
        
### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Intacct 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Intacct，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Intacct]。

    ![設定單一登入](./media/intacct-tutorial/tutorial_intacct_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中按一下 [Intacct] 圖格時，應該會自動登入您的 Intacct 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/intacct-tutorial/tutorial_general_01.png
[2]: ./media/intacct-tutorial/tutorial_general_02.png
[3]: ./media/intacct-tutorial/tutorial_general_03.png
[4]: ./media/intacct-tutorial/tutorial_general_04.png

[100]: ./media/intacct-tutorial/tutorial_general_100.png

[200]: ./media/intacct-tutorial/tutorial_general_200.png
[201]: ./media/intacct-tutorial/tutorial_general_201.png
[202]: ./media/intacct-tutorial/tutorial_general_202.png
[203]: ./media/intacct-tutorial/tutorial_general_203.png

