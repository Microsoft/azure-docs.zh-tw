---
title: 使用 machine learning 資料集定型
titleSuffix: Azure Machine Learning
description: 瞭解如何將您的資料提供給本機或遠端計算使用 Azure Machine Learning 資料集進行模型定型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 07/31/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.openlocfilehash: 688bec24cbcd88130470634abff0688ead8005ef
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98881681"
---
# <a name="train-models-with-azure-machine-learning-datasets"></a>使用 Azure Machine Learning 資料集來定型模型 

在本文中，您將瞭解如何使用 [Azure Machine Learning 資料集](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py) 來定型機器學習模型。  您可以使用本機或遠端計算目標中的資料集，而不需要擔心連接字串或資料路徑。 

Azure Machine Learning 資料集可讓您與 Azure Machine Learning 訓練功能（例如 [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py)、 [HyperDrive](/python/api/azureml-train-core/azureml.train.hyperdrive?preserve-view=true&view=azure-ml-py) 和 [Azure Machine Learning 管線](./how-to-create-machine-learning-pipelines.md)）緊密整合。

如果您尚未準備好將資料提供給模型定型，但想要將資料載入到您的筆記本以進行資料探索，請參閱如何 [流覽資料集中的資料](how-to-create-register-datasets.md#explore-data)。 

## <a name="prerequisites"></a>先決條件

若要建立和訓練資料集，您需要：

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前先建立免費帳戶。 立即試用[免費或付費版本的 Azure Machine Learning](https://aka.ms/AMLFree)。

* [Azure Machine Learning 工作區](how-to-manage-workspace.md)。

* [已安裝適用于 Python 的 AZURE MACHINE LEARNING SDK](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py) ( # B0 = 1.13.0) ，其中包含 `azureml-datasets` 套件。

> [!Note]
> 某些資料集類別具有 [azureml dataprep](/python/api/azureml-dataprep/?preserve-view=true&view=azure-ml-py) 封裝的相依性。 針對 Linux 使用者，只有下列發行版本才支援這些類別： Red Hat Enterprise Linux、Ubuntu、Fedora 和 CentOS。

## <a name="consume-datasets-in-machine-learning-training-scripts"></a>使用機器學習服務定型腳本中的資料集

如果您的結構化資料尚未註冊為資料集，請建立 TabularDataset，並直接在您的本機或遠端實驗的定型腳本中使用。

在此範例中，您會建立未註冊的 [TabularDataset](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py) ，並將它指定為 ScriptRunConfig 物件中用於定型的腳本引數。 如果您想要在工作區中重複使用此 TabularDataset 與其他實驗，請參閱 [如何將資料集註冊到您的工作區](how-to-create-register-datasets.md#register-datasets)。

### <a name="create-a-tabulardataset"></a>建立 TabularDataset

下列程式碼會從 web url 建立未註冊的 TabularDataset。  

```Python
from azureml.core.dataset import Dataset

web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path)
```

TabularDataset 物件可讓您將 TabularDataset 中的資料載入 pandas 或 Spark 資料框架，讓您可以使用熟悉的資料準備和定型程式庫，而不需要離開您的筆記本。

### <a name="access-dataset-in-training-script"></a>存取定型腳本中的資料集

下列 `--input-data` 程式碼會設定您將在設定定型回合時指定的腳本引數 (請參閱下一節) 。 當表格式資料集傳入做為引數值時，Azure ML 會將其解析為資料集的識別碼，然後您可以用它來存取定型腳本中的資料集 (而不需要在腳本) 中，將資料集的名稱或識別碼進行硬式編碼。 然後，它會使用 [`to_pandas_dataframe()`](/python/api/azureml-core/azureml.data.tabulardataset#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) 方法將該資料集載入至 pandas 資料框架，以便在定型之前進一步進行資料探索和準備。

> [!Note]
> 如果您的原始資料來源包含 NaN、空字串或空白值，則當您使用時， `to_pandas_dataframe()` 這些值會被取代為 *Null* 值。

如果您需要將備妥的資料從記憶體中的 pandas 資料框架載入至新的資料集，請將資料寫入本機檔案（例如 parquet），然後從該檔案建立新的資料集。 深入瞭解 [如何建立資料集](how-to-create-register-datasets.md)。

```Python
%%writefile $script_folder/train_titanic.py

import argparse
from azureml.core import Dataset, Run

parser = argparse.ArgumentParser()
parser.add_argument("--input-data", type=str)
args = parser.parse_args()

run = Run.get_context()
ws = run.experiment.workspace

# get the input dataset by ID
dataset = Dataset.get_by_id(ws, id=args.input_data)

# load the TabularDataset to pandas DataFrame
df = dataset.to_pandas_dataframe()
```

### <a name="configure-the-training-run"></a>設定定型回合

[ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrun?preserve-view=true&view=azure-ml-py)物件是用來設定和提交定型回合。

此程式碼會建立 ScriptRunConfig 物件， `src` 以指定

* 腳本的腳本目錄。 在此目錄中的所有檔案都會上傳到叢集節點以便執行。
* 定型腳本 *train_titanic .py*。
* 定型的輸入資料集， `titanic_ds` 做為腳本引數。 當資料傳遞給腳本時，Azure ML 會將其解析為對應的資料集識別碼。
* 執行的計算目標。
* 執行的環境。

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=script_folder,
                      script='train_titanic.py',
                      # pass dataset as an input with friendly name 'titanic'
                      arguments=['--input-data', titanic_ds.as_named_input('titanic')],
                      compute_target=compute_target,
                      environment=myenv)
                             
# Submit the run configuration for your training run
run = experiment.submit(src)
run.wait_for_completion(show_output=True)                             
```

## <a name="mount-files-to-remote-compute-targets"></a>將檔案掛接至遠端計算目標

如果您有非結構化資料，請建立 [FileDataset](/python/api/azureml-core/azureml.data.filedataset?preserve-view=true&view=azure-ml-py) ，並裝載或下載您的資料檔案，使其可供您的遠端計算目標用於定型。 深入瞭解您的遠端訓練實驗使用 [掛接與下載](#mount-vs-download) 的時機。 

下列範例會建立 FileDataset，並將資料集做為引數傳遞至定型腳本，以將資料集裝載至計算目標。 

> [!Note]
> 如果您使用自訂的 Docker 基底映射，則必須透過作為相依性來安裝保險絲， `apt-get install -y fuse` 才能讓資料集裝載運作。 瞭解如何 [建立自訂群組建映射](how-to-deploy-custom-docker-image.md#build-a-custom-base-image)。

### <a name="create-a-filedataset"></a>建立 FileDataset

下列範例會從 web url 建立未註冊的 FileDataset。 深入瞭解如何從其他來源 [建立資料集](how-to-create-register-datasets.md) 。

```Python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
mnist_ds = Dataset.File.from_files(path = web_paths)
```

### <a name="configure-the-training-run"></a>設定定型回合

建議您在透過函式的參數裝載時，將資料集當作引數傳遞 `arguments` `ScriptRunConfig` 。 如此一來，您就能透過引數，在定型腳本中取得 (掛接點) 的資料路徑。 如此一來，您就可以在任何雲端平臺上，使用相同的定型腳本進行本機的調試和遠端訓練。

下列範例會建立透過 FileDataset 傳遞的 ScriptRunConfig `arguments` 。 提交執行之後，資料集所參考的資料檔將會掛接 `mnist_ds` 至計算目標。

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=script_folder,
                      script='train_mnist.py',
                      # the dataset will be mounted on the remote compute and the mounted path passed as an argument to the script
                      arguments=['--data-folder', mnist_ds.as_mount(), '--regularization', 0.5],
                      compute_target=compute_target,
                      environment=myenv)

# Submit the run configuration for your training run
run = experiment.submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="retrieve-data-in-your-training-script"></a>取出定型腳本中的資料

下列程式碼示範如何在您的腳本中取出資料。

```Python
%%writefile $script_folder/train_mnist.py

import argparse
import os
import numpy as np
import glob

from utils import load_data

# retrieve the 2 arguments configured through `arguments` in the ScriptRunConfig
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# get the file paths on the compute
X_train_path = glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0]
X_test_path = glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0]
y_train_path = glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0]
y_test = glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0]

# load train and test set into numpy arrays
X_train = load_data(X_train_path, False) / 255.0
X_test = load_data(X_test_path, False) / 255.0
y_train = load_data(y_train_path, True).reshape(-1)
y_test = load_data(y_test, True).reshape(-1)
```

## <a name="mount-vs-download"></a>掛接 vs 下載

從 Azure Blob 儲存體、Azure 檔案儲存體、Azure Data Lake Storage Gen1、Azure Data Lake Storage Gen2、Azure SQL Database 和適用於 PostgreSQL 的 Azure 資料庫所建立的資料集，都支援裝載或下載任何格式的檔案。 

當您 **掛接** 資料集時，您會將資料集所參考的檔案附加至目錄， (掛接點) 並讓它可在計算目標上使用。 以 Linux 為基礎的計算支援裝載，包括 Azure Machine Learning 計算、虛擬機器和 HDInsight。 

當您 **下載** 資料集時，資料集所參考的所有檔案都會下載到計算目標。 所有計算類型都支援下載。 

如果您的腳本會處理資料集所參考的所有檔案，而您的計算磁片可以容納您的完整資料集，則建議下載以避免從儲存體服務串流資料的額外負荷。 如果您的資料大小超過計算磁片大小，就不可能進行下載。 在此案例中，我們建議您掛接，因為只會在處理時載入腳本所使用的資料檔案。

下列程式碼會掛接 `dataset` 至位於下列位置的臨時目錄： `mounted_path`

```python
import tempfile
mounted_path = tempfile.mkdtemp()

# mount dataset onto the mounted_path of a Linux-based compute
mount_context = dataset.mount(mounted_path)

mount_context.start()

import os
print(os.listdir(mounted_path))
print (mounted_path)
```

## <a name="get-datasets-in-machine-learning-scripts"></a>取得機器學習服務腳本中的資料集

註冊的資料集可在本機和遠端的計算叢集上存取，例如 Azure Machine Learning 計算。 若要跨實驗存取您註冊的資料集，請使用下列程式碼存取您的工作區，並取得先前提交之執行中所使用的資料集。 根據預設， [`get_by_name()`](/python/api/azureml-core/azureml.core.dataset.dataset?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-by-name-workspace--name--version--latest--) 類別上的方法 `Dataset` 會傳回已向工作區註冊之資料集的最新版本。

```Python
%%writefile $script_folder/train.py

from azureml.core import Dataset, Run

run = Run.get_context()
workspace = run.experiment.workspace

dataset_name = 'titanic_ds'

# Get a dataset by name
titanic_ds = Dataset.get_by_name(workspace=workspace, name=dataset_name)

# Load a TabularDataset into pandas DataFrame
df = titanic_ds.to_pandas_dataframe()
```

## <a name="access-source-code-during-training"></a>在定型期間存取原始程式碼

Azure Blob 儲存體的輸送量速度高於 Azure 檔案共用，且會調整為以平行方式啟動的大量作業。 因此，建議您將執行設定為使用 Blob 儲存體來傳輸原始程式碼檔案。

下列程式碼範例會在執行設定中指定要用於原始程式碼傳輸的 Blob 資料存放區。

```python 
# workspaceblobstore is the default blob storage
src.run_config.source_directory_data_store = "workspaceblobstore" 
```

## <a name="notebook-examples"></a>筆記本範例

+ [資料集筆記本](https://aka.ms/dataset-tutorial)會根據本文的概念進行示範和擴充。
+ 瞭解如何 [參數化 ML 管線中的資料集](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-dataset-and-pipelineparameter.ipynb)。

## <a name="troubleshooting"></a>疑難排解

* **資料集初始化失敗：等候掛接點就緒已超時**： 
  * 如果您沒有任何輸出 [網路安全性群組](../virtual-network/network-security-groups-overview.md) 規則，而且正在使用 `azureml-sdk>=1.12.0` 、更新及其相依性為 `azureml-dataset-runtime` 特定次要版本的最新版本，或如果您在執行中使用它，請重新建立環境，讓它可以有最新的修補程式來進行修正。 
  * 如果您使用 `azureml-sdk<1.12.0` ，請升級至最新版本。
  * 如果您有輸出 NSG 規則，請確定有允許服務標籤之所有流量的輸出規則 `AzureResourceMonitor` 。

### <a name="overloaded-azurefile-storage"></a>多載的 AzureFile 儲存體

如果您收到錯誤，請套用下列因應措施 `Unable to upload project files to working directory in AzureFile because the storage is overloaded` 。

如果您針對其他工作負載（例如資料傳輸）使用檔案共用，建議使用 blob，讓檔案共用可用於提交執行。 您也可以在兩個不同的工作區之間分割工作負載。

### <a name="passing-data-as-input"></a>以輸入的形式傳遞資料

*  **TypeError： FileNotFound：沒有這類檔案或目錄**：如果您提供的檔案路徑不是檔案所在的位置，就會發生此錯誤。 您必須確定您參考該檔案的方式，與您在計算目標上裝載資料集的位置一致。 若要確保具決定性狀態，建議您在將資料集裝載至計算目標時使用抽象路徑。 例如，在下列程式碼中，我們會將資料集裝載在計算目標的檔案系統根目錄下 `/tmp` 。 
    
    ```python
    # Note the leading / in '/tmp/dataset'
    script_params = {
        '--data-folder': dset.as_named_input('dogscats_train').as_mount('/tmp/dataset'),
    } 
    ```

    如果您未包含前置正斜線 '/'，則必須在工作目錄前面加上前置詞，例如 `/mnt/batch/.../tmp/dataset` 在計算目標上，以指出您要裝載資料集的位置。


## <a name="next-steps"></a>後續步驟

* 使用 TabularDatasets[自動將機器學習模型定型](how-to-auto-train-remote.md)。

* 使用 FileDatasets 將[影像分類模型定型](https://aka.ms/filedataset-samplenotebook)。

* [使用管線以資料集進行定型](./how-to-create-machine-learning-pipelines.md)。