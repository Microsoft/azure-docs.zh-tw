---
title: 關於在 Azure 可用性區域中的區域備援虛擬網路閘道 | Microsoft Docs
description: 了解在可用性區域中的 VPN 閘道和 ExpressRoute 閘道。
services: vpn-gateway
author: cherylmc
Customer intent: As someone with a basic network background, I want to understand zone-redundant gateways.
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: cherylmc
ms.openlocfilehash: 27bac5265a5e884b808c4ccb58fda0b2fffeb774
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46975644"
---
# <a name="about-zone-redundant-virtual-network-gateways-in-azure-availability-zones"></a>關於在 Azure 可用性區域中的區域備援虛擬網路閘道

您可以在 [Azure 可用性區域](../availability-zones/az-overview.md)中部署 VPN 和 ExpressRoute 閘道。 此方式可為虛擬網路閘道帶來復原力、延展性和更高的可用性。 在 Azure 可用性區域中部署閘道可從根本上和邏輯上分隔區域內的閘道，同時還能在發生區域層級的失敗時，保護您內部部署項目與 Azure 的網路連線。

### <a name="zrgw"></a>區域備援閘道

若要在各個可用性區域上自動部署虛擬網路閘道，您可以使用區域備援虛擬網路閘道。 透過區域備援閘道，您可以受益於區域備援來存取 Azure 上具任務關鍵性和延展性的服務。

<br>
<br>

![區域備援閘道圖示](./media/create-zone-redundant-vnet-gateway/zonered.png)

### <a name="zgw"></a>分區閘道

若要在特定區域中部署閘道，您可以使用分區閘道。 當您部署分區閘道時，閘道的所有執行個體都會部署在相同的可用性區域中。

<br>
<br>

![分區閘道圖示](./media/create-zone-redundant-vnet-gateway/zonal.png)

## <a name="gwskus"></a>閘道 SKU

區域備援和分區閘道是作為新的閘道 SKU 供使用。 在 Azure AZ 區域中，我們已新增新的虛擬網路閘道 SKU。 這些 SKU 與 ExpressRoute 和 VPN 閘道的對應現有 SKU 類似，不同之處在於這些 SKU 專用於區域備援和分區閘道。

新的閘道 SKU 為：

### <a name="vpn-gateway"></a>VPN 閘道

* VpnGw1AZ
* VpnGw2AZ
* VpnGw3AZ

### <a name="expressroute"></a>ExpressRoute

* ErGw1AZ
* ErGw2AZ
* ErGw3AZ

## <a name="pipskus"></a>公用 IP SKU

區域備援閘道和分區閘道皆會依賴 Azure 公用 IP 資源的「標準」SKU。 Azure 公用 IP 資源組態可決定您部署的閘道是區域備援或分區閘道。 如果您以「基本」SKU 建立公用 IP 資源，閘道就不會有任何區域備援，且閘道資源將會是區域性的。

### <a name="pipzrg"></a>區域備援閘道

當您使用**標準**公用 IP SKU 建立公開 IP 位址，但沒有指定區域時，行為會根據閘道是 VPN 閘道或 ExpressRoute 閘道而有所差異。 

* 如果是 VPN 閘道，會將兩個閘道執行個體部署在這三個區域的其中任兩個，以提供區域備援。 
* 如果是 ExpressRoute 閘道，由於可能會有兩個以上的執行個體，閘道可以橫跨所有三個區域。

### <a name="pipzg"></a>分區閘道

當您使用**標準**公用 IP SKU 建立公開 IP 位址，並指定區域 (1、2 或 3) 時，所有閘道執行個體都會部署在相同區域中。

### <a name="piprg"></a>區域閘道

當您使用**基本**公用 IP SKU 建立公開 IP 位址時，閘道會部署為區域閘道，而且閘道內不會建置任何區域備援。

## <a name="faq"></a>常見問題集

### <a name="what-will-change-when-i-deploy-these-new-skus"></a>部署這些新的 SKU 時，將會改變什麼項目？

從您的觀點來看，您可以部署具有區域備援的閘道。 這表示閘道的所有執行個體將會部署到各個 Azure 可用性區域，而且每個可用性區域都是不同的容錯網域和更新網域。 面對區域失敗時，這可讓您的閘道具有更高的可靠性、可用性及復原能力。

### <a name="can-i-use-the-azure-portal"></a>我是否可使用 Azure 入口網站？

是，您可以使用 Azure 入口網站來部署新的 SKU。 不過，您只會在具有 Azure 可用性區域的 Azure 區域中看到這些新的 SKU。

### <a name="what-regions-are-available-for-me-to-use-the-new-skus"></a>我可以在哪些區域中使用新的 SKU？

新的 SKU 可在具有 Azure 可用性區域的 Azure 區域 (美國中部、法國中部和西歐區域) 中取得。 接下來，我們會在其他 Azure 公用區域中推出區域備援閘道。

### <a name="can-i-changemigrateupgrade-my-existing-virtual-network-gateways-to-zone-redundant-or-zonal-gateways"></a>我可以將現有的虛擬網路閘道變更/遷移/升級為區域備援或分區閘道嗎？

目前不支援將現有的虛擬網路閘道遷移至區域備援或分區閘道。 不過，您可以刪除現有的閘道，然後重新建立區域備援或分區閘道。

### <a name="can-i-deploy-both-vpn-and-express-route-gateways-in-same-virtual-network"></a>可以在相同的虛擬網路中部署 VPN 和 Express Route 閘道嗎？

支援 VPN 和 Express Route 閘道共存於相同的虛擬網路中。 不過，您應該保留閘道子網路的 /27 IP 位址範圍。

## <a name="next-steps"></a>後續步驟

[建立區域備援虛擬網路閘道](create-zone-redundant-vnet-gateway.md)