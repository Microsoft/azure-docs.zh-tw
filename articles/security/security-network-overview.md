---
title: Azure 中的網路安全性概念和需求 | Microsoft Docs
description: 本文提供核心網路安全性概念和需求的基本說明，以及 Azure 在所有這些領域中所提供功能的相關資訊。
services: security
documentationcenter: na
author: TomShinder
manager: MBaldwin
editor: TomSh
ms.assetid: bedf411a-0781-47b9-9742-d524cf3dbfc1
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/17/2018
ms.author: terrylan
ms.openlocfilehash: ec8e4b1edfe815670664de83edf98a9e478101d2
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141913"
---
# <a name="azure-network-security-overview"></a>Azure 網路安全性概觀

Azure 包括一個強固的網路基礎結構，以支援您的應用程式和服務連線需求。 網路連線可能會出現在位於 Azure 的資源之間、內部部署與 Azure 裝載的資源之間，以及往返網際網路和 Azure 之間。

本文說明 Azure 在網路安全性領域中提供的部分選項。 您可以了解：

* Azure 網路
* 網路存取控制
* 安全遠端存取和跨單位連線
* 可用性
* 名稱解析
* 周邊網路 (DMZ) 架構
* 監視與威脅偵測
* Azure DDoS 保護

## <a name="azure-networking"></a>Azure 網路

Azure 需要虛擬機器連線至 Azure 虛擬網路。 虛擬網路是建置於實體 Azure 網路網狀架構之上的邏輯建構。 每個虛擬網路都與其他所有的虛擬網路隔離。 這可協助確保其他 Azure 客戶無法存取您部署中的網路流量。

深入了解：

* [虛擬網路概觀](../virtual-network/virtual-networks-overview.md)

## <a name="network-access-control"></a>網路存取控制

網路存取控制是指限制虛擬網路內與特定裝置或子網路間連線的動作。 網路存取控制的目標是限制只有已核准的使用者和裝置，才能存取您的虛擬機器和服務。 存取控制會以要允許或拒絕往返您虛擬機器或服務間連線的決策為基礎。

Azure 支援數種類型的網路存取控制，例如：

* 網路層控制
* 路由控制和強制通道
* 虛擬網路安全性應用裝置

### <a name="network-layer-control"></a>網路層控制

任何安全部署都需要某種程度的網路存取控制。 網路存取控制的目標是限制虛擬機器只能與必要的系統通訊。 其他通訊嘗試都會被封鎖。

>[!NOTE]
[Azure 儲存體安全性概觀](security-storage-overview.md)一文說明儲存體防火牆

#### <a name="network-security-rules-nsgs"></a>網路安全性規則 (NSG)

如果您需要基本的網路層級存取控制 (以 IP 位址和 TCP 或 UDP 通訊協定為基礎)，則可以使用網路安全性群組 (NSG)。 NSG 是基本且具狀態的封包過濾防火牆，可讓您根據 [5-Tuple](https://www.techopedia.com/definition/28190/5-tuple) \(英文\) 來控制存取權。 NSG 所含的功能可簡化管理工作並降低設定錯誤的可能性：

* **增強型安全性規則**可簡化 NSG 規則定義，並讓您能夠建立複雜的規則，而無須建立多個簡單的規則以達到相同的結果。
* **服務標記**是 Microsoft 建立的標籤，用來代表一組 IP 位址。 這些標記可動態更新，以納入與定義標籤所含內容的條件相符的 IP 範圍。 例如，如果您想要建立要對東部區域中的所有 Azure 儲存體套用的規則，您可以使用 Storage.EastUS
* **應用程式安全性群組**可讓您將資源部署至應用程式群組，並藉由建立使用這些應用程式群組的規則來控制對這些資源的存取。 例如，如果您已將 Web 伺服器部署至 'Webservers' 應用程式群組，您可以建立會套用 NSG 的規則，而允許從網際網路到 'Webservers' 應用程式群組中所有系統的 443 流量。

NSG 未提供應用程式層級檢查或已驗證的存取控制。

深入了解：

* [網路安全性群組](../virtual-network/security-overview.md)

#### <a name="asc-just-in-time-vm-access"></a>ASC Just-In-Time VM 存取

[Azure 資訊安全中心](../security-center/security-center-just-in-time.md)可以管理 VM 上的 NSG 及鎖定對 VM 的存取，直到具有適當角色型存取控制 [RBAC](../role-based-access-control/overview.md) 權限的使用者要求存取為止。 當使用者成功獲得授權後，ASC 便會對 NSG 進行修改，以允許在指定的時間內對選取的連接埠進行存取。 在這段時間到期後，NSG 就會還原為其先前受保護的狀態。

#### <a name="service-endpoints"></a>服務端點

服務端點是對您的流量進行控制的另一種方式。 您可以將您與支援的服務之間的通訊限定為透過直接連線的 Vnet。 從您的 VNet 到指定 Azure 服務的流量會保留在 Microsoft Azure 骨幹網路上。  

深入了解：

* [服務端點](../virtual-network/virtual-network-service-endpoints-overview.md#securing-azure-services-to-virtual-networks)

### <a name="route-control-and-forced-tunneling"></a>路由控制和強制通道

在您的虛擬網路上控制路由行為的能力非常重要。 如果路由的設定不正確，虛擬機器上所裝載的應用程式和服務可能會連線到未經授權的裝置，包括潛在攻擊者所擁有和操作的系統。

Azure 網路支援在虛擬網路上自訂網路流量路由行為的能力。 這可讓您改變虛擬網路中預設的路由表項目。 路由行為的控制可協助您確保來自特定裝置或裝置群組的所有流量，都會透過特定位置進入或離開您的虛擬網路。

例如，您在虛擬網路上可能有一個虛擬網路安全性設備。 您想要確定往返虛擬網路之間的所有流量都會經過該虛擬安全性設備。 若要這樣做，您可以在 Azure 中設定[使用者定義的路由](../virtual-network/virtual-networks-udr-overview.md) (UDR)。

[強制通道](https://www.petri.com/azure-forced-tunneling) \(英文\) 機制可讓您用來確保不允許服務起始與網際網路上裝置的連線。 請注意，這與接受連入連線，然後回應它們不同。 前端 Web 伺服器需要回應來自網際網路主機的要求，因此允許來自網際網路的流量傳入到這些 Web 伺服器，並允許 Web 伺服器回應。

您不想要允許的是前端 Web 伺服器起始傳出要求。 這類要求可能代表安全性風險，因為這些連線可用來下載惡意程式碼。 即使您希望這些前端伺服器起始到網際網路的連出要求，還是可能想要強制它們經過您的內部部署 Web Proxy。 這讓您能夠利用 URL 篩選和記錄功能的優勢。

因此，您可以使用強制通道來避免這個問題。 啟用強制通道時，會強制所有網際網路連線都要通過您的內部部署閘道。 您可以利用 UDR 的優勢來設定強制通道。

深入了解：

* [什麼是使用者定義路由和 IP 轉送](../virtual-network/virtual-networks-udr-overview.md)

### <a name="virtual-network-security-appliances"></a>虛擬網路安全性應用裝置

雖然 NSG、UDR 和強制通道都會在 [OSI 模型](https://en.wikipedia.org/wiki/OSI_model) \(英文\) 的網路和傳輸層上提供安全性層級，但您有時可能想要在高於網路的層級上啟用安全性。

例如，您的安全性需求可能包括︰

* 允許存取應用程式之前的驗證和授權
* 入侵偵測和入侵回應
* 高層級通訊協定的應用程式層檢查
* URL 篩選
* 網路層級防毒和反惡意程式碼
* 反 Bot 保護
* 應用程式存取控制
* 額外的 DDoS 保護 (高於由 Azure 網狀架構本身所提供的 DDoS 保護)

您可以使用 Azure 合作夥伴方案，來存取這些增強的網路安全性功能。 您可以造訪 [Azure Marketplace](https://azure.microsoft.com/marketplace/) 並搜尋「安全性」和「網路安全性」，來尋找最新的 Azure 合作夥伴網路安全性解決方案。

## <a name="secure-remote-access-and-cross-premises-connectivity"></a>安全遠端存取和跨單位連線

需要從遠端完成 Azure 資源的安裝、設定和管理。 此外，您可能想要部署其元件位於內部部署和 Azure 公用雲端中的[混合式 IT](http://social.technet.microsoft.com/wiki/contents/articles/18120.hybrid-cloud-infrastructure-design-considerations.aspx) \(英文\) 解決方案。 這些案例都需要安全遠端存取。

Azure 網路支援下列安全遠端存取案例︰

* 將個別工作站連線到虛擬網路
* 透過 VPN 將內部部署網路連線到虛擬網路
* 使用專用的 WAN 連結將內部部署網路連線到虛擬網路
* 使虛擬網路彼此連線

### <a name="connect-individual-workstations-to-a-virtual-network"></a>將個別工作站連線到虛擬網路

您可能想要讓個別的開發人員或操作人員在 Azure 中管理虛擬機器和服務。 例如，假設您需要存取虛擬網路上的虛擬機器。 但您的安全性原則不允許以 RDP 或 SSH 方式遠端存取個別虛擬機器。 在此情況下，您可以使用點對站 VPN 連接。

點對站 VPN 連線會使用 [SSTP VPN](https://technet.microsoft.com/library/cc731352.aspx) \(英文\) 通訊協定，讓您可以在使用者與虛擬網路之間設定私人的安全連線。 建立 VPN 連線時，使用者可以透過 VPN 連結以 RDP 或 SSH 方式連線到虛擬網路上的任何虛擬機器 (這假設使用者可進行驗證且已獲授權)。

深入了解：

* [使用 PowerShell 設定虛擬網路的點對站連線](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

### <a name="connect-your-on-premises-network-to-a-virtual-network-with-a-vpn"></a>透過 VPN 將內部部署網路連線到虛擬網路

您可能想要將整個公司網路或其各部分連線到虛擬網路。 這常見於組織要[將其內部部署資料中心延伸到 Azure](https://gallery.technet.microsoft.com/Datacenter-extension-687b1d84) \(英文\) 的混合式 IT 案例。 在許多情況下，組織會在 Azure 中裝載服務的組件，並於內部部署裝載組件。 例如，當解決方案包含位於 Azure 的前端 Web 伺服器以及位於內部部署的後端資料庫時，他們可能就會這樣做。 這些類型的「跨單位」連線也可以讓您以更安全的方式管理 Azure 所在的資源，並啟用像是將 Active Directory 網域控制站延伸到 Azure 等案例。

若要這麼做，其中一種方式是使用 [站對站 VPN](https://www.techopedia.com/definition/30747/site-to-site-vpn)。 站對站 VPN 和點對站 VPN 之間的差異是，後者會將單一裝置連線到虛擬網路。 站對站 VPN 會將整個網路 (例如您的內部部署網路) 連線到虛擬網路。 虛擬網路的站對站 VPN 會使用高度安全 IPsec 通道模式的 VPN 通訊協定。

深入了解：

* [使用 Azure 入口網站建立具有站對站 VPN 連線的 Resource Manager VNet](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)
* [規劃與設計 VPN 閘道](../vpn-gateway/vpn-gateway-plan-design.md)

### <a name="connect-your-on-premises-network-to-a-virtual-network-with-a-dedicated-wan-link"></a>使用專用的 WAN 連結將內部部署網路連線到虛擬網路

點對站和站對站 VPN 連接適用於啟用跨單位連接。 不過，有些組織認為它們有下列缺點︰

* VPN 連線會透過網際網路移動資料。 這會讓這些連線暴露在與透過公用網路移動資料有關的潛在安全性問題之中。 此外，無法保證網際網路連線的可靠性和可用性。
* 連線到虛擬網路的 VPN 連線可能沒有頻寬可供某些應用程式和用途使用，因為最大頻寬約為 200 Mbps。

需要最高安全性和可用性層級進行其跨單位連接的組織，一般會使用專用 WAN 連結連接到遠端網站。 Azure 讓您能夠使用可用來將內部部署網路連線到虛擬網路的專用 WAN 連結。 Azure ExpressRoute 會啟用此功能。

深入了解：

* [ExpressRoute 技術概觀](../expressroute/expressroute-introduction.md)

### <a name="connect-virtual-networks-to-each-other"></a>使虛擬網路彼此連線

您可以為您的部署使用多個虛擬網路。 您可能會基於各種原因來執行此動作。 您可能想要簡化管理，或者可能想要提高安全性。 不論將資源放在不同虛擬網路的動機為何，您有時可能想要讓每個網路上的資源彼此連線。

有一個選項是透過網際網路「繞回」，以將某個虛擬網路上的服務連線到另一個虛擬網路上的服務。 連線會從某個虛擬網路開始、經過網際網路，然後回到目的地虛擬網路。 這個選項會讓連線暴露在任何網際網路型通訊的固有安全性問題之中。

建立在兩個虛擬網路之間連線的站對站 VPN 可能是更好的選項。 此方法會使用與前述跨單位站對站 VPN 連線相同的 [IPSec 通道模式](https://technet.microsoft.com/library/cc786385.aspx)通訊協定。

這個方法的優點在於 VPN 連線是透過 Azure 網路網狀架構所建立的，而不是透過網際網路連線。 相較於透過網際網路連線的站對站 VPN，這會提供多一層的安全性。

深入了解：

* [使用 Azure Resource Manager 和 PowerShell 來設定 VNet 對 VNet 連線](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

另一種連接虛擬網路的方式是 [VNET 對等互連](../virtual-network/virtual-network-peering-overview.md)。 此功能可讓您連接兩個 Azure 網路，使其間的通訊透過 Microsoft 骨幹基礎結構進行，而完全不需要經由網際網路。 VNET 對等互連可連接相同區域內的兩個 VNET，或是跨 Azure 區域的兩個 VNET。 NSG 可用來限制不同子網路或系統之間的連線。

## <a name="availability"></a>可用性

可用性是任何安全性程式的重要元件。 如果您的使用者和系統無法存取它們需要透過網路存取的項目，則可以將服務視為入侵。 Azure 的網路技術支援下列高可用性機制：

* 以 HTTP 為基礎的負載平衡
* 網路層級負載平衡
* 全域負載平衡

負載平衡是一種機制，設計目的是將連接平均分散到多個裝置。 負載平衡的目的如下︰

* 提高可用性。 當您跨多部裝置對連線進行負載平衡時，即使一或多部裝置變成無法使用，也不會危及服務。 在其餘連線裝置上執行的服務可以繼續從服務提供內容。
* 提高效能。 當您跨多部裝置對連線進行負載平衡時，單一裝置不需負責所有處理作業。 而是將提供內容的處理和記憶體需求分散到多個裝置。

### <a name="http-based-load-balancing"></a>以 HTTP 為基礎的負載平衡

執行 Web 架構服務的組織通常會想在那些 Web 服務前面使用以 HTTP 為基礎的負載平衡器。 這有助於確保適當的效能和高可用性層級。 以網路為基礎的傳統負載平衡器會依賴網路和傳輸層通訊協定。 相反地，以 HTTP 為基礎的負載平衡器可根據 HTTP 通訊協定的特性做出決策。

Azure 應用程式閘道會針對您的 Web 架構服務提供以 HTTP 為基礎的負載平衡。 應用程式閘道支援：

* 以 Cookie 為基礎的工作階段同質性。 這個功能可確定建立到負載平衡器後方其中一部伺服器的連線，在用戶端與伺服器之間會保持不變。 這樣可確保交易的穩定性。
* SSL 卸載。 當用戶端與負載平衡器連線時，會使用 HTTPS (SSL) 通訊協定來加密該工作階段。 不過，為了提高效能，您可以使用 HTTP (未加密) 通訊協定，在負載平衡器與負載平衡器後方的 Web 伺服器之間進行連線。 這稱為「SSL 卸載」，因為負載平衡器後方的 Web 伺服器不會經歷與加密相關的處理器負荷。 因此，Web 伺服器應該可以更快速地為要求提供服務。
* 以 URL 為基礎的內容路由。 這個功能讓負載平衡器可以根據目標 URL 來決定要在何處轉送連線。 這所提供的彈性大於根據 IP 位址進行負載平衡決策的方案。

深入了解：

* [應用程式閘道概觀](../application-gateway/application-gateway-introduction.md)

### <a name="network-level-load-balancing"></a>網路層級負載平衡

相較於以 HTTP 為基礎的負載平衡，網路層級的負載平衡可根據 IP 位址和連接埠 (TCP 或 UDP) 號碼進行決策。
您可以使用 Azure Load Balancer，來獲取 Azure 中網路層級負載平衡的優點。 Load Balancer 的一些重要特性包括：

* 以 IP 位址和連接埠號碼為基礎的網路層級負載平衡。
* 支援任何應用程式層通訊協定。
* 負載平衡到 Azure 虛擬機器和雲端服務角色執行個體。
* 可用於網際網路對應 (外部負載平衡) 與非網際網路對應 (內部負載平衡) 的應用程式和虛擬機器。
* 端點監視，可用來判斷負載平衡器後方是否有任何服務變成無法使用。

深入了解：

* [多個虛擬機器或服務之間網際網路面向的負載平衡器](../load-balancer/load-balancer-internet-overview.md)
* [內部負載平衡器概觀](../load-balancer/load-balancer-internal-overview.md)

### <a name="global-load-balancing"></a>全域負載平衡

有些組織想要盡可能地擁有最高層級的可用性。 達到這個目的的其中一種方法是將應用程式裝載到全球分散的資料中心。 當應用程式裝載於世界各地的資料中心時，整個地緣政治區域可能會變成無法使用，但仍會啟動並執行應用程式。

此負載平衡策略也可以產生效能優勢。 您可以將服務的要求導向資料中心，而且是最接近提出要求裝置的資料中心。

在 Azure 中，使用 Azure 流量管理員，可以獲取全域負載平衡的優點。

深入了解：

* [什麼是流量管理員？](../traffic-manager/traffic-manager-overview.md)

## <a name="name-resolution"></a>名稱解析

名稱解析是您在 Azure 中裝載之所有服務的重大功能。 從安全性角度來看，入侵名稱解析功能可能會導致攻擊者將您網站的要求重新導向到攻擊者的網站。 安全名稱解析是所有雲端裝載服務的需求。

您需要解決兩種類型的名稱解析︰

* 內部名稱解析。 這可供您的虛擬網路、內部部署網路或兩者上的服務使用。 用於內部名稱解析的名稱無法透過網際網路來存取。 為了取得最佳安全性，重要的是外部使用者無法存取您的內部名稱解析結構描述。
* 外部名稱解析。 這可供您內部部署網路和虛擬網路外部的人員和裝置使用。 這些名稱在網際網路上是可見的，且可用來將連線導向至您的雲端式服務。

對於內部名稱解析，您有兩個選項︰

* 虛擬網路 DNS 伺服器。 當您建立新的虛擬網路時，即會為您建立 DNS 伺服器。 這部 DNS 伺服器可以解析位於該虛擬網路中電腦的名稱。 這部 DNS 伺服器無法設定且會由 Azure 網狀架構管理員來管理，因而可協助您保護名稱解析解決方案的安全。
* 使用您自己的 DNS 伺服器。 您可以選擇將自己選擇的 DNS 伺服器放在虛擬網路上。 這部 DNS 伺服器可以是整合 Active Directory 的 DNS 伺服器，或由 Azure 合作夥伴所提供的專用 DNS 伺服器解決方案 (可從 Azure Marketplace 取得)。

深入了解：

* [虛擬網路概觀](../virtual-network/virtual-networks-overview.md)
* [管理虛擬網路所使用的 DNS 伺服器](../virtual-network/manage-virtual-network.md#change-dns-servers)

針對外部名稱解析，您有兩個選項：

* 在內部部署裝載您自己的外部 DNS 伺服器。
* 使用服務提供者裝載您自己的外部 DNS 伺服器。

許多大型組織都會在內部部署裝載自己的 DNS 伺服器。 它們可以這麼做的原因是它們具有這麼做的網路專業知識和全域空間。

在大部分情況下，最好使用服務提供者來裝載您的 DNS 名稱解析服務。 這些服務提供者具有網路專業知識和全域空間，確保名稱解析服務的極高可用性。 可用性是 DNS 服務的不可或缺項目，因為如果您的名稱解析服務失敗，就沒有人可以連線到您的網際網路面向服務。

Azure 會以 Azure DNS 的形式提供高可用性且高效能的外部 DNS 解決方案。 這個外部名稱解析方案利用全球 Azure DNS 基礎結構。 它可讓您使用與您其他 Azure 服務所使用的相同認證、API、工具和帳單，將網域裝載於 Azure。 在 Azure 時，它也會繼承平台內建的增強式安全性控制。

深入了解：

* [Azure DNS 概觀](../dns/dns-overview.md)
* [Azure DNS 私人區域](../dns/private-dns-overview.md)可讓您在自動指派的名稱以外設定 Azure 資源的私人 DNS 名稱，而不需要新增自訂 DNS 解決方案。

## <a name="perimeter-network-architecture"></a>周邊網路架構

許多大型組織都會使用周邊網路來區隔其網路，並在網際網路與其服務之間建立緩衝區。 網路的周邊部分會被視為低安全性區域，而且不會有高價值資產放在該網路區段中。 您通常將看到網路安全性裝置會在周邊網路區段上有一個網路介面。 另一個網路介面會連線到具有可接受來自網際網路連入連線的虛擬機器和服務的網路。

您可以使用一些不同的方法來設計周邊網路。 根據您的網路安全性需求來決定是否要部署周邊網路。若您想要部署，則接著需決定要使用哪種類型的周邊網路。

深入了解：

* [Microsoft 雲端服務和網路安全性](../best-practices-network-security.md)

## <a name="monitoring-and-threat-detection"></a>監視與威脅偵測

Azure 提供的功能可協助您在這個重要領域中及早偵測、監視，並收集和檢視網路流量。

### <a name="azure-network-watcher"></a>Azure 網路監看員

Azure 網路監看員可協助您進行疑難排解，並提供一組全新的工具來協助識別安全性問題。

[安全性群組檢視](../network-watcher/network-watcher-security-group-view-overview.md)有助於稽核虛擬機器並確保其安全性合規性。 使用此功能以程式設計方式執行稽核，將組織所定義的基準原則與適用於您每部 VM 的有效規則進行比較。 這可以協助您識別所有設定漂移。

[封包擷取](../network-watcher/network-watcher-packet-capture-overview.md)可讓您擷取傳送至虛擬機器，或是自虛擬機器傳送的網路流量。 您可以收集網路統計資料並進行應用程式問題的疑難排解，這在調查網路入侵時十分有用。 您也可以搭配 Azure Functions 使用此功能，來啟動網路擷取以回應特定的 Azure 警示。

如需網路監看員以及如何開始在實驗室中測試部分功能的詳細資訊，請參閱 [Azure 網路監看員監視概觀](../network-watcher/network-watcher-monitoring-overview.md)。

>[!NOTE]
如需此服務可用性和狀態的最新通知，請查看 [Azure 更新頁面](https://azure.microsoft.com/updates/?product=network-watcher)。

### <a name="azure-security-center"></a>Azure 資訊安全中心

Azure 資訊安全中心可協助您預防、偵測和回應威脅，並加強提供對 Azure 資源安全性的能見度及控制權。 它在您的 Azure 訂用帳戶之間提供整合式安全性監視和原則管理，協助您偵測可能會忽略的威脅，且適用於大量的安全性解決方案。

資訊安全中心會透過下列方式，協助您將網路安全性最佳化並進行監視：

* 提供網路安全性建議。
* 監視網路安全性設定的狀態。
* 對端點和網路層級的網路型威脅發出警示。

深入了解：

* [Azure 資訊安全中心簡介](../security-center/security-center-intro.md)

### <a name="logging"></a>記錄

網路層級的記錄是任何網路安全性案例的重要功能。 在 Azure 中，您可以記錄針對 NSG 所取得的資訊，以取得網路層級的記錄資訊。 使用 NSG 記錄，您可以從下列項目取得資訊︰

* [活動記錄](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)。 使用這些記錄來檢視已提交至您 Azure 訂用帳戶的所有作業。 預設會啟用這些記錄，並且可在 Azure 入口網站內使用。 它們以前稱為稽核或作業記錄。
* 事件記錄。 這些記錄會提供已套用哪些 NSG 規則的相關資訊。
* 計數器記錄。 這些記錄可讓您知道套用每個 NSG 規則以拒絕或允許流量的次數。

您也可以使用 [Microsoft Power BI](https://powerbi.microsoft.com/what-is-power-bi/)(功能強大的資料視覺化工具) 來檢視和分析這些記錄檔。
深入了解：

* [網路安全性群組 (NSG) 的 Log Analytics](../virtual-network/virtual-network-nsg-manage-log.md)

## <a name="azure-ddos-protection"></a>Azure DDoS 保護

分散式阻斷服務 (DDoS) 攻擊是將應用程式移至雲端的客戶所面臨的最大可用性和安全性顧慮之一。 DDoS 攻擊會嘗試耗盡應用程式的資源，讓合法使用者無法使用該應用程式。 DDoS 攻擊可以鎖定可透過網際網路公開觸達的任何端點。
Microsoft 提供在 Azure 平台中名為**基本**的 DDoS 保護。 此功能不收費，且會持續監視和即時緩解常見的網路層級攻擊。 除了隨附於 DDoS 保護**基本**的保護功能以外，您也可以啟用**標準**選項。 DDoS Protection Standard 功能包括：

* **原生平台整合**：原生整合到 Azure。 包括透過 Azure 入口網站進行設定。 DDoS Protection Standard 了解您的資源和資源組態。
* **現成的保護：** 經過簡化的組態會在啟用 DDoS Protection Standard 後，立即保護虛擬網路上的所有資源。 不需要任何介入或使用者定義。 一旦偵測到攻擊，DDoS Protection Standard 就會立即自動減輕攻擊。
* **永遠可用流量監視：** 您的應用程式流量模式受到全年無休的全天候監視，以尋找 DDoS 攻擊的指標。 超出保護原則時，就會執行安全防護功能。
* **自適性調整：** 智慧型流量分析功能可了解不同時間的應用程式流量，並選取及更新最適合您服務的設定檔。 設定檔會隨著時間調整流量變更。 第 3 層至第 7 層保護：搭配 Web 應用程式防火牆使用時，可提供完整的堆疊 DDoS 保護。
* **廣泛的安全防護範圍：** 可利用全域功能降低超過 60 種不同攻擊類型的風險，以抵禦最大的已知 DDoS 攻擊。
* **攻擊計量：** 透過 Azure 監視器可以存取每個攻擊的摘要計量。
* **攻擊警示：** 警示可設定為在開始和停止攻擊時，並且在攻擊的持續時間內使用內建攻擊計量。 警示會整合到您的作業軟體，例如 Microsoft Azure Log Analytics、Splunk、Azure 儲存體、電子郵件和 Azure 入口網站。
* **成本保證：** 資料傳輸和應用程式相應放大服務會針對記載的 DDoS 攻擊計算點數。

深入了解：

* [DDOS 保護概觀](../virtual-network/ddos-protection-overview.md)