---
title: 適用於 Azure 的受信任網際網路連線指導方針
description: 適用於 Azure 與 SaaS 服務的受信任網際網路連線指導方針
services: security
author: dlapiduz
ms.assetid: 09511e03-a862-4443-81ac-ede815bdaf25
ms.service: security
ms.topic: article
ms.date: 06/20/2018
ms.author: dlap
ms.openlocfilehash: d52785dd7569560f4b6986080b14723762537ec8
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388300"
---
# <a name="trusted-internet-connections-guidance"></a>受信任網際網路連線指導方針

此文章涵蓋政府機構如何使用 Microsoft Azure Government 幫助實現受信任網際網路連線 (TIC) 方案。 此文章說明如何跨 Azure 基礎結構即服務 (IaaS) 與 Azure 平台即服務 (PaaS) 供應項目使用 Azure Government。

## <a name="trusted-internet-connections-overview"></a>受信任網際網路連線概觀

TIC 方案的目的是使聯邦機構使用之各個外部網路連線的安全性達到最佳化和標準化。 該原則述於[辦公室 (OMB) 備忘錄 M-08-05](https://georgewbush-whitehouse.archives.gov/omb/memoranda/fy2008/m08-05.pdf)內。

2007 年 11 月，OMB 建立了 TIC 計畫，以改善聯邦網路周邊安全性以及事件回應功能。 TIC 的設計目的在於對所有傳入和傳出的 .gov 流量執行網路分析，以識別特定特徵標記和以模式為基礎的資料。 TIC 可以找出行為異常現象，例如殭屍網路活動。 機構被授權合併其外部網路連線並路由傳送所有流量到入金偵測與預防裝置 (稱為 EINSTEIN)。 這些裝置是裝載於網路端點 (稱為_受信任的網際網路連線_.) 有限的環境。

TIC 的目標是讓機構了解以下事項：
- 誰在我的網路上 (授權或未經授權)？
- 何時存取我的網路？為何存取？
- 正在存取哪些資源？

所有機構外部連線現在都必須透過 OMB 核准的 TIC 來路由傳送。 聯邦機構必須以 TIC 存取權限提供者 (TICAP) 的身分參與 TIC 計畫，或是與主要一級網際網路服務提供者之一簽訂服務合約，藉以參與計畫。 這些提供者稱為受控的受信任網際網路通訊協定服務 (MTIPS) 提供者。 TIC 包括機構和 MTIPS 提供者執行的強制性關鍵功能。 在目前版本的 TIC 中，EINSTEIN 版本 2 入侵偵測與 EINSTEIN 版本 3 加速 (3A) 入侵預防裝置已部署在每個 TICAP 與 MTIPS。 機構與國土安全部 (DHS) 合作建立「了解備忘錄」以將 EINSTEIN 功能部署到聯邦系統中。

為了履行其保護 .gov 網路的責任，DHS 要求將機構靜流量資料之未經處理資料摘要與聯邦政府中的事件建立關聯，並使用專用工具執行分析。 DHS 路由器可在進入或退出介面時收集 IP 網路流量。 網路系統管理員可以淨流量資料，以確定流量來源和目的地、服務等級等資訊。 靜流量資料會被視為與標頭、來源 IP、目的地 IP 等類似的「非內容資料」。 非內容資料可讓 DHS 了解下列內容：誰在做什麼，以及做多久。

採用內部部署基礎架構的安全性原則、指南和架構也包含在該計畫內。 隨著政府機構移轉至雲端以節省成本、提高營運效率以及進行創新，TIC 的實作需求會使得網路流量變慢。 因此，政府機關使用者存取其雲端資料的速度與靈活性受限制。

## <a name="azure-networking-options"></a>Azure 網路功能選項

連線到 Azure 服務時，有三個主要選項：

- 直接網際網路連線：透過開放的網際網路連線直接連線到 Azure 服務。 媒介與連線都是公用的。 是依賴應用程式和傳輸層級加密來確保隱私權。 頻寬受站台與網際網路之間的連線能力而定。 使用一個以上的提供者來確保復原能力。
- 虛擬私人網路 (VPN)：使用 VPN 閘道連線到您私人使用的 Azure 虛擬網路。
媒介是公開的，因為它會周遊站台的標準網際網路連線，但會在通道中加密連線以確保隱私權。 頻寬受限於 VPN 裝置和選擇的設定。 Azure 點對站連線通常限制為 100 Mbps，而站對站連線限制為 1.25 Gbps。
- Azure ExpressRoute：ExpressRoute 會直接連線到 Microsoft 服務。 因為連線是透過隔離的光纖通道進行的，因此連線可以是公用連線或私人連線，取決於所使用的設定。 頻寬通常限制為最大 10 Gbps。

有幾種方法可以滿足「TIC 附錄 H (雲端注意事項)」需求，如國土安全部的「受信任網際網路連線 (TIC) 參考架構文件 2.0 版」中所指定。 在此文章中，DHS TIC 指導方針稱為 **TIC 2.0**。

若要啟用從**部門或機構 (D/A)** 到 Azure 或 Office 365 的連線，而不透過 D/A TIC 路由傳送流量，D/A 必須使用加密通道或雲端服務提供者 (CSP) 的專用連線。 CSP 服務可以確保不向公用網際網路提供 D/A 雲端資產的連線，以便機構人員直接存取。

Office 365 使用已啟用 [Microsoft 對等互連](https://docs.microsoft.com/azure/expressroute/expressroute-circuit-peerings#expressroute-routing-domains)的 Express Route，或使用 TLS 1.2 加密所有流量的網際網路連線，以遵守 TIC 2.0 附錄 H。 D/A 網路上的 D/A 終端使用者可以透過其機構網路和 TIC 基礎架構，利用網際網路進行連線。 所有 Office 365 的遠端網際網路存取皆被封鎖，並透過機構進行路由傳送。 D/A 也可以在已啟用 Microsoft 對等互連 (這是一種公用對等互連) 的情況下，透過 ExpressRoute 連線與 Office 365 連線。  

僅適用於 Azure，第二個選項 (VPN) 和第三個選項 (ExpressRoute) 與限制存取網際網路的服務結合使用時，皆可滿足這些要求。

![Microsoft 受信任網際網路連線 (TIC)](media/tic-diagram-a.png)

## <a name="azure-infrastructure-as-a-service-offerings"></a>Azure 基礎結構即服務供應項目

使用 Azure IaaS 來符合 TIC 原則規範相當簡單，因為 Azure 客戶會管理自己的虛擬網路路由。

若要確實符合 TIC 參考架構，最主要的需求是確保虛擬網路 (VNet) 是 D/A 網路的私人延伸。 若要成為_私人_延伸，除非透過內部部署的 TIC 網路連線，否則該原則會要求沒有任何流量離開您的網路。 此程序稱為「強制通道處理」。 針對 TIC 合規性，此程序會透過 TIC，將來自 CSP 環境中任何系統的所有流量，通過組織網路上的內部部署閘道，路由傳出網際網路。

Azure IaaS TIC 合規性分為兩個主要步驟：

- 步驟 1：設定。
- 步驟 2: 稽核。

### <a name="azure-iaas-tic-compliance-configuration"></a>Azure IaaS TIC 合規性：設定

若要以 Azure 設定好符合 TIC 的架構，必須先防止網際網路直接存取您的虛擬網路，然後強制要求網際網路流量必須通過內部部署網路。

#### <a name="prevent-direct-internet-access"></a>防止網際網路直接存取

Azure IaaS 網路是透過由子網路組成的虛擬網路進行，這些子網路與虛擬機器的網路介面控制器 (NIC) 相關聯。

支援 TIC 合規性最簡單的情節是確保虛擬機器或虛擬機器集合無法連線到任何外部資源。 使用網路安全性群組 (NSGs) 確定已中斷與外部網路的連線。 使用 NSGs 來控制連到您虛擬網路中一或多個 NIC 或子網路的流量。 NSG 包含存取控制規則，可根據流量方向、通訊協定、來源位址和連接埠與目的地位址和連接埠，允許或拒絕流量。 您可以隨時變更 NSG 的規則，而且變更會套用到所有相關聯的執行個體。 若要深入了解如何建立 NSG，請參閱[使用網路安全性群組篩選網路流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-create-nsg-arm-pportal)。

#### <a name="force-internet-traffic-through-an-on-premises-network"></a>強制網際網路流量通過內部部署網路

Azure 會自動建立系統路由，並將路由指派給虛擬網路中的每個子網路。 您無法建立或移除系統路由，但是您可以使用自訂路由覆寫某些系統路由。 Azure 會為每個子網路建立預設系統路由。 Azure 會將選擇性的預設路由加入特定子網路，或是當您使用特定 Azure 功能時則是加入每個子網路。 此類的路由可確保：
- 目的地位於虛擬網路內的流量仍會留在虛擬網路內。
- IANA 指定的私人位址空間 (如 10.0.0.0/8) 會被捨棄，除非它們已包含在虛擬網路位址空間中。
- 「最後手段」路由 0.0.0.0/0 到虛擬網路網際網路端點。

![TIC 強制通道](media/tic-diagram-c.png)

離開虛擬網路的所有流量都必須通過內部部署連線，以確保所有流量都會周遊 D/A TIC。 您可以透過建立使用者定義的路由，或透過交換邊界閘道協定 (BGP) 路由，在內部部署網路閘道和 Azure VPN 閘道之間建立自訂路由。 如需有關使用者定義路由詳細資訊，請參閱[虛擬網路流量路由：使用者定義路由](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#user-defined)。 如需有關 BGP 的詳細資訊，請參閱[虛擬網路流量路由：邊界閘道協定](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#border-gateway-protocol)。

#### <a name="add-user-defined-routes"></a>新增使用者定義路由

若您使用路由型虛擬網路閘道，您可以在 Azure 中強制執行通道處理。 新增會設定 0.0.0.0/0 流量至您虛擬網路閘道中**下一個躍點**的使用者定義路由 (UDR)。 Azure 會為使用者定義路由賦予較系統定義路由高的優先順序。 所有非虛擬網路流量都會被傳送到您的虛擬網路閘道，此閘道接著會將流量路由傳送到內部部署。 定義 UDR 之後，將該路由與您 Azure 環境中所有虛擬網路內的現有子網路或新子網路建立關聯。

![使用者定義路由與 TIC](media/tic-diagram-d.png)

#### <a name="use-the-border-gateway-protocol"></a>使用邊界閘道協定

若您使用 ExpressRoute 或已啟用 BGP 的虛擬網路閘道，則 BGP 是通告路由的慣用機制。 針對 BGP 通告路由 0.0.0.0/0，ExpressRoute 和 BGP 感知虛擬網路閘道可確保預設路由套用至您虛擬網路中的所有子網路。

### <a name="azure-iaas-tic-compliance-auditing"></a>Azure IaaS TIC 合規性：稽核

Azure 提供數種方式以稽核 TIC 合規性。

#### <a name="view-effective-routes"></a>檢視有效的路由

透過在 [Azure 入口網站](https://docs.microsoft.com/azure/virtual-network/virtual-network-routes-troubleshoot-portal#diagnose-using-azure-portal)或 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-network/virtual-network-routes-troubleshoot-powershell#diagnose-using-powershell) 中觀察 NIC 或使用者定義路由表的_有效路由_以確認您的預設路由已傳播。 [有效路由] 會顯示相關使用者定義路由、BGP 通告路由與套用到相關實體的系統路由，如下圖所示：

![有效路由](media/tic-screen-1.png)

> [!NOTE]
> 您無法檢視 NIC 的有效路由，除非它與執行中的虛擬機器相關聯。

#### <a name="use-azure-network-watcher"></a>使用 Azure 網路監看員

Azure 網路監看員提供數種工具，可用於稽核 TIC 合規性。 如需詳細資訊，請參閱[這篇有關網路監看員的概觀](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)。

##### <a name="capture-network-security-group-flow-logs"></a>擷取網路安全性群組流量記錄 

使用「網路監看員」來擷取網路流量記錄，這些記錄指出包圍 IP 流量的中繼資料。 網路流量記錄包含目標的來源和目的地位址，以及其他資料。 您可以使用此資料搭配來自虛擬網路閘道、內部部署邊緣裝置或 TIC 的記錄，以監視所有流量是否都已在內部部署環境路由傳送。 

## <a name="azure-platform-as-a-service-offerings"></a>Azure 平台即服務供應項目

Azure PaaS 服務 (例如 Azure 儲存體) 可透過網際網路連線的 URL 存取。 具有已核准認證的任何人都可以從任何位置存取資源 (例如儲存體帳戶)，而不需要周遊 TIC。 因此，許多政府客戶誤解 Azure PaaS 服務未符合 TIC 原則規範。 許多 Azure PaaS 服務都符合 TIC 原則規範。 當架構與符合 TIC 規範之 IaaS 環境 ([如先前所述](https://docs.microsoft.com/azure/security/compliance/compliance-tic#azure-infrastructure-as-a-service-offerings)) 相同且服務連結到 Azure 虛擬網路時，服務即符合規範。

當 Azure PaaS 服務與虛擬網路整合時，可從虛擬網路對服務進行私人存取。 您可以透過使用者定義路由或 BGP 來針對 0.0.0.0/0 套用自訂路由。 自訂路由可確保所有網際網路繫結流量會從內部部署環境路由傳送以周遊 TIC。 使用下列模式將 Azure 服務整合到虛擬網路：

- **部署專用的服務執行個體**：可以將越來越多的 PaaS 服務部署為具有虛擬網路附加端點的專用執行個體。 您能以「隔離」模式針對 PowerApps 部署 App Service Environment，讓網路端點可以限制於虛擬網路。 App Service Environment 接著可以裝載許多 Azure PaaS 服務，例如 Azure Web Apps、Azure API Management 與 Azure Functions。
- **使用虛擬網路服務端點**：越來越多的 PaaS 服務允許將其端點移動到虛擬網路私人 IP，而不是公用位址。

支援將專用執行個體部署到虛擬網路或使用服務端點的服務 (截至 2018 年 5 月為止) 列於下表中。

> [!Note]
> 可用性狀態對應到商業版本的 Azure 服務。 Azure Government 中 Azure 服務的可用性狀態擱置中。

### <a name="support-for-service-endpoints"></a>對服務端點的支援

|服務                        |可用性      |
|-------------------------------|------------------|
|Azure 金鑰保存庫                | 個人預覽版  |
|Azure Cosmos DB                | 個人預覽版  |
|身分識別服務              | 個人預覽版  |
|Azure Data Lake                | 個人預覽版  |
|適用於 PostgreSQL 的 Azure 資料庫  | 個人預覽版  |
|適用於 MySQL 的 Azure 資料庫       | 個人預覽版  |
|Azure SQL 資料倉儲       | 公開預覽版   |
|連接字串             | 公開上市 (GA) |
|Azure 儲存體                  | GA               |

### <a name="support-for-virtual-network-injection"></a>對虛擬網路插入的支援

|服務                               |可用性      |
|--------------------------------------|------------------|
|Azure SQL Database 受控執行個體   | 公開預覽版   |
|Azure Kubernetes Service (AKS)        | 公開預覽版   |
|Azure Service Fabric                  | GA               |
|Azure API 管理                  | GA               |
|Azure Active Directory                | GA               |
|Azure Batch                           | GA               |
|App Service 環境               | GA               |
|Azure Redis 快取                     | GA               |
|Azure HDInsight                       | GA               |
|虛擬機器擴展集             | GA               |
|Azure 雲端服務                  | GA               |


### <a name="virtual-network-integration-details"></a>虛擬網路整合詳細資料

下圖顯示存取 PaaS 服務的一般網路流程。 會從虛擬網路插入與虛擬網路服務通道處理顯示存取。 如需有關網路服務閘道、虛擬網路與服務標籤的詳細資訊，請參閱[網路與應用程式安全性群組：服務標籤](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)。

![TIC 的 PaaS 連線選項](media/tic-diagram-e.png)

1. 使用 ExpressRoute 私人連線至 Azure。 具有強制通道功能之 ExpressRoute 私人對等互連是用於強制 ExpressRoute 上所有客戶虛擬網路流量回到內部部署之中。 不需要 Microsoft 對等互連。
2. Azure VPN 閘道與 ExpressRoute 和 Microsoft 對等互連結合使用，可用於重疊客戶虛擬網路與內部部署邊緣之間端對端的 IPSec 加密。 
3. 使用網路 NSG 控制與客戶虛擬網路的網路連線，可讓客戶根據 IP、連接埠和通訊協定加以允許/拒絕。
4. 透過為客戶服務建立服務端點，將客戶虛擬網路延伸到 PaaS 服務。
5. PaaS 服務端點被保護為**預設拒絕全部**，並且僅允許從客戶虛擬網路內的指定子網路存取。 預設拒絕也包含源自於網際網路的連線。
6. 需要存取客戶虛擬網路中資源的其他 Azure 服務應為以下其中之一：  
   - 直接部署至虛擬網路。
   - 根據個別 Azure 服務的指導方針選擇性地允許。

#### <a name="option-a-deploy-a-dedicated-instance-of-a-service-virtual-network-injection"></a>選項 A：部署服務的專用執行個體 (虛擬網路插入)

客戶可使用虛擬網路 插入選擇性地將給定 Azure 服務的專用執行個體 (例如 HDInsight) 部署到他們自己的虛擬網路中。 服務執行個體已部署至客戶虛擬網路中的專用子網路。 虛擬網路插入允許透過非網際網路可路由位址存取服務資源。 內部部署執行個體可以透過虛擬網路位址空間使用 ExpressRoute 或站點到站點 VPN 直接存取服務執行個體，而不是透過開啟防火牆到公用網際網路位址空間。 當專用執行個體附加到端點時，您可以使用與針對 IaaS TIC 合規性使用之策略相同的策略。 預設路由可確保網際網路繫結流量會被重新導向到針對內部部署環境繫結的虛擬網路閘道。 您可以透過給定子網路的 NSG 來控制連入與連出存取。

![虛擬網路插入概觀](media/tic-diagram-f.png)

#### <a name="option-b-use-virtual-network-service-endpoints-service-tunnel"></a>選項 B：使用虛擬網路服務端點 (服務通道)

數目日益增加的 Azure 多組織用戶共享服務提供「服務端點」。 服務端點是整合到 Azure 虛擬網路的替代方法。 虛擬網路服務端點可透過直接連線，將您的虛擬網路 IP 位址空間和虛擬網路的身分識別延伸至服務。 從虛擬網路到 Azure 服務的流量一定會保留在 Azure 骨幹網路上。 

在您針對服務啟用服務端點之後，請使用由服務所公開的原則將服務的連線限制到該虛擬網路。 Azure 服務會強制要求在平台中執行存取檢查。 只有當要求源自允許的虛擬網路或子網路，或來源自用於識別您的內部部署流量的兩個 IP (若您使用 ExpressRoute) 時，才會授與對鎖定之資源的存取權。 使用此方法可有效防止傳入/傳出流量直接離開 PaaS 服務。

![服務端點概觀](media/tic-diagram-g.png)

## <a name="cloud-native-tools-for-network-situational-awareness"></a>適用於網路局勢意識的雲端原生工具

Azure 提供多種雲端原生工具，以協助確保您具備了解網路流量所需的局勢意識。 不需要此工具，就能符合 TIC 原則規範。 此工具可大幅改進您的安全性能力。

### <a name="azure-policy"></a>Azure 原則

[Azure 原則](https://azure.microsoft.com/services/azure-policy/) 是一個 Azure 服務，可為您的組織提供更好的稽核和實施合規性方案的能力。 Azure 原則目前在商業版本 Azure 服務中的公開預覽本中提供。 Azure 原則尚未在 Azure Government 中提供。 客戶現在可以規劃及測試其 Azure 原則規則，以確保未來的 TIC 合規性。 

Azure 原則以訂用帳戶層級為目標。 服務提供集中式介面，您可以在其中執行合規性工作，包括：
- 管理方案
- 設定原則定義
- 稽核合規性
- 強制合規性
- 管理例外狀況

隨著許多內建定義，系統管理員可以使用簡單的 JSON 範本來定義其自己的自訂定義。 Microsoft 建議您在可能的情況下，為稽核賦予較強制高的優先順序。

下列範例原則對於 TIC 合規性案例很有用：

|原則  |範例情節  |範本  |
|---------|---------|---------|
|強制執行使用者定義的路由表。 | 確保所有虛擬網路上的預設路由指向已核准的虛擬網路閘道，以便路由傳送到內部部署。    | 使用此[範本](https://docs.microsoft.com/azure/azure-policy/scripts/no-user-def-route-table)來開始。 |
|稽核是否未針對區域啟用網路監看員。  | 請確定已經針對所有使用區域啟用網路監看員。  | 使用此[範本](https://docs.microsoft.com/azure/azure-policy/scripts/net-watch-not-enabled)來開始。 |
|每個子網路上的 NSG x。  | 確保具有已封鎖之網際網路流量的 NSG (或已核准 NSG 的集合) 會套用至每個虛擬網路中的所有子網路。 | 使用此[範本](https://docs.microsoft.com/azure/azure-policy/scripts/nsg-on-subnet)來開始。 |
|每個 NIC 上的 NSG x。 | 確保具有已封鎖之網際網路流量的 NSG 會套用至所有虛擬機器上的所有 NIC。 | 使用此[範本](https://docs.microsoft.com/azure/azure-policy/scripts/nsg-on-nic)來開始。 |
|為虛擬機器網路介面使用已核准的虛擬網路。  | 確保所有 NIC 都在已核准的虛擬網路上。 | 使用此[範本](https://docs.microsoft.com/azure/azure-policy/scripts/use-approved-vnet-vm-nics)來開始。 |
|允許的位置。 | 確保將所有資源部署到具有符合規範之虛擬網路和網路監看員設定的區域。  | 使用此[範本](https://docs.microsoft.com/azure/azure-policy/scripts/allowed-locs)來開始。 |
|不允許的資源類型，例如 **PublicIPs**。 | 禁止部署沒有合規性計畫的資源類型。 使用此原則來禁止部署公用 IP 位址資源。 雖然 NSG 規則可用於有效封鎖連入網際網路流量，但避免使用公用 IP 可進一步減少受攻擊面。   | 使用此[範本](https://docs.microsoft.com/azure/azure-policy/scripts/not-allowed-res-type)來開始。  |

### <a name="network-watcher-traffic-analytics"></a>網路監看員流量分析

網路監看員[流量分析](https://azure.microsoft.com/blog/traffic-analytics-in-preview/)使用流量記錄資料和其他記錄來提供網路流量的高階概觀。 此資料可用於稽核 TIC 合規性並找出有問題的地方。 您可以使用高階儀表板來快速篩選與網際網路通訊的虛擬機器，並取得 TIC 路由重點清單。

![流量分析](media/tic-traffic-analytics-1.png)

使用**地圖**來快速識別您虛擬機器網際網路流量的可能實體目的地。 您可以識別可疑位置或模式變更並進行分類：

![地圖](media/tic-traffic-analytics-2.png)

使用**虛擬網路拓撲**來快速調查現有的虛擬網路：

![網路拓撲圖](media/tic-traffic-analytics-3.png)

### <a name="network-watcher-next-hop-tests"></a>網路監看員下一個躍點測試

由網路監看員監視之區域中的網路可以執行下一個躍點測試。 在 Azure 入口網站中，您可以輸入範例網路流量的來源與目的地，讓網路監看員解析下一個躍點目的地。 針對虛擬機器與範例網際網路位址執行此測試，以確保下一個躍點目的地是預期的網路虛擬閘道。

![下一個躍點測試](media/tic-network-watcher.png)

## <a name="conclusions"></a>結論

您可以輕鬆地設定 Microsoft Azure、Office 365 和 Dynamics 365 存取，以協助符合 2018 年 5 月撰寫和定義的 TIC 2.0 附錄 H 指導方針的規定。 Microsoft 承認 TIC 指導方針可能會變更。 當新的指導方針發行時，Microsoft 會及時盡力協助客戶符合該指導方針。

## <a name="appendix-trusted-internet-connections-patterns-for-common-workloads"></a>附錄：常見工作負載的受信任網際網路連線模式

| 類別 | 工作負載 | IaaS | 專用的 PaaS/虛擬網路插入  | 服務端點  |
|---------|---------|---------|---------|--------|
| 計算 | Azure Linux 虛擬機器 | 是 | | |
| 計算 | Azure Windows 虛擬機器 | 是 | | |
| 計算 | 虛擬機器擴展集 | 是 | | |
| 計算 | Azure Functions | | App Service 環境 | |
| Web 與行動裝置 | 內部 Web 應用程式 | | App Service 環境| |
| Web 與行動裝置 | 內部行動應用程式 | | App Service 環境 | |
| Web 與行動裝置 | API 應用程式 | | App Service 環境 | |
| 容器 | Azure Container Service | | | 是 |
| 容器 | Azure Kubernetes Service (AKS) \* | | | 是 |
| 資料庫 | 連接字串 | | Azure SQL Database 受控執行個體 \* | Azure SQL |
| 資料庫 | 適用於 MySQL 的 Azure 資料庫 | | | 是 |
| 資料庫 | 適用於 PostgreSQL 的 Azure 資料庫 | | | 是 |
| 資料庫 | Azure SQL 資料倉儲 | | | 是 |
| 資料庫 | Azure Cosmos DB | | | 是 |
| 資料庫 | Azure Redis 快取 | | 是 | |
| 儲存體 | Azure Blob 儲存體 | 是 | | |
| 儲存體 | Azure 檔案 | 是 | | |
| 儲存體 | Azure 佇列儲存體 | 是 | | |
| 儲存體 | Azure 資料表儲存體 | 是 | | |
| 儲存體 | Azure 磁碟儲存體 | 是 | | |

\* 2018 年 5 月的 Azure Government 公開預覽。
