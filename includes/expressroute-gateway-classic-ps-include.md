---
title: 包含檔案
description: 包含檔案
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 03/22/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 0bf55d2353d3524e65602c7e67b7adbf80432043
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30198072"
---
您必須先建立 VNET 和閘道器子網路，才能進行下列工作。

> [!NOTE]
> 這些範例不會套用到 S2S/ExpressRoute 並存組態。
> 如需有關在並存組態中使用閘道的詳細資訊，請參閱[設定並存連線](../articles/expressroute/expressroute-howto-coexist-classic.md#gw)

## <a name="add-a-gateway"></a>新增閘道

使用以下的命令建立閘道器。 所有的值請務必替換成您自己的值。

```powershell
New-AzureVNetGateway -VNetName "MyAzureVNET" -GatewayName "ERGateway" -GatewayType DynamicRouting -GatewaySKU  Standard
```

## <a name="verify-the-gateway-was-created"></a>確認已建立閘道

使用下列命令，確認已建立閘道。 這個命令也會擷取閘道器識別碼，您在其他作業會需要它。

```powershell
Get-AzureVNetGateway
```

## <a name="resize-a-gateway"></a>調整閘道器大小

有幾個 [閘道 SKU](../articles/expressroute/expressroute-about-virtual-network-gateways.md)。 您可以使用下列命令隨時變更閘道器 SKU。

> [!IMPORTANT]
> 此命令不適用於 UltraPerformance 閘道。 若要將您的閘道變更為 UltraPerformance 閘道，請先移除現有的 ExpressRoute 閘道，然後建立新的 UltraPerformance 閘道。 若要從 UltraPerformance 閘道降級您的閘道，請先移除 UltraPerformance 閘道，然後建立新的閘道。 
>
>

```powershell
Resize-AzureVNetGateway -GatewayId <Gateway ID> -GatewaySKU HighPerformance
```

## <a name="remove-a-gateway"></a>移除閘道器

使用以下的命令移除閘道器

```powershell
Remove-AzureVnetGateway -GatewayId <Gateway ID>
```