---
title: Azure 安全性與合規性藍圖 - 適用於 FedRAMP 的 IaaS Web 應用程式
description: Azure 安全性與合規性藍圖 - 適用於 FedRAMP 的 IaaS Web 應用程式
services: security
documentationcenter: na
author: jomolesk
manager: mbaldwin
editor: tomsh
ms.assetid: 8fe47cff-9c24-49e0-aa11-06ff9892a468
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/08/2018
ms.author: jomolesk
ms.openlocfilehash: 6a2a72f46c4d5faacb7d5871f4c917a5cd578e96
ms.sourcegitcommit: 6cf20e87414dedd0d4f0ae644696151e728633b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/06/2018
ms.locfileid: "34809160"
---
# <a name="azure-security-and-compliance-blueprint-iaas-web-application-for-fedramp"></a>Azure 安全性與合規性藍圖：適用於 FedRAMP 的 IaaS Web 應用程式

## <a name="overview"></a>概觀

[聯邦風險與授權管理計畫 (FedRAMP)](https://www.fedramp.gov) 為美國政府層面的計劃，提供雲端產品與服務的安全評估、授權及持續監視的標準方法。 此「Azure 安全性與合規性藍圖自動化」所提供的指引可用來部署符合 FedRAMP 規範的基礎結構即服務 (IaaS) 環境，且此環境適合網際網路面向的簡易 Web 應用程式。 此解決方案將一般參考架構的 Azure 資源部署和設定自動化，其中示範的方式讓客戶能符合特定的安全性和合規性需求，並且作為客戶在 Azure 上建置及設定他們自己的解決方案之基礎。 此解決方案會根據 NIST SP 800-53 實作 FedRAMP 高標準基準的控制措施子集。 如需 FedRAMP 需求和此解決方案的詳細資訊，請參閱[合規性文件](#compliance-documentation)。
> [!NOTE]
> 此解決方案會部署到 Azure Government。

此「Azure 安全性與合規性藍圖自動化」會使用預先設定的安全性措施，自動部署 IaaS Web 應用程式參考架構，以協助客戶符合 FedRAMP 需求的規範。 解決方案包含引導資源部署與設定的 Azure Resource Manager 範本和 PowerShell 指令碼。

此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。 未經修改即將應用程式部署至此環境，會無法完全符合 FedRAMP 高標準基準的需求。 請注意：
- 此架構提供了一個基準，可協助客戶以符合 FedRAMP 規範的方式來使用 Azure。
- 客戶須負責為任何使用架構建置的解決方進行置適當的安全性與合規性評量，因為需求可能會隨每個客戶的實作細節而有所不同。

如需此解決方案運作方式簡短概觀，請觀賞說明及示範其部署的此[影片](https://aka.ms/fedrampblueprintvideo) \(英文\)。

如需部署指示，請按一下[這裡](https://aka.ms/fedrampblueprintrepo) \(英文\)。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件
此解決方案會部署具有 SQL Server 後端之 IaaS Web 應用程式的參考架構。 架構包含 Web 層、資料層、Active Directory 基礎結構、應用程式閘道及負載平衡器。 部署到 Web 層和資料層的虛擬機器是設定在可用性設定組中，而 SQL Server 是設定在適用於高可用性的 AlwaysOn 可用性群組中。 虛擬機器已加入網域，而且使用 Active Directory 群組原則來強化作業系統層級的安全性與合規性設定。 防禦主機提供可讓系統管理員存取部署資源的安全連線。 **Azure 會建議設定 VPN 或 Azure ExpressRoute 連線，以便管理和將資料匯入參考架構子網路。**

![適用於 FedRAMP 的 IaaS Web 應用程式參考架構圖表](images/fedramp-iaaswa-architecture.png?raw=true "適用於 FedRAMP 的 IaaS Web 應用程式參考架構圖表")

此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。

- Azure 虛擬機器
    - (1) 防禦主機 (Windows Server 2016 Datacenter)
    - (2) Active Directory 網域控制站 (Windows Server 2016 Datacenter)
    - (2) SQL Server 叢集節點 (Windows Server 2016 上的 SQL Server 2017)
    - (2) Web/IIS (Windows Server 2016 Datacenter)
- 可用性設定組 (Availability Sets)
    - (1) Active Directory 網域控制站
    - (1) SQL 叢集節點
    - (1) Web/IIS
- Azure 虛擬網路
    - (1) /16 虛擬網路
    - (5) /24 子網路
    - DNS 設定設為這兩個網域控制站
- Azure Load Balancer
- Azure 應用程式閘道
    - (1) WAF 應用程式閘道已啟用
        - 防火牆模式：防止
        - 規則集：OWASP 3.0
        - 接聽程式：連接埠 443
- Azure 儲存體
    - (7) 異地備援儲存體帳戶
- Azure 雲端見證
- 復原服務保存庫
- Azure 金鑰保存庫
- Azure Active Directory (Azure AD)
- Azure Resource Manager
- Operations Management Suite (OMS)
- Azure 監視器

## <a name="deployment-architecture"></a>部署架構

下一節會詳細說明開發和實作元素。

**防禦主機**：防禦主機是一個單一進入點，提供可讓系統管理員存取部署資源的安全連線。 防禦主機的 NSG 僅允許在 TCP 連接埠 3389 上執行 RDP 連線。 客戶可以進一步設定防禦主機，以符合組織系統的強化需求。

### <a name="virtual-network"></a>虛擬網路
此架構會定義位址空間為 10.200.0.0/16 的私人虛擬網路。

**網路安全性群組**：此解決方案在虛擬網路內，將資源部署在具有個別的 Web 子網路、資料庫子網路、Active Directory 子網路及管理子網路的架構中。 套用至個別子網路的網路安全性群組規則以邏輯方式區隔了子網路，將子網路之間的流量限制為系統和管理功能所需的流量。

請參閱使用此解決方案部署之[網路安全性群組](https://github.com/Azure/fedramp-iaas-webapp/blob/master/nestedtemplates/virtualNetworkNSG.json)的設定。 客戶可以使用[此文件](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)當作指南來編輯上述檔案，以設定網路安全性群組。

每個子網路都有專用的網路安全性群組 (NSG)：
- 1 個 NSG 用於應用程式閘道 (LBNSG)
- 1 個 NSG 用於防禦主機 (MGTNSG)
- 1 個 NSG 用於主要和備份網域控制站 (ADNSG)
- 1 個 NSG 用於 SQL Server (SQLNSG)
- 1 個 NSG 用於 Web 層 (WEBNSG)

**子網路**：確認每個子網路都與對應的 NSG 建立關聯。

### <a name="data-at-rest"></a>待用資料

此架構使用數個加密措施來保護待用資料。

**Azure 儲存體**：為符合待用資料的加密需求，所有儲存體都會使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)。

**SQL Server**：SQL Server 設定成使用[透明資料加密 (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption)，它會執行即時的資料和記錄檔的加密和解密，以保護待用資料。 TDE 能保證已儲存的資料不會遭到未經授權的存取。

客戶也可設定下列 SQL Server 安全性量值：
-   [AD 驗證與授權](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)可讓您在一個中央位置管理資料庫使用者及其他 Microsoft 服務的身分識別。
-   [SQL 資料庫稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started)會追蹤資料庫事件並將事件寫入 Azure 儲存體帳戶中的稽核記錄。
-   [防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)可在授與適當權限之前，防止所有對資料庫伺服器的存取。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。
-   [SQL 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-get-started)可針對可疑的資料庫活動、潛在弱點、SQL 插入式攻擊，以及異常的資料庫存取模式，提供安全性警示，藉以在發生潛在威脅時啟用偵測及回應。
-   [Always Encrypted 資料行](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)可確保敏感性資料在資料庫系統內一律不會以純文字顯示。 啟用資料加密之後，只有具備金鑰存取權的用戶端應用程式或應用程式伺服器才可以存取純文字資料。
-   [SQL Database 動態資料遮罩](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)可以在參考架構部署好之後執行。 客戶必須調整動態資料遮罩設定，以配合其資料庫結構描述。

**Azure 磁碟加密**：Azure 磁碟加密可用來加密 Windows IaaS 虛擬機器磁碟。 [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)利用 Windows 的 BitLocker 功能來提供 OS 和資料磁碟的磁碟區加密。 此解決方案與 Azure Key Vault 整合，以協助控制和管理磁碟加密金鑰。

### <a name="identity-management"></a>身分識別管理

下列技術可在 Azure 環境中提供身分識別管理功能：
- [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/) 是 Microsoft 的多租用戶雲端式目錄和身分識別管理服務。
- 可以使用 Azure AD 執行對客戶部署之 Web 應用程式的驗證。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。  
- [Azure 角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal) 可以對 Azure 進行精確且專注的存取權管理。 訂用帳戶存取限於訂用帳戶系統管理員，而對資源的存取可根據使用者角色加以限制。
- 部署的 IaaS Active Directory 執行個體提供適用於部署 IaaS 虛擬機器之 OS 層級的身分識別管理。

### <a name="security"></a>安全性
**祕密管理**：解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和祕密。 Azure 金鑰保存庫可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。 Azure Key Vault 有助於管理此參考架構的 IaaS 虛擬機器磁碟加密金鑰和密碼。

**修補程式管理**：根據預設，此「Azure 安全性與合規性藍圖自動化」所部署的 Windows 虛擬機器會設定為從 Windows Update 服務接收自動更新。 這個解決方案也會部署 Azure 自動化解決方案，可透過此解決方案來建立更新部署，以在需要時將修補程式部署到 Windows 伺服器。

**惡意程式碼防護**：適用於虛擬機器的 [Microsoft 反惡意程式碼軟體](https://docs.microsoft.com/azure/security/azure-security-antimalware)提供即時防護功能，能幫助識別及移除病毒、間諜軟體及其他惡意軟體，具有可設定的警示，可在已知惡意或垃圾軟體嘗試在受保護的虛擬機器上安裝或執行時發出警示。

**應用程式閘道**：透過使用已啟用 Web 應用程式防火牆 (WAF) 和 OWASP 規則集的應用程式閘道，此架構可減少安全性弱點帶來的風險。 其他功能包括：

- [端對端 SSL](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- 啟用 [SSL 卸載](https://docs.microsoft.com/azure/application-gateway/application-gateway-ssl-portal)
- 停用 [TLS v1.0 和 v1.1](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- [Web 應用程式防火牆](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) \(WAF 模式\)
- 使用 OWASP 3.0 規則集的[防止模式](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-portal)

### <a name="business-continuity"></a>業務持續性

**高可用性**：在進行計劃性或非計劃性的維護事件期間，至少有一部虛擬機器可使用，進而符合 99.95% 的 Azure SLA。 此解決方案將所有 Web 層和資料層虛擬機器都部署在[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets)中。 可用性設定組可確保虛擬機器會分散到多個各自獨立的硬體叢集中，以提高可用性。 此外，此解決方案將 SQL Server 虛擬機器部署在可用性設定組中當作 [AlwaysOn 可用性群組](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-overview)。 Always On 可用性群組功能提供高可用性與災害復原功能。

**復原服務保存庫**：[復原服務保存庫](https://docs.microsoft.com/azure/backup/backup-azure-recovery-services-vault-overview)可裝載備份資料，並保護此架構中所有 Azure 虛擬機器的組態。 客戶可以使用復原服務保存庫，從 IaaS VM 還原檔案和資料夾，而非還原整個 VM，這樣可加速還原時間。

**雲端見證**：[雲端見證](https://docs.microsoft.com/windows-server/failover-clustering/whats-new-in-failover-clustering#BKMK_CloudWitness)是 Windows Server 2016 中容錯移轉叢集仲裁見證的一種類型，其運用 Azure 作為仲裁點。 如同其他任何仲裁見證，雲端見證會進行投票，而且可以參與仲裁計算，但它使用公開提供的標準 Azure Blob 儲存體。 這可消除裝載在公用雲端中 VM 的額外維修負荷。

### <a name="logging-and-auditing"></a>記錄與稽核

OMS 提供系統、使用者活動及系統健康情況的廣泛記錄。 OMS [Log Analytics](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) 解決方案可收集及分析 Azure 和內部部署環境中資源所產生的資料。

- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。 活動記錄可協助判斷作業的啟動者、發生時間和狀態。
- **診斷記錄：**[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)是每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、Azure 儲存體記錄、Key Vault 稽核記錄，以及應用程式閘道存取和防火牆記錄。
- **記錄封存**：所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存。 保留期是由使用者自訂，視組織特定的保留期需求，最長可達 730 天。 這些記錄會連線至 Azure Log Analytics 以進行處理、儲存及從儀表板顯示報告。

此外，此架構中會安裝下列 OMS 解決方案。 請注意，客戶須負責設定這些方案，以配合 FedRAMP 安全性控制項：
-   [AD 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：Active Directory 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器基礎結構，提供優先的建議清單。
-   [惡意程式碼評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-malware)：反惡意程式碼軟體解決方案會報告惡意程式碼、威脅及保護狀態。
-   [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)：Azure 自動化解決方案會儲存、執行和管理 Runbook。
-   [安全性與稽核](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started)：安全性和稽核儀表板會藉由提供有關安全性網域、值得注意的問題、偵測、威脅情報和常見的安全性查詢等計量，來提供資源安全性狀態的整體見解。
-   [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器架構，提供優先的建議清單給客戶。
-   [更新管理](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management)：更新管理解決方案可讓客戶管理作業系統的安全性更新，包括可用更新的狀態和安裝必要更新的程序。
-   [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈，以及沒有回應的代理程式數目和正在提交作業資料的代理程式數目。
-   [Azure 活動記錄](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：活動記錄分析解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。
-   [變更追蹤](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：變更追蹤解決方案可讓客戶輕鬆地識別環境中的變更。

**Azure 監視器**
[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)藉由讓組織稽核、建立警示及封存資料，包括追蹤客戶 Azure 資源中的 API 呼叫，協助使用者追蹤效能、維護安全性和識別趨勢。

## <a name="threat-model"></a>威脅模型
此參考架構的資料流程圖可供[下載](https://aka.ms/fedrampWAdfd)，或可以在以下位置找到。 此模型可協助客戶在進行修改時，了解系統基礎結構中的潛在風險要點。

![適用於 FedRAMP 的 IaaS Web 應用程式威脅模型](images/fedramp-iaaswa-threat-model.png?raw=true "適用於 FedRAMP 的 IaaS Web 應用程式威脅模型")

## <a name="compliance-documentation"></a>合規性文件

[Azure 安全性與合規性藍圖 – FedRAMP 高階客戶責任對照表](https://aka.ms/blueprinthighcrm)會列出 FedRAMP 高階基準所需的所有安全性控制項。 此對照表指出每個控制項的實作是由 Microsoft、客戶還是兩者共同負責。

[Azure 安全性與合規性藍圖 – FedRAMP IaaS Web 應用程式高階控制項實作對照表](https://aka.ms/blueprintwacim)會列出 FedRAMP 高階基準所需的所有安全性控制項。 此對照表提供 IaaS Web 應用程式架構涵蓋了哪些控制項的相關資訊，包括詳細說明此實作如何符合每個涵蓋控制項的需求。

## <a name="deploy-the-solution"></a>部署解決方案

此「Azure 安全性與合規性藍圖自動化」包含 JSON 設定檔和 PowerShell 指令碼，它們會由 Azure Resource Manager 的 API 服務來處理，以在 Azure 中部署資源。 詳細的部署指示可於[這裡](https://aka.ms/fedrampblueprintrepo) \(英文\) 取得。
> [!NOTE]
> 此解決方案會部署到 Azure Government。

#### <a name="quickstart"></a>快速入門
1. 將[這個](https://aka.ms/fedrampblueprintrepo) GitHub 存放庫複製或下載到您的本機工作站。

2. 執行預先部署的 PowerShell 指令碼：azure-blueprint/predeploy/Orchestration_InitialSetup.ps1。

3. 按一下下方的按鈕，登入 Azure 入口網站，然後輸入必要的 ARM 範本參數，再按一下 [購買]。

    [![部署至 Azure](http://azuredeploy.net/AzureGov.png)](https://portal.azure.us/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Ffedramp-iaas-webapp%2Fmaster%2Fazuredeploy.json)

## <a name="guidance-and-recommendations"></a>指引與建議
### <a name="vpn-and-expressroute"></a>VPN 和 ExpressRoute
您需要設定安全的 VPN 通道或 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)，以安全地連線到部署為此 IaaS Web 應用程式參考架構一部分的資源。 透過適當地設定 VPN 或 ExpressRoute，客戶可以在傳輸過程中新增資料保護層。

藉由實作與 Azure 的安全 VPN 通道，即可建立內部部署網路與 Azure 虛擬網路之間的虛擬私人連線。 此連線透過網際網路進行，可讓客戶在客戶的網路與 Azure 之間的加密連結內安全地「輸送」資訊。 站對站 VPN 是安全成熟的技術，各種規模的企業已部署數十年。 此選項使用 [IPsec 通道模式](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2003/cc786385(v=ws.10))作為加密機制。

由於 VPN 通道內的流量會透過站對站 VPN 周遊網際網路，因此 Microsoft 提供另一個更安全的連線選項。 Azure ExpressRoute 是 Azure 與內部部署位置或 Exchange 主機服務提供者之間專用的 WAN 連結。 ExpressRoute 連線不會經過網際網路，相較於一般網際網路連線，這些連線提供了更可靠、更快速、更低延遲、更安全的連線。 此外，因為這是客戶電信提供者的直接連線，所以資料不會透過網際網路傳輸，因此不會公開給網際網路。

如需實作將內部部署網路擴充至 Azure 之安全混合式網路的最佳做法，請參閱[這裡](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid)。

## <a name="disclaimer"></a>免責聲明

- 此文件僅供參考之用。 Microsoft 對本文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 本文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。  
- 本文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。  
- 客戶可以複製並使用這份文件，供內部參考之用。  
- 本文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。  
- 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
- 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
