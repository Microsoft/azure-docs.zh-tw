---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 TeamzSkill 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 TeamzSkill 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2020
ms.author: jeedes
ms.openlocfilehash: 22e237a3451c87fd4867949b9aa94d19b6fcbbdf
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98729423"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-teamzskill"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 TeamzSkill 整合

在本教學課程中，您將了解如何整合 TeamzSkill 與 Azure Active Directory (Azure AD)。 在整合 TeamzSkill 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 TeamzSkill 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 TeamzSkill。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 TeamzSkill 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* TeamzSkill 支援由 **SP 和 IDP** 起始的 SSO
* TeamzSkill 支援 **Just In Time** 使用者佈建

## <a name="adding-teamzskill-from-the-gallery"></a>從資源庫新增 TeamzSkill

若要進行將 TeamzSkill 整合到 Azure AD 中的設定，您必須從資源庫將 TeamzSkill 新增至受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增] 區段的搜尋方塊中，輸入 **TeamzSkill**。
1. 從結果面板中選取 [TeamzSkill]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。


## <a name="configure-and-test-azure-ad-sso-for-teamzskill"></a>設定和測試 TeamzSkill 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 TeamzSkill 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 TeamzSkill 中相關使用者之間的連結關聯性。

若要設定及測試與 TeamzSkill 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 TeamzSkill SSO](#configure-teamzskill-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 TeamzSkill 測試使用者](#create-teamzskill-test-user)** - 使 TeamzSkill 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [TeamzSkill] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入] 頁面上，按一下 [基本 SAML 設定] 的鉛筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 設定] 區段上，如果您想要以 **IDP** 起始模式設定應用程式，請輸入下列欄位的值：

    a. 在 [識別碼] 文字方塊中，使用下列模式來輸入 URL：`https://<CUSTOMER_SUBDOMAIN>.teamzskill.com/login/callback`

    b. 在 [回覆 URL] 文字方塊中，使用下列模式來輸入 URL：`https://<CUSTOMER_SUBDOMAIN>.teamzskill.com/login/callback`

1. 如果您想要以 **SP** 起始模式設定應用程式，請按一下 [設定其他 URL]，然後執行下列步驟：

    在 [登入 URL] 文字方塊中，以下列模式輸入 URL︰`https://<CUSTOMER_SUBDOMAIN>.teamzskill.com/login/callback`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的「識別碼」、「回覆 URL」及「登入 URL」來更新這些值。 請連絡 [TeamzSkill 用戶端支援小組](mailto:support@teamzskill.com)以取得這些值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

1. TeamzSkill 應用程式需要特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增至 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/default-attributes.png)

1. 除了上述屬性外，TeamzSkill 應用程式還需要在 SAML 回應中多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。
    
    | 名稱 |  來源屬性|
    | ------------ | --------- |
    | Firstname | user.givenname |
    | lastname | user.surname |
    | jobtitle | user.jobtitle |
    | department | user.department |
    | employeeid | user.employeeid |
    | postalcode | user.postalcode |
    | country | user.country |
    | 角色 (role) | user.assignedroles |

    > [!NOTE]
    > TeamzSkill 需要將使用者的角色指派給應用程式。 請在 Azure AD 中設定這些角色，以便為使用者指派適當的角色。 若要了解如何在 Azure AD 中設定角色，請參閱[此文章](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui--preview)。

1. 在 [以 SAML 設定單一登入]  頁面上的 [SAML 簽署憑證]  區段中，尋找 [同盟中繼資料 XML]  ，然後選取 [下載]  ，以下載憑證並將其儲存在電腦上。

    ![憑證下載連結](common/metadataxml.png)

1. 在 [設定 TeamzSkill] 區段上，根據您的需求複製適當的 URL。

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

在本節中，您會將 TeamzSkill 的存取權授與 B.Simon，使其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [TeamzSkill]。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您已如上述所述設定角色，可以從 **選取角色** 下拉式清單中選取。
1. 在 [新增指派] 對話方塊中，按一下 [指派] 按鈕。

## <a name="configure-teamzskill-sso"></a>設定 TeamzSkill SSO

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入 TeamzSkill。

1. 按一下 [使用者設定檔] 圖示，然後選取 [公司設定]。

    ![Teamzskill 中的公司設定](./media/teamzskill-tutorial/settings.png)

1. 在 [設定] 頁面中執行下列步驟。

    ![Teamzskill 中的設定](./media/teamzskill-tutorial/metadata.png)

    a. 瀏覽至 [公司] > [單一登入]，然後選取 [中繼資料上傳] 索引標籤。

    b. 將您從 Azure 入口網站複製的 [同盟中繼資料 XML] 值貼到 [XML 中繼資料] 欄位中。
     
    c. 然後按一下 [儲存]  。

### <a name="create-teamzskill-test-user"></a>建立 TeamzSkill 測試使用者

本節會在 TeamzSkill 中建立名為 B.Simon 的使用者。 TeamzSkill 支援依預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 如果 TeamzSkill 中還沒有使用者，當您嘗試存取 TeamzSkill 時，就會建立新的使用者。

## <a name="test-sso"></a>測試 SSO 

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。 

#### <a name="sp-initiated"></a>SP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至您可以在其中起始登入流程的 TeamzSkill 登入 URL。  

* 直接移至 TeamzSkill 登入 URL，然後從該處起始登入流程。

#### <a name="idp-initiated"></a>IDP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入您已設定 SSO 的 TeamzSkill 

您也可以使用 Microsoft 存取面板，以任何模式測試應用程式。 當您按一下存取面板中的 TeamzSkill 圖格時，如果是在 SP 模式中設定，您會重新導向至 [應用程式登入] 頁面來起始登入流程，如果是在 IDP 模式中設定，則應該會自動登入已設定 SSO 的 TeamzSkill。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/my-apps-portal-end-user-access.md)。


## <a name="next-steps"></a>後續步驟

設定 TeamzSkill 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。
