---
title: Azure 安全性與合規性藍圖 - 適用於 FedRAMP 自動化的資料倉儲
description: Azure 安全性與合規性藍圖 - 適用於 FedRAMP 自動化的資料倉儲
services: security
author: jomolesk
ms.assetid: 834d1ff6-8369-455f-b052-1ef301e3d7e6
ms.service: security
ms.topic: article
ms.date: 05/02/2018
ms.author: jomolesk
ms.openlocfilehash: 110ce131286f437e051dd859eb4d2baa29f106f6
ms.sourcegitcommit: c47ef7899572bf6441627f76eb4c4ac15e487aec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/04/2018
ms.locfileid: "33206355"
---
# <a name="azure-security-and-compliance-blueprint-data-warehouse-for-fedramp-automation"></a>Azure 安全性與合規性藍圖：適用於 FedRAMP 自動化的資料倉儲

## <a name="overview"></a>概觀

[聯邦風險與授權管理計畫 (FedRAMP)](https://www.fedramp.gov/) 為美國政府層面的計劃，提供雲端產品與服務的安全評估、授權及持續監視的標準方法。 此 Azure 安全性和合規性藍圖提供如何實行 Microsoft Azure 資料倉儲架構的指引，以協助實作 FedRAMP 高階控制項的子集。 此解決方案提供一般參考架構的 Azure 資源部署和組態指引，其中示範的方式讓客戶能符合特定的安全性和合規性需求，並且作為客戶在 Azure 上建置及設定他們自有資料倉儲解決方案的基礎。

與控制項實作指南相關聯的此參考架構和威脅模型，主要是用來作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。 未經修改即將應用程式部署至此環境，會無法完全符合 FedRAMP 高階基準的需求。 請注意：
- 架構提供了一個基準，可協助客戶以符合 FedRAMP 規範的方式來將工作負載部署至 Azure。
- 客戶須負責為任何使用架構建置的解決方進行置適當的安全性與合規性評量，因為需求可能會隨每個客戶的實作細節而有所不同。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件

此解決方案提供資料倉儲參考架構，可實作高效能且安全的雲端資料倉儲。 此架構中有兩個不同的資料層：一個層級可用於將資料匯入、儲存，以及暫存至叢集的 SQL 環境中，而另一個層級用於 Azure SQL 資料倉儲，其中使用 ETL 工具載入資料 (例如[PolyBase](https://docs.microsoft.com/azure/sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase) T-SQL 查詢) 以進行處理。 一旦資料儲存在 Azure SQL 資料倉儲，您就可以大規模執行分析。

Microsoft Azure 為客戶提供各種報告和分析服務。 此解決方案包括可從 Azure SQL 資料倉儲快速建立報告的 SQL Server Reporting Services (SSRS)。 所有 SQL 流量都會使用 SSL 透過內含的自我簽署憑證來加密。 Azure 建議的最佳做法是使用受信任的憑證授權單位，來增強安全性。

Azure SQL 資料倉儲中的資料會以單欄式儲存方式，儲存在關聯式資料表中，單欄式儲存是可大幅減少資料儲存體成本，又可提高查詢效能的格式。  根據使用需求，Azure SQL 資料倉儲的計算資源可以相應增加或減少，或在沒有使用中程序需要計算資源時完全關閉。

SQL 負載平衡器會管理 SQL 流量，確保維持高效能。 為具備高可用性和災害復原的能力，此參考架構中的所有虛擬機器都部署在可用性設定組中，並搭配使用設定在 AlwaysOn 可用性群組中的 SQL Server 執行個體。

此資料倉儲參考架構也包含用來管理架構中資源的 Active Directory (AD) 層。 Active Directory 子網路可讓您更願意採用較大型 AD 樹系結構，因為即使無法存取較大樹系，也可讓該環境的作業繼續執行。 所有虛擬機器都已透過網域加入 Active Directory 層，而且已使用 Active Directory 群組原則來強化作業系統層級的安全性與合規性設定。

作為管理防禦主機使用的虛擬機器，會提供可讓系統管理員存取部署資源的安全連線。 資料會透過此管理防禦主機載入暫存區域。 **Azure 會建議設定 VPN 或 Azure ExpressRoute 連線，以便管理和將資料匯入參考架構子網路。**

![FedRAMP 的資料倉儲參考架構圖表](images/fedramp-datawarehouse-architecture.png?raw=true "FedRAMP 的資料倉儲參考架構圖表")

此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。

Azure 虛擬機器
-   (1) 防禦主機
-   (2) Active Directory 網域控制站
-   (2) SQL Server 叢集節點
-   (1) SQL Server 見證

可用性設定組 (Availability Sets)
-   (1) Active Directory 網域控制站
-   (1) SQL 叢集節點和見證

虛擬網路
-   (4) 子網路
-   (4) 網路安全性群組

SQL 資料倉儲

SQL Server Reporting Services

Azure SQL Load Balancer

Azure Active Directory

復原服務保存庫

Azure Key Vault

Operations Management Suite (OMS)

## <a name="deployment-architecture"></a>部署架構

下一節會詳細說明開發和實作元素。

**SQL Data Warehouse**：[SQL 資料倉儲](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)是企業資料倉儲 (EDW)，利用大量平行處理 (MPP) 以快速地在數以 PB 計的資料中執行複雜的查詢。 使用簡單的 PolyBase T-SQL 查詢將巨量資料匯入 SQL 資料倉儲，然後使用 MPP 功能來執行高效能分析。

**SQL Server Reporting Services**：[SQL Server Reporting Services](https://docs.microsoft.com/sql/reporting-services/report-data/sql-azure-connection-type-ssrs) 可快速地以資料表、圖表、地圖、量測計、矩陣等形式，為 Azure SQL 資料倉儲建立報表。

**防禦主機**：防禦主機是單一進入點，可讓使用者存取此環境中已部署的資源。 防禦主機為已部署的資源提供安全連線，其允許的遠端流量僅限來自安全清單上的公用 IP 位址。 若要允許遠端桌面 (RDP) 流量，必須在網路安全性群組 (NSG) 中定義該流量的來源。

虛擬機器已透過下列設定建立為加入網域的防禦主機：
-   [反惡意程式碼軟體擴充功能](https://docs.microsoft.com/azure/security/azure-security-antimalware)
-   [OMS 擴充功能](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-extensions-oms)
-   [Azure 診斷擴充功能](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-extensions-diagnostics-template)
-   使用 Azure Key Vault 的 [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) (遵循 Azure Government、PCI DSS 和 HIPAA 等其他需求)
-   [自動關閉原則](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/) \(英文\)，可減少不使用虛擬機器資源時的耗用量。
-   [Windows Defender Credential Guard](https://docs.microsoft.com/windows/access-protection/credential-guard/credential-guard) 已啟用，因此認證和其他秘密都會在受保護的環境中執行，且該環境會與正在執行的作業系統隔離

### <a name="virtual-network"></a>虛擬網路
此參考架構會定義位址空間為 10.0.0.0/16 的私人虛擬網路。

**網路安全性群組**：[NSG](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 包含能允許或拒絕 VNet 內流量的存取控制清單 (ACL)。 NSG 可以用來保護子網路或個別 VM 層級的流量。 下列 NSG 已經存在:
  - 適用於資料層的 NSG (SQL Server 叢集、SQL Server 見證和 SQL Load Balancer)
  - 適用於管理防禦主機的 NSG
  - 適用於 Active Directory 的 NSG
  - 適用於 SQL Server Reporting Services 的 NSG

每個 NSG 都會開放特定連接埠及通訊協定，讓解決方案可安全且正確地運作。 此外，以下設定會針對每個 NSG 啟用：
  - 啟用[診斷記錄和事件](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)並儲存在儲存體帳戶
  - OMS Log Analytics 會連線至 [NSG 的診斷](https://github.com/krnese/AzureDeploy/blob/master/AzureMgmt/AzureMonitor/nsgWithDiagnostics.json)

**子網路**：確認每個子網路都與對應的 NSG 相關聯。

### <a name="data-at-rest"></a>待用資料
架構會透過加密、資料庫稽核及其他量值來保護待用資料。

**Azure 儲存體**為符合加密的待用資料需求，所有 [Azure 儲存體](https://azure.microsoft.com/services/storage/)都會使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)。

**Azure 磁碟加密**
[Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)利用 Windows 的 BitLocker 功能來提供 OS 和資料磁碟的磁碟區加密。 此解決方案與 Azure Key Vault 整合，以協助控制和管理磁碟加密金鑰。

**Azure SQL Database** Azure SQL Database 執行個體會使用下列資料庫安全性量值：
-   [AD 驗證與授權](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)可讓您在一個中央位置管理資料庫使用者及其他 Microsoft 服務的身分識別。
-   [SQL 資料庫稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started)會追蹤資料庫事件並將事件寫入 Azure 儲存體帳戶中的稽核記錄。
-   SQL Database 設定成使用[透明資料加密 (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)，它會執行即時的資料和記錄檔的加密和解密，以保護待用資訊。 TDE 能保證已儲存的資料不會遭到未經授權的存取。
-   [防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)可在授與適當權限之前，防止所有對資料庫伺服器的存取。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。
-   [SQL 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-get-started)可針對可疑的資料庫活動、潛在弱點、SQL 插入式攻擊，以及異常的資料庫存取模式，提供安全性警示，藉以在發生潛在威脅時啟用偵測及回應。
-   [Always Encrypted 資料行](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)可確保敏感性資料在資料庫系統內一律不會以純文字顯示。 啟用資料加密之後，只有具備金鑰存取權的用戶端應用程式或應用程式伺服器才可以存取純文字資料。
-   [SQL Database 動態資料遮罩](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)可以在參考架構部署好之後執行。 客戶必須調整動態資料遮罩設定，以配合其資料庫結構描述。

### <a name="business-continuity"></a>業務持續性
**高可用性**：伺服器工作負載會組成[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-manage-availability?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，以協助確保 Azure 中虛擬機器的高可用性。 在進行計劃性或非計劃性的維護事件期間，至少有一部虛擬機器可使用，進而符合 99.95% 的 Azure SLA。

**復原服務保存庫**：[復原服務保存庫](https://docs.microsoft.com/azure/backup/backup-azure-recovery-services-vault-overview)可裝載備份資料，並保護此架構中所有 Azure 虛擬機器的組態。 客戶可以使用復原服務保存庫，從 IaaS VM 還原檔案和資料夾，而非還原整個 VM，這樣可加速還原時間。

### <a name="logging-and-audit"></a>記錄與稽核
[Operations Management Suite (OMS)](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) 提供系統、使用者活動及系統健康情況的廣泛記錄。 OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) 解決方案可收集及分析 Azure 和內部部署環境中資源所產生的資料。
- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。
- **診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)包含每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、Azure Blob 儲存體、資料表和佇列記錄。
- **防火牆記錄**：應用程式閘道會提供完整的診斷和存取記錄。 防火牆記錄可供已啟用 WAF 的應用程式閘道資源使用。
- **記錄封存**：所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存，並包含定義的保留期 (2 天)。 這些記錄會連線至 Azure Log Analytics 以進行處理、儲存及從儀表板顯示報表。

此外，此架構包含下列 OMS 解決方案：
-   [AD 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：Active Directory 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器基礎結構，提供優先的建議清單。
-   [惡意程式碼評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-malware)：反惡意程式碼軟體解決方案會報告惡意程式碼、威脅及保護狀態。
-   [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)：Azure 自動化解決方案會儲存、執行和管理 Runbook。
-   [安全性與稽核](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started)：安全性和稽核儀表板會藉由提供有關安全性網域、值得注意的問題、偵測、威脅情報和常見的安全性查詢等計量，來提供資源安全性狀態的整體見解。
-   [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器架構，提供優先的建議清單給客戶。
-   [更新管理](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management)：更新管理解決方案可讓客戶管理作業系統的安全性更新，包括可用更新的狀態和安裝必要更新的程序。
-   [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈，以及沒有回應的代理程式數目和正在提交作業資料的代理程式數目。
-   [Azure 活動記錄](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：活動記錄分析解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。
-   [變更追蹤](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：變更追蹤解決方案可讓客戶輕鬆地識別環境中的變更。

### <a name="identity-management"></a>身分識別管理
下列技術可在 Azure 環境中提供身分識別管理功能：
-   [Active Directory (AD)](https://azure.microsoft.com/services/active-directory/) 可以是 Microsoft 的多租用戶雲端式目錄和身分識別管理服務。 解決方案的所有使用者都是在 Azure Active Directory 中建立，包括存取 SQL Database 的使用者。
-   應用程式的驗證是使用 Azure AD 執行。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。 此外，資料庫資料行加密會使用 Azure AD 向 Azure SQL Database 驗證應用程式。 如需詳細資訊，請參閱如何[保護 SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
-   [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測影響組織身分識別的潛在弱點，並為偵測到的組織身分識別相關可疑活動設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。
-   [Azure 角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure) 可以對 Azure 進行專注的存取權管理。 只有訂用帳戶管理員可擁有訂用帳戶的存取權。

若要了解使用 Azure SQL Database 安全性功能的詳細資訊，請參閱 [Contoso 診所示範應用程式](https://github.com/Microsoft/azure-sql-security-sample)範例。

### <a name="security"></a>安全性
**祕密管理**：解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和祕密。 Azure Key Vault 可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。

**惡意程式碼防護**：適用於虛擬機器的 [Microsoft 反惡意程式碼軟體](https://docs.microsoft.com/azure/security/azure-security-antimalware)提供即時防護功能，能幫助識別及移除病毒、間諜軟體及其他惡意軟體，具有可設定的警示，可在已知惡意或垃圾軟體嘗試在受保護的虛擬機器上安裝或執行時發出警示。

**修補程式管理**：根據預設，部署為此參考架構一部份的 Windows 虛擬機器會設定為從 Windows Update 服務接收自動更新。 此解決方案也包括 OMS [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)服務，您可以透過此服務建立更新的部署，以在有所需要時修補虛擬機器。


## <a name="guidance-and-recommendations"></a>指引與建議
### <a name="expressroute-and-vpn"></a>ExpressRoute 與 VPN
您需要設定 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或安全的 VPN 通道，以安全地連線到部署為此資料倉儲參考架構一部份的資源。 ExpressRoute 連線不會經過網際網路，相較於一般網際網路連線，這些連線提供了更可靠、更快速、更低延遲、更安全的連線。 透過適當地設定 ExpressRoute 或 VPN，客戶可以在傳輸過程中新增資料保護層。

### <a name="extract-transform-load-etl-process"></a>擷取、轉換和下載 (ETL) 程序
[PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) 可以將資料載入 Azure SQL 資料倉儲中，而不需要個別的 ETL 或匯入工具。 PolyBase 允許透過 T-SQL 查詢來存取資料。 Microsoft 的商業智慧和分析堆疊，以及與 SQL Server 相容的第三方工具，都可以搭配 PolyBase 使用。

### <a name="azure-active-directory-setup"></a>Azure Active Directory 設定
您必須使用 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) 來管理部署，以及為與環境互動的人員佈建存取權。 您只要[點按四下](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-get-started-express)，就可整合現有 Windows Server Active Directory 與 AAD。 客戶也可以透過讓部署的 Active Directory 基礎結構成為 AAD 樹系的子網域，來將部署的 Active Directory 基礎結構 (網域控制站) 繫結至現有 AAD。

### <a name="additional-services"></a>其他服務
雖然此資料倉儲架構並非用於 [Azure Commercial](https://azure.microsoft.com/overview/what-is-azure/) 環境的部署，但您可以透過此參考架構中所述的服務，以及其他只能在 Azure Commercial 環境中使用的服務來達成類似目標。 請注意，Azure Commercial 會將 FedRAMP JAB P-ATO 維持在中度影響等級，如此可讓政府機關及合作夥伴將中度敏感性資訊部署到雲端，以充分運用 Azure Commercial 環境。

Azure Commercial 提供各種服務來處理格式化和未格式化的資料儲存與暫存，好讓資料可用於資料倉儲，這些服務包括：
-   [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction) 是一個針對複雜的混合式「擷取、轉換和下載」(ETL)、「擷取、下載和轉換」(ELT) 及資料整合專案建立的受控雲端服務。 客戶可以使用 Azure Data Factory 建立並排程資料驅動的工作流程 (稱為管線)，以從不同的資料存放區擷取資料。 接著，客戶可以處理並轉換輸出的資料，並儲存至資料存放區，例如 Azure SQL 資料倉儲。
-   [Azure Data Lake Store](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview) 可讓您在單一位置擷取任何大小、類型和擷取速度的資料，以進行運作和探究分析。 Azure Data Lake Store 可與 Hadoop 生態系統中大部分的開放原始碼元件相容，並可與其他 Azure 服務 (例如 Azure SQL 資料倉儲) 順利整合。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖 (DFD) 可供[下載](https://aka.ms/blueprintdwthreatmodel)，或可以在以下位置找取得：

![適用於 FedRAMP 威脅模型的資料倉儲](images/fedramp-datawarehouse-threat-model.png?raw=true "適用於 FedRAMP 威脅模型的資料倉儲")

## <a name="compliance-documentation"></a>合規性文件
[Azure 安全性與合規性藍圖 – FedRAMP 高階客戶責任矩陣](https://aka.ms/blueprinthighcrm)會列出 FedRAMP 高階基準所需的所有安全性控制項。 同樣地，[Azure 安全性與合規性藍圖 – FedRAMP 一般客戶責任矩陣](https://aka.ms/blueprintcrmmod)會列出 FedRAMP 一般基準所需的所有安全性控制項。 這兩個文件會詳細說明每個控制項的實作是由 Microsoft、客戶或兩者共同負責。

[Azure 安全性與合規性藍圖 - FedRAMP 高階控制項實作矩陣](https://aka.ms/blueprintdwcimhigh)和 [Azure 安全性與合規性藍圖 - FedRAMP 一般控制項實作矩陣](https://aka.ms/blueprintdwcimmod)提供的資訊會針對每個 FedRAMP 基準，說明資料倉儲架構涵蓋哪些控制項，包括說明如何讓實作符合每個所涵蓋控制項的需求。

## <a name="disclaimer"></a>免責聲明

 - 此文件僅供參考之用。 Microsoft 對本文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 本文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
 - 本文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
 - 客戶可以複製並使用這份文件，供內部參考之用。
 - 本文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
 - 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
 - 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
