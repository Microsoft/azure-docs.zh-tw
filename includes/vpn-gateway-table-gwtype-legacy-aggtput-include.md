---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 6efec75884857d93f2e128104136bf59a1114594
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30197167"
---
下表依照閘道 SKU 顯示閘道類型和預估的彙總輸送量。 此表適用於 Resource Manager 與傳統部署模型。 

閘道 SKU 之間的價格並不相同。 如需詳細資訊，請參閱 [VPN 閘道價格](https://azure.microsoft.com/pricing/details/vpn-gateway)。

請注意下表未顯示 UltraPerformance 閘道 SKU。 如需 UltraPerformance SKU 的相關資訊，請參閱 [ExpressRoute](../articles/expressroute/expressroute-about-virtual-network-gateways.md) 文件。

|  | **VPN 閘道輸送量 (1)** | **VPN 閘道最大 IPsec 通道 (2)** | **ExpressRoute 閘道輸送量** | **VPN 閘道與 ExpressRoute 共存** |
| --- | --- | --- | --- | --- |
| **基本 SKU (3)(5)(6)** |100 Mbps |10 |500 Mbps (6) |否 |
| **標準 SKU (4)(5)** |100 Mbps |10 |1000 Mbps |yes |
| **高效能 SKU (4)** |200 Mbps |30 |2000 Mbps |yes |


(1) VPN 輸送量是根據相同 Azure 區域中 VNet 之間度量的概略估計。 這不是網際網路上跨單位連線的保證輸送量。 這是可能的最大輸送量測量。

(2) 通道的數目請參閱路由式 VPN。 原則式 VPN 只能支援一個站對站 VPN 通道。

(3) 基本 SKU 不支援 BGP。

(4) 此 SKU 不支援原則式 VPN。 只有基本 SKU 提供支援。

(5) 此 SKU 不支援主動-主動 S2S VPN 閘道連線。 只有高效能 SKU 支援主動-主動。

(6) 基本 SKU 已不再支援與 ExpressRoute 並用。
