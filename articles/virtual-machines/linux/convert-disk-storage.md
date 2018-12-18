---
title: 將 Azure 受控磁碟儲存體從標準轉換至進階，反之亦然 | Microsoft Docs
description: 如何使用 Azure CLI 將 Azure 受控磁碟儲存體從標準轉換至進階，反之亦然。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/12/2018
ms.author: cynthn
ms.openlocfilehash: 8bd5a6012ef13d4cd63d4e5e63b510e55e75ab63
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987266"
---
# <a name="convert-azure-managed-disks-storage-from-standard-to-premium-and-vice-versa"></a>將 Azure 受控磁碟儲存體從標準轉換至進階，反之亦然

受控磁碟提供三種儲存體選項：[進階 SSD](../windows/premium-storage.md)、標準 SSD (預覽) 及[標準 HDD](../windows/standard-storage.md)。 它可讓您根據您的效能需求，以最少的停機時間在選項之間輕鬆切換。 非受控磁碟不支援此功能。 但您可以輕鬆地[轉換為受控磁碟](convert-unmanaged-to-managed-disks.md)，以在磁碟類型之間輕鬆切換。

本文說明如何使用 Azure CLI 將受控磁碟從標準轉換至進階，反之亦然。 如果您需要安裝或升級 Azure CLI，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli.md)。 

## <a name="before-you-begin"></a>開始之前

* 轉換需要重新啟動 VM，因此請在預先存在的維護期間排定磁碟儲存體移轉。 
* 如果您使用非受控磁碟，請先[轉換為受控磁碟](convert-unmanaged-to-managed-disks.md)，以使用本文在儲存體選項之間切換。 


## <a name="convert-all-the-managed-disks-of-a-vm-from-standard-to-premium-and-vice-versa"></a>將 VM 的所有受控磁碟從標準轉換至進階，反之亦然

在下列範例中，我們會示範如何將 VM 的所有磁碟從標準儲存體切換成進階儲存體。 若要使用進階受控磁碟，您的 VM 必須使用可支援進階儲存體的 [VM 大小](sizes.md)。 此範例也會切換成可支援進階儲存體的大小。

 ```azurecli

#resource group that contains the virtual machine
rgName='yourResourceGroup'

#Name of the virtual machine
vmName='yourVM'

#Premium capable size 
#Required only if converting from standard to premium
size='Standard_DS2_v2'

#Choose between Standard_LRS and Premium_LRS based on your scenario
sku='Premium_LRS'

#Deallocate the VM before changing the size of the VM
az vm deallocate --name $vmName --resource-group $rgName

#Change the VM size to a size that supports premium storage 
#Skip this step if converting storage from premium to standard
az vm resize --resource-group $rgName --name $vmName --size $size

#Update the sku of all the data disks 
az vm show -n $vmName -g $rgName --query storageProfile.dataDisks[*].managedDisk -o tsv \
 | awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

#Update the sku of the OS disk
az vm show -n $vmName -g $rgName --query storageProfile.osDisk.managedDisk -o tsv \
| awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

az vm start --name $vmName --resource-group $rgName

```
## <a name="convert-a-managed-disk-from-standard-to-premium-and-vice-versa"></a>將受控磁碟從標準轉換至進階，反之亦然

對於您的開發/測試工作負載，您可能希望混合標準和進階磁碟，以降低成本。 您可以藉由升級至進階儲存體來完成此作業，僅限需要更佳效能的磁碟。 下列範例們會示範如何將 VM 的單一磁碟從標準儲存體切換成進階儲存體，反之亦然。 若要使用進階受控磁碟，您的 VM 必須使用可支援進階儲存體的 [VM 大小](sizes.md)。 此範例也會切換成可支援進階儲存體的大小。

 ```azurecli

#resource group that contains the managed disk
rgName='yourResourceGroup'

#Name of your managed disk
diskName='yourManagedDiskName'

#Premium capable size 
#Required only if converting from standard to premium
size='Standard_DS2_v2'

#Choose between Standard_LRS and Premium_LRS based on your scenario
sku='Premium_LRS'

#Get the parent VM Id 
vmId=$(az disk show --name $diskName --resource-group $rgName --query managedBy --output tsv)

#Deallocate the VM before changing the size of the VM
az vm deallocate --ids $vmId 

#Change the VM size to a size that supports premium storage 
#Skip this step if converting storage from premium to standard
az vm resize --ids $vmId --size $size

# Update the sku
az disk update --sku $sku --name $diskName --resource-group $rgName 

az vm start --ids $vmId 
```

## <a name="convert-a-managed-disk-from-standard-hdd-to-standard-ssd-and-vice-versa"></a>將受控磁碟從標準 HDD 轉換成標準 SSD，反之亦然

下列範例會示範如何將 VM 的單一磁碟從標準 HDD 切換成標準 SSD。

 ```azurecli

#resource group that contains the managed disk
rgName='yourResourceGroup'

#Name of your managed disk
diskName='yourManagedDiskName'

#Choose between Standard_LRS and StandardSSD_LRS based on your scenario
sku='StandardSSD_LRS'

#Get the parent VM Id 
vmId=$(az disk show --name $diskName --resource-group $rgName --query managedBy --output tsv)

#Deallocate the VM before changing the disk type
az vm deallocate --ids $vmId 

# Update the sku
az disk update --sku $sku --name $diskName --resource-group $rgName 

az vm start --ids $vmId 
```

## <a name="convert-using-the-azure-portal"></a>使用 Azure 入口網站進行轉換

您也可以使用 Azure 入口網站將非受控磁碟轉換為受控磁碟。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 從入口網站的 VM 清單中選取 VM。
3. 在 VM 刀鋒視窗中，從功能表選取 [磁碟]。
4. 在 [磁碟] 刀鋒視窗頂端，選取 [遷移至受控磁碟]。
5. 如果您的 VM 位於可用性設定組中，[遷移至受控磁碟] 刀鋒視窗上會出現警告，您需要先轉換可用性設定組。 此警告應有一個連結，您可以按一下該連結來轉換可用性設定組。 轉換可用性設定組後，或者如果您的 VM 不在可用性設定組中，請按一下 [遷移] 開始將磁碟遷移至受控磁碟的程序。 

VM 將會停止，並且在移轉完成後重新啟動。

## <a name="next-steps"></a>後續步驟

使用[快照](snapshot-copy-managed-disk.md)來取得 VM 的唯讀複本。

