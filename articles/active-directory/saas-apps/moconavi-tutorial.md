---
title: 教學課程：Azure Active Directory 與 moconavi 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 moconavi 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: e1916224-e1c2-426f-b233-0a2518fa41db
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/30/2018
ms.author: jeedes
ms.openlocfilehash: 3467b823e6c91d34ebd48c7f8bc29558a79c59e5
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36229541"
---
# <a name="tutorial-azure-active-directory-integration-with-moconavi"></a>教學課程：Azure Active Directory 與 moconavi 整合

在本教學課程中，您將了解如何整合 moconavi 與 Azure Active Directory (Azure AD)。

moconavi 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 moconavi 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 moconavi (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>先決條件

若要設定 Azure AD 與 moconavi 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 moconavi 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 moconavi
2. 設定並測試 Azure AD 單一登入

## <a name="adding-moconavi-from-the-gallery"></a>從資源庫新增 moconavi
若要設定將 moconavi 整合到 Azure AD 中，您需要從資源庫將 moconavi 新增到受控的 SaaS 應用程式清單。

**若要從資源庫新增 moconavi，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **moconavi**，從結果面板中選取 [moconavi]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 moconavi](./media/moconavi-tutorial/tutorial_moconavi_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 moconavi 來設定和測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 moconavi 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 moconavi 中相關使用者之間的連結關聯性。

若要使用 moconavi 來設定和測試 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 moconavi 測試使用者](#create-a-moconavi-test-user)** - 讓 moconavi 中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 moconavi 應用程式中設定單一登入。

**若要使用 moconavi 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [moconavi] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/moconavi-tutorial/tutorial_moconavi_samlbase.png)

3. 在 [moconavi 網域及 URL] 區段上，執行下列步驟：

    ![moconavi 網域及 URL 單一登入資訊](./media/moconavi-tutorial/tutorial_moconavi_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰`https://<yourserverurl>/moconavi-saml2/saml/login`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL：`https://<yourserverurl>/moconavi-saml2`

    C. 在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：`https://<yourserverurl>/moconavi-saml2/saml/SSO`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「單一登入 URL」、「識別碼」及「回覆 URL」來更新這些值。 請連絡 [moconavi 用戶端支援小組](mailto:support@recomot.co.jp)以取得這些值。

4. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/moconavi-tutorial/tutorial_moconavi_certificate.png)

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/moconavi-tutorial/tutorial_general_400.png)

6. 若要在 **moconavi** 端設定單一登入，您必須將已下載的**中繼資料 XML** 傳送給 [moconavi 支援小組](mailto:support@recomot.co.jp)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/moconavi-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/moconavi-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/moconavi-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/moconavi-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-moconavi-test-user"></a>建立 moconavi 測試使用者

在本節中，您要在 moconavi 中建立名為 Britta Simon 的使用者。 請與 [moconavi 支援小組](mailto:support@recomot.co.jp) 合作，在 moconavi 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 moconavi 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 moconavi，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [moconavi]。

    ![應用程式清單中的 moconavi 連結](./media/moconavi-tutorial/tutorial_moconavi_app.png)

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

1. 從 Microsoft Store 安裝 moconavi。

2. 啟動 moconavi。

3. 按一下 [Connect setting] \(連線設定\) 按鈕。

    ![測試單一登入](./media/moconavi-tutorial/testing1.png)

4. 在 [Connect to URL] \(連線至 URL\) 文字方塊中輸入 `https://mcs-admin.moconavi.biz/gateway`，然後按一下 [Done] \(完成\) 按鈕。

    ![測試單一登入](./media/moconavi-tutorial/testing2.png)

5. 在下列螢幕擷取畫面中，執行下列步驟：

    ![測試單一登入](./media/moconavi-tutorial/testing3.png)

    a. 在 [Input Authentication Key] \(輸入驗證金鑰\) 文字方塊中輸入**輸入驗證金鑰**：`azureAD`。

    b. 在 [Input User ID] \(輸入使用者識別碼\) 文字方塊中輸入**輸入使用者識別碼**：`your ad account`。

    c. 按一下 [LOGIN] \(登入\)。

6. 在 [Password] \(密碼\) 文字方塊中輸入您的 Azure AD 密碼，然後按一下 [Login] \(登入\) 按鈕。

    ![測試單一登入](./media/moconavi-tutorial/testing4.png)

7. Azure AD 驗證會在顯示功能表時順利完成。

    ![測試單一登入](./media/moconavi-tutorial/testing5.png)

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/moconavi-tutorial/tutorial_general_01.png
[2]: ./media/moconavi-tutorial/tutorial_general_02.png
[3]: ./media/moconavi-tutorial/tutorial_general_03.png
[4]: ./media/moconavi-tutorial/tutorial_general_04.png

[100]: ./media/moconavi-tutorial/tutorial_general_100.png

[200]: ./media/moconavi-tutorial/tutorial_general_200.png
[201]: ./media/moconavi-tutorial/tutorial_general_201.png
[202]: ./media/moconavi-tutorial/tutorial_general_202.png
[203]: ./media/moconavi-tutorial/tutorial_general_203.png

