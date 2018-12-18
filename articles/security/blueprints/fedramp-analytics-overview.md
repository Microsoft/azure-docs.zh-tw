---
title: Azure 安全性與合規性藍圖 - 適用於 FedRAMP 的分析
description: Azure 安全性與合規性藍圖 - 適用於 FedRAMP 的分析
services: security
author: jomolesk
ms.assetid: b4675715-668e-4557-9c1c-698aabf62ceb
ms.service: security
ms.topic: article
ms.date: 05/02/2018
ms.author: jomolesk
ms.openlocfilehash: db9e49cc4dc02b6864bee2dc4b73ff3c085f5380
ms.sourcegitcommit: c47ef7899572bf6441627f76eb4c4ac15e487aec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/04/2018
ms.locfileid: "33206362"
---
# <a name="azure-security-and-compliance-blueprint-analytics-for-fedramp"></a>Azure 安全性與合規性藍圖：適用於 FedRAMP 的分析

## <a name="overview"></a>概觀

[聯邦風險與授權管理計畫 (FedRAMP)](https://www.fedramp.gov/) 為美國政府層面的計劃，提供雲端產品與服務的安全評估、授權及持續監視的標準方法。 此 Azure 安全性和合規性藍圖提供如何實行 Microsoft Azure 分析架構的指引，以協助實作 FedRAMP 高階控制項的子集。 此解決方案提供一般參考架構的 Azure 資源部署和組態指引，其中示範的方式讓客戶能符合特定的安全性和合規性需求，並且作為客戶在 Azure 上建置及設定他們自有分析解決方案的基礎。

與控制項實作指南相關聯的此參考架構和威脅模型，主要是用來作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。 未經修改即將應用程式部署至此環境，會無法完全符合 FedRAMP 高階基準的需求。 請注意：
- 架構提供了一個基準，可協助客戶以符合 FedRAMP 規範的方式來將工作負載部署至 Azure。
- 客戶須負責為任何使用架構建置的解決方進行置適當的安全性與合規性評量，因為需求可能會隨每個客戶的實作細節而有所不同。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件

這個解決方案提供分析平台，客戶可以在其中建立自己的分析工具。 此參考架構概要說明了一般使用案例，其中客戶會透過 SQL/資料管理員執行大量資料匯入，或透過操作使用者更新操作資料，來輸入資料。 這兩個工作流程皆需要結合 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview)，才可將資料匯入 SQL Database。 客戶必須透過 Azure 入口網站來設定 Azure Functions，以處理專屬於每個客戶分析需求的匯入工作。

Microsoft Azure 已為客戶提供各種報表和分析服務；但是，此解決方案結合 Azure Analysis Services 與 Azure SQL Database，可透過更聰明的客戶資料模型，來快速瀏覽資料及傳遞更快的結果。 Azure Analytics Services 是機器學習的一種形式，其透過探索資料集之間的新關係，來加快查詢速度。 一旦透過數個統計函數來訓練模型之後，最多可以讓其他 7 個 (若包括客戶伺服器則是總計 8 個) 查詢集區與相同的表格式模型同步，以分散查詢工作負載並縮短回應時間。

若要加強分析和報告，您可以使用資料行存放區索引來設定 SQL 資料庫。 根據客戶的使用情況，Azure Analytics Services 和 SQL Database 可進行相應增加或減少，或甚至是完全關閉。 所有 SQL 流量都會使用 SSL 透過內含的自我簽署憑證來加密。 Azure 建議的最佳做法是使用受信任的憑證授權單位，來增強安全性。

一旦將資料上傳至 Azure SQL Database，並由 Azure Analysis Services 加以訓練後，操作使用者和 SQL/資料管理員就會使用 Power BI 來處理這些資料。 Power BI 會以直覺方式顯示資料，並同時提取多個資料集中的資訊，來繪製更深入的見解。 因為其適應性高且能輕鬆地與 Azure SQL Database 整合，可確保客戶能根據他們的業務需求進行設定，進而處理大量案例。

這整個解決方案都是建立在 Azure 儲存體的基礎上，而客戶會從 Azure 入口網站設定其帳戶。 Azure 儲存體會使用儲存體服務加密來加密所有資料，以維護待用資料的機密性。  異地備援儲存體 (GRS) 可確保發生在客戶主要資料中心內的有害事件不會造成資料遺失，因為系統會將次要複本儲存在數百英里外的個別位置。

為加強安全性，此架構使用 Azure Active Directory 與 Azure Key Vault 來管理資源。 並透過 Operations Management Suite (OMS) 和 Azure 監視器來監視系統健康情況。 客戶可設定這兩項監視服務來擷取記錄，並在易於瀏覽的單一儀表板上顯示系統健康情況。

Azure SQL Database 通常會透過 SQL Server Management Studio (SSMS) 來管理，而執行 SSMS 的本機電腦會設定為透過安全的 VPN 或 ExpressRoute 連線來存取 Azure SQL Database。 **Azure 會建議設定 VPN 或 Azure ExpressRoute 連線，以便管理和將資料匯入參考架構資源群組。**

![適用於 FedRAMP 的分析參考架構圖表](images/fedramp-analytics-reference-architecture.png?raw=true "適用於 FedRAMP 的分析參考架構圖表")

### <a name="roles"></a>角色
分析藍圖會概述三種一般使用者類型的案例：操作使用者、SQL/資料管理員和系統工程師。 Azure 角色型存取控制 (RBAC) 可讓您透過內建自訂角色實作精確的存取管理。 您可以使用資源來設定[角色型存取控制](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure)，以及概述和實作[預先定義的角色](https://docs.microsoft.com/azure/active-directory/role-based-access-built-in-roles)。

#### <a name="systems-engineer"></a>系統工程師
系統工程師擁有 Azure 的客戶訂用帳戶，並會透過 Azure 入口網站設定解決方案的部署。

#### <a name="sqldata-administrator"></a>SQL/資料管理員
SQL/資料管理員會建立大量資料匯入函式和操作資料更新函式，來將資料上傳至 Azure SQL 資料庫。 SQL/資料管理員不負責更新資料庫中的任何操作資料，但是可透過 Power BI 檢視資料。

#### <a name="operational-user"></a>操作使用者
操作使用者會定期更新資料，並擁有每日產生的資料。 操作使用者也會透過 Power BI 解譯結果。

### <a name="azure-services"></a>Azure 服務

此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。
- Azure Functions
- 連接字串
- Azure Analysis Service
- Azure Active Directory
- Azure Key Vault
- OMS
- Azure 監視器
- Azure 儲存體
- ExpressRoute/VPN 閘道
- Power BI 儀表板

## <a name="deployment-architecture"></a>部署架構
下一節會詳細說明開發和實作元素。

![替代文字](images/fedramp-analytics-components.png?raw=true "適用於 FedRAMP 的分析元件圖表")

**Azure Functions**：[Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview) 是可透過大部分程式設計語言，在雲端執行小型程式碼片段的解決方案。 此解決方案中的函式會整合 Azure 儲存體，以自動將客戶資料提取到雲端，藉此加速與其他 Azure 服務的整合。 函式可輕鬆地進行擴充，且只需在執行時支付費用。

**Azure Analysis Service**：[Azure Analysis Service](https://docs.microsoft.com/azure/analysis-services/analysis-services-overview) 可提供企業資料模型並與 Azure 資料平台服務整合。 Azure Analysis Service 可透過將多個來源的資料結合至單一資料模型中，來加快瀏覽大量資料的速度。

**Power BI**：客戶如果想嘗試從其資料處理投入量中取得更好的見解，[Power BI](https://docs.microsoft.com/power-bi/service-azure-and-power-bi) 可為他們提供分析和報表功能。

### <a name="networking"></a>網路
**網路安全性群組**：您可以設定 [NSG](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 來管理在已部署資源和服務上進行導向的流量。 網路安全性群組會設定為使用「預設為拒絕」配置，而且只允許存取控制清單 (ACL) 中預先設定的流量。

每個 NSG 都會開放特定連接埠及通訊協定，讓解決方案可安全且正確地運作。 此外，以下設定會針對每個 NSG 啟用：
  - 啟用[診斷記錄和事件](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)並儲存在儲存體帳戶
  - OMS [Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-networking-analytics) 會連線至 NSG 的診斷記錄。

### <a name="data-at-rest"></a>待用資料
架構會透過加密、資料庫稽核及其他量值來保護待用資料。

**資料複寫** Azure Government 有兩個[資料複寫](https://docs.microsoft.com/azure/storage/common/storage-redundancy)選項：
 - 資料複寫的預設設定是**異地備援儲存體 (GRS)**，以非同步方式將客戶資料儲存在主要區域以外的個別資料中心。 這可確保在主要資料中心裡的資料全部遺失時，能進行資料復原。
 - 您可以透過 Azure 儲存體帳戶交換設定**本地備援儲存體 (LRS)**。 LRS 可複寫儲存體縮放單位中的資料，裝載縮放單位的區域與客戶建立其帳戶的區域相同。 所有資料會同時複寫，以確保沒有備份資料會因為主要儲存體縮放單位失敗而遺失。

**Azure 儲存體**為符合加密待用資料的需求，所有部署在此參考架構中的服務都會利用 [Azure 儲存體](https://azure.microsoft.com/services/storage/)，其使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)來儲存資料。

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

### <a name="logging-and-audit"></a>記錄與稽核
[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-get-started)會將監視資料全部顯示出來，包括活動記錄、計量和診斷資料，讓客戶可以完整地看到系統健康情況。  
[Operations Management Suite (OMS)](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) 提供系統、使用者活動及系統健康情況的廣泛記錄。 OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) 解決方案可收集及分析 Azure 和內部部署環境中資源所產生的資料。
- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。
- **診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)包含每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、Azure Blob 儲存體、資料表和佇列記錄。
- **防火牆記錄**：應用程式閘道會提供完整的診斷和存取記錄。 防火牆記錄可供已啟用 WAF 的應用程式閘道資源使用。
- **記錄封存**：所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存，並包含定義的保留期 (2 天)。 這些記錄會連線至 Azure Log Analytics 以進行處理、儲存及從儀表板顯示報表。

此外，此架構包含下列 OMS 解決方案：
-   [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)：Azure 自動化解決方案會儲存、執行和管理 Runbook。
-   [安全性與稽核](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started)：安全性和稽核儀表板會藉由提供有關安全性網域、值得注意的問題、偵測、威脅情報和常見的安全性查詢等計量，來提供資源安全性狀態的整體見解。
-   [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器架構，提供優先的建議清單給客戶。
-   [Azure 活動記錄](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：活動記錄分析解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。

### <a name="identity-management"></a>身分識別管理
-   應用程式的驗證是使用 Azure AD 執行。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。 此外，資料庫資料行加密會使用 Azure AD 向 Azure SQL Database 驗證應用程式。 如需詳細資訊，請參閱如何[保護 SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
-   [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測影響組織身分識別的潛在弱點，並為偵測到的組織身分識別相關可疑活動設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。
-   [Azure 角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure) 可以對 Azure 進行專注的存取權管理。 只有訂用帳戶管理員可擁有訂用帳戶的存取權。

若要了解使用 Azure SQL Database 安全性功能的詳細資訊，請參閱 [Contoso 診所示範應用程式](https://github.com/Microsoft/azure-sql-security-sample)範例。

### <a name="security"></a>安全性
**祕密管理**：解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和祕密。 Azure Key Vault 可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。

## <a name="guidance-and-recommendations"></a>指引與建議

### <a name="expressroute-and-vpn"></a>ExpressRoute 與 VPN
您需要設定 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或安全的 VPN 通道，以安全地連線到部署為此資料分析參考架構一部分的資源。 ExpressRoute 連線不會經過網際網路，相較於一般網際網路連線，這些連線提供了更可靠、更快速、更低延遲、更安全的連線。 透過適當地設定 ExpressRoute 或 VPN，客戶可以在傳輸過程中新增資料保護層。

### <a name="azure-active-directory-setup"></a>Azure Active Directory 設定
您必須使用 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) 來管理部署，以及為與環境互動的人員佈建存取權。 您只要[點按四下](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-get-started-express)，就可整合現有 Windows Server Active Directory 與 AAD。 客戶也可以透過讓部署的 Active Directory 基礎結構成為 AAD 樹系的子網域，來將部署的 Active Directory 基礎結構 (網域控制站) 繫結至現有 AAD。

### <a name="additional-services"></a>其他服務
#### <a name="iaas---vm-vonsiderations"></a>IaaS - VM 考量
此 PaaS 解決方案不會結合任何 Azure IaaS VM。 客戶可以建立 Azure VM 來執行這幾個 PaaS 服務。 在此案例中，您可利用適用於業務持續性和 OMS 的特定功能與服務：

##### <a name="business-continuity"></a>業務持續性
- **高可用性**：伺服器工作負載會組成[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-manage-availability?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，以協助確保 Azure 中虛擬機器的高可用性。 在進行計劃性或非計劃性的維護事件期間，至少有一部虛擬機器可使用，進而符合 99.95% 的 Azure SLA。

- **復原服務保存庫**：[復原服務保存庫](https://docs.microsoft.com/azure/backup/backup-azure-recovery-services-vault-overview)可裝載備份資料，並保護此架構中所有 Azure 虛擬機器的組態。 客戶可以使用復原服務保存庫，從 IaaS VM 還原檔案和資料夾，而非還原整個 VM，這樣可加速還原時間。

##### <a name="oms"></a>OMS
-   [AD 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：Active Directory 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器基礎結構，提供優先的建議清單。
-   [惡意程式碼評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-malware)：反惡意程式碼軟體解決方案會報告惡意程式碼、威脅及保護狀態。
-   [更新管理](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management)：更新管理解決方案可讓客戶管理作業系統的安全性更新，包括可用更新的狀態和安裝必要更新的程序。
-   [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈，以及沒有回應的代理程式數目和正在提交作業資料的代理程式數目。
-   [變更追蹤](https://docs.microsoft.com/azure/automation/automation-change-tracking)：變更追蹤解決方案可讓客戶輕鬆地識別環境中的變更。

##### <a name="security"></a>安全性
- **惡意程式碼防護**：適用於虛擬機器的 [Microsoft 反惡意程式碼軟體](https://docs.microsoft.com/azure/security/azure-security-antimalware)提供即時防護功能，能幫助識別及移除病毒、間諜軟體及其他惡意軟體，具有可設定的警示，可在已知惡意或垃圾軟體嘗試在受保護的虛擬機器上安裝或執行時發出警示。
- **修補程式管理**：根據預設，部署為此參考架構一部份的 Windows 虛擬機器會設定為從 Windows Update 服務接收自動更新。 此解決方案也包括 OMS [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)服務，您可以透過此服務建立更新的部署，以在有所需要時修補虛擬機器。

#### <a name="azure-commercial"></a>Azure Commercial
雖然此資料分析架構並非用於 [Azure Commercial](https://azure.microsoft.com/overview/what-is-azure/) 環境的部署，但您可以透過此參考架構中所述的服務，以及其他只能在 Azure Commercial 環境中使用的服務來達成類似目標。 請注意，Azure Commercial 會將 FedRAMP JAB P-ATO 維持在中度影響等級，如此可讓政府機關及合作夥伴將中度敏感性資訊部署到雲端，以充分運用 Azure Commercial 環境。

Azure Commercial 提供各種不同的分析服務，讓您可以從大量資料中取得見解：
-   [Azure Machine Learning Studio](https://docs.microsoft.com/azure/machine-learning/studio/what-is-ml-studio) 是 [Cortana Intelligence Suite](https://azure.microsoft.com/overview/ai-platform/) 的元件，用於從一個或多個資料來源開發預測分析模型。 統計函數會用於多個反覆項目，進而產生有效模型，然後讓 Power BI 這類應用程式可以使用。
-   [Azure Data Lake Store](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview) 可讓您在單一位置擷取任何大小、類型和擷取速度的資料，以進行運作和探究分析。 Azure Data Lake Store 可與 Hadoop 生態系統中大部分的開放原始碼元件相容，並可與其他 Azure 服務順利整合。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖 (DFD) 可供[下載](https://aka.ms/blueprintanalyticsthreatmodel)，或可以在以下位置找取得：

## <a name="compliance-documentation"></a>合規性文件
[Azure 安全性與合規性藍圖 – FedRAMP 高階客戶責任矩陣](https://aka.ms/blueprinthighcrm)會列出 FedRAMP 高階基準所需的所有安全性控制項。 同樣地，[Azure 安全性與合規性藍圖 – FedRAMP 一般客戶責任矩陣](https://aka.ms/blueprintanalyticscimmod)會列出 FedRAMP 一般基準所需的所有安全性控制項。 這兩個文件會詳細說明每個控制項的實作是由 Microsoft、客戶或兩者共同負責。

[Azure 安全性與合規性藍圖 - FedRAMP 高階控制項實作矩陣](https://aka.ms/blueprintanalyticscimhigh)和 [Azure 安全性與合規性藍圖 - FedRAMP 一般控制項實作矩陣](https://aka.ms/blueprintanalyticscimmod)提供的資訊會針對每個 FedRAMP 基準，說明資料分析架構涵蓋哪些控制項，包括說明如何讓實作符合每個所涵蓋控制項的需求。

## <a name="disclaimer"></a>免責聲明

 - 此文件僅供參考之用。 Microsoft 對本文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 本文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
 - 本文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
 - 客戶可以複製並使用這份文件，供內部參考之用。
 - 本文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
 - 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
 - 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
