---
title: 為常用的工作安裝及使用 CLI - Azure Machine Learning
description: 了解如何為 Azure Machine Learning 中最常用的機器學習工作安裝及使用 CLI。
services: machine-learning
author: haining
ms.author: haining
manager: cgronlun
ms.reviewer: mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: conceptual
ms.date: 03/10/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 06e85845d41b240638a5b5b4d75d64fd460a99bf
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953322"
---
# <a name="install-and-use-the-machine-learning-cli-for-top-tasks-in-azure-machine-learning"></a>為 Azure Machine Learning 中常用的工作安裝及使用機器學習 CLI

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

Azure Machine Learning 服務是整合式端對端的資料科學與進階分析解決方案。 專業資料科學家可以使用 Azure Machine Learning 服務以雲端規模準備資料、開發實驗及部署模型。 

Azure Machine Learning 提供命令列介面 (CLI)，讓您可以：
+ 管理工作區和專案
+ 設定計算目標
+ 執行訓練實驗
+ 檢視過去執行的歷程記錄和計量
+ 在生產環境中將模型部署為 Web 服務
+ 管理生產環境模型和服務

本文說明一些最有用的 CLI 命令，讓您方便參考。 

![Azure Machine Learning CLI](media/cli-for-azure-machine-learning/flow.png)

## <a name="what-you-need-to-get-started"></a>若要開始，您需要：

您需要 Azure 訂用帳戶的參與者存取權，或是您可以在當中部署模型的資源群組。 此外，您還必須安裝 Azure Machine Learning Workbench 才能執行 CLI。 

>[!IMPORTANT]
>Azure Machine Learning 服務所提供的 CLI 與 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) 不同，後者是用來管理 Azure 資源。

## <a name="get-and-start-cli"></a>取得並啟動 CLI

若要取得此 CLI，請安裝 Azure Machine Learning Workbench，可以從這裡下載：
    + Windows - https://aka.ms/azureml-wb-msi 
    + MacOS - https://aka.ms/azureml-wb-dmg 

若要啟動 CLI：
+ 在 Azure Machine Learning Workbench 中，從功能表中的 [檔案] -> [開啟命令提示字元] 啟動 CLI。

## <a name="get-command-help"></a>取得命令說明 

在命令之後使用 `--debug` 或 `--help` 可以取得 CLI 命令的更多相關資訊，例如 `az ml <xyz> --debug`，其中 `<xyz>` 是命令名稱。 例如︰
```azurecli
az ml computetarget --debug 

az ml experiment --help
```

## <a name="common-cli-tasks-for-azure-machine-learning"></a>Azure Machine Learning 的常用 CLI 工作 

本節說明您可使用 CLI 執行的最常用工作，包括：
+ [設定計算目標](#target)
+ [提交要遠端執行的作業](#jobs)
+ [使用 Jupyter Notebook](#jupyter)
+ [與執行歷程記錄互動](#history)
+ [設定環境以利作業化](#o16n)

<a name="target"></a>

### <a name="set-up-a-compute-target"></a>設定計算目標

您可以計算在不同目標下的機器學習模型，包括：
+ 在本機模式
+ 在資料科學 VM (DSVM)
+ 在 HDInsight 叢集

若要連結資料科學 VM 目標：
```azurecli
az ml computetarget attach remotedocker -n <target name> -a <ip address or FQDN> -u <username> -w <password>
``` 

若要連結 HDInsight 目標：
```azurecli
az ml computetarget attach cluster -n <target name> -a <cluster name, e.g. myhdicluster-ssh.azurehdinsight.net> -u <ssh username> -w <ssh password>
```

在 **aml_config** 資料夾中，您可以變更 conda 相依性。 

此外，您可以使用 PySpark、Python 或 GPU DSVM 中的 Python進行操作。 

若要定義 Python 作業模式：
+ 針對 Python，在 `<target name>.runconfig` 中新增 `Framework:Python` 

+ 針對 PySpark，在 `<target name>.runconfig` 中新增 `Framework:PySpark` 

+ 針對 GPU DSVM 中的 Python，
    1. 在 `<target name>.runconfig` 中新增 `Framework:Python` 

    1. 也在 `<target name>.compute` 中新增 `baseDockerImage: microsoft/mmlspark:plus-gpu-0.9.9 and nvidiaDocker:true`

若要準備計算目標：
```azurecli
az ml experiment prepare -c <target name>
```

>[!TIP]
>若要顯示您的訂用帳戶：<br/>
>`az account show`<br/>
>
>若要設定您的訂用帳戶：<br/>
>`az account set –s "my subscription name" `

<a name="jobs"></a>

### <a name="submit-remote-jobs"></a>提交遠端作業

若要將作業提交至遠端目標：
```azurecli
az ml experiment submit -c <target name> myscript.py
```

<a name="jupyter"></a>

### <a name="work-with-jupyter-notebooks"></a>使用 Jupyter Notebook

若要啟動 Jupyter Notebook：
```azurecli
az ml notebook start
```

此命令會在 localhost 啟動 Jupyter Notebook。 您可以選取核心 Python 3 在本機操作，或選取核心 `<target name>` 在遠端 VM 操作。

<a name="history"></a>

### <a name="interact-with-and-explore-the-run-history"></a>與執行歷程記錄互動並瀏覽

若要列出執行歷程記錄：
```azurecli
az ml history list -o table
```

若要列出所有完成的執行：
```azurecli
az ml history list --query "[?status=='Completed']" -o table
```

若要尋找最佳精確度的執行：
```azurecli
az ml history list --query "@[?Accuracy != null] | max_by(@, &Accuracy).Accuracy"
```

您也可以下載每次執行所產生的檔案。 

若要將資料夾輸出中所儲存的模型升階：
```azurecli
az ml history promote -r <run id> -ap outputs/model.pkl -n <model name>
```

若要下載該模型：
```azurecli
az ml asset download -l assets/model.pkl.link -d <model folder path>
```

<a name="o16n"></a>

### <a name="configure-your-environment-to-operationalize"></a>設定環境以利作業化

若要設定您的作業化環境，您必須建立：

> [!div class="checklist"]
> * 資源群組 
> * 儲存體帳戶
> * Azure Container Registry (ACR)
> * Application Insights 帳戶
> * Azure Container Service (ACS) 叢集上的 Kubernetes 部署


若要設定要在 Docker 容器中測試的本機部署：
```azurecli
az ml env setup -l <region, e.g. eastus2> -n <env name> -g <resource group name>
```

若要設定 ACS 叢集搭配 Kubernetes：
```azurecli
az ml env setup -l <region, e.g. eastus2> -n <env name> -g <resource group name> --cluster
```

若要監視部署的狀態：
```azurecli
az ml env show -n <environment name> -g <resource group name>
```

若要設定應該使用的環境：
```azurecli
az ml env set -n <environment name> -g <resource group name>
```

## <a name="next-steps"></a>後續步驟

從下列其中一篇文章著手： 
+ [安裝及開始使用 Azure Machine Learning](quickstart-installation.md)
+ [分類 Iris 資料教學課程：第一部分](tutorial-classifying-iris-part-1.md)

從下列其中一篇文章進階：
+ [用於管理模型的 CLI 命令](model-management-cli-reference.md)
