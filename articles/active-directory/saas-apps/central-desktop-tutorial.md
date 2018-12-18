---
title: 教學課程：Azure Active Directory 與 Central Desktop 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Central Desktop 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: b805d485-93db-49b4-807a-18d446c7090e
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/08/2017
ms.author: jeedes
ms.openlocfilehash: 82a6911c85dd1438aa8f60cb36194a2916bc91e7
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39429041"
---
# <a name="tutorial-azure-active-directory-integration-with-central-desktop"></a>教學課程：Azure Active Directory 與 Central Desktop 整合

在本教學課程中，您將了解如何整合 Central Desktop 與 Azure Active Directory (Azure AD)。

將 Central Desktop 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Central Desktop 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Central Desktop。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需有關 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Central Desktop 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Central Desktop 單一登入的訂用帳戶

> [!NOTE]
> 我們不建議使用生產環境來測試本教學課程中的步驟。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您還沒有 Azure AD 試用環境，您可以[取得一個月免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫新增 Central Desktop
1. 設定並測試 Azure AD 單一登入

## <a name="add-central-desktop-from-the-gallery"></a>從資源庫新增 Central Desktop
若要設定將 Central Desktop 整合到 Azure AD 中，您需要將 Central Desktop 從資源庫新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Central Desktop，請執行下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Central Desktop**。 從結果面板中選取 [Central Desktop]，然後選取 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Central Desktop](./media/central-desktop-tutorial/tutorial_centraldesktop_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Central Desktop 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Central Desktop 與 Azure AD 中互相對應的使用者。 換句話說，您必須在 Azure AD 使用者與 Central Desktop 中的相關使用者之間建立連結。

在 Central Desktop 中，為 [Username] \(使用者名稱\) 提供與 Azure AD 中**使用者名稱**相同的值。 現在您已經建立兩名使用者之間的連結。

若要設定及測試與 Central Desktop 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. [設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)，讓您的使用者能夠使用此功能。
1. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
1. [建立 Central Desktop 測試使用者](#create-a-central-desktop-test-user)，以在 Central Desktop 中建立一個與 Azure AD 中代表 Britta Simon 之使用者連結的 Britta Simon 對應項目。
1. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. [測試單一登入](#test-single-sign-on)，以驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Central Desktop 應用程式中設定單一登入。

**若要設定與 Central Desktop 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Central Desktop] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

1. 若要啟用單一登入，請在 [單一登入] 對話方塊的 [模式] 下拉式清單中，選取 [SAML 登入]。
 
    ![單一登入對話方塊](./media/central-desktop-tutorial/tutorial_centraldesktop_samlbase.png)

1. 在 [Central Desktop 網域及 URL] 區段中，執行下列步驟：

    ![Central Desktop 網域及 URL 單一登入資訊](./media/central-desktop-tutorial/tutorial_centraldesktop_url.png)

    a. 在 [登入 URL] 方塊中，以下列模式輸入 URL：`https://<companyname>.centraldesktop.com`

    b. 在 [識別碼] 方塊中，以下列模式輸入 URL：
    | |
    |--|
    | `https://<companyname>.centraldesktop.com/saml2-metadata.php`|
    | `https://<companyname>.imeetcentral.com/saml2-metadata.php`|

    c. 在 [回覆 URL] 方塊中，使用下列模式輸入 URL：`https://<companyname>.centraldesktop.com/saml2-assertion.php`    
     
    > [!NOTE] 
    > 這些都不是真正的值。 請使用實際的識別碼、回覆 URL 及登入 URL 來更新這些值。 請連絡 [Central Desktop 用戶端支援小組](https://imeetcentral.com/contact-us) \(英文\) 以取得這些值。 

1. 在 [SAML 簽署憑證] 區段中，選取 [憑證]。 然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/central-desktop-tutorial/tutorial_centraldesktop_certificate.png) 

1. 選取 [儲存] 按鈕。

    ![設定單一登入儲存按鈕](./media/central-desktop-tutorial/tutorial_general_400.png)
    
1. 在 [Central Desktop 設定] 區段中，選取 [設定 Central Desktop] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![Central Desktop 設定](./media/central-desktop-tutorial/tutorial_centraldesktop_configure.png) 

1. 登入您的 **Central Desktop** 租用戶。

1. 移至 [設定] 。 選取 [Advanced] \(進階\)，然後選取 [Single Sign On] \(單一登入\)。

    ![設定 - 進階](./media/central-desktop-tutorial/ic769563.png "設定 - 進階")

1. 在 [Single Sign On Settings] \(單一登入設定\) 頁面上，執行下列步驟：

    ![單一登入設定](./media/central-desktop-tutorial/ic769564.png "單一登入設定")
    
    a. 選取 [啟用 SAML v2 單一登入] 。
    
    b. 在 [SSO URL] 方塊中，貼上您從 Azure 入口網站複製的 **SAML 實體識別碼**值。
    
    c. 在 [SSO Login URL] \(SSO 登入 URL\) 方塊中，貼上您從 Azure 入口網站複製的 **SAML 單一登入服務 URL**。
    
    d. 在 [SSO Logout URL] \(SSO 登出 URL\) 方塊中，貼上您從 Azure 入口網站複製的**登出 URL**。

1. 在 [Message Signature Verification Method] \(訊息簽章驗證方法\) 區段中，執行下列步驟：

    ![訊息簽章驗證方法](./media/central-desktop-tutorial/ic769565.png "訊息簽章驗證方法") 選取 [憑證] 。
    
    b. 在 [SSO Certificate] \(SSO 憑證\) 清單中，選取 [RSH SHA256]。
    
    c. 在「記事本」中開啟所下載的憑證。 接著，複製憑證的內容並將它貼到 [SSO Certificate] \(SSO 憑證\) 欄位中。
        
    d. 選取 [顯示 SAMLv2 登入頁面的連結] 。
    
    e. 選取 [更新]。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本。 從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，請選取 [單一登入] 索引標籤，然後透過底部的 [設定] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請採取下列步驟：**

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/central-desktop-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]。 然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/central-desktop-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，請選取 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/central-desktop-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，採取下列步驟：

    ![[使用者] 對話方塊](./media/central-desktop-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-a-central-desktop-test-user"></a>建立 Central Desktop 測試使用者

若要讓 Azure AD 使用者能夠登入，必須先在 Central Desktop 應用程式中佈建他們。 本節說明如何在 Central Desktop 中建立 Azure AD 使用者帳戶。

> [!NOTE]
> 若要佈建 Azure AD 使用者帳戶，您可以使用任何其他 Central Desktop 使用者帳戶建立工具，或是 Central Desktop 所提供的 API。

**將使用者帳戶佈建到 Central Desktop：**

1. 登入您的 Central Desktop 租用戶。

1. 移至 [People] \(人員\) > [Internal Members] \(內部成員\)。

1. 選取 [Add Internal Members] \(新增內部成員\)。

    ![People](./media/central-desktop-tutorial/ic781051.png "People")
    
1. 在 [Email Address of New Members] \(新成員的電子郵件地址\) 方塊中，輸入您想要佈建的 Azure AD 帳戶，然後選取 [Next] \(下一步\)。

    ![新成員的電子郵件地址](./media/central-desktop-tutorial/ic781052.png "新成員的電子郵件地址")

1. 選取 [Add Internal member(s)] \(新增內部成員\)。

    ![新增內部成員](./media/central-desktop-tutorial/ic781053.png "新增內部成員")
   
   >[!NOTE]
   >您新增的使用者會收到一封電子郵件，其中包含用來啟用其帳戶的確認連結。
   
### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Central Desktop 的存取權授與使用者 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Central Desktop，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視。 移至目錄檢視，然後移至 [企業應用程式]。

1. 選取 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Central Desktop]。

    ![應用程式清單中的 Central Desktop 連結](./media/central-desktop-tutorial/tutorial_centraldesktop_app.png)  

1. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增] 按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊中，選取 [使用者] 清單中的 [Britta Simon]。

1. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，請使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中選取 [Central Desktop] 圖格時，會自動登入您的 Central Desktop 應用程式。
如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/central-desktop-tutorial/tutorial_general_01.png
[2]: ./media/central-desktop-tutorial/tutorial_general_02.png
[3]: ./media/central-desktop-tutorial/tutorial_general_03.png
[4]: ./media/central-desktop-tutorial/tutorial_general_04.png

[100]: ./media/central-desktop-tutorial/tutorial_general_100.png

[200]: ./media/central-desktop-tutorial/tutorial_general_200.png
[201]: ./media/central-desktop-tutorial/tutorial_general_201.png
[202]: ./media/central-desktop-tutorial/tutorial_general_202.png
[203]: ./media/central-desktop-tutorial/tutorial_general_203.png

