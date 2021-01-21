---
title: 教學課程：使用您自己的資料
titleSuffix: Azure Machine Learning
description: Azure Machine Learning 入門系列的第 4 部分，說明如何在遠端訓練執行中使用您自己的資料。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 09/15/2020
ms.custom: tracking-python
ms.openlocfilehash: c08d9ee6704203d0634d7a1b90a57de9c6a99d31
ms.sourcegitcommit: a0c1d0d0906585f5fdb2aaabe6f202acf2e22cfc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98622790"
---
# <a name="tutorial-use-your-own-data-part-4-of-4"></a>教學課程：使用您自己的資料 (第 4 部，共 4 部)

本教學課程會示範如何上傳和使用您自己的資料，以在 Azure Machine Learning 中訓練機器學習模型。

本教學課程是 *四部分教學課程系列的第 4 部分*，您會在其中了解 Azure Machine Learning 的基本概念，以及在 Azure 中完成作業型機器學習工作。 本教學課程建基於您在[第 1 部分：設定](tutorial-1st-experiment-sdk-setup-local.md)、[第 2 部分：執行 "Hello World"](tutorial-1st-experiment-hello-world.md) 及[第 3 部分：訓練模型](tutorial-1st-experiment-sdk-train.md)。

在[第 3 部分：定型模型](tutorial-1st-experiment-sdk-train.md)中，資料下載方式是透過 PyTorch API 中的內建 `torchvision.datasets.CIFAR10` 方法。 不過，在許多情況下，您會在遠端訓練回合中使用自己的資料。 本文將說明 Azure Machine Learning 中可用來處理自己資料的工作流程。

在本教學課程中，您：

> [!div class="checklist"]
> * 設定訓練指令碼以使用本機目錄中的資料。
> * 在本機測試訓練指令碼。
> * 將資料上傳至 Azure。
> * 建立控制指令碼。
> * 了解新的 Azure Machine Learning 概念 (傳遞參數、資料集、資料存放區)。
> * 提交並執行您的訓練指令碼。
> * 在雲端中檢視程式碼輸出。

## <a name="prerequisites"></a>Prerequisites

* 完成系列的[第 3 部分](tutorial-1st-experiment-sdk-train.md)。

## <a name="adjust-the-training-script"></a>調整訓練指令碼

現在 Azure Machine Learning 中有執行中的訓練指令碼 (tutorial/src/train.py)，而且您可以監視模型效能。 讓我們藉由引進引數來將訓練指令碼參數化。 使用引數可讓您輕鬆比較不同的超參數。

我們的訓練指令碼現在會設定為每次執行時下載 CIFAR10 資料集。 下列 Python 程式碼已調整為從目錄讀取資料。

>[!NOTE] 
> 使用 `argparse` 參數化指令碼。

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/code/pytorch-cifar10-your-data/train.py":::

### <a name="understanding-the-code-changes"></a>了解程式碼變更

`train.py` 中的程式碼已使用 `argparse` 程式庫來設定 `data_path`、`learning_rate` 和 `momentum`。

```python
# .... other code
parser = argparse.ArgumentParser()
parser.add_argument('--data_path', type=str, help='Path to the training data')
parser.add_argument('--learning_rate', type=float, default=0.001, help='Learning rate for SGD')
parser.add_argument('--momentum', type=float, default=0.9, help='Momentum for SGD')
args = parser.parse_args()
# ... other code
```

此外，`train.py` 指令碼已經過調整，可更新最佳化工具來使用使用者定義的參數：

```python
optimizer = optim.SGD(
    net.parameters(),
    lr=args.learning_rate,     # get learning rate from command-line argument
    momentum=args.momentum,    # get momentum from command-line argument
)
```
> [!div class="nextstepaction"]
> [我已調整訓指令碼](?success=adjust-training-script#test-locally) [我遇到問題](https://www.research.net/r/7C6W7BQ?issue=adjust-training-script)

## <a name="test-the-script-locally"></a><a name="test-locally"></a> 在本機測試指令碼

您的指令碼現在可接受 data path 作為引數。 若要開始使用，請在本機進行測試。 將名為 `data` 的資料夾新增至教學課程目錄結構中。 您的目錄結構應如下所示：

```txt
tutorial
└──.azureml
|  └──config.json
|  └──pytorch-env.yml
└──data
└──src
|  └──hello.py
|  └──model.py
|  └──train.py
└──01-create-workspace.py
└──02-create-compute.py
└──03-run-hello.py
└──04-run-pytorch.py
```

如果您在上一個教學課程中，未於本機執行 `train.py`，您將不會有 `data/` 目錄。 在此情況下，請使用 `train.py` 指令碼中的 `download=True`，在本機執行 `torchvision.datasets.CIFAR10` 方法。

此外，若要在本機上執行，請確定您離開教學課程環境並啟用新的 conda 環境：

```bash
conda deactivate                # If you are still using the tutorial environment, exit it
```

```bash
conda env create -f .azureml/pytorch-env.yml    # create the new conda environment with updated dependencies
```

```bash
conda activate pytorch-aml-env          # activate new conda environment
```

若要在本機執行修改過的訓練指令碼，請呼叫：

```bash
python src/train.py --data_path ./data --learning_rate 0.003 --momentum 0.92
```

藉由將本機路徑傳入資料，您就無須下載 CIFAR10 資料集。 此外，您也可以針對 learning rate 和 momentum 超參數試驗不同的值，而不一定要在訓練指令碼中進行硬式編碼。

> [!div class="nextstepaction"]
> [我已在本機測試指令碼](?success=test-locally#upload) [我遇到問題](https://www.research.net/r/7C6W7BQ?issue=test-locally)

## <a name="upload-the-data-to-azure"></a><a name="upload"></a> 將資料上傳至 Azure

若要在 Azure Machine Learning 中執行此指令碼，您必須讓您的訓練資料可在 Azure 中使用。 您的 Azure Machine Learning 工作區配備「預設」 資料存放區。 這是您可以用來儲存訓練資料的 Azure Blob 儲存體帳戶。

>[!NOTE] 
> Azure Machine Learning 可讓您連結其他雲端式資料存放區來儲存您的資料。 如需詳細資訊，請參閱[資料存放區文件](./concept-data.md)。  

在 `tutorial` 目錄中，建立名為 `05-upload-data.py` 的新 Python 控制指令碼：

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/05-upload-data.py":::

`target_path` 值會指定資料存放區上用來上傳 CIFAR10 資料的路徑。

>[!TIP] 
> 雖然您使用 Azure Machine Learning 來上傳資料，但您可以使用 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)來上傳特定檔案。 如果您需要 ETL 工具，可以使用 [Azure Data Factory](../data-factory/introduction.md) 將您的資料內嵌到 Azure 中。

在已啟用 tutorial1 Conda 環境的視窗中，執行 Python 檔案以上傳資料。 (上傳應快速且小於 60 秒。)

```bash
python 05-upload-data.py
```

您應該會看到下列標準輸出：

```txt
Uploading ./data\cifar-10-batches-py\data_batch_2
Uploaded ./data\cifar-10-batches-py\data_batch_2, 4 files out of an estimated total of 9
.
.
Uploading ./data\cifar-10-batches-py\data_batch_5
Uploaded ./data\cifar-10-batches-py\data_batch_5, 9 files out of an estimated total of 9
Uploaded 9 files
```

> [!div class="nextstepaction"]
> [我已上傳資料](?success=upload-data#control-script) [我遇到問題](https://www.research.net/r/7C6W7BQ?issue=upload-data)

## <a name="create-a-control-script"></a><a name="control-script"></a> 建立控制指令碼

如同您先前的動作，建立名為 `06-run-pytorch-data.py` 的新 Python 控制指令碼：

```python
# 06-run-pytorch-data.py
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import ScriptRunConfig
from azureml.core import Dataset

if __name__ == "__main__":
    ws = Workspace.from_config()
    datastore = ws.get_default_datastore()
    dataset = Dataset.File.from_files(path=(datastore, 'datasets/cifar10'))

    experiment = Experiment(workspace=ws, name='day1-experiment-data')

    config = ScriptRunConfig(
        source_directory='./src',
        script='train.py',
        compute_target='cpu-cluster',
        arguments=[
            '--data_path', dataset.as_named_input('input').as_mount(),
            '--learning_rate', 0.003,
            '--momentum', 0.92],
    )
    # set up pytorch environment
    env = Environment.from_conda_specification(
        name='pytorch-env',
        file_path='./.azureml/pytorch-env.yml'
    )
    config.run_config.environment = env

    run = experiment.submit(config)
    aml_url = run.get_portal_url()
    print("Submitted to compute cluster. Click link below")
    print("")
    print(aml_url)
```

### <a name="understand-the-code-changes"></a>了解程式碼變更

控制指令碼類似於[此系列第 3 部分](tutorial-1st-experiment-sdk-train.md)的指令碼，但新增了下列幾行：

:::row:::
   :::column span="":::
      `dataset = Dataset.File.from_files( ... )`
   :::column-end:::
   :::column span="2":::
      [資料集](/python/api/azureml-core/azureml.core.dataset.dataset?preserve-view=true&view=azure-ml-py)會用來參考您上傳至 Azure Blob 儲存體的資料。 資料集是在您資料之上的抽象層，其設計目的是為了改善可靠性和可信度。
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config = ScriptRunConfig(...)`
   :::column-end:::
   :::column span="2":::
      [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) 已修改為包含將傳遞至 `train.py` 的引數清單。 `dataset.as_named_input('input').as_mount()` 引數表示指定的目錄將會「掛接」至計算目標。
   :::column-end:::
:::row-end:::

> [!div class="nextstepaction"]
> [我已建立控制指令碼](?success=control-script#submit-to-cloud)[我遇到問題](https://www.research.net/r/7C6W7BQ?issue=control-script)

## <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit-to-cloud"></a> 將執行提交至 Azure Machine Learning

現在請重新提交回合以使用新的設定：

```bash
python 06-run-pytorch-data.py
```

此程式碼會在 Azure Machine Learning Studio 中列印實驗的 URL。 如果您移至該連結，就能夠看到您的程式碼正在執行。

> [!div class="nextstepaction"]
> [我已重新提交執行](?success=submit-to-cloud#inspect-log) [我遇到問題](https://www.research.net/r/7C6W7BQ?issue=submit-to-cloud)

### <a name="inspect-the-log-file"></a><a name="inspect-log"></a> 檢查記錄檔

在 Studio 中，移至實驗執行 (藉由選取先前的 URL 輸出)，後面接著 [輸出 + 記錄]。 選取 `70_driver_log.txt` 檔案。 您應該會看見下列輸出：

```txt
Processing 'input'.
Processing dataset FileDataset
{
  "source": [
    "('workspaceblobstore', 'datasets/cifar10')"
  ],
  "definition": [
    "GetDatastoreFiles"
  ],
  "registration": {
    "id": "XXXXX",
    "name": null,
    "version": null,
    "workspace": "Workspace.create(name='XXXX', subscription_id='XXXX', resource_group='X')"
  }
}
Mounting input to /tmp/tmp9kituvp3.
Mounted input to /tmp/tmp9kituvp3 as folder.
Exit __enter__ of DatasetContextManager
Entering Run History Context Manager.
Current directory:  /mnt/batch/tasks/shared/LS_root/jobs/dsvm-aml/azureml/tutorial-session-3_1600171983_763c5381/mounts/workspaceblobstore/azureml/tutorial-session-3_1600171983_763c5381
Preparing to call script [ train.py ] with arguments: ['--data_path', '$input', '--learning_rate', '0.003', '--momentum', '0.92']
After variable expansion, calling script [ train.py ] with arguments: ['--data_path', '/tmp/tmp9kituvp3', '--learning_rate', '0.003', '--momentum', '0.92']

Script type = None
===== DATA =====
DATA PATH: /tmp/tmp9kituvp3
LIST FILES IN DATA PATH...
['cifar-10-batches-py', 'cifar-10-python.tar.gz']
```

注意：

- Azure Machine Learning 已自動為您將 Blob 儲存體掛接到計算叢集。
- 控制指令碼中使用的 ``dataset.as_named_input('input').as_mount()`` 會解析為掛接點。

> [!div class="nextstepaction"]
> [我已檢查記錄檔](?success=inspect-log#clean-up-resources) [我遇到問題](https://www.research.net/r/7C6W7BQ?issue=inspect-log)

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

您也可以保留資源群組，但刪除單一工作區。 顯示工作區屬性，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，我們已了解如何使用 `Datastore` 將資料上傳至 Azure。 資料存放區會作為您工作區的雲端儲存體，提供您持久且彈性的位置來保存資料。

您已了解如何修改您的訓練指令碼，以透過命令列接受資料路徑。 藉由使用 `Dataset`，您已能夠將目錄掛接到遠端回合。 

現在您已有了模型，接著請了解：

* 如何[使用 Azure Machine Learning 部署模型](how-to-deploy-and-where.md)。
