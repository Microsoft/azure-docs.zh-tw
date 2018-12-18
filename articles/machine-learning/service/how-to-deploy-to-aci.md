---
title: 將 Web 服務部署至 Azure 容器執行個體 (ACI) - Azure Machine Learning
description: 了解如何使用 Azure Machine Learning 服務，將已訓練的模型作為 Web 服務在 Azure 容器執行個體 (ACI) 上部署。 本文說明三種在 ACI 上部署模型的不同方式。 它們程式碼行數各有不同，您在部署的命名部分能獲得的控制程度也有所不同。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: raymondl
author: raymondlaghaeian
ms.reviewer: sgilley
ms.date: 09/24/2018
ms.openlocfilehash: 5a62d4b0b324d8b2536e408132210f07f08e8bb8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958691"
---
# <a name="deploy-web-services-to-azure-container-instances"></a>將 Web 服務部署至 Azure 容器執行個體 

您可以將您已訓練的模型作為 Web 服務，在 [Azure 容器執行個體](https://azure.microsoft.com/services/container-instances/) (ACI)、[Azure Kubernetes Service](https://azure.microsoft.com/services/kubernetes-service/) (AKS)、IoT Edge 裝置，或[現場可程式化閘道陣列 (FPGA)](concept-accelerate-with-fpgas.md) 上進行部署 

ACI 通常比 AKS 便宜，並且使用 4 到 6 行程式碼即可完成設定。 ACI 是測試部署的完美選項。 稍後，當您準備好針對高規格、生產用途使用您的模型及 Web 服務時，您可以[將它們部署至 AKS](how-to-deploy-to-aks.md)。

本文說明三種在 ACI 上部署模型的不同方式。 它們程式碼行數各有不同，您在部署的命名部分能獲得的控制程度也有所不同。 從程式碼數量最少、控制程度最小的方法，到程式碼數量最多、控制程度最大的方法，ACI 選項有：

* 使用 `Webservice.deploy()` 從模型檔案部署 
* 使用 `Webservice.deploy_from_model()` 從已註冊的模型部署
* 使用 `Webservice.deploy_from_image()` 從映像部署已註冊模型

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。


## <a name="prerequisites"></a>必要條件

- 已安裝 Azure Machine Learning 工作區與適用於 Python 的 Azure Machine Learning SDK。 使用[開始使用 Azure Machine Learning 快速入門](quickstart-get-started.md)了解如何取得這些必要條件。

- Azure Machine Learning 工作區物件

    ```python
    from azureml.core import Workspace
    ws = Workspace.from_config()
    ```

- 要部署的模型。 本文件中的範例使用您在遵循[訓練模型](tutorial-train-models-with-aml.md)教學課程時建立的模型。 若您不想使用此模型，請修改這些步驟來參考您的模型模型。  您也需要撰寫您自己的評分指令碼，才能執行您的模型。


## <a name="configure-an-image"></a>設定映像

設定用於儲存所有模型檔案的 Docker 映像。
1. [使用這些指令](tutorial-deploy-models-with-aml.md#create-scoring-script)建立評分指令碼 (score.py)

1. [使用這些指令](tutorial-deploy-models-with-aml.md#create-environment-file)建立環境檔案 (myenv.yml) 

1. 使用這兩個檔案，利用 SDK 在 Python 中設定 Docker 映像，如下所示：

    ```python
    from azureml.core.image import ContainerImage

    image_config = ContainerImage.image_configuration(execution_script = "score.py",
                                                      runtime = "python",
                                                      conda_file = "myenv.yml",
                                                      description = "Image with mnist model",
                                                      tags = {"data": "mnist", "type": "classification"}
                                                     )
    ```

## <a name="configure-the-aci-container"></a>設定 ACI 容器

透過指定 CPU 數量和您 ACI 容器所需的 RAM GB 數，設定 ACI 容器。 針對許多模型，預設的一個核心和 1 GB RAM 便已足夠。 若您稍後需要更多，請重新建立映像並重新部署服務。  

```python
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores = 1, 
                                               memory_gb = 1, 
                                               tags = {"data": "mnist", "type": "classification"},
                                               description = 'Handwriting recognition')
```

## <a name="register-a-model"></a>註冊模型

> 若您是[從模型檔案部署](#deploy-from-model-file) (`Webservice.deploy()`)，請跳過此必要條件。

註冊模型，使用 [`Webservice.deploy_from_model`](#deploy-from-registered-model) 或 [`Webservice.deploy_from_image`](#deploy-from-image)。 或者，若您已有已註冊的模型，請現在擷取它。

### <a name="retrieve-a-registered-model"></a>擷取已註冊的模型
若您使用 Azure Machine Learning 訓練您的模型，模型可能已在您的工作區中註冊。  例如，[訓練模型](tutorial-train-models-with-aml.md)教學課程的最後一個步驟便會註冊模型。  您接著便可以擷取已註冊的模型來進行部署。

```python
from azureml.core.model import Model

model_name = "sklearn_mnist"
model=Model(ws, model_name)
```
  
### <a name="register-a-model-file"></a>註冊模型檔案

若您的模型是在其它地方建置，您仍然可以將它註冊到您的工作區。  若要註冊模型，模型檔案 (此範例為 `sklearn_mnist_model.pkl`) 必須位於目前的工作目錄中。 接著使用 `Model.register()`，在工作區中將該檔案註冊成稱為 `sklearn_mnist` 的模型。
    
```python
from azureml.core.model import Model

model_name = "sklearn_mnist"
model = Model.register(model_path = "sklearn_mnist_model.pkl",
                        model_name = model_name,
                        tags = {"data": "mnist", "type": "classification"},
                        description = "Mnist handwriting recognition",
                        workspace = ws)
```


## <a name="option-1-deploy-from-model-file"></a>選項 1：從模型檔案部署

從模型檔案部署的選項需要撰寫的程式碼數量最少，但針對元件命名所能提供的控制程度也最低。 此選項會從模型檔案開始，並會為您將它註冊到工作區。  但是，您無法命名模型，或是將它與標籤或描述建立關聯。  

此選項使用 SDK 方法，Webservice.deploy()。  

**估計時間**：部署大約需要 6 到 7 分鐘。

1. 請確認模型檔案位於您的本機工作目錄中。

1. 開啟必要條件的模型檔案 (score.py)，將 `init()` 區段變更為：

    ```python
    def init():
        global model
        # retreive the local path to the model using the model name
        model_path = Model.get_model_path('sklearn_mnist_model.pkl')
        model = joblib.load(model_path)
    ```

1. 部署您的模型檔案。

    ```python
    from azureml.core.webservice import Webservice
    
    service_name = 'aci-mnist-1'
    service = Webservice.deploy(deployment_config = aciconfig,
                                    image_config = image_config,
                                    model_paths = ['sklearn_mnist_model.pkl'],
                                    name = service_name,
                                    workspace = ws)
    
    service.wait_for_deployment(show_output = True)
    print(service.state)
    ```

1. 您現在可以[測試 Web 服務](#test-web-service)。

## <a name="option-2-deploy-from-registered-model"></a>選項 2：從已註冊的模型部署

從已註冊模型檔案部署的選項需要的程式碼數較多，但針對輸出命名所能提供的控制程度也較高。 此選項是可讓您方便部署您所擁有的已註冊模型。  但是，您無法命名 Docker 映像。  

此選項使用 SDK 方法，Webservice.deploy_from_model()。

**估計時間**：使用此選項部署大約需要 8 分鐘。

1. 執行程式碼以設定 Docker 容器及 ACI 容器，並指定已註冊模型。

    ```python
    from azureml.core.webservice import Webservice

    service_name = 'aci-mnist-2'
    service = Webservice.deploy_from_model(deployment_config = aciconfig,
                                           image_config = image_config,
                                           models = [model], # this is the registered model object
                                           name = service_name,
                                           workspace = ws)
    service.wait_for_deployment(show_output = True)
    print(service.state)
    ```

1. 您現在可以[測試 Web 服務](#test-web-service)。

## <a name="option-3-deploy-from-image"></a>選項 3：從映像部署

使用 `Webservice.deploy_from_image()` 部署已註冊模型 (`model`)。 此方法可讓您分別建立 Docker 映像，然後從該映像進行部署。

1. 在工作區下使用 `ContainerImage.create()` 建置並註冊 Docker 映像

    此方法可透過在個別步驟中建立映像，讓您對映像有更高的控制能力。  已註冊模型 (`model`) 已包含在映像內。
    
    ```python
    from azureml.core.image import ContainerImage
    
    image = ContainerImage.create(name = "myimage1",
                                  models = [model], # this is the registered model object
                                  image_config = image_config,
                                  workspace = ws)
    
    image.wait_for_creation(show_output = True)
    ```
**估計時間**： 約 3 分鐘。

1. 使用 `Webservice.deploy_from_image()` 將 Docker 映像作為服務部署

    現在，將映像部署至 ACI。  
    
    ```python
    from azureml.core.webservice import Webservice
    
    service_name = 'aci-mnist-3'
    service = Webservice.deploy_from_image(deployment_config = aciconfig,
                                                image = image,
                                                name = service_name,
                                                workspace = ws)
    service.wait_for_deployment(show_output = True)
    print(service.state)
    ```   
 
**估計時間**： 約 3 分鐘。

此方法可讓您針對建立及命名部署中的元件具備最高的控制能力。

您現在可以測試 Web 服務。

## <a name="test-the-web-service"></a>測試 Web 服務

無論使用的方法為何，Web 服務都是相同的。  若要取得預測，請使用服務的 `run` 方法。  

```python
# Load Data
import os
import urllib

os.makedirs('./data', exist_ok = True)

urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz', filename = './data/test-images.gz')

from utils import load_data
X_test = load_data('./data/test-images.gz', False) / 255.0

from sklearn import datasets
import numpy as np
import json

# find 5 random samples from test set
n = 5
sample_indices = np.random.permutation(X_test.shape[0])[0:n]

test_samples = json.dumps({"data": X_test[sample_indices].tolist()})
test_samples = bytes(test_samples, encoding = 'utf8')

# predict using the deployed model
prediction = service.run(input_data = test_samples)
print(prediction)
```


## <a name="clean-up-resources"></a>清除資源

若您不打算使用此 Web 服務，請刪除它，避免產生任何費用。

```python
service.delete()
```

## <a name="next-steps"></a>後續步驟

了解如何針對更大規模的部署[部署至 Azure Kubernetes Service](how-to-deploy-to-aks.md)。 
