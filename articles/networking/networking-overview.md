---
title: Azure 網路 | Microsoft Docs
description: 深入了解 Azure 網路服務和功能。
services: networking
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/19/2017
ms.author: jdial
ms.openlocfilehash: 2e5862e99b3e883554b42341d2c1dbe9d8b8ec72
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34366975"
---
# <a name="azure-networking"></a>Azure 網路

Azure 提供各種不同的網路功能，它們可以合併或分開使用。 按一下下列任一項重要功能，以深入了解相關資訊︰
- [Azure 資源之間的連線](#connectivity)︰在雲端的一個安全的虛擬私人網路中連接所有 Azure 資源。
- [網際網路連線](#internet-connectivity)︰透過網際網路在 Azure 資源之間進行通訊。
- [內部部署連線](#on-premises-connectivity)：將內部部署網路連線至 Azure 資源，採取的方式有經由網際網路透過虛擬私人網路 (VPN)，或透過與 Azure 的專用連線。
- [負載平衡和流量導向](#load-balancing)︰將流量負載平衡到相同位置的伺服器，以及將流量導向不同位置的伺服器。
- [安全性](#security)︰篩選網路子網路或個別虛擬機器 (VM) 之間的網路流量。
- [路由](#routing)︰在您的 Azure 和內部部署資源之間使用預設路由或完全控制路由。
- [管理能力](#manageability)︰監視和管理您的 Azure 網路資源。
- [部署和設定工具](#tools)︰使用網頁型入口網站或跨平台命令列工具來部署和設定網路資源。

## <a name="Connectivity"></a>Azure 資源之間的連線

Azure 資源 (例如虛擬機器、雲端服務、 虛擬機器擴展集和 Azure App Service Environment) 可透過 Azure 虛擬網路 (VNet) 和彼此進行私下通訊。 VNet 是專屬於您[訂用帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fnetworking%2ftoc.json)的 Azure 雲端邏輯隔離。 您可以在每個 Azure 訂用帳戶和 Azure [區域](https://azure.microsoft.com/regions)內實作多個 VNet。 每個 VNet 會與其他 VNet 隔離。 對於每個 VNet，您可以︰

- 使用公用和私人 (RFC 1918) 位址指定自訂私人 IP 位址空間。 Azure 會從您指派的位址空間，將私人 IP 位址指派給連線至 VNet 的資源。
- 將 VNet 分成一或多個子網路，並將 VNet 位址空間的一部分配置給每個子網路。
- 使用 Azure 提供的名稱解析，或指定自有的 DNS 伺服器以供連線至 VNet 的資源使用。

若要深入了解 Azure 虛擬網路服務，請閱讀[虛擬網路概觀](../virtual-network/virtual-networks-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。 您可以將 VNet 互相連線，讓連線至任一 VNet 的資源能夠跨越 VNet 彼此通訊。 您可以使用下列一個或兩個選項將 VNet 彼此連線：

- **對等互連︰** 讓連線至相同 Azure 區域內不同 Azure VNet 的資源彼此通訊。 如果資源已連線到相同的 VNet，則跨越 VNet 的頻寬和延遲就一樣。 若要深入了解對等互連，請閱讀[虛擬網路對等互連概觀](../virtual-network/virtual-network-peering-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **VPN 閘道︰** 讓連線至不同 Azure 區域內不同 Azure VNet 的資源彼此通訊。 VNet 之間的流量流經 Azure VPN 閘道。 VNet 之間的頻寬受限於頻寬的閘道。 若要深入了解將 VNet 與 VPN 閘道連線，請閱讀[跨區域設定 VNet 對 VNet 連線](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

## <a name="internet-connectivity"></a>網際網路連線

根據預設，連線至 VNet 的所有 Azure 資源都具有網際網路的輸出連線能力。 資源的私人 IP 位址會由 Azure 基礎結構進行來源網路位址轉譯 (SNAT) 成為公用 IP 位址。 若要深入了解輸出網際網路連線能力，請閱讀[了解 Azure 中的輸出連線](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

若要進行從網際網路通對 Azure 資源的輸入通訊，或進行對網際網路的輸出通訊 (未經 SNAT)，則必須指派公用 IP 位址給資源。 若要深入了解公用 IP 位址，請閱讀[公用 IP 位址](../virtual-network/virtual-network-public-ip-address.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

## <a name="on-premises-connectivity"></a>內部部署連線

您可以透過 VPN 連線或直接的私人連線，在您的 VNet 中安全地存取資源。 若要在 Azure 虛擬網路和內部部署網路之間傳送網路流量，您必須建立虛擬網路閘道。 您要設定閘道的設定以建立所需的連線類型，即 VPN 或 ExpressRoute。

您可以使用下列選項的任意組合，將內部部署網路連線至 VNet︰

**點對站 (透過 SSTP 的 VPN)**

下圖顯示多部電腦與 VNet 之間不同的站對台連線︰

![點對站](./media/networking-overview/point-to-site.png)

在單一電腦與 VNet 之間會建立此連線。 如果您剛開始使用 Azure，此連線類型就很適合您，也適用於開發人員，因為它幾乎不需要變更您現有的網路。 這也是從會議或住家等遠端位置連線時的便利方式。 點對站連線通常會透過相同的虛擬網路閘道結合站對站連線。 連線會使用 SSTP 通訊協定，透過網際網路提供電腦與 VNet 之間的加密通訊。 點對站 VPN 的延遲無法預期，因為流量會周遊網際網路。

**站對站 (IPsec/IKE VPN 通道)**

![網站間](./media/networking-overview/site-to-site.png)

內部部署的 VPN 裝置與 Azure VPN 閘道之間會建立此連線。 此連線類型可讓您授權的任何內部部署資源存取 VNet。 此連線是 IPSec/IKE VPN，可透過網際網路提供內部部署裝置與 Azure VPN 閘道之間的加密通訊。 您可以將多個內部部署網站連線到同一個 VPN 閘道。 每個網站的內部部署 VPN 裝置，都必須有一個對外開放、不是位於 NAT 後方的公用 IP 位址。 站對站連線的延遲無法預期，因為流量會周遊網際網路。

**ExpressRoute (專用的私人連線)**

![ExpressRoute](./media/networking-overview/expressroute.png)

您的網路與 Azure 之間會透過 ExpressRoute 合作夥伴建立此類型的連線。 此連線是私人連線。 流量不會周遊網際網路。 ExpressRoute 連線的延遲無法預期，因為流量不會周遊網際網路。 ExpressRoute 可以結合站對站連線。

若要深入了解所有先前的連線選項，請閱讀[連線拓撲圖](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

## <a name="load-balancing"></a>負載平衡和流量方向

Microsoft Azure 提供多個服務，可管理分配網路流量和負載平衡的方式。 您可以合併或分開使用下列任一項功能︰

**DNS 負載平衡**

Azure 流量管理員服務提供全域 DNS 負載平衡。 流量管理員會根據下列其中一項路由方式，以狀況良好之端點的 IP 位址回應用戶端︰
- **地理︰** 根據使用者的 DNS 查詢來自哪個地理位置，將用戶端導向特定端點 (Azure、外部或巢狀)。 在必須知道用戶端所在的地理區域並根據此位置進行路由的情況下，適合採取此方式。 例如，遵守資料主權規定、內容和使用者經驗的當地語系化，以及測量來自不同區域的流量。
- **效能︰** 傳回至用戶端的 IP 位址「最靠近」用戶端。 「最靠近」的端點不一定是地理距離測量上最靠近的端點。 此方法是透過測量網路延遲來決定最靠近的端點。 流量管理員會維護「網際網路延遲資料表」，以追蹤 IP 位址範圍與每個 Azure 資料中心之間的往返時間。
- **優先順序：** 流量會導向主要 (最高優先順序) 端點。 如果主要端點無法使用，流量管理員會將流量路由傳送至第二個端點。 如果主要和次要端點都無法供使用，系統就會將流量傳送到第三個端點，依此類推。 端點的可用性是取決於已設定的狀態 (已啟用或已停用) 和持續的端點監視。
- **加權循環配置資源：** 針對每個要求，流量管理員會隨機選擇可用的端點。 選擇端點的機率是根據指派給所有可用端點的權數。 所有端點都使用相同的權數會形成平均的流量分配。 在特定端點上使用較高或較低的權數會導致 DNS 回應中較經常或較不常傳回這些端點。

下圖顯示導向 Web App 端點的 Web 應用程式要求。 端點也可以是其他 Azure 服務，例如 VM 和雲端服務。

![流量管理員](./media/networking-overview/traffic-manager.png)

用戶端直接連線至該端點。 Azure 流量管理員會在偵測到端點狀況不良時，將用戶端重新導向至其他狀況良好的端點。 若要深入了解流量管理員，請閱讀 [Azure 流量管理員概觀](../traffic-manager/traffic-manager-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

**應用程式負載平衡**

Azure 應用程式閘道服務提供應用程式傳遞控制站 (ADC) 服務。 應用程式閘道為您的應用程式提供各種第 7 層 (HTTP/HTTPS) 負載平衡功能，包括保護 Web 應用程式以防範漏洞和攻擊的 Web 應用程式防火牆。 應用程式閘道還會將 CPU 密集 SSL 終止卸載至應用程式閘道，讓客戶最佳化 Web 伺服陣列的產能。 

其他第 7 層路由功能包括循環配置連入流量、以 Cookie 為基礎的工作階段同質、URL 路徑型路由，以及在單一應用程式閘道背後代管多個網站的能力。 應用程式閘道可以設定為連結網際網路的閘道、內部專用閘道或兩者混合。 應用程式閘道完全由 Azure 管理、可調整且可用性極高。 它提供一組豐富的診斷和記錄功能，很好管理。 若要深入了解應用程式閘道，請閱讀[應用程式閘道概觀](../application-gateway/application-gateway-introduction.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

下圖顯示以 URL 路徑為基礎的路由與應用程式閘道︰

![應用程式閘道](./media/networking-overview/application-gateway.png)

**網路負載平衡**

Azure Load Balancer 針對所有 UDP 和 TCP 通訊協定提供高效能、低延遲的第 4 層負載平衡。 它會管理輸入及輸出連線。 您可以設定公用和內部負載平衡端點。 您可以使用 TCP 和 HTTP 健全狀況探查選項定義規則，將輸入連線對應至後端集區目的地，以管理服務可用性。 若要深入了解負載平衡器，請閱讀[負載平衡器概觀](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

下圖顯示同時使用外部和內部負載平衡器的網際網路對向多層應用程式︰

![負載平衡器](./media/networking-overview/load-balancer.png)

## <a name="security"></a>安全性

您可以使用下列選項來篩選 Azure 資源的輸入和輸出流量︰

- **網路︰** 您可以實作 Azure 網路安全性群組 (NSG) 來篩選 Azure 資源的輸入和輸出流量。 每個 NSG 可包含一或多個輸入和輸出規則。 每個規則可指定用來篩選流量的來源 IP 位址、目的地 IP 位址、連接埠和通訊協定。 NSG 可套用至個別的子網路和個別的 VM。 若要深入了解 NSG，請閱讀[網路安全性群組概觀](../virtual-network/security-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **應用程式︰** 同時使用應用程式閘道和 Web 應用程式防火牆，可保護您的應用程式以防範漏洞和攻擊。 常見的範例包括 SQL 插入式攻擊、跨網站指令碼和格式不正確的標頭。 應用程式閘道會篩選掉此流量，使它無法到達您的 Web 伺服器。 您可以設定要啟用的規則。 您可以設定 SSL 交涉原則，以允許停用某些原則。 若要深入了解 Web 應用程式防火牆，請閱讀 [Web 應用程式防火牆](../application-gateway/application-gateway-web-application-firewall-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

如果您需要 Azure 未提供的網路功能，或想要使用您在內部部署使用的網路應用程式，您可以在 VM 中實作這些產品，並將它們連線到您的 VNet。 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances) 包含數種不同的 VM，這些 VM 已預先設定您目前可能使用的網路應用程式。 這些預先設定的 VM 通常稱為網路虛擬裝置 (NVA)。 NVA 已搭載防火牆和 WAN 最佳化等應用程式。

## <a name="routing"></a>路由

Azure 預設會建立路由表，讓連線至任何 VNet 中任何子網路的資源彼此通訊。 您可以實作下列一或兩個類型的路由，覆寫 Azure 所建立的預設路由︰
- **使用者定義的︰** 您可以建立自訂路由表，其中的路由可控制每個子網路的流量會路由傳送至的位置。 若要深入了解使用者定義的路由，請閱讀[使用者定義的路由](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)文章。
- **邊界閘道協定 (BGP)︰** 如果您使用 Azure VPN 閘道或 ExpressRoute 連線將 VNet 連線至內部部署網路，則可將 BGP 路由傳播至 VNet。 BGP 是常用於網際網路的標準路由通訊協定，可交換兩個或多個網路之間的路由和可執行性資訊。 在 Azure 虛擬網路的內容中使用時，BGP 會啟用 Azure VPN 閘道，以及內部部署 VPN 裝置 (稱為 BGP 對等互連或鄰近項目) 來交換「路由」，其會通知這兩個閘道對要通過閘道的首碼或所涉及之路由器的可用性和可執行性。 BGP 也可以傳播從一個 BGP 對等互連到所有其他 BGP 對等所識別的 BGP 閘道，來啟用多個網路之間的傳輸路由。 若要深入了解 BGP，請參閱 [BGP 搭配 Azure VPN 閘道概觀](../vpn-gateway/vpn-gateway-bgp-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

## <a name="manageability"></a>可管理性

Azure 提供下列工具來監視和管理網路功能︰
- **活動記錄︰** 所有 Azure 資源都有提供作業發生、作業狀態和起始作業者的活動記錄。 若要深入了解活動記錄，請閱讀[活動記錄概觀](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **診斷記錄：** 網路資源會定期和自發地建立事件，並記錄到 Azure 儲存體帳戶、傳送到 Azure 事件中樞或 Azure Log Analytics。 診斷記錄可讓您深入了解資源的健全狀況。 負載平衡器 (網際網路對向)、網路安全性群組、路由和應用程式閘道均提供診斷記錄。 若要深入了解診斷記錄，請閱讀[診斷記錄概觀](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **計量：** 計量是在一段時間內所收集到關於資源的效能測量數據和計數器。 計量可用來根據臨界值觸發警示。 目前有針對應用程式閘道的計量。 若要深入了解計量，請閱讀[計量概觀](../monitoring-and-diagnostics/monitoring-overview-metrics.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **疑難排解︰** 在 Azure 入口網站中，可直接存取疑難排解資訊。 此資訊有助於診斷和下列網路資源有關的常見問題：ExpressRoute、VPN 閘道、應用程式閘道、網路安全性記錄、路由、DNS、負載平衡器和流量管理員。
- **角色型存取控制 (RBAC)：** 使用角色型存取控制 (RBAC)，控制誰可以建立和管理網路資源。 若要深入了解 RBAC，請閱讀[開始使用 RBAC](../role-based-access-control/overview.md?toc=%2fazure%2fnetworking%2ftoc.json) 一文。 
- **封包擷取︰** Azure 網路監看員服務可透過 VM 內的擴充功能，在 VM 上執行封包擷取。 Linux 和 Windows VM 均提供此功能。 若要深入了解封包擷取，請閱讀[封包擷取概觀](../network-watcher/network-watcher-packet-capture-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **驗證 IP 流量：** 網路監看員可讓您驗證 Azure VM 和遠端資源之間的 IP 流量，以決定允許或拒絕封包。 系統管理員可利用此功能快速診斷連線問題。 若要深入了解如何驗證 IP 流量，請閱讀 [IP 流量驗證概觀](../network-watcher/network-watcher-ip-flow-verify-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **疑難排解 VPN 連線︰** 網路監看員的 VPN 疑難排解員功能可查詢連線或閘道，以確認資源的健全狀況。 若要深入了解對 VPN 連線進行疑難排解，請閱讀 [VPN 連線疑難排解概觀](../network-watcher/network-watcher-troubleshoot-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **檢視網路拓撲︰** 使用網路監看員，以圖形化方式檢視 VNet 中的網路資源。 若要深入了解檢視網路拓撲，請閱讀[拓撲概觀](../network-watcher/network-watcher-topology-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。

## <a name="tools"></a>部署和設定工具

您可以使用下列任一項工具來部署和設定 Azure 網路資源︰

- **Azure 入口網站︰** 在瀏覽器中執行的圖形化使用者介面。 開啟 [Azure 入口網站](http://portal.azure.com)。
- **Azure PowerShell：** 從 Windows 電腦管理 Azure 的命令列工具。 若要深入了解 Azure PowerShell，請閱讀 [Azure PowerShell 概觀](/powershell/azure/overview?view=azurermps-3.8.0?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **Azure 命令列介面 (CLI)：** 從 Linux、macOS 或 Windows 電腦管理 Azure 的命令列工具。 若要深入了解 Azure CLI，請閱讀 [Azure CLI 概觀](/cli/azure/get-started-with-azure-cli?toc=%2fazure%2fnetworking%2ftoc.json)一文。
- **Azure Resource Manager 範本：** 定義 Azure 解決方案之基礎結構和組態的檔案 (JSON 格式)。 透過範本，您可以在整個生命週期中重複部署方案，並確信您的資源會以一致的狀態部署。 若要深入了解製作範本，請閱讀[建立範本的最佳做法](../azure-resource-manager/resource-manager-template-best-practices.md?toc=%2fazure%2fnetworking%2ftoc.json)一文。 您可以使用 Azure 入口網站、CLI 或 PowerShell 來部署範本。 若要立即開始使用範本，請部署 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/?term=network)程式庫中的其中一個預先設定範本。 

## <a name="pricing"></a>價格

有些 Azure 網路服務需要收費，其他則是免費提供。 如需詳細資訊，請參閱[虛擬網路](https://azure.microsoft.com/pricing/details/virtual-network)、[VPN 閘道](https://azure.microsoft.com/pricing/details/vpn-gateway)、[應用程式閘道](https://azure.microsoft.com/pricing/details/application-gateway/)、[負載平衡器](https://azure.microsoft.com/pricing/details/load-balancer)、[網路監看員](https://azure.microsoft.com/pricing/details/network-watcher)、[DNS](https://azure.microsoft.com/pricing/details/dns)、[流量管理員](https://azure.microsoft.com/pricing/details/traffic-manager) 和 [ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute) 的定價頁面。

## <a name="next-steps"></a>後續步驟

- 完成[建立第一個虛擬網路](../virtual-network/quick-create-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)一文中的步驟，以建立第一個 VNet，並將一些 VM 連線至該 VNet。
- 完成[設定點對站連線](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)一文中的步驟，以將您的電腦連線至 VNet。
- 完成[建立網際網路對向負載平衡器](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)一文中的步驟，將網際網路流量負載平衡到公用伺服器。
