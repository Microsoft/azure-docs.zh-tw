---
title: 教學課程：Azure Active Directory 與 ServiceNow 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 ServiceNow 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: a5a1a264-7497-47e7-b129-a1b5b1ebff5b
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2018
ms.author: jeedes
ms.openlocfilehash: 5d5c4d5e26fa21488dd637805a4c22bd3ed18a7f
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421078"
---
# <a name="tutorial-azure-active-directory-integration-with-servicenow"></a>教學課程：Azure Active Directory 與 ServiceNow 整合

在本教學課程中，您會了解如何整合 ServiceNow 與 Azure Active Directory (Azure AD)。

ServiceNow 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中管控可存取 ServiceNow 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 ServiceNow (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 ServiceNow 的整合作業，您需要下列項目：

- Azure AD 訂用帳戶
- 若為 ServiceNow，ServiceNow 的執行個體或租用戶 (Calgary 版本或更新版本)
- 若為 ServiceNow Express，ServiceNow Express 的執行個體 (Helsinki 版本或更新版本)
- ServiceNow 租用戶必須啟用[多個提供者單一登入外掛程式](http://wiki.servicenow.com/index.php?title=Multiple_Provider_Single_Sign-On#gsc.tab=0)。 [提交服務要求](https://hi.service-now.com)即可達到此目的。
- 若要進行自動設定，請為 ServiceNow 啟用多重提供者外掛程式。

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 ServiceNow
1. 設定並測試 Azure AD 單一登入

## <a name="adding-servicenow-from-the-gallery"></a>從資源庫新增 ServiceNow
若要設定 ServiceNow 與 Azure AD 整合，您需要從資源庫將 ServiceNow 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 ServiceNow，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **ServiceNow**，從結果面板中選取 [ServiceNow]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 ServiceNow](./media/servicenow-tutorial/tutorial_servicenow_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 ServiceNow 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 ServiceNow 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 ServiceNow 中的相關使用者之間建立連結關聯性。

在 ServiceNow 中，將 Azure AD 中 [使用者名稱] 的值指派為 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試透過 ServiceNow 使用 Azure AD 單一登入功能，您需要完成下列建置組塊：

1. **[針對 ServiceNow 設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on-for-servicenow)** - 讓您的使用者能夠使用此功能。
1. **[針對 ServiceNow Express 設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on-for-servicenow-express)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 ServiceNow 測試使用者](#create-a-servicenow-test-user)** - 使 ServiceNow 中 Britta Simon 的對應使用者連結到使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on-for-servicenow"></a>為 ServiceNow 設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 ServiceNow 應用程式中設定單一登入。

**若要使用 ServiceNow 設定 Azure AD 單一登入功能，請執行下列步驟：**

1. 在 Azure 入口網站的 [ServiceNow] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/servicenow-tutorial/tutorial_servicenow_samlbase.png)

1. 在 [ServiceNow 網域與 URL] 區段上，執行下列步驟：

    ![ServiceNow 網域與 URL 單一登入資訊](./media/servicenow-tutorial/tutorial_servicenow_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<instance-name>.service-now.com/navpage.do`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<instance-name>.service-now.com`

    > [!NOTE] 
    > 這些都不是真正的值。 您將從實際的登入 URL 和識別碼更新這些值，本教學課程稍後將會說明。

1. 在 [SAML 簽署憑證] 區段上，執行下列步驟： 

    ![憑證下載連結](./media/servicenow-tutorial/tutorial_servicenow_certificate.png)

    a. 按一下 [複製] 按鈕以複製 [應用程式同盟中繼資料 Url]，並將它貼到記事本，因為本教學課程稍後會使用這個應用程式同盟中繼資料 Url。

    b. 按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/servicenow-tutorial/tutorial_general_400.png)

1. 以系統管理員身分登入您的 ServiceNow 應用程式。

1. 遵循下一個步驟，啟動 [整合 - 多個提供者單一登入安裝程式] 外掛程式︰

    a. 在左側導覽窗格中，從搜尋列搜尋 [系統定義] 區段，然後按一下 [外掛程式]。

    ![啟動外掛程式](./media/servicenow-tutorial/tutorial_servicenow_03.png "啟動外掛程式")

     b. 搜尋**整合 - 多個提供者單一登入安裝程式**。

     ![啟動外掛程式](./media/servicenow-tutorial/tutorial_servicenow_04.png "啟動外掛程式")

    c. 選取外掛程式。 按一下滑鼠右鍵並選取 [啟動/升級]。

    d. 按一下 [啟用] 按鈕。

1. 有兩種方式可分別對 **ServiceNow** 進行自動和手動設定。

1. 若要自動設定 **ServiceNow**，請遵循下列步驟

    a. 返回 Azure 入口網站中的 [ServiceNow] 單一登入頁面。

    b. 我們提供 ServiceNow 單鍵設定服務，也就是讓 Azure AD 自動設定 ServiceNow，以便進行 SAML 型驗證。 若要啟用這項服務，請移至 [ServiceNow 設定] 區段，按一下 [設定 ServiceNow] 以開啟 [設定單一登入] 視窗。

    ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_configure.png)

    c. 在 [設定單一登入] 表單中輸入 ServiceNow 執行個體名稱、管理員使用者名稱和管理員密碼，然後按一下 [立即設定]。 請注意，提供的系統管理員使用者名稱必須在 ServiceNow 中指派 **security_admin** 角色，才能運作。 否則，若要手動設定 ServiceNow 以將 Azure AD 當作 SAML 識別提供者，請按一下 [手動設定單一登入]，然後從 [快速參考] 區段複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定應用程式 URL](./media/servicenow-tutorial/configure.png "設定應用程式 URL")

    d. 以系統管理員身分登入您的 ServiceNow 應用程式。

    e. 在自動設定中，所有的必要設定都會在 **ServiceNow** 端上設定，但依預設不會啟用 [X.509 憑證]。 您必須以手動方式將其對應至您在 ServiceNow 中的身分識別提供者。 請遵循下列步驟執行相同作業：
    
    * 在左側的導覽窗格中，按一下 [多重提供者 SSO] 下的 [身分識別提供者]。

      ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_07.png "設定單一登入")

    * 按一下自動產生的身分識別提供者

      ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_08.png "設定單一登入")

    * 向下捲動至 [X.509 憑證] 區段。 選取 [編輯]。

      ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_09.png "設定單一登入")
    
    * 選取憑證，然後按一下向右箭號圖示，以新增憑證

      ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_11.png "設定單一登入")

    * 按一下 [檔案] 。

    * 在頁面右上角按一下 [啟動]。

1. 若要手動設定 **ServiceNow**，請遵循下列步驟

1. 以系統管理員身分登入您的 ServiceNow 應用程式。

1. 在左側導覽窗格中，從搜尋列搜尋 [多重提供者 SSO] 區段，然後按一下 [屬性]。

    ![設定應用程式 URL](./media/servicenow-tutorial/tutorial_servicenow_06.png "設定應用程式 URL")

1. 在 [多個提供者 SSO 內容]  對話方塊上，執行下列步驟：

    ![設定應用程式 URL](./media/servicenow-tutorial/ic7694981.png "設定應用程式 URL")

    a. 針對 [啟用多個提供者 SSO]，選取 [是]。

    b. 針對 [啟用從所有識別提供者將使用者自動匯入使用者資料表]，選取 [是]。

    c. 針對 [啟用偵錯記錄以供多個提供者 SSO 整合]，選取 [是]。

    d. 在 [使用者資料表上的欄位...] 文字方塊中，輸入 **user_name**。

    e. 按一下 [檔案] 。

1. 在左側導覽窗格中，從搜尋列搜尋 [多重提供者 SSO] 區段，然後按一下 [x509 憑證]。

    ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_05.png "設定單一登入")

1. 在 [X.509 憑證] 對話方塊中，按一下 [新增]。

    ![設定單一登入](./media/servicenow-tutorial/ic7694974.png "設定單一登入")

1. 在 [X.509 憑證]  對話方塊中，執行下列步驟：

    ![設定單一登入](./media/servicenow-tutorial/ic7694975.png "設定單一登入")

    a. 在 [名稱] 文字方塊中，輸入您的組態名稱 (例如：**TestSAML2.0**)。

    b. 選取 [使用中] 。

    c. 針對 [格式]，選取 [PEM]。

    d. 針對 [類型]，選取 [信任存放區憑證]。

    e. 在記事本中開啟您從 Azure 下載的 Base64 編碼憑證，將其內容複製到剪貼簿，然後貼到 [PEM 憑證] 文字方塊。

     f. 按一下 [提交] 。

1. 在左側的導覽窗格中，按一下 [識別提供者] 。

    ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_07.png "設定單一登入")

1. 在 [識別提供者] 對話方塊中，按一下 [新增]。

    ![設定單一登入](./media/servicenow-tutorial/ic7694977.png "設定單一登入")

1. 在 [識別提供者] 對話方塊中，按一下 [SAML2 更新 1？]。

    ![設定單一登入](./media/servicenow-tutorial/ic7694978.png "設定單一登入")

1. 在 [SAML2 Update1 內容] 對話方塊上，執行下列步驟：

    ![設定單一登入](./media/servicenow-tutorial/idp.png "設定單一登入")

    a. 在 [匯入識別提供者中繼資料]對話方塊中，選取 [URL] 選項。

    b. 輸入您從 Azure 入口網站複製的 [應用程式同盟中繼資料 URL]。

    c. 按一下 [匯入] 。

1. 這樣會讀取 IdP 中繼資料 URL 並填入所有欄位資訊。

    ![設定單一登入](./media/servicenow-tutorial/ic7694982.png "設定單一登入")

    a. 在 [名稱] 文字方塊中，輸入您的組態名稱 (例如：**SAML 2.0**)。
    
    b. 複製 [ServiceNow 首頁] 值，貼入 Azure 入口網站 [ServiceNow 網域和 URL] 區段的 [單一登入 URL] 文字方塊中。

    > [!NOTE]
    > 串連您的 [ServieNow 租用戶 URL] 和 **/navpage.do** (例如：`https://fabrikam.service-now.com/navpage.do`)，就是 ServiceNow 執行個體首頁。

    c. 複製 [實體識別碼/簽發者] 值，貼入 Azure 入口網站 [ServiceNow 網域和 URL] 區段的 [識別碼] 文字方塊中。

    d. 按一下 [進階] 。 在 [使用者欄位] 文字方塊中，輸入 **email** 或 **user_name**，這取決於會用哪一個欄位來唯一識別 ServiceNow 部署中的使用者。

    > [!NOTE]
    > 移至 Azure 入口網站的 [ServiceNow] > [屬性] > [單一登入] 區段，並將所要的欄位對應至 [nameidentifier] 屬性，即可設定 Azure AD 以發出 Azure AD 使用者識別碼 (使用者主體名稱) 或電子郵件地址做為 SAML 權杖中的唯一識別碼。 所選屬性儲存在 Azure AD 中的值 (例如使用者主體名稱) 必須符合所輸入欄位 (例如 user_name) 儲存在 ServiceNow 中的值

    e. [x509 憑證] 下方列出您在上一個步驟中建立的憑證。

    > [!NOTE]
    > 若未按一下測試連線按鈕，ServiceNow 即不允許啟動 Idp，若要執行相同作業，請遵循下列步驟。

1. 從您建立於組態中的新身分識別提供者中按一下功能表圖示，然後從清單中選取 [copy sys_id]

    ![設定單一登入](./media/servicenow-tutorial/ic7694992.png "設定單一登入")

1. 在左上方的搜尋方塊中搜尋 **sys_properties.list**，然後按 Enter 鍵。

    ![設定單一登入](./media/servicenow-tutorial/ic7694993.png "設定單一登入")

1. 按一下 [新增] 。

    ![設定單一登入](./media/servicenow-tutorial/ic7694994.png "設定單一登入")

1. 在 [系統屬性] 區段中，執行下列步驟：

    ![設定單一登入](./media/servicenow-tutorial/ic7694995.png "設定單一登入")

    a. 在名稱文字方塊中輸入 `glide.authenticate.sso.redirect.idp` 值。

    b. 在 [值] 文字方塊中，貼上您在前述步驟中複製的 copy sys_id 值。

    c. 選取 [私人]。

    d. 按一下 [提交] 。

1. 按一下 [新增] 。

    ![設定單一登入](./media/servicenow-tutorial/ic7694994.png "設定單一登入")

1. 在 [系統屬性] 區段中，執行下列步驟：

    ![設定單一登入](./media/servicenow-tutorial/ic7694996.png "設定單一登入")

    a. 在名稱文字方塊中輸入 `glide.authenticate.multisso.test.connection.mandatory` 值。

    b. 在 [值] 文字方塊中，輸入 **false**。

    c. 按一下 [提交] 。

1. 完成前述步驟後，現在您已可啟動新的身分識別提供者，且您的 SSO 也應可運作

> [!NOTE]
> 同時也請注意，您必須在新的 incognito 視窗測試新的 Idp 設定

### <a name="configure-azure-ad-single-sign-on-for-servicenow-express"></a>為 ServiceNow Express 設定 Azure AD 單一登入

1. 在 Azure 入口網站的 [ServiceNow] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_samlbase.png)

1. 在 [ServiceNow 網域與 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_url.png)

    a. 在 [登入 URL] 文字方塊中，以下列模式輸入值：`https://<instance-name>.service-now.com/navpage.do`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<instance-name>.service-now.com`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [ServiceNow 用戶端支援小組](https://www.servicenow.com/support/contact-support.html)以取得這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_certificates.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/servicenow-tutorial/tutorial_general_400.png)

1. 我們提供 ServiceNow 單鍵設定服務，也就是讓 Azure AD 自動設定 ServiceNow，以便進行 SAML 型驗證。 若要啟用這項服務，請移至 [ServiceNow 設定] 區段，按一下 [設定 ServiceNow] 以開啟 [設定單一登入] 視窗。

    ![設定單一登入](./media/servicenow-tutorial/tutorial_servicenow_configure.png)

1. 在 [設定單一登入] 表單中輸入 ServiceNow 執行個體名稱、管理員使用者名稱和管理員密碼，然後按一下 [立即設定]。 請注意，提供的系統管理員使用者名稱必須在 ServiceNow 中指派 **security_admin** 角色，才能運作。 否則，若要手動設定 ServiceNow 以將 Azure AD 當作 SAML 識別提供者，請按一下 [手動設定單一登入]，然後從 [快速參考] 區段複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定應用程式 URL](./media/servicenow-tutorial/configure.png "設定應用程式 URL")

1. 以系統管理員身分登入您的 ServiceNow Express 應用程式。

1. 在左側導覽窗格中按一下 [單一登入]。

    ![設定應用程式 URL](./media/servicenow-tutorial/ic7694980ex.png "設定應用程式 URL")

1. 在 [單一登入] 對話方塊中，按一下右上方的 [設定] 圖示，然後設定下列屬性︰

    ![設定應用程式 URL](./media/servicenow-tutorial/ic7694981ex.png "設定應用程式 URL")

    a. 將 [啟用多個提供者 SSO] 切換到右邊。
    
    b. 將 [啟用偵錯記錄以供多個提供者 SSO 整合] 切換到右邊。
    
    c. 在 [使用者資料表上的欄位...] 文字方塊中，輸入 **user_name**。

1. 在 [單一登入] 對話方塊上，按一下 [新增憑證]。

    ![設定單一登入](./media/servicenow-tutorial/ic7694973ex.png "設定單一登入")

1. 在 [X.509 憑證]  對話方塊中，執行下列步驟：

    ![設定單一登入](./media/servicenow-tutorial/ic7694975.png "設定單一登入")

    a. 在 [名稱] 文字方塊中，輸入您的組態名稱 (例如：**TestSAML2.0**)。

    b. 選取 [使用中] 。

    c. 針對 [格式]，選取 [PEM]。

    d. 針對 [類型]，選取 [信任存放區憑證]。

    e. 在記事本中開啟您從 Azure 入口網站下載的 Base64 編碼憑證，將其內容複製到剪貼簿，然後貼到 [PEM 憑證] 文字方塊。

    f. 按一下 [更新] 

1. 在 [單一登入] 對話方塊上，按一下 [新增 IdP]。

    ![設定單一登入](./media/servicenow-tutorial/ic7694976ex.png "設定單一登入")

1. 在 [新增識別提供者] 對話方塊的 [設定識別提供者] 之下，執行下列步驟：

    ![設定單一登入](./media/servicenow-tutorial/ic7694982ex.png "設定單一登入")

    a. 在 [名稱] 文字方塊中，輸入您的組態名稱 (例如：**SAML 2.0**)。

    b. 在 [識別提供者 URL] 欄位中，貼上您從 Azure 入口網站複製的 [識別提供者識別碼] 值。
    
    c. 在 [識別提供者的 AuthnRequest] 欄位中，貼上您從 Azure 入口網站複製的 [驗證要求 URL] 值。

    d. 在 [識別提供者的 SingleLogoutRequest] 欄位中，貼上您從 Azure 入口網站複製的 [單一登出服務 URL] 值

    e. 針對 [識別提供者憑證]，選取您在上一個步驟中建立的憑證。

1. 按一下 [進階設定]，然後在 [其他識別提供者屬性] 之下，執行下列步驟︰

    ![設定單一登入](./media/servicenow-tutorial/ic7694983ex.png "設定單一登入")

    a. 在 [IDP 的 SingleLogoutRequest 通訊協定繫結] 文字方塊中，輸入 **urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect**。

    b. 在 [名稱識別碼原則] 文字方塊中，輸入 **urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified**。

    c. 在 **AuthnContextClassRef 方法**中，輸入 `http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/password`。

    d. 取消選取 [建立 AuthnContextClass]。

1. 在 [其他服務提供者屬性] 之下，執行下列步驟︰

    ![設定單一登入](./media/servicenow-tutorial/ic7694984ex.png "設定單一登入")

    a. 在 [ServiceNow 首頁] 文字方塊中，輸入您的 ServiceNow 執行個體首頁的 URL。

    > [!NOTE]
    > 串連您的 [ServieNow 租用戶 URL] 和 **/navpage.do** (例如：`https://fabrikam.service-now.com/navpage.do`)，就是 ServiceNow 執行個體首頁。

    b. 在 [對象識別碼/核發者] 文字方塊中，輸入您的 ServiceNow 租用戶 URL。

    c. 在 [對象 URI] 文字方塊中，輸入您的 ServiceNow 租用戶 URL。

    d. 在 [時鐘誤差] 文字方塊中，輸入 **60**。

    e. 在 [使用者欄位] 文字方塊中，輸入 **email** 或 **user_name**，這取決於會用哪一個欄位來唯一識別 ServiceNow 部署中的使用者。

    > [!NOTE]
    > 移至 Azure 入口網站的 [ServiceNow] > [屬性] > [單一登入] 區段，並將所要的欄位對應至 [nameidentifier] 屬性，即可設定 Azure AD 以發出 Azure AD 使用者識別碼 (使用者主體名稱) 或電子郵件地址做為 SAML 權杖中的唯一識別碼。 所選屬性儲存在 Azure AD 中的值 (例如使用者主體名稱) 必須符合所輸入欄位 (例如 user_name) 儲存在 ServiceNow 中的值

    f. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/servicenow-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/servicenow-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/servicenow-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/servicenow-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-servicenow-test-user"></a>建立 ServiceNow 測試使用者

本節的目標是在 ServiceNow 中建立名為 Britta Simon 的使用者。 ServiceNow 支援自動使用者佈建，該功能預設為啟用。 您可以在[這裡](servicenow-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

> [!NOTE]
> 如果您需要手動建立使用者，請連絡 [ServiceNow 用戶端支援小組](https://www.servicenow.com/support/contact-support.html)

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 ServiceNow 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 ServiceNow，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [ServiceNow]。

    ![應用程式清單中的 ServiceNow 連結](./media/servicenow-tutorial/tutorial_servicenow_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在 [存取面板] 中按一下 [ServiceNow] 圖格時，應該會自動登入您的 ServiceNow 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](servicenow-provisioning-tutorial.md)


<!--Image references-->

[1]: ./media/servicenow-tutorial/tutorial_general_01.png
[2]: ./media/servicenow-tutorial/tutorial_general_02.png
[3]: ./media/servicenow-tutorial/tutorial_general_03.png
[4]: ./media/servicenow-tutorial/tutorial_general_04.png

[100]: ./media/servicenow-tutorial/tutorial_general_100.png

[200]: ./media/servicenow-tutorial/tutorial_general_200.png
[201]: ./media/servicenow-tutorial/tutorial_general_201.png
[202]: ./media/servicenow-tutorial/tutorial_general_202.png
[203]: ./media/servicenow-tutorial/tutorial_general_203.png
