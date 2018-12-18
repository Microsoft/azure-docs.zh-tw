---
title: Azure 快速入門 - 建立 Batch AI 叢集 - Azure CLI | Microsoft Docs
description: 快速入門 - 建立 Batch AI 叢集來訓練機器學習和 AI 模型 - Azure CLI
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
ms.openlocfilehash: 0d4ba7edfb22a6710222c854ceb2bf86284d2d77
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44057489"
---
# <a name="quickstart-create-a-cluster-for-batch-ai-training-jobs-using-the-azure-cli"></a>快速入門：使用 Azure CLI 建立 Batch AI 訓練作業的叢集

本快速入門說明如何使用 Azure CLI 來建立 Batch AI 叢集，以供用來訓練 AI 和機器學習模型。 Batch AI 是一項受控服務，可讓資料科學家和 AI 研究人員在 Azure 虛擬機器的叢集上大規模地訓練 AI 和機器學習模型。

此叢集一開始只有一個 GPU 節點。 完成本快速入門之後，您就會擁有可相應增加並用來訓練模型的叢集。 請使用 Batch AI、[Azure Machine Learning](../machine-learning/service/overview-what-is-azure-ml.md) 工具或 [Visual Studio Tools for AI](https://github.com/Microsoft/vs-tools-for-ai) 將訓練作業提交至叢集。

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.38 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

本快速入門假設您會在 Bash 殼層 (在 Cloud Shell 中或本機電腦上) 執行命令。

## <a name="create-a-resource-group"></a>建立資源群組

使用 `az group create` 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 eastus2 位置建立名為 myResourceGroup 的資源群組。 請務必選擇「美國東部 2」等可以使用 Batch AI 服務的位置。

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

若要建立 Batch AI 叢集，請使用 `az batchai cluster create` 命令。 下列範例會使用下列屬性建立叢集：

* 包含 NC6 VM 大小的單一節點，其具有一個 NVIDIA Tesla K80 GPU。 
* 執行預設的 Ubuntu Server 映像，此映像的設計目的是要裝載容器型應用程式，以供用於大部分的訓練工作負載。 
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

此命令的輸出會顯示叢集屬性。 需要幾分鐘的時間來建立和啟動節點。 若要查看叢集的狀態，請執行 `az batchai cluster show` 命令。 

```azurecli-interactive
az batchai cluster show \
    --name mycluster \
    --workspace myworkspace \
    --resource-group myResourceGroup \
    --output table
```

在叢集建立早期，輸出會類似下列內容，其顯示叢集處於 `resizing` 狀態：

```bash
Name       Resource Group    Workspace    VM Size       State      Idle    Running    Preparing    Leaving    Unusable
---------  ----------------  -----------  ------------  -------  ------  ---------  -----------  ---------  ----------
mycluster  myResourceGroup   myworkspace  STANDARD_NC6  resizing      0          0            0          0           0

```
當叢集狀態為 `steady` 且單一節點為 `Idle` 時，叢集便可供使用。

## <a name="list-cluster-nodes"></a>列出叢集節點 

如果您需要連線到叢集節點 (在此案例為單一節點) 以安裝應用程式或進行維護，請執行 `az batchai cluster node list` 命令來取得連線資訊：


```azurecli-interactive
az batchai cluster node list \
    --cluster mycluster \
    --workspace myworkspace \
    --resource-group myResourceGroup 
 ```

JSON 輸出如下：

```JSON
[
  {
    "ipAddress": "40.68.254.143",
    "nodeId": "tvm-1816144089_1-20180626t233430z",
    "port": 50000.0
  }
]
```
您可以使用此資訊對節點建立 SSH 連線。 例如，在下列命令中替代成節點的正確 IP 位址：

```bash
ssh myusername@40.68.254.143 -p 50000
``` 
結束 SSH 工作階段以便繼續。

## <a name="resize-the-cluster"></a>調整叢集大小

當您使用叢集來執行訓練作業時，您可能需要更多計算資源。 例如，若要將分散式訓練作業的大小增加為 2 個節點，請執行 [batch ai cluster resize](/cli/azure/batchai/cluster#az-batchai-cluster-resize) 命令：

```azurecli-interactive
az batchai cluster resize \
    --name mycluster \
    --workspace myworkspace \
    --resource-group myResourceGroup \
    --target 2
```

叢集會花幾分鐘的時間來調整大小。

## <a name="clean-up-resources"></a>清除資源

如果您想要繼續進行 Batch AI 教學課程和範例，請使用本快速入門所建立的 Batch AI 工作區。 

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
    --resource-group myResourceGroup \
```

若不再需要，您可以使用 `az group delete` 命令來移除 Batch AI 資源的資源群組。 

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何使用 Azure CLI 來建立 Batch AI 叢集。 若要了解如何使用 Batch AI 叢集來訓練模型，請繼續進行用來訓練深度學習模型的快速入門。

> [!div class="nextstepaction"]
> [訓練深度學習模型](./quickstart-tensorflow-training-cli.md)
