---
title: 關於 Azure Machine Learning CLI 擴充功能
description: 深入了解 Azure Machine Learning 的 Machine Learning CLI 擴充功能。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: reference
ms.reviewer: jmartens
ms.author: jordane
author: jpe316
ms.date: 09/24/2018
ms.openlocfilehash: f8dae6de835173181430a98c19c7dd1fb3ebaa9f
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47158898"
---
# <a name="what-is-the-azure-machine-learning-cli"></a>什麼是 Azure Machine Learning CLI？

Azure Machine Learning 命令列介面 (CLI) 是專門提供給使用 Azure Machine Learning 服務的資料科學家和開發人員的擴充功能。 這個擴充功能可讓您迅速將機器學習服務工作流程自動化，並放入生產環境，例如：
+ 執行實驗來建立機器學習服務模型

+ 註冊供客戶使用的機器學習服務模型

+ 封裝、部署和追蹤機器學習模型的生命週期

此機器學習 CLI 是 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) 的擴充功能，是以 Azure Machine Learning 服務的 Python 型 <a href="http://aka.ms/aml-sdk" target="_blank">SDK</a> 為基礎。

> [!NOTE]
> CLI 目前為早期預覽狀態，日後將會更新。

## <a name="installing-and-uninstalling"></a>安裝和解除安裝

您可以至我們的預覽版 PyPi 索引，使用下列命令安裝 CLI：
```AzureCLI
az extension add -s https://azuremlsdktestpypi.blob.core.windows.net/wheels/sdk-release/Preview/E7501C02541B433786111FE8E140CAA1/azure_cli_ml-0.1.50-py2.py3-none-any.whl --pip-extra-index-urls  https://azuremlsdktestpypi.azureedge.net/sdk-release/Preview/E7501C02541B433786111FE8E140CAA1
```

使用此命令可移除 CLI：
```AzureCLI
az extension remove -n azure-cli-ml
```

您可以使用上述的**移除**和**新增**步驟來更新 CLI。

## <a name="using-the-cli-vs-the-sdk"></a>使用 CLI 與 SDK 的比較
CLI 較適合開發營運人員的自動化，或適用於連續整合和交貨管線。 已經過最佳化處理，可處理不頻繁而且高度參數化的工作。 

範例包括：
- 計算佈建
- 參數化實驗提交
- 模型註冊、映像建立
- 服務部署

建議資料科學家使用 Azure ML SDK。

## <a name="common-machine-learning-cli-commands"></a>常見的機器學習 CLI 命令
> [!NOTE]
> 可用來成功執行下列命令的檔案範例可於[這裡](https://github.com/Azure/MachineLearningNotebooks/tree/cli/cli)找到。

在任何命令列環境中使用多樣化的 `az ml` 命令，包括 Azure 入口網站雲端殼層。

一些常用命令如下所示：

### <a name="workspace-creation--compute-setup"></a>工作區建立和計算設定

+ 建立 Azure Machine Learning 工作區，也就是機器學習的最上層資源。
   ```AzureCLI
   az ml workspace create -n myworkspace -g myresourcegroup
   ```

+ 設定 CLI 預設使用此工作區。
   ```AzureCLI
   az configure --defaults aml_workspace=myworkspace group=myresourcegroup
   ```

+ 建立 DSVM (資料科學 VM)。 您也可以建立 BatchAI 叢集來用於分散式訓練，或建立 AKS 叢集來用於部署。
  ```AzureCLI
  az ml computetarget setup dsvm -n mydsvm
  ```

### <a name="experiment-submission"></a>實驗提交
+ 附加至專案 (回合組態) 來提交實驗。 可用來追蹤實驗回合。
  ```AzureCLI
  az ml project attach --experiment-name myhistory
  ```

+ 在您選擇的計算目標上，提交對 Azure Machine Learning 服務的實驗。 此範例會針對本機計算環境來執行。 請確定 Conda 環境檔案會擷取 Python 相依性。

  ```AzureCLI
  az ml run submit -c local train.py
  ```

+ 檢視已提交的實驗清單。
```AzureCLI
az ml history list
```

### <a name="model-registration-image-ceation--deployment"></a>模型註冊、映像建立和部署

+ 向 Azure Machine Learning 註冊模型。
  ```AzureCLI
  az ml model register -n mymodel -m sklearn_regression_model.pkl
  ```

+ 建立映像，以納入您的機器學習模型和相依性。 
  ```AzureCLI
  az ml image create container -n myimage -r python -m mymodel:1 -f score.py -c myenv.yml
  ```

+ 將封裝的模型部署到包括 ACI 和 AKS 的目標。
  ```AzureCLI
  az ml service create aci -n myaciservice --image-id myimage:1
  ```
    
## <a name="full-command-list"></a>完整命令清單
您也可以執行 ```az ml COMMANDNAME -h``` 找出 CLI 擴充功能 (和其支援的參數) 的命令完整清單。 
