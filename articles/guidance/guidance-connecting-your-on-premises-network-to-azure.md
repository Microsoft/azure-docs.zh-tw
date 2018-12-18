---
title: 將內部部署網路連線到 Azure | Microsoft Docs
description: 說明並比較可用來連線到諸如 Azure、Office 365 和 Dynamics CRM Online 等 Microsoft 雲端服務的各種方法。
services: ''
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: ''
ms.assetid: 294d39ad-40e0-476a-a362-57911b1172b7
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/25/2016
ms.author: cherylmc
ms.openlocfilehash: 37d83d3b6dea1763d85f2411816ba2fee4279100
ms.sourcegitcommit: e19f6a1709b0fe0f898386118fbef858d430e19d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/16/2018
ms.locfileid: "26730333"
---
# <a name="connecting-your-on-premises-network-to-azure"></a>將內部部署網路連線到 Azure
Microsoft 提供多種類型的雲端服務。 您可以透過公用網際網路連線到所有服務，同時也可以透過網際網路或透過 Microsoft 的直接、私人連線，使用虛擬私人網路 (VPN) 通道連線到某些服務。 本文以您使用的 Microsoft 雲端服務類型為基礎，協助您判斷最符合您需求的連線選項。 大部分的組織使用多種連線類型，如下所述。

## <a name="connecting-over-the-public-internet"></a>透過公用網際網路連線
此連線類型能直接透過網際網路存取 Microsoft 雲端服務，如下所示。

![Internet](./media/guidance-connecting-your-on-premises-network-to-azure/internet.png "Internet")

此連線通常是用來連線到 Microsoft 雲端服務的第一個類型。 下表列出此連線類型的優缺點。

| **優點** | **考量** |
| --- | --- |
| 只要所有用戶端裝置都可無限制存取所有 IP 位址和網際網路上的連接埠，就不需要在內部部署網路進行任何修改。 |雖然流量通常是使用 HTTPS 進行加密，但在傳輸過程中可能會因周遊公用網際網路而遭到攔截。 |
| 可以連線到公用網際網路中公開的所有 Microsoft 雲端服務。 |因連線周遊網際網路而造成無法預期的延遲。 |
| 使用現有的網際網路連線。 | |
| 不需要管理任何連線裝置。 | |

因為您是使用現有的網際網路連線，所以此連線沒有任何連線或頻寬成本。

## <a name="connecting-with-a-point-to-site-connection"></a>連線點對站連線
這種連線類型會透過網際網路，經由安全通訊端通道通訊協定 (SSTP) 通道存取某些 Microsoft 雲端服務，如下所示。

![P2S](./media/guidance-connecting-your-on-premises-network-to-azure/p2s.png "點對站連線")

會透過現有的網際網路連線進行連線，但需要使用 Azure VPN 閘道。 下表列出此連線類型的優缺點。

| **優點** | **考量** |
| --- | --- |
| 只要所有用戶端裝置都可無限制存取所有 IP 位址和網際網路上的連接埠，就不需要在內部部署網路進行任何修改。 |雖然流量是使用 IPSec 進行加密，但在傳輸過程中可能會因周遊公用網際網路而遭到攔截。 |
| 使用現有的網際網路連線。 |因連線周遊網際網路而造成無法預期的延遲。 |
| 每個閘道最多每秒 200 MB 的輸送量。 |需要建立及管理內部部署網路上每部裝置與每部裝置必須連線之每個閘道之間的個別連線。 |
| 可以用來連線到 Azure 服務，而這些服務可連線到諸如 Azure 虛擬機器和 Azure 雲端服務等 Azure 虛擬網路 (VNet)。 |需要在 Azure VPN 閘道進行最基本的管理。 |
| 無法用來連線到 Microsoft Office 365 或 Dynamics CRM Online。 | |
| 無法用來連線到不能與 VNet 連線的 Azure 服務。 | |

深入了解 [VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md)服務、其[定價](https://azure.microsoft.com/pricing/details/vpn-gateway)和輸出資料轉送[定價](https://azure.microsoft.com/pricing/details/data-transfers)。

## <a name="connecting-with-a-site-to-site-connection"></a>連線站對站連線
這種連線類型會透過網際網路，經由 IPSec 通道存取某些 Microsoft 雲端服務，如下所示。

![S2S](./media/guidance-connecting-your-on-premises-network-to-azure/s2s.png "站對站連線")

連線會透過現有的網際網路連線，但需要以其相關聯的定價和傳出資料轉送定價來使用 Azure VPN 閘道。 下表列出此連線類型的優缺點。

| **優點** | **考量** |
| --- | --- |
| 內部部署網路上的所有裝置都可以與連線至 VNet 的 Azure 服務進行通訊，因此不需要針對每部裝置設定個別的連線。 |雖然流量是使用 IPSec 進行加密，但在傳輸過程中可能會因周遊公用網際網路而遭到攔截。 |
| 使用現有的網際網路連線。 |因連線周遊網際網路而造成無法預期的延遲。 |
| 可以用來連線到 Azure 服務，而這些服務可連線到諸如虛擬機器和雲端服務等 VNet。 |必須設定及管理已驗證的內部部署 VPN 裝置 *。 |
| 每個閘道最多每秒 200 MB 的輸送量。 |需要在 Azure VPN 閘道進行最基本的管理。 |
| 可以強制透過內部部署網路從雲端虛擬機器起始的輸出流量，利用使用者定義的路由或邊界閘道通訊協定 (BGP)** 進行檢查和記錄。 |無法用來連線到 Microsoft Office 365 或 Dynamics CRM Online。 |
| 無法用來連線到不能與 VNet 連線的 Azure 服務。 | |
| 如果您使用的服務會將連線起始回到內部部署裝置，且您的安全性原則需要它，在內部部署網路與 Azure 之間可能就需要防火牆。 | |

* *檢視[已驗證的 VPN 裝置清單](../vpn-gateway/vpn-gateway-about-vpn-devices.md#devicetable)。
* \*\*深入了解使用[使用者定義的路由](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md)或 [BGP](../vpn-gateway/vpn-gateway-bgp-overview.md) 強制從 Azure VNet 路由至內部部署裝置。

## <a name="connecting-with-a-dedicated-private-connection"></a>連線專用的私人連線
這種連線類型會透過 Microsoft 的專用私人連線，以不周遊網際網路的方式存取所有的 Microsoft 雲端服務，如下所示。

![ER](./media/guidance-connecting-your-on-premises-network-to-azure/er.png "ExpressRoute 連線")

連線需要使用 ExpressRoute 服務和連線提供者的連線。 下表列出此連線類型的優缺點。

| **優點** | **考量** |
| --- | --- |
| 因為使用了服務提供者的專用連線，所以無法在傳輸過程中透過公用網際網路流量攔截。 |需要內部部署路由器管理。 |
| 每個 ExpressRoute 電路最高每秒 10 Gb 的頻寬，和每個閘道最多每秒 2 Gb 的輸送量。 |需要連線服務提供者的專用連線。 |
| 由於 Microsoft 專用連線不會周遊網際網路，因此可預測延遲。 |可能需要在一或多個 Azure VPN 閘道進行最基本的管理 (如果連線到 VNet 的電路)。 |
| 不需要將通訊加密，但您可以視需要將流量加密。 |如果您使用的雲端服務會將連線起始回到內部部署裝置，在內部部署網路與 Azure 之間可能就需要防火牆。 |
| 能夠直接連線到所有的 Microsoft 雲端服務，但有少數例外狀況*。 |對於無法連線至 VNet 的服務，內部部署 IP 位址的網路位址轉譯 (NAT) 必須進入 Microsoft 資料中心。\*\* |
| 可以強制透過內部部署網路從雲端虛擬機器起始的輸出流量，利用 BGP 進行檢查和記錄。 | |

* *檢視無法與 ExpressRoute 搭配使用的[服務清單](../expressroute/expressroute-faqs.md#supported-services)。 您的 Azure 訂用帳戶必須受到核准，才能連線到 Office 365。  如需詳細資料，請參閱 [Azure ExpressRoute for Office 365](https://support.office.com/article/Azure-ExpressRoute-for-Office-365-6d2534a2-c19c-4a99-be5e-33a0cee5d3bd?ui=en-US&rs=en-US&ad=US&fromAR=1) 一文。
* \*\*深入了解 ExpressRoute [NAT](../expressroute/expressroute-nat.md) 需求。

深入了解 [ExpressRoute](../expressroute/expressroute-introduction.md)，及其相關聯[定價](https://azure.microsoft.com/pricing/details/expressroute)和[連線提供者](../expressroute/expressroute-locations.md#locations)。

## <a name="additional-considerations"></a>其他考量
* 在上述幾個選項中，對於可支援的 VNet 連線、閘道連線及其他條件，都提供不同的上限標準。 建議您檢閱 Azure [網路限制](../azure-subscription-service-limits.md#networking-limits)，了解是否會對您選擇要使用的連線類型造成任何影響。
* 如果您打算從站對站 VPN 連線將閘道連線至與 ExpressRoute 閘道相同的 VNet，應該先熟悉重要的條件約束。 如需詳細資料，請參閱[設定 ExpressRoute 和站對站並存連線](../expressroute/expressroute-howto-coexist-resource-manager.md#limits-and-limitations)一文。

## <a name="next-steps"></a>後續步驟
下列資源會說明如何實作本文所涵蓋的連線類型。

* [實作點對站連線](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)
* [實作站對站連線](guidance-hybrid-network-vpn.md)
* [實作專用的私人連線](guidance-hybrid-network-expressroute.md)
* [使用站對站連線實作專用的私人連線以獲得高可用性](guidance-hybrid-network-expressroute-vpn-failover.md)
