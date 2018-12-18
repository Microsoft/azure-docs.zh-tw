---
title: 在 Azure 中擴充 Windows VM 的 OS 磁碟機 | Microsoft Docs
description: 使用 Resource Manager 部署模型中的 Azure PowerShell，擴充虛擬機器的 OS 磁碟機大小。
services: virtual-machines-windows
documentationcenter: ''
author: kirpasingh
manager: roshar
editor: ''
tags: azure-resource-manager
ms.assetid: d9edfd9f-482f-4c0b-956c-0d2c2c30026c
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 07/05/2018
ms.author: kirpas
ms.openlocfilehash: 097aa128affb6a7ac0770c861ab70ec9d04e2d98
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319110"
---
# <a name="how-to-expand-the-os-drive-of-a-virtual-machine"></a>如何擴充虛擬機器的 OS 磁碟機

當您在資源群組中，透過從 [Azure Marketplace](https://azure.microsoft.com/marketplace/) 部署映像來建立新的虛擬機器 (VM) 時，預設的作業系統磁碟機為 127 GB (根據預設，有些映像的作業系統磁碟大小比較小)。 即使可以將資料磁碟加入到 VM (數量取決於您所選擇的 SKU)，而且建議將應用程式和需要大量 CPU 的工作負載安裝在這些附加的磁碟上，但客戶有時候還是必須擴充作業系統磁碟機以支援特定的案例，例如︰

- 支援將元件安裝在作業系統磁碟機上的舊型應用程式。
- 從具有較大作業系統磁碟機的內部部署移轉實體電腦或虛擬機器。


> [!IMPORTANT]
> 調整 Azure 虛擬機器的作業系統磁碟大小將致使它重新啟動。
>
> 在擴充磁碟之後，您必須[擴充 OS 內的磁碟區](#expand-the-volume-within-the-os)以使用較大的磁碟。
> 

## <a name="resize-a-managed-disk"></a>調整受控磁碟大小

在系統管理模式下開啟您的 Powershell ISE 或 Powershell 視窗，並依照下列步驟進行︰

1. 在資源管理模式下登入您的 Microsoft Azure 帳戶，並選取您的訂用帳戶，如下所示︰
   
   ```powershell
   Connect-AzureRmAccount
   Select-AzureRmSubscription –SubscriptionName 'my-subscription-name'
   ```
2. 設定資源群組名稱和 VM 名稱，如下所示：
   
   ```powershell
   $rgName = 'my-resource-group-name'
   $vmName = 'my-vm-name'
   ```
3. 取得您 VM 的參考，如下所示︰
   
   ```powershell
   $vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```
4. 在調整磁碟大小之前停止 VM，如下所示︰
   
    ```Powershell
    Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
    ```
5. 取得受控 OS 磁碟的參考。 將受控 OS 磁碟的大小設定為所需的值並更新磁碟，如下所示︰
   
   ```Powershell
   $disk= Get-AzureRmDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
   $disk.DiskSizeGB = 1023
   Update-AzureRmDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
   ```   
   > [!WARNING]
   > 新的大小應該大於現有的磁碟大小。 作業系統磁碟允許的上限為 2048 GB。 (可以將 VHD blob 展開超過這個大小，但作業系統只能夠搭配第一個 2048 GB 的空間運作。)
   > 
   > 
6. 更新 VM 可能需要幾秒鐘的時間。 命令執行完畢之後，重新啟動 VM，如下所示︰
   
   ```Powershell
   Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```

這樣就大功告成了！ 現在 RDP 到 VM，開啟 [電腦管理] \(或 [磁碟管理])，然後使用剛配置的空間擴充磁碟機。

## <a name="resize-an-unmanaged-disk"></a>調整非受控磁碟大小

在系統管理模式下開啟您的 Powershell ISE 或 Powershell 視窗，並依照下列步驟進行︰

1. 在資源管理模式下登入您的 Microsoft Azure 帳戶，並選取您的訂用帳戶，如下所示︰
   
   ```Powershell
   Connect-AzureRmAccount
   Select-AzureRmSubscription –SubscriptionName 'my-subscription-name'
   ```
2. 設定資源群組名稱和 VM 名稱，如下所示：
   
   ```Powershell
   $rgName = 'my-resource-group-name'
   $vmName = 'my-vm-name'
   ```
3. 取得您 VM 的參考，如下所示︰
   
   ```Powershell
   $vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```
4. 在調整磁碟大小之前停止 VM，如下所示︰
   
    ```Powershell
    Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
    ```
5. 將非受控 OS 磁碟的大小設定為所需的值並更新 VM，如下所示︰
   
   ```Powershell
   $vm.StorageProfile.OSDisk.DiskSizeGB = 1023
   Update-AzureRmVM -ResourceGroupName $rgName -VM $vm
   ```
   
   > [!WARNING]
   > 新的大小應該大於現有的磁碟大小。 作業系統磁碟允許的上限為 2048 GB。 (可以將 VHD blob 展開超過這個大小，但作業系統只能夠搭配第一個 2048 GB 的空間運作。)
   > 
   > 
   
6. 更新 VM 可能需要幾秒鐘的時間。 命令執行完畢之後，重新啟動 VM，如下所示︰
   
   ```Powershell
   Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```


## <a name="scripts-for-os-disk"></a>用於 OS 磁碟的指令碼

以下為受控和非受控磁碟的完整指令碼，供您參考：


**受控磁碟**

```Powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
$disk= Get-AzureRmDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
$disk.DiskSizeGB = 1023
Update-AzureRmDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
```

**非受控磁碟**

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
$vm.StorageProfile.OSDisk.DiskSizeGB = 1023
Update-AzureRmVM -ResourceGroupName $rgName -VM $vm
Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
```

## <a name="resizing-data-disks"></a>調整資料磁碟大小

本文主要著重於擴充 VM 的 OS 磁碟，但指令碼也可用於擴充連結到 VM 的資料磁碟。 例如，若要擴充連接至 VM 的第一個資料磁碟，請將 `StorageProfile` 的 `OSDisk` 物件取代成 `DataDisks` 陣列，並使用數值索引取得第一個連接的資料磁碟的參考，如下所示︰

**受控磁碟**

```powershell
$disk= Get-AzureRmDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.DataDisks[0].Name
$disk.DiskSizeGB = 1023
```


**非受控磁碟**

```powershell
$vm.StorageProfile.DataDisks[0].DiskSizeGB = 1023
```



同樣地，您可以使用如上所示的索引或磁碟的 **Name** 屬性，參考連結到 VM 的其他資料磁碟︰


**受控磁碟**

```powershell
(Get-AzureRmDisk -ResourceGroupName $rgName -DiskName ($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).Name).DiskSizeGB = 1023
```

**非受控磁碟**

```powershell
($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'}).DiskSizeGB = 1023
```

## <a name="expand-the-volume-within-the-os"></a>擴充 OS 內的磁碟區

當您擴充 VM 的磁碟後，就必須進入 OS 並擴充磁碟區，以納入新的空間。 有數種方法可以擴充磁碟分割。 本章節說明如何使用 RDP 連線與 VM 建立連線，以使用 **DiskPart**擴充磁碟分割。

1. 開啟連到 VM 的 RDP 連線。

2.  開啟命令提示字元，然後鍵入 **diskpart**。

2.  在 **DISKPART** 提示處，鍵入 `list volume`。 記下您想要擴充的磁碟區。

3.  在 **DISKPART** 提示處，鍵入 `select volume <volumenumber>`。 如此會選取您想在相同磁碟上擴充成連續空間的磁碟區 *volumenumber*。

4.  在 **DISKPART** 提示處，鍵入 `extend [size=<size>]`。 如此會以 *size* (MB) 為單位，擴充選取的磁碟區。


## <a name="next-steps"></a>後續步驟

您也可使用 [Azure 入口網站](attach-managed-disk-portal.md)連結磁碟。
