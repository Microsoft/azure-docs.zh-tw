---
title: 變更 Vm 可用性設定組
description: 瞭解如何使用 Azure PowerShell 來變更虛擬機器的可用性設定組。
ms.service: virtual-machines
author: cynthn
ms.topic: how-to
ms.date: 01/31/2020
ms.author: cynthn
ms.openlocfilehash: 54f59a052132826897cfbc8dda59bc73fb6ad8d9
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98200561"
---
# <a name="change-the-availability-set-for-a-vm"></a>變更 VM 的可用性設定組
下列步驟說明如何使用 Azure PowerShell 來變更 VM 的可用性設定組。 只有在建立 VM 時，才能將 VM 新增到可用性設定組中。 若要變更可用性設定組，您必須將虛擬機器刪除，然後再重新建立。 

本文適用于 Linux 和 Windows Vm。

本文最後一次測試是在 2019 年 2 月 12 日，使用 [Azure Cloud Shell](https://shell.azure.com/powershell) 和 [Az PowerShell 模組](/powershell/azure/install-az-ps) 1.2.0 版進行的。

此範例不會檢查 VM 是否已連接到負載平衡器。 如果您的 VM 已連結至負載平衡器，您將需要更新腳本來處理該案例。 


## <a name="change-the-availability-set"></a>變更可用性設定組 

下列指令碼提供一個範例，此範例會收集必要資訊、刪除原始 VM，然後在新的可用性設定組中重新建立 VM。

```powershell
# Set variables
    $resourceGroup = "myResourceGroup"
    $vmName = "myVM"
    $newAvailSetName = "myAvailabilitySet"

# Get the details of the VM to be moved to the Availability Set
    $originalVM = Get-AzVM `
       -ResourceGroupName $resourceGroup `
       -Name $vmName

# Create new availability set if it does not exist
    $availSet = Get-AzAvailabilitySet `
       -ResourceGroupName $resourceGroup `
       -Name $newAvailSetName `
       -ErrorAction Ignore
    if (-Not $availSet) {
    $availSet = New-AzAvailabilitySet `
       -Location $originalVM.Location `
       -Name $newAvailSetName `
       -ResourceGroupName $resourceGroup `
       -PlatformFaultDomainCount 2 `
       -PlatformUpdateDomainCount 2 `
       -Sku Aligned
    }
    
# Remove the original VM
    Remove-AzVM -ResourceGroupName $resourceGroup -Name $vmName    

# Create the basic configuration for the replacement VM. 
    $newVM = New-AzVMConfig `
       -VMName $originalVM.Name `
       -VMSize $originalVM.HardwareProfile.VmSize `
       -AvailabilitySetId $availSet.Id
 
# For a Linux VM, change the last parameter from -Windows to -Linux 
    Set-AzVMOSDisk `
       -VM $newVM -CreateOption Attach `
       -ManagedDiskId $originalVM.StorageProfile.OsDisk.ManagedDisk.Id `
       -Name $originalVM.StorageProfile.OsDisk.Name `
       -Windows

# Add Data Disks
    foreach ($disk in $originalVM.StorageProfile.DataDisks) { 
    Add-AzVMDataDisk -VM $newVM `
       -Name $disk.Name `
       -ManagedDiskId $disk.ManagedDisk.Id `
       -Caching $disk.Caching `
       -Lun $disk.Lun `
       -DiskSizeInGB $disk.DiskSizeGB `
       -CreateOption Attach
    }
    
# Add NIC(s) and keep the same NIC as primary
    foreach ($nic in $originalVM.NetworkProfile.NetworkInterfaces) {    
    if ($nic.Primary -eq "True")
    {
            Add-AzVMNetworkInterface `
               -VM $newVM `
               -Id $nic.Id -Primary
               }
           else
               {
                 Add-AzVMNetworkInterface `
                -VM $newVM `
                 -Id $nic.Id 
                }
      }

# Recreate the VM
    New-AzVM `
       -ResourceGroupName $resourceGroup `
       -Location $originalVM.Location `
       -VM $newVM `
       -DisableBginfoExtension
```

## <a name="next-steps"></a>後續步驟

透過新增額外 [資料磁碟](attach-managed-disk-portal.md)，將額外的存放裝置新增到您的 VM。
