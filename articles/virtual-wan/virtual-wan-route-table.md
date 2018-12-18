---
title: 建立 Azure 虛擬 WAN 虛擬中樞路由表以引導到 NVA |Microsoft Docs
description: 虛擬 WAN 虛擬中樞路由表可將流量引導到網路虛擬設備。
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to work with routing tables for NVA.
ms.openlocfilehash: 56321c7c2239c0f1f0e73126d90a521e3cafb853
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984185"
---
# <a name="create-a-virtual-hub-route-table-to-steer-traffic-to-a-network-virtual-appliance"></a>建立虛擬中樞路由表以將流量引導到網路虛擬設備

這篇文章會示範如何將流量從虛擬中樞引導到網路虛擬設備。 

![虛擬 WAN 的圖表](./media/virtual-wan-route-table/vwanroute.png)

在本文中，您將了解如何：

* 建立 WAN
* 建立中樞
* 建立中樞虛擬網路連線
* 建立中樞路由
* 建立路由表
* 套用路由表

## <a name="before-you-begin"></a>開始之前

請確認您已符合下列條件：

1. 您有網路虛擬設備 (NVA)，它是您選用的第三方軟體，通常從 Azure Marketplace (連結) 佈建到虛擬網路。
2. 您有一個指派給 NVA 網路介面的私人 IP。 
3. NVA 無法部署在虛擬中樞。 它必須部署在不同的 VNet。 在本文中，VNet 稱為 'DMZ VNet'。
4. 'DMZ VNet' 可以有一或多個虛擬網路與其連接。 在本文中，此 VNet 是稱為「間接輪幅 VNet」。 這些 Vnet 可以使用 VNet 對等互連連接到 DMZ VNet。
5. 請確認您已建立 2 個 Vnet。 它們會當作輪輻 Vnet。 在本文中，VNet 輪輻位址空間是 10.0.2.0/24 和 10.0.3.0/24。 如果您需要如何建立 VNet 的詳細資訊，請參閱[使用 PowerShell 建立虛擬網路](../virtual-network/quick-create-powershell.md)。
6. 請確定任何 Vnet 中都沒有虛擬網路閘道。

## <a name="signin"></a>1.登入

務必安裝最新版的 Resource Manager PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 這一點很重要，因為較早版本的 Cmdlet 未包含您在此練習中所需的現行值。

1. 以提高的權限開啟 PowerShell 主控台並登入您的 Azure 帳戶。 此 Cmdlet 會提示您提供登入認證。 登入之後，它會下載您的帳戶設定以供 Azure PowerShell 使用。

  ```powershell
  Connect-AzureRmAccount
  ```
2. 取得您的 Azure 訂用帳戶清單。

  ```powershell
  Get-AzureRmSubscription
  ```
3. 指定您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Name of subscription"
  ```

## <a name="rg"></a>2.建立資源

1. 建立資源群組。

  ```powershell
  New-AzureRmResourceGroup -Location "West US" -Name "testRG"
  ```
2. 建立虛擬 WAN。

  ```powershell
  $virtualWan = New-AzureRmVirtualWan -ResourceGroupName "testRG" -Name "myVirtualWAN" -Location "West US"
  ```
3. 建立虛擬中樞。

  ```powershell
  New-AzureRmVirtualHub -VirtualWan $virtualWan -ResourceGroupName "testRG" -Name "westushub" -AddressPrefix "10.0.1.0/24"
  ```

## <a name="connections"></a>3.建立連線

從「間接輪幅 VNet」與 DMZ VNet 到虛擬中樞之間建立中樞虛擬網路連線。

  ```powershell
  $remoteVirtualNetwork1= Get-AzureRmVirtualNetwork -Name “indirectspoke1” -ResourceGroupName “testRG”
  $remoteVirtualNetwork2= Get-AzureRmVirtualNetwork -Name “indirectspoke2” -ResourceGroupName “testRG”
  $remoteVirtualNetwork3= Get-AzureRmVirtualNetwork -Name “dmzvnet” -ResourceGroupName “testRG”

  New-AzureRmVirtualHubVnetConnection -ResourceGroupName “testRG” -VirtualHubName “westushub” -Name  “testvnetconnection1” -RemoteVirtualNetwork $remoteVirtualNetwork1
  New-AzureRmVirtualHubVnetConnection -ResourceGroupName “testRG” -VirtualHubName “westushub” -Name  “testvnetconnection2” -RemoteVirtualNetwork $remoteVirtualNetwork2
  New-AzureRmVirtualHubVnetConnection -ResourceGroupName “testRG” -VirtualHubName “westushub” -Name  “testvnetconnection3” -RemoteVirtualNetwork $remoteVirtualNetwork3
  ```

## <a name="route"></a>4.建立虛擬中樞路由

在本文中，「間接輪幅 VNet」位址空間是 10.0.2.0/24 和 10.0.3.0/24，而 DMZ NVA 網路介面私人 IP 位址是 10.0.4.5。

```powershell
$route1 = New-AzureRmVirtualHubRoute -AddressPrefix @("10.0.2.0/24", "10.0.3.0/24") -NextHopIpAddress "10.0.4.5"
```

## <a name="applyroute"></a>5.建立虛擬中樞路由表

建立虛擬中樞路由表，然後對它套用已建立的路由。
 
```powershell
$routeTable = New-AzureRmVirtualHubRouteTable -Route @($route1)
```

## <a name="commit"></a>6.認可變更

將變更認可到虛擬中樞。

```powershell
Set-AzureRmVirtualHub -VirtualWanId $virtualWan.Id -ResourceGroupName "testRG" -Name "westushub” -RouteTable $routeTable
```

## <a name="cleanup"></a>清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源。 將 "myResourceGroup" 取代為您的資源群組名稱，然後執行下列 PowerShell 命令：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

若要深入了解虛擬 WAN，請參閱[虛擬 WAN 概觀](virtual-wan-about.md)頁面。