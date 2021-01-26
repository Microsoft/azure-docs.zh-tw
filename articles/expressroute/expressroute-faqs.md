---
title: 常見問題集 - Azure ExpressRoute | Microsoft Docs
description: ExpressRoute 常見問題集包含支援的 Azure 服務、費用、資料和連線、SLA、提供者和位置、頻寬等資訊及其他技術詳細資料。
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: duau
ms.openlocfilehash: 1be7331b0c2309350316d1c88c54e6018400463c
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98789342"
---
# <a name="expressroute-faq"></a>ExpressRoute 常見問題集

## <a name="what-is-expressroute"></a>什麼是 ExpressRoute？

ExpressRoute 這項 Azure 服務可讓您在 Microsoft 資料中心和內部部署或共置設備中的基礎結構之間建立私人連線。 ExpressRoute 連線不會經過公用網際網路，相較於網際網路一般連線，它提供了更高的安全性、可靠性、速度以及更低的延遲。

### <a name="what-are-the-benefits-of-using-expressroute-and-private-network-connections"></a>使用 ExpressRoute 和私人網路連線的優勢有哪些？

ExpressRoute 連線不會經過公用網際網路。 相較於網際網路一般連線，它們提供了更高的安全性、可靠性、速度，以及更低且一致的延遲。 在某些情況下，使用 ExpressRoute 連線在內部部署裝置和 Azure 之間傳輸資料，也可以產生重大的成本效益。

### <a name="where-is-the-service-available"></a>哪裡可以使用此服務？

請參閱以下頁面，以取得服務位置和可用性：[ExpressRoute 合作夥伴和位置](expressroute-locations.md)。

### <a name="how-can-i-use-expressroute-to-connect-to-microsoft-if-i-dont-have-partnerships-with-one-of-the-expressroute-carrier-partners"></a>如果我和其中一個 ExpressRoute 載波合作夥伴沒有合作關係，我要如何使用 ExpressRoute 來與 Microsoft 連線？

您可以在區域載波和陸地乙太網路連線中選取其中一個支援的 Exchange 提供者位置。 您接著可以在提供者位置上與 Microsoft 對等互連。 檢查 [ExpressRoute 合作夥伴和位置](expressroute-locations.md) 的最後一部分，以查看您的服務提供者是否存在於任何 Exchange 位置中。 您接著可以訂購一個 ExpressRoute 電路，透過服務提供者連線至 Azure。

### <a name="how-much-does-expressroute-cost"></a>ExpressRoute 需要多少錢？

如需價格資訊，請查看 [價格詳細資訊](https://azure.microsoft.com/pricing/details/expressroute/) 。

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-do-i-have-this-bandwidth-allocated-for-ingress-and-egress-traffic-separately"></a>如果我支付指定頻寬的 ExpressRoute 線路，是否另外配置了此頻寬給輸入和輸出流量？

是的，ExpressRoute 線路頻寬是雙工。 例如，如果您購買 200 mbps ExpressRoute 線路，則會採購 200 mbps 的輸入流量，以及 200 mbps 的輸出流量。

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-does-the-vpn-connection-i-purchase-from-my-network-service-provider-have-to-be-the-same-speed"></a>如果我支付指定頻寬的 ExpressRoute 電路，則我是否必須從網路服務提供者購買相同速度的 VPN 連線？

否。 您可以從服務提供者購買任何速度的 VPN 連線。 不過，您與 Azure 的連線受限於您所購買的 ExpressRoute 線路頻寬。

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-do-i-have-the-ability-to-use-more-than-my-procured-bandwidth"></a>如果我支付指定頻寬的 ExpressRoute 線路，是否能夠使用超過購買頻寬的能力？

是，您可以使用 ExpressRoute 線路的次要連線上可用的頻寬，最多可使用您所購買之頻寬限制的兩倍。 您線路的內建冗余設定是使用主要和次要連線（每個採購頻寬）設定為兩個 Microsoft Enterprise Edge 路由器 (Msee) 。 如有必要，可透過次要連線使用的頻寬可用於額外的流量。 因為次要連線是用來進行冗余，所以不保證，且不應該用於持續時間的額外流量。 若要深入瞭解如何使用這兩個 connnections 來傳輸流量，請參閱 [使用 AS 路徑](./expressroute-optimize-routing.md#solution-use-as-path-prepending)前置。

如果您打算只使用您的主要連線來傳輸流量，則連線的頻寬是固定的，而且嘗試超額訂閱將會導致封包下降。 如果流量流經 ExpressRoute 閘道，閘道 SKU 的頻寬是固定的，而且不會高載。 如需每個閘道 SKU 的頻寬，請參閱 [關於 ExpressRoute 虛擬網路閘道](expressroute-about-virtual-network-gateways.md#aggthroughput)。

### <a name="can-i-use-the-same-private-network-connection-with-virtual-network-and-other-azure-services-simultaneously"></a>我是否可以在虛擬網路和其他 Azure 服務中同時使用相同的私人網路連線？

是。 設定 ExpressRoute 線路之後，您就可以同時存取虛擬網路內部的服務和其他 Azure 服務。 您可透過私用對等互連路徑連線至虛擬網路，並透過 Microsoft 對等互連路徑連線至其他服務。

### <a name="how-are-vnets-advertised-on-expressroute-private-peering"></a>如何在 ExpressRoute 私人對等互連上公告 Vnet？

ExpressRoute 閘道會公告 Azure VNet 的 *位址空間*，您無法在子網層級包含/排除這些位址空間。 其一律是公告的 VNet 位址空間。 此外，如果使用 VNet 對等互連，且對等互連 VNet 已啟用「使用遠端閘道」，則也會公告對等互連 VNet 的位址空間。

### <a name="how-many-prefixes-can-be-advertised-from-a-vnet-to-on-premises-on-expressroute-private-peering"></a>在 ExpressRoute 私人對等互連上，有多少首碼可以從 VNet 公告至內部部署？

在單一 ExpressRoute 連線上或使用閘道傳輸透過 VNet 對等互連，最多可以公告 200 個首碼。 例如，如果您在連線到 ExpressRoute 線路的單一 VNet 上有 199 個位址空間，這些首碼的所有 199 個位址空間都會公告至內部部署。 或者，如果您已啟用 VNet 來允許閘道傳輸，並使用 [允許遠端閘道] 選項啟用 1 個位址空間和 150 個輪輻 Vnet，則使用閘道部署的 VNet 會將 151 個首碼公告至內部部署。

### <a name="what-happens-if-i-exceed-the-prefix-limit-on-an-expressroute-connection"></a>如果超過 ExpressRoute 連線上的首碼限制，會發生什麼事？

ExpressRoute 線路與閘道之間的連線 (以及使用閘道傳輸的對等互連 Vnet，如果適用的話) 將會關閉。 不再超過首碼限制時，就會重新建立連線。  

### <a name="can-i-filter-routes-coming-from-my-on-premises-network"></a>可以篩選來自內部部署網路的路由嗎？

篩選/包含路由的唯一方式是在內部部署邊緣路由器上進行。 使用者定義的路由可以新增在 VNet 中，以影響特定的路由，但這會是靜態路由，而不是 BGP 公告的一部分。

### <a name="does-expressroute-offer-a-service-level-agreement-sla"></a>ExpressRoute 是否提供服務等級協定 (SLA)？

如需詳細資訊，請參閱 [ExpressRoute SLA](https://azure.microsoft.com/support/legal/sla/) 頁面。

## <a name="supported-services"></a>支援的服務

ExpressRoute 針對各種類型的服務支援[三個路由網域](expressroute-circuit-peerings.md)：私人對等互連、Microsoft 對等互連和公用對等互連 (已取代)。

### <a name="private-peering"></a>私人對等互連

**支援：**

* 包括所有虛擬機器和雲端服務在內的虛擬網路

### <a name="microsoft-peering"></a>Microsoft 對等互連

如果已針對 Azure Microsoft 對等互連啟用您的 ExpressRoute 線路，就可以透過線路存取在 Azure 中使用的[公用 IP 位址範圍](../virtual-network/public-ip-addresses.md#public-ip-addresses)。 Azure Microsoft 對等互連可讓您存取目前裝載於 Azure 上的服務 (是否有地理限制，視您的線路 SKU 而定)。 若要驗證特定服務的可用性，您可以參閱該服務的文件，以查看是否有針對該服務發佈的保留範圍。 然後，查閱目標服務的 IP 範圍，並與 [Azure IP 範圍和服務標籤 – 公用雲端 XML 檔案](https://www.microsoft.com/download/details.aspx?id=56519)中列出的範圍進行比較。 或者，您也可以針對有問題的服務開啟支援票證，以釐清狀況。

**支援：**

* [Microsoft 365](/microsoft-365/enterprise/azure-expressroute)
* Power BI - 可透過 Azure 區域社群取得。請參閱[這裡](/power-bi/service-admin-where-is-my-tenant-located)，以了解如何找出 Power BI 租用戶的區域。
* Azure Active Directory
* [Azure DevOps](https://blogs.msdn.microsoft.com/devops/2018/10/23/expressroute-for-azure-devops/) (Azure 全域服務社群)
* 適用于 IaaS (虛擬機器、虛擬網路閘道、負載平衡器等 ) 的 Azure 公用 IP 位址  
* 也支援大部分的其他 Azure 服務。 請直接檢查您要用來驗證支援的服務。

**不支援：**

* CDN
* Azure Front Door
* [Windows 虛擬桌面](https://azure.microsoft.com/services/virtual-desktop/)
* Multi-factor Authentication Server (舊版)
* 流量管理員

### <a name="public-peering"></a>公用對等互連

新的 ExpressRoute 線路已停用公用對等互連。 Azure 服務現在可以使用 Microsoft 對等互連。 如果您在公用對等互連被取代之前建立了線路，則可以選擇使用 Microsoft 對等互連或公用對等互連，視您想要的服務而定。

如需公用對等互連的詳細資訊和設定步驟，請參閱 [ExpressRoute 公用對等互連](about-public-peering.md)。

### <a name="why-i-see-advertised-public-prefixes-status-as-validation-needed-while-configuring-microsoft-peering"></a>在設定 Microsoft 對等互連時，為什麼我會看到狀態為「需要驗證」的「已公告公用首碼」？

Microsoft 會驗證指定的「已公告公用首碼」和「對等互連 ASN」(或「客戶 ASN」) 是否已在網際網路路由登錄中指派給您。 如果您要從另一個實體取得公用首碼，以及如果未使用路由登錄來記錄指派，則自動驗證將不會完成，而且需要手動驗證。 如果自動驗證失敗，您會看到「需要驗證」訊息。

如果您看到「需要驗證」訊息，請收集其中顯示實體已將公用首碼指派給您組織的文件，而此實體列示為路由登錄中的首碼擁有者，然後開啟支援票證，提交這些文件進行手動驗證，如下所示。

![顯示新的支援要求 (支援票證的螢幕擷取畫面，) 「公用首碼的擁有權證明」。](./media/expressroute-faqs/ticket-portal-msftpeering-prefix-validation.png)

### <a name="is-dynamics-365-supported-on-expressroute"></a>ExpressRoute 支援 Dynamics 365 嗎？

Dynamics 365 和 Common Data Service (CD) 環境裝載於 Azure 上，因此客戶可以從 Azure 資源的基礎 ExpressRoute 支援獲益。 如果您的路由器篩選條件包含 Dynamics 365/CD 環境裝載所在的 Azure 區域，則您可以連線到其服務端點。

> [!NOTE]
> 如果 ExpressRoute 線路是部署在相同的 [地緣政治區域](./expressroute-locations-providers.md#expressroute-locations)內，則透過 Azure Expressroute 的 Dynamics 365 連線 **不** 需要 [expressroute Premium](#expressroute-premium) 。

## <a name="data-and-connections"></a>資料與連線

### <a name="are-there-limits-on-the-amount-of-data-that-i-can-transfer-using-expressroute"></a>我可以使用 ExpressRoute 傳輸的資料量是否有所限制？

我們不會設定資料傳輸量的限制。 如需頻寬費率的相關資訊，請參閱 [價格詳細資訊](https://azure.microsoft.com/pricing/details/expressroute/) 。

### <a name="what-connection-speeds-are-supported-by-expressroute"></a>ExpressRoute 支援的連線速度為何？

支援的頻寬供應項目：

50 Mbps、100 Mbps、200 Mbps、500 Mbps、1 Gbps、2 Gbps、5 Gbps、10 Gbps

### <a name="which-service-providers-are-available"></a>可以使用的服務提供者有哪些？

如需服務提供者和位置清單，請參閱 [ExpressRoute 合作夥伴和位置](expressroute-locations.md) 。

## <a name="technical-details"></a>技術詳細資訊

### <a name="what-are-the-technical-requirements-for-connecting-my-on-premises-location-to-azure"></a>將我的內部部署位置連線至 Azure 會有哪些技術需求？

請參閱 [ExpressRoute 必要條件頁面](expressroute-prerequisites.md)以取得相關需求。

### <a name="are-connections-to-expressroute-redundant"></a>連線至 ExpressRoute 是多餘的嗎？

是。 為提供高可用性，每個 ExpressRoute 線路都設有交叉連線的備援配對。

### <a name="will-i-lose-connectivity-if-one-of-my-expressroute-links-fail"></a>如果我的其中一個 ExpressRoute 連結失敗，連線是否就會中斷？

如果其中一個交叉連線失敗，您的連線就會中斷。 有備援連線可支援您的網路負載，並為 ExpressRoute 線路提供高可用性。 為達到線路層級的復原能力，您可以在其他對等互連位置另外建立線路。

### <a name="how-do-i-implement-redundancy-on-private-peering"></a>如何在私人對等互連上實作備援？

來自不同對等互連位置的多個 ExpressRoute 線路，或來自相同對等互連位置的四個連線，可以連接到相同的虛擬網路，以在單一線路無法使用時提供高可用性。 然後，您可以 [將較高](./expressroute-optimize-routing.md#solution-assign-a-high-weight-to-local-connection) 的權數指派給其中一個本機連接，以使用特定的線路。 強烈建議客戶至少設定兩個 ExpressRoute 線路，以避免發生單一失敗點。 

如需高可用性的設計，請參閱[這裡](./designing-for-high-availability-with-expressroute.md)，如需災害復原的設計，請參閱[這裡](./designing-for-disaster-recovery-with-expressroute-privatepeering.md)。  

### <a name="how-i-do-implement-redundancy-on-microsoft-peering"></a>我要如何在 Microsoft 對等互連上實作備援？

當客戶使用 Microsoft 對等互連來存取 Azure 公用服務（例如 Azure 儲存體或 Azure SQL），以及使用 Microsoft 對等互連來 Microsoft 365 的客戶在不同的對等互連位置中執行多個線路，以避免單一失敗點時，強烈建議使用這項功能。 客戶可以在這兩個線路上公告相同的首碼，並使用 [AS PATH 前置](./expressroute-optimize-routing.md#solution-use-as-path-prepending) 或公告不同的首碼，以判斷內部部署的路徑。

如需高可用性的設計，請參閱[這裡](./designing-for-high-availability-with-expressroute.md)。

### <a name="how-do-i-ensure-high-availability-on-a-virtual-network-connected-to-expressroute"></a>如何確保在連線到 ExpressRoute 之虛擬網路上的高可用性？

您可以在相同的對等互連位置中連接到您虛擬網路的多個 ExpressRoute 線路，或連接不同對等互連位置中的 ExpressRoute 線路，以達到高可用性 (例如，新加坡新加坡 2) 至虛擬網路。 如果有一個 ExpressRoute 線路當機時，連線就會容錯移轉到另一個 ExpressRoute 線路。 根據預設，離開您虛擬網路的流量會以等價多路徑路由 (ECMP) 作為基礎進行路由。 您可以使用「連線權數」來偏好兩條線路中的某一條。 如需詳細資訊，請參閱[最佳化 ExpressRoute 路由](expressroute-optimize-routing.md)。

### <a name="how-do-i-ensure-that-my-traffic-destined-for-azure-public-services-like-azure-storage-and-azure-sql-on-microsoft-peering-or-public-peering-is-preferred-on-the-expressroute-path"></a>如何確保我在 Microsoft 對等互連或公用對等互連上傳送至 Azure 公用服務 (例如 Azure 儲存體和 Azure SQL) 的流量是 ExpressRoute 路徑上偏好的流量？

您必須在路由器上實作 *本機喜好設定* 屬性，以確保從內部部署至 Azure 的路徑一律是 ExpressRoute 線路上偏好的路徑。

如需有關 BGP 路徑選擇和常用路由器組態的其他詳細資料，請參閱[這裡](./expressroute-optimize-routing.md#path-selection-on-microsoft-and-public-peerings)。 

### <a name="if-im-not-co-located-at-a-cloud-exchange-and-my-service-provider-offers-point-to-point-connection-do-i-need-to-order-two-physical-connections-between-my-on-premises-network-and-microsoft"></a><a name="onep2plink"></a>如果我不要在雲端交換中共置，而我的服務提供者提供點對點連線，我需要在內部部署網路與 Microsoft 之間訂購兩個實體連線嗎？

如果您的服務提供者可以透過實體連線建立兩個乙太網路的虛擬線路，您就只需要一個實體連線。 實體連線 (例如光纖) 的終點在實體層 1 (L1) 裝置 (請見下圖)。 兩個乙太網路虛擬電路都會標記不同的 VLAN ID，一個供主要線路使用，一個供次要線路使用。 這些 VLAN ID 位於外部 802.1Q 乙太網路標頭中。 內部 802.1Q 乙太網路標頭 (不顯示) 會對應至特定的 [ExpressRoute 路由網域](expressroute-circuit-peerings.md)。

![此圖會反白顯示第1層 (L1) 主要和次要虛擬電路，以組成客戶網站上的交換器和 ExpressRoute 位置之間的實體連線。](./media/expressroute-faqs/expressroute-p2p-ref-arch.png)

### <a name="can-i-extend-one-of-my-vlans-to-azure-using-expressroute"></a>我可以使用 ExpressRoute 來擴充其中一個至 Azure 的 VLAN 嗎？

否。 我們不支援至 Azure 的第 2 層連線擴充程式。

### <a name="can-i-have-more-than-one-expressroute-circuit-in-my-subscription"></a>我的訂用帳戶是否可以有多個 ExpressRoute 電路？

是。 您的訂用帳戶可以有多個 ExpressRoute 電路。 預設限制是設定為 10。 如需提高限制，請連絡 Microsoft 支援服務。

### <a name="can-i-have-expressroute-circuits-from-different-service-providers"></a>我可以使用其他服務提供者的 ExpressRoute 電路嗎？

是。 您可以使用許多服務提供者的 ExpressRoute 電路。 每個 ExpressRoute 線路只會與一個服務提供者產生關聯。 

### <a name="i-see-two-expressroute-peering-locations-in-the-same-metro-for-example-singapore-and-singapore2-which-peering-location-should-i-choose-to-create-my-expressroute-circuit"></a>我在相同的城市中看到兩個 ExpressRoute 對等互連位置，例如新加坡和新加坡 2。 我應該選擇哪一個對等互連位置來建立我的 ExpressRoute 線路？
如果您的服務提供者在兩個地點都有提供 ExpressRoute，您可以與提供者合作，挑選其中一個地點來設定 ExpressRoute。 

### <a name="can-i-have-multiple-expressroute-circuits-in-the-same-metro-can-i-link-them-to-the-same-virtual-network"></a>我可以在相同的城市擁有多個 ExpressRoute 線路嗎？ 是否可以將這些線路連結至相同的虛擬網路？

是。 您可以有多個 ExpressRoute 線路，且服務提供者不一定要相同。 如果城市中有多個 ExpressRoute 對等互連位置，且線路是建立在不同的對等互連位置，您就可以將這些線路連結至相同的虛擬網路。 如果線路是在相同的對等互連位置建立，您最多可以將四個線路連結到相同的虛擬網路。

### <a name="how-do-i-connect-my-virtual-networks-to-an-expressroute-circuit"></a>我要如何將虛擬網路連線至 ExpressRoute 電路

基本步驟為：

* 建立 ExpressRoute 線路，並要求服務提供者將它啟用。
* 您或提供者必須設定 BGP 對等互連。
* 將虛擬網路連結至 ExpressRoute 線路。

如需詳細資訊，請參閱 [ExpressRoute 工作流程線路佈建和線路狀態](expressroute-workflows.md)。

### <a name="are-there-connectivity-boundaries-for-my-expressroute-circuit"></a>我的 ExpressRoute 電路是否有連線範圍？

是。 [ExpressRoute 合作夥伴和位置](expressroute-locations.md)文章提供 ExpressRoute 線路的連線範圍概觀。 ExpressRoute 電路的連線能力僅限於單一地緣政治區域。 透過啟用 ExpressRoute 進階功能，可將連線能力擴充到不同地緣政治區域。

### <a name="can-i-link-to-more-than-one-virtual-network-to-an-expressroute-circuit"></a>我可以將多個虛擬網路連結至 ExpressRoute 電路嗎？

是。 您在標準 ExpressRoute 線路上最多可以有 10 個虛擬網路，而在[進階 ExpressRoute 電路](#expressroute-premium)上最多可以有 100 個虛擬網路。 

### <a name="i-have-multiple-azure-subscriptions-that-contain-virtual-networks-can-i-connect-virtual-networks-that-are-in-separate-subscriptions-to-a-single-expressroute-circuit"></a>我有多個含有虛擬網路的 Azure 訂用帳戶。 我可以將各個訂用帳戶的虛擬網路連線到單一 ExpressRoute 電路嗎？

是。 您可以使用單一 ExpressRoute 線路，在與線路相同的訂用帳戶或不同訂用帳戶中，最多連結 10 個虛擬網路。 您可透過啟用 ExpressRoute 進階功能來提高此限制。 請注意，專用電路的連線能力和頻寬費用將會套用至 ExpressRoute 線路擁有者;所有虛擬網路都會共用相同的頻寬。

如需詳細資訊，請參閱[在多個訂用帳戶中共用 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)。

### <a name="i-have-multiple-azure-subscriptions-associated-to-different-azure-active-directory-tenants-or-enterprise-agreement-enrollments-can-i-connect-virtual-networks-that-are-in-separate-tenants-and-enrollments-to-a-single-expressroute-circuit-not-in-the-same-tenant-or-enrollment"></a>我有多個與不同 Azure Active Directory 租用戶或 Enterprise 合約註冊相關聯的 Azure 訂用帳戶。 可以將個別租用戶和註冊中的虛擬網路連線到不在相同租用戶或註冊中的單一 ExpressRoute 線路嗎？

是。 ExpressRoute 授權可以跨訂用帳戶、租用戶和註冊的界限，無需任何額外的設定。 請注意，專用電路的連線能力和頻寬費用將會套用至 ExpressRoute 線路擁有者;所有虛擬網路都會共用相同的頻寬。

如需詳細資訊，請參閱[在多個訂用帳戶中共用 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)。

### <a name="are-virtual-networks-connected-to-the-same-circuit-isolated-from-each-other"></a>連線到相同電路的虛擬網路會相互隔離嗎？

否。 從路由的觀點來看，所有連結至相同 ExpressRoute 線路的虛擬網路都屬於相同的路由網域，因此不會相互隔離。 如果需要路由隔離，您必須建立個別的 ExpressRoute 線路。

### <a name="can-i-have-one-virtual-network-connected-to-more-than-one-expressroute-circuit"></a>我可以將一個虛擬網路連結至多個 ExpressRoute 電路嗎？

是。 在相同或不同的對等互連位置中，您最多可以將單一虛擬網路連結至 4 個 ExpressRoute 線路。 

### <a name="can-i-access-the-internet-from-my-virtual-networks-connected-to-expressroute-circuits"></a>我可以從連線至 ExpressRoute 線路的虛擬網路來存取網際網路嗎？

是。 如果您尚未透過 BGP 工作階段公告預設路由 (0.0.0.0/0) 或網際網路路由首碼，就可從連結至 ExpressRoute 線路的虛擬網路連線到網際網路。

### <a name="can-i-block-internet-connectivity-to-virtual-networks-connected-to-expressroute-circuits"></a>針對連線至 ExpressRoute 線路的虛擬網路，我可以封鎖其網際網路連線嗎？

是。 針對部署於虛擬網路內的虛擬機器，您可以公告預設路由 (0.0.0.0/0)，以封鎖所有的網際網路連線，並將所有流量透過 ExpressRoute 線路路由傳送出去。

> [!NOTE]
> 如果從通告的路由中提取 0.0.0.0/0 的公告路由 (例如，由於) 發生中斷或設定錯誤，Azure 將會提供 [系統路由](../virtual-network/virtual-networks-udr-overview.md#system-routes) 給已連線的虛擬網路上的資源，以提供網際網路的連線能力。  為了確保封鎖網際網路的輸出流量，建議您將網路安全性群組放在具有網際網路流量輸出拒絕規則的所有子網上。

如果您公告預設路由，針對透過 Microsoft 對等互連 (例如 Azure 儲存體和 SQL DB) 所提供的服務，我們會強制讓流量回到您的內部。 您將必須設定路由器，透過 Microsoft 對等互連路徑或透過網際網路以將流量傳回 Azure。 如果您已啟用服務的服務端點，就不會將服務的流量強制到您的內部部署。 流量會保持在 Azure 中樞網路內。 如需深入了解服務端點，請參閱[虛擬網路服務端點](../virtual-network/virtual-network-service-endpoints-overview.md?toc=%2fazure%2fexpressroute%2ftoc.json)

### <a name="can-virtual-networks-linked-to-the-same-expressroute-circuit-talk-to-each-other"></a>連結至相同 ExpressRoute 電路的虛擬網路是否可以互通訊息？

是。 在連線到相同 ExpressRoute 電路的虛擬網路中部署的虛擬機器可以互相通訊。 建議您設定 [虛擬網路對等互連](../virtual-network/virtual-network-peering-overview.md) ，以加速此通訊。

### <a name="can-i-use-site-to-site-connectivity-for-virtual-networks-in-conjunction-with-expressroute"></a>我是否可以將虛擬網路的站對站連線與 ExpressRoute 搭配使用?

是。 ExpressRoute 可與站對站 VPN 並存。 請參閱[設定 ExpressRoute 和站對站並存連線](expressroute-howto-coexist-resource-manager.md)。

### <a name="why-is-there-a-public-ip-address-associated-with-the-expressroute-gateway-on-a-virtual-network"></a>為什麼虛擬網路上的 ExpressRoute 閘道會有相關聯的公用 IP 位址？

公用 IP 位址僅供內部管理使用，並不會構成虛擬網路的安全性風險。

### <a name="are-there-limits-on-the-number-of-routes-i-can-advertise"></a>是否有可通告的路由數限制？

是。 我們接受私人對等互連最多使用 4000 個路由首碼；Microsoft 對等互連最多使用 200 個路由首碼。 如果您啟用 ExpressRoute 進階功能，您可以針對私人對等互連將此值提高至 10,000 個路由。

### <a name="are-there-restrictions-on-ip-ranges-i-can-advertise-over-the-bgp-session"></a>我可以透過 BGP 工作階段通告的 IP 範圍有無限制？

對於 Microsoft 對等互連 BGP 工作階段，我們不接受私人首碼 (RFC1918)。 我們接受 Microsoft 和私人對等互連上的任何首碼大小 (最高可達 /32)。

### <a name="what-happens-if-i-exceed-the-bgp-limits"></a>如果超過 BGP 限制，該怎麼辦？

BGP 工作階段將會被刪除。 當首碼計數降至限制以下時，系統便會重設 BGP 工作階段。

### <a name="what-is-the-expressroute-bgp-hold-time-can-it-be-adjusted"></a>什麼是 ExpressRoute BGP 保留時間？ 可調整此保留時間嗎？

保留時間為 180。 會每隔 60 秒傳送保持運作訊息。 這些是 Microsoft 端固定的設定，無法變更。 您可以設定不同的計時器，系統將會據此交涉 BGP 工作階段參數。

### <a name="can-i-change-the-bandwidth-of-an-expressroute-circuit"></a>我可以變更 ExpressRoute 電路的頻寬嗎？

是，您可以在 Azure 入口網站中，或使用 PowerShell，嘗試增加 ExpressRoute 線路的頻寬。 如果您建立線路的實體連接埠上有可用的容量，則變更會成功。 

如果變更失敗，就表示目前的連接埠上沒有足夠的容量，因此必須建立頻寬較高的新 ExpressRoute 線路。或者，該位置已無額外的容量，在此情況下就無法增加頻寬。 

您也必須追蹤連線提供者的後續情形，以確保他們更新網路內的流速以支援增加頻寬。 不過，您無法減少 ExpressRoute 線路的頻寬。 您必須建立頻寬較低的新 ExpressRoute 線路，並刪除舊的線路。

### <a name="how-do-i-change-the-bandwidth-of-an-expressroute-circuit"></a>我如何變更 ExpressRoute 電路的頻寬？

您可以使用 REST API 或 PowerShell Cmdlet 來更新 ExpressRoute 線路的頻寬。

## <a name="expressroute-premium"></a>ExpressRoute Premium

### <a name="what-is-expressroute-premium"></a>什麼是 ExpressRoute Premium？

ExpressRoute Premium 是下列功能的集合：

* 將私用對等互連的路由表限制，從 4000 個路由提高到 10,000 個路由。
* 可在 ExpressRoute 線路 (預設為 10) 上啟用的增加的 VNet 與 ExpressRoute Global Reach 連線數目。 如需詳細資訊，請參閱 [ExpressRoute 限制](#limits)表格。
* Microsoft 365 的連線能力
* 透過 Microsoft 核心網路的全球連線。 您現在可將某一個地緣政治區域中的 VNet 與另一個區域中的 ExpressRoute 線路連結。<br>
    **範例：**

    *  您可將在歐洲西部建立的 VNet 連結至在矽谷建立的 ExpressRoute 線路。 
    *  在 Microsoft 對等互連上，會公告其他地緣政治區域的首碼，舉例來說，讓您可以從矽谷的線路連線至歐洲西部的 SQL Azure。


### <a name="how-many-vnets-and-expressroute-global-reach-connections-can-i-enable-on-an-expressroute-circuit-if-i-enabled-expressroute-premium"></a><a name="limits"></a>若啟用 ExpressRoute 進階版，我可以在 ExpressRoute 線路上啟用多少 VNet 與 ExpressRoute Global Reach 連線？

下表顯示 ExpressRoute 限制和每個 ExpressRoute 線路的 VNet 與 ExpressRoute Global Reach 連線數目：

[!INCLUDE [ExpressRoute limits](../../includes/expressroute-limits.md)]

### <a name="how-do-i-enable-expressroute-premium"></a>我要如何啟用 ExpressRoute Premium？

當啟用功能時便可啟用 ExpressRoute Premium 功能，並可透過更新線路狀態將它關閉。 您可以在建立線路時啟用 ExpressRoute Premium，或呼叫 REST API / PowerShell Cmdlet 來啟用。

### <a name="how-do-i-disable-expressroute-premium"></a>我要如何停用 ExpressRoute Premium？

您可以透過呼叫 REST API 或 PowerShell Cmdlet 來停用 ExpressRoute Premium。 在停用 ExpressRoute Premium 之前，您必須確定已經調整連線需求以符合預設限制。 如果您的使用率級別超越預設限制，停用 ExpressRoute Premium 的要求將會失敗。

### <a name="can-i-pick-and-choose-the-features-i-want-from-the-premium-feature-set"></a>我可以從進階功能集中挑選和選擇所需功能嗎？

否。 您無法挑選功能。 當您開啟 ExpressRoute Premium 時，我們便會將所有功能啟用。

### <a name="how-much-does-expressroute-premium-cost"></a>ExpressRoute Premium 需要多少錢？

如需成本相關資訊，請參閱 [價格詳細資訊](https://azure.microsoft.com/pricing/details/expressroute/) 。

### <a name="do-i-pay-for-expressroute-premium-in-addition-to-standard-expressroute-charges"></a>除了標準的 ExpressRoute 費用以外，我是否仍需支付 ExpressRoute Premium？

是。 除了 ExpressRoute 電路費用和連線提供者所需費用以外，還需另行支付 ExpressRoute Premium 費用。

## <a name="expressroute-local"></a>ExpressRoute Local
### <a name="what-is-expressroute-local"></a>什麼是 ExpressRoute Local？
除了標準 SKU 和進階 SKU 以外，ExpressRoute Local 也是 ExpressRoute 線路的 SKU。 Local 的重要功能是，ExpressRoute 對等互連位置的本機線路可讓您在相同的 Metro 或其附近只存取一或兩個 Azure 區域。 相反地，標準線路可讓您存取地緣政治區域中的所有 Azure 區域，而進階線路則可讓您存取全球的所有 Azure 區域。 

### <a name="what-are-the-benefits-of-expressroute-local"></a>ExpressRoute Local 有哪些優點？
儘管您需要針對標準或進階 ExpressRoute 線路支付輸出資料傳輸的費用，但不會針對您的 ExpressRoute Local 線路個別支付輸出資料傳輸的費用。 換句話說，ExpressRoute Local 的價格包括資料傳輸費用。 如果您有大量資料要傳輸，ExpressRoute Local 是更經濟實惠的解決方案，進而您可以透過私人連線將資料帶到所需 Azure 區域附近的 ExpressRoute 對等互連位置。 

### <a name="what-features-are-available-and-what-are-not-on-expressroute-local"></a>哪些功能可供使用，以及哪些功能不在 ExpressRoute Local 上？
相較於標準 ExpressRoute 線路，Local 線路具有一組相同的功能，除了：
* Azure 區域的存取範圍，如上所述
* ExpressRoute Global Reach 無法在 Local 上使用

ExpressRoute Local 也具有與標準相同的資源限制 (例如每個線路的 Vnet 數目)。 

### <a name="where-is-expressroute-local-available-and-which-azure-regions-is-each-peering-location-mapped-to"></a>哪裡提供 ExpressRoute Local，以及每個對等互連位置對應至哪些 Azure 區域？
ExpressRoute Local 可在一或兩個 Azure 區域已關閉的對等互連位置上使用。 若該州或省或國家/地區沒有 Azure 區域，則無法在對等互連位置上使用 ExpressRoute Local。 請參閱[位置頁面](expressroute-locations-providers.md)上的確切對應。  

## <a name="expressroute-for-microsoft-365"></a>Microsoft 365 的 ExpressRoute

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

### <a name="how-do-i-create-an-expressroute-circuit-to-connect-to-microsoft-365-services"></a>如何? 建立 ExpressRoute 線路以連接到 Microsoft 365 服務？

1. 請檢閱 [ExpressRoute 必要條件頁面](expressroute-prerequisites.md)，以確定您符合需求。
2. 若要確保符合您的連線需求，請檢閱 [ExpressRoute 合作夥伴和位置](expressroute-locations.md)文章中的服務提供者和位置清單。
3. 藉由審視 [Microsoft 365 的網路規劃和效能微調](/microsoft-365/enterprise/network-planning-and-performance)來規劃您的容量需求。
4. 遵循工作流程中所列的步驟來設定連線：[ExpressRoute 工作流程線路佈建和線路狀態](expressroute-workflows.md)。

> [!IMPORTANT]
> 設定與 Microsoft 365 服務的連線時，請確定您已啟用 ExpressRoute premium 附加元件。
> 
> 

### <a name="can-my-existing-expressroute-circuits-support-connectivity-to-microsoft-365-services"></a>我現有的 ExpressRoute 線路是否可支援 Microsoft 365 服務的連線能力？

是。 您可以設定現有的 ExpressRoute 線路，以支援 Microsoft 365 服務的連線能力。 確定您有足夠的容量可連接到 Microsoft 365 服務，而且您已啟用 premium 附加元件。 [Microsoft 365 的網路規劃和效能調整](/microsoft-365/enterprise/network-planning-and-performance) 可協助您規劃連線需求。 另請參閱 [建立和修改 ExpressRoute 電路](expressroute-howto-circuit-classic.md)。

### <a name="what-microsoft-365-services-can-be-accessed-over-an-expressroute-connection"></a>您可以透過 ExpressRoute 連線存取哪些 Microsoft 365 服務？

請參閱 [Microsoft 365 url 和 IP 位址範圍](/microsoft-365/enterprise/urls-and-ip-address-ranges) ] 頁面，以取得透過 ExpressRoute 支援的最新服務清單。

### <a name="how-much-does-expressroute-for-microsoft-365-services-cost"></a>Microsoft 365 服務的 ExpressRoute 需要多少費用？

Microsoft 365 服務需要啟用 premium 附加元件。 請參閱[定價詳細資料頁面](https://azure.microsoft.com/pricing/details/expressroute/)以了解成本。

### <a name="what-regions-is-expressroute-for-microsoft-365-supported-in"></a>ExpressRoute for Microsoft 365 支援哪些區域？

如需相關資訊，請參閱 [ExpressRoute 合作夥伴和位置](expressroute-locations.md)。

### <a name="can-i-access-microsoft-365-over-the-internet-even-if-expressroute-was-configured-for-my-organization"></a>即使已針對我的組織設定 ExpressRoute，還是可以透過網際網路存取 Microsoft 365？

是。 即使已針對您的網路設定 ExpressRoute，仍可透過網際網路連線 Microsoft 365 服務端點。 如果您所在位置的網路設定為透過 ExpressRoute 連線到 Microsoft 365 服務，請洽詢貴組織的網路團隊。

### <a name="how-can-i-plan-for-high-availability-for-microsoft-365-network-traffic-on-azure-expressroute"></a>如何針對 Azure ExpressRoute 上的 Microsoft 365 網路流量規劃高可用性？
請參閱 [Azure ExpressRoute 的高可用性和容錯移轉](/microsoft-365/enterprise/network-planning-with-expressroute)中的建議事項

### <a name="can-i-access-office-365-us-government-community-gcc-services-over-an-azure-us-government-expressroute-circuit"></a>我是否可以透過 Azure 美國政府 ExpressRoute 電路存取 Office 365 US Government Community (GCC) 服務？

是。 Office 365 GCC 服務端點可以透過 Azure 美國政府 ExpressRoute 存取。 不過，您必須先在 Azure 入口網站上開立支援票證，以提供您想要公告給 Microsoft 的首碼。 在解決該支援票證之後，您便可以連線至 Office 365 GCC 服務。 

## <a name="route-filters-for-microsoft-peering"></a>Microsoft 對等互連的路由篩選

### <a name="i-am-turning-on-microsoft-peering-for-the-first-time-what-routes-will-i-see"></a>第一次開啟 Microsoft 對等互連時，會看到哪些路由？

您不會看到任何路由。 您必須將路由篩選連結至線路，才能開始公告首碼。 如需指示，請參閱[針對 Microsoft 對等互連設定路由篩選](how-to-routefilter-powershell.md)。

### <a name="i-turned-on-microsoft-peering-and-now-i-am-trying-to-select-exchange-online-but-it-is-giving-me-an-error-that-i-am-not-authorized-to-do-it"></a>我已開啟 Microsoft 對等互連，而現在正嘗試選取 Exchange Online，但出現我並未獲得授權這麼做的錯誤。

當使用路由篩選時，任何客戶都可開啟 Microsoft 對等互連。 不過，若要使用 Microsoft 365 服務，您仍然必須獲得 Microsoft 365 授權。

### <a name="i-enabled-microsoft-peering-prior-to-august-1-2017-how-can-i-take-advantage-of-route-filters"></a>我在 2017 年 8 月 1 日之前啟用了 Microsoft 對等互連，該如何利用路由篩選？

您現有的線路會繼續通告 Microsoft 365 的首碼。 如果您想要透過相同的 Microsoft 對等互連新增 Azure 公用首碼公告，您可以建立路由篩選，並選取您需要公告的服務 (包括 Microsoft 365 服務 () 您需要) ，然後將篩選附加至您的 Microsoft 對等互連。 如需指示，請參閱[針對 Microsoft 對等互連設定路由篩選](how-to-routefilter-powershell.md)。

### <a name="i-have-microsoft-peering-at-one-location-now-i-am-trying-to-enable-it-at-another-location-and-i-am-not-seeing-any-prefixes"></a>我在某個位置有 Microsoft 對等互連，現在正嘗試於另一個位置啟用它，但沒有看見任何首碼。

* 在 2017 年 8 月 1 日以前設定之 ExpressRoute 線路的 Microsoft 對等互連，會透過 Microsoft 對等互連公告所有服務首碼，即使未定義路由篩選也一樣。

* 在 2017 年 8 月 1 日當日或以後設定之 ExpressRoute 線路的 Microsoft 對等互連，不會公告任何首碼，直到路由篩選連結至線路為止。 根據預設，您不會看見任何首碼。

## <a name="expressroute-direct"></a><a name="expressRouteDirect">ExpressRoute Direct</a>

[!INCLUDE [ExpressRoute Direct](../../includes/expressroute-direct-faq-include.md)]

## <a name="global-reach"></a><a name="globalreach"></a>Global Reach

[!INCLUDE [Global Reach](../../includes/expressroute-global-reach-faq-include.md)]

## <a name="privacy"></a>隱私權

### <a name="does-the-expressroute-service-store-customer-data"></a>ExpressRoute 服務是否會儲存客戶資料？

否。