---
title: Azure 安全性與合規性藍圖 - 適用於 UK OFFICIAL 的三層式 IaaS Web 應用程式
description: Azure 安全性與合規性藍圖 - 適用於 UK OFFICIAL 的三層式 IaaS Web 應用程式
services: security
author: jomolesk
ms.assetid: 9c32e836-0564-4906-9e15-f070d2707e63
ms.service: security
ms.topic: article
ms.date: 02/08/2018
ms.author: jomolesk
ms.openlocfilehash: d40e23a7cc113a9db297a7dbf00a2372063dfb52
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39059210"
---
# <a name="azure-security-and-compliance-blueprint---three-tier-iaas-web-application-for-uk-official"></a>Azure 安全性與合規性藍圖 - 適用於 UK OFFICIAL 的三層式 IaaS Web 應用程式

## <a name="overview"></a>概觀

 本文提供指引和自動化指令碼，以提供適合處理許多在英國已歸類為 OFFICIAL 之工作負載的 Microsoft Azure 三層式 Web 型架構。

 透過採用「基礎結構即程式碼」的方式，由 [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) 範本集合所部署的環境，將能符合英國國家網路安全中心 (NCSC) 的 14 個[雲端安全性準則](https://www.ncsc.gov.uk/guidance/implementing-cloud-security-principles) \(英文\)，以及網際網路安全中心 (CIS) 的 [重大安全性控制措施](https://www.cisecurity.org/critical-controls.cfm) \(英文\)。

 NCSC 建議客戶使用他們的雲端安全性原則來評估服務的安全性屬性，並協助了解客戶與供應商之間的責任分工。 我們已針對每個原則提供相關資訊，以協助您了解責任的劃分方式。

 Microsoft 技術白皮書[使用 Microsoft Azure 對於英國雲端的 14 個安全性控制措施](https://gallery.technet.microsoft.com/14-Cloud-Security-Controls-670292c1) \(英文\) 支援此架構和對應的 Azure Resource Manager 範本。 本白皮書記載 Azure 服務符合 UK NCSC 的 14 個雲端安全性原則的方式，藉以讓組織能夠快速地符合在 Microsoft Azure 雲端上於全球及英國使用雲端式服務所衍生的合規性義務。

 此範本會針對工作負載部署基礎結構。 必須安裝並設定應用程式程式碼及支援的商務層和資料層的軟體。 詳細的部署指示可於[這裡](https://aka.ms/ukwebappblueprintrepo) \(英文\) 取得。

 如果您沒有 Azure 訂用帳戶，則可快速且輕鬆地進行註冊：[開始使用 Azure](https://azure.microsoft.com/get-started/)。

## <a name="architecture-diagram-and-components"></a>架構圖表與元件

 Azure 範本能在支援 UK OFFICIAL 工作負載的 Azure 雲端環境中，提供三層式 Web 應用程式架構。 此架構能提供可將內部部署網路擴充至 Azure 的安全混合式環境，以允許從公司或網際網路安全地存取 Web 型工作負載。

![適用於 UK OFFICIAL 的三層式 IaaS Web 應用程式參考架構圖表](images/ukofficial-iaaswa-architecture.png?raw=true "適用於 UK OFFICIAL 的三層式 IaaS Web 應用程式參考架構圖表")

 此解決方案會使用下列 Azure 服務。 部署架構的詳細資料位於[部署架構](#deployment-architecture)一節中。

(1) 個 /16 虛擬網路：作業 VNet
- (3) 個 /24 子網路：3 層式 (網路、商務、資料)
- (1) 個 /27 子網路：ADDS
- (1) 個 /27 子網路：閘道子網路
- (1) 個 /29 子網路：應用程式閘道子網路
- 使用預設 (Azure 提供) 的 DNS
- 針對管理 VNet 啟用對等互連
- 網路安全性群組 (NSG) 以管理流量流程

(1) 個 /24 虛擬網路：管理 VNet
- (1) 個 /27 子網路
- 使用 (2) 個 ADDS DNS 和 (1) 個 Azure DNS 項目
- 針對作業 VNet 啟用對等互連
- 網路安全性群組 (NSG) 以管理流量流程

(1) 個應用程式閘道
- WAF：已啟用
- WAF 模式：預防
- 規則集：OWASP 3.0
- 位於連接埠 80 上的 HTTP 接聽程式
- 透過 NSG 進行規範的連線/流量
- 已定義公用 IP 位址端點 (Azure)

(1) 個 VPN：路由式、站對站 IPSec VPN 通道
- 已定義公用 IP 位址端點 (Azure)
- 透過 NSG 進行規範的連線/流量
- (1) 個區域網路閘道 (內部部署端點)
- (1) 個 Azure 網路閘道 (Azure 端點)

(9) 部虛擬機器：所有 VM 皆使用 Azure IaaS Antimalware DSC 設定完成部署

- (2) 個 Active Directory Domain Services 網域控制站 (Windows Server 2012 R2)
  - (2) 個 DNS 伺服器角色：每部 VM 1 個
  - (2) 個已連線到作業 VNet 的 NIC：每部 VM 1 個
  - 上述兩者皆已加入於範本中定義的網域
    - 在部署過程中建立的網域


- (1) 部 Jumpbox (防禦主機) 管理 VM
  - 1 個在具有公用 IP 位址之管理 VNet 上的 NIC
    - NSG 是用於限制針對特定來源的流量 (輸入/輸出)
  - 未加入網域


- (2) 部網路層 VM
  - (2) 個 IIS 伺服器角色：每部 VM 1 個
  - (2) 個已連線到作業 VNet 的 NIC：每部 VM 1 個
  - 未加入網域


- (2) 部商務層 VM
  - (2) 個已連線到作業 VNet 的 NIC：每部 VM 1 個
  - 未加入網域


- (2) 部資料層 VM
  - (2) 個已連線到作業 VNet 的 NIC：每部 VM 1 個
  - 未加入網域

可用性設定組 (Availability Sets)
- (1) 個 Active Directory 網域控制站 VM 集合：2 部 VM
- (1) 個網路層 VM 集合：2 部 VM
- (1) 個商務層 VM 集合：2 部 VM
- (1) 個資料層 VM 集合：2 部 VM

負載平衡器
- (1) 個網路層負載平衡器
- (1) 個商務層負載平衡器
- (1) 個 資料層負載平衡器

儲存體
- 一共 (14) 個儲存體帳戶
  - Active Directory 網域控制站可用性設定組
    - (2) 個主要本地備援儲存體 (LRS) 帳戶：每部 VM 1 個  
    - (1) 個針對 ADDS 可用性設定組的診斷本地備援儲存體 (LRS) 帳戶
  - 管理 Jumpbox VM
    - (1) 個針對 Jumpbox VM 的主要本地備援儲存體 (LRS) 帳戶
    - (1) 個針對 Jumpbox VM 的診斷本地備援儲存體 (LRS) 帳戶
  - 網路層 VM
    - (2) 個主要本地備援儲存體 (LRS) 帳戶：每部 VM 1 個  
    - (1) 個針對網路層可用性設定組的診斷本地備援儲存體 (LRS) 帳戶
  - 商務層 VM
    - (2) 個主要本地備援儲存體 (LRS) 帳戶：每部 VM 1 個  
    - (1) 個針對商務層可用性設定組的診斷本地備援儲存體 (LRS) 帳戶
  - 資料層 VM
    - (2) 個主要本地備援儲存體 (LRS) 帳戶：每部 VM 1 個  
    - (1) 個針對資料層可用性設定組的診斷本地備援儲存體 (LRS) 帳戶

### <a name="deployment-architecture"></a>部署架構：

**內部部署網路**：實作於組織中的私人區域網路。

**生產環境 VNet**：生產環境 [VNet](https://docs.microsoft.com/azure/Virtual-Network/virtual-networks-overview) (虛擬網路) 會裝載在 Azure 中執行的應用程式及其他作業資源。 每個 VNet 可能會包含數個可用來隔離及管理網路流量的子網路。

**網路層**：處理傳入的 HTTP 要求。 回應會透過此層傳回。

**商務層**：實作商務程序及其他針對系統的功能性邏輯。

**資料庫層**：提供持續性資料儲存體，使用 [SQL Server AlwaysOn 可用性群組](https://msdn.microsoft.com/library/hh510230.aspx)來取得高可用性。 客戶可以使用 [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview) 作為替代的 PaaS。

**閘道**：[VPN 閘道](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways)會提供內部部署網路與生產環境 VNet 中路由器之間的連線。

**網際網路閘道和公用 IP 位址**：網際網路閘道會透過網際網路向使用者公開應用程式服務。 存取這些服務的流量會透過[應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)來進行保護，應用程式閘道能提供第 7 層路由和負載平衡功能，並搭配 Web 應用程式防火牆 (WAF) 保護。

**管理 VNet**：此 [VNet](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) 所包含的資源能針對在生產環境 VNet 中執行的工作負載，實作管理與監視功能。

**Jumpbox**：也稱為[防禦主機](https://en.wikipedia.org/wiki/Bastion_host) \(英文\)，這個位於網路上的安全 VM，可供系統管理員用來連線到生產環境 VNet 中的 VM。 Jumpbox 具有 NSG，能夠儘允許來自安全清單上公用 IP 位址的遠端流量。 若要允許遠端桌面 (RDP) 流量，必須在 NSG 中定義該流量的來源。 生產環境資源的管理是使用安全的 Jumpbox VM 透過 RDP 進行。

**使用者定義的路由**：[使用者定義的路由](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)會用來定義 Azure VNet 內 IP 流量的流程。

**網路對等互連的 VNET**：生產環境 VNet 和管理 VNet 會使用 [VNet 對等互連](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)來連線。
系統仍然會將這些 VNet 作為不同的資源來管理，但針對這些虛擬機器的所有連線用途則會顯示為單一資源。 這些網路會使用私人 IP 位址與彼此進行直接通訊。 VNet 對等互連的 VNet 皆需位於同一個 Azure 區域。

**網路安全性群組**：[NSG](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 包含能允許或拒絕 VNet 內之流量的存取控制清單。 NSG 可以用來保護子網路或個別 VM 層級的流量。

**Active Directory Domain Services (AD DS)**：此架構會提供專用的 [Active Directory Domain Services](https://technet.microsoft.com/library/hh831484.aspx) 部署。

**記錄與稽核**：[Azure 活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)會擷取在您訂用帳戶中的資源上所採取的作業，例如起始作業的人員、作業發生的時間、作業的狀態，以及可能有助於您研究作業的其他屬性值。 Azure 活動記錄為可擷取訂用帳戶上所有動作的 Azure 平台服務。 必要時可將記錄封存或匯出。

**網路監視與警示**：[Azure 網路監看員](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)是一個可針對 VNet 內的網路流量提供網路封包擷取、流程記錄、拓撲工具及診斷的平台服務。

## <a name="guidance-and-recommendations"></a>指引與建議

### <a name="business-continuity"></a>商務持續性

**高可用性**：伺服器工作負載會組成[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-manage-availability?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，以協助確保 Azure 中虛擬機器的高可用性。 此設定有助於確保在規劃或未規劃的維護事件發生期間，至少會有一部虛擬機器可供使用並符合 99.95% Azure SLA。

### <a name="logging-and-audit"></a>記錄與稽核

**監視**：[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-get-started)是一個可提供單一來源以監視所有 Azure 資源的活動記錄、計量和診斷記錄的平台服務。 可針對 Azure 監視器加以設定，以針對來自 Azure 中資源的計量和記錄進行視覺化、查詢、路由傳送、封存及反應。 建議使用資源型存取控制來保護稽核線索，以協助確保使用者不具修改記錄的能力。

**活動記錄**：設定 [Azure 活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)，以針對在訂用帳戶中資源上所執行的作業提供深入解析。

**診斷記錄**：[診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)是由資源發出的所有記錄。 這些記錄可能包含 Windows 事件系統記錄、Blob、資料表和佇列記錄。

**防火牆記錄**：應用程式閘道會提供完整的診斷和存取記錄。 防火牆記錄檔可供已啟用 WAF 的應用程式閘道資源使用。

**記錄封存**：可針對封存及定義的保留期間需求，設定記錄資料儲存體以寫入集中式 Azure 儲存體帳戶。 可以使用 Azure Log Analytics 或透過協力廠商 SIEM 系統來處理記錄。

### <a name="identity"></a>身分識別

**Active Directory Domain Services**：此架構能在 Azure 中提供 Active Directory Domain Services 部署。 如需在 Azure 中實作 Active Directory 的特定建議，請參閱下列文章：

[將 Active Directory Domain Services (AD DS) 擴充至 Azure](https://docs.microsoft.com/azure/guidance/guidance-identity-adds-extend-domain)。

[在 Azure 虛擬機器中部署 Windows Server Active Directory 的指導方針](https://msdn.microsoft.com/library/azure/jj156090.aspx)。

**Active Directory 整合**：作為專用 AD DS 架構的替代方案，客戶可能會想要使用 [Azure Active Directory](https://docs.microsoft.com/azure/guidance/guidance-ra-identity#using-azure-active-directory) 整合或 [Azure 中已加入內部部署樹系的 Active Directory](https://docs.microsoft.com/azure/guidance/guidance-ra-identity#using-active-directory-in-azure-joined-to-an-on-premises-forest)。

### <a name="security"></a>安全性

**管理安全性**：此藍圖可讓系統管理員使用來自信任來源的 RDP，連線到管理 VNet 和 Jumpbox。 管理 VNet 的網路流量是使用 NSG 來控制。 針對連接埠 3389 的存取，僅限於來自可存取包含 Jumpbox 之子網路的受信任 IP 範圍的流量。

客戶可能也會考慮使用[增強式安全性系統管理模型](https://technet.microsoft.com/windows-server-docs/security/securing-privileged-access/securing-privileged-access) \(機器翻譯\)，來在連線到管理 VNet 和 Jumpbox 時保護環境的安全。 若要增強安全性，建議客戶使用[具特殊權限的存取工作站](https://technet.microsoft.com/windows-server-docs/security/securing-privileged-access/privileged-access-workstations#what-is-a-privileged-access-workstation-paw) \(機器翻譯\) 和 RDGateway 設定。 使用網路虛擬裝置及公開/私人 DMZ，將能提供進一步的安全性增強功能。

**保護網路**：建議針對每個子網路使用[網路安全性群組](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) (NSG) 來提供第二層保護，以避免輸入的流量略過設定不正確或已停用的閘道。 範例：[用於部署 NSG 的 Resource Manager 範本](https://github.com/mspnp/template-building-blocks/tree/v1.0.0/templates/buildingBlocks/networkSecurityGroups) \(英文\)。

**保護公用端點**：網際網路閘道會透過網際網路向使用者公開應用程式服務。 存取這些服務的流量會透過[應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)來進行保護，應用程式閘道能提供 Web 應用程式防火牆和 HTTPS 通訊協定管理。

**IP 範圍**：架構中的 IP 範圍是建議的範圍。 建議客戶考量自己的環境並使用適當的範圍。

**混合式連線**：雲端式工作負載會使用 Azure VPN 閘道，透過 IPSEC VPN 連線到內部部署資料中心。 客戶應該確保他們是使用適當的 VPN 閘道來連線到 Azure。 範例：[VPN 閘道 Resource Manager 範本](https://github.com/mspnp/template-building-blocks/tree/v1.0.0/templates/buildingBlocks/vpn-gateway-vpn-connection) \(英文\)。 執行具有巨量資料需求的大規模、任務關鍵性工作負載的客戶，可能會想要考慮使用 [ExpressRoute](https://docs.microsoft.com/azure/guidance/guidance-hybrid-network-expressroute) \(英文\) 的混合式網路架構，以取得針對 Microsoft 雲端服務的私人網路連線能力。

**分開考量**：此參考架構會將用於管理作業和商務作業的 VNet 分隔開來。 透過遵循 [Microsoft 雲端服務和網路安全性](https://docs.microsoft.com/azure/best-practices-network-security)最佳作法在網路區段之間使用 NSG，將 VNet 和子網路分隔開來將能允許流量管理 (包括流量輸入和輸出限制)。

**資源管理**：Azure 資源 (例如 VM、VNet 及負載平衡器) 是透過將它們組成 [Azure 資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groupsresource) 來管理。 接著可將資源型存取控制角色指派給每個資源群組，以限制只有經授權的使用者可以存取。

**存取控制限制**：使用[角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal) (RBAC) 來透過[自訂角色](https://docs.microsoft.com/azure/role-based-access-control/custom-roles)管理應用程式中的資源。RBAC 可以用來限制 DevOps 可於每個層級上執行的作業。 授與權限時，請使用[最小權限的原則](https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1)。 記錄所有的系統管理作業並定期執行稽核，以確保所有設定變更都是經過規劃的。

**網際網路存取**：此參考架構會運用 [Azure 應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)作為網際網路面向閘道和負載平衡器。 作為 [Azure 應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)的替代項目，有些客戶可能也會考慮使用協力廠商網路虛擬裝置來取得額外的網路安全性層。

**Azure 資訊安全中心**：[Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)能提供訂用帳戶中資源之安全性狀態的集中檢視，並提供有助於預防資源被入侵的相關建議。 它也可以用來啟用更細微的原則。 例如，可將原則套用到特定的資源群組，來使企業能夠調整針對風險的作法。 建議客戶在其 Azure 訂用帳戶中啟用 Azure 資訊安全中心。

## <a name="ncsc-cloud-security-principles-compliance-documentation"></a>NCSC 雲端安全性原則合規性文件

Crown Commercial Service (一所致力於改善政府相關商業和採購活動的機構) 已將 Microsoft 範圍內企業雲端服務的分類更新為 G-Cloud v6，以涵蓋其 OFFICIAL 層級的所有供應項目。 Azure 與 G-Cloud 的詳細資料可在 [Azure 英國 G-Cloud 安全性評定摘要](https://www.microsoft.com/en-us/trustcenter/compliance/uk-g-cloud) \(英文\) 上取得。

此藍圖符合 NCSC [雲端安全性準則](https://www.ncsc.gov.uk/guidance/implementing-cloud-security-principles) \(英文\) 中所記載的 14 個雲端安全性準則，以協助確保環境能支援已分類為 UK-OFFICIAL (英國官方) 的工作負載。

[客戶責任矩陣](https://aka.ms/ukofficial-crm) \(英文\) (Excel 活頁簿) 會將 14 個雲端安全性原則全部列出，且矩陣會針對每個原則 (或原則的子項目)，表示原則的實作應該是由 Microsoft 或客戶負責，還是由兩者共同承擔。

[準則實作對照表](https://aka.ms/ukofficial-iaaswa-pim) \(英文\) (Excel 活頁簿) 會將 14 個雲端安全性準則全部列出，且對照表會針對已在客戶責任對照表中指定為客戶責任的每個準則 (或準則的子項目)，表示 1) 藍圖自動化是否會實作該準則，以及 2) 實作如何符合準則需求的說明。

此外，雲端安全聯盟 (CSA) 已發佈雲端控制矩陣，以支援正在評估雲端提供者的客戶，以及識別出應在移至雲端服務之前先獲得解答的問題。 作為回應，Microsoft Azure 已回答 CSA 共識評量行動問卷 ([CSA CAIQ](https://www.microsoft.com/en-us/TrustCenter/Compliance/CSA) \(英文\))，其中描述 Microsoft 因應建議原則的方式。

## <a name="deploy-the-solution"></a>部署解決方案

部署使用者可以用兩種方法來部署此藍圖自動化。 第一種方法會使用 PowerShell 指令碼，而第二種方法會利用 Azure 入口網站來部署參考架構。 詳細的部署指示可於[這裡](https://aka.ms/ukofficial-iaaswa-repo) \(英文\) 取得。

## <a name="disclaimer"></a>免責聲明

 - 此文件僅供參考之用。 Microsoft 對本文件中的資訊不做任何明示、暗示或成文之擔保。 這份文件係依「現狀」提供。 本文件中說明的資訊與畫面 (包括 URL 及其他網際網路網站參考資料) 如有變更，恕不另行通知。 閱讀這份文件的客戶須自行承擔使用風險。
 - 本文件未提供給客戶任何 Microsoft 產品或解決方案中任何智慧財產的任何法定權利。
 - 客戶可以複製並使用這份文件，供內部參考之用。
 - 本文件的某些建議可能會導致資料、網路或 Azure 計算資源使用量增加，並可能增加客戶的 Azure 授權或訂用帳戶成本。
 - 此架構的目的是作為基礎，讓客戶進行調整以符合其特定需求，不應完全未經修改就用於生產環境中。
 - 此文件編製做為參考資料，且不應以此文件定義為客戶能符合特定合規性需求和法規的所有手段。 對於核准的客戶實作，客戶應向其組織尋求法務支援。
