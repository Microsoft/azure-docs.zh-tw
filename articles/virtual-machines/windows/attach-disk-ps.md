---
title: 使用 PowerShell 將資料磁碟附加到 Azure 中的 Windows VM | Microsoft Docs
description: 如何使用 PowerShell 搭配 Resource Manager 部署模型，將新的或現有的資料磁碟附加至 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 10/11/2017
ms.author: cynthn
ms.openlocfilehash: 384203134d1588053f91b66d32e9b0bf1ec69306
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38680909"
---
# <a name="attach-a-data-disk-to-a-windows-vm-using-powershell"></a>使用 PowerShell 將資料磁碟附加至 Windows VM

本文說明如何使用 PowerShell 將新的及現有的磁碟附加至 Windows 虛擬機器。 

這麼做之前，請先檢閱下列提示：
* 虛擬機器的大小會控制您可以連接的資料磁碟數目。 如需詳細資訊，請參閱 [虛擬機器的大小](sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* 若要使用進階儲存體，您需要啟用進階儲存體的 VM 大小，像是 DS 系列或 GS 系列的虛擬機器。 如需詳細資訊，請參閱[進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](premium-storage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 6.0.0 版或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。


## <a name="add-an-empty-data-disk-to-a-virtual-machine"></a>將空的資料磁碟新增至虛擬機器

這個範例示範如何將空的資料磁碟新增至現有的虛擬機器。

### <a name="using-managed-disks"></a>使用受控磁碟

```azurepowershell-interactive
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US' 
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzureRmDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128
$dataDisk1 = New-AzureRmDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzureRmVM -Name $vmName -ResourceGroupName $rgName 
$vm = Add-AzureRmVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzureRmVM -VM $vm -ResourceGroupName $rgName
```

### <a name="using-managed-disks-in-an-availability-zone"></a>在可用性區域中使用受控磁碟
若要在可用性區域中建立磁碟，請使用 [New-AzureRmDiskConfig](/powershell/module/azurerm.compute/new-azurermdiskconfig) 並搭配 `-Zone` 參數。 下列範例會在區域 1 建立磁碟。


```powershell
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US 2' 
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzureRmDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128 -Zone 1
$dataDisk1 = New-AzureRmDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzureRmVM -Name $vmName -ResourceGroupName $rgName 
$vm = Add-AzureRmVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzureRmVM -VM $vm -ResourceGroupName $rgName
```


### <a name="initialize-the-disk"></a>初始化磁碟

新增空的磁碟之後，您需要將它初始化。 若要初始化磁碟，您可以登入 VM 並使用磁碟管理。 如果您在建立時於 VM 上啟用 WinRM 和憑證，您可以使用遠端 PowerShell 來初始化磁碟。 您也可以使用自訂指令碼擴充： 

```azurepowershell-interactive
    $location = "location-name"
    $scriptName = "script-name"
    $fileName = "script-file-name"
    Set-AzureRmVMCustomScriptExtension -ResourceGroupName $rgName -Location $locName -VMName $vmName -Name $scriptName -TypeHandlerVersion "1.4" -StorageAccountName "mystore1" -StorageAccountKey "primary-key" -FileName $fileName -ContainerName "scripts"
```
        
指令碼檔案可以包含如下的程式碼來初始化磁碟︰

```azurepowershell-interactive
    $disks = Get-Disk | Where partitionstyle -eq 'raw' | sort number

    $letters = 70..89 | ForEach-Object { [char]$_ }
    $count = 0
    $labels = "data1","data2"

    foreach ($disk in $disks) {
        $driveLetter = $letters[$count].ToString()
        $disk | 
        Initialize-Disk -PartitionStyle MBR -PassThru |
        New-Partition -UseMaximumSize -DriveLetter $driveLetter |
        Format-Volume -FileSystem NTFS -NewFileSystemLabel $labels[$count] -Confirm:$false -Force
    $count++
    }
```


## <a name="attach-an-existing-data-disk-to-a-vm"></a>將現有的資料磁碟附加至 VM

您也可以將現有的受控磁碟連結至虛擬機器作為資料磁碟。 

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$location = "East US" 
$dataDiskName = "myDisk"
$disk = Get-AzureRmDisk -ResourceGroupName $rgName -DiskName $dataDiskName 

$vm = Get-AzureRmVM -Name $vmName -ResourceGroupName $rgName 

$vm = Add-AzureRmVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzureRmVM -VM $vm -ResourceGroupName $rgName
```

## <a name="next-steps"></a>後續步驟

建立[快照集](snapshot-copy-managed-disk.md)。
