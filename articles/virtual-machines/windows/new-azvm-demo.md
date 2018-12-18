---
title: 使用 Azure Cloud Shell 中簡化的 New-AzureRMVM Cmdlet 建立 Windows VM | Microsoft Docs
description: 快速了解如何在 Azure Cloud Shell 中使用簡化的 New-AzureRMVM Cmdlet 來建立 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 12/12/2017
ms.author: cynthn
ROBOTS: NOINDEX
ms.openlocfilehash: defc871caed429eeda1f8672323b48a9c0007c8e
ms.sourcegitcommit: e45b2aa85063d33853560ec4bc867f230c1c18ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43371732"
---
# <a name="create-a-windows-virtual-machine-with-the-simplified-new-azurermvm-cmdlet-in-cloud-shell"></a>在 Cloud Shell 中使用簡化的 New-AzureRMVM Cmdlet 來建立 Windows 虛擬機器 

[New-AzureRMVM](https://docs.microsoft.com/powershell/module/azurerm.compute/new-azurermvm?view=azurermps-6.8.1) Cmdlet 已新增一組簡化的參數，用於使用 PowerShell 建立新的 VM。 本主題說明如何在 Azure Cloud Shell 中使用 PowerShell 搭配預先安裝的最新版本 New-AzureVM Cmdlet 來建立新的 VM。 我們將使用會使用智慧型預設值來自動建立所有必要資源的簡化參數集。 

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。


[!INCLUDE [cloud-shell-powershell](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 5.1.1 或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-the-vm"></a>建立 VM

您可以使用 [New-AzureRMVM](https://docs.microsoft.com/powershell/module/azurerm.compute/new-azurermvm?view=azurermps-6.8.1) Cmdlet 來建立虛擬機器，其中包括使用來自於 Azure Marketplace 的 Windows Server 2016 Datacenter 映像等智慧型預設值。 您可以使用 New-AzureRMVM 僅搭配 **-Name** 參數，它將會使用該值做為所有資源名稱。 在此範例中，我們將 **-Name** 參數設為 *myVM*。 

請確定已在 Cloud Shell 中選取了 **PowerShell**，然後鍵入：

```azurepowershell-interactive
New-AzureRMVm -Name myVM
```

系統會要求您建立 VM 的使用者名稱和密碼，稍後在本主題中當您連線到 VM 時會用到。 密碼長度必須是 12-123 個字元，且符合下列四個複雜性需求的其中三項：1 個小寫字元、1 個大寫字元、1 個數字和 1 個特殊字元。

只要花一分鐘即可建立虛擬機器和相關聯的資源。 完成時，您可以看到所有使用 [Get-AzureRmResource](/powershell/module/azurerm.resources/get-azurermresource) Cmdlet 建立的資源。

```azurepowershell-interactive
Get-AzureRmResource `
    -ResourceGroupName myVMResourceGroup | Format-Table Name
```

## <a name="connect-to-the-vm"></a>連接至 VM

完成部署之後，建立與虛擬機器的遠端桌面連線。

使用 [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 命令，以傳回虛擬機器的公用 IP 位址。 記下此 IP 位址。

```azurepowershell-interactive
Get-AzureRmPublicIpAddress `
    -ResourceGroupName myVMResourceGroup | Select IpAddress
```

在您的本機電腦上開啟命令提示字元，並使用 **mstsc** 命令以啟動遠端桌面工作階段與新的虛擬機器。 以虛擬機器的 IP 位址取代 &lt;publicIPAddress&gt;。 出現提示時，請輸入建立 VM 時指定的使用者名稱和密碼。

```
mstsc /v:<publicIpAddress>
```
## <a name="specify-different-resource-names"></a>指定不同的資源名稱

您也可以為資源提供更具描述性的名稱，並且仍自動建立。 以下是我們為新 VM 命名多個資源的範例，包括新的資源群組。

```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 3389
```

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組、VM 和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myVM
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本主題中，您已使用 New-AzVM 部署了簡單的虛擬機器，並透過 RDP 連線到該虛擬機器。 若要深入了解 Azure 虛擬機器，請繼續 Windows VM 的教學課程。

> [!div class="nextstepaction"]
> [Azure Windows 虛擬機器教學課程](./tutorial-manage-vm.md)
