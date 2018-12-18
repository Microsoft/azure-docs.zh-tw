---
title: 使用 Git 存放庫搭配 Azure Machine Learning Workbench 專案 | Microsoft Docs
description: 本文說明如何使用 Git 存放庫搭配 Azure Machine Learning Workbench 專案。
services: machine-learning
author: hning86
ms.author: haining
manager: haining
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 11/18/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 16c102641321117f4776d761aba6c2148d15f1f5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995633"
---
# <a name="use-a-git-repo-with-a-machine-learning-workbench-project"></a>使用 Git 存放庫搭配 Machine Learning Workbench 專案

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


了解 Azure Machine Learning Workbench 如何使用 Git 提供版本控制，並確保資料科學實驗中的重現性。 了解如何將您的專案與雲端 Git 存放庫產生關聯。

Machine Learning Workbench 是針對 Git 整合所設計。 在您建立新專案時，專案資料夾會自動「Git 初始化」為本機 Git 存放庫。 系統也會建立第二個隱藏的本機 Git 存放庫，以及名稱為 AzureMLHistory/\<project GUID\> 的分支。 該分支會持續追蹤每個執行的專案資料夾變更。 

將 Azure Machine Learning 專案與 Git 存放庫關聯，便能從本機或遠端進行自動版本控制。 Git 存放庫裝載於 Azure DevOps。 由於 Machine Learning 專案與 Git 存放庫關聯，因此擁有遠端存放庫存取權的任何使用者都可以將最新的原始程式碼下載到其他電腦 (漫遊)。  

> [!NOTE]
> Azure DevOps 有自己的存取控制清單 (ACL)，與 Azure Machine Learning 測試服務無關。 Git 存放庫和 Machine Learning 工作區或專案之間的使用者存取權可能會有所不同。 您可能需要管理存取權。 
> 
> 無論您是要讓小組成員擁有 Machine Learning 專案的程式碼層級存取權，或只是要共用工作區，您都需要授與正確權限給使用者以存取 Azure DevOps Git 存放庫。 

若要使用 Git 管理版本控制，可以使用 master 分支，或在存放庫中建立其他分支。 您也可以使用本機 Git 存放庫，並推送至遠端 Git 存放庫 (如果已佈建)。

此圖表描述 Azure DevOps Git 存放庫和 Machine Learning 專案之間的關係：

![Azure Machine Learning Git](media/using-git-ml-project/aml_git.png)

若要開始使用遠端 Git 存放庫，請完成下列各節所述的步驟。

> [!NOTE]
> 目前，Azure Machine Learning 只在 Azure DevOps 組織上支援 Git 存放庫。

## <a name="step-1-create-a-machine-learning-experimentation-account"></a>步驟 1. 建立 Machine Learning 測試帳戶
建立 Machine Learning 測試帳戶並安裝 Azure Machine Learning Workbench 應用程式。 如需詳細資訊，請參閱[安裝與建立快速入門](quickstart-installation.md)。

## <a name="step-2-create-an-azure-devops-project-or-use-an-existing-project"></a>步驟 2. 建立 Azure DevOps 專案，或使用現有的專案
在 [Azure 入口網站](https://portal.azure.com/)中，建立新的專案：
1. 選取 [+]。
2. 搜尋 **Team 專案**。
3. 輸入必要資訊：
    - **名稱**：小組名稱。
    - **版本控制**：選取 [Git]。
    - **訂用帳戶**：選取擁有 Machine Learning 測試帳戶的訂用帳戶。
    - **位置**：最好選擇接近 Machine Learning 測試資源的區域。
4. 選取 [建立] 。 

![在 Azure 入口網站中建立專案](media/using-git-ml-project/create_vsts_team.png)

請確定使用您用來存取 Machine Learning Workbench 的相同 Azure Active Directory (Azure AD) 帳戶登入。 否則，系統無法使用您的 Azure AD 認證存取 Machine Learning Workbench。 例外情況是如果您使用命令列建立 Machine Learning 專案，並提供個人存取權杖以存取 Git 存放庫。 本文稍後會有詳細討論。

若要直接前往您建立的專案，請使用此 URL： https://\<專案名稱\>.visualstudio.com。

## <a name="step-3-set-up-a-machine-learning-project-and-git-repo"></a>步驟 3. 設定 Machine Learning 專案和 Git 存放庫

若要設定 Machine Learning 專案，您有兩個選擇：
- 建立具有遠端 Git 存放庫的 Machine Learning 專案
- 將現有的 Machine Learning 專案與 Azure DevOps Git 存放庫關聯

### <a name="create-a-machine-learning-project-that-has-a-remote-git-repo"></a>建立具有遠端 Git 存放庫的 Machine Learning 專案
開啟 Machine Learning Workbench 並建立新的專案。 在 [Git 存放庫] 方塊中，輸入步驟 2 的 Azure DevOps Git 存放庫 URL。 它通常看起來像這樣： https://\<Azure DevOps 組織名稱\>.visualstudio.com/_git/\<專案名稱\>

![建立具有 Git 存放庫的 Machine Learning 專案](media/using-git-ml-project/create_project_with_git_rep.png)

您也可以使用 Azure 命令列工具 (Azure CLI) 建立專案。 您可以選擇輸入個人存取權杖。 Machine Learning 可以使用此權杖存取 Git 存放庫，而不是使用您的 Azure AD 認證：

```
# Create a new project that has a Git repo by using a personal access token.
$ az ml project create -a <Experimentation account name> -n <project name> -g <resource group name> -w <workspace name> -r <Git repo URL> --vststoken <Azure DevOps personal access token>
```

> [!IMPORTANT]
> 如果選擇空白的專案範本，您選擇使用的 Git 存放庫可能會已具有 master 分支。 Machine Learning 只會在本機複製 master 分支。 它會將 aml_config 資料夾與其他專案中繼資料檔案新增至本機專案資料夾。 
>
> 但如果您選擇任何其他專案範本，您的 Git 存放庫*不能*已經具有 master 分支。 如果有，就會發生錯誤。 替代方式為使用 `az ml project create` 命令搭配 `--force` 參數建立專案。 這會刪除原始 master 分支中的檔案，並以所選範本中的新檔案取代它們。

現在已建立新的 Machine Learning 專案，而且已啟用遠端 Git 存放庫整合。 專案資料夾永遠 Git 初始化為本機 Git 存放庫。 Git 遠端會設定為遠端 Azure DevOps Git 存放庫，因此可以將認可推送至遠端 Git 存放庫。

### <a name="associate-an-existing-machine-learning-project-with-an-azure-devops-git-repo"></a>將現有的 Machine Learning 專案與 Azure DevOps Git 存放庫關聯
您可以建立不含 Azure DevOps Git 存放庫的 Machine Learning 專案，而只依賴本機 Git 存放庫來取得執行歷程記錄快照集。 之後可以使用下列命令，將 Azure DevOps Git 存放庫與這個現有的 Machine Learning 專案關聯：

```azurecli
# Ensure that you are in the project path so Azure CLI has the context of your current project.
$ az ml project update --repo https://<Azure DevOps organization name>.visualstudio.com/_git/<project name>
```

> [!NOTE] 
> 您只能在沒有與 Git 存放庫關聯的 Machine Learning 專案上執行更新存放庫作業。 而且，將 Git 存放庫與 Machine Learning 關聯之後，就無法將它移除。

## <a name="step-4-capture-a-project-snapshot-in-the-git-repo"></a>步驟 4. 在 Git 存放庫中擷取專案快照集
在專案中執行幾個指令碼，並於每次執行之間做些變更。 您可以在桌面應用程式中進行此作業，或使用 `az ml experiment submit` 命令從 Azure CLI 進行。 如需詳細資訊，請參閱[將鳶尾花教學課程分類](tutorial-classifying-iris-part-1.md)。 每次執行時，如果專案資料夾中的任何檔案有任何變更，則會認可整個專案資料夾的快照集，並推送至名為 AzureMLHistory/\<專案 GUID\> 之分支下的遠端 Git 存放庫。 若要檢視分支和認可 (包括 AzureMLHistory/\<專案 GUID\> 分支)，請前往 Azure DevOps Git 存放庫 URL。 

> [!NOTE] 
> 系統僅在指令碼執行之前認可快照集。 目前，資料準備執行或 Notebook 資料格執行不會觸發快照集。

![執行歷程記錄分支](media/using-git-ml-project/run_history_branch.png)

> [!IMPORTANT] 
> 最好不要使用 Git 命令在歷程記錄分支中操作。 這可能會影響執行歷程記錄。 請改為使用 master 分支，或為您自己的 Git 作業建立其他分支。

## <a name="step-5-restore-a-previous-project-snapshot"></a>步驟 5。 還原先前的專案快照集 
若要將整個專案資料夾還原至先前的執行歷程記錄快照集狀態，請在 Machine Learning Workbench 中執行下列動作：
1. 在活動列 (沙漏圖示) 中，選取 [執行]。
2. 在 [執行清單] 檢視中，選取想要還原的執行。
3. 在 [執行詳細資料] 檢視中，選取 [還原]。

![執行歷程記錄分支](media/using-git-ml-project/restore_project.png)

您可以選擇性地在 Machine Learning Workbench 的 Azure CLI 視窗中使用下列命令：

```azurecli
# Discover the run I want to restore a snapshot from.
$ az ml history list -o table

# Restore the snapshot from a specific run.
$ az ml project restore --run-id <run ID>
```

請謹慎執行此命令。 執行此命令時，會以該特定執行開始時所建立的快照集覆寫整個專案資料夾。 您的專案會維持在最新分支上。 這表示您會遺失目前的專案資料夾中的**所有變更**。  

在執行此作業之前，您應該使用 Git 將變更認可到最新分支。

## <a name="step-6-use-the-master-branch"></a>步驟 6. 使用主要分支
為了避免不小心遺失目前的專案狀態，有一種作法是將專案認可到 Git 存放庫的 master 分支 (或您自行建立的任何分支)。 您可以從命令列或慣用的 Git 用戶端工具使用 Git，在 master 分支上進行操作。 例如︰

```sh
# Check status to make sure you are on the master branch (or branch of your choice).
$ git status

# Stage all changes.
$ git add -A

# Commit all changes locally on the master branch.
$ git commit -m 'these are my updates so far'

# Push changes to the remote Azure DevOps Git repo master branch.
$ git push origin master
```

現在，您可以完成步驟 5，安全地將專案還原至較早的快照集。 您隨時可以回到剛剛在 master 分支上所做的認可。

## <a name="authentication"></a>驗證
如果您只是依賴 Machine Learning 中的執行歷程記錄功能建立專案快照集以及將它們還原，則不需要擔心 Git 存放庫驗證。 Machine Learning 測試服務層會處理驗證。

不過，如果使用自己的 Git 工具管理版本控制，則必須在 Azure DevOps 中處理針對遠端 Git 存放庫的驗證。 在 Machine Learning 中，會使用 HTTPS 通訊協定將遠端 Git 存放庫新增至本機存放庫作為 Git 遠端。 這表示當您發出 Git 命令 (例如推送或提取) 至遠端時，需要提供使用者名稱和密碼，或個人存取權杖。 若要在 Azure DevOps Git 存放庫中建立個人存取權杖，請依照[使用個人存取權杖進行驗證](https://docs.microsoft.com/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate) \(英文\) 中的指示進行。

## <a name="next-steps"></a>後續步驟
- 了解如何[使用 Team 資料科學程序來組織專案結構](how-to-use-tdsp-in-azure-ml.md)。
