---
title: Azure 快速入門 - 深度學習訓練 - Azure CLI | Microsoft Docs
description: 快速入門 - 快速了解如何使用 Azure CLI 與 Batch AI 在單一 GPU 上訓練 TensorFlow 深度學習類神經網路
services: batch-ai
documentationcenter: na
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.custom: ''
ms.service: batch-ai
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: CLI
ms.topic: quickstart
ms.date: 09/03/2018
ms.author: danlep
ms.openlocfilehash: 99d864a5d519ce56a559bea4db7fe89a113e47b9
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44157917"
---
# <a name="quickstart-train-a-deep-learning-model-with-batch-ai"></a>快速入門：使用 Batch AI 訓練深度學習模型

本快速入門說明如何在 Batch AI 所管理的具 GPU 功能虛擬機器上，訓練深度學習模型範例。 Batch AI 是一項受控服務，可讓資料科學家和 AI 研究人員在 Azure 虛擬機器的叢集上大規模地訓練 AI 和機器學習模型。 

在此範例中，您會使用 Azure CLI 來設定 Batch AI，以在手寫數字的 [MNIST 資料庫](http://yann.lecun.com/exdb/mnist/)上訓練 [TensorFlow](https://www.tensorflow.org/) 類神經網路範例。 完成本快速入門後，您會了解使用 Batch AI 來訓練 AI 或機器學習模型的重要概念，並準備好嘗試訓練更大規模的不同模型。

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.38 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

本快速入門假設您會在 Bash 殼層 (在 Cloud Shell 中或本機電腦上) 執行命令。 如果您已完成[使用 Azure CLI 建立 Batch AI 叢集](quickstart-create-cluster-cli.md)的快速入門，請略過前兩個用來建立資源群組和 Batch AI 叢集的步驟。

## <a name="create-a-resource-group"></a>建立資源群組

使用 `az group create` 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 eastus2 位置建立名為 myResourceGroup 的資源群組。 請務必選擇「美國東部 2」位置，或其他可以使用 Batch AI 服務的位置。 

```azurecli-interactive 
az group create \
    --name myResourceGroup \
    --location eastus2
```

## <a name="create-a-batch-ai-cluster"></a>建立 Batch AI 帳戶

首先，使用 `az batchai workspace create` 命令來建立 Batch AI 工作區。 您需要工作區來組織 Batch AI 叢集和其他資源。

```azurecli-interactive
az batchai workspace create \
    --workspace myworkspace \
    --resource-group myResourceGroup
```

若要建立 Batch AI 叢集，請使用 `az batchai cluster create` 命令。 下列範例會使用下列屬性建立單節點叢集：

* 使用 NC6 VM 大小，其具有一個 NVIDIA Tesla K80 GPU。 Azure 提供數個具有不同 NVIDIA GPU 的 VM 大小。
* 執行預設的 Ubuntu Server 映像，此映像的設計目的是要裝載容器型應用程式。 您可以在此散發套件上執行大部分的訓練工作負載。 
* 新增名為 myusername 的使用者帳戶，並產生 SSH 金鑰 (如果本機環境的預設金鑰位置 (~/.ssh) 中還沒有這些金鑰)。

```azurecli-interactive
az batchai cluster create \
    --name mycluster \
    --workspace myworkspace \
    --resource-group myResourceGroup \
    --vm-size Standard_NC6 \
    --target 1 \
    --user-name myusername \
    --generate-ssh-keys
```

此命令的輸出會顯示叢集屬性。 需要幾分鐘的時間來建立和啟動節點。 若要查看叢集狀態，請執行 `az batchai cluster show` 命令。

```azurecli-interactive
az batchai cluster show \
    --name mycluster \
    --workspace myworkspace \
    --resource-group myResourceGroup \
    --output table
```

在叢集建立早期，輸出會類似下列內容，其顯示叢集為 `resizing`：

```bash
Name       Resource Group    Workspace    VM Size       State      Idle    Running    Preparing    Leaving    Unusable
---------  ----------------  -----------  ------------  -------  ------  ---------  -----------  ---------  ----------
mycluster  myResourceGroup   myworkspace  STANDARD_NC6  resizing      0          0            0          0           0

```

繼續進行下列步驟來上傳訓練指令碼，並在叢集狀態變更時建立訓練作業。 當叢集為 `steady` 且單一節點為 `Idle` 時，叢集便準備好執行訓練作業。

## <a name="upload-training-script"></a>上傳訓練指令碼

使用 `az storage account create` 命令來建立儲存體帳戶，以便儲存訓練指令碼和訓練輸出。

```azurecli-interactive
az storage account create \
    --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```

使用 `az storage share create` 命令在帳戶中建立名為 `myshare` 的 Azure 檔案共用：

```azurecli-interactive
az storage share create \
    --name myshare \
    --account-name mystorageaccount
```

使用 `az storage directory create` 命令在 Azure 檔案共用中建立目錄。 為訓練指令碼建立 `scripts` 目錄，並為訓練輸出建立 `logs`：

```azurecli-interactive
# Create /scripts directory in file share
az storage directory create \
    --name scripts \
    --share-name myshare \
    --account-name mystorageaccount

# Create /logs directory in file share 
az storage directory create \
    --name logs \
    --share-name myshare \
    --account-name mystorageaccount
```

在 Bash 殼層中，建立本機工作目錄，並下載 TensorFlow [convolutional.py](https://raw.githubusercontent.com/tensorflow/models/master/tutorials/image/mnist/convolutional.py) 範例。 Python 指令碼會在 60,000 個從 0 到 9 的手寫數字 MNIST 影像集上，訓練卷積類神經網路。 然後，它會在一組測試範例上測試模型。

```bash
wget https://raw.githubusercontent.com/tensorflow/models/master/tutorials/image/mnist/convolutional.py
```

使用 `az storage file upload` 命令將指令碼上傳至共用中的 `scripts` 目錄。

```azurecli-interactive
az storage file upload \
    --share-name myshare \
    --path scripts \
    --source convolutional.py \
    --account-name mystorageaccount
```

## <a name="submit-training-job"></a>提交訓練作業

首先，使用 `az batchai experiment create` 命令在工作區中建立 Batch AI 實驗。 實驗是相關 Batch AI 作業的邏輯容器。

```azurecli-interactive
az batchai experiment create \
    --name myexperiment \
    --workspace myworkspace \
    --resource-group myResourceGroup
```

在工作目錄中，建立建立含有下列內容的訓練作業設定檔 `job.json`。 提交訓練作業時會傳遞此設定檔。

這個 `job.json` 檔案所含的設定，會尋找 Python 指令碼檔案並在 GPU 節點的 TensorFlow 容器中執行此檔案。 它也會指定 Azure 儲存體中用來儲存作業輸出檔案的位置。 `<AZURE_BATCHAI_STORAGE_ACCOUNT>` 指出作業提交期間會指定儲存體帳戶名稱。

```json
{
    "$schema": "https://raw.githubusercontent.com/Azure/BatchAI/master/schemas/2018-05-01/job.json",
    "properties": {
        "nodeCount": 1,
        "tensorFlowSettings": {
            "pythonScriptFilePath": "$AZ_BATCHAI_JOB_MOUNT_ROOT/myshare/scripts/convolutional.py"
        },
        "stdOutErrPathPrefix": "$AZ_BATCHAI_JOB_MOUNT_ROOT/myshare/logs",
        "mountVolumes": {
            "azureFileShares": [
                {
                    "azureFileUrl": "https://<AZURE_BATCHAI_STORAGE_ACCOUNT>.file.core.windows.net/myshare",
                    "relativeMountPath": "myshare"
                }
            ]
        },
        "containerSettings": {
            "imageSourceRegistry": {
                "image": "tensorflow/tensorflow:1.8.0-gpu"
            }
        }
    }
}
```

請使用 `az batchai job create` 命令在節點上提交作業，以傳遞 `job.json` 設定檔和儲存體帳戶名稱：

```azurecli-interactive
az batchai job create \
    --name myjob \
    --cluster mycluster \
    --experiment myexperiment \
    --workspace myworkspace \
    --resource-group myResourceGroup \
    --config-file job.json \
    --storage-account-name mystorageaccount
```

此命令會傳回作業屬性，然後花幾分鐘的時間來完成。 若要監視此作業的進度，請使用 `az batchai job file stream` 命令來串流節點上標準輸出目錄中的 `stdout-wk-0.txt` 檔案。 作業開始執行之後，訓練指令碼便會產生這個檔案。  

```azurecli-interactive
az batchai job file stream \
    --job myjob \
    --experiment myexperiment \
    --workspace myworkspace \
    --resource-group myResourceGroup \
    --file-name stdout-wk-0.txt
```

範例輸出︰

```
File found with URL "https://mystorageaccount.file.core.windows.net/logs/00000000-0000-0000-0000-000000000000/myResourceGroup/workspaces/myworkspace/experiments/myexperiment/jobs/myjob/<JOB_ID>/stdouterr/stdout-wk-0.txt?sv=2016-05-31&sr=f&sig=Kih9baozMao8Ugos%2FVG%2BcsVsSeY1O%2FTocCNvLQhwtx4%3D&se=2018-06-20T22%3A07%3A30Z&sp=rl". Start streaming
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
Initialized!
Step 0 (epoch 0.00), 14.9 ms
Minibatch loss: 8.334, learning rate: 0.010000
Minibatch error: 85.9%
Validation error: 84.6%
Step 100 (epoch 0.12), 9.7 ms
Minibatch loss: 3.240, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 7.7%
Step 200 (epoch 0.23), 8.3 ms
Minibatch loss: 3.335, learning rate: 0.010000
Minibatch error: 7.8%
Validation error: 4.5%
Step 300 (epoch 0.35), 8.3 ms
Minibatch loss: 3.157, learning rate: 0.010000
Minibatch error: 3.1%
...
Step 8500 (epoch 9.89), 8.3 ms
Minibatch loss: 1.605, learning rate: 0.006302
Minibatch error: 0.0%
Validation error: 0.9%
Test error: 0.8%
```

串流會在作業完成時停止。 指令碼範例會訓練超過 10 個 Epoch 或通過訓練資料集。 在此範例中，於 10 個 Epoch 之後，經過訓練的模型在執行時只會有 0.8% 的測試錯誤。

## <a name="get-job-output"></a>取得作業輸出

Batch AI 會在儲存體帳戶中為每個作業的輸出建立唯一的資料夾結構。 請使用此路徑設定 JOB_OUTPUT_PATH 環境變數。 然後，使用 `az storage file list` 命令列出儲存體中的輸出檔案：

```azurecli-interactive
export JOB_OUTPUT_PATH=$(az batchai job show --name myjob --experiment myexperiment --workspace myworkspace --resource-group myResourceGroup --query jobOutputDirectoryPathSegment --output tsv)

az storage file list \
    --share-name myshare/logs \
    --account-name mystorageaccount \
    --path $JOB_OUTPUT_PATH/stdouterr \
    --output table
```

輸出會類似：

```
Name               Content Length  Type    Last Modified
---------------  ----------------  ------  ---------------
execution.log               14866  file
stderr-wk-0.txt              1527  file
stdout-wk-0.txt             11027  file
```

使用 `az storage file download` 命令將一或多個檔案下載到本機工作目錄。 例如︰

```azurecli-interactive
az storage file download \
    --share-name myshare/logs \
    --account-name mystorageaccount \
    --path $JOB_OUTPUT_PATH/stdouterr/stdout-wk-0.txt
```

## <a name="clean-up-resources"></a>清除資源

如果您想要繼續進行 Batch AI 教學課程和範例，請使用本快速入門所建立的 Batch AI 工作區、叢集和儲存體帳戶。 

節點執行時，您要支付 Batch AI 叢集的費用。 如果您想要在沒有作業需要執行時保留叢集設定，請將叢集大小調整為 0 個節點。 

```azurecli-interactive
az batchai cluster resize \
    --name mycluster \
    --workspace myworkspace \
    --resource-group myResourceGroup \
    --target 0
```

之後，請將其大小調整為 1 個以上的節點，以便執行作業。 當您不再需要叢集時，請使用 `az batchai cluster delete` 命令加以刪除：

```azurecli-interactive
az batchai cluster delete \
    --name mycluster \
    --workspace myworkspace \
    --resource-group myResourceGroup
```

若不再需要，您可以使用 `az group delete` 命令來移除 Batch AI 資源群組和儲存體資源。 

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟
在本快速入門中，您已了解如何使用 Batch AI，透過 Azure CLI 在單一 GPU VM 上訓練 TensorFlow 深度學習模型範例。 若要了解如何在更大的 GPU 叢集上分散模型訓練，請繼續進行 Batch AI 教學課程。

> [!div class="nextstepaction"]
> [分散式訓練教學課程](./tutorial-horovod-tensorflow.md)