---
title: Azure CLI 指令碼範例 - 使用 VHD 建立 VM | Microsoft Docs
description: Azure CLI 指令碼範例 - 使用虛擬硬碟建立 VM。
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
ms.date: 03/09/2017
ms.author: allclark
ms.custom: mvc
ms.openlocfilehash: 271390757b7e79eb29c5a3c14ca9ee1b38b53e29
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "41920029"
---
# <a name="create-a-vm-with-a-virtual-hard-disk"></a>使用虛擬硬碟建立 VM

此範例會使用 VHD 建立虛擬機器。
它會建立資源群組、儲存體帳戶和容器，然後透過將 VHD 上傳至容器來建立 VM。
它會將 SSH 公用金鑰以公開金鑰取代，使得您可以存取 VM。

您將需要可開機的 VHD。 指令碼會尋找 `~/sample.vhd`。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-vhd/create-vm-vhd.sh "Create VM using a VHD")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli-interactive 
az group delete -n az-cli-vhd
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、虛擬機器、可用性設定組、負載平衡器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az storage account list](https://docs.microsoft.com/cli/azure/storage/account#az_storage_account_list) | 列出儲存體帳戶 |
| [az storage account check-name](https://docs.microsoft.com/cli/azure/storage/account#az_storage_account_check_name) | 檢查儲存體帳戶名稱有效，而且該它尚不存在 |
| [az storage account keys list](https://docs.microsoft.com/cli/azure/storage/account/keys#az_storage_account_keys_list) | 列出儲存體帳戶的金鑰 |
| [az storage blob exists](https://docs.microsoft.com/cli/azure/storage/blob#az_storage_blob_exists) | 檢查 blob 是否存在 |
| [az storage container create](https://docs.microsoft.com/cli/azure/storage/container#az_storage_container_create) | 在儲存體帳戶中建立容器。 |
| [az storage blob upload](https://docs.microsoft.com/cli/azure/storage/blob#az_storage_blob_upload) | 透過上傳 VHD 在容器中建立 blob。 |
| [az vm list](https://docs.microsoft.com/cli/azure/vm#az_vm_list) | 搭配 `--query` 可檢查 VM 名稱是否正在使用中。 | 
| [az vm create](https://docs.microsoft.com/cli/azure/vm/availability-set#az_vm_availability_set_create) | 建立虛擬機器。 |
| [az vm list-ip-addresses](https://docs.microsoft.com/cli/azure/vm#az_vm_list-ip-addresses) | 取得所建立 VM 的 IP 位址。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Linux VM 文件](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他的虛擬機器 CLI 指令碼範例。
