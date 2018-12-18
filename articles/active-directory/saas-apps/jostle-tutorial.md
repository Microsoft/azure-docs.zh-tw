---
title: 教學課程：Azure Active Directory 與 Jostle 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Jostle 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 9ca4ca1f-8f68-4225-81a6-1666b486d6a8
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/30/2017
ms.author: jeedes
ms.openlocfilehash: b988d908b995b1144837c8642a8864a87e2fc61a
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39437513"
---
# <a name="tutorial-azure-active-directory-integration-with-jostle"></a>教學課程：Azure Active Directory 與 Jostle 整合

在本教學課程中，您會了解如何整合 Jostle 與 Azure Active Directory (Azure AD)。

Jostle 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Jostle 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Jostle (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Jostle 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Jostle 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Jostle
1. 設定並測試 Azure AD 單一登入

## <a name="adding-jostle-from-the-gallery"></a>從資源庫新增 Jostle
若要設定將 Jostle 整合到 Azure AD 中，您需要從資源庫將 Jostle 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Jostle，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]

1. 在視窗頂端，按一下 [新增]。

    ![add_01](./media/jostle-tutorial/add_01.png)

1. 在 [新增應用程式] 下方的搜尋方塊中，鍵入 **Jostle**。

    ![add_02](./media/jostle-tutorial/add_02.png)

1. 在結果窗格中，選取 [Jostle]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/jostle-tutorial/tutorial_jostle_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Jostle 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Jostle 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Jostle 中相關使用者之間的連結關聯性。

在 Jostle 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Jostle 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Jostle 測試使用者](#creating-a-jostle-test-user)** - 在 Jostle 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表 Britta Simon 的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Jostle 應用程式中設定單一登入。

**若要設定與 Jostle 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Jostle] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![設定單一登入](./media/jostle-tutorial/tutorial_jostle_samlbase.png)

1. 在 [Jostle 網域與 URL] 區段中，執行下列步驟：

    ![url_01](./media/jostle-tutorial/url_01.png)

    a. 在 [登入 URL] 文字方塊中，輸入：`https://login-prod.jostle.us`

    b. 在 [識別碼] 文字方塊中，輸入：`https://jostle.us`

    c. 按一下 [顯示進階 URL 設定] 旁的方塊

    d. 在 [回覆 URL] 文字方塊中，輸入：`https://login-prod.jostle.us/saml/SSO/alias/newjostle.us`

1. 在 [使用者屬性] 區段的 [使用者識別碼] 欄位中，輸入：`user.userprincipalname`

    ![url_02](./media/jostle-tutorial/url_02.png)

1. 按一下視窗頂端的 [儲存]。

1. 移至 [SAML 簽署憑證]，並確認它設定為 [作用中]。 然後按一下 [中繼資料 XML] 下載中繼資料檔案。

    ![url_03](./media/jostle-tutorial/url_03.png)

1. 若要在 Jostle 端設定單一登入，您必須將已下載的中繼資料 XML 傳送給 [Jostle 支援小組](mailto:support@jostle.me)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
>

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/jostle-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![建立 Azure AD 測試使用者](./media/jostle-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。

    ![建立 Azure AD 測試使用者](./media/jostle-tutorial/create_aaduser_03.png)

1. 在 [使用者]  對話頁面上，執行下列步驟：

    ![建立 Azure AD 測試使用者](./media/jostle-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="creating-a-jostle-test-user"></a>建立 Jostle 測試使用者

在本節中，您要在 Jostle 中建立名為 Britta Simon 的使用者。 如果您不知道如何在 Jostle 新增 Britta Simon，請連絡 [Jostle 支援小組](mailto:support@jostle.me)，以新增測試使用者並啟用 SSO。

> [!NOTE]
> Azure Active Directory 帳戶的持有者會收到一封電子郵件，並依照連結在啟用其帳戶前進行確認。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Jostle 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200]

**若要將 Britta Simon 指派給 Jostle，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

1. 在應用程式清單中，選取 [Jostle] 。

    ![設定單一登入](./media/jostle-tutorial/tutorial_jostle_app.png)

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Jostle] 圖格時，您應該會取得 Jostle 應用程式的自動登入頁面。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/jostle-tutorial/tutorial_general_01.png
[2]: ./media/jostle-tutorial/tutorial_general_02.png
[3]: ./media/jostle-tutorial/tutorial_general_03.png
[4]: ./media/jostle-tutorial/tutorial_general_04.png

[100]: ./media/jostle-tutorial/tutorial_general_100.png

[200]: ./media/jostle-tutorial/tutorial_general_200.png
[201]: ./media/jostle-tutorial/tutorial_general_201.png
[202]: ./media/jostle-tutorial/tutorial_general_202.png
[203]: ./media/jostle-tutorial/tutorial_general_203.png
