---
title: 適用於 Azure Machine Learning 硬體加速的 FPGA 套件
description: 了解適用於 Azure Machine Learning 使用者的 Python 套件。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: reference
ms.reviewer: jmartens
ms.author: tedway
author: tedway
ms.date: 05/07/2018
ROBOTS: NOINDEX
ms.openlocfilehash: cb1abdce3bbd7349695ece70ff336c7e513c0918
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162180"
---
# <a name="azure-machine-learning-hardware-acceleration-package"></a>Azure Machine Learning 硬體加速套件

>[!Note]
>**此文章已過時。** 已淘汰此 FPGA 套件。 Azure ML SDK 已新增此功能的支援。 此套件的支援將會逐漸結束。 [檢視支援時間表](overview-what-happened-to-workbench.md#timeline)。 了解已更新的 [FPGA 支援](concept-accelerate-with-fpgas.md)。

Azure Machine Learning 硬體加速套件是適用於 Azure Machine Learning 的 Python 可安裝 pip 的延伸模組，可讓資料科學家和 AI 開發人員快速：

+ 利用量化版本的 ResNet 50 將影像功能化

+ 根據這些功能為分類器定型

+ 將模型部署至 Azure 上的[現場可程式化閘陣列 (FPGA)](concept-accelerate-with-fpgas.md)，以達到超低延遲推斷

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

1. Azure Machine Learning 模型管理帳戶。 如需有關建立帳戶的詳細資訊，請參閱 [Azure Machine Learning 快速入門和 Workbench 安裝](../desktop-workbench/quickstart-installation.md)文件。 

1. 您必須安裝此套件。 

 
## <a name="how-to-install-the-package"></a>如何安裝套件

1. 下載並安裝最新版的 [Git](https://git-scm.com/downloads)。

2. 安裝 [Anaconda (Python 3.6)](https://conda.io/miniconda.html)

   若要下載預先設定的 Anaconda 環境，請從 Git 提示字元使用下列命令：

    ```
    git clone https://aka.ms/aml-real-time-ai
    ```
1. 若要建立環境，請開啟 **Anaconda 提示字元**，並使用下列命令：

    ```
    conda env create -f aml-real-time-ai/environment.yml
    ```

1. 若要啟用環境，請使用下列命令：

    ```
    conda activate amlrealtimeai
    ```

## <a name="sample-code"></a>範例程式碼

此範例程式碼會引導您使用 SDK，將模型部署至 FPGA。

1. 匯入套件：
   ```python
   import amlrealtimeai
   from amlrealtimeai import resnet50
   ```

1. 前置處理影像：
   ```python 
   from amlrealtimeai.resnet50.model import LocalQuantizedResNet50
   model_path = os.path.expanduser('~/models')
   model = LocalQuantizedResNet50(model_path)
   print(model.version)
   ```

1. 將影像功能化：
   ```python 
   from amlrealtimeai.resnet50.model import LocalQuantizedResNet50
   model_path = os.path.expanduser('~/models')
   model = LocalQuantizedResNet50(model_path)
   print(model.version)
   ```

1. 建立分類器：
   ```python
   model.import_graph_def(include_featurizer=False)
   print(model.classifier_input)
   print(model.classifier_output)
   ```

1. 建立服務定義：
   ```python
   from amlrealtimeai.pipeline import ServiceDefinition, TensorflowStage, BrainWaveStage
   save_path = os.path.expanduser('~/models/save')
   service_def_path = os.path.join(save_path, 'service_def.zip')

   service_def = ServiceDefinition()
   service_def.pipeline.append(TensorflowStage(tf.Session(), in_images, image_tensors))
   service_def.pipeline.append(BrainWaveStage(model))
   service_def.pipeline.append(TensorflowStage(tf.Session(), model.classifier_input, model.classifier_output))
   service_def.save(service_def_path)
   print(service_def_path)
   ```
 
1. 準備要在 FPGA 上執行的模型：
   ```python
   from amlrealtimeai import DeploymentClient

   subscription_id = "<Your Azure Subscription ID>"
   resource_group = "<Your Azure Resource Group Name>"
   model_management_account = "<Your AzureML Model Management Account Name>"

   model_name = "resnet50-model"
   service_name = "quickstart-service"

   deployment_client = DeploymentClient(subscription_id, resource_group, model_management_account)
   ```

1. 部署要在 FPGA 上執行的模型：
   ```python
   service = deployment_client.get_service_by_name(service_name)
   model_id = deployment_client.register_model(model_name, service_def_path)

   if(service is None):
      service = deployment_client.create_service(service_name, model_id)    
   else:
      service = deployment_client.update_service(service.id, model_id)
   ```

1. 建立用戶端：
    ```python
   from amlrealtimeai import PredictionClient
   client = PredictionClient(service.ipAddress, service.port)  
   ```

1. 呼叫 API：
   ```python
   image_file = R'C:\path_to_file\image.jpg'
   results = client.score_image(image_file)
   ```

## <a name="reporting-issues"></a>回報問題

使用[論壇](https://aka.ms/aml-forum)回報您遇到的任何套件相關問題。

## <a name="next-steps"></a>後續步驟

[在 FPGA 上將模型部署為 Web 服務](how-to-deploy-fpga-web-service.md)