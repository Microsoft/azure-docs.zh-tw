---
title: Azure CLI 指令碼範例 - 建立專用的 Batch AI 叢集 | Microsoft Docs
description: Azure CLI 指令碼範例 - 建立和管理專用節點 (虛擬機器) 的 Batch AI 叢集
services: batch-ai
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: batch-ai
ms.devlang: azurecli
ms.topic: sample
ms.tgt-pltfrm: multiple
ms.workload: na
ms.date: 07/26/2018
ms.author: danlep
ms.openlocfilehash: 10f3444f81dfaeac4331f0b7798ade7eefbd29fb
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44058079"
---
# <a name="cli-example-create-and-manage-a-batch-ai-cluster-of-dedicated-nodes"></a>CLI 範例：建立和管理專用節點的 Batch AI 叢集

此指令碼會示範一些可在 Azure CLI 中用來建立和管理 Batch AI 叢集 (由專用節點 (虛擬機器) 所組成) 的命令。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.38 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI 2.0](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/batch-ai/create-cluster/create-cluster-dedicated.sh "Create Batch AI cluster - dedicated nodes")]

## <a name="clean-up-deployment"></a>清除部署

執行下列命令來移除資源群組及其所有相關聯的資源。

```azurecli-interactive
# Remove resource group for the cluster.
az group delete --name myResourceGroup

# Remove resource group for the auto-storage account.
az group delete --name batchaiautostorage
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az batchai workspace create](/cli/azure/batchai/workspace#az-batchai-workspace-create) | 建立 Batch AI 工作區。 |
| [az batchai cluster create](/cli/azure/batchai/cluster#az-batchai-cluster-create) | 建立 Batch AI 叢集。 |
| [az batchai cluster show](/cli/azure/batchai/cluster#az-batchai-cluster-show) | 顯示關於 Batch AI 叢集的資訊。 |
| [az batchai cluster node list](/cli/azure/batchai/cluster/node#az-batchai-cluster-show) | 列出 Batch AI 叢集中的節點。 |
| [az batchai cluster resize](/cli/azure/batchai/cluster#az-batchai-cluster-resize) | 調整 Batch AI 叢集的大小。  |
| [az group delete](/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
