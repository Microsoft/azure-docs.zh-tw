---
title: 將傳統 VM 移轉到 ARM 受控磁碟 VM | Microsoft Docs
description: 將單一 Azure VM 從傳統部署模型移轉至 Resource Manager 部署模型中的受控磁碟。
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
ms.date: 06/15/2017
ms.author: cynthn
ms.openlocfilehash: d0307b26741a6bbbf29626e670467cdd72697646
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33943576"
---
# <a name="manually-migrate-a-classic-vm-to-a-new-arm-managed-disk-vm-from-the-vhd"></a>手動從 VHD 將傳統 VM 移轉到新的 ARM 受控磁碟 VM 


本節協助您將現有 Azure VM 從傳統部署模型移轉至 Resource Manager 部署模型中的[受控磁碟](managed-disks-overview.md)。


## <a name="plan-for-the-migration-to-managed-disks"></a>規劃移轉至受控磁碟

本節可協助您做出最佳的 VM 和磁碟類型決策。


### <a name="location"></a>位置

挑選 Azure 受控磁碟可用的位置。 如果您要移轉至進階受控磁碟，也請確保進階儲存體可用於您打算移轉至的區域。 如需可用位置的最新資訊，請參閱[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/#services)。

### <a name="vm-sizes"></a>VM 大小

如果您要移轉至進階受控磁碟，您必須將 VM 大小更新為 VM 所在區域中進階儲存體可支援的大小。 檢閱進階儲存體可支援的 VM 大小。 Azure VM 大小的規格已列在 [虛擬機器的大小](sizes.md)一文中。
請檢閱使用於進階儲存體的虛擬機器效能特性，然後選擇最適合您的工作負載的 VM 大小。 確定 VM 上有足夠的磁碟流量頻寬。

### <a name="disk-sizes"></a>磁碟大小

**進階受控磁碟**

有七種型別的進階受控磁碟可以搭配 VM 使用，而且每種都有特定的 IOP 和輸送量限制。 為您的 VM 選擇進階磁碟類型時，請根據應用程式在容量、效能、延展性以及尖峰負載方面的需求，將這些限制納入考量。

| 進階磁碟類型  | P4    | P6    | P10   | P20   | P30   | P40   | P50   | 
|---------------------|-------|-------|-------|-------|-------|-------|-------|
| 磁碟大小           | 128 GB| 512 GB| 128 GB| 512 GB            | 1024 GB (1 TB)    | 2048 GB (2 TB)    | 4095 GB (4 TB)    | 
| 每一磁碟的 IOPS       | 120   | 240   | 500   | 2300              | 5000              | 7500              | 7500              | 
| 每一磁碟的輸送量 | 每秒 25 MB  | 每秒 50 MB  | 每秒 100 MB | 每秒 150 MB | 每秒 200 MB | 每秒 250 MB | 每秒 250 MB | 

**標準受控磁碟**

有七種型別的標準受控磁碟可搭配 VM 使用。 每種類型的容量各不相同，但其 IOPS 和輸送量限制相同。 根據您應用程式的容量需求，選擇標準受控磁碟的類型。

| 標準磁碟類型  | S4               | S6               | S10              | S20              | S30              | S40              | S50              | 
|---------------------|---------------------|---------------------|------------------|------------------|------------------|------------------|------------------| 
| 磁碟大小           | 30 GB            | 64 GB            | 128 GB           | 512 GB           | 1024 GB (1 TB)   | 2048 GB (2TB)    | 4095 GB (4 TB)   | 
| 每一磁碟的 IOPS       | 500              | 500              | 500              | 500              | 500              | 500             | 500              | 
| 每一磁碟的輸送量 | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 


### <a name="disk-caching-policy"></a>磁碟快取原則 

**進階受控磁碟**

根據預設，所有 Premium 資料磁碟的磁碟快取原則都是*唯讀*，而連接至 VM 的 Premium 作業系統磁碟的磁碟快取原則則是*讀寫*。 為使應用程式的 IO 達到最佳效能，建議使用此組態設定。 對於頻繁寫入或唯寫的資料磁碟 (例如 SQL Server 記錄檔)，停用磁碟快取可獲得更佳的應用程式效能。

### <a name="pricing"></a>價格

請檢閱[受控磁碟的價格](https://azure.microsoft.com/pricing/details/managed-disks/)。 進階受控磁碟與進階非受控磁碟的價格相同。 但標準受控磁碟與標準非受控磁碟的價格不同。


## <a name="checklist"></a>檢查清單

1.  如果您要移轉至進階受控磁碟，請確定它可在您要移轉到的區域中使用。

2.  決定您將要使用的新 VM 系列。 如果您要移轉至進階受控磁碟，它應可支援進階儲存體。

3.  決定在您要移轉至的區域中可用的確切 VM 大小。 VM 大小必須足以支援您所擁有的資料磁碟數目。 例如，如果您有 4 個資料磁碟，VM 必須有 2 個或更多核心。 也請考慮處理能力、記憶體和網路頻寬需求。

4.  請備妥目前 VM 的詳細資料，包括磁碟和對應 VHD Blob 的清單。

為您的應用程式做好停機準備。 若要進行全新移轉，您必須停止目前系統中的所有處理。 只有這樣，您才可以使其維持在可移轉至新平台的一致狀態。 停機持續時間取決於磁碟中要移轉的資料量。


## <a name="migrate-the-vm"></a>移轉 VM

為您的應用程式做好停機準備。 若要進行全新移轉，您必須停止目前系統中的所有處理。 只有這樣，您才可以使其維持在可移轉至新平台的一致狀態。 停機持續時間取決於磁碟中要移轉的資料量。

這個部分需要有 Azure PowerShell 模組 6.0.0 版或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 您也需要執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。


1.  首先，設定一般參數：

    ```powershell
    $resourceGroupName = 'yourResourceGroupName'
    
    $location = 'your location' 
    
    $virtualNetworkName = 'yourExistingVirtualNetworkName'
    
    $virtualMachineName = 'yourVMName'
    
    $virtualMachineSize = 'Standard_DS3'
    
    $adminUserName = "youradminusername"
    
    $adminPassword = "yourpassword" | ConvertTo-SecureString -AsPlainText -Force
    
    $imageName = 'yourImageName'
    
    $osVhdUri = 'https://storageaccount.blob.core.windows.net/vhdcontainer/osdisk.vhd'
    
    $dataVhdUri = 'https://storageaccount.blob.core.windows.net/vhdcontainer/datadisk1.vhd'
    
    $dataDiskName = 'dataDisk1'
    ```

2.  使用傳統 VM 中的 VHD 建立受控 OS 磁碟。

    請確定您已將 OS VHD 的完整 URI 提供給 $osVhdUri 參數。 此外，根據您要移轉至的磁碟類型 (進階或標準)，將 **-AccountType** 輸入為 **Premium_LRS** 或 **Standard_LRS**。

    ```powershell
    $osDisk = New-AzureRmDisk -DiskName $osDiskName -Disk (New-AzureRmDiskConfig '
    -AccountType Premium_LRS -Location $location -CreateOption Import -SourceUri $osVhdUri) '
    -ResourceGroupName $resourceGroupName
    ```

3.  將 OS 磁碟附加至新的 VM。

    ```powershell
    $VirtualMachine = New-AzureRmVMConfig -VMName $virtualMachineName -VMSize $virtualMachineSize
    $VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -ManagedDiskId $osDisk.Id '
    -StorageAccountType Premium_LRS -DiskSizeInGB 128 -CreateOption Attach -Windows
    ```

4.  從資料 VHD 檔案建立受控資料磁碟，並將它新增到新的 VM。

    ```powershell
    $dataDisk1 = New-AzureRmDisk -DiskName $dataDiskName -Disk (New-AzureRmDiskConfig '
    -AccountType Premium_LRS -Location $location -CreationDataCreateOption Import '
    -SourceUri $dataVhdUri ) -ResourceGroupName $resourceGroupName
    
    $VirtualMachine = Add-AzureRmVMDataDisk -VM $VirtualMachine -Name $dataDiskName '
    -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1
    ```

5.  藉由設定公用 IP、虛擬網路和 NIC，建立新的 VM。

    ```powershell
    $publicIp = New-AzureRmPublicIpAddress -Name ($VirtualMachineName.ToLower()+'_ip') '
    -ResourceGroupName $resourceGroupName -Location $location -AllocationMethod Dynamic
    
    $vnet = Get-AzureRmVirtualNetwork -Name $virtualNetworkName -ResourceGroupName $resourceGroupName
    
    $nic = New-AzureRmNetworkInterface -Name ($VirtualMachineName.ToLower()+'_nic') '
    -ResourceGroupName $resourceGroupName -Location $location -SubnetId $vnet.Subnets[0].Id '
    -PublicIpAddressId $publicIp.Id
    
    $VirtualMachine = Add-AzureRmVMNetworkInterface -VM $VirtualMachine -Id $nic.Id
    
    New-AzureRmVM -VM $VirtualMachine -ResourceGroupName $resourceGroupName -Location $location
    ```

> [!NOTE]
>可能有一些支援應用程式所需的其他步驟不包含在本指南中。
>
>

## <a name="next-steps"></a>後續步驟

- 連接至虛擬機器。 如需指示，請參閱 [如何連接和登入執行 Windows 的 Azure 虛擬機器](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

