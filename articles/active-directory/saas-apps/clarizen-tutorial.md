---
title: 教學課程：Azure Active Directory 與 Clarizen 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Clarizen 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 28acce3e-22a0-4a37-8b66-6e518d777350
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2017
ms.author: jeedes
ms.openlocfilehash: 510bf383848725f3864c40af02c2b309370237f0
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39438081"
---
# <a name="tutorial-azure-active-directory-integration-with-clarizen"></a>教學課程：Azure Active Directory 與 Clarizen 整合

在本教學課程中，您會了解如何整合 Azure Active Directory (Azure AD) 與 Clarizen。 這項整合提供下列優點︰

- 您可以在 Azure AD 中控制可存取 Clarizen 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Clarizen (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

本教學課程中的案例由二個主要工作組成：

1. 從資源庫新增 Clarizen。
1. 設定和測試 Azure AD 單一登入。

若您想了解軟體即服務 (SaaS) 應用程式與 Azure AD 整合的更多詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件
若要設定與 Clarizen 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用單一登入的 Clarizen 訂用帳戶

若要測試本教學課程中的步驟，請遵循下列建議：

- 在測試環境中測試 Azure AD 單一登入。 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 測試環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="add-clarizen-from-the-gallery"></a>從資源庫新增 Clarizen
若要設定 Clarizen 與 Azure AD 整合，從資源庫將 Clarizen 新增至受控 SaaS 應用程式清單中。

1. 在 [Azure 入口網站](https://portal.azure.com)的左窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 圖示][1]

1. 按一下 [企業應用程式]。 然後按一下 [所有應用程式]。

    ![按一下 [企業應用程式] 和 [所有應用程式]][2]

1. 按一下對話方塊頂端的 [新增] 按鈕。

    ![[新增] 按鈕][3]

1. 在搜尋方塊中，輸入 **Clarizen**。

    ![在搜尋方塊中，輸入「Clarizen」](./media/clarizen-tutorial/tutorial_clarizen_000.png)

1. 在結果窗格中，選取 [Clarizen]，然後按一下 [新增] 以新增應用程式。

    ![在結果窗格中選取 Clarizen](./media/clarizen-tutorial/tutorial_clarizen_0001.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在下列章節中，您會以 Britta Simon 的測試使用者身分，使用 Clarizen 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Clarizen 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Clarizen 中相關使用者之間的連結關聯性。 將 Azure AD 中**使用者名稱**的值指派為 Clarizen 中 **Username** 的值，即可建立此連結關聯性。

若要設定及測試與 Clarizen 搭配運作的 Azure AD 單一登入，完成下列建構元素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)**，讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)**，以使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Clarizen 測試使用者](#create-a-clarizen-test-user)**，使 Clarizen 中對應的 Britta Simon 連結到她在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)**，讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)** 驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入
在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Clarizen 應用程式中設定單一登入。

1. 在 Azure 入口網站的 [Clarizen] 應用程式整合頁面上，按一下 [單一登入]。

    ![按一下 [單一登入]][4]

1. 在 [單一登入] 對話方塊上，選取 [SAML 登入] 做為 [模式]，以啟用單一登入。

    ![選取 [SAML 登入]](./media/clarizen-tutorial/tutorial_clarizen_01.png)

1. 在 [Clarizen 網域及 URL] 區段上，執行下列步驟：

    ![識別碼和回覆 URL 方塊](./media/clarizen-tutorial/tutorial_clarizen_02.png)

    a. 在 [識別碼] 方塊中，以下列形式輸入值：[Clarizen]

    b. 在 [回覆 URL] 方塊中，使用下列模式輸入 URL：**https://<company name>.clarizen.com/Clarizen/Pages/Integrations/SAML/SamlResponse.aspx**

    > [!NOTE]
    > 這些不是真正的值。 您必須使用實際的識別碼和回覆 URL。 在此建議您使用唯一的字串值做為識別碼。 若要取得實際的值，請連絡 [Clarizen 支援小組](https://success.clarizen.com/hc/en-us/requests/new)。

1. 在 [SAML 簽署憑證] 區段上，按一下 [建立新憑證]。

    ![按一下 [建立新的憑證]](./media/clarizen-tutorial/tutorial_clarizen_03.png)    

1. 在 [建立新的憑證] 對話方塊中，按一下行事曆圖示並選取到期日。 然後按一下 [儲存] 。

    ![選取並儲存到期日](./media/clarizen-tutorial/tutorial_general_300.png)

1. 在 [SAML 簽署憑證] 區段上，選取 [啟用新憑證]，然後按一下 [儲存]。

    ![選取核取方塊以啟用新憑證](./media/clarizen-tutorial/tutorial_clarizen_04.png)

1. 在 [變換憑證] 對話方塊中，按一下 [確定]。

    ![按一下 [確定] 確認您想要啟用憑證](./media/clarizen-tutorial/tutorial_general_400.png)

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![按一下 [憑證 (Base64)] 以開始下載](./media/clarizen-tutorial/tutorial_clarizen_05.png)

1. 在 [Clarizen 組態] 區段上，按一下 [設定 Clarizen] 以開啟 [設定登入] 視窗。

    ![按一下 [設定 Clarizen]](./media/clarizen-tutorial/tutorial_clarizen_06.png)

    ![[登入設定] 視窗，包括檔案和 URL](./media/clarizen-tutorial/tutorial_clarizen_07.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Clarizen 公司網站。

1. 按一下您的使用者名稱，然後按一下 [設定]。

    ![按一下您的使用者名稱底下的 [設定]](./media/clarizen-tutorial/tutorial_clarizen_001.png "設定")

1. 按一下 [全域設定] 索引標籤。然後，在 [同盟驗證] 旁邊，按一下 [編輯]。

    ![[全域設定] 索引標籤](./media/clarizen-tutorial/tutorial_clarizen_002.png "全域設定")

1. 在 [同盟驗證] 對話方塊中，執行下列步驟：

    ![[同盟驗證] 對話方塊](./media/clarizen-tutorial/tutorial_clarizen_003.png "同盟驗證")

    a. 選取 [啟用同盟驗證]。

    b. 按一下 [上傳]  來上傳您下載的憑證。

    c. 在 [單一登入 URL] 文字方塊中，輸入來自 Azure AD 應用程式組態視窗的 [SAML 單一登入服務 URL] 值。

    d. 在 [單一登入 URL] 方塊中，輸入來自 Azure AD 應用程式組態視窗的 [登出 URL] 值。

    e. 選取 [使用 POST] 。

    f. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
在 Azure 入口網站中，建立名稱為 Britta Simon 的測試使用者。

![Azure AD 測試使用者的名稱和電子郵件地址][100]

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 圖示](./media/clarizen-tutorial/create_aaduser_01.png)

1. 按一下 [使用者和群組]，然後按一下 [所有使用者] 以顯示使用者清單。

    ![按一下 [使用者和群組] 與 [所有使用者]](./media/clarizen-tutorial/create_aaduser_02.png)

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。

    ![[新增] 按鈕](./media/clarizen-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![在 [使用者] 對話方塊中填入名稱、電子郵件地址和密碼](./media/clarizen-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入 Britta Simon 帳戶的電子郵件地址。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-clarizen-test-user"></a>建立 Clarizen 測試使用者

本節的目標是在 Clarizen 中建立名為 Britta Simon 的使用者。 Clarizen 支援自動使用者佈建，該功能預設為啟用。 您可以在[這裡](clarizen-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

**如果您需要手動建立使用者，請執行下列步驟：**

若要讓 Azure AD 使用者登入 Clarizen，您必須佈建使用者帳戶。 Clarizen 需以手動方式佈建。

1. 以系統管理員身分登入您的 Clarizen 公司網站。

1. 按一下 [人員] 。

    ![按一下 [人員]](./media/clarizen-tutorial/create_aaduser_001.png "人員")

1. 按一下 [邀請使用者] 。

    ![[邀請使用者] 按鈕](./media/clarizen-tutorial/create_aaduser_002.png "邀請使用者")

1. 在 [邀請人員] 對話方塊中，執行下列步驟：

    ![[邀請人員] 對話方塊](./media/clarizen-tutorial/create_aaduser_003.png "邀請人員")

    a. 在 [電子郵件] 方塊中，輸入 Britta Simon 帳戶的電子郵件地址。

    b. 按一下 [邀請] 。

    > [!NOTE]
    > Azure Active Directory 帳戶的持有者會收到一封電子郵件，並依照連結在啟用其帳戶前進行確認。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者
您要把 Clarizen 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![已指派測試使用者][200]

1. 在 Azure 入口網站中，開啟應用程式檢視，瀏覽至目錄檢視，按一下 [企業應用程式]，然後按一下 [所有應用程式]。

    ![按一下 [企業應用程式] 和 [所有應用程式]][201]

1. 在應用程式清單中，選取 [Clarizen]。

    ![在清單中選取 Clarizen](./media/clarizen-tutorial/tutorial_clarizen_50.png)

1. 在左窗格中，按一下 [使用者和群組]。

    ![按一下 [使用者和群組]][202]

1. 按一下 [新增]  按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增] 按鈕和 [新增指派] 對話方塊][203]

1. 在 [使用者和群組] 對話方塊中，在使用者清單中選取 [Britta Simon]。

1. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，按一下 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入
使用存取面板來測試您的 Azure AD 單一登入組態。

當您在 [存取面板] 中按一下 [Clarizen] 圖格時，應該會自動登入您的 Clarizen 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](clarizen-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/clarizen-tutorial/tutorial_general_01.png
[2]: ./media/clarizen-tutorial/tutorial_general_02.png
[3]: ./media/clarizen-tutorial/tutorial_general_03.png
[4]: ./media/clarizen-tutorial/tutorial_general_04.png

[100]: ./media/clarizen-tutorial/tutorial_general_100.png

[200]: ./media/clarizen-tutorial/tutorial_general_200.png
[201]: ./media/clarizen-tutorial/tutorial_general_201.png
[202]: ./media/clarizen-tutorial/tutorial_general_202.png
[203]: ./media/clarizen-tutorial/tutorial_general_203.png