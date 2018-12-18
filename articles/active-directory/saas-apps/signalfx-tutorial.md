---
title: 教學課程：Azure Active Directory 與 SignalFx 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SignalFx 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 6d5ab4b0-29bc-4b20-8536-d64db7530f32
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/16/2018
ms.author: jeedes
ms.openlocfilehash: 0d21a409669cc7d7fceeec9787efbe31d880597c
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39437843"
---
# <a name="tutorial-azure-active-directory-integration-with-signalfx"></a>教學課程：Azure Active Directory 與 SignalFx 整合

在本教學課程中，您將了解如何整合 SignalFx 與 Azure Active Directory (Azure AD)。

SignalFx 與 Azure AD 的整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 SignalFx 的人員。
- 您可以讓使用者透過其 Azure AD 帳戶自動登入 SignalFx (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 SignalFx 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 SignalFx 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 SignalFx
1. 設定並測試 Azure AD 單一登入

## <a name="adding-signalfx-from-the-gallery"></a>從資源庫新增 SignalFx
若要進行將 SignalFx 整合到 Azure AD 中的設定，您必須從資源庫將 SignalFx 新增到受控 SaaS 應用程式的清單。

**若要從資源庫新增 SignalFx，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **SignalFx**，從結果面板中選取 [SignalFx]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 SignalFx](./media/signalfx-tutorial/tutorial_signalfx_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 SignalFx 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 SignalFx 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 SignalFx 中的相關使用者之間建立連結關聯性。

若要使用 SignalFx 來設定並測試 Azure AD 單一登入，您必須完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 SignalFx 測試使用者](#create-a-signalfx-test-user)** - 使 SignalFx 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 SignalFx 應用程式中設定單一登入。

**若要使用 SignalFx 設定 Azure AD 單一登入功能，請執行下列步驟：**

1. 在 Azure 入口網站的 [SignalFx] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/signalfx-tutorial/tutorial_signalfx_samlbase.png)

1. 在 [SignalFx 網域與 URL] 區段上，執行下列步驟：

    ![SignalFx 網域與 URL 單一登入資訊](./media/signalfx-tutorial/tutorial_signalfx_url.png)

    a. 在 [識別碼] 文字方塊中，輸入 URL：`https://api.signalfx.com/v1/saml/metadata`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://api.signalfx.com/v1/saml/acs/<integration ID>`

    > [!NOTE] 
    > 上述值並非真正的值。 您會使用實際的「回覆 URL」來更新值，稍後會在本教學課程中說明。

1. SignalFx 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中，管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。   

    ![設定單一登入](./media/signalfx-tutorial/tutorial_signalfx_attribute.png)

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱 | 屬性值 |
    | ------------------- | -------------------- |    
    | User.FirstName          | user.givenname |
    | User.email          | user.mail |
    | PersonImmutableID       | user.userprincipalname    |
    | User.LastName       | user.surname    |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入的新增](./media/signalfx-tutorial/tutorial_attribute_04.png)

    ![設定單一登入的新增屬性](./media/signalfx-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 讓 [命名空間] 保持空白。
    
    e. 按一下 [確定] 。
 
1. 在 [SAML 簽署憑證] 區段上，執行下列步驟： 

    ![憑證下載連結](./media/signalfx-tutorial/tutorial_signalfx_certificate.png)

    a. 按一下「複製」按鈕複製「應用程式同盟中繼資料 URL」，並將它貼到 [記事本]。

    b. 按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/signalfx-tutorial/tutorial_general_400.png)

1. 在 [SignalFx 組態] 區段上，按一下 [設定 SignalFx] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體識別碼]。

    ![SignalFx 組態](./media/signalfx-tutorial/tutorial_signalfx_configure.png) 

1. 以系統管理員身分登入您的 SignalFx 公司網站。

1. 在 SignalFx 頂端按一下 [整合]，以開啟 [整合] 頁面。

    ![SignalFx 整合](./media/signalfx-tutorial/tutorial_signalfx_intg.png)

1. 按一下 [登入服務] 區段下的 [Azure Active Directory] 圖格。
 
    ![SignalFx saml](./media/signalfx-tutorial/tutorial_signalfx_saml.png)

1. 按一下 [新增整合]，並在 [安裝] 索引標籤下執行下列步驟：
 
    ![SignalFx samlintgpage](./media/signalfx-tutorial/tutorial_signalfx_azure.png)

    a. 在 [名稱] 文字方塊中輸入新的整合名稱，例如 **OurOrgName SAML SSO**。

    b. 複製 [整合識別碼] 值，並為其附加 [回覆 URL]，如同 Azure 入口網站中 [SignalFx 網域與 URL] 區段之 [回覆 URL] 文字方塊中的 `https://api.signalfx.com/v1/saml/acs/<integration ID>`。

    c. 按一下 [上傳檔案]，將從 Azure 入口網站下載的 [Base64 編碼憑證] 上傳至 [憑證] 文字方塊。

    d. 在 [簽發者 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    e. 在 [中繼資料 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [應用程式同盟中繼資料 Url]。

    f. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/signalfx-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/signalfx-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/signalfx-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/signalfx-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
  
### <a name="create-a-signalfx-test-user"></a>建立 SignalFx 測試使用者

本節目標是在 SignalFx 中建立名為 Britta Simon 的使用者。 SignalFx 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 當您嘗試存取 SignalFx 時，如果 SignalFx 還沒有使用者，它將會建立新的使用者。

當使用者第一次從 SAML SSO 登入 SignalFx 時，[SignalFx 支援小組](mailto:kmazzola@signalfx.com)會傳送一則電子郵件給使用者，其中包含他們必須點按以進行驗證的連結。 只有使用者第一次登入時才會有此要求；後續的登入嘗試即不需要電子郵件驗證。

>[!Note]
>如果您需要手動建立使用者，請連絡 [SignalFx 支援小組](mailto:kmazzola@signalfx.com)

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 SignalFx 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 SignalFx，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [SignalFx]。

    ![應用程式清單中的 SignalFx 連結](./media/signalfx-tutorial/tutorial_signalfx_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [SignalFx] 圖格時，應該會自動登入您的 SignalFx 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/signalfx-tutorial/tutorial_general_01.png
[2]: ./media/signalfx-tutorial/tutorial_general_02.png
[3]: ./media/signalfx-tutorial/tutorial_general_03.png
[4]: ./media/signalfx-tutorial/tutorial_general_04.png

[100]: ./media/signalfx-tutorial/tutorial_general_100.png

[200]: ./media/signalfx-tutorial/tutorial_general_200.png
[201]: ./media/signalfx-tutorial/tutorial_general_201.png
[202]: ./media/signalfx-tutorial/tutorial_general_202.png
[203]: ./media/signalfx-tutorial/tutorial_general_203.png

