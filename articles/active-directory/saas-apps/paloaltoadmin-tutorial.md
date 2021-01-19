---
title: 教學課程：Azure Active Directory 與 Palo Alto Networks - Admin UI 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Palo Alto 網路 - 系統管理 UI 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: 57b1d47fa40c0af4bced1e4169fe60cd759ee2f3
ms.sourcegitcommit: f6f928180504444470af713c32e7df667c17ac20
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/07/2021
ms.locfileid: "97963633"
---
# <a name="tutorial-azure-active-directory-integration-with-palo-alto-networks---admin-ui"></a>教學課程：Azure Active Directory 與 Palo Alto Networks - Admin UI 整合

在本教學課程中，您將了解如何整合 Palo Alto 網路 - 系統管理 UI 與 Azure Active Directory (Azure AD)。
將 Palo Alto 網路 - 系統管理 UI 與 Azure AD 整合，能提供下列優點：

* 您可以在 Azure AD 中控制可存取 Palo Alto 網路 - 系統管理 UI 的人員。
* 您可以讓使用者使用其 Azure AD 帳戶自動登入 Palo Alto Networks - Admin UI (單一登入)。
* 您可以在 Azure 入口網站中集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Palo Alto 網路 - 系統管理 UI 整合，您需要下列項目：

* Azure AD 訂用帳戶。 如果您沒有 Azure AD 環境，您可以在[這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月的試用帳戶
* 已啟用 Palo Alto Networks - Admin UI 單一登入的訂用帳戶

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD 單一登入。

* Palo Alto Networks - Admin UI 支援 **SP** 起始的 SSO
* Palo Alto Networks - Admin UI 支援 **Just-In-Time** 使用者佈建

## <a name="adding-palo-alto-networks---admin-ui-from-the-gallery"></a>從資源庫新增 Palo Alto 網路 - 系統管理 UI

若要設定將 Palo Alto 網路 - 系統管理 UI 整合到 Azure AD 中，您需要從資源庫將 Palo Alto 網路 - 系統管理 UI 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]。
1. 在 [從資源庫新增] 區段的搜尋方塊中輸入 **Palo Alto Networks - Admin UI**。
1. 從結果面板中選取 [Palo Alto Networks - Admin UI]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso"></a>設定並測試 Azure AD SSO

在本節中，您會以名為 **B.Simon** 的測試使用者身分，設定及測試與 Palo Alto Networks - Admin UI 搭配運作的 Azure AD 單一登入。
若要讓單一登入能夠運作，必須建立 Azure AD 使用者與 Palo Alto Networks - Admin UI 中相關使用者之間的連結關聯性。

若要搭配 Palo Alto Networks - Admin UI 設定並測試 Azure AD 單一登入，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    * **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    * **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Palo Alto Networks - Admin UI SSO](#configure-palo-alto-networks---admin-ui-sso)** - 在應用程式端設定單一登入設定。
    * **[建立 Palo Alto Networks - Admin UI 測試使用者](#create-palo-alto-networks---admin-ui-test-user)** ：使 Palo Alto Networks - Admin UI 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 **Azure 入口網站** 的 **Palo Alto Networks - Admin UI** 應用程式整合頁面上，找到 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入] 頁面上，按一下 [基本 SAML 設定] 的鉛筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，執行下列步驟：

    a. 在 [登入 URL]  文字方塊中，以下列模式輸入 URL︰`https://<Customer Firewall FQDN>/php/login.php`

    b. 在 [識別碼] 方塊中，使用下列模式輸入 URL：`https://<Customer Firewall FQDN>:443/SAML20/SP`

    c. 在 [回覆 URL] 文字方塊中，以下列格式輸入判斷提示取用者服務 (ACS) URL：`https://<Customer Firewall FQDN>:443/SAML20/SP/ACS`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「單一登入 URL」、「識別碼」及「回覆 URL」來更新這些值。 請連絡 [Palo Alto 網路 - 系統管理 UI 用戶端支援小組](https://support.paloaltonetworks.com/support) \(英文\) 以取得這些值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。
    >
    > [識別碼] 和 [回覆 URL] 上都需要連接埠 443，因為這些值已硬式編碼至 Palo Alto 防火牆。 移除連接埠號碼會在登入期間產生錯誤 (如果已移除)。

    > [識別碼] 和 [回覆 URL] 上都需要連接埠 443，因為這些值已硬式編碼至 Palo Alto 防火牆。 移除連接埠號碼會在登入期間產生錯誤 (如果已移除)。

1. Palo Alto Networks - Admin UI 應用程式需要特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增至您的 SAML 權杖屬性組態。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/default-attributes.png)

   > [!NOTE]
   > 這些屬性值只是範例，因此您必須對應 *username* 和 *adminrole* 的適當值。 此外還有另一個選用屬性 *accessdomain*，可用來限制管理員對防火牆上特定虛擬系統的存取。

1. 除了上述屬性外，Palo Alto Networks - Admin UI 應用程式還需要在 SAML 回應中多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。

    | 名稱 |  來源屬性|
    | --- | --- |
    | username | user.userprincipalname |
    | adminrole | customadmin |
    | | |

    > [!NOTE]
    > adminrole 值應該與在步驟 9 中於 **Palo Alto Networks** 中設定的角色名稱相同。 

    > [!NOTE]
    > 如需關於這些屬性的詳細資訊，請參閱下列文章：
    > * [系統管理 UI (adminrole) 的系統管理角色設定檔](https://www.paloaltonetworks.com/documentation/80/pan-os/pan-os/firewall-administration/manage-firewall-administrators/configure-an-admin-role-profile)
    > * [系統管理 UI (accessdomain) 的裝置存取網域](https://docs.paloaltonetworks.com/pan-os/8-0/pan-os-web-interface-help/device/device-access-domain.html)

1. 在 [以 SAML 設定單一登入] 頁面的 [SAML 簽署憑證] 區段中按一下 [下載]，以依據您的需求從指定選項下載 **同盟中繼資料 XML**，並儲存在您的電腦上。

    ![憑證下載連結](common/metadataxml.png)

1. 在 [設定 Palo Alto Networks - Admin UI] 區段上，依據您的需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)


### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下頁面底部的 [新增] 。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 Palo Alto Networks - Admin UI 的存取權授與 B.Simon，讓她能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]，然後選取 [所有應用程式]。
1. 在應用程式清單中，選取 [Palo Alto 網路 - 系統管理 UI]。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派] 對話方塊中，按一下 [指派] 按鈕。

## <a name="configure-palo-alto-networks---admin-ui-sso"></a>設定 Palo Alto Networks - Admin UI SSO

1. 以系統管理員的身分，在新視窗中開啟 Palo Alto 網路防火牆的系統管理 UI。

2. 選取 [裝置] 索引標籤。

    ![[裝置] 索引標籤](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_admin1.png)

3. 在左窗格中選取 [SAML 身分識別提供者]，然後選取 [匯入] 以匯入中繼資料檔案。

    ![匯入中繼資料檔案按鈕](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_admin2.png)

4. 在 [SAML 身分識別提供者伺服器設定檔匯入] 視窗中，執行下列動作：

    ![[SAML 身分識別提供者伺服器設定檔匯入] 視窗](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_idp.png)

    a. 在 [設定檔名稱] 方塊中提供名稱，例如 **AzureAD 系統管理 UI**。

    b. 在 [身分識別提供者中繼資料] 下選取 [瀏覽]，然後選取您先前從 Azure 入口網站下載的 metadata.xml 檔案。

    c. 清除 [驗證身分識別提供者憑證] 核取方塊。

    d. 選取 [確定]。

    e. 若要認可防火牆的組態，請選取 [認可]。

5. 在左窗格中選取 [SAML 身分識別提供者]，然後選取您在先前的步驟中建立的 SAML 身分識別提供者設定檔 (例如 **AzureAD 系統管理 UI**)。

    ![SAML 身分識別提供者設定檔](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_idp_select.png)

6. 在 [SAML 身分識別提供者伺服器設定檔] 視窗中，執行下列動作：

    ![[SAML 身分識別提供者伺服器設定檔] 視窗](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_slo.png)
  
    a. 在 [身分識別提供者 SLO URL] 方塊中，將先前匯入的 SLO URL 取代為下列 URL：`https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0`
  
    b. 選取 [確定]。

7. 在 Palo Alto 網路防火牆的系統管理 UI 中選取 [裝置]，然後選取 [管理員角色]

8. 選取 [新增] 按鈕。

9. 在 [管理員角色設定檔] 視窗的 [名稱] 方塊中，提供管理員角色的名稱 (例如 **fwadmin**)。 管理員角色名稱應符合身分識別提供者所傳送的 SAML 管理員角色屬性名稱。 在 Azure 入口網站的 [使用者屬性] 中已建立管理員角色名稱和值。

    ![設定 Palo Alto 網路管理員角色](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_adminrole.png)
  
10. 在防火牆的系統管理 UI 上選取 [裝置]，然後選取 [驗證設定檔]。

11. 選取 [新增] 按鈕。

12. 在 [驗證設定檔] 視窗中，執行下列動作： 

    ![[驗證設定檔] 視窗](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_authentication_profile.png)

    a. 在 [名稱] 方塊中提供名稱，例如 **AzureSAML_Admin_AuthProfile**。

    b. 選取 [類型] 下拉式清單中，選取 [SAML]。 

    c. 在 [IdP 伺服器設定檔] 下拉式清單中，選取適當的 SAML 身分識別提供者伺服器設定檔 (例如 **AzureAD 系統管理 UI**)。

    c. 選取 [啟用單一登出] 核取方塊。

    d. 在 [管理員角色屬性] 方塊中輸入屬性名稱，例如 **adminrole**。

    e. 選取 [進階] 索引標籤，然後在 [允許清單] 下選取 [新增]。

    ![[進階] 索引標籤上的 [新增] 按鈕](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_allowlist.png)

    f. 選取 [全部] 核取方塊，或選取可以使用此設定檔進行驗證的特定使用者和群組。  
    當使用者進行驗證時，防火牆會針對此清單中的項目比對相關聯的使用者名稱或群組。 如果您未新增項目，則無法驗證任何使用者。

    g. 選取 [確定]。

13. 若要使用 Azure 讓管理員能夠使用 SAML SSO，請選取 [裝置] > [設定]。 在 [設定] 窗格中選取 [管理] 索引標籤，然後選取 [驗證設定] 下的 [設定] (「齒輪」) 按鈕。

    ![[設定] 按鈕](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_authsetup.png)

14. 選取您在 [驗證設定檔] 視窗中建立的 SAML 驗證設定檔 (例如 **AzureSAML_Admin_AuthProfile**)。

    ![[驗證設定檔] 欄位](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_authsettings.png)

15. 選取 [確定]。

16. 若要認可組態，請選取 [認可]。

### <a name="create-palo-alto-networks---admin-ui-test-user"></a>建立 Palo Alto Networks - Admin UI 測試使用者

Palo Alto 網路 - 系統管理 UI 支援 Just-In-Time 使用者佈建。 如果使用者尚不存在，在成功驗證之後即會在系統中自動建立。 您不需要手動建立使用者。

## <a name="test-sso"></a>測試 SSO

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。 

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至您可以在其中起始登入流程的 Palo Alto Networks - Admin UI 登入 URL。 

* 直接移至 Palo Alto Networks - Admin UI 登入 URL，然後從該處起始登入流程。

* 您可以使用 Microsoft 的「我的應用程式」。 當您在我的應用程式中按一下 Palo Alto Networks - Admin UI 圖格時，系統應該會將您自動登入您設定 SSO 的 Palo Alto Networks - Admin UI。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](../user-help/my-apps-portal-end-user-access.md)。


## <a name="next-steps"></a>後續步驟

設定 Palo Alto Networks - Admin UI 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。