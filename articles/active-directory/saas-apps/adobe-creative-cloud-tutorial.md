---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 Adobe Creative Cloud 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Adobe Creative Cloud 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/13/2021
ms.author: jeedes
ms.openlocfilehash: 82ab206db86aace60dca130d8f094a2a19318763
ms.sourcegitcommit: a0c1d0d0906585f5fdb2aaabe6f202acf2e22cfc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98621907"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-adobe-creative-cloud"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 Adobe Creative Cloud 整合

在本教學課程中，您會了解如何整合 Adobe Creative Cloud 與 Azure Active Directory (Azure AD)。 在整合 Adobe Creative Cloud 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 Adobe Creative Cloud 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Adobe Creative Cloud。
* 在 Azure 入口網站集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Adobe Creative Cloud 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* Adobe Creative Cloud 支援由 **SP** 初始化的 SSO

## <a name="add-adobe-creative-cloud-from-the-gallery"></a>從資源庫新增 Adobe Creative Cloud

若要設定將 Adobe Creative Cloud 整合至 Azure AD 中，您需要從資源庫將 Adobe Creative Cloud 新增至受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]。
1. 在 [從資源庫新增] 區段的搜尋方塊中輸入 **Adobe Creative Cloud**。
1. 從結果面板選取 [Adobe Creative Cloud]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-adobe-creative-cloud"></a>設定及測試適用於 Adobe Creative Cloud 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 Adobe Creative Cloud 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Adobe Creative Cloud 中相關使用者之間的連結關聯性。

若要設定及測試與 Adobe Creative Cloud Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Adobe Creative Cloud SSO](#configure-adobe-creative-cloud-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 Adobe Creative Cloud 測試使用者](#create-adobe-creative-cloud-test-user)** - 在 Adobe Creative Cloud 中，建立一個與 Azure AD 中代表使用者的項目連結的 B.Simon 對應項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [ **Adobe Creative Cloud** 應用程式整合] 頁面上，尋找 [ **管理** ] 區段並選取 [ **單一登入**]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入] 頁面上，按一下 [基本 SAML 設定] 的鉛筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，輸入下列欄位的值：

    a. 在 [登入 URL] 文字方塊中，輸入 URL：`https://adobe.com`

    b. 在 [識別碼 (實體識別碼)]  文字方塊中，使用下列模式輸入 URL：`https://www.okta.com/saml2/service-provider/<token>`

    > [!NOTE]
    > 識別碼值不是實際值。 請遵循 **設定 Adobe Cloud SSO** 一節步驟 4 中的指導方針。 在該步驟中，您可以開啟 **同盟中繼資料 XML 檔案**，並從中取得實體識別碼值，並將其放入 Azure AD 設定中作為識別碼值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

1. Adobe Creative Cloud 應用程式預期應有特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增至 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示預設屬性清單。

    ![image](common/edit-attribute.png)

1. 除了上述屬性外，Adobe Creative Cloud 應用程式還需要在 SAML 回應中多傳回幾個屬性，如下所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。

    | 名稱 | 來源屬性|
    |----- | --------- |
    | 名字 | user.givenname |
    | 姓氏 | user.surname |
    | 電子郵件 | user.mail |

    > [!NOTE]
    > 使用者需要具備有效的 Microsoft 365 ExO 授權，電子郵件宣告值才會填入 SAML 回應中。

1. 在 [以 SAML 設定單一登入] 頁面上的 [SAML 簽署憑證] 區段中，尋找 [同盟資料 XML]，然後選取 [下載]，以下載 XML 中繼資料檔案並將其儲存在電腦上。

    ![憑證下載連結](common/certificatebase64.png)

1. 在 [安裝 Adobe Creative Cloud] 區段上，依據您的需求複製適當的 URL。

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

在本節中，您會將 Adobe Creative Cloud 的存取權授與 b. Simon，讓她能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]   > [所有應用程式]  。
1. 在應用程式清單中，選取 [Adobe Creative Cloud]。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  。 然後，在 [新增指派]  對話方塊中，選取 [使用者和群組]  。
1. 在 [使用者和群組]  對話方塊的使用者清單中，選取 [B.Simon]  。 然後，選擇畫面底部的 [選取]  按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派]  對話方塊中，選取 [指派]  。

## <a name="configure-adobe-creative-cloud-sso"></a>設定 Adobe Creative Cloud SSO

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入 [Adobe Admin Console](https://adminconsole.adobe.com)。

1. 移至上方導覽列中的 [設定]，然後選擇 [身分識別]。 目錄清單隨即開啟。 選取您想要的同盟目錄。

1. 在 [目錄詳細資料] 頁面上，選取 [設定]。

1. 複製 [實體識別碼] 和 [ACS URL] ([判斷提示取用者服務 URL] 或 [回覆 URL])。 在 Azure 入口網站的適當欄位中輸入 URL。

    ![在應用程式端設定單一登入](./media/adobe-creative-cloud-tutorial/tutorial_adobe-creative-cloud_003.png)

    a. 在 [設定應用程式設定] 對話方塊中，使用 Adobe 提供給您的 [實體識別碼] 值作為 [識別碼]。

    b. 在 [設定應用程式設定] 對話方塊中，使用 Adobe 提供給您的 [ACS URL (判斷提示取用者服務 URL)] 值作為 [回覆 URL]。

1. 在頁面底部附近，上傳您從 Azure 入口網站下載的 **同盟資料 XML** 檔案。 

    ![同盟資料 XML 檔案](https://helpx.adobe.com/content/dam/help/en/enterprise/kb/configure-microsoft-azure-with-adobe-sso/jcr_content/main-pars/procedure/proc_par/step_228106403/step_par/image_copy/saml_signinig_certificate.png "IdP 中繼資料 XML")

1. 選取 [儲存]。

### <a name="create-adobe-creative-cloud-test-user"></a>建立 Adobe Creative Cloud 測試使用者

若要讓 Azure AD 使用者能夠登入 Adobe Creative Cloud，必須將他們佈建到 Adobe Creative Cloud。 Adobe Creative Cloud 需以手動方式佈建。

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>若要佈建使用者帳戶，請執行下列步驟：

1. 以系統管理員身分登入 [Adobe Admin Console](https://adminconsole.adobe.com)。

2. 在 Adobe 主控台內將使用者新增為同盟識別碼，並將它們指派給產品設定檔。 如需新增使用者的詳細資訊，請參閱 [在 Adobe 管理主控台中新增使用者](https://helpx.adobe.com/enterprise/using/users.html#Addusers)。

3. 此時，在 Adobe 登入表單中輸入您的電子郵件地址/upn，按下 tab 鍵，隨後應備份至 Azure AD 同盟：
   * Web 存取：www\.adobe.com > 登入
   * 在桌面應用程式公用程式內 > 登入
   * 在應用程式內 > 說明 > 登入

## <a name="test-sso"></a>測試 SSO

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。 

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向至 Adobe Creative Cloud 登入 URL，您可以在其中起始登入流程。 

* 直接前往 Adobe Creative Cloud 登入 URL，並從該處起始登入流程。

* 您可以使用 Microsoft 的「我的應用程式」。 當您按一下我的應用程式中的 Adobe Creative Cloud 圖格時，這會重新導向至 Adobe Creative Cloud 登入 URL。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。

## <a name="next-steps"></a>後續步驟

設定 Adobe Creative Cloud 後，您可以強制執行工作階段控制，以即時防止組織的敏感性資料遭到外洩及滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app)
