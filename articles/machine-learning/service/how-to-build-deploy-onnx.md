---
title: ONNX 和 Azure Machine Learning | 建立及部署模型
description: 深入了解 ONNX 以及如何使用 Azure Machine Learning 來建立及部署 ONNX 模型
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: prasantp
author: prasanthpul
ms.date: 09/24/2018
ms.openlocfilehash: d4ce2dc67b0d9229ac2605ab317594ea345c19b2
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434067"
---
# <a name="onnx-and-azure-machine-learning-create-and-deploy-interoperable-ai-models"></a>ONNX 和 Azure Machine Learning：建立及部署可交互作用的 AI 模型

[開放神經網路交換](http://onnx.ai) (ONNX) 格式是代表機器學習模型的開放標準。 ONNX 受到[合作夥伴社群](http://onnx.ai/supported-tools)的支援，例如 Microsoft 建立了相容的架構和工具。 Microsoft 致力於開發開放且可交互作用的 AI，讓資料科學家和開發人員可以：

+ 使用自己所選擇的架構來建立模型並加以定型
+ 以最少的整合工作在各個平台之間部署模型

Microsoft 會讓其產品 (包括 Azure 和 Windows) 支援 ONNX，以協助您達成這些目標。  

## <a name="why-choose-onnx"></a>為何選擇 ONNX？
您透過 ONNX 所取得的互通性，可以讓絕佳的想法更快付諸實行。 使用了 ONNX，資料科學家可以選擇他們喜好的架構來進行作業。 同樣地，開發人員可以耗費較少的時間讓模型準備好用於生產環境，並且跨雲端與邊線進行部署。  

您可以從許多架構匯出 ONNX 模型，這些架構包括 PyTorch、Chainer、Microsoft Cognitive Toolkit (CNTK)、MXNet 和 ML.Net。 其他架構 (例如 TensorFlow、Keras、SciKit-Learn 等等) 有轉換器可用。

另外還有工具的生態系統，可用來視覺化及加速 ONNX 模型。 許多預先定型的 ONNX 模型也適用於常見案例。

您可以使用 Azure Machine Learning 和 ONNX Runtime 將 [ONNX 模型可以部署](#deploy)到雲端。 也可以使用 [Windows ML](https://docs.microsoft.com/windows/ai/)，將其部署到 Windows 10 裝置。 甚至可以使用從 ONNX 社群取得的轉換器，將其部署到其他平台。 

[ ![顯示定型、轉換和部署的 ONNX 流程圖](media/concept-onnx/onnx.png) ] (./media/concept-onnx/onnx.png#lightbox)

## <a name="create-onnx-models-in-azure"></a>在 Azure 中建立 ONNX 模型

您可以透過數種方式來建立 ONNX 模型：
+ 在 Azure Machine Learning 服務中將模型定型，並且將其轉換或匯出為 ONNX (請參閱本文底部的範例)

+ 從 [ONNX Model Zoo](https://github.com/onnx/models) 取得預先定型的 ONNX 模型

+ 從 [Azure 自訂視覺服務](https://docs.microsoft.com/azure/cognitive-services/Custom-Vision-Service/)產生自訂 ONNX 模型

## <a name="exportconvert-your-models-to-onnx"></a>將您的模型匯出/轉換為 ONNX

您也可以將現有模型轉換為 ONNX。

|模型的架構|轉換範例或工具|
|-----|-------|
|PyTorch|[Jupyter Notebook](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb)|
|Microsoft&nbsp;Cognitive&nbsp;Toolkit&nbsp;(CNTK)|[Jupyter Notebook](https://github.com/onnx/tutorials/blob/master/tutorials/CntkOnnxExport.ipynb)|
|TensorFlow|[tensorflow-onnx 轉換器](https://github.com/onnx/tensorflow-onnx)|
|Chainer|[Jupyter Notebook](https://github.com/onnx/tutorials/blob/master/tutorials/ChainerOnnxExport.ipynb)|
|MXNet|[Jupyter Notebook](https://github.com/onnx/tutorials/blob/master/tutorials/MXNetONNXExport.ipynb)|
|Keras、ScitKit-Learn、CoreML<br/>XGBoost 和 libSVM|[WinMLTools](https://docs.microsoft.com/windows/ai/convert-model-winmltools)|

您可以在 [ONNX 教學課程網站](https://github.com/onnx/tutorials)找到支援架構和轉換器的最新清單。

<a name="deploy"></a>

## <a name="deploy-onnx-models-in-azure"></a>在 Azure 中部署 ONNX 模型

使用 Azure Machine Learning 服務，您可以部署、管理及監視 ONNX 模型。 使用標準[部署工作流程](concept-model-management-and-deployment.md)和 ONNX Runtime，您可以建立裝載於雲端中的 REST 端點。 請參閱本文結尾的 Jupyter Notebook 完整範例，自行嘗試看看。 

### <a name="install-and-configure-the-onnx-runtime"></a>安裝和設定 ONNX Runtime

ONNX Runtime 是適用於 ONNX 模型的高效能推斷引擎。 其隨附 Python API，並提供 CPU 和 GPU 兩者的硬體加速。 目前支援 ONNX 1.2 模型，並且在 Ubuntu 16.04 Linux 上執行。 [CPU](https://pypi.org/project/onnxruntime) 和 [GPU](https://pypi.org/project/onnxruntime-gpu) 套件都可於 [PyPi.org](https://pypi.org) 上取得。

若要安裝 ONNX Runtime，請使用：
```python
pip install onnxruntime
```

若要在您的 Python 指令碼中呼叫 ONNX Runtime，請使用：
```python
import onnxruntime

session = onnxruntime.InferenceSession("path to model")
```

模型隨附的文件通常會告訴您使用模型所需的輸入和輸出。 您也可以使用視覺效果工具 (例如 [Netron](https://github.com/lutzroeder/Netron)) 來檢視模型。 ONNX Runtime 也可讓您查詢模型中繼資料、輸入和輸出：
```python
session.get_modelmeta()
first_input_name = session.get_inputs()[0].name
first_output_name = session.get_outputs()[0].name
```

若要推斷您的模型，請使用 `run`，傳入您想要系統傳回的輸出清單 (如果您想要傳回全部則留空)，以及傳入輸入值的對應。 結果是輸出清單。
```python
results = session.run(["output1", "output2"], {"input1": indata1, "input2": indata2})
results = session.run([], {"input1": indata1, "input2": indata2})
```

如需完整的 API 參考，請參閱 [ONNX Runtime 參考文件](https://aka.ms/onnxruntime-python)。

### <a name="example-deployment-steps"></a>部署步驟範例

以下是部署 ONNX 模型的範例：

1. 初始化您的 Azure Machine Learning 工作區。 如果您還沒有工作區，請在[本快速入門](quickstart-get-started.md)中深入了解如何建立工作區。

   ```python
   from azureml.core import Workspace

   ws = Workspace.from_config()
   print(ws.name, ws.resource_group, ws.location, ws.subscription_id, sep = '\n')
   ```

2. 向 Azure Machine Learning 註冊模型。

   ```python
   from azureml.core.model import Model

   model = Model.register(model_path = "model.onnx",
                          model_name = "MyONNXmodel",
                          tags = ["onnx"],
                          description = "test",
                          workspace = ws)
   ```

3. 建立具有模型和任何相依性的映像。

   ```python
   from azureml.core.image import ContainerImage
   
   image_config = ContainerImage.image_configuration(execution_script = "score.py",
                                                     runtime = "python",
                                                     conda_file = "myenv.yml",
                                                     description = "test",
                                                     tags = ["onnx"]
                                                    )

   image = ContainerImage.create(name = "myonnxmodelimage",
                                 # this is the model object
                                 models = [model],
                                 image_config = image_config,
                                 workspace = ws)

   image.wait_for_creation(show_output = True)
   ```

   `score.py` 檔案包含評分邏輯，並且必須包含在映像中。 此檔案是用來在映像中執行模型。 如需如何建立評分指令碼的指示，請參閱本[教學課程](tutorial-deploy-models-with-aml.md#create-scoring-script)。 ONNX 模型的範例檔案如下所示：

   ```python
   import onnxruntime
   import json
   import numpy as np
   import sys
   from azureml.core.model import Model

   def init():
       global model_path
       model_path = Model.get_model_path(model_name = 'MyONNXmodel')

   def run(raw_data):
       try:
           data = json.loads(raw_data)['data']
           data = np.array(data)
        
           sess = onnxruntime.InferenceSession(model_path)
           result = sess.run(["outY"], {"inX": data})
        
           return json.dumps({"result": result.tolist()})
       except Exception as e:
           result = str(e)
           return json.dumps({"error": result})
   ```

   `myenv.yml` 檔案說明映像所需的相依性。 請參閱本[教學課程](tutorial-deploy-models-with-aml.md#create-environment-file)，以取得如何建立環境檔案的指示，例如此範例檔案：

   ```
   from azureml.core.conda_dependencies import CondaDependencies 

   myenv = CondaDependencies()
   myenv.add_pip_package("azureml-core")
   myenv.add_pip_package("onnxruntime")

   with open("myenv.yml","w") as f:
    f.write(myenv.serialize_to_string())
   ```

4. 將使用 Azure Machine Learning 的 ONNX 模型部署至：
   + Azure 容器執行個體 (ACI)：[深入了解...](how-to-deploy-to-aci.md)

   + Azure Kubernetes Service (AKS)：[深入了解...](how-to-deploy-to-aks.md)


## <a name="examples"></a>範例
 
下列 Notebook 示範如何部署使用 Azure Machine Learning 的 ONNX 模型： 
+ `/onnx/onnx-inference-mnist.ipynb`
 
+ `/onnx/onnx-inference-emotion-recognition.ipynb`
 
請取得此筆記本：
 
[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="more-info"></a>其他資訊

深入了解 ONNX 或參與專案：
+ [ONNX 專案網站](http://onnx.ai)

+ [GitHub 上的 ONNX 程式碼](https://github.com/onnx/onnx)
