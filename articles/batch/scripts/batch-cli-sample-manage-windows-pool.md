---
title: Azure CLI 指令碼範例 - Batch 中的 Windows 集區 | Microsoft Docs
description: Azure CLI 指令碼範例 - 在 Batch 中建立和管理 Windows 集區
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
ms.openlocfilehash: 18fc7f9b0675f5c83f716aedfa6968d1ce48c0ed
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968997"
---
# <a name="cli-example-create-and-manage-a-windows-pool-in-azure-batch"></a>CLI 範例：在 Azure Batch 中建立和管理 Windows 集區

此指令碼示範 Azure CLI 中一些可用的命令，用於建立和管理 Azure Batch 中的 Windows 計算節點集區。 有兩種方式可以設定 Windows 集區，即使用雲端服務組態或虛擬機器組態。 此範例示範如何使用雲端服務組態來建立 Windows 集區。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/batch/manage-pool/manage-pool-windows.sh "Manage Windows Cloud Services Pool")]

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
| [az batch account login](https://docs.microsoft.com/cli/azure/batch/account#az-batch-account-login) | 對指定的 Batch 帳戶驗證以進行進一步的 CLI 互動。 |
| [az batch pool create](https://docs.microsoft.com/cli/azure/batch/pool#az-batch-pool-create) | 建立計算節點的集區。  |
| [az batch pool set](https://docs.microsoft.com/cli/azure/batch/pool#az-batch-pool-set) | 更新集區的屬性。  |
| [az batch pool autoscale enable](https://docs.microsoft.com/cli/azure/batch/pool/autoscale#az-batch-pool-autoscale-enable) | 在集區啟用自動調整規模並套用公式。  |
| [az batch pool show](https://docs.microsoft.com/cli/azure/batch/pool#az-batch-pool-show) | 顯示集區的屬性。  |
| [az batch pool autoscale disable](https://docs.microsoft.com/cli/azure/batch/pool/autoscale#az-batch-pool-autoscale-disable) | 在集區停用自動調整規模。 |
| [az group delete](/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |


## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
