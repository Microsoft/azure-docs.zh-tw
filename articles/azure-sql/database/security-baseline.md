---
title: Azure 安全性基準
description: 適用于 Azure SQL Database 和 Azure SQL 受控執行個體的 azure 安全性基準
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/21/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 12c7fd1d8ee36b562cd651f50fd0565825441883
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98197382"
---
# <a name="azure-security-baseline-for-azure-sql-database--sql-managed-instance"></a>適用于 Azure SQL Database & SQL 受控執行個體的 Azure 安全性基準
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

適用于 Azure SQL Database 的 Azure 安全性基準包含可協助您改善部署安全性狀態的建議。

此服務的基準取自 [Azure 安全性效能評定 1.0 版](../../security/benchmarks/overview.md)，其會提供如何在 Azure 上使用最佳做法指引來保護雲端解決方案的建議。

如需詳細資訊，請參閱 [Azure 安全性基準概觀](../../security/benchmarks/security-baselines-overview.md) (機器翻譯)。

## <a name="network-security"></a>網路安全性

如需詳細資訊，請參閱[安全性控制：網路安全性](../../security/benchmarks/security-control-network-security.md) (機器翻譯)。

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1：在虛擬網路上使用網路安全性群組或 Azure 防火牆來保護資源

**指導** 方針：您可以啟用 Azure Private Link 以允許存取 Azure PaaS 服務 (例如，透過虛擬網路中的私人端點，SQL Database) 和 azure 裝載的客戶/合作夥伴服務。 虛擬網路和服務間的流量會在通過 Microsoft 骨幹網路時隨之減少，降低資料在網際網路中公開的風險。

若要允許流量觸達 Azure SQL Database，請使用 SQL服務標籤，允許透過網路安全性群組輸出流量。

虛擬網路規則可讓 Azure SQL Database 只接受從虛擬網路內所選子網傳送的通訊。

如何設定 Azure SQL Database 的 Private Link：

https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview#how-to-set-up-private-link-for-azure-sql-database

如何針對伺服器使用虛擬網路服務端點和規則：

https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2：監視和記錄 VNet、子網路與 NIC 的設定和流量

**指導** 方針：使用 Azure 資訊安全中心並針對您 Azure SQL Database 部署到的子網補救網路保護建議。

針對將連線至 Azure SQL Database 的 Azure 虛擬機器 (VM) ，啟用網路安全性群組 (NSG) 流量記錄，以保護這些 Vm，並將記錄傳送至 Azure 儲存體帳戶以進行流量審核。

您也可將 NSG 流量記錄傳送到 Log Analytics 工作區，並使用流量分析來提供對 Azure 雲端流量的見解。 流量分析的優點包括能將網路活動視覺化並找出作用點、識別安全性威脅、了解流量模式並找到錯誤的網路設定。

如何啟用 NSG 流量記錄： 

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

瞭解 Azure 資訊安全中心所提供的網路安全性：

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

如何啟用及使用流量分析： 

https://docs.microsoft.com/azure/network-watcher/traffic-analytics

瞭解 Azure 資訊安全中心所提供的網路安全性：

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="13-protect-critical-web-applications"></a>1.3：保護重要的 Web 應用程式

**指導** 方針：不適用;這項建議適用于裝載 web 應用程式的 Azure 應用程式服務或計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4：拒絕與已知惡意 IP 位址通訊

**指導** 方針：在與您的 Azure SQL Database 相關聯的虛擬網路上啟用 DDoS 保護標準，以防止分散式阻絕服務攻擊。 使用 Azure 資訊安全中心的整合式威脅情報，以拒絕與已知為惡意或未使用的網際網路 IP 位址通訊。

如何設定 DDoS 保護： 

https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

瞭解 Azure 資訊安全中心整合威脅情報：

https://docs.microsoft.com/azure/security-center/security-center-alerts-data-services

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="15-record-network-packets-and-flow-logs"></a>1.5：記錄網路封包和流量記錄

**指導** 方針：適用于將連線至 Azure SQL Database 實例)  (Vm 的 Azure 虛擬機器，請啟用網路安全性群組 (NSG) 流量記錄，以保護這些 vm，並將記錄傳送至 Azure 儲存體帳戶以進行流量審核。 如果需要調查異常活動，請啟用網路監看員封包捕獲。

如何啟用 NSG 流量記錄： 

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

如何啟用網路監看員：

https://docs.microsoft.com/azure/network-watcher/network-watcher-create

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6：部署網路型入侵偵測/入侵預防系統 (IDS/IPS)

**指導** 方針：為 Azure SQL Database 啟用「Advanced 威脅防護」 (ATP) 。  一旦有可疑活動、潛在弱點、SQL 插入式攻擊以及異常的資料庫存取和查詢模式發生時，使用者就會收到警示。 Advanced 威脅防護也會整合警示與 Azure 資訊安全中心。

瞭解和使用 Azure SQL Database 的 Advanced 威脅防護： https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="17-manage-traffic-to-web-applications"></a>1.7：管理 Web 應用程式的流量

**指導** 方針：不適用;這項建議適用于裝載 web 應用程式的 Azure 應用程式服務或計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8：將網路安全性規則的複雜性和系統管理負擔降至最低

**指導** 方針：使用虛擬網路服務標籤來定義網路安全性群組或 Azure 防火牆上的網路存取控制。 建立安全性規則時，您可以使用服務標籤取代特定的 IP 位址。 在規則的適當來源或目的地欄位中指定服務標籤名稱 (例如 ApiManagement)，即可允許或拒絕對應服務的流量。 Microsoft 會管理服務標籤包含的位址前置詞，並隨著位址變更自動更新服務標籤。

使用 Azure SQL Database 的服務端點時，需要輸出至 Azure SQL Database 的公用 IP 位址：網路安全性群組 (Nsg) 必須開啟，Azure SQL Database Ip 才能允許連線。 您可以使用 NSG 服務標記進行 Azure SQL Database。

瞭解 Azure SQL Database 的服務端點服務標記：

https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview#limitations

瞭解和使用服務標記：

https://docs.microsoft.com/azure/virtual-network/service-tags-overview

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9：維護網路裝置的標準安全性設定

**指導** 方針：使用 Azure 原則為您的 Azure SQL Database 定義和執行網路安全性設定。 您可以使用 "Microsoft .Sql" 命名空間來定義自訂原則定義，或使用針對伺服器網路保護所設計的任何內建原則定義。 適用于伺服器的內建網路安全性原則範例如下：「SQL Database 應使用虛擬網路服務端點」。

使用 Azure 藍圖藉由在單一藍圖定義中封裝關鍵環境成品（例如 Azure 資源管理範本、Azure 角色型存取控制 (Azure RBAC) 和原則）來簡化大規模的 Azure 部署。 輕鬆地將藍圖套用至新的訂閱、環境，以及透過版本控制來微調控制和管理。

如何設定和管理 Azure 原則： https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

如何建立 Azure 藍圖： https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="110-document-traffic-configuration-rules"></a>1.10：文件流量設定規則

**指導** 方針：將標記用於網路安全性群組 (NSG) 以及與網路安全性和流量相關的其他資源。 對於個別的 NSG 規則，使用 [描述] 欄位，針對允許進出網路流量的任何規則指定商務需求和/或持續時間 (等等)。

使用任何與標記相關的內建 Azure 原則定義，例如「需要標籤和其值」，以確保所有資源都是使用標籤建立的，並會通知您現有的未標記資源。

您可使用 Azure PowerShell 或 Azure CLI，根以據資源的標籤對資源進行查詢或執行動作。

如何建立和使用標籤： 

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11：使用自動化工具來監視網路資源設定並偵測變更

**指導** 方針：使用 Azure 活動記錄監視網路資源設定，並偵測與您伺服器相關的網路資源變更。 在 Azure 監視器中建立警示，以在重要網路資源發生變更時觸發。

如何檢視及擷取 Azure 活動記錄事件： 

https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

如何在 Azure 監視器中建立警示： 

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="logging-and-monitoring"></a>記錄和監視

如需詳細資訊，請參閱[安全性控制：記錄和監視](../../security/benchmarks/security-control-logging-monitoring.md) (機器翻譯)。

### <a name="21-use-approved-time-synchronization-sources"></a>2.1：使用已核准的時間同步處理來源

**指導** 方針： Microsoft 會維護 Azure 資源的時間來源。 您可以更新計算部署的時間同步處理。

如何設定 Azure 計算資源的時間同步處理：

https://docs.microsoft.com/azure/virtual-machines/windows/time-sync

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2：設定中央安全性記錄管理

**指導** 方針：啟用 Azure SQL Database 的審核，以追蹤資料庫事件，並將它們寫入 Azure 儲存體帳戶、log Analytics 工作區或事件中樞的 audit 記錄檔。

此外，您可以將 Azure SQL 診斷遙測串流至 Azure SQL 分析，這是一種雲端解決方案，可監視大規模和跨多個訂用帳戶的 Azure SQL Database 和 Azure SQL 受控執行個體效能。 可協助您收集 Azure SQL Database 效能計量，並以視覺效果方式呈現，而且有內建智慧可以執行效能疑難排解。

如何設定 Azure SQL Database 的審核：

https://docs.microsoft.com/azure/sql-database/sql-database-auditing

如何使用 Azure 監視器收集平臺記錄和計量：

https://docs.microsoft.com/azure/sql-database/sql-database-metrics-diag-logging

如何將診斷串流至 Azure SQL 分析：

https://docs.microsoft.com/azure/sql-database/sql-database-metrics-diag-logging#stream-into-azure-sql-analytics

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3：啟用 Azure 資源的稽核記錄

**指導** 方針：在您的伺服器上啟用審核，並針對 (Azure 儲存體、Log Analytics 或事件中樞) 的 [審核記錄] 選擇儲存位置。

如何啟用 Azure SQL Database 的審核：

https://docs.microsoft.com/azure/sql-database/sql-database-auditing

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4：從作業系統收集安全性記錄

**指導** 方針：不適用;此基準測試適用于計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="25-configure-security-log-storage-retention"></a>2.5：設定安全性記錄儲存體保留期

**指導** 方針：將 Azure SQL Database 記錄儲存在 Log Analytics 工作區時，請根據您組織的合規性法規來設定記錄保留期限。

如何設定記錄保留參數：

https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="26-monitor-and-review-logs"></a>2.6：監視和檢閱記錄

**指導** 方針：分析和監視異常行為的記錄，並定期查看結果。 使用 Azure 資訊安全中心的 Advanced 威脅防護，對您 Azure SQL Database 實例相關的異常活動發出警示。 或者，根據計量值或與您 Azure SQL Database 實例相關的 Azure 活動記錄專案來設定警示。

瞭解 Azure SQL Database 的 Advanced 威脅防護和警示：

https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview

如何設定 Azure SQL Database 的自訂警示：

https://docs.microsoft.com/azure/sql-database/sql-database-insights-alerts-portal?view=azps-1.4.0

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7：啟用異常活動的警示

**指導** 方針：使用 Azure 資訊安全中心 Advanced 威脅防護進行 Azure SQL Database 進行監視，並在異常活動時發出警示。 為您的 SQL 資料庫啟用 Azure Defender for SQL。 適用于 SQL 的 Azure Defender 包含用來呈現和減緩潛在資料庫弱點的功能，以及偵測可能表示對資料庫有威脅的異常活動。

瞭解 Azure SQL Database 的 Advanced 威脅防護和警示：

https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview

如何啟用適用于 SQL 的 Azure Defender 以進行 Azure SQL Database：

https://docs.microsoft.com/azure/azure-sql/database/azure-defender-for-sql

如何在 Azure 資訊安全中心中管理警示：

https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="28-centralize-anti-malware-logging"></a>2.8：集中化反惡意程式碼記錄

**指導** 方針：不適用;針對 Azure SQL Database，反惡意程式碼解決方案是由 Microsoft 在基礎平臺上管理。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="29-enable-dns-query-logging"></a>2.9：啟用 DNS 查詢記錄

**指導** 方針：不適用;DNS 記錄不適用於 Azure SQL Database。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="210-enable-command-line-audit-logging"></a>2.10：啟用命令列稽核記錄

**指導** 方針：不適用;命令列的審核功能不適用於 Azure SQL Database。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

## <a name="identity-and-access-control"></a>身分識別與存取控制

如需詳細資訊，請參閱[安全性控制：身分識別與存取控制](../../security/benchmarks/security-control-identity-access-control.md) (機器翻譯)。

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1：維護系統管理帳戶的清查

**指導** 方針： Azure Active Directory (Azure AD) 有必須明確指派且可查詢的內建角色。 使用 Azure AD PowerShell 模組執行臨機操作查詢，以探索屬於系統管理群組成員的帳戶。

如何使用 PowerShell 在 Azure AD 中取得目錄角色：

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

如何使用 PowerShell 在 Azure AD 中取得目錄角色的成員：

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="32-change-default-passwords-where-applicable"></a>3.2：在適用的情況下變更預設密碼

**指導** 方針： Azure Active Directory 沒有預設密碼的概念。 布建 Azure SQL Database 實例時，建議您選擇整合驗證與 Azure Active Directory。

如何使用 Azure SQL 設定及管理 Azure Active Directory authentication：

https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="33-use-dedicated-administrative-accounts"></a>3.3：使用專用的系統管理帳戶

**指導** 方針：使用專用的系統管理帳戶來建立原則和程式。 使用 Azure 資訊安全中心的身分識別與存取管理來監視系統管理帳戶的數目。

了解 Azure 資訊安全中心身分識別與存取： 

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4：使用單一登入 (SSO) 搭配 Azure Active Directory

**指導** 方針：不適用;雖然您可以設定 Azure Active Directory Authentication 來與 Azure SQL Database 整合，但不支援單一登入。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5：針對所有以 Azure Active Directory 為基礎的存取使用多重要素驗證

**指導** 方針：啟用 Azure Active Directory (Azure AD) MULTI-FACTOR AUTHENTICATION (MFA) ，並遵循 Azure 資訊安全中心身分識別和存取管理建議。

如何在 Azure 中啟用 MFA：

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

如何在 Azure 資訊安全中心監視身分識別與存取： 

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6：使用專用電腦 (特殊權限存取工作站) 進行所有系統管理工作

**指導** 方針：使用具特殊許可權的存取工作站 (PAW) ，並將 Multi-Factor Authentication MFA 設定為登入和設定 Azure 資源。

了解特殊權限存取工作站： 

https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/

如何在 Azure 中啟用 MFA：

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7：針對來自系統管理帳戶的可疑活動留下記錄和發出警示

**指導** 方針：當環境中發生可疑或不安全的活動時，請使用 Azure Active Directory 的安全性報告來產生記錄和警示。

針對 Azure SQL Database 使用 Advanced 威脅防護來偵測異常活動，指出有不尋常且可能有害的嘗試存取或惡意探索資料庫。

如何識別標示為具風險活動的 Azure AD 使用者：

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk

如何在 Azure 資訊安全中心中監視使用者身分識別和存取活動：

https://docs.microsoft.com/azure/security-center/security-center-identity-access

查看 Advanced 威脅防護和潛在警示：

https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview#advanced-threat-protection-alerts

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8：僅從核准的位置管理 Azure 資源

**指導** 方針：使用名為「位置」的條件式存取，只允許來自特定 IP 位址範圍或國家/地區之邏輯群組的入口網站和 Azure 資源管理存取權。

如何在 Azure 中設定具名位置： https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="39-use-azure-active-directory"></a>3.9：使用 Azure Active Directory

**指導** 方針：為您的伺服器建立 Azure Active Directory (Azure AD) 系統管理員。

如何使用 Azure SQL 設定及管理 Azure Active Directory authentication：

https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure

如何建立和設定 Azure AD 實例：

https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10：定期檢閱並協調使用者存取

**指導** 方針： Azure Active Directory (Azure AD) 提供記錄以協助探索過時的帳戶。 此外，使用 Azure 身分識別存取審核來有效率地管理群組成員資格、企業應用程式的存取權，以及角色指派。 您可以定期審核使用者的存取權，以確保只有適當的使用者可以繼續存取。

如何使用 Azure 身分識別存取權檢閱： 

https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11：監視對已停用帳戶的存取嘗試

**指導** 方針：設定 Azure Active Directory (Azure AD) 使用 Azure SQL 進行驗證，並建立 Azure Active Directory 使用者帳戶的診斷設定，將審核記錄和登入記錄傳送至 Log Analytics 工作區。 在 Log Analytics 工作區中設定所需的警示。

如何使用 Azure SQL 設定及管理 Azure Active Directory authentication：

https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure

如何將 Azure 活動記錄整合到 Azure 監視器中： 

https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12：帳戶登入行為偏差警示

**指導** 方針：使用 Azure Active Directory (Azure AD) Identity Protection 和風險偵測，以針對偵測到與使用者身分識別相關的可疑動作，設定自動回應。 此外，您可以將資料內嵌到 Azure Sentinel 以進行進一步的調查。

如何查看 Azure AD 的風險登入：

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

如何設定及啟用 Identity Protection 風險原則：

https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13：在支援案例期間為 Microsoft 提供相關客戶資料的存取權

**指導** 方針：在 Microsoft 需要存取客戶資料的支援案例中，Azure 客戶加密箱會提供一個介面，讓客戶可以檢查及核准或拒絕客戶資料存取要求。

瞭解客戶加密箱：

https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="data-protection"></a>資料保護

如需詳細資訊，請參閱[安全性控制：資料保護](../../security/benchmarks/security-control-data-protection.md) (機器翻譯)。

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1：維護敏感性資訊的詳細目錄

**指引**：使用標籤協助追蹤可儲存或處理敏感性資訊的 Azure 資源。

如何建立和使用標記：

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2：隔離儲存或處理敏感性資訊的系統

**指引**：針對開發、測試和生產，實作不同的訂用帳戶及/或管理群組。 資源應以 Vnet/子網分隔、正確標記，並在 NSG 或 Azure 防火牆內受到保護。 儲存或處理敏感性資料的資源應該是隔離的。 使用 Private Link;在 Vnet 內部署 Azure SQL Database，並使用私人端點私下連接。

如何建立其他 Azure 訂用帳戶： 

https://docs.microsoft.com/azure/billing/billing-create-subscription

如何建立管理群組： 

https://docs.microsoft.com/azure/governance/management-groups/create

如何建立和使用標記：

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

如何設定 Azure SQL Database 的 Private Link：

https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview#how-to-set-up-private-link-for-azure-sql-database

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3：監視並封鎖未經授權的敏感性資訊傳輸

**指導** 方針：針對 Azure SQL Database 儲存或處理敏感性資訊的資料庫，請使用標記將資料庫和相關資源標記為機密。 將 Private Link 與 Azure SQL Database 實例上的網路安全性群組服務標籤一起設定，以防止機密資訊的遭到外泄。

針對 Microsoft 管理的基礎平台，Microsoft 會將所有客戶內容視為敏感性資訊，並竭盡全力防範客戶資料外洩和暴露。 為確保 Azure 中的客戶資料安全無虞，Microsoft 已實作並維護一套強大的資料保護控制和功能。

如何設定 Private Link 和 Nsg 以防止 Azure SQL Database 實例上的資料遭到外泄：

https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview

瞭解 Azure 中的客戶資料保護：

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4：加密傳輸中的所有敏感性資訊

**指導** 方針： Azure SQL Database 透過傳輸層安全性來加密移動中的資料，以保護您的資料。 SQL Database 針對所有連線強制執行加密 (SSL/TLS) 。 這可確保不論連接字串中的加密或 TrustServerCertificate 設定，所有資料都會在用戶端與伺服器之間加密「傳輸中」。

瞭解傳輸中的 Azure SQL 加密：

https://docs.microsoft.com/azure/sql-database/sql-database-security-overview#information-protection-and-encryption

**Azure 資訊安全中心監視**：不適用

**責任**：Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5：使用作用中探索工具來識別敏感性資料

**指導** 方針：使用 Azure SQL Database 資料探索和分類功能。 資料探索和分類提供內建于 Azure SQL Database 的先進功能，可用於探索、分類、標記 &amp; 保護資料庫中的敏感性資料。

如何使用 Azure SQL Database 的資料探索和分類：

https://docs.microsoft.com/azure/sql-database/sql-database-data-discovery-and-classification

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6：使用 Azure RBAC 來控制資源的存取權

**指導** 方針：使用 Azure Active Directory (Azure AD) 來驗證和控制 Azure SQL Database 實例的存取權。

如何整合 Azure SQL Database 與 Azure Active Directory 以進行驗證：

https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication

如何控制 Azure SQL Database 中的存取：

https://docs.microsoft.com/azure/sql-database/sql-database-control-access

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7：使用主機型資料外洩防護來強制執行存取控制

**指導** 方針： Microsoft 管理 Azure SQL Database 的基礎結構，並已實行嚴格的控制，以防止客戶資料遺失或洩漏。

瞭解 Azure 中的客戶資料保護：

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8：加密待用的敏感性資訊

**指導** 方針：透明資料加密 (TDE) 可透過加密待用資料，協助保護 Azure SQL Database、azure SQL 受控執行個體和 Azure 資料倉儲免于惡意離線活動的威脅。 它會對資料庫、相關聯的備份和待用的交易記錄檔執行即時加密和解密，而不需變更應用程式。 根據預設，SQL Database 和 SQL 受控執行個體中所有新部署的資料庫都會啟用 TDE。 TDE 加密金鑰可由 Microsoft 或客戶管理。

如何管理透明資料加密，並使用您自己的加密金鑰：

https://docs.microsoft.com/azure/sql-database/transparent-data-encryption-azure-sql?tabs=azure-portal#manage-transparent-data-encryption

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9：針對重要 Azure 資源的變更留下記錄和發出警示

**指導** 方針：使用 Azure 監視器搭配 Azure 活動記錄，以針對 Azure SQL Database 和其他重要或相關資源的生產實例進行變更時，建立警示。

如何建立 Azure 活動記錄事件的警示： 

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Azure 資訊安全中心監視**：是

**責任**：客戶

## <a name="vulnerability-management"></a>弱點管理

如需詳細資訊，請參閱[安全性控制：弱點管理](../../security/benchmarks/security-control-vulnerability-management.md) (機器翻譯)。

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1：執行自動化弱點掃描工具

**指導** 方針：啟用適用于 SQL 的 Azure Defender 進行 Azure SQL Database，並遵循在您的伺服器上執行弱點評定 Azure 資訊安全中心的建議。

如何在 Azure SQL Database 上執行弱點評定：

https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment

如何啟用適用于 SQL 的 Azure Defender：

https://docs.microsoft.com/azure/azure-sql/database/azure-defender-for-sql

如何實行 Azure 資訊安全中心弱點評定建議：

https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2：部署自動化的作業系統修補程式管理解決方案

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3：部署自動化的第三方軟體修補程式管理解決方案

**指導** 方針：不適用;此基準測試適用于計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4：比較連續性弱點掃描

**指導** 方針：啟用 Azure SQL Database 實例的定期週期性掃描;這會將弱點評估設定為每週自動在您的資料庫上執行掃描。 掃描結果摘要將傳送到您所提供的電子郵件地址。 比較結果，以確認已補救弱點。

如何在 Azure 資訊安全中心中匯出弱點評定報告：

https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment#implementing-vulnerability-assessment

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5：使用風險評等程序來排定所發現弱點的補救優先順序

**指導** 方針：使用 (Azure 資訊安全中心提供的安全分數) 的預設風險評等。

瞭解 Azure 資訊安全中心安全分數：

https://docs.microsoft.com/azure/security-center/security-center-secure-score

**Azure 資訊安全中心監視**：是

**責任**：客戶

## <a name="inventory-and-asset-management"></a>清查和資產管理

如需詳細資訊，請參閱[安全性控制：清查和資產管理](../../security/benchmarks/security-control-inventory-asset-management.md) (機器翻譯)。

### <a name="61-use-azure-asset-discovery"></a>6.1：使用 Azure 資產探索

**指導** 方針：使用 Azure Resource Graph 來查詢及探索所有資源 (包括訂用帳戶 () 中的 Azure SQL Database) 。  確保租用戶中有適當的 (讀取) 權限，且能列舉所有 Azure 訂用帳戶以及訂用帳戶中的資源。

雖然可透過 Resource Graph 探索傳統 Azure 資源，但強烈建議您從現在開始建立並使用 Azure Resource Manager 資源。

如何使用 Azure Resource Graph 建立查詢： https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

如何檢視 Azure 訂用帳戶： https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

了解 Azure RBAC： https://docs.microsoft.com/azure/role-based-access-control/overview

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="62-maintain-asset-metadata"></a>6.2：維護資產中繼資料

**指引**：將標籤套用至提供中繼資料的 Azure 資源，以邏輯方式依分類組織這些資源。

如何建立和使用標記：

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="63-delete-unauthorized-azure-resources"></a>6.3：刪除未經授權的 Azure 資源

**指導** 方針：使用標記、管理群組和個別訂用帳戶（如果適用）來組織和追蹤資產。 請定期調節清查，並確保會及時刪除訂用帳戶中未經授權的資源。

如何建立其他 Azure 訂用帳戶： 

https://docs.microsoft.com/azure/billing/billing-create-subscription

如何建立管理群組： 

https://docs.microsoft.com/azure/governance/management-groups/create

如何建立和使用標記：

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4：維護受核准 Azure 資源和軟體標題的清查

**指導** 方針：定義您的計算資源的已核准 Azure 資源清單和已核准的軟體

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5：監視未經核准的 Azure 資源

**指引**：使用下列內建原則定義，以利用 Azure 原則對可在客戶訂用帳戶中建立的資源類型施加限制：

- 不允許的資源類型

- 允許的資源類型

使用 Azure Resource Graph 來查詢/探索您的訂用帳戶 (s) 中的資源。 確保已核准環境中的所有 Azure 資源。

如何設定和管理 Azure 原則： https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

如何使用 Azure Graph 建立查詢： https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6：監視計算資源內未經核准的軟體應用程式

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7：移除未經核准的 Azure 資源和軟體應用程式

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="68-use-only-approved-applications"></a>6.8：僅使用已核准的應用程式

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="69-use-only-approved-azure-services"></a>6.9：僅使用已核准的 Azure 服務

**指導** 方針：使用 Azure 原則針對可在客戶訂用帳戶 (s 中建立的資源類型，使用下列內建原則定義來限制其) ：

- 不允許的資源類型

- 允許的資源類型

使用 Azure Resource Graph 來查詢/探索您的訂用帳戶 (s) 中的資源。 確保已核准環境中的所有 Azure 資源。

如何設定和管理 Azure 原則： https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

如何使用 Azure 原則拒絕特定的資源類型： https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="610-implement-approved-application-list"></a>6.10：實作已核准的應用程式清單

**指導** 方針：不適用;這項建議適用于在計算資源上執行的應用程式。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11：透過指令碼限制使用者與 Azure Resource Manager 互動的能力

**指引**：使用 Azure 條件式存取，藉由對「Microsoft Azure 管理」應用程式設定「封鎖存取」，以限制使用者與 Azure Resource Manager 互動的能力。

如何設定條件式存取以封鎖 Azure Resource Manager 的存取： https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12：限制使用者在計算資源內執行指令碼的能力

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13：以實體或邏輯方式隔離高風險的應用程式

**指導** 方針：不適用;這項建議適用于裝載桌面或 web 應用程式的 App Service 或計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

## <a name="secure-configuration"></a>安全設定

如需詳細資訊，請參閱[安全性控制：安全設定](../../security/benchmarks/security-control-secure-configuration.md) (機器翻譯)。

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1：為所有 Azure 資源建立安全設定

**指導** 方針：使用 Azure 原則或 Azure 資訊安全中心建議進行 Azure SQL Database 維護所有 Azure 資源的安全性設定。

如何設定和管理 Azure 原則：

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="72-establish-secure-operating-system-configurations"></a>7.2：建立安全的作業系統設定

**指導** 方針：不適用;這是適用于計算資源的建議。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3：維護安全的 Azure 資源設定

**指引**：使用 Azure 原則 [拒絕] 和 [在不存在時部署]，以對 Azure 資源強制使用安全設定。

如何設定和管理 Azure 原則： 

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

了解 Azure 原則效果： 

https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4：維護安全的作業系統設定

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5：安全地儲存 Azure 資源的設定

**指導** 方針：如果使用自訂 Azure 原則定義，請使用 Azure DevOps 或 Azure Repos 安全地儲存和管理您的程式碼。

如何在 Azure DevOps 中儲存程式碼： 

https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Azure Repos 文件： 

https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="76-securely-store-custom-operating-system-images"></a>7.6：安全地儲存自訂作業系統映像

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="77-deploy-system-configuration-management-tools"></a>7.7：部署系統設定管理工具

**指導** 方針：使用 "Microsoft .sql" 命名空間中 Azure 原則別名來建立自訂原則，以警示、審核和強制執行系統組態。 此外，開發流程和管線以管理原則例外狀況。

如何設定和管理 Azure 原則： 

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8：部署作業系統的系統設定管理工具

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9：為 Azure 服務實作自動化的設定監視

**指導** 方針：利用 Azure 資訊安全中心來執行 Azure SQL Database 的基準掃描。

如何修復 Azure 資訊安全中心中的建議：

https://docs.microsoft.com/azure/security-center/security-center-sql-service-recommendations

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10：為作業系統實作自動化的設定監視

**指引**：不適用，這項建議主要用於計算資源。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="711-manage-azure-secrets-securely"></a>7.11：安全地管理 Azure 祕密

**指導** 方針：使用 Azure Key Vault 儲存 Azure SQL Database 透明資料加密 (TDE) 的加密金鑰。

如何保護儲存在 Azure SQL Database 中的機密資料，並將加密金鑰儲存在 Azure Key Vault 中：

https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="712-manage-identities-securely-and-automatically"></a>7.12：安全且自動地管理身分識別

**指導** 方針：使用受控識別，在 Azure Active Directory (Azure AD) 中為 Azure 服務提供自動管理的身分識別。 受控識別可讓您向任何支援 Azure AD 驗證的服務進行驗證，包括 Azure Key Vault，而不需要在您的程式碼中提供任何認證。

教學課程：使用 Windows VM 系統指派的受控識別來存取 Azure SQL：

https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql

如何設定受控識別： 

https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13：消除非預期的認證公開

**指導** 方針：執行認證掃描器來識別程式碼中的認證。 認證掃描器也有助於將探索到的認證移至更安全的位置，例如 Azure Key Vault。

如何設定認證掃描器： https://secdevtools.azurewebsites.net/helpcredscan.html

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="malware-defense"></a>惡意程式碼防禦

如需詳細資訊，請參閱[安全性控制：惡意程式碼防禦](../../security/benchmarks/security-control-malware-defense.md) (機器翻譯)。

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1：使用集中管理的反惡意程式碼軟體

**指引**：不適用，這項建議主要用於計算資源。 Microsoft 會處理基礎平臺的反惡意程式碼。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2：預先掃描要上傳至非計算 Azure 資源的檔案

**指導** 方針：支援 Azure 服務的基礎主機上已啟用 Microsoft 反惡意程式碼 (例如 Azure App Service) ，但不會對客戶內容執行。

預先掃描即將上傳至非計算 Azure 資源的任何內容，例如 App Service、Data Lake Storage、Blob 儲存體、Azure SQL Database 等等。Microsoft 無法在這些實例中存取您的資料。

瞭解 Azure 雲端服務和虛擬機器的 Microsoft Antimalware： https://docs.microsoft.com/azure/security/fundamentals/antimalware

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3：確保更新反惡意程式碼軟體和簽章

**指引**：不適用，這項建議主要用於計算資源。 Microsoft 會處理基礎平臺的反惡意程式碼。

**Azure 資訊安全中心監視**：不適用

**責任**：不適用

## <a name="data-recovery"></a>資料復原

如需詳細資訊，請參閱[安全性控制：資料復原](../../security/benchmarks/security-control-data-recovery.md) (機器翻譯)。

### <a name="91-ensure-regular-automated-back-ups"></a>9.1：確保會定期自動備份

**指導** 方針：為了保護您的業務免于資料遺失，Azure SQL Database 每隔12小時自動建立完整資料庫備份、每隔12小時建立一次差異資料庫備份，以及每隔 5-10 分鐘備份一次交易記錄。 所有服務層級的備份至少會儲存在 GRS 儲存體中，至少7天。 所有服務層級，但基本支援可設定時間點還原的備份保留期限，最多35天。

為了符合不同的合規性需求，您可以針對每週、每月及/或每年備份選取不同的保留期限。 儲存體耗用量取決於選取的備份頻率和保留期間。

使用 Azure SQL Database 瞭解備份和商務持續性：

https://docs.microsoft.com/azure/sql-database/sql-database-business-continuity

**Azure 資訊安全中心監視**：是

**責任**：共用

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2：執行完整的系統備份，並備份任何客戶管理的金鑰

**指導** 方針： Azure SQL Database 會自動建立保留7到35天的資料庫備份，並使用 Azure 讀取權限異地冗余儲存體 (GRS) ，以確保即使資料中心無法使用，仍會保留這些備份。 這些備份會自動建立。 如有需要，請為您的 Azure SQL 資料庫啟用長期異地多餘備份。

如果透明資料加密使用客戶管理的金鑰，請確定您的金鑰已備份。

瞭解 Azure SQL Database 中的備份：

https://docs.microsoft.com/azure/sql-database/sql-database-automated-backups?tabs=single-database

如何在 Azure 中備份金鑰保存庫金鑰：

https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3：驗證所有備份，包括客戶管理的金鑰

**指導** 方針：確保能夠定期對 Azure 備份內的內容進行資料還原。 如有必要，請測試將內容還原至隔離的 VLAN。 測試已備份之客戶管理金鑰的還原。

如何在 Azure 中還原金鑰保存庫金鑰：

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0

如何使用時間點還原來復原 Azure SQL Database 備份：

https://docs.microsoft.com/azure/sql-database/sql-database-recovery-using-backups#point-in-time-restore

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4：確保備份和客戶管理的金鑰受到保護

**指導** 方針：啟用 Azure Key Vault 中的虛刪除，以防止金鑰遭到意外或惡意刪除。

如何在 Key Vault 中啟用虛刪除：

https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

**Azure 資訊安全中心監視**：是

**責任**：客戶

## <a name="incident-response"></a>事件回應

如需詳細資訊，請參閱[安全性控制：事件回應](../../security/benchmarks/security-control-incident-response.md) (機器翻譯)。

### <a name="101-create-an-incident-response-guide"></a>10.1：建立事件回應指南

**指導** 方針：確定有撰寫的事件回應計畫，可定義人員角色以及事件處理/管理階段。

如何設定 Azure 資訊安全中心內的工作流程自動化： 

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2：建立事件評分和優先順序程序

**指導** 方針：資訊安全中心會將嚴重性指派給警示，以協助您優先處理每個警示的順序，如此一來，當資源遭到入侵時，您就可以立即取得。 嚴重性的依據，在於資訊安全中心對於據以發出警示的發現結果或分析結果有多少信心，以及認定導致警示的活動背後存在惡意意圖的把握程度。

Azure 資訊安全中心中的安全性警示： https://docs.microsoft.com/azure/security-center/security-center-alerts-overview

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="103-test-security-response-procedures"></a>10.3：測試安全性回應程序

**指引**：進行練習以定期測試系統的事件回應功能。 找出弱點和落差，並視需要修訂計畫。

您可以參考 NIST 的發行集：適用于 IT 方案和功能的測試、訓練和練習程式指南：

https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4：提供安全性事件連絡人詳細資料，並設定安全性事件的警示通知

**指導** 方針： Microsoft 安全性回應中心 (MSRC) 發現您的資料已被非法或未經授權的物件存取，microsoft 會使用安全性事件連絡人資訊來與您聯繫。

如何設定 Azure 資訊安全中心安全性連絡人：

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5：將安全性警示併入事件回應系統

**指引**：使用「連續匯出」功能來匯出 Azure 資訊安全中心的警示和建議。 「連續匯出」可供以手動或持續不斷的方式來匯出警示和建議。 您可以使用 Azure 資訊安全中心資料連線器，將警示串流至 Sentinel。

如何設定連續匯出： 

https://docs.microsoft.com/azure/security-center/continuous-export

如何將警示串流至 Azure Sentinel：

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="106-automate-the-response-to-security-alerts"></a>10.6：自動回應安全性警示

**指引**：使用 Azure 資訊安全中心的工作流程自動化功能，以透過 "Logic Apps" 自動觸發對安全性警示和建議的回應。

如何設定工作流程自動化和 Logic Apps： 

https://docs.microsoft.com/azure/security-center/workflow-automation

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

## <a name="penetration-tests-and-red-team-exercises"></a>滲透測試和 Red Team 練習

如需詳細資訊，請參閱[安全性控制：滲透測試和 Red Team 練習](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md) (機器翻譯)。

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1：進行 Azure 資源的定期滲透測試，並確保在 60 天內補救所有重大安全性發現

**指導** 方針：請遵循 Microsoft 的參與規則，以確保您的滲透測試不違反 Microsoft 原則：

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.

您可以在以下位置找到 Microsoft 針對 Microsoft 受控雲端基礎結構、服務和應用程式進行的 Microsoft 策略、Red 小組和即時網站滲透測試的詳細資訊： https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Azure 資訊安全中心監視**：不適用

**責任**：共用

## <a name="next-steps"></a>後續步驟

- 請參閱 [Azure 安全性效能評定](../../security/benchmarks/overview.md) (機器翻譯)
- 深入了解 [Azure 安全性基準](../../security/benchmarks/security-baselines-overview.md) (機器翻譯)