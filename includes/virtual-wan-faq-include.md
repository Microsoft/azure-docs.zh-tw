---
title: 包含檔案
description: 包含檔案
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 10/05/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 761b68ca99df8ae5b4d379b95e7d2a300f7e6238
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48874128"
---
### <a name="what-is-the-difference-between-an-azure-virtual-network-gateway-vpn-gateway-and-an-azure-virtual-wan-vpngateway"></a>Azure 虛擬網路閘道 (VPN 閘道) 與 Azure 虛擬 WAN vpngateway 之間有什麼差異？

虛擬 WAN 提供大規模站對站連線，而且是針對輸送量、延展性和易用性而建置的。 ExpressRoute 與點對站連線功能目前為預覽狀態。 CPE 分支裝置會自動佈建，並且連線到 Azure 虛擬 WAN。 這些裝置都可以從 SD-WAN 和 VPN 合作夥伴日益成長的生態系統中取用。 請參閱[慣用的夥伴清單](https://go.microsoft.com/fwlink/p/?linkid=2019615)。

### <a name="which-device-providers-virtual-wan-partners-are-supported-at-launch-time"></a>啟動時支援哪些裝置提供者 (虛擬 WAN 合作夥伴)？ 

目前有許多合作夥伴支援完全自動化的虛擬 WAN 體驗。 如需詳細資訊，請參閱[虛擬 WAN 合作夥伴](https://go.microsoft.com/fwlink/p/?linkid=2019615)。 

### <a name="what-are-the-virtual-wan-partner-automation-steps"></a>虛擬 WAN 合作夥伴自動化步驟是什麼？

若要了解合作夥伴自動化步驟，請參閱[虛擬 WAN 合作夥伴自動化](../articles/virtual-wan/virtual-wan-configure-automation-providers.md)。

### <a name="am-i-required-to-use-a-preferred-partner-device"></a>我是否必須使用偏好的合作夥伴裝置？

否。 您可以使用任何支援 VPN 且符合 IKEv2/IKEv1 IPsec 支援需求的裝置。

### <a name="how-do-virtual-wan-partners-automate-connectivity-with-azure-virtual-wan"></a>虛擬 WAN 合作夥伴如何將與 Azure 虛擬 WAN 的連線自動化？

軟體定義的連線解決方案通常會使用控制器或裝置佈建中心，來管理它們的分支裝置。 控制器可以使用 Azure API，將與 Azure 虛擬 WAN 的連線自動化。 如需詳細資訊，請參閱虛擬 WAN 夥伴自動化。

### <a name="does-virtual-wan-change-any-existing-connectivity-features"></a>虛擬 WAN 是否會變更任何現有的連線功能？   

現有的 Azure 連線功能不會有任何變更。

### <a name="are-there-new-resource-manager-resources-available-for-virtual-wan"></a>虛擬 WAN 是否有任何新的 Resource Manager 資源可用？
  
是，虛擬 WAN 導入了新的 Resource Manager 資源。 如需詳細資訊，請參閱[概觀](https://go.microsoft.com/fwlink/p/?LinkId=2004389)。

### <a name="how-many-vpn-devices-can-connect-to-a-single-hub"></a>有多少部 VPN 裝置可以連線到單一中樞？

每個虛擬中樞支援多達 1000 個連線。 每個連線是由兩個通道所組成，其組態為主動-主動。 通道會在 Azure 虛擬中樞 vpngateway 內終止。

### <a name="can-the-on-premises-vpn-device-connect-to-multiple-hubs"></a>內部部署 VPN 裝置是否可以連線到多個中樞？

是。 開始時的流量會從內部部署裝置流至最近的 Microsoft 邊緣，然後流至虛擬中樞。

### <a name="is-global-vnet-peering-supported-with-azure-virtual-wan"></a>Azure 虛擬 WAN 是否支援全域 VNet 對等互連？ 

 否。

### <a name="can-spoke-vnets-connected-to-a-virtual-hub-communicate-with-each-other"></a>連線到虛擬中樞的輪輻 VNet 之間，是否可以彼此通訊？

是。 您可以在連線到虛擬中樞的輪輻之間直接進行 VNet 對等互連。 如需詳細資訊，請參閱[虛擬網路對等互連](../articles/virtual-network/virtual-network-peering-overview.md)。

### <a name="can-i-deploy-and-use-my-favorite-network-virtual-appliance-in-an-nva-vnet-with-azure-virtual-wan"></a>是否可以使用 Azure 虛擬 WAN 部署及使用我的最愛網路虛擬設備 (在 NVA VNet 中)？

是，您可以將您最愛的網路虛擬設備 (NVA) VNet 連線到 Azure 虛擬 WAN。 首先，使用中樞虛擬網路連線將網路虛擬設備 VNet 連線到中樞。 接著，建立包含指向虛擬設備之下一個躍點的虛擬中樞路由。 您可以將多個路由套用到虛擬中樞路由表。 連線到 NVA VNet 的任何輪輻還必須連線到虛擬中樞，以確保輪輻 VNet 路由會傳播到內部部署系統。

### <a name="can-an-nva-vnet-have-a-virtual-network-gateway"></a>NVA VNet 是否可以有虛擬網路閘道？

否。 如果 NVA VNet 連線到虛擬中樞，則不能有虛擬網路閘道。 

### <a name="is-there-support-for-bgp"></a>是否支援 BGP？

是，有支援 BGP。 若要確保來自 NVA VNet 的路由會適當地公告，如果輪輻已連線至 NVA VNet，後者接著連線至虛擬中樞，則輪輻必須停用 BGP。 此外，請將輪輻 VNet 連線到虛擬中樞，以確保輪輻 VNet 路由會傳播到內部部署系統。

### <a name="can-i-direct-traffic-using-udr-in-the-virtual-hub"></a>是否可以使用虛擬中樞的 UDR 來引導流量？

是，您可以使用虛擬中樞路由表將流量導向 VNet。

### <a name="is-there-any-licensing-or-pricing-information-for-virtual-wan"></a>虛擬 WAN 是否有任何授權或價格資訊？
 
是。 請參閱[定價](https://azure.microsoft.com/pricing/details/virtual-wan/)頁面。

### <a name="how-do-new-partners-that-are-not-listed-in-your-launch-partner-list-get-onboarded"></a>未列在產品發佈合作夥伴清單中的新合作夥伴要如何上線？

將電子郵件傳送至 azurevirtualwan@microsoft.com。 理想的合作夥伴是有裝置可用來佈建 IKEv1 或 IKEv2 IPSec 連線的合作夥伴。

### <a name="is-it-possible-to-construct-azure-virtual-wan-with-a-resource-manager-template"></a>是否可以使用 Resource Manager 範本來建構 Azure 虛擬 WAN？

可以使用 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network)來建立一個簡易設定的虛擬 WAN，其中包含一個中樞和一個 vpnsite。 虛擬 WAN 主要是 REST 或入口網站驅動的服務。

### <a name="is-branch-to-branch-connectivity-allowed-in-virtual-wan"></a>虛擬 WAN 中是否允許分支對分支連線？

是，在 VPN 的虛擬 WAN 與 VPN 對 ExpressRoute 中可以使用分支對分支連線。 雖然 VPN 站對站是已公開推出，但 ExpressRoute 與點對站目前為預覽狀態。

### <a name="does-branch-to-branch-traffic-traverse-through-the-azure-virtual-wan"></a>分支對分支流量是否會透過 Azure 虛擬 WAN 而周遊？

是。

### <a name="how-is-virtual-wan-different-from-the-existing-azure-virtual-network-gateway"></a>虛擬 WAN 與現有的 Azure 虛擬網路閘道有何不同？

虛擬網路閘道 VPN 限定為 30 個通道。 對於連線，您應將虛擬 WAN 用於大規模的 VPN。 您可以在所有區域 (中西部區域除外) 的中樞內以 2 Gbps 連線到最多 1000 個分支連線。 中西部區域可使用 20 Gbps。 未來我們會將 20 Gbps 推展到其他區域。 連線是從內部部署 VPN 裝置到虛擬中樞的主動-主動通道。 您可以在每個區域都有一個中樞，這表示您可以跨中樞連線到超過 1000 個分支。

### <a name="does-this-virtual-wan-require-expressroute-from-each-site"></a>此虛擬 WAN 是否需要來自每個網站的 ExpressRoute？

否，此虛擬 WAN 不需要來自每個網站的 ExpressRoute。 它會透過從裝置到 Azure 虛擬 WAN 中樞的網際網路連結，使用標準 IPsec 站對站連線。 您的網站可以使用 ExpressRoute 線路連線到提供者網路。 針對虛擬中樞內使用 ExpressRoute 來連線的網站，它們可以具有分支，以在 VPN 與 ExpressRoute 之間將流量分流。 

### <a name="is-there-a-network-throughput-limit-when-using-azure-virtual-wan"></a>使用 Azure 虛擬 WAN 時是否有網路輸送量限制？

每個中樞/區域的分支數目限制為 1000 個連線，且中樞內的合併連線速度限制為 2 G。 美國中西部例外，其總計為 20 Gbps。 未來我們會將 20 Gbps 推展到其他區域。

### <a name="does-virtual-wan-allow-the-on-premises-device-to-utilize-multiple-isps-in-parallel-or-is-it-always-a-single-vpn-tunnel"></a>虛擬 WAN 是否允許內部部署裝置以平行方式使用多個 ISP，還是一律為單一 VPN 通道？

是，您可以根據分支裝置，使用來自單一分支的主動-主動通道 (2 個通道 = 1 個 Azure 虛擬 WAN 連線)。

### <a name="how-is-traffic-routed-on-the-azure-backbone"></a>流量在 Azure 骨幹上的路由傳送方式為何？

流量會依循下列模式：分支裝置->ISP->Microsoft Edge->Microsoft DC->Microsoft Edge->ISP->分支裝置

### <a name="in-this-model-what-do-you-need-at-each-site-just-an-internet-connection"></a>在此模型中，每個站台需要什麼？ 有網際網路連線就可以嗎？

是。 網際網路連線與實體裝置；最好是我們的整合式[合作夥伴](https://go.microsoft.com/fwlink/p/?linkid=2019615)所提供的。 您可以選擇從慣用的裝置手動管理 Azure 的設定與連線。
