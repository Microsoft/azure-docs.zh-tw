---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/28/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 4ee182202cf1ecbbb0845541269f7241de26c170
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30326489"
---
### <a name="gwipnoconnection"></a>修改區域網路閘道 'GatewayIpAddress' - 沒有閘道連線

如果您想要連線的 VPN 裝置已變更其公用 IP 位址，您需要修改區域網路閘道，以反映該變更。 使用範例來修改沒有閘道連線的區域網路閘道。

修改此值時，您也可以同時修改位址首碼。 務必使用區域網路閘道的現有名稱，以便覆寫目前的設定。 如果您使用不同的名稱，您會建立新的區域網路閘道，而不是覆寫現有閘道。

```azurepowershell-interactive
New-AzureRmLocalNetworkGateway -Name Site1 `
-Location "East US" -AddressPrefix @('10.101.0.0/24','10.101.1.0/24') `
-GatewayIpAddress "5.4.3.2" -ResourceGroupName TestRG1
```

### <a name="gwipwithconnection"></a>修改區域網路閘道 'GatewayIpAddress' - 現有閘道連線

如果您想要連線的 VPN 裝置已變更其公用 IP 位址，您需要修改區域網路閘道，以反映該變更。 如果閘道連線已存在，您需要先移除連線。 連線移除之後，您可以修改閘道 IP 位址，然後重新建立新連線。 您也可以同時修改位址首碼。 這會導致您 VPN 連線的停機時間。 修改閘道 IP 位址時，您不需要刪除 VPN 閘道。 您只需要移除連線。
 

1. 移除連線。 您可以使用 'Get-AzureRmVirtualNetworkGatewayConnection' Cmdlet 來找到連線的名稱。

  ```azurepowershell-interactive
  Remove-AzureRmVirtualNetworkGatewayConnection -Name VNet1toSite1 `
  -ResourceGroupName TestRG1
  ```
2. 修改 'GatewayIpAddress' 值。 您也可以同時修改位址首碼。 請務必使用區域網路閘道的現有名稱來覆寫目前的設定。 否則，您會建立新的區域網路閘道，而不是覆寫現有閘道。

  ```azurepowershell-interactive
  New-AzureRmLocalNetworkGateway -Name Site1 `
  -Location "East US" -AddressPrefix @('10.101.0.0/24','10.101.1.0/24') `
  -GatewayIpAddress "104.40.81.124" -ResourceGroupName TestRG1
  ```
3. 建立連線。 在此範例中，我們會設定 IPsec 連線類型。 當您重新建立連線時，請使用針對設定指定的連線類型。 對於其他連線類型，請參閱 [PowerShell Cmdlet](https://msdn.microsoft.com/library/mt603611.aspx) 頁面。  若要取得 VirtualNetworkGateway 名稱，您可以執行 'Get-AzureRmVirtualNetworkGateway' Cmdlet。
   
    設定變數。

  ```azurepowershell-interactive
  $local = Get-AzureRMLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1 `
  $vnetgw = Get-AzureRmVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1
  ```
   
    建立連線。

  ```azurepowershell-interactive 
  New-AzureRmVirtualNetworkGatewayConnection -Name VNet1Site1 -ResourceGroupName TestRG1 `
  -Location "East US" `
  -VirtualNetworkGateway1 $vnetgw `
  -LocalNetworkGateway2 $local `
  -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
  ```