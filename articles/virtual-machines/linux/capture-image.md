---
title: 使用 Azure CLI 在 Azure 中擷取 Linux VM 的映像 | Microsoft Docs
description: 使用 Azure CLI 擷取要用於大量部署的 Azure VM 映像。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: e608116f-f478-41be-b787-c2ad91b5a802
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 03/22/2018
ms.author: cynthn
ms.openlocfilehash: 98d98c1337830ce54c7ff96c19812169be129584
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946811"
---
# <a name="how-to-create-an-image-of-a-virtual-machine-or-vhd"></a>如何建立虛擬機器或 VHD 的映像

<!-- generalize, image - extended version of the tutorial-->

若要建立虛擬機器 (VM) 的多個複本以在 Azure 中使用，請擷取 VM 或 OS VHD 的映像。 若要建立映像，您必須將個人的帳戶資訊移除，使多次部署更安全。 在下列步驟中，您將取消佈建現有的 VM、解除配置，然後建立映像。 您可以使用此映像，在您訂用帳戶的任何資源群組中建立 VM。

如果您想要建立一份現有 Linux VM 的副本以進行備份或偵錯，或是從內部部署 VM 上傳特定的 Linux VHD，請參閱[從自訂的磁碟映像上傳及建立 Linux VM](upload-vhd.md)。  

您也可以使用 **Packer** 建立您的自訂設定。 如需有關使用 Packer 的詳細資訊，請參閱[如何在 Azure 中使用 Packer 來建立 Linux 虛擬機器映像](build-image-with-packer.md)。


## <a name="before-you-begin"></a>開始之前
請確保符合下列必要條件︰

* 您需要在 Resource Manager 部署模型中使用受控磁碟建立的 Azure VM。 如果您尚未建立 Linux VM，可以使用[入口網站](quick-create-portal.md)、[Azure CLI](quick-create-cli.md) 或 [Resource Manager 範本](create-ssh-secured-vm-from-template.md)。 視需要設定 VM。 例如，[新增資料磁碟](add-disk.md)、套用更新，並安裝應用程式。 

* 您還需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

## <a name="quick-commands"></a>快速命令

如需本主題的簡化版本，以進行測試、評估或深入了解 Azure 中的 VM，請參閱[使用 CLI 建立 Azure VM 的自訂映像](tutorial-custom-images.md)。


## <a name="step-1-deprovision-the-vm"></a>步驟 1：取消佈建 VM
使用 Azure VM 代理程式來取消佈建 VM，以將電腦特定的檔案和資料刪除。 在來源 Linux VM 上使用 `waagent` 命令搭配 -deprovision+user 參數。 如需詳細資訊，請參閱 [Azure Linux 代理程式使用者指南](../extensions/agent-linux.md)。

1. 使用 SSH 用戶端連線到 Linux VM。
2. 在 SSH 視窗中，輸入下列命令：
   
    ```bash
    sudo waagent -deprovision+user
    ```
<br>
   > [!NOTE]
   > 只在您想要擷取作為映像的 VM 上執行這個命令。 這不能保證映像檔中的所有機密資訊都會清除完畢或適合轉散發。 +user 參數也會移除最後一個佈建的使用者帳戶。 如果您想要保留 VM 中的帳戶認證，只要使用 -deprovision 就地保留使用者帳戶即可。
 
3. 輸入 **y** 繼續。 您可以加入 **-force** 參數，便不用進行此確認步驟。
4. 在命令完成之後，請輸入 **exit**。 此步驟會關閉 SSH 用戶端。

## <a name="step-2-create-vm-image"></a>步驟 2：建立 VM 映像
使用 Azure CLI 將 VM 標記為一般化，並擷取映像。 在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 *myResourceGroup*、*myVnet* 和 *myVM*。

1. 使用 [az vm deallocate](/cli//azure/vm#deallocate) 解除配置已取消佈建的 VM。 下列範例會解除配置名為 myResourceGroup 資源群組中名為 myVM 的 VM：
   
    ```azurecli
    az vm deallocate \
      --resource-group myResourceGroup \
      --name myVM
    ```

2. 使用 [az vm generalize](/cli//azure/vm#generalize)，將 VM 標記為一般化。 下列範例會將名為 myResourceGroup 的資源群組中名為 myVM 的 VM 標記為一般化：
   
    ```azurecli
    az vm generalize \
      --resource-group myResourceGroup \
      --name myVM
    ```

3. 使用 [az image create](/cli/azure/image#az_image_create) 建立 VM 資源的映像。 下列範例會使用名為 myVM 的 VM 資源，在 myResourceGroup 資源群組中建立名為 myImage 的映像：
   
    ```azurecli
    az image create \
      --resource-group myResourceGroup \
      --name myImage --source myVM
    ```
   
   > [!NOTE]
   > 此映像與來源 VM 建立於相同的資源群組中。 您可以從此映像，在您訂用帳戶的任何資源群組中建立 VM。 從管理觀點來看，您可能想為您的 VM 資源和映像建立特定的資源群組。
   >
   > 如果您想要將映像儲存於區域復原的儲存體中，則需要在支援[可用性區域](../../availability-zones/az-overview.md)且包含 `--zone-resilient true` 參數的區域中建立它。

## <a name="step-3-create-a-vm-from-the-captured-image"></a>步驟 3：從擷取的映像建立 VM
使用您以 [az vm create](/cli/azure/vm#az_vm_create) 建立的映像來建立 VM。 下列範例會從名為 myImage 的映像建立名為 myVMDeployed 的 VM：

```azurecli
az vm create \
   --resource-group myResourceGroup \
   --name myVMDeployed \
   --image myImage\
   --admin-username azureuser \
   --ssh-key-value ~/.ssh/id_rsa.pub
```

### <a name="creating-the-vm-in-another-resource-group"></a>在另一個資源群組中建立 VM 

您可以從某個映像，在您訂用帳戶的任何資源群組中建立 VM。 若要在與映像不同的資源群組中建立 VM，請指定您映像的完整資源識別碼。 使用 [az image list](/cli/azure/image#az_image_list) 來檢視映像清單。 輸出類似於下列範例：

```json
"id": "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/images/myImage",
   "location": "westus",
   "name": "myImage",
```

下列範例藉由指定映像資源識別碼，進而使用 [az vm create](/cli/azure/vm#az_vm_create) 在與來源映像不同的資源群組中建立 VM︰

```azurecli
az vm create \
   --resource-group myOtherResourceGroup \
   --name myOtherVMDeployed \
   --image "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/images/myImage" \
   --admin-username azureuser \
   --ssh-key-value ~/.ssh/id_rsa.pub
```


## <a name="step-4-verify-the-deployment"></a>步驟 4：驗證部署

現在使用您建立的虛擬機器的 SSH 來驗證部署並開始使用新的 VM。 若要透過 SSH 連接，請利用 [az vm show](/cli/azure/vm#az_vm_show) 尋找您 VM 的 IP 位址或 FQDN：

```azurecli
az vm show \
   --resource-group myResourceGroup \
   --name myVMDeployed \
   --show-details
```

## <a name="next-steps"></a>後續步驟
您可以從來源 VM 映像建立多個 VM。 如果您需要變更您的映像︰ 

- 從映像建立 VM。
- 進行任何更新或組態變更。
- 再次遵循相關步驟，以取消佈建、解除配置、一般化及建立映像。
- 在日後的部署中使用這個新映像。 如有需要，刪除原始的映像。

如需使用 CLI 管理 VM 的詳細資訊，請參閱 [Azure CLI](/cli/azure)。
