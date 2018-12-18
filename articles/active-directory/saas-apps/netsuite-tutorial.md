---
title: 教學課程：Azure Active Directory 與 NetSuite 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 NetSuite 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: dafa0864-aef2-4f5e-9eac-770504688ef4
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2018
ms.author: jeedes
ms.openlocfilehash: 511fdcf587d16a59ff2bb11dfc55504b2218a569
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39431399"
---
# <a name="tutorial-azure-active-directory-integration-with-netsuite"></a>教學課程：Azure Active Directory 與 NetSuite 整合

在本教學課程中，您會了解如何整合 NetSuite 與 Azure Active Directory (Azure AD)。

NetSuite 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 NetSuite 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 NetSuite (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 NetSuite 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 NetSuite 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 NetSuite
1. 設定並測試 Azure AD 單一登入

## <a name="adding-netsuite-from-the-gallery"></a>從資源庫新增 NetSuite
若要設定 NetSuite 與 Azure AD 整合，您需要從資源庫將 NetSuite 新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 NetSuite，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]

1. 按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，鍵入 **NetSuite**，從結果面板中選取 **NetSuite**，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 NetSuite](./media/netsuite-tutorial/tutorial_netsuite_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 NetSuite 來設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 NetSuite 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 NetSuite 中的相關使用者之間，建立連結關聯性。

建立此連結關聯性的方法，就是將 Azure AD 中**使用者名稱**的值指派為 NetSuite 中 **Username** 的值。

若要使用 NetSuite 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 NetSuite 測試使用者](#creating-a-netsuite-test-user)** - 讓 NetSuite 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 NetSuite 應用程式中設定單一登入。

**若要使用 NetSuite 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [NetSuite] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![設定單一登入](./media/NetSuite-tutorial/tutorial_NetSuite_samlbase.png)

1. 在 [NetSuite 網域及 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/NetSuite-tutorial/tutorial_NetSuite_url.png)

    在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：

    `https://<tenant-name>.NetSuite.com/saml2/acs`

    `https://<tenant-name>.na1.NetSuite.com/saml2/acs`

    `https://<tenant-name>.na2.NetSuite.com/saml2/acs`

    `https://<tenant-name>.sandbox.NetSuite.com/saml2/acs`

    `https://<tenant-name>.na1.sandbox.NetSuite.com/saml2/acs`

    `https://<tenant-name>.na2.sandbox.NetSuite.com/saml2/acs`
    
    > [!NOTE]
    > 這些不是真正的值。 請使用實際的「回覆 URL」來更新這些值。 請連絡 [NetSuite 支援小組](http://www.NetSuite.com/portal/services/support.shtml)以取得這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/NetSuite-tutorial/tutorial_NetSuite_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/NetSuite-tutorial/tutorial_general_400.png)

1. 在 [NetSuite 組態] 區段上，按一下 [設定 NetSuite] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/NetSuite-tutorial/tutorial_NetSuite_configure.png)

1. 在瀏覽器中開啟新索引標籤，以系統管理員的身分登入您的 NetSuite 公司網站。

1. 在頁面頂端的工具列中，按一下 [設定]，巡覽至 [公司]，然後按一下 [啟用功能]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-setupsaml.png)

1. 在頁面中間的工具列中，按一下 [SuiteCloud]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-suitecloud.png)

1. 在 [管理驗證] 區段底下，選取 [SAML 單一登入] 以在 NetSuite 中啟用 [SAML 單一登入] 選項。

    ![設定單一登入](./media/NetSuite-tutorial/ns-ticksaml.png)

1. 在頁面頂端的工具列上，按一下 [設定]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-setup.png)

1. 從 [設定工作] 清單中，按一下 [整合]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-integration.png)

1. 在 [管理驗證] 區段中，按一下 [SAML 單一登入]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-saml.png)

1. 在 [SAML 設定] 頁面上的 [NetSuite 組態] 區段底下，執行下列步驟：

    ![設定單一登入](./media/NetSuite-tutorial/ns-saml-setup.png)
  
    a. 選取 [主要驗證方法]。

    b. 對標示為 [SAMLV2 識別提供者中繼資料] 的欄位，選取 [上傳 IDP 中繼資料檔案]。 然後按一下 [瀏覽] ，上傳您從 Azure 入口網站下載的中繼資料檔案。

    c. 按一下 [提交] 。

1. 在 Azure AD 中，按一下 [檢視及編輯所有其他使用者屬性] 核取方塊並新增屬性。

    ![設定單一登入](./media/NetSuite-tutorial/ns-attributes.png)

1. 在 [屬性名稱] 欄位中，輸入 `account`。 在 [屬性值]  欄位中，輸入您的 NetSuite 帳戶識別碼。 這個值是常數，並會隨著帳戶變更。 如何找出帳戶識別碼的指示如下：

    ![設定單一登入](./media/NetSuite-tutorial/ns-add-attribute.png)

    a. 在 NetSuite 中，按一下 [設定]，巡覽至 [公司]，然後從頂端導覽功能表按一下 [公司資訊]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-com.png)

    ![設定單一登入](./media/NetSuite-tutorial/ns-account-id.png)

    b. 在右側資料行上的 [公司資訊]頁面上，複製 [帳戶識別碼]。

    c. 將您從 NetSuite 帳戶複製的 [帳戶識別碼] 貼至 Azure AD 中的 [屬性值] 欄位中。 

1. 在使用者可以執行單一登入 NetSuite 之前，必須先在 NetSuite 中指派適當的權限。 請依照下列指示來指派這些權限。

    a. 在頂端的導覽功能表上，按一下 [設定]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-setup.png)

    b. 在左方的瀏覽功能表上，選取 [使用者/角色]，然後按一下 [管理角色]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-manage-roles.png)

    c. 按一下 [新角色] 。

    d. 針對新角色輸入**名稱**。

    ![設定單一登入](./media/NetSuite-tutorial/ns-new-role.png)

    e. 按一下 [檔案] 。

    f. 在頂端的功能表中，按一下 [權限] 。 然後按一下 [設定] 。

    ![設定單一登入](./media/NetSuite-tutorial/ns-sso.png)

    g. 選取 [SAML 單一登入]，然後按一下 [新增]。

    h. 按一下 [檔案] 。

    i. 在頂端的瀏覽功能表上，按一下 [設定]，然後按一下 [設定管理員]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-setup.png)

    j. 在左方的瀏覽功能表上，選取 [使用者/角色]，然後按一下 [管理使用者]。

    ![設定單一登入](./media/NetSuite-tutorial/ns-manage-users.png)

    k. 選取測試使用者。 按一下 [編輯]，然後巡覽至 [存取] 索引標籤。

    ![設定單一登入](./media/NetSuite-tutorial/ns-edit-user.png)

    l. 在 [角色] 對話方塊中，指派您已建立的適當角色。

    ![設定單一登入](./media/NetSuite-tutorial/ns-add-role.png)

    m. 按一下 [檔案] 。
 
### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/NetSuite-tutorial/create_aaduser_01.png) 

1.  若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/NetSuite-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。
 
    ![建立 Azure AD 測試使用者](./media/NetSuite-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/NetSuite-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。 

### <a name="creating-a-netsuite-test-user"></a>建立 NetSuite 測試使用者

本節會在 NetSuite 中建立名為 Britta Simon 的使用者。 NetSuite 支援預設啟用的 Just-In-Time 佈建。
在這一節沒有您需要進行的動作項目。 嘗試存取 NetSuite 時若尚無使用者，則會建立新使用者。


### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 NetSuite 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 NetSuite，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [NetSuite]。

    ![設定單一登入](./media/NetSuite-tutorial/tutorial_NetSuite_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

若要測試您的單一登入設定，請開啟位於 [https://myapps.microsoft.com](https://myapps.microsoft.com/) 的存取面板、登入測試帳戶，然後按一下 [NetSuite]。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](NetSuite-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/NetSuite-tutorial/tutorial_general_01.png
[2]: ./media/NetSuite-tutorial/tutorial_general_02.png
[3]: ./media/NetSuite-tutorial/tutorial_general_03.png
[4]: ./media/NetSuite-tutorial/tutorial_general_04.png

[100]: ./media/NetSuite-tutorial/tutorial_general_100.png

[200]: ./media/NetSuite-tutorial/tutorial_general_200.png
[201]: ./media/NetSuite-tutorial/tutorial_general_201.png
[202]: ./media/NetSuite-tutorial/tutorial_general_202.png
[203]: ./media/NetSuite-tutorial/tutorial_general_203.png

