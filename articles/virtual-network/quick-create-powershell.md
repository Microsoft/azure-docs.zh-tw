---
title: 建立虛擬網路 - 快速入門 - Azure PowerShell | Microsoft Docs
description: 在此快速入門中，您將了解如何使用 Azure 入口網站建立虛擬網路。 虛擬網路可讓 Azure 資源 (例如虛擬機器) 彼此及與網際網路進行私密通訊。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to create a virtual network so that virtual machines can communicate with privately with each other and with the internet.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: quickstart
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/09/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: b8b67b235b54fb5bde738ed5cc1605e08d182a69
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38688080"
---
# <a name="quickstart-create-a-virtual-network-using-powershell"></a>快速入門：使用 PowerShell 建立虛擬網路

虛擬網路可讓 Azure 資源 (例如虛擬機器 (VM)) 彼此及與網際網路進行私密通訊。 在此快速入門中，您將了解如何建立虛擬網路。 建立虛擬網路之後，您需將兩個 VM 部署至虛擬網路中。 接著，您需從網際網路連線至其中一個 VM，然後在兩個 VM 之間進行私密通訊。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在進行此快速入門時，您必須使用 AzureRM PowerShell 模組 5.4.1 版或更新版本。 若要尋找已安裝的版本，請執行 ` Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-virtual-network"></a>建立虛擬網路

建立虛擬網路之前，您必須先建立資源群組來包含虛擬網路。 使用 [New-AzureRmResourceGroup](/powershell/module/AzureRM.Resources/New-AzureRmResourceGroup)建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroup -Location EastUS
```

使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立虛擬網路。 以下範例會在 *EastUS* 位置建立名為 *myVirtualNetwork* 的預設虛擬網路：

```azurepowershell-interactive
$virtualNetwork = New-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Azure 資源會部署至虛擬網路內的子網路，因此您必須建立子網路。 請使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 來建立子網路設定。 

```azurepowershell-interactive
$subnetConfig = Add-AzureRmVirtualNetworkSubnetConfig `
  -Name default `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork
```

使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/Set-AzureRmVirtualNetwork) 將子網路設定寫入至虛擬網路，這會在虛擬網路內建立子網路：

```azurepowershell-interactive
$virtualNetwork | Set-AzureRmVirtualNetwork
```

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路內建立兩個 VM：

### <a name="create-the-first-vm"></a>建立第一個 VM

使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 來建立 VM。 執行接下來的命令時，系統會提示您輸入認證。 您輸入的值會設定為 VM 的使用者名稱和密碼。 `-AsJob` 選項會在背景建立 VM，以便您繼續進行下一步。

```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "default" `
    -Name "myVm1" `
    -AsJob
```

將會傳回類似以下範例輸出的輸出，而 Azure 則會開始在背景建立 VM。

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command                  
--     ----            -------------   -----         -----------     --------             -------                  
1      Long Running... AzureLongRun... Running       True            localhost            New-AzureRmVM     
```

### <a name="create-the-second-vm"></a>建立第二個 VM 

輸入下列命令：

```azurepowershell-interactive
New-AzureRmVm `
  -ResourceGroupName "myResourceGroup" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "default" `
  -Name "myVm2"
```

建立 VM 需要幾分鐘的時間。 在先前的命令完成並將輸出傳回給 PowerShell 之前，請勿繼續下一步。

## <a name="connect-to-a-vm-from-the-internet"></a>從網際網路連線至 VM

請使用 [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 來傳回 VM 的公用 IP 位址。 以下範例會傳回 *myVm1* VM 的公用 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress `
  -Name myVm1 `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

使用前一個命令傳回的公用 IP 位址來取代下列命令中的 `<publicIpAddress>`，然後輸入下列命令： 

```
mstsc /v:<publicIpAddress>
```

將會建立一個「遠端桌面通訊協定」(.rdp) 檔案，並下載至您的電腦。 開啟所下載的 rdp 檔案。 如果出現提示，請選取 [連接]。 輸入您在建立 VM 時指定的使用者名稱和密碼。 您可能需要選取 [其他選擇]，然後選取 [使用不同的帳戶]，以指定您在建立 VM 時輸入的認證。 選取 [確定] 。 您可能會在登入過程中收到憑證警告。 如果您收到警告，請選取 [是] 或 [繼續] 以繼續進行連線。

## <a name="communicate-between-vms"></a>虛擬機器之間的通訊

從 *myVm1* VM 上的 PowerShell，輸入 `ping myvm2`。 Ping 會失敗，因為 Ping 使用網際網路控制訊息通訊協定 (ICMP)，而 Windows 防火牆依預設並不允許 ICMP 通過。

若要允許 *myVm2* 在稍後的步驟中對 *myVm1* 執行 Ping，請從 PowerShell 輸入下列命令，這會允許連入的 ICMP 通過 Windows 防火牆：

```powershell
New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
```

關閉對 *myVm1* 的遠端桌面連線。 

再次完成[從網際網路連線至 VM](#connect-to-a-vm-from-the-internet) 中的步驟，但連線至 *myVm2*。 

從 *myVm2* VM 上的命令提示字元中，輸入 `ping myvm1`。

您會從 *myVm1* 收到回覆，因為您在先前的步驟中允許 ICMP 通過 *myVm1* VM 上的 Windows 防火牆。

關閉對 *myVm2* 的遠端桌面連線。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurepowershell-interactive 
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在此快速入門中，您建立了一個預設的虛擬網路和兩個 VM。 您從網際網路連線至其中一個 VM，然後在該 VM 與另一個 VM 進行私密通訊。 若要深入了解虛擬網路設定，請參閱[管理虛擬網路](manage-virtual-network.md)。 

Azure 預設會允許虛擬機器之間進行不受限制的私密通訊，但只允許從網際網路到 Windows VM 的輸入遠端桌面連線。 若要了解如何允許或限制進出 VM 的不同網路通訊類型，請繼續進行[篩選網路流量](tutorial-filter-network-traffic.md)教學課程。
