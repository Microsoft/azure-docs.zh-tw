---
title: 教學課程：Azure Active Directory 與 Meta Networks Connector 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Meta Networks Connector 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 4ae5f30d-113b-4261-b474-47ffbac08bf7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/26/2018
ms.author: jeedes
ms.openlocfilehash: a3f40624e51ef287d70bed547eba7ec9e0882b0e
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669190"
---
# <a name="tutorial-azure-active-directory-integration-with-meta-networks-connector"></a>教學課程：Azure Active Directory 與 Meta Networks Connector 整合

在本教學課程中，您將了解如何整合 Meta Networks Connector 與 Azure Active Directory (Azure AD)。

Meta Networks Connector 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Meta Networks Connector 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Meta Networks Connector (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Meta Networks Connector 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Meta Networks Connector 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Meta Networks Connector
1. 設定並測試 Azure AD 單一登入

## <a name="adding-meta-networks-connector-from-the-gallery"></a>從資源庫新增 Meta Networks Connector
若要設定 Meta Networks Connector 與 Azure AD 整合，您需要從資源庫將 Meta Networks Connector 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Meta Networks Connector，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 
    
    ![Azure Active Directory 按鈕][1]
    
1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。
    
    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。
    
    ![新增應用程式按鈕][3]
    
1. 在搜尋方塊中，輸入 **Meta Networks Connector**，從結果面板中選取 [Meta Networks Connector]，然後按一下 [新增] 按鈕以新增應用程式。
    
    ![結果清單中的 Meta Networks Connector](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Meta Networks Connector 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Meta Networks Connector 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Meta Networks Connector 中的相關使用者之間建立連結關聯性。

若要使用 Meta Networks Connector 來設定並測試 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Meta Networks Connector 測試使用者](#create-a-meta-networks-connector-test-user)** - 使 Meta Networks Connector 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Meta Networks Connector 應用程式中設定單一登入。

**若要設定與 Meta Networks Connector 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Meta Networks Connector] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Meta Networks Connector 網域及 URL] 區段上執行下列步驟：

    ![Meta Networks Connector 網域及 URL 單一登入資訊](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_url.png)

    1. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://login.nsof.io/v1/<ORGANIZATION-SHORT-NAME>/saml/metadata`
    
    1. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://login.nsof.io/v1/<ORGANIZATION-SHORT-NAME>/sso/saml`
    
1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![Meta Networks Connector 網域及 URL 單一登入資訊](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_url1.png)

    1. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<ORGANIZATION-SHORT-NAME>.metanetworks.com/login`
    
    1. 在 [轉送狀態] 文字方塊中，使用下列模式輸入 URL：`https://<ORGANIZATION-SHORT-NAME>.metanetworks.com/#/`
    
    > [!NOTE]
    > 這些都不是真正的值。 本教學課程稍後會說明如何使用實際的「識別碼」、「回覆 URL」及「登入 URL」來更新這些值。
    
1. Meta Networks Connector 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中，管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。
    
    ![設定單一登入](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_attribute.png)
    
1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱 | 屬性值 | NAMESPACE|
    | ---------------| --------------- | -------- |
    | firstname | user.givenname | |
    | lastname | user.surname | |
    | emailaddress| user.mail| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | 名稱 | user.userprincipalname| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | 電話 | user.telephonenumber | |

    1. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

        ![設定單一登入](./media/metanetworksconnector-tutorial/tutorial_attribute_04.png)
    
        ![設定單一登入](./media/metanetworksconnector-tutorial/tutorial_attribute_05.png)   
    
    1. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    1. 在 [值] 清單中，選取該列所顯示的值。
    
    1. 在 [命名空間] 文字方塊中，輸入該資料列顯示的命名空間值。
    
    1. 按一下 [確定]。
    
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。
    
    ![憑證下載連結](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_certificate.png)
    
1. 在 [Meta Networks Connector 設定] 區段上，按一下 [設定 Meta Networks Connector] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。
    
    ![設定單一登入](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_configure.png)
    
1. 按一下 [儲存]  按鈕。
    
    ![設定單一登入儲存按鈕](./media/metanetworksconnector-tutorial/tutorial_general_400.png)
    
1. 在瀏覽器中開啟新索引標籤，登入您的 Meta Networks Connector 系統管理員帳戶。
    
    > [!NOTE]
    > Meta Networks Connector 是安全的系統。 因此，在存取其入口網站之前，您必須先將公用 IP 位址列入其系統端的允許清單。 若要取得公用 IP 位址，請遵循[此處](https://whatismyipaddress.com/)指定的下列連結。 將 IP 位址傳送至 [Meta Networks Connector 用戶端支援小組](mailto:support@metanetworks.com)，以將 IP 位址列入允許清單。
    
1. 移至 [系統管理員]，然後選取 [設定]。
    
    ![設定單一登入](./media/metanetworksconnector-tutorial/configure3.png)
    
1. 確定 [記錄網際網路流量] 和 [強制執行 VPN MFA] 已設定為關閉。
    
    ![設定單一登入](./media/metanetworksconnector-tutorial/configure1.png)
    
1. 移至 [系統管理員]，然後選取 [SAML]。
    
    ![設定單一登入](./media/metanetworksconnector-tutorial/configure4.png)
    
1. 在 [詳細資料] 頁面上，執行下列步驟：
    
    ![設定單一登入](./media/metanetworksconnector-tutorial/configure2.png)
    
    1. 複製 [SSO URL] 值並將它貼至 [Meta Networks Connector 網域和 URL] 區段的 [登入 URL] 文字方塊。
    
    1. 複製 [收件者 URL] 值並將它貼至 [Meta Networks Connector 網域和 URL] 區段的 [回覆 URL] 文字方塊。
    
    1. 複製 [對象 URI (SP 實體識別碼)] 值並將它貼至 [Meta Networks Connector 網域和 URL] 區段的 [識別碼 (實體識別碼)] 文字方塊。
    
    1. 啟用 SAML
    
1. 在 [一般] 索引標籤上，執行下列步驟：

    ![設定單一登入](./media/metanetworksconnector-tutorial/configure5.png)

    1. 在 [識別提供者單一登入 URL] 中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    1. 在 [識別提供者簽發者] 中，貼上您從 Azure 入口網站複製的 **SAML 實體識別碼**值。

    1. 從記事本中的 Azure 入口網站開啟已下載的憑證，將它貼至 [X.509 憑證] 文字方塊。

    1. 啟用 [Just-In-Time 佈建]。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。
    
![建立 Azure AD 測試使用者][100]
    
**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。
    
    ![Azure Active Directory 按鈕](./media/metanetworksconnector-tutorial/create_aaduser_01.png)
    
1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![[使用者和群組] 與 [所有使用者] 連結](./media/metanetworksconnector-tutorial/create_aaduser_02.png)
    
1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。
    
    ![[新增] 按鈕](./media/metanetworksconnector-tutorial/create_aaduser_03.png)
    
1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/metanetworksconnector-tutorial/create_aaduser_04.png)
    
    1. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    1. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。
    
    1. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。
    
    1. 按一下頁面底部的 [新增] 。
    
### <a name="create-a-meta-networks-connector-test-user"></a>建立 Meta Networks Connector 測試使用者

本節目標是在 Meta Networks Connector 中建立名為 Britta Simon 的使用者。 Meta Networks Connector 支援預設啟用的 Just-in-Time 佈建。 在這一節沒有您需要進行的動作項目。 嘗試存取 Meta Networks Connector 期間會建立新使用者 (如果尚不存在)。

>[!Note]
>如果您需要手動建立使用者，請連絡 [Meta Networks Connector 用戶端支援小組](mailto:support@metanetworks.com)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Meta Networks Connector 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Meta Networks Connector，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。
    
    ![指派使用者][201]
    
1. 在應用程式清單中，選取 [Meta Networks Connector]。
    
    ![應用程式清單中的 Meta Networks Connector 連結](./media/metanetworksconnector-tutorial/tutorial_metanetworksconnector_app.png)  
    
1. 在左側功能表中，按一下 [使用者和群組]。
    
    ![[使用者和群組] 連結][202]
    
1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。
    
    ![[新增指派] 窗格][203]
    
1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。
    
1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。
    
1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Meta Networks Connector] 圖格時，應該就會自動登入 Meta Networks Connector 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

- [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
- [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/metanetworksconnector-tutorial/tutorial_general_01.png
[2]: ./media/metanetworksconnector-tutorial/tutorial_general_02.png
[3]: ./media/metanetworksconnector-tutorial/tutorial_general_03.png
[4]: ./media/metanetworksconnector-tutorial/tutorial_general_04.png

[100]: ./media/metanetworksconnector-tutorial/tutorial_general_100.png

[200]: ./media/metanetworksconnector-tutorial/tutorial_general_200.png
[201]: ./media/metanetworksconnector-tutorial/tutorial_general_201.png
[202]: ./media/metanetworksconnector-tutorial/tutorial_general_202.png
[203]: ./media/metanetworksconnector-tutorial/tutorial_general_203.png

