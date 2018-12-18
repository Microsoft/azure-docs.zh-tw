---
title: 教學課程：Azure Active Directory 與 Bambu by Sprout Social 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Bambu by Sprout Social 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: d2b9ddbc-cab7-40d6-aca1-5b171cab4199
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2017
ms.author: jeedes
ms.openlocfilehash: 317fb1dd1235793d31ab39f5cb2a99766ca33150
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49116154"
---
# <a name="tutorial-azure-active-directory-integration-with-bambu-by-sprout-social"></a>教學課程：Azure Active Directory 與 Bambu by Sprout Social 整合

在本教學課程中，您將了解如何將 Bambu by Sprout Social 與 Azure Active Directory (Azure AD) 整合。

將 Bambu by Sprout Social 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Bambu by Sprout Social 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Bambu by Sprout Social (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Bambu by Sprout Social 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Bambu by Sprout Social 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Bambu by Sprout Social
1. 設定並測試 Azure AD 單一登入

## <a name="adding-bambu-by-sprout-social-from-the-gallery"></a>從資源庫新增 Bambu by Sprout Social
若要設定將 Bambu by Sprout Social 整合到 Azure AD 中，您需要從資源庫將 Bambu by Sprout Social 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Bambu by Sprout Social，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 按一下對話方塊頂端的 [新應用程式] 按鈕來新增新的應用程式。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Bambu by Sprout Social**。

    ![建立 Azure AD 測試使用者](./media/bambubysproutsocial-tutorial/tutorial_bambubysproutsocial_search.png)

1. 在結果窗格中，選取 [Bambu by Sprout Social]，然後按一下 [加入] 按鈕來新增該應用程式。

    ![建立 Azure AD 測試使用者](./media/bambubysproutsocial-tutorial/tutorial_bambubysproutsocial_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Bambu by Sprout Social 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Bambu by Sprout Social 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Bambu by Sprout Social 中的相關使用者之間建立連結關聯性。

建立此連結關聯性的方法，就是將 Azure AD 中 [使用者名稱] 的值，指派為 Bambu by Sprout Social 中 [Username] 的值。

若要設定及測試與 Bambu by Sprout Social 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Bambu by Sprout Social 測試使用者](#creating-a-bambu-by-sprout-social-test-user)** - 在 Bambu by Sprout Social 中建立一個與 Azure AD 中代表 Britta Simon 的項目連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Bambu by Sprout Social 應用程式中設定單一登入。

**若要設定與 Bambu by Sprout Social 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Bambu by Sprout Social] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，選取 [SAML 型登入] 做為 [模式]，以啟用單一登入。
 
    ![設定單一登入](./media/bambubysproutsocial-tutorial/tutorial_bambubysproutsocial_samlbase.png)

1. 在 [Bambu by Sprout Social 網域和 URL] 區段中，使用者不需要執行任何步驟，因為應用程式已經與 Azure 預先整合。 

    ![設定單一登入](./media/bambubysproutsocial-tutorial/tutorial_bambubysproutsocial_url.png)

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/bambubysproutsocial-tutorial/tutorial_bambubysproutsocial_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/bambubysproutsocial-tutorial/tutorial_general_400.png)
    
1. 在 [Bambu by Sprout Social 組態] 區段上，按一下 [設定 Bambu by Sprout Social] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/bambubysproutsocial-tutorial/tutorial_bambubysproutsocial_configure.png) 

1. 若要在 **Bambu by Sprout Social** 這一端設定單一登入，您必須將所下載的「中繼資料 XML」和「SAML 單一登入服務 URL」傳送給 [Bambu by Sprout Social 支援小組](mailto:support@getbambu.com)。 他們將會設定妥當，讓兩端的 SAML SSO 連線都設定正確。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

<!--### Next steps

To ensure users can sign-in to Bambu by Sprout Social after it has been configured to use Azure Active Directory, review the following tasks and topics:

- User accounts must be pre-provisioned into Bambu by Sprout Social prior to sign-in. To set this up, see Provisioning.
 
- Users must be assigned access to Bambu by Sprout Social in Azure AD to sign-in. To assign users, see Users.
 
- To configure access polices for Bambu by Sprout Social users, see Access Policies.
 
- For additional information on deploying single sign-on to users, see [this article](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis#deploying-azure-ad-integrated-applications-to-users).-->


### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/bambubysproutsocial-tutorial/create_aaduser_01.png) 

1. 移至 [使用者和群組]，然後按一下 [所有使用者] 以顯示使用者清單。
    
    ![建立 Azure AD 測試使用者](./media/bambubysproutsocial-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。
 
    ![建立 Azure AD 測試使用者](./media/bambubysproutsocial-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/bambubysproutsocial-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **Britta Simon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 Britta Simon 的「電子郵件地址」。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-bambu-by-sprout-social-test-user"></a>建立 Bambu by Sprout Social 測試使用者

應用程式僅在使用者佈建時支援，驗證之後，會在應用程式中自動建立使用者。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Bambu by Sprout Social 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Bambu by Sprout Social，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Bambu by Sprout Social]。

    ![設定單一登入](./media/bambubysproutsocial-tutorial/tutorial_bambubysproutsocial_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Bambu by Sprout Social] 圖格時，應該會自動登入您的 Bambu by Sprout Social 應用程式。 如需 [存取面板] 的詳細資訊，請參閱 [存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/bambubysproutsocial-tutorial/tutorial_general_01.png
[2]: ./media/bambubysproutsocial-tutorial/tutorial_general_02.png
[3]: ./media/bambubysproutsocial-tutorial/tutorial_general_03.png
[4]: ./media/bambubysproutsocial-tutorial/tutorial_general_04.png

[100]: ./media/bambubysproutsocial-tutorial/tutorial_general_100.png

[200]: ./media/bambubysproutsocial-tutorial/tutorial_general_200.png
[201]: ./media/bambubysproutsocial-tutorial/tutorial_general_201.png
[202]: ./media/bambubysproutsocial-tutorial/tutorial_general_202.png
[203]: ./media/bambubysproutsocial-tutorial/tutorial_general_203.png
