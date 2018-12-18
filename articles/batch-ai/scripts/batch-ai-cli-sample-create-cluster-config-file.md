---
title: Azure CLI 指令碼範例 - 使用設定檔建立 Batch AI 叢集 | Microsoft Docs
description: Azure CLI 指令碼範例 - 藉由在 JSON 檔案中指定組態設定來建立 Batch AI 叢集。
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
ms.date: 08/16/2018
ms.author: danlep
ms.openlocfilehash: a1e472d237977d1948c69828d8ec391522896774
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44058086"
---
# <a name="cli-example-create-a-batch-ai-cluster-using-a-cluster-configuration-file"></a>CLI 範例：使用叢集設定檔來建立 Batch AI 叢集

此指令碼會示範如何使用 JSON 設定檔來指定 Batch AI 叢集的設定。 請對 `az batchai cluster create` 使用這些設定，而非使用對應的命令列參數。 當您需要在叢集節點上裝載多個檔案系統，或想要在數個叢集使用相同的設定時，就很適合使用設定檔。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.38 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI 2.0](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/batch-ai/create-cluster/create-cluster-config-file.sh "Create Batch AI cluster - configuration file")]

## <a name="clean-up-deployment"></a>清除部署

執行下列命令來移除資源群組及其所有相關聯的資源。

```azurecli-interactive
# Remove resource group for the cluster.
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | 建立儲存體帳戶。 |
| [az storage share create](/cli/azure/storage/share#az-storage-share-create) | 在儲存體帳戶中建立檔案共用。 |
| [az batchai workspace create](/cli/azure/batchai/workspace#az-batchai-workspace-create) | 建立 Batch AI 工作區。 |
| [az batchai cluster create](/cli/azure/batchai/cluster#az-batchai-cluster-create) | 建立 Batch AI 叢集。 |
| [az batchai cluster show](/cli/azure/batchai/cluster#az-batchai-cluster-show) | 顯示關於 Batch AI 叢集的資訊。 |
| [az group delete](/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
