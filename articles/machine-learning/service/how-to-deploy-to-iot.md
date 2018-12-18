---
title: 從 Azure Machine Learning 服務將模型部署至 IoT Edge 裝置| Microsoft Docs
description: 了解如何從 Azure Machine Learning 服務將模型部署至 Azure IoT Edge 裝置。 將模型部署到邊緣裝置可讓您在裝置上為資料評分，而不需要將它傳送到雲端並等候回應。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: shipatel
author: shivanipatel
manager: cgronlun
ms.reviewer: larryfr
ms.date: 09/24/2018
ms.openlocfilehash: 03d692ddfd6f41fd559e9b921f0214a9cd2ada22
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225220"
---
# <a name="prepare-to-deploy-models-on-iot-edge"></a>準備在 IoT Edge 上部署模型

在此文件中，了解如何使用 Azure Machine Learning 服務準備定型模型，以部署到 Azure IoT Edge 裝置。

Azure IoT Edge 裝置是以 Linux 或 Windows 為基礎並執行 Azure IoT Edge 執行階段的裝置。 機器學習模型可以部署到這些裝置作為 IoT Edge 模組。 將模型部署到 IoT Edge 裝置可讓裝置直接使用模型，而不需要將資料傳送到雲端進行處理。 您可以獲得更快的回應時間並傳輸較少的資料。

將模型部署到邊緣裝置之前，請先使用此文件中的步驟準備模型與裝置。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* Azure Machine Learning 服務工作區。 若要建立工作區，請使用[開始使用 Azure Machine Learning 服務](quickstart-get-started.md)文件中的步驟。

* 開發環境。 如需詳細資訊，請參閱[如何設定開發環境](how-to-configure-environment.md)文件。

* Azure 訂用帳戶中的 [Azure IoT 中樞](../../iot-hub/iot-hub-create-through-portal.md)。 

* 已定型的模型。 如需模型定型方式範例，請參閱[使用 Azure Machine Learning 將映像分類模型定型](tutorial-train-models-with-aml.md)文件。

## <a name="prepare-the-iot-device"></a>準備 IoT 裝置

若要了解如何註冊裝置並安裝 IoT 執行階段，請依照[快速入門：將您的第一個 IoT Edge 模組部署至 Linux x64 裝置](../../iot-edge/quickstart-linux.md)文件中的步驟執行。

## <a name="register-the-model"></a>註冊模型

Azure IoT Edge 模組是以容器映像為基礎。 若要將模型部署至 IoT Edge 裝置，請使用下列步驟在 Azure Machine Learning 工作區註冊模型，並建立 Docker 映像。 

> [!IMPORTANT]
> 若您使用 Azure Machine Learning 將模型定型，模型可能已在工作區中註冊，此時請略過步驟 3。

1. 初始化工作區並載入 config.json 檔案：

    ```python
    from azureml.core  import Workspace

    #Load existing workspace from the config file info.
    ws  = Workspace.from_config()
    ```    

1. 將模型註冊到工作區中。 以您的模型路徑、名稱、標記與描述取代預設文字：

    ```python
    from azureml.core.model import Model
    model = Model.register(model_path = "model.pkl", # this path points to the local file
                        model_name = "best_model", # the model gets registered as this name
                        tags = {'attribute': "myattribute", 'classification': "myclassification"},
                        description = "My awesome model",
                        workspace = ws)
    ```    

1. 擷取已註冊的模型： 

    ```python
    from azureml.core.model import Model

    model_name = "best_model"
    model = Model(ws, model_name)                     
    ```    

## <a name="create-a-docker-image"></a>建立 Docker 映像

1. 建立名為 `score.py` 的**評分指令碼**。 此檔案用來在映像內部執行模型。 它必須包含下列函式：

    * `init()` 函式，通常會將模型載入至全域物件。 此函式只會在 Docker 容器啟動時執行一次。 

    * `run(input_data)` 函式會使用模型依據輸入資料預測值。 回合的輸入與輸出通常會使用 JSON 進行序列化與還原序列化，但支援其他格式。

    如需範例，請參閱[映像分類教學課程](tutorial-deploy-models-with-aml.md#make-script)。

1. 建立名為 `myenv.yml` 的**環境檔案**。 這個檔案是 Conda 環境規格，而且會列出指令碼與模型所需的所有相依性。 如需範例，請參閱[映像分類教學課程](tutorial-deploy-models-with-aml.md#make-myenv)。

1. 使用 `score.py` 與 `myenv.yml` 檔案設定 Docker 映像：
    
    ```python
    from azureml.core.image import Image, ContainerImage
    
    #Image configuration
    image_config = ContainerImage.image_configuration( runtime = "python", 
                           execution_script = "score.py",
                           conda_file = "myenv.yml", 
                           tags = {"attributes", "calssification"},
                           description = "Image that contains my model",
                           
                        )
    ```    

1. 使用模型與映像設定建立映像：

    ```python
    image = ContainerImage.create (name = "myimage", 
                           models = [model], #this is the model object
                           image_config = image_config,
                           workspace = ws
                        )
    ```     

    建立映像需要大約 5 分鐘的時間。

## <a name="get-the-container-registry-credentials"></a>取得容器登錄認證

Azure IoT 需要 Azure Machine Learning 服務儲存 Docker 映像之容器登錄的認證。 使用下列步驟來取得該認證：

1. 登入 [Azure 入口網站](https://portal.azure.com/signin/index)。

1. 前往您的 Azure Machine Learning 工作區並選取 [概觀]。 前往容器登錄設定，選取 [登錄] 連結。

    ![容器登錄項目影像](./media/how-to-deploy-to-iot/findregisteredcontainer.png)

1. 進入容器登錄之後，請選取 [存取金鑰]，然後啟用管理使用者。

    ![存取金鑰畫面影像](./media/how-to-deploy-to-iot/findaccesskey.png)

1. 儲存登入伺服器、使用者名稱與密碼的值。 

   必須透過這些認證提供存取權，IoT Edge 裝置才能存取您私人容器登錄中的映像。

## <a name="next-steps"></a>後續步驟

您已經完成部署的準備工作。 現在，請使用[從 Azure 入口網站部署 Azure IoT Edge 模組](../../iot-edge/how-to-deploy-modules-portal.md)文件中的步驟部署邊緣裝置。 設定裝置的__登錄設定__時，請使用之前設定的認證。
