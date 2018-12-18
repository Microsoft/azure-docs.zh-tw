---
title: 教學課程：Azure Active Directory 與 Springer Link 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Springer Link 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 58cdf029-bdc0-43c4-a469-b921c2a669bd
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/16/2017
ms.author: jeedes
ms.openlocfilehash: 8138c7605b0024dfe4569e33843cb1e9d169271f
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39438761"
---
# <a name="tutorial-azure-active-directory-integration-with-springer-link"></a>教學課程：Azure Active Directory 與 Springer Link 整合

在本教學課程中，您將了解如何整合 Springer Link 與 Azure Active Directory (Azure AD)。

Springer Link 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Springer Link 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Springer Link (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Springer Link 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Springer Link 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Springer Link
1. 設定並測試 Azure AD 單一登入

## <a name="adding-springer-link-from-the-gallery"></a>從資源庫新增 Springer Link
若要設定 Springer Link 與 Azure AD 整合，您需要從資源庫將 Springer Link 新增至受控 SaaS 應用程式清單。

**若要從資源庫新增 Springer Link，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，鍵入 **Springer Link**，從結果面板中選取 [Springer Link]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Springer Link](./media/springerlink-tutorial/tutorial_springerlink_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Springer Link 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Springer Link 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Springer Link 中的相關使用者之間建立連結關聯性。

在 Springer Link 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要使用 Springer Link 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Springer Link 應用程式中設定單一登入。

**若要使用 Springer Link 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Springer Link] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/springerlink-tutorial/tutorial_springerlink_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Springer Link 網域與 URL] 區段上執行下列步驟：

    ![Springer Link 網域和 URL 單一登入資訊](./media/springerlink-tutorial/tutorial_springerlink_url1.png)

    a. 在 [識別碼] 文字方塊中，輸入 URL：`https://fsso.springer.com`

    b. 在 [回覆 URL] 文字方塊中，輸入 URL：`https://fsso-qa1.springer.com/federation/Consumer/metaAlias/SpringerServiceProvider`    

1. 按一下 [顯示進階 URL 設定]。 如果您想要以 **SP** 起始模式設定應用程式：

    ![Springer Link 網域和 URL 單一登入資訊](./media/springerlink-tutorial/tutorial_springerlink_url.png)

    在 [登入 URL] 文字方塊中，輸入 URL：`https://fsso.springer.com/federation/Consumer/metaAlias/SpringerServiceProvider`

1. 在 [SAML 簽署憑證] 區段中，按一下「複製」按鈕複製「應用程式同盟中繼資料 URL」，並將它貼到 [記事本]。 

    ![憑證下載連結](./media/springerlink-tutorial/tutorial_springerlink_certificate.png)    

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/springerlink-tutorial/tutorial_general_400.png)

1. 若要在 **Springer Link** 端設定單一登入，您必須將**應用程式同盟中繼資料 Url** 傳送給 [Springer Link 支援小組](mailto:identity@springernature.com)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/springerlink-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/springerlink-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/springerlink-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/springerlink-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Springer Link 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Springer Link，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Springer Link]。

    ![應用程式清單中的 Springer Link 連結](./media/springerlink-tutorial/tutorial_springerlink_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Springer Link] 磚時，應該會自動登入您的 Springer Link 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/springerlink-tutorial/tutorial_general_01.png
[2]: ./media/springerlink-tutorial/tutorial_general_02.png
[3]: ./media/springerlink-tutorial/tutorial_general_03.png
[4]: ./media/springerlink-tutorial/tutorial_general_04.png

[100]: ./media/springerlink-tutorial/tutorial_general_100.png

[200]: ./media/springerlink-tutorial/tutorial_general_200.png
[201]: ./media/springerlink-tutorial/tutorial_general_201.png
[202]: ./media/springerlink-tutorial/tutorial_general_202.png
[203]: ./media/springerlink-tutorial/tutorial_general_203.png

