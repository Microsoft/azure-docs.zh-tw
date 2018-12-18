---
title: 將 Windows 虛擬機器從非受控磁碟轉換成受控磁碟 - Azure 受控磁碟 | Microsoft Docs
description: 如何在 Resource Manager 部署模型中使用 PowerShell 將 Windows VM 從非受控磁碟轉換成受控磁碟
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
ms.date: 07/12/2018
ms.author: cynthn
ms.openlocfilehash: 6e7d4a0ab6d79e1615f921965fb3d77998eaf90c
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39000942"
---
# <a name="convert-a-windows-virtual-machine-from-unmanaged-disks-to-managed-disks"></a>將 Windows 虛擬機器從非受控磁碟轉換成受控磁碟

如果現有的 Windows 虛擬機器 (VM) 使用非受控磁碟，您可以透過 [Azure 受控磁碟](managed-disks-overview.md)服務，將這些 VM 轉換成使用受控磁碟。 此程序會轉換 OS 磁碟和任何附加的資料磁碟。

本文說明如何使用 Azure PowerShell 來轉換 VM。 如果您需要安裝或升級 Azure PowerShell，請參閱[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。

## <a name="before-you-begin"></a>開始之前


* 檢閱[規劃移轉至受控磁碟](on-prem-to-azure.md#plan-for-the-migration-to-managed-disks)。

* 檢閱[關於移轉至受控磁碟的常見問題](faq-for-disks.md#migrate-to-managed-disks)。

[!INCLUDE [virtual-machines-common-convert-disks-considerations](../../../includes/virtual-machines-common-convert-disks-considerations.md)]




## <a name="convert-single-instance-vms"></a>轉換單一執行個體 VM
本節說明如何將單一執行個體 Azure VM 從非受控磁碟轉換為受控磁碟。 (如果您的 VM 位於可用性設定組中，請參閱下一節)。 

1. 使用 [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm) Cmdlet 將 VM 解除配置。 下列範例會解除配置 `myResourceGroup` 資源群組中名為 `myVM` 的 VM： 

  ```azurepowershell-interactive
  $rgName = "myResourceGroup"
  $vmName = "myVM"
  Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName -Force
  ```

2. 使用 [ConvertTo-AzureRmVMManagedDisk](/powershell/module/azurerm.compute/convertto-azurermvmmanageddisk) Cmdlet 將 VM 轉換成受控磁碟。 下列程序會轉換先前的 VM (包括 OS 磁碟和任何資料磁碟)，然後啟動虛擬機器：

  ```azurepowershell-interactive
  ConvertTo-AzureRmVMManagedDisk -ResourceGroupName $rgName -VMName $vmName
  ```



## <a name="convert-vms-in-an-availability-set"></a>轉換可用性設定組中的 VM

如果您想要轉換為受控磁碟的 VM 位於可用性設定組中，您必須先將此可用性設定組轉換為受控可用性設定組。

1. 使用 [Update-AzureRmAvailabilitySet](/powershell/module/azurerm.compute/update-azurermavailabilityset) Cmdlet 來轉換可用性設定組。 下列範例會更新 `myResourceGroup` 資源群組中名為 `myAvailabilitySet` 的可用性設定組：

  ```azurepowershell-interactive
  $rgName = 'myResourceGroup'
  $avSetName = 'myAvailabilitySet'

  $avSet = Get-AzureRmAvailabilitySet -ResourceGroupName $rgName -Name $avSetName
  Update-AzureRmAvailabilitySet -AvailabilitySet $avSet -Sku Aligned 
  ```

  如果您的可用性設定組所在區域只有 2 個受控容錯網域，但非受控容錯網域的數目為 3，此命令就會顯示類似以下的錯誤：「指定的錯誤網域計數 3 必須介於 1 到 2 之間。」 若要解決此錯誤，請將容錯網域更新為 2，並將 `Sku` 更新為 `Aligned`，如下所示：

  ```azurepowershell-interactive
  $avSet.PlatformFaultDomainCount = 2
  Update-AzureRmAvailabilitySet -AvailabilitySet $avSet -Sku Aligned
  ```

2. 解除配置並轉換可用性設定組中的 VM。 下列指令碼會使用 [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm) Cmdlet 將每個 VM 解除配置，使用 [ConvertTo-AzureRmVMManagedDisk](/powershell/module/azurerm.compute/convertto-azurermvmmanageddisk) 轉換它，然後在轉換過程中將它自動重新啟動：

  ```azurepowershell-interactive
  $avSet = Get-AzureRmAvailabilitySet -ResourceGroupName $rgName -Name $avSetName

  foreach($vmInfo in $avSet.VirtualMachinesReferences)
  {
     $vm = Get-AzureRmVM -ResourceGroupName $rgName | Where-Object {$_.Id -eq $vmInfo.id}
     Stop-AzureRmVM -ResourceGroupName $rgName -Name $vm.Name -Force
     ConvertTo-AzureRmVMManagedDisk -ResourceGroupName $rgName -VMName $vm.Name
  }
  ```


## <a name="troubleshooting"></a>疑難排解

如果在轉換期間發生錯誤，或如果 VM 因為先前轉換問題而處於失敗狀態，請再次執行 `ConvertTo-AzureRmVMManagedDisk` Cmdlet。 簡單重試通常會讓這種情況解除封鎖。
轉換之前，請確定所有 VM 延伸模組都處於 「佈建成功」狀態，否則轉換將會失敗，錯誤碼 409。


## <a name="convert-using-the-azure-portal"></a>使用 Azure 入口網站進行轉換

您也可以使用 Azure 入口網站將非受控磁碟轉換為受控磁碟。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 從入口網站的 VM 清單中選取 VM。
3. 在 VM 刀鋒視窗中，從功能表選取 [磁碟]。
4. 在 [磁碟] 刀鋒視窗頂端，選取 [遷移至受控磁碟]。
5. 如果您的 VM 位於可用性設定組中，[遷移至受控磁碟] 刀鋒視窗上會出現警告，您需要先轉換可用性設定組。 此警告應有一個連結，您可以按一下該連結來轉換可用性設定組。 轉換可用性設定組後，或者如果您的 VM 不在可用性設定組中，請按一下 [遷移] 開始將磁碟遷移至受控磁碟的程序。 

VM 將會停止，並且在移轉完成後重新啟動。

## <a name="next-steps"></a>後續步驟

[將標準受控磁碟轉換成進階受控磁碟](convert-disk-storage.md)

使用[快照](snapshot-copy-managed-disk.md)來取得 VM 的唯讀複本。

