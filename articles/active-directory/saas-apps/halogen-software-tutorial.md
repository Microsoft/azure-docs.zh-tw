---
title: 教學課程：Azure Active Directory 與 Halogen Software 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Halogen Software 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 2ca2298d-9a0c-4f14-925c-fa23f2659d28
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/23/2017
ms.author: jeedes
ms.openlocfilehash: a7be918118d86da7e1134f5ce46e5f163ba62601
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39432842"
---
# <a name="tutorial-azure-active-directory-integration-with-halogen-software"></a>教學課程：Azure Active Directory 與 Halogen Software 整合

在此教學課程中，您將了解如何整合 Halogen Software 與 Azure Active Directory (Azure AD)。

Halogen Software 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Halogen Software 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Halogen Software (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Halogen Software 整合，您需要以下項目：

- Azure AD 訂用帳戶
- 已啟用 Halogen Software 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫加入 Halogen Software
1. 設定並測試 Azure AD 單一登入

## <a name="adding-halogen-software-from-the-gallery"></a>從資源庫加入 Halogen Software

若要設定 Halogen Software 與 Azure AD 整合，您需要從資源庫將 Halogen Software 加入 受控 SaaS app 的清單。

**若要從資源庫新增 Halogen Software，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Halogen Software**。

    ![建立 Azure AD 測試使用者](./media/halogen-software-tutorial/tutorial_halogensoftware_search.png)

1. 在結果窗格中，選取 [Halogen Software]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/halogen-software-tutorial/tutorial_halogensoftware_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Halogen Software 搭配運作的 Azure AD 單一登入。

單一登入若要運作，Azure AD 必須知道 Halogen Software 與 Azure AD 中互相對應的使用者。 換句話說，Azure AD 使用者和 Halogen Software 中的相關使用者之間必須建立連結關聯性。

在 Halogen Software 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要使用 Halogen Software 設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Halogen Software 測試使用者](#creating-a-halogen-software-test-user)** - 在 Halogen Software 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Halogen Software 應用程式中設定單一登入。

**若要使用 Halogen Software 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Halogen Software] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/halogen-software-tutorial/tutorial_halogensoftware_samlbase.png)

1. 在 [Halogen Software 網域及 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/halogen-software-tutorial/tutorial_halogensoftware_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://global.hgncloud.com/<companyname>`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL：`https://global.halogensoftware.com/<companyname>`、`https://global.hgncloud.com/<companyname>`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Halogen Software 用戶端支援小組](https://support.halogensoftware.com/)以取得這些值。 
 


1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/halogen-software-tutorial/tutorial_halogensoftware_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/halogen-software-tutorial/tutorial_general_400.png)

1. 在不同的瀏覽器視窗中，以系統管理員身分登入您的 **Halogen Software** 。

1. 按一下 [選項]  索引標籤。 
   
    ![何謂 Azure AD Connect][12]

1. 在左瀏覽窗格中，按一下 [SAML 組態] 。 
   
    ![何謂 Azure AD Connect][13]

1. 在 [SAML 組態]  頁面上，執行下列步驟： 

    ![何謂 Azure AD Connect][14]

     a. 對於 [唯一識別碼]，選取 [NameID]。

     b. 對於 [唯一識別碼對應到]，選取 [Username]。
  
     c. 若要更新您已下載的中繼資料檔，請按一下 [瀏覽] 來選取檔案，然後按一下 [上傳檔案]。
 
     d. 若要測試組態，請按一下 [執行測試]。 
    
    >[!NOTE]
    >您需要等候「SAML 測試已完成。請關閉此視窗。」訊息顯示。 然後關閉已開啟的瀏覽器視窗。 完成測試之後才會啟用 [啟用 SAML] 核取方塊。 
     
     e. 選取 [啟用 SAML] 。
    
     f. 按一下 [儲存變更] 。 

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)


### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/halogen-software-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/halogen-software-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/halogen-software-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/halogen-software-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon** 作為名稱。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-halogen-software-test-user"></a>建立 Halogen Software 測試使用者

本節目標是在 Halogen Software 中建立名為 Britta Simon 的使用者。

**若要建立名為 Britta Simon 的使用者，請執行以下步驟：**

1. 以系統管理員身分登入您的 **Halogen Software** 。

1. 按一下 [使用者中心] 索引標籤，然後按一下 [建立使用者]。
   
    ![何謂 Azure AD Connect][300]  

1. 在 [新增使用者]  對話頁面上，執行下列步驟：
   
    ![何謂 Azure AD Connect][301]

    a. 在 [名字] 文字方塊中，輸入使用者的名字，例如 **Britta**。
    
    b. 在 [姓氏] 文字方塊中，輸入使用者的姓氏，例如 **Simon**。 

    c. 在 [使用者名稱] 文字方塊中輸入 **Britta Simon**，亦即 Azure 入口網站中的使用者名稱。

    d. 在 [密碼] 文字方塊中輸入 Britta 的密碼。
    
    e. 按一下 [檔案] 。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Halogen Software 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要指派 Britta Simon 到 Halogen Software，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Halogen Software] 。

    ![設定單一登入](./media/halogen-software-tutorial/tutorial_halogensoftware_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。

當您在 [存取面板] 中按一下 [Halogen Software] 磚時，您應該會自動登入您的 Halogen Software 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/halogen-software-tutorial/tutorial_general_01.png
[2]: ./media/halogen-software-tutorial/tutorial_general_02.png
[3]: ./media/halogen-software-tutorial/tutorial_general_03.png
[4]: ./media/halogen-software-tutorial/tutorial_general_04.png

[12]: ./media/halogen-software-tutorial/tutorial_halogen_12.png

[13]: ./media/halogen-software-tutorial/tutorial_halogen_13.png

[14]: ./media/halogen-software-tutorial/tutorial_halogen_14.png

[100]: ./media/halogen-software-tutorial/tutorial_general_100.png

[200]: ./media/halogen-software-tutorial/tutorial_general_200.png
[201]: ./media/halogen-software-tutorial/tutorial_general_201.png
[202]: ./media/halogen-software-tutorial/tutorial_general_202.png
[203]: ./media/halogen-software-tutorial/tutorial_general_203.png

[300]: ./media/halogen-software-tutorial/tutorial_halogen_300.png

[301]: ./media/halogen-software-tutorial/tutorial_halogen_301.png
