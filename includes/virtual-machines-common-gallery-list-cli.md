---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 09/20/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: f2e252a000c5633b2cc1ef34683d7d2c7c133c03
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045919"
---
## <a name="using-rbac-to-share-images"></a>使用 RBAC 來共用映像

您可以使用角色型存取控制 (RBAC) 來跨訂用帳戶共用映像。 具有映像版本 (甚至可跨訂用帳戶) 讀取權限的使用者，都能夠使用映像版本來部署虛擬機器。

如需使用 RBAC 共用資源的詳細資訊，請參閱[使用 RBAC 和 Azure CLI 管理存取](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli)。


## <a name="list-information"></a>清單資訊

使用 [az sig list](/cli/azure/sig#az-sig-list) 來取得可用映像資源庫的位置、狀態和其他資訊。

```azurecli-interactive 
az sig list -o table
```

使用 [az sig image-definition list](/cli/azure/sig/image-definition#az-sig-image-definition-list) 來列出資源庫中的映像定義，包括 OS 類型和狀態的相關資訊。

```azurecli-interactive 
az sig image-definition list -g myGalleryRG -r myGallery -o table
```

使用 [az sig image-version list](/cli/azure/sig/image-version#az-sig-image-version-list)，列出資源庫中共用的映像版本。

```azurecli-interactive
az sig image-version list -g myGalleryRG -r myGallery -i myGalleryImage -o table
```

使用 [az sig image-version show](/cli/azure/sig/image-version#az-sig-image-version-show) 來取得映像版本的識別碼。

```
az sig image-version show \
-g myGalleryRG \     
-r myGallery \     
-i myGalleryImage \     
--gallery-image-version-name 1.0.0 \     
--query "id"
```