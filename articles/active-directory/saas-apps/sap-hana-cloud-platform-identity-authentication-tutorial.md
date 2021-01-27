---
title: 教學課程：Azure Active Directory 與 SAP Cloud Platform Identity Authentication 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SAP Cloud Platform Identity Authentication 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/18/2021
ms.author: jeedes
ms.openlocfilehash: dc0cd57eb32baaeac0850337bbead3a73dec9292
ms.sourcegitcommit: 100390fefd8f1c48173c51b71650c8ca1b26f711
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98897325"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-cloud-platform-identity-authentication"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 SAP Cloud Platform Identity Authentication 整合

在本教學課程中，您將了解如何整合 SAP Cloud Platform Identity Authentication 與 Azure Active Directory (Azure AD)。 在整合 SAP Cloud Platform Identity Authentication 與 Azure AD 時，您可以：

* 在 Azure AD 中控制可存取 SAP Cloud Platform Identity Authentication 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 SAP Cloud Platform Identity Authentication。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 SAP Cloud Platform Identity Authentication 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD 單一登入。

* SAP Cloud Platform Identity Authentication 支援由 **SP** 和 **IDP** 起始的 SSO

深入探討技術細節之前，一定要了解您即將查看的概念。 SAP Cloud Platform Identity Authentication 和 Active Directory 同盟服務可讓您利用 SAP Cloud Platform Identity Authentication 所保護的 SAP 應用程式和服務，跨越 Azure AD (如 IdP) 所保護的應用程式或服務實作 SSO。

SAP Cloud Platform Identity Authentication 目前作為 SAP 應用程式的領導 Proxy 識別提供者。 而 Azure Active Directory 做為此設定中的識別提供者。 

下圖說明此關聯性：

![建立 Azure AD 測試使用者](./media/sap-hana-cloud-platform-identity-authentication-tutorial/architecture-01.png)

透過此設定，您的 SAP Cloud Platform Identity Authentication 租用戶會設定為 Azure Active Directory 中信任的應用程式。

接著會在 SAP Cloud Platform Identity Authentication 管理主控台中設定您想以此方式保護的所有 SAP 應用程式和服務。

因此，必須在 SAP Cloud Platform Identity Authentication 中進行授與 SAP 應用程式和服務存取權的授權作業 (相對於在 Azure Active Directory 中設定授權)。

藉由透過 Azure Active Directory Marketplace 將 SAP Cloud Platform Identity Authentication 設定為應用程式，您不需要設定個別宣告或 SAML 判斷提示。

> [!NOTE]
> 目前只有 Web SSO 經過這兩個合作夥伴的測試。 應用程式對 API 或 API 對 API 通訊所需的流程應能運作，但尚未經過測試。 它們會在後續活動期間進行測試。

## <a name="adding-sap-cloud-platform-identity-authentication-from-the-gallery"></a>從資源庫新增 SAP Cloud Platform Identity Authentication

若要設定將 SAP Cloud Platform Identity Authentication 整合到 Azure AD 中，您需要將 SAP Cloud Platform Identity Authentication 從資源庫新增到受控 SaaS 應用程式清單中。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增] 區段的搜尋方塊中，輸入 **SAP Cloud Platform Identity Authentication**。
1. 從結果面板中選取 [SAP Cloud Platform Identity Authentication]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。


## <a name="configure-and-test-azure-ad-sso-for-sap-cloud-platform-identity-authentication"></a>設定及測試 SAP Cloud Platform Identity Authentication 的 Azure AD SSO

使用名為 **B.Simon** 的測試使用者，設定及測試與 SAP Cloud Platform Identity Authentication 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 SAP Cloud Platform Identity Authentication 中相關使用者之間的連結關聯性。

若要設定及測試與 SAP Cloud Platform Identity Authentication 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 SAP Cloud Platform Identity Authentication SSO](#configure-sap-cloud-platform-identity-authentication-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 SAP Cloud Platform Identity Authentication 測試使用者](#create-sap-cloud-platform-identity-authentication-test-user)** - 使 SAP Cloud Platform Identity Authentication 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [SAP Cloud Platform Identity Authentication] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

4. 在 [基本 SAML 組態] 區段上，若您想要以 **IDP** 起始模式進行設定，請執行下列步驟：

    a. 在 [識別碼]  文字方塊中，使用下列模式來輸入 URL：`<IAS-tenant-id>.accounts.ondemand.com`

    b. 在 [回覆 URL]  文字方塊中，使用下列模式來輸入 URL：`https://<IAS-tenant-id>.accounts.ondemand.com/saml2/idp/acs/<IAS-tenant-id>.accounts.ondemand.com`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 更新這些值。 請連絡 [SAP Cloud Platform Identity Authentication 客戶支援小組](https://cloudplatform.sap.com/capabilities/security/trustcenter.html)以取得這些值。 如果您不知道此識別碼值，請參閱關於[租用戶 SAML 2.0 設定](https://help.hana.ondemand.com/cloud_identity/frameset.htm?e81a19b0067f4646982d7200a8dab3ca.html)的 SAP Cloud Platform Identity Authentication 文件。

5. 如果您想要以 **SP** 起始模式設定應用程式，請按一下 [設定其他 URL]，然後執行下列步驟：

    ![SAP Cloud Platform Identity Authentication 網域及 URL 單一登入資訊](common/metadata-upload-additional-signon.png)

    在 [登入 URL]  文字方塊中，以下列模式輸入 URL︰`{YOUR BUSINESS APPLICATION URL}`

    > [!NOTE]
    > 這不是真實的值。 請使用實際的登入 URL 來更新此值。 請使用您特有的商務應用程式登入 URL。 如有任何疑問，請連絡 [SAP Cloud Platform Identity Authentication 客戶支援小組](https://cloudplatform.sap.com/capabilities/security/trustcenter.html)。

1. SAP Cloud Platform Identity Authentication 應用程式需要特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增至 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/default-attributes.png)

1. 除了上述屬性外，SAP Cloud Platform Identity Authentication 應用程式還需要在 SAML 回應中多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。

    | 名稱 | 來源屬性|
    | ---------------| --------------- |
    | firstName | user.givenname |

8. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，按一下 [下載]  以依據您的需求從指定選項下載 [中繼資料 XML]  ，並儲存在您的電腦上。

    ![憑證下載連結](common/metadataxml.png)

9. 在 [設定 SAP Cloud Platform Identity Authentication] 區段上，依據您的需求複製適當的 URL。

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

在本節中，您會將 SAP Cloud Platform Identity Authentication 的存取權授與 B.Simon，使其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 **SAP Cloud Platform Identity Authentication**。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。

1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。

1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-sap-cloud-platform-identity-authentication-sso"></a>設定 SAP Cloud Platform Identity Authentication SSO

1. 若要自動執行 SAP Cloud Platform Identity Authentication 內的設定，您必須按一下 [安裝擴充功能] 來安裝「我的應用程式安全登入瀏覽器擴充功能」。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 SAP Cloud Platform Identity Authentication] 便會將您導向至 SAP Cloud Platform Identity Authentication 應用程式。 請從該處提供用以登入 SAP Cloud Platform Identity Authentication 的管理員認證。 瀏覽器延伸模組將會自動為您設定應用程式，並自動執行步驟 3 到 7。

    ![設定組態](common/setup-sso.png)

3. 如果您想要手動設定 SAP Cloud Platform Identity Authentication，請在不同的網頁瀏覽器視窗中，移至 SAP Cloud Platform Identity Authentication 管理主控台。 URL 具有下列模式：`https://<tenant-id>.accounts.ondemand.com/admin`。 然後參閱位於[與 Microsoft Azure AD 整合](https://developers.sap.com/tutorials/cp-ias-azure-ad.html)的 SAP Cloud Platform Identity Authentication 相關文件。

2. 在 Azure 入口網站中，選取 [儲存] 按鈕。

3. 只有當您想要為另一個 SAP 應用程式新增和啟用 SSO 時，才繼續以下步驟。 重複 **從資源庫新增 SAP Cloud Platform Identity Authentication** 一節底下的步驟。

4. 在 Azure 入口網站的 [SAP Cloud Platform Identity Authentication] 應用程式整合分頁上，選取 [連結的登入]。

    ![設定連結的登入](./media/sap-hana-cloud-platform-identity-authentication-tutorial/linked-sign-on.png)

5. 儲存組態。

> [!NOTE]
> 新的應用程式會利用先前 SAP 應用程式的單一登入設定。 請確定您在 SAP Cloud Platform Identity Authentication 管理主控台中使用相同的公司識別提供者。

### <a name="create-sap-cloud-platform-identity-authentication-test-user"></a>建立 SAP Cloud Platform Identity Authentication 測試使用者

您不需要在 SAP Cloud Platform Identity Authentication 中建立使用者。 Azure AD 使用者存放區中的使用者可以使用 SSO 功能。

SAP Cloud Platform Identity Authentication 支援 [識別身分同盟] 選項。 此選項可讓應用程式檢查公司識別提供者所驗證的使用者是否存在於 SAP Cloud Platform Identity Authentication 的使用者存放區中。

[識別身分同盟] 選項預設為停用。 如果已啟用 [識別身分同盟]，則只有匯入 SAP Cloud Platform Identity Authentication 的使用者可以存取應用程式。

如需如何啟用或停用與 SAP Cloud Platform Identity Authentication 之識別身分同盟的詳細資訊，請參閱[設定與 SAP Cloud Platform Identity Authentication 之使用者存放區的識別身分同盟](https://help.sap.com/viewer/6d6d63354d1242d185ab4830fc04feb1/Cloud/c029bbbaefbf4350af15115396ba14e2.html)中的＜啟用與 SAP Cloud Platform Identity Authentication 的識別身分同盟＞。

## <a name="test-sso"></a>測試 SSO 

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

#### <a name="sp-initiated"></a>SP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至您可以在其中起始登入流程的 SAP Cloud Platform Identity Authentication 登入 URL。

* 直接移至 SAP Cloud Platform Identity Authentication 登入 URL，然後從該處起始登入流程。

#### <a name="idp-initiated"></a>IDP 起始：

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入已設定 SSO 的 SAP Cloud Platform Identity Authentication

您也可以使用 Microsoft「我的應用程式」，以任何模式測試應用程式。 當您按一下「我的應用程式」中的 [SAP Cloud Platform Identity Authentication] 圖格時，如果是在 SP 模式中設定，您會重新導向至 [應用程式登入] 頁面來起始登入流程，如果是在 IDP 模式中設定，則應該會自動登入已設定 SSO 的 SAP Cloud Platform Identity Authentication。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。

## <a name="next-steps"></a>後續步驟

設定 SAP Cloud Platform Identity Authentication 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-aad)