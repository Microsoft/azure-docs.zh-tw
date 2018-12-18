---
title: 教學課程：Azure Active Directory 與 Wdesk 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Wdesk 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 06900a91-a326-4663-8ba6-69ae741a536e
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/22/2017
ms.author: jeedes
ms.openlocfilehash: 9ff4d38240bf44cdb3112730b6f6962feed09a6a
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39444431"
---
# <a name="tutorial-azure-active-directory-integration-with-wdesk"></a>教學課程：Azure Active Directory 與 Wdesk 整合

在本教學課程中，您會了解如何整合 Wdesk 與 Azure Active Directory (Azure AD)。

整合 Wdesk 與 Azure AD 有下列優點：

- 您可以在 Azure AD 中控制可存取 Wdesk 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Wdesk (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要知道與 Azure AD 整合的 SaaS 應用程式詳細資訊，請參閱： [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Wdesk 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Wdesk 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Wdesk
1. 設定並測試 Azure AD 單一登入

## <a name="adding-wdesk-from-the-gallery"></a>從資源庫新增 Wdesk
若要設定將 Wdesk 整合到 Azure AD 中，您需要從資源庫將 Wdesk 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Wdesk，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，鍵入 **Wdesk**。

    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/tutorial_wdesk_search.png)

1. 在結果窗格中，選取 [Wdesk]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/tutorial_wdesk_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Wdesk 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Wdesk 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Wdesk 中的相關使用者之間建立連結關聯性。

在 Wdesk 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要使用 Wdesk 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Wdesk 測試使用者](#creating-a-wdesk-test-user)** - 使 Wdesk 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Wdesk 應用程式中設定單一登入。

**若要使用 Wdesk 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Wdesk] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Wdesk 網域和 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_url.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<subdomain>.wdesk.com/auth/saml/sp/metadata/<instancename>`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<subdomain>.wdesk.com/auth/saml/sp/consumer/<instancename>`

1. 按一下 [顯示進階 URL 設定]。 如果您想要在 **SP** 起始模式中設定應用程式，請執行下列步驟：

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_url1.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<subdomain>.wdesk.com/auth/login/saml/<instancename>`
     
    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 當您設定 SSO 時，您可以從 WDesk 入口網站取得這些值。 
  
1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/wdesk-tutorial/tutorial_general_400.png)
    
1. 在不同的網頁瀏覽器視窗中，以安全性系統管理員身分登入 Wdesk。

1. 按一下左下方的 [管理員]，選擇 [帳戶管理員]：
 
     ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig1.png)

1. 在 Wdesk 管理員中，依序巡覽至 [安全性]、[SAML] > [SAML 設定]：

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig2.png)

1. 勾選 [一般設定] 下的 [Enable SAML Single Sign On] \(啟用 SAML 單一登入)：

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig3.png)

1. 在 [Service Provider Details] \(提供者詳細資料) 區段中，執行下列步驟：

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig4.png)

      a. 將**登入 URL** 複製並貼入 Azure 入口網站的 [登入 URL] 文字方塊。
   
      b. 將**中繼資料 URL** 複製並貼入 Azure 入口網站的 [識別碼] 文字方塊。
       
      c. 將**取用者 URL** 複製並貼入 Azure 入口網站的 [回覆 URL] 文字方塊。
   
      d. 按一下 Azure 入口網站的 [儲存]，以儲存變更。      

1. 按一下 [Configure IdP Settings] \(設定 IdP 設定) 開啟 [Edit IdP Settings] \(編輯 IdP 設定) 對話方塊。 按一下 [選擇檔案] 找出從 Azure 入口網站儲存的 **Metadata.xml** 檔案，然後將它上傳。
    
    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig5.png)
  
1. 按一下 [儲存變更] 。

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_ssoconfigsavebutton.png)

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-wdesk-test-user"></a>建立 Wdesk 測試使用者

為使 Azure AD 使用者登入 Wdesk，必須將他們佈建到 Wdesk。 在 Wdesk 中，需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以安全性系統管理員身分登入 Wdesk。
1. 巡覽至 [管理員] > [帳戶管理員]。

     ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig1.png)

1. 按一下 [人員] 下的 [成員]。

1. 現在按一下 [新增成員] 開啟 [新增成員] 對話方塊。 
   
    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/createuser1.png)  

1. 在 [使用者] 文字方塊中，輸入使用者的使用者名稱，例如 **brittasimon@contoso.com**，然後按一下 [繼續] 按鈕。

    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/createuser3.png)

1.  如下所示輸入詳細資料：
  
    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/createuser4.png)
 
    a. 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件，例如 **brittasimon@contoso.com**。

    b. 在 [名字] 文字方塊中，輸入使用者的名字，例如 **Britta**。

    c. 在 [姓氏] 文字方塊中，輸入使用者的姓氏，例如 **Simon**。

1. 按一下 [儲存成員] 按鈕。  

    ![建立 Azure AD 測試使用者](./media/wdesk-tutorial/createuser5.png)

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Wdesk 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Wdesk，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Wdesk]。

    ![設定單一登入](./media/wdesk-tutorial/tutorial_wdesk_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Wdesk] 圖格時，應該會自動登入您的 Wdesk 應用程式。
如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。


## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/wdesk-tutorial/tutorial_general_01.png
[2]: ./media/wdesk-tutorial/tutorial_general_02.png
[3]: ./media/wdesk-tutorial/tutorial_general_03.png
[4]: ./media/wdesk-tutorial/tutorial_general_04.png

[100]: ./media/wdesk-tutorial/tutorial_general_100.png

[200]: ./media/wdesk-tutorial/tutorial_general_200.png
[201]: ./media/wdesk-tutorial/tutorial_general_201.png
[202]: ./media/wdesk-tutorial/tutorial_general_202.png
[203]: ./media/wdesk-tutorial/tutorial_general_203.png

