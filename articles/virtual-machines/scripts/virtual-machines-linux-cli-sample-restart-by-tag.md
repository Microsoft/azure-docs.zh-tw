---
title: Azure CLI 指令碼範例 - 重新啟動 VM | Microsoft Docs
description: Azure CLI 指令碼範例 - 依標記和識別碼重新啟動 VM
services: virtual-machines-linux
documentationcenter: virtual-machines
author: allclark
manager: douge
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/01/2017
ms.author: allclark
ms.custom: mvc
ms.openlocfilehash: 4c409dfcfba14093d5e2cf28df4bb6c5d1bd9533
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34725967"
---
# <a name="restart-vms"></a>重新啟動 VM

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

這個範例顯示取得一些 VM 並加以重新啟動的幾種方式。

第一種方式可重新啟動資源群組中的所有 VM。

```bash
az vm restart --ids $(az vm list --resource-group myResourceGroup --query "[].id" -o tsv)
```

第二種方式可使用 `az resouce list` 取得已標記的 VM 並篩選至屬於 VM 的資源，然後重新啟動這些 VM。

```bash
az vm restart --ids $(az resource list --tag "restart-tag" --query "[?type=='Microsoft.Compute/virtualMachines'].id" -o tsv)
```

這個範例適用於 Bash 殼層。 如需在 Windows 用戶端上執行 Azure CLI 指令碼的選項，請參閱[在 Windows 上安裝 Azure CLI](/cli/azure/install-azure-cli-windows)。


## <a name="sample-script"></a>範例指令碼

此範例有三個指令碼。
第一個指令碼會佈建虛擬機器。
它會使用不等待選項，因此命令會傳回，而不會在每個 VM 上等待佈建。
第二個指令碼會等待 VM 完整佈建。
第三個指令碼會重新啟動所有已佈建的 VM，然後只重新啟動已標記的 VM。

### <a name="provision-the-vms"></a>佈建 VM

此指令碼會建立資源群組，然後建立三部要重新啟動的 VM。
其中兩部已加上標記。

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/restart-by-tag/provision.sh "Provision the VMs")]

### <a name="wait"></a>等候

此指令碼會每 20 秒檢查一次佈建狀態，直到三部 VM 都已佈建，或其中一部佈建失敗為止。

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/restart-by-tag/wait.sh "Wait for the VMs to be provisioned")]

### <a name="restart-the-vms"></a>重新啟動 VM

此指令碼重新啟動資源群組中的所有 VM，然後只重新啟動已標記的 VM。

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/restart-by-tag/restart.sh "Restart VMs by tag")]

## <a name="clean-up-deployment"></a>清除部署 

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組、VM 和所有相關資源。

```azurecli-interactive 
az group delete -n myResourceGroup --no-wait --yes
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、虛擬機器、可用性設定組、負載平衡器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az vm create](https://docs.microsoft.com/cli/azure/vm/availability-set#az_vm_availability_set_create) | 建立虛擬機器。  |
| [az vm list](https://docs.microsoft.com/cli/azure/vm#az_vm_list) | 與 `--query` 搭配使用，確保在重新啟動 VM 之前先加以佈建，然後取得 VM 識別碼加以重新啟動。 |
| [az resource list](https://docs.microsoft.com/cli/azure/vm#az_vm_list) | 與 `--query` 搭配使用，使用標籤取得 VM 的識別碼。 |
| [az vm restart](https://docs.microsoft.com/cli/azure/vm#az_vm_list) | 重新啟動 VM。 |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Linux VM 文件](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他的虛擬機器 CLI 指令碼範例。
