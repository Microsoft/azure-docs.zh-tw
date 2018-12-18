---
title: Azure Machine Learning 模型管理安裝和組態 |Microsoft Docs
description: 本文件說明在 Azure Machine Learning 中安裝和設定模型管理有關的步驟和概念。
services: machine-learning
author: aashishb
ms.author: aashishb
manager: hjerez
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 12/6/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 84068376f35bc4df6672cffcc0ac3438b12edbb5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979670"
---
# <a name="model-management-setup"></a>模型管理安裝

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]


本文件可讓您開始使用 Azure ML 模型管理部署機器學習模型作為 Web 服務並進行管理。 

使用 Azure ML 模型管理後，可以有效部署和管理使用多種架構建置的 Machine Learning 模型，這些架構包含 SparkML、Keras、TensorFlow、Microsoft Cognitive Toolkit 或 Python。 

在本文件結束時，您應該已經設定模型管理環境，並準備部署您的機器學習模型。

## <a name="what-you-need-to-get-started"></a>若要開始，您需要：
若要善用本指南，您應該對於您可以部署模型的 Azure 訂用帳戶或資源群組具備參與者存取權。
CLI 已預先安裝於 Azure Machine Learning Workbench 和 [Azure DSVM](https://docs.microsoft.com/azure/machine-learning/machine-learning-data-science-virtual-machine-overview) 上。

## <a name="using-the-cli"></a>使用 CLI
若要從 Workbench 使用命令列介面 (CLI)，請按一下 [檔案] -> [開啟命令提示字元]。 

在資料科學虛擬機器上，連接並開啟命令提示字元。 輸入 `az ml -h` 查看選項。 如需有關命令的詳細資訊，請使用 --help 旗標。

在其他所有系統上，您必須安裝 CLI。

>[!NOTE]
> 在 Linux DSVM 上的 Jupyter Notebook 中，您可以使用下列命令格式存取 Azure CLI 和 Azure ML CLI。  **這特別適用於 Linux DSVM 上的 Jupyter Notebook**。  這些命令會存取 Notebook 中目前的 Python 核心 (例如 conda `py35` 環境)
>```
>import sys
>! {sys.executable} -m azure.cli login
>! {sys.executable} -m azure.cli ml -h
>```

### <a name="installing-or-updating-on-windows"></a>在 Windows 上安裝 (或更新)

從 https://www.python.org/ 安裝 Python。 確定您已選取安裝 pip。

使用系統管理員身分開啟命令提示字元，並執行下列命令：

```cmd
pip install -r https://aka.ms/az-ml-o16n-cli-requirements-file
```

### <a name="installing-or-updating-on-linux"></a>在 Linux 上安裝 (或更新)
從命令列執行下列命令，並遵循提示進行：

```bash
sudo -i
pip install -r https://aka.ms/az-ml-o16n-cli-requirements-file
```

### <a name="configuring-docker-on-linux"></a>在 Linux 上設定 Docker
若要在 Linux 上設定 Docker 以供非根使用者使用，請遵循下列指示進行：[適用於 Linux 的安裝後步驟\(Post-installation steps for Linux\)](https://docs.docker.com/engine/installation/linux/linux-postinstall/)

>[!NOTE]
> 在 Linux DSVM 上，您可以執行下列指令碼正確設定 Docker。 **執行指令碼後，務必登出再重新登入。**
>```
>sudo /opt/microsoft/azureml/initial_setup.sh
>```

## <a name="deploying-your-model"></a>部署您的模型
使用 CLI 部署模型作為 Web 服務。 Web 服務可以部署在本機或叢集。

開始先進行本機部署、並驗證您的模型和程式碼正常運作，然後部署至叢集用於生產等級用途。

若要開始，您需要設定您的部署環境。 環境設定是一次完成的工作。 設定完成後，可以對於後續部署重複使用該環境。 請參閱下一節以了解更多詳細資料。

完成環境設定時：
- 系統會提示您登入 Azure。 若要登入，請使用網頁瀏覽器開啟頁面 https://aka.ms/devicelogin，並輸入所提供的代碼以進行驗證。
- 在驗證過程中，會提示您用來驗證的帳戶。 重要：選取具有有效 Azure 訂用帳戶和有足夠權限可在帳戶中建立資源的帳戶。 完成登入後，即會顯示您的訂用帳戶資訊，且系統會提示您是否要繼續使用選取的帳戶。

### <a name="environment-setup"></a>環境設定
若要開始安裝程序，您必須輸入下列命令來註冊一些環境提供者：

```azurecli
az provider register -n Microsoft.MachineLearningCompute
az provider register -n Microsoft.ContainerRegistry
az provider register -n Microsoft.ContainerService
```
#### <a name="local-deployment"></a>本機部署
若要在本機電腦上部署和測試您的 Web 服務，請使用下列命令設定本機環境。 資源群組名稱為選用。

```azurecli
az ml env setup -l [Azure Region, e.g. eastus2] -n [your environment name] [-g [existing resource group]]
```
>[!NOTE] 
>本機 Web 服務部署會要求您在本機電腦上安裝 Docker。 
>

本機環境安裝命令會在您的訂用帳戶中建立下列資源：
- 資源群組 (如果未提供，或提供的名稱不存在)
- 儲存體帳戶
- Azure Container Registry (ACR)
- Application Insights 帳戶

安裝順利完成之後，使用下列命令設定將使用的環境：

```azurecli
az ml env set -n [environment name] -g [resource group]
```

#### <a name="cluster-deployment"></a>叢集部署
對於高度調整生產案例使用叢集部署。 這將設定 Kubernetes 為協調器的 ACS 叢集。 ACS 叢集可以向外延展，以處理 Web 服務呼叫的較大輸送量。

若要將您的 web 服務部署到生產環境中，請先使用下列命令設定環境：

```azurecli
az ml env setup --cluster -n [your environment name] -l [Azure region e.g. eastus2] [-g [resource group]]
```

叢集環境安裝命令會在您的訂用帳戶中建立下列資源：
- 資源群組 (如果未提供，或提供的名稱不存在)
- 儲存體帳戶
- Azure Container Registry (ACR)
- Azure 容器服務 (ACS) 叢集上的 Kubernetes 部署
- Application Insights 帳戶

>[!IMPORTANT]
> 若要成功建立叢集環境，您必須要有 Azure 訂用帳戶或資源群組上的參與者存取權。

很快就能夠建立資源群組、儲存體帳戶和 ACR。 ACS 部署最多需要 20 分鐘的時間。 

若要檢查進行中叢集佈建的狀態，請使用下列命令：

```azurecli
az ml env show -n [environment name] -g [resource group]
```

安裝式完成之後，您需要設定要用於這個部署的環境。 使用下列命令：

```azurecli
az ml env set -n [environment name] -g [resource group]
```

>[!NOTE] 
> 建立環境之後，對於後續的部署，您只需要使用上述的 set 命令即可重複使用該環境。
>

### <a name="create-a-model-management-account"></a>建立模型管理帳戶
需要模型管理帳戶才能部署模型。 對於每個訂用帳戶只需要進行一次，而且可以在多個部署中重複使用相同的帳戶。

若要建立新的帳戶，請使用以下命令：

```azurecli
az ml account modelmanagement create -l [Azure region, e.g. eastus2] -n [your account name] -g [resource group name] --sku-instances [number of instances, e.g. 1] --sku-name [Pricing tier for example S1]
```

若要使用現有的帳戶，請使用以下命令：
```azurecli
az ml account modelmanagement set -n [your account name] -g [resource group it was created in]
```

由於此程序和環境均已就緒，且已建立模型管理帳戶來提供管理和部署機器學習模型所需的功能 (如需概略說明，請參閱 [Azure Machine Learning 模型管理](model-management-overview.md))。

## <a name="next-steps"></a>後續步驟

* 如需如何部署在本機電腦或叢集上執行之 Web 服務的說明，請繼續參閱[將機器學習服務模型部署為 Web 服務](model-management-service-deploy.md)。
* 嘗試資源庫中的其中一個範例。
