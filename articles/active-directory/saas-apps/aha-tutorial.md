---
title: 教學課程：Azure Active Directory 與 Aha! | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Aha! 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: ad955d3d-896a-41bb-800d-68e8cb5ff48d
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/10/2017
ms.author: jeedes
ms.openlocfilehash: 30f0f316727cfcf20daa58c35d0ba11c25311898
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39044113"
---
# <a name="tutorial-azure-active-directory-integration-with-aha"></a>教學課程：Azure Active Directory 與 Aha!

在本教學課程中，您會了解如何將 Aha! 與 Azure Active Directory (Azure AD) 整合。

將 Aha! 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Aha! 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Aha! (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Aha! 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Aha! 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 新增 Aha! (從資源庫)
2. 設定並測試 Azure AD 單一登入

## <a name="adding-aha-from-the-gallery"></a>新增 Aha! (從資源庫)
若要設定將 Aha! 整合到 Azure AD 中，您需要從資源庫將 Aha! 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Aha!，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

4. 在搜尋方塊中，輸入 **Aha!**。

    ![建立 Azure AD 測試使用者](./media/aha-tutorial/tutorial_aha_search.png)

5. 在結果面板中，選取 [Aha!]，然後按一下 [新增] 按鈕以新增該應用程式。

    ![建立 Azure AD 測試使用者](./media/aha-tutorial/tutorial_aha_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會設定及測試與 Aha! 搭配運作的 Azure AD 單一登入 (以名為 "Britta Simon" 的測試使用者為基礎)。

若要讓單一登入能夠運作，Azure AD 必須知道 Aha! 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Aha! 中的相關使用者之間 建立連結關聯性。

在 Aha! 中，將 [Username] 的值指派為 Azure AD 中 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 Aha! 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Aha! 測試使用者](#creating-an-aha-test-user)** - 在 Aha! 中建立一個與 Azure AD 中代表 Britta Simon 之項目 連結的 Britta Simon 對應項目。
4. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Aha! 應用程式中設定單一登入。

**若要設定與 Aha! 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Aha!] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/aha-tutorial/tutorial_aha_samlbase.png)

3. 在 [Aha!網域及 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/aha-tutorial/tutorial_aha_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<companyname>.aha.io/session/new`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<companyname>.aha.io`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Aha!用戶端支援小組](https://www.aha.io/company/contact)以取得這些值。 
 
4. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/aha-tutorial/tutorial_aha_certificate.png) 

5. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/aha-tutorial/tutorial_general_400.png)

6. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Aha! 公司網站。

7. 在頂端的功能表中，按一下 [設定] 。

    ![設定](./media/aha-tutorial/IC798950.png "設定")

8. 按一下 [帳戶] 。
   
    ![設定檔](./media/aha-tutorial/IC798951.png "設定檔")

9. 按一下 [安全性和單一登入] 。
   
    ![安全性和單一登入](./media/aha-tutorial/IC798952.png "安全性和單一登入")

10. 在 [單一登入] 區段中，針對 [識別提供者] 選取 [SAML2.0]。
   
    ![安全性和單一登入](./media/aha-tutorial/IC798953.png "安全性和單一登入")

11. 在 [單一登入]  組態頁面上，執行下列步驟：
    
    ![單一登入](./media/aha-tutorial/IC798954.png "單一登入")
    
       a. 在 [名稱]  文字方塊中，輸入您的組態名稱。

       b. 針對 [設定使用]，選取 [中繼資料檔]。
   
       c. 若要上傳您下載的中繼資料檔，請按一下 [瀏覽] 。
   
       d. 按一下 [更新] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/aha-tutorial/create_aaduser_01.png) 

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/aha-tutorial/create_aaduser_02.png) 

3. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/aha-tutorial/create_aaduser_03.png) 

4. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/aha-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-an-aha-test-user"></a>建立 Aha! 測試使用者

若要讓 Azure AD 使用者能夠登入 Aha!，必須將他們佈建到 Aha!。  

Aha! 的佈建是自動化工作。 沒有您適用的動作項目。

第一次嘗試單一登入時，會視需要自動建立使用者。

>[!NOTE]
>您可以使用任何其他的 Aha! 使用者帳戶建立工具或 Aha! 提供的 API 來佈建 AAD 使用者帳戶。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Aha! 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Aha!，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Aha!]。

    ![設定單一登入](./media/aha-tutorial/tutorial_aha_app.png) 

3. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

如果要測試您的單一登入設定，請開啟存取面板。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/aha-tutorial/tutorial_general_01.png
[2]: ./media/aha-tutorial/tutorial_general_02.png
[3]: ./media/aha-tutorial/tutorial_general_03.png
[4]: ./media/aha-tutorial/tutorial_general_04.png

[100]: ./media/aha-tutorial/tutorial_general_100.png

[200]: ./media/aha-tutorial/tutorial_general_200.png
[201]: ./media/aha-tutorial/tutorial_general_201.png
[202]: ./media/aha-tutorial/tutorial_general_202.png
[203]: ./media/aha-tutorial/tutorial_general_203.png

