---
title: 教學課程：Azure Active Directory 與 Citrix ShareFile 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Citrix ShareFile 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: e14fc310-bac4-4f09-99ef-87e5c77288b6
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2018
ms.author: jeedes
ms.openlocfilehash: 9919be128ae651b589a37f957cc59ce6d171143f
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39431727"
---
# <a name="tutorial-azure-active-directory-integration-with-citrix-sharefile"></a>教學課程：Azure Active Directory 與 Citrix ShareFile 整合

在本教學課程中，您會了解如何整合 Citrix ShareFile 與 Azure Active Directory (Azure AD)。

Citrix ShareFile 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Citrix ShareFile 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶，自動登入 Citrix ShareFile (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Citrix ShareFile 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Citrix ShareFile 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Citrix ShareFile
1. 設定和測試 Azure AD 單一登入

## <a name="add-citrix-sharefile-from-the-gallery"></a>從資源庫新增 Citrix ShareFile
若要設定 Citrix ShareFile 與 Azure AD 整合，您需要從資源庫將 Citrix ShareFile 新增到受控 SaaS App 清單。

**若要從資源庫新增 Citrix ShareFile，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Citrix ShareFile**，從結果面板中選取 **Citrix ShareFile**，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Citrix ShareFile](./media/sharefile-tutorial/tutorial_sharefile_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Citrix ShareFile 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Citrix ShareFile 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Citrix ShareFile 中的相關使用者之間建立連結關聯性。

在 Citrix ShareFile 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Citrix ShareFile 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Citrix ShareFile 測試使用者](#create-a-citrix-sharefile-test-user)** - 在 Citrix ShareFile 中建立一個與 Azure AD 中代表 Britta Simon 的項目連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Citrix ShareFile 應用程式中設定單一登入。

**若要使用 Citrix ShareFile 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Citrix ShareFile] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/sharefile-tutorial/tutorial_sharefile_samlbase.png)

1. 在 [Citrix ShareFile 網域與 URL] 區段上，執行下列步驟：

    ![Citrix ShareFile 網域與 URL 單一登入資訊](./media/sharefile-tutorial/tutorial_sharefile_url.png)
    
    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<tenant-name>.sharefile.com/saml/login`

    b. 在 [識別碼 (實體識別碼)] 文字方塊中，使用下列模式輸入 URL：

    | |
    |---|
    | `https://<tenant-name>.sharefile.com`|
    | `https://<tenant-name>.sharefile.com/saml/info`|
    | `https://<tenant-name>.sharefile1.com/saml/info`|
    | `https://<tenant-name>.sharefile1.eu/saml/info`|
    | `https://<tenant-name>.sharefile.eu/saml/info`|
    | |
    
    c. 在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：
    | |
    |---|
    | `https://<tenant-name>.sharefile.com/saml/acs`|
    | `https://<tenant-name>.sharefile.eu/saml/<URL path>`|
    | `https://<tenant-name>.sharefile.com/saml/<URL path>`|
    | |

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「單一登入 URL」、「識別碼」及「回覆 URL」來更新這些值。 請連絡 [Citrix ShareFile 用戶端支援小組](https://www.citrix.co.in/products/sharefile/support.html)以取得這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/sharefile-tutorial/tutorial_sharefile_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/sharefile-tutorial/tutorial_general_400.png)

1. 在 [Citrix ShareFile 設定] 區段中，按一下 [設定 Citrix ShareFile] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![Citrix ShareFile 設定](./media/sharefile-tutorial/tutorial_sharefile_configure.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 **Citrix ShareFile** 公司網站。

1. 在頂端工具列中，按一下 [管理] 。

1. 在左側瀏覽窗格中，選取 [設定單一登入] 。
   
    ![帳戶管理](./media/sharefile-tutorial/ic773627.png "帳戶管理")

1. 在 [單一登入/SAML 2.0 組態] 對話頁面的 [基本設定] 下方，執行下列步驟：
   
    ![單一登入](./media/sharefile-tutorial/ic773628.png "單一登入")
   
    a. 按一下 [啟用 SAML] 。
    
    b. 在 [您的 IDP 簽發者/實體識別碼] 文字方塊中，貼上從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    c. 按一下 [X.509 憑證] 欄位旁的 [變更]，然後上傳您從 Azure 入口網站下載的憑證。
    
    d. 在 [登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。
    
    e. 在 [登出 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

1. 在 Citrix ShareFile 管理入口網站上，按一下 [儲存]  。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/sharefile-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/sharefile-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/sharefile-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/sharefile-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-citrix-sharefile-test-user"></a>建立 Citrix ShareFile 測試使用者

若要讓 Azure AD 使用者可以登入 Citrix ShareFile，必須將他們佈建到 Citrix ShareFile。 Citrix ShareFile 需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 登入您的 **Citrix ShareFile** 租用戶。

1. 按一下 [管理使用者] \> [管理使用者首頁] \> [+ 建立員工]。
   
   ![建立員工](./media/sharefile-tutorial/IC781050.png "建立員工")

1. 在 [基本資訊] 區段中，執行下列步驟：
   
   ![基本資訊](./media/sharefile-tutorial/IC799951.png "的基本資訊")
   
   a. 在 [電子郵件地址] 文字方塊中，將 Britta Simon 的電子郵件地址輸入為 **brittasimon@contoso.com**。
   
   b. 在 [名字] 文字方塊中，輸入 **Britta** 作為使用者的**名字**。
   
   c. 在 [姓氏] 文字方塊中，輸入 **Simon** 作為使用者的**姓氏**。

1. 按一下 [新增使用者] 。
  
   >[!NOTE]
   >Azure AD 帳戶持有者將收到一封電子郵件，依循連結來確認其帳戶，帳戶就會生效。您可以使用任何其他 Citrix ShareFile 使用者帳戶建立工具或 Citrix ShareFile 所提供的 API 來佈建 Azure AD 使用者帳戶。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Citrix ShareFile 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Citrix ShareFile，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Citrix ShareFile]。

    ![應用程式清單中的 Citrix ShareFile 連結](./media/sharefile-tutorial/tutorial_sharefile_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Citrix ShareFile] 圖格時，應該會自動登入您的 Citrix ShareFile 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/sharefile-tutorial/tutorial_general_01.png
[2]: ./media/sharefile-tutorial/tutorial_general_02.png
[3]: ./media/sharefile-tutorial/tutorial_general_03.png
[4]: ./media/sharefile-tutorial/tutorial_general_04.png

[100]: ./media/sharefile-tutorial/tutorial_general_100.png

[200]: ./media/sharefile-tutorial/tutorial_general_200.png
[201]: ./media/sharefile-tutorial/tutorial_general_201.png
[202]: ./media/sharefile-tutorial/tutorial_general_202.png
[203]: ./media/sharefile-tutorial/tutorial_general_203.png
