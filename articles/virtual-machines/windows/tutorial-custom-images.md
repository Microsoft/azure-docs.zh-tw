---
title: 教學課程 - 使用 Azure PowerShell 建立自訂的 VM 映像 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure PowerShell 在 Azure 中建立自訂虛擬機器映像
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 03/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 930f5dc0369c998039b9cc0c089f60de17993be0
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37435148"
---
# <a name="tutorial-create-a-custom-image-of-an-azure-vm-with-azure-powershell"></a>教學課程：使用 Azure PowerShell 建立 Azure VM 的自訂映像

自訂映像類似 Marketplace 映像，但您要自行建立它們。 自訂映像可用於啟動程序設定，例如，預先載入應用程式、應用程式設定和其他 OS 設定。 在本教學課程中，您將建立自己的 Azure 虛擬機器自訂映像。 您會了解如何：

> [!div class="checklist"]
> * 執行 sysprep 及一般化 VM
> * 建立自訂映像
> * 從自訂映像建立 VM
> * 列出訂用帳戶中的所有映像
> * 删除映像

## <a name="before-you-begin"></a>開始之前

下列步驟將詳細說明如何將現有 VM 轉換成可重複使用的自訂映像，以便讓您用來建立新的 VM 執行個體。

若要完成本教學課程中的範例，您目前必須具有虛擬機器。 如有需要，這個[指令碼範例](../scripts/virtual-machines-windows-powershell-sample-create-vm.md)可以為您建立一部虛擬機器。 逐步完成教學課程之後，請視需要取代資源群組和 VM 名稱。

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，您必須使用 AzureRM 模組版本 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

## <a name="prepare-vm"></a>準備 VM

若要建立虛擬機器的映像，您需要藉由一般化 VM、解除配置，然後在 Azure 中將來源 VM 標示為一般化，來準備 VM。

### <a name="generalize-the-windows-vm-using-sysprep"></a>使用 Sysprep 將 Windows VM 一般化

Sysprep 會移除您的所有個人帳戶資訊以及其他項目，並準備電腦以做為映像。 如需 Sysprep 的詳細資訊，請參閱 [如何使用 Sysprep：簡介](http://technet.microsoft.com/library/bb457073.aspx)。


1. 連接至虛擬機器。
2. 以系統管理員身分開啟 [命令提示字元] 視窗。 切換至 %windir%\system32\sysprep 目錄，然後執行 sysprep.exe。
3. 在 [系統準備工具] 對話方塊中，選取 [進入系統全新體驗 (OOBE)]，並確認已勾選 [一般化] 核取方塊。
4. 在 [關機選項] 中選取 [關機]，然後按一下 [確定]。
5. Sysprep 完成時，會關閉虛擬機器。 **不要重新啟動 VM**。

### <a name="deallocate-and-mark-the-vm-as-generalized"></a>解除配置並將 VM 標示為一般化

若要建立映像，必須將 VM 解除配置並在 Azure 中標示為一般化。

使用 [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm) 將 VM 解除配置。

```azurepowershell-interactive
Stop-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM -Force
```

使用 [Set-AzureRmVm](/powershell/module/azurerm.compute/set-azurermvm)將虛擬機器的狀態設定為 `-Generalized`。 
   
```azurepowershell-interactive
Set-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM -Generalized
```


## <a name="create-the-image"></a>建立映像

現在您可以使用 [New-AzureRmImageConfig](/powershell/module/azurerm.compute/new-azurermimageconfig) 和 [New-AzureRmImage](/powershell/module/azurerm.compute/new-azurermimage)建立 VM 的映象。 下列範例會從名為 myVM 的 VM 建立名為 myImage 的映像。

取得虛擬機器。 

```azurepowershell-interactive
$vm = Get-AzureRmVM -Name myVM -ResourceGroupName myResourceGroup
```

建立映像組態。

```azurepowershell-interactive
$image = New-AzureRmImageConfig -Location EastUS -SourceVirtualMachineId $vm.ID 
```

建立映像。

```azurepowershell-interactive
New-AzureRmImage -Image $image -ImageName myImage -ResourceGroupName myResourceGroup
``` 

 
## <a name="create-vms-from-the-image"></a>從映像建立 VM

現在您已有映像，您可以從映像建立一個或多個新的 VM。 從自訂映像建立 VM 類似於使用 Marketplace 映像建立 VM。 使用 Marketplace 映像時，您必須提供關於映像、映像提供者、供應項目、SKU 和版本的資訊。 使用針對 [New-AzureRMVM]() Cmdlet 簡化的參數集時，只要自訂映像位在同一個資源群組，您就只需要提供自訂映像的名稱。 

該範例會根據 *myResourceGroup* 中的 *myImage* 建立名為 *myVMfromImage* 的 VM。


```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVMfromImage" `
    -ImageName "myImage" `
    -Location "East US" `
    -VirtualNetworkName "myImageVnet" `
    -SubnetName "myImageSubnet" `
    -SecurityGroupName "myImageNSG" `
    -PublicIpAddressName "myImagePIP" `
    -OpenPorts 3389
```

## <a name="image-management"></a>映像管理 

以下範例是一些常見的映像管理作業，以及如何使用 PowerShell 完成這些作業。

依名稱列出所有映像。

```azurepowershell-interactive
$images = Get-AzureRMResource -ResourceType Microsoft.Compute/images 
$images.name
```

删除映像。 此範例會刪除 myResourceGroup 中名為 myOldImage 的映像。

```azurepowershell-interactive
Remove-AzureRmImage `
    -ImageName myOldImage `
    -ResourceGroupName myResourceGroup
```

## <a name="next-steps"></a>後續步驟

您在本教學課程中建立了自訂 VM 映像。 您已了解如何︰

> [!div class="checklist"]
> * 執行 sysprep 及一般化 VM
> * 建立自訂映像
> * 從自訂映像建立 VM
> * 列出訂用帳戶中的所有映像
> * 删除映像

請前進到下一個教學課程，以了解如何使虛擬機器具備高可用性。

> [!div class="nextstepaction"]
> [建立高可用性 VM](tutorial-availability-sets.md)



