---
title: 使用 Azure Machine Learning 訓練 TensorFlow 模型
description: 了解如何使用 TensorFlow 估算器執行 TensorFlow 模型的單一節點與分散式的定型
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: minxia
author: mx-iao
ms.reviewer: sgilley
ms.date: 09/24/2018
ms.openlocfilehash: ba43593e90b78aaa0083faf4f8162a7663c0ad47
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974216"
---
# <a name="how-to-train-tensorflow-models"></a>如何將 TensorFlow 模型定型

為了能使用 TensorFlow 深度訓練類神經網路 (DNN)，Azure Machine Learning 提供了估算器的自訂 TensorFlow 類別。 Azure SDK 的 TensorFlow 估算器 (請勿與 [`tf.estimator.Estimator`](https://www.tensorflow.org/api_docs/python/tf/estimator/Estimator) 類別混為一談) 可讓使用者輕鬆地提交 Azure 計算上的單一節點和分散式執行的 TensorFlow 訓練作業。

## <a name="single-node-training"></a>單一節點訓練
使用 TensorFlow估算器的訓練，類似於使用[基礎估算器](how-to-train-ml-models.md)，因此請先詳讀操作說明文章，確定已了解文章中說明的概念。
  
若要執行 TensorFlow 作業，請具現化 `TensorFlow` 物件。 您應該已建立了自己的[計算目標](how-to-set-up-training-targets.md#batch)物件`compute_target`。

```Python
from azureml.train.dnn import TensorFlow

script_params = {
    '--batch-size': 50,
    '--learning-rate': 0.01,
}

tf_est = TensorFlow(source_directory='./my-tf-proj',
                    script_params=script_params,
                    compute_target=compute_target,
                    entry_script='train.py',
                    conda_packages=['scikit-learn'],
                    use_gpu=True)
```

我們在這裡會為 TensorFlow 建構函式指定以下參數：
* `source_directory`：包含訓練作業所需之所有程式碼的本機目錄。 此資料夾是從您的本機電腦複製到遠端計算
* `script_params`：用以指定您的訓練指令碼 `entry_script` 命令列引數的字典，格式為 <命令列引數, 值> 組
* `compute_target`：您的訓練指令碼將在其上執行的遠端計算，在此案例中是 [Batch AI](how-to-set-up-training-targets.md#batch) 叢集
* `entry_script`：要在遠端計算上執行之訓練指令碼的檔案路徑 (相對於 `source_directory`)。 此檔案 (以及此檔案所相依的其他任何檔案) 都應位於此資料夾
* `conda_packages`：透過 conda 安裝的 Python 套件清單 (您的訓練指令碼所需的套件)。 在此情況下，訓練指令碼會使用 `sklearn` 來載入資料，因此請指定安裝此套件。  
建構函式有另一個名為 `pip_packages` 的參數，可以視需要將之用於任何 pip 套件
* `use_gpu`：請將此旗標設定為 `True`，以利用 GPU 進行訓練。 預設為 `False`。

由於您目前使用的是 TensorFlow 估算器，因此用於訓練的容器會預設納入 TensorFlow 套件，也會含有在 CPU 與 GPU 上進行訓練所需的相關相依性。

接著請提交 TensorFlow 作業：
```Python
run = exp.submit(tf_est)
```

## <a name="distributed-training"></a>分散式訓練
TensorFlow 估算器也可讓您在 Azure VM 的 CPU 與 GPU 叢集上大規模訓練模型。 您可以使用少量 API 呼叫輕鬆地執行分散式 TensorFlow 訓練，而 Azure Machine Learning 會在背景管理執行這類工作負載所需的一切基礎結構和協調流程。

Azure Machine Learning 支援 TensorFlow 中兩種分散式訓練方法：
1. 使用 [Horovod](https://github.com/uber/horovod) 架構的 MPI 型分散式訓練
2. 透過參數伺服器方法的原生[分散式 TensorFlow](https://www.tensorflow.org/deploy/distributed)

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) 是分散式訓練專用的開放原始碼 ring-allreduce 架構，由 Uber 開發而成。

若要使用 Horovod 架構執行分散式 TensorFlow，請建立 TensorFlow 物件，如下所示：

```Python
from azureml.train.dnn import TensorFlow

tf_est = TensorFlow(source_directory='./my-tf-proj',
                    script_params={},
                    compute_target=compute_target,
                    entry_script='train.py',
                    node_count=2,
                    process_count_per_node=1,
                    distributed_backend='mpi',
                    use_gpu=True)
```

上述程式碼會對 TensorFlow 建構函式公開以下新參數：
* `node_count`：用於訓練作業的節點數目。 此引數預設為 `1`
* `process_count_per_node`：要在每個節點上執行的處理序 (或「工作者」) 數目。 此引數預設為 `1`
* `distributed_backend`：用於啟動分散式訓練的後端，由估算器透過 MPI 提供。 此引數預設為 `None`。 如果您想要使用 MPI (和 Horovod) 執行平行或分散式訓練 (例如 `node_count`>1 或 `process_count_per_node`>1，或兩者)，請設定 `distributed_backend='mpi'`。 Azure Machine Learning 所使用的 MPI 實作是[開放式 MPI](https://www.open-mpi.org/)。

上述範例將執行有兩個工作者的分散式訓練，每個節點一個工作者。

系統會為您安裝 Horovod 及其相依項目，因此您只要在訓練指令碼 `train.py` 中匯入 Horovod 即可，如下所示：
```Python
import tensorflow as tf
import horovod
```

最後請提交 TensorFlow 作業：
```Python
run = exp.submit(tf_est)
```

### <a name="parameter-server"></a>參數伺服器
您也可以執行[原生分散式 TensorFlow](https://www.tensorflow.org/deploy/distributed)，其會使用參數伺服器模型。 在此方法中，您會在參數伺服器與工作者的整個叢集中進行訓練。 工作者會在訓練期間會計算升降度，而參數伺服器會彙總升降度。

建構 TensorFlow 物件：
```Python
from azureml.train.dnn import TensorFlow

tf_est = TensorFlow(source_directory='./my-tf-proj',
                    script_params={},
                    compute_target=compute_target,
                    entry_script='train.py',
                    node_count=2,
                    worker_count=2,
                    parameter_server_count=1,
                    distributed_backend='ps',
                    use_gpu=True)
```

請留意上述程式碼中 TensorFlow 建構函式的以下參數：
* `worker_count`：工作者數目。 此引數預設為 `1`
* `parameter_server_count`：參數伺服器數目。 此引數預設為 `1`
* `distributed_backend`：用於分散式訓練的後端。 此引數預設為 `None`。 若要透過參數伺服器進行分散式訓練，需要設定 `distributed_backend='ps'`

#### <a name="note-on-tfconfig"></a>注意 `TF_CONFIG`
還需要 [`tf.train.ClusterSpec`](https://www.tensorflow.org/api_docs/python/tf/train/ClusterSpec) 其叢集的網路位址與連接埠，如此 Azure Machine Learning 才能設定 `TF_CONFIG` 環境變數。

`TF_CONFIG` 環境變數為 JSON 字串。 以下為參數伺服器變數的範例：
```
TF_CONFIG='{
    "cluster": {
        "ps": ["host0:2222", "host1:2222"],
        "worker": ["host2:2222", "host3:2222", "host4:2222"],
    },
    "task": {"type": "ps", "index": 0},
    "environment": "cloud"
}'
```

如果您使用的是 TensorFlow 的高層級 [`tf.estimator`](https://www.tensorflow.org/api_docs/python/tf/estimator) API，則 TensorFlow 將會剖析此 `TF_CONFIG` 變數並建置叢集規格。 

如果改用 TensorFlow 較低層級的核心 API 進行訓練，則必須剖析 `TF_CONFIG` 變數，並自行在訓練程式碼中建置 `tf.train.ClusterSpec`。 在[本範例](https://aka.ms/aml-notebook-tf-ps)中，**訓練指令碼**即需如此處理，如下所示：

```Python
import os, json
import tensorflow as tf

tf_config = os.environ.get('TF_CONFIG')
if not tf_config or tf_config == "":
    raise ValueError("TF_CONFIG not found.")
tf_config_json = json.loads(tf_config)
cluster_spec = tf.train.ClusterSpec(cluster)

```

撰寫完訓練程式碼並建立 TensorFlow 物件後，即可提交您的訓練作業：
```Python
run = exp.submit(tf_est)
```

## <a name="examples"></a>範例
如需單一節點 TensorFlow 訓練的教學課程，請參閱：
* `training/03.train-tune-deploy-tensorflow/03.train-tune-deploy-tensorflow.ipynb`

如需透過 Horovod 進行分散式 TensorFlow 訓練的教學課程，請參閱：
* `training/04.distributed-tensorflow-with-horovod/04.distributed-tensorflow-with-horovod.ipynb`

如需原生分散式 TensorFlow 的教學課程，請參閱：
* `training/05.distributed-tensorflow-with-parameter-server/05.distributed-tensorflow-with-parameter-server.ipynb`

取得以下筆記：

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>後續步驟
* [追蹤定型期間的執行計量](how-to-track-experiments.md) (英文)
* [微調超參數](how-to-tune-hyperparameters.md) (英文)
* [部署定型的模型](how-to-deploy-and-where.md) (英文)
