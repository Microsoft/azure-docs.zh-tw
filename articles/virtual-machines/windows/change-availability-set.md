---
title: 變更 VM 可用性設定組 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 和 Resource Manager 部署模型來變更虛擬機器的可用性設定組。
keywords: ''
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/30/2018
ms.author: cynthn
ms.openlocfilehash: 2ce73ca0a50a9884337258be0fab00528800db27
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34701814"
---
# <a name="change-the-availability-set-for-a-windows-vm"></a>變更 Windows VM 的可用性設定組
下列步驟說明如何使用 Azure PowerShell 來變更 VM 的可用性設定組。 只有在建立 VM 時，才能將 VM 新增到可用性設定組中。 若要變更可用性設定組，您必須將虛擬機器刪除，然後再重新建立。 

## <a name="change-the-availability-set"></a>變更可用性設定組 

下列指令碼提供一個範例，此範例會收集必要資訊、刪除原始 VM，然後在新的可用性設定組中重新建立 VM。

```powershell
# Set variables
    $resourceGroup = "myResourceGroup"
    $vmName = "myVM"
    $newAvailSetName = "myAvailabilitySet"

# Get VM Details
    $originalVM = get-azurermvm `
       -ResourceGroupName $resourceGroup `
       -Name $vmName

# Remove the original VM
    Remove-AzureRmVM -ResourceGroupName $resourceGroup -Name $vmName

# Create new availability set if it does not exist
    $availSet = Get-AzureRmAvailabilitySet `
       -ResourceGroupName $resourceGroup `
       -Name $newAvailSetName `
       -ErrorAction Ignore
    if (-Not $availSet) {
    $availSet = New-AzureRmAvailabilitySet `
       -Location $originalVM.Location `
       -Name $newAvailSetName `
       -ResourceGroupName $resourceGroup `
       -PlatformFaultDomainCount 2 `
       -PlatformUpdateDomainCount 2 `
       -Sku Aligned
    }

# Create the basic configuration for the replacement VM
    $newVM = New-AzureRmVMConfig `
       -VMName $originalVM.Name `
       -VMSize $originalVM.HardwareProfile.VmSize `
       -AvailabilitySetId $availSet.Id
  
    Set-AzureRmVMOSDisk `
       -VM $newVM -CreateOption Attach `
       -ManagedDiskId $originalVM.StorageProfile.OsDisk.ManagedDisk.Id `
       -Name $originalVM.StorageProfile.OsDisk.Name `
       -Windows

# Add Data Disks
    foreach ($disk in $originalVM.StorageProfile.DataDisks) { 
    Add-AzureRmVMDataDisk -VM $newVM `
       -Name $disk.Name `
       -ManagedDiskId $disk.ManagedDisk.Id `
       -Caching $disk.Caching `
       -Lun $disk.Lun `
       -DiskSizeInGB $disk.DiskSizeGB `
       -CreateOption Attach
    }
    
# Add NIC(s)
    foreach ($nic in $originalVM.NetworkProfile.NetworkInterfaces) {
        Add-AzureRmVMNetworkInterface `
           -VM $newVM `
           -Id $nic.Id
    }

# Recreate the VM
    New-AzureRmVM `
       -ResourceGroupName $resourceGroup `
       -Location $originalVM.Location `
       -VM $newVM `
       -DisableBginfoExtension
```

## <a name="next-steps"></a>後續步驟

透過新增額外 [資料磁碟](attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，將額外的存放裝置新增到您的 VM。

