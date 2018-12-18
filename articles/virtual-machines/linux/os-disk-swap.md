---
title: 使用 CLI 交換 Azure VM 的 OS 磁碟 | Microsoft Docs'
description: 使用 CLI 變更 Azure 虛擬機器所使用的作業系統磁碟。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 04/24/2018
ms.author: cynthn
ms.openlocfilehash: 983c2e6d03735ba26f7660fc07dcf1a05ef88189
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960391"
---
# <a name="change-the-os-disk-used-by-an-azure-vm-using-the-cli"></a>使用 CLI 變更 Azure VM 所使用的 OS 磁碟


如果您目前有 VM，但想要交換備份磁碟的磁碟或另一個 OS 磁碟，可以使用 Azure CLI 來交換 OS 磁碟。 您不需要刪除並重新建立 VM。 甚至可以使用另一個資源群組中的受控磁碟，只要該磁碟並非使用中即可。

必須停止\解除配置虛擬機器，然後才能使用不同受控磁碟的資源識別碼取代該受控磁碟的資源識別碼。 

請確定虛擬機器大小和儲存類型能和您想要附加的磁碟相容。 例如，如果您想要使用的磁碟是進階儲存體，VM 就必須能夠支援進階儲存體 (例如 DS 系列的大小)。

本文需要 Azure CLI 2.0.25 版或更高版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 


使用 [az disk list](/cli/azure/disk#list) 來取得資源群組中的磁碟清單。

```azurecli-interactive
az disk list \
   -g myResourceGroupDisk \
   --query '[*].{diskId:id}' \
   --output table
```


使用 [az vm stop](/cli/azure/vm#stop)，在交換磁碟之前停止\解除配置 VM。

```azurecli-interactive
az vm stop \
   -n myVM \
   -g myResourceGroup
```


使用 [az vm update](/cli/azure/vm#az-vm-update)，並搭配含有新磁碟之完整資源識別碼的 `--osdisk` 參數 

```azurecli-interactive 
az vm update \
   -g myResourceGroup \
   -n myVM \
   --os-disk /subscriptions/<subscription ID>/resourceGroups/swap/providers/Microsoft.Compute/disks/myDisk 
   ```
   
使用 [az vm start](/cli/azure/vm#start) 來啟動 VM。

```azurecli-interactive
az vm start \
   -n myVM \
   -g myResourceGroup
```

   
**後續步驟**

若要建立磁碟複本，請參閱[製作磁碟的快照](snapshot-copy-managed-disk.md)。