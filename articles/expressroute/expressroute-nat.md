---
title: ExpressRoute 線路的 NAT 需求 | Microsoft Docs
description: 此頁面提供為 ExpressRoute 線路設定和管理 NAT 的詳細需求。
documentationcenter: na
services: expressroute
author: cherylmc
manager: carmonm
editor: ''
ms.assetid: 867bf936-c851-485f-84c8-d8d6e33fee9f
ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/12/2017
ms.author: cherylmc
ms.openlocfilehash: 2a9903b0e3c04a7098f7a8e529801483b10af142
ms.sourcegitcommit: 5d772f6c5fd066b38396a7eb179751132c22b681
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2017
ms.locfileid: "23409260"
---
# <a name="expressroute-nat-requirements"></a>ExpressRoute NAT 需求
若要使用 ExpressRoute 連線到 Microsoft 雲端服務，您必須設定和管理 NAT。 有些連線提供者會以受控服務來支援設定和管理路由。 請洽詢您的連線服務提供者，以查看他們是否提供這類服務。 如果沒有，您必須遵守以下所述的需求。 

如需不同路由網域的概觀，請檢閱 [ExpressRoute 線路和路由網域](expressroute-circuit-peerings.md) 頁面。 為了符合 Azure 公用和 Microsoft 對等的公用 IP 位址需求，建議在您的網路與 Microsoft 之間設定 NAT。 本節提供您需要設定的 NAT 基礎結構的詳細描述。

## <a name="nat-requirements-for-microsoft-peering"></a>Microsoft 對等的 NAT 需求
Microsoft 對等路徑可讓您連接到不支援透過 Azure 公用對等路徑存取的 Microsoft 雲端服務。 服務清單包括 Office 365 服務，例如 Exchange Online、SharePoint Online、商務用 Skype 和 Dynamics 365。 Microsoft 預計在 Microsoft 對等上支援雙向連線能力。 以 Microsoft 雲端服務為目的地的流量，必須由 SNAT 轉譯成有效的公用 IPv4 位址，才能進入 Microsoft 網路。 從 Microsoft 雲端服務送到您的網路的流量，必須在網際網路邊緣經過 SNAT 轉譯，才可防止[非對稱式路由](expressroute-asymmetric-routing.md)。 下圖提供如何為 Microsoft 對等設定 NAT 的高階圖片。

![](./media/expressroute-nat/expressroute-nat-microsoft.png) 

### <a name="traffic-originating-from-your-network-destined-to-microsoft"></a>從您的網路出發到 Microsoft 的流量
* 您必須確定流量進入的 Microsoft 對等路徑具備有效的公用 IPv4 位址。 Microsoft 必須能夠向區域路由網際網路登錄 (RIR) 和網際網路路由登錄 (IRR) 驗證 IPv4 NAT 位址集區的擁有者。 將會根據所比對的 AS 編號和用於 NAT 的 IP 位址執行檢查。 如需路由登錄的詳細資訊，請參閱 [ExpressRoute 路由需求](expressroute-routing.md) 頁面。
* 用於 Azure 公用對等設定和其他 ExpressRoute 線路的 IP 位址，不得透過 BGP 工作階段向 Microsoft 公告。 透過此對等公告的 NAT IP 首碼長度沒有限制。
  
  > [!IMPORTANT]
  > 向 Microsoft 公告的 NAT IP 集區不得向網際網路公告。 這樣會中斷其他 Microsoft 服務的連線。
  > 
  > 

### <a name="traffic-originating-from-microsoft-destined-to-your-network"></a>從 Microsoft 出發到您的網路的流量
* 在某些情況下，需要由 Microsoft 對您網路中裝載的服務端點起始連線。 常見的例子就是從 Office 365 連接到您網路中裝載的 ADFS 伺服器。 在這種情況下，必須將您網路中適當的首碼透露給 Microsoft 對等。 
* 您必須在網際網路邊緣進行 Microsoft 流量的 SNAT 轉譯，才可防止[非對稱式路由](expressroute-asymmetric-routing.md)。 一律會透過 ExpressRoute 傳送具有一個目的地 IP 且符合透過 ExpressRoute 接收之路由的要求**和回覆**。 如果要求是透過網際網路收到，而回覆是透過 ExpressRoute 傳送，則存在非對稱式路由。 在網際網路邊緣進行 Microsoft 流量的 SNAT 轉譯，可強制回覆流量回到網際網路邊緣，進而解決問題。

![非對稱式路由與 ExpressRoute](./media/expressroute-asymmetric-routing/AsymmetricRouting2.png)

## <a name="nat-requirements-for-azure-public-peering"></a>Azure 公用對等的 NAT 需求
Azure 公用對等路徑可讓您連接到裝載於 Azure 中的所有服務的公用 IP 位址。 其中包括 [ExpessRoute 常見問題集](expressroute-faqs.md) 列出的服務，以及由 ISV 裝載於 Microsoft Azure 上的任何服務。 

> [!IMPORTANT]
> 連線到公用對等的 Microsoft Azure 服務時，一律是從您的網路出發到 Microsoft 網路。 因此，無法透過 ExpressRoute 從 Microsoft Azure 服務將工作階段起始到您的網路。 若已嘗試這麼做，傳送至這些通知 IP 的封包將使用網際網路，而不是 ExpressRoute。
> 

以公用對等的 Microsoft Azure 為目的地的流量，必須由 SNAT 轉譯成有效的公用 IPv4 位址，才能進入 Microsoft 網路。 下圖提供如何設定 NAT 以符合上述需求的高階圖片。

![](./media/expressroute-nat/expressroute-nat-azure-public.png) 

### <a name="nat-ip-pool-and-route-advertisements"></a>NAT IP 集區和路由通告
您必須確定流量進入的 Azure 公用對等路徑具備有效的公用 IPv4 位址。 Microsoft 必須能夠向區域路由網際網路登錄 (RIR) 和網際網路路由登錄 (IRR) 驗證 IPv4 NAT 位址集區的擁有權。 將會根據所比對的 AS 編號和用於 NAT 的 IP 位址執行檢查。 如需路由登錄的詳細資訊，請參閱 [ExpressRoute 路由需求](expressroute-routing.md) 頁面。

透過此對等公告的 NAT IP 首碼長度沒有限制。 您必須監視 NAT 集區，確保您未耗盡 NAT 工作階段。

> [!IMPORTANT]
> 向 Microsoft 公告的 NAT IP 集區不得向網際網路公告。 這樣會中斷其他 Microsoft 服務的連線。
> 
> 

## <a name="next-steps"></a>後續步驟
* 請參閱[路由](expressroute-routing.md)和 [QoS](expressroute-qos.md) 的需求。
* 如需工作流程資訊，請參閱 [ExpressRoute 線路佈建工作流程和線路狀態](expressroute-workflows.md)。
* 設定 ExpressRoute 連線。
  
  * [建立 ExpressRoute 線路](expressroute-howto-circuit-portal-resource-manager.md)
  * [設定路由](expressroute-howto-routing-portal-resource-manager.md)
  * [將 VNet 連結到 ExpressRoute 線路](expressroute-howto-linkvnet-portal-resource-manager.md)

