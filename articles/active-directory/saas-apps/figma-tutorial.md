---
title: 教學課程：Azure Active Directory 與 Figma 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Figma 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 8569cae1-87dd-4c40-9bbb-527ac80d6a96
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/30/2018
ms.author: jeedes
ms.openlocfilehash: 4094de1a1c17e844d96ac789bb4bc1655fdc1546
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669238"
---
# <a name="tutorial-azure-active-directory-integration-with-figma"></a>教學課程：Azure Active Directory 與 Figma 整合

在本教學課程中，您會了解如何整合 Figma 與 Azure Active Directory (Azure AD)。

Figma 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Figma 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Figma (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Figma 整合，您需要下列項目：

- Azure AD 訂用帳戶
- [已啟用單一登入的 Figma 訂用帳戶](https://www.figma.com/pricing/)

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。 Figma 專業團隊的新客戶和現行訂閱者可以連絡 Figma 來[將其訂用帳戶升級](https://www.figma.com/pricing/)為 Figma 組織層。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Figma
2. 設定並測試 Azure AD 單一登入

## <a name="adding-figma-from-the-gallery"></a>從資源庫新增 Figma

若要設定將 Figma 整合到 Azure AD 中，您需要從資源庫將 Figma 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Figma，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Figma**，從結果面板中選取 [Figma]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Figma](./media/figma-tutorial/tutorial_figma_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Figma 搭配運作的 Azure AD 單一登入。

為了讓單一登入能運作，Azure AD 必須連結到 Figma。  若要使用 Figma 來設定並測試 Azure AD 單一登入，您需要完成下列步驟：

1. [**請連絡 Figma 支援小組**](mailto:support@figma.com?subject=SAML+Config)來啟動您組織的 SAML 設定，並取得 ORG_SAML_CONFIG_ID。
2. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
3. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
4. **[建立 Figma 測試使用者](#create-a-figma-test-user)** - 在 Figma 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
5. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
6. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Figma 應用程式中設定單一登入。

**若要使用 Figma 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Figma] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/figma-tutorial/tutorial_figma_samlbase.png)

3. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Figma 網域和 URL] 區段上執行下列步驟：

    ![Figma 網域與 URL 單一登入資訊](./media/figma-tutorial/tutorial_figma_url1.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://www.figma.com/saml/<ORG_SAML_CONFIG_ID>`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://www.figma.com/saml/<ORG_SAML_CONFIG_ID>/consume`

4. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![Figma 網域與 URL 單一登入資訊](./media/figma-tutorial/tutorial_figma_url2.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://www.figma.com/saml/<ORG_SAML_CONFIG_ID>/start`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 請連絡 [Figma 支援小組](mailto:support@figma.com?subject=SAML+Config)以取得這些值。

5. 在 [SAML 簽署憑證] 區段中，按一下「複製」按鈕複製「應用程式同盟中繼資料 URL」，並將它貼到 [記事本]。

    ![憑證下載連結](./media/figma-tutorial/tutorial_figma_certificate.png)

6. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/figma-tutorial/tutorial_general_400.png)

7. 若要設定 Figma 端的單一登入，請填寫此表單：[https://goo.gl/forms/XkRB1z5ed4eVUzXn2](https://goo.gl/forms/XkRB1z5ed4eVUzXn2)。 它會接受您步驟 #5 中的**應用程式同盟中繼資料 URL**。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/figma-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/figma-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/figma-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/figma-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-figma-test-user"></a>建立 Figma 測試使用者

本節目標是在 Figma 中建立名為 Britta Simon 的使用者。 Figma 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 當您嘗試存取 Figma 時，如果 Figma 還沒有使用者，它就會建立新的使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Figma 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Figma，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Figma]。

    ![應用程式清單中的 Figma 連結](./media/figma-tutorial/tutorial_figma_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Figma 磚時，應該會自動登入您的 Figma 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/figma-tutorial/tutorial_general_01.png
[2]: ./media/figma-tutorial/tutorial_general_02.png
[3]: ./media/figma-tutorial/tutorial_general_03.png
[4]: ./media/figma-tutorial/tutorial_general_04.png

[100]: ./media/figma-tutorial/tutorial_general_100.png

[200]: ./media/figma-tutorial/tutorial_general_200.png
[201]: ./media/figma-tutorial/tutorial_general_201.png
[202]: ./media/figma-tutorial/tutorial_general_202.png
[203]: ./media/figma-tutorial/tutorial_general_203.png