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
ms.openlocfilehash: 605533f25b36a92a660301d28aa63cb2ecdd44f4
ms.sourcegitcommit: 11321f26df5fb047dac5d15e0435fce6c4fde663
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37909995"
---
建立虛擬網路閘道時，您必須指定想要使用的閘道 SKU。 根據工作負載、輸送量、功能和 SLA 的類型，選取符合您需求的 SKU。

###  <a name="benchmark"></a>依通道、連線和輸送量區分的閘道 SKU

[!INCLUDE [Aggregated throughput by SKU](./vpn-gateway-table-gwtype-aggtput-include.md)]

[!INCLUDE [classic SKU](./vpn-gateway-classic-sku-support-include.md)]

###  <a name="feature"></a>依功能集區分的閘道 SKU

新式 VPN 閘道 SKU 可簡化閘道上提供的功能集：

| **SKU**| **特性**|
| ---    | ---         |
|**基本** (\*\*)   | **路由式 VPN**：10 個 P2S 通道；沒有適用於 P2S 的 RADIUS 驗證；沒有適用於 P2S 的 IKEv2<br>**原則式 VPN** (IKEv1)：1 個通道；沒有 P2S|
| **VpnGw1、VpnGw2 和 VpnGw3** | **路由式 VPN**：最多 30 個通道 ( * )，P2S、BGP、主動-主動、自訂 IPsec/IKE 原則、ExpressRoute/VPN 共存 |
|        |             |

( * ) 您可以設定 "PolicyBasedTrafficSelectors"，將以路由為基礎的 VPN 閘道 (VpnGw1、VpnGw2、VpnGw3) 連線至多個內部部署以原則為基礎的防火牆裝置。 如需詳細資訊，請參閱[使用 PowerShell 將 VPN 閘道連線至多個內部部署原則式 VPN 裝置](../articles/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps.md)。

(\*\*) 基本 SKU 被視為舊版 SKU。 基本 SKU 有某些功能限制。 您無法調整使用基本 SKU 的閘道大小來成為新的閘道 SKU，您必須改以變更為新的 SKU，而需要刪除並重新建立 VPN 閘道。

###  <a name="workloads"></a>閘道 SKU - 生產與開發測試工作負載

由於 SLA 和功能集的差異，我們建議將下列 SKU 用於產生與開發測試：

| **工作負載**                       | **SKU**               |
| ---                                | ---                    |
| **生產、重要工作負載** | VpnGw1、VpnGw2、VpnGw3 |
| **開發測試或概念證明**   | 基本 (\*\*)                 |
|                                    |                        |

(\*\*) 基本 SKU 被視為舊版 SKU，而且有功能限制。 請先確認其有支援您需要的功能，再使用基本 SKU。

如果您使用舊式 SKU (舊版)，生產 SKU 建議為標準和高效能。 如需舊式 SKU 的資訊和指示，請參閱[閘道 SKU (舊版)](../articles/vpn-gateway/vpn-gateway-about-skus-legacy.md)。
