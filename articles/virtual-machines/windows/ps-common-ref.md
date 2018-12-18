---
title: Azure 虛擬機器的常用 PowerShell 命令 | Microsoft Docs
description: 可讓您在 Azure 中開始建立及管理 Windows VM 的常用 PowerShell 命令。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ba3839a2-f3d5-4e19-a5de-95bfb1c0e61e
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 06/01/2018
ms.author: cynthn
ms.openlocfilehash: ff861c21250a042191651ab4a4cffbf3928e4f26
ms.sourcegitcommit: c722760331294bc8532f8ddc01ed5aa8b9778dec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34738559"
---
# <a name="common-powershell-commands-for-creating-and-managing-azure-virtual-machines"></a>用於建立及管理 Azure 虛擬機器的常用 PowerShell 命令

本文涵蓋一些 Azure PowerShell 命令，您可用來建立及管理您的 Azure 訂用帳戶中的虛擬機器。  如需特定命令列參數和選項的詳細說明，您可以使用 **Get-Help** *命令*。

如需如何安裝最新版 Azure PowerShell、選取訂用帳戶，以及登入帳戶的相關資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

如果您要執行本文中的多個命令，下列變數可能會相當實用：

- $location - 虛擬機器的位置。 您可以使用 [Get-AzureRmLocation](https://docs.microsoft.com/powershell/module/azurerm.resources/get-azurermlocation) 來尋找適合您的[地理區域](https://azure.microsoft.com/regions/)。
- $myResourceGroup - 包含虛擬機器之資源群組的名稱。
- $myVM - 虛擬機器的名稱。

## <a name="create-a-vm---simplified"></a>建立 VM (簡化)

| Task | 命令 |
| ---- | ------- |
| 建立簡易 VM | [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) -Name $myVM <BR></BR><BR></BR> New-AzureRMVM 有一組「已簡化」的參數，其只需要單一名稱。 -Name 的值會用來作為所有建立新 VM 所需資源的名稱。 您可以指定多個，但這是唯一必要的值。|
| 從自訂映像建立 VM | New-AzureRmVm -ResourceGroupName $myResourceGroup -Name $myVM ImageName "myImage" -Location $location  <BR></BR><BR></BR>您必須已建立自己的[受控映像](capture-image-resource.md)。 您可以使用映像來建立多個相同的 VM。 |



## <a name="create-a-vm-configuration"></a>建立 VM 組態

| Task | 命令 |
| ---- | ------- |
| 建立 VM 組態 |$vm = [New-AzureRmVMConfig](https://docs.microsoft.com/powershell/module/azurerm.compute/new-azurermvmconfig) -VMName $myVM -VMSize "Standard_D1_v1"<BR></BR><BR></BR>VM 組態用來定義或更新 VM 的設定。 系統會使用 VM 的名稱及其 [大小](sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)初始化組態。 |
| 新增組態設定 |$vm = [Set-AzureRmVMOperatingSystem](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmoperatingsystem) -VM $vm -Windows -ComputerName $myVM -Credential $cred -ProvisionVMAgent -EnableAutoUpdate<BR></BR><BR></BR>包含[認證](https://technet.microsoft.com/library/hh849815.aspx)的作業系統設定會新增至您先前使用 New-AzureRmVMConfig 建立的組態物件。 |
| 新增網路介面 |$vm = [Add-AzureRmVMNetworkInterface](https://docs.microsoft.com/powershell/resourcemanager/azurerm.compute/v2.5.0/Add-AzureRmVMNetworkInterface) -VM $vm -Id $nic.Id<BR></BR><BR></BR>VM 必須有[網路介面](../virtual-machines-windows-ps-create.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)才可在虛擬網路中進行通訊。 您也可以使用 [Get-AzureRmNetworkInterface](https://docs.microsoft.com/powershell/module/azurerm.compute/add-azurermvmnetworkinterface) 擷取現有的網路介面物件。 |
| 指定平台映像 |$vm = [Set-AzureRmVMSourceImage](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmsourceimage) -VM $vm -PublisherName "publisher_name" -Offer "publisher_offer" -Skus "product_sku" -Version "latest"<BR></BR><BR></BR>[映像資訊](cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 會新增至您先前使用 New-AzureRmVMConfig 建立的組態物件。 只有在您設定作業系統磁碟使用平台映像時，才會使用此命令傳回的物件。 |
| 建立 VM |[New-AzureRmVM]() -ResourceGroupName $myResourceGroup -Location $location -VM $vm<BR></BR><BR></BR>所有資源都會在[資源群組](../../azure-resource-manager/powershell-azure-resource-manager.md)中建立。 執行此命令之前，請執行 New-AzureRmVMConfig、Set-AzureRmVMOperatingSystem、Set-AzureRmVMSourceImage、Add-AzureRmVMNetworkInterface 和 Set-AzureRmVMOSDisk。 |
| 更新 VM |[Update-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/update-azurermvm) -ResourceGroupName $myResourceGroup -VM $vm<BR></BR><BR></BR>使用 Get-AzureRmVM 取得目前的 VM 組態，變更 VM 物件上的組態設定，然後執行此命令。 |

## <a name="get-information-about-vms"></a>取得 VM 的相關資訊

| Task | 命令 |
| ---- | ------- |
| 列出訂用帳戶中的 VM |[Get-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvm) |
| 列出資源群組中的 VM |Get-AzureRmVM -ResourceGroupName $myResourceGroup<BR></BR><BR></BR>若要取得您的訂用帳戶中的資源群組清單，請使用 [Get-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/get-azurermresourcegroup)初始化組態。 |
| 取得 VM 的相關資訊 |Get-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM |

## <a name="manage-vms"></a>管理 VM
| Task | 命令 |
| --- | --- |
| 啟動 VM |[Start-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/start-azurermvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| 停止 VM |[Stop-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/stop-azurermvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| 重新啟動執行中的 VM |[Restart-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/restart-azurermvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| 刪除 VM |[Remove-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/remove-azurermvm) -ResourceGroupName $myResourceGroup -Name $myVM |


## <a name="next-steps"></a>後續步驟
* 請參閱 [使用 Resource Manager 和 PowerShell 建立 Windows VM](../virtual-machines-windows-ps-create.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中建立虛擬機器的基本步驟。

