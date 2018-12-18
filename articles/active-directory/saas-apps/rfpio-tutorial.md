---
title: 教學課程：Azure Active Directory 與 RFPIO 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 RFPIO 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 87187076-7b50-4247-814f-f217b052703f
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/16/2017
ms.author: jeedes
ms.openlocfilehash: 04ba94e3263af03279b74b4832b8291ad6414274
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39426555"
---
# <a name="tutorial-azure-active-directory-integration-with-rfpio"></a>教學課程：Azure Active Directory 與 RFPIO 整合

在本教學課程中，您會了解如何整合 RFPIO 與 Azure Active Directory (Azure AD)。

RFPIO 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 RFPIO 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 RFPIO (單一登入)。
- 您可以在 Azure 入口網站集中管理您的帳戶。

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 RFPIO 整合，您需要下列項目：

- Azure AD 訂用帳戶。
- 已啟用 RFPIO 單一登入的訂用帳戶。

> [!NOTE]
> 我們不建議使用生產環境來測試本教學課程中的步驟。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以取得[一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫新增 RFPIO。
1. 設定並測試 Azure AD 單一登入。

## <a name="add-rfpio-from-the-gallery"></a>從資源庫新增 RFPIO
若要設定將 RFPIO 整合到 Azure AD 中，您需要從資源庫將 RFPIO 新增到受控 SaaS 應用程式清單。

### <a name="to-add-rfpio-from-the-gallery"></a>從資源庫新增 RFPIO

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，選取 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **RFPIO**。

    ![建立 Azure AD 測試使用者](./media/rfpio-tutorial/tutorial_rfpio_search.png)

1. 在結果窗格中，選取 [RFPIO]，然後選取 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/rfpio-tutorial/tutorial_rfpio_addfromgallery.png)

##  <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 RFPIO 設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 RFPIO 中的對應使用者與 Azure AD 中的使用者之間的關聯性。 換句話說，必須建立 Azure AD 使用者和 RFPIO 中相關使用者之間的連結關聯性。

在 RFPIO 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 RFPIO 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** -- 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** -- 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 RFPIO 測試使用者](#creating-a-rfpio-test-user)** -- 在 RFPIO 中建立一個與 Azure AD 中代表使用者的項目連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** -- 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#testing-single-sign-on)** -- 驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 RFPIO 應用程式中設定單一登入。

**若要設定與 RFPIO 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [RFPIO] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/rfpio-tutorial/tutorial_rfpio_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [RFPIO 網域和 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/rfpio-tutorial/tutorial_rfpio_url.png)

    a. 在 [識別碼] 文字方塊中，輸入 URL：`https://www.rfpio.com`

    ![設定單一登入](./media/rfpio-tutorial/tutorial_rfpio_url1.png)

    b. 按一下 [顯示進階 URL 設定]。

    c. 在 [回覆狀態] 文字方塊中輸入一個字串值。 請連絡 [RFPIO 支援小組](https://www.rfpio.com/contact/)以取得此值。 

1. 按一下 [顯示進階 URL 設定]。 如果您想要以 **SP** 起始模式設定應用程式： 

    ![設定單一登入](./media/rfpio-tutorial/tutorial_rfpio_url2.png)

    在 [登入 URL] 文字方塊中，輸入 URL：`https://www.app.rfpio.com`

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/rfpio-tutorial/tutorial_rfpio_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/rfpio-tutorial/tutorial_general_400.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 **RFPIO** 網站。

1. 按一下左上角的下拉式清單。

    ![設定單一登入](./media/rfpio-tutorial/app1.png)

1. 按一下 [組織設定]。 

    ![設定單一登入](./media/rfpio-tutorial/app2.png)

1. 按一下 [功能和整合]。

    ![設定單一登入](./media/rfpio-tutorial/app4.png)

1. 在 [SAML SSO 組態] 中按一下 [編輯]。

    ![設定單一登入](./media/rfpio-tutorial/app3.png)

1. 在本節中執行下列動作：

    ![設定單一登入](./media/rfpio-tutorial/app5.png)
    
    a. 複製 [下載的中繼資料 XML] 內容，然後貼到 [身分識別組態] 欄位中。

    > [!NOTE]
    >若要複製下載的 [中繼資料 XML] 內容，請使用 [Notepad++] 或適當的 [XML 編輯器]。 

    b. 按一下 [驗證] 。

    c. 按一下 [驗證] 後，將 [SAML (已啟用)] 翻轉為開啟狀態。

    d. 按一下 [提交] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/rfpio-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/rfpio-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/rfpio-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/rfpio-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-rfpio-test-user"></a>建立 RFPIO 測試使用者

若要讓 Azure AD 使用者登入 RFPIO，必須將他們佈建到 RFPIO。  
RFPIO 需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 RFPIO 公司網站。

1. 按一下左上角的下拉式清單。

    ![設定單一登入](./media/rfpio-tutorial/app1.png)

1. 按一下 [組織設定]。 

    ![設定單一登入](./media/rfpio-tutorial/app2.png)

1. 按一下 [小組成員]。

    ![設定單一登入](./media/rfpio-tutorial/app6.png)

1. 按一下 [新增成員]。

    ![設定單一登入](./media/rfpio-tutorial/app7.png)

1. 在 [新增成員] 區段中。 執行下列動作：

    ![設定單一登入](./media/rfpio-tutorial/app8.png)

    a. 在 [每行輸入一個電子郵件] 欄位中輸入[電子郵件地址]。

    b. 請根據您的需求選取 [角色]。

    c. 按一下[新增成員]。
        
    > [!NOTE]
    > Azure Active Directory 帳戶的持有者會收到一封電子郵件，並依照連結在啟用其帳戶前進行確認。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 RFPIO 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 RFPIO，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [RFPIO]。

    ![設定單一登入](./media/rfpio-tutorial/tutorial_rfpio_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中按一下 RFPIO 圖格時，應該會自動登入您的 RFPIO 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [有關如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/rfpio-tutorial/tutorial_general_01.png
[2]: ./media/rfpio-tutorial/tutorial_general_02.png
[3]: ./media/rfpio-tutorial/tutorial_general_03.png
[4]: ./media/rfpio-tutorial/tutorial_general_04.png

[100]: ./media/rfpio-tutorial/tutorial_general_100.png

[200]: ./media/rfpio-tutorial/tutorial_general_200.png
[201]: ./media/rfpio-tutorial/tutorial_general_201.png
[202]: ./media/rfpio-tutorial/tutorial_general_202.png
[203]: ./media/rfpio-tutorial/tutorial_general_203.png

