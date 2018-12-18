---
title: 新功能 Azure AD 版本資訊 | Microsoft Docs
description: 了解 Azure Active Directory (Azure AD) 的新功能，例如最新的版本資訊、已知問題、錯誤 (bug) 修正、已被取代的功能，以及即將進行的變更。
services: active-directory
author: eross-msft
manager: mtillman
featureFlags:
- clicktale
ms.assetid: 06a149f7-4aa1-4fb9-a8ec-ac2633b031fb
ms.service: active-directory
ms.component: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: lizross
ms.reviewer: dhanyahk
ms.openlocfilehash: d39cfddc42ea0e03f6b0f6c8d1c0160839742518
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393905"
---
# <a name="whats-new-in-azure-active-directory"></a>Azure Active Directory 的新增功能？

>藉由將此 [URL](https://docs.microsoft.com/api/search/rss?search=%22whats%20new%20in%20azure%20active%20directory%22&locale=en-us/) 新增至您的 ![RSS 圖示](./media/whats-new/feed-icon-16x16.png) 摘要讀取程式，即可獲知何時要造訪此頁面來進行更新。

Azure AD 會持續不斷進行改進。 為了讓您隨時掌握最新的開發訊息，本文提供下列相關資訊：

- 最新版本
- 已知問題
- 錯誤修正
- 已被取代的功能
- 方案變更

此頁面會每月更新，因此請定期瀏覽。

---
## <a name="september-2018"></a>2018 年 9 月
 
### <a name="updated-administrator-role-permissions-for-dynamic-groups"></a>已更新動態群組的系統管理員角色權限

**類型：** 固定  
**服務類別：** 群組管理  
**產品功能：** 共同作業

我們已修正某個問題，讓特定系統管理員角色現在可以建立和更新動態成員資格規則，而不必成為群組的擁有者。

角色如下：

- 全域管理員或公司撰寫者

- Intune 服務管理員

- 使用者帳戶管理員

如需詳細資訊，請參閱[建立動態群組並檢查狀態](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-create-rule)

---

### <a name="simplified-single-sign-on-sso-configuration-settings-for-some-third-party-apps"></a>簡化了某些第三方應用程式的單一登入 (SSO) 組態設定

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** SSO

我們了解，因為每個應用程式的組態都有獨特的本質，所以為軟體即服務 (SaaS) 應用程式設定單一登入 (SSO) 並不容易。 我們已建置經過簡化的組態體驗，來為下列第三方 SaaS 應用程式自動填入 SSO 組態設定：

- Zendesk

- ArcGis Online

- Jamf Pro

若要開始使用這個一鍵式體驗，請移至應用程式的 **Azure 入口網站** > [SSO 組態] 頁面。 如需詳細資訊，請參閱 [SaaS 應用程式與 Azure Active Directory 的整合](https://docs.microsoft.com/azure/active-directory/saas-apps/tutorial-list)

---

### <a name="azure-active-directory---where-is-your-data-located-page"></a>[Azure Active Directory - 資料位於何處？] 頁面

**類型：** 新功能  
**服務類別：** 其他  
**產品功能：** GoLocal

從 [Azure Active Directory - 資料位於何處] 頁面選取公司的區域，以檢視是哪一個 Azure 資料中心裝載了所有 Azure AD 服務的 Azure AD 待用資料。 您可以根據貴公司區域的特定 Azure AD 服務來篩選資訊。

若要存取這項功能以及需要詳細資訊，請參閱 [Azure Active Directory - 資料位於何處](http://aka.ms/AADDataMap)。

---

### <a name="new-deployment-plan-available-for-the-my-apps-access-panel"></a>適用於 [我的應用程式] 存取面板的新部署方案

**類型：** 新功能  
**服務類別：** 我的應用程式  
**產品功能：** SSO

查看適用於 [我的應用程式] 存取面板的新部署方案 (http://aka.ms/deploymentplans)。
[我的應用程式] 存取面板可讓使用者在單一位置尋找及存取其應用程式。 此入口網站也提供使用者自助服務的機會，例如要求存取應用程式和群組，或代表其他人管理這些資源的存取權。

如需詳細資訊，請參閱[什麼是「我的應用程式」入口網站？](https://docs.microsoft.com/azure/active-directory/user-help/active-directory-saas-access-panel-introduction)

---

### <a name="new-troubleshooting-and-support-tab-on-the-sign-ins-logs-page-of-the-azure-portal"></a>Azure 入口網站的 [登入記錄] 頁面上有新的 [疑難排解和支援] 索引標籤

**類型：** 新功能  
**服務類別：** 報告  
**產品功能：** 監視和報告

Azure 入口網站的 [登入] 頁面上有新的 [疑難排解和支援] 索引標籤，其目的是要協助系統管理員和支援工程師針對與 Azure AD 登入相關的問題進行疑難排解。這個新的索引標籤會提供錯誤碼、錯誤訊息和補救建議 (如果有的話) 來協助您解決問題。 如果您無法解決問題，我們也提供了新的辦法供您使用**複製到剪貼簿**體驗來新建支援票證，其會在支援票證的記錄檔中填入 [要求識別碼] 和 [日期 (UTC)] 欄位。  

![具有新索引標籤的登入記錄](media/whats-new/troubleshooting-and-support.png)

---

### <a name="enhanced-support-for-custom-extension-properties-used-to-create-dynamic-membership-rules"></a>針對用來建立動態成員資格規則的自訂擴充屬性強化其支援

**類型：** 已變更的功能  
**服務類別：** 群組管理  
**產品功能：** 共同作業

經由此更新，您現在可以從動態使用者群組規則建立器按一下 [取得自訂擴充屬性] 連結，輸入唯一的應用程式識別碼，然後接收在為使用者建立動態成員資格規則時所要使用的自訂擴充屬性完整清單。 您也可以重新整理這份清單，來針對該應用程式取得任何新的自訂擴充屬性。

如需針對動態成員資格規則使用自訂擴充屬性的詳細資訊，請參閱[擴充屬性和自訂擴充屬性](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-dynamic-membership#extension-properties-and-custom-extension-properties)

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>針對 Azure AD 應用程式型條件式存取之新的已核准用戶端應用程式

**類型：** 方案變更  
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護

下列應用程式已列入[已核准的用戶端應用程式](https://docs.microsoft.com/azure/active-directory/conditional-access/technical-reference.md#approved-client-app-requirement)清單：

- Microsoft To-Do

- Microsoft Stream

如需詳細資訊，請參閱

- [Azure AD 應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

---

### <a name="new-support-for-self-service-password-reset-from-the-windows-7881-lock-screen"></a>針對 Windows 7/8/8.1 鎖定畫面中的自助式密碼重設提供了新的支援

**類型：** 新功能  
**服務類別：** SSPR  
**產品功能：** 使用者驗證

在設定這項新功能之後，使用者就會在執行 Windows 7、Windows 8 或 Windows 8.1 的裝置中，從其 [鎖定] 畫面看到用以重設其密碼的連結。 藉由按一下該連結，系統就會引導使用者完成密碼重設流程，過程就和透過網頁瀏覽器所進行的一樣。

如需詳細資訊，請參閱[如何從 Windows 7、8 和 8.1 啟用密碼重設](https://aka.ms/ssprforwindows78)

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>變更通知：授權碼將不再供重複使用 

**類型：** 方案變更  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證

從 2018 年 10 月 10 日開始，Azure AD 將不再接受將先前使用的驗證碼用於應用程式。 這項安全性變更有助於讓 Azure AD 與 OAuth 規格一致，且將會對 v1 和 v2 端點強制執行。

如果您的應用程式重複使用授權碼取得多個資源的權杖，建議您先使用授權碼取得重新整理權杖，再使用該重新整理權杖取得其他資源的額外權杖。 授權碼只能使用一次，但重新整理權杖可跨多個資源多次使用。 在 OAuth 程式碼流程期間嘗試重複使用驗證碼的任何應用程式，會收到 invalid_grant 錯誤。

>[!Note]
>為了盡量避免應用程式損壞，依賴這個模式且每天會有超過 10 個登入的應用程式，已獲得例外狀況。

針對這項變更以及其他與通訊協定相關的變更，請參閱[新驗證功能的完整清單](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes)。

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---september-2018"></a>Azure AD 應用程式資源庫中有新的同盟應用程式可用 - 2018 年 9 月

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合
 
我們已在 2018 年 9 月將下列這 16 個提供同盟支援的全新應用程式新增至應用程式資源庫：

[Uberflip](https://docs.microsoft.com/azure/active-directory/saas-apps/uberflip-tutorial)、[Comeet Recruiting Software](https://docs.microsoft.com/azure/active-directory/saas-apps/comeetrecruitingsoftware-tutorial)、[Workteam](https://docs.microsoft.com/azure/active-directory/saas-apps/workteam-tutorial)、[ArcGIS Enterprise](https://docs.microsoft.com/azure/active-directory/saas-apps/arcgisenterprise-tutorial)、[Nuclino](https://docs.microsoft.com/azure/active-directory/saas-apps/nuclino-tutorial)、[JDA Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/jdacloud-tutorial)、[Snowflake](https://docs.microsoft.com/azure/active-directory/saas-apps/snowflake-tutorial)、NavigoCloud、[Figma](https://docs.microsoft.com/azure/active-directory/saas-apps/figma-tutorial)、join.me、[ZephyrSSO](https://docs.microsoft.com/azure/active-directory/saas-apps/zephyrsso-tutorial)、[Silverback](https://docs.microsoft.com/azure/active-directory/saas-apps/silverback-tutorial)、Riverbed Xirrus EasyPass、[Rackspace SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/rackspacesso-tutorial)、Enlyft SSO for Azure、SurveyMonkey、[Convene](https://docs.microsoft.com/azure/active-directory/saas-apps/convene-tutorial)、[dmarcian](https://docs.microsoft.com/azure/active-directory/saas-apps/dmarcian-tutorial)

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。 如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://aka.ms/azureadapprequest)。

---

### <a name="support-for-additional-claims-transformations-methods"></a>支援其他宣告轉換方法

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** SSO

我們已導入新的宣告轉換方法 ToLower() 和 ToUpper()，您可從以 SAML 為基礎的 [單一登入設定] 頁面套用至 SAML 權杖。

如需詳細資訊，請參閱[如何針對 Azure AD 中的企業應用程式自訂 SAML 權杖中發出的宣告](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization)

---

### <a name="updated-saml-based-app-configuration-ui-preview"></a>經過更新的 SAML 型應用程式設定 UI (預覽)

**類型：** 已變更的功能  
**服務類別：** 企業應用程式  
**產品功能：** SSO

在經過更新的 SAML 型應用程式設定 UI 中，您會獲得：

- 經過更新的 SAML 型應用程式設定逐步解說體驗。

- 更加了解設定中少了什麼或有何錯誤。

- 能夠新增多個用於到期憑證通知的電子郵件地址。

- 新的宣告轉換方法 ToLower() 和 ToUpper() 等等。

- 有辦法為企業應用程式上傳您自己的權杖簽署憑證。

- 有辦法設定 SAML 應用程式的 NameID 格式以及將 NameID 值設定為目錄擴充。

若要開啟此更新後的檢視，請從 [單一登入] 頁面頂端按一下 [試用新的體驗] 連結。 如需詳細資訊，請參閱[教學課程：為應用程式設定透過 Azure Active Directory 進行的 SAML 型單一登入](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-single-sign-on-portal)。

---

## <a name="august-2018"></a>2018 年 8 月

### <a name="changes-to-azure-active-directory-ip-address-ranges"></a>Azure Active Directory IP 位址範圍的變更

**類型：** 方案變更  
**服務類別：** 其他  
**產品功能：** 平台

我們正在將更大的 IP 範圍引入 Azure AD 中，這表示如果您已為您的防火牆、路由器或網路安全性群組設定 Azure AD IP 位址範圍，則必須進行更新。 我們會完成此更新，因此您不需要在 Azure AD 新增端點時，再次變更防火牆、路由器或網路安全群組的 IP 範圍組態。 

接下來的兩個月，網路流量會移至這些新的範圍。 若要繼續使用不中斷的服務，您必須在 2018 年 9 月 10 日之前，將這些更新值新增至您的 IP 位址：

- 20.190.128.0/18 

- 40.126.0.0/18 

我們強烈建議您，在所有網路流量都移至新的範圍之前，不要移除舊的 IP 位址範圍。 若要了解有關移動的更新及可移除舊範圍的時機，請參閱 [Office 365 URL 與 IP 位址範圍](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)。

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>變更通知：授權碼將不再供重複使用 

**類型：** 方案變更  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證

從 2018 年 10 月 10 日開始，Azure AD 將不再接受將先前使用的驗證碼用於應用程式。 這項安全性變更有助於讓 Azure AD 與 OAuth 規格一致，且將會對 v1 和 v2 端點強制執行。

如果您的應用程式重複使用授權碼取得多個資源的權杖，建議您先使用授權碼取得重新整理權杖，再使用該重新整理權杖取得其他資源的額外權杖。 授權碼只能使用一次，但重新整理權杖可跨多個資源多次使用。 在 OAuth 程式碼流程期間嘗試重複使用驗證碼的任何應用程式，會收到 invalid_grant 錯誤。

>[!Note]
>為了盡量避免應用程式損壞，依賴這個模式且每天會有超過 10 個登入的應用程式，已獲得例外狀況。

針對這項變更以及其他與通訊協定相關的變更，請參閱[新驗證功能的完整清單](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes)。
 
---

### <a name="converged-security-info-management-for-self-service-password-sspr-and-multi-factor-authentication-mfa"></a>適用於自助式密碼 (SSPR) 和 Multi-Factor Authentication ( MFA) 的融合式安全性資訊管理

**類型：** 新功能  
**服務類別：** SSPR  
**產品功能：** 使用者驗證
 
這項新功能可協助人員以單一位置和相同方式管理其 SSPR 和 MFA 的安全性資訊 (例如，電話號碼、行動應用程式等等)；不同於先前必須在兩個不同的位置管理。

此融合式體驗也適用於使用 SSPR 或 MFA 的人員。 此外，如果您的組織未強制執行 MFA 或 SSPR 註冊，人員仍可從「我的應用程式」入口網站註冊您的組織所允許的任何 MFA 或 SSPR 安全性資訊方法。

這是可選擇加入的公開預覽。 系統管理員可以為租用戶中選定的群組或所有使用者開啟新體驗 (如有需要)。 如需融合式體驗的詳細資訊，請參閱[融合式體驗部落格](https://cloudblogs.microsoft.com/enterprisemobility/2018/08/06/mfa-and-sspr-updates-now-in-public-preview/)

---

### <a name="new-http-only-cookies-setting-in-azure-ad-application-proxy-apps"></a>Azure AD 應用程式 Proxy 應用程式中新的僅限 HTTP Cookie 設定

**類型：** 新功能  
**服務類別：** 應用程式 Proxy  
**產品功能：** 存取控制

您的應用程式 Proxy 應用程式中有新的設定，名為「僅限 HTTP Cookies」。 此設定有助於提供更高的安全性，因為它可在應用程式 Proxy 存取和工作階段 Cookie 的 HTTP 回應標頭中加入 HTTPOnly 旗標，以禁止透過用戶端指令碼存取 Cookie，並進一步防止複製或修改 Cookie 之類的動作。 雖然過去並未使用此旗標，但您的 Cookie 一向都會加密，並使用 SSL 連線進行傳輸，以利防止不當的修改。

此設定與使用 ActiveX 控制項的應用程式不相容，例如遠端桌面。 面臨此情況時，建議您關閉這項設定。

如需「僅限 HTTP Cookie」設定的詳細資訊，請參閱[使用 Azure AD 應用程式 Proxy 發佈應用程式](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-publish-azure-portal)。

---

### <a name="privileged-identity-management-pim-for-azure-resources-supports-management-group-resource-types"></a>適用於 Azure 資源的 Privileged Identity Management (PIM) 支援「管理群組」資源類型

**類型：** 新功能  
**服務類別：** Privileged Identity Management  
**產品功能：** Privileged Identity Management
 
Just-In-Time 的啟用和指派設定現已可套用至「管理群組」資源類型，一如您對訂用帳戶、資源群組和資源 (例如 VM、App Service 等) 所做的一般。 此外，任何人只要具備為管理群組提供系統管理員存取權的角色，都可以在 PIM 中探索及管理該項資源。

如需 PIM 和 Azure 資源的詳細資訊，請參閱[使用 Privileged Identity Management 探索與管理 Azure 資源](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-resource-roles-discover-resources)
 
---

### <a name="application-access-preview-provides-faster-access-to-the-azure-ad-portal"></a>「應用程式存取」(預覽) 可加快存取 Azure AD 入口網站的速度

**類型：** 新功能  
**服務類別：** Privileged Identity Management  
**產品功能：** Privileged Identity Management
 
目前，在啟用使用 PIM 的角色時，權限可能要 10 分鐘以上才會生效。 如果您選擇使用目前處於公開預覽狀態的「應用程式存取」，系統管理員將可在啟用要求完成後隨即存取 Azure AD 入口網站。

目前，「應用程式存取」僅支援 Azure AD 入口網站體驗和 Azure 資源。 如需 PIM 和「應用程式存取」的詳細資訊，請參閱[什麼是 Azure AD Privileged Identity Management？](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-configure)
 
---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---august-2018"></a>Azure AD 應用程式資源庫中有新的同盟應用程式可用 - 2018 年 8 月

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合
 
我們已在 2018 年 8 月將下列這 16 個提供同盟支援的全新應用程式新增至應用程式資源庫：

[Hornbill](https://docs.microsoft.com/azure/active-directory/saas-apps/hornbill-tutorial)、[Bridgeline Unbound](https://docs.microsoft.com/azure/active-directory/saas-apps/bridgelineunbound-tutorial)、[Sauce Labs - 行動和 Web 測試](https://docs.microsoft.com/azure/active-directory/saas-apps/saucelabs-mobileandwebtesting-tutorial)、[Meta Networks Connector](https://docs.microsoft.com/azure/active-directory/saas-apps/metanetworksconnector-tutorial)、[Way We Do](https://docs.microsoft.com/azure/active-directory/saas-apps/waywedo-tutorial)、[Spotinst](https://docs.microsoft.com/azure/active-directory/saas-apps/spotinst-tutorial)、[ProMaster (by Inlogik)](https://docs.microsoft.com/azure/active-directory/saas-apps/promaster-tutorial)、SchoolBooking、[4me](https://docs.microsoft.com/azure/active-directory/saas-apps/4me-tutorial)、[Dossier](https://docs.microsoft.com/azure/active-directory/saas-apps/DOSSIER-tutorial)、[N2F - Expense reports](https://docs.microsoft.com/azure/active-directory/saas-apps/n2f-expensereports-tutorial)、[Comm100 Live Chat](https://docs.microsoft.com/azure/active-directory/saas-apps/comm100livechat-tutorial)、[SafeConnect](https://docs.microsoft.com/azure/active-directory/saas-apps/safeconnect-tutorial)、[ZenQMS](https://docs.microsoft.com/azure/active-directory/saas-apps/zenqms-tutorial)、[eLuminate](https://docs.microsoft.com/azure/active-directory/saas-apps/eluminate-tutorial)、[Dovetale](https://docs.microsoft.com/azure/active-directory/saas-apps/dovetale-tutorial)。

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。 如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://aka.ms/azureadapprequest)。

---

### <a name="native-tableau-support-is-now-available-in-azure-ad-application-proxy"></a>Azure AD 應用程式 Proxy 現已提供原生 Tableau 支援

**類型：** 已變更的功能  
**服務類別：** 應用程式 Proxy  
**產品功能：** 存取控制

在我們將預先驗證通訊協定從 OpenID Connect 更新為 OAuth 2.0 Code Grant 通訊協定後，您無須再進行任何額外的設定，即可搭配使用 Tableau 與應用程式 Proxy。 此一通訊協定變更也有助於應用程式 Proxy 更妥善地支援更現代化的應用程式，因為此後只需使用 HTTP 重新導向即可，而這在 JavaScript 和 HTML 標記中通常都可受到支援。

如需與我們的 Tableau 原生支援有關的詳細資訊，請參閱 [Azure AD 應用程式 Proxy 現已具備原生 Tableau 支援](https://blogs.technet.microsoft.com/applicationproxyblog/2018/08/14/azure-ad-application-proxy-now-with-native-tableau-support)。

---

### <a name="new-support-to-add-google-as-an-identity-provider-for-b2b-guest-users-in-azure-active-directory-preview"></a>在 Azure Active Directory (預覽) 中新增 Google 供 B2B 來賓使用者作為識別提供者的新支援

**類型：** 新功能  
**服務類別：** B2B  
**產品功能：** B2B/B2C

藉由在組織中設定與 Google 的同盟，您可以讓受邀的 Gmail 使用者使用其現有的 Google 帳戶登入您的共用應用程式和資源，而不需要建立個人 Microsoft 帳戶 (MSA) 或 Azure AD 帳戶。

這是可選擇加入的公開預覽。 如需 Google 同盟的詳細資訊，請參閱[將 Google 新增為 B2B 來賓使用者的識別提供者](https://docs.microsoft.com/azure/active-directory/b2b/google-federation)。

---

## <a name="july-2018"></a>2018 年 7 月

### <a name="improvements-to-azure-active-directory-email-notifications"></a>Azure Active Directory 電子郵件通知的改進功能

**類型：** 已變更的功能  
**服務類別：** 其他  
**產品功能：** 身分識別生命週期管理
 
Azure Active Directory (Azure AD) 電子郵件現在提供更新的設計，以及從下列服務傳送時，對寄件者電子郵件地址和寄件者顯示名稱進行的變更：
 
- Azure AD 存取權檢閱
- Azure AD Connect Health 
- Azure AD Identity Protection 
- Azure AD 特殊權限身分識別管理
- 企業應用程式過期憑證通知
- 企業應用程式佈建服務通知
 
電子郵件通知將會從下列電子郵件地址和顯示名稱傳送：

- 電子郵件地址：azure-noreply@microsoft.com
- 顯示名稱：Microsoft Azure
 
如需一些新電子郵件設計的範例和詳細資訊，請參閱 [Azure AD PIM 中的電子郵件通知](https://go.microsoft.com/fwlink/?linkid=2005832) \(英文\)。

---

### <a name="azure-ad-activity-logs-are-now-available-through-azure-monitor"></a>Azure AD 活動記錄現已可透過 Azure 監視器取得

**類型：** 新功能  
**服務類別：** 報告  
**產品功能：** 監視和報告

Azure AD 活動記錄現在已可在 Azure 監視器的公開預覽 (Azure 的全平台監視服務) 中取得。 除了這些改善，Azure 監視器還會為您提供長期保留與無縫整合：

- 藉由將記錄檔路由傳送到您自己的 Azure 儲存體帳戶來進行的長期保留。

- 無縫的 SIEM 整合，而不需您撰寫或維護自訂指令碼。

- 與您自己的自訂解決方案、分析工具或事件管理解決方案進行的無縫整合。

如需這些新功能的詳細資訊，請參閱我們的部落格 [Azure 監視器診斷中的 Azure AD 活動記錄目前處於公開預覽狀態](https://cloudblogs.microsoft.com/enterprisemobility/2018/07/26/azure-ad-activity-logs-in-azure-monitor-diagnostics-now-in-public-preview/) \(英文\) 與 [Azure 監視器 (預覽) 中的 Azure Active Directory 活動記錄](https://docs.microsoft.com/azure/active-directory/reporting-azure-monitor-diagnostics-overview)文件。

---

### <a name="conditional-access-information-added-to-the-azure-ad-sign-ins-report"></a>新增至 Azure AD 登入報告的條件式存取資訊

**類型：** 新功能  
**服務類別：** 報告  
**產品功能：** 身分識別安全性與保護
 
此更新可讓您查看當使用者登入時要評估哪些原則以及原則結果。 此外，報告現在包含使用者所使用的用戶端應用程式類型，讓您可以識別舊版通訊協定流量。 現在也會針對相互關聯識別碼搜尋報告項目，其可在使用者所看到的錯誤訊息中找到，而且可用來識別相符的登入要求並進行疑難排解。

---

### <a name="view-legacy-authentications-through-sign-ins-activity-logs"></a>透過登入活動記錄來檢視舊版驗證

**類型：** 新功能  
**服務類別：** 報告  
**產品功能：** 監視和報告
 
隨著登入活動記錄中 **Client App** 欄位的導入，客戶現在可看見使用舊版驗證的使用者。 客戶可使用登入 MS Graph API 來存取此資訊，或透過 Azure AD 入口網站中的登入活動記錄使用 **Client App** 控制項來篩選舊版驗證。 如需更多詳細資料，請查看文件。

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---july-2018"></a>Azure AD 應用程式庫中現在提供新的同盟應用程式 - 2018 年 7 月

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合
 
我們已在 2018 年 7 月將下列這 16 個提供同盟支援的全新應用程式新增至應用程式庫：

[Innovation Hub](https://docs.microsoft.com/azure/active-directory/saas-apps/innovationhub-tutorial)、[Leapsome](https://docs.microsoft.com/azure/active-directory/saas-apps/leapsome-tutorial)、[Certain Admin SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/certainadminsso-tutorial)、PSUC Staging、[iPass SmartConnect](https://docs.microsoft.com/azure/active-directory/saas-apps/ipasssmartconnect-tutorial)、[Screencast-O-Matic](https://docs.microsoft.com/azure/active-directory/saas-apps/screencast-tutorial)、PowerSchool Unified Classroom、[Eli Onboarding](https://docs.microsoft.com/azure/active-directory/saas-apps/elionboarding-tutorial)、[Bomgar Remote Support](https://docs.microsoft.com/azure/active-directory/saas-apps/bomgarremotesupport-tutorial)、[Nimblex](https://docs.microsoft.com/azure/active-directory/saas-apps/nimblex-tutorial)、[Imagineer WebVision](https://docs.microsoft.com/azure/active-directory/saas-apps/imagineerwebvision-tutorial)、[Insight4GRC](https://docs.microsoft.com/azure/active-directory/saas-apps/insight4grc-tutorial)、[SecureW2 JoinNow Connector](https://docs.microsoft.com/azure/active-directory/saas-apps/securejoinnow-tutorial)、[Kanbanize](https://review.docs.microsoft.com/azure/active-directory/saas-apps/kanbanize-tutorial)、[SmartLPA](https://review.docs.microsoft.com/azure/active-directory/saas-apps/smartlpa-tutorial)、[Skills Base](https://docs.microsoft.com/azure/active-directory/saas-apps/skillsbase-tutorial)

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。 如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://aka.ms/azureadapprequest)。

---
 
### <a name="new-user-provisioning-saas-app-integrations---july-2018"></a>新的使用者佈建 SaaS 應用程式整合 - 2018 年 7 月

**類型：** 新功能  
**服務類別：** 應用程式佈建  
**產品功能：** 協力廠商整合
 
Azure AD 可讓您自動化在 SaaS 應用程式 (例如 Dropbox、Salesforce、ServiceNow 等等) 中建立、維護和移除使用者身分識別的作業。 我們已在 2018 年 7 月，於 Azure AD 應用程式庫中，為以下應用程式新增了使用者佈建支援：

- [Cisco Spark](https://docs.microsoft.com/azure/active-directory/saas-apps/cisco-spark-provisioning-tutorial)

- [Cisco WebEx](https://docs.microsoft.com/azure/active-directory/saas-apps/cisco-webex-provisioning-tutorial)

- [Bonusly](https://docs.microsoft.com/azure/active-directory/saas-apps/bonusly-provisioning-tutorial)

如需 Azure AD 資源庫中支援使用者佈建的所有應用程式清單，請參閱 [SaaS 應用程式與 Azure Active Directory 的整合](https://aka.ms/appstutorial)。

---

### <a name="connect-health-for-sync---an-easier-way-to-fix-orphaned-and-duplicate-attribute-sync-errors"></a>適用於同步處理的 Connect Health - 這個更簡單的方式可用來修正孤立和重複屬性同步處理錯誤

**類型：** 新功能  
**服務類別：** AD Connect  
**產品功能：** 監視和報告
 
Azure AD Connect Health 導入了自助式補救，以協助您醒目提示並修正同步處理錯誤。 此功能會對重複屬性同步處理錯誤進行疑難排解，並修正來自 Azure AD 的孤立物件。 此診斷具有下列好處：

- 縮減重複屬性同步處理錯誤，提供特定的修正

- 針對專用的 Azure AD 案例套用修正，在單一步驟中解決錯誤

- 無須升級或設定即可啟用開啟並使用此功能

如需詳細資訊，請參閱[對重複屬性同步處理錯誤進行診斷和修復](https://docs.microsoft.com/azure/active-directory/connect-health/active-directory-aadconnect-health-diagnose-sync-errors)

---

### <a name="visual-updates-to-the-azure-ad-and-msa-sign-in-experiences"></a>對於 Azure AD 和 MSA 登入體驗的視覺效果更新

**類型：** 已變更的功能  
**服務類別：** Azure AD  
**產品功能：** 使用者驗證

我們已更新 Microsoft 線上服務登入體驗的 UI，例如針對 Office 365 與 Azure。 此變更讓畫面變得更簡潔且更直接。 如需此變更的詳細資訊，請參閱[即將推出的 Azure AD 登入體驗改善](https://cloudblogs.microsoft.com/enterprisemobility/2018/04/04/upcoming-improvements-to-the-azure-ad-sign-in-experience/) \(英文\) 部落格。

---

### <a name="new-release-of-azure-ad-connect---july-2018"></a>Azure AD Connect 的新版本 - 2018 年 7 月

**類型：** 已變更的功能  
**服務類別：** 應用程式佈建  
**產品功能：** 身分識別生命週期管理

Azure AD Connect 的最新版本包括： 

- 錯誤 (bug) 修正和可支援性更新 

- Ping Federate 整合已正式運作

- 最新 SQL 2012 用戶端的更新 

如需此更新的詳細資訊，請參閱 [Azure AD Connect︰版本發行歷程記錄](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-version-history)

---

### <a name="updates-to-the-terms-of-use-tou-end-user-ui"></a>使用規定 (ToU) 使用者 UI 的更新

**類型：** 已變更的功能  
**服務類別：** 使用規定  
**產品功能：** 治理

我們正在更新 TOU 終端使用者 UI 中的接受字串。

**目前的文字。** 若要存取 [tenantName] 資源，您必須接受使用規定。<br>**新文字。** 若要存取 [tenantName] 資源，您必須閱讀使用規定。

**目前的文字：** 選擇接受即表示您接受上述所有使用規定。<br>**新文字：** 請按一下 [接受] 以確認您已閱讀並了解使用規定。

---
 
### <a name="pass-through-authentication-supports-legacy-protocols-and-applications"></a>傳遞驗證支援舊版通訊協定和應用程式

**類型：** 已變更的功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
傳遞驗證現在支援舊版通訊協定和應用程式。 現在完全支援下列限制：

- 使用者登入舊版 Office 用戶端應用程式 Office 2010 和 Office 2013，而不需新式驗證。

- 僅在 Exchange 混合式環境中的 Office 2010 上存取行事曆共用和空閒/忙碌資訊。

- 使用者不需新式驗證，即可登入商務用 Skype 用戶端應用程式。

- 使用者登入 PowerShell 1.0 版。

- 使用 iOS 安裝小幫手的 Apple 裝置註冊計劃 (Apple DEP)。 

---
 
### <a name="converged-security-info-management-for-self-service-password-reset-and-multi-factor-authentication"></a>適用於自助式密碼重設及 Multi-Factor Authentication 的融合式安全性資訊管理

**類型：** 新功能  
**服務類別：** SSPR  
**產品功能：** 使用者驗證

這項新功能可讓使用者在單一體驗中，針對自助式密碼重設 (SSPR) 與 Multi-Factor Authentication (MFA) 管理其安全性資訊 (例如，電話號碼、電子郵件地址、行動裝置應用程式等等)。 使用者將不再需要在兩個不同體驗中，針對 SSPR 和 MFA 註冊相同的安全性資訊。 這個新體驗也適用於具有 SSPR 或 MFA 的使用者。

如果組織未強制執行 MFA 或 SSPR 註冊，使用者就能透過 **My Apps** 入口網站註冊他們的安全性資訊。 使用者可以從這裡註冊已針對 MFA 或 SSPR 啟用的任何方法。 

這是可選擇加入的公開預覽。 系統管理員可以針對一組選取的使用者或租用戶中的所有使用者開啟新體驗 (如有需要)。

---
 
### <a name="use-the-microsoft-authenticator-app-to-verify-your-identity-when-you-reset-your-password"></a>當您重設密碼時，請使用 Microsoft Authenticator 應用程式來驗證您的身分識別

**類型：** 已變更的功能  
**服務類別：** SSPR  
**產品功能：** 使用者驗證

此功能可讓非系統管理員在使用來自 Microsoft Authenticator (或任何其他驗證器應用程式) 的通知或代碼重設密碼時驗證他們的身分識別。 在系統管理員開啟這個自助式密碼重設方法之後，已透過 aka.ms/mfasetup aka.ms/setupsecurityinfo 註冊行動裝置應用程式的使用者就能在重設密碼時，使用其行動裝置應用程式作為驗證方法。

行動裝置應用程式通知可以只開啟來作為需要兩個方法來重設密碼之原則的一部分。

---

## <a name="june-2018"></a>2018 年 6 月

### <a name="change-notice-security-fix-to-the-delegated-authorization-flow-for-apps-using-azure-ad-activity-logs-api"></a>變更注意事項：對使用 Azure AD 活動記錄 API 的應用程式委派授權流程進行安全性修正

**類型：** 方案變更  
**服務類別：** 報告  
**產品功能：** 監視和報告

由於我們強制執行了更穩固的安全性，因此必須對使用委派授權流程來存取 [Azure AD 活動記錄 API](https://aka.ms/aadreportsapi) 的應用程式變更權限。 這項變更會在 **2018 年 6 月 26 日**生效。

如果您有任何應用程式使用 Azure AD 活動記錄 API，請遵循下列步驟，以確保應用程式不會在變更執行後發生中斷。

**更新您的應用程式權限**

1. 登入 Azure 入口網站、選取 [Azure Active Directory]，然後選取 [應用程式註冊]。
2. 選取使用 Azure AD 活動記錄 API 的應用程式、選取 [設定]、選取 [必要權限]，然後選取 [Windows Azure Active Directory] API。
3. 在 [啟用存取] 刀鋒視窗的 [委派權限] 區域中，選取 [讀取目錄] 資料旁的方塊，然後選取 [儲存]。
4. 選取 [授與權限]，然後選取 [是]。
    
    >[!Note]
    >您必須是全域管理員，才能授與權限給應用程式。

如需詳細資訊，請參閱必要條件的 [授與權限](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-prerequisites-azure-portal#grant-permissions) 區域來存取 Azure AD 報告 API 文章。

---

### <a name="configure-tls-settings-to-connect-to-azure-ad-services-for-pci-dss-compliance"></a>將 TLS 設定為連線至 Azure AD 服務，以符合 PCI DSS 合規性

**類型：** 新功能  
**服務類別：** N/A  
**產品功能：** 平台

傳輸層安全性 (TLS) 是提供兩個通訊應用程式之間隱私權和資料完整性的通訊協定，是目前所使用最廣泛部署的安全性通訊協定。

[PCI 安全性標準委員會](https://www.pcisecuritystandards.org/)判定 TLS 和安全通訊端層 (SSL) 的早期版本必須停用，以利啟用更安全的全新應用程式通訊協定，此合規性自 **2018 年 6 月 30 日**起開始。 這項變更意味著如果連線到 Azure AD 服務並要求符合 PCI DSS，就必須停用 TLS 1.0。 TLS 有許多版本可用，但 TLS 1.2 是 Azure Active Directory 服務最新的可用版本。 若是結合使用用戶端/伺服器及瀏覽器/伺服器，強烈建議您直接使用 TLS 1.2。

過時的瀏覽器可能不支援較新的 TLS 版本，例如 TLS 1.2。 若要查看瀏覽器支援的 TLS 版本，請前往 [Qualys SSL 實驗室](https://www.ssllabs.com/)網站並按一下 [測試您的瀏覽器]。 建議您升級到最新版本的網頁瀏覽器，最好只啟用 TLS 1.2。

**透過瀏覽器啟用 TLS 1.2**

- **Microsoft Edge 和 Internet Explorer (這兩者都使用 Internet Explorer 設定)**

    1. 開啟 [Internet Explorer]，然後選取 [工具]、 > [網際網路選項] > 、[進階]。
    2. 在 [安全性] 區域中，選取 [使用 TLS 1.2]，然後選取 [確定]。
    3. 關閉所有瀏覽器視窗，並重新啟動 Internet Explorer。 

- **Google Chrome**

    1. 開啟 Google Chrome、在位址列輸入 *chrome://settings/*，然後按 **Enter** 鍵。
    2. 展開 [進階] 選項、前往 [系統] 區域，然後選取 [開啟 Proxy 設定]。
    3. 在 [網際網路內容] 方塊中，選取 [進階] 索引標籤、前往 [安全性] 區域、選取 [使用 TLS 1.2]，然後選取 [確定]。
    4. 關閉所有瀏覽器視窗，並重新啟動 Google Chrome。

- **Mozilla Firefox**

    1. 開啟 Firefox、在位址列輸入 *about:config*，然後按 **Enter** 鍵。
    2. 搜尋字詞 *TLS*，然後選取 [security.tls.version.max] 項目。
    3. 將值設為 **3** 以強制瀏覽器使用最新版本 TLS 1.2，然後選取 [確定]。

        >[!NOTE]
        >Firefox 60.0 版本支援 TLS 1.3，因此也可以將 security.tls.version.max 值設為 **4**。

    4. 關閉所有瀏覽器視窗，並重新啟動 Mozilla Firefox。

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---june-2018"></a>Azure AD 應用程式庫中有新的同盟應用程式可用 - 2018 年 6 月

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合
 
我們已在 2018 年 6 月將下列這 15 個提供同盟支援的全新應用程式新增至應用程式庫：

[Skytap](https://docs.microsoft.com/azure/active-directory/active-directory-saas-skytap-tutorial)、[Settling music](https://docs.microsoft.com/azure/active-directory/active-directory-saas-settlingmusic-tutorial)、[SAML 1.1 Token enabled LOB App](https://docs.microsoft.com/azure/active-directory/active-directory-saas-saml-tutorial)、[Supermood](https://docs.microsoft.com/azure/active-directory/active-directory-saas-supermood-tutorial)、[Autotask](https://docs.microsoft.com/azure/active-directory/active-directory-saas-autotaskendpointbackup-tutorial)、[Endpoint Backup](https://docs.microsoft.com/azure/active-directory/active-directory-saas-autotaskendpointbackup-tutorial)、[Skyhigh Networks](https://docs.microsoft.com/azure/active-directory/active-directory-saas-skyhighnetworks-tutorial)、Smartway2、[TonicDM](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tonicdm-tutorial)、[Moconavi](https://docs.microsoft.com/azure/active-directory/active-directory-saas-moconavi-tutorial)、[Zoho One](https://docs.microsoft.com/azure/active-directory/active-directory-saas-zohoone-tutorial)、[SharePoint on-premises](https://docs.microsoft.com/azure/active-directory/active-directory-saas-sharepoint-on-premises-tutorial)、[ForeSee CX Suite](https://docs.microsoft.com/azure/active-directory/active-directory-saas-foreseecxsuite-tutorial)、[Vidyard](https://docs.microsoft.com/azure/active-directory/active-directory-saas-vidyard-tutorial)、[ChronicX](https://docs.microsoft.com/azure/active-directory/active-directory-saas-chronicx-tutorial)

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。 如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing)。 

---

### <a name="azure-ad-password-protection-is-available-in-public-preview"></a>Azure AD 密碼保護可在公開預覽時使用

**類型：** 新功能  
**服務類別：** 身分識別保護  
**產品功能：** 使用者驗證

使用 Azure AD 密碼保護可協助排除所在環境中容易猜到的密碼。 排除這些密碼可協助降低密碼噴灑 (password spray) 類型攻擊危害的風險。

具體來說，Azure AD 密碼保護可協助您：

- 保護 Azure AD 和 Windows Server Active Directory (AD) 中的組織帳戶。 
- 避免使用者使用超過 500 個最常用密碼清單中的密碼，以及這些密碼超過 100 萬個字元的替換變化。
- 針對 Azure AD 和內部部署 Windows Server AD，從 Azure AD 入口網站中的單一位置管理 Azure AD 密碼保護。

如需 Azure AD 密碼保護的詳細資訊，請參閱[排除組織中的不當密碼](https://aka.ms/aadpasswordprotectiondocs)。

---

### <a name="new-all-guests-conditional-access-policy-template-created-during-terms-of-use-tou-creation"></a>在建立使用規定 (ToU) 期間所擬定的全新「所有來賓」條件式存取原則範本

**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 治理

在建立使用規定 (ToU) 期間，也會針對「所有來賓」和「所有應用程式」擬定全新的條件式存取原則範本。 這個全新原則範本適用於新建立的 ToU，簡化了來賓建立和強制程序。

如需詳細資訊，請參閱 [Azure Active Directory 使用規定功能](https://docs.microsoft.com/azure/active-directory/active-directory-tou)。

---

### <a name="new-custom-conditional-access-policy-template-created-during-terms-of-use-tou-creation"></a>在建立使用規定 (ToU) 期間所擬定的全新「自訂」條件式存取原則範本

**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 治理

在建立使用規定 (ToU) 期間，也會擬定全新的「自訂」條件式存取原則範本。 這個全新原則範本可讓您建立 ToU，然後立即前往條件式存取原則建立刀鋒視窗，無須手動瀏覽入口網站。

如需詳細資訊，請參閱 [Azure Active Directory 使用規定功能](https://docs.microsoft.com/azure/active-directory/active-directory-tou)。

---

### <a name="new-and-comprehensive-guidance-about-deploying-azure-multi-factor-authentication"></a>關於部署 Azure Multi-Factor Authentication 的全新完整指導

**類型：** 新功能  
**服務類別：** 其他  
**產品功能：** 身分識別安全性與保護
 
我們針對如何在組織中部署 Azure Multi-Factor Authentication (MFA) 推出了全新的逐步指導。

如需檢視 MFA 部署指南，請前往 GitHub 上的[識別部署指南](https://aka.ms/DeploymentPlans)存放庫。 如需提供有關部署指南的意見反應，請使用[部署方案意見反應單](https://aka.ms/deploymentplanfeedback)。 如果對於部署指南有任何疑問，請於 [IDGitDeploy](mailto:idgitdeploy@microsoft.com) 聯絡我們。

---

### <a name="azure-ad-delegated-app-management-roles-are-in-public-preview"></a>Azure AD 委派的應用程式管理角色位於公開預覽

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 存取控制

管理員現在可以委派應用程式管理工作，而無須指派全域管理員角色。 全新角色和功能包括：

- **全新的標準 Azure AD 管理員角色：**

    - **應用程式系統管理員。** 授予管理所有應用程式各方面的權限，包括註冊、SSO 設定、應用程式指派和授權、應用程式 Proxy 設定及同意 (Azure AD 資源除外)。

    - **雲端應用程式系統管理員。** 授予所有應用程式系統管理員的能力，但應用程式 Proxy 除外，因為應用程式 Proxy 不提供內部存取。

    - **應用程式開發人員。** 授予建立應用程式註冊的權限，即使已關閉 [允許使用者註冊應用程式] 選項亦同。

- **擁有權 (設定每個應用程式註冊與每個企業應用程式，類似群組擁有權的程序：**
 
    - **應用程式註冊擁有者。** 授予管理自有應用程式註冊各方面之權限，包括應用程式資訊清單，以及新增其他擁有者。

    - **企業應用程式擁有者。** 授予管理自有企業應用程式許多方面的權限，包括 SSO 設定、應用程式指派及同意 (Azure AD 資源除外)。

如需公開預覽的詳細資訊，請參閱 [Azure AD 委派的應用程式管理角色位於公開預覽！](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/13/hallelujah-azure-ad-delegated-application-management-roles-are-in-public-preview/) 部落格。 如需有關角色和權限的詳細資訊，請參閱[在 Azure Active Directory 中指派系統管理員角色](https://docs.microsoft.com/azure/active-directory/active-directory-assign-admin-roles-azure-portal)。

---

## <a name="may-2018"></a>2018 年 5 月

### <a name="expressroute-support-changes"></a>ExpressRoute 支援變更

**類型：** 方案變更  
**服務類別：** 驗證 (登入)  
**產品功能：** 平台  

Azure Active Directory (Azure AD) 等這類 SaaS 供應項目旨在設計為適合直接透過網際網路使用，而不需要 ExpressRoute 或任何其他私用 VPN 通道。 因為這個緣故，在 **2018 年 8 月 1 日**，我們會對使用 Azure 公用對等互連和在 Microsoft 對等互連中使用 Azure 社群的 Azure AD 服務，停止支援 ExpressRoute。 受此變更影響的任何服務可能會注意到 Azure AD 流量逐漸從 ExpressRoute 移到網際網路。

雖然即將變更支援，但我們也知道在有些狀況下，您仍需要對驗證流量使用一組特定線路。 因為這個緣故，Azure AD 將會繼續支援使用 ExpressRoute 的每個租用戶 IP 範圍限制，以及針對包含「其他 Office 365 線上服務」社群的 Microsoft 對等互連，支援其中已存在的服務。 如果服務會受到影響，但您需要 ExpressRoute，則必須執行下列作業：

- **如果您位在 Azure 公用對等互連上。** 移至 Microsoft 對等互連，並註冊**其他的 Office 365 線上服務 (12076:5100)** 社群。 如需如何從 Azure 公用對等互連移至 Microsoft 對等互連的詳細資訊，請參閱[將公用對等互連移至 Microsoft 對等互連](https://docs.microsoft.com/azure/expressroute/how-to-move-peering)一文。

- **如果您在 Microsoft 對等互連上。** 註冊**其他 Office 365 線上服務 (12076:5100)** 社群。 如需路由需求的詳細資訊，請參閱 ExpressRoute 路由需求文章中的 [BGP 社群支援區段](https://docs.microsoft.com/azure/expressroute/expressroute-routing#bgp)。

如果您必須繼續使用特定線路，必須與您的 Microsoft 帳戶小組討論，了解如何取得使用**其他 Office 365 線上服務 (12076:5100)** 社群的授權。 由 MS Office 管理的檢閱面板會確認您是否需要這些線路，並確定您了解保留這些線路的技術含意。 未經授權的訂用帳戶若嘗試建立 Office 365 路由篩選，將會收到錯誤訊息。 
 
---

### <a name="microsoft-graph-apis-for-administrative-scenarios-for-tou"></a>Microsoft Graph API 適用於 TOU 的系統管理案例

**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 開發人員體驗
 
我們已新增 Microsoft Graph API 的 Azure AD 使用規定系統管理作業。 您可以建立、更新、刪除使用規定物件。

---

### <a name="add-azure-ad-multi-tenant-endpoint-as-an-identity-provider-in-azure-ad-b2c"></a>新增 Azure AD 多租用戶端點作為 Azure AD B2C 中的識別提供者

**類型：** 新功能  
**服務類別：** B2C - 取用者身分識別管理  
**產品功能：** B2B/B2C
 
您現在可以使用自訂原則，新增 Azure AD 通用端點作為 Azure AD B2C 中的識別提供者。 這使您能夠為登入您應用程式的所有 Azure AD 使用者提供單一進入點。 如需詳細資訊，請參閱 [Azure Active Directory B2C：允許使用者使用自訂原則登入多租用戶 Azure AD 識別提供者](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-setup-commonaad-custom)。

---

### <a name="use-internal-urls-to-access-apps-from-anywhere-with-our-my-apps-sign-in-extension-and-the-azure-ad-application-proxy"></a>使用內部 URL，透過我們的 My Apps 登入延伸模組與 Azure AD 應用程式 Proxy 從任何地方存取應用程式

**類型：** 新功能  
**服務類別：** 我的應用程式  
**產品功能：** SSO
 
使用者現在即使在公司網路外，也能使用適用於 Azure AD 的 My Apps Secure Sign-in 延伸模組，透過內部 URL 存取應用程式。 這適用於您使用 Azure AD 應用程式 Proxy 發行的所有應用程式，以及亦安裝存取面板瀏覽器延伸模組的所有瀏覽器。 URL 重新導向功能會在使用者登入延伸模組後自動啟用。 該延伸模組可在 [Edge](https://go.microsoft.com/fwlink/?linkid=845176)、[Chrome](https://go.microsoft.com/fwlink/?linkid=866367) 及 [Firefox](https://go.microsoft.com/fwlink/?linkid=866366) 上下載。

---
 
### <a name="azure-active-directory---data-in-europe-for-europe-customers"></a>Azure Active Directory - 歐洲客戶在歐洲中的資料

**類型：** 新功能  
**服務類別：** 其他  
**產品功能：** GoLocal

歐洲客戶必須將其資料保留在歐洲，而不得於歐洲資料中心外部複寫，以期符合隱私權與歐洲法律。 [本文](https://go.microsoft.com/fwlink/?linkid=872328)提供了特定詳細資料，說明有哪些身分識別資訊會儲存在歐洲，並提供會儲存在歐洲資料中心外之資訊的詳細資料。 

---
 
### <a name="new-user-provisioning-saas-app-integrations---may-2018"></a>新的使用者佈建 SaaS 應用程式整合 - 2018 年 5 月

**類型：** 新功能  
**服務類別：** 應用程式佈建  
**產品功能：** 協力廠商整合
 
Azure AD 可讓您自動化在 SaaS 應用程式 (例如 Dropbox、Salesforce、ServiceNow 等等) 中建立、維護和移除使用者身分識別的作業。 我們在 2018 年 5 月，於 Azure AD 應用程式庫為以下應用程式新增了使用者佈建支援：

- [BlueJeans](https://docs.microsoft.com/azure/active-directory/active-directory-saas-bluejeans-provisioning-tutorial)

- [Cornerstone OnDemand](https://docs.microsoft.com/azure/active-directory/active-directory-saas-cornerstone-ondemand-provisioning-tutorial)

- [Zendesk](https://docs.microsoft.com/azure/active-directory/active-directory-saas-zendesk-provisioning-tutorial)

如需 Azure AD 資源庫中支援使用者佈建的所有應用程式清單，請參閱 [https://aka.ms/appstutorial](https://aka.ms/appstutorial)。

---
 
### <a name="azure-ad-access-reviews-of-groups-and-app-access-now-provides-recurring-reviews"></a>群組和應用程式存取的 Azure AD 存取權檢閱現在提供定期檢閱

**類型：** 新功能  
**服務類別：** 存取權檢閱  
**產品功能：** 治理
 
Azure AD Premium P2 中現已正式提供群組和應用程式的存取權檢閱功能。  系統管理員能夠設定群組成員資格的存取權檢閱以及應用程式指派，使其在固定時間間隔重複發生，例如每月或每季。

---

### <a name="azure-ad-activity-logs-sign-ins-and-audit-are-now-available-through-ms-graph"></a>Azure AD 活動記錄 (登入與稽核) 現可透過 MS Graph 使用

**類型：** 新功能  
**服務類別：** 報告  
**產品功能：** 監視和報告
 
包含登入與稽核記錄的 Azure AD 活動記錄現已可透過 MS Graph 使用。 我們已公開兩個可透過 MS Graph 來存取這些記錄的端點。 請查看我們的[文件](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-getting-started-azure-portal)，了解如何開始以程式設計方式存取 Azure AD 報告 API。 

---
 
### <a name="improvements-to-the-b2b-redemption-experience-and-leave-an-org"></a>改善 B2B 兌換體驗，以及離開組織

**類型：** 新功能  
**服務類別：** B2B  
**產品功能：** B2B/B2C

**Just-in-time 兌換：** 在您使用 B2B API 與來賓使用者共用資源後，不必傳出特殊的邀請電子郵件。 在大多數情況下，來賓使用者可存取資源，並且會即時帶過兌換體驗。 錯過的電子郵件不會產生任何影響。 也不會再向您的來賓使用者詢問「您是否已經按一下系統傳給您的兌換連結?」。 這表示只要 SPO 使用了邀請管理員，所有使用者都能得到相同標準的雲端附件 URL，不論使用者處於內外部，也不論其兌換狀態。

**新式兌換體驗：** 兌換登陸頁面不再分割。 使用者會看到新式的同意體驗，包含組織隱私權聲明的邀請，這與第三方應用程式的做法相似。

**來賓使用者可離開組織：** 當使用者與組織的關係結束後，可自行離開組織。 使邀請組織的管理員不會再收到要求「移除」的請求，也不會再產生支援票證。

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---may-2018"></a>Azure AD 應用程式庫中有新的同盟應用程式可用 - 2018 年 5 月

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合
 
我們在 2018 年 5 月將下列這 18 個提供同盟支援的全新應用程式新增至應用程式庫：

[AwardSpring](https://docs.microsoft.com/azure/active-directory/active-directory-saas-awardspring-tutorial)、Infogix Data3Sixty Govern、[Yodeck](https://docs.microsoft.com/azure/active-directory/active-directory-saas-infogix-tutorial)、[Jamf Pro](https://docs.microsoft.com/azure/active-directory/active-directory-saas-jamfprosamlconnector-tutorial)、[KnowledgeOwl](https://docs.microsoft.com/azure/active-directory/active-directory-saas-knowledgeowl-tutorial)、[Envi MMIS](https://docs.microsoft.com/azure/active-directory/active-directory-saas-envimmis-tutorial)、[LaunchDarkly](https://docs.microsoft.com/azure/active-directory/active-directory-saas-launchdarkly-tutorial)、[Adobe Captivate Prime](https://docs.microsoft.com/azure/active-directory/active-directory-saas-adobecaptivateprime-tutorial)、[Montage Online](https://docs.microsoft.com/azure/active-directory/active-directory-saas-montageonline-tutorial)、[まなびポケット](https://docs.microsoft.com/azure/active-directory/active-directory-saas-manabipocket-tutorial)、OpenReel、[Arc Publishing - SSO](https://docs.microsoft.com/azure/active-directory/active-directory-saas-arc-tutorial)、[PlanGrid](https://docs.microsoft.com/azure/active-directory/active-directory-saas-plangrid-tutorial)、[iWellnessNow](https://docs.microsoft.com/azure/active-directory/active-directory-saas-iwellnessnow-tutorial)、[Proxyclick](https://docs.microsoft.com/azure/active-directory/active-directory-saas-proxyclick-tutorial)、[Riskware](https://docs.microsoft.com/azure/active-directory/active-directory-saas-riskware-tutorial)、[Flock](https://docs.microsoft.com/azure/active-directory/active-directory-saas-flock-tutorial)、[Reviewsnap](https://docs.microsoft.com/azure/active-directory/active-directory-saas-reviewsnap-tutorial)

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。

如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing)。

---
 
### <a name="new-step-by-step-deployment-guides-for-azure-active-directory"></a>全新 Azure Active Directory 逐步部署指南

**類型：** 新功能  
**服務類別：** 其他  
**產品功能：** 目錄
 
如何部署 Azure Active Directory (Azure AD) 的全新逐步指南，包括自助式密碼重設 (SSPR)、單一登入 (SSO)、條件式存取 (CA)、應用程式 Proxy、使用者佈建、Active Directory 同盟服務 (ADFS) 至傳遞驗證 (PTA)，以及 ADFS 至密碼雜湊同步處理 (PHS)。

如需檢視部署指南，請前往 GitHub 上的[識別部署指南](https://aka.ms/DeploymentPlans)存放庫。 如需提供有關部署指南的意見反應，請使用[部署方案意見反應單](https://aka.ms/deploymentplanfeedback)。 如果對於部署指南有任何疑問，請於 [IDGitDeploy](mailto:idgitdeploy@microsoft.com) 聯絡我們。

---

### <a name="enterprise-applications-search---load-more-apps"></a>企業應用程式搜尋 - 載入更多應用程式

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** SSO
 
尋找您的應用程式/服務主體時發生問題嗎？ 我們已新增讓您載入企業應用程式清單中更多應用程式的功能。 根據預設，我們會顯示 20 項應用程式。 您現在可以按一下 [載入更多] 來檢視其他應用程式。 

---
 
### <a name="the-may-release-of-aadconnect-contains-a-public-preview-of-the-integration-with-pingfederate-important-security-updates-many-bug-fixes-and-new-great-new-troubleshooting-tools"></a>五月發行的 AADConnect 包含與 PingFederate 之整合、重要安全性更新、許多 Bug 修正和全新絕佳疑難排解工具的公開預覽。 

**類型：** 已變更的功能  
**服務類別：** AD Connect  
**產品功能：** 身分識別生命週期管理
 
五月發行的 AADConnect 包含與 PingFederate 之整合、重要安全性更新、許多 Bug 修正和全新絕佳疑難排解工具的公開預覽。 您可以在[這裡](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-version-history#118190)找到版本資訊。

---

### <a name="azure-ad-access-reviews-auto-apply"></a>Azure AD 存取權檢閱：自動套用

**類型：** 已變更的功能  
**服務類別：** 存取權檢閱  
**產品功能：** 治理

Azure AD Premium P2 中現已正式提供群組和應用程式的存取權檢閱功能。 系統管理員可以設定為在存取權檢閱完成時，自動將檢閱者的變更套用至該群組或應用程式。 系統管理員也可以指定在檢閱者未回應、移除存取權、保留存取權或採取系統建議時，使用者繼續存取所會發生的動作。 

---

### <a name="id-tokens-can-no-longer-be-returned-using-the-query-responsemode-for-new-apps"></a>新的應用程式無法再使用 query response_mode 傳回 ID 權杖。 

**類型：** 已變更的功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
在 2018 年 4 月 25 日當天或之後所建立的應用程式，將無法再使用 **query** response_mode 來要求 **id_token**。  這會使 Azure AD 能與 OIDC 規格保持一致，協助減少您的應用程式受攻擊面。  在 2018/4/25 之前所建立的應用程式，可以使用 **query** response_mode 與 **id_token** 的 response_type。  從 AAD 要求 id_token 時所傳回的錯誤為 **AADSTS70007: 要求權杖時，‘query’ 不是支援的 ‘response_mode’ 值**。

在建立新應用程式物件時 (例如，應用程式 Proxy 使用方式)，**fragment** 與 **form_post** response_modes 會繼續生效，請務必在這些 response_modes 建立新應用程式前使用其中一個 response_modes。  
 
---
 
## <a name="april-2018"></a>2018 年 4 月 

### <a name="azure-ad-b2c-access-token-are-ga"></a>Azure AD B2C 存取權杖已正式推出

**類型：** 新功能  
**服務類別：** B2C - 取用者身分識別管理  
**產品功能：** B2B/B2C 

您現在可以使用存取權杖來存取受 Azure AD B2C 保護的 Web API。 這項功能將從公開預覽移至正式推出 (GA)。 設定 Azure AD B2C 應用程式和 Web API 的 UI 經驗已改進，並進行了其他小幅的改良。
 
如需詳細資訊，請參閱 [Azure AD B2C：要求存取權杖](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-access-tokens)。

---

### <a name="test-single-sign-on-configuration-for-saml-based-applications"></a>為 SAML 式應用程式測試單一登入設定

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** SSO

在設定 SAML 型 SSO 應用程式時，您無法在設定頁面上測試整合。 如果您在登入時發生錯誤，您可以提供測試經驗中的錯誤，而 Azure AD 會提供給您解決特定問題的解決步驟。

如需詳細資訊，請參閱

- [設定對不在 Azure Active Directory 應用程式庫中的應用程式的單一登入](https://docs.microsoft.com/azure/active-directory/active-directory-saas-custom-apps)
- [如何偵錯 SAML 型單一登入 Azure Active Directory 中的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-debugging)

---
 
### <a name="azure-ad-terms-of-use-now-has-per-user-reporting"></a>Azure AD 使用規定現在具備每位使用者回報

**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 合規性
 
系統管理員現在可以選取特定 ToU，並查看同意該 ToU 的所有使用者以及其發生日期/時間。

如需詳細資訊，請參閱 [Azure AD 使用規定特性](https://docs.microsoft.com/azure/active-directory/active-directory-tou)。

---
 
### <a name="azure-ad-connect-health-risky-ip-for-ad-fs-extranet-lockout-protection"></a>Azure AD Connect Health：AD FS 外部網路鎖定保護的具風險 IP 

**類型：** 新功能  
**服務類別：** 其他  
**產品功能：** 監視和報告

Connect Health 現在能夠支援每小時或每日偵測超出失敗 U/P 登入閾值的 IP 位址。 此功能所提供的功能如下：

- 完整報告，其中顯示 IP 位址，以及每小時/每日產生的失敗登入數目 (可自訂閾值)。
- 電子郵件形式警示，其中顯示已超出每小時/每日失敗 U/P 登入閾值的特定 IP 位址。
- 用來執行詳細資料分析的下載選項

如需詳細資訊，請參閱[有風險的 IP 報告](https://aka.ms/aadchriskyip)。

---
 
### <a name="easy-app-config-with-metadata-file-or-url"></a>輕鬆以中繼資料檔案或 URL 設定應用程式

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** SSO

在企業應用程式頁面上，系統管理員可以上傳 SAML 中繼資料檔，以設定 AAD 資源庫和非資源庫應用程式的 SAML 型登入。

此外，您可以使用 Azure AD 應用程式同盟中繼資料 URL 來設定目標應用程式的 SSO。

如需詳細資訊，請參閱[設定對不在 Azure Active Directory 應用程式庫中的應用程式的單一登入](https://docs.microsoft.com/azure/active-directory/active-directory-saas-custom-apps)。

---

### <a name="azure-ad-terms-of-use-now-generally-available"></a>Azure AD 使用規定現已正式推出

**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 合規性
 

Azure AD 使用規定已經從公用預覽移至正式推出。

如需詳細資訊，請參閱 [Azure AD 使用規定特性](https://docs.microsoft.com/azure/active-directory/active-directory-tou)。

---

### <a name="allow-or-block-invitations-to-b2b-users-from-specific-organizations"></a>允許或封鎖對特定組織的 B2B 使用者的邀請

**類型：** 新功能  
**服務類別：** B2B  
**產品功能：** B2B/B2C
 

您現在可以指定想要共用的夥伴組織，以及在 Azure AD B2B 共同作業中與其共同作業。 若要這樣做，您可以選擇建立特定的允許或拒絕網域清單。 使用這些功能封鎖網域時，員工便無法再傳送邀請給該網域中的人員。

這可協助您控制資源的存取，同時讓已核准的使用者擁有平順的經驗。

此 B2B 共同作業功能可供所有 Azure Active Directory 客戶使用，而且可搭配 Azure AD Premium 功能使用，例如條件式存取和身分識別保護，以便進一步控制外部商務使用者何時與如何登入及取得存取權。

如需詳細資訊，請參閱[允許或封鎖對特定組織的 B2B 使用者的邀請](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-allow-deny-list)。

---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Azure AD 應用程式庫中可用的新同盟應用程式

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合

我們在 2018 年 4 月將下列這 13 個提供同盟支援的全新應用程式新增至應用程式庫：

Criterion HCM、[FiscalNote](https://docs.microsoft.com/azure/active-directory/active-directory-saas-fiscalnote-tutorial)、[Secret Server (On-Premises)](https://docs.microsoft.com/azure/active-directory/active-directory-saas-secretserver-on-premises-tutorial)、[Dynamic Signal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-dynamicsignal-tutorial)、[mindWireless](https://docs.microsoft.com/azure/active-directory/active-directory-saas-mindwireless-tutorial)、[OrgChart Now](https://docs.microsoft.com/azure/active-directory/active-directory-saas-orgchartnow-tutorial), [Ziflow](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ziflow-tutorial)、[AppNeta Performance Monitor](https://docs.microsoft.com/azure/active-directory/active-directory-saas-appneta-tutorial)、[Elium](https://docs.microsoft.com/azure/active-directory/active-directory-saas-elium-tutorial)、[Fluxx Labs](https://docs.microsoft.com/azure/active-directory/active-directory-saas-fluxxlabs-tutorial)、[Cisco Cloud](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ciscocloud-tutorial)、Shelf、[SafetyNet](https://docs.microsoft.com/azure/active-directory/active-directory-saas-safetynet-tutorial)

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。

如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing)。

---
 
### <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-applications-public-preview"></a>授與 Azure AD 中的 B2B 使用者您內部部署應用程式的存取權 (公開預覽)

**類型：** 新功能  
**服務類別：** B2B  
**產品功能：** B2B/B2C

作為使用 Azure Active Directory (Azure AD) B2B 共同作業功能邀請夥伴組織的來賓使用者進入您 Azure AD 的組織，您現在可以對這些 B2B 使用者提供內部部署應用程式的存取權。 這些內部部署應用程式可以搭配 Kerberos 限制委派 (KCD) 來使用 SAML 型驗證或整合式 Windows 驗證 (IWA)。

如需詳細資訊，請參閱[對 Azure AD 中的 B2B 使用者授與內部部署應用程式的存取權](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-hybrid-cloud-to-on-premises)。

---
 
### <a name="get-sso-integration-tutorials-from-the-azure-marketplace"></a>從 Azure Marketplace 取得 SSO 整合教學課程

**類型：** 已變更的功能  
**服務類別：** 其他  
**產品功能：** 協力廠商整合

如果 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps?page=1)中所列的應用程式支援 SAML 型單一登入，按一下**立即取得**即可為您提供與該應用程式相關聯的整合教學課程。 

---

### <a name="faster-performance-of-azure-ad-automatic-user-provisioning-to-saas-applications"></a>Azure AD 自動將使用者佈建至 SaaS 應用程式的效能更快

**類型：** 已變更的功能  
**服務類別：** 應用程式佈建  
**產品功能：** 協力廠商整合
 
先前使用 SaaS 應用程式 (例如 Salesforce、ServiceNow 及 Box) 適用之 Azure Active Directory 使用者佈建連接器 的客戶，如果其 Azure AD 租用戶包含合計超過 100,000 個使用者和群組，而且使用使用者和群組指派來判斷應該佈建哪些使用者，則可能會遭遇低落的效能。

在 2018 年 4 月 2 日，Azure AD 佈建服務部署了非常顯著的效能增強功能，可大幅減少在 Azure Active Directory 與目標 SaaS 應用程式之間執行首次同步處理所需的時間量。

因此，許多曾經花許多天或未曾完成應用程式首次同步處理的客戶，現在只需要幾分鐘或幾小時數就能完成首次同步處理。

如需詳細資訊，請參閱[佈建期間會發生什麼事？](https://docs.microsoft.com/azure/active-directory/active-directory-saas-app-provisioning#what-happens-during-provisioning)

---

### <a name="self-service-password-reset-from-windows-10-lock-screen-for-hybrid-azure-ad-joined-machines"></a>從已加入混合式 Azure AD 之電腦的 Windows 10 鎖定畫面進行自助式密碼重設

**類型：** 已變更的功能  
**服務類別：** 自助式密碼重設  
**產品功能：** 使用者驗證
 
我們已更新 Windows 10 SSPR 功能，進而包含已加入混合式 Azure AD 的電腦支援。 這項功能適用於 Windows 10 RS4，可讓使用者從 Windows 10 電腦的鎖定畫面重設其密碼。 已啟用且註冊自助式密碼重設的使用者可以利用這項功能。

如需詳細資訊，請參閱[從登入畫面重設 Azure AD 密碼](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-sspr-windows)。
 
---

## <a name="march-2018"></a>2018 年 3 月
 
### <a name="certificate-expire-notification"></a>憑證到期通知

**類型：** 固定  
**服務類別：** 企業應用程式  
**產品功能：** SSO
 
當資源庫或非資源庫應用程式即將到期時，Azure AD 會傳送通知。 

針對 SAML 型單一登入設定的企業應用程式，有些使用者未收到相關通知。 此問題已解決。 若憑證將在 7、30 和 60 天內到期，Azure AD 就會傳送通知。 您可以在稽核記錄中看到此事件。 

如需詳細資訊，請參閱

- [在 Azure Active Directory 中管理同盟單一登入的憑證](https://docs.microsoft.com/azure/active-directory/active-directory-sso-certs)
- [Azure Active Directory 入口網站中的稽核活動報告](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-activity-audit-logs)
 
---
 
### <a name="twitter-and-github-identity-providers-in-azure-ad-b2c"></a>Azure AD B2C 中的 Twitter 與 GitHub 識別提供者

**類型：** 新功能  
**服務類別：** B2C - 取用者身分識別管理  
**產品功能：** B2B/B2C
 
您現在能以身分識別提供者的身分在 Azure AD B2C 中新增 Twitter 或 GitHub。 Twitter 將從公開預覽進行至 GA。 GitHub 即將在公開預覽中發行。

如需詳細資訊，請參閱[什麼是 Azure AD B2B 共同作業？](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b)。
 
---

### <a name="restrict-browser-access-using-intune-managed-browser-with-azure-ad-application-based-conditional-access-for-ios-and-android"></a>針對 iOS 和 Android，搭配 Azure AD 應用程式型條件式存取使用 Intune Managed Browser 來限制瀏覽器存取

**類型：** 新功能  
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護
 
**現已在公開預覽版中推出！**

**Intune Managed Browser SSO：** 您的員工可以在各個原生用戶端 (例如 Microsoft Outlook) 上使用單一登入，以及針對所有與 Azure AD 連接的應用程式使用 Intune Managed Browser。

**Intune Managed Browser 條件式存取支援：** 您現在可以使用應用程式型條件式存取原則來要求員工使用 Intune Managed Browser。

如需深入了解，請參閱我們的[部落格文章](https://cloudblogs.microsoft.com/enterprisemobility/2018/03/15/the-intune-managed-browser-now-supports-azure-ad-sso-and-conditional-access/) \(英文\)。

如需詳細資訊，請參閱

- [設定應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

- [設定 Managed Browser 原則](https://aka.ms/managedbrowser)  

---
 
### <a name="app-proxy-cmdlets-in-powershell-ga-module"></a>Powershell GA 模組中的應用程式 Proxy Cmdlet

**類型：** 新功能  
**服務類別：** 應用程式 Proxy  
**產品功能：** 存取控制
 
Powershell GA 模組中目前提供應用程式 Proxy Cmdlet 的支援！ 這需要您隨時更新 Powershell 模組 - 如果您超過一年未更新，某些 Cmdlet 可能會停止運作。 

如需詳細資訊，請參閱 [AzureAD](https://docs.microsoft.com/powershell/module/Azuread/?view=azureadps-2.0)。
 
---
 
### <a name="office-365-native-clients-are-supported-by-seamless-sso-using-a-non-interactive-protocol"></a>無縫式 SSO 是使用非互動式通訊協定來支援 Office 365 原生用戶端

**類型：** 新功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
使用 Office 365 原生用戶端 (版本 16.0.8730.xxxx 和更新版本) 的使用者在使用無縫式 SSO 時，能獲得無訊息登入體驗。 能有此項支援是因為已對 Azure AD 新增非互動式通訊協定 (WS-Trust)。

如需詳細資訊，請參閱[原生用戶端上的登入如何與無縫 SSO 搭配運作？](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso-how-it-works#how-does-sign-in-on-a-native-client-with-seamless-sso-work)
 
---

### <a name="users-get-a-silent-sign-on-experience-with-seamless-sso-if-an-application-sends-sign-in-requests-to-azure-ads-tenant-endpoints"></a>如果應用程式將登入要求傳送至 Azure AD 的租用戶端點，使用者便可在使用無縫式 SSO 時獲得無訊息登入體驗

**類型：** 新功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
如果應用程式 (例如，`https://contoso.sharepoint.com`) 將登入要求傳送至 Azure AD 的租用戶端點 - 亦即 `https://login.microsoftonline.com/contoso.com/<..>` 或 `https://login.microsoftonline.com/<tenant_ID>/<..>` - 而不是 Azure AD 的一般端點 (`https://login.microsoftonline.com/common/<...>`)，使用者便可在使用無縫式 SSO 時獲得無訊息登入體驗。

如需詳細資訊，請參閱 [Azure Active Directory 無縫單一登入](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso)。 

---
 
### <a name="need-to-add-only-one-azure-ad-url-instead-of-two-urls-previously-to-users-intranet-zone-settings-to-roll-out-seamless-sso"></a>只需要將一個 Azure AD URL，而不是先前的兩個 URL 新增至使用者的內部網路區域設定，即可推出無縫式 SSO

**類型：** 新功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
若要對使用者推出無縫式 SSO，您必須使用 Active Directory 中的群組原則，僅將一個 Azure AD URL 新增至使用者的內部網路區域設定：`https://autologon.microsoftazuread-sso.com`。 過去，客戶必須新增兩個 URL。

如需詳細資訊，請參閱 [Azure Active Directory 無縫單一登入](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso)。 
 
---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Azure AD 應用程式庫中可用的新同盟應用程式

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合

我們在 2018 年 3 月將下列這 15 個提供同盟支援的全新應用程式新增至應用程式庫：

[Boxcryptor](https://docs.microsoft.com/azure/active-directory/active-directory-saas-boxcryptor-tutorial)、[CylancePROTECT](https://docs.microsoft.com/azure/active-directory/active-directory-saas-cylanceprotect-tutorial)、Wrike、[SignalFx](https://docs.microsoft.com/azure/active-directory/active-directory-saas-signalfx-tutorial)、Assistant by FirstAgenda、[YardiOne](https://docs.microsoft.com/azure/active-directory/active-directory-saas-yardione-tutorial)、Vtiger CRM、Inwink、[Amplitude](https://docs.microsoft.com/azure/active-directory/active-directory-saas-amplitude-tutorial)、[Spacio](https://docs.microsoft.com/azure/active-directory/active-directory-saas-spacio-tutorial)、[ContractWorks](https://docs.microsoft.com/azure/active-directory/active-directory-saas-contractworks-tutorial)、[Bersin](https://docs.microsoft.com/azure/active-directory/active-directory-saas-bersin-tutorial)、[Mercell](https://docs.microsoft.com/azure/active-directory/active-directory-saas-mercell-tutorial)、[Trisotech Digital Enterprise Server](https://docs.microsoft.com/azure/active-directory/active-directory-saas-trisotechdigitalenterpriseserver-tutorial)、[Qumu Cloud](https://docs.microsoft.com/azure/active-directory/active-directory-saas-qumucloud-tutorial)。
 
如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。

如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing)。 

---
 
### <a name="pim-for-azure-resources-is-generally-available"></a>Azure 資源的 PIM 已正式推出

**類型：** 新功能  
**服務類別：** Privileged Identity Management  
**產品功能：** Privileged Identity Management
 
如果您將 Azure AD Privileged Identity Management 用於目錄角色，現在可以將 PIM 的時間限制存取和指派功能用於 Azure 資源角色，例如訂用帳戶、資源群組、虛擬機器和 Azure Resource Manager 支援的任何其他資源。 啟用角色 Just-In-Time 時，強制執行 Multi-Factor Authentication，並搭配核准的變更時間範圍排定啟用。 此外，此版本新增公開預覽期間未提供的增強功能，包括更新的 UI、核准工作流程，並且能夠延長即將到期的角色和更新過期的角色。

如需詳細資訊，請參閱 [適用於 Azure 資源的 PIM (預覽)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac)
 
---
 
### <a name="adding-optional-claims-to-your-apps-tokens-public-preview"></a>將選用宣告新增至您的應用程式權杖 (公開預覽)

**類型：** 新功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
您的 Azure AD 應用程式現在可以在 JWT 或 SAML 權杖中要求自訂或選用宣告。  這些有關使用者或租用戶的宣告並未預設包含在權杖中，因為有大小或適用性方面的條件約束。  這目前在 1.0 和 2.0 版端點上的 Azure AD 應用程式中處於公開預覽階段。  請參閱文件，以了解可新增的宣告以及如何編輯應用程式資訊清單來要求這些宣告。  

如需詳細資訊，請參閱 [Azure AD 中的選用宣告](https://docs.microsoft.com/azure/active-directory/develop/active-directory-optional-claims)。
 
---
 
### <a name="azure-ad-supports-pkce-for-more-secure-oauth-flows"></a>Azure AD 支援 PKCE 以獲得更安全的 OAuth 流程

**類型：** 新功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
已更新 Azure AD 文件以發佈 PKCE 的支援，PKCE 可讓 OAuth 2.0 授權碼授與流程期間的通訊更安全。  1.0 和 2.0 版端點上同時支援 S256 與純文字 code_challenges。 

如需詳細資訊，請參閱[要求授權碼](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols-oauth-code#request-an-authorization-code)。 
 
---
 
### <a name="support-for-provisioning-all-user-attribute-values-available-in-the-workday-getworkers-api"></a>Workday Get_Workers API 中提供佈建所有使用者屬性值的支援

**類型：** 新功能  
**服務類別：** 應用程式佈建  
**產品功能：** 協力廠商整合
 
輸入佈建 (從 Workday 至 Active Directory 和 Azure AD) 的公開預覽現在支援擷取和佈建 Workday Get_Workers API 中可用的所有屬性值。 在隨附於 Workday 輸入佈建連接器初始版本的屬性之外，這會新增數百個其他標準和自訂屬性的支援。

如需詳細資訊，請參閱：[自訂 Workday 使用者屬性的清單](https://docs.microsoft.com/azure/active-directory/active-directory-saas-workday-inbound-tutorial#customizing-the-list-of-workday-user-attributes)

---

### <a name="changing-group-membership-from-dynamic-to-static-and-vice-versa"></a>將群組成員資格從動態變更為靜態，反之亦然

**類型：** 新功能  
**服務類別：** 群組管理  
**產品功能：** 共同作業
 
您可以變更在群組中管理成員資格的方式。 當您想要在系統中保留相同的群組名稱和識別碼，讓任何現有的群組參考仍然有效時，這非常實用；建立新的群組需要更新這些參考。
我們已更新 Azure AD 系統管理中心，以支援這項功能。 現在，客戶可以將現有的群組從動態成員資格轉換為指派的成員資格，反之亦然。 現有的 PowerShell Cmdlet 還是可以使用。

如需詳細資訊，請參閱[將動態成員資格變更為靜態，反之亦然](https://docs.microsoft.com/azure/active-directory/active-directory-groups-dynamic-membership-azure-portal#changing-dynamic-membership-to-static-and-vice-versa)

---

### <a name="improved-sign-out-behavior-with-seamless-sso"></a>透過無縫式 SSO 改進登出行為

**類型：** 已變更的功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證
 
在過去，即使使用者明確登出受 Azure AD 保護的應用程式，但他們如果嘗試在其公司網路內從已加入網域的裝置再次存取 Azure AD 應用程式，系統就會使用無縫式 SSO 自動將他們登入。 而這項變更會用來支援登出。  這可讓使用者選擇使用相同或不同的 Azure AD 帳戶來進行登入，而不是使用「無縫 SSO」來自動登入。

如需詳細資訊，請參閱 [Azure Active Directory 無縫單一登入](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso)
 
---
 
### <a name="application-proxy-connector-version-154020-released"></a>應用程式 Proxy 連接器 1.5.402.0 版已發行

**類型：** 已變更的功能  
**服務類別：** 應用程式 Proxy  
**產品功能：** 身分識別安全性與保護
 
此連接器版本會在 11 月前逐漸推出。 這個新的連接器版本包含下列變更：

- 連接器現在會設定網域層級而非子網域層級的 cookie。 這可確保較順暢的 SSO 體驗，並避免多餘的驗證提示。
- 支援區塊編碼要求
- 改良的連接器健康情況監視 
- 多項 錯誤 (bug) 修正及穩定性改進

如需詳細資訊，請參閱[了解 Azure AD 應用程式 Proxy 連接器](https://docs.microsoft.com/azure/active-directory/application-proxy-understand-connectors)。
 
---

## <a name="february-2018"></a>2018 年 2 月
 
### <a name="improved-navigation-for-managing-users-and-groups"></a>改進使用者和群組的管理瀏覽體驗

**類型：** 方案變更  
**服務類別：** 目錄管理  
**產品功能：** 目錄

使用者和群組的管理瀏覽體驗已經簡化。 您現在可以從目錄概觀直接瀏覽至所有使用者清單，且更容易存取已刪除的使用者清單。 您也可以從目錄概觀直接瀏覽至所有群組清單，且更容易存取群組管理設定。 此外，從目錄概觀頁面，您還可搜尋使用者、群組、企業應用程式或應用程式註冊。 

---

### <a name="availability-of-sign-ins-and-audit-reports-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet"></a>21Vianet (Azure 中國 21Vianet) 所營運的 Microsoft Azure 提供登入和稽核報告

**類型：** 新功能  
**服務類別：** Azure Stack  
**產品功能：** 監視和報告

21Vianet (Azure 中國 21Vianet) 執行個體所營運的 Microsoft Azure 現已提供 Azure AD 活動記錄報告。 以下是包含的記錄：

- **登入活動記錄** - 包括與您租用戶相關的所有登入記錄。

- **自助密碼稽核記錄** - 包括所有 SSPR 稽核記錄。

- **目錄管理稽核記錄** - 包括所有目錄管理相關的稽核記錄，例如使用者管理、應用程式管理等。

藉由這些記錄，您便可深入了解您環境的運作情況。 提供的資料可讓您：

- 判斷使用者如何利用您的應用程式和服務。

- 針對會阻止使用者完成其工作的問題進行疑難排解。

如需有關如何使用這些報告的詳細資訊，請參閱 [Azure Active Directory 報告](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-azure-portal)。

---

### <a name="use-report-reader-role-non-admin-role-to-view-azure-ad-activity-reports"></a>使用「報告讀取者」角色 (非管理員角色) 來檢視 Azure AD 活動報告

**類型：** 新功能  
**服務類別：** 報告  
**產品功能：** 監視和報告

在對於讓非管理員角色能夠存取 Azure AD 活動記錄的客戶意見反應中，我們已讓具備「報告讀取者」角色的使用者除了能夠使用我們的「圖形 API」之外，也能夠存取 Azure 入口網站內的「登入」和「稽核」活動。 

如需有關如何使用這些報告的詳細資訊，請參閱 [Azure Active Directory 報告](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-azure-portal)。 

---

### <a name="employeeid-claim-available-as-user-attribute-and-user-identifier"></a>提供 EmployeeID 宣告作為使用者屬性和使用者識別碼

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** SSO
 
您可以從企業應用程式 UI 設定 **EmployeeID**，作為 SAML 登入應用程式中成員使用者和 B2B 來賓的使用者識別碼和使用者屬性。

如需詳細資訊，請參閱[針對 Azure Active Directory 中的企業應用程式自訂 SAML 權杖中發出的宣告](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization)。

---

### <a name="simplified-application-management-using-wildcards-in-azure-ad-application-proxy"></a>在 Azure AD 應用程式 Proxy 中使用萬用字元來簡化應用程式管理

**類型：** 新功能  
**服務類別：** 應用程式 Proxy  
**產品功能：** 使用者驗證
 
為了讓應用程式部署更簡單並減輕您的系統管理額外負荷，我們現已支援使用萬用字元來發佈應用程式的功能。 若要發佈萬用字元應用程式，您可以依照標準的應用程式發佈流程，但在內部和外部 URL 中使用萬用字元。

如需詳細資訊，請參閱 [Azure Active Directory 應用程式 Proxy 中的萬用字元應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-wildcard)

---

### <a name="new-cmdlets-to-support-configuration-of-application-proxy"></a>可支援設定應用程式 Proxy 的新 Cmdlet

**類型：** 新功能  
**服務類別：** 應用程式 Proxy  
**產品功能：** 平台

最新版的「AzureAD PowerShell 預覽」模組包含可讓客戶使用 PowerShell 來設定「應用程式 Proxy 應用程式」的新 Cmdlet。

這些新 Cmdlet 包括： 

- Get-AzureADApplicationProxyApplication
- Get-AzureADApplicationProxyApplicationConnectorGroup
- Get-AzureADApplicationProxyConnector
- Get-AzureADApplicationProxyConnectorGroup
- Get-AzureADApplicationProxyConnectorGroupMembers
- Get-AzureADApplicationProxyConnectorMemberOf
- New-AzureADApplicationProxyApplication
- New-AzureADApplicationProxyConnectorGroup
- Remove-AzureADApplicationProxyApplication
- Remove-AzureADApplicationProxyApplicationConnectorGroup
- Remove-AzureADApplicationProxyConnectorGroup
- Set-AzureADApplicationProxyApplication
- Set-AzureADApplicationProxyApplicationConnectorGroup
- Set-AzureADApplicationProxyApplicationCustomDomainCertificate
- Set-AzureADApplicationProxyApplicationSingleSignOn
- Set-AzureADApplicationProxyConnector
- Set-AzureADApplicationProxyConnectorGroup

---
 
### <a name="new-cmdlets-to-support-configuration-of-groups"></a>可支援設定群組的新 Cmdlet

**類型：** 新功能  
**服務類別：** 應用程式 Proxy  
**產品功能：** 平台

最新版的 AzureAD PowerShell 模組包含可管理 Azure AD 中群組的 Cmdlet。 這些 Cmdlet 先前是在 AzureADPreview 模組中提供的，現已新增到 AzureAD 模組中

現已發行而可正式運作的 Group Cmdlet 包括： 

- Get-AzureADMSGroup
- New-AzureADMSGroup
- Remove-AzureADMSGroup
- Set-AzureADMSGroup
- Get-AzureADMSGroupLifecyclePolicy
- New-AzureADMSGroupLifecyclePolicy
- Remove-AzureADMSGroupLifecyclePolicy
- Add-AzureADMSLifecyclePolicyGroup
- Remove-AzureADMSLifecyclePolicyGroup
- Reset-AzureADMSLifeCycleGroup   
- Get-AzureADMSLifecyclePolicyGroup

---
 
### <a name="a-new-release-of-azure-ad-connect-is-available"></a>提供新版 Azure AD Connect

**類型：** 新功能  
**服務類別：** AD 同步  
**產品功能：** 平台
 
Azure AD Connect 是在 Azure AD 與內部部署資料來源 (包括 Windows Server Active Directory 和 LDAP) 之間同步處理資料時，建議使用的工具。

>[!Important]
>這個組建導入了結構描述和同步處理規則變更。 「Azure AD Connect 同步處理服務」會在升級後觸發「完整匯入」和「完整同步處理」步驟。 如需有關如何變更此行為的資訊，請參閱[如何延遲升級之後的完整同步處理](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-upgrade-previous-version#how-to-defer-full-synchronization-after-upgrade)。

此版本包含下列更新和變更：

**已修正的問題**

- 修正計時視窗切換至下一個頁面時，在 [資料分割篩選] 頁面背景工作的問題。

- 修正在 ConfigDB 自訂動作期間造成存取違規的錯誤 (bug)。

- 修正從 SQL 復原時連線逾時的錯誤 (bug)。

- 修正含 SAN 萬用字元的憑證在進行必要條件檢查時失敗的錯誤 (bug)。

- 修正造成 miiserver.exe 在 AAD 連接器匯出期間當機的錯誤 (bug)。

- 修正當執行動作造成 AAD Connect 精靈變更設定時，錯誤密碼嘗試會記錄在 DC 上的錯誤 (bug)

**新功能和改進**
 
- 應用程式遙測 - 系統管理員可以開啟/關閉此類別的資料。

- Azure AD 健康情況資料 - 系統管理員必須瀏覽健康情況入口網站，才能控制其健康情況設定。 一旦變更服務原則，代理程式會讀取並強制執行它。

- 新增裝置回寫設定動作和頁面初始化的進度列。

- 藉由 HTML 報告改善一般診斷並以 ZIP 文字/HTML 報告提供完整資料收集。

- 改善自動升級的可靠性，並新增其他遙測，以確保您可以判斷伺服器的健康情況。

- 限制具特殊權限帳戶對 AD 連接器帳戶的可用權限。 針對新的安裝，精靈會在建立 MSOL 帳戶之後，限制具特殊權限帳戶對 MSOL 帳戶的權限。 這些變更會影響「自動建立」帳戶的相關快速安裝和自訂安裝。

- 將安裝程式變更為在進行 AADConnect 全新安裝時不需要 SA 權限。

- 可針對特定物件的同步處理問題進行疑難排解的新公用程式。 目前，此公用程式會檢查下列各項：

    - 同步處理的使用者物件與 Azure AD 租用戶中使用者帳戶的 UserPrincipalName 不符。
  
    - 是否因為網域篩選而從同步處理篩選物件
  
    - 是否因為組織單位 (OU) 篩選而從同步處理篩選物件

- 可針對特定使用者帳戶同步處理儲存在內部部署 Active Directory 中之目前密碼雜湊的新公用程式。 此公用程式不需要變更密碼。 

---
 
### <a name="applications-supporting-intune-app-protection-policies-added-for-use-with-azure-ad-application-based-conditional-access"></a>新增支援 Intune 應用程式防護原則的應用程式，以與 Azure AD 應用程式型條件式存取搭配使用

**類型：** 已變更的功能  
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護

我們已新增更多支援應用程式型條件式存取的應用程式。 現在，您可以使用這些已核准的用戶端應用程式，來存取 Office 365 及其他已連線至 Azure AD 的雲端應用程式。

以下是會在二月底前新增的應用程式：

- Microsoft Power BI

- Microsoft Launcher

- Microsoft Invoicing

如需詳細資訊，請參閱

- [已核准的用戶端應用程式需求](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#approved-client-app-requirement)
- [Azure AD 應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

---

### <a name="terms-of-use-update-to-mobile-experience"></a>更新行動體驗的使用規定 

**類型：** 已變更的功能  
**服務類別：** 使用規定  
**產品功能：** 合規性

顯示使用規定時，您現在可以按一下 [檢視時發生問題嗎?請按一下這裡]。 按一下此連結會在您裝置上以原生方式開啟使用規定。 不論文件中的字型大小或裝置的螢幕大小為何，您都可以視需要縮放並閱讀文件。 

---
 
## <a name="january-2018"></a>2018 年 1 月
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Azure AD 應用程式庫中可用的新同盟應用程式 

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合

在 2018 年 1 月，於應用程式庫中新增了下列支援同盟的新應用程式：

[IBM OpenPages](https://go.microsoft.com/fwlink/?linkid=864698)、[OneTrust Privacy Management Software](https://go.microsoft.com/fwlink/?linkid=861660)、[Dealpath](https://go.microsoft.com/fwlink/?linkid=863526)、[IriusRisk Federated Directory 及 [Fidelity NetBenefits](https://go.microsoft.com/fwlink/?linkid=864701)。

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。

如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing)。 

---
 
### <a name="sign-in-with-additional-risk-detected"></a>偵測到有額外風險的登入

**類型：** 新功能  
**服務類別：** 身分識別保護  
**產品功能：** 身分識別安全性與保護

您針對偵測到風險事件所獲得的深入解析會與您的 Azure AD 訂用帳戶息息相關。 使用 Azure AD Premium P2 版本時，您會獲得有關所有基礎偵測的最詳細資訊。

使用 Azure AD Premium P1 版本時，您授權未涵蓋的偵測會顯示為「偵測到有額外風險的登入」風險事件。

如需詳細資訊，請參閱 [Azure Active Directory 風險事件](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-risk-events)。
 
---

### <a name="hide-office-365-applications-from-end-users-access-panels"></a>從使用者存取面板中隱藏 Office 365 應用程式

**類型：** 新功能  
**服務類別：** 我的應用程式  
**產品功能：** SSO

您現在可以透過新的使用者設定，以更好的方式管理 Office 365 應用程式在使用者存取面板上的顯示方式。 如果您偏好只在 Office 入口網站中顯示 Office 應用程式，此選項將有助於減少使用者存取面板中的應用程式數量。 此設定位於 [使用者設定] 中，並且標示為 [使用者只能在 Office 365 入口網站中看見 Office 365 應用程式]。

如需詳細資訊，請參閱[從 Azure Active Directory 的使用者體驗中隱藏應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-hide-third-party-app)。

---
 
### <a name="seamless-sign-into-apps-enabled-for-password-sso-directly-from-apps-url"></a>啟用無縫應用程式登入功能，可直接從應用程式的 URL 進行密碼 SSO 

**類型：** 新功能  
**服務類別：** 我的應用程式  
**產品功能：** SSO

現在透過便利的工具即可使用 [我的應用程式] 瀏覽器擴充功能，此工具會以捷徑的形式在您的瀏覽器中提供 [我的應用程式] 單一登入功能。 安裝之後，使用者將會在其瀏覽器中看到一個鬆餅圖示，可讓使用者快速存取應用程式。 使用者現在可以利用：

- 此功能從應用程式的登入頁面直接登入以密碼 SSO 為基礎的應用程式
- 使用快速搜尋功能來啟動任何應用程式
- 擴充功能所提供的最近使用過的應用程式捷徑
- 此擴充功能適用於 Edge、Chrome 及 Firefox。
 
如需詳細資訊，請參閱[我的應用程式安全登入擴充功能](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction#my-apps-secure-sign-in-extension)。

---

### <a name="azure-ad-administration-experience-in-azure-classic-portal-has-been-retired"></a>Azure 傳統入口網站中的 Azure AD 系統管理體驗已淘汰

**類型：** 已被取代   
**服務類別：** Azure AD  
**產品功能：** 目錄

Azure 傳統入口網站中的 Azure AD 系統管理體驗自 2018 年 1 月 8 日起已淘汰。 Azure 傳統入口網站本身也隨之一起淘汰。 日後您應該使用 [Azure AD 系統管理中心](https://aad.portal.azure.com)來進行所有以入口網站為基礎的 Azure AD 系統管理。
 
---

### <a name="the-phonefactor-web-portal-has-been-retired"></a>PhoneFactor Web 入口網站已淘汰

**類型：** 已被取代  
**服務類別：** Azure AD  
**產品功能：** 目錄
 
PhoneFactor Web 入口網站自 2018 年 1 月 8 日起已淘汰。 此入口網站是用來進行 MFA 伺服器的系統管理，但這些功能已移至位於 portal.azure.com 的 Azure 入口網站。 

MFA 設定位於：**Azure Active Directory \> MFA Server**
 
---
 
### <a name="deprecate-azure-ad-reports"></a>取代 Azure AD 報告

**類型：** 已被取代  
**服務類別：** 報告  
**產品功能：** 身分識別生命週期管理  


在新的「Azure Active Directory 系統管理」主控台正式運作且現在已有適用於活動和安全性報告的新 API 之後，"/reports" 端點底下的報告 API 自 2017 年 12 月 31 日結束起已淘汰。

**可用的內容？**

在轉換至新系統管理主控台的過程中，我們提供了 2 個新 API 來擷取 Azure AD 活動記錄。 新的 API 集合除了能提供更豐富的稽核和登入活動外，還提供更豐富的篩選和排序功能。 現在可透過 Microsoft Graph 的 Identity Protection 風險事件 API 來存取先前透過安全性報告所使用的資料。

如需詳細資訊，請參閱

- [開始使用 Azure Active Directory 報告 API](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-getting-started-azure-portal)

- [開始使用 Azure Active Directory Identity Protection 和 Microsoft Graph](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection-graph-getting-started)

---

## <a name="december-2017"></a>2017 年 12 月

### <a name="terms-of-use-in-the-access-panel"></a>存取面板中的使用規定

**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 合規性
 
您現在可以移至「存取面板」來檢視先前所接受的使用規定。

請遵循下列步驟：

1. 前往 [MyApp 入口網站](https://myapps.microsoft.com)並登入。

2. 在右上角中，選取您的名稱，然後從清單中選取 [設定檔]。 

3. 在您的 [設定檔] 上，選取 [檢閱使用規定]。 

4. 現在您可以檢閱您已接受的使用規定。 

如需詳細資訊，請參閱 [Azure AD 使用規定特性 (預覽)](https://docs.microsoft.com/azure/active-directory/active-directory-tou)。
 
---
 
### <a name="new-azure-ad-sign-in-experience"></a>新的 Azure AD 登入體驗

**類型：** 新功能  
**服務類別：** Azure AD  
**產品功能：** 使用者驗證
 
Azure AD 和 Microsoft 帳戶身分識別系統 UI 已經過重新設計，具有一致的外觀與風格。 此外，Azure AD 登入頁面會先收集使用者名稱，然後在第二個畫面收集認證。

如需詳細資訊，請參閱[新的 Azure AD 登入體驗 (目前為公開預覽)](https://cloudblogs.microsoft.com/enterprisemobility/2017/08/02/the-new-azure-ad-signin-experience-is-now-in-public-preview/) \(英文\)。
 
---
 
### <a name="fewer-sign-in-prompts-a-new-keep-me-signed-in-experience-for-azure-ad-sign-in"></a>較少的登入提示次數：適用於 Azure AD 登入的新「讓我保持登入」體驗

**類型：** 新功能  
**服務類別：** Azure AD  
**產品功能：** 使用者驗證
 
Azure AD 登入頁面上的 [讓我保持登入] 核取方塊已被新的提示取代，此提示會在您成功驗證之後出現。 

如果您對此提示的回應是 [是]，服務就會提供您一個持續性的重新整理權杖。 此行為與在舊體驗中選取 [讓我保持登入] 核取方塊相同。 針對同盟租用戶，此提示會在您已成功向同盟服務驗證之後顯示。

如需詳細資訊，請參閱[較少的登入提示次數：適用於 Azure AD 登入的新「讓我保持登入」體驗 (預覽)](https://cloudblogs.microsoft.com/enterprisemobility/2017/09/19/fewer-login-prompts-the-new-keep-me-signed-in-experience-for-azure-ad-is-in-preview/) \(英文\)。 

---

### <a name="add-configuration-to-require-the-terms-of-use-to-be-expanded-prior-to-accepting"></a>新增設定以要求在接受使用規定前先展開使用規定

**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 合規性
 
一個系統管理員選項，可要求其使用者在接受使用規定之前，先展開該規定。

請選取 [開啟] 或 [關閉] 來要求使用者展開使用規定。 [開啟] 設定會要求使用者必須先檢視使用規定，才能接受該規定。

如需詳細資訊，請參閱 [Azure AD 使用規定特性 (預覽)](https://docs.microsoft.com/azure/active-directory/active-directory-tou)。
 
---

### <a name="scoped-activation-for-eligible-role-assignments"></a>適用於合格角色指派的限域啟用

**類型：** 新功能  
**服務類別：** Privileged Identity Management  
**產品功能：** Privileged Identity Management
 
您可以使用限域啟用，以比原始指派預設值更少的自主性來啟用合格的 Azure 資源角色指派。 例如，假設您獲指派為租用戶中訂用帳戶的擁有者。 在使用限域啟用的情況下，您最多可以針對該訂用帳戶內包含的 5 個資源 (例如資源群組與虛擬機器) 啟用擁有者角色。 限制啟用範圍可降低對重要 Azure 資源執行不需要之變更的可能性。

如需詳細資訊，請參閱[什麼是 Azure AD Privileged Identity Management？](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure)。
 
---
 
### <a name="new-federated-apps-in-the-azure-ad-app-gallery"></a>Azure AD App 資源庫中的新同盟應用程式

**類型：** 新功能  
**服務類別：** 企業應用程式  
**產品功能：** 協力廠商整合

我們在 2017 年 12 月將下列這些提供同盟支援的全新應用程式新增至應用程式庫：

[Accredible](https://go.microsoft.com/fwlink/?linkid=863523) \(機器翻譯\)、Adobe Experience Manager、[EFI Digital StoreFront](https://go.microsoft.com/fwlink/?linkid=861685) \(機器翻譯\)、[Communifire](https://go.microsoft.com/fwlink/?linkid=861676) CybSafe \(機器翻譯\)、[FactSet](https://go.microsoft.com/fwlink/?linkid=863525) \(機器翻譯\)、[IMAGE WORKS](https://go.microsoft.com/fwlink/?linkid=863517) \(機器翻譯\)、[MOBI](https://go.microsoft.com/fwlink/?linkid=863521)[MobileIron Azure AD 整合](https://go.microsoft.com/fwlink/?linkid=858027) \(機器翻譯\)、[Reflektive](https://go.microsoft.com/fwlink/?linkid=863518) \(機器翻譯\)、[SAML SSO for Bamboo by resolution GmbH](https://go.microsoft.com/fwlink/?linkid=863520) \(機器翻譯\)、[SAML SSO for Bitbucket by resolution GmbH](https://go.microsoft.com/fwlink/?linkid=863519)[Vodeclic](https://go.microsoft.com/fwlink/?linkid=863522) \(機器翻譯\)、WebHR、Zenegy Azure AD 整合。

如需應用程式的詳細資訊，請參閱[與 Azure Active Directory 整合的 SaaS 應用程式](https://aka.ms/appstutorial)。

如需在 Azure AD 應用程式庫中列出應用程式的詳細資訊，請參閱[在 Azure Active Directory 應用程式庫中列出您的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing)。 
 
---

### <a name="approval-workflows-for-azure-ad-directory-roles"></a>Azure AD 目錄角色的核准工作流程

**類型：** 已變更的功能  
**服務類別：** Privileged Identity Management  
**產品功能：** Privileged Identity Management
 
Azure AD 目錄角色的核准工作流程已正式運作。

使用核准工作流程時，特殊權限角色系統管理員可以要求合格角色成員必須先要求角色啟用，才能使用特殊權限角色。 可以將核准責任委派給多個使用者和群組。 合格角色成員會在核准完成且其角色生效時收到通知。

---
 
### <a name="pass-through-authentication-skype-for-business-support"></a>傳遞驗證：商務用 Skype 支援

**類型：** 已變更的功能  
**服務類別：** 驗證 (登入)  
**產品功能：** 使用者驗證

傳遞驗證現在支援使用者登入可支援新式驗證 (包括線上和混合式拓撲) 的「商務用 Skype」用戶端應用程式。 

如需詳細資訊，請參閱[以新式驗證支援的商務用 Skype 技術](https://technet.microsoft.com/library/mt803262.aspx) \(英文\)。
 
---

### <a name="updates-to-azure-ad-privileged-identity-management-for-azure-rbac-preview"></a>適用於 Azure RBAC 的 Azure AD Privileged Identity Management 更新 (預覽)

**類型：** 已變更的功能  
**服務類別：** Privileged Identity Management  
**產品功能：** Privileged Identity Management
 
有了適用於 Azure 角色型存取控制 (RBAC) 的 Azure AD Privileged Identity Management (PIM) 公開預覽更新，您現在可以：

* 使用恰到好處的系統管理。
* 要求必須經過核准才能啟用資源角色。
* 排定要求必須經過核准才能取得 Azure AD 和 Azure RBAC 角色的未來角色啟用。
 
如需詳細資訊，請參閱[適用於 Azure 資源的 Privileged Identity Management (預覽)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac) \(機器翻譯\)。

---
 
## <a name="november-2017"></a>2017 年 11 月
 
### <a name="access-control-service-retirement"></a>存取控制服務淘汰

**類型：** 方案變更  
**服務類別：** 存取控制服務  
**產品功能：** 存取控制服務 

Azure Active Directory 存取控制 (也稱為「存取控制」服務) 將於 2018 年底淘汰。 在未來幾週，我們將會提供包括詳細排程和概要移轉指引的詳細資訊。 您可以在此頁面留言詢問有關「存取控制」服務的任何問題，小組成員將會回答您的留言。

---

### <a name="restrict-browser-access-to-the-intune-managed-browser"></a>限制瀏覽器對 Intune Managed Browser 的存取 

**類型：** 方案變更  
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護

您可以藉由使用 Intune Managed Browser 作為已核准的應用程式，限制瀏覽器對 Office 365 和其他已連線至 Azure AD 之雲端應用程式的存取。 

您現在可以為應用程式型條件式存取設定下列條件：

**用戶端應用程式：** 瀏覽器

**變更的影響為何？**

目前，當您使用此條件時，系統會封鎖存取。 當預覽已可供使用時，所有存取都會要求使用 Managed Browser 應用程式。 

請在即將推出的部落格和版本資訊中，查看此功能及更多資訊。 

如需詳細資訊，請參閱 [Azure AD 中的條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)。
 
---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>針對 Azure AD 應用程式型條件式存取之新的已核准用戶端應用程式

**類型：** 方案變更  
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護

下列應用程式已列入[已核准的用戶端應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#approved-client-app-requirement)清單：

- [Microsoft Kaizala](https://www.microsoft.com/garage/profiles/kaizala/) \(英文\)
- [Microsoft StaffHub](https://staffhub.office.com/what-it-is)

如需詳細資訊，請參閱

- [已核准的用戶端應用程式需求](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#approved-client-app-requirement)
- [Azure AD 應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

---

### <a name="terms-of-use-support-for-multiple-languages"></a>針對多種語言的使用規定支援

**類型：** 新功能    
**服務類別：** 使用規定  
**產品功能：** 合規性

系統管理員現在可以建立包含多個 PDF 文件的新使用規定。 您能以相對應的語言標記這些 PDF 文件。 系統會根據使用者的喜好設定，顯示相符語言的 PDF。 如果沒有相符項目，則會顯示預設語言。

---
 

### <a name="real-time-password-writeback-client-status"></a>即時密碼回寫用戶端狀態

**類型：** 新功能  
**服務類別**自助式密碼重設  
**產品功能：** 使用者驗證

您現在可以檢閱內部部署密碼回寫用戶端的狀態。 [[密碼重設](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/PasswordReset)] 頁面的 [內部部署整合] 區段中有提供此選項。 

如果您在連線至內部部署回寫用戶端時遇到問題，將會看見包含下列資訊的錯誤訊息：

- 有關無法連線至內部部署回寫用戶端之原因的資訊。
- 可協助您解決該問題之文件的連結。 

如需詳細資訊，請參閱[內部部署整合](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-how-it-works#on-premises-integration)。

---

### <a name="azure-ad-app-based-conditional-access"></a>Azure AD 應用程式型條件式存取 
 
**類型：** 新功能  
**服務類別：** Azure AD  
**產品功能：** 身分識別安全性與保護

您現在可以藉由使用 [Azure AD 應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)，限制只有支援 Intune 應用程式防護原則的[已核准用戶端應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#approved-client-app-requirement)，才能存取 Office 365 和其他已連線至 Azure AD 的雲端應用程式。 系統會使用 Intune 應用程式防護原則，來設定及保護這些用戶端應用程式上的公司資料。

透過結合[應用程式型](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)和[裝置型](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-policy-connected-applications)的條件式存取原則，您將能彈性地保護個人及公司裝置上的資料。

下列條件及控制項現已能搭配應用程式型條件式存取使用：

**支援的平台條件**

- iOS
- Android

**用戶端應用程式條件**

- 行動裝置應用程式和桌面用戶端

**存取控制**

- 需要已核准的用戶端應用程式

如需詳細資訊，請參閱 [Azure AD 應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)。
 
---

### <a name="manage-azure-ad-devices-in-the-azure-portal"></a>在 Azure 入口網站中管理 Azure AD 裝置

**類型：** 新功能  
**服務類別：** 裝置註冊和管理  
**產品功能：** 身分識別安全性與保護

您現在只要在一個位置，就可以找到所有已連線至 Azure AD 的裝置，以及與裝置相關的活動。 現已推出在 Azure 入口網站中管理所有裝置身分識別和設定的新系統管理體驗。 在這個版本中，您可以︰

- 檢視所有可供在 Azure AD 中進行條件式存取的裝置。
- 檢視屬性，包括您的混合式已加入 Azure AD 的裝置。
- 尋找已加入 Azure AD 之裝置的 BitLocker 金鑰、透過 Intune 管理裝置等等。
- 管理 Azure AD 裝置相關的設定。

如需詳細資訊，請參閱[使用 Azure 入口網站來管理裝置](https://docs.microsoft.com/azure/active-directory/device-management-azure-portal)。

---

### <a name="support-for-macos-as-a-device-platform-for-azure-ad-conditional-access"></a>支援使用 macOS 作為 Azure AD 條件式存取的裝置平台 

**類型：** 新功能    
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護 

您現在可以在 Azure AD 條件式存取原則中，包含 (或排除) macOS 作為裝置平台條件。 除了將 macOS 新增為支援的裝置平台之外，您也可以：

- **使用 Intune 來註冊和管理 macOS 裝置。** 與其他平台 (例如 iOS 和 Android) 類似，有可供 macOS 用來進行統一註冊的公司入口網站應用程式。 您可以使用適用於 macOS 的新公司入口網站應用程式，向 Intune 註冊裝置，並向 Azure AD 註冊它。
- **確保 macOS 裝置遵守組織在 Intune 中定義的合規性原則。** 您現在可以在 Azure 入口網站上的 Intune 中，設定適用於 macOS 裝置的合規性原則。 
- **限制只有符合規範的 macOS 裝置才能存取 Azure AD 中的應用程式。** 條件式存取原則撰寫提供 macOS 作為個別裝置平台選項。 現在您可以針對在 Azure 中設定的目標應用程式，撰寫 macOS 特定的條件式存取原則。

如需詳細資訊，請參閱

- [使用 Intune 為 macOS 裝置建立裝置合規性原則](https://aka.ms/macoscompliancepolicy)
- [Azure AD 中的條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)
 
---

### <a name="network-policy-server-extension-for-azure-multi-factor-authentication"></a>適用於 Azure Multi-Factor Authentication 的網路原則伺服器擴充功能 

**類型：** 新功能    
**服務類別：** 多重要素驗證  
**產品功能：** 使用者驗證

適用於 Azure Multi-Factor Authentication 的「網路原則伺服器」擴充功能會藉由使用現有的伺服器，將雲端式 Multi-Factor Authentication 功能新增至您的驗證基礎結構。 有了「網路原則伺服器」擴充功能，您便可以在現有的驗證流程中新增通話、簡訊或電話應用程式驗證。 您無須安裝、設定及維護新的伺服器。 

這個擴充功能是針對想要保護虛擬私人網路連線但又不想部署 Azure Multi-Factor Authentication Server 的組織所建立。 「網路原則伺服器」擴充功能會作為 RADIUS 與雲端式 Azure Multi-Factor Authentication 之間的配接器，可為同盟或同步的使用者提供第二驗證要素。


如需詳細資訊，請參閱[將現有的網路原則伺服器基礎結構與 Azure Multi-Factor Authentication 整合](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-nps-extension)。

 
---

### <a name="restore-or-permanently-remove-deleted-users"></a>還原或永久移除已刪除的使用者

**類型：** 新功能    
**服務類別：** 使用者管理  
**產品功能：** 目錄 


在 Azure AD 系統管理中心中，您現在可以：

- 還原已刪除的使用者。 
- 永久刪除使用者。


**試用此功能：**

1. 在 Azure AD 系統管理中心內，於 [管理] 區段中選取 [[所有使用者](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/All)]。 

2. 從 [顯示] 清單，選取 [最近刪除的使用者]。 

3. 選取一或多個最近刪除的使用者，然後將這些使用者還原或是永久刪除。

 
---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>針對 Azure AD 應用程式型條件式存取之新的已核准用戶端應用程式

 
**類型：** 已變更的功能  
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護


下列應用程式已新增至[已核准的用戶端應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#approved-client-app-requirement)清單：

- Microsoft Planner
- Azure 資訊保護 


如需詳細資訊，請參閱

- [已核准的用戶端應用程式需求](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#approved-client-app-requirement)
- [Azure AD 應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)


---

### <a name="use-or-between-controls-in-a-conditional-access-policy"></a>在條件式存取原則中的控制項之間使用 "OR" 


**類型：** 已變更的功能    
**服務類別：** 條件式存取  
**產品功能：** 身分識別安全性與保護

 
您現在可以針對條件式存取控制項使用 "OR" (需要其中一個選取的控制項)。 您可以使用此功能建立在存取控制項之間具有 "OR" 的原則。 例如，您可以使用此功能來建立原則，以要求使用者使用 Multi-Factor Authentication 進行登入，「或」使用符合規範的裝置進行登入。

如需詳細資訊，請參閱 [Azure AD 條件式存取中的控制項](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-controls)。

 
---

### <a name="aggregation-of-real-time-risk-events"></a>即時風險事件的彙總


**類型：** 已變更的功能    
**服務類別：** 身分識別保護  
**產品功能：** 身分識別安全性與保護


在 Azure AD Identity Protection 中，現在會針對每個風險事件類型，彙總於指定日子源自於相同 IP 位址的所有即時風險事件。 這項變更會限制所顯示的風險事件量，但對使用者安全性並無任何影響。

基礎即時偵測仍然會在使用者每次登入時運作。 如果您已設定登入風險安全性原則以進行 Multi-Factor Authentication 或封鎖存取，在每次發生具風險的登入時，仍然會觸發該原則。

 
---
 




## <a name="october-2017"></a>2017 年 10 月


### <a name="deprecate-azure-ad-reports"></a>取代 Azure AD 報告


**類型：** 方案變更  
**服務類別：** 報告  
**產品功能：** 身分識別生命週期管理  



Azure 入口網站可讓您使用：

- 新的 Azure AD 管理主控台。
- 活動和安全報告的新 API。
 
由於有這些新功能，因此 /reports 端點下的報告 API 已在 2017 年 12 月 10 日停用。 

---

### <a name="automatic-sign-in-field-detection"></a>自動登入欄位偵測


**類型：** 固定   
**服務類別：** 我的應用程式  
**產品功能：** 單一登入  



Azure AD 針對呈現 HTML 使用者名稱和密碼欄位的應用程式，支援自動登入欄位偵測。 [如何自動擷取應用程式的登入欄位](https://docs.microsoft.com/azure/active-directory/application-config-sso-problem-configure-password-sso-non-gallery#how-to-manually-capture-sign-in-fields-for-an-application)記錄下列步驟。 您可以在藉由在 [Azure 入口網站](http://aad.portal.azure.com) 的 [企業應用程式] 頁面上新增 [非資源庫] 應用程式來尋找此功能。 此外，您也可以在此新應用程式上將 [單一登入] 模式設定為 [密碼單一登入]，輸入 Web URL，然後再儲存頁面。
 
之前因為服務問題，而將此功能暫時停用。 此問題已解決，因此已恢復自動登入欄位偵測功能。

---

### <a name="new-multi-factor-authentication-features"></a>新的 Multi-Factor Authentication 功能


**類型：** 新功能  
**服務類別：** 多重要素驗證  
**產品功能：** 身分識別安全性與保護  



多重要素驗證 (MFA) 是保護您組織不可或缺的環節。 為了使認證更具彈性，並讓使用體驗更加順暢，我們已新增下列功能： 

- 將多重要素挑戰結果直接整合至 Azure AD 登入報告中，包括以程式設計方式存取 MFA 的結果。
- 將 MFA 設定更深入地整合至 Azure 入口網站的 Azure AD 設定體驗中。

在此公開預覽中，MFA 管理和報告功能是核心 Azure AD 設定體驗相當重要的一部分。 現在您可以在 Azure AD 體驗內管理 MFA 管理入口網站功能。

如需詳細資訊，請參閱[在 Azure 入口網站中 MFA 報告的參考](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-activity-sign-ins-mfa)。 


---

### <a name="terms-of-use"></a>使用規定



**類型：** 新功能  
**服務類別：** 使用規定  
**產品功能：** 合規性  



您可以使用 Azure AD 使用規定向使用者顯示資訊，例如法務或合規性需求的相關免責聲明。

在下列案例中，您可以使用 Azure AD 使用規定：

- 適用於貴組織中所有使用者的一般使用規定
- 依使用者屬性 (例如，醫生與護士或國內員工與國際員工 (藉由動態群組的方式完成)) 區分的特定使用規定
- 適用於存取高影響力商務應用程式 (例如 Salesforce) 的特定使用規定

如需詳細資訊，請參閱 [Azure AD 使用規定](https://docs.microsoft.com/azure/active-directory/active-directory-tou)。


---

### <a name="enhancements-to-privileged-identity-management"></a>針對 Privileged Identity Management 的增強功能


**類型：** 新功能  
**服務類別：** Privileged Identity Management  
**產品功能：** Privileged Identity Management  


有了 Azure AD Privileged Identity Management，您便可以管理、控制及監視下列項目對組織內 Azure 資源 (預覽) 的存取行為：

- 訂用帳戶
- 資源群組
- 虛擬機器 

所有在 Azure 入口網站內利用 Azure RBAC 功能的資源，都可以運用 Azure AD Privileged Identity Management 所提供的安全性和生命週期管理功能。

如需詳細資訊，請參閱[適用於 Azure 資源的 Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac)。


---

### <a name="access-reviews"></a>存取權檢閱


**類型：** 新功能  
**服務類別：** 存取權檢閱  
**產品功能：** 合規性  



組織可以使用存取權檢閱 (預覽) 來有效率地管理群組成員資格，以及對企業應用程式的存取權： 

- 您可以重新認證來賓使用者存取權，方法為使用應用程式的存取權和群組的成員資格的存取權檢閱。 檢閱者可以根據存取權檢閱所提供的深入解析，有效率地決定是否要允許來賓繼續存取。
- 您可以利用存取權檢閱，重新認證員工對應用程式的存取權和群組成員資格。

您可以將存取權檢閱控制項收集到與組織相關的程式，以追蹤合規性或與風險相關之應用程式的檢閱。

如需詳細資訊，請參閱 [Azure AD 存取權檢閱](https://docs.microsoft.com/azure/active-directory/active-directory-azure-ad-controls-access-reviews-overview)。


---

### <a name="hide-third-party-applications-from-my-apps-and-the-office-365-app-launcher"></a>從 [我的應用程式] 和 Office 365 應用程式啟動器隱藏協力廠商應用程式



**類型：** 新功能  
**服務類別：** 我的應用程式  
**產品功能：** 單一登入  



您現在可以透過新的**隱藏應用程式**屬性，以更好的方式管理在使用者入口網站上顯示的應用程式。 當出現後端服務的應用程式圖格或重複的圖格，而造成使用者的應用程式啟動器凌亂時，您可以隱藏應用程式來協助改善此情況。 該切換參數位於協力廠商應用程式的 [屬性] 區段，標示為 [是否要向使用者顯示?] 您也可以透過 PowerShell 以程式設計方式隱藏應用程式。 

如需詳細資訊，請參閱[從 Azure AD 的使用者體驗中隱藏協力廠商應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-hide-third-party-app)。 


**可用的內容？**

 在轉換至新系統管理主控台的過程中，有兩個可擷取 Azure AD 活動記錄的新 API 可供使用。 新的 API 集合除了能提供更豐富的稽核和登入活動外，還提供更豐富的篩選和排序功能。 現在透過 Microsoft Graph 中的 Identity Protection 風險事件 API，即可存取先前透過安全性報告提供的資料。


## <a name="september-2017"></a>2017 年 9 月

### <a name="hotfix-for-identity-manager"></a>適用於 Identity Manager 的 Hotfix


**類型：** 已變更的功能  
**服務類別：** Identity Manager  
**產品功能：** 身分識別生命週期管理  



Identity Manager 2016 Service Pack 1 的 Hotfix 彙總套件 (組建 4.4.1642.0) 已自 2017 年 9 月 25 日起可供使用。 此彙總套件：

- 可解決問題並新增增強功能。
- 是一個累積更新，可取代到 Identity Manager 2016 組建 4.4.1459.0 為止的所有 Identity Manager 2016 Service Pack 1 更新。 
- 要求您必須具備 Identity Manager 2016 組建 4.4.1302.0。 

如需詳細資訊，請參閱 [適用於 Identity Manager 2016 Service Pack 1 的 Hotfix 彙總套件 (組建 4.4.1642.0) 已可供使用](https://support.microsoft.com/help/4021562) \(機器翻譯\)。 

---