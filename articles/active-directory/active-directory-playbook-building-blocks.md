---
title: Azure Active Directory 概念證明腳本構成要素 | Microsoft Docs
description: 探索並快速實作身分識別和存取管理案例
services: active-directory
keywords: azure active directory, 腳本, 概念證明, PoC
documentationcenter: ''
author: dstefanMSFT
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/04/2017
ms.author: dstefan
ms.openlocfilehash: c8517114f51b5aed8f8e31a19b672721b109775e
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46298297"
---
# <a name="azure-active-directory-proof-of-concept-playbook-building-blocks"></a>Azure Active Directory 概念證明腳本：構成要素

## <a name="catalog-of-roles"></a>角色目錄

| 角色 | 說明 | 概念證明 (PoC) 責任 |
| --- | --- | --- |
| **身分識別架構 / 開發小組** | 此小組通常負責設計解決方案、實作原型、推動核准，以及最後將解決方案交付運作 | 他們會提供環境，並從管理的角度評估不同的案例 |
| **內部部署身分識別作業小組** | 管理內部部署的不同身分識別來源：Active Directory 樹系、LDAP 目錄、HR 系統，以及同盟識別提供者。 | 提供 PoC 案例所需的內部部署資源存取權。<br/>應該儘量不要牽涉到他們|
| **應用程式技術擁有者** | 將與 Azure AD 整合之不同雲端應用程式和服務的技術擁有者 | 提供有關 SaaS 應用程式 (可能是用於測試的執行個體) 的詳細資料 |
| **Azure AD 全域管理員** | 管理 Azure AD 組態 | 提供認證以設定同步處理服務。 通常在 PoC 期間與「身分識別架構」屬於相同小組，但在作業階段則為不同小組|
| **資料庫小組** | 資料庫基礎結構的擁有者 | 針對特定的案例準備，提供對 SQL 環境 (ADFS 或 Azure AD Connect) 的存取權。<br/>應該儘量不要牽涉到他們 |
| **網路小組** | 網路基礎結構的擁有者 | 提供網路層級的必要存取權，以供同步處理伺服器正確存取資料來源和雲端服務 (防火牆規則、已開啟的連接埠、IPSec 規則等) |
| **安全性小組** | 定義安全性策略、分析來自不同來源的安全性報告，並追蹤所發現的事項。 | 提供目標安全性評估案例 |

## <a name="common-prerequisites-for-all-building-blocks"></a>所有構成要素的一般必要條件

以下是 Azure AD Premium 之任何 POC 都需要的一些必要條件。

| 必要條件 | 資源 |
| --- | --- |
| 已定義有效 Azure 訂用帳戶的 Azure AD 租用戶 | [如何取得 Azure Active Directory 租用戶](develop/quickstart-create-new-tenant.md)<br/>**注意：** 如果您已經有具備 Azure AD Premium 授權的環境，您可以瀏覽至 https://aka.ms/accessaad 來取得沒有上限的訂用帳戶 <br/>深入了解： https://blogs.technet.microsoft.com/enterprisemobility/2016/02/26/azure-ad-mailbag-azure-subscriptions-and-azure-ad-2/ 和 https://technet.microsoft.com/library/dn832618.aspx |
| 已定義並確認的網域 | [將自訂網域名稱新增到 Azure Active Directory](active-directory-domains-add-azure-portal.md)<br/>**注意：** 有些工作負載 (例如 Power BI) 可能已在幕後佈建 Azure AD 租用戶。 若要檢查指定網域是否與租用戶相關聯，請瀏覽至 https://login.microsoftonline.com/{domain}/v2.0/.well-known/openid-configuration。 如果您獲得成功的回應，即表示網域已指派給租用戶，而可能需要接管。 如果是這種情況，請連絡 Microsoft 以獲得進一步的指導。 深入了解接管選項：[什麼是 Azure 的自助式註冊？](users-groups-roles/directory-self-service-signup.md) |
| 已啟用 Azure AD Premium 或 EMS 試用版 | [Azure Active Directory Premium 可免費使用一個月](https://azure.microsoft.com/trial/get-started-active-directory/) |
| 您已將 Azure AD Premium 或 EMS 授權指派給 PoC 使用者 | [在 Azure Active Directory 中進行自身和使用者的授權](active-directory-licensing-get-started-azure-portal.md) |
| Azure AD 全域管理員認證 | [在 Azure Active Directory 中指派系統管理員角色](users-groups-roles/directory-assign-admin-roles.md) |
| 選擇性但強烈建議使用：作為後援的平行實驗室環境 | [Azure AD Connect 的必要條件](hybrid/how-to-connect-install-prerequisites.md) |

## <a name="directory-synchronization---password-hash-sync-phs---new-installation"></a>目錄同步作業 - 密碼雜湊同步處理 (PHS) - 全新安裝

預估完成時間：PoC 使用者不到 1,000 人時少於一小時

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 用來執行 Azure AD Connect 的伺服器 | [Azure AD Connect 的必要條件](hybrid/how-to-connect-install-prerequisites.md) |
| 在相同網域中且屬於安全性群組及 OU 的目標 POC 使用者 | [自訂 Azure AD Connect 安裝](hybrid/how-to-connect-install-custom.md#domain-and-ou-filtering) |
| 識別 POC 所需的 Azure AD Connect 功能 | [將 Active Directory 與 Azure Active Directory 連接 - 設定同步處理功能](hybrid/how-to-connect-install-roadmap.md#configure-sync-features) |
| 您有內部部署和雲端環境所需的認證  | [Azure AD Connect：帳戶與權限](hybrid/reference-connect-accounts-permissions.md) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 下載最新版的 Azure AD Connect | [下載 Microsoft Azure Active Directory Connect](https://www.microsoft.com/download/details.aspx?id=47594) |
| 使用最簡單的路徑來安裝 Azure AD Connect：快速 <br/>1.篩選至目標 OU 以將「同步處理週期」時間縮到最短<br/>2.選擇內部部署群組中的一組目標使用者<br/>3.部署其他 POC 主題所需的功能 | [Azure AD Connect：自訂安裝：網域和 OU 篩選](hybrid/how-to-connect-install-custom.md#domain-and-ou-filtering) <br/>[Azure AD Connect：自訂安裝：群組型篩選](hybrid/how-to-connect-install-custom.md#sync-filtering-based-on-groups)<br/>[Azure AD Connect：整合內部部署身分識別與 Azure Active Directory：設定同步處理功能](hybrid/how-to-connect-install-roadmap.md#configure-sync-features) |
| 開啟 Azure AD Connect UI 並看著執行中的設定檔完成 (匯入、同步處理及匯出) | [Azure AD Connect 同步處理：排程器](hybrid/how-to-connect-sync-feature-scheduler.md) |
| 開啟 [Azure AD 管理入口網站](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/)，移至 [所有使用者] 刀鋒視窗、新增 [授權單位來源] 資料行，並確認有顯示那些使用者後，將他們適當地標示成來自 [Windows Server AD] | [Azure AD 管理入口網站](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) |

### <a name="considerations"></a>考量

1. 若要了解密碼雜湊同步處理的安全性考量，請參閱[這裡](hybrid/how-to-connect-password-hash-synchronization.md)。  如果試驗生產使用者的密碼雜湊同步處理確定不可行，則請考慮下列替代方案：
   * 在生產環境網域中建立測試使用者。 確定您不會同步處理任何其他帳戶
   * 移至 UAT 環境
2.  如果您想要採用同盟，就值得了解在 POC 之外將同盟解決方案與內部部署「識別提供者」建立關聯的成本，並將它對照您要獲得的效益進行評估：
    * 它在關鍵路徑中，因此您必須針對高可用性進行設計
    * 它是一項您需要規劃容量的內部部署服務
    * 它是一項您需要監視/維護/修補的內部部署服務

深入了解：[了解 Office 365 身分識別和 Azure Active Directory - 同盟身分識別](https://support.office.com/article/Understanding-Office-365-identity-and-Azure-Active-Directory-06a189e7-5ec6-4af2-94bf-a22ea225a7a9#bk_federated)

## <a name="branding"></a>商標

預估完成時間：15 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 資產 (影像、標誌等)；為了獲得最佳視覺效果，請確定資產採用建議的大小。 | [在 Azure Active Directory 中將公司商標新增到登入頁面](active-directory-branding-custom-signon-azure-portal.md) |
| 選擇性：如果環境具有 ADFS 伺服器，則需有該伺服器的存取權以自訂 Web 佈景主題 | [AD FS 使用者登入自訂](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/operations/ad-fs-user-sign-in-customization) |
| 用以執行使用者登入體驗的用戶端電腦 |  |
| 選擇性：用以驗證體驗的行動裝置 |  |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 移至「Azure AD 管理入口網站」 | [Azure AD 管理入口網站 - 公司商標](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/LoginTenantBranding) |
| 上傳登入頁面的資產 (大型標誌、小型標誌、標籤等)。 如果您有 AD FS，可視需要，與 ADFS 登入頁面採用相同的資產 | [在登入和存取面板頁面新增公司商標：可自訂元素](fundamentals/customize-branding.md) |
| 等候幾分鐘讓變更完全生效 |  |
| 以 POC 使用者認證登入 https://myapps.microsoft.com |  |
| 在瀏覽器中確認外觀及操作 | [在登入和存取面板頁面新增公司商標](fundamentals/customize-branding.md) |
| (選擇性) 在其他裝置中確認外觀及操作 |  |

### <a name="considerations"></a>考量

如果在自訂後舊的外觀及操作仍在，則請排清瀏覽器用戶端快取，然後重試作業。

## <a name="group-based-licensing"></a>群組型授權

預估完成時間：10 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 所有 POC 使用者都是安全性群組 (不論是雲端還是內部部署) 的成員 | [在 Azure Active Directory 中建立群組和新增使用者](fundamentals/active-directory-groups-create-azure-portal.md) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 移至「Azure AD 管理入口網站」中的 [授權] 刀鋒視窗 | [Azure AD 管理入口網站：授權](https://portal.azure.com/#blade/Microsoft_AAD_IAM/LicensesMenuBlade/Products) |
| 將授權指派給含有 POC 使用者的安全性群組。 | [將授權指派給 Azure Active Directory 中的使用者群組](users-groups-roles/licensing-groups-assign.md) |

### <a name="considerations"></a>考量

如果有任何問題，請移至[使用群組來管理 Azure Active Directory 授權的案例、限制及已知問題](users-groups-roles/licensing-group-advanced.md)

## <a name="saas-federated-sso-configuration"></a>SaaS 同盟 SSO 組態

預估完成時間：60 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 可用的 SaaS 應用程式測試環境。 在本指南中，我們將使用 ServiceNow 作為範例。<br/>強烈建議您使用測試執行個體，以將瀏覽現有資料品質與對應的阻力降到最低。 | 請移至 https://developer.servicenow.com/app.do#!/home 以開始進行取得測試執行個體的程序 |
| 對 ServiceNow 管理主控台的存取權 | [教學課程：Azure Active Directory 與 ServiceNow 整合](saas-apps/servicenow-tutorial.md) |
| 一組要作為應用程式指派目標的使用者。 建議使用包含 PoC 使用者的安全性群組。 <br/>如果無法建立該群組，則請將使用者直接指派給 PoC 的應用程式 | [在 Azure Active Directory 中將使用者或群組指派給企業應用程式](manage-apps/assign-user-or-group-access-portal.md) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 將來自「Microsoft 文件」的教學課程分享給所有執行者  | [教學課程：Azure Active Directory 與 ServiceNow 整合](saas-apps/servicenow-tutorial.md) |
| 設定工作會議並與每個執行者一起依照教學課程步驟操作。 | [教學課程：Azure Active Directory 與 ServiceNow 整合](saas-apps/servicenow-tutorial.md) |
| 將應用程式指派給「必要條件」中所識別的群組。 如果 POC 的範圍內具有條件式存取，您可以稍後再重新瀏覽該部分並新增 MFA 等。 <br/>請注意，這會開始佈建程序 (如果已設定) |  [在 Azure Active Directory 中將使用者或群組指派給企業應用程式](manage-apps/assign-user-or-group-access-portal.md) <br/>[在 Azure Active Directory 中建立群組和新增使用者](fundamentals/active-directory-groups-create-azure-portal.md) |
| 使用「Azure AD 管理入口網站」從資源庫新增 ServiceNow 應用程式| [Azure AD 管理入口網站：企業應用程式](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/Overview) <br/>[Azure Active Directory 中企業應用程式管理的新功能](active-directory-enterprise-apps-whats-new-azure-portal.md) |
| 在 ServiceNow 應用程式的 [單一登入] 刀鋒視窗中，啟用 [SAML 登入] |  |
| 在 [單一登入 URL] 和 [識別碼] 欄位中填入您的 ServiceNow URL<br/>核取 [使新憑證為使用中] 方塊<br/>並 [儲存] 設定 |  |
| 開啟面板下部的 [設定 ServiceNow] 刀鋒視窗，以檢視可供您設定 ServiceNow 的自訂指示 |  |
| 依照指示操作來設定 ServiceNow |  |
| 在 ServiceNow 應用程式的 [佈建] 刀鋒視窗中，啟用 [自動] 佈建 | [在新的 Azure 入口網站中管理企業應用程式的使用者帳戶佈建](manage-apps/configure-automatic-user-provisioning-portal.md) |
| 等候幾分鐘讓佈建完成。  此時，您可以查看佈建報告 |  |
| 以擁有存取權限的測試使用者身分登入 https://myapps.microsoft.com/ | [何謂存取面板？](user-help/active-directory-saas-access-panel-introduction.md) |
| 按一下剛建立之應用程式的圖格。 確認存取 |  |
| 您可以視需要查看應用程式使用情況報告。 請注意，會有一些延遲，因此您需要等候一些時間，才能在報告中看到流量。 | [Azure Active Directory 入口網站中的登入活動報告：受控應用程式的使用情況](reports-monitoring/concept-sign-ins.md#usage-of-managed-applications)<br/>[Azure Active Directory 報告保留原則](reports-monitoring/reference-reports-data-retention.md) |

### <a name="considerations"></a>考量

1. 上述[教學課程](saas-apps/servicenow-tutorial.md)是參考舊的 Azure AD 管理體驗。 但 PoC 是根據[快速入門](active-directory-enterprise-apps-whats-new-azure-portal.md#quickstart-get-going-with-your-new-application-right-away)體驗。
2. 如果資源庫中沒有目標應用程式，則您可以使用「自備應用程式」。 深入了解：[Azure Active Directory 中企業應用程式管理的新功能：從一個位置新增自訂應用程式](active-directory-enterprise-apps-whats-new-azure-portal.md#add-custom-applications-from-one-place)

## <a name="saas-password-sso-configuration"></a>SaaS 密碼 SSO 組態

預估完成時間：15 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| SaaS 應用程式的測試環境。 其中一個「密碼 SSO」範例就是 HipChat 和 Twitter。 針對任何其他應用程式，您需要含有 HTML 登入表單之頁面的確切 URL。 | [Microsoft Azure Marketplace 上的 Twitter](https://azuremarketplace.microsoft.com/marketplace/apps/aad.twitter)<br/>[Microsoft Azure Marketplace 上的 HipChat](https://azuremarketplace.microsoft.com/marketplace/apps/aad.hipchat) |
| 應用程式的測試帳戶。 | [註冊 Twitter](https://twitter.com/signup?lang=en)<br/>[免費註冊：HipChat](https://www.hipchat.com/sign_up) |
| 一組要作為應用程式指派目標的使用者。 建議使用包含這些使用者的安全性群組。 | [在 Azure Active Directory 中將使用者或群組指派給企業應用程式](manage-apps/assign-user-or-group-access-portal.md) |
| 對電腦的本機系統管理員存取權，以部署適用於 Internet Explorer、Chrome 或 Firefox 的「存取面板擴充功能」 | [適用於 IE 的存取面板擴充功能](https://account.activedirectory.windowsazure.com/Applications/Installers/x64/Access%20Panel%20Extension.msi)<br/>[適用於 Chrome 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=311859&clcid=0x409)<br/>[適用於 Firefox 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=626998&clcid=0x409) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 安裝瀏覽器擴充功能 | [適用於 IE 的存取面板擴充功能](https://account.activedirectory.windowsazure.com/Applications/Installers/x64/Access%20Panel%20Extension.msi)<br/>[適用於 Chrome 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=311859&clcid=0x409)<br/>[適用於 Firefox 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=626998&clcid=0x409) |
| 從資源庫設定應用程式 | [Azure Active Directory 中企業應用程式管理的新功能：全新和改進的應用程式資源庫](active-directory-enterprise-apps-whats-new-azure-portal.md#improvements-to-the-azure-active-directory-application-gallery) |
| 設定密碼 SSO | [在新的 Azure 入口網站中管理企業應用程式的單一登入：密碼型登入](manage-apps/what-is-single-sign-on.md#how-does-single-sign-on-with-azure-active-directory-work)。|
| 將應用程式指派給「必要條件」中所識別的群組 | [在 Azure Active Directory 中將使用者或群組指派給企業應用程式](manage-apps/assign-user-or-group-access-portal.md) |
| 以擁有存取權限的測試使用者身分登入 https://myapps.microsoft.com/ |  |
| 按一下剛建立之應用程式的圖格。 | [何謂存取面板？：不含身分識別佈建的密碼型 SSO](user-help/active-directory-saas-access-panel-introduction.md#password-based-sso-without-identity-provisioning) |
| 提供應用程式認證 | [何謂存取面板？：不含身分識別佈建的密碼型 SSO](user-help/active-directory-saas-access-panel-introduction.md#password-based-sso-without-identity-provisioning) |
| 關閉瀏覽器並重新執行登錄。 這次使用者應該會看到順暢的應用程式存取。 |  |
| 您可以視需要查看應用程式使用情況報告。 請注意，會有一些延遲，因此您需要等候一些時間，才能在報告中看到流量。 | [Azure Active Directory 入口網站中的登入活動報告：受控應用程式的使用情況](reports-monitoring/concept-sign-ins.md#usage-of-managed-applications)<br/>[Azure Active Directory 報告保留原則](reports-monitoring/reference-reports-data-retention.md) |

### <a name="considerations"></a>考量

如果資源庫中沒有目標應用程式，則您可以使用「自備應用程式」。 深入了解：[Azure Active Directory 中企業應用程式管理的新功能：從一個位置新增自訂應用程式](active-directory-enterprise-apps-whats-new-azure-portal.md#add-custom-applications-from-one-place)

 請記住下列需求：
   * 應用程式應該要有已知的登入 URL
   * 登入頁面應該包含一個 HTML 表單，其中具有一或多個可供瀏覽器擴充功能自動填入的欄位。 至少，應該包含使用者名稱和密碼。

## <a name="saas-shared-accounts-configuration"></a>SaaS 共用帳戶組態

預估完成時間：30 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 事先備妥的目標應用程式清單和確切的登入 URL。 您可以使用 Twitter 作為範例。 | [Microsoft Azure Marketplace 上的 Twitter](https://azuremarketplace.microsoft.com/marketplace/apps/aad.twitter)<br/>[註冊 Twitter](https://twitter.com/signup?lang=en) |
| 此 SaaS 應用程式的共用認證。 | [使用 Azure AD 來共用帳戶](active-directory-sharing-accounts.md)<br/>[Azure AD 的 Facebook、Twitter 及 LinkedIn 自動密碼變換現已提供預覽版！ - Enterprise Mobility and Security 部落格] (https://blogs.technet.microsoft.com/enterprisemobility/2015/02/20/azure-ad-automated-password-roll-over-for-facebook-twitter-and-linkedin-now-in-preview/ ) |
| 至少兩個將存取相同帳戶之小組成員的認證。 他們必須是安全性群組的成員。 | [在 Azure Active Directory 中將使用者或群組指派給企業應用程式](manage-apps/assign-user-or-group-access-portal.md) |
| 對電腦的本機系統管理員存取權，以部署適用於 Internet Explorer、Chrome 或 Firefox 的「存取面板擴充功能」 | [適用於 IE 的存取面板擴充功能](https://account.activedirectory.windowsazure.com/Applications/Installers/x64/Access%20Panel%20Extension.msi)<br/>[適用於 Chrome 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=311859&clcid=0x409)<br/>[適用於 Firefox 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=626998&clcid=0x409) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 安裝瀏覽器擴充功能 | [適用於 IE 的存取面板擴充功能](https://account.activedirectory.windowsazure.com/Applications/Installers/x64/Access%20Panel%20Extension.msi)<br/>[適用於 Chrome 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=311859&clcid=0x409)<br/>[適用於 Firefox 的存取面板擴充功能](https://go.microsoft.com/fwLink/?LinkID=626998&clcid=0x409) |
| 從資源庫設定應用程式 | [Azure Active Directory 中企業應用程式管理的新功能：全新和改進的應用程式資源庫](active-directory-enterprise-apps-whats-new-azure-portal.md#improvements-to-the-azure-active-directory-application-gallery) |
| 設定密碼 SSO | [在新的 Azure 入口網站中管理企業應用程式的單一登入：密碼型登入](manage-apps/what-is-single-sign-on.md#how-does-single-sign-on-with-azure-active-directory-work)。|
| 為「必要條件」中所識別的群組指派認證時，將應用程式指派給他們 | [在 Azure Active Directory 中將使用者或群組指派給企業應用程式](manage-apps/assign-user-or-group-access-portal.md) |
| 以不同的使用者身分登入來以「相同共用帳戶」身分存取應用程式  |  |
| 您可以視需要查看應用程式使用情況報告。 請注意，會有一些延遲，因此您需要等候一些時間，才能在報告中看到流量。 | [Azure Active Directory 入口網站中的登入活動報告：受控應用程式的使用情況](reports-monitoring/concept-sign-ins.md#usage-of-managed-applications)<br/>[Azure Active Directory 報告保留原則](reports-monitoring/reference-reports-data-retention.md) |


### <a name="considerations"></a>考量

如果資源庫中沒有目標應用程式，則您可以使用「自備應用程式」。 深入了解：[Azure Active Directory 中企業應用程式管理的新功能：從一個位置新增自訂應用程式](active-directory-enterprise-apps-whats-new-azure-portal.md#add-custom-applications-from-one-place)

 請記住下列需求：
   * 應用程式應該要有已知的登入 URL
   * 登入頁面應該包含一個 HTML 表單，其中具有一或多個可供瀏覽器擴充功能自動填入的欄位。 至少，應該包含使用者名稱和密碼。

## <a name="app-proxy-configuration"></a>應用程式 Proxy 組態

預估完成時間：20 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| Microsoft Azure AD 基本或進階訂用帳戶，以及您是全域管理員的 Azure AD 目錄 | [Azure Active Directory 版本](fundamentals/active-directory-whatis.md) |
| 一個裝載於內部部署環境而您想要設定以供遠端存取的 Web 應用程式 |  |
| 執行 Windows Server 2012 R2 或 Windows 8.1 或更新版本的伺服器，您可以在該伺服器上安裝「應用程式 Proxy 連接器」 | [了解 Azure AD 應用程式 Proxy 連接器](manage-apps/application-proxy-connectors.md) |
| 如果路徑中有防火牆，請確定防火牆已開啟，如此連接器才能對「應用程式 Proxy」提出 HTTPS (TCP) 要求 | [在 Azure 入口網站中啟用應用程式 Proxy：應用程式 Proxy 先決條件](manage-apps/application-proxy-enable.md#application-proxy-prerequisites) |
| 如果您的組織使用 Proxy 伺服器來連接到網際網路，請參閱部落格文章＜使用現有的內部部署 Proxy 伺服器＞，以詳細了解如何設定這些伺服器 | [使用現有的內部部署 Proxy 伺服器](manage-apps/application-proxy-configure-connectors-with-proxy-servers.md) |


### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 在伺服器上安裝連接器 | [在 Azure 入口網站中啟用應用程式 Proxy：安裝並註冊連接器](manage-apps/application-proxy-enable.md#install-and-register-a-connector) |
| 在 Azure AD 中將內部部署應用程式發佈為「應用程式 Proxy」應用程式 | [使用 Azure AD 應用程式 Proxy 發佈應用程式](manage-apps/application-proxy-publish-azure-portal.md) |
| 指派測試使用者 | [使用 Azure AD 應用程式 Proxy 發佈應用程式 - 新增測試使用者](manage-apps/application-proxy-publish-azure-portal.md#add-a-test-user) |
| (選擇性) 為您的使用者設定單一登入體驗 | [使用 Azure AD 應用程式 Proxy 提供單一登入](manage-apps/application-proxy-configure-single-sign-on-password-vaulting.md) |
| 以指派的使用者身分登入 MyApps 入口網站來測試應用程式 | https://myapps.microsoft.com |

### <a name="considerations"></a>考量

1. 雖然我們建議將連接器放在您的公司網路中，但在某些情況下，您會發現將它放在雲端會有較佳的效能。 深入了解：[使用 Azure Active Directory 應用程式 Proxy 時的網路拓撲考量](manage-apps/application-proxy-network-topology.md)
2. 如需進一步的安全性詳細資料，以及這如何僅透過維護輸出連線來提供特別安全的遠端存取解決方案，請參閱：[使用 Azure AD 應用程式 Proxy 從遠端存取應用程式的安全性考量](manage-apps/application-proxy-security.md)

## <a name="generic-ldap-connector-configuration"></a>一般 LDAP 連接器組態

預估完成時間：60 分鐘

> [!IMPORTANT]
> 這是一種需要對 FIM/MIM 有某種程度熟悉的進階組態。 如果在生產環境中使用，建議您瀏覽[頂級支援](https://support.microsoft.com/premier)以查看此組態的相關問題。

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 已安裝及設定 Azure AD Connect | 構成要素：[目錄同步作業 - 密碼雜湊同步處理](#directory-synchronization--password-hash-sync-phs--new-installation) |
| 符合需求的 ADLDS 執行個體 | [一般 LDAP 連接器技術參考：一般 LDAP 連接器概觀](https://docs.microsoft.com/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericldap#overview-of-the-generic-ldap-connector) |
| 使用者所使用之工作負載及與這些工作負載關聯之屬性的清單 | [Azure AD Connect 同步處理：同步處理至 Azure Active Directory 的屬性](hybrid/reference-connect-sync-attributes-synchronized.md) |


### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 新增一般 LDAP 連接器 | [一般 LDAP 連接器技術參考：建立新的連接器](https://docs.microsoft.com/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericldap#create-a-new-connector) |
| 為已建立的連接器建立執行設定檔 (完整匯入、差異匯入、完整同步處理、差異同步處理、匯出) | [建立管理代理程式執行設定檔 (英文)](https://technet.microsoft.com/library/jj590219(v=ws.10).aspx)<br/> [使用連接器搭配 Auzre AD Connect Sync Service Manager](hybrid/how-to-connect-sync-service-manager-ui-connectors.md)|
| 執行完整匯入設定檔，並確認連接器空間中有物件 | [搜尋連接器空間物件](https://technet.microsoft.com/library/jj590287(v=ws.10).aspx)<br/>[使用連接器搭配 Auzre AD Connect Sync Service Manager：搜尋連接器空間](hybrid/how-to-connect-sync-service-manager-ui-connectors.md#search-connector-space) |
| 建立同步處理規則，讓 Metaverse 中的物件具有必要的工作負載屬性 | [Azure AD Connect 同步處理：變更預設組態的最佳做法：同步處理規則的變更](hybrid/how-to-connect-sync-best-practices-changing-default-configuration.md#changes-to-synchronization-rules)<br/>[Azure AD Connect 同步處理：了解宣告式佈建](hybrid/concept-azure-ad-connect-sync-declarative-provisioning.md)<br/>[Azure AD Connect 同步處理：了解宣告式佈建運算式](hybrid/concept-azure-ad-connect-sync-declarative-provisioning-expressions.md) |
| 啟動完整的同步處理週期 | [Azure AD Connect 同步處理：排程器：啟動排程器](hybrid/how-to-connect-sync-feature-scheduler.md#start-the-scheduler) |
| 發生問題時進行疑難排解 | [針對未同步至 Azure AD 的物件進行疑難排解](hybrid/tshoot-connect-object-not-syncing.md) |
| 確認 LDAP 使用者可以登入及存取應用程式 | https://myapps.microsoft.com |

### <a name="considerations"></a>考量

> [!IMPORTANT]
> 這是一種需要對 FIM/MIM 有某種程度熟悉的進階組態。 如果在生產環境中使用，建議您瀏覽[頂級支援](https://support.microsoft.com/premier)以查看此組態的相關問題。

## <a name="groups---delegated-ownership"></a>群組 - 委派的擁有權

預估完成時間：10 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 已設定 SaaS 應用程式 (同盟 SSO 或密碼 SSO) | 構成要素：[SaaS 同盟 SSO 組態](#saas-federated-sso-configuration) |
| 已識別在 #1 中獲指派應用程式存取權的「雲端群組」 | 構成要素：[SaaS 同盟 SSO 組態](#saas-federated-sso-configuration) <br/>[在 Azure Active Directory 中建立群組和新增使用者](fundamentals/active-directory-groups-create-azure-portal.md) |
| 有群組擁有者的認證可用 | [使用 Azure Active Directory 群組來管理資源的存取權](fundamentals/active-directory-manage-groups.md) |
| 已識別存取應用程式之資訊工作者的認證 | [何謂存取面板？](user-help/active-directory-saas-access-panel-introduction.md) |


### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 識別已獲授與應用程式存取權的群組，並設定所指定群組的擁有者| [在 Azure Active Directory 中管理群組的設定](fundamentals/active-directory-groups-settings-azure-portal.md) |
| 以群組擁有者身分登入，在存取面板的 [群組] 索引標籤中查看群組成員資格 | [Azure Active Directory 群組管理頁面](https://account.activedirectory.windowsazure.com/r#/groups) |
| 新增您想要測試的資訊工作者 |  |
| 以資訊工作者身分登入，確認該圖格可用 | [何謂存取面板？](user-help/active-directory-saas-access-panel-introduction.md) |

### <a name="considerations"></a>考量

如果應用程式已啟用佈建，您可能需要先等候幾分鐘讓佈建完成，才能以資訊工作者身分存取應用程式。

## <a name="saas-and-identity-lifecycle"></a>SaaS 和身分識別生命週期

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 已設定 SaaS 應用程式 (同盟 SSO 或密碼 SSO) | 構成要素：[SaaS 同盟 SSO 組態](#saas-federated-sso-configuration) |
| 已識別在 #1 中獲指派應用程式存取權的「雲端群組」 | 構成要素：[SaaS 同盟 SSO 組態](#saas-federated-sso-configuration) <br/>[在 Azure Active Directory 中建立群組和新增使用者](fundamentals/active-directory-groups-create-azure-portal.md) |
| 已識別存取應用程式之資訊工作者的認證 | [何謂存取面板？](user-help/active-directory-saas-access-panel-introduction.md) |


### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 從應用程式所屬的群組中移除使用者 | [管理 Azure Active Directory 租用戶中使用者的群組成員資格](fundamentals/active-directory-groups-members-azure-portal.md) |
| 等候幾分鐘來執行取消佈建 | [Azure AD 中的自動化 SaaS 應用程式使用者佈建：自動化佈建如何運作？](manage-apps/user-provisioning.md#how-does-automatic-provisioning-work) |
| 在個別的瀏覽器工作階段上，以資訊工作者身分登入「我的應用程式」入口網站，並確認該圖格已經消失 | http://myapps.microsoft.com |


### <a name="considerations"></a>考量

推測離職者和/或准假案例的 PoC 案例。 如果使用者在內部部署 AD 中被停用或移除，就不再能夠登入 SaaS 應用程式。

## <a name="self-service-access-to-application-management"></a>以自助方式存取應用程式管理

預估完成時間：10 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 識別屬於安全性群組、將要求應用程式存取權的 POC 使用者 | 構成要素：[SaaS 同盟 SSO 組態](#saas-federated-sso-configuration) |
| 已部署「目標應用程式」 | 構成要素：[SaaS 同盟 SSO 組態](#saas-federated-sso-configuration) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 移至「Azure AD 管理入口網站」中的 [企業應用程式] 刀鋒視窗 | [Azure AD 管理入口網站：企業應用程式](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps/menuId/) |
| 根據自助必要條件設定應用程式 | [Azure Active Directory 中企業應用程式管理的新功能：設定自助式應用程式存取](active-directory-enterprise-apps-whats-new-azure-portal.md#configure-self-service-application-access) |
| 以資訊工作者身分登入「我的應用程式」入口網站 | http://myapps.microsoft.com |
| 注意頁面頂端的 [+加入應用程式] 按鈕。 請使用它來存取應用程式 |  |

### <a name="considerations"></a>考量

所選應用程式可能會有佈建需求，因此直接移至該應用程式可能會造成一些錯誤。 如果所選應用程式支援以 Azure AD 進行佈建並且已設定，您便可以利用這個機會來顯示整個端對端運作的流程。 如需進一步的建議，請參閱 [SaaS 同盟 SSO 組態](#saas-federated-sso-configuration)的構成要素

## <a name="self-service-password-reset"></a>自助式密碼重設

預估完成時間：15 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 在您的租用戶中啟用自助式密碼管理。 | [IT 系統管理員的 Azure Active Directory 密碼重設](user-help/active-directory-passwords-update-your-own-password.md) |
| 啟用密碼回寫以管理來自內部部署環境的密碼。 請注意，這需要特定的 Azure AD Connect 版本 | [密碼回寫先決條件](authentication/howto-sspr-writeback.md) |
| 識別將使用此功能的 PoC 使用者，並確定他們是安全性群組的成員。 使用者必須是非系統管理員，才能完整展示此功能 | [自訂：Azure AD 密碼管理：限制對密碼重設的存取](authentication/howto-sspr-writeback.md) |


### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 瀏覽至「Azure AD 管理入口網站」中的 [密碼重設] | [Azure AD 管理入口網站：密碼重設](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/PasswordReset) |
| 決定密碼重設原則。 針對 POC 用途，您可以使用通話和問與答。建議啟用在登入存取面板時要求註冊的功能 |  |
| 以資訊工作者身分登出和登入 |  |
| 提供依步驟 2 設定的「自助式密碼重設」資料 | https://aka.ms/ssprsetup |
| 關閉瀏覽器 |  |
| 以您在步驟 4 中使用的資訊工作者身分重新開始登入程序 |  |
| 重設密碼 | [更新自己的密碼：重設密碼](user-help/active-directory-passwords-update-your-own-password.md) |
| 嘗試以您的新密碼登入 Azure AD 及內部部署資源 |  |

### <a name="considerations"></a>考量

1. 如果升級 Azure AD Connect 將會造成不便，請考慮將它用於雲端帳戶，或以另一個環境作為其示範背景
2. 系統管理員具有不同的原則，因此使用系統管理員帳戶來重設密碼可能會誤導 PoC 而造成混淆。 請務必使用一般使用者帳戶來測試重設作業


## <a name="azure-multi-factor-authentication-with-phone-calls"></a>使用通話進行 Azure Multi-Factor Authentication

預估完成時間：10 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 識別將使用 MFA 的 POC 使用者  |  |
| 可供 MFA 挑戰使用的收訊良好手機  | [什麼是 Azure Multi-Factor Authentication？](authentication/multi-factor-authentication.md) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 移至「Azure AD 管理入口網站」中的 [使用者和群組] 刀鋒視窗 | [Azure AD 管理入口網站：使用者和群組](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/Overview/menuId/) |
| 關閉 [所有使用者] 刀鋒視窗 |  |
| 在頂端列中，選擇 [Multi-Factor Authentication] 按鈕 | Azure MFA 入口網站的直接 URL： https://aka.ms/mfaportal |
| 在 [使用者] 設定中，選取 PoC 使用者並啟用他們來進行 MFA | [Azure Multi-Factor Authentication 中的使用者狀態](authentication/howto-mfa-userstates.md) |
| 以 PoC 使用者身分登入，然後逐步執行提出證明的程序  |  |

### <a name="considerations"></a>考量

1. 此構成要素中的 PoC 步驟會明確地為使用者的所有登入設定 MFA。 此外，還有「條件式存取」和 Identity Protection 之類的其他工具，可在更具目標性的案例中進行 MFA。 這將是從 POC 移到生產環境時需要考量的事項。
2. 為了方便起見，此構成要素中的 PoC 步驟會明確地使用「通話」作為 MFA 方法。 當您從 POC 轉換到生產環境時，建議您儘可能使用 [Microsoft Authenticator](user-help/microsoft-authenticator-app-how-to.md) 之類的應用程式作為第二要素。
深入了解：[DRAFT NIST 特殊出版品 800-63B (英文)](https://pages.nist.gov/800-63-3/sp800-63b.html)

## <a name="mfa-conditional-access-for-saas-applications"></a>SaaS 應用程式的 MFA 條件式存取

預估完成時間：10 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 識別要作為原則目標的 PoC 使用者。 這些使用者應該在安全性群組中，以限定條件式存取原則的範圍 | [SaaS 同盟 SSO 組態](#saas-federated-sso-configuration) |
| 已設定 SaaS 應用程式 |  |
| 已將 PoC 使用者指派給應用程式 |  |
| 有 POC 使用者的認證可用 |  |
| 已為 MFA 註冊 POC 使用者。 使用收訊良好的手機 | https://aka.ms/ssprsetup |
| 內部網路中的裝置。 在內部位址範圍內設定的 IP 位址 | 尋找您的 IP 位址： https://www.bing.com/search?q=what%27s+my+ip |
| 外部網路中的裝置 (可以是使用電訊廠商行動網路的手機) |  |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 移至「Azure AD 管理入口網站」的 [條件式存取] 刀鋒視窗 | [Azure AD 管理入口網站：條件式存取](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ConditionalAccessBlade/Policies) |
| 建立「條件式存取」原則：<br/>- 以 [使用者和群組] 底下的「PoC 使用者」為目標<br/>- 以 [雲端應用程式] 底下的「PoC 應用程式」為目標<br/>- 以所有位置為目標，但 [條件] -> [位置] 底下的受信任位置除外 **注意：** 受信任的 IP 是在 [MFA 入口網站](https://account.activedirectory.windowsazure.com/UserManagement/MfaSettings.aspx)中設定<br/>- 在 [授權] 底下要求多重要素驗證 | [建立條件式存取原則](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-mfa#create-your-conditional-access-policy) \(英文\) |
| 從公司網路內部存取應用程式 | [測試條件式存取原則](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-mfa#test-your-conditional-access-policy) \(英文\) |
| 從公用網路存取應用程式 | [測試條件式存取原則](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-mfa#test-your-conditional-access-policy) \(英文\) |

### <a name="considerations"></a>考量

如果您要使用同盟，您可以使用內部部署的「識別提供者」(IdP)，透過宣告來溝通內部/外部公司網路狀態。 您可以使用這項技術而不需管理 IP 位址清單，在大型組織中，評估和管理 IP 位址可能會相當複雜。 在該設定中，您需要解釋「網路漫遊」情況 (使用者從內部網路登入，並在已登入的情況下切換位置，例如咖啡店)，並確定您了解該意涵。 深入了解：[使用 Azure Multi-Factor Authentication 與 AD FS 保護雲端資源：同盟使用者的可信任 IP](authentication/howto-mfa-adfs.md#trusted-ips-for-federated-users)

## <a name="privileged-identity-management-pim"></a>Privileged Identity Management (PIM)

預估完成時間：15 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 識別將參與 PIM 之 POC 的全域管理員 | [開始使用 Azure AD Privileged Identity Management](privileged-identity-management/pim-getting-started.md) |
| 識別將成為「安全性系統管理員」的全域管理員 | [開始使用 Azure AD Privileged Identity Management](privileged-identity-management/pim-getting-started.md)<br/> [Azure Active Directory PIM 中不同的系統管理角色](privileged-identity-management/pim-roles.md) |
| 選擇性：確認全域管理員是否具有可在 PIM 中執行電子郵件通知的電子郵件存取權 | [什麼是 Azure AD Privileged Identity Management？：設定角色啟用設定](privileged-identity-management/pim-configure.md#configure-the-role-activation-settings)


### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 以全域管理員身分 (GA) 登入 https://portal.azure.com，並啟動載入 PIM 刀鋒視窗。 執行此步驟的「全域管理員」會植入為安全性系統管理員。  讓我們將此執行者稱為 GA1 | [在 Azure AD Privileged Identity Management 中使用安全性精靈](privileged-identity-management/pim-security-wizard.md) |
| 識別全域管理員，並將他們從永久移至合格。 為了清楚起見，這應該與步驟 1 中使用的管理員不同。 讓我們將此執行者稱為 GA2 | [Azure AD Privileged Identity Management：如何新增或移除使用者角色](privileged-identity-management/pim-how-to-add-role-to-user.md)<br/>[什麼是 Azure AD Privileged Identity Management？：設定角色啟用設定](privileged-identity-management/pim-configure.md#configure-the-role-activation-settings)  |
| 現在，以 GA2 身分登入 https://portal.azure.com，並嘗試變更 [使用者設定]。 請注意，部分選項是呈現灰色。 | |
| 在新索引標籤及與步驟 3 相同的工作階段中，現在瀏覽至 https://portal.azure.com，然後將 PIM 刀鋒視窗新增至儀表板中。 | [開始使用 PIM](privileged-identity-management/pim-getting-started.md) |
| 要求啟用「全域管理員」角色 | [如何在 Azure AD Privileged Identity Management 中啟用或停用角色：啟用角色](privileged-identity-management/pim-how-to-activate-role.md#activate-a-role) |
| 請注意，如果 GA2 從未註冊 MFA，將必須註冊 Azure MFA |  |
| 回到步驟 3 中的原始索引標籤，然後按一下瀏覽器中的 [重新整理] 按鈕。 請注意，您現在已具有能夠變更 [使用者設定] 的存取權 | |
| (選擇性) 如果您的全域管理員已啟用電子郵件功能，則只要查看 GA1 和 GA2 的收件匣，就可以看到已啟用該角色的通知 |  |
| 8 查看稽核歷程記錄並檢視報告，以確認顯示 GA2 的權限提升。 | [什麼是 Azure AD Privileged Identity Management？：檢閱角色活動](privileged-identity-management/pim-configure.md#review-role-activity) |

### <a name="considerations"></a>考量

此功能是 Azure AD Premium P2 和/或 EMS E5 的一部分

## <a name="discovering-risk-events"></a>探索風險事件

預估完成時間：20 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 已下載並安裝 Tor 瀏覽器的裝置 | [下載 Tor 瀏覽器](https://www.torproject.org/projects/torbrowser.html.en#downloads) |
| 對 POC 使用者的存取權以執行登入 | [Azure Active Directory Identity Protection 腳本](identity-protection/playbook.md) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 開啟 Tor 瀏覽器 | [下載 Tor 瀏覽器](https://www.torproject.org/projects/torbrowser.html.en#downloads) |
| 以 POC 使用者帳戶登入 https://myapps.microsoft.com | [Azure Active Directory Identity Protection 腳本：模擬風險事件](identity-protection/playbook.md#simulating-risk-events) |
| 等候 5-7 分鐘 |  |
| 以全域管理員身分登入 https://portal.azure.com，然後開啟 [Identity Protection] 刀鋒視窗 | https://aka.ms/aadipgetstarted |
| 開啟 [風險事件] 刀鋒視窗。 您應該會在 [從匿名 IP 位址登入] 底下看到一個項目  | [Azure Active Directory Identity Protection 腳本：模擬風險事件](identity-protection/playbook.md#simulating-risk-events) |

### <a name="considerations"></a>考量

此功能是 Azure AD Premium P2 和/或 EMS E5 的一部分

## <a name="deploying-sign-in-risk-policies"></a>部署登入風險原則

預估完成時間：10 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 已下載並安裝 Tor 瀏覽器的裝置 | [下載 Tor 瀏覽器](https://www.torproject.org/projects/torbrowser.html.en#downloads) |
| 以 POC 使用者身分存取來執行登入測試 |  |
| 已向 MFA 註冊 POC 使用者。 請務必使用收訊良好的手機 | 構成要素：[使用通話進行 Azure Multi-Factor Authentication](#azure-multi-factor-authentication-with-phone-calls) |


### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 以全域管理員身分登入 https://portal.azure.com，然後開啟 [Identity Protection] 刀鋒視窗 | https://aka.ms/aadipgetstarted |
| 依下列方式啟用登入風險原則：<br/>- 指派至：POC 使用者<br/>- 條件：中度或更高的登入風險 (從匿名位置登入是視為中度風險層級)<br/>- 控制項：需要 MFA | [Azure Active Directory Identity Protection 腳本：登入風險](identity-protection/playbook.md) |
| 開啟 Tor 瀏覽器 | [下載 Tor 瀏覽器](https://www.torproject.org/projects/torbrowser.html.en#downloads) |
| 以 PoC 使用者帳戶登入 https://myapps.microsoft.com |  |
| 注意 MFA 挑戰 | [使用 Azure AD Identity Protection 時的登入體驗：有風險的登入復原](identity-protection/flows.md#risky-sign-in-recovery)

### <a name="considerations"></a>考量

此功能是 Azure AD Premium P2 和/或 EMS E5 的一部分。 若要深入了解風險事件，請瀏覽：[Azure Active Directory 風險事件](reports-monitoring/concept-risk-events.md)

## <a name="configuring-certificate-based-authentication"></a>設定憑證式驗證

預估完成時間：20 分鐘

### <a name="pre-requisites"></a>先決條件

| 必要條件 | 資源 |
| --- | --- |
| 具有從「企業 PKI」佈建之使用者憑證的裝置 (Windows、iOS 或 Android) | [部署使用者憑證](https://msdn.microsoft.com/library/cc770857.aspx) |
| 與 ADFS 同盟的 Azure AD 網域 | [Azure AD Connect 和同盟](hybrid/how-to-connect-fed-whatis.md)<br/>[Active Directory 憑證服務概觀](https://technet.microsoft.com/library/hh831740.aspx)|
| 針對 iOS 裝置，安裝 Microsoft Authenticator 應用程式 | [開始使用 Microsoft Authenticator 應用程式](user-help/microsoft-authenticator-app-how-to.md) |

### <a name="steps"></a>步驟

| 步驟 | 資源 |
| --- | --- |
| 在 ADFS 上啟用「憑證驗證」 | [設定驗證原則：在 Windows Server 2012 R2 全域設定主要驗證](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/operations/configure-authentication-policies#to-configure-primary-authentication-globally-in-windows-server-2012-r2) |
| 選擇性：在 Azure AD 中為 Exchange Active Sync 用戶端啟用「憑證驗證」 | [開始在 Azure Active Directory 中使用憑證式驗證](./authentication/active-directory-certificate-based-authentication-get-started.md) |
| 瀏覽至「存取面板」並使用「使用者憑證」進行驗證 | https://myapps.microsoft.com |

### <a name="considerations"></a>考量

若要深入了解這項部署的注意事項，請瀏覽[ADFS：使用 Azure AD 和 Office 365 進行憑證驗證 (英文)](https://blogs.msdn.microsoft.com/samueld/2016/07/19/adfs-certauth-aad-o365/)



> [!NOTE]
> 使用者憑證的持有應該受到保護。 可透過管理裝置或使用 PIN (在使用智慧卡的情況下) 的方式來保護。



[!INCLUDE [active-directory-playbook-toc](../../includes/active-directory-playbook-steps.md)]
