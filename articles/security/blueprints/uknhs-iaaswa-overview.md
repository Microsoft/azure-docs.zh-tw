---
title: Azure 安全性與合規性藍圖 - 適用於 UK NHS 的 IaaS Web 應用程式
description: Azure 安全性與合規性藍圖 - 適用於 UK NHS 的 IaaS Web 應用程式
services: security
author: jomolesk
ms.assetid: 6081bab2-315a-4af4-92a1-7c773f449d66
ms.service: security
ms.topic: article
ms.date: 06/15/2018
ms.author: jomolesk
ms.openlocfilehash: 763ef04fbd14820ebecaa114d539b1026ebf4131
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37343154"
---
# <a name="azure-security-and-compliance-blueprint-iaas-web-application-for-uk-nhs"></a>Azure 安全性與合規性藍圖：適用於 UK NHS 的 IaaS Web 應用程式

## <a name="overview"></a>概觀

此 Azure 安全性與合規性藍圖提供基礎結構即服務 (IaaS) Web 應用程式的參考架構和指導方針，適用於醫療保健資料的收集、儲存和擷取。 此解決方案會示範客戶可如何遵守《[雲端安全性最佳做法指南](https://digital.nhs.uk/data-and-information/looking-after-information/data-security-and-information-governance/nhs-and-social-care-data-off-shoring-and-the-use-of-public-cloud-services/health-and-social-care-cloud-security-good-practice-guide)》中提供的指導方針，此指南是由英國 (UK) 衛生和社會照護部門 (DHSC) 的夥伴 [NHS Digital](https://digital.nhs.uk/) 所發佈。 《雲端安全性最佳做法指南》是根據英國國家網路安全中心 (NCSC) 發佈的 14 個[雲端安全性準則](https://www.ncsc.gov.uk/guidance/implementing-cloud-security-principles)。

此參考架構、實作指南和威脅模型主要是用來作為基礎，讓客戶進行調整以符合其特定需求，不應未經其他設定就用於生產環境中。 客戶須負責為任何使用架構建置的解決方進行置適當的安全性與合規性評量，因為需求可能會隨每個客戶的實作細節而有所不同。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件

此解決方案會部署具有 SQL Server 後端之 IaaS Web 應用程式的參考架構。 架構包含 Web 層、資料層、Active Directory 基礎結構、應用程式閘道及負載平衡器。 部署到 Web 層和資料層的虛擬機器是設定在可用性設定組中，而 SQL Server 是設定在適用於高可用性的 Always On 可用性群組中。 虛擬機器已加入網域，而且使用 Active Directory 群組原則來強化作業系統層級的安全性與合規性設定。

解決方案使用 Azure 儲存體帳戶，客戶可以設定要使用儲存體服務加密來維護待用資料的機密性。 Azure 會在客戶所選的資料中心內儲存三份資料以供復原之用。 異地備援儲存體可確保資料會複寫到數百英哩遠的次要資料中心，並在該資料中心儲存為三份複本，防止客戶主要資料中心內的不良事件導致資料遺失。

為加強安全性，此解決方案中的所有資源都會透過 Azure Resource Manager 以資源群組方式管理。 Azure Active Directory 角色型存取控制可用來控制已部署資源的存取，包括其在 Azure Key Vault 中的金鑰。 透過 Azure 資訊安全中心和 Azure 監視器監視系統健康情況。 客戶可設定這兩項監視服務來擷取記錄，並在易於瀏覽的單一儀表板上顯示系統健康情況。 Azure 應用程式閘道已設定為防止模式中的防火牆，不允許非 TLSv1.2 的流量。

管理防禦主機提供可讓系統管理員存取部署資源的安全連線。 **Microsoft 會建議設定 VPN 或 ExpressRoute 連線，以便管理和將資料匯入參考架構子網路。**

![適用於 UK NHS 的 IaaS Web 應用程式參考架構圖表](images/uknhs-iaaswa-architecture.png?raw=true "適用於 UK NHS 的 IaaS Web 應用程式參考架構圖表")

此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。

- Azure 虛擬機器
    - (1) 管理/防禦 (Windows Server 2016 Datacenter)
    - (2) Active Directory 網域控制站 (Windows Server 2016 Datacenter)
    - (2) SQL Server 叢集節點 (Windows Server 2016 上的 SQL Server 2017)
    - (2) Web/IIS (Windows Server 2016 Datacenter)
- 可用性設定組 (Availability Sets)
    - (1) Active Directory 網域控制站
    - (1) SQL 叢集節點
    - (1) Web/IIS
- Azure 虛擬網路
    - (1) /16 網路
    - (5) /24 網路
    - (5) 網路安全性群組
    - 復原服務保存庫
- Azure 應用程式閘道
    - (1) Web 應用程式防火牆
        - 防火牆模式：防止
        - 規則集：OWASP 3.0
        - 接聽程式連接埠：443
- Azure Active Directory
- Azure 雲端見證
- Azure 金鑰保存庫
- Azure Load Balancer
- Azure 監視器
- Azure Resource Manager
- Azure 資訊安全中心
- Azure 自動化
- Azure 儲存體
    - (7) 異地備援儲存體帳戶

## <a name="deployment-architecture"></a>部署架構

下一節會詳細說明部署和實作元素。

**防禦主機**：防禦主機是單一進入點，可讓使用者存取此環境中已部署的資源。 防禦主機為已部署的資源提供安全連線，其允許的遠端流量僅限來自安全清單上的公用 IP 位址。 若要允許遠端桌面 (RDP) 流量，必須在網路安全性群組中定義該流量的來源。

此解決方案會建立虛擬機器，作為具有下列設定之加入網域的防禦主機：
-   [反惡意程式碼軟體擴充功能](https://docs.microsoft.com/azure/security/azure-security-antimalware)
-   [Azure 診斷擴充功能](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-extensions-diagnostics-template)
-   使用 Azure Key Vault 的 [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)
-   [自動關閉原則](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/) \(英文\)，可減少不使用虛擬機器資源時的耗用量。
-   [Windows Defender Credential Guard](https://docs.microsoft.com/windows/access-protection/credential-guard/credential-guard) 已啟用，因此認證和其他祕密都會在受保護的環境中執行，且該環境會與正在執行的作業系統隔離

### <a name="virtual-network"></a>虛擬網路

此架構會定義位址空間為 10.200.0.0/16 的私人虛擬網路。

**網路安全性群組**：此解決方案在虛擬網路內，將資源部署在具有個別的 Web 子網路、資料庫子網路、Active Directory 子網路及管理子網路的架構中。 套用至個別子網路的網路安全性群組規則以邏輯方式區隔了子網路，將子網路之間的流量限制為系統和管理功能所需的流量。

請參閱使用此解決方案部署之[網路安全性群組](https://github.com/Azure/fedramp-iaas-webapp/blob/master/nestedtemplates/virtualNetworkNSG.json)的設定。 組織可以使用[此文件](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)當作指南來編輯上述的檔案，以設定網路安全性群組。

每個子網路都有專用的網路安全性群組：
- 1 個網路安全性群組用於應用程式閘道 (LBNSG)
- 1 個網路安全性群組用於堡壘主機 (MGTNSG)
- 1 個網路安全性群組用於主要和備份網域控制站 (ADNSG)
- 1 個網路安全性群組用於 SQL Server 及雲端見證 (SQLNSG)
- 1 個網路安全性群組用於 Web 層 (WEBNSG)

### <a name="data-in-transit"></a>資料傳輸中
Azure 預設會加密與 Azure 資料中心的所有通訊。 此外，透過 Azure 入口網站與 Azure 儲存體進行的所有交易都會透過 HTTPS 發生。

### <a name="data-at-rest"></a>待用資料

架構會透過加密、資料庫稽核及其他量值來保護待用資料。

**Azure 儲存體**：為符合加密的待用資料需求，所有 [Azure 儲存體](https://azure.microsoft.com/services/storage/)都會使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)。 這有助於保護與防衛資料，以支援組織的安全性承諾及 NHS Digital 所定義的合規性需求。

**Azure 磁碟加密**：[Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)利用 Windows 的 BitLocker 功能來提供資料磁碟的磁碟區加密。 此解決方案與 Azure Key Vault 整合，以協助控制和管理磁碟加密金鑰。

**Azure SQL Database**：Azure SQL Database 執行個體會使用下列資料庫安全性量值：

- [Active Directory 驗證與授權](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)可讓您在一個中央位置管理資料庫使用者及其他 Microsoft 服務的身分識別。
- [SQL 資料庫稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started)會追蹤資料庫事件並將事件寫入 Azure 儲存體帳戶中的稽核記錄。
- Azure SQL Database 設定成使用[透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)，它會執行資料庫、相關備份及交易記錄檔的即時加密和解密，以保護待用資訊。 透明資料加密能保證已儲存的資料不會遭到未經授權的存取。
- [防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)可在授與適當權限之前，防止所有對資料庫伺服器的存取。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。
- [SQL 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-get-started)可針對可疑的資料庫活動、潛在弱點、SQL 插入式攻擊，以及異常的資料庫存取模式，提供安全性警示，藉以在發生潛在威脅時啟用偵測及回應。
- [加密資料行](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)可確保敏感性資料在資料庫系統內一律不會以純文字顯示。 啟用資料加密之後，只有具備金鑰存取權的用戶端應用程式或應用程式伺服器才可以存取純文字資料。
- [SQL Database 動態資料遮罩](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)可藉由遮罩資料，使不具權限的使用者或應用程式無法看見敏感性資料。 動態資料遮罩可以自動探索潛在的敏感性資料，並建議套用適當的遮罩。 這有助於找出並減少資料的存取，使其不會透過未經授權的存取而結束資料庫。 客戶須負責調整動態資料遮罩設定，以配合其資料庫結構描述。

### <a name="identity-management"></a>身分識別管理

下列技術可在 Azure 環境中提供管理資料存取的功能：

- [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) 是 Microsoft 的多租用戶雲端式目錄和身分識別管理服務。 解決方案的所有使用者都是在 Azure Active Directory 中建立，包括存取 Azure SQL Database 的使用者。
- 應用程式的驗證是使用 Azure Active Directory 執行。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。 此外，資料庫資料行加密會使用 Azure Active Directory 向 Azure SQL Database 驗證應用程式。 如需詳細資訊，請參閱如何[保護 Azure SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
- [Azure 角色型存取控制](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure)可讓系統管理員定義微調存取權限，只對使用者授與其執行工作所需的存取權數量。 系統管理員可以只允許存取資料的特定動作，而不是授與所有使用者不受限制的 Azure 資源使用權限。 只有訂用帳戶管理員可擁有訂用帳戶的存取權。
- [Azure Active Directory Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started) 可讓客戶將可存取特定資訊的使用者人數降至最低。 系統管理員可以使用 Azure Active Directory Privileged Identity Management 來探索、限制和監視特殊權限的身分識別和其對資源的存取。 如有需要，這項功能也可用來強制執行隨選 Just-In-Time 系統管理存取權。
- [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測影響組織身分識別的潛在弱點，並為偵測到的組織身分識別相關可疑活動設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。


### <a name="security"></a>安全性

**祕密管理**：解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和祕密。 Azure 金鑰保存庫可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。 下列 Azure Key Vault 功能可協助客戶保護資料及存取這類資料：

- 進階存取原則是視需要設定的。
- Key Vault 存取原則是使用金鑰和祕密的最低必要權限所定義的。
- Key Vault 中的所有金鑰和祕密都有到期日。
- Key Vault 中的所有金鑰都會受到特製化硬體安全模組的保護。 金鑰類型為硬體安全性模組保護的 2048 位元 RSA 金鑰。
- 針對所有使用者與身分識別，皆使用角色型存取控制來授與最低要求權限。
- 金鑰保存庫的診斷記錄的保留期至少 365 天。
- 允許的金鑰密碼編譯作業僅限於需要的密碼編譯項目。
- 此解決方案與 Azure Key Vault 整合，以管理 IaaS 虛擬機器磁碟加密金鑰和祕密。

**修補程式管理**：根據預設，部署為此參考架構一部分的 Windows 虛擬機器會設定為從 Windows Update 服務接收自動更新。 此解決方案也包括 [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)服務，您可以透過此服務建立更新的部署，以在有所需要時修補虛擬機器。

**惡意程式碼防護**：適用於虛擬機器的 [Microsoft 反惡意程式碼軟體](https://docs.microsoft.com/azure/security/azure-security-antimalware)提供即時防護功能，能幫助識別及移除病毒、間諜軟體及其他惡意軟體，具有可設定的警示，可在已知惡意或垃圾軟體嘗試在受保護的虛擬機器上安裝或執行時發出警示。

**Azure 資訊安全中心**：使用 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)，客戶可以在工作負載之間集中套用及管理安全性原則、限制暴露於威脅的程度，以及偵測和回應攻擊。 此外，Azure 資訊安全中心會存取 Azure 服務的現有組態，以提供設定和服務建議，協助改善安全性狀態並保護資料。

Azure 資訊安全中心會使用不同的偵測功能，向客戶警示以其環境為目標的潛在攻擊。 這些警示包含觸發警示的項目、鎖定為目標的資源，以及攻擊來源等重要資訊。 Azure 資訊安全中心有一組[預先定義的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)，一旦發生威脅或可疑活動時便會觸發這些警示。 Azure 資訊安全中心內的[自訂警示規則](https://docs.microsoft.com/azure/security-center/security-center-custom-alert)可讓客戶根據從其環境所收集到的資料，定義新的安全性警示。

Azure 資訊安全中心提供優先的安全性警示和事件，讓客戶更容易探索及解決潛在的安全性問題。 針對每個偵測到的威脅會包含[威脅情報報告](https://docs.microsoft.com/azure/security-center/security-center-threat-report)，以協助事件回應小組調查和修復威脅。

另外，此參考架構會運用 Azure 資訊安全中心的[弱點評量](https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations)。 設定之後，夥伴代理程式 (如 Qualys) 會向夥伴的管理平台回報弱點資料。 接著，夥伴的管理平台會向 Azure 資訊安全中心提供弱點和健康情況監視資料，可讓客戶快速識別易受攻擊的虛擬機器。

**Azure 應用程式閘道**：使用已設定 Web 應用程式防火牆和已啟用 OWASP 規則集的 Azure 應用程式閘道，此架構可減少安全性弱點帶來的風險。 其他功能包括：

- [端對端 SSL](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- 啟用 [SSL 卸載](https://docs.microsoft.com/azure/application-gateway/application-gateway-ssl-portal)
- 停用 [TLS v1.0 和 v1.1](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- [Web 應用程式防火牆](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) (防止模式)
- 使用 OWASP 3.0 規則集的[防止模式](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-portal)
- 啟用[診斷記錄](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)
- [自訂健康情況探查](https://docs.microsoft.com/azure/application-gateway/application-gateway-create-gateway-portal)
- [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center)和 [Azure Advisor](https://docs.microsoft.com/azure/advisor/advisor-security-recommendations) 會提供額外的保護和通知。 Azure 資訊安全中心也會提供評價系統。

### <a name="business-continuity"></a>業務持續性

**高可用性**：解決方案會將所有虛擬機器部署在[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets)中。 可用性設定組可確保虛擬機器會分散到多個各自獨立的硬體叢集中，以提高可用性。 在進行計劃性或非計劃性的維護事件期間，至少有一部虛擬機器可使用，進而符合 99.95% 的 Azure SLA。

**復原服務保存庫**：[復原服務保存庫](https://docs.microsoft.com/azure/backup/backup-azure-recovery-services-vault-overview)可裝載備份資料，並保護此架構中所有 Azure 虛擬機器的組態。 客戶可以使用復原服務保存庫，從 IaaS 虛擬機器還原檔案和資料夾，而非還原整個虛擬機器，這樣可加速還原時間。

**雲端見證**：[雲端見證](https://docs.microsoft.com/windows-server/failover-clustering/whats-new-in-failover-clustering#BKMK_CloudWitness)是 Windows Server 2016 中容錯移轉叢集仲裁見證的一種類型，其運用 Azure 作為仲裁點。 如同其他任何仲裁見證，雲端見證會進行投票，而且可以參與仲裁計算，但它使用公開提供的標準 Azure Blob 儲存體。 這可消除裝載在公用雲端中虛擬機器的額外維修負荷。

### <a name="logging-and-auditing"></a>記錄與稽核

Azure 服務會廣泛記錄系統、使用者活動及系統健康情況：
- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。 活動記錄可協助判斷作業的啟動器、出現時間和狀態。
- **診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)包含每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、Azure 儲存體記錄、Key Vault 稽核記錄，以及應用程式閘道存取和防火牆記錄。 所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存。 保留期是由使用者自訂，視組織特定的保留期需求，最長可達 730 天。

**Log Analytics**：這些記錄會合併到 [Log Analytics](https://azure.microsoft.com/services/log-analytics/) 以進行處理、儲存及從儀表板顯示報告。 所收集的資料會針對每種資料類型組織成個別的資料表，以便一起分析所有的資料 (不論其原始來源為何)。 此外，Azure 資訊安全中心會與 Log Analytics 整合，讓客戶使用 Log Analytics 查詢來存取其安全性事件資料，並將其與來自其他服務的資料合併。

此架構包含下列 Log Analytics [管理解決方案](https://docs.microsoft.com/azure/log-analytics/log-analytics-add-solutions)：
-   [Active Directory 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：Active Directory 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並針對部署的伺服器基礎結構，提供優先的建議清單。
- [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器架構，提供優先的建議清單給客戶。
- [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈，以及沒有回應的代理程式數目和正在提交作業資料的代理程式數目。
-   [活動記錄分析](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：活動記錄分析解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。

**Azure 自動化**：[Azure 自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)會儲存、執行和管理 Runbook。 在此解決方案中，Runbook 會協助從 Azure SQL Database 中收集記錄。 自動化[變更追蹤](https://docs.microsoft.com/azure/automation/automation-change-tracking)解決方案可讓客戶輕鬆地識別環境中的變更。

**Azure 監視器**：[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)藉由讓組織稽核、建立警示及封存資料，包括追蹤使用者 Azure 資源中的 API 呼叫，協助使用者追蹤效能、維護安全性和識別趨勢。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖可供[下載](https://aka.ms/uknhs-iaaswa-tm)，或可以在以下位置找到。 此模型可協助客戶在進行修改時，了解系統基礎結構中的潛在風險要點。

![適用於 UK NHS 的 IaaS Web 應用程式威脅模型](images/uknhs-iaaswa-threat-model.png?raw=true "適用於 UK NHS 的 IaaS Web 應用程式威脅模型")

## <a name="compliance-documentation"></a>合規性文件

[Azure 安全性與合規性藍圖 – UK NHS 客戶責任對照表](https://aka.ms/uknhs-crm)會列出所有 UK NHS 需求。 此對照表詳細說明每個準則的實作是由 Microsoft、客戶還是兩者共同負責。

[Azure 安全性與合規性藍圖 - UK NHS IaaS Web 應用程式實作矩陣](https://aka.ms/uknhs-iaaswa-cim)提供 IaaS Web 應用程式架構處理哪些 UK NHS 需求的相關資訊，包括詳細說明此實作如何符合每個涵蓋準則的需求。

## <a name="guidance-and-recommendations"></a>指引與建議

### <a name="vpn-and-expressroute"></a>VPN 和 ExpressRoute

您需要設定安全的 VPN 通道或 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)，以安全地連線到部署為此 IaaS Web 應用程式參考架構一部分的資源。 透過適當地設定 VPN 或 ExpressRoute，客戶可以在傳輸過程中新增資料保護層。

藉由實作與 Azure 的安全 VPN 通道，即可建立內部部署網路與 Azure 虛擬網路之間的虛擬私人連線。 此連線透過網際網路進行，可讓客戶在客戶的網路與 Azure 之間的加密連結內安全地&quot;輸送&quot;資訊。 站對站 VPN 是安全成熟的技術，各種規模的企業已部署數十年。 此選項使用 [IPsec 通道模式](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2003/cc786385(v=ws.10))作為加密機制。

由於 VPN 通道內的流量會透過站對站 VPN 周遊網際網路，因此 Microsoft 提供另一個更安全的連線選項。 Azure ExpressRoute 是 Azure 與內部部署位置或 Exchange 主機服務提供者之間專用的 WAN 連結。 ExpressRoute 連線不會經過網際網路，相較於一般網際網路連線，這些連線提供了更可靠、更快速、更低延遲、更安全的連線。 此外，因為這是客戶電信提供者的直接連線，所以資料不會透過網際網路傳輸，因此不會公開給網際網路。

如需實作將內部部署網路擴充至 Azure 之安全混合式網路的最佳做法，請參閱[這裡](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid)。

## <a name="disclaimer"></a>免責聲明

- 此文件僅供參考之用。 Microsoft 對本文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 本文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
- 本文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
- 客戶可以複製並使用這份文件，供內部參考之用。
- 本文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
- 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
- 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
