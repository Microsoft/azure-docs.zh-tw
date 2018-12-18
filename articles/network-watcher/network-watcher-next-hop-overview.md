---
title: Azure 網路監看員中的下一個躍點簡介 | Microsoft Docs
description: 本文提供網路監看員下一個躍點功能的概觀。
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: febf7bca-e0b7-41d5-838f-a5a40ebc5aac
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 28eacdce922e26d391cf34f78cb03ead9c6887a1
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391261"
---
# <a name="use-next-hop-to-diagnose-virtual-machine-routing-problems"></a>使用下一個躍點來診斷虛擬機器路由問題

來自虛擬機器 (VM) 的流量會根據與網路介面 (NIC) 相關聯的有效路由來傳送到目的地。 下一個躍點會從特定 VM 和 NIC 取得封包的下一個躍點類型和 IP 位址。 了解下一個躍點可協助您判斷流量是否會導向預定目的地，或流量是否不會傳送到其他地方。 若未正確設定路由，而讓流量導向內部部署位置或虛擬應用裝置，就可能會導致連線問題。 下一個躍點也會傳回與下一個躍點相關聯的路由表。 如果路由定義為使用者定義的路由，便會傳回該路由。 否則，下一個躍點會傳回**系統路由**。

![下一個躍點概觀](./media/network-watcher-next-hop-overview/figure1.png)

下一個躍點功能可能會傳回的下一個躍點如下所示：

* Internet
* VirtualAppliance
* VirtualNetworkGateway
* VirtualNetwork
* VirtualNetworkPeering
* VirtualNetworkServiceEndpoint 
* MicrosoftEdge
* None

若要深入了解每個下一個躍點類型，請參閱[路由概觀](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。

## <a name="next-steps"></a>後續步驟

若要了解如何使用下一個躍點來診斷 VM 網路路由問題，請參閱[診斷 VM 網路路由問題](diagnose-vm-network-routing-problem.md)。
