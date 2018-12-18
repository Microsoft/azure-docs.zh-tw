---
title: Azure 安全性與合規性藍圖 - 適用於 Australia PROTECTED 的 PaaS Web 應用程式
description: Azure 安全性與合規性藍圖 - 適用於 Australia PROTECTED 的 PaaS Web 應用程式
services: security
author: meladie
ms.assetid: 708aa129-b226-4e02-85c6-1f86e54564e4
ms.service: security
ms.topic: article
ms.date: 08/23/2018
ms.author: meladie
ms.openlocfilehash: bd6ac05da5f60fdbe86897ce03ccd6646e6bf191
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49390257"
---
# <a name="azure-security-and-compliance-blueprint---paas-web-application-for-australia-protected"></a>Azure 安全性與合規性藍圖 - 適用於 Australia PROTECTED 的 PaaS Web 應用程式

## <a name="overview"></a>概觀

此 Azure 安全性與合規性藍圖提供平台即服務 (PaaS) 環境部署的指導方針，以用於收集、儲存及擷取符合澳洲政府資訊安全性手冊 (ISM) 規範 (由澳洲國防情報局 (ASD) 所編纂) 的 AU-PROTECTED 政府資料。 此藍圖顯示一般參考架構，並有助於示範如何在安全且符合規範的多層環境中處理敏感性政府資料。

此參考架構、實作指南與威脅模型提供一個基礎，讓客戶進行其自己的規劃與系統認證程序，協助客戶以符合 ASD 規範的方式將工作負載部署到 Azure。 客戶可能會選擇實作 Azure VPN 閘道或 ExpressRoute 以使用同盟服務，並將內部部署資源與 Azure 資源整合。 客戶必須考慮使用內部部署資源的安全性意涵。 必須進行額外設定才能符合所有需求，因為它們可能會根據每個客戶的實作特性而不同。

為符合 ASD 規範，必須由「資訊安全註冊評估員」稽核系統。 客戶須負責為任何使用架構建置的解決方進行適當的安全性與合規性評量，因為需求可能會隨每個客戶的實作細節而有所不同。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件
此解決方案提供具有 Azure SQL Database 後端之 PaaS Web 應用程式的參考架構。 Web 應用程式裝載在隔離的 Azure App Service 環境中，它是 Azure 資料中心內的私人專用環境。 環境會在 Azure 管理的虛擬機器之間對 Web 應用程式流量進行負載平衡。 所有 Web 應用程式連線都需要 1.2 版以上的 TLS。 此架構也包含網路安全性群組、應用程式閘道、Azure DNS 和 Load Balancer。

此架構能提供可將內部部署網路延伸至 Azure 的安全混合式環境，以允許由組織私人區域網路的公司使用者或從網際網路安全地存取 Web 型工作負載。 針對內部部署解決方案，客戶必須負責安全性、作業與合規性的各方面。

此解決方案中包括的 Azure 資源可以使用 VPN 閘道並透過 ExpressRoute 以 IPSec VPN 方式連線到內部部署網路或資料中心共置設備 (例如坎培拉的疾病管制局 (CDC))。 決定是否使用 VPN 時應該考慮傳輸資料的分類與網路路徑。 執行具有巨量資料需求的大規模、任務關鍵性工作負載的客戶，應該考慮使用 ExpressRoute 的混合式網路架構，以取得針對 Azure 服務的私人網路連線能力。 請參閱[指導方針與建議](#guidance-and-recommendations)一節以取得有關 Azure 連線機制的詳細資料。

與 Azure Active Directory 的同盟應該用來讓使用者可以使用內部部署認證來進行驗證並使用內部部署 Active Directory 同盟服務基礎結構來存取雲端中的所有資源。 Active Directory 同盟服務可以為此混合式環境提供簡化、安全的身分識別同盟與 Web 單一登入功能。 請參閱[指導方針與建議](#guidance-and-recommendations)一節以取得有關 Azure Active Directory 設定方式的詳細資料。

解決方案會使用 Azure 儲存體帳戶，客戶可加以設定，以便使用儲存體服務加密來維護待用資料的機密性。 Azure 會在客戶所選的區域內儲存三份資料複本以供復原之用。 Azure 區域是部署在復原性良好的區域組中，而且異地備援儲存體可確保資料將會被複寫到第二個區域，而且也會有三份複本。 這可防止客戶主要資料位置的不良反應造成資料遺失。

為加強安全性，此解決方案中的所有 Azure 資源都會透過 Azure Resource Manager 以資源群組方式管理。 Azure Active Directory 角色型存取控制可用來控制對於已部署的資源和 Azure Key Vault 中的金鑰所做的存取。 透過 Azure 資訊安全中心和 Azure 監視器來監視系統健康情況。 客戶可設定這兩項監視服務來擷取記錄，並在易於瀏覽的單一儀表板上顯示系統健康情況。 Azure 應用程式閘道已設定為防止模式中的防火牆，而且不允許非 TLS v1.2 以上版本的流量。 解決方案會利用 Azure 應用程式服務環境 v2，以在非多租用戶環境中隔離 Web 層。

![適用於 AU-PROTECTED 的 PaaS Web 應用程式參考架構](images/au-protected-paaswa-architecture.png?raw=true "適用於 AU-PROTECTED 的 PaaS Web 應用程式參考架構圖表")

此解決方案會使用下列 Azure 服務。 進一步詳細資訊位於[部署架構](#deployment-architecture)一節。

- 應用程式閘道
    - Web 應用程式防火牆
        - 防火牆模式：防止
        - 規則集：OWASP
        - 接聽程式連接埠：443
- Application Insights
- Azure Active Directory
- Azure 應用程式服務環境 v2
- Azure 自動化
- Azure DNS
- Azure 金鑰保存庫
- Azure Load Balancer
- Azure 監視器
- Azure Resource Manager
- Azure 資訊安全中心
- 連接字串
- Azure 儲存體
- Azure Log Analytics
- Azure 虛擬網路
    - (1) /16 網路
    - (4) /24 網路
    - 網路安全性群組
- 網路安全性群組
- 復原服務保存庫
- Azure Web 應用程式

此藍圖包含尚未通過澳洲網路安全中心 (ACSC) 受保護分類認證而可使用的 Azure 服務。 Microsoft 建議客戶檢閱與這些 Azure 服務相關的已發行安全性與稽核報告，並使用其風險管理架構來判斷 Azure 服務是否適合其內部認證並在受保護分類使用。

## <a name="deployment-architecture"></a>部署架構
下一節會詳細說明部署和實作元素。

**Azure Resource Manager**：[Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) 可讓客戶將解決方案中的資源作為群組使用。 客戶可以透過單一協調的作業來部署、更新或刪除解決方案的所有資源。 客戶可使用部署的範本，且該範本可以用於不同的環境，例如測試、預備和生產環境。 資源管理員會提供安全性、稽核和標記功能，以協助客戶在部署後管理資源。

**防禦主機**：防禦主機是單一進入點，可讓使用者存取此環境中已部署的資源。 防禦主機為已部署的資源提供安全連線，其允許的遠端流量僅限來自安全清單上的公用 IP 位址。 若要允許遠端桌面 (RDP) 流量，必須在網路安全性群組中定義該流量的來源。

此解決方案會建立虛擬機器，作為具有下列設定之加入網域的防禦主機：
-   [反惡意程式碼軟體擴充功能](https://docs.microsoft.com/azure/security/azure-security-antimalware)
-   [Azure 診斷擴充功能](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-extensions-diagnostics-template)
-   使用 Azure Key Vault 的 [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)
-   [自動關閉原則](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/) \(英文\)，可減少不使用虛擬機器資源時的耗用量。

**App Service 環境 v2**：[Azure App Service 環境](https://docs.microsoft.com/azure/app-service/environment/intro)是 App Service 的功能，可提供完全隔離和專用的環境，以便安全地大規模執行 App Service 應用程式。

App Service 環境已經過隔離，可執行只有單一客戶的應用程式，且一律會部署到虛擬網路。 客戶對於輸入和輸出的應用程式網路流量都有更細微的控制，且應用程式可以透過虛擬網路建立與內部部署公司資源的高速安全連線。

針對此架構使用 App Service 環境，可以使用下列控制措施/設定：

- App Service 環境必須設定為使用隔離的服務方案
    - 針對屬於不同分類的應用程式設定不同的 App Service 環境
- 在安全的 Azure 虛擬網路和網路安全性規則內裝載
- 使用自我簽署的內部負載平衡器憑證設定 App Service 環境，以進行 HTTPS 通訊。 Microsoft 建議的最佳做法是使用受信任的憑證授權單位，來增強安全性。
- [內部負載平衡模式](https://docs.microsoft.com/azure/app-service-web/app-service-environment-with-internal-load-balancer) (模式 3)
- 停用 [TLS v1.0 和 v1.1](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-custom-settings)
- 變更 [TLS 加密](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-custom-settings)
- 控制[輸入流量的 N/W 連接埠](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-control-inbound-traffic)
- [Web 應用程式防火牆 - 限制資料](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-web-application-firewall)
- 允許 [Azure SQL Database 流量](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-network-architecture-overview)

**Azure Web 應用程式**：[Azure Web Apps](https://docs.microsoft.com/azure/app-service/) 可讓客戶以選定的程式設計語言來建置並託管 Web 應用程式，無需管理基礎結構。 它提供自動調整功能與高度可用性，支援 Windows 與 Linux，而且可讓您從 GitHub、Azure DevOps Services 或任何 Git 存放庫進行自動部署。

### <a name="virtual-network"></a>虛擬網路
此架構會定義位址空間為 10.200.0.0/16 的私人虛擬網路。

**網路安全性群組**：[網路安全性群組](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)包含能允許或拒絕虛擬網路內流量的存取控制清單。 網路安全性群組可用來保護子網路或個別虛擬機器層級的流量。 有下列網路安全性群組：
- 1 個網路安全性群組用於應用程式閘道
- 1 個網路安全性群組用於 App Service 環境
- 1 個網路安全性群組用於 Azure SQL Database
- 1 個網路安全性群組用於用於堡壘主機

每個網路安全性群組都會開放特定連接埠及通訊協定，讓解決方案可安全且正確地運作。 此外，下列組態會針對每個網路安全性群組啟用：

  - 啟用[診斷記錄和事件](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)並儲存在儲存體帳戶
  - Azure Log Analytics 會連線至[網路安全性群組的診斷](https://github.com/krnese/AzureDeploy/blob/master/AzureMgmt/AzureMonitor/nsgWithDiagnostics.json)

**子網路**：確認每個子網路都與對應的網路安全性群組建立關聯。

**Azure DNS**：網域名稱系統 (或 DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-overview) 是 DNS 網域的主機服務，採用 Azure 基礎結構來提供名稱解析。 只要將網域裝載於 Azure，使用者就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具和計費方式來管理 DNS 記錄。 Azure DNS 也支援私人 DNS 網域。

**Azure Load Balancer**：[Azure Load Balancer](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview) 可讓客戶調整應用程式，並為服務建立高可用性。 對於所有 TCP 和 UDP 應用程式，Load Balancer 支援輸入和輸出案例、提供低延遲和高輸送量，且最多可相應增加為數百萬個流程。

### <a name="data-in-transit"></a>資料傳輸中
Azure 預設會加密與 Azure 資料中心的所有通訊。 

針對客戶自有網路中的傳輸中受保護資料，架構會使用網際網路或 ExpressRoute 搭配已使用 IPSEC 設定的 VPN 閘道。

此外，透過 Azure 管理入口網站與 Azure 進行的所有交易都會透過 HTTPS 使用 TLS v1.2 發生。

### <a name="data-at-rest"></a>待用資料
架構會透過加密、資料庫稽核及其他量值來保護待用資料。

**Azure 儲存體**：為符合加密的待用資料需求，所有 [Azure 儲存體](https://azure.microsoft.com/services/storage/)都會使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)。 這有助於保護與防衛資料，以支援組織的安全性承諾及澳洲政府所定義的合規性需求。

**Azure 磁碟加密**：[Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)會利用 Windows 的 BitLocker 功能來提供資料磁碟的磁碟區加密。 此解決方案與 Azure Key Vault 整合，以協助控制和管理磁碟加密金鑰。

**Azure SQL Database**：Azure SQL Database 執行個體會使用下列資料庫安全性量值：
-   [Active Directory 驗證與授權](https://docs.microsoft.com/azure/sql-database/sql-database-AAD-authentication)可讓您在一個中央位置，管理資料庫使用者及其他 Microsoft 服務的身分識別。
-   [SQL 資料庫稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started)會追蹤資料庫事件並將事件寫入 Azure 儲存體帳戶中的稽核記錄。
-   Azure SQL Database 設定成使用[透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)，會執行資料庫、相關備份及交易記錄檔的即時加密和解密，以保護待用資訊。 透明資料加密能保證已儲存的資料不會遭到未經授權的存取。
-   [防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)可在授與適當權限之前，防止所有對資料庫伺服器的存取。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。
-   [SQL 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-get-started)可針對可疑的資料庫活動、潛在弱點、SQL 插入式攻擊，以及異常的資料庫存取模式，提供安全性警示，藉以在發生潛在威脅時啟用偵測及回應。 SQL 威脅偵測會整合警示與 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)，其包括可疑活動的詳細資料，以及如何調查與降低威脅的建議。
-   [Always Encrypted 資料行](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)可確保敏感性資料在資料庫系統內一律不會以純文字顯示。 啟用資料加密之後，只有具備金鑰存取權的用戶端應用程式或應用程式伺服器才可以存取純文字資料。
- [SQL Database 動態資料遮罩](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)可藉由遮避資料，使不具權限的使用者或應用程式無法看見敏感性資料。 動態資料遮罩可以自動探索潛在的敏感性資料，並建議套用適當的遮罩。 這有助於減少存取，敏感性資料就不會經由未經授權的存取而離開資料庫。 客戶必須調整動態資料遮罩設定，以配合其資料庫結構描述。

### <a name="identity-management"></a>身分識別管理
客戶可能會使用內部部署 Active Directory 同盟服務來與 [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) 這個 Microsoft 的多租用戶雲端型目錄及身分識別管理服務建立同盟。 [Azure Active Directory Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect) 會將內部部署目錄與 Azure Active Directory 整合。 此解決方案中的所有使用者都需要 Azure Active Directory 帳戶，包括存取 Azure SQL Database 的使用者。 使用同盟登入時，使用者可以使用內部部署認證登入 Azure Active Directory 並向 Azure 資源驗證。

此外，下列 Azure Active Directory 功能有助於管理對 Azure 環境中資料的存取：
- 應用程式的驗證是使用 Azure Active Directory 執行。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。 此外，資料庫資料行加密會使用 Azure Active Directory 向 Azure SQL Database 驗證應用程式。 如需詳細資訊，請參閱如何[保護 Azure SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
- [Azure 角色型存取控制](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure)可讓系統管理員定義微調存取權限，只對使用者授與其執行工作所需的存取權數量。 系統管理員可以只允許存取資料的特定動作，而不是授與所有使用者不受限制的 Azure 資源使用權限。 只有訂用帳戶管理員可擁有訂用帳戶的存取權。
- [Azure Active Directory Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started) 可讓客戶將可存取特定資訊的使用者人數降至最低。 系統管理員可以使用 Azure Active Directory Privileged Identity Management 來探索、限制和監視特殊權限的識別和其對資源的存取。 如有需要，這項功能也可用來強制執行隨選 Just-In-Time 系統管理存取權。
- [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測影響組織身分識別的潛在弱點，並為偵測到的組織身分識別相關可疑活動設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。

**Azure Multi-Factor Authentication**：為保護身分識別，應該實作多重要素驗證。 [Azure Multi-Factor Authentication](https://azure.microsoft.com/services/multi-factor-authentication/) 是一個容易使用、可調整且可靠的解決方案，可提供第二種驗證方法來保護使用者。 Azure Multi-Factor Authentication 採用雲端技術且與您內部部署的 Active Directory 和自訂應用程式整合。 此保護功能可以延伸到高任務關鍵性的案例。

### <a name="security"></a>安全性
**祕密管理**：解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和祕密。 Azure 金鑰保存庫可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。 下列 Azure Key Vault 功能可協助客戶保護資料：
- 進階存取原則是視需要設定的。
- Key Vault 存取原則是使用金鑰和祕密的最低必要權限所定義的。
- Key Vault 中的所有金鑰和祕密都有到期日。
- Key Vault 中的所有金鑰都會受到特製化硬體安全模組的保護。 金鑰類型是硬體安全性模組保護的 2048 位元 RSA 金鑰。
- 針對所有使用者與身分識別，皆使用角色型存取控制來授與最低要求權限。
- 金鑰保存庫的診斷記錄的保留期至少 365 天。
- 允許的金鑰密碼編譯作業僅限於需要的密碼編譯項目。

**Azure 資訊安全中心**：客戶可以使用 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)，在工作負載之間集中套用及管理安全性原則、限制暴露於威脅的程度，以及偵測和回應攻擊。 此外，Azure 資訊安全中心會存取 Azure 服務的現有組態，以提供設定和服務建議，協助改善安全性狀態並保護資料。

Azure 資訊安全中心會使用不同的偵測功能，向客戶警示以其環境為目標的潛在攻擊。 這些警示包含觸發警示的項目、鎖定為目標的資源，以及攻擊來源等重要資訊。 Azure 資訊安全中心有一組[預先定義的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)，一旦發生威脅或可疑活動時便會觸發這些警示。 Azure 資訊安全中心內的[自訂警示規則](https://docs.microsoft.com/azure/security-center/security-center-custom-alert)可讓客戶根據從其環境所收集到的資料，定義新的安全性警示。

Azure 資訊安全中心提供依優先順序排列的安全性警示和事件，讓客戶更容易探索及解決潛在的安全性問題。 針對每個偵測到的威脅會產生[威脅情報報告](https://docs.microsoft.com/azure/security-center/security-center-threat-report)，以協助事件回應小組調查威脅並進行補救。

**應用程式閘道**：透過已啟用 Web 應用程式防火牆和 OWASP 規則集的應用程式閘道，此架構可減少安全性弱點帶來的風險。 其他功能包括：

- [端對端 SSL](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- 啟用 [SSL 卸載](https://docs.microsoft.com/azure/application-gateway/application-gateway-ssl-portal)
- 停用 [TLS v1.0 和 v1.1](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- [Web 應用程式防火牆](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) (防止模式)
- 使用 OWASP 3.0 規則集的[防止模式](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-portal)
- 啟用[診斷記錄](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)
- [自訂健康情況探查](https://docs.microsoft.com/azure/application-gateway/application-gateway-create-gateway-portal)
- [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center)和 [Azure Advisor](https://docs.microsoft.com/azure/advisor/advisor-security-recommendations) 會提供額外的保護和通知。 Azure 資訊安全中心也會提供評價系統。

### <a name="logging-and-auditing"></a>記錄與稽核

Azure 服務會廣泛記錄系統、使用者活動及系統健康情況：
- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。 活動記錄可協助判斷作業的啟動器、出現時間和狀態。
- **診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)包含每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、Azure 儲存體記錄、Key Vault 稽核記錄，以及應用程式閘道存取和防火牆記錄。 所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存。 保留期是由使用者自訂，視組織特定的保留期需求，最長可達 730 天。

**Log Analytics**：這些記錄會合併到 [Log Analytics](https://azure.microsoft.com/services/log-analytics/) 以進行處理、儲存，並從儀表板顯示報告。 所收集的資料會針對每種資料類型組織成個別的資料表，以便一起分析所有的資料 (不論其原始來源為何)。 此外，Azure 資訊安全中心會與 Log Analytics 整合，讓客戶使用 Log Analytics 查詢來存取其安全性事件資料，並將其與來自其他服務的資料合併。

此架構包含下列 Log Analytics [管理解決方案](https://docs.microsoft.com/azure/log-analytics/log-analytics-add-solutions)：
-   [Active Directory 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：「Active Directory 健康情況檢查」解決方案會定期評估伺服器環境的風險和健康情況，並針對部署的伺服器基礎結構，提供依優先順序排列的建議清單。
- [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器架構，提供優先的建議清單給客戶。
- [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈，以及沒有回應的代理程式數目和正在提交作業資料的代理程式數目。
-   [活動記錄分析](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：「活動記錄分析」解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。

**Azure 自動化**：[Azure 自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)會儲存、執行和管理 Runbook。 在此解決方案中，Runbook 會協助從 Azure SQL Database 中收集記錄。 自動化[變更追蹤](https://docs.microsoft.com/azure/automation/automation-change-tracking)解決方案可讓客戶輕鬆地識別環境中的變更。

**Azure 監視器**：[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)藉由讓組織稽核、建立警示及封存資料，包括追蹤使用者 Azure 資源中的 API 呼叫，協助使用者追蹤效能、維護安全性和識別趨勢。

Azure 網路監看員：[Azure 網路監看員]https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)提供了相關工具，可對 Azure 虛擬網路中的資源進行監視、診斷、檢視計量，以及啟用或停用記錄。  實體應該跟對 NSG 與虛擬機器實作網路監看員流量記錄。 這些記錄應該存放在只用來存放安全性記錄的專屬儲存體帳戶中，而且必須使用角色行存取控制來保護對該儲存體帳戶的存取。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖可供[下載](https://aka.ms/au-protected-paaswa-tm)，或可以在以下位置找到。 此模型可協助客戶在進行修改時，了解系統基礎結構中的潛在風險要點。

![適用於 AU-PROTECTED 的 PaaS Web 應用程式威脅模型](images/au-protected-paaswa-threat-model.png?raw=true "適用於 AU-PROTECTED 的 PaaS Web 應用程式威脅模型圖表")

## <a name="compliance-documentation"></a>合規性文件
此合規性文件是由 Microsoft 根據 Microsoft 所提供的平台與服務所建立。 由於客戶部署的廣泛性，此文件只針對 Azure 環境中裝載的解決方案提供一般方法。 客戶可以根據自己的作業環境與商業結果來尋找並使用替代產品與服務。 選擇使用內部部署資源的客戶必須解決那些內部部署資源的安全性與作業問題。 客戶可以自訂已記錄的解決方案以滿足其特定內部部署與安全性需求。

[Azure 安全性與合規性藍圖 - AU-PROTECTED 客戶責任對照表](https://aka.ms/au-protected-crm)列出 AU-Prot 所需的所有安全性控制措施。 此對照表詳細說明每個控制項的實作是由 Microsoft、客戶還是兩者共同負責。

[Azure 安全性與合規性藍圖 - AU-PROTECTED PaaS Web 應用程式實作對照表](https://aka.ms/au-protected-paaswa-cim)提供 PaaS Web 應用程式架構處理了哪些 AU-PROTECTED 控制措施的相關資訊，包括詳細說明此實作如何滿足每個涵蓋控制措施的需求。

## <a name="guidance-and-recommendations"></a>指引與建議
### <a name="vpn-and-expressroute"></a>VPN 和 ExpressRoute

針對分類資訊，您需要設定安全的 IPSec VPN 通道，以安全地連線到部署為此 IaaS Web 應用程式參考架構一部分的資源。 透過適當地設定 IPSec VPN，客戶可以在傳輸過程中加上資料保護層。

透過實作與 Azure 的安全 IPSec VPN 通道，即可建立內部部署網路與 Azure 虛擬網路之間的虛擬私人連線。 此連線可以透過網際網路進行，可讓客戶在客戶的網路與 Azure 之間的加密連結內安全地「輸送」資訊。 站對站 VPN 是安全成熟的技術，各種規模的企業已部署數十年。 

因為 VPN 通道內的流量會透過站對站 VPN 周遊網際網路，因此 Microsoft 提供私人連線選項。 Azure ExpressRoute 是 Azure 與內部部署位置或 Exchange 主機服務提供者之間專用的連結，而且會被視為私人網路。 ExpressRoute 連線不會經過網際網路，相較於一般網際網路連線，這些連線提供更可靠、更快速、延遲更低的連線。 此外，因為這是客戶電信提供者的直接連線，所以資料不會透過網際網路傳輸，因此不會公開給網際網路。

如需實作將內部部署網路擴充至 Azure 之安全混合式網路的最佳做法，請參閱[這裡](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid)。 

為協助保護已分類的資料 (不論是使用網際網路或 Azure ExpressRoute)，客戶都必須透過套用下列設定以設定其 IPSec VPN：

•   客戶 VPN 啟動器必須將 SA 存留期設定為不超過 14400 秒的值。
•   客戶 VPN 啟動器必須停用 IKEv1 加強模式。
•   客戶 VPN 啟動器必須設定「完整轉寄密碼」。
•   客戶 VPN 啟動必須設定為使用 HMAC-SHA256 或更高的選項。

[這裡](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-devices)提供 VPN 裝置與 IPSec/ IKE 參數的設定選項。

### <a name="azure-active-directory-setup"></a>Azure Active Directory 設定
您必須使用 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) 來管理部署，以及為與環境互動的人員佈建存取權。 您只要[點按四下](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-get-started-express)，就可整合現有的 Windows Server Active Directory 與 Azure Active Directory。

此外，[Azure Active Directory Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect) 允許客戶設定內部部署 [Active Directory 同盟服務]( https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-azure-adfs)與 Azure Active Directory 的同盟。 使用同盟登入時，客戶可以讓使用者使用其內部部署密碼登入 Azure Active Directory 型服務，而且在其公司網路上不需要再次輸入密碼。 透過使用與「Active Directory 同盟服務」同盟的選項，您可以部署「Active Directory 同盟服務」的新安裝，或者您可以指定 Windows Server 2012 R2 伺服器陣列中的現有安裝。

若要防止已分類的資料同步到 Azure Active Directory，客戶可以在 Azure Active Directory Connect 中套用下列設定，以限制複寫到 Azure Active Directory 的屬性：

- [啟用篩選](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-configure-filtering)
- [停用密碼雜湊同步處理](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-hash-synchronization)
-   [停用密碼回寫](https://docs.microsoft.com/azure/active-directory/authentication/quickstart-sspr)
-   [停用裝置回寫](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-feature-device-writeback)
-   將[防止意外刪除](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-feature-prevent-accidental-deletes)與[自動升級](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-feature-automatic-upgrade)維持為預設設定


## <a name="disclaimer"></a>免責聲明

 - 此文件僅供參考之用。 Microsoft 對本文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 本文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
 - 本文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
 - 客戶可以複製並使用這份文件，供內部參考之用。
 - 本文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
 - 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
 - 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
