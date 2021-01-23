---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 Slack 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Slack 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/28/2020
ms.author: jeedes
ms.openlocfilehash: 6e2428967b8e3b4c677752955ea743c5b7d144e5
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98729604"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-slack"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 Slack 整合

在本教學課程中，您將了解如何整合 Slack 與 Azure Active Directory (Azure AD)。 在整合 Slack 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 Slack。
* 讓使用者使用其 Azure AD 帳戶自動登入 Slack。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Slack 單一登入 (SSO) 的訂用帳戶。

> [!NOTE]
> 如果您需要在一個租用戶中整合多個 Slack 執行個體，每個應用程式的識別碼都可以是變數。

> [!NOTE]
> 也可以在 Azure AD 美國政府雲端環境中使用此整合。 您可以在 Azure AD 美國政府雲端應用程式庫中找到此應用程式，並以與公用雲端相同的方式進行設定。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* Slack 支援 **SP** 起始的 SSO
* Slack 支援 **Just In Time** 使用者佈建
* Slack 支援 [**自動** 使用者佈建](./slack-provisioning-tutorial.md)

> [!NOTE]
> 此應用程式的識別碼是固定的字串值，因此一個租用戶中只能設定一個執行個體。

## <a name="adding-slack-from-the-gallery"></a>從資源庫新增 Slack

若要設定將 Slack 整合到 Azure AD 中，您需要從資源庫將 Slack 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]。
1. 在 [從資源庫新增] 區段的搜尋方塊中輸入 **Slack**。
1. 從結果面板選取 [Slack]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-slack"></a>設定和測試 Slack 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 Slack 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Slack 中相關使用者之間的連結關聯性。

若要設定及測試與 Slack 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Slack SSO](#configure-slack-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 Slack 測試使用者](#create-slack-test-user)** - 使 Slack 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [Slack] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，輸入下列欄位的值：

    a. 在 [登入 URL]  文字方塊中，使用下列模式輸入 URL：`https://<DOMAIN NAME>.slack.com/sso/saml/start`

    b. 在 [識別碼 (實體識別碼)] 文字方塊中，輸入 URL：`https://slack.com`
    
    c. 針對 [回覆 URL]，輸入下列其中一個 URL 模式：
    
    | 回覆 URL|
    |----------|
    | `https://<DOMAIN NAME>.slack.com/sso/saml` |
    | `https://<DOMAIN NAME>.enterprise.slack.com/sso/saml` |

    > [!NOTE]
    > 這些都不是真正的值。 您必須使用實際的「登入 URL」及「回覆 URL」來更新這些值。 請連絡 [Slack 用戶端支援小組](https://slack.com/help/contact)以取得此值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

    > [!NOTE]
    > 如果您有多個 Slack 執行個體需要與租用戶整合，**識別碼 (實體識別碼)** 可以是變數。 使用模式 `https://<DOMAIN NAME>.slack.com`。 在此案例中，您也必須使用相同的值，與 Slack 中的另一個設定配對。

1. Slack 應用程式需要特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增至您的 SAML 權杖屬性組態。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/edit-attribute.png)

1. 除了上述屬性外，Slack 應用程式還需要在 SAML 回應中多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。 您也必須新增 `email` 屬性。 如果使用者沒有電子郵件地址，請將 **emailaddress** 對應至 **user.userprincipalname**，以及將 **email** 對應至 **user.userprincipalname**。

    | 名稱 | 來源屬性 |
    | -----|---------|
    | emailaddress | user.userprincipalname |
    | 電子郵件 | user.userprincipalname |

   > [!NOTE]
   > 若要設定服務提供者 (SP) 組態，您必須按一下 SAML 設定頁面中 [進階選項] 旁的 [展開]。 在 [服務提供者簽發者] 方塊中，輸入工作區 URL。 預設值為 slack.com。 

1. 在 [以 SAML 設定單一登入] 頁面的 [SAML 簽署憑證] 區段中，尋找 [憑證 (Base64)] 並選取 [下載]，以下載憑證並將其儲存在電腦上。

    ![憑證下載連結](common/certificatebase64.png)

1. 在 [設定 Slack] 區段上，根據您的需求複製適當的 URL。

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

在本節中，您會將 Slack 的存取權授與 B.Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]，然後選取 [所有應用程式]。
1. 在應用程式清單中，選取 [Slack]。
1. 在應用程式的概觀頁面中尋找 [管理] 區段，然後選取 [使用者和群組]。

1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派] 對話方塊中，按一下 [指派] 按鈕。

## <a name="configure-slack-sso"></a>設定 Slack SSO

1. 若要自動執行 Slack 內的設定，您必須按一下 [安裝擴充功能] 來安裝「我的應用程式安全登入瀏覽器擴充功能」。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 Slack] 便會將您導向至 Slack 應用程式。 請從該處提供用以登入 Slack 的管理員認證。 瀏覽器擴充功能會自動為您設定應用程式，並自動執行步驟 3 到 6。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 Slack，請在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Slack 公司網站。

2. 瀏覽至 [Microsoft Azure AD]，然後移至 [小組設定]。

     ![在 Microsoft Azure AD 上設定單一登入](./media/slack-tutorial/tutorial-slack-team-settings.png)

3. 在 [小組設定] 區段中，按一下 [驗證] 索引標籤，然後按一下 [變更設定]。

    ![在小組設定上設定單一登入](./media/slack-tutorial/tutorial-slack-authentication.png)

4. 在 [SAML 驗證設定]  對話方塊上，執行下列步驟：

    ![在 SAML 驗證設定上設定單一登入](./media/slack-tutorial/tutorial-slack-save-authentication.png)

    a.  在 [SAML 2.0 端點 (HTTP)] 文字方塊中，貼上您從 Azure 入口網站複製的 [登入 URL] 值。

    b.  在 [識別提供者簽發者] 文字方塊中，貼上您從 Azure 入口網站複製的 [Azure AD 識別碼] 值。

    c.  在記事本中開啟您下載的憑證檔，將其內容複製到剪貼簿上，然後貼到 [公開憑證] 文字方塊中。

    d. 針對您的 Slack 小組適當地設定上述三個設定。 如需這些這設定的詳細資訊，請參閱這裡的 **Slack 的 SSO 設定指南**。 `https://get.slack.help/hc/articles/220403548-Guide-to-single-sign-on-with-Slack%60`

    ![在應用程式端設定單一登入](./media/slack-tutorial/tutorial-slack-expand.png)

    e. 按一下 [展開]，然後在 [服務提供者簽發者] 文字方塊中輸入 `https://slack.com`。

    f.  按一下 [儲存組態] 。
    
    > [!NOTE]
    > 如果您有多個需要與 Azure AD 整合的 Slack 執行個體，請將 `https://<DOMAIN NAME>.slack.com` 設定為 **服務提供者簽發者**，使其可以與 Azure 應用程式 **識別碼** 設定配對。

### <a name="create-slack-test-user"></a>建立 Slack 測試使用者

本節的目標是要在 Slack 中建立一個名為 B.Simon 的使用者。 Slack 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 當您嘗試存取 Slack 時，如果 Slack 還沒有使用者，它就會建立新的使用者。 Slack 也支援自動使用者佈建，您可以在[這裡](slack-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

> [!NOTE]
> 如果您需要手動建立使用者，就需要連絡 [Slack 支援小組](https://slack.com/help/contact)。

> [!NOTE]
> Azure AD Connect 是一種同步處理工具，可以將內部部署的 Active Directory 身分識別同步處理至 Azure AD，之後這些已同步的使用者也可如同其他雲端使用者般地使用應用程式。

## <a name="test-sso"></a>測試 SSO

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至您可以在其中起始登入流程的 Slack 登入 URL。

* 直接移至 Slack 登入 URL，然後從該處起始登入流程。

* 您可以使用 Microsoft 的「我的應用程式」。 當您按一下「我的應用程式」中的 [Slack] 圖格時，將會重新導向至 Slack 登入 URL。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。

## <a name="next-steps"></a>後續步驟

設定 Slack 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-aad)