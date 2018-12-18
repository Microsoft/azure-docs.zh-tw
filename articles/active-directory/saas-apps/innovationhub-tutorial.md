---
title: 教學課程：Azure Active Directory 與 Innovation Hub 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Innovation Hub 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: d72e4da0-0123-409b-96c2-e613f3f83fb1
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2018
ms.author: jeedes
ms.openlocfilehash: 6486c43ed9eaf1e829598cfc9177a96e0bed9fe1
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39437544"
---
# <a name="tutorial-azure-active-directory-integration-with-innovation-hub"></a>教學課程：Azure Active Directory 與 Innovation Hub 整合

在本教學課程中，您將了解如何整合 Innovation Hub 與 Azure Active Directory (Azure AD)。

Innovation Hub 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Innovation Hub 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Innovation Hub (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Innovation Hub 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Innovation Hub 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Innovation Hub
1. 設定並測試 Azure AD 單一登入

## <a name="adding-innovation-hub-from-the-gallery"></a>從資源庫新增 Innovation Hub
若要設定 Innovation Hub 與 Azure AD 整合，您需要從資源庫將 Innovation Hub 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Innovation Hub，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Innovation Hub**，從結果面板中選取 [Innovation Hub]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Innovation Hub](./media/innovationhub-tutorial/tutorial_innovationhub_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Innovation Hub 設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Innovation Hub 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Innovation Hub 中的相關使用者之間建立連結關聯性。

若要使用 Innovation Hub 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Innovation Hub 測試使用者](#create-an-innovation-hub-test-user)** - 在 Innovation Hub 中建立一個與 Azure AD 中代表 Britta Simon 使用者連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Innovation Hub 應用程式中設定單一登入。

**若要設定與 Innovation Hub 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Innovation Hub] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/innovationhub-tutorial/tutorial_innovationhub_samlbase.png)

1. 在 [Innovation Hub 網域與 URL] 區段中，執行下列步驟：

    ![Innovation Hub 網域與 URL 單一登入資訊](./media/innovationhub-tutorial/tutorial_innovationhub_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<domainname>.innohb.com/auth/saml2/login`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<domainname>.innohb.com`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Innovation Hub 用戶端支援小組](mailto:support@readify.net)以取得這些值。

1. Innovation Hub 應用程式需要特定格式的 SAML 判斷提示。 請設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。

    ![設定單一登入](./media/innovationhub-tutorial/attribute.png)

1. 在 [使用者屬性] 區段中，按一下 [檢視及編輯所有其他使用者屬性] 核取方塊，可展開屬性。 在每個顯示的屬性上執行下列步驟-

    | 屬性名稱 | 屬性值 | 命名空間值|
    | ---------------| --------------- |----------------|
    | displayname | user.userprincipalname | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | | |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/innovationhub-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/innovationhub-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 從 [命名空間值] 清單中，輸入針對該資料列顯示的命名空間值。

    e. 按一下 [確定] 。

1. 在 [SAML 簽署憑證] 區段中，按一下「複製」按鈕複製「應用程式同盟中繼資料 URL」，並將它貼到 [記事本]。

    ![憑證下載連結](./media/innovationhub-tutorial/tutorial_innovationhub_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/innovationhub-tutorial/tutorial_general_400.png)

1. 若要在 **Innovation Hub** 端設定單一登入，您必須將複製起來的**同盟中繼資料 URL** 傳送給 [Innovation Hub 支援小組](mailto:support@readify.net)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/innovationhub-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/innovationhub-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/innovationhub-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/innovationhub-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-an-innovation-hub-test-user"></a>建立 Innovation Hub 測試使用者

本節的目標是在 Innovation Hub 中建立名為 Britta Simon 的使用者。 Innovation Hub 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 當您嘗試存取 Innovation Hub 時，如果 Innovation Hub 還沒有使用者，它將會建立新的使用者。
>[!Note]
>如果您需要手動建立使用者，請連絡 [Innovation Hub 支援小組](mailto:support@readify.net)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Innovation Hub 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派到 Innovation Hub，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Innovation Hub]。

    ![應用程式清單中的 Innovation Hub 連結](./media/innovationhub-tutorial/tutorial_innovationhub_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Innovation Hub] 磚時，應該會自動登入您的 Innovation Hub 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
<!--Image references-->

[1]: ./media/innovationhub-tutorial/tutorial_general_01.png
[2]: ./media/innovationhub-tutorial/tutorial_general_02.png
[3]: ./media/innovationhub-tutorial/tutorial_general_03.png
[4]: ./media/innovationhub-tutorial/tutorial_general_04.png

[100]: ./media/innovationhub-tutorial/tutorial_general_100.png

[200]: ./media/innovationhub-tutorial/tutorial_general_200.png
[201]: ./media/innovationhub-tutorial/tutorial_general_201.png
[202]: ./media/innovationhub-tutorial/tutorial_general_202.png
[203]: ./media/innovationhub-tutorial/tutorial_general_203.png

