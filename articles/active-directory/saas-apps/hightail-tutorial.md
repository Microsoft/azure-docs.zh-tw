---
title: 教學課程：Azure Active Directory 與 Hightail 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Hightail 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: e15206ac-74b0-46e4-9329-892c7d242ec0
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/15/2018
ms.author: jeedes
ms.openlocfilehash: 1151044d5c1002c808ae1214086aff5fad84a55e
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39431329"
---
# <a name="tutorial-azure-active-directory-integration-with-hightail"></a>教學課程：Azure Active Directory 與 Hightail 整合

在本教學課程中，您將了解如何整合 Hightail 與 Azure Active Directory (Azure AD)。

Hightail 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Hightail 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Hightail (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Hightail 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Hightail 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫加入 Hightail
1. 設定並測試 Azure AD 單一登入

## <a name="adding-hightail-from-the-gallery"></a>從資源庫加入 Hightail
若要設定 Hightail 與 Azure AD 整合，您需要從資源庫將 Hightail 加入受控 SaaS 應用程式清單中。

**若要從資源庫加入 Hightail，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Hightail**。

    ![建立 Azure AD 測試使用者](./media/hightail-tutorial/tutorial_hightail_search.png)

1. 在結果窗格中，選取 [Hightail]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/hightail-tutorial/tutorial_hightail_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Hightail 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Hightail 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Hightail 中的相關使用者之間建立連結關聯性。

在 Hightail 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試對 Hightail 的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Hightail 測試使用者](#creating-a-hightail-test-user)** - 在 Hightail 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Hightail 應用程式中設定單一登入。

**若要使用 Hightail 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Hightail] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![設定單一登入](./media/hightail-tutorial/tutorial_hightail_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Hightail 網域及 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/hightail-tutorial/tutorial_hightail_url.png)

    在 [回覆 URL] 文字方塊中，輸入 URL：`https://www.hightail.com/samlLogin?phi_action=app/samlLogin&subAction=handleSamlResponse`

    > [!NOTE]
    > [回覆 URL] 值不是真正的值。 您將會使用實際的「回覆 URL」來更新 [回覆 URL] 值，本教學課程稍後會說明。

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![設定單一登入](./media/hightail-tutorial/tutorial_hightail_url1.png)

    在 [登入 URL] 文字方塊中，輸入 URL：`https://www.hightail.com/loginSSO`

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/hightail-tutorial/tutorial_hightail_certificate.png) 

1. Hightail 應用程式需要特定格式的 SAML 判斷提示。 請設定此應用程式的下列宣告。 您可以從應用程式的 [屬性] 索引標籤管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。 

    ![設定單一登入](./media/hightail-tutorial/tutorial_hightail_attribute.png) 

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱 | 屬性值 |
    | ------------------- | -------------------- |
    | 名字 | user.givenname |
    | 姓氏 | user.surname |
    | 電子郵件 | user.mail |    
    | UserIdentity | user.mail |
    
    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/hightail-tutorial/tutorial_officespace_04.png)

    ![設定單一登入](./media/hightail-tutorial/tutorial_officespace_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 讓 [命名空間] 保持空白。

    e. 按一下 [確定] 。

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/hightail-tutorial/tutorial_general_400.png)

1. 在 [Hightail 組態] 區段上，按一下 [設定 Hightail] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/hightail-tutorial/tutorial_hightail_configure.png)

    >[!NOTE]
    >在 Hightail 應用程式設定單一登入之前，請透過 Hightail 小組將您的電子郵件網域列入允許清單，以便使用此網域的所有使用者都可以使用單一登入功能。

1. 在另一個瀏覽器視窗中，開啟 **Hightail** 管理入口網站。

1. 按一下頁面右上角的**使用者圖示**。 

    ![設定單一登入](./media/hightail-tutorial/configure1.png)

1. 按一下 [View Admin Console] \(檢視管理主控台\) 索引標籤。

    ![設定單一登入](./media/hightail-tutorial/configure2.png)

1. 在頂端功能表中，按一下 [SAML] 索引標籤，然後執行下列步驟：

    ![設定單一登入](./media/hightail-tutorial/configure3.png)

    a. 在 [Login URL] \(登入 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    b. 在記事本中開啟從 Azure 入口網站下載的 Base-64 編碼憑證，將憑證內容複製到剪貼簿，然後貼到 [SAML Certificate] \(SAML 憑證\) 文字方塊中。

    c. 按一下 [COPY] \(複製\) 以複製執行個體的 SAML 取用者 URL，然後將其貼在 Azure 入口網站上 [Hightail 網域及 URL] 區段的 [回覆 URL] 文字方塊中。

    d. 按一下 [Save Configurations] \(儲存設定\)。

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/hightail-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/hightail-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/hightail-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/hightail-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-hightail-test-user"></a>建立 Hightail 測試使用者

本節目標是在 Hightail 中建立名為 Britta Simon 的使用者。 

在這一節沒有您需要進行的動作項目。 Hightail 支援以自訂宣告為基礎的 just-in-time 使用者佈建。 如果您已如上方 **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** 一節中所示設定自訂宣告，會在尚未存在使用者的應用程式中自動建立使用者。 

>[!NOTE]
>如果您需要手動建立使用者，您必須透過 [Hightail 支援小組](mailto:support@hightail.com)。 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Hightail 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Hightail，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Hightail] 。

    ![設定單一登入](./media/hightail-tutorial/tutorial_hightail_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中按一下 [Hightail] 圖格時，應該會自動登入您的 Hightail 應用程式。


## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/hightail-tutorial/tutorial_general_01.png
[2]: ./media/hightail-tutorial/tutorial_general_02.png
[3]: ./media/hightail-tutorial/tutorial_general_03.png
[4]: ./media/hightail-tutorial/tutorial_general_04.png

[100]: ./media/hightail-tutorial/tutorial_general_100.png

[200]: ./media/hightail-tutorial/tutorial_general_200.png
[201]: ./media/hightail-tutorial/tutorial_general_201.png
[202]: ./media/hightail-tutorial/tutorial_general_202.png
[203]: ./media/hightail-tutorial/tutorial_general_203.png

