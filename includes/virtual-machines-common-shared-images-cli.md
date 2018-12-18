---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: axayjo
ms.service: virtual-machines
ms.topic: include
ms.date: 09/13/2018
ms.author: akjosh; cynthn
ms.custom: include file
ms.openlocfilehash: 830deb7569772b610b7e6abde649830b7ad67a92
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045759"
---
## <a name="before-you-begin"></a>開始之前

若要完成本文中的範例，您必須具有一般化虛擬機器的現有受控映像。 如需詳細資訊，請參閱[教學課程：使用 Azure CLI 2.0 建立 Azure VM 的自訂映像](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-custom-images)。 

## <a name="preview-register-the-feature"></a>預覽：註冊功能

共用映像資源庫處於預覽狀態，但您必須先註冊該功能，才能使用它。 若要註冊共用映像資源庫功能：

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name GalleryPreview
az provider register -n Microsoft.Compute
```

註冊該功能可能需要幾分鐘的時間。 您可以使用以下命令來檢查進度：

```azurecli-interactive
az provider show -n Microsoft.Compute
```

## <a name="create-an-image-gallery"></a>建立映像資源庫 

映像資源庫是用於啟用映像共用的主要資源。 資源庫名稱在您的訂用帳戶內必須是唯一的。 使用 [az sig create](/cli/azure/sig#az-sig-create) 建立映像資源庫。 下列範例會在 *myGalleryRG* 中建立名為 *myshare* 的資源庫。

```azurecli-interactive
az group create --name myGalleryRG --location WestCentralUS
az sig create -g myGalleryRG --gallery-name myGallery
```

## <a name="create-an-image-definition"></a>建立映像定義

使用 [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create)，在資源庫中建立初始映像定義。

```azurecli-interactive 
az sig image-definition create \
   -g myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku 16.04-LTS \
   --os-type Linux 
```

## <a name="create-an-image-version"></a>建立映像版本 
 
使用 [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create) 建立所需的映像版本。 您必須傳入受控映像的識別碼，以作為建立映像版本的基準。 您可以使用 [az image list](/cli/azure/image?view#az-image-list) 來取得資源群組中映像的相關資訊。 在此範例中，我們的映像版本是 *1.0.0*，而我們即將在「美國中西部」、「美國中南部」和「美國東部 2」區域中總計建立 5 個複本。

```azurecli-interactive 
az sig image-version create \
   -g myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "WestCentralUS" "SouthCentralUS=1" "EastUS2=1" \
   --replica-count 5 \
   --managed-image "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
```

