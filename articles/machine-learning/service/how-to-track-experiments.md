---
title: 追蹤實驗與訓練計量 - Azure Machine Learning | Microsoft Docs
description: 使用 Azure Machine Learning 服務，您可以追蹤您的實驗並監視計量，以加強模型建立程序。 了解如何將記錄新增至您的訓練指令碼、如何提交實驗、如何檢查執行中作業的進度，以及如何檢視執行結果。
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/24/2018
ms.openlocfilehash: ced10a54d569531b06ee47b646130f43cedd2963
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984597"
---
# <a name="track-experiments-and-training-metrics-in-azure-machine-learning"></a>使用 Azure Machine Learning 追蹤實驗與訓練計量

在 Azure Machine Learning 服務中，您可以追蹤您的實驗並監視計量，以加強模型建立程序。 在本文中，您將了解將記錄新增到訓練指令碼、如何使用 **start_logging** 和 **ScriptRunConfig** 提交實驗、如何檢查執行中作業的進度，以及如何檢視執行結果的不同方法。 

## <a name="list-of-training-metrics"></a>訓練計量的清單 

訓練實驗時，可以將下列計量新增到執行中。 若要檢視執行時可追蹤內容的更詳細清單，請參閱 [SDK 參考文件](https://docs.microsoft.com/python/api/overview/azure/azure-ml-sdk-overview?view=azure-ml-py)。

|類型| Python 函式 | 注意|
|----|:----:|:----:|
|純量值 | `run.log(name, value, description='')`| 使用指定名稱將計量值記錄到執行中。 將計量記錄到執行中，會導致該計量儲存在實驗的執行記錄中。  您可以在執行中多次記錄相同的計量，結果會視為該計量的向量。|
|清單| `run.log_list(name, value, description='')`|使用指定名稱將清單計量值記錄到執行中。|
|資料列| `run.log_row(name, description=None, **kwargs)`|使用 *log_row* 建立資料表計量，其中包含 kwargs 中描述的資料行。 每個具名的參數都會產生一個具有指定值的資料行。  可以呼叫一次 *log_row* 以記錄任意 Tuple，或者在迴圈中多次呼叫以產生完整的資料表。|
|資料表| `run.log_table(name, value, description='')`| 使用指定名稱將資料表計量記錄到執行中。 |
|映像| `run.log_image(name, path=None, plot=None)`|記錄映像計量以執行記錄。 使用 log_image 將映像檔案或 matplotlib 繪圖記錄到執行中。  這些映像會顯示在執行記錄中，並可供比較。|
|標記執行| `run.tag(key, value=None)`|使用字串索引鍵和可選字串值標記執行。|
|上傳檔案或目錄|`run.upload_file(name, path_or_stream)`|將檔案上傳到執行記錄。 執行會自動擷取特定輸出目錄中的檔案，對於大多數執行類型，預設為「./outputs」。  只有在需要上傳其他檔案或未指定輸出目錄時，才使用 upload_file。 我們建議在名稱中加上 `outputs`，以便將其上傳到輸出目錄。 您可以透過呼叫 `run.get_file_names()`，列出與該執行記錄相關聯的所有檔案|

> [!NOTE]
> 純量、清單、資料列和資料表的計量可具有以下類型：浮點數、整數或字串。

## <a name="log-metrics-for-experiments"></a>記錄實驗的計量

如果您想要追蹤或監視您的實驗，必須新增程式碼，以在提交執行時開始記錄。 以下是觸發執行提交的方法：
* __Run.start_logging__ - 將記錄函式加入您的定型指令碼中，並在指定的實驗中，啟動互動式記錄工作階段。 **start_logging** 可建立互動式執行，以便用於 Notebook 等環境中。 工作階段期間記錄的所有計量都會加入實驗的執行記錄中。
* __ScriptRunConfig__ - 將記錄函式加入定型指令碼中，並在執行時載入整個指令碼資料夾。  **ScriptRunConfig** 是一種用於設定指令碼執行設定的類別。 使用此選項，您可以新增監視程式碼以通知完成，或取得視覺小工具以進行監視。

## <a name="set-up-the-workspace-and-experiment"></a>設定工作區與實驗
新增記錄並提交實驗之前，您必須設定工作區與實驗。

1. 使用工作區。 若要深入了解設定工作區設定，請依照[快速入門](https://docs.microsoft.com/azure/machine-learning/service/quickstart-get-started)進行操作。

  ```python
  from azureml.core import Workspace, Run
  import azureml.core
  
  ws = Workspace(workspace_name = <<workspace_name>>,
               subscription_id = <<subscription_id>>,
               resource_group = <<resource_group>>)
   ```

2. 建立實驗。

  ```python
  from azureml.core import Experiment

  # make up an arbitrary name
  experiment_name = 'train-in-notebook'
  exp = Experiment(workspace_object = ws, name = experiment_name)
  ```
  
## <a name="option-1-use-startlogging"></a>選項 1：使用 start_logging

**start_logging** 可建立互動式執行，以便用於 Notebook 等環境中。 工作階段期間記錄的所有計量都會加入實驗的執行記錄中。

以下範例在本機 Jupyter Notebook 中，將簡單的 sklearn Ridge 模型定型。 若要深入了解將實驗提交至不同的環境，請參閱[使用 Azure Machine Learning 服務設定模型定型的計算目標](https://docs.microsoft.com/azure/machine-learning/service/how-to-set-up-training-targets)。

1. 在本機 Jupyter Notebook 中建立定型指令碼。 

  ``` python
  # load diabetes dataset, a well-known small dataset that comes with scikit-learn
  from sklearn.datasets import load_diabetes
  from sklearn.linear_model import Ridge
  from sklearn.metrics import mean_squared_error
  from sklearn.model_selection import train_test_split
  from sklearn.externals import joblib

  X, y = load_diabetes(return_X_y = True)
  columns = ['age', 'gender', 'bmi', 'bp', 's1', 's2', 's3', 's4', 's5', 's6']
  X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.2, random_state = 0)
  data = {
      "train":{"X": X_train, "y": y_train},        
      "test":{"X": X_test, "y": y_test}
  }
  reg = Ridge(alpha = 0.03)
  reg.fit(data['train']['X'], data['train']['y'])
  preds = reg.predict(data['test']['X'])
  print('Mean Squared Error is', mean_squared_error(preds, data['test']['y']))
  joblib.dump(value = reg, filename = 'model.pkl');
  ```

2. 使用 Azure Machine Learning 服務 SDK 新增實驗定型，並將保存的模型上傳至實驗執行記錄。 以下程式碼新增標記、記錄，並將模型檔案上傳到實驗執行中。

  ```python
  run = Run.start_logging(experiment = exp)
  run.tag("Description","My first run!")
  run.log('alpha', 0.03)
  reg = Ridge(alpha = 0.03)
  reg.fit(data['train']['X'], data['train']['y'])
  preds = reg.predict(data['test']['X'])
  run.log('mse', mean_squared_error(preds, data['test']['y']))
  joblib.dump(value = reg, filename = 'model.pkl')
  # Upload file directly to the outputs folder
  run.upload_file(name = 'outputs/model.pkl', path_or_stream = './model.pkl')

  run.complete()
  ```

指令碼以 ```run.complete()``` 結尾，完成時會標記執行。  這通常用於互動式 Notebook 環境。

## <a name="option-2-use-scriptrunconfig"></a>選項 2：使用 ScriptRunConfig

**ScriptRunConfig** 是一種用於設定指令碼執行設定的類別。 使用此選項，您可以新增監視程式碼以通知完成，或取得視覺小工具以進行監視。

此範例擴展上述的基本 sklearn Ridge 模型。 它執行簡單的參數掃掠以掃掠模型的 Alpha 值，以在實驗下的執行中擷取計量和定型的模型。 此範例會針對使用者管理的環境在本機上執行。 

1. 建立定型指令碼。 這使用 ```%%writefile%%```，將定型程式碼寫出指令碼資料夾作為 ```train.py```。

  ```python
  %%writefile $project_folder/train.py

  import os
  from sklearn.datasets import load_diabetes
  from sklearn.linear_model import Ridge
  from sklearn.metrics import mean_squared_error
  from sklearn.model_selection import train_test_split
  from azureml.core.run import Run
  from sklearn.externals import joblib

  import numpy as np

  #os.makedirs('./outputs', exist_ok = True)

  X, y = load_diabetes(return_X_y = True)

  run = Run.get_submitted_run()

  X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.2, random_state = 0)
  data = {"train": {"X": X_train, "y": y_train},
          "test": {"X": X_test, "y": y_test}}

  # list of numbers from 0.0 to 1.0 with a 0.05 interval
  alphas = np.arange(0.0, 1.0, 0.05)

  for alpha in alphas:
      # Use Ridge algorithm to create a regression model
      reg = Ridge(alpha = alpha)
      reg.fit(data["train"]["X"], data["train"]["y"])

      preds = reg.predict(data["test"]["X"])
      mse = mean_squared_error(preds, data["test"]["y"])
      # log the alpha and mse values
      run.log('alpha', alpha)
      run.log('mse', mse)

      model_file_name = 'ridge_{0:.2f}.pkl'.format(alpha)
      # save model in the outputs folder so it automatically get uploaded
      with open(model_file_name, "wb") as file:
          joblib.dump(value = reg, filename = model_file_name)

      # upload the model file explicitly into artifacts 
      run.upload_file(name = model_file_name, path_or_stream = model_file_name)

      # register the model
      #run.register_model(file_name = model_file_name)

      print('alpha is {0:.2f}, and mse is {1:0.2f}'.format(alpha, mse))
  
  ```

2. ```train.py``` 指令碼參考 ```mylib.py```。 此檔案可讓您取得要在 ridge 模型中使用的 Alpha 值清單。

  ```python
  %%writefile $script_folder/mylib.py
  import numpy as np

  def get_alphas():
      # list of numbers from 0.0 to 1.0 with a 0.05 interval
      return np.arange(0.0, 1.0, 0.05)
  ```

3. 設定使用者管理的本機環境。

  ```python
  from azureml.core.runconfig import RunConfiguration

  # Editing a run configuration property on-fly.
  run_config_user_managed = RunConfiguration()

  run_config_user_managed.environment.python.user_managed_dependencies = True

  # You can choose a specific Python environment by pointing to a Python path 
  #run_config.environment.python.interpreter_path = '/home/user/miniconda3/envs/sdk2/bin/python'
  ```

4. 提交要在使用者管理環境中執行的 ```train.py``` 指令碼。 提交整個指令碼資料夾進行定型，包括 ```mylib.py``` 檔案。

  ```python
  from azureml.core import ScriptRunConfig

  src = ScriptRunConfig(source_directory = script_folder, script = 'train.py', run_config = run_config_user_managed)
  run = exp.submit(src)
  ```
  
## <a name="view-run-details"></a>檢視執行詳細資料

### <a name="monitor-run-with-jupyter-notebook-widgets"></a>使用 Jupyter Notebook 小工具執行監視
當您使用 **ScriptRunConfig** 方法以提交執行時，您可以查看使用 Jupyter Notebook 小工具執行的進度。 就像執行提交一樣，小工具為非同步工作，並每隔 10 至 15 秒提供即時更新，直到工作完成為止。

1. 等待執行完成時，檢視 Jupyter 小工具。

  ```python
  from azureml.train.widgets import RunDetails
  RunDetails(run).show()
  ```

  ![Jupyter Notebook 小工具的螢幕擷取畫面](./media/how-to-track-experiments/widgets.PNG)

### <a name="get-log-results-upon-completion"></a>在完成時取得記錄檔結果

模型定型和監視在背景進行，以便您可以在等待時執行其他工作。 您也可以等到模型完成定型，再執行更多程式碼。 當您使用 **ScriptRunConfig** 時，可以使用 ```run.wait_for_completion(show_output = True)``` 以顯示模型定型完成的時間。 ```show_output``` 旗標為您提供詳細資訊輸出。 
  
### <a name="query-run-metrics"></a>查詢執行計量

您可以使用 ```run.get_metrics()``` 檢視定型模型的計量。 您現在可以取得上述範例中記錄的所有計量，以確定最佳模型。

## <a name="view-the-experiment-in-the-azure-portal"></a>在 Azure 入口網站中檢視實驗

當實驗完成執行時，您可以瀏覽記錄的實驗執行記錄。 您可以使用兩種方式執行此動作：

* 直接取得執行的 URL ```print(run.get_portal_url())```
* 藉由提交執行名稱 (在此情況下為 ```run```)，以檢視執行詳細資料。 這會指向實驗名稱、識別碼、類型、狀態、詳細資料頁面、Azure 入口網站的連結，以及文件連結。

執行連結將帶您直接前往 Azure 入口網站中的執行詳細資料頁面。 在這裡，您可以看到實驗中記錄的任何屬性、追蹤計量、映像和圖表。 在此情況下，我們會記錄 MSE 和 Alpha 值。

  ![Azure 入口網站執行詳細資料的螢幕擷取畫面](./media/how-to-track-experiments/run-details-page-web.PNG)

您也可以檢視執行的任何輸出或記錄，或下載讓您提交的實驗快照集，以便與其他人共用實驗資料夾。

## <a name="example-notebooks"></a>Notebook 範例
下列 Notebook 示範了此文章中說明的概念：
* `01.getting-started/01.train-within-notebook/01.train-within-notebook.ipynb`
* `01.getting-started/02.train-on-local/02.train-on-local.ipynb`

取得這些 Notebook：[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>後續步驟

請嘗試下列步驟，了解如何使用適用於 Python 的 Azure Machine Learning SDK：

* 關於如何註冊最佳模型，並在教學課程中加以部署的範例，請參閱[使用 Azure Machine Learning 定型映像分類模型](tutorial-train-models-with-aml.md)。

* 了解如何[使用 Azure Machine Learning 定型 PyTorch 模型](how-to-train-pytorch.md)。
