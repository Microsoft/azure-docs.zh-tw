---
title: 透過 ExpressRoute 設定 BFD | Microsoft Docs
description: 本文件提供如何透過 ExpressRoute 線路的私用對等互連設定 BFD 的相關指示。
documentationcenter: na
services: expressroute
author: rambk
manager: tracsman
editor: ''
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 8/17/2018
ms.author: rambala
ms.openlocfilehash: 97ac49ff5cc50621fd902f77c2543a428c7e8bae
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49384747"
---
# <a name="configure-bfd-over-expressroute"></a>透過 ExpressRoute 設定 BFD

ExpressRoute 支援透過私用對等互連進行雙向轉送偵測 (BFD) 。 藉由透過 ExpressRoute 啟用 BFD，您可以加速 Microsoft Enterprise Edge (MSEE) 裝置與您終止 ExpressRoute 線路 (PE) 所在之路由器間的連結失敗偵測。 您可以透過客戶 Edge 路由裝置或協力廠商 Edge 路由裝置來終止 ExpressRoute (前提是您已使用受控的第 3 層連線服務)。 本文件將逐步引導您了解 BFD 的需求，以及如何透過 ExpressRoute 啟用 BFD。

## <a name="need-for-bfd"></a>需要 BFD

下圖顯示透過 ExpressRoute 路線啟用 BFD 的好處：[![1]][1]

您可以透過第 2 層連線或受控的第 3 層連線啟用 ExpressRoute 線路。 在任一種情況下，如果 ExpressRoute 連線路徑中有一或多個第 2 層裝置，則偵測路徑中是否有任何連結失敗的責任就會加諸於覆蓋的 BGP 之上。

在 MSEE 裝置上，BGP 存留期和保留時間通常會分別設定為 60 和 180 秒。 因此，在發生連結失敗之後，最多需要三分鐘的時間才能偵測到任何連結失敗，並將流量切換至其他連線。

您可以藉由在客戶 Edge 對等互連裝置上設定較低的 BGP 存留期和保留時間來控制 BGP 計時器。 如果這兩個對等互連裝置之間的 BGP 計時器不相符，則對等之間的 BGP 工作階段會使用較低的計時器值。 BGP 存留期最低可設為 3 秒，而保留時間可設為數十秒鐘。 不過，由於通訊協定是程序密集的，因此，設定 BGP 計時器的可能性較低。

在此案例中，BFD 可以提供協助。 BFD 會以次秒時間間隔提供低額外負荷的連結失敗偵測。 


## <a name="enabling-bfd"></a>啟用 BFD

BFD 預設會設定於 MSEE 上所有新建立的 ExpressRoute 私用對等互連介面上。 因此，若要啟用 BFD，您只需在 PE 上設定 BFD。 設定 BFD 是雙步驟程序：您需要在介面上設定 BFD，然後將它連結到 BGP 工作階段。

範例 PE (使用 Cisco IOS XE) 設定如下所示。 

    interface TenGigabitEthernet2/0/0.150
      description private peering to Azure
      encapsulation dot1Q 15 second-dot1q 150
      ip vrf forwarding 15
      ip address 192.168.15.17 255.255.255.252
      bfd interval 300 min_rx 300 multiplier 3


    router bgp 65020
      address-family ipv4 vrf 15
        network 10.1.15.0 mask 255.255.255.128
        neighbor 192.168.15.18 remote-as 12076
        neighbor 192.168.15.18 fall-over bfd
        neighbor 192.168.15.18 activate
        neighbor 192.168.15.18 soft-reconfiguration inbound
      exit-address-family

>[!NOTE]
>若要在已經存在的私用對等互連下方啟用 BFD，您需要重設該對等互連。 請參閱[重設 ExpressRoute 對等互連][ResetPeering]
>

## <a name="bfd-timer-negotiation"></a>BFD 計時器交涉

在 BFD 對等之間，這兩個對等中速度較慢者會決定傳輸速率。 MSEE BFD 傳輸/接收間隔會設定為 300 毫秒。 您可以藉由設定較高的值，強制讓這些間隔變得更長；但不要太短。

>[!NOTE]
>如果您已設定異地備援的 ExpressRoute 私用對等互連線路，或使用站對站 IPSec VPN 連線作為 ExpressRoute 私用對等互連的備份，透過私用對等互連啟用 BFD 有助於在發生 ExpressRoute 連線失敗之後更快速地進行容錯移轉。 
>

## <a name="next-steps"></a>後續步驟

如需詳細資訊或協助，請看看下列連結：

- [建立和修改 ExpressRoute 路線][CreateCircuit]
- [建立和修改 ExpressRoute 路線的路由][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-bfd/BFD_Need.png "BFD 會加速連結失敗推算時間"

<!--Link References-->
[CreateCircuit]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-circuit-portal-resource-manager 
[CreatePeering]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-routing-portal-resource-manager
[ResetPeering]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-reset-peering






