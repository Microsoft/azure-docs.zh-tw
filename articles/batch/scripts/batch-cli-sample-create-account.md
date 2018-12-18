---
title: Azure CLI 指令碼範例 - 建立 Batch 帳戶 - Batch 服務 | Microsoft Docs
description: Azure CLI 指令碼範例 - 在 Batch 服務模式中建立 Batch 帳戶
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: batch
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 01/29/2018
ms.author: danlep
ms.openlocfilehash: d1c3d892e79138e75d93ae024460c3d8394029f8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980184"
---
# <a name="cli-example-create-a-batch-account-in-batch-service-mode"></a>CLI 範例 - 在 Batch 服務模式中建立 Batch 帳戶

此指令碼會在 Batch 服務模式中建立 Azure Batch 帳戶，並顯示如何查詢或更新帳戶的各種屬性。 當您在預設 Batch 服務模式中建立 Batch 帳戶時，其計算節點是由 Batch 服務在內部指派。 配置的計算節點受個別的 vCPU (核心) 配額限制，帳戶可透過共用金鑰認證或 Azure Active Dirctory 權杖進行驗證。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/batch/create-account/create-account.sh "Create Account")]

## <a name="clean-up-deployment"></a>清除部署

執行下列命令來移除資源群組及其所有相關聯的資源。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az batch account create](/cli/azure/batch/account#az-batch-account-create) | 建立 Batch 帳戶。 |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | 建立儲存體帳戶。 |
| [az batch account set](/cli/azure/batch/account#az-batch-account-set) | 更新 Batch 帳戶的屬性。  |
| [az batch account show](/cli/azure/batch/account#az-batch-account-show) | 擷取指定的 Batch 帳戶的詳細資料。  |
| [az batch account keys list](/cli/azure/batch/account/keys#az-batch-account-keys-list) | 擷取指定的 Batch 帳戶的存取金鑰。  |
| [az batch account login](/cli/azure/batch/account#az-batch-account-login) | 對指定的 Batch 帳戶驗證以進行進一步的 CLI 互動。  |
| [az group delete](/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](/cli/azure)。
