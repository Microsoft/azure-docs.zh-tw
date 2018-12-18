---
title: 將 Azure 中的 Linux 虛擬機器從非受控磁碟轉換成受控磁碟 - Azure 受控磁碟 | Microsoft Docs
description: 如何在 Resource Manager 部署模型中使用 Azure CLI 將 Linux VM 從非受控磁碟轉換成受控磁碟
services: virtual-machines-linux
documentationcenter: ''
author: roygara
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 12/15/2017
ms.author: rogarana
ms.openlocfilehash: 62ac70134c1bdf9d178d59723ff73561049a4bbf
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993065"
---
# <a name="convert-a-linux-virtual-machine-from-unmanaged-disks-to-managed-disks"></a>將 Linux 虛擬機器從非受控磁碟轉換成受控磁碟

如果您現有的 Linux 虛擬機器 (VM) 使用非受控磁碟，您可以將 VM 轉換為使用 [Azure 受控磁碟](../linux/managed-disks-overview.md)。 此程序會轉換 OS 磁碟和任何附加的資料磁碟。

本文說明如何使用 Azure CLI 來轉換 VM。 如果您需要安裝或升級 Azure CLI，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="before-you-begin"></a>開始之前
* 檢閱[關於移轉至受控磁碟的常見問題](faq-for-disks.md#migrate-to-managed-disks)。

[!INCLUDE [virtual-machines-common-convert-disks-considerations](../../../includes/virtual-machines-common-convert-disks-considerations.md)]


## <a name="convert-single-instance-vms"></a>轉換單一執行個體 VM
本節說明如何將單一執行個體 Azure VM 從非受控磁碟轉換為受控磁碟。 (如果您的 VM 位於可用性設定組中，請參閱下一節)。您可以使用此程序將 VM 從進階 (SSD) 非受控磁碟轉換成進階受控磁碟，或從標準 (HDD) 非受控磁碟轉換成標準受控磁碟。

1. 使用 [az vm deallocate](/cli/azure/vm#az_vm_deallocate) 將 VM 解除配置。 下列範例會解除配置 `myResourceGroup` 資源群組中名為 `myVM` 的 VM：

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

2. 使用 [az vm convert](/cli/azure/vm#az_vm_convert) 將 VM 轉換成受控磁碟。 下列程序會轉換名為 `myVM` 的 VM，包括 OS 磁碟和任何資料磁碟︰

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

3. 轉換成受控磁碟之後，使用 [az vm start](/cli/azure/vm#az_vm_start) 來啟動 VM。 下列範例會啟動 `myResourceGroup` 資源群組中名為 `myVM` 的 VM。

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## <a name="convert-vms-in-an-availability-set"></a>轉換可用性設定組中的 VM

如果您想要轉換為受控磁碟的 VM 位於可用性設定組中，您必須先將此可用性設定組轉換為受控可用性設定組。

轉換可用性設定組之前，必須先解除配置可用性設定組中的所有 VM。 在可用性設定組本身轉換成受控可用性設定組之後，請規劃將所有 VM 轉換成受控磁碟。 然後，啟動所有 VM 並繼續像平常一樣運作。

1. 使用 [az vm availability-set list](/cli/azure/vm/availability-set#az_vm_availability_set_list) 來列出可用性設定組中的所有 VM。 下列範例會列出 `myResourceGroup` 資源群組中名為 `myAvailabilitySet` 的可用性設定組中的所有 VM：

    ```azurecli
    az vm availability-set show \
        --resource-group myResourceGroup \
        --name myAvailabilitySet \
        --query [virtualMachines[*].id] \
        --output table
    ```

2. 使用 [az vm deallocate](/cli/azure/vm#az_vm_deallocate) 將所有 VM 解除配置。 下列範例會解除配置 `myResourceGroup` 資源群組中名為 `myVM` 的 VM：

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

3. 使用 [az vm availability-set convert](/cli/azure/vm/availability-set#az_vm_availability_set_convert) 來轉換可用性設定組。 下列範例會轉換 `myResourceGroup` 資源群組中名為 `myAvailabilitySet` 的可用性設定組：

    ```azurecli
    az vm availability-set convert \
        --resource-group myResourceGroup \
        --name myAvailabilitySet
    ```

4. 使用 [az vm convert](/cli/azure/vm#az_vm_convert) 將所有 VM 轉換成受控磁碟。 下列程序會轉換名為 `myVM` 的 VM，包括 OS 磁碟和任何資料磁碟︰

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

5. 轉換成受控磁碟之後，使用 [az vm start](/cli/azure/vm#az_vm_start) 來啟動所有 VM。 下列範例會啟動 `myResourceGroup` 資源群組中名為 `myVM` 的 VM：

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## <a name="next-steps"></a>後續步驟
如需儲存體選項的詳細資訊，請參閱 [Azure 受控磁碟概觀](../windows/managed-disks-overview.md)。
