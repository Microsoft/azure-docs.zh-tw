---
title: 使用 Azure CLI 從特殊化映射版本建立 VM
description: 使用 Azure CLI，在共用映射庫中使用特製化映射版本建立 VM。
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/23/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: dedefcd18a2860bbcae9a0ac6b5b07550d9cbf3f
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98881935"
---
# <a name="create-a-vm-using-a-specialized-image-version-with-the-azure-cli"></a>使用具有 Azure CLI 的特製化映射版本建立 VM

從儲存在共用映射庫中的 [特製化映射版本](./shared-image-galleries.md#generalized-and-specialized-images) 建立 VM。 如果想要使用一般化映射版本建立 VM，請參閱 [從一般化映射版本建立 vm](vm-generalized-image-version-cli.md)。

視需要取代此範例中的資源名稱。 

使用 [az sig 映射定義清單](/cli/azure/sig/image-definition#az-sig-image-definition-list) 來列出資源庫中的映射定義，以查看定義的名稱和識別碼。

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --query "[].[name, id]" \
   --output tsv
```

使用 [az vm create](/cli/azure/vm#az-vm-create) 建立 VM，並使用 --specialized 參數來指出映像是特製化映像。 

使用 `--image` 的映像定義識別碼，從可用的最新映像版本建立 VM。 您也可以藉由提供 `--image` 的映像版本識別碼，從特定版本建立 VM。 

在此範例中，我們會從最新版本的 myImageDefinition 映像建立 VM。

```azurecli
az group create --name myResourceGroup --location eastus
az vm create --resource-group myResourceGroup \
    --name myVM \
    --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
    --specialized
```


## <a name="next-steps"></a>後續步驟
[Azure 映射產生器 (預覽版) ](./image-builder-overview.md) 可協助自動建立映射版本，您甚至可以使用它來更新和 [建立現有映射版本的新映射版本](./linux/image-builder-gallery-update-image-version.md)。 

您也可以使用範本建立共用映像庫資源。 有數個 Azure 快速入門範本可以使用： 

- [建立共用映像資源庫](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [在共用映像資源庫中建立映像定義](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [在共用映像資源庫中建立映像版本](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [從映像版本建立 VM](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)