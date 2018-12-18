---
title: 教學課程：Azure Active Directory 與 Help Scout 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Help Scout 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 0aad9910-0bc1-4394-9f73-267cf39973ab
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/14/2017
ms.author: jeedes
ms.openlocfilehash: 0bbdf576c38207349bb45e7b54f3ffc85ecf3d36
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39449429"
---
# <a name="tutorial-azure-active-directory-integration-with-help-scout"></a>教學課程：Azure Active Directory 與 Help Scout 整合

在本教學課程中，您會了解如何將 Help Scout 與 Azure Active Directory (Azure AD) 整合。

Help Scout 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中管控可存取 Help Scout 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Help Scout (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Help Scout 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Help Scout 單一登入的訂用帳戶

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Help Scout
1. 設定並測試 Azure AD 單一登入

## <a name="adding-help-scout-from-the-gallery"></a>從資源庫新增 Help Scout
若要設定將 Help Scout 整合到 Azure AD 中，您需要從資源庫將 Help Scout 新增到受控 SaaS app 清單。

**若要從資源庫新增 Help Scout，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Help Scout**，從結果面板中選取 [Help Scout]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Help Scout](./media/helpscout-tutorial/tutorial_helpscout_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Help Scout 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Help Scout 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Help Scout 中相關使用者之間的連結關聯性。

Help Scout 使用電子郵件地址來登入，因此若要建立連結關聯性，請在 Azure AD 中使用相同的**電子郵件地址**作為**使用者名稱**。

若要使用 Help Scout 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Help Scout 測試使用者](#create-a-help-scout-test-user)** - 在 Help Scout 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Help Scout 應用程式中設定單一登入。

**若要使用 Help Scout 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Help Scout] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/helpscout-tutorial/tutorial_helpscout_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Help Scout 網域和 URL] 區段上執行下列步驟：

    ![Help Scout 網域與 URL 單一登入資訊](./media/helpscout-tutorial/tutorial_helpscout_url.png)

    a. **識別碼**是 Help Scout 的 **「對象 URI (服務提供者實體識別碼)」**，以 `urn:` 開始

    b. **回覆 URL** 是 Help Scout 的 **「回傳 URL (判斷提示取用者服務 URL)」**，以 `https://` 開始 

    > [!NOTE] 
    > 這些 URL 中的值僅供示範。 您必須從實際的「回覆 URL」和「識別碼」來更新這些值。 您可以從 [驗證] 區段下的 [單一登入] 索引標籤取得這些值，本教學課程稍後會說明。

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![Help Scout 網域與 URL 單一登入資訊](./media/helpscout-tutorial/tutorial_helpscout_url1.png)

    在 [登入 URL] 文字方塊中，將 URL 輸入為：`https://secure.helpscout.net/members/login/`
     
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/helpscout-tutorial/tutorial_helpscout_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/helpscout-tutorial/tutorial_general_400.png)


1. 在 [Help Scout 設定] 區段上，按一下 [設定 Help Scout] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/helpscout-tutorial/config.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Help Scout 公司網站。

1. 登入後，在頂端功能表中按一下「管理」，然後從下拉式功能表中選取「公司」。

    ![設定單一登入](./media/helpscout-tutorial/settings1.png) 
 
1. 在左側功能表中選取「驗證」。 

    ![設定單一登入](./media/helpscout-tutorial/settings2.png) 

1. 這會帶您前往 [SAML 設定] 區段，請執行下列步驟：

    ![設定單一登入](./media/helpscout-tutorial/settings3.png) 
 
    a. 複製**回傳 URL (判斷提示取用者服務 URL)** 值，並在 Azure 入口網站的 Help Scout [網域和 URL] 區段下的 [回覆 URL] 方塊中貼上值。
    
    b. 複製**對象 URI (服務提供者實體識別碼)** 值，並在 Azure 入口網站的 Help Scout [網域和 URL] 區段下的 [識別碼] 方塊中貼上值。

1. 將 [啟用 SAML] 切換為開啟，並執行下列步驟：

    ![設定單一登入](./media/helpscout-tutorial/settings4.png) 
 
    a. 在 [單一登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [單一登入服務 URL] 值。
    
    b. 按一下 [上傳憑證] 以上傳從 Azure 入口網站下載的**憑證 (Base64)**。

    c. 在 [電子郵件網域] 文字方塊中，輸入組織的電子郵件網域，如 `contoso.com`。 您可以使用逗號來分隔多個網域。 每當 Help Scout 使用者或系統管理員在 [Help Scout 登入頁面](https://secure.helpscout.net/members/login/)中輸入該特定網域時，便會路由傳送至「識別提供者」，以驗證其認證。

    d. 最後，如果您希望使用者只能透過此方法登入 Help Scout，則可以切換開啟 [強制 SAML 登入]。 如果您仍想保留以 Help Scout 認證登入的選項，則可以將其切換為關閉。 即使啟用此選項，帳戶擁有者一律可以使用其帳戶密碼來登入 Help Scout。

    e. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/helpscout-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/helpscout-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/helpscout-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/helpscout-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-help-scout-test-user"></a>建立 Help Scout 測試使用者

本節目標是在 Help Scout 中建立名為 Britta Simon 的使用者。 Help Scout 支援預設啟用的 Just-In-Time 佈建。

在這一節沒有您需要進行的動作項目。 如果 Help Scout 中還沒有使用者，當您嘗試存取 Help Scout 時，就會建立新的使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Help Scout 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Help Scout，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Help Scout]。

    ![應用程式清單中的 Help Scout 連結](./media/helpscout-tutorial/tutorial_helpscout_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Help Scout] 圖格時，應該會自動登入您的 Help Scout 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/helpscout-tutorial/tutorial_general_01.png
[2]: ./media/helpscout-tutorial/tutorial_general_02.png
[3]: ./media/helpscout-tutorial/tutorial_general_03.png
[4]: ./media/helpscout-tutorial/tutorial_general_04.png

[100]: ./media/helpscout-tutorial/tutorial_general_100.png

[200]: ./media/helpscout-tutorial/tutorial_general_200.png
[201]: ./media/helpscout-tutorial/tutorial_general_201.png
[202]: ./media/helpscout-tutorial/tutorial_general_202.png
[203]: ./media/helpscout-tutorial/tutorial_general_203.png

