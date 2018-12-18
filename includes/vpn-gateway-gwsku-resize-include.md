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
ms.openlocfilehash: c06e69dd9d1997500589659e936dc25ee01ed145
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "30196759"
---
對於目前的 SKU (VpnGw1、VpnGw2、VPNGW3)，若想要調整閘道 SKU 大小以升級到更強大的 SKU，可使用 `Resize-AzureRmVirtualNetworkGateway` PowerShell Cmdlet。 您也可以使用此 Cmdlet 將閘道 SKU 大小降級。 如果您使用基本閘道 SKU，[ 會改為使用這些指示](../articles/vpn-gateway/vpn-gateway-about-skus-legacy.md#resize)調整您的閘道大小。

下列 PowerShell 範例示範將閘道 SKU 的大小調整為 VpnGw2。

```powershell
$gw = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
Resize-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku VpnGw2
```

您也可以在 Azure 入口網站中調整閘道大小，請進入虛擬網路閘道的 [組態] 頁面，並從下拉式清單中選取不同的 SKU。