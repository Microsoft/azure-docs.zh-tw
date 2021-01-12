---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 Check Point CloudGuard Dome9 Arc 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Check Point CloudGuard Dome9 Arc 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/16/2020
ms.author: jeedes
ms.openlocfilehash: 29157b7230ec0e1bef612d6f4ea7c3f6a6cd104d
ms.sourcegitcommit: 67b44a02af0c8d615b35ec5e57a29d21419d7668
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/06/2021
ms.locfileid: "97914533"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-check-point-cloudguard-dome9-arc"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 Check Point CloudGuard Dome9 Arc 整合

在本教學課程中，您將了解如何整合 Check Point CloudGuard Dome9 Arc 與 Azure Active Directory (Azure AD)。 當您整合 Check Point CloudGuard Dome9 Arc 與 Azure AD 時，您可以：

* 在 Azure AD 中控制可存取 Check Point CloudGuard Dome9 Arc 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Check Point CloudGuard Dome9 Arc。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Check Point CloudGuard Dome9 Arc 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* Check Point CloudGuard Dome9 Arc 支援 **SP 和 IDP** 起始的 SSO

> [!NOTE]
> 此應用程式的識別碼是固定的字串值，因此一個租用戶中只能設定一個執行個體。

## <a name="adding-check-point-cloudguard-dome9-arc-from-the-gallery"></a>從資源庫新增 Check Point CloudGuard Dome9 Arc

若要設定將 Check Point CloudGuard Dome9 Arc 整合到 Azure AD 中，您必須從資源庫將 Check Point CloudGuard Dome9 Arc 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增]  區段的搜尋方塊中，輸入 **Check Point CloudGuard Dome9 Arc**。
1. 從結果面板選取 [Check Point CloudGuard Dome9 Arc]  ，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-check-point-cloudguard-dome9-arc"></a>針對 Check Point CloudGuard Dome9 Arc 設定和測試 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 Check Point CloudGuard Dome9 Arc 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Check Point CloudGuard Dome9 Arc 中相關使用者之間的連結關聯性。

若要設定及測試與 Check Point CloudGuard Dome9 Arc 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Check Point CloudGuard Dome9 Arc SSO](#configure-check-point-cloudguard-dome9-arc-sso)** ，在應用程式端設定單一登入設定。
    1. **[建立 Check Point CloudGuard Dome9 Arc 測試使用者](#create-check-point-cloudguard-dome9-arc-test-user)** 使 Check Point CloudGuard Dome9 Arc 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 **Check Point CloudGuard Dome9 Arc** 應用程式整合頁面上，尋找 **管理** 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入] 頁面上，按一下 [基本 SAML 設定] 的鉛筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 設定]  區段上，如果您想要以 **IDP** 起始模式設定應用程式，請輸入下列欄位的值：

    在 [回覆 URL] 文字方塊中，使用下列模式來輸入 URL：`https://secure.dome9.com/sso/saml/<yourcompanyname>`

1. 如果您想要以 **SP** 起始模式設定應用程式，請按一下 [設定其他 URL]  ，然後執行下列步驟：

    在 [登入 URL]  文字方塊中，以下列模式輸入 URL︰`https://secure.dome9.com/sso/saml/<yourcompanyname>`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的回覆 URL 與登入 URL 更新這些值。 您會從 [設定 Check Point CloudGuard Dome9 Arc SSO]  區段取得 `<company name>` 值，本教學課程稍後將加以說明。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

1. Check Point CloudGuard Dome9 Arc 應用程式預期應有特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增至 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/edit-attribute.png)

1. 除了以上屬性之外，Check Point CloudGuard Dome9 Arc 應用程式還預期 SAML 回應中會再多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。
    
    | 名稱 |  來源屬性|
    | ---------------| --------------- |
    | memberof | user.assignedroles |

    >[!NOTE]
    >請按一下[此處](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps#app-roles-ui)來了解如何在 Azure AD 中建立角色。

1. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，尋找 [憑證 (Base64)]  並選取 [下載]  ，以下載憑證並將其儲存在電腦上。

    ![憑證下載連結](common/certificatebase64.png)

1. 在 [設定 Check Point CloudGuard Dome9 Arc]  區段上，依據您的需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下 [建立]。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Check Point CloudGuard Dome9 Arc 的存取權授與 B. Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Check Point CloudGuard Dome9 Arc]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您已如上述所述設定角色，可以從 **選取角色** 下拉式清單中選取。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-check-point-cloudguard-dome9-arc-sso"></a>設定 Check Point CloudGuard Dome9 Arc SSO

1. 若要自動執行 Check Point CloudGuard Dome9 Arc 內的設定，您必須按一下 [安裝擴充功能]  以安裝「我的應用程式安全登入瀏覽器擴充功能」  。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 Check Point CloudGuard Dome9 Arc]  ，系統會將您引導至 Check Point CloudGuard Dome9 Arc 應用程式。 請從該處提供用以登入 Check Point CloudGuard Dome9 Arc 的管理員認證。瀏覽器擴充功能會自動為您設定應用程式，並自動執行步驟 3 到 6。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 Check Point CloudGuard Dome9 Arc，請開啟新的網頁瀏覽器視窗，並以系統管理員身分登入 Check Point CloudGuard Dome9 Arc 公司網站，然後執行下列步驟：

2. 按一下右上角的 [設定檔設定]  ，然後按一下 [帳戶設定]  。 

    ![顯示 [設定檔設定] 功能表的螢幕擷取畫面，其中已選取 [帳戶設定]。](./media/dome9arc-tutorial/configure1.png)

3. 瀏覽至 [SSO]  ，然後按一下 [啟用]  。

    ![顯示已選取 [SSO] 索引標籤和 [啟用] 的螢幕擷取畫面。](./media/dome9arc-tutorial/configure2.png)

4. 在 [SSO 設定] 區段中，執行下列步驟：

    ![Check Point CloudGuard Dome9 Arc 組態](./media/dome9arc-tutorial/configure3.png)

    a. 在 [帳戶識別碼]  文字方塊中輸入公司名稱。 此值會在 Azure 入口網站 [基本 SAML 組態]  區段所提及的 [回覆]  及 [登入]  URL 中使用。

    b. 在 [簽發者]  文字方塊中，貼上您從 Azure 入口網站複製的 [Azure AD 識別碼]  值。

    c. 在 [Idp 端點 URL]  文字方塊中，貼上您從 Azure 入口網站複製的 [登入 URL]  值。

    d. 在記事本中開啟您下載的 Base64 編碼的憑證，將其內容複製到剪貼簿，然後貼到 [X.509 憑證]  文字方塊。

    e. 按一下 [檔案]  。

### <a name="create-check-point-cloudguard-dome9-arc-test-user"></a>建立 Check Point CloudGuard Dome9 Arc 測試使用者

若要讓 Azure AD 使用者能夠登入 Check Point CloudGuard Dome9 Arc，必須將他們佈建到應用程式。 Check Point CloudGuard Dome9 Arc 支援 Just-In-Time 佈建；但是，若要正常佈建，使用者必須選取特定 **角色**，並將該角色指派給使用者。

   >[!Note]
   >關於 **角色** 建立和其他詳細資料，請連絡 [Check Point CloudGuard Dome9 Arc 用戶端支援小組](mailto:Dome9@checkpoint.com)。

**若要手動佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 Check Point CloudGuard Dome9 Arc 公司網站。

2. 按一下 [使用者和角色]  ，然後按一下 [使用者]  。

    ![顯示 [使用者和角色] 的螢幕擷取畫面，其中已選取 [使用者] 動作。](./media/dome9arc-tutorial/user1.png)

3. 按一下 [新增使用者]  。

    ![顯示 [使用者和角色] 的螢幕擷取畫面，其中已選取 [新增使用者] 按鈕。](./media/dome9arc-tutorial/user2.png)

4. 在 [建立使用者]  區段中，執行下列步驟：

    ![新增員工](./media/dome9arc-tutorial/user3.png)

    a. 在 [電子郵件]  文字方塊中，輸入使用者的電子郵件，例如 B.Simon@contoso.com。

    b. 在 [名字]  文字方塊中，輸入使用者的名字，例如 B。

    c. 在 [姓氏]  文字方塊中，輸入使用者的姓氏，例如 Simon。

    d. 將 [SSO 使用者]  設定為 [開啟]  。

    e. 按一下 [建立]  。

## <a name="test-sso"></a>測試 SSO 

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。 

#### <a name="sp-initiated"></a>SP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至您可以在其中起始登入流程的 Check Point CloudGuard Dome9 ArcC 登入 URL。  

* 直接移至 Check Point CloudGuard Dome9 Arc 登入 URL，然後從該處起始登入流程。

#### <a name="idp-initiated"></a>IDP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入您已設定 SSO 的 Check Point CloudGuard Dome9 Arc 

您也可以使用 Microsoft「我的應用程式」，以任何模式測試應用程式。 當您按一下「我的應用程式」中的 [Check Point CloudGuard Dome9 Arc] 圖格時，如果是在 SP 模式中設定，您會重新導向至 [應用程式登入] 頁面來起始登入流程，如果在 IDP 模式中設定，則應該會自動登入已設定 SSO 的 Check Point CloudGuard Dome9 Arc。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。


## <a name="next-steps"></a>後續步驟

當您設定了 Check Point CloudGuard Dome9 Arc 之後，就能施行工作階段控制項，即時保護您組織的敏感性資料免遭外洩及滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。