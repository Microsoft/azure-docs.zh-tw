---
title: Azure 安全性與合規性藍圖 - 適用於 GDPR 的分析
description: Azure 安全性與合規性藍圖 - 適用於 GDPR 的分析
services: security
author: jomolesk
ms.assetid: fe97b5e5-72d8-4930-bbe9-ead2b6ef3542
ms.service: security
ms.topic: article
ms.date: 05/14/2018
ms.author: jomolesk
ms.openlocfilehash: b4f40dfced7060dd01df7410d07ac5b7cfdf3176
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45580696"
---
# <a name="azure-security-and-compliance-blueprint-analytics-for-gdpr"></a>Azure 安全性與合規性藍圖：適用於 GDPR 的分析

## <a name="overview"></a>概觀
一般資料保護規定 (GDPR) 包含許多有關收集、儲存和使用個人資訊的需求，包括組織如何識別及保護個人資料、容納透明度需求、偵測和報告個人資料缺口，以及訓練隱私權人員和其他員工。 GDPR 讓個人可進一步控制其個人資料，並對收集、處理或分析個人資料的組織加諸許多新義務。 GDPR 會針對提供商品和服務給歐盟 (EU) 居民，或收集和分析 EU 居民相關資料的組織，施加新的規則。 無論組織位於何處都適用 GDPR。

Microsoft 設計的 Azure 具有領先業界的安全性措施和隱私權原則，可保護雲端資料，包括 GDPR 所識別的個人資料類別。 Microsoft 的[合約條款](http://aka.ms/Online-Services-Terms)要求 Microsoft 符合處理器需求。

此 Azure 安全性與合規性藍圖提供在 Azure 中部署資料分析架構的指導方針，可協助達成 GDPR 的需求。 這個解決方案會示範客戶可以達成特定安全性與合規性需求的方式，並做為基礎讓客戶能在 Azure 中建置並設定自己的資料分析解決方案。 客戶可以利用此參考架構，並遵循 Microsoft 的[四步驟流程](https://aka.ms/gdprebook)來達成 GDPR 合規性：
1. 探索：識別存在哪些個人資料，及其所在位置。
2. 管理：管理個人資料使用及存取的方式。
3. 保護：建立安全性控制，以防止、偵測及回應弱點和資料缺口。
4. 報告：保留必要文件以及管理資料要求和缺口通知。

與實作指南相關聯的此參考架構和威脅模型，主要是用來作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。 請注意：
- 架構提供了一個基準，可協助客戶以符合 GDPR 規範的方式來將工作負載部署至 Azure。
- 客戶須負責為任何使用架構建置的解決方進行適當的安全性與合規性評量，因為需求可能會隨每個客戶的實作細節而有所不同。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件
這個解決方案提供分析平台，客戶可以在其中建立自己的分析工具。 參考架構概要說明了一般使用案例，其中客戶會透過 SQL/資料管理員執行大量資料匯入，或透過操作使用者更新操作資料，來輸入資料。 這兩個工作流程皆需要結合 Azure Functions，才可將資料匯入 Azure SQL Database。 客戶必須透過 Azure 入口網站來設定 Azure Functions，以處理專屬於每個客戶分析需求的匯入工作。

Azure 已為客戶提供各種報表和分析服務；不過，此解決方案結合 Azure Machine Learning 服務與 Azure SQL Database，可透過更聰明的客戶資料模型，來快速瀏覽資料及傳遞更快的結果。 Azure Machine Learning 是機器學習的一種形式，其透過探索資料集之間的新關係，來加快查詢速度。 一旦透過數個統計函數來將模型定型之後，最多可以讓其他 7 個 (若包括客戶伺服器則是總計 8 個) 查詢集區與相同的表格式模型同步，以分散查詢工作負載並縮短回應時間。

若要加強分析和報告，您可以使用資料行存放區索引來設定 Azure SQL 資料庫。 根據客戶的使用情況，Azure Machine Learning 和 Azure SQL Database 可進行相應增加或減少，或甚至是完全關閉。 所有 SQL 流量都會使用 SSL 透過內含的自我簽署憑證來加密。 Azure 建議的最佳做法是使用受信任的憑證授權單位，來增強安全性。

一旦將資料上傳至 Azure SQL Database，並由 Azure Machine Learning 定型後，操作使用者和 SQL/資料管理員就會使用 Power BI 來處理這些資料。 Power BI 會以直覺方式顯示資料，並同時提取多個資料集中的資訊，來繪製更深入的見解。 因為其適應性高且能輕鬆地與 Azure SQL Database 整合，可確保客戶能根據他們的業務需求進行設定，進而處理大量案例。

整個解決方案都是建立在 Azure 儲存體的基礎上，而客戶會從 Azure 入口網站進行設定。 Azure 儲存體會使用儲存體服務加密來加密所有資料，以維護待用資料的機密性。 異地備援儲存體 (GRS) 可確保發生在客戶主要資料中心內的有害事件不會造成資料遺失，因為系統會將次要複本儲存在數百英里外的個別位置。

為加強安全性，此架構使用 Azure Active Directory 與 Azure Key Vault 來管理資源。 並透過 Operations Management Suite (OMS) 和 Azure 監視器來監視系統健康情況。 客戶可設定這兩項監視服務來擷取記錄，並在易於瀏覽的單一儀表板上顯示系統健康情況。

Azure SQL Database 通常會透過 SQL Server Management Studio (SSMS) 來管理，而執行 SSMS 的本機電腦會設定為透過安全的 VPN 或 ExpressRoute 連線來存取 Azure SQL Database。 **Azure 會建議設定 VPN 或 ExpressRoute 連線，以便管理和將資料匯入參考架構資源群組**。

![適用於 GDPR 的分析參考架構圖表](images/gdpr-analytics-architecture.png?raw=true "適用於 GDPR 的分析參考架構圖表")

此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。

- Azure Functions
- 連接字串
- Azure Machine Learning
- Azure Active Directory
- Azure 金鑰保存庫
- Operations Management Suite (OMS)
- Azure 監視器
- Azure 儲存體
- Power BI 儀表板
- Azure 資料目錄
- Azure 資訊安全中心
- Application Insights
- Azure Event Grid
- 網路安全性群組

## <a name="deployment-architecture"></a>部署架構
下一節會詳細說明部署和實作元素。

**Azure 事件方格**
[Azure 事件方格](https://docs.microsoft.com/azure/event-grid/overview)可讓客戶以事件型架構輕鬆地建立應用程式。 使用者可以選取想要訂閱的 Azure 資源，並提供事件處理常式或 Webhook 端點以作為事件的傳送目的地。 客戶可以在建立事件訂閱時將查詢參數新增至 Webhook URL，以保護 Webhook 端點。 Azure 事件方格只支援 HTTPS Webhook 端點。 Azure 事件方格讓客戶能控制給予不同使用者進行各種管理作業的存取層級，這些作業包含列出事件訂閱、建立新訂閱及產生金鑰等等。 事件方格會利用 Azure 角色型存取控制 (RBAC)。

**Azure Functions**
[Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview) 是無伺服器計算服務，可讓使用者依需求執行程式碼，無需明確佈建或管理基礎結構。 使用 Azure Functions 執行指令碼或一段程式碼來回應各種事件。

**Azure Machine Learning**
[Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/preview/) 是一個資料科學技術，可讓電腦使用現有資料來預測未來的行為、結果和趨勢。

**Azure 資料目錄**：[資料目錄](https://docs.microsoft.com/azure/data-catalog/data-catalog-what-is-data-catalog)能讓管理資料的使用者輕鬆地探索和了解資料來源。 您可以註冊、標記常用資料來源，並在其中搜尋個人資料。 資料會保留在現有的位置，但其中繼資料的複本會連同資料來源位置的參考，一起新增至資料目錄。 此中繼資料也會編製索引，透過搜尋輕鬆找到每個資料來源，並讓探索資料來源的使用者了解每個資料來源。

### <a name="virtual-network"></a>虛擬網路
此參考架構會定義位址空間為 10.0.0.0/16 的私人 VNet。

**網路安全性群組**：[NSG](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 包含能允許或拒絕 VNet 內流量的存取控制清單 (ACL)。 NSG 可以用來保護子網路或個別 VM 層級的流量。 下列 NSG 已經存在:
  - 適用於 Active Directory 的 NSG
  - 工作負載的 NSG

每個 NSG 都會開放特定連接埠及通訊協定，讓解決方案可安全且正確地運作。 此外，以下設定會針對每個 NSG 啟用：
  - 啟用[診斷記錄和事件](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)並儲存在儲存體帳戶
  - OMS Log Analytics 會連線至 [NSG 的診斷](https://github.com/krnese/AzureDeploy/blob/master/AzureMgmt/AzureMonitor/nsgWithDiagnostics.json)

**子網路**：確認每個子網路都與對應的 NSG 相關聯。

### <a name="data-in-transit"></a>資料傳輸中
Azure 預設會加密與 Azure 資料中心的所有通訊。 透過 Azure 入口網站與 Azure 儲存體進行的所有交易都會透過 HTTPS 發生。

### <a name="data-at-rest"></a>待用資料

架構會透過加密、資料庫稽核及其他量值來保護待用資料。

**Azure 儲存體**為符合加密的待用資料需求，所有 [Azure 儲存體](https://azure.microsoft.com/services/storage/)都會使用[儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)。 這有助於保護與防衛個人資料，以支援組織的安全性承諾及 GDPR 所定義的合規性需求。

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
- [SQL 動態資料遮罩 (DDM)](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started) 可透過遮罩資料，使不具權限的使用者或應用程式無法看見敏感性個人資料。 DDM 可以自動探索潛在敏感性資料，並建議套用適當的遮罩。 這有助於識別符合 GDPR 保護資格的個人資料，並降低存取以確保這類資料不會因未經授權的存取而離開資料庫。 **注意：客戶必須調整 DDM 設定，以配合其資料庫結構描述。**

### <a name="identity-management"></a>身分識別管理
下列技術可在 Azure 環境中提供管理個人資料存取的功能：
-   [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) 是 Microsoft 的多租用戶雲端式目錄和身分識別管理服務。 此解決方案的所有使用者都是在 AAD 中建立，包括存取 Azure SQL Database 的使用者。
-   應用程式的驗證是使用 AAD 執行。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。 此外，資料庫資料行加密會使用 AAD 向 Azure SQL Database 驗證應用程式。 如需詳細資訊，請參閱如何[保護 SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
-   [Azure 角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure) 可讓系統管理員定義微調存取權限，只對使用者授與其執行工作所需的存取權數量。 系統管理員可以只允許存取個人資料的特定動作，而不是授與所有使用者不受限制的 Azure 資源使用權限。 只有訂用帳戶管理員可擁有訂用帳戶的存取權。
- [AAD Privileged Identity Management (PIM)](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started) 可讓客戶將可存取特定資訊 (例如個人資料) 的使用者人數降至最低。  系統管理員可以使用 AAD Privileged Identity Management 來探索、限制和監視特殊權限的身分識別和其對資源的存取。 如有需要，此功能也可用來強制執行隨選 Just-In-Time 系統管理存取權。
-   [AAD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測影響組織身分識別的潛在弱點，並為偵測到的組織身分識別相關可疑動作設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。

### <a name="security"></a>安全性
**祕密管理**解決方案會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 來管理金鑰和秘密。 Azure 金鑰保存庫可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。 下列 Azure Key Vault 功能可協助客戶保護個人資料及存取這類資料：
- 進階存取原則是視需要設定的。
- Key Vault 存取原則是使用金鑰和祕密的最低必要權限所定義的。
- Key Vault 中的所有金鑰和祕密都有到期日。
- Key Vault 中的所有金鑰都會受到特製化硬體安全模組 (HSM) 的保護。 金鑰類型為 HSM 保護的 2048 位元 RSA 金鑰。
- 所有使用者和身分識別皆使用 RBAC 授與最低必要權限。
- 金鑰保存庫的診斷記錄的保留期至少 365 天。
- 允許的金鑰密碼編譯作業僅限於需要的密碼編譯項目。

**安全性警訊**：[Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)可讓客戶監視流量、收集記錄及分析資料來源以防威脅。 此外，Azure 資訊安全中心會存取 Azure 服務的現有設定，以提供設定和服務建議，協助改善安全性狀態並保護個人資料。 Azure 資訊安全中心針對每個偵測到的威脅會包含[威脅情報報告](https://docs.microsoft.com/azure/security-center/security-center-threat-report)，以協助事件回應小組調查和修復威脅。

### <a name="logging-and-auditing"></a>記錄與稽核

[Operations Management Suite (OMS)](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) 提供系統、使用者活動及系統健康狀態的廣泛記錄。 OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) 解決方案可收集及分析 Azure 和內部部署環境中資源所產生的資料。
- **活動記錄**：[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您深入了解在訂用帳戶資源上執行的作業。 活動記錄可協助判斷作業的啟動器、出現時間和狀態。
- **診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)包含每個資源發出的所有記錄。 這些記錄包含 Windows 事件系統記錄、Azure Blob 儲存體、資料表和佇列記錄。
- **記錄封存**：所有診斷記錄都會寫入到集中且加密的 Azure 儲存體帳戶進行封存，並包含定義的保留期 (2 天)。 這些記錄會連線至 Azure Log Analytics 以進行處理、儲存及從儀表板顯示報表。

此外，此架構包含下列 OMS 解決方案：
-   [AD 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment)：Active Directory 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器基礎結構，提供優先的建議清單。
-   [惡意程式碼評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-malware)：反惡意程式碼軟體解決方案會報告惡意程式碼、威脅及保護狀態。
-   [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker)：Azure 自動化解決方案會儲存、執行和管理 Runbook。
-   [安全性與稽核](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started)：安全性和稽核儀表板會透過提供有關安全性網域、值得注意的問題、偵測、威脅情報和常見的安全性查詢等計量，來提供資源安全性狀態的整體見解。
-   [SQL 評定](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment)：SQL 健康情況檢查解決方案會定期評估伺服器環境的風險和健康情況，並專門針對部署的伺服器架構，提供優先的建議清單給客戶。
-   [更新管理](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management)：更新管理解決方案可讓客戶管理作業系統的安全性更新，包括可用更新的狀態和安裝必要更新的程序。
-   [代理程式健全狀況](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth)：代理程式健全狀況解決方案會報告部署的代理程式數目和其地理分佈，以及沒有回應的代理程式數目和正在提交作業資料的代理程式數目。
-   [Azure 活動記錄](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity)：活動記錄分析解決方案可協助您分析客戶所有 Azure 訂用帳戶的 Azure 活動記錄。
-   [變更追蹤](https://docs.microsoft.com/azure/automation/automation-change-tracking)：變更追蹤解決方案可讓客戶輕鬆地識別環境中的變更。

**Azure 監視器**
[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)透過讓組織稽核、建立警示及封存資料，包括追蹤客戶 Azure 資源中的 API 呼叫，協助客戶追蹤效能、維護安全性和識別趨勢。

**Application Insights**
[Application Insights](https://docs.microsoft.com/azure/application-insights/) 是多個平台上的 Web 開發人員所適用的可延伸「應用程式效能管理」(APM) 服務。 使用它來監視即時 Web 應用程式。 它會偵測效能異常，其中包括強大的分析工具可協助您診斷問題，並了解使用者實際上如何運用應用程式。 它是設計來協助使用者持續改善效能和可用性。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖可供[下載](https://aka.ms/gdprAnalyticsdfd)，或可以在以下位置找到。 此模型可協助客戶在進行修改時，了解系統基礎結構中的潛在風險要點。

![適用於 GDPR 的分析威脅模型](images/gdpr-analytics-threat-model.png?raw=true "適用於 GDPR 的分析威脅模型")

## <a name="compliance-documentation"></a>合規性文件
[Azure 安全性與合規性藍圖 - GDPR 客戶責任矩陣](https://aka.ms/gdprCRM)列出所有 GDPR 條款中的控制者和處理者責任。 請注意，在 Azure 服務中，客戶通常是控制者，而 Microsoft 則扮演處理者的角色。

[Azure 安全性與合規性藍圖 - GDPR 資料分析實作矩陣](https://aka.ms/gdprAnalytics)提供資料分析架構處理哪些 GDPR 條款的相關資訊，包括詳細說明此實作如何符合每條涵蓋條款的需求。


## <a name="guidance-and-recommendations"></a>指導方針與建議
### <a name="vpn-and-expressroute"></a>VPN 和 ExpressRoute
您需要設定安全的 VPN 通道或 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)，以安全地連線到部署為此資料分析參考架構一部分的資源。 透過適當地設定 VPN 或 ExpressRoute，客戶可以在傳輸過程中新增資料保護層。

透過實作與 Azure 的安全 VPN 通道，即可建立內部部署網路與 Azure 虛擬網路之間的虛擬私人連線。 此連線透過網際網路進行，可讓客戶在客戶的網路與 Azure 之間的加密連結內安全地「輸送」資訊。 站對站 VPN 是安全成熟的技術，各種規模的企業已部署數十年。 此選項使用 [IPsec 通道模式](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2003/cc786385(v=ws.10))作為加密機制。

由於 VPN 通道內的流量會透過站對站 VPN 周遊網際網路，因此 Microsoft 提供另一個更安全的連線選項。 Azure ExpressRoute 是 Azure 與內部部署位置或 Exchange 主機服務提供者之間專用的 WAN 連結。 ExpressRoute 連線不會經過網際網路，相較於一般網際網路連線，這些連線提供了更可靠、更快速、更低延遲、更安全的連線。 此外，因為這是客戶電信提供者的直接連線，所以資料不會透過網際網路傳輸，因此不會公開給網際網路。

如需實作將內部部署網路擴充至 Azure 之安全混合式網路的最佳做法，請參閱[這裡](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid)。

### <a name="extract-transform-load-etl-process"></a>擷取-轉換-載入 (ETL) 程序
[PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) 可以將資料載入 Azure SQL Database 中，而不需要個別的 ETL 或匯入工具。 PolyBase 允許透過 T-SQL 查詢來存取資料。 Microsoft 的商業智慧和分析堆疊，以及與 SQL Server 相容的協力廠商工具，都可以搭配 PolyBase 使用。

### <a name="azure-active-directory-setup"></a>Azure Active Directory 設定
您必須使用 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) 來管理部署，以及為與環境互動的人員佈建存取權。 您只要[點按四下](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-get-started-express)，就可整合現有 Windows Server Active Directory 與 AAD。 客戶也可以透過讓部署的 Active Directory 基礎結構成為 AAD 樹系的子網域，來將部署的 Active Directory 基礎結構 (網域控制站) 繫結至現有 AAD。

## <a name="disclaimer"></a>免責聲明

 - 此文件僅供參考之用。 Microsoft 對此文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 此文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
 - 此文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
 - 客戶可以複製並使用這份文件，供內部參考之用。
 - 此文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
 - 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
 - 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
