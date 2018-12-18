---
title: Azure 安全性與合規性藍圖 - 適用於 UK OFFICIAL 工作負載的 PaaS Web 應用程式裝載
description: Azure 安全性與合規性藍圖 - 適用於 UK OFFICIAL 工作負載的 PaaS Web 應用程式裝載
services: security
author: jomolesk
ms.assetid: 446105ad-a863-44f5-a964-6ead1dac4787
ms.service: security
ms.topic: article
ms.date: 07/13/2018
ms.author: jomolesk
ms.openlocfilehash: 6fe85d7ac527179ab39e89739f5744f3aa1ef8e2
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297550"
---
# <a name="azure-security-and-compliance-blueprint-paas-web-application-hosting-for-uk-official-workloads"></a>Azure 安全性與合規性藍圖：適用於 UK OFFICIAL 工作負載的 PaaS Web 應用程式裝載

## <a name="azure-security-and-compliance-blueprints"></a>Azure 安全性與合規性藍圖

Azure 藍圖是由指引文件和自動化範本所組成，可部署雲端式架構，以將解決方案提供給有認證或合規性需求的案例。 Azure 藍圖是指引和自動化範本集合，可讓 Microsoft Azure 客戶透過佈建可擴充來符合任何進一步需求的基礎架構，加速達成其商務目標。

## <a name="overview"></a>概觀

這篇 Azure 安全性與合規性藍圖提供指引和自動化指令碼，以提供 Microsoft Azure [平台即服務 (PaaS)](https://azure.microsoft.com/overview/what-is-paas/) 託管 Web 應用程式架構，該架構適合處理許多已歸類為 [UK OFFICIAL](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/715778/May-2018_Government-Security-Classifications-2.pdf) 的工作負載。 此安全性分類涵蓋了公共部門所建立或處理的大部分資訊。 這包括例行商務作業和服務，這些作業和服務若在媒體中遺失、遭竊或發佈，則可能會有不利的影響。 OFFICIAL 分類與私人企業的典型威脅設定檔大致相同，均可提供寶貴的資訊和服務。 UK OFFICIAL 預期需要保護英國政府的資料或服務，以免遭受能力和資源有限的攻擊者威脅或入侵，例如 (但不限於) 激進駭客、單一議題壓力團體、調查記者、有能力的個別駭客，以及大多數的犯罪個體和集團。

此藍圖會經由英國國家網路安 全中心 (National Cyber Security Centre，NCSC) 審閱並符合 NCSC 的 14 個雲端安全性準則。

此較購使用[平台即服務](https://azure.microsoft.com/overview/what-is-paas/)元件來提供一個環境，讓客戶無須購買軟體授權、管理基礎應用程式基礎結構，以及中介軟體或開發工具及其他資源，避免其所帶來的花費及複雜性。 客戶會管理他們所開發的應用程式和服務，並將焦點放在實現商業價值，而 Microsoft Azure 會管理其他 Azure 資源 (例如虛擬機器、儲存體和網路功能)，並將更多的基礎結構管理[責任分工](https://docs.microsoft.com/azure/security/security-paas-deployments#division-of-responsibility)加諸於 Azure 平台。 [Azure App Service](https://azure.microsoft.com/services/app-service/) 會提供自動調整、高度可用性，支援 Windows 和 Linux，並可從 GitHub、Azure DevOps 或任何 Git 存放庫作為預設服務來啟用自動化部署。 開發人員可以透過 App Service，專注於實現商業價值，而沒有管理基礎結構的額外負荷。 他們可以建置全新的 Java、PHP、Node.js、Python、HTML 或 C# Web 應用程式，或將現有的雲端或內部部署 Web 應用程式遷移至 Azure App Service (雖然需要徹底的實質審查和測試來確認效能)。

此藍圖著重於為公用以及後台系統使用者佈建安全的基礎[平台即服務](https://azure.microsoft.com/overview/what-is-paas/) Web 架構介面。 此藍圖設計案例會考慮使用 Azure 裝載 Web 架構服務，以便公開使用者安全地提交、檢視及管理敏感性資料；而且後台系統或政府操作員也可以安全地處理公用使用者所提交的敏感性資料。 此案例的使用案例包括：

- 提交退稅的使用者，以及處理提交的政府操作者；
- 透過 Web 架構應用程式要求服務的使用者，以及驗證和提供服務的後台系統使用者；或
- 搜尋和檢視政府服務相關公開網域說明資訊的使用者。

使用 [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) 範本和 Azure 命令列介面指令碼，此藍圖所部署的環境將符合英國國家網路安全中心 (NCSC) 的 14 個[雲端安全性準則](https://www.ncsc.gov.uk/guidance/implementing-cloud-security-principles) \(英文\)，以及網際網路安全中心 (CIS) 的 [重大安全性控制措施](https://www.cisecurity.org/critical-controls.cfm) \(英文\)。 NCSC 建議客戶使用他們的雲端安全性準則來評估服務的安全性屬性，並協助了解客戶與供應商之間的責任分工。 Microsoft 已針對每個準則提供相關資訊，以協助您更了解責任的劃分方式。 Microsoft 技術白皮書[使用 Microsoft Azure 對於英國雲端的 14 個安全性控制措施](https://gallery.technet.microsoft.com/14-Cloud-Security-Controls-670292c1) \(英文\) 支援此架構和對應的 Azure Resource Manager 範本。 此架構已經由 NCSC 審閱，並符合 UK NCSC 的 14 個雲端安全性準則，藉以讓公共部門組織能夠快速地符合在 Microsoft Azure 雲端上於全球及英國使用雲端式服務所衍生的合規性義務。 此範本會針對工作負載部署基礎結構。 客戶必須安裝和設定應用程式程式碼及支援的商務層和資料層軟體。 詳細的部署指示可於[這裡](https://aka.ms/ukofficial-paaswa-repo/) \(英文\) 取得。

此藍圖是基礎架構。 我們的客戶可以使用此藍圖作為其 OFFICIAL 分類 Web 架構工作負載的基礎，並隨著他們自己的需求來擴充範本和資源。 此藍圖是以 [UK-OFFICA 三層式 IaaS Web 應用程式藍圖](https://aka.ms/ukofficial-iaaswa)的準則為基礎，可為我們的客戶提供[基礎結構即服務 (IaaS)](https://azure.microsoft.com/overview/what-is-iaas/) 和 PaaS 實作選項以便裝載 Web 架構工作負載。

若要部署此藍圖，則需要有 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，則可快速且輕鬆地免費進行註冊：開始使用 Azure。 如需部署指示，請按一下[這裡](https://aka.ms/ukofficial-paaswa-repo/) \(英文\)。

## <a name="architecture-and-components"></a>架構與元件

此藍圖能在支援 UK OFFICIAL 工作負載的 Azure 雲端環境中，提供 Web 應用程式裝載解決方案。 此架構提供可利用 Azure 平台即服務功能的安全環境。 在此環境中，已部署兩個 App Service Web 應用程式 (一個用於公用使用者，一個用於後台系統使用者)，其具有 API 應用程式層可為 Web 前端提供商務服務。 Azure SQL Database 已部署為應用程式的受控關聯式資料存放區。 從外部平台連線這些元件的能力以及這些元件之間的連線能力都會透過 TLS 1.2 加密，以確保資料傳輸中隱私權，且具有經由 Azure Active Directory 驗證的存取權。

![適用於 UK OFFICIAL 工作負載的 PaaS Web 應用程式裝載參考架構圖表](images/ukofficial-paaswa-architecture.png?raw=true "適用於 UK OFFICIAL 工作負載的 PaaS Web 應用程式裝載參考架構圖表")

在部署架構中，也會部署安全的儲存體佈建、監視和記錄、統一安全性管理和進階威脅防護及管理功能，確保客戶擁有針對此解決方案保護及監視其環境所需的工具。

此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。

- Azure Active Directory
- 受控服務識別
- App Service 方案
- Web 應用程式
- API 應用程式
- Azure DNS
- Key Vault
- Azure 監視器
- Application Insights
- Log Analytics
- Azure Resource Manager
- Azure 資訊安全中心
- 連接字串
- Azure 儲存體

## <a name="deployment-architecture"></a>部署架構

下一節會詳細說明部署和實作元素。

### <a name="security"></a>安全性

#### <a name="identity-and-authentication"></a>身分識別和驗證

此藍圖可確保透過目錄和身分識別管理服務來保護資源的存取權。 此架構會充分利用[身分識別作為安全界限](https://docs.microsoft.com/azure/security/security-paas-deployments#identity-as-the-primary-security-perimeter)。 

下列技術可在 Azure 環境中提供身分識別管理功能：

- [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/) 是 Microsoft 的多租用戶雲端式目錄和身分識別管理服務。 解決方案的所有使用者都是在 Azure Active Directory 中建立，包括存取 SQL Database 的使用者。
- 使用 Azure AD 可執行操作者面向 Web 應用程式的驗證和 Azure 資源管理的存取。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)。
- 資料庫資料行加密會使用 Azure AD 向 Azure SQL Database 驗證應用程式。 如需詳細資訊，請參閱 [Always Encrypted：保護 SQL Database 中的敏感性資料](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)。
- 公民面向 Web 應用程式會設定為公開存取。 若要允許透過 Active Directory 或社交網路身分識別提供者來建立及驗證帳戶，[Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) 可以視需要進行整合。
- [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 會偵測潛在弱點和有風險的帳戶，並提供相關建議，以增強貴組織身分識別的安全性狀態，並為偵測到的組織身分識別相關可疑活動設定自動回應，以及調查可疑事件並採取適當動作來解決這些可疑事件。
- [Azure 角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal) 可以對 Azure 進行精確且專注的存取權管理。 可存取訂用帳戶的身分會限制為訂用帳戶系統管理員，而可存取 Azure Key Vault 的身分則是限制為需要金鑰管理權限的使用者。
- 透過運用 [Azure Active Directory 條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)，客戶可以根據特定條件 (例如位置、裝置、狀態和登入風險)，對其環境中的應用程式或使用者存取強制使用其他安全性控制項。
- 結合應用程式設計最佳做法的 [Azure DDoS 保護](https://docs.microsoft.com/azure/security/security-paas-deployments#security-advantages-of-a-paas-cloud-service-model)，可提供 DDoS 攻擊的防禦，包含永遠可用的流量監視，以及即時移轉一般網路層級的攻擊。 使用 PaaS 架構時，平台層級 DDoS 保護對客戶而言是透明的並已併入平台中，但請務必請注意，客戶需負起應用程式安全性設計責任。

#### <a name="data-in-transit"></a>資料傳輸中

使用[傳輸層安全性/安全通訊端層 (TLS/SSL)](https://www.microsoft.com/TrustCenter/Security/Encryption) 可保護從外部傳輸以及在 Azure 元件之間傳輸的資料，其使用以共用密碼為基礎的對稱式密碼編譯來加密通訊透過網路傳輸的通訊。 根據預設，使用 TLS 1.2 來保護網路流量。

#### <a name="security-and-malware-protection"></a>安全性和惡意程式碼防護

[Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)可提供所有 Azure 資源安全性狀態的集中檢閱。 只需看一眼，您就可以確認安全性控制項是否已就緒並正確設定，且可以快速找出任何需要注意的資源。

[Azure Advisor](https://docs.microsoft.com/azure/advisor/advisor-overview) 是個人化的雲端顧問，可協助您依最佳做法來最佳化您的 Azure 部署。 它可分析您的資源組態和使用量遙測，然後建議可協助您改善 Azure 資源的成本效益、效能、高可用性和安全性的解決方案。

[Microsoft Antimalware](https://docs.microsoft.com/azure/security/azure-security-antimalware) 是即時保護功能，有助於識別和移除病毒、間諜軟體和其他惡意軟體。 預設安裝於基礎 PaaS 虛擬機器基礎結構，並由 Azure 網狀架構以透明方式對客戶進行管理。

### <a name="paas-services-in-this-blueprint"></a>此藍圖中的 PaaS 服務

#### <a name="azure-app-service"></a>Azure App Service

Azure Web Apps 可為以 Java、PHP、Node.js Python、HTML 和 C# 開發的 Web 應用程式，提供完全受控的 Web 裝載環境，而不必管理基礎結構。 它提供自動調整及高度可用性，支援 Windows 和 Linux，並可從 [Azure DevOps](https://azure.microsoft.com/services/visual-studio-team-services/)或任何 Git 存放庫啟用自動部署。

App Service 符合 [ISO、SOC 和 PCI 規範](https://www.microsoft.com/TrustCenter/)並可使用 [Azure Active Directory](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-active-directory-authentication) 或社交登入 ([Google](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-google-authentication)、[Facebook](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-facebook-authentication)、[Twitter](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-twitter-authentication) 和 [Microsoft 驗證](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-microsoft-authentication)來驗證使用者。

基本、標準和進階方案適用於生產工作負載，並會在專用虛擬機器執行個體上執行。 每個執行個體均可支援多個應用程式和網域。 App services 也支援 [IP 位址限制](https://docs.microsoft.com/azure/app-service/app-service-ip-restrictions)來保護受信任的 IP 位址流量 (如有需要)，也支援[受控服務識別](https://docs.microsoft.com/azure/app-service/app-service-managed-service-identity)以便安全連線至其他 PaaS 服務，例如 [Key Vault](https://azure.microsoft.com/services/key-vault/) 和 [Azure SQL Database](https://azure.microsoft.com/services/sql-database/)。 如果需要額外的安全性，隔離式方案會在私人專用 Azure 環境中裝載您的應用程式，適合需要與內部部署網路安全連線或額外效能和規模的應用程式使用。

此範本會部署下列 App Service 實體：

- [標準](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview) App Service 方案層次
- 多個 Web App [部署位置](https://docs.microsoft.com/azure/app-service/web-sites-staged-publishing)：Dev、Preview、QA、UAT 以及 Production (預設位置)。
- [受控服務識別](https://docs.microsoft.com/azure/app-service/app-service-managed-service-identity)以連線到 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) (這也可供存取 [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 
- 與 [Azure Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-azure-web-apps) 整合以監視效能
- [診斷記錄檔](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs) 
- 度量[警示](https://docs.microsoft.com/azure/application-insights/app-insights-alerts) 
- [Azure API 應用程式](https://azure.microsoft.com/services/app-service/api/) 

#### <a name="azure-sql-database"></a>連接字串

SQL Database 是 Microsoft Azure 中的一般用途關聯式資料庫受控服務，可支援關聯式資料、JSON、空間和 XML 等結構。 SQL Database 提供受控的單一 SQL 資料庫、[彈性集區](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-pool)中的受控 SQL 資料庫，以及 SQL [受控執行個體](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance) (處於公開預覽狀態)。 除了實現 [可動態調整的效能] https://docs.microsoft.com/azure/sql-database/sql-database-service-tiers)外，更能提供各種選項，例如用於極限分析和報告的[資料行存放區索引](https://docs.microsoft.com/sql/relational-databases/indexes/columnstore-indexes-overview)，以及用來處理極限交易的[記憶體內部 OLTP](https://docs.microsoft.com/azure/sql-database/sql-database-in-memory)。 Microsoft 可順暢地處理 SQL 程式碼基底的所有修補和更新，並抽走基礎結構的所有管理功能。

此藍圖中的 Azure SQL Database

Azure SQL Database 執行個體會使用下列資料庫安全性量值：

- [伺服器層級和資料庫層級防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，或透過使用[虛擬網路規則](https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview)的[虛擬網路服務端點](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)。
- [透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)有助於保護 Azure SQL Database 和 Azure 資料倉儲來抵禦惡意活動的威脅。 它會對資料庫、相關聯的備份和待用的交易記錄檔執行即時加密和解密，而不需變更應用程式。
- 您可以使用 [Azure AD 驗證](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)，在單一中央位置集中管理資料庫使用者和其他 Microsoft 服務的身分識別。 中央識別碼管理提供單一位置以管理資料庫使用者並簡化權限管理。
- 使用 Azure Active Directory 進行資料庫管理
- [稽核記錄](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)至儲存體帳戶
- 失敗 DB 連線的計量[警示](https://docs.microsoft.com/azure/application-insights/app-insights-alerts)
- [SQL 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection)
- [Always Encrypted 資料行](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault)

### <a name="azure-storage"></a>Azure 儲存體

Microsoft [Azure 儲存體](https://azure.microsoft.com/services/storage/)是 Microsoft 管理的雲端服務，可提供高度可用、安全、持久、可擴充和備援的儲存體。 Azure 儲存體包含 Blob 儲存體、檔案儲存體和佇列儲存體。

#### <a name="azure-storage-in-this-blueprint"></a>此藍圖中的 Azure 儲存體

此範本使用下列 Azure 儲存體元件：

- [儲存體服務加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption) 
- 只允許 HTTPS 連線

#### <a name="data-at-rest"></a>待用資料

透過[儲存體服務加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)，所有寫入至 Azure 儲存體的資料都會透過 256 位元 AES 加密 (可用的最強大區塊編碼器之一) 進行加密。 您可以搭配 SSE 使用 Microsoft 管理的加密金鑰，或使用[自己的加密金鑰](https://docs.microsoft.com/azure/storage/common/storage-service-encryption-customer-managed-keys)。

可透過使用[虛擬網路規則](https://docs.microsoft.com/azure/storage/common/storage-network-security)的[虛擬網路服務端點](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)來保護儲存體帳戶。

在[安全性指南](https://docs.microsoft.com/azure/storage/common/storage-security-guide)中可找到保護 Azure 儲存體的詳細資訊。


### <a name="secrets-management"></a>祕密管理

#### <a name="azure-key-vault"></a>Azure 金鑰保存庫

[Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview) 用來保護應用程式金鑰和祕密，以確保第三方不能存取它們。 Key Vault 的用意並非作為使用者密碼的存放區。 它可讓您建立多個安全的容器，稱之為保存庫。 這些保存庫由硬體安全性模組 (HSM) 支援。 保存庫藉由集中儲存應用程式祕密，協助減少意外遺失安全性資訊的機會。 Key Vault 也會控制和記錄其中所儲存項目的存取權。 Azure Key Vault 可以處理要求及更新傳輸層安全性 (TLS) 憑證，並提供穩健憑證生命週期管理解決方案所需的功能。

#### <a name="azure-key-vault-in-this-blueprint"></a>此藍圖中的 Azure Key Vault

- 保留儲存體存取金鑰，並將讀取權限授與給客戶面向 Web 應用程式的[受控服務識別](https://docs.microsoft.com/azure/app-service/app-service-managed-service-identity)
- 保留 SQL Server DBA 密碼 (在個別的保存庫中)
- 診斷記錄

### <a name="monitoring-logging-and-audit"></a>監視、記錄和稽核

#### <a name="log-analytics"></a>Log Analytics

[Log Analytics](https://azure.microsoft.com/services/log-analytics/) 是 Azure 中的一項服務，可協助您收集和分析雲端和內部部署環境中的資源所產生的資料。

#### <a name="log-analytics-in-this-blueprint"></a>此藍圖中的 Log Analytics

- SQL 評估
- Key Vault 診斷
- Application Insights 連線
- Azure 活動記錄檔

#### <a name="application-insights"></a>Application Insights

[Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-overview) 是多個平台上的 Web 開發人員所適用的可延伸應用程式效能管理 (APM) 服務。 用來監視即時 Web 應用程式，它會自動偵測效能異常、 析效能、診斷問題，以及了解使用者與應用程式的互動方式。 Application Insights 可以部署在各種平台上，包括裝載在內部部署環境或雲端的 Node.js 和 J2EE。 它可與您的 DevOps 程序整合，並有各種開發工具的連接點。

#### <a name="application-insights-in-this-blueprint"></a>此藍圖中的 Application Insights

此範本使用下列 Application Insights 元件：

- 每個網站的 Application Insights 儀表板 (操作員、客戶和 API)

#### <a name="azure-activity-logs"></a>Azure 活動記錄檔

[Azure 活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs#what-you-can-do-with-the-activity-log)可稽核訂用帳戶的控制程度事件。 您可以使用活動記錄檔來判斷訂用帳戶中的資源上任何寫入作業 (PUT、POST、DELETE) 的「內容、對象和時間」。 您也可以了解作業的狀態和其他相關屬性。

#### <a name="azure-monitor"></a>Azure 監視器

[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor)會透過允許集合計量、活動記錄及診斷記錄，來啟用 Azure 服務的核心監視功能。 Azure 監視器可針對 Microsoft Azure 中的大多數服務提供基本等級的基礎結構計量與記錄。

## <a name="threat-model"></a>威脅模型

此參考架構的資料流程圖可供[下載](https://aka.ms/ukofficial-paaswa-tm)，或可以在以下位置找到。 此模型可協助客戶在進行修改時，了解系統基礎結構中的潛在風險要點。

![適用於 UK OFFICIAL 工作負載的 PaaS Web 應用程式裝載威脅模型](images/ukofficial-paaswa-threat-model.png?raw=true "適用於 UK OFFICIAL 工作負載的 PaaS Web 應用程式裝載威脅模型")

## <a name="ncsc-cloud-security-principles-compliance-documentation"></a>NCSC 雲端安全性準則合規性文件

Crown Commercial Service (一所致力於改善政府相關商業和採購活動的機構) 已將 Microsoft 範圍內企業雲端服務的分類更新為 G-Cloud v6，以涵蓋其 OFFICIAL 層級的所有供應項目。 Azure 與 G-Cloud 的詳細資料可在 [Azure 英國 G-Cloud 安全性評定摘要](https://www.microsoft.com/trustcenter/compliance/uk-g-cloud) \(英文\) 上取得。

此藍圖符合 NCSC [雲端安全性準則](https://www.ncsc.gov.uk/guidance/implementing-cloud-security-principles) \(英文\) 中所記載的 14 個雲端安全性準則，以協助確保環境能支援已分類為 UK OFFICIAL (英國官方) 的工作負載。

[Azure 安全性與合規性藍圖 - UK OFFICIAL 客戶責任矩陣](https://aka.ms/ukofficial-crm) \(英文\) (Excel 活頁簿) 會將 14 個雲端安全性準則全部列出，而且會針對每個準則 (或準則的子項目)，表示準則的實作應該是由 Microsoft 或客戶負責，還是由兩者共同承擔。

[Azure 安全性與合規性藍圖 - 適用於 UK OFFICIAL 的 PaaS Web 應用程式準則實作矩陣](https://aka.ms/ukofficial-paaswa-pim) \(英文\) (Excel 活頁簿) 會將 14 個雲端安全性準則全部列出，而且會針對已在客戶責任矩陣中指定為客戶責任的每個準則 (或準則的子項目)，表示 1) 藍圖是否實作該準則，以及 2) 實作如何符合準則需求的說明。  

此外，雲端安全聯盟 (CSA) 已發佈雲端控制矩陣，以支援正在評估雲端提供者的客戶，以及識別出應在移至雲端服務之前先獲得解答的問題。 作為回應，Microsoft Azure 已回答 CSA 共識評量行動問卷 ([CSA CAIQ](https://www.microsoft.com/TrustCenter/Compliance/CSA) \(英文\))，其中描述 Microsoft 因應建議準則的方式。

## <a name="third-party-assessment"></a>第三方組件

此藍圖會經由英國國家網路安 全中心 (National Cyber Security Centre，NCSC) 審閱並符合 NCSC 的 14 個雲端安全性準則

自動化範本已經由英國 Customer Success Unit Azure Cloud Solution Architect 小組和 Microsoft 合作夥伴 [Ampliphae](http://www.ampliphae.com/) 測試。


## <a name="deploy-the-solution"></a>部署解決方案

此「Azure 安全性與合規性藍圖自動化」包含 JSON 設定檔和 PowerShell 指令碼，它們會由 Azure Resource Manager 的 API 服務來處理，以在 Azure 中部署資源。 詳細的部署指示可於[這裡](https://aka.ms/ukofficial-paaswa-repo) \(英文\) 取得。

提供了三種部署方法：簡單「快速」的 [Azure CLI 2](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 適用於快速建置測試環境；參數化 [Azure CLI 2](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 方法可提供更高的工作負載環境組態；以及以 Azure 入口網站為基礎的部署，操作員可透過 Azure 入口網站指定部署參數。 

1.  將[這個](https://aka.ms/ukofficial-paaswa-repo) GitHub 存放庫複製或下載到您的本機工作站。
2.  檢閱[方法 1：Azure CLI 2 (Express 版)](https://aka.ms/ukofficial-paaswa-repo/#method-1-azure-cli-2-express-version) 和執行所提供的命令。
3.  檢閱[方法 1a：Azure CLI 2 (透過指令碼引數設定部署)](https://aka.ms/ukofficial-paaswa-repo/#method-1a-azure-cli-2-configuring-the-deployment-via-script-arguments) 和執行所提供的命令
4.  檢閱[方法 2：Azure 入口網站開發程序](https://aka.ms/ukofficial-paaswa-repo/#method-2-azure-portal-deployment-process)和執行所列的命令

## <a name="guidance-and-recommendations"></a>指引與建議

### <a name="api-management"></a>API 管理

[Azure API 管理](https://azure.microsoft.com/services/api-management/)可使用於 API App Service 之前，以提供額外的安全性、節流和控制項層次來公開、代理及保護 API。

### <a name="azure-b2c"></a>Azure B2C

[Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) 可當作控制項實作，以允許使用者註冊、建立身分識別，以及啟用公用 Web 應用程式的授權和存取控制項。

## <a name="disclaimer"></a>免責聲明

- 此文件僅供參考之用。 Microsoft 對本文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 本文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
- 本文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
- 客戶可以複製並使用這份文件，供內部參考之用。
- 本文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
- 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
- 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
