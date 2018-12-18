---
title: Azure Machine Learning 模型管理安裝和組態 |Microsoft Docs
description: 本文件說明在 Azure Machine Learning 中安裝和設定模型管理有關的步驟和概念。
services: machine-learning
author: raymondlaghaeian
ms.author: raymondl
manager: hjerez
ms.reviewer: jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 08/29/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 6660657141cc5aac532d121b61c7c8db6a24ccda
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968708"
---
# <a name="model-management-setup"></a>模型管理安裝

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



## <a name="overview"></a>概觀
本文件可讓您開始使用 Azure ML 模型管理部署機器學習模型作為 Web 服務並進行管理。 

使用 Azure ML 模型管理後，可以有效部署和管理使用多種架構建置的 Machine Learning 模型，這些架構包含 SparkML、Keras、TensorFlow、Microsoft Cognitive Toolkit 或 Python。 

在本文件結束時，您應該已經設定模型管理環境，並準備部署您的機器學習模型。

## <a name="what-you-need-to-get-started"></a>若要開始，您需要：
若要善用本指南，您應該對於您可以部署模型的 Azure 訂用帳戶具備擁有者存取權。
CLI 已預先安裝於 Azure Machine Learning Workbench 和 [Azure DSVM](https://docs.microsoft.com/azure/machine-learning/machine-learning-data-science-virtual-machine-overview) 上。

## <a name="using-the-cli"></a>使用 CLI
若要從 Workbench 使用命令列介面 (CLI)，請按一下 [檔案] -] 開啟命令列介面。 

在資料科學虛擬機器上，連接並開啟命令提示字元。 輸入 `az ml -h` 查看選項。 如需有關命令的詳細資訊，請使用 --help 旗標。

在其他所有系統上，您必須安裝 CLI。

### <a name="installing-or-updating-on-windows"></a>在 Windows 上安裝 (或更新)

從 https://www.python.org/ 安裝 Python。 確定您已選取安裝 pip。

使用系統管理員身分開啟命令提示字元，並執行下列命令：

```cmd
pip install azure-cli
pip install azure-cli-ml
```
 
>[!NOTE]
>如果您使用較早的版本，請使用下列命令先加以解除安裝：
>

```cmd
pip uninstall azure-cli-ml
```

### <a name="installing-or-updating-on-linux"></a>在 Linux 上安裝 (或更新)
從命令列執行下列命令，並遵循提示進行：

```bash
sudo -i
pip install azure-cli
pip install azure-cli-ml
```

安裝完成之後，請執行下列命令：

```bash
sudo /opt/microsoft/azureml/initial_setup.sh
```

>[!NOTE]
>先登出再重新登入 SSH 工作階段，以便讓變更生效。
>您可以使用前面的命令來更新 DSVM 上的舊版 CLI。
>

## <a name="deploying-your-model"></a>部署您的模型
使用 CLI 部署模型作為 Web 服務。 Web 服務可以部署在本機或叢集。

開始先進行本機部署、並驗證您的模型和程式碼正常運作，然後部署至叢集用於生產等級用途。

若要開始，您需要設定您的部署環境。 環境設定是一次完成的工作。 設定完成後，可以對於後續部署重複使用該環境。 請參閱下一節以了解更多詳細資料。

完成環境設定時：
- 系統會提示您登入 Azure。 若要登入，請使用網頁瀏覽器開啟頁面 https://aka.ms/devicelogin，並輸入所提供的代碼以進行驗證。
- 在驗證過程中，會提示您用來驗證的帳戶。 重要事項：選取具有有效 Azure 訂用帳戶而且有充分的權限在帳戶中建立資源的帳戶。登入完成時，會顯示您的訂用帳戶資訊，並提示您是否要使用選取的帳戶繼續。

### <a name="environment-setup"></a>環境設定
若要開始安裝程序，您必須輸入下列命令註冊環境提供者：

```azurecli
az provider register -n Microsoft.MachineLearningCompute
```

#### <a name="local-deployment"></a>本機部署
若要在本機電腦上部署和測試您的 Web 服務，請使用下列命令設定本機環境：

```azurecli
az ml env setup -l [location of Azure Region, e.g. eastus2] -n [your environment name] [-g [existing resource group]]
```
>[!NOTE] 
>本機 Web 服務部署會要求您在本機電腦上安裝 Docker。 
>

本機環境安裝命令會在您的訂用帳戶中建立下列資源：
- 資源群組 (如果未提供)
- 儲存體帳戶
- Azure Container Registry (ACR)
- Application insights

安裝順利完成之後，使用下列命令設定將使用的環境：

```azurecli
az ml env set -n [environment name] -g [resource group]
```

#### <a name="cluster-deployment"></a>叢集部署
對於高度調整生產案例使用叢集部署。 這將設定 Kubernetes 為協調器的 ACS 叢集。 ACS 叢集可以向外延展，以處理 Web 服務呼叫的較大輸送量。

若要將您的 web 服務部署到生產環境中，請先使用下列命令設定環境：

```azurecli
az ml env setup -c --name [your environment name] --location [Azure region e.g. eastus2] [-g [resource group]]
```

叢集環境安裝命令會在您的訂用帳戶中建立下列資源：
- 資源群組 (如果未提供)
- 儲存體帳戶
- Azure Container Registry (ACR)
- Azure Container Service (ACS) 叢集上的 Kubernetes 部署
- Application insights

很快就能夠建立資源群組、儲存體帳戶和 ACR。 ACS 部署最多需要 20 分鐘的時間。 

安裝式完成之後，您需要設定要用於這個部署的環境。 使用下列命令：

```azurecli
az ml env set -n [environment name] -g [resource group]
```

>[!NOTE] 
> 建立環境之後，對於後續的部署，您只需要使用上述的 set 命令即可重複使用該環境。
>

>[!NOTE] 
>若要建立 HTTPS 端點，請在建立叢集時，於 az ml env setup 中使用 --cert-name 和 --cert-pem 選項來指定 SSL 憑證。 這會設定讓叢集透過 https 為要求提供服務，使用所提供的憑證來保護安全。 設定完成之後，請建立一個指向叢集 FQDN 的 CNAME DNS 記錄。

### <a name="create-an-account"></a>建立帳戶
需要有帳戶才能部署模型。 對於每個帳戶只需要進行一次，而且可以在多個部署中重複使用相同的帳戶。

若要建立新的帳戶，請使用以下命令：

```azurecli
az ml account modelmanagement create -l [Azure region, e.g. eastus2] -n [your account name] -g [resource group name] --sku-instances [number of instances, e.g. 1] --sku-name [Pricing tier for example S1]
```

若要使用現有的帳戶，請使用以下命令：
```azurecli
az ml account modelmanagement set -n [your account name] -g [resource group it was created in]
```

### <a name="deploy-your-model"></a>部署模型
您現在即可準備部署已儲存的模型作為 Web 服務。 

```azurecli
az ml service create realtime --model-file [model file/folder path] -f [scoring file e.g. score.py] -n [your service name] -s [schema file e.g. service_schema.json] -r [runtime for the Docker container e.g. spark-py or python] -c [conda dependencies file for additional python packages]
```

### <a name="next-steps"></a>後續步驟
嘗試資源庫中的其中一個範例。
