---
title: 教學課程：Azure Active Directory 與 Recognize 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Recognize 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: cfad939e-c8f4-45a0-bd25-c4eb9701acaa
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/10/2017
ms.author: jeedes
ms.openlocfilehash: b2d5acfcb722845d7f346668597c073319f273f0
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39448779"
---
# <a name="tutorial-azure-active-directory-integration-with-recognize"></a>教學課程：Azure Active Directory 與 Recognize 整合

在本教學課程中，您會了解如何整合 Recognize 與 Azure Active Directory (Azure AD)。

將 Recognize 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制誰能夠存取 Recognize
- 您可以讓您的使用者使用其 Azure AD 帳戶自動登入 Recognize (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Recognize 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Recognize 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在這裡取得一個月試用：[試用供應項目](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Recognize
1. 設定並測試 Azure AD 單一登入

## <a name="adding-recognize-from-the-gallery"></a>從資源庫新增 Recognize
若要設定將 Recognize 整合至 Azure AD，您必須將資源庫中的 Recognize 新增至您的受控 SaaS 應用程式清單中。

**若要從資源庫新增 Recognize，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中輸入 **Recognize**。

    ![建立 Azure AD 測試使用者](./media/recognize-tutorial/tutorial_recognize_search.png)

1. 在結果面板中，選取 [Recognize]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/recognize-tutorial/tutorial_recognize_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Recognize 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Recognize 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Recognize 中的相關使用者之間建立連結關聯性。

在 Recognize 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要使用 Recognize 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Recognize 測試使用者](#creating-a-recognize-test-user)** - 使 Recognize 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Recognize 應用程式中設定單一登入。

**若要使用 Recognize 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Recognize] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/recognize-tutorial/tutorial_recognize_samlbase.png)

1. 在 [Recognize 網域及 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/recognize-tutorial/tutorial_recognize_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://recognizeapp.com/<your-domain>/saml/sso`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://recognizeapp.com/<your-domain>`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Recognize 客戶支援小組](mailto:support@recognizeapp.com)以取得登入 URL，而且您可以從 [SSO 設定] 區段 (本教學課程稍後會說明) 中開啟 [服務提供者中繼資料 URL] 以取得識別碼值。 . 
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/recognize-tutorial/tutorial_recognize_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/recognize-tutorial/tutorial_general_400.png)

1. 在 [Recognize 組態] 區段上，按一下 [設定 Recognize] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/recognize-tutorial/tutorial_recognize_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以管理員身分登入您的 Recognize 租用戶。

1. 按一下右上角的 [功能表]。 移至 [Company Admin] (公司管理員)。
   
    ![在應用程式端設定單一登入](./media/recognize-tutorial/tutorial_recognize_000.png)

1. 在左側的導覽窗格上，按一下 [設定] 。
   
    ![在應用程式端設定單一登入](./media/recognize-tutorial/tutorial_recognize_001.png)

1. 在 [SSO Settings] \(SSO 設定) 區段下執行下列步驟。
   
    ![在應用程式端設定單一登入](./media/recognize-tutorial/tutorial_recognize_002.png)
    
    a. 將 **[啟用 SSO]** 選取為 **ON**。

    b. 在 [IDP 實體識別碼] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。
    
    c. 在 [Sso 目標 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。
    
    d. 在 [Slo 目標 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。 
    
    e. 在記事本中開啟您下載的**憑證 (Base64)** 檔案，將其內容複製到剪貼簿，然後貼到 [憑證] 文字方塊中
    
    f. 按一下 [儲存設定]  按鈕。 

1. 在 [SSO Settings] 區段中，複製 [Service Provider Metadata url] 下方的 URL。
   
    ![在應用程式端設定單一登入](./media/recognize-tutorial/tutorial_recognize_003.png)

1. 在空白瀏覽器下方開啟**中繼資料 URL 連結**，以下載中繼資料文件。 然後，從檔案中複製 EntityDescriptor 值 (entityID) 值，然後貼到 Azure 入口網站中 [Recognize 網域及 URL] 區段的 [識別碼] 文字方塊中。
    
    ![在應用程式端設定單一登入](./media/recognize-tutorial/tutorial_recognize_004.png)

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/recognize-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/recognize-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/recognize-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/recognize-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-recognize-test-user"></a>建立 Recognize 測試使用者

若要讓 Azure AD 使用者能夠登入 Recognize，必須將他們佈建到 Recognize。 在 Recognize 的情況下，佈建是手動工作。

此應用程式不支援 SCIM 佈建，但有能夠佈建使用者的替代使用者同步處理作業。 

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以管理員身分登入您的 Recognize 公司網站。

1. 按一下右上角的 [功能表]。 移至 [Company Admin] \(公司管理員)。

1. 在左側的導覽窗格上，按一下 [設定] 。

1. 在 [使用者同步處理] 區段中執行下列步驟。
   
   ![新增使用者](./media/recognize-tutorial/tutorial_recognize_005.png "新增使用者")
   
   a. 對於 [已啟用同步處理] 選取 **ON**。
   
   b. 對於 [Choose sync provider] (選擇同步處理提供者)，選取 [Microsoft / Office 365]。
   
   c. 按一下 [Run User Sync] \(執行使用者同步處理)。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Recognize 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Recognize，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Recognize] 。

    ![設定單一登入](./media/recognize-tutorial/tutorial_recognize_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用存取面板來測試您的 Azure AD 單一登入組態。

當您在 [存取面板] 中按一下 [Recognize] 圖格時，應該會自動登入您的 Recognize 應用程式。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/recognize-tutorial/tutorial_general_01.png
[2]: ./media/recognize-tutorial/tutorial_general_02.png
[3]: ./media/recognize-tutorial/tutorial_general_03.png
[4]: ./media/recognize-tutorial/tutorial_general_04.png

[100]: ./media/recognize-tutorial/tutorial_general_100.png

[200]: ./media/recognize-tutorial/tutorial_general_200.png
[201]: ./media/recognize-tutorial/tutorial_general_201.png
[202]: ./media/recognize-tutorial/tutorial_general_202.png
[203]: ./media/recognize-tutorial/tutorial_general_203.png

