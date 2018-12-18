---
title: 教學課程：Azure Active Directory 與 Zoom 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Zoom 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 0ebdab6c-83a8-4737-a86a-974f37269c31
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/28/2017
ms.author: jeedes
ms.openlocfilehash: 57ae31245a356a4cd5769fe71ef471922bf6faf9
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39440129"
---
# <a name="tutorial-azure-active-directory-integration-with-zoom"></a>教學課程：Azure Active Directory 與 Zoom 整合

在本教學課程中，您將了解如何整合 Zoom 與 Azure Active Directory (Azure AD)。

Zoom 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Zoom 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Zoom (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Zoom 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Zoom 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Zoom
1. 設定並測試 Azure AD 單一登入

## <a name="adding-zoom-from-the-gallery"></a>從資源庫新增 Zoom
若要設定將 Zoom 整合到 Azure AD 中，您需要從資源庫將 Zoom 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Zoom，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Zoom**，從結果面板中選取 [Zoom]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Zoom](./media/zoom-tutorial/tutorial_zoom_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Zoom 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Zoom 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Zoom 中相關使用者之間的連結關聯性。

在 Zoom 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Zoom 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Zoom 測試使用者](#create-a-zoom-test-user)** - 使 Zoom 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Zoom 應用程式中設定單一登入。

**若要使用 Zoom 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Zoom] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/zoom-tutorial/tutorial_zoom_samlbase.png)

1. 在 [Zoom 網域與 URL] 區段中，執行下列步驟：

    ![Zoom 網域與 URL 單一登入資訊](./media/zoom-tutorial/tutorial_zoom_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<companyname>.zoom.us`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `<companyname>.zoom.us`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Zoom 客戶支援小組](https://support.zoom.us/hc)以取得這些值。

1. Zoom 應用程式會預期要有特定格式的 SAML 判斷提示，這需要您將自訂屬性對應新增到您的 SAML 權杖屬性設定。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 

    ![設定單一登入](./media/zoom-tutorial/tutorial_attribute.png)

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱 | 屬性值 | 命名空間值 |
    | ------------------- | -----------|--------- |    
    | 電子郵件地址 | user.mail | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/mail`|
    | 名字 | user.givenname | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`|
    | 姓氏 | user.surname | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname `|
    | 電話號碼 | user.telephonenumber | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/phone`|
    | department | user.department | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/department`|

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/zoom-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/zoom-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 在 [命名空間] 文字方塊中，輸入該資料列顯示的命名空間值。
    
    e. 按一下 [確定] 。 
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/zoom-tutorial/tutorial_zoom_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/zoom-tutorial/tutorial_general_400.png)

1. 在 [Zoom 組態] 區段上，按一下 [設定 Zoom] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![Zoom 組態](./media/zoom-tutorial/tutorial_zoom_configure.png)

1. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Zoom 公司網站。

1. 按一下 [單一登入]  索引標籤。
   
    ![[單一登入] 索引標籤](./media/zoom-tutorial/IC784700.png "單一登入")

1. 按一下 [安全性控制] 索引標籤，然後移至 [單一登入] 設定。

1. 在 [單一登入] 區段中，執行下列步驟：
   
    ![[單一登入] 區段](./media/zoom-tutorial/IC784701.png "單一登入")
   
    a. 在 [登入頁面 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。
   
    b. 在 [登出頁面 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。
     
    c. 在記事本中開啟您的 base-64 編碼的憑證，將其內容複製到您的剪貼簿，然後貼到 [識別提供者憑證]  文字方塊中。

    d. 在 [簽發者] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。 

    e. 按一下 [檔案] 。

    > [!NOTE] 
    > 如需詳細資訊，請瀏覽 Zoom 文件[https://zoomus.zendesk.com/hc/en-us/articles/115005887566](https://zoomus.zendesk.com/hc/en-us/articles/115005887566)

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/zoom-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/zoom-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/zoom-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/zoom-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-zoom-test-user"></a>建立 Zoom 測試使用者

若要讓 Azure AD 使用者可以登入 Zoom，必須將他們佈建到 Zoom。 Zoom 需以手動的方式佈建。

### <a name="to-provision-a-user-account-perform-the-following-steps"></a>若要佈建使用者帳戶，請執行下列步驟：

1. 以系統管理員身分登入您的 **Zoom** 公司網站。
 
1. 按一下 [帳戶管理] 索引標籤，再按一下 [使用者管理]。

1. 在 [使用者管理] 區段中按一下 [新增使用者] 。
   
    ![使用者管理](./media/zoom-tutorial/IC784703.png "使用者管理")

1. 在 [新增使用者]  頁面上，執行下列步驟：
   
    ![新增使用者](./media/zoom-tutorial/IC784704.png "新增使用者")
   
    a. 選取 [基本] 做為 [使用者類型]。

    b. 在 [電子郵件]  文字方塊中，輸入您要佈建之有效 Azure AD 帳戶的電子郵件地址。

    c. 按一下 [新增] 。

> [!NOTE]
> 您可以使用任何其他的 Zoom 使用者帳戶建立工具，或 Zoom 提供的 API，以佈建 Azure Active Directory 使用者帳戶。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Zoom 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Zoom，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Zoom]。

    ![應用程式清單中的 Zoom 連結](./media/zoom-tutorial/tutorial_zoom_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

本節的目標是要使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中按一下 [Zoom] 圖格時，應該會自動登入您的 Zoom 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/zoom-tutorial/tutorial_general_01.png
[2]: ./media/zoom-tutorial/tutorial_general_02.png
[3]: ./media/zoom-tutorial/tutorial_general_03.png
[4]: ./media/zoom-tutorial/tutorial_general_04.png

[100]: ./media/zoom-tutorial/tutorial_general_100.png

[200]: ./media/zoom-tutorial/tutorial_general_200.png
[201]: ./media/zoom-tutorial/tutorial_general_201.png
[202]: ./media/zoom-tutorial/tutorial_general_202.png
[203]: ./media/zoom-tutorial/tutorial_general_203.png

