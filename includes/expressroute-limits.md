---
title: 包含檔案
description: 包含檔案
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 06/12/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c681e5f68299c03685dd7722f8dc671e49026f78
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43303850"
---
#### <a name="expressroute-limits"></a>ExpressRoute 限制
下列限制適用於每個訂用帳戶的 ExpressRoute 資源。

| 資源 | 預設限制 |
| --- | --- |
| 每個訂用帳戶的 ExpressRoute 電路 |10 |
| 每個訂用帳戶 (Azure Resource Manager) 每個區域的 ExpressRoute 電路 |10 |
| 具有 ExpressRoute Standard 的 Azure 私人對等互連的最大路由數目 |4,000 |
| 具有 ExpressRoute Premium 附加元件的 Azure 私人對等互連的最大路由數目 |10,000 |
| 具有 ExpressRoute Standard 的 Azure Microsoft 對等互連的最大路由數目 |200 |
| 具有 ExpressRoute Premium 附加元件的 Azure Microsoft 對等互連的最大路由數目 |200 |
| ExpressRoute 線路連結至相同虛擬網路中不同對等互連位置的最大數目 |4 |
| 每個 ExpressRoute 電路允許的虛擬網路連結數目 |請參閱下表 |

#### <a name="number-of-virtual-networks-per-expressroute-circuit"></a>每個 ExpressRoute 電路的虛擬網路數目
| **電路大小** | **Standard 的 VNet 連結數目** | **具有 Premium 附加元件的 VNet 連結數目** |
| --- | --- | --- |
| 50 Mbps |10 |20 |
| 100 Mbps |10 |25 |
| 200 Mbps |10 |25 |
| 500 Mbps |10 |40 |
| 1 Gbps |10 |50 |
| 2 Gbps |10 |60 |
| 5 Gbps |10 |75 |
| 10 Gbps |10 |100 |

