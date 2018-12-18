---
title: 在 Azure Machine Learning 中管理和部署模型
description: 了解如何使用 Azure Machine Learning 部署、管理和監視模型，以持續予以改善。 您可以在本機電腦上或從其他來源中，部署使用 Azure Machine Learning 所訓練的模型。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
author: hjerez
ms.author: hjerez
ms.date: 09/24/2018
ms.openlocfilehash: d3e0b63d42ad8c6d4765f5120c26c5dfdf5ad6fb
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166532"
---
# <a name="manage-deploy-and-monitor-models-with-azure-machine-learning"></a>使用 Azure Machine Learning 管理、部署和監視模型

在本文中，您可以了解如何使用 Azure Machine Learning 部署、管理和監視模型，以持續予以改善。 您可以在本機電腦上或從其他來源中，部署使用 Azure Machine Learning 所訓練的模型。 

下圖說明完整部署工作流程：[![Azure Machine Learning 的部署工作流程](media/concept-model-management-and-deployment/deployment-pipeline.png) ](media/concept-model-management-and-deployment/deployment-pipeline.png#lightbox)

部署工作流程包含下列步驟：
1. 在裝載於 Azure Machine Learning 工作區的登錄中**註冊模型**
1. **註冊映像**，以將具有評分指令碼之模型與可攜式容器中的相依性配對 
1. **部署**映像為雲端或邊緣裝置中的 Web 服務
1. **監視和收集資料**

每個步驟都可以單獨執行或當成單一部署命令的一部分來執行。 此外，您還可以將部署整合到 **CI/CD 工作流程**，如本圖所示。

[![Azure Machine Learning 持續整合/持續部署 (CI/CD) 週期](media/concept-model-management-and-deployment/model-ci-cd.png) ](media/concept-model-management-and-deployment/model-ci-cd.png#lightbox)


## <a name="step-1-register-model"></a>步驟 1：註冊模型

模型登錄可記錄您 Azure Machine Learning 工作區中的所有模型。
模型是透過名稱與版本來識別的。 每次註冊與現有模型名稱相同的模型時，登錄都會遞加版本。 您也可以在可於搜尋模型時使用的註冊期間提供額外中繼資料標記。

您無法刪除映像正在使用的模型。

## <a name="step-2-register-image"></a>步驟 2：註冊映像

映像可讓您部署可靠的模型，以及使用模型所需的所有元件。 映像包含下列項目：

* 模型
* 評分引擎
* 評分檔案或應用程式
* 對模型評分所需的任何相依性

映像也可以包含用於記錄和監視的 SDK 元件。 SDK 記錄資料可以用來微調或重新訓練模型 (包含模型的輸入和輸出)。

Azure Machine Learning 支援最熱門架構，但一般而言，任何可使用 PIP 安裝的架構可以運作。

建立工作區時，因此為該工作區使用的其他數個其他 Azure 資源。
所有用來建立映像的物件都會儲存在您工作區的 Azure 儲存體帳戶中。 映像會建立並儲存在 Azure Container Registry 中。 建立映像時，您可以提供額外的中繼資料標籤，而映像登錄也會儲存它們，並可以對其查詢以找到您的映像。

## <a name="step-3-deploy-image"></a>步驟 3：部署映像

您可以將已註冊的映像部署至雲端或邊緣裝置。 部署程序會建立監視、負載平衡和自動調整您模型所需的所有資源。 在部署期間提供安全性資產，可以使用憑證驗證保護對已部署服務的存取。 您也可以升級現有的部署，以使用較新的映像。

Web 服務部署亦為可搜尋。 例如，您可以搜尋特定模型或映像的所有部署。

[ ![推斷目標](media/concept-model-management-and-deployment/inferencing-targets.png) ](media/concept-model-management-and-deployment/inferencing-targets.png#lightbox)

您可以將映像部署至雲端中的下列[部署目標](how-to-deploy-and-where.md)：

* Azure 容器執行個體
* Azure Kubernetes Service
* Azure FPGA 電腦
* Azure IoT Edge 裝置

部署您的服務時，會自動負載平衡推斷要求，而且會縮放叢集以符合任何隨需的爆增。 [您服務的遙測](https://docs.microsoft.com/python/api/azureml-telemetry/azureml.telemetry?view=azure-ml-py)可以擷取到與您工作區建立關聯的 Azure Application Insights 服務。

## <a name="step-4-monitor-models-and-collect-data"></a>步驟 4：監視模型並收集資料

有用於模型記錄和資料擷取的 SDK 可供使用，因此您可以監視模型的輸入、輸出和其他相關資料。 資料會儲存為您工作區 Azure 儲存體帳戶中的 Blob。

若要搭配使用 SDK 與您的模型，請將 SDK 匯入至評分指令碼或應用程式。 您接著可以使用 SDK 來記錄資料 (例如參數、結果或輸入詳細資料)。

如果您決定每次部署映像時都[啟用模型資料收集](how-to-enable-data-collection.md)，則會自動佈建擷取資料所需的詳細資料 (例如個人 Blob 存放區的認證)。

> [!Important]
> Microsoft 看不到從您模型所收集的資料。 資料會直接傳送至您工作區的 Azure 儲存體帳戶。

## <a name="next-steps"></a>後續步驟

深入了解使用 Azure Machine Learning 服務[部署模型的方式和位置](how-to-deploy-and-where.md)。
