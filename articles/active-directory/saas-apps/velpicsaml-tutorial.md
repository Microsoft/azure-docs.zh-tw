---
title: 教學課程：Azure Active Directory 與 Velpic SAML 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Velpic SAML 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 28acce3e-22a0-4a37-8b66-6e518d777350
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/04/2017
ms.author: jeedes
ms.openlocfilehash: 2a8fe814801d00b3f27e52484d742cbe6c7b0770
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49114001"
---
# <a name="tutorial-azure-active-directory-integration-with-velpic-saml"></a>教學課程：Azure Active Directory 與 Velpic SAML 整合

在本教學課程中，您將了解如何整合 Velpic SAML 與 Azure Active Directory (Azure AD)。

Velpic SAML 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Velpic SAML 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Velpic SAML (單一登入)
- 您可以在 Azure 管理入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Velpic SAML 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Velpic SAML 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Velpic SAML
1. 設定並測試 Azure AD 單一登入

## <a name="adding-velpic-saml-from-the-gallery"></a>從資源庫新增 Velpic SAML
若要設定 Velpic SAML 與 Azure AD 整合，您需要從資源庫將 Velpic SAML 新增到受控 SaaS app 清單。

**若要從資源庫新增 Velpic SAML，請執行下列步驟：**

1. 在 **[Azure 管理入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 按一下對話方塊頂端的 [新增] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Velpic SAML**。

    ![建立 Azure AD 測試使用者](./media/velpicsaml-tutorial/tutorial_velpicsaml_search.png)

1. 在結果窗格中，選取 [Velpic SAML]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/velpicsaml-tutorial/tutorial_velpicsaml_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Velpic SAML 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Velpic SAML 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Velpic SAML 中相關使用者之間的連結關聯性。

建立此連結關聯性的方法是將 Azure AD 中**使用者名稱**的值指定為 Velpic SAML 中 **Username** 的值。

若要設定及測試與 Velpic SAML 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Velpic SAML 測試使用者](#creating-a-velpic-saml-test-user)** - 在 Velpic SAML 中建立一個與 Azure AD 中代表 Britta Simon 的項目連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 管理入口網站中啟用 Azure AD 單一登入，並在您的 Velpic SAML 應用程式中設定單一登入。

**若要設定與 Velpic SAML 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 管理入口網站的 [Velpic SAML] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，選取 [SAML 型登入] 做為 [模式]，以啟用單一登入。
 
    ![設定單一登入](./media/velpicsaml-tutorial/tutorial_velpicsaml_samlbase.png)

1. 在 [Velpic SAML 網域和 URL] 區段中輸入詳細資料

    ![設定單一登入](./media/velpicsaml-tutorial/tutorial_velpicsaml_url.png)

    a. 在 [登入 URL] 文字方塊中，輸入下列值：`https://<sub-domain>.velpicsaml.net`

    b. 在 [識別碼] 文字方塊中，貼上 **「單一登入 URL」** 值`https://auth.velpic.com/saml/v2/<entity-id>/login`
    
    > [!NOTE]
    > 請注意，登入 URL 會由 Velpic SAML 小組提供，而識別碼值則會在您於 Velpic SAML 端設定 SSO 外掛程式時可供取得。 您需要從 Velpic SAML 應用程式頁面將該值複製並貼到此處。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/velpicsaml-tutorial/tutorial_velpicsaml_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/velpicsaml-tutorial/tutorial_general_400.png)

1. 在 [Velpic SAML 組態] 區段上，按一下 [設定 Velpic SAML] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段複製 SAML 實體識別碼。

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Velpic SAML 公司網站。

1. 按一下 [管理] 索引標籤，然後移至 [整合] 區段，您必須在此處按一下 [外掛程式] 按鈕以建立用於登入的新外掛程式。

    ![外掛程式](./media/velpicsaml-tutorial/velpic_1.png)

1. 按一下 [新增外掛程式] 按鈕。
    
    ![外掛程式](./media/velpicsaml-tutorial/velpic_2.png)

1. 按一下 [新增外掛程式] 頁面中的 [SAML] 圖格。
    
    ![外掛程式](./media/velpicsaml-tutorial/velpic_3.png)

1. 為新的 SAML 外掛程式輸入名稱，然後按一下 [新增] 按鈕。

    ![外掛程式](./media/velpicsaml-tutorial/velpic_4.png)

1. 輸入詳細資料，如下所示︰

    ![外掛程式](./media/velpicsaml-tutorial/velpic_5.png)

    a. 在 [名稱] 文字方塊中，輸入 SAML 外掛程式的名稱。

    b. 在 [簽發者 URL] 文字方塊中，貼上您從 Azure 入口網站的 [設定登入] 視窗中所複製的 [SAML 實體識別碼]。

    c. 在 [提供者中繼資料設定] 中，上傳您從 Azure 入口網站下載的中繼資料 XML 檔案。

    d. 您也可以選擇啟用 [自動建立新使用者] 核取方塊，來啟用 SAML 即時佈建。 如果使用者不存在於 Velpic，而且這個旗標未啟用，從 Azure 登入時就會失敗。 如果該旗標已啟用，則系統會在使用者登入時自動將其佈建到 Velpic。 

    e. 從文字方塊複製**單一登入 URL**，並將它貼到 Azure 入口網站。
    
    f. 按一下 [檔案] 。

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節目標是在 Azure 管理入口網站中建立名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 管理入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/velpicsaml-tutorial/create_aaduser_01.png) 

1. 移至 [使用者和群組]，然後按一下 [所有使用者] 以顯示使用者清單。
    
    ![建立 Azure AD 測試使用者](./media/velpicsaml-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。
 
    ![建立 Azure AD 測試使用者](./media/velpicsaml-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/velpicsaml-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-velpic-saml-test-user"></a>建立 Velpic SAML 測試使用者

您通常不需要進行此步驟，因為應用程式支援即時使用者佈建。 如果自動使用者佈建未啟用，那麼您可以如下所述手動建立使用者。

以系統管理員身分登入您的 Velpic SAML 公司網站，然後執行下列步驟：
    
1. 按一下 [管理] 索引標籤並移至 [使用者] 區段，然後按一下 [新增] 按鈕來新增使用者。

    ![新增使用者](./media/velpicsaml-tutorial/velpic_7.png)

1. 在 [建立新的使用者] 對話方塊中，執行下列步驟。

    ![user](./media/velpicsaml-tutorial/velpic_8.png)
    
    a. 在 [名字] 文字方塊中，輸入 Britta Simon 的名字。

    b. 在 [姓氏] 文字方塊中，輸入 Britta Simon 的姓氏。

    c. 在 [使用者名稱] 文字方塊中，輸入 Britta Simon 的使用者名稱。

    d. 在 [電子郵件] 文字方塊中，輸入 Britta Simon 帳戶的電子郵件地址。

    e. 其餘資訊是選擇性的，您可以視需要來填入。
    
    f. 按一下 [儲存] 。  

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會授權 Britta Simon 存取 Velpic SAML，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 Velpic SAML，請執行以下步驟：**

1. 在 Azure 管理入口網站中，開啟應用程式檢視，然後瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Velpic SAML]。

    ![設定單一登入](./media/velpicsaml-tutorial/tutorial_velpicsaml_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

1. 當您按一下存取面板中的 [Velpic SAML] 圖格時，您應該會看到 Velpic SAML 應用程式的登入頁面。 您應該會在登入頁面上看到 [以 Azure AD 登入] 按鈕。

    ![外掛程式](./media/velpicsaml-tutorial/velpic_6.png)

1. 按一下 [以 Azure AD 登入] 按鈕以使用 Azure AD 帳戶登入 Velpic。


## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/velpicsaml-tutorial/tutorial_general_01.png
[2]: ./media/velpicsaml-tutorial/tutorial_general_02.png
[3]: ./media/velpicsaml-tutorial/tutorial_general_03.png
[4]: ./media/velpicsaml-tutorial/tutorial_general_04.png

[100]: ./media/velpicsaml-tutorial/tutorial_general_100.png

[200]: ./media/velpicsaml-tutorial/tutorial_general_200.png
[201]: ./media/velpicsaml-tutorial/tutorial_general_201.png
[202]: ./media/velpicsaml-tutorial/tutorial_general_202.png
[203]: ./media/velpicsaml-tutorial/tutorial_general_203.png

