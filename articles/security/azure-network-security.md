---
title: Azure 網路安全性 | Microsoft Docs
description: 了解雲端式計算服務，其中包含各式各樣的計算執行個體和服務，可自動相應增加或縮小以符合您應用程式或企業的需求。
services: security
documentationcenter: na
author: UnifyCloud
manager: mbaldwin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: TomSh
ms.openlocfilehash: b1485ff933df7991e1c112981ae10d93bf5613da
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37061536"
---
# <a name="azure-network-security"></a>Azure 網路安全性

我們知道安全性是雲端中的首要工作和其重要性，因為您可在其中找到精確且及時的 Azure 安全性資訊。 針對您的應用程式和服務使用 Azure 的最佳原因之一，就是可以利用 Azure 的各種安全性工具和功能。 這些工具和功能可協助您在 Azure 平台上建立安全的解決方案。

Microsoft Azure 提供客戶資料的機密性、完整性和可用性，同時也能釐清責任。 為了協助您從客戶的觀點深入了解如何在 Microsoft Azure 內實作一組網路安全性控制，因而編寫了本文＜Azure 網路安全性＞來提供可透過 Microsoft Azure 取得之網路安全性控制的完整介紹。

這份文件旨在說明您可以設定的各種網路控制，以增強您在 Azure 中部署之解決方案的安全性。 如果您對於 Microsoft 如何保護 Azure 平台本身的網路網狀架構有興趣，請參閱 [Microsoft 信任中心](https://microsoft.com/en-us/trustcenter/cloudservices/azure)的 Azure 安全性一節。

## <a name="azure-platform"></a>Azure 平台

Azure 是一個公用雲端服務平台，支援廣泛的作業系統、程式設計語言、架構、工具、資料庫及裝置等選擇。  它可以透過 Docker 整合執行 Linux 容器；使用 JavaScript、Python、.NET、PHP、Java 及 Node.js 建置應用程式；為 iOS、Android 及 Windows 裝置建置後端。 Azure 雲端服務支援數百萬名開發人員和 IT 專家早已仰賴和信任的相同技術。

當您建置 IT 資產或將其移轉至公用雲端服務提供者時，您正是依賴該組織的能力，利用他們提供來管理您雲端架構資產安全性的服務與控制，來保護您的應用程式和資料。

Azure 的基礎結構設計涵蓋設備與應用程式，可同時裝載數以百萬計的客戶，並提供可靠的基礎供企業符合其安全性需求。 此外，Azure 也為您提供各式各樣可設定的安全性選項及控制它們的功能，讓您能夠自訂安全性以符合組織部署的特殊需求。

## <a name="abstract"></a>摘要

Microsoft 公用雲端服務提供超大規模的服務和基礎結構、企業級的功能，以及許多混合式連線選項。 您可以選擇透過網際網路或透過 Azure ExpressRoute (提供私人網路連線能力) 存取這些服務。 Microsoft Azure 平台可讓您順暢地將基礎結構延伸至雲端並建置多層式架構。 另外，協力廠商可以提供安全性服務和虛擬設備，以啟用增強的功能。

Azure 的網路服務設計可將彈性、可用性、復原、安全性和完整性最大化。 這份白皮書提供 Azure 網路功能的詳細資料，以及客戶如何使用 Azure 原生安全性功能來協助保護其資訊資產的詳細資訊。

本白皮書的預定對象包括：

- 尋求 Azure 中可用和支援安全性解決方案的技術管理員、網路管理員和開發人員。

-   想要進一步了解 Azure 公用雲端網路安全性討論中相關 Azure 網路技術和服務的 SME 或商務程序主管。

## <a name="azure-networking-big-picture"></a>Azure 網路服務概觀
Microsoft Azure 包括強大網路基礎結構以支援您的應用程式和服務連線需求。 在 Azure 的資源之間、內部部署與 Azure 託管資源之間，以及到網際網路和 Azure 與來自網際網路和 Azure，可能會有網路連線。

![Azure 網路服務概觀](media/azure-network-security/azure-network-security-fig-1.png)

[Azure 網路基礎結構](https://docs.microsoft.com/azure/virtual-machines/windows/infrastructure-networking-guidelines)可讓 Azure 資源與虛擬網路 (VNet) 彼此安全地連線。 VNet 是您的網路在雲端中的身分。 VNet 是專屬於您訂用帳戶的 Azure 雲端網路邏輯隔離。 您可以將 VNet 連線到內部部署網路。

Azure 支援內部部署網路的專用 WAN 連結連線以及採用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)的 Azure 虛擬網路。 Azure 與網站之間的連結會使用不經由公用網際網路的專用連線。 如果 Azure 應用程式正在多個資料中心內執行，您可使用 [Azure 流量管理員](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-overview)，以智慧方式將使用者的要求路由傳送到應用程式的各執行個體。 您也可以將流量路由傳送到不在 Azure 中執行的服務，只要可以從網際網路存取這些服務即可。

## <a name="enterprise-view-of-azure-networking-components"></a>Azure 網路元件的企業觀點
Azure 有許多與網路安全性討論的網路元件。 我們會說明這些網路元件，並著重於與其相關的安全性問題。

> [!Note]
> 我們不會說明 Azure 網路的所有 – 我們只討論為您在 Azure 中部署的服務和應用程式規劃和設計安全網路基礎結構的關鍵元件。

本白皮書涵蓋下列 Azure 網路企業功能：

-   基本網路連線

-   混合式連線

-   安全性控制

-   網路驗證

### <a name="basic-network-connectivity"></a>基本網路連線

[Azure 虛擬網路](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)服務可讓 Azure 資源與虛擬網路 (VNet) 安全地彼此連線。 VNet 是您的網路在雲端中的身分。 VNet 是專屬於您訂用帳戶的 Azure 網路基礎結構邏輯隔離。 您也可以使用站對站 VPN 和專用[WAN 連結](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)，讓 VNet 彼此連線和連線到內部部署網路。

![基本網路連線](media/azure-network-security/azure-network-security-fig-2.png)

了解您使用 VM 在 Azure 中裝載伺服器，問題後，問題視這些 VM 如何連線到網路。 答案是 VM 會連線到 [Azure 虛擬網路](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)。

Azure 虛擬網路就像您在內部部署環境中搭配自己的虛擬化平台解決方案使用的虛擬網路，例如 Microsoft Hyper-V 或 VMware。

#### <a name="intra-vnet-connectivity"></a>VNet 內部連線

您可以將 VNet 互相連線，讓連線至任一 VNet 的資源能夠跨越 VNet 彼此通訊。 您可以使用下列一個或兩個選項將 VNet 彼此連線：

- **對等互連︰** 讓連線至相同 Azure 位置內不同 Azure VNet 的資源彼此通訊。 如果資源已連線到相同的 VNet，則跨越 VNet 的頻寬和延遲就一樣。 若要深入了解對等互連，請參閱[虛擬網路對等互連](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)。

 ![對等互連](media/azure-network-security/azure-network-security-fig-3.png)

- **VNet 對 VNet 連線︰** 讓連線至相同或不同 Azure 位置內不同 Azure VNet 的資源彼此通訊。 不同於對等互連，VNet 之間的頻寬有限，因為流量必須通過 Azure VPN 閘道。

![VNet 對 VNet 連線](media/azure-network-security/azure-network-security-fig-4.png)


若要深入了解如何透過 VNet 對 VNet 連線來連線 VNet，請閱讀[設定 VNet 對 VNet 連線](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal?toc=%2fazure%2fvirtual-network%2ftoc.json)一文。

#### <a name="azure-virtual-network-capabilities"></a>Azure 虛擬網路功能：

如您所見，Azure 虛擬網路會提供虛擬機器來連線到網路，使其能夠以安全的方式連線到其他網路資源。 不過，基本連線能力只是起頭。 Azure 虛擬網路服務的下列功能揭露 Azure 虛擬網路的安全性特性：

-   隔離

-   網際網路連線

-   Azure 資源連線

-   VNet 連線

-   內部部署連線能力

-   流量篩選

-   路由

**隔離**

VNet 會彼此[隔離](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)。 您可以為使用相同 [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 位址區塊的開發、測試和生產環境建立個別的 VNet。 相反地，您可以建立多個使用不同 CIDR 位址區塊的 VNet 並將這些網路連接在一起。 您可以將 VNet 分成多個子網路。

Azure 會針對連線至 VNet 的 VM 和[雲端服務](https://azure.microsoft.com/services/cloud-services/)角色執行個體提供內部名稱解析。 您可以選擇地將 VNet 設定成使用自己的 DNS 伺服器，而不是使用 Azure 內部名稱解析。

您可以在每個 Azure [訂用帳戶](https://docs.microsoft.com/azure/azure-glossary-cloud-terminology?toc=%2fazure%2fvirtual-network%2ftoc.json)和 Azure [區域](https://docs.microsoft.com/azure/azure-glossary-cloud-terminology?toc=%2fazure%2fvirtual-network%2ftoc.json)內實作多個 VNet。 每個 VNet 會與其他 VNet 隔離。 對於每個 VNet，您可以︰

-   使用公用和私人 (RFC 1918) 位址指定自訂私人 IP 位址空間。 Azure 會從您指派的位址空間，將私人 IP 位址指派給連線至 VNet 的資源。

-   將 VNet 分成一或多個子網路，並將 VNet 位址空間的一部分配置給每個子網路。

-   使用 Azure 提供的名稱解析，或指定自有的 DNS 伺服器以供連線至 VNet 的資源使用。 若要深入了解 VNet 中的名稱解析，請閱讀 [VM 和雲端服務的名稱解析](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances)。

**網際網路連線**

根據預設，連線至 VNet 的所有 [Azure 虛擬機器 (VM)](https://docs.microsoft.com/azure/virtual-machines/windows/) 和雲端服務角色執行個體都可以存取網際網路。 您也可以視需要啟用特定資源的輸入存取。根據預設，連線至 VNet 的所有 Azure 虛擬機器 (VM) 和雲端服務角色執行個體都可以存取網際網路。 您也可以視需要啟用特定資源的輸入存取。

根據預設，連線至 VNet 的所有資源都具有網際網路的輸出連線能力。 資源的私人 IP 位址會由 Azure 基礎結構進行來源網路位址轉譯 (SNAT) 成為公用 IP 位址。 您可以實作自訂路由和流量篩選，以變更預設連線能力。 若要深入了解輸出網際網路連線能力，請閱讀[了解 Azure 中的輸出連線](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections?toc=%2fazure%2fvirtual-network%2ftoc.json)。

若要進行從網際網路通對 Azure 資源的輸入通訊，或進行對網際網路的輸出通訊 (未經 SNAT)，則必須指派公用 IP 位址給資源。 若要深入了解公用 IP 位址，請閱讀[公用 IP 位址](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address)。

**Azure 資源連線能力**

[Azure 資源](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) (例如雲端服務和 VM) 可以連線至相同的 VNet。 如果資源位於不同的子網路，則可使用私人 IP 位址彼此連接。 Azure 會提供子網路、VNet 和內部部署網路之間的預設路由，因此您不必設定及管理路由。

您可以將數個 Azure 資源連線至 VNet，例如虛擬機器 (VM)、雲端服務、App Service 環境及虛擬機器擴展集。 VM 會透過網路介面 (NIC) 連線至 VNet 內的子網路。 若要深入了解 NIC，請閱讀[網路介面](https://docs.microsoft.com/azure/virtual-network/virtual-network-network-interface)。

**VNet 連線能力**

[VNet](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) 可以彼此連線，讓連線至任何 VNet 的資源能夠與任何其他 VNet 上的任何資源進行通訊。

您可以將 VNet 互相連線，讓連線至任一 VNet 的資源能夠跨越 VNet 彼此通訊。 您可以使用下列一個或兩個選項將 VNet 彼此連線：

- **對等互連︰** 讓連線至相同 Azure 位置內不同 Azure VNet 的資源彼此通訊。 如果資源已連線到相同的 VNet，則跨越 VNet 的頻寬和延遲就一樣。若要深入了解對等互連，請閱讀[虛擬網路對等互連](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)。

- **VNet 對 VNet 連線︰** 讓連線至相同或不同 Azure 位置內不同 Azure VNet 的資源彼此通訊。 不同於對等互連，VNet 之間的頻寬有限，因為流量必須通過 Azure VPN 閘道。 若要深入了解如何透過 VNet 對 VNet 連線來連線 VNet。 若要深入了解，請參閱[設定 VNet 對 VNet 連線](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal?toc=%2fazure%2fvirtual-network%2ftoc.json)。

**內部部署連線能力**

VNet 可以透過您的網路與 Azure 之間的私人網路連線，或透過經由網際網路的站對站 VPN 連線，連線到[內部部署](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)網路。

您可以使用下列選項的任意組合，將內部部署網路連線至 VNet︰

- **點對站虛擬私人網路 (VPN)：** 建立於一部連線到您網路的電腦與 VNet 之間。 如果您剛開始使用 Azure，此連線類型就很適合您，也適用於開發人員，因為它幾乎不需要變更您現有的網路。 連線會使用 SSTP 通訊協定，透過網際網路提供電腦與 VNet 之間的加密通訊。 點對站 VPN 的延遲無法預期，因為流量會周遊網際網路。

- **站對站 VPN：** 建立於 VPN 裝置與 Azure VPN 閘道之間。 此連線類型可讓您授權的任何內部部署資源存取 VNet。 此連線是 IPsec/IKE VPN，可透過網際網路提供內部部署裝置與 Azure VPN 閘道之間的加密通訊。 站對站連線的延遲無法預期，因為流量會周遊網際網路。

- **Azure ExpressRoute：** 透過 ExpressRoute 合作夥伴，建立於您的網路與 Azure 之間。 此連線是私人連線。 流量不會周遊網際網路。 ExpressRoute 連線的延遲無法預期，因為流量不會周遊網際網路。 若要深入了解所有先前的連線選項，請閱讀[連線拓撲圖](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways?toc=%2fazure%2fvirtual-network%2ftoc.json)。

**流量篩選**

可以依照來源 IP 位址和連接埠、目的地 IP 位址和連接埠以及通訊協定，對 VM 和雲端服務角色執行個體[網路流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)進行輸入及輸出的篩選。

您可以使用下列一個或兩個選項，篩選子網路之間的網路流量︰

- **網路安全性群組 (NSG)：** 每個 NSG 可以包含多個輸入和輸出安全性規則，讓您依照來源和目的地 IP 位址、連接埠和通訊協定篩選流量。 您可以將 NSG 套用至 VM 中的每個 NIC。 您也可以將 NSG 套用至 NIC 或其他 Azure 資源所連線的子網路。 若要深入了解 NSG，請閱讀[網路安全性群組](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)。

- **虛擬網路設備：** 虛擬網路設備是執行軟體的 VM，該軟體可執行網路功能 (例如防火牆)。 請檢視 Azure Marketplace 中可用的 NVA 清單。 此外，也可取得提供 WAN 最佳化和其他網路流量功能的 NVA。 NVA 通常使用於使用者定義或 BGP 路由。 您也可以使用 NVA 來篩選 VNet 之間的流量。

**路由**

您可以透過設定自己的路由，或使用透過網路閘道的 BGP 路由，選擇性地覆寫 Azure 的預設路由。

根據預設，Azure 會建立路由表，讓連線至任何 VNet 中任何子網路的資源彼此通訊。 您可以實作下列一個或兩個選項，覆寫 Azure 所建立的預設路由︰

- **使用者定義的路由︰** 您可以建立自訂路由表，其中的路由可控制每個子網路的流量會路由傳送至的位置。 若要深入了解使用者定義的路由，請閱讀[使用者定義的路由](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)。

- **BGP 路由︰** 如果您使用 Azure VPN 閘道或 ExpressRoute 連線將 VNet 連線至內部部署網路，則可將 BGP 路由傳播至 VNet。

### <a name="hybrid-internet-connectivity-connect-to-an-on-premises-network"></a>混合式網際網路連線：連線到內部部署網路
您可以使用下列選項的任意組合，將內部部署網路連線至 VNet︰

-   網際網路連線

-   點對站 VPN (P2S VPN)

-   站對站 VPN (S2S VPN)

-   ExpressRoute

#### <a name="internet-connectivity"></a>網際網路連線

如其名所示，網際網路連線可讓不同的公用端點接觸留存在虛擬網路中的工作負載，以便從網際網路存取您的工作負載。 使用[網際網路對向負載平衡器](https://docs.microsoft.com/azure/load-balancer/load-balancer-internet-overview)，或只要將公用 IP 位址指派給 VM，即可公開這些工作負載。 如此一來，網際網路上的任何項目就能夠觸達該虛擬機器，前提是主機防火牆、[網路安全性群組 (NSG)](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 和[使用者定義的路由](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)允許該情況發生。

在此案例中，您可以讓需要公開的應用程式接觸網際網路，而且能夠從任何位置或從特定位置連線至該應用程式 (視您的工作負載組態而定)。

#### <a name="point-to-site-vpn-or-site-to-site-vpn"></a>點對站 VPN 或站對站 VPN
這兩者屬於相同的類別。 兩者都要求您的 VNet 擁有 VPN 閘道，而且您可以使用 VPN 用戶端連線到它，讓您的工作站成為[點對站組態](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal)的一部分，也可以設定內部部署 [VPN 裝置](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-devices)使其能夠終止站對站 VPN。 如此一來，內部部署裝置可以連線到 VNet 中的資源。

點對站 (P2S) 設定可讓您建立從個別的用戶端電腦到虛擬網路的安全連線。 P2S 是透過 SSTP (安全通訊端通道通訊協定) 的 VPN 連線。

![點對站 VPN](media/azure-network-security/azure-network-security-fig-5.png)

當您想要從遠端位置 (例如從住家或會議中心) 連線至 VNet 時，或只有幾個需要連線至虛擬網路的用戶端時，點對站連線是很實用的解決方案。

P2S 連線不需要 VPN 裝置或公眾對應 IP 位址即可運作。 您可從用戶端電腦建立 VPN 連線。 因此，不建議透過 P2S 連線到 Azure，以免需要持續從許多內部部署裝置和電腦連線到您的 Azure 網路。

![站對站 VPN](media/azure-network-security/azure-network-security-fig-6.png)

> [!Note]
> 如需有關點對站連線的詳細資訊，請參閱[點對站常見問題集](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal)。

站對站 VPN 閘道連線可用來透過 IPsec/IKE (IKEv1 或 IKEv2) VPN 通道，將內部部署網路連線到 Azure 虛擬網路。

此類型的連線需要位於內部部署的 VPN 裝置，且您已對該裝置指派對外開放的公用 IP 位址。 此連線透過網際網路進行，可讓您在您的網路與 Azure 之間的加密連結內的「輸送」資訊。 站對站 VPN 是安全成熟的技術，各種規模的企業已部署數十年。 使用 [IPsec 通道模式](https://technet.microsoft.com/library/cc786385.aspx)可執行通道加密。

雖然站對站 VPN 是受信任、可靠且已確立的技術，通道中的流量會周遊網際網路。 此外，最大頻寬相對受限於大約 200 Mbps。

如果您需要跨單位連線的例外安全性或效能層級，建議您對跨單位連線使用 Azure ExpressRoute。 ExpressRoute 是內部部署位置或 Exchange 主機服務提供者之間專用的 WAN 連結。 因為這是電信公司連線，所以您的資料不會透過網際網路傳輸，因此不會暴露於網際網路通訊固有的潛在風險。

> [!Note]
> 如需 VPN 閘道的詳細資訊，請參閱[關於 VPN 閘道](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways)。

#### <a name="dedicated-wan-link"></a>專用的 WAN 連結
Microsoft Azure ExpressRoute 可讓您透過連線提供者所提供的專用私人連線，將內部部署網路擴充至 Azure。

ExpressRoute 連線不會經過公用網際網路。 相較於一般網際網路連線，這可讓 ExpressRoute 連線提供更可靠、更快速、延遲更短和更安全的連線。

![ 專用的 WAN 連結](media/azure-network-security/azure-network-security-fig-7.png)

> [!Note]
> 如需如何使用 ExpressRoute 將您的網路連線至 Microsoft 的詳細資訊，請參閱 [ExpressRoute 連線模型](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways)和 [ExpressRoute 技術概觀](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)。

如同站對站 VPN 選項，ExpressRoute 也可讓您連線到不一定只在一個 VNet 中的資源。 事實上，視 SKU 而定，您可以連線到 10 個 VNet。 如果您有[進階附加元件](https://docs.microsoft.com/azure/expressroute/expressroute-faqs)，則可能連線至最多 100 個 VNet (視頻寬而定)。 若要深入了解這些連線類型的外觀，請閱讀[連線拓撲圖](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="security-controls"></a>安全性控制
Azure 虛擬網路可提供與其他虛擬網路隔離的安全、邏輯網路，並支援您在內部部署網路上使用的許多安全性控制。 客戶可使用子網路建立自己的結構：他們使用自己的私人 IP 位址範圍、設定路由表、網路安全性群組、存取控制清單 (ACL)、閘道和虛擬設備，以在雲端執行其工作負載。

以下是您可以在 Azure 虛擬網路上使用的安全性控制：

-   網路存取控制

-   使用者定義的路由

-   網路安全性設備

-   應用程式閘道

-   Azure Web 應用程式防火牆

-   網路可用性控制

#### <a name="network-access-controls"></a>網路存取控制
雖然 Azure 虛擬網路 (VNet) 是 Azure 網路模型的基石並可提供隔離和保護，但[網路安全性群組群組 (NSG)](https://blogs.msdn.microsoft.com/igorpag/2016/05/14/azure-network-security-groups-nsg-best-practices-and-lessons-learned/)是您用來在網路層級強制執行和控制網路流量規則的主要工具。

![ 網路存取控制](media/azure-network-security/azure-network-security-fig-8.png)


您可以從客戶網路上的系統，透過跨單位連線或直接網際網路通訊，允許或拒絕虛擬網路內的工作負載，以控制存取權。

在圖表中，VNet 和 NSG 都位於 Azure 整體安全性堆疊的特定層中，其中的 NSG、UDR 和網路虛擬設備可用來建立安全性界限，以保護受保護網路中的應用程式部署。

NSG 使用 5 個 Tuple 來評估流量 (且使用於您為 NSG 設定的規則)：

-   [來源和目的地 IP 位址](https://support.microsoft.com/help/969029/the-functionality-for-source-ip-address-selection-in-windows-server-2008-and-in-windows-vista-differs-from-the-corresponding-functionality-in-earlier-versions-of-windows)

-   [來源和目的地連接埠](https://technet.microsoft.com/library/dd197515)

-   通訊協定：[傳輸控制通訊協定 (TCP)](https://technet.microsoft.com/library/cc940037.aspx) 或[使用者資料包通訊協定 (UDP)](https://technet.microsoft.com/library/cc940034.aspx)

這表示您可以控制單一 VM 與一組 VM、單一 VM 與另一個單一 VM，或整個子網路之間的存取。 同樣地，請記住，這是簡單的具狀態封包篩選，而不是完整的封包檢查。 網路安全性群組中沒有通訊協定驗證或網路層級 IDS 或 IPS 功能。

NSG 隨附一些您應該注意的內建規則。 它們是：

-   **允許特定虛擬網路內的所有流量：** 相同 Azure 虛擬網路上的所有 VM 可以彼此通訊。

-   **允許 Azure 負載平衡輸入：** 此規則允許從 Azure 負載平衡器的任何來源位址到任何目的地位址的流量。

-   **拒絕所有輸入：** 此規則會封鎖您已明確允許且源自網際網路的所有流量。

-   **允許輸出至網際網路的所有流量：** 此規則允許 VM 起始對網際網路的連線。 如果您不想起始這些連線，您需要建立規則來封鎖這些連線，或強制執行強制通道。

#### <a name="system-routes-and-user-defined-routes"></a>系統路由和使用者定義的路由

當您在 Azure 中將虛擬機器 (VM) 新增到虛擬網路 (VNet) 時，您會發現 VM 能自動透過網路彼此通訊。 您不需要指定閘道，即使 VM 位於不同子網路。

VM 到公開網際網路的通訊同樣適用，甚至當存在 Azure 到您的資料中心的混合連線時，也適用於您的內部網路。

![系統路由](media/azure-network-security/azure-network-security-fig-9.png)

這個通訊流程是可能的，因為 Azure 會使用一連串系統路由來定義 IP 流量流動的方式。 系統路由控制下列案例的通訊流程：

-   從同一子網路。

-   從 VNet 中的子網路到另一個子網路。

-   從 VM 到網際網路。

-   透過 VPN 閘道從 VNet 到另一個 VNet。

-   透過 VNet 對等互連 ([服務鏈結](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview))，從 VNet 到另一個 VNet。

-   透過 VPN 閘道從 VNet 到您的內部網路。

許多企業都有嚴格的安全性和合規性需求，而需要內部檢查所有的網路封包，以強制執行特定原則。 Azure 提供的機制稱為[強制通道](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-forced-tunneling)，其藉由建立自訂路由或透過 ExpressRoute 或 VPN 的[邊界閘道通訊協定 (BGP)](https://docs.microsoft.com/windows-server/remote/remote-access/bgp/border-gateway-protocol-bgp) 公告，將來自 VM 的流量路由傳送至內部部署環境。

Azure 中的強制通道會透過虛擬網路使用者定義路由 (UDR) 進行設定。 將流量重新導向至在內部部署網站時，會表示為至 Azure VPN 閘道的「預設路由」。

下節列出 Azure 虛擬網路路由表和路由目前的限制：

-   每個虛擬網路的子網路皆有內建的系統路由表。 系統路由表具有下列 3 個路由群組：

 -  **本機 VNet 路由：** 直接連接到相同虛擬網路中的目的地 VM

 - **內部部署路由：** 連接到 Azure VPN 閘道

 -  **預設路由：** 直接連接到網際網路。 系統將會卸除尚未由前兩個路由涵蓋之私人 IP 位址目的地的封包。

-   隨著使用者定義路由的發行，您可以建立路由表以新增預設路由，然後建立路由表與 VNet 子網路的關聯，以便啟用這些子網路上的強制通道。

-   您需要在連接到虛擬網路的內部部署本機網站間設定「預設網站」。

-   強制通道必須與具有動態路由 VPN 閘道 (非靜態閘道) 的 VNet 相關聯。

- ExpressRoute 強制通道不會透過這項機制進行設定，相反地，將由透過 ExpressRoute BGP 對等互連工作階段的廣告預設路由進行啟用。

> [!Note]
> 如需詳細資訊，請參閱 [ExpressRoute 文件](https://azure.microsoft.com/documentation/services/expressroute/)。

#### <a name="network-security-appliances"></a>網路安全性設備
雖然網路安全性群組和使用者定義路由可以在 [OSI 模型](https://en.wikipedia.org/wiki/OSI_model)的網路和傳輸層提供特定網路安全性測量，但在某些情況下，您會想要或需要在更高層級的網路堆疊啟用安全性。 在這類情況下，建議您部署 Azure 合作夥伴所提供的虛擬網路安全性應用裝置。

![網路安全性設備](./media/azure-network-security/azure-network-security-fig-10.png)

Azure 網路安全性設備可改善 VNet 安全性和網路功能，這些功能透過 [Azure Marketplace](https://azuremarketplace.microsoft.com) 由許多廠商提供。 可以部署這些虛擬安全性設備以提供：

-   高可用性的防火牆

-   入侵預防

-   入侵偵測

-   Web 應用程式防火牆 (WAF)

-   WAN 最佳化

-   路由

-   負載平衡

-   VPN

-   憑證管理

-   Active Directory

-   多重要素驗證

#### <a name="application-gateway"></a>應用程式閘道

[Microsoft Azure 應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)是專用的虛擬設備，它會以服務形式提供應用程式傳遞控制器 (ADC)。

 ![應用程式閘道](./media/azure-network-security/azure-network-security-fig-11.png)

應用程式閘道可將 CPU 密集 SSL 終止卸載至應用程式閘道 (SSL-offloading)，讓您最佳化 Web 伺服陣列的效能和可用性。 它也提供其他第 7 層路由功能，包括：

-   傳入流量的循環配置資源散發

-   Cookie 型工作階段同質

-   URL 路徑型路由

-   在單一應用程式閘道後面裝載多個網站的功能


[Web 應用程式防火牆 (WAF)](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) 也會提供為應用程式閘道的一部分。 這會保護 Web 應用程式免於遭遇常見的 Web 弱點和攻擊。 應用程式閘道可以設定為網際網路對向閘道、內部專用閘道或兩者混合。

可以在偵測或預防模式中執行應用程式閘道 WAF。 常見的使用案例是讓系統管理員在偵測模式中執行，以觀察惡意模式的流量。 一旦偵測到潛在的漏洞，尋求預防模式來封鎖可疑的傳入流量。

 ![應用程式閘道](./media/azure-network-security/azure-network-security-fig-12.png)

此外，應用程式閘道 WAF 可協助您監視 Web 應用程式來抵禦攻擊，方法是使用與 [Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview)和 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)整合的即時 WAF 記錄，追蹤 WAF 警示並輕鬆地監視趨勢。

JSON 格式化記錄會直接移至客戶的儲存體帳戶。 您可以完全掌控這些記錄，而且可以套用自己的保留原則。

您也可以使用 [Azure 記錄整合](https://aka.ms/AzLog)，將這些記錄內嵌至自己的分析系統中。 WAF 記錄也會與 [Log Analytics](../log-analytics/log-analytics-overview.md) 整合，以便您使用 Log Analytics 來執行複雜精細的查詢。

#### <a name="azure-web-application-firewall-waf"></a>Azure Web 應用程式防火牆 (WAF)

Web 應用程式已逐漸成為利用常見已知弱點的惡意攻擊目標，例如 SQL 插入式、跨網站指令碼攻擊，以及出現在 [OWASP 前 10 大](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)中的其他攻擊。 若要防止應用程式發生這類攻擊，需要對多層次的應用程式拓撲執行嚴格的維護、修補和監視。

 ![Azure Web 應用程式防火牆 (WAF)](./media/azure-network-security/azure-network-security-fig-13.png)

集中式 [Web 應用程式防火牆 (WAF)](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) 可以防禦 Web 攻擊並簡化安全性管理，而不需要變更任何應用程式。

相較於保護每個個別的 Web 應用程式，WAF 方案還可透過在中央位置修補已知弱點，更快地因應安全性威脅。 現有的應用程式閘道可以輕易地轉換成已啟用 Web 應用程式防火牆的應用程式閘道。

#### <a name="network-availability-controls"></a>網路可用性控制

有不同的選項可使用 Microsoft Azure 來分散網路流量。 這些選項的運作方式彼此間會有差異，其具備不同的功能組合並支援不同的案例。 它們每個都可單獨使用，或組合在一起。

以下是網路可用性控制：

-   Azure Load Balancer

-   應用程式閘道

-   流量管理員

**Azure Load Balancer**

為您的應用程式提供高可用性和網路效能。 這是 Layer 4 (TCP、UDP) 負載平衡器，可將連入流量分配到負載平衡集中所定義服務的狀況良好執行個體。

 ![Azure Load Balancer](media/azure-network-security/azure-network-security-fig-14.png)


Azure Load Balancer 可以設定為：

-   對虛擬機器的連入網際網路流量進行負載平衡。 這個組態稱為 [網際網路面向的負載平衡](https://docs.microsoft.com/azure/load-balancer/load-balancer-internet-overview)。

-   平衡虛擬網路中的虛擬機器之間、雲端服務中的虛擬機器之間，或內部部署電腦與跨單位部署虛擬網路中的虛擬機器之間的流量負載。 這個組態稱為 [內部負載平衡](https://docs.microsoft.com/azure/load-balancer/load-balancer-internal-overview)。

-   將外部流量轉送到特定的虛擬機器。

雲端中的所有資源都需要可從網際網路觸及的公用 IP 位址。 Azure 中的雲端基礎結構會針對其資源使用無法路由傳送的 IP 位址。 Azure 會使用具有公用 IP 位址的網路位址轉譯 (NAT) 來與網際網路通訊。

 **應用程式閘道**

 應用程式閘道是在應用程式層級 (OSI 網路參考堆疊中的第 7 層) 上運作。 它會用來當做反向 Proxy 服務，終止用戶端連線，並將要求轉寄到後端端點。

 **流量管理員**

Microsoft Azure 流量管理員可讓您控制使用者流量，將流量分散到不同資料中心的服務端點。 流量管理員支援的服務端點包括 Azure VM、Web Apps 和雲端服務。 您也可以將流量管理員使用於外部、非 Azure 端點。

流量管理員會使用網域名稱系統 (DNS)，根據[流量路由方法](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods)和端點的健全狀況，將用戶端要求導向到最適當的端點。 流量管理員提供各種流量路由方法，以符合不同的應用程式需求、端點健全狀況[監視](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-monitoring)、及自動容錯移轉。 流量管理員可針對失敗彈性應變，包括整個 Azure 區域失敗。

Azure 流量管理員可讓您控制流量分散到應用程式端點的方式。 端點是裝載於 Azure 內部或外部的任何網際網路對向服務。

流量管理員提供兩大優點︰

-   根據其中一種[流量路由方法](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods)分散流量。

-   [連續監視端點健康狀態](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-monitoring)以及在端點失敗時自動容錯移轉。

當用戶端嘗試連接至服務時，它必須先將服務的 DNS 名稱解析為 IP 位址。 然後用戶端才能連接到該 IP 位址以存取服務。 流量管理員會使用 DNS，根據流量路由方法的規則，將用戶端導向特定的服務端點。 用戶端會直接連線至選取的端點。 流量管理員不是 Proxy 或閘道。 流量管理員看不到在用戶端與服務之間傳遞的流量。

### <a name="azure-network-validation"></a>Azure 網路驗證

Azure 網路驗證以確定正在運作的 Azure 網路，因為它已設定且可在完成驗證是使用服務和功能可用來監視網路。 利用 Azure 網路監看員，您可以存取多項記錄和診斷功能，進而深入了解您的網路效能與健康情況。 這些功能可透過入口網站、Power Shell、CLI、Rest API 和 SDK 存取。

Azure 作業安全性是指使用者可在 Microsoft Azure 中用來保護其資料、應用程式和其他資產的服務、控制及功能。 Azure 作業安全性的基礎架構涵蓋透過 Microsoft 特有之各種功能獲得的知識，包括 Microsoft 安全性開發週期 (SDL)、Microsoft Security Response Center 方案，以及對網路安全性威脅型態的深層認知。

-   [Azure Operations Management Suite](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-overview)

-   [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)

-   [Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview)

-   [Azure 網路監看員](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)

-   [Azure 儲存體分析](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics)

-   Azure Resource Manager

#### <a name="azure-resource-manager"></a>Azure Resource Manager

操作 Microsoft Azure 的人員和程序或許是平台最重要的安全性特性。 這一節說明 Microsoft 的全域資料中心基礎結構中，可協助您增強及維護安全性、持續性和隱私權的功能。

應用程式的基礎結構通常由許多元件所組成 – 或許是虛擬機器、儲存體帳戶和虛擬網路，或者 web 應用程式、資料庫、資料庫伺服器和第三方服務。 您看不到這些元件作為個別的實體，而是看到它們作為單一實體相關且彼此相依的組件。 您會想要將其當成群組來部署、管理和監視。 Azure Resource Manager 可讓您將方案中的資源作為群組使用。

您可以透過單一、協調的作業來部署、更新或刪除方案的所有資源。 您會使用部署的範本，且該範本可以用於不同的環境，例如測試、預備和生產環境。 Resource Manager 會提供安全性、稽核和標記功能，以協助您在部署後管理您的資源。

**使用 Resource Manager 的優點**

Resource Manager 會提供數個優點：

-   您可以以群組形式部署、管理及監視方案的所有資源，而不是個別處理這些資源。

-   您可以在整個方案週期重複部署方案，並確信您的資源會部署在一致的狀態中。

-   您可以透過宣告式範本而非指令碼來管理基礎結構。

-   您可以定義之間的相依性，使得以正確的順序部署資源。

-   因為角色型存取控制 (RBAC) 會原生整合至管理平台，您可以將存取控制套用至資源群組中的所有服務。

-   您可以將標籤套用至資源，以便以邏輯方式組織訂用帳戶中的所有資源。

-   您可以檢視共用標籤之資源群組的成本，以釐清您的組織的計費方式。

> [!Note]
> Resource Manager 提供一個部署和管理方案的新方式。 如果您使用較舊的部署模型並想要了解這些變更，請參閱[瞭解Resource Manager 部署和傳統部署](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-model)。

## <a name="azure-network-logging-and-monitoring"></a>Azure 網路記錄和監視

Azure 提供許多工具來監視、預防、偵測及回應網路安全性事件。 可供您在此領域中使用的最強大工具包括：

-   網路監看員

-   網路資源層級監視

-   Log Analytics

### <a name="network-watcher"></a>網路監看員

[網路監看員](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview) - 網路監看員的功能隨附了案例式監視。 這項服務包括封包擷取、下一個躍點、IP 流量驗證、安全性群組檢視、NSG 流量記錄。 案例層級監視可提供端對端的網路資源檢視，而非個別的網路資源監視。

 ![網路監看員](./media/azure-network-security/azure-network-security-fig-15.png)

網路監看員是一項區域性服務，可讓您監視與診斷位於和進出 Azure 的網路案例層級條件。 網路監看員提供的網路診斷和視覺效果工具，可幫助您了解、診斷及洞悉您在 Azure 中的網路。

網路監看員目前具有下列功能︰

#### <a name="topology"></a>拓撲

[拓撲](https://docs.microsoft.com/azure/network-watcher/network-watcher-topology-overview)會傳回虛擬網路中的網路資源之圖形。 此圖描述了資源之間的互相連線來代表端對端網路連線。 在入口網站中，拓撲會依每個虛擬網路為基礎傳回資源物件。 即使在不會顯示的資源群組中，關聯性會依網路監看員區域外部的資源之間的線條描述。 在入口網站檢視傳回的資源會以圖表方式顯示的網路元件子集。 若要查看網路資源完整清單，您可以使用 [PowerShell](https://docs.microsoft.com/azure/network-watcher/network-watcher-topology-powershell) 或 [REST](https://docs.microsoft.com/azure/network-watcher/network-watcher-topology-rest)。

隨著資源傳回，它們之間的連線會在兩個關聯性之間模型化。

- **內含項目** - 虛擬網路包含其中含有 NIC 的子網路。

- **相關聯** - NIC 與 VM 相關聯。

#### <a name="variable-packet-capture"></a>變數封包擷取

網路監看員[變數封包擷取](https://docs.microsoft.com/azure/network-watcher/network-watcher-packet-capture-overview)可讓您建立封包擷取工作階段來追蹤虛擬機器的流入和流出流量。 封包擷取有助於被動和主動地診斷網路異常。 其他用途包括收集網路統計資料、取得有關網路入侵的資訊，以及偵錯用戶端與伺服器間的通訊等等。

封包擷取是透過網路監看員從遠端啟動的虛擬機器擴充功能。 這項功能可以減輕在所需虛擬機器上手動執行封包擷取的工作負擔，進而省下寶貴的時間。 可以透過入口網站、PowerShell、CLI 或 REST API 觸發封包擷取。 觸發封包擷取方式的其中一個範例是使用虛擬機器警示。

#### <a name="ip-flow-verify"></a>IP 流量驗證

[IP 流量驗證](https://docs.microsoft.com/azure/network-watcher/network-watcher-ip-flow-verify-overview)會根據 5 個 tuple 資訊檢查進出虛擬機器的封包是受到允許或拒絕。 這些資訊包括方向、通訊協定、本機 IP、遠端 IP、本機連接埠和遠端連接埠。 如果封包遭到安全性群組拒絕，則會傳回拒絕封包之規則的名稱。 雖然任何來源或目的地 IP 均可供選擇，但這項功能可協助系統管理員快速診斷網際網路和內部部署環境的往來連線問題。

IP 流量驗證是以虛擬機器的網路介面作為目標。 接著會根據所設的設定，驗證該網路介面往來的流量。 這項功能可用於確認網路安全性群組中的規則是否會封鎖虛擬機器的輸入或輸出流量。

#### <a name="next-hop"></a>下一個躍點

決定在 Azure 網路網狀架構中路由傳送之封包的[下一個躍點](https://docs.microsoft.com/azure/network-watcher/network-watcher-next-hop-overview)，讓您得以診斷任何設定錯誤的使用者定義路由。 來自 VM 的流量會根據與 NIC 相關聯的有效路由來傳送到目的地。 下一個躍點會從特定虛擬機器和 NIC 取得封包的下一個躍點類型和 IP 位址。 這有助於判斷封包會被導向到目的地，還是流量會被吸收掉。

下一個躍點也會傳回與下一個躍點相關聯的路由表。 向下一個躍點查詢路由是否定義為使用者定義的路由時，便會傳回該路由。 否則，下一個躍點會傳回「系統路由」。

#### <a name="security-group-view"></a>安全性群組檢視

取得套用至 VM 的有效和已套用安全性規則。 網路安全性群組會在子網路層級或 NIC 層級產生關聯。 若在子網路層級產生關聯，它會套用至子網路中的所有 VM 執行個體。 網路[安全性群組檢視](https://docs.microsoft.com/azure/network-watcher/network-watcher-security-group-view-overview)會針對虛擬機器傳回所有於 NIC 和子網路層級產生關聯的已設定 NSG 和規則，提供設定的深入解析。 此外，VM 中的每個 NIC 也會傳回有效的安全性規則。 使用 [網路安全性群組] 檢視，您就可以評估 VM 的網路弱點，例如開啟的連接埠。 您也可以根據[所設定的安全性規則和有效安全性規則之間的比較](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-auditing-powershell)，驗證網路安全性群組是否如預期般運作。

#### <a name="nsg-flow-logging"></a>NSG 流量記錄

 網路安全性群組的流量記錄可讓您擷取群組中的安全性規則所允許或拒絕之流量的相關記錄。 流程是由 5個 Tuple 資訊定義的，這些資訊分別是來源 IP、目的地 IP、來源連接埠、目的地連接埠和通訊協定。

[網路安全性群組流程記錄](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview)是網路監看員的一項功能，可讓您檢視透過網路安全性群組輸入和輸出 IP 流量的相關資訊。

#### <a name="virtual-network-gateway-and-connection-troubleshooting"></a>虛擬網路閘道和連線疑難排解

網路監看員提供了許多功能，因為它的作用就是為了讓您了解您在 Azure 中的網路資源。 這些功能的其中之一便是資源疑難排解。 [資源疑難排解](https://docs.microsoft.com/azure/network-watcher/network-watcher-troubleshoot-manage-rest)可透過 PowerShell、CLI 或 REST API 來呼叫。 一經呼叫，網路監看員就會檢查虛擬網路閘道或連線的健全狀況，並傳回其調查結果。

這一節會帶領您逐步完成資源疑難排解目前可用的不同管理工作。

-   [針對虛擬網路閘道進行疑難排解](https://docs.microsoft.com/azure/network-watcher/network-watcher-troubleshoot-manage-rest)

-   [針對連線進行疑難排解](https://docs.microsoft.com/azure/network-watcher/network-watcher-troubleshoot-manage-rest)

#### <a name="network-subscription-limits"></a>網路訂用帳戶限制

[網路訂用帳戶限制](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)可為您提供區域中，訂用帳戶的每個網路資源使用量詳細資料與可用資源數上限的對照情況。

#### <a name="configuring-diagnostics-log"></a>設定診斷記錄

網路監看員提供[診斷記錄](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)檢視。 此檢視包含所有支援診斷記錄的網路資源。 從這個檢視中，您可以方便且快速地啟用和停用網路資源。

### <a name="network-resource-level-monitoring"></a>網路資源層級監視

資源層級監視可使用下列功能︰

#### <a name="audit-log"></a>稽核記錄檔

做為網路組態一部分所執行的作業會記錄下來。 這些稽核記錄是建立各種合規性的必備項目。 這些記錄可在 Azure 入口網站中檢視，或使用 Power BI 之類的 Microsoft 工具或協力廠商工具來擷取。 稽核記錄可透過入口網站、PowerShell、CLI 和 Rest API 來取得。

> [!Note]
> 如需稽核記錄的詳細資訊，請參閱[使用 Resource Manager 來稽核作業](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit)。
針對所有網路資源所進行的作業都會有稽核記錄。


#### <a name="metrics"></a>度量

計量是在一段時間內所收集到的效能測量數據和計數器。 計量目前適用於應用程式閘道。 計量可用來根據臨界值觸發警示。 Azure 應用程式閘道預設會監視其後端集區中所有資源的健康狀況，並自動從集區中移除任何被視為狀況不良的資源。 應用程式閘道會繼續監視狀況不良的執行個體，一旦其恢復可用狀態並回應健康狀況探查，就會將其新增回狀況良好後端集區中。 應用程式閘道會以後端 HTTP 設定中定義的相同連接埠傳送健康狀態探查。 此組態可確保探查所測試的連接埠會和客戶用來連接到後端的連接埠相同。

> [!Note]
> 請參閱[應用程式閘道診斷](https://docs.microsoft.com/azure/application-gateway/application-gateway-probe-overview)，以檢視如何使用計量來建立警示。

#### <a name="diagnostic-logs"></a>診斷記錄檔

網路資源會定期和自發地建立事件，並記錄到儲存體帳戶、傳送到事件中樞或 Log Analytics。 這些記錄可讓您深入了解資源的健全狀況。 您可以在 Power BI 和 Log Analytics 等工具中檢視這些記錄。 若要了解如何檢視診斷記錄，請瀏覽 [Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-networking-analytics)。

診斷記錄適用於[負載平衡器](https://docs.microsoft.com/azure/load-balancer/load-balancer-monitor-log)、[網路安全性群組](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)、路由和[應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)。

網路監看員提供診斷記錄檢視。 此檢視包含所有支援診斷記錄的網路資源。 從這個檢視中，您可以方便且快速地啟用和停用網路資源。

### <a name="log-analytics"></a>Log Analytics

[Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-overview) 是 Azure 中的一項服務，可監視您的雲端和內部部署環境，以維護其可用性和效能。 它會收集您的雲端和內部部署環境中的資源所產生的資料，以及從其他監視工具提供橫跨多個來源的分析。

Log Analytics 提供下列解決方案來監視您的網路︰

-   網路效能監視器 (NPM)

-   Azure 應用程式閘道分析

-   Azure 網路安全性群組分析

#### <a name="network-performance-monitor-npm"></a>網路效能監視器 (NPM)
[網路效能監視器](https://docs.microsoft.com/azure/log-analytics/log-analytics-network-performance-monitor)管理解決方案是網路監視解決方案，可監視網路的健康狀態、可用性和連線能力。

可用來監視下列項目之間的連線能力︰

-   公用雲端與內部部署環境

-   資料中心與使用者地點 (分公司)

-   裝載各種多層式應用程式的子網路。


#### <a name="azure-application-gateway-analytics-in-log-analytics"></a>Log Analytics 中的 Azure 應用程式閘道分析

應用程式閘道支援下列記錄︰

-   ApplicationGatewayAccessLog

-   ApplicationGatewayPerformanceLog

-   ApplicationGatewayFirewallLog

應用程式閘道支援下列度量︰

-   5 分鐘輸送量

#### <a name="azure-network-security-group-analytics-in-log-analytics"></a>Log Analytics 中的 Azure 網路安全性群組分析

[網路安全性群組](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)支援下列記錄︰

- **NetworkSecurityGroupEvent：** 包含要將 NSG 規則套用到以 MAC 位址為基礎的 VM 和執行個體角色的項目。 每隔 60 秒會收集一次這些規則的狀態。

- **NetworkSecurityGroupRuleCounter：** 包含套用每個 NSG 規則以拒絕或允許流量之次數的項目。

## <a name="next-steps"></a>後續步驟
閱讀一些有深度的安全性主題，以深入了解安全性：

-   [網路安全性群組 (NSG) 的 Log Analytics](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log)

-   [驅動雲端中斷的網路創新功能](https://azure.microsoft.com/blog/networking-innovations-that-drive-the-cloud-disruption/)

-   [SONiC：Microsoft Global Cloud 支援的網路切換軟體](https://azure.microsoft.com/blog/sonic-the-networking-switch-software-that-powers-the-microsoft-global-cloud/)

-   [Microsoft 建置其快速且可靠全域網路的方式](https://azure.microsoft.com/blog/how-microsoft-builds-its-fast-and-reliable-global-network/)

-   [點燃網路創新](https://azure.microsoft.com/blog/lighting-up-network-innovation/)
