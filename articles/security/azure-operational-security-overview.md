---
title: Azure 作業安全性概觀 | Microsoft Docs
description: 本文提供 Azure 作業安全性概觀。
services: security
documentationcenter: na
author: unifycloud
manager: mbaldwin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: tomsh
ms.openlocfilehash: 706e946e61f7d5f48ce5071e90d724af717449d9
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45575213"
---
# <a name="azure-operational-security-overview"></a>Azure 作業安全性概觀
[Azure 作業安全性](https://docs.microsoft.com/azure/security/azure-operational-security)是指使用者可在 Microsoft Azure 中用來保護其資料、應用程式和其他資產的服務、控制措施與功能。 它是一個架構，其中包含透過各種 Microsoft 特有功能所獲得的知識。 這些功能包括 Microsoft 安全性開發週期 (SDL)、Microsoft Security Response Center 程式，以及對網路安全性威脅環境的深層認知。

## <a name="operations-management-suite"></a>Operations Management Suite
IT 作業小組負責管理資料中心基礎結構、應用程式和資料，包括這些系統的穩定性與安全性。 不過，要在日趨複雜的 IT 環境中取得安全性深入解析，通常需要組織將多個安全性和管理系統的資料加以併用。

[Microsoft Operations Management Suite](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-overview) 是雲端式 IT 管理解決方案，可協助您管理及保護內部部署和雲端基礎結構。 它的核心功能是由在 Azure 中執行的下列服務所提供的。 每個服務都提供特定的管理功能。 您可以結合服務，以達成不同的管理案例。 

### <a name="log-analytics"></a>Log Analytics
[Azure Log Analytics](http://azure.microsoft.com/documentation/services/log-analytics) 透過將受控資源中的資料收集到中央存放庫，來提供 Operations Management Suite 的監視服務。 此資料可以包括事件、效能資料，或透過 API 提供的自訂資料。 收集資料之後，即可將其用於警示、分析和匯出。 

您可以合併各種來源的資料，並結合來自 Azure 服務與現有內部部署環境的資料。 Log Analytics 也可清楚劃分資料的收集與對該資料採取的動作，讓所有類型的資料均可使用所有動作。

### <a name="automation"></a>自動化
[Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)提供方法，讓您將通常會在雲端和企業環境中執行的手動、長時間執行、易發生錯誤且重複性高的工作加以自動化。 它可以節省時間並提高管理工作的可靠性。 甚至還能為這些工作設定排程以定期自動執行。 您可以使用 Runbook 來將程序自動化，或使用 Desired State Configuration 來將設定管理自動化。

### <a name="backup"></a>Backup 
[Azure 備份](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup)是以 Azure 為基礎的服務，可用來備份 (或保護) 和還原 Microsoft Cloud 中的資料。 Azure 備份會以一個可靠、安全且具成本競爭力的雲端式解決方案，來取代您的現有內部部署或異地備份解決方案。 

Azure 備份提供元件，讓您可以下載並部署於適當的電腦或伺服器上，或是雲端中。 您部署的元件或代理程式，取決於您想要保護的項目。 所有 Azure 備份元件 (無論您要保護的是內部部署或雲端中的資料) 都可用來將資料備份至 Azure 中的 Azure 復原服務保存庫。 

如需詳細資訊，請參閱 [Azure 備份元件表格](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup#which-azure-backup-components-should-i-use)。

### <a name="site-recovery"></a>Site Recovery
[Azure Site Recovery](http://azure.microsoft.com/documentation/services/site-recovery) 透過協調內部部署虛擬和實體機器至 Azure 或次要網站的複寫，來提供商務持續性。 如果主要網站無法使用，您會容錯移轉到次要位置，讓使用者可以繼續工作。 您會在系統回到正常運作時進行容錯回復。 使用 Azure 資訊安全中心來執行更聰明且有效的威脅偵測。

## <a name="azure-active-directory"></a>Azure Active Directory
[Azure Active Directory (Azure AD)](https://docs.microsoft.com/azure/active-directory/active-directory-enable-sso-scenario) 是全方位的身分識別服務：

-   啟用身分識別與存取管理 (IAM) 作為雲端服務。
-   提供集中式存取管理、單一登入 (SSO) 與報告功能。
-   支援 Azure Marketplace 中[數千個應用程式](https://azure.microsoft.com/marketplace/active-directory/) (包括 Salesforce、Google Apps、Box 與 Concur) 的整合式存取管理。

Azure AD 還隨附一套完整的[身分識別管理功能](https://docs.microsoft.com/azure/security/security-identity-management-overview#security-monitoring-alerts-and-machine-learning-based-reports)，其中包括：

- [Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication)
- [自助密碼管理](https://azure.microsoft.com/resources/videos/self-service-password-reset-azure-ad/)
- [自助式群組管理](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-update-your-own-password)
- [具有特殊權限的帳戶管理](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure)
- [角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/overview)
- [應用程式使用情況監視](https://docs.microsoft.com/azure/active-directory/connect-health/active-directory-aadconnect-health)
- [豐富的稽核功能](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-activity-audit-logs)
- [安全性監視和警示](https://docs.microsoft.com/azure/operations-management-suite/oms-security-responding-alerts)

使用 Azure Active Directory 時，您為合作夥伴和客戶 (企業或消費者) 發佈的所有應用程式都可享有相同的身分識別與存取管理功能。 這可讓您大幅降低營運成本。

## <a name="azure-security-center"></a>Azure 資訊安全中心
[Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)有助於您利用加強對 Azure 資源安全性的可見度 (和控制權)，以預防、偵測並回應威脅。 它提供您訂用帳戶之間的整合式安全性監視和原則管理功能。 它有助於偵測可能會被忽視的威脅，並可搭配廣泛的安全性解決方案生態系統使用。

透過提供虛擬機器的安全性設定可見度並監視威脅，在 Azure 中[保護虛擬機器 (VM) 資料](https://docs.microsoft.com/azure/security-center/security-center-linux-virtual-machine)。 資訊安全中心可以監視虛擬機器以取得︰

-   含建議設定規則的作業系統安全性設定。
-   遺失的系統安全性和重大更新。
-   端點保護建議。
-   磁碟加密驗證。
-   網路型攻擊。

資訊安全中心會使用[角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)。 RBAC 會提供[內建角色](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)，可指派給 Azure 中的使用者、群組與服務。

資訊安全中心會評估資源的組態，以識別安全性問題與弱點。 在資訊安全中心，只有當您獲指派為資源所屬訂用帳戶或資源群組的擁有者、參與者或讀取者角色時，才能看到與資源相關的資訊。

>[!Note]
>如需角色與資訊安全中心所允許動作的詳細資訊，請參閱 [Azure 資訊安全中心的權限](https://docs.microsoft.com/azure/security-center/security-center-permissions)。

資訊安全中心會使用 Microsoft Monitoring Agent。 這是與 Operations Management Suite 和 Log Analytics 服務所使用代理程式相同的代理程式。 從這個代理程式收集的資料會儲存在與您 Azure 訂用帳戶相關聯的現有 Log Analytics [工作區](https://docs.microsoft.com/azure/log-analytics/log-analytics-manage-access)或新的工作區中 (將 VM 的地理位置納入考量)。

## <a name="azure-monitor"></a>Azure 監視器
您雲端應用程式中的效能問題可能會對企業產生影響。 透過多個互連的元件和頻繁的發行，隨時都可能導致效能降低。 此外，如果您正在開發應用程式，您的使用者通常會探索到您在測試時未發現的問題。 您應該立即知道這些問題，並應具備可用以診斷和修正問題的工具。

[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor)是用以監視 Azure 上執行之服務的基本工具。 它會提供關於服務輸送量及周遭環境之基礎結構等級的資料。 如果您在 Azure 中管理所有應用程式，並決定要相應增加或減少資源，Azure 監視器就是起點。

您還可以使用監視資料來取得應用程式的深入見解。 這些知識可協助您提升應用程式效能或維護性，或是將原本需要手動介入的動作自動化。 

Azure 監視器包括下列元件。

### <a name="azure-activity-log"></a>Azure 活動記錄檔
[Azure 活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)能讓您了解在訂用帳戶中資源上執行的作業。 它先前稱為「稽核記錄」或「作業記錄」，因為它會回報訂用帳戶中控制層面的事件。

### <a name="azure-diagnostic-logs"></a>Azure 診斷記錄
[Azure 診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)是由資源發出，並提供關於該資源作業的豐富、經常性資料。 這些記錄的內容會依資源類型而有所不同。

Windows 事件系統記錄是適用於 VM 的診斷記錄類別之一。 Blob、資料表和佇列記錄均為適用於儲存體帳戶的診斷記錄類別。

診斷記錄與[活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)不同。 活動記錄能讓您了解訂用帳戶中的資源所執行之作業。 診斷記錄能讓您了解您的資源自行執行的作業。

### <a name="metrics"></a>度量
Azure 監視器提供遙測功能，讓您可以在 Azure 上查看工作負載的效能與健康情況。 Azure 遙測資料最重要的類型是由大多數 Azure 資源所發出的[計量](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics) (也稱為效能計數器)。 Azure 監視器提供數種方式可設定及取用這些度量進行監視與疑難排解。

### <a name="azure-diagnostics"></a>Azure 診斷
Azure 診斷會啟用為部署的應用程式收集診斷資料的功能。 您可以使用來自各種來源的診斷延伸模組。 目前支援的項目包括 [Azure 雲端服務角色](https://docs.microsoft.com/azure/vs-azure-tools-configure-roles-for-cloud-service)、執行 Microsoft Windows 的 [Azure 虛擬機器](https://docs.microsoft.com/azure/vs-azure-tools-configure-roles-for-cloud-service)，以及 [Azure Service Fabric](https://docs.microsoft.com/azure/monitoring-and-diagnostics/azure-diagnostics)。


## <a name="azure-network-watcher"></a>Azure 網路監看員
客戶可以透過協調和組合個別網路資源 (例如虛擬網路、Azure ExpressRoute、Azure 應用程式閘道及負載平衡器)，在 Azure 中建置端對端網路。 每個網路資源都可進行監視。

端對端網路可具備複雜的設定並在資源之間進行互動。 結果會產生需要透過 [Azure 網路監看員](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)進行案例型監視的複雜案例。

網路監看員可簡化 Azure 網路的監視和診斷程序。 您可以在網路監看員中使用診斷和視覺效果工具來：
- 在 Azure 虛擬機器上進行遠端封包擷取。
- 使用流量記錄獲取對網路流量的見解。
- 診斷 Azure VPN 閘道和連線。

網路監看員目前具有下列功能︰

- [拓撲](https://docs.microsoft.com/azure/network-watcher/network-watcher-topology-overview)：提供資源群組中網路資源之間各種互相連線和關聯的檢視。
-   [變數封包擷取](https://docs.microsoft.com/azure/network-watcher/network-watcher-packet-capture-overview)：擷取進出虛擬機器的封包資料。 進階篩選選項和微調控制項 (例如能夠設定時間和大小限制) 可讓您靈活擷取資料。 封包資料可以 .cap 格式儲存在 Blob 存放區或本機磁碟上。
-   [IP 流量驗證](https://docs.microsoft.com/azure/network-watcher/network-watcher-ip-flow-verify-overview)：根據流量資訊的 5-Tuple 封包參數 (目的地 IP、來源 IP、目的地連接埠、來源連接埠和通訊協定) 檢查允許或拒絕封包。 如果安全性群組拒絕了封包，則會傳回拒絕封包的規則和群組。
-   [下一個躍點](https://docs.microsoft.com/azure/network-watcher/network-watcher-next-hop-overview)：決定在 Azure 網路網狀架構中路由傳送封包的下一個躍點，讓您可以診斷任何設定錯誤的使用者定義路由。
-   [安全性群組檢視](https://docs.microsoft.com/azure/network-watcher/network-watcher-security-group-view-overview)：取得要在 VM 上套用的有效和已套用安全性規則。
-   [網路安全性群組的 NSG 流量記錄](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview)：可讓您擷取群組中安全性規則所允許或拒絕流量的相關記錄。 流量是由 5-Tuple 資訊所定義的：來源 IP、目的地 IP、來源連接埠、目的地連接埠和通訊協定。
-   [虛擬網路閘道和連線疑難排解](https://docs.microsoft.com/azure/network-watcher/network-watcher-troubleshoot-manage-rest)：提供針對虛擬網路閘道和連線進行疑難排解的功能。
-   [網路訂用帳戶限制](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)：可讓您根據限制檢視網路資源使用狀況。
-   [診斷記錄](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)：提供單一窗格以啟用或停用資源群組中網路資源的診斷記錄。

如需詳細資訊，請參閱[設定網路監看員](https://docs.microsoft.com/azure/network-watcher/network-watcher-create)。

## <a name="devops"></a>DevOps
在[開發人員作業 (DevOps)](https://www.visualstudio.com/learn/what-is-devops/) \(英文\) 應用程式開發之前，小組會負責收集軟體程式的商務需求，並撰寫程式碼。 接著，個別的 QA 小組會在隔離的開發環境中測試程式。 如果已符合需求，QA 小組就會針對要部署的作業發行程式碼。 開發小組已進一步分割成諸如網路功能與資料庫等群組。 每次將軟體程式「扔過牆」給獨立的小組時，就會加上一個瓶頸。

DevOps 可讓小組以更快速且成本較低的方式提供更安全、更高品質的解決方案。 使用軟體與服務時，客戶預期能獲得動態且可靠的經驗。 小組必須快速地逐一查看軟體更新，並衡量更新的影響。 他們必須使用新的開發反覆項目迅速回應，以解決問題或提供更多價值。  

諸如 Microsoft Azure 的雲端平台已移除傳統的瓶頸，並協助基礎結構獲得更高價值。 在每一項業務中，都是以軟體支配作為業務成果的主要區分及要素。 沒有任何組織、開發人員或 IT 工作者可以或應該避免 DevOps 行動。

成熟的 DevOps 從業人員會採用下列幾種做法。 這些做法[需要人員](https://www.visualstudio.com/learn/what-is-devops-culture/)以業務情節作為基礎來制定策略。 工具可協助將各種作法自動化。

-   [靈活的規劃與專案管理](https://www.visualstudio.com/learn/what-is-agile/)技術，可用來規劃工作並加以隔離為短期衝刺、管理小組產能，並協助小組快速地適應瞬息萬變的業務需求。
-   [版本控制，通常搭配 Git](https://www.visualstudio.com/learn/what-is-git/)，可讓位於全球各地的小組共用來源，並與軟體開發工具整合，從而將發行管線自動化。
-   [持續整合](https://www.visualstudio.com/learn/what-is-continuous-integration/) \(英文\) 會驅動進行中的合併和程式碼測試，以便及早找出缺失。  其他優點包括減少對抗合併問題的時間，以及開發小組的快速意見反應。
-   [持續傳遞](https://www.visualstudio.com/learn/what-is-continuous-delivery/) \(英文\) 軟體解決方案到生產環境與測試環境，有助於組織快速修正錯誤 (Bug)，並回應日益變遷的商業需求。
-   [監視](https://www.visualstudio.com/learn/what-is-monitoring/) \(英文\) 執行中的應用程式 (包括監視生產環境以了解應用程式健康情況，以及監視客戶的使用狀況) 能協助組織產生假設並快速驗證或反駁策略。  擷取豐富的資料並以各種記錄格式加以儲存。
-   [基礎結構即程式碼 (IaC)](https://www.visualstudio.com/learn/what-is-infrastructure-as-code/) \(英文\) 是一種作法，可用來將建立及卸除網路和虛擬機器的程序加以自動化並驗證，以協助提供安全且穩定的應用程式裝載平台。
-   [微服務](https://www.visualstudio.com/learn/what-are-microservices/) \(英文\) 架構可用來將商務使用案例隔離為小型的可重複使用服務。  此架構能提升延展性和效率。

## <a name="next-steps"></a>後續步驟
若要了解 Operations Management Suite 安全性與稽核解決方案，請參閱下列文章：

- [安全性與合規性](https://www.microsoft.com/cloud-platform/security-and-compliance)
- [在 Operations Management Suite 安全性和稽核解決方案內監視及回應安全性警示](https://docs.microsoft.com/azure/operations-management-suite/oms-security-responding-alerts)
- [在 Operations Management Suite 安全性和稽核解決方案內監視資源](https://docs.microsoft.com/azure/operations-management-suite/oms-security-monitoring-resources)
