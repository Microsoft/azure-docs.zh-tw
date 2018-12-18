---
title: 包含檔案
description: 包含檔案
services: media-services
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 04/13/2018
ms.author: juliako
ms.custom: include file
ms.openlocfilehash: 9ecb07a2cb278f6cde4ffdc3b252cb9e816d08da
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38733283"
---
## <a name="create-a-media-services-account"></a>建立媒體服務帳戶

您必須先建立媒體服務帳戶。 本節說明使用 Azure CLI 建立帳戶的必要條件。

### <a name="create-a-resource-group"></a>建立資源群組

使用下列命令建立資源群組。 Azure 資源群組是一個邏輯容器，在其中可部署及管理 Azure 媒體服務和相關聯的儲存體帳戶等資源。

```azurecli-interactive
az group create --name amsResourceGroup --location westus2
```

### <a name="create-a-storage-account"></a>建立儲存體帳戶

建立媒體服務帳戶時，您需要提供 Azure 儲存體帳戶資源的名稱。 指定的儲存體帳戶會附加到您的媒體服務帳戶。 

您只能有一個**主要**儲存體帳戶，而與您的媒體服務帳戶相關聯的**次要**儲存體帳戶可以有任意數量。 媒體服務支援**一般用途 v2** (GPv2) 或**一般用途 v1** (GPv1) 帳戶。 僅有 Blob 的帳戶不允許作為**主要**帳戶。 如果您要深入了解儲存體帳戶，請參閱 [Azure 儲存體帳戶選項](../articles/storage/common/storage-account-options.md)。 

如需如何在媒體服務中使用儲存體帳戶的詳細資訊，請參閱[儲存體帳戶](../articles/media-services/latest/storage-account-concept.md)。

以下命令會建立即將與媒體服務帳戶相關聯的儲存體帳戶。 在下列指令碼中，您可將 `storageaccountforams` 替換為您的值。 帳戶名稱的長度必須小於 24。

```azurecli-interactive
az storage account create --name storageaccountforams \  
--kind StorageV2 \
--sku Standard_RAGRS \
--resource-group amsResourceGroup
```

### <a name="create-a-media-services-account"></a>建立媒體服務帳戶

以下 Azure CLI 命令會建立新的媒體服務帳戶。 您可以取代下列值：`amsaccount`  `storageaccountforams` (必須符合您為儲存體帳戶提供的值)，以及 `amsResourceGroup` (必須符合您為資源群組提供的值)。

```azurecli-interactive
az ams account create --name amsaccount --resource-group amsResourceGroup --storage-account storageaccountforams
```
