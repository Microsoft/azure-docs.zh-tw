---
title: 將 Azure 受控磁碟儲存體從標準轉換至進階，反之亦然 | Microsoft Docs
description: 如何使用 Azure PowerShell 將 Azure 受控磁碟從標準轉換至進階，反之亦然。
services: virtual-machines-windows
documentationcenter: ''
author: ramankum
manager: kavithag
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 08/07/2017
ms.author: ramankum
ms.openlocfilehash: 6cfbc458a4bd751b8b871501d19db641e3980767
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719319"
---
# <a name="convert-azure-managed-disks-storage-from-standard-to-premium-and-vice-versa"></a>將 Azure 受控磁碟儲存體從標準轉換至進階，反之亦然

受控磁碟提供三種儲存體選項：[進階 SSD](../windows/premium-storage.md)、標準 SSD (預覽) 及[標準 HDD](../windows/standard-storage.md)。 它可讓您根據您的效能需求，以最少的停機時間在選項之間輕鬆切換。 非受控磁碟不支援此功能。 但您可以輕鬆地[轉換為受控磁碟](convert-unmanaged-to-managed-disks.md)，以在磁碟類型之間輕鬆切換。

本文說明如何使用 Azure PowerShell 將受控磁碟從標準轉換至進階，反之亦然。 如果您需要安裝或升級 Azure PowerShell，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-6.8.1)。

## <a name="before-you-begin"></a>開始之前

* 轉換需要重新啟動 VM，因此請在預先存在的維護期間排定磁碟儲存體移轉。 
* 如果您使用非受控磁碟，請先[轉換為受控磁碟](convert-unmanaged-to-managed-disks.md)，以使用本文在儲存體選項之間切換。 
* 本文需要 Azure PowerShell 模組 6.0.0 版或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 您也需要執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。


## <a name="convert-all-the-managed-disks-of-a-vm-from-standard-to-premium-and-vice-versa"></a>將 VM 的所有受控磁碟從標準轉換至進階，反之亦然

在下列範例中，我們會示範如何將 VM 的所有磁碟從標準儲存體切換成進階儲存體。 若要使用進階受控磁碟，您的 VM 必須使用可支援進階儲存體的 [VM 大小](sizes.md)。 此範例也會切換成可支援進階儲存體的大小。

```azurepowershell-interactive
# Name of the resource group that contains the VM
$rgName = 'yourResourceGroup'

# Name of the your virtual machine
$vmName = 'yourVM'

# Choose between StandardLRS and PremiumLRS based on your scenario
$storageType = 'Premium_LRS'

# Premium capable size
# Required only if converting storage from standard to premium
$size = 'Standard_DS2_v2'

# Stop and deallocate the VM before changing the size
Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName -Force

$vm = Get-AzureRmVM -Name $vmName -resourceGroupName $rgName

# Change the VM size to a size that supports premium storage
# Skip this step if converting storage from premium to standard
$vm.HardwareProfile.VmSize = $size
Update-AzureRmVM -VM $vm -ResourceGroupName $rgName

# Get all disks in the resource group of the VM
$vmDisks = Get-AzureRmDisk -ResourceGroupName $rgName 

# For disks that belong to the selected VM, convert to premium storage
foreach ($disk in $vmDisks)
{
    if ($disk.ManagedBy -eq $vm.Id)
    {
        $diskUpdateConfig = New-AzureRmDiskUpdateConfig –AccountType $storageType
        Update-AzureRmDisk -DiskUpdate $diskUpdateConfig -ResourceGroupName $rgName `
        -DiskName $disk.Name
    }
}

Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
```
## <a name="convert-a-managed-disk-from-standard-to-premium-and-vice-versa"></a>將受控磁碟從標準轉換至進階，反之亦然

對於您的開發/測試工作負載，您可能希望混合標準和進階磁碟，以降低成本。 您可以藉由升級至進階儲存體來完成此作業，僅限需要更佳效能的磁碟。 下列範例們會示範如何將 VM 的單一磁碟從標準儲存體切換成進階儲存體，反之亦然。 若要使用進階受控磁碟，您的 VM 必須使用可支援進階儲存體的 [VM 大小](sizes.md)。 此範例也會切換成可支援進階儲存體的大小。

```azurepowershell-interactive

$diskName = 'yourDiskName'
# resource group that contains the managed disk
$rgName = 'yourResourceGroupName'
# Choose between StandardLRS and PremiumLRS based on your scenario
$storageType = 'Premium_LRS'
# Premium capable size 
$size = 'Standard_DS2_v2'

$disk = Get-AzureRmDisk -DiskName $diskName -ResourceGroupName $rgName

# Get parent VM resource
$vmResource = Get-AzureRmResource -ResourceId $disk.ManagedBy

# Stop and deallocate the VM before changing the storage type
Stop-AzureRmVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name -Force

$vm = Get-AzureRmVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name 

# Change the VM size to a size that supports premium storage
# Skip this step if converting storage from premium to standard
$vm.HardwareProfile.VmSize = $size
Update-AzureRmVM -VM $vm -ResourceGroupName $rgName

# Update the storage type
$diskUpdateConfig = New-AzureRmDiskUpdateConfig -AccountType $storageType -DiskSizeGB $disk.DiskSizeGB
Update-AzureRmDisk -DiskUpdate $diskUpdateConfig -ResourceGroupName $rgName `
-DiskName $disk.Name

Start-AzureRmVM -ResourceGroupName $vm.ResourceGroupName -Name $vm.Name
```

## <a name="convert-a-managed-disk-from-standard-hdd-to-standard-ssd-and-vice-versa"></a>將受控磁碟從標準 HDD 轉換成標準 SSD，反之亦然

下列範例會示範如何將 VM 的單一磁碟從標準 HDD 切換成標準 SSD，反之亦然。

```azurepowershell-interactive

$diskName = 'yourDiskName'
# resource group that contains the managed disk
$rgName = 'yourResourceGroupName'
# Choose between Standard_LRS and StandardSSD_LRS based on your scenario
$storageType = 'StandardSSD_LRS'

$disk = Get-AzureRmDisk -DiskName $diskName -ResourceGroupName $rgName

# Get parent VM resource
$vmResource = Get-AzureRmResource -ResourceId $disk.ManagedBy

# Stop and deallocate the VM before changing the storage type
Stop-AzureRmVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name -Force

$vm = Get-AzureRmVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name 

# Update the storage type
$diskUpdateConfig = New-AzureRmDiskUpdateConfig -AccountType $storageType -DiskSizeGB $disk.DiskSizeGB
Update-AzureRmDisk -DiskUpdate $diskUpdateConfig -ResourceGroupName $rgName `
-DiskName $disk.Name

Start-AzureRmVM -ResourceGroupName $vm.ResourceGroupName -Name $vm.Name
```

## <a name="next-steps"></a>後續步驟

使用[快照](snapshot-copy-managed-disk.md)來取得 VM 的唯讀複本。

