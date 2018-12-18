---
title: 跨單位連線的規劃和設計：Azure VPN 閘道| Microsoft Docs
description: 深入了解跨單位、混合式和 VNet 對 VNet 連線的 VPN 閘道規劃與設計
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: d5aaab83-4e74-4484-8bf0-cc465811e757
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/27/2017
ms.author: cherylmc
ms.openlocfilehash: 0ebc3ef4a64432e993dd6ed69766bb64544fe433
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23125476"
---
# <a name="planning-and-design-for-vpn-gateway"></a>規劃與設計 VPN 閘道

跨單位及 VNet 對 VNet 組態的規劃與設計有可能很簡單，也可能很複雜，需視您的網路需求而定。 本文將逐步引導您完成基本的規劃和設計考量。

## <a name="planning"></a>規劃

### <a name="compare"></a>跨單位連線選項

如果您想要將內部部署站台安全地連接到虛擬網路，可以使用下列三種不同方法：站對站、點對站及 ExpressRoute。 比較可使用的不同跨單位連線。 您選擇的選項可能取決於各種不同的考量，例如：

* 您的方案需要哪種輸送量?
* 您想讓通訊透過經由安全 VPN 的公用網際網路或透過私人連線?
* 您有可供使用的公用 IP 位址嗎?
* 您打算使用 VPN 裝置嗎? 如果是，它相容嗎？
* 您只連線幾台電腦或您想要網站持續連線?
* 您想要建立的方案需要什麼類型的 VPN 閘道?
* 您應該使用哪一種閘道 SKU？

### <a name="planningtable"></a>規劃表

下表可以協助您為您的解決方案決定最佳的連線選項。

[!INCLUDE [vpn-gateway-cross-premises](../../includes/vpn-gateway-cross-premises-include.md)]

### <a name="gwsku"></a>閘道 SKU

[!INCLUDE [vpn-gateway-table-gwtype-aggtput](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

### <a name="wf"></a>工作流程

下列清單列出常見的雲端連線工作流程：

1. 設計和規劃連線的拓撲，列出所有您想要連接的網路位址空間。
2. 建立 Azure 虛擬網路。 
3. 建立虛擬網路的 VPN 閘道。
4. 建立及設定連接至內部網路或其他虛擬網路的連線 (視需要)。
5. 建立並設定 Azure VPN 閘道的點對站連線 (視需要)。

## <a name="design"></a>設計
### <a name="topologies"></a>連線拓撲

先來看看 [關於 VPN 閘道](vpn-gateway-about-vpngateways.md) 一文中的圖表。 本文包含基本圖表、每種拓撲的部署模型，以及您可以用來部署組態的可用部署工具。

### <a name="designbasics"></a>設計基本概念

下列各節將討論 VPN 閘道的基本概念。 

#### <a name="servicelimits"></a>網路服務限制

捲動表格以檢視[網路服務限制](../azure-subscription-service-limits.md#networking-limits)。 上述限制可能會影響您的設計。

#### <a name="subnets"></a>關於子網路

當您建立連線時，必須考量您的子網路範圍。 子網路位址範圍不能重疊。 當一個虛擬網路或內部部署位置包含的位址空間與其他位置重複時，就會發生子網路重疊的情況。 這表示您需要請本機內部部署網路的網路工程師為您切割出一個範圍，以供您用於 Azure IP 位址空間/子網路。 您需要本機內部部署網路並未使用的位址空間。

使用 VNet 對 VNet 連線時，也要注意避免重疊的子網路。 如果您的子網路重疊且 IP 位址同時存在於傳送端和目的地 VNet，VNet 對 VNet 連線就會失敗。 Azure 無法將資料路由傳送到另一個 VNet，因為目的地位址是傳送端 VNet 的一部分。

「VPN 閘道」需要名為閘道子網路的特定子網路。 所有閘道子網路都必須命名為 GatewaySubnet 才能正常運作。 因此，請不要將閘道子網路命名為不同的名稱，也不要將 VM 或任何其他項目部署至閘道子網路。 請參閱 [閘道子網路](vpn-gateway-about-vpn-gateway-settings.md#gwsub)。

#### <a name="local"></a>關於區域網路閘道

區域網路閘道通常是指您的內部部署位置。 在傳統部署模型中，區域網路閘道被稱為「區域網路站台」。 當您設定區域網路閘道時，您會賦予它名稱、指定內部部署 VPN 裝置的公用 IP 位址，以及指定位於內部部署位置中的位址首碼。 Azure 會查看網路流量的目的地位址首碼、查閱您為區域網路閘道指定的組態，然後根據這些來路由傳送封包。 您可以視需要修改位址首碼。 如需詳細資訊，請參閱[區域網路閘道](vpn-gateway-about-vpn-gateway-settings.md#lng)。

#### <a name="gwtype"></a>關於閘道類型

為拓撲選取正確的閘道類型相當重要。 如果您選取錯誤的類型，您的閘道將無法正常運作。 閘道類型會指定閘道本身如何連接以及為何它是 Resource Manager 部署模型的必要組態設定。

閘道類型如下：

* Vpn
* ExpressRoute

#### <a name="connectiontype"></a>關於連線類型

每個組態皆需要特定的連線類型。 連線類型如下：

* IPsec
* Vnet2Vnet
* ExpressRoute
* VPNClient

#### <a name="vpntype"></a>關於 VPN 類型

每個組態都需要特定 VPN 類型。 如果您要結合兩個組態，例如建立連往相同 VNet 的站對站連線和點對站連線，您必須使用同時符合這兩個連線需求的 VPN 類型。

[!INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]

下表顯示 VPN 類型所對應的每一種連線組態。 請確定閘道的 VPN 類型符合您想要建立的組態。 

[!INCLUDE [vpn-gateway-table-vpntype](../../includes/vpn-gateway-table-vpntype-include.md)]

### <a name="devices"></a>站對站連線的 VPN 裝置

不論部署模型為何，若要設定站對站連線，您都需要下列項目︰

* 與 Azure VPN 閘道相容的 VPN 裝置
* 不在 NAT 後方的公開 IPv4 IP 位址

您必須具備設定 VPN 裝置的經驗，或是有人可以為您設定裝置。

[!INCLUDE [vpn-gateway-configure-vpn-device-rm](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

### <a name="forcedtunnel"></a>考量強制通道路由

對於多數組態，您可以設定強制通道。 強制通道可讓您透過站對站 VPN 通道，重新導向或「強制」所有網際網路繫結流量傳回內部部署位置，以便進行檢查和稽核。 這是多數企業 IT 原則的重要安全性需求。 

若不使用強制通道，則 Azure 中來自 VM 的網際網路繫結流量會永遠從 Azure 網路基礎結構直接向外周遊到網際網路，而您無法選擇檢查或稽核流量。 未經授權的網際網路存取可能會導致資訊洩漏或其他類型的安全性漏洞。

可以在這兩種部署模型中使用不同的工具，設定強制通道的連線。 如需詳細資訊，請參閱[設定強制通道](vpn-gateway-forced-tunneling-rm.md)。

**強制通道圖表**

![Azure VPN 閘道強制通道圖表](./media/vpn-gateway-plan-design/forced-tunneling-diagram.png)

## <a name="next-steps"></a>後續步驟

如需可協助您進行設計的詳細資訊，請參閱 [VPN 閘道常見問題集](vpn-gateway-vpn-faq.md)一文和[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)一文。

如需有關特定閘道設定的詳細資訊，請參閱 [關於 VPN 閘道設定](vpn-gateway-about-vpn-gateway-settings.md)。