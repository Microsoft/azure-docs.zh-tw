---
title: Azure 快速入門 - 執行 Batch 作業 - CLI
description: 快速了解如何使用 Azure CLI 執行 Batch 作業。
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 07/03/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 75703b4b10bbf3864076952990df2e6c2a9ab28d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969643"
---
# <a name="quickstart-run-your-first-batch-job-with-the-azure-cli"></a>快速入門：使用 Azure CLI 執行您的第一個 Batch 作業

Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本快速入門示範如何使用 Azure CLI 建立 Batch 帳戶、計算節點 (虛擬機器) 的「集區」，以及在集區上執行「工作」的「作業」。 每個範例工作都會在其中一個集區節點上執行基本命令。 完成本快速入門之後，您將了解 Batch 服務的重要概念，並可準備使用更多真實的工作負載來大規模試用 Batch。

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 eastus2 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create \
    --name myResourceGroup \
    --location eastus2
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

您可以連結 Azure 的儲存體帳戶與您的 Batch 帳戶。 本快速入門雖然不要求這麼做，但儲存體帳戶很適合用於部署應用程式以及儲存大部分真實工作負載的輸入和輸出資料。 使用 [az storage account create](/cli/azure/storage/account#az-storage-account-create) 命令在資源群組中建立儲存體帳戶。

```azurecli-interactive
az storage account create \
    --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```

## <a name="create-a-batch-account"></a>建立批次帳戶：

使用 [az batch account create](/cli/azure/batch/account#az-batch-account-create) 命令建立 Batch 帳戶。 您需有帳戶才能建立計算資源 (運算節點的集區) 和 Batch 作業。

下列範例會在 myResourceGroup 中建立名為 mybatchaccount 的 Batch 帳戶，並連結您所建立的儲存體帳戶。  

```azurecli-interactive 
az batch account create \
    --name mybatchaccount \
    --storage-account mystorageaccount \
    --resource-group myResourceGroup \
    --location eastus2
```

若要建立及管理計算集區和作業，您需要向 Batch 進行驗證。 使用 [az batch account login](/cli/azure/batch/account#az-batch-account-login) 命令登入帳戶。 登入之後，`az batch` 命令會使用此帳戶內容。

```azurecli-interactive 
az batch account login \
    --name mybatchaccount \
    --resource-group myResourceGroup \
    --shared-key-auth
```

## <a name="create-a-pool-of-compute-nodes"></a>建立計算節點的集區

您現在已有一個 Batch 帳戶，請使用 [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create) 命令建立 Linux 計算節點的範例集區。 下列範例會建立名為 mypool 的集區，其中包含 2 個大小為 Standard_A1_v2 並執行 Ubuntu 16.04 LTS 的節點。 建議的節點大小可為此快速範例提供良好的效能與成本平衡。
 
```azurecli-interactive
az batch pool create \
    --id mypool --vm-size Standard_A1_v2 \
    --target-dedicated-nodes 2 \
    --image canonical:ubuntuserver:16.04-LTS \
    --node-agent-sku-id "batch.node.ubuntu 16.04" 
```

Batch 會立即建立集區，但需花費數分鐘的時間來配置和啟動計算節點。 在這段期間，集區處於 `resizing` 狀態。 若要查看集區的狀態，請執行 [az batch pool show](/cli/azure/batch/pool#az-batch-pool-show) 命令。 此命令會顯示集區的所有屬性，而且您可以查詢特定屬性。 下列命令可取得集區的配置狀態：

```azurecli-interactive
az batch pool show --pool-id mypool \
    --query "allocationState"
```

繼續執行下列步驟以建立一個作業和數項工作，然而集區狀態一直在改變。 當配置狀態為 `steady` 且所有節點正在執行時，集區已準備好執行工作。 

## <a name="create-a-job"></a>建立工作

既然您有集區，請建立要在其中執行的作業。  Batch 作業是一或多項工作的邏輯群組。 作業包含工作通用的設定，例如優先順序以及要執行工作的集區。 使用 [az batch job create](/cli/azure/batch/job#az-batch-job-create) 命令建立 Batch 作業。 下列範例會在 mypool 集區上建立 myjob 作業。 一開始作業沒有任何工作。

```azurecli-interactive 
az batch job create \
    --id myjob \
    --pool-id mypool
```

## <a name="create-tasks"></a>建立工作

現在使用 [az batch task create](/cli/azure/batch/task#az-batch-task-create) 命令來建立要在作業中執行的一些工作。 在此範例中，您可建立四個完全相同的工作。 每個工作都會執行 `command-line`，以顯示計算節點上的 Batch 環境變數，然後等候 90 秒。 當您使用 Batch 時，您會在此命令列中指定您的應用程式或指令碼。 Batch 提供數種方法來將應用程式和指令碼部署至計算節點。

下列 Bash 指令碼會建立 4 個平行工作 (mytask1 至 mytask4)。

```azurecli-interactive 
for i in {1..4}
do
   az batch task create \
    --task-id mytask$i \
    --job-id myjob \
    --command-line "/bin/bash -c 'printenv | grep AZ_BATCH; sleep 90s'"
done
```

命令輸出會顯示每項工作的設定。 Batch 會將工作分散於計算節點。

## <a name="view-task-status"></a>檢視工作狀態

建立工作之後，Batch 會將它排入佇列以在集區上執行。 一旦節點可用來執行工作，此工作就會執行。

使用 [az batch task show](/cli/azure/batch/task#az-batch-task-show) 命令來檢視 Batch 工作的狀態。 下列範例顯示其中一個集區節點上執行的 mytask1 詳細資料。

```azurecli-interactive 
az batch task show \
    --job-id myjob \
    --task-id mytask1
```

命令輸出包含許多詳細資料，但請記下工作命令列的 `exitCode` 和 `nodeId`。 `exitCode` 為 0 表示工作命令列已順利完成。 `nodeId` 表示工作執行所在集區節點的識別碼。

## <a name="view-task-output"></a>檢視工作輸出

若要列出計算節點上某個工作所建立的檔案，請使用 [az batch task file list](/cli/azure/batch/task#az-batch-task-file-list) 命令。 下列命令會列出 mytask1 所建立的檔案： 

```azurecli-interactive 
az batch task file list \
    --job-id myjob \
    --task-id mytask1 \
    --output table
```

輸出大致如下：

```
Name        URL                                                                                         Is Directory      Content Length
----------  ------------------------------------------------------------------------------------------  --------------  ----------------
stdout.txt  https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/stdout.txt  False                  695
certs       https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/certs       True
wd          https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/wd          True
stderr.txt  https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/stderr.txt  False                     0

```

若要將其中一個輸出檔案下載到本機目錄，請使用 [az batch task file download](/cli/azure/batch/task#az-batch-task-file-download) 命令。 在此範例中，工作輸出位於 `stdout.txt` 中。 

```azurecli-interactive
az batch task file download \
    --job-id myjob \
    --task-id mytask1 \
    --file-path stdout.txt \
    --destination ./stdout.txt
```

您可以在文字編輯器中檢視 `stdout.txt` 的內容。 內容會顯示在節點上設定的 Azure Batch 環境變數。 當您建立自己的 Batch 作業時，您可以在工作命令列中，以及由命令列執行的應用程式和指令碼中，參照這些環境變數。 例如︰

```
AZ_BATCH_TASK_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1
AZ_BATCH_NODE_STARTUP_DIR=/mnt/batch/tasks/startup
AZ_BATCH_CERTIFICATES_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1/certs
AZ_BATCH_ACCOUNT_URL=https://mybatchaccount.eastus2.batch.azure.com/
AZ_BATCH_TASK_WORKING_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1/wd
AZ_BATCH_NODE_SHARED_DIR=/mnt/batch/tasks/shared
AZ_BATCH_TASK_USER=_azbatch
AZ_BATCH_NODE_ROOT_DIR=/mnt/batch/tasks
AZ_BATCH_JOB_ID=myjobl
AZ_BATCH_NODE_IS_DEDICATED=true
AZ_BATCH_NODE_ID=tvm-257509324_2-20180703t215033z
AZ_BATCH_POOL_ID=mypool
AZ_BATCH_TASK_ID=mytask1
AZ_BATCH_ACCOUNT_NAME=mybatchaccount
AZ_BATCH_TASK_USER_IDENTITY=PoolNonAdmin
```
## <a name="clean-up-resources"></a>清除資源
如果您想要繼續執行 Batch 教學課程和範例，請使用本快速入門中建立的 Batch 帳戶和連結的儲存體帳戶。 Batch 帳戶本身不收費。

即使沒有排定的作業，您仍需支付節點執行時的集區費用。 當您不再需要集區時，請使用 [az batch pool delete](/cli/azure/batch/pool#az-batch-pool-delete) 命令加以刪除。 當您刪除集區時，節點上的所有工作輸出也會跟著刪除。 

```azurecli-interactive
az batch pool delete --pool-id mypool
```

若不再需要，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令將資源群組、Batch 帳戶、集區和所有相關資源移除。 刪除資源，如下所示：

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您建立了 Batch 帳戶、Batch 集區和 Batch 作業。 此作業執行了範例工作，而您檢視了在其中一個節點上建立的輸出。 您既然了解 Batch 服務的重要概念，即可準備使用更多真實的工作負載來大規模試用 Batch。 若要深入了解 Azure Batch，請繼續進行 Azure Batch 教學課程。 


> [!div class="nextstepaction"]
> [Azure Batch 教學課程](./tutorial-parallel-dotnet.md)
