---
title: 教學課程：Azure Active Directory 與 Proxyclick 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Proxyclick 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 5c58a859-71c2-4542-ae92-e5f16a8e7f18
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/21/2018
ms.author: jeedes
ms.openlocfilehash: d93c5486d9c23558995742fc27e1222834cf4452
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39446308"
---
# <a name="tutorial-azure-active-directory-integration-with-proxyclick"></a>教學課程：Azure Active Directory 與 Proxyclick 整合

在本教學課程中，您會了解如何整合 Proxyclick 與 Azure Active Directory (Azure AD)。

將 Proxyclick 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Proxyclick 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 Proxyclick (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Proxyclick 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用單一登入的 Proxyclick 訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Proxyclick
1. 設定並測試 Azure AD 單一登入

## <a name="adding-proxyclick-from-the-gallery"></a>從資源庫新增 Proxyclick
若要設定以將 Proxyclick 整合到 Azure AD 中，您需要從資源庫將 Proxyclick 新增到受控的 SaaS 應用程式清單。

**若要從資源庫新增 Proxyclick，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Proxyclick**，從結果面板中選取 [Proxyclick]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 [Proxyclick]](./media/proxyclick-tutorial/tutorial_proxyclick_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Proxyclick 來設定和測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 需要知道 Proxyclick 與 Azure AD 中互相對應的使用者。 換句話說，需要在 Azure AD 使用者和 Proxyclick 中的相關使用者之間建立連結關聯性。

若要搭配 Proxyclick 設定和測試 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Proxyclick 測試使用者](#create-a-proxyclick-test-user)** - 讓 Proxyclick 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 Proxyclick 應用程式中設定單一登入。

**若要搭配 Proxyclick 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Proxyclick] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/proxyclick-tutorial/tutorial_proxyclick_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Proxyclick 網域及 URL] 區段上執行下列步驟：

    ![[Proxyclick 網域及 URL] 單一登入資訊](./media/proxyclick-tutorial/tutorial_proxyclick_url2.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://saml.proxyclick.com/init/<companyId>`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://saml.proxyclick.com/consume/<companyId>`

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![[Proxyclick 網域及 URL] 單一登入資訊](./media/proxyclick-tutorial/tutorial_proxyclick_url1.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://saml.proxyclick.com/init/<companyId>`

    > [!NOTE]
    > 這些都不是真正的值。 您將會使用實際的識別碼、回覆 URL 與登入 URL (稍後會在本教學課程中說明) 來更新這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/proxyclick-tutorial/tutorial_proxyclick_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/proxyclick-tutorial/tutorial_general_400.png)

1. 在 [Proxyclick 設定] 區段上，按一下 [設定 Proxyclick] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![[Proxyclick 設定]](./media/proxyclick-tutorial/tutorial_proxyclick_configure.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Proxyclick 公司網站。

1. 選取 [Account & Settings] \(帳戶和設定\)。

    ![[Proxyclick 設定]](./media/proxyclick-tutorial/configure1.png)

1. 向下捲動至 [INTEGRATIONS] \(整合\)，並選取 [SAML]。

    ![[Proxyclick 設定]](./media/proxyclick-tutorial/configure2.png)

1. 在 [SAML] 區段中，執行下列步驟：

    ![[Proxyclick 設定]](./media/proxyclick-tutorial/configure3.png)

    a. 複製 [SAML Consumer URL] \(SAML 取用者 URL\) 值，並將其貼到 Azure 入口網站上 [Proxyclick 網域及 URL] 區段的 [回覆 URL] 文字方塊中。

    b. 複製 [SAML SSO Redirect URL] \(SAML SSO 重新導向 URL\) 值，並將其貼到 Azure 入口網站上 [Proxyclick 網域及 URL] 區段的 [登入 URL] 和 [識別碼] 文字方塊中。

    c. 將 [SAML Request Method] \(SAML 要求方法\) 選取為 [HTTP Redirect] \(HTTP 重新導向\)。

    d. 在 [Issuer] \(簽發者\) 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    e. 在 [SAML 2.0 Endpoint URL] \(SAML 2.0 端點 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    f. 在記事本中開啟您從 Azure 入口網站下載的憑證檔，然後將它貼到 [Certificate] \(憑證\) 文字方塊中。

    g. 按一下 [儲存變更] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/proxyclick-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/proxyclick-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/proxyclick-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/proxyclick-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-proxyclick-test-user"></a>建立 Proxyclick 測試使用者

若要讓 Azure AD 使用者能夠登入 Proxyclick，必須將使用者佈建到 Proxyclick。 Proxyclick 需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 Proxyclick 公司網站。

1. 在頂端導覽列上按一下 [Colleagues] \(同事\)。

    ![新增員工](./media/proxyclick-tutorial/user1.png)

1. 按一下 [Add Colleague] \(新增同事\)。

    ![新增員工](./media/proxyclick-tutorial/user2.png)

1. 在 [Add a colleague] \(新增同事\) 區段中，執行下列步驟：

    ![新增員工](./media/proxyclick-tutorial/user3.png)

    a. 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件地址，例如 **brittasimon@contoso.com**。

    b. 在 [名字] 文字方塊中，輸入使用者的名字，例如 Britta。

    c. 在 [姓氏] 文字方塊中，輸入使用者的姓氏，例如 Simon。

    d. 按一下 [新增使用者] 。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Proxyclick 的存取權授與 Britta Simon，讓該使用者能使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Proxyclick，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Proxyclick]。

    ![應用程式清單中的 [Proxyclick] 連結](./media/proxyclick-tutorial/tutorial_proxyclick_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您按一下 [存取面板] 中的 [Proxyclick] 圖格時，應該就會自動登入 Proxyclick 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/proxyclick-tutorial/tutorial_general_01.png
[2]: ./media/proxyclick-tutorial/tutorial_general_02.png
[3]: ./media/proxyclick-tutorial/tutorial_general_03.png
[4]: ./media/proxyclick-tutorial/tutorial_general_04.png

[100]: ./media/proxyclick-tutorial/tutorial_general_100.png

[200]: ./media/proxyclick-tutorial/tutorial_general_200.png
[201]: ./media/proxyclick-tutorial/tutorial_general_201.png
[202]: ./media/proxyclick-tutorial/tutorial_general_202.png
[203]: ./media/proxyclick-tutorial/tutorial_general_203.png

