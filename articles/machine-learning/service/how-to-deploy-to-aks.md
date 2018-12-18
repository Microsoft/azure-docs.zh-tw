---
title: 從 Azure Machine Learning 服務將模型部署至 Kubernetes | Microsoft Docs
description: 了解如何從 Azure Machine Learning 服務將模型部署至 Azure Kubernetes Service。 模型會部署為 Web 服務。 Azure Kubernetes Service 非常適合用於大規模生產工作負載。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: raymondl
author: raymondlaghaeian
manager: cgronlun
ms.reviewer: larryfr
ms.date: 09/24/2018
ms.openlocfilehash: f74521f77420fcfc60e99dd3d70574d5e94cf084
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967739"
---
# <a name="how-to-deploy-models-from-azure-machine-learning-service-to-azure-kubernetes-service"></a>如何從 Azure Machine Learning 服務將模型部署至 Azure Kubernetes Service

針對大規模生產案例，您可以將模型部署至 Azure Kubernetes Service (AKS)。 Azure Machine Learning 可以使用現有的 AKS 叢集或在部署期間建立的新叢集。 模型會以 Web 服務的方式部署至 AKS。

部署至 AKS 可為 Web 服務提供自動調整、記錄、模型資料收集，以及快速回應時間等優點。

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

- 已安裝 Azure Machine Learning 工作區、包含您指令碼的本機目錄，以及適用於 Python 的 Azure Machine Learning SDK。 了解如何使用[如何設定開發環境](how-to-configure-environment.md)文件來取得這些必要條件。

- 定型的機器學習模型。 如果您沒有模型，請參閱[將影像分類模型定型](tutorial-train-models-with-aml.md)教學課程。

## <a name="initialize-the-workspace"></a>初始化工作區

若要初始化工作區，請載入包含工作區資訊的 `config.json` 檔案。

```python
from azureml.coreazureml import Workspace

# Load existing workspace from the config file info.
ws = Workspace.from_config()
```

## <a name="register-the-model"></a>註冊模型

若要註冊現有的模型，請指定模型路徑、描述與標記。

```python
from azureml.core.model import Model

model = Model.register(model_path = "model.pkl", # this points to a local file
                        model_name = "best_model", # this is the name the model is registered as
                        tags = {"data": "diabetes", "type": "regression"},
                        description = "Ridge regression model to predict diabetes",
                        workspace = ws)
```

## <a name="create-a-docker-image"></a>建立 Docker 映像

Azure Kubernetes Service 使用 Docker 映像。 若要建立映像，請使用下列步驟：

1. 若要設定映像，必須建立評分指令碼與環境檔案。 如需建立指令碼與環境檔案的範例，請參閱下列映像分類範例小節：

    * [建立評分指令碼 (score.py)](tutorial-deploy-models-with-aml.md#create-scoring-script)

    * [建立環境檔案 (myenv.yml)](tutorial-deploy-models-with-aml.md#create-environment-file) 

   下列範例會使用這些檔案設定映像：

    ```python
    from azureml.core.image import ContainerImage

    # Image configuration
    image_config = ContainerImage.image_configuration(execution_script = "score.py",
                                                        runtime = "python",
                                                        conda_file = "myenv.yml",
                                                        description = "Image with ridge regression model",
                                                        tags = {"data": "diabetes", "type": "regression"}
                                                        )
    ```

1. 若要建立映像，請使用模型與映像設定。 此作業可能需要大約 5 分鐘才能完成：

    ```python
    image = ContainerImage.create(name = "myimage1",
                                    # this is the model object
                                    models = [model],
                                    image_config = image_config,
                                    workspace = ws)

    # Wait for the create process to complete
    image.wait_for_creation(show_output = True)
    ```

## <a name="create-the-aks-cluster"></a>建立 AKS 叢集

下列程式碼片段示範如何建立 AKS 叢集。 此程序大約需要 20 分鐘才能完成：

> [!IMPORTANT]
> 建立 AKS 叢集是工作區的一次性程序。 建立之後，就可以重複使用此叢集進行多個部署。 如果刪除叢集或包含該叢集的資源群組，就必須在下次需要部署時建立新的叢集。


```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this)
prov_config = AksCompute.provisioning_configuration()

aks_name = 'aml-aks-1' 
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws, 
                                    name = aks_name, 
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
print(aks_target.provisioning_state)
print(aks_target.provisioning_errors)
```

### <a name="attach-existing-aks-cluster-optional"></a>附加現有的 AKS 叢集 (選擇性)

如果 Azure 訂用帳戶中有現有的 AKS 叢集，您可以使用它來部署映像。 下列程式碼片段示範如何將叢集附加至您的工作區。 

> [!IMPORTANT]
> 僅支援 AKS 1.8.7 版。

```python
# Get the resource id from https://porta..azure.com -> Find your resource group -> click on the Kubernetes service -> Properties
resource_id = '/subscriptions/<your subscription id>/resourcegroups/<your resource group>/providers/Microsoft.ContainerService/managedClusters/<your aks service name>'

# Set to the name of the cluster
cluster_name='my-existing-aks' 

# Attatch the cluster to your workgroup
aks_target = AksCompute.attach(workspace=ws, name=cluster_name, resource_id=resource_id)

# Wait for the operation to complete
aks_target.wait_for_provisioning(True)
```

## <a name="deploy-your-web-service"></a>部署您的 Web 服務

下列程式碼片段示範如何將映像部署至 AKS 叢集：

```python
from azureml.core.webservice import Webservice, AksWebservice

# Set configuration and service name
aks_config = AksWebservice.deploy_configuration()
aks_service_name ='aks-service-1'
# Deploy from image
aks_service = Webservice.deploy_from_image(workspace = ws, 
                                            name = aks_service_name,
                                            image = image,
                                            deployment_config = aks_config,
                                            deployment_target = aks_target)
# Wait for the deployment to complete
aks_service.wait_for_deployment(show_output = True)
print(aks_service.state)
```

> [!TIP]
> 如果部署期間發生錯誤，請使用 `aks_service.get_logs()` 來檢視 AKS 服務記錄。 記錄資訊可能會指出發生錯誤的原因。

## <a name="test-the-web-service"></a>測試 Web 服務

使用 `aks_service.run()` 來測試 Web 服務。 下列程式碼片段示範如何將資料傳遞至服務並顯示預測：

```python
import json

test_sample = json.dumps({'data': [
    [1,2,3,4,5,6,7,8,9,10], 
    [10,9,8,7,6,5,4,3,2,1]
]})
test_sample = bytes(test_sample,encoding = 'utf8')

prediction = aks_service.run(input_data = test_sample)
print(prediction)
```

## <a name="cleanup"></a>清除

若要刪除服務、映像與模型，請使用下列程式碼片段：

```python
aks_service.delete()
image.delete()
model.delete()
```
