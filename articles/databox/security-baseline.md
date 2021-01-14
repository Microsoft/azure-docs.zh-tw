---
title: 適用于 Azure 資料箱的 Azure 安全性基準
description: 適用于 Azure 資料箱的 Azure 安全性基準
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 2773a76ed1166a26ebc12b97550876eaa66bad14
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98197314"
---
# <a name="azure-security-baseline-for-azure-data-box"></a>適用于 Azure 資料箱的 Azure 安全性基準

適用于 Azure 資料箱的 Azure 安全性基準包含可協助您改善部署安全性狀態的建議。

此服務的基準取自 [Azure 安全性效能評定 1.0 版](../security/benchmarks/overview.md)，其會提供如何在 Azure 上使用最佳做法指引來保護雲端解決方案的建議。

如需詳細資訊，請參閱 [Azure 安全性基準概觀](../security/benchmarks/security-baselines-overview.md)。

## <a name="network-security"></a>網路安全性

*如需詳細資訊，請參閱 [安全性控制：網路安全性](../security/benchmarks/security-control-network-security.md)。*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1：保護虛擬網路內的 Azure 資源

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。 您可以透過 Azure 入口網站，控制從資料箱到 Azure 託管儲存體的流量。 當您利用資料箱時，資料會透過 Azure 骨幹傳送。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2：監視和記錄虛擬網路、子網和 Nic 的設定和流量

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。 您可以透過 Azure 入口網站，控制從資料箱到 Azure 託管儲存體的流量。 當您利用資料箱時，資料會透過 Azure 骨幹傳送。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="13-protect-critical-web-applications"></a>1.3：保護重要的 Web 應用程式

**指引**：不適用；這項建議適用於在 Azure App Service 或計算資源上執行的 Web 應用程式。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4：拒絕與已知惡意 IP 位址通訊

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。 您可以透過 Azure 入口網站，控制從資料箱到 Azure 託管儲存體的流量。 當您利用資料箱時，資料會透過 Azure 骨幹傳送。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="15-record-network-packets"></a>1.5：記錄網路封包

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。 您可以透過 Azure 入口網站，控制從資料箱到 Azure 託管儲存體的流量。 當您利用資料箱時，資料會透過 Azure 骨幹傳送。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6：部署以網路為基礎的入侵偵測/入侵防護系統 (IDS/IPS) 

**指導** 方針：指導方針： Azure 資料箱所使用的端點都是由 Microsoft 管理。 您必須負責任何您想要部署到內部部署系統的其他控制項。

* [瞭解 Azure 資料箱安全性](./data-box-security.md)

* [Azure 資料箱的埠資訊](./data-box-system-requirements.md#port-requirements)

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="17-manage-traffic-to-web-applications"></a>1.7：管理 Web 應用程式的流量

**指引**：不適用；這項建議適用於在 Azure App Service 或計算資源上執行的 Web 應用程式。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8：將網路安全性規則的複雜性和系統管理負擔降至最低

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9：維護網路裝置的標準安全性設定

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="110-document-traffic-configuration-rules"></a>1.10：文件流量設定規則

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11：使用自動化工具來監視網路資源設定並偵測變更

**指導** 方針：不適用;您的 Azure 資料箱不能與虛擬網路相關聯。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

## <a name="logging-and-monitoring"></a>記錄和監視

*如需詳細資訊，請參閱 [安全性控制：記錄和監視](../security/benchmarks/security-control-logging-monitoring.md)。*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1：使用已核准的時間同步處理來源

**指導** 方針： Microsoft 會維護用於 Azure 資源的時間來源，例如，記錄中的時間戳記。 如果沒有連線到可存取客戶網站上 NTP 伺服器的網路，Azure 資料箱時間可能會漂移。

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="22-configure-central-security-log-management"></a>2.2：設定中央安全性記錄管理

**指導** 方針：對應到您的資料箱訂單中的每個步驟，您可以採取多個動作來控制訂單的存取權、審核事件、追蹤訂單，以及解讀產生的各種記錄。

* [瞭解 Azure 資料箱的追蹤和事件記錄](./data-box-logs.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3：啟用 Azure 資源的稽核記錄

**指導** 方針：對應到您的資料箱訂單中的每個步驟，您可以採取多個動作來控制訂單的存取權、審核事件、追蹤訂單，以及解讀產生的各種記錄。

* [瞭解 Azure 資料箱的追蹤和事件記錄](./data-box-logs.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4：從作業系統收集安全性記錄

**指導** 方針：這項建議適用于計算資源。 資料箱具有包含安全性記錄的 audit 記錄和支援封裝。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="25-configure-security-log-storage-retention"></a>2.5：設定安全性記錄儲存體保留期

**指導** 方針：不適用

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="26-monitor-and-review-logs"></a>2.6：監視和檢閱記錄

**指導** 方針：對應到您的資料箱訂單中的每個步驟，您可以採取多個動作來控制訂單的存取權、審核事件、追蹤訂單，以及解讀產生的各種記錄。

* [瞭解 Azure 資料箱的追蹤和事件記錄](./data-box-logs.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7：啟用異常活動的警示

**指導** 方針：對應到您的資料箱訂單中的每個步驟，您可以採取多個動作來控制訂單的存取權、審核事件、追蹤訂單，以及解讀產生的各種記錄。

* [瞭解 Azure 資料箱的追蹤和事件記錄](./data-box-logs.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="28-centralize-anti-malware-logging"></a>2.8：集中化反惡意程式碼記錄

**指導** 方針：不適用;Azure 資料箱不會處理或產生反惡意程式碼的相關記錄。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="29-enable-dns-query-logging"></a>2.9：啟用 DNS 查詢記錄

**指導** 方針：不適用;Azure 資料箱不會處理或產生 DNS 相關的記錄。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="210-enable-command-line-audit-logging"></a>2.10：啟用命令列稽核記錄

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

## <a name="identity-and-access-control"></a>身分識別與存取控制

*如需詳細資訊，請參閱 [安全性控制：身分識別與存取控制](../security/benchmarks/security-control-identity-access-control.md)。*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1：維護系統管理帳戶的詳細目錄

**指導**：維護具有您 Azure 資料箱之系統管理存取權的使用者帳戶清查。 您可以使用訂用帳戶 Azure 入口網站中的身分識別和存取控制 (IAM) 窗格，來設定 Azure 角色型存取控制 (Azure RBAC) 。 角色會套用至 Active Directory 中的使用者、群組、服務主體和受控識別。您可以在訂單首次建立時，控制可存取訂單的人員。 在不同範圍設定 Azure 角色，以控制資料箱訂單的存取權。 Azure 角色可決定存取權的類型–讀寫、唯讀、讀寫作業的子集。

* [瞭解自訂角色](../role-based-access-control/custom-roles.md)

* [如何設定活頁簿的 Azure RBAC](../sentinel/quickstart-get-visibility.md)

* [瞭解如何設定順序的存取控制](./data-box-logs.md#set-up-access-control-on-the-order)

**Azure 資訊安全中心監視**：否

**責任**：客戶

### <a name="32-change-default-passwords-where-applicable"></a>3.2：在適用的情況下變更預設密碼

**指引**：Azure AD 沒有預設密碼的概念。 需要密碼的其他 Azure 資源會強制建立具有複雜性需求和密碼長度下限的密碼，這會因服務而有所不同。 您必須負責可能使用預設密碼的協力廠商應用程式和 marketplace 服務。

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="33-use-dedicated-administrative-accounts"></a>3.3：使用專用的系統管理帳戶

**指引**：建立使用專用系統管理帳戶的標準作業程序。 使用 Azure 資訊安全中心的身分識別與存取管理來監視系統管理帳戶的數目。

此外，為了協助您追蹤專用系統管理帳戶，您可使用來自 Azure 資訊安全中心或內建 Azure 原則的建議，例如：
- 應將一個以上的擁有者指派給您的訂用帳戶
- 具有擁有者權限的已取代帳戶應該從您的訂用帳戶中移除
- 具有擁有者權限的外部帳戶應該從您的訂用帳戶中移除

* [如何使用 Azure 資訊安全中心來監視身分識別和存取 (預覽)](../security-center/security-center-identity-access.md)

* [如何使用 Azure 原則](../governance/policy/tutorials/create-and-manage.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4：使用單一登入 (SSO) 搭配 Azure Active Directory

**指導** 方針：不適用;您資料箱訂單的存取權是透過 Azure 入口網站，並保留給具有擁有者或參與者的租使用者角色的帳戶。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5：針對所有以 Azure Active Directory 為基礎的存取使用多重要素驗證

**指導** 方針：不適用

**Azure 資訊安全中心監視**：否

**責任**：不適用

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6：使用專用電腦 (特殊權限存取工作站) 進行所有系統管理工作

**指導** 方針：使用特殊許可權存取工作站 (PAW) ，Azure AD Multi-Factor Authentication 已啟用 MFA (，以登入並設定您的) 訂單。

* [特殊權限存取工作站](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

* [規劃雲端式 Azure AD Multi-Factor Authentication 部署](../active-directory/authentication/howto-mfa-getstarted.md) (機器翻譯)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7：來自系統管理帳戶的可疑活動記錄和警示

**指導** 方針：不適用，Azure 資料箱沒有自己的系統管理帳戶。 您可以透過 Azure 入口網站來存取它。 不過，您可以設定您的 Azure 訂用帳戶，以在環境中發生可疑或不安全的活動時，使用 Azure Active Directory (AD) Privileged Identity Management (PIM) 來產生記錄和警示。

此外，使用 Azure AD 風險偵測來檢視有風險的使用者行為相關警示和報告。

* [如何部署 Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [了解 Azure AD 風險偵測](../active-directory/identity-protection/overview-identity-protection.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8：僅從核准的位置管理 Azure 資源

**指引**：使用條件式存取具名位置，只允許從 IP 位址範圍或國家/區域的特定邏輯群組存取 Azure 入口網站。

* [如何在 Azure 中設定具名位置](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="39-use-azure-active-directory"></a>3.9：使用 Azure Active Directory

**指導** 方針：在適用的情況下，使用 AZURE ACTIVE DIRECTORY (AD) 作為中央驗證和授權系統。 Azure AD 會對待用資料和傳輸中資料使用增強式加密，以保護資料安全。 Azure AD 也會對使用者認證進行 Salt 處理、雜湊處理並安全儲存資料。

* [如何建立及設定 Azure AD 執行個體](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10：定期檢閱並協調使用者存取

**指導** 方針： AZURE ACTIVE DIRECTORY (AD) 提供記錄檔，以協助您探索過時的帳戶。 此外，使用 Azure 身分識別存取審核來有效率地管理群組成員資格、企業應用程式的存取權，以及角色指派。 您可以定期檢閱使用者的存取權，以確定只有適當的使用者具有持續存取權。

若為數據箱設備，將不會即時支援這項功能。 您可以在作業結束時檢查記錄。

* [瞭解 Azure AD 報告](../active-directory/reports-monitoring/index.yml)

* [如何使用 Azure 身分識別存取權檢閱](../active-directory/governance/access-reviews-overview.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11：監視嘗試存取已停用的認證

**指導** 方針：在適用的情況下，使用 AZURE ACTIVE DIRECTORY (AD) 作為中央驗證和授權系統。 Azure AD 會對待用資料和傳輸中資料使用增強式加密，以保護資料安全。 Azure AD 也會對使用者認證進行 Salt 處理、雜湊處理並安全儲存資料。

您可以存取 Azure AD 登入活動、audit 和風險事件記錄檔來源，讓您可以與 Azure Sentinel 或協力廠商 SIEM 整合。

您可以建立 Azure AD 使用者帳戶的診斷設定，並將審核記錄和登入記錄傳送至 Log Analytics 工作區，以簡化此程式。 您可以在 Log Analytics 中設定所需的記錄警示。

Azure 資料箱服務記錄不會寫入 Log Analytics 工作區。

* [如何將 Azure 活動記錄整合到 Azure 監視器中](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

* [如何讓 Azure Sentinel 上線](../sentinel/quickstart-onboard.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12：帳戶登入行為偏差警示

**指導** 方針：對於控制項平面上的帳戶登入行為偏差 (例如 Azure 入口網站) ，請使用 Azure AD Identity Protection 和風險偵測功能，來設定偵測到與使用者身分識別相關的可疑動作的自動回應。 您也可將資料內嵌到 Azure Sentinel 中，以便進一步調查。

* [如何查看 Azure AD 具風險的登入](../active-directory/identity-protection/overview-identity-protection.md)

* [如何設定和啟用身分識別保護風險原則](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [如何使 Azure Sentinel 上線](../sentinel/quickstart-onboard.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13：在支援案例期間為 Microsoft 提供相關客戶資料的存取權

**指導** 方針： Azure 資料箱目前不支援客戶加密箱。

* [客戶加密箱支援的服務清單](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

## <a name="data-protection"></a>資料保護

*如需詳細資訊，請參閱 [安全性控制：資料保護](../security/benchmarks/security-control-data-protection.md)。*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1：維護敏感性資訊的詳細目錄

**指導** 方針：不適用 Azure 資料箱。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2：隔離儲存或處理敏感性資訊的系統

**指導** 方針： Azure 資料箱將會布建在您要授與存取權的訂用帳戶中。 沒有可保護或隔離的公用端點。 具有訂用帳戶擁有者或參與者存取權的使用者可以使用資料箱存取權。

在資料上傳至 Azure 期間，資料箱設備和用來上傳資料的服務會進行隔離。

* [如何開始使用 Azure 資料箱](./data-box-quickstart-portal.md)

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3：監視並封鎖未經授權的敏感性資訊傳輸

**指導** 方針： Microsoft 管理 Azure 資料箱的基礎結構，並已實行嚴格的控制，以防止客戶資料遺失或洩漏。 當資料箱位於客戶網站時，請遵循最佳作法，以確保傳輸的機密資料受到保護。

* [瞭解 Azure 中的客戶資料保護](../security/fundamentals/protection-customer-data.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：共用

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4：加密傳輸中的所有敏感性資訊

**指導** 方針： Microsoft 管理 Azure 資料箱的基礎結構，並已實行嚴格的控制，以防止客戶資料遺失或洩漏。 當資料箱位於客戶網站時，請遵循最佳作法，以確保傳輸的機密資料受到保護。

* [瞭解 Azure 資料箱中的資料移轉](./data-box-faq.md)

* [資料箱安全性總覽](./data-box-security.md)

**Azure 資訊安全中心監視**：不適用

**責任**：共用

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5：使用作用中探索工具來識別敏感性資料

**指導** 方針：目前無法使用;Azure 資料箱的資料識別、分類和遺失防護功能尚未提供。 Microsoft 會管理 Azure 資料箱的基礎結構，並已實行嚴格的控制，以防止客戶資料遺失或洩漏。

* [瞭解 Azure 中的客戶資料保護](../security/fundamentals/protection-customer-data.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：不適用

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6：使用 Azure RBAC 來控制資源的存取權

**指導** 方針：確定您擁有訂用帳戶的擁有者或參與者存取權，才能建立資料箱訂單。 您也可以在資源層級定義資料箱讀者和資料箱參與者角色。

* [瞭解如何開始使用 Azure 資料箱](./data-box-quickstart-portal.md)

* [瞭解如何設定存取控制](./data-box-logs.md#set-up-access-control-on-the-order)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7：使用主機型資料外洩防護來強制執行存取控制

**指引**：不適用，這項建議主要用於計算資源。 Microsoft 管理 Azure 資料箱的基礎結構，並已實行嚴格的控制，以防止客戶資料遺失或洩漏。

* [Azure 客戶資料保護](../security/fundamentals/protection-customer-data.md)

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8：加密待用的敏感性資訊

**指導** 方針： Azure 資料箱為待用資料實行 AES 256 位加密。

Azure 資料箱針對待用資料執行 AES 256 位加密。此外，Azure 資料箱會保護裝置的解除鎖定金鑰 (也稱為裝置密碼) ，此金鑰是用來透過加密金鑰來鎖定裝置。 根據預設，會使用 Microsoft 管理的金鑰來加密資料箱訂單的裝置解除鎖定金鑰。 若要進一步控制裝置解除鎖定金鑰，您也可以提供客戶管理的金鑰。 客戶管理的金鑰必須建立並儲存在 Azure Key Vault 中。

* [瞭解資料箱資料保護](./data-box-security.md)

* [在 Azure Key Vault 中使用客戶管理的金鑰 Azure 資料箱](./data-box-customer-managed-encryption-key-portal.md)

**Azure 資訊安全中心監視**：不適用

**責任**：共用

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9：針對重要 Azure 資源的變更留下記錄和發出警示

**指導** 方針：使用 Azure 監視器搭配 Azure 活動記錄檔，以建立 Azure 資料箱和其他重要或相關資源發生變更時的警示。

* [如何建立 Azure 活動記錄事件的警示](../azure-monitor/platform/alerts-activity-log.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

## <a name="vulnerability-management"></a>弱點管理

*如需詳細資訊，請參閱 [安全性控制：弱點管理](../security/benchmarks/security-control-vulnerability-management.md)。*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1：執行自動化弱點掃描工具

**指導** 方針： Microsoft 會在支援 Azure 資料箱的基礎系統上執行弱點管理。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2：部署自動化的作業系統修補程式管理解決方案

**指導** 方針：當資料箱出貨時，會隨最新的更新安裝。 我們不會執列欄位更新。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3：為協力廠商軟體專案部署自動化的修補程式管理解決方案

**指導** 方針：當資料箱出貨時，會隨最新的更新安裝。 我們不會執列欄位更新。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4：比較連續性弱點掃描

**指導** 方針： Microsoft 會在支援 Azure 資料箱的基礎系統上執行弱點管理。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5：使用風險評等程序來排定所發現弱點的補救優先順序

**指導** 方針： Microsoft 會在支援 Azure 資料箱的基礎系統上執行弱點管理。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

## <a name="inventory-and-asset-management"></a>清查和資產管理

*如需詳細資訊，請參閱 [安全性控制：清查和資產管理](../security/benchmarks/security-control-inventory-asset-management.md)。*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1：使用自動化資產探索解決方案

**指導** 方針：不適用，找不到任何資料箱資產可供探索。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="62-maintain-asset-metadata"></a>6.2：維護資產中繼資料

**指導** 方針：不適用，沒有適用于資料箱的資產中繼資料。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="63-delete-unauthorized-azure-resources"></a>6.3：刪除未經授權的 Azure 資源

**指導** 方針：不適用資料箱服務可確保不會使用未經授權的 Azure 資源。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4：定義和維護已核准 Azure 資源的清查

**指導** 方針：不適用;資料箱服務層級沒有任何資料。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5：監視未經核准的 Azure 資源

**指導** 方針：不適用，資料箱服務層級沒有任何資料。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6：監視計算資源內未經核准的軟體應用程式

**指導** 方針：不適用，資料箱服務層級沒有任何資料。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7：移除未經核准的 Azure 資源和軟體應用程式

**指導** 方針：不適用，資料箱服務層級沒有任何資料。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="68-use-only-approved-applications"></a>6.8：僅使用已核准的應用程式

**指導** 方針：不適用，資料箱服務層級沒有任何資料。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="69-use-only-approved-azure-services"></a>6.9：僅使用已核准的 Azure 服務

**指導** 方針：不適用，資料箱服務只會使用已核准的 Azure 服務。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10：維護已核准軟體標題的清查

**指導** 方針：不適用;資料箱服務只會使用已核准的軟體專案。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11：限制使用者與 Azure Resource Manager 互動的能力

**指引**：設定 Azure 條件式存取，以藉由對「Microsoft Azure 管理」應用程式設定「封鎖存取」，限制使用者與 Azure Resource Manager 互動的能力。

* [如何設定條件式存取以封鎖 Azure Resource Manager 的存取](../role-based-access-control/conditional-access-azure-management.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12：限制使用者在計算資源內執行指令碼的能力

**指導** 方針：不適用;資料箱服務不支援執行腳本。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13：以實體或邏輯方式隔離高風險的應用程式

**指導** 方針：不適用;資料箱服務沒有在 Azure App 服務上執行的 web 應用程式。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

## <a name="secure-configuration"></a>安全設定

*如需詳細資訊，請參閱 [安全性控制：安全設定](../security/benchmarks/security-control-secure-configuration.md)。*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1：為所有 Azure 資源建立安全設定

**指導** 方針： Azure 資料箱隨附預先設定的最佳作法安全性設定。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="72-establish-secure-operating-system-configurations"></a>7.2：建立安全的作業系統設定

**指導** 方針： Azure 資料箱隨附預先設定的最佳作法安全性設定。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3：維護安全的 Azure 資源設定

**指導** 方針： Azure 資料箱隨附預先設定的資源最佳作法安全性設定，且無法變更。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4：維護安全的作業系統設定

**指導** 方針： Azure 資料箱隨附預先設定的資源最佳作法安全性設定，且無法變更。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5：安全地儲存 Azure 資源的設定

**指導** 方針：會安全地儲存所有資料箱設定。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="76-securely-store-custom-operating-system-images"></a>7.6：安全地儲存自訂作業系統映像

**指導** 方針：會安全地儲存所有資料箱作業系統映射。

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7：部署適用于 Azure 資源的設定管理工具

**指導** 方針：不適用，Azure 資料箱設定無法變更。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8：部署作業系統的設定管理工具

**指導** 方針：不適用，Azure 資料箱設定無法變更。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9：執行 Azure 資源的自動化設定監視

**指導** 方針：不適用，Azure 資料箱設定無法變更。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10：為作業系統實作自動化的設定監視

**指導** 方針：不適用，Azure 資料箱設定無法變更。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="711-manage-azure-secrets-securely"></a>7.11：安全地管理 Azure 祕密

**指導** 方針：必須建立客戶管理的金鑰，並將其儲存在 Azure Key Vault 中。

* [如何在 Azure 資料箱的 Azure Key Vault 中使用客戶管理的金鑰](./data-box-customer-managed-encryption-key-portal.md)

**Azure 資訊安全中心監視**：不適用

**責任**：共用

### <a name="712-manage-identities-securely-and-automatically"></a>7.12：安全且自動地管理身分識別

**指導** 方針：不適用;Azure 資料箱不會利用受控識別。

* [支援受控識別的 Azure 服務](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13：消除非預期的認證公開

**指導** 方針： Microsoft 在資料箱程式碼上執行認證掃描器。 此外，Microsoft 也會安全地保護認證。 執行認證掃描器來識別程式碼中的認證。 認證掃描器也有助於將探索到的認證移至更安全的位置，例如 Azure Key Vault。

* [如何設定認證掃描器](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Azure 資訊安全中心監視**：N/A

**責任**：共用

## <a name="malware-defense"></a>惡意程式碼防禦

*如需詳細資訊，請參閱 [安全性控制：惡意程式碼防禦](../security/benchmarks/security-control-malware-defense.md)。*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1：使用集中管理的反惡意程式碼軟體

**指引**：不適用，這項準則主要用於計算資源。 支援 Azure 服務的基礎主機上已啟用 Microsoft 反惡意程式碼 (例如 Azure App Service) ，但不會對客戶內容執行。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2：預先掃描要上傳至非計算 Azure 資源的檔案

**指導** 方針：支援 azure 服務的基礎主機上已啟用 Microsoft 反惡意程式碼 (例如，azure 客戶加密箱) ，但不會在客戶內容上執行。

您必須負責預先掃描即將上傳至非計算 Azure 資源的任何內容。 Microsoft 無法存取客戶資料，因此無法代表您執行客戶內容的反惡意程式碼掃描。

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3：確保更新反惡意程式碼軟體和簽章

**指引**：不適用，這項建議主要用於計算資源。 支援 Azure 服務的基礎主機上已啟用 Microsoft Antimalware，但不會對客戶內容執行。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

## <a name="data-recovery"></a>資料復原

*如需詳細資訊，請參閱 [安全性控制：資料復原](../security/benchmarks/security-control-data-recovery.md)。*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1：確保會定期自動備份

**指導** 方針：不適用資料箱服務，不需要備份。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2：執行完整的系統備份並備份任何客戶管理的金鑰

**指導** 方針：請務必備份任何資料和客戶管理的金鑰。 資料箱不會進行任何備份。

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3：驗證所有備份，包括客戶管理的金鑰

**指導** 方針：您應該先確認所有資料都在 Azure 儲存體帳戶，再從您的內部部署刪除。

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4：確保備份和客戶管理的金鑰的保護

**指導** 方針：確定您擁有的任何備份或客戶管理的金鑰，都已根據最佳作法進行保護。

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="incident-response"></a>事件回應

*如需詳細資訊，請參閱 [安全性控制：事件回應](../security/benchmarks/security-control-incident-response.md)。*

### <a name="101-create-an-incident-response-guide"></a>10.1：建立事件回應指南

**指引**：為組織製作事件回應指南。 請確定有書面的事件回應計畫，其中定義人員的所有角色，以及從偵測到事件後檢討的事件處理/管理階段。

* [建立自有安全性事件回應程序的指引](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Microsoft 安全性回應中心的事件剖析](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [利用 NIST 的電腦安全性事件處理指南，協助建立自己的事件回應計畫](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2：建立事件評分和優先順序程序

**指引**：資訊安全中心會指派每個警示的嚴重性，以協助設定應優先調查哪些警示。 嚴重性會依據資訊安全中心對用於發出警示的發現或分析其信心程度，以及信賴等級具有活動背後會導致警示的惡意意圖。

此外，使用標記清楚地標示訂用帳戶 (例如， 生產、非生產) 並建立命名系統，以清楚地識別及分類 Azure 資源，尤其是處理敏感性資料的資源。 您需負責根據發生事件的 Azure 資源和環境的重要性，設定警示的補救優先順序。

* [Azure 資訊安全中心的安全性警示](../security-center/security-center-alerts-overview.md)

* [使用標記來組織 Azure 資源](../azure-resource-manager/management/tag-resources.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="103-test-security-response-procedures"></a>10.3：測試安全性回應程序

**指引**：進行練習以定期測試系統的事件回應功能，進而協助保護您的 Azure 資源。 找出弱點和落差，並視需要修訂計畫。

* [NIST 的出版品 - Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4：提供安全性事件連絡人詳細資料，並設定安全性事件的警示通知

**指引**：如果 Microsoft 安全性回應中心 (MSRC) 發現您的資料遭到非法或未經授權的對象存取，Microsoft 將使用安全性事件連絡資訊來連絡您。 事後檢討事件，確保問題已解決。

* [如何設定 Azure 資訊安全中心的安全性連絡人](../security-center/security-center-provide-security-contact-details.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5：將安全性警示併入事件回應系統

**指引**：使用「連續匯出」功能來匯出 Azure 資訊安全中心警示和建議，協助找出 Azure 資源的風險。 「連續匯出」可讓您以手動或持續不斷的方式來匯出警示和建議。 您可使用 Azure 資訊安全中心的資料連接器，將警示串流至 Azure Sentinel。

* [如何設定連續匯出](../security-center/continuous-export.md)

* [如何將警示串流至 Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="106-automate-the-response-to-security-alerts"></a>10.6：自動回應安全性警示

**指導** 方針：使用 Azure 資訊安全中心中的工作流程自動化功能，透過「Logic Apps」安全性警示和建議來自動觸發回應，以保護您的 Azure 資源。

* [如何設定工作流程自動化和 Logic Apps](../security-center/workflow-automation.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="penetration-tests-and-red-team-exercises"></a>滲透測試和 Red Team 練習

*如需詳細資訊，請參閱 [安全性控制：滲透測試和 Red Team 練習](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md)。*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1：進行 Azure 資源的定期滲透測試，並確保修復所有重要的安全性結果

**指導** 方針： Microsoft 在資料箱裝置上進行滲透測試和弱點掃描。 您可以自行進行滲透測試和弱點掃描。 如果您選擇這樣做，請遵循 Microsoft 的 Engagement 規則，以確保您的滲透測試不違反 Microsoft 原則。 針對受 Microsoft 管理的雲端基礎結構、服務和應用程式，使用 Microsoft 的策略和執行的 Red 小組和即時網站滲透測試。

* [滲透測試運作規則](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Azure 資訊安全中心監視**：不適用

**責任**：共用

## <a name="next-steps"></a>後續步驟

- 請參閱 [Azure 安全性效能評定](../security/benchmarks/overview.md)
- 深入了解 [Azure 資訊安全性基準](../security/benchmarks/security-baselines-overview.md)