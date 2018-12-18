---
title: 為自動化機器學習服務設定遠端計算目標 - Azure Machine Learning 服務
description: 此文章說明如何搭配 Azure Machine Learning 服務在資料科學虛擬機器 (DSVM) 遠端計算目標上，使用自動化機器學習服務來建置模型
services: machine-learning
author: nacharya1
ms.author: nilesha
ms.reviewer: sgilley
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 2ec0dea7e50747f8af337874c8f12463cecb8df7
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47163472"
---
# <a name="train-models-with-automated-machine-learning-in-the-cloud"></a>使用雲端中的自動化機器學習來將模型定型

在 Azure Machine Learning 中，您可以在所管理的不同類型計算資源上針對模型定型。 計算目標可以是本機電腦或在雲端中的電腦。

您可以輕鬆地相應增加或相應放大機器學習實驗，方法是新增其他計算目標，例如以 Ubuntu 為基礎的資料科學虛擬機器 (DSVM) 或 Azure Batch AI。 DSVM 是 Microsoft Azure 雲端上的自訂 VM 映像，專為進行資料科學建置。 它已預先安裝和預先設定許多熱門的資料科學和其他工具。  

在此文章中，您會了解如何在 DSVM 上使用自動化 ML 服務來建置模型。 您可以在 [GitHub 中的這些範例 Notebook](https://aka.ms/aml-notebooks) 中找到使用 Azure Batch AI 的範例。  

## <a name="how-does-remote-differ-from-local"></a>遠端與本機有何不同？

[使用自動化機器學習服務將分類模型定型](tutorial-auto-train-models.md)教學課程將教導您如何使用本機電腦，使用自動化 ML 來將模型定型。  在本機定型時的工作流程也適用於遠端目標。 不過，使用遠端計算時，自動化 ML 實驗反覆項目會以非同步方式執行。 這可讓您取消特定反覆項目、監看執行狀態，或繼續處理 Jupyter Notebook 中的其他資料格。 若要從遠端進行定型，請先建立遠端計算目標 (例如 Azure DSVM)。  接著，設定遠端資源，並在該處提交您的程式碼。

此文章說明在遠端 DSVM 上執行自動化 ML 實驗所需的額外步驟。  以下程式碼會使用來自教學課程的工作區物件 `ws`。

```python
ws = Workspace.from_config()
```

## <a name="create-resource"></a>建立資源

如果 DSVM 尚未存在，請在工作區 (`ws`) 中加以建立。 如果先前已建立 DSVM，此程式碼會略過建立程序，並將現有的資源詳細資料載入到 `dsvm_compute` 物件。  

**估計時間**：建立 VM 需要大約 5 分鐘的時間。

```python
from azureml.core.compute import DsvmCompute

dsvm_name = 'mydsvm' #Name your DSVM
try:
    dsvm_compute = DsvmCompute(ws, dsvm_name)
    print('found existing dsvm.')
except:
    print('creating new dsvm.')
    # Below is using a VM of SKU Standard_D2_v2 which is 2 core machine. You can check Azure virtual machines documentation for additional SKUs of VMs.
    dsvm_config = DsvmCompute.provisioning_configuration(vm_size = "Standard_D2_v2")
    dsvm_compute = DsvmCompute.create(ws, name = dsvm_name, provisioning_configuration = dsvm_config)
    dsvm_compute.wait_for_completion(show_output = True)
```

您現在可以使用 `dsvm_compute` 物件作為遠端計算目標。

DSVM 名稱限制包括：
+ 必須少於 64 個字元。  
+ 不得包含下列任一字元：`\` ~ ! @ # $ % ^ & * ( ) = + _ [ ] { } \\\\ | ; : \' \\" , < > / ?.`

>[!Warning]
>如果建立失敗並顯示有關 Marketplace 購買資格的訊息：
>    1. 移至 [Azure 入口網站](https://portal.azure.com)
>    1. 開始建立 DSVM 
>    1. 選取 [想要以程式設計方式建立]，啟用以程式設計方式建立的功能
>    1. 結束而不實際建立 VM
>    1. 重新執行建立程式碼

此程式碼不會為佈建的 DSVM 建立使用者名稱或密碼。 如果您想要直接連線到 VM，請移至 [Azure 入口網站](https://portal.azure.com)來佈建認證。  


## <a name="access-data-using-getdata-file"></a>使用 get_data 檔案存取資料

提供定型資料的遠端資源存取。 針對在遠端計算上執行的自動化機器學習實驗，資料必須使用 `get_data()` 函式來擷取。  

若要提供存取，您必須：
+ 建立包含 `get_data()` 函式的 get_data.py 檔案 
* 將該檔案放在包含指令碼的資料夾根目錄中 

您可以在 get_data.py 檔案中封裝用來從 Blob 儲存體或本機磁碟讀取資料的程式碼。 在下列程式碼範例中，資料來自 sklearn 套件。

>[!Warning]
>如果您要使用遠端計算，則必須使用 `get_data()`，在該處執行資料轉換。 如果您需要為 get_data() 的資料轉換過程安裝額外的程式庫，請遵循其他所需步驟。 如需詳細資訊，請參閱 [auto-ml-dataprep 範例 Notebook](https://aka.ms/aml-auto-ml-data-prep )。


```python
# Create a project_folder if it doesn't exist
if not os.path.exists(project_folder):
    os.makedirs(project_folder)

#Write the get_data file.
%%writefile $project_folder/get_data.py

from sklearn import datasets
from scipy import sparse
import numpy as np

def get_data():
    
    digits = datasets.load_digits()
    X_digits = digits.data[10:,:]
    y_digits = digits.target[10:]

    return { "X" : X_digits, "y" : y_digits }
```

## <a name="configure-experiment"></a>設定實驗

指定 `AutoMLConfig` 的設定。  (查看[完整參數清單]()及其可能值。)

在這些設定中，`run_configuration` 會設定為 `run_config` 物件，其中包含 DSVM 的設定和組態。  

```python
from azureml.train.automl import AutoMLConfig
import time
import logging

automl_settings = {
    "name": "AutoML_Demo_Experiment_{0}".format(time.time()),
    "max_time_sec": 600,
    "iterations": 20,
    "n_cross_validations": 5,
    "primary_metric": 'AUC_weighted',
    "preprocess": False,
    "concurrent_iterations": 10,
    "verbosity": logging.INFO
}

automl_config = AutoMLConfig(task='classification',
                             debug_log='automl_errors.log',
                             path=project_folder,
                             compute_target = dsvm_compute,
                             data_script=project_folder + "/get_data.py",
                             **automl_settings
                            )
```

## <a name="submit-training-experiment"></a>提交定型實驗

現在提交設定以自動選取演算法、超參數，並針對模型定型。 (了解 `submit` 方法的[設定詳細資訊]()。)

```python
from azureml.core.experiment import Experiment
experiment=Experiment(ws, 'automl_remote')
remote_run = experiment.submit(automl_config, show_output=True)
```
您將看到類似以下的輸出：

    Running on remote compute: mydsvmParent Run ID: AutoML_015ffe76-c331-406d-9bfd-0fd42d8ab7f6
    ***********************************************************************************************
    ITERATION: The iteration being evaluated.
    PIPELINE:  A summary description of the pipeline being evaluated.
    DURATION: Time taken for the current iteration.
    METRIC: The result of computing score on the fitted pipeline.
    BEST: The best observed score thus far.
    ***********************************************************************************************
    
     ITERATION     PIPELINE                               DURATION                METRIC      BEST
             2      Standardize SGD classifier            0.0                      0.954     0.954
             7      Normalizer DT                         0.0                      0.161     0.954
             0      Scale MaxAbs 1 extra trees            0.0                      0.936     0.954
             4      Robust Scaler SGD classifier          0.0                      0.867     0.954
             1      Normalizer kNN                        0.0                      0.984     0.984
             9      Normalizer extra trees                0.0                      0.834     0.984
             5      Robust Scaler DT                      0.0                      0.736     0.984
             8      Standardize kNN                       0.0                      0.981     0.984
             6      Standardize SVM                       2.2                      0.984     0.984
            10      Scale MaxAbs 1 DT                     0.0                      0.077     0.984
            11      Standardize SGD classifier            0.0                      0.863     0.984
             3      Standardize gradient boosting         5.4                      0.971     0.984
            12      Robust Scaler logistic regression     2.0                      0.955     0.984
            14      Scale MaxAbs 1 SVM                    0.0                      0.989     0.989
            13      Scale MaxAbs 1 gradient boosting      3.4                      0.971     0.989
            15      Robust Scaler kNN                     0.0                      0.904     0.989
            17      Standardize kNN                       0.0                      0.974     0.989
            16      Scale 0/1 gradient boosting           2.8                      0.968     0.989
            18      Scale 0/1 extra trees                 0.0                      0.828     0.989
            19      Robust Scaler kNN                     0.0                      0.983     0.989


## <a name="explore-results"></a>瀏覽結果

您可以使用與[定型教學課程](tutorial-auto-train-models.md#explore-the-results)中相同的 Jupyter 小工具，來查看結果的圖形和資料表。

```python
from azureml.train.widgets import RunDetails
RunDetails(remote_run).show()
```
以下是小工具的靜態影像。  在筆記本中，您可以按一下資料表中的任一行以查看執行的屬性，以及該次執行的輸出記錄檔。   您也可以使用圖表上方的下拉式清單，以針對每個反覆項目檢視每個可用度量的圖表。

![小工具資料表](./media/how-to-auto-train-remote/table.png)
![小工具繪圖](./media/how-to-auto-train-remote/plot.png)

小工具會顯示 URL，您可以使用它來查看並瀏覽個別執行的詳細資料。
 
### <a name="view-logs"></a>檢視記錄檔

請在 DSVM 的 /tmp/azureml_run/{iterationid}/azureml-logs 下尋找記錄檔。

## <a name="example"></a>範例

`automl/03.auto-ml-remote-execution.ipynb` 筆記本會示範本文中的概念。  取得此筆記本：

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>後續步驟

了解[如何設定自動定型的設定](how-to-configure-auto-train.md)。