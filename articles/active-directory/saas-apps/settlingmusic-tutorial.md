---
title: 教學課程：Azure Active Directory 與 Settling music 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Settling music 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 6f86a8a2-4bd0-40cc-b1b4-752fce123328
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/07/2018
ms.author: jeedes
ms.openlocfilehash: fda6ca2efb670c8087252428e417a3e0901fa748
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39449058"
---
# <a name="tutorial-azure-active-directory-integration-with-settling-music"></a>教學課程：Azure Active Directory 與 Settling music 整合

在本教學課程中，您會了解如何整合 Settling music 與 Azure Active Directory (Azure AD)。

將 Settling music 與 Azure AD 整合提供以下優點：

- 您可以在 Azure AD 中控制可存取 Settling music 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 Settling music (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Settling music 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用單一登入的 Settling music 訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Settling music
1. 設定並測試 Azure AD 單一登入

## <a name="adding-settling-music-from-the-gallery"></a>從資源庫新增 Settling music
若要設定以將 Settling music 整合到 Azure AD 中，您必須從資源庫將 Settling music 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Settling music，請執行以下步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Settling music**，從結果面板中選取 [Settling music]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 [Settling music]](./media/settlingmusic-tutorial/tutorial_settlingmusic_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Settling music 來設定和測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 需要知道 Settling music 與 Azure AD 中互相對應的使用者。 換句話說，需要在 Azure AD 使用者與 Settling music 中的相關使用者之間建立連結關聯性。

若要搭配 Settling music 設定和測試 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Settling music 測試使用者](#create-a-settling-music-test-user)** - 讓 Settling music 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Settling music 應用程式中設定單一登入。

**若要搭配 Settling music 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Settling music] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/settlingmusic-tutorial/tutorial_settlingmusic_samlbase.png)

1. 在 [Settling music 網域及 URL] 區段上，執行下列步驟：

    ![[Settling music 網域及 URL] 單一登入資訊](./media/settlingmusic-tutorial/tutorial_settlingmusic_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<SUBDOMAIN>.rakurakuseisan.jp/<USERACCOUNT>/`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<SUBDOMAIN>.rakurakuseisan.jp/<USERACCOUNT>/`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Settling music 用戶端支援小組](https://rakurakuseisan.jp/) \(日文\) 以取得這些值。 
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/settlingmusic-tutorial/tutorial_settlingmusic_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/settlingmusic-tutorial/tutorial_general_400.png)

1. 在 [Settling music 設定] 區段上，按一下 [設定 Settling music] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL 和 SAML 單一登入服務 URL]。

    ![[Settling music 設定]](./media/settlingmusic-tutorial/tutorial_settlingmusic_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以安全性系統管理員身分登入 Settling music。

1. 在頁面頂端按一下 [management] \(管理\) 索引標籤。

    ![Settling music 步驟 1](./media/settlingmusic-tutorial/tutorial_settlingmusic_step1.png)

1. 按一下 [System setting] \(系統設定\) 索引標籤。

    ![Settling music 步驟 2](./media/settlingmusic-tutorial/tutorial_settlingmusic_step2.png)

1. 切換至 [Security] \(安全性\) 索引標籤。

    ![Settling music 步驟 3](./media/settlingmusic-tutorial/tutorial_settlingmusic_step3.png)

1. 在 [Single sign-on setting] \(單一登入設定\) 區段上，執行下列步驟：

    ![Settling music 步驟 5](./media/settlingmusic-tutorial/tutorial_settlingmusic_step4.png)

    a. 按一下 [To enable] \(啟用\)。

    b. 在 [Login URL of the ID provider] \(識別碼提供者的登入 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    c. 在 [ID provider logout URL] \(識別碼提供者登出 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    d. 按一下 [Choose File] \(選擇檔案\) 以上傳您從 Azure 入口網站下載的**憑證 (Base64)**。

    e. 按一下 [儲存]  按鈕。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/settlingmusic-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/settlingmusic-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/settlingmusic-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/settlingmusic-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-settling-music-test-user"></a>建立 Settling music 測試使用者

在本節中，您會在 Settling music 中建立名為 Britta Simon 的使用者。 請與 [Settling music 用戶端支援小組](https://rakurakuseisan.jp/) \(日文\) 合作，以在 Settling music 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Settling music 的存取權授與 Britta Simon，讓該使用者能使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派到 Settling music，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Settling music]。

    ![應用程式清單中的 [Settling music] 連結](./media/settlingmusic-tutorial/tutorial_settlingmusic_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您按一下 [存取面板] 中的 [Settling music] 圖格時，系統應該會自動將您登入 Settling music 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/settlingmusic-tutorial/tutorial_general_01.png
[2]: ./media/settlingmusic-tutorial/tutorial_general_02.png
[3]: ./media/settlingmusic-tutorial/tutorial_general_03.png
[4]: ./media/settlingmusic-tutorial/tutorial_general_04.png

[100]: ./media/settlingmusic-tutorial/tutorial_general_100.png

[200]: ./media/settlingmusic-tutorial/tutorial_general_200.png
[201]: ./media/settlingmusic-tutorial/tutorial_general_201.png
[202]: ./media/settlingmusic-tutorial/tutorial_general_202.png
[203]: ./media/settlingmusic-tutorial/tutorial_general_203.png

