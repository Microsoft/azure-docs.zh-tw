---
title: 移轉至 Azure Machine Learning 服務
description: 了解如何從舊版本升級或移轉至最新版的 Azure Machine Learning 服務。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: haining
author: haining
ms.date: 09/24/2018
ms.openlocfilehash: 0c5cfcee65b176147125dd2edbff2497b1eee70b
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47158559"
---
# <a name="migrate-to-the-latest-version-of-azure-machine-learning-service"></a>移轉至最新版的 Azure Machine Learning 服務 

**如果您已安裝 Workbench (預覽) 應用程式及/或具有實驗和模型管理預覽帳戶，則請使用本文移轉至最新版本。**  如果您尚未安裝 Workbench 預覽版，或者沒有實驗及/或模型管理帳戶，則不需要移轉任何項目。

## <a name="what-can-i-migrate"></a>我可以移轉的項目為何？
在 Azure Machine Learning 服務的第一個預覽版中建立的大部分成品，都會儲存在您自己的本機或雲端儲存體中。 這些成品將不會消失。 若要移轉，請使用已更新的 Azure Machine Learning 服務再次登錄成品。 

下表和下文說明現有資產和資源在移動至最新版 Azure Machine Learning 服務前後的用途。 您也可以繼續使用舊版本和資產一段時間 ([請參閱轉換支援時間表](overview-what-happened-to-workbench.md#timeline))。

|舊版本的資產或資源|我可以移轉嗎？|動作|
|-----------------|:-------------:|-------------|
|機器學習模型 (作為本機檔案)|是|無。 運作方式與之前相同。|
|模型相依性和結構描述 (作為本機檔案)|是|無。 運作方式與之前相同。|
|專案|是|[將本機資料夾連結至新的工作區](#projects)。|
|執行歷程記錄|否|[可下載](#history)一段時間。|
|資料準備檔案|否|使用新的 Azure Machine Learning 資料準備 SDK 或使用 Azure Databricks 來準備用於建立模型的[任何大小資料集](#dataprep)。|
|計算目標|否|在新的工作區中註冊它們。|
|已註冊的模型|否|在新的工作區下重新註冊模型。|
|已註冊的資訊清單|否|無。 在新的工作區中，資訊清單不再是概念。|
|已註冊的映像|否|在新的工作區下重新建立部署 Docker 映像。|
|已部署的 Web 服務|否|無。 其運作方式仍如現狀 <br/>或[使用最新版本重新予以部署](#services)。|
|實驗和 <br/>模型管理帳戶|否|改為[建立工作區](#resources)。|
|機器學習 CLI 和 SDK|否|針對新的工作，使用新的 [CLI](reference-azure-machine-learning-cli.md) 和 [SDK](http://aka.ms/aml-sdk)。|


深入了解[此版本的變更內容](overview-what-happened-to-workbench.md)？

>[!Warning]
>本文適用對象不是 Azure Machine Learning Studio 使用者。 這適用於 Azure Machine Learning 服務客戶，而這些客戶已安裝 Workbench (預覽) 應用程式及/或具有實驗和模型管理預覽帳戶。

<a name="resources"></a>

## <a name="azure-resources"></a>Azure 資源

實驗帳戶、模型管理帳戶和機器學習計算環境這類資源無法移轉至最新版的 Azure Machine Learning 服務。 請參閱[時間表](overview-what-happened-to-workbench.md#timeline)，以了解資產將持續運作多長時間。

在 [Azure 入口網站](quickstart-get-started.md)中建立 Azure Machine Learning 工作區，以開始使用最新版本。 只有在 Edge、Chrome 和 Firefox 上才支援入口網站的工作區儀表板。

這個新的工作區是最上層服務資源，並可讓您使用 Azure Machine Learning 服務的所有最新功能。 深入了解此[工作區和架構](concept-azure-machine-learning-architecture.md)。

<a name="projects"></a>

## <a name="projects"></a>專案

專案現在是在最新版本中本機電腦的目錄中，而不是專案位在雲端的工作區中。 請參閱[最新架構](concept-azure-machine-learning-architecture.md)的圖表。 

若要繼續使用包含您檔案和指令碼的本機目錄，請在 ['experiment.submit'](http://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment.experiment?view=azure-ml-py) Python 命令中或使用 'az ml project attach' CLI 命令指定目錄的名稱。

例如︰
```python
run = exp.submit(source_directory = script_folder, script = 'train.py', run_config = run_config_system_managed)
```

<a name="services"></a>

## <a name="deployed-web-services"></a>已部署的 Web 服務

若要移轉 Web 服務，請使用新的 SDK 或 CLI 將模型重新部署至新部署目標。 不需要變更原始評分檔案、模型檔案相依性檔案、環境檔案和結構描述檔案。 

在最新版本中，模型會以 Web 服務形式部署至 [Azure 容器執行個體](how-to-deploy-to-aci.md) (ACI) 或 [Azure Kubernetes Service](how-to-deploy-to-aks.md) (AKS) 叢集。 

請在下列文章中深入了解：
+ [部署至 ACI](how-to-deploy-to-aci.md)
+ [部署至 AKS](how-to-deploy-to-aks.md)
+ [教學課程：使用 Azure Machine Learning 服務部署模型](tutorial-deploy-models-with-aml.md)

[支援先前的 CLI 端](overview-what-happened-to-workbench.md#timeline)時，您無法管理原本使用模型管理帳戶所部署的 Web 服務。 不過，只要仍支援 Azure Container Service (ACS)，這些 Web 服務就會繼續運作。

<a name="history"></a>

## <a name="run-history-records"></a>執行歷程記錄

雖然您無法繼續新增至舊工作區下的現有執行歷程記錄，但是可以使用先前的 CLI 匯出歷程記錄。 [支援先前的 CLI 端](overview-what-happened-to-workbench.md#timeline)時，您就無法再匯出這些執行歷程記錄。

開始使用新的 CLI 和 SDK 來訓練模型以及追蹤執行歷程記錄。 您可以了解如何使用[教學課程：使用 Azure Machine Learning 服務訓練模型](tutorial-train-models-with-aml.md)。

使用先前的 CLI 匯出執行歷程記錄：

```azurecli
#list all runs
az ml history list

#get details about a particular run
az ml history info

# download all artifacts of a run
az ml history download
```

<a name="dataprep"></a>

## <a name="data-preparation-files"></a>資料準備檔案
需要使用 Workbench，才能移植資料準備檔案。 但您仍然可以使用新的 Azure Machine Learning 資料準備 SDK 或使用用於巨量資料集的 Azure Databricks，來準備用於建立模型的任何大小資料集。  [了解如何取得資料準備 SDK](how-to-data-prep.md)。 

## <a name="next-steps"></a>後續步驟

如需快速入門以顯示如何使用最新版 Azure Machine Learning 服務來建立工作區、建立專案、執行指令碼，以及探索指令碼的執行歷程記錄，請嘗試[開始使用 Azure Machine Learning 服務](quickstart-get-started.md)。

如需此工作流程的更深入體驗，請遵循完整教學課程，其包含使用 Azure Machine Learning 服務訓練和部署模型的詳細步驟。 

> [!div class="nextstepaction"]
> [教學課程：訓練和部署模型](tutorial-train-models-with-aml.md)
