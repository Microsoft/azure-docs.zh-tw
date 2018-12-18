---
title: 教學課程：使用 Azure Machine Learning 服務將影像分類模型定型
description: 本教學課程會示範如何使用 Azure Machine Learning 服務，搭配 scikit-learn 在 Python Jupyter Notebook 中將影像分類模型定型。 本教學課程是兩部分系列的第一部分。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: tutorial
author: hning86
ms.author: haining
ms.reviewer: sgilley
ms.date: 09/24/2018
ms.openlocfilehash: e6e49a03ee76c50cb2fff492bfd50b2820abafe4
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343753"
---
# <a name="tutorial-1-train-an-image-classification-model-with-azure-machine-learning-service"></a>教學課程 #1：使用 Azure Machine Learning 服務將影像分類模型定型

在本教學課程中，您會在本機和遠端計算資源，將機器學習模型定型。 您將使用 Python Jupyter Notebook 中的 Azure Machine Learning 服務 (預覽) 定型和部署工作流程。  然後，您可以使用 Notebook 作為範本，以自己的資料將您自己的機器學習服務模型定型。 本教學課程是**兩部分教學課程系列的第一部分**。  

本教學課程可讓您使用 [MNIST](http://yann.lecun.com/exdb/mnist/) 資料集，並搭配 [scikit-learn](http://scikit-learn.org) 與 Azure Machine Learning 服務定型簡單的羅吉斯迴歸。  MNIST 是熱門的資料集，由 70,000 個灰階影像所組成。 每個影像是 28x28 個像素的手寫數字，代表 0 至 9 的數字。 目標是要建立多類別分類器，來識別特定影像代表的數字。 

了解如何：

> [!div class="checklist"]
> * 設定開發環境
> * 存取並檢查資料
> * 使用熱門的 scikit-learn 機器學習程式庫來定型簡單的羅吉斯迴歸 
> * 在遠端叢集定型多個模型
> * 檢閱定型結果，並註冊最佳的模型

您稍後將在[本教學課程的第二部分](tutorial-deploy-models-with-aml.md)了解如何選取模型並部署它。 

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="get-the-notebook"></a>取得 Notebook

為了方便起見，此教學課程以 [Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/01.train-models.ipynb) 形式提供。 在 Azure Notebooks 或您自己的 Jupyter Notebook 伺服器中執行 `01.train-models.ipynb` Notebook。

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-in-azure-notebook.md)]

>[!NOTE]
> 本教學課程使用 Azure Machine Learning SDK 0.168 版進行測試 

## <a name="set-up-your-development-environment"></a>設定開發環境

針對您開發工作的所有設定都可以在 Python Notebook 中完成。  設定包括：

* 匯入 Python 套件
* 連線到工作區，來啟用本機電腦和遠端資源之間的通訊
* 建立實驗，以追蹤所有執行
* 建立要用於定型的遠端計算目標

### <a name="import-packages"></a>匯入套件

匯入在本工作階段所需的 Python 套件。 也會顯示 Azure Machine Learning SDK 版本。

```python
%matplotlib inline
import numpy as np
import matplotlib
import matplotlib.pyplot as plt

import azureml
from azureml.core import Workspace, Run

# check core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

### <a name="connect-to-workspace"></a>連線到工作區

從現有的工作區建立工作區物件。 `Workspace.from_config()` 會讀取檔案 **config.json**，並將詳細資料載入到名為 `ws` 的物件。

```python
# load workspace configuration from the config.json file in the current folder.
ws = Workspace.from_config()
print(ws.name, ws.location, ws.resource_group, ws.location, sep = '\t')
```

### <a name="create-experiment"></a>建立實驗

建立實驗，以追蹤您工作區中的執行。 工作區可以有多個實驗。 

```python
experiment_name = 'sklearn-mnist'

from azureml.core import Experiment
exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="create-remote-compute-target"></a>建立遠端計算目標

Azure Batch AI 是一項受控服務，可讓資料科學家在 Azure 虛擬機器 (包括具有 GPU 支援的 VM) 叢集上定型機器學習模型。  在本教學課程中，您可以建立 Azure Batch AI 叢集作為定型環境。 此程式碼會為您建立叢集，如果它尚未存在於您的工作區。 

 **建立叢集需要大約 5 分鐘的時間。** 如果叢集已在工作區中，此程式碼會使用它，並略過建立程序。


```python
from azureml.core.compute import ComputeTarget, BatchAiCompute
from azureml.core.compute_target import ComputeTargetException

# choose a name for your cluster
batchai_cluster_name = "traincluster"

try:
    # look for the existing cluster by name
    compute_target = ComputeTarget(workspace=ws, name=batchai_cluster_name)
    if type(compute_target) is BatchAiCompute:
        print('found compute target {}, just use it.'.format(batchai_cluster_name))
    else:
        print('{} exists but it is not a Batch AI cluster. Please choose a different name.'.format(batchai_cluster_name))
except ComputeTargetException:
    print('creating a new compute target...')
    compute_config = BatchAiCompute.provisioning_configuration(vm_size="STANDARD_D2_V2", # small CPU-based VM
                                                                #vm_priority='lowpriority', # optional
                                                                autoscale_enabled=True,
                                                                cluster_min_nodes=0, 
                                                                cluster_max_nodes=4)

    # create the cluster
    compute_target = ComputeTarget.create(ws, batchai_cluster_name, compute_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it uses the scale settings for the cluster
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
    # Use the 'status' property to get a detailed status for the current cluster. 
    print(compute_target.status.serialize())
```

您現在具有必要的套件和計算資源，可以在雲端將模型定型。 

## <a name="explore-data"></a>探索資料

將模型定型之前，您需要了解用來定型的資料。  您也需要將資料複製到雲端，以便您的雲端定型環境可以存取它。  在本節中，您將了解如何：

* 下載 MNIST 資料集
* 顯示一些範例影像
* 將資料上傳至雲端

### <a name="download-the-mnist-dataset"></a>下載 MNIST 資料集

下載 MNIST 資料集，並將檔案儲存到 `data` 本機目錄。  會下載定型和測試用的影像與標籤。  


```python
import os
import urllib.request

os.makedirs('./data', exist_ok = True)

urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz', filename='./data/train-images.gz')
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz', filename='./data/train-labels.gz')
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz', filename='./data/test-images.gz')
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz', filename='./data/test-labels.gz')
```

### <a name="display-some-sample-images"></a>顯示一些範例影像

將壓縮的檔案載入到 `numpy` 陣列。 然後使用 `matplotlib` 來繪製 30 個來自資料集的隨機影像，並且在其上加上標籤。 請注意，此步驟需要 `load_data` 函式，它包含在 `util.py` 檔案中。 這個檔案包含在範例資料夾。 請確定將它放在與此 Notebook 相同的資料夾。 `load_data` 函式會將壓縮檔案剖析為 numpy 陣列。



```python
# make sure utils.py is in the same directory as this code
from utils import load_data

# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the model converge faster.
X_train = load_data('./data/train-images.gz', False) / 255.0
y_train = load_data('./data/train-labels.gz', True).reshape(-1)

X_test = load_data('./data/test-images.gz', False) / 255.0
y_test = load_data('./data/test-labels.gz', True).reshape(-1)

# now let's show some randomly chosen images from the traininng set.
count = 0
sample_size = 30
plt.figure(figsize = (16, 6))
for i in np.random.permutation(X_train.shape[0])[:sample_size]:
    count = count + 1
    plt.subplot(1, sample_size, count)
    plt.axhline('')
    plt.axvline('')
    plt.text(x=10, y=-10, s=y_train[i], fontsize=18)
    plt.imshow(X_train[i].reshape(28, 28), cmap=plt.cm.Greys)
plt.show()
```

影像的隨機範例會顯示：

![影像的隨機範例](./media/tutorial-train-models-with-aml/digits.png)

現在您已了解這些影像外觀與預期的預測結果。

### <a name="upload-data-to-the-cloud"></a>將資料上傳至雲端

現在，請將您本機電腦中的資料上傳至 Azure 讓資料可從遠端存取，以便能存取資料進行遠端定型。 資料存放區是與您工作區建立關聯的方便建構，讓您可以上傳/下載資料，並從遠端計算目標與其互動。 它受到 Azure Blob 儲存體帳戶的支援。

MNIST 檔案會上傳到資料存放區根目錄中，名為 `mnist` 的目錄。

```python
ds = ws.get_default_datastore()
print(ds.datastore_type, ds.account_name, ds.container_name)

ds.upload(src_dir='./data', target_path='mnist', overwrite=True, show_progress=True)
```
您現在已經具備開始將模型定型所需的一切。 

## <a name="train-a-model-locally"></a>在本機將模型定型

從 scikit-learn 在本機定型最簡單的羅吉斯迴歸模型。

**在本機定型，可能需要一或兩分鐘**，視您的電腦設定而定。

```python
%%time
from sklearn.linear_model import LogisticRegression

clf = LogisticRegression()
clf.fit(X_train, y_train)
```

接下來，使用測試集進行預測，並計算精確度。 

```python
y_hat = clf.predict(X_test)
print(np.average(y_hat == y_test))
```

本機模型精確度會顯示：

`0.9202`

只要短短幾行程式碼，您便有 92% 的精確度。

## <a name="train-on-a-remote-cluster"></a>在遠端叢集上定型

現在您可以建置具有不同正規化速率的模型，展開這個簡單的模型。 這次您將在遠端資源上將模型定型。  

針對這項工作，將工作提交至您稍早設定的遠端定型叢集。  若要提交工作，您要：
* 建立目錄
* 建立定型指令碼
* 建立估計工具
* 提交工作 

### <a name="create-a-directory"></a>建立目錄

建立目錄，以從您的電腦傳遞必要程式碼到遠端資源。

```python
import os
script_folder = './sklearn-mnist'
os.makedirs(script_folder, exist_ok=True)
```

### <a name="create-a-training-script"></a>建立定型指令碼

若要將工作提交到叢集，請先建立定型指令碼。 執行下列程式碼，在您剛建立的目錄中建立定型指令碼，稱為 `train.py`。 此定型會為定型演算法新增正規化速率，因此會產生與本機版本稍有不同的模型。

```python
%%writefile $script_folder/train.py

import argparse
import os
import numpy as np

from sklearn.linear_model import LogisticRegression
from sklearn.externals import joblib

from azureml.core import Run
from utils import load_data

# let user feed in 2 parameters, the location of the data files (from datastore), and the regularization rate of the logistic regression model
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = os.path.join(args.data_folder, 'mnist')
print('Data folder:', data_folder)

# load train and test set into numpy arrays
# note we scale the pixel intensity values to 0-1 (by dividing it with 255.0) so the model can converge faster.
X_train = load_data(os.path.join(data_folder, 'train-images.gz'), False) / 255.0
X_test = load_data(os.path.join(data_folder, 'test-images.gz'), False) / 255.0
y_train = load_data(os.path.join(data_folder, 'train-labels.gz'), True).reshape(-1)
y_test = load_data(os.path.join(data_folder, 'test-labels.gz'), True).reshape(-1)
print(X_train.shape, y_train.shape, X_test.shape, y_test.shape, sep = '\n')

# get hold of the current run
run = Run.get_context()

print('Train a logistic regression model with regularizaion rate of', args.reg)
clf = LogisticRegression(C=1.0/args.reg, random_state=42)
clf.fit(X_train, y_train)

print('Predict the test set')
y_hat = clf.predict(X_test)

# calculate accuracy on the prediction
acc = np.average(y_hat == y_test)
print('Accuracy is', acc)

run.log('regularization rate', np.float(args.reg))
run.log('accuracy', np.float(acc))

os.makedirs('outputs', exist_ok=True)
# note file saved in the outputs folder is automatically uploaded into experiment record
joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')
```

請注意指令碼取得資料並儲存模型的方式：

+ 定型指令碼會讀取引數，以尋找包含資料的目錄。  當您稍後提交工作時，您會指向這個引數的資料存放區：`parser.add_argument('--data-folder', type = str, dest = 'data_folder', help = 'data directory mounting point')`

    
+ 定型指令碼會將模型儲存到名為 outputs 的目錄。 <br/>
`joblib.dump(value = clf, filename = 'outputs/sklearn_mnist_model.pkl')`<br/>
寫入此目錄之任何項目會自動上傳到您的工作區。 稍後在本教學課程中，您會從這個目錄存取您的模型。

會從定型指令碼參考檔案 `utils.py`，以便正確載入資料集。  請將這個指令碼複製到指令碼資料夾中，以便可以存取它，以及遠端資源上的定型指令碼。


```python
import shutil
shutil.copy('utils.py', script_folder)
```


### <a name="create-an-estimator"></a>建立估計工具

估計工具物件用來提交執行。  執行下列程式碼來建立您的估計工具，以定義：

* 估計工具物件的名稱，`est`
* 包含指令碼的目錄。 在此目錄中的所有檔案都會上傳到叢集節點以便執行。 
* 計算目標。  在此情況下，您將使用您所建立的 Batch AI 叢集
* 定型指令碼名稱，train.py
* 來自定型指令碼的必要參數 
* 定型所需的 Python 套件

在本教學課程中，這個目標會是 Batch AI 叢集。 在專案目錄中的所有檔案都會上傳到叢集節點以便執行。 data_folder 會設定為使用資料存放區 (`ds.as_mount()`)。

```python
from azureml.train.estimator import Estimator

script_params = {
    '--data-folder': ds.as_mount(),
    '--regularization': 0.8
}

est = Estimator(source_directory=script_folder,
                script_params=script_params,
                compute_target=compute_target,
                entry_script='train.py',
                conda_packages=['scikit-learn'])
```


### <a name="submit-the-job-to-the-cluster"></a>將工作提交至叢集

提交估計工具物件以執行實驗。

```python
run = exp.submit(config=est)
run
```

由於呼叫是非同步的，只要工作啟動，它就會傳回**準備**或**執行**狀態。

## <a name="monitor-a-remote-run"></a>監視遠端執行

第一次執行總計會花費**大約 10 分鐘**。 但針對後續的執行，只要指令碼相依性不變，便會重複使用相同的影像，因此容器啟動時間會快許多。

以下是在您等候時，會發生的事：

- **影像建立**：會建立一個符合估計工具所指定 Python 環境的 Docker 影像。 影像會上傳到工作區。 影像建立和上傳會花費**大約 5 分鐘**。 

  這個階段會針對每個 Python 環境發生一次，因為容器會被快取以供後續執行之用。  在影像建立期間，會將記錄串流至執行歷程記錄中。 您可以使用這些記錄檔監視影像建立程序。

- **調整**：如果遠端叢集需要更多的節點，以執行比目前可用數更多的執行，則會自動新增其他節點。 調整通常會花費**大約 5 分鐘。**

- **執行**：在這個階段，會將必要的指令碼與檔案傳送到計算目標，然後掛接/複製資料存放區，再來則會執行 entry_script。 當執行工作時，stdout 和 ./logs 目錄會串流至執行歷程記錄。 您可以使用這些記錄檔監視執行的進度。

- **後置處理**：執行的 ./outputs 目錄會複製到您工作區中的執行歷程記錄，因此您能存取這些結果。


您可以用多種方式檢查執行工作的進度。 本教學課程使用 Jupyter 小工具以及 `wait_for_completion` 方法。 

### <a name="jupyter-widget"></a>Jupyter 小工具

使用 Jupyter 小工具觀看執行的進度。  就像執行提交一樣，小工具為非同步工作，並每隔 10 至 15 秒提供即時更新，直到工作完成為止。


```python
from azureml.train.widgets import RunDetails
RunDetails(run).show()
```

以下是在定型結束時，顯示的小工具靜止快照集：

![Notebook 小工具](./media/tutorial-train-models-with-aml/widget.png)

### <a name="get-log-results-upon-completion"></a>在完成時取得記錄檔結果

模型定型和監視會在背景中發生。 請等到模型完成定型，再執行更多程式碼。 使用 `wait_for_completion` 可顯示模型定型何時完成。 


```python
run.wait_for_completion(show_output=False) # specify True for a verbose log
```

### <a name="display-run-results"></a>顯示執行結果

您現在在遠端叢集上已有定型的模型。  擷取模型的精確度：

```python
print(run.get_metrics())
```
輸出顯示遠端模型的精確度稍微高於本機模型，因為在定型期間新增了正規化速率。  

`{'regularization rate': 0.8, 'accuracy': 0.9204}`

在部署教學課程中，您將更詳細地探索這個模型。

## <a name="register-model"></a>註冊模型

定型指令碼的最後一個步驟，會寫入檔案 `outputs/sklearn_mnist_model.pkl`，它位於工作執行所在叢集的 VM 中，名為 `outputs` 的目錄。 `outputs` 目錄的特殊之處，在於此目錄中的所有內容會自動上傳至您的工作區。  此內容會出現在您工作區下，實驗的執行記錄中。 因此，模型檔案現在也會提供於您的工作區中。

您可以看到與該執行建立關聯的檔案。

```python
print(run.get_file_names())
```

請在工作區中註冊模型，以便您 (或其他共同作業者) 稍後可以查詢、檢查和部署此模型。

```python
# register model 
model = run.register_model(model_name='sklearn_mnist', model_path='outputs/sklearn_mnist_model.pkl')
print(model.name, model.id, model.version, sep = '\t')
```

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

您也可以刪除 Azure 受控計算叢集。 不過，因為自動調整規模已開啟，且叢集最小值為 0，所以這個特定資源在不使用時將不會產生額外的計算費用。


```python
# optionally, delete the Azure Managed Compute cluster
compute_target.delete()
```

## <a name="next-steps"></a>後續步驟

在本 Azure Machine Learning 服務教學課程中，您使用了 Python 來：

> [!div class="checklist"]
> * 設定開發環境
> * 存取並檢查資料
> * 使用熱門的 scikit-learn 機器學習程式庫來定型簡單的羅吉斯迴歸
> * 在遠端叢集定型多個模型
> * 檢閱定型詳細資料，並註冊最佳的模型

您已準備好，使用教學課程系列下一個部分中的指示，來部署這個已註冊的模型：

> [!div class="nextstepaction"]
> [教學課程 2 - 部署模型](tutorial-deploy-models-with-aml.md)
