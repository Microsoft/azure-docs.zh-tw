---
title: 教學課程：使用 Azure Machine Learning 服務在 Azure 容器執行個體 (ACI) 中部署影像分類模型
description: 本教學課程會示範如何使用 Azure Machine Learning 服務，搭配 scikit-learn 在 Python Jupyter Notebook 中部署影像分類模型。  本教學課程是兩部分系列的第二部分。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: tutorial
author: hning86
ms.author: haining
ms.reviewer: sgilley
ms.date: 09/24/2018
ms.openlocfilehash: b1ae3fb5de3e28f44ef39731f49a33e2086255a1
ms.sourcegitcommit: 4edf9354a00bb63082c3b844b979165b64f46286
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/04/2018
ms.locfileid: "48785336"
---
# <a name="tutorial-2--deploy-an-image-classification-model-in-azure-container-instance-aci"></a>教學課程 #2：在 Azure 容器執行個體 (ACI) 中部署影像分類模型

本教學課程是**兩部分教學課程系列的第二部分**。 在[先前的教學課程](tutorial-train-models-with-aml.md)中，您定型了機器學習模型，並在您的雲端工作區內註冊模型。  

現在，您已準備好將模型作為 Web 服務，於 [Azure 容器執行個體](https://docs.microsoft.com/azure/container-instances/) (ACI) 中進行部署。 Web 服務是一種封裝評分邏輯和模型本身的映像 (在此案例中為 Docker 映像)。 

在教學課程的這一部分中，您會使用 Azure Machine Learning 服務 (預覽) 來：

> [!div class="checklist"]
> * 設定您的測試環境
> * 從您的工作區擷取模型
> * 於本機測試模型
> * 將模型部署至 ACI
> * 測試已部署的模型

ACI 不適合用於生產環境部署，但非常適合用來測試及了解工作流程。 如需可調整的生產環境部署，請考慮使用 [Azure Kubernetes Service](how-to-deploy-to-aks.md)。

## <a name="get-the-notebook"></a>取得 Notebook

為了方便起見，此教學課程以 [Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/02.deploy-models.ipynb) 形式提供。 在 Azure Notebooks 或您自己的 Jupyter Notebook 伺服器中執行 `02.deploy-models.ipynb` Notebook。

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-in-azure-notebook.md)]


## <a name="prerequisites"></a>必要條件

完成[教學課程 #1：使用 Azure Machine Learning 服務定型影像分類模型](tutorial-train-models-with-aml.md) Notebook中的模型定型。  


## <a name="set-up-the-environment"></a>設定 Azure 環境

著手開始設定測試環境。

### <a name="import-packages"></a>匯入套件

匯入本教學課程所需的 Python 套件。

```python
%matplotlib inline
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
 
import azureml
from azureml.core import Workspace, Run

# display the core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

### <a name="retrieve-the-model"></a>擷取模型

您已在先前教學課程中的工作區內註冊模型。 現在，請載入此工作區，並將模型下載至您的本機目錄。


```python
from azureml.core import Workspace
from azureml.core.model import Model

ws = Workspace.from_config()
model=Model(ws, 'sklearn_mnist')
model.download(target_dir = '.')
import os 
# verify the downloaded model file
os.stat('./sklearn_mnist_model.pkl')
```

## <a name="test-model-locally"></a>於本機測試模型

在部署之前，請透過以下方式確定您的模型可在本機正常運作：
* 載入測試資料
* 預測測試資料
* 檢查混淆矩陣

### <a name="load-test-data"></a>載入測試資料

從定型教學課程中建立的 **./data/** 目錄載入測試資料。

```python
from utils import load_data

# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the neural network converge faster

X_test = load_data('./data/test-images.gz', False) / 255.0
y_test = load_data('./data/test-labels.gz', True).reshape(-1)

```

### <a name="predict-test-data"></a>預測測試資料

饋送測試資料集給模型，以取得預測。

```python
import pickle
from sklearn.externals import joblib

clf = joblib.load('./sklearn_mnist_model.pkl')
y_hat = clf.predict(X_test)
```

###  <a name="examine-the-confusion-matrix"></a>檢查混淆矩陣

產生混淆矩陣，查看測試集中有多少範例的分類正確。 請注意不正確預測中的錯誤分類值。 

```python
from sklearn.metrics import confusion_matrix

conf_mx = confusion_matrix(y_test, y_hat)
print(conf_mx)
print('Overall accuracy:', np.average(y_hat == y_test))
```

輸出會顯示混淆矩陣：

    [[ 960    0    1    2    1    5    6    3    1    1]
     [   0 1112    3    1    0    1    5    1   12    0]
     [   9    8  920   20   10    4   10   11   37    3]
     [   4    0   17  921    2   21    4   12   20    9]
     [   1    2    5    3  915    0   10    2    6   38]
     [  10    2    0   41   10  770   17    7   28    7]
     [   9    3    7    2    6   20  907    1    3    0]
     [   2    7   22    5    8    1    1  950    5   27]
     [  10   15    5   21   15   27    7   11  851   12]
     [   7    8    2   13   32   13    0   24   12  898]]
    Overall accuracy: 0.9204
   

使用 `matplotlib`，將混淆矩陣作為圖表顯示。 在此圖表中，X 軸代表實際值，Y 軸代表預測值。 每一格中的顏色代表錯誤率。 顏色越淡，表示錯誤率越高。 例如，很多 5 都錯誤分類為 3。 因此，您會在 (5,3) 看到明亮的方格。

```python
# normalize the diagnal cells so that they don't overpower the rest of the cells when visualized
row_sums = conf_mx.sum(axis=1, keepdims=True)
norm_conf_mx = conf_mx / row_sums
np.fill_diagonal(norm_conf_mx, 0)

fig = plt.figure(figsize=(8,5))
ax = fig.add_subplot(111)
cax = ax.matshow(norm_conf_mx, cmap=plt.cm.bone)
ticks = np.arange(0, 10, 1)
ax.set_xticks(ticks)
ax.set_yticks(ticks)
ax.set_xticklabels(ticks)
ax.set_yticklabels(ticks)
fig.colorbar(cax)
plt.ylabel('true labels', fontsize=14)
plt.xlabel('predicted values', fontsize=14)
plt.savefig('conf.png')
plt.show()
```

![混淆矩陣](./media/tutorial-deploy-models-with-aml/confusion.png)

## <a name="deploy-as-web-service"></a>作為 Web 服務部署

當您測試完模型，並且對結果感到滿意之後，便可將模型作為 ACI 中的託管 Web 服務進行部署。 

若要建置正確的 ACI 環境，請提供下列項目：
* 示範如何使用模型的評分指令碼
* 顯示需要安裝哪些套件的環境檔案
* 用於建置 ACI 的設定檔
* 您之前定型的模型

<a name="make-script"></a>

### <a name="create-scoring-script"></a>建立評分指令碼

建立稱為 score.py 的評分指令碼。Web 服務呼叫會使用此指令碼示範如何使用模型。

您必須在評分指令碼中包含兩個必要函式：
* `init()` 函式，通常會將模型載入全域物件。 此函式只會在 Docker 容器啟動時執行一次。 

* `run(input_data)` 函式會使用模型依據輸入資料預測值。 run 的輸入和輸出通常會使用 JSON 進行序列化及還原序列化，但也支援其他格式。

```python
%%writefile score.py
import json
import numpy as np
import os
import pickle
from sklearn.externals import joblib
from sklearn.linear_model import LogisticRegression

from azureml.core.model import Model

def init():
    global model
    # retreive the path to the model file using the model name
    model_path = Model.get_model_path('sklearn_mnist')
    model = joblib.load(model_path)

def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # make prediction
    y_hat = model.predict(data)
    return json.dumps(y_hat.tolist())
```

<a name="make-myenv"></a>

### <a name="create-environment-file"></a>建立環境檔案

接下來，建立稱為 myenv.yml 的環境檔案，指定指令碼的所有套件相依性。 此檔案會用於確保 Docker 映像上已安裝所有相依性。 此模型需要 `scikit-learn` 和 `azureml-sdk`。

```python
from azureml.core.conda_dependencies import CondaDependencies 

myenv = CondaDependencies()
myenv.add_conda_package("scikit-learn")

with open("myenv.yml","w") as f:
    f.write(myenv.serialize_to_string())
```
檢閱 `myenv.yml` 檔案的內容。

```python
with open("myenv.yml","r") as f:
    print(f.read())
```

### <a name="create-configuration-file"></a>建立組態檔

建立部署設定檔，並指定您 ACI 容器所需要的 CPU 數量及 RAM GB 數。 雖然這取決於您的模型，但預設的 1 核心及 1 GB RAM 通常對許多模型來說便已足夠。 若您稍後需要更多，您需要重新建立映像並重新部署服務。

```python
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1, 
                                               memory_gb=1, 
                                               tags={"data": "MNIST",  "method" : "sklearn"}, 
                                               description='Predict MNIST with sklearn')
```

### <a name="deploy-in-aci"></a>在 ACI 中部署
**預估完成時間：7 到 8 分鐘**

設定映像並部署。 下列程式碼會執行這些步驟：

1. 使用以下程式碼建置映像：
   * 評分檔案 (`score.py`)
   * 環境檔案 (`myenv.yml`)
   * 模型檔案
1. 在工作區下註冊該映像。 
1. 將映像傳送至 ACI 容器。
1. 使用映像在 ACI 中啟動容器。
1. 取得 Web 服務 HTTP 端點。


```python
%%time
from azureml.core.webservice import Webservice
from azureml.core.image import ContainerImage

# configure the image
image_config = ContainerImage.image_configuration(execution_script="score.py", 
                                                  runtime="python", 
                                                  conda_file="myenv.yml")

service = Webservice.deploy_from_model(workspace=ws,
                                       name='sklearn-mnist-svc',
                                       deployment_config=aciconfig,
                                       models=[model],
                                       image_config=image_config)

service.wait_for_deployment(show_output=True)
```

取得評分 Web 服務的 HTTP 端點，該端點會接受 REST 用戶端呼叫。 此端點可和任何想要測試 Web 服務，或想要整合應用程式與服務的人共用。 

```python
print(service.scoring_uri)
```


## <a name="test-deployed-service"></a>測試已部署的服務

先前您已使用模型的本機版本，對所有測試資料進行評分。 現在，您可以使用來自測試資料的 30 個影像隨機樣本測試已部署模型。  

下列程式碼會執行這些步驟：
1. 將資料作為 JSON 陣列傳送至 ACI 中的託管 Web 服務。 

1. 使用 SDK 的 `run` API 叫用服務。 您也可以使用 HTTP 工具 (例如 curl) 進行原始呼叫。

1. 印出傳回的預測，並將它們與輸入影像繪製在一起。 紅色字型及反轉影像 (黑色上的白色) 用於醒目提示分類錯誤的樣本。 

 因為模型的準確度很高，您可能需要執行下列程式碼數次，才能看到分類錯誤的樣本。

```python
import json

# find 30 random samples from test set
n = 30
sample_indices = np.random.permutation(X_test.shape[0])[0:n]

test_samples = json.dumps({"data": X_test[sample_indices].tolist()})
test_samples = bytes(test_samples, encoding = 'utf8')

# predict using the deployed model
result = json.loads(service.run(input_data=test_samples))

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
    
    plt.text(x=10, y =-10, s=result[i], fontsize=18, color=font_color)
    plt.imshow(X_test[s].reshape(28, 28), cmap=clr_map)
    
    i = i + 1
plt.show()
```

此為其中一個測試影像隨機樣本的結果：![結果](./media/tutorial-deploy-models-with-aml/results.png)

您也可以傳送原始 HTTP 要求來測試 Web 服務。

```python
import requests
import json

# send a random row from the test set to score
random_index = np.random.randint(0, len(X_test)-1)
input_data = "{\"data\": [" + str(list(X_test[random_index])) + "]}"

headers = {'Content-Type':'application/json'}

# for AKS deployment you'd need to the service key in the header as well
# api_key = service.get_key()
# headers = {'Content-Type':'application/json',  'Authorization':('Bearer '+ api_key)} 

resp = requests.post(service.scoring_uri, input_data, headers=headers)

print("POST to url", service.scoring_uri)
#print("input data:", input_data)
print("label:", y_test[random_index])
print("prediction:", resp.text)
```

## <a name="clean-up-resources"></a>清除資源

若要保留資源群組及工作區，以用於其他教學課程和探索，您可以使用此 API 呼叫，只刪除 ACI 部署：

```python
service.delete()
```

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]


## <a name="next-steps"></a>後續步驟

在本 Azure Machine Learning 服務教學課程中，您使用了 Python 來：

> [!div class="checklist"]
> * 設定您的測試環境
> * 從您的工作區擷取模型
> * 於本機測試模型
> * 將模型部署至 ACI
> * 測試已部署的模型
 
您也可以試試[自動選取演算法]()教學課程，了解 Azure Machine Learning 服務為您模型自動選取及調整最佳演算法，並為您建置該模型的方式。
