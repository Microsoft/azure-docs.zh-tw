---
title: 教學課程：使用自動化的機器學習 - Azure Machine Learning 服務將分類模型定型
description: 了解如何使用自動化機器學習產生機器學習模型。  Azure Machine Learning 可以自動化方式為您執行資料前置處理、演算法選擇和超參數選擇。 接著可以使用 Azure Machine Learning 服務來部署最終模型。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: tutorial
author: nacharya1
ms.author: nilesha
ms.reviewer: sgilley
ms.date: 09/24/2018
ms.openlocfilehash: df1c19c0e16b9862b09dcc652ef2831e0c5bf3a5
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48802350"
---
# <a name="tutorial-train-a-classification-model-with-automated-machine-learning-in-azure-machine-learning-service"></a>教學課程：使用 Azure Machine Learning 服務中的自動化機器學習將分類模型定型

在此教學課程中，您將了解如何使用自動化機器學習產生機器學習模型 (自動化 ML)。  Azure Machine Learning 服務可以自動化方式為您執行資料前置處理、演算法選擇和超參數選擇。 然後可以遵循[部署模型](tutorial-deploy-models-with-aml.md)教學課程中的工作流程來部署最終模型。

![流程圖](./media/tutorial-auto-train-models/flow2.png)

類似於[將模型定型教學課程](tutorial-train-models-with-aml.md)，本教學課程會分類來自 [MNIST](http://yann.lecun.com/exdb/mnist/) 資料集的數字 (0-9) 手寫影像。 但這次不指定演算法，或微調超參數。 自動化 ML 技術會逐一嘗試演算法和超參數的多種組合，直到根據您的準則找到最佳模型為止。

您將學習如何：

> [!div class="checklist"]
> * 設定開發環境
> * 存取並檢查資料
> * 使用您本機電腦上的自動化分類器來定型
> * 探索結果
> * 檢閱定型結果
> * 註冊最佳模型

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="get-the-notebook"></a>取得 Notebook

為了方便起見，此教學課程以 [Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/03.auto-train-models.ipynb) 形式提供。 在 Azure Notebooks 或您自己的 Jupyter Notebook 伺服器中執行 `03.auto-train-models.ipynb` Notebook。


[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-in-azure-notebook.md)]


## <a name="set-up-your-development-environment"></a>設定開發環境

針對您開發工作的所有設定都可以在 Python Notebook 中完成。  設定包括：

* 匯入 Python 套件
* 設定工作區來啟用本機電腦和遠端資源之間的通訊
* 建立目錄來儲存定型指令碼

### <a name="import-packages"></a>匯入套件
匯入本教學課程中所需的 Python 套件。


```python
import azureml.core
import pandas as pd
from azureml.core.workspace import Workspace
from azureml.train.automl.run import AutoMLRun
import time
import logging
from sklearn import datasets
from matplotlib import pyplot as plt
from matplotlib.pyplot import imshow
import random
import numpy as np
```

### <a name="configure-workspace"></a>設定工作區

從現有的工作區建立工作區物件。 `Workspace.from_config()` 會讀取檔案 **aml_config/config.json**，並將詳細資料載入到名為 `ws` 的物件。  `ws` 用於本教學課程的其餘程式碼。

具有工作區物件之後，請指定實驗名稱，並建立本機目錄且向工作區註冊。 所有執行歷程記錄會記錄在指定的實驗下。

```python
ws = Workspace.from_config()
# project folder to save your local files
project_folder = './sample_projects/automl-local-classification'
# choose a name for the run history container in the workspace
experiment_name = 'automl-classifier'

import os

output = {}
output['SDK version'] = azureml.core.VERSION
output['Subscription ID'] = ws.subscription_id
output['Workspace'] = ws.name
output['Resource Group'] = ws.resource_group
output['Location'] = ws.location
output['Project Directory'] = project_folder
pd.set_option('display.max_colwidth', -1)
pd.DataFrame(data=output, index=['']).T
```

## <a name="explore-data"></a>探索資料

初步定型教學課程使用 MNIST 資料集的高解析度版本 (28x28 像素)。  因為自動化 ML 定型需要反覆運算許多次，所以本教學課程使用較低解析度版本的影像 (8x8 像素) 來示範概念，同時加速每次反覆運算所需的時間。  

```python
from sklearn import datasets

digits = datasets.load_digits()

# Exclude the first 100 rows from training so that they can be used for test.
X_train = digits.data[100:,:]
y_train = digits.target[100:]
```

### <a name="display-some-sample-images"></a>顯示一些範例影像

將資料載入到 `numpy` 陣列中。 然後使用 `matplotlib` 來繪製 30 個來自資料集的隨機影像，並且在其上加上標籤。

```python
count = 0
sample_size = 30
plt.figure(figsize = (16, 6))
for i in np.random.permutation(X_train.shape[0])[:sample_size]:
    count = count + 1
    plt.subplot(1, sample_size, count)
    plt.axhline('')
    plt.axvline('')
    plt.text(x = 2, y = -2, s = y_train[i], fontsize = 18)
    plt.imshow(X_train[i].reshape(8, 8), cmap = plt.cm.Greys)
plt.show()
```
影像的隨機範例會顯示：

![數字](./media/tutorial-auto-train-models/digits.png)


您現在已有必要的套件和資料，可以進行模型的自動定型。 

## <a name="train-a-model"></a>將模型定型

若將模型自動定型，請先定義實驗的組態設定，然後執行實驗。


### <a name="define-settings"></a>定義設定

定義實驗設定和模型設定。

|屬性| 本教學課程中的值 |說明|
|----|----|---|
|**primary_metric**|AUC 加權 | 您想要最佳化的度量。|
|**max_time_sec**|12,000|每次反覆運算的時間限制 (秒)|
|**反覆運算次數**|20|反覆運算次數。 在每次反覆運算中，會使用具有特定管線的資料來定型模型|
|**n_cross_validations**|3|交叉驗證分割數目|
|**preprocess**|False| *True/False* 啟用實驗，以對輸入執行前置處理。  前置處理會處理「遺漏的資料」，並執行一些常見的「功能擷取」|
|**exit_score**|0.9985|*雙精度浮點數*值，指出 *primary_metric* 的目標。 一旦超過目標時，會終止執行|
|**blacklist_algos**|['kNN','LinearSVM']|「字串」的「陣列」，指出要略過的演算法。
|

```python
from azureml.train.automl import AutoMLConfig

##Local compute 
Automl_config = AutoMLConfig(task = 'classification',
                             primary_metric = 'AUC_weighted',
                             max_time_sec = 12000,
                             iterations = 20,
                             n_cross_validations = 3,
                             preprocess = False,
                             exit_score = 0.9985,
                             blacklist_algos = ['kNN','LinearSVM'],
                             X = X_train,
                             y = y_train,
                             path=project_folder)
```

### <a name="run-the-experiment"></a>執行實驗

開始在本機執行實驗。 將計算目標定義為本機，並將輸出設定為 true，以檢視實驗進度。


```python
from azureml.core.experiment import Experiment
experiment=Experiment(ws, experiment_name)
local_run = experiment.submit(Automl_config, show_output=True)
```

隨著每次反覆運算的進行，會顯示如下所示的輸出，一次一行。  您會每隔 **10-15 秒**看到新的一行。

    Running locally
    Parent Run ID: AutoML_ca0c807b-b7bf-4809-a963-61c6feb73ea1
    ***********************************************************************************************
    ITERATION: The iteration being evaluated.
    PIPELINE:  A summary description of the pipeline being evaluated.
    DURATION: Time taken for the current iteration.
    METRIC: The result of computing score on the fitted pipeline.
    BEST: The best observed score thus far.
    ***********************************************************************************************
    
     ITERATION     PIPELINE                               DURATION                METRIC      BEST
             0      Normalizer extra trees                0:00:15.955367           0.988     0.988
             1      Normalizer extra trees                0:00:14.203088           0.952     0.988
             2      Normalizer lgbm_classifier            0:00:15.089057           0.994     0.994
             3      Normalizer SGD classifier             0:00:14.866700           0.500     0.994
             4      Normalizer SGD classifier             0:00:13.740577           0.983     0.994
             5      Normalizer DT                         0:00:13.879204           0.937     0.994
             6      Normalizer SGD classifier             0:00:13.379975           0.980     0.994
             7      Normalizer lgbm_classifier            0:00:15.953293           0.997     0.997
    Stopping criteria reached. Ending experiment.

## <a name="explore-the-results"></a>探索結果

使用 Jupyter 小工具，或藉由檢查實驗歷程記錄，探索實驗的結果。

### <a name="jupyter-widget"></a>Jupyter 小工具

使用 Jupyter Notebook 小工具可查看圖形以及所有結果的資料表。

```python
from azureml.train.widgets import RunDetails
RunDetails(local_run).show()
```

以下是小工具的靜態影像。  在 Notebook 中，您可以使用圖表上方的下拉式清單，以針對每次反覆運算檢視每個可用度量的圖表。

![小工具資料表](./media/tutorial-auto-train-models/table.png)
![小工具繪圖](./media/tutorial-auto-train-models/graph.png)

### <a name="retrieve-all-iterations"></a>擷取所有反覆運算

檢視實驗歷程記錄，並查看每次反覆運算執行的個別度量。

```python
children = list(local_run.get_children())
metricslist = {}
for run in children:
    properties = run.get_properties()
    metrics = {k: v for k, v in run.get_metrics().items() if isinstance(v, float)}
    metricslist[int(properties['iteration'])] = metrics

import pandas as pd
rundata = pd.DataFrame(metricslist).sort_index(1)
rundata
```

下表顯示結果：


<!-- hello world -->

<table><thead><tr>
        <th></th>
        <th>0</th>
        <th>1</th>
        <th>2</th>
        <th>3</th>
        <th>4</th>
        <th>5</th>
        <th>6</th>
        <th>7</th>
    </tr></thead>
<tbody>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row0" class="row_heading level0 row0" >AUC_macro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col0" class="data row0 col0" >0.988094</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col1" class="data row0 col1" >0.951981</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col2" class="data row0 col2" >0.993606</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col3" class="data row0 col3" >0.5</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col4" class="data row0 col4" >0.982724</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col5" class="data row0 col5" >0.936998</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col6" class="data row0 col6" >0.979978</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row0_col7" class="data row0 col7" >0.996639</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row1" class="row_heading level0 row1" >AUC_micro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col0" class="data row1 col0" >0.988104</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col1" class="data row1 col1" >0.948402</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col2" class="data row1 col2" >0.99413</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col3" class="data row1 col3" >0.463035</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col4" class="data row1 col4" >0.976078</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col5" class="data row1 col5" >0.945169</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col6" class="data row1 col6" >0.968913</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row1_col7" class="data row1 col7" >0.997027</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row2" class="row_heading level0 row2" >AUC_weighted</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col0" class="data row2 col0" >0.987943</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col1" class="data row2 col1" >0.952255</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col2" class="data row2 col2" >0.993513</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col3" class="data row2 col3" >0.5</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col4" class="data row2 col4" >0.982801</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col5" class="data row2 col5" >0.937292</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col6" class="data row2 col6" >0.979973</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row2_col7" class="data row2 col7" >0.99656</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row3" class="row_heading level0 row3" >AUC_weighted_max</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col0" class="data row3 col0" >0.987943</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col1" class="data row3 col1" >0.987943</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col2" class="data row3 col2" >0.993513</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col3" class="data row3 col3" >0.993513</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col4" class="data row3 col4" >0.993513</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col5" class="data row3 col5" >0.993513</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col6" class="data row3 col6" >0.993513</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row3_col7" class="data row3 col7" >0.99656</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row4" class="row_heading level0 row4" >accuracy</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col0" class="data row4 col0" >0.852093</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col1" class="data row4 col1" >0.666464</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col2" class="data row4 col2" >0.898057</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col3" class="data row4 col3" >0.0701284</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col4" class="data row4 col4" >0.832662</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col5" class="data row4 col5" >0.701827</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col6" class="data row4 col6" >0.83325</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row4_col7" class="data row4 col7" >0.925752</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row5" class="row_heading level0 row5" >average_precision_score_macro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col0" class="data row5 col0" >0.929167</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col1" class="data row5 col1" >0.786258</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col2" class="data row5 col2" >0.961497</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col3" class="data row5 col3" >0.1</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col4" class="data row5 col4" >0.917486</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col5" class="data row5 col5" >0.685547</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col6" class="data row5 col6" >0.906611</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row5_col7" class="data row5 col7" >0.977775</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row6" class="row_heading level0 row6" >average_precision_score_micro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col0" class="data row6 col0" >0.932596</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col1" class="data row6 col1" >0.728331</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col2" class="data row6 col2" >0.964138</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col3" class="data row6 col3" >0.0909031</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col4" class="data row6 col4" >0.880136</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col5" class="data row6 col5" >0.757538</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col6" class="data row6 col6" >0.859813</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row6_col7" class="data row6 col7" >0.980408</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row7" class="row_heading level0 row7" >average_precision_score_weighted</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col0" class="data row7 col0" >0.930681</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col1" class="data row7 col1" >0.788964</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col2" class="data row7 col2" >0.962007</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col3" class="data row7 col3" >0.102123</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col4" class="data row7 col4" >0.918785</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col5" class="data row7 col5" >0.692041</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col6" class="data row7 col6" >0.908293</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row7_col7" class="data row7 col7" >0.977699</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row8" class="row_heading level0 row8" >balanced_accuracy</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col0" class="data row8 col0" >0.917902</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col1" class="data row8 col1" >0.814509</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col2" class="data row8 col2" >0.94491</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col3" class="data row8 col3" >0.5</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col4" class="data row8 col4" >0.909248</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col5" class="data row8 col5" >0.833428</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col6" class="data row8 col6" >0.907412</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row8_col7" class="data row8 col7" >0.959351</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row9" class="row_heading level0 row9" >f1_score_macro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col0" class="data row9 col0" >0.850511</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col1" class="data row9 col1" >0.643116</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col2" class="data row9 col2" >0.899262</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col3" class="data row9 col3" >0.013092</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col4" class="data row9 col4" >0.825054</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col5" class="data row9 col5" >0.691712</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col6" class="data row9 col6" >0.819627</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row9_col7" class="data row9 col7" >0.926081</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row10" class="row_heading level0 row10" >f1_score_micro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col0" class="data row10 col0" >0.852093</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col1" class="data row10 col1" >0.666464</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col2" class="data row10 col2" >0.898057</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col3" class="data row10 col3" >0.0701284</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col4" class="data row10 col4" >0.832662</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col5" class="data row10 col5" >0.701827</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col6" class="data row10 col6" >0.83325</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row10_col7" class="data row10 col7" >0.925752</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row11" class="row_heading level0 row11" >f1_score_weighted</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col0" class="data row11 col0" >0.852134</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col1" class="data row11 col1" >0.646049</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col2" class="data row11 col2" >0.898705</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col3" class="data row11 col3" >0.00933691</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col4" class="data row11 col4" >0.830731</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col5" class="data row11 col5" >0.696538</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col6" class="data row11 col6" >0.824547</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row11_col7" class="data row11 col7" >0.925778</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row12" class="row_heading level0 row12" >log_loss</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col0" class="data row12 col0" >0.554364</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col1" class="data row12 col1" >1.15728</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col2" class="data row12 col2" >0.51741</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col3" class="data row12 col3" >2.30397</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col4" class="data row12 col4" >1.94009</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col5" class="data row12 col5" >1.57663</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col6" class="data row12 col6" >2.1848</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row12_col7" class="data row12 col7" >0.250725</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row13" class="row_heading level0 row13" >norm_macro_recall</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col0" class="data row13 col0" >0.835815</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col1" class="data row13 col1" >0.629003</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col2" class="data row13 col2" >0.890167</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col3" class="data row13 col3" >0</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col4" class="data row13 col4" >0.818755</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col5" class="data row13 col5" >0.666629</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col6" class="data row13 col6" >0.814739</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row13_col7" class="data row13 col7" >0.918851</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row14" class="row_heading level0 row14" >precision_score_macro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col0" class="data row14 col0" >0.855892</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col1" class="data row14 col1" >0.707715</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col2" class="data row14 col2" >0.90195</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col3" class="data row14 col3" >0.00701284</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col4" class="data row14 col4" >0.84882</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col5" class="data row14 col5" >0.729611</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col6" class="data row14 col6" >0.855384</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row14_col7" class="data row14 col7" >0.927881</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row15" class="row_heading level0 row15" >precision_score_micro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col0" class="data row15 col0" >0.852093</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col1" class="data row15 col1" >0.666464</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col2" class="data row15 col2" >0.898057</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col3" class="data row15 col3" >0.0701284</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col4" class="data row15 col4" >0.832662</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col5" class="data row15 col5" >0.701827</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col6" class="data row15 col6" >0.83325</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row15_col7" class="data row15 col7" >0.925752</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row16" class="row_heading level0 row16" >precision_score_weighted</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col0" class="data row16 col0" >0.859204</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col1" class="data row16 col1" >0.711918</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col2" class="data row16 col2" >0.903523</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col3" class="data row16 col3" >0.00500676</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col4" class="data row16 col4" >0.861209</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col5" class="data row16 col5" >0.737586</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col6" class="data row16 col6" >0.863524</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row16_col7" class="data row16 col7" >0.928403</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row17" class="row_heading level0 row17" >recall_score_macro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col0" class="data row17 col0" >0.852234</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col1" class="data row17 col1" >0.666102</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col2" class="data row17 col2" >0.901151</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col3" class="data row17 col3" >0.1</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col4" class="data row17 col4" >0.83688</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col5" class="data row17 col5" >0.699966</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col6" class="data row17 col6" >0.833265</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row17_col7" class="data row17 col7" >0.926966</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row18" class="row_heading level0 row18" >recall_score_micro</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col0" class="data row18 col0" >0.852093</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col1" class="data row18 col1" >0.666464</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col2" class="data row18 col2" >0.898057</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col3" class="data row18 col3" >0.0701284</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col4" class="data row18 col4" >0.832662</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col5" class="data row18 col5" >0.701827</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col6" class="data row18 col6" >0.83325</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row18_col7" class="data row18 col7" >0.925752</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row19" class="row_heading level0 row19" >recall_score_weighted</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col0" class="data row19 col0" >0.852093</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col1" class="data row19 col1" >0.666464</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col2" class="data row19 col2" >0.898057</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col3" class="data row19 col3" >0.0701284</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col4" class="data row19 col4" >0.832662</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col5" class="data row19 col5" >0.701827</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col6" class="data row19 col6" >0.83325</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row19_col7" class="data row19 col7" >0.925752</td> 
    </tr>    <tr> 
        <th id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180level0_row20" class="row_heading level0 row20" >weighted_accuracy</th> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col0" class="data row20 col0" >0.851054</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col1" class="data row20 col1" >0.66639</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col2" class="data row20 col2" >0.895428</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col3" class="data row20 col3" >0.049121</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col4" class="data row20 col4" >0.829247</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col5" class="data row20 col5" >0.702754</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col6" class="data row20 col6" >0.833464</td> 
        <td id="T_32497c5c_a5a9_11e8_a10f_c49ded1c6180row20_col7" class="data row20 col7" >0.924723</td> 
    </tr></tbody> 
</table> 
<!-- hello world -->

## <a name="register-the-best-model"></a>註冊最佳模型 

使用 `local_run` 物件，以取得最佳的模型並將其註冊到工作區。 

```python
# find the run with the highest accuracy value.
best_run, fitted_model = local_run.get_output()

# register model in workspace
description = 'Automated Machine Learning Model'
tags = None
local_run.register_model(description=description, tags=tags)
local_run.model_id # Use this id to deploy the model as a web service in Azure
```

## <a name="test-the-best-model"></a>測試最佳模型

使用模型來預測一些隨機數字。  顯示預測值和影像。  紅色字型及反轉影像 (黑色上的白色) 用於醒目提示分類錯誤的樣本。

因為模型的準確度很高，您可能需要執行下列程式碼數次，才能看到分類錯誤的樣本。

```python
# find 30 random samples from test set
n = 30
X_test = digits.data[:100, :]
y_test = digits.target[:100]
sample_indices = np.random.permutation(X_test.shape[0])[0:n]
test_samples = X_test[sample_indices]


# predict using the  model
result = fitted_model.predict(test_samples)

# compare actual value vs. the predicted values:
i = 0
plt.figure(figsize = (20, 1))

for s in sample_indices:
    plt.subplot(1, n, i + 1)
    plt.axhline('')
    plt.axvline('')
    
    # use different color for misclassified sample
    font_color = 'red' if y_test[s] != result[i] else 'black'
    clr_map = plt.cm.gray if y_test[s] != result[i] else plt.cm.Greys
    
    plt.text(x = 2, y = -2, s = result[i], fontsize = 18, color = font_color)
    plt.imshow(X_test[s].reshape(8, 8), cmap = clr_map)
    
    i = i + 1
plt.show()
```


![results](./media/tutorial-auto-train-models/results.png)

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]


## <a name="next-steps"></a>後續步驟

在本 Azure Machine Learning 服務教學課程中，您使用了 Python 來：

> [!div class="checklist"]
> * 設定開發環境
> * 存取並檢查資料
> * 使用自訂參數在本機以自動化分類器進行定型
> * 探索結果
> * 檢閱定型結果
> * 註冊最佳模型

深入了解[如何設定自動定型的設定]()或[如何對遠端資源使用自動定型](how-to-auto-train-remote.md)。  

