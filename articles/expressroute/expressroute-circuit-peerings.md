---
title: Azure ExpressRoute 線路和路由網域 | Microsoft Docs
description: 此頁面提供 ExpressRoute 線路和路由網域的概觀。
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 10/01/2018
ms.author: cherylmc
ms.openlocfilehash: 8ef231982cfdc9a2c4387ee47e651fdf1af0baaf
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388420"
---
# <a name="expressroute-circuits-and-routing-domains"></a>ExpressRoute 線路和路由網域
 您必須訂購 *ExpressRoute 線路* ，才能將內部部署基礎結構透過連線提供者連接到 Microsoft。 下圖所示的是 WAN 與 Microsoft 之間連線的邏輯表示法。

![](./media/expressroute-circuit-peerings/expressroute-basic.png)

## <a name="expressroute-circuits"></a>ExpressRoute 線路
*ExpressRoute 線路* 代表內部部署基礎結構與 Microsoft 雲端服務之間透過連線提供者的邏輯連線。 您可以訂購多條 ExpressRoute 線路。 每一條線路可以位於相同或不同區域，且可透過不同的連線提供者連線到您的內部環境。 

ExpressRoute 線路不對應至任何實體裝置。 線路由一個稱為服務金鑰 (s 金鑰) 的標準 GUID 唯一識別。 服務金鑰是 Microsoft、連線提供者與您之間唯一會交換的資訊。 S 金鑰不是安全性用途的密碼。 ExpressRoute 線路與 s 金鑰之間存在 1:1 對應。

ExpressRoute 線路最多可以有三個獨立對等：Azure 公用、Azure 私用和 Microsoft。 每個對等為一組獨立的 BGP 工作階段，每個都重複設定，以確保較高的可用性。 ExpressRoute 線路與路由網域之間存在 1:N (1 <= N <= 3) 對應。 每個 ExpressRoute 電路可以啟用任何一個、兩個或全部三個對等。

每個電路有固定的頻寬 (50 Mbps、100 Mbps、200 Mbps、500 Mbps、1 Gbps、10 Gbps)，而且對應至連線提供者和對等位置。 您選取的頻寬由線路上的所有對等共用。 

### <a name="quotas-limits-and-limitations"></a>配額和限制
每個 ExpressRoute 線路會套用預設的配額和限制。 如需最新的配額資訊，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](../azure-subscription-service-limits.md) 。

## <a name="expressroute-routing-domains"></a>ExpressRoute 路由網域
ExpressRoute 線路有多個相關聯的路由網域：Azure 公用、Azure 私用和 Microsoft。 每個路由網域在一對路由器上的設定完全相同 (使用主動-主動或載入共用組態)，可確保較高的可用性。 Azure 服務分成 *Azure 公用*和 *Azure 私用*兩類來代表 IP 定址配置。

![](./media/expressroute-circuit-peerings/expressroute-peerings.png)

### <a name="azure-private-peering"></a>Azure 私用對等
部署於虛擬網路內的 Azure 計算服務 (也就是虛擬機器 (IaaS) 和雲端服務 (PaaS)) 可透過私用對等網域進行連線。 私人對等互連網域會被視為核心網路至 Microsoft Azure 的受信任延伸。 您可以在核心網路與 Azure 虛擬網路 (VNet) 之間設定雙向連線。 此對等可讓您直接連線到虛擬機器和雲端服務的私人 IP 位址。  

您可以將多個虛擬網路連線到私人對等互連網域。 如需限制的相關資訊，請檢閱 [常見問題集頁面](expressroute-faqs.md) 。 如需最新的限制資訊，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](../azure-subscription-service-limits.md) 。  如需路由組態的詳細資訊，請參閱 [路由](expressroute-routing.md) 頁面。

### <a name="microsoft-peering"></a>Microsoft 對等互連

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

與 Microsoft 線上服務 (Office 365、Dynamics 365 及 Azure PaaS 服務) 的連線將會透過 Microsoft 對等互連進行。 我們會透過 Microsoft 對等路由網域啟用 WAN 與 Microsoft 雲端服務之間的雙向連線。 您只能透過您或連線提供者所擁有的公用 IP 位址連線到 Microsoft 雲端服務，且必須遵守所有已定義的規則。 如需詳細資訊，請參閱 [ExpressRoute 必要條件](expressroute-prerequisites.md)頁面。

如需支援的服務、費用和組態的詳細資訊，請參閱 [常見問題集頁面](expressroute-faqs.md) 。 如需有關提供 Microsoft 對等支援的連線提供者清單的詳細資訊，請參閱 [ExpressRoute 位置](expressroute-locations.md) 頁面。

### <a name="azure-public-peering"></a>Azure 公用對等互連 (已被新的線路取代)

> [!IMPORTANT]
> 新的線路無法使用 Azure 公用對等互連。  
>   

公用 IP 位址上提供如 Azure 儲存體、SQL Database 和網站等服務。 您可以透過公用對等互連路由網域，私人連線到在公用 IP 位址上託管的服務 (包括雲端服務的 VIP)。 您可以將公用對等網域連線到 DMZ，並從您的 WAN 連線到所有 Azure 服務的公用 IP 位址，而無需透過網際網路進行連線。 

一律會從您的 WAN 啟動連線到 Microsoft Azure 服務。 Microsoft Azure 服務將無法透過這個路由網域啟動連線到您的網路。 一旦啟用公用對等互連，您便能夠連線到所有 Azure 服務。 我們不允許您選擇性地選取已通告路由的服務。

您可以在您的網路內定義自訂路由篩選條件，以便僅取用所需的路由。 如需路由組態的詳細資訊，請參閱 [路由](expressroute-routing.md) 頁面。 

如需有關透過公用對等路由網域支援的服務詳細資訊，請參閱[常見問題集](expressroute-faqs.md)。

## <a name="routing-domain-comparison"></a>路由網域的比較
下表是這三種路由網域的比較：

|  | **私人對等互連** | **Microsoft 對等互連** |  **公用對等互連** (已被新的線路取代) |
| --- | --- | --- | --- |
| **每個對等支援的前置詞數目最大值** |預設為 4000，ExpressRoute Premium 中為 10000 |200 |200 |
| **支援的 IP 位址範圍** |您的 WAN 內任何有效的 IP 位址。 |您或您的連線提供者所擁有的公用 IP 位址。 |您或您的連線提供者所擁有的公用 IP 位址。 |
| **AS 編號需求** |私密和公用 AS 編號。 您必須擁有公用 AS 編號，才能選擇使用其中一個編號。 |私密和公用 AS 編號。 不過，您必須證明公用 IP 位址的擁有權。 |私密和公用 AS 編號。 不過，您必須證明公用 IP 位址的擁有權。 |
| **支援的 IP 通訊協定**| IPv4 |  IPv4、IPv6 | IPv4 |
| **路由介面 IP 位址** |RFC1918 和公用 IP 位址 |在路由註冊中向您註冊的公用 IP 位址。 |在路由註冊中向您註冊的公用 IP 位址。 |
| **MD5 雜湊支援** |是 |yes |是 |

您可以選擇啟用一或多個路由網域，使之成為 ExpressRoute 線路的一部分。 如果想要將所有路由網域合併成單一路由網域，您可以選擇將所有路由網域放在相同的 VPN。 您也可以將他們放在不同的路由網域，類似上圖。 建議的設定是將私用對等直接連線到核心網路，而將公用和 Microsoft 對等連結連線到您的 DMZ。

如果您選擇使用全部三種對等互連工作階段，就必須擁有三個 BGP 工作階段配對 (每個對等互連類型一組)。 BGP 工作階段配對提供高可用性連結。 若您透過第 2 層連線提供者來連線，則要負責設定和管理路由。 如需深入瞭解，請檢閱有關設定 ExpressRoute 的 [工作流程](expressroute-workflows.md) 。

## <a name="expressroute-health"></a>ExpressRoute 健康情況
您可能會基於可用性、與 VNet 的連線能力及頻寬使用率，使用[網路效能監控](https://docs.microsoft.com/azure/networking/network-monitoring-overview) (NPM) 來監視 ExpressRoute 線路。

NPM 會監視 Azure 私用對等互連和 Microsoft 對等互連的健康情況。  如需詳細資訊，請參閱我們的[文章](https://azure.microsoft.com/blog/monitoring-of-azure-expressroute-in-preview/) \(英文\)。

## <a name="next-steps"></a>後續步驟
* 尋找服務提供者。 請參閱 [ExpressRoute 服務提供者和位置](expressroute-locations.md)。
* 請確定符合所有必要條件。 請參閱 [ExpressRoute 必要條件](expressroute-prerequisites.md)。
* 設定 ExpressRoute 連線。
  * [建立和管理 ExpressRoute 線路](expressroute-howto-circuit-portal-resource-manager.md)
  * [設定 ExpressRoute 線路的路由 (對等戶連)](expressroute-howto-routing-portal-resource-manager.md)

