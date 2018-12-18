---
title: 教學課程：Azure Active Directory 與 direct 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 direct 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 7c2cd1f0-d14c-42f0-94a8-9b800008b285
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/06/2018
ms.author: jeedes
ms.openlocfilehash: 7e693a721e5556970607fafd8ff187d3b06c913e
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44092881"
---
# <a name="tutorial-azure-active-directory-integration-with-direct"></a>教學課程：Azure Active Directory 與 direct 整合

在本教學課程中，您會了解如何整合 direct 與 Azure Active Directory (Azure AD)。

direct 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 direct 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 direct (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 direct 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 direct 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 direct
2. 設定並測試 Azure AD 單一登入

## <a name="adding-direct-from-the-gallery"></a>從資源庫新增 direct

若要設定將 direct 整合到 Azure AD 中，您需要從資源庫將 direct 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 direct，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

4. 在搜尋方塊中，輸入 **direct**。 從結果面板中選取 [direct]，然後選取 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/direct-tutorial/tutorial_direct_addfromgallery.png)

## <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 direct 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 direct 與 Azure AD 中互相對應的使用者。 換句話說，必須要建立某位 Azure AD 使用者與 direct 中相關使用者之間的連結關聯性。

在 direct 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 direct 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 direct 測試使用者](#creating-a-direct-test-user)** - 在 direct 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
4. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 direct 應用程式中設定單一登入。

**若要使用 direct 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [direct] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/direct-tutorial/tutorial_direct_samlbase.png)

3. 如果您想要以「IDP 起始模式」設定應用程式，請在 [direct 網域和 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/direct-tutorial/tutorial_direct_url.png)

    在 [識別碼] 文字方塊中，輸入 URL：`https://direct4b.com/`

4. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]：

    ![設定單一登入](./media/direct-tutorial/tutorial_direct_url1.png)

     在 [登入 URL] 文字方塊中，輸入 URL：`https://direct4b.com/sso` 

5. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/direct-tutorial/tutorial_direct_certificate.png) 

6. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/direct-tutorial/tutorial_general_400.png)

7. 若要在 **direct** 端設定單一登入，您必須將已下載的**中繼資料 XML** 傳送給 [direct 支援小組](https://direct4b.com/ja/support.html#inquiry)。

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/direct-tutorial/create_aaduser_01.png) 

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/direct-tutorial/create_aaduser_02.png) 

3. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。

    ![建立 Azure AD 測試使用者](./media/direct-tutorial/create_aaduser_03.png) 

4. 在 [使用者]  對話頁面上，執行下列步驟：

    ![建立 Azure AD 測試使用者](./media/direct-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="creating-a-direct-test-user"></a>建立 direct 測試使用者

在本節中，您要在 direct 中建立名為 Britta Simon 的使用者。 請與 [direct 支援小組](https://direct4b.com/ja/support.html#inquiry)合作，在 direct 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 direct 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 direct，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [direct]。

    ![設定單一登入](./media/direct-tutorial/tutorial_direct_app.png) 

3. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

1. 如果您想要在 **IDP 起始模式**中進行測試：

    當您在存取面板中按一下 [direct] 圖格時，應該會自動登入您的 **direct** 應用程式。

2. 如果您想要在 **SP 起始模式**中進行測試：

    a. 按一下存取面板中的 [direct] 圖格，系統就會將您重新導向至應用程式登入頁面。

    b. 在顯示的文字方塊中輸入您的 `subdomain`，然後按 '次へ (下一步)'，您應該就會自動登入您的 **direct** 應用程式。

如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/direct-tutorial/tutorial_general_01.png
[2]: ./media/direct-tutorial/tutorial_general_02.png
[3]: ./media/direct-tutorial/tutorial_general_03.png
[4]: ./media/direct-tutorial/tutorial_general_04.png

[100]: ./media/direct-tutorial/tutorial_general_100.png

[200]: ./media/direct-tutorial/tutorial_general_200.png
[201]: ./media/direct-tutorial/tutorial_general_201.png
[202]: ./media/direct-tutorial/tutorial_general_202.png
[203]: ./media/direct-tutorial/tutorial_general_203.png