---
title: Azure 安全性與合規性藍圖 - 適用於 NIST SP 800-171 的資料倉儲
description: Azure 安全性與合規性藍圖 - 適用於 NIST SP 800-171 的資料倉儲
services: security
author: jomolesk
ms.assetid: dbc9cafe-115d-4965-b0d4-fcf235a064c8
ms.service: security
ms.topic: article
ms.date: 07/31/2018
ms.author: jomolesk
ms.openlocfilehash: f774aed837fac1829413493c98b6df0e4ca30600
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393208"
---
# <a name="azure-security-and-compliance-blueprint---data-warehouse-for-nist-sp-800-171"></a>Azure 安全性與合規性藍圖 - 適用於 NIST SP 800-171 的資料倉儲

## <a name="overview"></a>概觀
[NIST 特刊 800-171](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-171.pdf) 提供的指導方針適用於保護位於非聯邦資訊系統與組織中的受管控非機密資訊 (CUI)。 NIST SP 800-171 建立 14 組安全性需求，以供保護 CUI 的機密性。

此「Azure 安全性與合規性藍圖」提供指導方針，協助客戶在 Azure 中部署資料倉儲架構，以實作部分的 NIST SP 800-171 控制項。 此解決方案示範客戶可以符合特定安全性和合規性需求的方式。 客戶在 Azure 中建置並設定自己的資料倉儲解決方案時，也可以之為基礎。

與實作指南相關聯的此參考架構和威脅模型，主要是用來作為基礎，讓客戶進行調整以符合其特定需求。 不應完全未經修改就用於生產環境中。 客戶必須負責針對使用此架構建立的任何解決方案，進行適當的安全性與合規性評量。 需求可能會因為每個客戶的實作詳細規格而有所不同。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件
此解決方案提供參考架構，可實作高效能且安全的雲端資料倉儲。 在此架構中有兩個不同的資料層。 一層是在叢集的 SQL 環境內匯入、儲存並暫存資料。 另一層則適用於 SQL 資料倉儲。 利用這一層，可使用擷取-轉換-載入 (ETL) 工具 (例如 [PolyBase](https://docs.microsoft.com/azure/sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase) T-SQL 查詢) 進行處理來載入資料。 資料儲存在 SQL 資料倉儲之後，您就可以大規模執行分析。

Azure 為客戶提供各種報告和分析服務。 此解決方案包括可從 SQL 資料倉儲快速建立報告的 SQL Server Reporting Services。 所有 SQL 流量都會使用 SSL 透過內含的自我簽署憑證來加密。 建議使用的最佳做法是使用受信任的憑證授權單位，來增強安全性。

Azure SQL 資料倉儲會將資料儲存到具有單欄式儲存體的關聯式資料表。 這種格式會大幅減少資料儲存體成本，同時可善查詢效能。 根據使用需求，SQL 資料倉儲的計算資源可以相應增加或減少，或在沒有使用中程序需要計算資源時完全關閉。

SQL Server 負載平衡器會管理 SQL 流量，確保維持高效能。 此參考架構中的所有虛擬機器 (VM) 都部署在可用性設定組中，並搭配使用在 AlwaysOn 可用性群組中設定的 SQL Server 執行個體。 此設定提供高可用性和災害復原的能力。

此資料倉儲參考架構也包含用來管理架構中資源的 Active Directory 層。 Active Directory 子網路可讓您更願意採用較大型 Active Directory 樹系結構。 如此一來，即使無法存取較大樹系，也可讓該環境繼續運作。 所有 VM 都會加入 Active Directory 層的網域。 使用 Active Directory 群組原則來強化作業系統層級的安全性與合規性設定。

解決方案會使用 Azure 儲存體帳戶，客戶可進行設定，以便使用儲存體服務加密來維護待用資料的機密性。 Azure 會在客戶所選的資料中心內儲存三份資料複本以供復原之用。 異地備援儲存體可確保資料會複寫至數百英哩外的次要資料中心，並於該資料中心內再次儲存為第三份複本。 這種做法可防止客戶主要資料中心的不良反應造成資料遺失。

為加強安全性，此解決方案中的所有資源都會透過 Azure Resource Manager 以資源群組方式管理。 Azure Active Directory (Azure AD) 角色型存取控制 (RBAC) 可用來控制對已部署的資源所做的存取。 這些資源會包含 Azure Key Vault 中的客戶金鑰。 透過 Azure 資訊安全中心和 Azure 監視器來監視系統健康情況。 客戶可設定這兩項監視服務來擷取記錄。 在易於瀏覽的單一儀表板上顯示系統健康情況。

VM 可作為管理防禦主機使用。 提供可讓系統管理員存取部署資源的安全連線。 資料會透過此管理防禦主機載入暫存區域。 *建議您設定 VPN 或 Azure ExpressRoute 連線，以便管理和將資料匯入參考架構子網路。*

![適用於 NIST SP 800-171 的資料倉儲參考架構圖表](images/nist171-dw-architecture.png "適用於 NIST SP 800-171 的資料倉儲參考架構圖表")

此解決方案會使用下列 Azure 服務。 如需詳細資訊，請參閱[部署架構](#deployment-architecture)小節。

- 可用性設定組
    - Active Directory 網域控制站
    - SQL Server 叢集節點和見證
- Azure Active Directory
- Azure 資料目錄
- Azure 金鑰保存庫
- Azure 監視器
- Azure 資訊安全中心
- Azure Load Balancer
- Azure 儲存體
- Azure Log Analytics
- Azure 虛擬機器
    - (1) 防禦主機
    - (2) Active Directory 網域控制站
    - (2) SQL Server 叢集節點
    - (1) SQL Server 見證
- Azure 虛擬網路
    - (1) /16 個網路
    - (4) /24 個網路
    - (4) 網路安全性群組
- 復原服務保存庫
- SQL 資料倉儲
- SQL Server Reporting Services

## <a name="deployment-architecture"></a>部署架構
下一節會詳細說明部署和實作元素。

**Azure SQL 資料倉儲**：[SQL 資料倉儲](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)是企業資料倉儲，其利用大量平行處理，快速地在數以 PB 計的資料中執行複雜的查詢。 使用者可以使用簡單的 PolyBase T-SQL 查詢將巨量資料匯入 SQL 資料倉儲，然後使用大量平行處理功能來執行高效能分析。

**SQL Server Reporting Services**：[SQL Server Reporting Services](https://docs.microsoft.com/sql/reporting-services/report-data/sql-azure-connection-type-ssrs) 可快速地以資料表、圖表、地圖、量測計、矩陣等形式，為 SQL 資料倉儲建立報表。

**Azure 資料目錄**：[資料目錄](https://docs.microsoft.com/azure/data-catalog/data-catalog-what-is-data-catalog)能讓管理資料的使用者輕鬆地探索和了解資料來源。 您可以註冊、標記常用資料來源，並在其中搜尋資料。 資料會保留在現有的位置，但其中繼資料的複本會新增至資料目錄。 資料來源位置的參考也包含在內。 此中繼資料會編製索引，透過搜尋輕鬆找到每個資料來源。 編製索引也讓探索資料來源的使用者了解每個資料來源。

**防禦主機**：防禦主機是單一進入點，使用者可用以存取此環境中已部署的資源。 防禦主機為已部署的資源提供安全連線，其允許的遠端流量僅限來自安全清單上的公用 IP 位址。 若要允許遠端桌面流量，必須在網路安全性群組中定義該流量的來源。

此解決方案會建立 VM，作為具有下列設定之加入網域的防禦主機：
-   [反惡意程式碼軟體擴充功能](https://docs.microsoft.com/azure/security/azure-security-antimalware)。
-   [Azure 診斷擴充功能](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-extensions-diagnostics-template)。
-   使用 Key Vault 的 [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)。
-   [自動關閉原則](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/) \(英文\)，可減少不使用 VM 資源時的耗用量。
-   [Windows Defender Credential Guard](https://docs.microsoft.com/windows/access-protection/credential-guard/credential-guard) 已啟用，因此認證和其他祕密都會在受保護的環境中執行，且該環境會與正在執行的作業系統隔離。

### <a name="virtual-network"></a>虛擬網路
此參考架構會定義位址空間為 10.0.0.0/16 的私人虛擬網路。

**網路安全性群組**：[網路安全性群組 (NSG)](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 包含能允許或拒絕虛擬網路內流量的存取控制清單。 NSG 可以用來保護子網路或個別 VM 層級的流量。 下列 NSG 已經存在:
  - 適用於資料層的 NSG (SQL Server 叢集、SQL Server 見證和 SQL Load Balancer)
  - 適用於管理防禦主機的 NSG
  - 適用於 Active Directory 的 NSG
  - 適用於 SQL Server Reporting Services 的 NSG

每個 NSG 都會開放特定連接埠及通訊協定，讓解決方案可安全且正確地運作。 此外，以下設定會針對每個 NSG 啟用：
  - 啟用[診斷記錄和事件](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)並儲存在儲存體帳戶。
  - Log Analytics 會連線至 [NSG 的診斷](https://github.com/krnese/AzureDeploy/blob/master/AzureMgmt/AzureMonitor/nsgWithDiagnostics.json)。

**子網路**：確認每個子網路都與對應的 NSG 建立關聯。

### <a name="data-at-rest"></a>待用資料
架構會透過多個量值來保護待用資料。 這些量值包括加密和資料庫稽核。

**Azure 儲存體**：為符合加密的待用資料需求，所有[儲存體](https://azure.microsoft.com/services/storage/)都會使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)。 此功能有助於保護與防衛資料，以支援組織的安全性承諾及合規性需求。

**Azure 磁碟加密**：[磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)使用 Windows 的 BitLocker 功能來提供作業系統和資料磁碟的磁碟區加密。 此解決方案與 Key Vault 整合，以協助控制和管理磁碟加密金鑰。

**Azure SQL Database**：SQL Database 執行個體會使用下列資料庫安全性量值：
-   [Active Directory 驗證與授權](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)可讓您在一個中央位置，管理資料庫使用者及其他 Microsoft 服務的身分識別。
-   [SQL Database 稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started)會追蹤資料庫事件並將事件寫入 Azure 儲存體帳戶中的稽核記錄。
-   SQL Database 會設定為使用[透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)。 其會執行資料庫、相關聯的備份及交易記錄檔的即時加密和解密，以保護待用資訊。 透明資料加密能保證已儲存的資料不會遭到未經授權的存取。
-   [防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)可在授與適當權限之前，防止所有對資料庫伺服器的存取。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。
-   [SQL 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-get-started)可在發生潛在威脅時啟用偵測及回應。 針對可疑的資料庫活動、潛在弱點、SQL 插入式攻擊，以及異常的資料庫存取模式，提供安全性警示。
-   [加密資料行](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)可確保敏感性資料在資料庫系統內一律不會以純文字顯示。 資料加密啟用之後，只有具備金鑰存取權的用戶端應用程式或應用程式伺服器才可以存取純文字資料。
- [擴充屬性](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addextendedproperty-transact-sql)可用來中斷資料主體的處理。 使用者可將自訂屬性新增至資料庫物件。 他們也可以將資料標記為「已中止」，以支援應用程式邏輯來防止處理相關聯的財務資料。
- [資料列層級安全性](https://docs.microsoft.com/sql/relational-databases/security/row-level-security)可讓使用者定義原則，限制資料存取以中斷處理。
- [SQL Database 動態資料遮罩](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)可透過遮蔽資料，使不具權限的使用者或應用程式無法看見敏感性資料。 其可以自動探索潛在的敏感性資料，並建議套用適當的遮罩。 動態資料遮罩有助於減少存取，敏感性資料就不會經由未經授權的存取而離開資料庫。 *客戶須負責調整設定，以符合其資料庫結構描述。*

### <a name="identity-management"></a>身分識別管理
下列技術可在 Azure 環境中提供管理資料存取的功能：
-   [Azure AD](https://azure.microsoft.com/services/active-directory/) 是 Microsoft 多租用戶雲端式目錄和身分識別管理服務。 此解決方案的所有使用者都是在 Azure AD 中建立，包括存取 SQL 資料庫的使用者。
-   應用程式的驗證是使用 Azure AD 執行。 如需詳細資訊，請參閱如何[整合應用程式與 Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。 資料庫資料行加密也會使用 Azure AD 向 SQL Database 驗證應用程式。 如需詳細資訊，請參閱如何[保護 SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
-   [Azure RBAC](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure) 可由系統管理員用來定義微調存取權限。 因此可以僅授與使用者執行其作業所需的存取權。 系統管理員可以只允許存取資源和資料的特定動作，而不是授與所有使用者不受限制的 Azure 資源存取權。 只有訂用帳戶管理員可擁有訂用帳戶的存取權。
- [Azure Active Directory Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started) 可由客戶用來將可存取特定資訊 (例如資料) 的使用者人數降至最低。 系統管理員可以使用 Azure AD Privileged Identity Management 來探索、限制和監視特殊權限的身分識別和其對資源的存取。 如有需要，此功能也可用來強制執行隨選 Just-In-Time 系統管理存取權。
- [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測可能影響您組織身分識別的潛在弱點。 其會為偵測到的組織身分識別相關可疑活動設定自動回應。 還會調查可疑事件以採取適當動作來解決它們。

### <a name="security"></a>安全性
**祕密管理**：解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和祕密。 Key Vault 可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和祕密。 下列 Key Vault 功能可協助客戶保護資料：
- 進階存取原則是視需要設定的。
- Key Vault 存取原則是使用金鑰和祕密的最低必要權限所定義的。
- Key Vault 中的所有金鑰和祕密都有到期日。
- Key Vault 中的所有金鑰都會受到特製化硬體安全模組的保護。 金鑰類型是硬體安全性模組保護的 2048 位元 RSA 金鑰。
- 所有使用者和身分識別皆使用 RBAC 授與最低必要權限。
- 金鑰保存庫的診斷記錄的保留期至少 365 天。
- 允許的金鑰密碼編譯作業僅限於需要的密碼編譯項目。

**修補程式管理**：根據預設，部署為此參考架構一部分的 Windows VM 會設定為從 Windows Update 服務接收自動更新。 此解決方案也包括 [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)服務，您可以透過此服務建立更新的部署，以在有所需要時修補 VM。

**惡意程式碼防護**：適用於VM 的 [Microsoft Antimalware](https://docs.microsoft.com/azure/security/azure-security-antimalware) 提供即時保護功能，有助於識別和移除病毒、間諜軟體和其他惡意軟體。 客戶可以設定警示，在已知的惡意或垃圾軟體嘗試安裝或在受保護的 VM 上執行時產生。

**Azure 資訊安全中心**：客戶可以使用[資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)，在工作負載之間集中套用及管理安全性原則、限制暴露於威脅的程度，以及偵測和回應攻擊。 資訊安全中心也會存取 Azure 服務的現有組態，以提供設定和服務建議，協助改善安全性狀態並保護資料。

資訊安全中心會使用不同的偵測功能，向客戶警示以其環境為目標的潛在攻擊。 這些警示包含觸發警示的項目、鎖定為目標的資源，以及攻擊來源等重要資訊。 資訊安全中心有一組預先定義的[安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)，一旦發生威脅或可疑活動時便會觸發這些警示。 客戶可使用[自訂警示規則](https://docs.microsoft.com/azure/security-center/security-center-custom-alert)，根據從其環境所收集到的資料，定義新的安全性警示。

資訊安全中心提供優先的安全性警示和事件。 資訊安全中心可讓客戶更容易探索及解決潛在的安全性問題。 針對每個偵測到的威脅會產生[威脅情報報告](https://docs.microsoft.com/azure/security-center/security-center-threat-report)。 事件回應小組可在調查威脅並進行補救時使用這份報告。

此參考架構也會使用資訊安全中心中的[漏洞評量](https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations)功能。 設定之後，夥伴代理程式 (例如 Qualys) 會向夥伴的管理平台回報弱點資料。 接著，夥伴的管理平台會提供弱點和健康情況監視資料給資訊安全中心。 客戶可以使用此資訊，快速識別易受攻擊的 VM。

### <a name="business-continuity"></a>業務持續性
**高可用性**：伺服器工作負載會組成[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-manage-availability?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，以協助確保 Azure 中 VM 的高可用性。 在進行計劃性或非計劃性的維護事件期間，至少有一部 VM 可使用，進而符合 99.95% 的 Azure SLA。

**復原服務保存庫**：[復原服務保存庫](https://docs.microsoft.com/azure/backup/backup-azure-recovery-services-vault-overview)可裝載備份資料，並保護此架構中所有 VM 的組態。 客戶可以使用復原服務保存庫，從 IaaS VM 還原檔案和資料夾，而非還原整個 VM。 此流程可加速還原時間。

### <a name="logging-and-auditing"></a>記錄與稽核

Azure 服務會廣泛記錄系統、使用者活動及系統健康情況：
- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。 活動記錄可協助判斷作業的啟動器、出現時間和狀態。
- **診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)包含每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、儲存體記錄、Key Vault 稽核記錄，以及 Azure 應用程式閘道存取和防火牆記錄。 所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存。 使用者可設定最多 730 天的保留期限，以符合其特定需求。

**Log Analytics**：這些記錄會合併到 [Log Analytics](https://azure.microsoft.com/services/log-analytics/) 以進行處理、儲存，並從儀表板顯示報告。 所收集的資料會針對 Operations Management Suite 工作區內的每種資料類型組織成個別的資料表。 如此一來，所有的資料都能一起分析 (不論其原始來源為何)。 資訊安全中心會與 Log Analytics 整合。 客戶可以使用 Log Analytics 查詢來存取其安全性事件資料，並將其與來自其他服務的資料合併。

此架構包含下列 Log Analytics [管理解決方案](https://docs.microsoft.com/azure/log-analytics/log-analytics-add-solutions)：
-   [Active Directory 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：Active Directory 健康情況檢查方案可定期評估伺服器環境的風險和健全狀況。 其能針對已部署的伺服器基礎結構提供依照優先順序排列的具體建議清單。
- [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查方案可定期評估伺服器環境的風險和健康情況。 其能針對已部署的伺服器基礎結構提供客戶依照優先順序排列的具體建議清單。
- [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈。 還會報告沒有回應的代理程式數目和提交作業資料的代理程式數目。
-   [活動記錄分析](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：「活動記錄分析」解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。

**Azure 自動化**：[自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)會儲存、執行和管理 Runbook。 在此解決方案中，Runbook 會協助從 SQL Database 中收集記錄。 客戶可以使用自動化[變更追蹤](https://docs.microsoft.com/azure/automation/automation-change-tracking)解決方案輕鬆地識別環境中的變更。

**Azure 監視器**：[監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)可幫助使用者追蹤效能、維護安全性及找出趨勢。 組織可用來稽核、建立警示及封存資料。 還可以追蹤使用者 Azure 資源中的 API 呼叫。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖可供[下載](https://aka.ms/nist171-dw-tm)，或可以在這裡找到。 此模型可協助客戶在進行修改時，了解系統基礎結構中的潛在風險要點。

![適用於 NIST SP 800-171 的資料倉儲威脅模型](images/nist171-dw-threat-model.png "適用於 NIST SP 800-171 的資料倉儲威脅模型")

## <a name="compliance-documentation"></a>合規性文件
[Azure 安全性與合規性藍圖 – NIST SP 800-171 客戶責任對照表](https://aka.ms/nist171-crm)會列出 NIST SP 800-171 要求的所有安全性控制項。 此對照表詳細說明每個控制項的實作是由 Microsoft、客戶還是兩者共同負責。

[Azure 安全性與合規性藍圖 - NIST SP 800-171 資料倉儲控制項實作對照表](https://aka.ms/nist171-dw-cim)提供資料倉儲架構涵蓋哪些 NIST SP 800-171 控制項的相關資訊。 它包括詳細說明此實作如何符合每個涵蓋控制項的需求。

## <a name="guidance-and-recommendations"></a>指導方針與建議

### <a name="vpn-and-expressroute"></a>VPN 和 ExpressRoute
您必須設定安全的 VPN 通道或 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)，以安全地連線到部署為此資料倉儲參考架構一部分的資源。 透過適當地設定 VPN 或 ExpressRoute，客戶可以在傳輸過程中新增資料保護層。

透過實作與 Azure 的安全 VPN 通道，即可建立內部部署網路與 Azure 虛擬網路之間的虛擬私人連線。 此連線透過網際網路進行。 客戶能使用此連線，在客戶的網路與 Azure 之間的加密連結內安全地「輸送」資訊。 站對站 VPN 是安全成熟的技術，各種規模的企業已部署數十年。 此選項使用 [IPsec 通道模式](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2003/cc786385(v=ws.10))作為加密機制。

由於 VPN 通道內的流量會透過站對站 VPN 周遊網際網路，因此 Microsoft 提供另一個更安全的連線選項。 ExpressRoute 是 Azure 與內部部署位置或 Exchange 主機服務提供者之間專用的 WAN 連結。 ExpressRoute 連線會與客戶的電信提供者直接連線。 因此，資料不會透過網際網路傳輸，也不會公開給網際網路。 相較於一般連線，這些連線可提供更為可靠、速度更快、延遲更低且安全性更高的網際網路連線。

如需實作將內部部署網路擴充至 Azure 之安全混合式網路的最佳做法，請參閱[這裡](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid)。

### <a name="extract-transform-load-process"></a>擷取-轉換-載入程序
[PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) 可以將資料載入 SQL 資料倉儲中，而不需要個別的 ETL 或匯入工具。 PolyBase 允許透過 T-SQL 查詢來存取資料。 Microsoft 的商業智慧和分析堆疊，以及與 SQL Server 相容的協力廠商工具，都可以搭配 PolyBase 使用。

### <a name="azure-ad-setup"></a>Azure AD 設定
您必須使用 [Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) 來管理部署，以及為與環境互動的人員佈建存取權。 您只要[點按四下](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-get-started-express)，就可以整合內部部署 Active Directory 與 Azure AD。 客戶也可以將部署的 Active Directory 基礎結構 (網域控制站) 繫結至 Azure AD。 若要這樣做，可讓部署的 Active Directory 基礎結構成為 Azure AD 樹系的子網域。

### <a name="optional-services"></a>選擇性服務
Azure 提供各種不同的服務，以協助儲存及暫存格式化與未格式化的資料。 根據客戶需求，可將下列服務新增至此參考架構：
-   [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction) 是一個針對複雜混合式 ETL 擷取-載入-轉換及資料整合專案建置的受控雲端服務。 Data Factory 中的功能可協助追蹤及尋找資料。 視覺化和監視工具可識別資料何時抵達及其來源。 客戶可建立並排定資料驅動工作流程 (稱為管線)，以從不同的資料存放區擷取資料。 客戶可使用管線從內部和外部來源內嵌資料。 接著，客戶可以處理並轉換資料，以輸出至資料存放區，例如 SQL 資料倉儲。
- 客戶可以在 [Azure Data Lake Store](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview) 中暫存非結構化的資料，在一個位置以任何擷取速度來擷取任何大小和類型的資料，以進行運作和探勘分析。 Azure Data Lake 具有擷取及轉換資料的功能。 Data Lake Store 與 Hadoop 生態系統中的大多數開放原始碼元件相容。 此外，還與其他 Azure 服務完美整合，例如 SQL 資料倉儲。

## <a name="disclaimer"></a>免責聲明

 - 此文件僅供參考之用。 Microsoft 對此文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 此文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
 - 此文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
 - 客戶可以複製並使用這份文件，供內部參考之用。
 - 此文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
 - 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
 - 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
