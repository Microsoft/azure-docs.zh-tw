---
title: 教學課程：Azure Active Directory 與 Asana 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Asana 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 837e38fe-8f55-475c-87f4-6394dc1fee2b
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: jeedes
ms.openlocfilehash: f5ea7a330891d4befeb6388bbe7f37b2a4aa848f
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39438203"
---
# <a name="tutorial-azure-active-directory-integration-with-asana"></a>教學課程：Azure Active Directory 與 Asana 整合

在本教學課程中，您會了解如何整合 Asana 與 Azure Active Directory (Azure AD)。

將 Asana 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Asana 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Asana (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Asana 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Asana 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Asana
1. 設定並測試 Azure AD 單一登入

## <a name="adding-asana-from-the-gallery"></a>從資源庫新增 Asana
若要設定將 Asana 整合到 Azure AD 中，您需要從資源庫將 Asana 新增到受控 SaaS 應用程式清單。

**若要從資源庫加入 Asana，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Asana**，從結果面板中選取 [Asana]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/asana-tutorial/tutorial_asana_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Asana 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Asana 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Asana 中的相關使用者之間建立連結關聯性。

在 Asana 中，將 [Username] 的值指派為 Azure AD 中 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 Asana 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Asana 測試使用者](#create-an-asana-test-user)** - 在 Asana 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Asana 應用程式中設定單一登入。

**若要設定與 Asana 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Asana] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/asana-tutorial/tutorial_asana_samlbase.png)

1. 在 [Asana 網域及 URL] 區段上，執行下列步驟：

    ![Asana 網域與 URL 單一登入資訊](./media/asana-tutorial/tutorial_asana_url.png)

    a. 在 [登入 URL] 文字方塊中，輸入 URL：`https://app.asana.com/`

    b. 在 [識別碼] 文字方塊中，輸入值：`https://app.asana.com/`

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/asana-tutorial/tutorial_asana_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/asana-tutorial/tutorial_general_400.png)

1. 在 [Asana 組態] 區段上，按一下 [設定 Asana] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![Asana 設定](./media/asana-tutorial/tutorial_asana_configure.png)

1. 在不同的瀏覽器視窗中，登入您的 Asana 應用程式。 若要在 Asana 中設定 SSO，請按一下螢幕右上角的工作區名稱來存取工作區設定。 然後，按一下 [\<您的工作區名稱\> 設定]。

    ![Asana sso 設定](./media/asana-tutorial/tutorial_asana_09.png)

1. 在 [組織設定] 視窗中，按一下 [管理]。 然後按一下 [成員必須透過 SAML 登入]  以啟用 SSO 組態。 執行下列步驟：

    ![設定單一登入組織設定](./media/asana-tutorial/tutorial_asana_10.png)  

     a. 在 [登入頁面 URL] 文字方塊中，貼上「SAML 單一登入服務 URL」。

     b. 在從 Azure 入口網站下載的憑證上按一下滑鼠右鍵，然後使用「記事本」或您慣用的文字編輯器來開啟該憑證檔案。 複製開始和結束憑證標題之間的內容，然後將它貼到 [X.509 憑證] 文字方塊中。

1. 按一下 [檔案] 。 如需進一步協助，請移至 [用於設定 SSO 的 Asana 指南](https://asana.com/guide/help/premium/authentication#gl-saml) 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕](./media/asana-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/asana-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。

    ![建立 Azure AD 測試使用者](./media/asana-tutorial/create_aaduser_03.png)

1. 在 [使用者]  對話頁面上，執行下列步驟：

    ![[新增] 按鈕](./media/asana-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-an-asana-test-user"></a>建立 Asana 測試使用者

本節的目標是在 Asana 中建立名為 Britta Simon 的使用者。 Asana 支援自動使用者佈建，該功能預設為啟用。 在[這裡](asana-provisioning-tutorial.md)可以找到更多關於如何設定自動使用者佈建的詳細資料。

**如果您需要手動建立使用者，請執行下列步驟：**

在本節中，您會在 Asana 中建立名為 Britta Simon 的使用者。

1. 在 [Asana] 移至左面板上的 [小組] 區段。 按一下加號按鈕。

    ![建立 Azure AD 測試使用者](./media/asana-tutorial/tutorial_asana_12.png)

1. 在文字方塊中輸入電子郵件 britta.simon@contoso.com，然後選取 [邀請]。

1. 按一下 [傳送邀請] 。 新的使用者會在她的電子郵件帳戶收到一封電子郵件。 她將需要建立並驗證帳戶。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Asana 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Asana，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

1. 在應用程式清單中，選取 [Asana]。

    ![應用程式清單中的 Asana 連結](./media/asana-tutorial/tutorial_asana_app.png)

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

本節的目標是要測試您的 Azure AD 單一登入。

移至 Asana 登入頁面。 在 [電子郵件地址] 文字方塊中，插入電子郵件地址 britta.simon@contoso.com。 讓 [密碼] 文字方塊保持空白，然後按一下 [登入]。 系統會將您重新導向至 Azure AD 登入頁面。 完成您的 Azure AD 認證。 現在，您已在 Asana 上登入。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](asana-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/asana-tutorial/tutorial_general_01.png
[2]: ./media/asana-tutorial/tutorial_general_02.png
[3]: ./media/asana-tutorial/tutorial_general_03.png
[4]: ./media/asana-tutorial/tutorial_general_04.png

[100]: ./media/asana-tutorial/tutorial_general_100.png

[200]: ./media/asana-tutorial/tutorial_general_200.png
[201]: ./media/asana-tutorial/tutorial_general_201.png
[202]: ./media/asana-tutorial/tutorial_general_202.png
[203]: ./media/asana-tutorial/tutorial_general_203.png
[10]: ./media/asana-tutorial/tutorial_general_060.png
[11]: ./media/asana-tutorial/tutorial_general_070.png