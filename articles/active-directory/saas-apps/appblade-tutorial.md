---
title: 教學課程：Azure Active Directory 與 AppBlade 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 AppBlade 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 3360d7aa-6518-4f99-88bd-b7f7258183e8
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/09/2017
ms.author: jeedes
ms.openlocfilehash: 66c893a89138d7daf7d8118d8e2b1d8389d40ea4
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36229293"
---
# <a name="tutorial-azure-active-directory-integration-with-appblade"></a>教學課程：Azure Active Directory 與 AppBlade 整合

在本教學課程中，您會了解如何將 AppBlade 與 Azure Active Directory (Azure AD) 整合。

將 AppBlade 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 AppBlade 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 AppBlade (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>先決條件

若要設定 Azure AD 與 AppBlade 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 AppBlade 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 AppBlade
2. 設定並測試 Azure AD 單一登入

## <a name="adding-appblade-from-the-gallery"></a>從資源庫新增 AppBlade
若要設定將 AppBlade 整合到 Azure AD 中，您需要從資源庫將 AppBlade 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 AppBlade，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

4. 在搜尋方塊中，輸入 **AppBlade**。

    ![建立 Azure AD 測試使用者](./media/appblade-tutorial/tutorial_appblade_search.png)

5. 在結果面板中，選取 [AppBlade]，然後按一下 [新增] 按鈕以新增該應用程式。

    ![建立 Azure AD 測試使用者](./media/appblade-tutorial/tutorial_appblade_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 AppBlade 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 AppBlade 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 AppBlade 中的相關使用者之間建立連結關聯性。

在 AppBlade 中，將 [Username] 的值指派為 Azure AD 中 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 AppBlade 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 AppBlade 測試使用者](#creating-an-appblade-test-user)** - 在 AppBlade 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的 Britta Simon 對應項目。
4. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 AppBlade 應用程式中設定單一登入。

**若要設定與 AppBlade 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [AppBlade] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/appblade-tutorial/tutorial_appblade_samlbase.png)

3. 在 [AppBlade 網域及 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/appblade-tutorial/tutorial_appblade_url.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰`https://<companyname>.appblade.com/saml/<tenantid>`

    > [!NOTE] 
    > 這不是真實的值。 請使用實際的「登入 URL」來更新此值。 請連絡 [AppBlade 用戶端支援小組](mailto:support@appblade.com)以取得此值。 
 
4. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/appblade-tutorial/tutorial_appblade_certificate.png) 

5. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/appblade-tutorial/tutorial_general_400.png)

6. 若要在 **AppBlade** 端設定單一登入，您必須將已下載的「中繼資料 XML」傳送給 [AppBlade 支援小組](mailto:support@appblade.com)。 此外，請要求他們將 [SSO 簽發者 URL] 設定為 `https://appblade.com/saml`。 單一登入必須要有這項設定才能運作。


> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
 
### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/appblade-tutorial/create_aaduser_01.png) 

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/appblade-tutorial/create_aaduser_02.png) 

3. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/appblade-tutorial/create_aaduser_03.png) 

4. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/appblade-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-an-appblade-test-user"></a>建立 AppBlade 測試使用者

本節的目標是要在 AppBlade 中建立一個名為 Britta Simon 的使用者。 AppBlade 支援預設啟用的 Just-In-Time 佈建。 **請確定您已搭配 AppBlade 來設定網域名稱，以便進行使用者佈建。之後，將只有 Just-In-Time 使用者佈建能夠運作。**

如果使用者的電子郵件地址結尾是 AppBlade 為您帳戶設定的網域，則使用者會自動以您指定的權限層級加入成為該帳戶的成員，指定的權限層級為「基本」(只能安裝應用程式的基本使用者)、「小組成員」(可以上傳新應用程式版本及管理專案的使用者) 或「系統管理員」(完整的帳戶系統管理員權限) 其中之一。 通常您會選擇「基本」，然後透過「系統管理員登入」手動提升使用者權限 (AppBlade 需要預先設定一個電子郵件型系統管理員登入，或是在登入後代表客戶提升使用者權限)。

在這一節沒有您需要進行的動作項目。 嘗試存取 AppBlade 時，如果使用者還不存在，就會建立新使用者。 

> [!NOTE]
> 如果您需要手動建立使用者，則必須連絡 [AppBlade 支援小組](mailto:support@appblade.com)。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 AppBlade 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 AppBlade，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [AppBlade]。

    ![設定單一登入](./media/appblade-tutorial/tutorial_appblade_app.png) 

3. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用存取面板來測試您的 Azure AD 單一登入組態。  
當您在「存取面板」中按一下 [AppBlade] 圖格時，應該會自動登入您的 AppBlade 應用程式。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/appblade-tutorial/tutorial_general_01.png
[2]: ./media/appblade-tutorial/tutorial_general_02.png
[3]: ./media/appblade-tutorial/tutorial_general_03.png
[4]: ./media/appblade-tutorial/tutorial_general_04.png

[100]: ./media/appblade-tutorial/tutorial_general_100.png

[200]: ./media/appblade-tutorial/tutorial_general_200.png
[201]: ./media/appblade-tutorial/tutorial_general_201.png
[202]: ./media/appblade-tutorial/tutorial_general_202.png
[203]: ./media/appblade-tutorial/tutorial_general_203.png

