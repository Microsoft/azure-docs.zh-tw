---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 Burp Suite Enterprise Edition 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Burp Suite Enterprise Edition 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/09/2020
ms.author: jeedes
ms.openlocfilehash: 480e501f8683fad405abc5e07cb43eec8ae8444c
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98727849"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-burp-suite-enterprise-edition"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 Burp Suite Enterprise Edition 整合

在本教學課程中，您將了解如何整合 Burp Suite Enterprise Edition 與 Azure Active Directory (Azure AD)。 在整合 Burp Suite Enterprise Edition 與 Azure AD 時，您可以：

* 在 Azure AD 中控制可存取 Burp Suite Enterprise Edition 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Burp Suite Enterprise Edition。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Burp Suite Enterprise Edition 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。


* Burp Suite Enterprise Edition 支援 **IDP** 起始的 SSO

* Burp Suite Enterprise Edition 支援 **Just In Time** 使用者佈建


## <a name="adding-burp-suite-enterprise-edition-from-the-gallery"></a>從資源庫新增 Burp Suite Enterprise Edition

若要設定將 Burp Suite Enterprise Edition 整合到 Azure AD 中，您需要從資源庫將 Burp Suite Enterprise Edition 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增] 區段的搜尋方塊中，輸入 **Burp Suite Enterprise Edition**。
1. 從結果面板中選取 [Burp Suite Enterprise Edition]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。


## <a name="configure-and-test-azure-ad-sso-for-burp-suite-enterprise-edition"></a>設定和測試適用於 Burp Suite Enterprise Edition 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 Burp Suite Enterprise Edition 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Burp Suite Enterprise Edition 中相關使用者之間的連結關聯性。

若要設定及測試與 Burp Suite Enterprise Edition 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Burp Suite Enterprise Edition SSO](#configure-burp-suite-enterprise-edition-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 Burp Suite Enterprise Edition 測試使用者](#create-burp-suite-enterprise-edition-test-user)** - 使 Burp Suite Enterprise Edition 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 **Burp Suite Enterprise Edition** 應用程式整合頁面上，尋找 **管理** 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [以 SAML 設定單一登入]  頁面上，輸入下列欄位的值：

    a. 在 [識別碼]  文字方塊中，使用下列模式來輸入 URL：`https://<BURPSUITEDOMAIN:PORT>/saml`

    b. 在 [回覆 URL]  文字方塊中，使用下列模式來輸入 URL：`https://<BURPSUITEDOMAIN:PORT>/api-internal/saml/acs`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 更新這些值。 請連絡 [Burp Suite Enterprise Edition 用戶端支援小組](mailto:support@portswigger.net)以取得這些值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

1. Burp Suite Enterprise Edition 應用程式需要特定格式的 SAML 判斷提示，要求您加入自訂屬性對應到您的 SAML 權杖屬性組態。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/edit-attribute.png)

1. 除了上述屬性外，Burp Suite Enterprise Edition 應用程式還需要在 SAML 回應中多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。


    | 名稱 | 來源屬性|
    | ---------------| --------------- |    
    | 群組 | user.groups |

1. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，尋找 [憑證 (Base64)]  並選取 [下載]  ，以下載憑證並將其儲存在電腦上。

    ![憑證下載連結](common/certificatebase64.png)

1. 在 [設定 Burp Suite Enterprise Edition] 區段上，根據您的需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下頁面底部的 [新增]  。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Burp Suite Enterprise Edition 的存取權授與 B.Simon，讓她能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Burp Suite Enterprise Edition]。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-burp-suite-enterprise-edition-sso"></a>設定 Burp Suite Enterprise Edition SSO

若要設定 **Burp Suite Enterprise Edition** 端的單一登入，您必須將從 Azure 入口網站下載的 [憑證 (Base64)] 和複製的適當 URL 傳送給 [Burp Suite Enterprise Edition 支援小組](mailto:support@portswigger.net)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

### <a name="create-burp-suite-enterprise-edition-test-user"></a>建立 Burp Suite Enterprise Edition 測試使用者

本節會在 Burp Suite Enterprise Edition 中建立名為 Britta Simon 的使用者。 Burp Suite Enterprise Edition 支援依預設啟用的 Just-In-Time 使用者佈建。 在這一節沒有您需要進行的動作項目。 如果 Burp Suite Enterprise Edition 中還沒有任何使用者存在，在驗證之後就會建立新的使用者。

## <a name="test-sso"></a>測試 SSO 

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

* 在 Azure 入口網站中按一下 [測試此應用程式]，應該會自動登入您已設定 SSO 的 Burp Suite Enterprise Edition

* 您可以使用 Microsoft 的「我的應用程式」。 在我的應用程式中按一下 [Burp Suite Enterprise Edition]，應該會自動登入您已設定 SSO 的 Burp Suite Enterprise Edition。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。


## <a name="next-steps"></a>後續步驟

設定 Burp Suite Enterprise Edition 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。