---
title: 教學課程 - 使用 Azure CLI 建立自訂的 VM 映像 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure CLI 在 Azure 中建立自訂虛擬機器映像
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/13/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: d8e6afb61fd9fa25b220f3267ee718f0859ae9a4
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948786"
---
# <a name="tutorial-create-a-custom-image-of-an-azure-vm-with-the-azure-cli"></a>教學課程：使用 Azure CLI 建立 Azure VM 的自訂映像

自訂映像類似 Marketplace 映像，但您要自行建立它們。 自訂映像可用於啟動程序設定，例如，預先載入應用程式、應用程式設定和其他 OS 設定。 在本教學課程中，您將建立自己的 Azure 虛擬機器自訂映像。 您會了解如何：

> [!div class="checklist"]
> * 取消佈建及一般化 VM
> * 建立自訂映像
> * 從自訂映像建立 VM
> * 列出訂用帳戶中的所有映像
> * 删除映像

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="before-you-begin"></a>開始之前

下列步驟將詳細說明如何將現有 VM 轉換成可重複使用的自訂映像，以便讓您用來建立新的 VM 執行個體。

若要完成本教學課程中的範例，您目前必須具有虛擬機器。 如有需要，這個[指令碼範例](../scripts/virtual-machines-linux-cli-sample-create-vm-nginx.md)可以為您建立一部虛擬機器。 逐步完成教學課程之後，請視需要取代資源群組和 VM 名稱。

## <a name="create-a-custom-image"></a>建立自訂映像

若要建立虛擬機器的映像，您需要藉由取消佈建、解除配置，然後將來源 VM 標示為一般化，來準備 VM。 一旦已備妥 VM，便可以建立映像。

### <a name="deprovision-the-vm"></a>取消佈建 VM 

取消佈建會藉由移除電腦特定的資訊來將 VM 一般化。 這個一般化讓您能夠從單一映像部署多部 VM。 解除佈建期間，將主機名稱重設為 localhost.localdomain。 SSH 主機金鑰、名稱伺服器設定、根密碼及快取的 DHCP 租用也會一併刪除。

若要取消佈建 VM，請使用 Azure VM 代理程式 (waagent)。 Azure VM 代理程式會安裝於 VM 上，並管理佈建以及與 Azure 網狀架構控制器進行互動。 如需詳細資訊，請參閱 [Azure Linux 代理程式使用者指南](../extensions/agent-linux.md)。

使用 SSH 連接到您的 VM，並執行命令以取消佈建 VM。 利用 `+user` 引數，同時刪除最後佈建的使用者帳戶及所有相關聯的資料。 以 VM 的公用 IP 位址取代範例 IP 位址。

透過 SSH 連接到 VM。
```bash
ssh azureuser@52.174.34.95
```
取消佈建 VM。

```bash
sudo waagent -deprovision+user -force
```
關閉 SSH 工作階段。

```bash
exit
```

### <a name="deallocate-and-mark-the-vm-as-generalized"></a>解除配置並將 VM 標示為一般化

若要建立映像，必須解除配置 VM。 使用 [az vm deallocate](/cli//azure/vm#deallocate) 解除配置 VM。 
   
```azurecli-interactive 
az vm deallocate --resource-group myResourceGroup --name myVM
```

最後，使用 [az vm generalize](/cli//azure/vm#generalize) 將 VM 的狀態設為一般化，如此一來，Azure 平台就會知道 VM 已一般化。 您只能從一般化的 VM 建立映像。
   
```azurecli-interactive 
az vm generalize --resource-group myResourceGroup --name myVM
```

### <a name="create-the-image"></a>建立映像

現在，您可以使用 [az image create](/cli//azure/image#create) 來建立 VM 的映像。 下列範例會從名為 myVM 的 VM 建立名為 myImage 的映像。
   
```azurecli-interactive 
az image create \
    --resource-group myResourceGroup \
    --name myImage \
    --source myVM
```
 
## <a name="create-vms-from-the-image"></a>從映像建立 VM

現在您已有映像，您可以使用 [az vm create](/cli/azure/vm#az_vm_create) 從映像建立一個或多個新的 VM。 下列範例會從名為 myImage 的映像建立名為 myVMfromImage 的 VM。

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroup \
    --name myVMfromImage \
    --image myImage \
    --admin-username azureuser \
    --generate-ssh-keys
```

## <a name="image-management"></a>映像管理 

以下範例是一些常見的映像管理作業，以及如何使用 Azure CLI 完成這些作業。

以資料表格式依名稱列出所有映像。

```azurecli-interactive 
az image list \
    --resource-group myResourceGroup
```

删除映像。 此範例會刪除 myResourceGroup 中名為 myOldImage 的映像。

```azurecli-interactive 
az image delete \
    --name myOldImage \
    --resource-group myResourceGroup
```

## <a name="next-steps"></a>後續步驟

您在本教學課程中建立了自訂 VM 映像。 您已了解如何︰

> [!div class="checklist"]
> * 取消佈建及一般化 VM
> * 建立自訂映像
> * 從自訂映像建立 VM
> * 列出訂用帳戶中的所有映像
> * 删除映像

請前進到下一個教學課程，了解高可用性的虛擬機器。

> [!div class="nextstepaction"]
> [建立高可用性 VM](tutorial-availability-sets.md)。

