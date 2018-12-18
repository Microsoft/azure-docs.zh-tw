---
title: 教學課程：Azure Active Directory 與 Rightscale 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Rightscale 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 3a8d376d-95fb-4dd7-832a-4fdd4dd7c87c
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/08/2017
ms.author: jeedes
ms.openlocfilehash: 6d251400baed6b15d96eb7a2ef64d4217631613b
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39435976"
---
# <a name="tutorial-azure-active-directory-integration-with-rightscale"></a>教學課程：Azure Active Directory 與 Rightscale 整合

在本教學課程中，您會了解如何整合 Rightscale 與 Azure Active Directory (Azure AD)。

Rightscale 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Rightscale 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Rightscale (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Rightscale 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 RightScale 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Rightscale
1. 設定並測試 Azure AD 單一登入

## <a name="adding-rightscale-from-the-gallery"></a>從資源庫新增 Rightscale
若要設定將 Rightscale 整合到 Azure AD 中，您需要從資源庫將 Rightscale 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Rightscale，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Rightscale**。

    ![建立 Azure AD 測試使用者](./media/rightscale-tutorial/tutorial_rightscale_search.png)

1. 在結果面板中，選取 [Rightscale]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/rightscale-tutorial/tutorial_rightscale_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Rightscale 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Rightscale 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Rightscale 中的相關使用者之間，建立連結關聯性。

在 Rightscale 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Rightscale 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Rightscale 測試使用者](#creating-a-rightscale-test-user)** - 使 RightScale 中對應的 Britta Simon 連結到該該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Rightscale 應用程式中設定單一登入。

**若要設定與 Rightscale 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Rightscale] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_samlbase.png)

1. 如果您想要以 **IDP 起始模式**設定應用程式，則不需要在 [Rightscale 網域及 URL]**和 Url** 區段中執行任何步驟，因為應用程式已預先整合到 Azure。

    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_url.png)

1. 如果您想要以 **SP 起始模式**設定應用程式，請在 [Rightscale 網域及 URL] 區段上執行下列步驟：
    
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_url1.png)

    a. 按一下 [顯示進階 URL 設定]。

    b. 在 [登入 URL] 文字方塊中，輸入 URL：`https://login.rightscale.com/`

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/rightscale-tutorial/tutorial_general_400.png)

1. 在 [Rightscale 組態] 區段上，按一下 [設定 Rightscale] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體識別碼] 和 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_configure.png) 
<CS>
1. 若要取得為應用程式設定的 SSO，您必須以系統管理員身分登入 RightScale 租用戶。

    a. 在頂端的功能表中，按一下 [設定] 索引標籤，然後選取 [單一登入]。
   
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_001.png) 

    b. 按一下 [新增] 按鈕來新增**您的 SAML 身分識別提供者**。
   
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_002.png) 
 
    c. 在 [顯示名稱] 文字方塊中，輸入您的公司名稱。
   
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_003.png)
 
    d. 選取 [Allow RightScale-initiated SSO using a discovery hint] (允許使用 Discovery Hint 的 RightScale 起始 SSO)，然後在下方文字方塊中輸入您的**網域名稱**。
   
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_004.png)

    e. 將您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值，貼到 RightScale 的 [SAML SSO 端點] 文字方塊中。
   
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_006.png)

    f. 將您從 Azure 入口網站複製的 [SAML 實體識別碼] 值，貼到 RightScale 中的 [SAML 實體識別碼]。
   
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_008.png)

    g. 按一下 [瀏覽器]  按鈕來上傳您從 Azure 入口網站下載的憑證。
   
    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_009.png)

    h. 按一下 [檔案] 。
<CE>
> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/rightscale-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/rightscale-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/rightscale-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/rightscale-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-rightscale-test-user"></a>建立 Rightscale 測試使用者

在本節中，您會在 RightScale 中建立名為 Britta Simon 的使用者。 請與 [Rightscale 客戶支援小組](mailto:support@rightscale.com)合作，在 RightScale 平台中新增使用者。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Rightscale 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Rightscale，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Rightscale]。

    ![設定單一登入](./media/rightscale-tutorial/tutorial_rightscale_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。  

當您在「存取面板」中按一下 [RightScale] 磚時，應該會自動登入您的 RightScale 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/rightscale-tutorial/tutorial_general_01.png
[2]: ./media/rightscale-tutorial/tutorial_general_02.png
[3]: ./media/rightscale-tutorial/tutorial_general_03.png
[4]: ./media/rightscale-tutorial/tutorial_general_04.png

[100]: ./media/rightscale-tutorial/tutorial_general_100.png

[200]: ./media/rightscale-tutorial/tutorial_general_200.png
[201]: ./media/rightscale-tutorial/tutorial_general_201.png
[202]: ./media/rightscale-tutorial/tutorial_general_202.png
[203]: ./media/rightscale-tutorial/tutorial_general_203.png

