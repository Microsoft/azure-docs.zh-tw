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
ms.openlocfilehash: 7e33d4ed7100287ef6b22aa4c90fd52671363902
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31613614"
---
此工作的步驟會根據下列組態參考清單中的值來使用 VNet。 其他設定和名稱也會概述於這份清單中。 雖然我們會根據這份清單中的值加入變數，但不會在任何步驟中直接使用這份清單。 您可以複製清單以供參考，並使用您自己的值來取代其中的值。

**組態參考清單**

* 虛擬網路名稱 = "TestVNet"
* 虛擬網路位址空間 = 192.168.0.0/16
* 資源群組 = "TestRG"
* Subnet1 名稱 = "FrontEnd" 
* Subnet1 位址空間 = "192.168.1.0/24"
* 閘道器子網路名稱："GatewaySubnet"，您必須一律將閘道器子網路命名為 *GatewaySubnet*。
* 閘道子網路位址空間 = "192.168.200.0/26"
* 區域 = "East US"
* 閘道名稱 = "GW"
* 閘道 IP 名稱 = "GWIP"
* 閘道 IP 組態名稱 = "gwipconf"
* 類型 = "ExpressRoute"，ExpressRoute 組態需要有這個類型。
* 閘道公用 IP 名稱 = "gwpip"

## <a name="add-a-gateway"></a>新增閘道
1. 連接到您的 Azure 訂用帳戶。

  ```powershell 
  Connect-AzureRmAccount
  Get-AzureRmSubscription 
  Select-AzureRmSubscription -SubscriptionName "Name of subscription"
  ```
2. 宣告您在本練習中使用的變數。 請務必編輯範例，以反映您要使用的設定。

  ```powershell 
  $RG = "TestRG"
  $Location = "East US"
  $GWName = "GW"
  $GWIPName = "GWIP"
  $GWIPconfName = "gwipconf"
  $VNetName = "TestVNet"
  ```
3. 將虛擬網路物件儲存為變數。

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG
  ```
4. 將閘道子網路加入至虛擬網路。 閘道子網路必須命名為 "GatewaySubnet"。 您應建立 /27 或更大 (/26、/25 等) 的閘道子網路。

  ```powershell
  Add-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix 192.168.200.0/26
  ```
5. 設定組態。

  ```powershell
  $vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
  ```
6. 將閘道子網路儲存為變數。

  ```powershell
  $subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
  ```
7. 要求公用 IP 位址。 建立閘道之前需要有 IP 位址。 您無法指定想要使用的 IP 位址；該 IP 位址會以動態方式進行配置。 下一個組態章節將使用此 IP 位址。 AllocationMethod 必須是動態的。

  ```powershell
  $pip = New-AzureRmPublicIpAddress -Name $GWIPName  -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
  ```
8. 建立適用於閘道的組態。 閘道器組態定義要使用的子網路和公用 IP 位址。 在此步驟中，您要指定在建立閘道將使用的組態。 這個步驟不會實際建立閘道物件。 使用以下的範例來建立閘道器組態。

  ```powershell
  $ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
  ```
9. 建立閘道。 在這個步驟中， **-GatewayType** 特別重要。 您必須使用值 **ExpressRoute**。 執行這些 Cmdlet 之後，閘道需要花費 45 分鐘或更久的時間來建立。

  ```powershell
  New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG -Location $Location -IpConfigurations $ipconf -GatewayType Expressroute -GatewaySku Standard
  ```

## <a name="verify-the-gateway-was-created"></a>確認已建立閘道
使用下列命令，確認已建立閘道：

```powershell
Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG
```

## <a name="resize-a-gateway"></a>調整閘道器大小
有幾個 [閘道 SKU](../articles/expressroute/expressroute-about-virtual-network-gateways.md)。 您可以使用下列命令隨時變更閘道器 SKU。

> [!IMPORTANT]
> 此命令不適用於 UltraPerformance 閘道。 若要將您的閘道變更為 UltraPerformance 閘道，請先移除現有的 ExpressRoute 閘道，然後建立新的 UltraPerformance 閘道。 若要從 UltraPerformance 閘道降級您的閘道，請先移除 UltraPerformance 閘道，然後建立新的閘道。
> 
> 

```powershell
$gw = Get-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
Resize-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance
```

## <a name="remove-a-gateway"></a>移除閘道器
使用下列命令來移除閘道：

```powershell
Remove-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
```
