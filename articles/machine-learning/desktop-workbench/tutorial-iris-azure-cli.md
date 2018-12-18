---
title: Azure Machine Learning 預覽功能的教學課程文章 - 命令列介面  | Microsoft Docs
description: 此教學課程逐步解說從命令列介面完成 Iris 分類端對端所需的所有步驟。
services: machine-learning
author: jpe316
ms.author: jordane
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: tutorial
ms.date: 10/15/2017
ROBOTS: NOINDEX
ms.openlocfilehash: b9b515a2ccaedc40fc531d6a1cc58cbe98212b4a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967892"
---
# <a name="tutorial-classifying-iris-using-the-command-line-interface"></a>教學課程：使用命令列介面分類 Iris

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

Azure Machine Learning 服務 (預覽) 是一套整合的端對端資料科學以及進階分析解決方案，可供專業資料科學家用來以雲端規模準備資料、開發測試以及部署模型。

在本教學課程中，您會了解如何使用 Azure Machine Learning 預覽功能中的命令列介面 (CLI)，以進行： 
> [!div class="checklist"]
> * 設定測試帳戶並建立工作區
> * 建立專案
> * 將實驗提交至多個計算目標
> * 升級並註冊定型模型
> * 部署 Web 服務以評分新資料

## <a name="prerequisites"></a>必要條件
若要完成本教學課程，您需要：
- 存取 Azure 訂用帳戶與權限，以建立該訂用帳戶中的資源。 
  
  如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

- 如[快速入門：安裝及啟動 Azure Machine Learning 服務](quickstart-installation.md)所述，安裝 Azure Machine Learning Workbench 應用程式。 

  >[!IMPORTANT]
  >請勿建立 Azure Machine Learning 服務帳戶，因為您會使用本文中的 CLI 來執行該作業。
 
## <a name="getting-started"></a>開始使用
Azure Machine Learning 命令列介面 (CLI) 可讓您執行端對端資料科學工作流程所需的所有工作。 您可透過下列方式存取 CLI 工具：

### <a name="option-1-launch-azure-ml-cli-from-azure-ml-workbench-log-in-dialog-box"></a>選項 1。 從 Azure ML Workbench 登入對話方塊啟動 Azure ML CLI
當您啟動 Azure ML Workbench 並且第一次登入時，如果您還沒有測試帳戶的存取權，會出現下列畫面：

![找不到帳戶](media/tutorial-iris-azure-cli/no_account_found.png)

按一下對話方塊中的 [命令列視窗] 連結，以啟動命令列視窗。

### <a name="option-2-launch-azure-ml-cli-from-azure-ml-workbench-app"></a>選項 2。 從 Azure ML Workbench 應用程式啟動 Azure ML CLI
如果您已具備測試帳戶的存取權，便可以成功登入。 接著，您便可以按一下 [檔案] --> [開啟命令提示字元] 功能表，開啟命令列視窗。

### <a name="option-3-enable-azure-ml-cli-in-an-arbitrary-command-line-window"></a>選項 3。 在任意命令列視窗中啟用 Azure ML CLI
您也可以在任何命令列視窗中啟用 Azure ML CLI。 啟動命令視窗，並輸入下列命令即可：

```sh
# Windows Command Prompt
set PATH=%LOCALAPPDATA%\amlworkbench\Python;%LOCALAPPDATA%\amlworkbench\Python\Scripts;%PATH%

# Windows PowerShell
$env:Path = $env:LOCALAPPDATA+"\amlworkbench\Python;"+$env:LOCALAPPDATA+"\amlworkbench\Python\Scripts;"+$env:Path

# macOS Bash Shell
PATH=$HOME/Library/Caches/AmlWorkbench/Python/bin:$PATH
```
若要進行永久變更，您可以在 Windows 上使用 `SETX`。 對於 macOS，您可以使用 `setenv`。

>[!TIP]
>您可以藉由設定上述環境變數，在您最愛的終端機視窗中啟用 Azure CLI。

## <a name="step-1-log-in-to-azure"></a>步驟 1. 登入 Azure
第一個步驟是從 AMLWorkbench 應用程式開啟 CLI (檔案 > 開啟命令提示字元)。 這麼做可確保您有正確的 Python 環境，而且有可用的 ML CLI 命令。 

現在，您可以在 CLI 中設定正確的內容，以存取和管理 Azure 資源。
 
```azure-cli
# log in
$ az login

# list all subscriptions
$ az account list -o table

# set the current subscription
$ az account set -s <subscription id or name>
```

## <a name="step-2-create-a-new-azure-machine-learning-experimentation-account-and-workspace"></a>步驟 2. 建立新的 Azure Machine Learning 測試帳戶與工作區

在此步驟中，您會建立一個新的測試帳戶和一個新的工作區。 如需測試帳戶與工作區的詳細資料，請參閱 [Azure Machine Learning 概念](overview-general-concepts.md)。

> [!NOTE]
> 測試帳戶需要儲存體帳戶，用來儲存實驗執行的輸出。 儲存體帳戶名稱在 Azure 中必須為全域唯一，因為沒有與其相關聯的 URL。 如果您未指定現有的儲存體帳戶，您的測試帳戶名稱會用來建立新的儲存體帳戶。 請務必使用唯一的名稱，否則您會收到錯誤，例如 _「已使用名稱為 \<storage_account_name> 的儲存體帳戶。」_ 或者，您可以使用 `--storage` 引數，以提供現有的儲存體帳戶。

```azure-cli
# create a resource group 
$ az group create --name <resource group name> --location <supported Azure region>

# create a new experimentation account with a new storage account
$ az ml account experimentation create --name <experimentation account name> --resource-group <resource group name>

# create a new experimentation account with an existing storage account
$ az ml account experimentation create --name <experimentation account name>  --resource-group <resource group name> --storage <storage account Azure Resource ID>

# create a workspace in the experimentation account
az ml workspace create --name <workspace name> --account <experimentation account name> --resource-group <resource group name>
```

## <a name="step-2a-optional-share-a-workspace-with-co-worker"></a>步驟 2.a (選用) 與同事共用工作區
您可以在這裡探討如何與同事共用工作區的存取權。 共用存取測試帳戶或專案的步驟是相同的。 只需要更新獲取 Azure 資源識別碼的方式。

```azure-cli
# find the workspace Azure Resource ID
$az ml workspace show --name <workspace name> --account <experimentation account name> --resource-group <resource group name>

# add Bob to this workspace as a new owner
$az role assignment create --assignee bob@contoso.com --role owner --scope <workspace Azure Resource ID>
```

> [!TIP]
> 上述命令中的 `bob@contoso.com` 必須是目前訂用帳戶所屬之目錄中有效的 Azure AD 身分識別。

## <a name="step-3-create-a-new-project"></a>步驟 3. 建立新專案
下一個步驟是建立新專案。 有數種方式可以開始使用新專案。

### <a name="create-a-new-blank-project"></a>建立新的空白專案

```azure-cli
# create a new project
$ az ml project create --name <project name> --workspace <workspace name> --account <experimentation account name> --resource-group <resource group name> --path <local folder path>
```

### <a name="create-a-new-project-with-a-default-project-template"></a>使用預設專案範本建立新的專案
您可以使用預設範本來建立新的專案。

```azure-cli
$ az ml project create --name <project name> --workspace <workspace name> --account <experimentation account name> --resource-group <resource group name> --path <local folder path> --template
```

### <a name="create-a-new-project-associated-with-a-cloud-git-repository"></a>建立與雲端 Git 儲存機制相關聯的新專案
您可以建立與 Azure DevOps Git 存放庫相關聯的新專案。 每次提交測試時，整個專案資料夾中的快照集就會認可至遠端 Git 存放庫。 如需詳細資料，請參閱[將 Git 儲存機制與 Azure Machine Learning Workbench 專案搭配使用](using-git-ml-project.md)。

> [!NOTE]
> Azure Machine Learning 僅支援在 Azure DevOps 中建立的 Git 存放庫。

```azure-cli
$ az ml project create --name <project name> --workspace <workspace name> --account <experimentation account name> --resource-group <resource group name> --path <local folder path> --repo <VSTS repo URL>
```
> [!TIP]
> 如果您收到錯誤訊息「存放庫 URL 可能無效或使用者可能沒有存取權」，您可以在 Azure DevOps (在_安全性_、_新增個人存取權杖_功能表底下) 中建立安全性權杖，並在建立專案時，使用 `--vststoken` 引數。 

### <a name="sample_create"></a>從範例建立新專案
在此範例中，您會使用範例專案作為範本來建立新專案。

```azure-cli
# List the project samples, find the Classifying Iris sample
$ az ml project sample list

# Create a new project from the sample
az ml project create --name <project name> --workspace <workspace name> --account <experimentation account name> --resource-group <resource group name> --path <local folder path> --template-url https://github.com/MicrosoftDocs/MachineLearningSamples-Iris
```
一旦建立您的專案，使用 `cd` 命令來輸入專案目錄。

## <a name="step-4-run-the-training-experiment"></a>步驟 4 執行訓練實驗 
下列步驟假設您具有包含 Iris 範例的專案 (請參閱[從線上範例建立新專案](#sample_create))。

### <a name="prepare-your-environment"></a>準備您的環境 
對於 Iris 範例，您必須安裝 matplotlib。

```azure-cli
$ pip install matplotlib
```

###  <a name="submit-the-experiment"></a>提交實驗

```azure-cli
# Execute the file
$ az ml experiment submit --run-configuration local iris_sklearn.py
```

### <a name="iterate-on-your-experiment-with-descending-regularization-rates"></a>隨著遞減的正規化速率反覆執行實驗
運用一些創意，只要將 Python 指令碼放在一起，即可透過不同的正規化速率提交實驗。 (您可能需要編輯檔案，以指向正確的專案路徑。)

```azure-cli
$ python run.py
```

## <a name="step-5-view-run-history"></a>步驟 5。 檢視執行歷程記錄
下列命令會列出先前執行的記錄。 

```azure-cli
$ az ml history list -o table
```
執行上述命令會顯示屬於此專案的所有執行清單。 您也可以看到精確度和正規化速率指標也一併列出。 這可以輕鬆地從清單中找出最佳執行。

## <a name="step-5a-view-attachment-created-by-a-given-run"></a>步驟 5.a 檢視由特定執行建立的附件 
若要檢視特定執行相關聯的附件，您可以使用執行歷程記錄的資訊命令。 從上述清單中尋找特定執行的執行識別碼。

```azure-cli
$ az ml history info --run <run id> --artifact driver_log
```

若要從執行下載構件，您可以使用下列命令：

```azure-cli
# Stream a given attachment 
$ az ml history info --run <run id> --artifact <artifact location>
```

## <a name="step-6-promote-artifacts-of-a-run"></a>步驟 6. 升級執行的構件 
其中一個執行有更好的 AUC；因此，這使用於建立一個評分 Web 服務以部署到生產時。 若要這麼做，您必須先將構件升級至資產。

```azure-cli
$ az ml history promote --run <run id> --artifact-path outputs/model.pkl --name model.pkl
```

這會在具有 `model.pkl.link` 檔案的專案目錄中建立 `assets` 資料夾。 此連結檔案是用來參考已升級的資產。

## <a name="step-7-download-the-files-to-be-operationalized"></a>步驟 7. 下載要實際運作的檔案
下載升級的模型，您即可使用這些檔案來建立預測 Web 服務。 

```azure-cli
$ az ml asset download --link-file assets\pickle.link -d asset_download
```

## <a name="step-8-set-up-your-model-management-environment"></a>步驟 8。 設定您的模型管理環境 
建立一個環境以部署 Web 服務。 您可以使用 Docker 在本機電腦上執行 Web 服務。 或將其部署至 ACS 叢集，以進行可高度調整的作業。 

```azure-cli
# Create new local operationalization environment
$ az ml env setup -l <supported Azure region> -n <env name>
# Once setup is complete, set your environment for current context
$ az ml env set -g <resource group name> -n <env name>
```

## <a name="step-9-create-a-model-management-account"></a>步驟 9. 建立模型管理帳戶 
需要有模型管理帳戶，才能部署及追蹤生產環境中的模型。 

```azure-cli
$ az ml account modelmanagement create -n <model management account name> -g <resource group name> -l <supported Azure region>
```

## <a name="step-10-create-a-web-service"></a>步驟 10. 建立 Web 服務
建立 Web 服務，以使用您部署的模型傳回預測。 

```azure-cli
$ az ml service create realtime -m asset_download/model.pkl -f score_iris.py -r python –n <web service name>
```

## <a name="step-11-run-the-web-service"></a>步驟 11. 執行 Web 服務
使用來自上一步驟之輸出的 Web 服務識別碼，呼叫 Web 服務並進行測試。 

```azure-cli
# Get web service usage information 
$ az ml service usage realtime -i <web service id>

# Call the web service with the run command:
$ az ml service run realtime -i <web service id> -d <input data>
```

## <a name="step-12-deleting-all-the-resources"></a>步驟 12. 刪除所有資源 
刪除所有已建立的資源以完成本教學課程，除非您想要繼續進行此作業。 

若要這麼做，請刪除包含資源的資源群組。 

```azure-cli
az group delete --name <resource group name>
```

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已經了解如何使用 Azure Machine Learning 來： 
> [!div class="checklist"]
> * 設定測試帳戶並建立工作區
> * 建立專案
> * 將實驗提交至多個計算目標
> * 升級並註冊定型模型
> * 為模型管理建立模型管理帳戶
> * 建立一個環境以供部署 Web 服務
> * 部署 Web 服務並評分新資料
