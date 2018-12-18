---
title: 建立 Azure 媒體服務帳戶 - Azure CLI | Microsoft Docs
description: 使用 Azure CLI 指令碼建立 Azure 媒體服務帳戶。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: 9e6279b2f93cb68043fa96866d8269b7d60b9b25
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49379241"
---
# <a name="cli-example-create-an-azure-media-services-account"></a>CLI 範例：建立 Azure 媒體服務帳戶

本主題中的 Azure CLI 指令碼會示範如何建立 Azure 媒體服務帳戶。

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../../cli_scripts/media-services/media-services-create-account/Create-Account.sh "Create Account")]

## <a name="clean-up-deployment"></a>清除部署

執行下列命令來移除資源群組及其所有相關聯的資源。

```azurecli-interactive
az group delete --name amsResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | 建立儲存體帳戶。 |
| [az ams account create](https://docs.microsoft.com/cli/azure/ams/account?view=azure-cli-latest#az-ams-account-create) | 建立媒體服務帳戶。 |
| [az ams account sp create](https://docs.microsoft.com/cli/azure/ams/account/sp?view=azure-cli-latest#az-ams-account-sp-create) | 建立具有密碼的服務主體，並將其存取權設定給 Azure 媒體服務帳戶。 
| [az group delete](/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |


## <a name="next-steps"></a>後續步驟

如需其他範例，請參閱 [Azure CLI 範例](../cli-samples.md)。
