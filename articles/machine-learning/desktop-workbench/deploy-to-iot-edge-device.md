---
title: 將 Azure Machine Learning 模型部署到 Azure IoT Edge 裝置 | Microsoft Docs
description: 本文說明如何將 Azure Machine Learning 模型部署到 Azure IoT Edge 裝置。
services: machine-learning
author: tedway
ms.author: tedway
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 08/24/2018
ms.openlocfilehash: 24d3cf0c4b1a1283e7a6a7f61f0bb23dae7143d5
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43113553"
---
# <a name="deploy-an-azure-machine-learning-model-to-an-azure-iot-edge-device"></a>將 Azure Machine Learning 模型部署到 Azure IoT Edge 裝置

Azure Machine Learning 模型可以容器化為 Docker 型 Web 服務。 Azure IoT Edge 可讓您從遠端將容器部署到裝置上。 一起使用這些服務，以在邊緣執行您的模型，進而加快回應時間並減少資料傳輸。 

其他指令碼與指示位於[適用於 Azure IoT Edge 的 AI 工具組](http://aka.ms/AI-toolkit)。

## <a name="operationalize-the-model"></a>模型運算化

Azure IoT Edge 模組是以容器映像為基礎。 若要將 Machine Learning 模型部署到 IoT Edge 裝置，您需要建立 Docker 映像。

依照[部署 Azure Machine Learning 模型管理 Web 服務](model-management-service-deploy.md)中的指示進行，讓您的模型可以運作，以使用該模型建立 Docker 映像。

## <a name="deploy-to-azure-iot-edge"></a>部署至 Azure IoT Edge

擁有模型的映像之後，即可將它部署到任何 Azure IoT Edge 裝置。 所有 Machine Learning 模型都可以在 IoT Edge 裝置上執行。 

### <a name="set-up-an-iot-edge-device"></a>設定 IoT Edge 裝置

使用 Azure IoT Edge 文件來準備裝置。 

1. [向 Azure IoT 中樞註冊裝置](../../iot-edge/how-to-register-device-portal.md)。 此程序的輸出是您可用來設定實體裝置的連接字串。 
2. 在實體裝置上安裝 IoT Edge 執行階段，並使用連接字串進行設定。 您可以在 [Windows](../../iot-edge/how-to-install-iot-edge-windows-with-windows.md) 或 [Linux](../../iot-edge/how-to-install-iot-edge-linux.md) 裝置上安裝執行階段。  


### <a name="find-the-machine-learning-container-image-location"></a>找出 Machine Learning 容器映像位置
您需要 Machine Learning 容器映像的位置。 若要找出容器映像位置：

1. 登入 [Azure 入口網站](http://portal.azure.com/)。
2. 在 [Azure Container Registry] 中，選取您想要檢查的登錄。
3. 在登錄刀鋒視窗中，按一下 [儲存機制] 以查看所有儲存機制和其映像的清單。

當您在 Azure 入口網站中查看您的容器登錄時，請擷取容器登錄認證。 這些認證必須提供給 IoT Edge 裝置，以便它從私人登錄提取映像。 

1. 在容器登錄中，按一下 [存取金鑰]。 
2. [啟用] 系統管理員使用者 (如果尚未啟用)。 
3. 儲存 [登入伺服器]、[使用者名稱] 及 [密碼] 的值。 

### <a name="deploy-the-container-image-to-your-device"></a>將容器映像部署到您的裝置

取得容器映像和容器登錄認證，您就準備好將機器學習模型部署到您的 IoT Edge 裝置。 

請依照[從 Azure 入口網站部署 IoT Edge 模組](../../iot-edge/how-to-deploy-modules-portal.md)中的指示，在 IoT Edge 裝置上啟動您的模型。 











