---
title: 在 Azure Machine Learning Workbench 中漫遊及共同作業 | Microsoft Docs
description: 了解如何在 Machine Learning Workbench 中設定漫遊及共同作業。
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 11/16/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 07a9d46dff17b43d01a6b411292cf240c32476f3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983713"
---
# <a name="roaming-and-collaboration-in-azure-machine-learning-workbench"></a>在 Azure Machine Learning Workbench 中漫遊及共同作業

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

本文說明如何使用 Azure Machine Learning Workbench 設定專案，以在電腦之間漫遊以及和小組成員共同作業。 

當您建立具有遠端 Git 存放庫連結 的 Azure Machine Learning 專案時，專案中繼資料和快照集會儲存在雲端。 您可以使用雲端連結從不同的電腦存取專案 (漫遊)。 也可以提供專案存取權給小組成員，來和他們共同作業。 

## <a name="prerequisites"></a>必要條件
1. 安裝 Machine Learning Workbench 應用程式。 確定您擁有 Azure Machine Learning 測試帳戶的存取權。 如需詳細資訊，請參閱[安裝指南](quickstart-installation.md)。

2. 存取 [Azure DevOps](https://www.visualstudio.com)，並建立要連結專案的存放庫。 如需詳細資訊，請參閱[使用 Git 存放庫搭配 Machine Learning Workbench 專案](using-git-ml-project.md)。

## <a name="create-a-new-machine-learning-project"></a>建立新的 Machine Learning 專案
開啟 Machine Learning Workbench，然後建立新的專案 (例如，名為鳶尾花的專案)。 在 [Visualstudio.com GIT 存放庫 URL] 方塊中，輸入 Azure DevOps Git 存放庫的有效 URL。 

> [!IMPORTANT]
> 如果選擇空白的專案範本，您選擇使用的 Git 存放庫可能會已具有 master 分支。 Machine Learning 只會在本機複製 master 分支。 它會將 aml_config 資料夾與其他專案中繼資料檔案新增至本機專案資料夾。 
>
> 但如果您選擇任何其他專案範本，您的 Git 存放庫*不能*已經具有 master 分支。 如果有，就會發生錯誤。 替代方式為使用 `az ml project create` 命令搭配 `--force` 參數建立專案。 這會刪除原始 master 分支中的檔案，並以所選範本中的新檔案取代它們。

建立專案之後，針對專案內的任何指令碼，提交幾次執行。 這個動作會將專案狀態認可至遠端 Git 存放庫的執行歷程記錄分支。 

> [!NOTE] 
> 只有執行觸發程序的指令碼會認可至執行歷程記錄分支。 資料準備執行和 Notebook 執行不會在執行歷程記錄分支上觸發專案快照集。

如果您已經設定 Git 驗證，也可以在 master 分支中操作。 您也可以建立新的分支。 

範例： 
```
# Check current repo status.
$ git status

# Stage all changes in the current repo.
$ git add -A

# Commit changes.
$ git commit -m "my commit fixes this weird bug!"

# Push to the remote repo.
$ git push origin master
```

## <a name="roaming"></a>漫遊
<a name="roaming"></a>

### <a name="open-machine-learning-workbench-on-a-second-computer"></a>在第二部電腦上開啟 Machine Learning Workbench
Azure DevOps Git 存放庫與您的專案連結後，就可以從已安裝 Machine Learning Workbench 的任何電腦上存取鳶尾花專案。 

若要從另一部電腦上存取鳶尾花專案，必須使用建立專案時所使用的相同認證來登入應用程式。 您也需要使用相同的 Machine Learning 測試帳戶與工作區。 鳶尾花專案會依字母順序與工作區內的其他專案一起列出。 

### <a name="download-the-project-on-a-second-computer"></a>在第二部電腦上下載專案
在第二部電腦上開啟工作區時，與鳶尾花專案相鄰的圖示會與典型資料夾圖示不同。 下載圖示指出專案的內容位於雲端，且專案已經可以下載到目前的電腦。 

![建立專案](./media/roaming-and-collaboration/downloadable-project.png)

選取鳶尾花專案以開始下載。 當下載完成時，該專案便可在第二部電腦上存取。 

在 Windows 上，專案會位於 C:\Users\\<使用者名稱\>\Documents\AzureML。

在 macOS 上，專案會位於 /home/\<使用者名稱\>/Documents/AzureML。

在未來版本中，我們計畫增強功能，讓您可以選取目的地資料夾。 

> [!NOTE]
> 如果您在 Machine Learning 目錄中有名稱與專案完全相同的資料夾，下載就會失敗。 若要解決這個問題，請暫時將現有的資料夾重新命名。


### <a name="work-on-the-downloaded-project"></a>使用下載的專案 
新下載的專案會反映專案中上次執行的專案狀態。 每次提交執行時，專案狀態的快照集會自動認可到 Azure DevOps Git 存放庫中的執行歷程記錄分支。 在第二部電腦上具現化專案時，會使用與最新執行相關聯的快照集。 
 

## <a name="collaboration"></a>共同作業
您可以在連結至 Azure DevOps Git 存放庫的專案上和小組成員共同作業。 您可以針對 Machine Learning 測試帳戶、工作區及專案，將權限指派給使用者。 目前您可以使用 Azure CLI 執行 Azure Resource Manager 命令。 您也可以使用 [Azure 入口網站](https://portal.azure.com)。 如需詳細資訊，請參閱[使用 Azure 入口網站新增使用者](#portal)。    

### <a name="use-the-command-line-to-add-users"></a>使用命令列新增使用者
舉例來說，Alice 為鳶尾花專案的擁有者。 Alice 想要和 Bob 共用該專案的存取權。 

Alice 選取 [檔案] 功能表，然後選取 [命令提示字元] 功能表項目。 系統會開啟具有鳶尾花專案的 [命令提示字元] 視窗。 Alice 接著可以決定要授與 Bob 的存取層級。 她可以透過執行下列命令來授予權限：  

```azurecli
# Find the Resource Manager ID of the Experimentation account.
az ml account experimentation show --query "id"

# Add Bob to the Experimentation account as a Contributor.
# Bob now has read/write access to all workspaces and projects under the account by inheritance.
az role assignment create --assignee bob@contoso.com --role Contributor --scope <Experimentation account Resource Manager ID>

# Find the Resource Manager ID of the workspace.
az ml workspace show --query "id"

# Add Bob to the workspace as an Owner.
# Bob now has read/write access to all projects under the workspace by inheritance. Bob can invite or remove other users.
az role assignment create --assignee bob@contoso.com --role Owner --scope <workspace Resource Manager ID>
```

角色指派之後 (無論是透過直接指派或繼承方式)，Bob 就可以在 Machine Learning Workbench 專案清單中看見該專案。 Bob 可能需要重新啟動應用程式才能看到專案。 接著，Bob 可以透過[漫遊](#roaming)中所述的方式下載該專案，並開始與 Alice 共同作業。 

在專案上共同作業之所有使用者的執行歷程記錄，都會認可到相同的遠端 Git 存放庫。 當 Alice 進行執行時，Bob 可以在 Machine Learning Workbench 應用程式中，於該專案的執行歷程記錄區段中看到該執行。 Bob 也可以將專案還原成任何執行的狀態，包括由 Alice 所啟動的執行。 

透過共用專案的遠端 Git 存放庫，也可讓 Alice 和 Bob 在 master 分支上共同作業。 如有需要，他們也可以建立個人分支，並使用 Git 提取要求及合併來共同作業。 

### <a name="use-the-azure-portal-to-add-users"></a>使用 Azure 入口網站來新增使用者
<a name="portal"></a>

Machine Learning 測試帳戶、工作區及專案都是 Azure Resource Manager 資源。 若要指派角色，您可以使用 [Azure 入口網站](https://portal.azure.com)中的 [存取控制] 連結。 

使用 [所有資源] 檢視來找出要新增使用者的資源。 選取 [存取控制 (IAM)] 連結，然後選取 [新增使用者]。 

<img src="./media/roaming-and-collaboration/iam.png" width="320px">

## <a name="sample-collaboration-workflow"></a>共同作業工作流程範例
讓我們透過範例來了解共同作業的流程。 Contoso 員工 Alice 和 Bob 想要透過使用 Machine Learning Workbench，來在某個資料科學專案上共同作業。 他們的身分識別屬於相同的 Contoso Azure Active Directory (Azure AD) 租用戶。 Alice 和 Bob 採取的步驟如下：

1. Alice 在 Azure DevOps 專案中建立一個空白的 Git 存放庫。 該 Azure DevOps 專案應位於在 Contoso Azure AD 租用戶之下建立的 Azure 訂用帳戶中。 

2. Alice 在她的電腦上建立 Machine Learning 測試帳戶、工作區，以及 Machine Learning Workbench 專案。 她在建立專案時輸入 Azure DevOps Git 存放庫 URL。

3. Alice 開始在專案上工作。 她建立了一些指令碼並執行了幾個執行。 每次執行時，系統會將整個專案資料夾的快照集以認可的形式自動推送 Machine Learning Workbench 所建立之 Azure DevOps Git 存放庫的執行歷程記錄分支。

4. Alice 對於進行中的工作內容感到很滿意。 她想要認可她在本機 master 分支中所做的變更，並將該變更推送 Azure DevOps Git 存放庫 master 分支中。 在 Machine Learning Workbench 中開啟該專案的情況下，她開啟 [命令提示字元] 視窗，然後輸入下列命令：
    
    ```sh
    # Verify that the Git remote is pointing to the Azure DevOps Git repo.
    $ git remote -v

    # Verify that the current branch is master.
    $ git branch

    # Stage all changes.
    $ git add -A

    # Commit changes with a comment.
    $ git commit -m "this is a good milestone"

    # Push the commit to the master branch of the remote Git repo in Azure DevOps.
    $ git push
    ```

5. Alice 將 Bob 作為參與者新增到工作區。 她可以在 Azure 入口網站中進行此作業，或使用較早之前所述的 `az role assignment` 命令。 Alice 也將 Azure DevOps Git 存放庫的讀取/寫入權限授與 Bob。

6. Bob 在他的電腦上登入 Machine Learning Workbench。 他可以看到 Alice 和他共用的工作區。 他可以看到列於工作區底下的鳶尾花專案。 

7. Bob 選取專案名稱。 該專案會下載到他的電腦上。
    * 下載的專案檔是執行歷程記錄中所記錄之最近一次執行的快照集複本。 它們並不是 master 分支中最新的認可。
    * 本機專案資料夾會設定在 master 分支上，並含有取消暫存的變更。

8. Bob 可以瀏覽由 Alice 執行的執行。 他可以還原任何先前執行的快照集。

9. Bob 想要取得 Alice 推送的最新變更，然後開始在不同的分支中作業。 Bob 在 Machine Learning Workbench 中開啟 [命令提示字元] 視窗，然後執行下列命令：

    ```sh
    # Verify that the Git remote is pointing to the Azure DevOps Git repo.
    $ git remote -v

    # Verify that the current branch is master.
    $ git branch

    # Get the latest commit in the Azure DevOps Git master branch and overwrite current files.
    $ git pull --force

    # Create a new local branch named "bob" so that Bob's work is done in the "bob" branch
    $ git checkout -b bob
    ```

10. Bob 修改專案並提交新的執行。 變更是在 bob 分支上進行。 Alice 也可以看到 Bob 的執行。

11. Bob 已準備好將他的變更推送到遠端 Git 存放庫。 為避免和 Alice 正在處理的 master 分支發生衝突，Bob 將他的工作推送到也稱為 bob 的新遠端分支。

    ```sh
    # Verify that the current branch is "bob," and that it has unstaged changes.
    $ git status
    
    # Stage all changes.
    $ git add -A

    # Commit the changes with a comment.
    $ git commit -m "I found a cool new trick."

    # Create a new branch on the remote Azure DevOps Git repo, and then push the changes.
    $ git push origin bob
    ```

12. 為了向 Alice 示範他的酷炫新程式碼，Bob 在遠端 Git 存放庫上建立從 bob 分支到 master 分支的提取要求。 Alice 可以將該提取要求合併到 master 分支中。

## <a name="next-steps"></a>後續步驟
- 深入了解[使用 Git 存放庫搭配 Machine Learning Workbench 專案](using-git-ml-project.md)。
