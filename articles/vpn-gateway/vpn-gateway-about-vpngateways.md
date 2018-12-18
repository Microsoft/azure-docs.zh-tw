---
title: Azure VPN 閘道 | Microsoft Docs
description: 深入了解 VPN 閘道是什麼，以及您可以使用 VPN 閘道連線到 Azure 虛擬網路的方式。 包括 IPsec/IKE 站對站跨單位和 VNet 對 VNet 解決方案，以及點對站 VPN。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: As someone with a basic network background, but is new to Azure, I want to understand the capabilities of Azure VPN Gateway so that I can securely connect to my Azure virtual networks.
ms.assetid: 2358dd5a-cd76-42c3-baf3-2f35aadc64c8
ms.service: vpn-gateway
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/19/2018
ms.author: cherylmc
ms.openlocfilehash: 01052d8058d1b1e3a03f584ad66da2ad79f4c425
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32193201"
---
# <a name="what-is-vpn-gateway"></a>什麼是 VPN 閘道？

VPN 閘道是特定的虛擬網路閘道類型，可透過公用網際網路在 Azure 虛擬網路與內部部署位置之間傳送加密流量。 您也可以使用 VPN 閘道，透過 Microsoft 網路來傳送 Azure 虛擬網路之間的已加密流量。 每個虛擬網路只能有一個 VPN 閘道。 不過，您可以對相同的 VPN 閘道建立多個連線。 當您對相同的 VPN 閘道建立多個連線時，所有 VPN 通道都會共用可用的閘道頻寬。

## <a name="whatis"></a>什麼是虛擬網路閘道？

虛擬網路閘道是由部署到您所建立的特定子網路 (稱為「閘道子網路」) 的兩部以上虛擬機器所組成。 當您建立虛擬網路閘道時，會建立位於閘道子網路中的 VM。 虛擬網路閘道 VM 會設定為包含閘道特有的路由表和閘道服務。 您無法直接設定屬於虛擬網路閘道的 VM，您不該將額外的資源部署至閘道子網路。

建立虛擬網路閘道最多可能需要花費 45 分鐘的時間來完成。 建立虛擬網路閘道時，閘道 VM 會部署到閘道子網路，並使用您指定的設定進行設定。 您所設定的其中一個設定是閘道類型。 閘道類型 'vpn' 會指定所建立虛擬網路閘道的類型是 VPN 閘道。 建立 VPN 閘道之後，您可以在 VPN 閘道與另一個 VPN 閘道 (VNet 對 VNet) 之間建立 IPsec/IKE VPN 通道連線，或在 VPN 閘道與內部部署 VPN 裝置 (站對站) 之間建立跨單位 IPsec/IKE VPN 通道連線。 您也可以建立點對站 VPN 連線 (透過 IKEv2 或 SSTP 的 VPN)，它可讓您從遠端位置連線到您的虛擬網路，例如從會議或住家。

## <a name="configuring"></a>設定 VPN 閘道

VPN 閘道連線需仰賴多個具有特定設定的資源。 大部分的資源可以分別進行設定，雖然必須以特定順序設定某些資源。

### <a name="settings"></a>設定

您為每個資源選擇的設定，對於建立成功連線而言極為重要。 如需 VPN 閘道個別資源和設定的資訊，請參閱 [關於 VPN 閘道設定](vpn-gateway-about-vpn-gateway-settings.md)。 本文包含的資訊可協助您了解閘道類型、閘道 SKU、VPN 類型、連線類型、閘道子網路、區域網路閘道，以及您需要考量的各種其他資源設定。

### <a name="tools"></a>部署工具

您可以使用設定工具 (例如 Azure 入口網站) 開始建立及設定資源。 您可以稍後再決定切換到另一個工具 (如 PowerShell) 來設定其他資源，或是在適用的時機修改現有資源。 您目前無法在 Azure 入口網站中進行每一項資源和資源設定。 文章中各連線拓撲的指示會指定何時需要特定組態工具。 

### <a name="models"></a>部署模型

目前有兩種部署模型適用於 Azure。 當您設定 VPN 閘道時，您採用的步驟取決於用來建立虛擬網路的部署模型。 例如，如果您使用傳統部署模型建立 VNet，您會使用傳統部署模型的指導方針和指示來建立和進行 VPN 閘道設定。 如需部署模型的詳細資訊，請參閱 [了解 Resource Manager 和傳統部署模型](../azure-resource-manager/resource-manager-deployment-model.md)。

### <a name="planningtable"></a>規劃表

下表可以協助您為您的解決方案決定最佳的連線選項。

[!INCLUDE [cross-premises](../../includes/vpn-gateway-cross-premises-include.md)]

## <a name="gwsku"></a>閘道 SKU

建立虛擬網路閘道時，您必須指定想要使用的閘道 SKU。 根據工作負載、輸送量、功能和 SLA 的類型，選取符合您需求的 SKU。 如需閘道 SKU 的詳細資訊，包括支援功能、實際執行環境和開發測試和設定步驟，請參閱[閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。

### <a name="benchmark"></a>依通道、連線和輸送量區分的閘道 SKU

[!INCLUDE [Aggregated throughput by SKU](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

## <a name="diagrams"></a>連線拓撲圖表

請務必知道 VPN 閘道連線有不同的組態可用。 您必須決定哪個組態最符合您的需求。 在下列各節中，您可以檢視有關下列 VPN 閘道連線的資訊和拓撲圖表︰下列各節包含的資料表可列出︰

* 可用的部署模型
* 可用的設定工具
* 直接帶您前往某篇文章的連結 (如果可用)

使用圖形和描述來協助選取符合您需求的連線拓撲。 這些圖表顯示主要基準拓撲，但您也可以使用這些圖表做為指導方針來建置更複雜的組態。

## <a name="s2smulti"></a>站對站以及多網站 (IPsec/IKE VPN 通道)

### <a name="S2S"></a>站對站

網站間 (S2S) VPN 閘道連線是透過 IPsec/IKE (IKEv1 或 IKEv2) VPN 通道建立的連線。 S2S 連線可以用於跨單位與混合式組態。 S2S 連線需要位於內部部署的 VPN 裝置，其具有指派的公用 IP 位址且不是位於 NAT 後方。 如需選取 VPN 裝置的資訊，請參閱 [VPN 閘道常見問題集 - VPN 裝置](vpn-gateway-vpn-faq.md#s2s)。

![Azure VPN 閘道站對站連接範例](./media/vpn-gateway-about-vpngateways/vpngateway-site-to-site-connection-diagram.png)

### <a name="Multi"></a>多站台

這類型的連線是站對站連線的變化。 您可以從虛擬網路閘道建立多個 VPN 連線，通常會連接至多個內部部署網站。 處理多重連線時，您必須使用路由式 VPN 類型 (也就是使用傳統 VNet 時的動態閘道)。 因為每個虛擬網路只能有一個 VPN 閘道，所以透過該閘道的所有連線會共用可用的頻寬。 這種組態通常稱為「多站台」連線。

![Azure VPN 閘道多網站連接範例](./media/vpn-gateway-about-vpngateways/vpngateway-multisite-connection-diagram.png)

### <a name="deployment-models-and-methods-for-site-to-site-and-multi-site"></a>站對站和多網站的部署模型和方法

[!INCLUDE [site-to-site and multi-site table](../../includes/vpn-gateway-table-site-to-site-include.md)]

## <a name="P2S"></a>點對站 (透過 IKEv2 或 SSTP 的 VPN)

點對站 (P2S) VPN 閘道連線可讓您建立從個別用戶端電腦到您的虛擬網路的安全連線。 P2S 連線的建立方式是從用戶端電腦開始。 此解決方案適合於想要從遠端位置 (例如從住家或會議) 連線到 Azure VNet 的遠距工作者。 當您只有少數用戶端必須連線至 VNet 時，P2S VPN 也是很實用的解決方案 (而不是 S2S VPN)。

如同 S2S 連線，P2S 連線不需要內部部署公眾對應 IP 位址或 VPN 裝置。 P2S 連線可與 S2S 連線透過相同的 VPN 閘道一起使用，前提是這兩個連線的所有設定需求都相容。 如需點對站連線的詳細資訊，請參閱[關於點對站 VPN](point-to-site-about.md)。


![Azure VPN 閘道點對站連接範例](./media/vpn-gateway-about-vpngateways/point-to-site.png)

### <a name="deployment-models-and-methods-for-p2s"></a>P2S 的部署模型和方法

[!INCLUDE [vpn-gateway-table-site-to-site](../../includes/vpn-gateway-table-point-to-site-include.md)]

## <a name="V2V"></a>VNet 對 VNet 連線 (IPsec/IKE VPN 通道)

將虛擬網路連接至另一個虛擬網路 (VNet 對 VNet)，類似於將 VNet 連接至內部部署網站位置。 這兩種連線類型都使用 VPN 閘道提供使用 IPsec/IKE 的安全通道。 您甚至可以將多網站連線組態與 VNet 對 VNet 通訊結合。 這可讓您建立結合了跨單位連線與內部虛擬網路連線的網路拓撲。

您所連接的 VNet 可以：

* 在相同或不同的區域中
* 在相同或不同的訂用帳戶中 
* 在相同或不同的部署模型中

![Azure VPN 閘道 VNet 對 VNet 連接範例](./media/vpn-gateway-about-vpngateways/vpngateway-vnet-to-vnet-connection-diagram.png)

### <a name="connections-between-deployment-models"></a>部署模型之間的連線

Azure 目前有兩種部署模型：傳統和 Resource Manager。 如果您已使用 Azure 一段時間，則可能具有傳統 VNet 上執行的 Azure VM 和執行個體角色。 較新的 VM 和角色執行個體可能會在 Resource Manager 中建立的 VNet 中執行。 您可以建立 Vnet 間的連線，讓其中一個 VNet 中的資源直接與另一個 VNet 中的資源通訊。

### <a name="vnet-peering"></a>VNet 對等互連

只要您的虛擬網路符合特定需求，您就能夠使用 VNet 對等互連來建立連線。 VNet 對等互連不會使用虛擬網路閘道。 如需詳細資訊，請參閱 [VNet 對等互連](../virtual-network/virtual-network-peering-overview.md)。

### <a name="deployment-models-and-methods-for-vnet-to-vnet"></a>VNet 對 VNet 的部署模型和方法

[!INCLUDE [vpn-gateway-table-vnet-to-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

## <a name="ExpressRoute"></a>ExpressRoute (私人連線)

ExpressRoute 可讓您透過連線提供者所提供的私人連線，將內部部署網路延伸至 Microsoft 雲端。 透過 ExpressRoute，您可以建立 Microsoft 雲端服務的連線，例如 Microsoft Azure、Office 365 和 CRM Online。 從任意點對任意點 (IP VPN) 網路、點對點乙太網路，或在共置設施上透過連線提供者的虛擬交叉連接，都可以進行連線。

ExpressRoute 連線不會經過公用網際網路。 相較於一般網際網路連線，這可讓 ExpressRoute 連線提供更可靠、更快速、延遲更短和更安全的連線。

ExpressRoute 連線會以虛擬網路閘道作為其必要組態的一部分。 在 ExpressRoute 連線中，虛擬網路閘道的閘道類型已設定為 'ExpressRoute' 而不是 'Vpn'。 雖然透過 ExpressRoute 線路傳輸的流量預設並未加密，但有可能建立一個解決方案，讓您透過 ExpressRoute 線路傳送已加密的流量。 如需 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 技術概觀](../expressroute/expressroute-introduction.md)。

## <a name="coexisting"></a>站對站及 ExpressRoute 並存連線

ExpressRoute 是從 WAN (不透過公用網際網路) 到 Microsoft 服務 (包括 Azure) 的私人連線。 站對站 VPN 流量會以加密方式透過公用網際網路進行傳輸。 能夠對相同的虛擬網路設定網站間 VPN 和 ExpressRoute 連線有諸多好處。

您可以將網站間 VPN 設定為 ExpressRoute 的安全容錯移轉路徑，或使用網站間 VPN 來連線至不在您網路中但透過 ExpressRoute 連接的網站。 請注意，對於相同的虛擬網路，此組態需要兩個虛擬網路閘道，一個使用 'Vpn' 閘道類型，而另一個使用 'ExpressRoute' 閘道類型。

![ExpressRoute 和 VPN 閘道並存連接範例](./media/vpn-gateway-about-vpngateways/expressroute-vpngateway-coexisting-connections-diagram.png)

### <a name="deployment-models-and-methods-for-s2s-and-expressroute-coexist"></a>S2S 和 ExpressRoute 的部署模型和方法並存

[!INCLUDE [vpn-gateway-table-coexist](../../includes/vpn-gateway-table-coexist-include.md)]

## <a name="pricing"></a>價格

[!INCLUDE [vpn-gateway-about-pricing-include](../../includes/vpn-gateway-about-pricing-include.md)]

如需 VPN 閘道之閘道 SKU 的詳細資訊，請參閱[閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。

## <a name="faq"></a>常見問題集

如需 VPN 閘道的常見問題集，請參閱 [VPN 閘道常見問題集](vpn-gateway-vpn-faq.md)。

## <a name="next-steps"></a>後續步驟

- 規劃您的 VPN 閘道設定。 請參閱 [VPN 閘道的規劃與設計](vpn-gateway-plan-design.md)。
- 如需詳細資訊，請參閱 [VPN 閘道常見問題集](vpn-gateway-vpn-faq.md)。
- 檢視[訂用帳戶與服務限制](../azure-subscription-service-limits.md#networking-limits)。
- 了解 Azure 的一些其他重要[網路功能](../networking/networking-overview.md)。
