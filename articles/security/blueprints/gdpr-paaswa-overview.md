---
title: Azure 安全性與合規性藍圖 - 適用於 GDPR 的 PaaS Web 應用程式
description: Azure 安全性與合規性藍圖 - 適用於 GDPR 的 PaaS Web 應用程式
services: security
author: jomolesk
ms.assetid: 229759a1-f984-4985-a3fd-3719a7d1c8ff
ms.service: security
ms.topic: article
ms.date: 05/14/2018
ms.author: jomolesk
ms.openlocfilehash: 26227e1a6766a80bbcef3cfda3f2faee82396fe3
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45577049"
---
# <a name="azure-security-and-compliance-blueprint---paas-web-application-for-gdpr"></a>Azure 安全性與合規性藍圖 - 適用於 GDPR 的 PaaS Web 應用程式

## <a name="overview"></a>概觀
一般資料保護規定 (GDPR) 包含許多有關收集、儲存和使用個人資訊的需求，包括組織如何識別及保護個人資料、容納透明度需求、偵測和報告個人資料缺口，以及訓練隱私權人員和其他員工。 GDPR 讓個人可進一步控制其個人資料，並對收集、處理或分析個人資料的組織加諸許多新義務。 GDPR 會針對提供商品和服務給歐盟 (EU) 居民，或收集和分析 EU 居民相關資料的組織，施加新的規則。 無論組織位於何處都適用 GDPR。

Microsoft 設計的 Azure 具有領先業界的安全性措施和隱私權原則，可保護雲端資料，包括 GDPR 所識別的個人資料類別。 Microsoft 的[合約條款](http://aka.ms/Online-Services-Terms)要求 Microsoft 符合處理器需求。

此「Azure 安全性與合規性藍圖」提供指引以部署平台即服務 (PaaS) 環境，此環境適合網際網路面向的簡易 Web 應用程式。 此解決方案示範的方式讓客戶能符合 GDPR 特定的安全性和合規性需求，並且作為客戶在 Azure 上建置及設定自有 PaaS Web 應用程式解決方案的基礎。 客戶可以在符合 GDPR 規範的過程中利用此參考架構，並遵循 Microsoft 的[四個步驟程序](https://aka.ms/gdprebook)：
1. 探索：識別存在哪些個人資料，及其所在位置。
2. 管理：管理個人資料使用及存取的方式。
3. 保護：建立安全性控制，以防止、偵測及回應弱點和資料缺口。
4. 報告：保留必要文件以及管理資料要求和缺口通知。

與實作指南相關聯的此參考架構和威脅模型，主要是用來作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。 請注意：
- 架構提供了一個基準，可協助客戶以符合 GDPR 規範的方式來將工作負載部署至 Azure。
- 客戶須負責為任何使用架構建置的解決方進行適當的安全性與合規性評量，因為需求可能會隨每個客戶的實作細節而有所不同。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件
此解決方案提供具有 Azure SQL Database 後端之 PaaS Web 應用程式的參考架構。 Web 應用程式裝載在隔離的 Azure App Service 環境中，其為 Azure 資料中心內的私人專用環境。 環境會在 Azure 管理的 VM 之間負載平衡 Web 應用程式流量。 此架構也包含網路安全性群組、應用程式閘道、Azure DNS 和 Load Balancer。 此外，Application Insights 會透過 Operations Management Suite (OMS) 提供即時應用程式效能管理與分析。 **Azure 會建議設定 VPN 或 ExpressRoute 連線，以便管理和將資料匯入參考架構子網路。**

![適用於 GDPR 的 PaaS Web 應用程式參考架構圖表](images/gdpr-paaswa-architecture.png?raw=true "適用於 GDPR 的 PaaS Web 應用程式參考架構圖表")

此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。

- Azure Active Directory (AAD)
- Azure 金鑰保存庫
- 連接字串
- 應用程式閘道
    - (1) WAF 應用程式閘道已啟用
        - 防火牆模式：防止
        - 規則集：OWASP 3.0
        - 接聽程式：連接埠 443
- Azure 虛擬網路
- 網路安全性群組
- Azure DNS
- Azure 儲存體
- Operations Management Suite (OMS)
- Azure 監視器
- Application Insights
- Azure 資訊安全中心
- App Service Environment v2
- Azure Load Balancer
- Azure Web 應用程式
- Azure Resource Manager

## <a name="deployment-architecture"></a>部署架構
下一節會詳細說明部署和實作元素。

**Azure Resource Manager**：[Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) 可讓客戶將解決方案中的資源作為群組使用。 客戶可以透過單一協調的作業來部署、更新或刪除解決方案的所有資源。 客戶可使用部署的範本，且該範本可以用於不同的環境，例如測試、預備和生產環境。 資源管理員會提供安全性、稽核和標記功能，以協助客戶在部署後管理資源。

**App Service 環境 v2**：[Azure App Service 環境](https://docs.microsoft.com/azure/app-service/environment/intro)是 App Service 的功能，可提供完全隔離和專用的環境，以便安全地大規模執行 App Service 應用程式。

ASE 已經過隔離，可僅執行單一客戶的應用程式，且一律會部署到虛擬網路。 客戶對於輸入和輸出的應用程式網路流量都有更細微的控制，且應用程式可以透過虛擬網路建立與內部部署公司資源的高速安全連線。

下列控制項/設定可允許此架構使用 ASE：

- 在安全的 Azure 虛擬網路和網路安全性規則內主控
- 使用自我簽署的 ILB 憑證設定 ASE，以進行 HTTPS 通訊
- [內部負載平衡模式](https://docs.microsoft.com/azure/app-service-web/app-service-environment-with-internal-load-balancer) (模式 3)
- 停用 [TLS 1.0](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-custom-settings)
- 變更 [TLS 加密](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-custom-settings)
- 控制[輸入流量的 N/W 連接埠](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-control-inbound-traffic)
- [WAF – 限制資料](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-web-application-firewall)
- 允許 [Azure SQL Database 流量](https://docs.microsoft.com/azure/app-service-web/app-service-app-service-environment-network-architecture-overview)

**Azure Web 應用程式**：[Azure Web Apps](https://docs.microsoft.com/azure/app-service/) 可讓客戶以選定的程式設計語言來建置並託管 Web 應用程式，無需管理基礎結構。 它提供自動調整及高度可用性，支援 Windows 和 Linux，並可從 GitHub、Azure DevOps 或任何 Git 存放庫啟用自動部署。

### <a name="virtual-network"></a>虛擬網路
架構會定義位址空間為 10.200.0.0/16 的私人 VNet。

**網路安全性群組**：[NSG](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 包含能允許或拒絕 VNet 內流量的存取控制清單 (ACL)。 NSG 可以用來保護子網路或個別 VM 層級的流量。 下列 NSG 已經存在:
- 1 個 NSG 用於應用程式閘道
- 1 個 NSG 用於 App Service 環境
- 1 個 NSG 用於 Azure SQL Database

每個 NSG 都會開放特定連接埠及通訊協定，讓解決方案可安全且正確地運作。 此外，以下設定會針對每個 NSG 啟用：
  - 啟用[診斷記錄和事件](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)並儲存在儲存體帳戶
  - OMS Log Analytics 會連線至 [NSG 的診斷](https://github.com/krnese/AzureDeploy/blob/master/AzureMgmt/AzureMonitor/nsgWithDiagnostics.json)

**子網路**：確認每個子網路都與對應的 NSG 相關聯。

**Azure DNS**：網域名稱系統 (或 DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-overview) 是 DNS 網域的主機服務，採用 Azure 基礎結構來提供名稱解析。 只要將網域裝載於 Azure，使用者就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具和計費方式來管理 DNS 記錄。 Azure DNS 也支援私人 DNS 網域。

**Azure Load Balancer**：[Azure Load Balancer](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview) 可讓客戶調整應用程式，並為服務建立高可用性。 對於所有 TCP 和 UDP 應用程式，Load Balancer 支援輸入和輸出案例、提供低延遲和高輸送量，且最多可相應增加為數百萬個流程。

### <a name="data-in-transit"></a>資料傳輸中
Azure 預設會加密與 Azure 資料中心的所有通訊。 透過 Azure 入口網站與 Azure 儲存體進行的所有交易都會透過 HTTPS 發生。

### <a name="data-at-rest"></a>待用資料

架構會透過加密、資料庫稽核及其他量值來保護待用資料。

**Azure 儲存體**：為符合加密的待用資料需求，所有 [Azure 儲存體](https://azure.microsoft.com/services/storage/)都會使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)。 這有助於保護與防衛個人資料，以支援組織的安全性承諾及 GDPR 所定義的合規性需求。

**Azure 磁碟加密**
[Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)利用 Windows 的 BitLocker 功能來提供資料磁碟的磁碟區加密。 此解決方案與 Azure Key Vault 整合，以協助控制和管理磁碟加密金鑰。

**Azure SQL Database**：Azure SQL Database 執行個體會使用下列資料庫安全性量值：
-   [AD 驗證與授權](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)可讓您在一個中央位置管理資料庫使用者及其他 Microsoft 服務的身分識別。
-   [SQL 資料庫稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started)會追蹤資料庫事件並將事件寫入 Azure 儲存體帳戶中的稽核記錄。
-   Azure SQL Database 設定成使用[透明資料加密 (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)，它會執行資料庫、相關備份及交易記錄檔的即時加密和解密，以保護待用資訊。 TDE 能保證已儲存的個人資料不會遭到未經授權的存取。
-   [防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)可在授與適當權限之前，防止所有對資料庫伺服器的存取。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。
-   [SQL 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-get-started)可針對可疑的資料庫活動、潛在弱點、SQL 插入式攻擊，以及異常的資料庫存取模式，提供安全性警示，藉以在發生潛在威脅時啟用偵測及回應。
-   [Always Encrypted 資料行](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)可確保敏感性個人資料在資料庫系統內一律不會以純文字顯示。 啟用資料加密之後，只有具備金鑰存取權的用戶端應用程式或應用程式伺服器才可以存取純文字資料。
- [擴充屬性](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addextendedproperty-transact-sql)可用來中斷資料主體的處理，因為它可讓使用者將自訂屬性新增至資料庫物件，並將資料標記為「已中止」，以支援應用程式邏輯來防止處理相關聯的個人資料。
- [資料列層級安全性](https://docs.microsoft.com/sql/relational-databases/security/row-level-security)可讓使用者定義原則，限制資料存取以中斷處理。
- [SQL 資料庫動態資料遮罩 (DDM)](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started) 可藉由遮罩資料，使不具權限的使用者或應用程式無法看見敏感性個人資料。 DDM 可以自動探索潛在的敏感性資料，並建議套用適當的遮罩。 這有助於識別符合 GDPR 保護資格的個人資料，並降低存取以確保這類資料不會因未經授權的存取而離開資料庫。 **注意：客戶必須調整 DDM 設定，以配合其資料庫結構描述。**

### <a name="identity-management"></a>身分識別管理
下列技術可在 Azure 環境中提供管理個人資料存取的功能：
-   [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) 是 Microsoft 的多租用戶雲端式目錄和身分識別管理服務。 此解決方案的所有使用者都是在 AAD 中建立，包括存取 Azure SQL Database 的使用者。
-   應用程式的驗證是使用 AAD 執行。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。 此外，資料庫資料行加密會使用 AAD 向 Azure SQL Database 驗證應用程式。 如需詳細資訊，請參閱如何[保護 Azure SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
-   [Azure 角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure) 可讓系統管理員定義微調存取權限，只對使用者授與其執行工作所需的存取權數量。 系統管理員可以只允許存取個人資料的特定動作，而不是授與所有使用者不受限制的 Azure 資源使用權限。 只有訂用帳戶管理員可擁有訂用帳戶的存取權。
- [AAD Privileged Identity Management (PIM)](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started) 可讓客戶將可存取特定資訊 (例如個人資料) 的使用者人數降至最低。  系統管理員可以使用 AAD Privileged Identity Management 來探索、限制和監視特殊權限的身分識別和其對資源的存取。 如有需要，這項功能也可用來強制執行隨選 Just-In-Time 系統管理存取權。
- [AAD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測影響組織身分識別的潛在弱點，並為偵測到的組織身分識別相關可疑動作設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。

### <a name="security"></a>安全性
**祕密管理**解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和祕密。 Azure 金鑰保存庫可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。 下列 Azure Key Vault 功能可協助客戶保護個人資料及存取這類資料：
- 進階存取原則是視需要設定的。
- Key Vault 存取原則是使用金鑰和祕密的最低必要權限所定義的。
- Key Vault 中的所有金鑰和祕密都有到期日。
- Key Vault 中的所有金鑰都會受到特製化硬體安全模組 (HSM) 的保護。 金鑰類型為 HSM 保護的 2048 位元 RSA 金鑰。
- 所有使用者和身分識別皆使用 RBAC 授與最低必要權限。
- 金鑰保存庫的診斷記錄的保留期至少 365 天。
- 允許的金鑰密碼編譯作業僅限於需要的密碼編譯項目。

**安全性警訊**：[Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)可讓客戶監視流量、收集記錄並分析資料來源中的威脅。 此外，Azure 資訊安全中心會存取 Azure 服務的現有設定，以提供設定和服務建議，協助改善安全性狀態並保護個人資料。 Azure 資訊安全中心針對每個偵測到的威脅會包含[威脅情報報告](https://docs.microsoft.com/azure/security-center/security-center-threat-report)，以協助事件回應小組調查和修復威脅。

**應用程式閘道**透過使用已啟用 Web 應用程式防火牆 (WAF) 和 OWASP 規則集的應用程式閘道，此架構可減少安全性弱點帶來的風險。 其他功能包括：

- [端對端 SSL](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- 啟用 [SSL 卸載](https://docs.microsoft.com/azure/application-gateway/application-gateway-ssl-portal)
- 停用 [TLS v1.0 和 v1.1](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- [Web 應用程式防火牆](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) \(WAF 模式\)
- 使用 OWASP 3.0 規則集的[防止模式](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-portal)
- 啟用[診斷記錄](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)
- [自訂健康情況探查](https://docs.microsoft.com/azure/application-gateway/application-gateway-create-gateway-portal)
- [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center)和 [Azure Advisor](https://docs.microsoft.com/azure/advisor/advisor-security-recommendations) 會提供額外的保護和通知。 Azure 資訊安全中心也會提供評價系統。

### <a name="logging-and-auditing"></a>記錄與稽核

OMS 提供系統、使用者活動及系統健康情況的廣泛記錄。 OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) 解決方案可收集及分析 Azure 和內部部署環境中資源所產生的資料。
- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。 活動記錄可協助判斷作業的啟動器、出現時間和狀態。
- **診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)包含每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、Azure 儲存體記錄、Key Vault 稽核記錄，以及應用程式閘道存取和防火牆記錄。
- **記錄封存**：所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存。 保留期是由使用者自訂，視組織特定的保留期需求，最長可達 730 天。 這些記錄會連線至 Azure Log Analytics 以進行處理、儲存及從儀表板顯示報表。

此外，此架構包含下列 OMS 解決方案：
-   [AD 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：Active Directory 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器基礎結構，提供優先的建議清單。
-   [惡意程式碼評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-malware)：反惡意程式碼軟體解決方案會報告惡意程式碼、威脅及保護狀態。
-   [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)：Azure 自動化解決方案會儲存、執行和管理 Runbook。 在此解決方案中，Runbook 會協助從 Application Insights 和 Azure SQL Database 中收集記錄。
-   [安全性與稽核](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started)：安全性和稽核儀表板會藉由提供有關安全性網域、值得注意的問題、偵測、威脅情報和常見的安全性查詢等計量，來提供資源安全性狀態的整體見解。
-   [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器架構，提供優先的建議清單給客戶。
-   [更新管理](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management)：更新管理解決方案可讓客戶管理作業系統的安全性更新，包括可用更新的狀態和安裝必要更新的程序。
-   [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈，以及沒有回應的代理程式數目和正在提交作業資料的代理程式數目。
-   [Azure 活動記錄](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：活動記錄分析解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。
-   [變更追蹤](https://docs.microsoft.com/azure/automation/automation-change-tracking)：變更追蹤解決方案可讓客戶輕鬆地識別環境中的變更。

**Azure 監視器**
[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)藉由讓組織稽核、建立警示及封存資料，包括追蹤客戶 Azure 資源中的 API 呼叫，協助使用者追蹤效能、維護安全性和識別趨勢。

**Application Insights**
[Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-overview) 是多個平台上的 Web 開發人員所適用的可延伸「應用程式效能管理」(APM) 服務。 Application Insights 會偵測效能異常，客戶可以用它來監視即時 Web 應用程式。 其中包括強大的分析工具可協助客戶診斷問題，並了解使用者實際上如何運用應用程式。 它是設計來協助客戶持續改善效能和可用性。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖可供[下載](https://aka.ms/gdprPaaSdfd)，或可以在以下位置找到。 此模型可協助客戶在進行修改時，了解系統基礎結構中的潛在風險要點。

![適用於 GDPR 的 PaaS Web 應用程式威脅模型](images/gdpr-paaswa-threat-model.png?raw=true "適用於 GDPR 的 PaaS Web 應用程式威脅模型")

## <a name="compliance-documentation"></a>合規性文件
[Azure 安全性與合規性藍圖 - GDPR 客戶責任矩陣](https://aka.ms/gdprCRM)列出所有 GDPR 條款中的控制者和處理者責任。 請注意，在 Azure 服務中，客戶通常是控制者，而 Microsoft 則扮演處理者的角色。

[Azure 安全性與合規性藍圖 - GDPR PaaS Web 應用程式實作矩陣](https://aka.ms/gdprPaaSWeb)提供 PaaS Web 應用程式架構處理哪些 GDPR 條款的相關資訊，包括詳細說明此實作如何符合每條涵蓋條款的需求。


## <a name="guidance-and-recommendations"></a>指引與建議
### <a name="vpn-and-expressroute"></a>VPN 和 ExpressRoute
您需要設定安全的 VPN 通道或 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)，以安全地連線到部署為此 PaaS Web 應用程式參考架構一部分的資源。 透過適當地設定 VPN 或 ExpressRoute，客戶可以在傳輸過程中新增資料保護層。

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
