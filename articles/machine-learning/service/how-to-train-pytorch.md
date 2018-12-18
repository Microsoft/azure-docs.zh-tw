---
title: 使用 Azure Machine Learning 來將 PyTorch 模型定型
description: 了解如何使用 PyTorch 估算器執行 PyTorch 的單一節點和分散式定型
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: minxia
author: mx-iao
ms.reviewer: sgilley
ms.date: 09/24/2018
ms.openlocfilehash: e569b63f676fb750bcbab88dda6cda39156d41f5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977023"
---
# <a name="how-to-train-pytorch-models"></a>了解如何將 PyTorch 模型定型

針對使用 PyTorch 的深度類神經網路 (DNN)，Azure Machine Learning 提供估算器的自訂 PyTorch 類別。 Azure SDK 的 PyTorch 估算器可讓您輕鬆提交在 Azure 計算上執行的單一節點和分散式 PyTorch 定型工作。

## <a name="single-node-training"></a>單一節點定型
使用 PyTorch 估算器來定型，類似於使用[基礎估算器](how-to-train-ml-models.md)，因此請先詳讀操作說明文章，並確定已了解文章中說明的概念。
  
若要執行 PyTorch 作業，請將 `PyTorch` 物件具現化。 您應該已經建立[計算目標](how-to-set-up-training-targets.md#batch)物件 `compute_target` 與[資料存放區](how-to-access-data.md)物件 `ds`。

```Python
from azureml.train.dnn import PyTorch

script_params = {
    '--data_dir': ds
}

pt_est = PyTorch(source_directory='./my-pytorch-proj',
                 script_params=script_params,
                 compute_target=compute_target,
                 entry_script='train.py',
                 use_gpu=True)
```

我們在這裡會為 PyTorch 建構函式指定下列參數：
* `source_directory`：包含定型作業所需之所有程式碼的本機目錄。 此資料夾是從您的本機電腦複製到遠端計算
* `script_params`：用於指定您的定型指令碼 `entry_script` 命令列引數的字典，格式為 <命令列引數, 值> 組
* `compute_target`：您的定型指令碼將在其上執行的遠端計算，在此案例中是 [Batch AI](how-to-set-up-training-targets.md#batch) 叢集
* `entry_script`：要在遠端計算上執行之定型指令碼的檔案路徑 (相對於 `source_directory`)。 此檔案 (以及此檔案所相依的其他任何檔案) 都應位於此資料夾
* `conda_packages`：要透過 Conda 安裝的 Python 套件清單 (其中包含您的定型指令碼所需的套件)。
建構函式有另一個名為 `pip_packages` 的參數，您可以視需要將此參數用於任何 pip 套件
* `use_gpu`：請將此旗標設定為 `True`，以利用 GPU 進行定型。 預設為 `False`

由於您目前使用的是 PyTorch 估算器，因此用於定型的容器預設會包含 PyTorch 套件，以及在 CPU 與 GPU 上進行定型所需的相關相依性。

接著，提交 PyTorch 作業：
```Python
run = exp.submit(pt_est)
```

## <a name="distributed-training"></a>分散式定型
PyTorch 估算器也可以讓您在 Azure VM 的 CPU 與 GPU 叢集上大規模將模型定型。 您可以使用少量 API 呼叫輕鬆地執行分散式 PyTorch 定型，而 Azure Machine Learning 會在背景管理執行這類工作負載所需的一切基礎結構和協調流程。

Azure Machine Learning 目前支援使用 Horovod 架構的 PyTorch MPI 型分散式定型。

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) \(英文\) 是 Uber 開發的分散式定型專用開放原始碼 Ring-Allreduce 架構。

若要執行使用 Horovod 架構的分散式 PyTorch，請建立 PyTorch 物件，如下所示：

```Python
from azureml.train.dnn import PyTorch

pt_est = PyTorch(source_directory='./my-pytorch-project',
                 script_params={},
                 compute_target=compute_target,
                 entry_script='train.py',
                 node_count=2,
                 process_count_per_node=1,
                 distributed_backend='mpi',
                 use_gpu=True)
```

上面的程式碼公開下列新參數給 PyTorch 建構函式：
* `node_count`：用於定型作業的節點數目。 此引數預設為 `1`
* `process_count_per_node`：要在每個節點上執行的處理序 (或「背景工作角色」) 數目。 此引數預設為 `1`
* `distributed_backend`：用於啟動分散式定型的後端，由估算器透過 MPI 提供。 此引數預設為 `None`。 如果您想要使用 MPI (與 Horovod) 執行平行或分散式定型 (例如 `node_count`>1 或 `process_count_per_node`>1，或兩者)，請設定 `distributed_backend='mpi'`。 Azure Machine Learning 所使用的 MPI 實作是[開放式 MPI](https://www.open-mpi.org/) \(英文\)。

上述範例將執行有兩個背景工作角色的分散式定型，每個節點都有一個背景工作角色。

系統會為您安裝 Horovod 與其相依性，因此您只要在定型指令碼 `train.py` 中匯入 Horovod 即可，如下所示：
```Python
import torch
import horovod
```

最後，提交您的分散式 PyTorch 作業：
```Python
run = exp.submit(pt_est)
```

## <a name="examples"></a>範例
如需單一節點的 PyTorch 定型教學課程，請參閱：
* `training/01.train-tune-deploy-pytorch/01.train-tune-deploy-pytorch.ipynb`

如需使用 Horovod 的分散式 PyTorch 教學課程，請參閱：
* `training/02.distributed-pytorch-with-horovod/02.distributed-pytorch-with-horovod.ipynb`

取得這些 Notebook：

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>後續步驟
* [追蹤定型期間的執行計量](how-to-track-experiments.md)
* [調整超參數](how-to-tune-hyperparameters.md)
* [部署定型的模型](how-to-deploy-and-where.md)
