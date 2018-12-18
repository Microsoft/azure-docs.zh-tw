---
title: 教學課程：Azure Active Directory 與 CloudPassage 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 CloudPassage 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: bfe1f14e-74e4-4680-ac9e-f7355e1c94cc
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/22/2017
ms.author: jeedes
ms.openlocfilehash: c44b9345da32f907efacfe2b7bb1cf09de0a6345
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39427528"
---
# <a name="tutorial-azure-active-directory-integration-with-cloudpassage"></a>教學課程：Azure Active Directory 與 CloudPassage 整合

在本教學課程中，您會了解如何整合 CloudPassage 與 Azure Active Directory (Azure AD)。

CloudPassage 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 CloudPassage 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 CloudPassage (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 CloudPassage 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 CloudPassage 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 CloudPassage
1. 設定並測試 Azure AD 單一登入

## <a name="adding-cloudpassage-from-the-gallery"></a>從資源庫新增 CloudPassage
若要設定 CloudPassage 與 Azure AD 整合，您需要從資源庫將 CloudPassage 新增到受控 SaaS App 清單。

**若要從資源庫新增 CloudPassage，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **CloudPassage**。

    ![建立 Azure AD 測試使用者](./media/cloudpassage-tutorial/tutorial_cloudpassage_search.png)

1. 在結果窗格中，選取 [CloudPassage]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/cloudpassage-tutorial/tutorial_cloudpassage_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 CloudPassage 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 CloudPassage 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 CloudPassage 中的相關使用者之間建立連結關聯性。

在 CloudPassage 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要使用 CloudPassage 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 CloudPassage 測試使用者](#creating-a-cloudpassage-test-user)** - 在 CloudPassage 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 CloudPassage 應用程式中設定單一登入。

**若要使用 CloudPassage 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [CloudPassage] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_cloudpassage_samlbase.png)

1. 在 [CloudPassage 網域及 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_cloudpassage_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://portal.cloudpassage.com/saml/init/accountid`

    b. 在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：`https://portal.cloudpassage.com/saml/consume/accountid`。 您也可以在 CloudPassage 入口網站的 [單一登入設定] 區段中按一下 [SSO 安裝文件]，為這個屬性設定值。

    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_cloudpassage_05.png)
     
    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的回覆 URL 與登入 URL 更新這些值。 請連絡 [CloudPassage 用戶端支援小組](https://www.cloudpassage.com/company/contact/)以取得這些值。 

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_cloudpassage_certificate.png) 

1. 您的 CloudPassage 應用程式需要特定格式的 SAML 判斷提示，其會要求您將自訂屬性對應新增到 SAML Token 屬性設定。 以下螢幕擷取畫面顯示上述的範例。
   
   ![設定單一登入](./media/cloudpassage-tutorial/tutorial_cloudpassage_25.png) 

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：

    | 屬性名稱 | 屬性值 |
    | --- | --- |
    | firstname |user.givenname |
    | lastname |user.surname |
    | 電子郵件 |user.mail |
    
    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_attribute_04.png)
    
    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定] 。

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_general_400.png)
    
1. 在 [CloudPassage 組態] 區段上，按一下 [設定 CloudPassage] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_cloudpassage_configure.png) 

1. 在不同的瀏覽器視窗中，以系統管理員身分登入您的 CloudPassage 公司網站。

1. 在頂端功能表中，按一下 [設定]，然後按一下 [網站管理]。 
   
    ![設定單一登入][12]

1. 按一下 [驗證設定]  索引標籤。 
   
    ![設定單一登入][13]

1. 在 [單一登入設定]  區段中，執行下列步驟： 
   
    ![設定單一登入][14]

    a. 選取 [啟用單一登入 (SSO) (SSO 安裝程式文件)/] 核取方塊。
    
    b. 將 [SAML 實體識別碼] 貼入 [SAML 簽發者 URL] 文字方塊中。
  
    c. 將 [SAML 單一登入服務 URL] 貼入 [登入入口網站 URL] 文字方塊中。
  
    d. 將 [登出 URL] 貼入 [登出登陸頁面] 文字方塊中。
  
    e. 在記事本中開啟您下載的憑證，將已下載憑證的內容複製到剪貼簿上，然後貼到 [x 509 憑證] 文字方塊中。
  
    f. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/cloudpassage-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/cloudpassage-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/cloudpassage-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/cloudpassage-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-cloudpassage-test-user"></a>建立 CloudPassage 測試使用者

本節目標是在 CloudPassage 中建立名為 Britta Simon 的使用者。

**若要在 CloudPassage 中建立名為 Britta Simon 的使用者，請執行以下步驟：**

1. 以系統管理員身分登入您的 **CloudPassage** 公司網站。 

1. 在頂端工具列中，按一下 [設定]，然後按一下 [網站管理]。 
   
   ![建立 CloudPassage 測試使用者][22] 

1. 按一下 [使用者] 索引標籤，然後按一下 [新增使用者]。 
   
   ![建立 CloudPassage 測試使用者][23]

1. 在 [新增使用者]  區段中，執行下列步驟： 
   
   ![建立 CloudPassage 測試使用者][24]
    
    a. 在 [名字] 文字方塊中，輸入 Britta。 
  
    b. 在 [姓氏] 文字方塊中，輸入 Simon。
  
    c. 在 [使用者名稱] 文字方塊、[電子郵件] 文字方塊和 [重新輸入電子郵件] 文字方塊中，輸入 Britta 在 Azure AD 中的使用者名稱。
  
    d. 對於 [存取類型]，選取 [啟用 Halo 入口網站存取]。
  
    e. 按一下 [新增] 。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 CloudPassage 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 CloudPassage，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [CloudPassage] 。

    ![設定單一登入](./media/cloudpassage-tutorial/tutorial_cloudpassage_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。

當您在存取面板中按一下 [CloudPassage] 磚時，應該會自動登入您的 CloudPassage 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/cloudpassage-tutorial/tutorial_general_01.png
[2]: ./media/cloudpassage-tutorial/tutorial_general_02.png
[3]: ./media/cloudpassage-tutorial/tutorial_general_03.png
[4]: ./media/cloudpassage-tutorial/tutorial_general_04.png
[12]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_07.png
[13]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_08.png
[14]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_09.png
[15]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_10.png
[22]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_15.png
[23]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_16.png
[24]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_17.png

[100]: ./media/cloudpassage-tutorial/tutorial_general_100.png

[200]: ./media/cloudpassage-tutorial/tutorial_general_200.png
[201]: ./media/cloudpassage-tutorial/tutorial_general_201.png
[202]: ./media/cloudpassage-tutorial/tutorial_general_202.png
[203]: ./media/cloudpassage-tutorial/tutorial_general_203.png

