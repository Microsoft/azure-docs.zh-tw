---
title: 教學課程：Azure Active Directory 與 iPass SmartConnect 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 iPass SmartConnect 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: dee6d039-f9bb-49a2-a408-5ed40ef17d9f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2018
ms.author: jeedes
ms.openlocfilehash: ecfdd3fae1d394e3b57fcd325f44cad0d1a98534
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39444889"
---
# <a name="tutorial-azure-active-directory-integration-with-ipass-smartconnect"></a>教學課程：Azure Active Directory 與 iPass SmartConnect 整合

在本教學課程中，您將了解如何整合 iPass SmartConnect 與 Azure Active Directory (Azure AD)。

將 iPass SmartConnect 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 iPass SmartConnect 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 iPass SmartConnect (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 iPass SmartConnect 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 iPass SmartConnect 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 iPass SmartConnect
1. 設定並測試 Azure AD 單一登入

## <a name="adding-ipass-smartconnect-from-the-gallery"></a>從資源庫新增 iPass SmartConnect
若要設定將 iPass SmartConnect 整合到 Azure AD 中，您需要從資源庫將 iPass SmartConnect 新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 iPass SmartConnect，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **iPass SmartConnect**，從結果面板中選取 [iPass SmartConnect]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 iPass SmartConnect](./media/ipasssmartconnect-tutorial/tutorial_ipasssmartconnect_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 iPass SmartConnect 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 iPass SmartConnect 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 iPass SmartConnect 中的相關使用者之間建立連結關聯性。

若要設定及測試與 iPass SmartConnect 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 iPass SmartConnect 測試使用者](#create-an-ipass-smartconnect-test-user)** - 在 iPass SmartConnect 中建立一個與 Azure AD 中代表 Britta Simon 之使用者連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 iPass SmartConnect 應用程式中設定單一登入。

**若要設定與 iPass SmartConnect 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [iPass SmartConnect] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/ipasssmartconnect-tutorial/tutorial_ipasssmartconnect_samlbase.png)

1. 在 [iPass SmartConnect 網域及 URL] 區段上，如果您想要以 **IDP** 起始模式設定應用程式，則不需要執行任何步驟。

    ![iPass SmartConnect 網域及 URL 單一登入資訊](./media/ipasssmartconnect-tutorial/tutorial_ipasssmartconnect_url1.png)

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![iPass SmartConnect 網域及 URL 單一登入資訊](./media/ipasssmartconnect-tutorial/tutorial_ipasssmartconnect_url2.png)

    在 [登入 URL] 文字方塊中，輸入 URL：`https://om-activation.ipass.com/ClientActivation/ssolanding.go`

1. iPass SmartConnect 應用程式會預期要有特定格式的 SAML 判斷提示。 請設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。

    ![設定單一登入](./media/ipasssmartconnect-tutorial/attribute.png)

1. 在 [使用者屬性] 區段中，按一下 [檢視及編輯所有其他使用者屬性] 核取方塊，可展開屬性。 在每個顯示的屬性上執行下列步驟-

    | 屬性名稱 | 屬性值 | 命名空間值|
    | ---------------| --------------- |----------------|
    | firstName | user.givenname |   |
    | lastName | user.surname | |
    | 電子郵件 | user.userprincipalname | |
    | username | user.userprincipalname | |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/ipasssmartconnect-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/ipasssmartconnect-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 將該資料列的命名空間值保留空白。

    e. 按一下 [確定] 。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/ipasssmartconnect-tutorial/tutorial_ipasssmartconnect_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/ipasssmartconnect-tutorial/tutorial_general_400.png)

1. 若要在 **iPass SmartConnect** 端設定單一登入，您必須將已下載的**中繼資料 XML** 及您的**網域名稱**傳送給 [iPass SmartConnect 支援小組](mailto:help@ipass.com)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/ipasssmartconnect-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/ipasssmartconnect-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/ipasssmartconnect-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/ipasssmartconnect-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-an-ipass-smartconnect-test-user"></a>建立 iPass SmartConnect 測試使用者

在本節中，您會在 iPass SmartConnect 中建立名為 Britta Simon 的使用者。 請與 [iPass SmartConnect 支援小組](mailto:help@ipass.com)合作，以新增需要在 iPass SmartConnect 平台中列入白名單的使用者或網域。 如果是由小組新增網域，將會自動將使用者佈建到 iPass SmartConnect 平台。 您必須先建立和啟動使用者，然後才能使用單一登入。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 iPass SmartConnect 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 iPass SmartConnect，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

1. 在應用程式清單中，選取 [iPass SmartConnect] 。

    ![應用程式清單中的 iPass SmartConnect 連結](./media/ipasssmartconnect-tutorial/tutorial_ipasssmartconnect_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

**若要在「SP 起始」流程中測試應用程式，請執行下列步驟：**

a. 從[這裡](https://om-activation.ipass.com/ClientActivation/ssolanding.go)下載 Windows iPass SmartConnect 用戶端。

![應用程式清單中的 iPass SmartConnect 連結](./media/ipasssmartconnect-tutorial/testing3.png)

b. 安裝用戶端並啟動。

c. 按一下 [Get Started] \(開始使用\)。

![應用程式清單中的 iPass SmartConnect 連結](./media/ipasssmartconnect-tutorial/testing1.png) 

d. 輸入含網域的 Azure 使用者名稱。 按一下 [Continue] \(繼續\)。 這會重新導向到 Azure 登入頁面

![應用程式清單中的 iPass SmartConnect 連結](./media/ipasssmartconnect-tutorial/testing2.png) 

e. 成功驗證之後，就會開始用戶端啟用作業。 將會啟用用戶端。

**若要在「IdP 起始」流程中測試應用程式，請執行下列步驟：**

a. 登入 [https://myapps.microsoft.com](https://myapps.microsoft.com)。

b. 按一下 [iPass SmartConnect] 應用程式。

c. 這會啟動 SSA 頁面，按一下 [Download App for Windows] \(下載適用於 Windows 的應用程式\) 以安裝 iPass SmartConnect 用戶端。

![應用程式清單中的 iPass SmartConnect 連結](./media/ipasssmartconnect-tutorial/testing4.png)

d. 安裝之後，第一次啟動的用戶端將會在接受條款及條件後自動開始啟用。

e. 如果未開始啟用，請按一下 SSA 頁面上的 [Activate] \(啟用\) 按鈕來起始啟用作業。

f. 將會啟用用戶端。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/ipasssmartconnect-tutorial/tutorial_general_01.png
[2]: ./media/ipasssmartconnect-tutorial/tutorial_general_02.png
[3]: ./media/ipasssmartconnect-tutorial/tutorial_general_03.png
[4]: ./media/ipasssmartconnect-tutorial/tutorial_general_04.png

[100]: ./media/ipasssmartconnect-tutorial/tutorial_general_100.png

[200]: ./media/ipasssmartconnect-tutorial/tutorial_general_200.png
[201]: ./media/ipasssmartconnect-tutorial/tutorial_general_201.png
[202]: ./media/ipasssmartconnect-tutorial/tutorial_general_202.png
[203]: ./media/ipasssmartconnect-tutorial/tutorial_general_203.png

