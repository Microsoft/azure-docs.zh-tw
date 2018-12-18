---
title: Team 資料科學程序專案負責人工作 - Azure | Microsoft Docs
description: 資料科學 Team 專案上專案負責人工作的概述。
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 6a618efc6860371883bff7ebb953880293ad3120
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303876"
---
# <a name="project-lead-tasks"></a>專案負責人工作

本教學課程概述希望專案負責人為他/她的專案小組完成的工作。 目標是建立在 [Team 資料科學程序](overview.md) (TDSP) 上標準化的共同作業小組環境。 TDSP 是 Microsoft 開發的架構，提供結構化的一系列活動，以有效率地執行雲端式、預測性分析解決方案。 如需人員角色的概述，以及對此程序進行標準化之資料科學小組所處理相關聯工作的大綱，請參閱 [Team 資料科學程序角色和工作](roles-tasks.md)。

**專案負責人**會管理特定資料科學專案的個別資料科學家日常活動。 由專案負責人完成以便設定此環境之工作的工作流程，如下圖所述：

![1](./media/project-lead-tasks/project-leads-1-tdsp-creating-projects.png)

本主題目前涵蓋專案負責人此工作流程的工作 1、2 和 6。

>[AZURE.NOTE] 我們概述在下列指示中使用 Azure DevOps 為專案設定 TDSP 小組環境所需的步驟。 我們會指定如何使用 Azure DevOps 完成這些工作，因為這是我們在 Microsoft 中實作 TDSP 的方式。 如果針對您的群組使用另一個程式碼裝載平台，必須由小組負責人完成的工作通常不會變更。 但是完成這些工作的方式將會不同。


## <a name="repositories-and-directories"></a>存放庫和目錄

本教學課程會使用存放庫和目錄的縮寫名稱。 這些名稱讓您更容易遵循存放庫和目錄之間的作業。 這個標記法 (R 適用於 Git 存放庫、D 適用於您 DSVM 上的本機目錄) 在下列各節中使用：

- **R3**：Git 上的小組 **ProjectTemplate** 存放庫，您的小組負責人已設定。
- **R5**：您為專案設定之 Git 上的專案存放庫。
- **D3**：從 R3 複製的本機目錄。
- **D5**：從 R5 複製的本機目錄。


## <a name="0-prerequisites"></a>0.必要條件

完成[資料科學小組的群組管理員工作](group-manager-tasks.md)中所述，指派給您的群組管理員的工作，以及[資料科學小組的小組負責人工作](team-lead-tasks.md)中所述，指派給您的小組負責人的工作，即可滿足必要條件。 

簡單來說，就是在您開始小組負責人工作之前，必須符合下列需求： 

- 您的**群組 Azure DevOps Services** (或其他某些程式碼裝載平台上的群組帳戶) 已由您的群組管理員設定。
- 您的 **TeamProjectTemplate 存放庫** (R3) 已由您計劃使用之程式碼裝載平台上的小組負責人，在群組帳戶之下設定。
- 您已經被小組負責人**授權**，為您的小組在群組帳戶上建立存放庫。
- Git 必須安裝在您的機器上。 如果您使用資料科學虛擬機器 (DSVM)，則已預先安裝 Git，而您可以繼續作業。 否則，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)。  
- 如果您使用 **Windows DSVM**，您必須在機器上安裝 [Git 認證管理員 (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) \(英文\)。 在 README.md 檔案中，向下捲動至 [下載並安裝] 區段，然後按一下 [最新的安裝程式]。 這樣會帶您到最新的安裝程式分頁。 從這裡下載 .exe 安裝程式並執行它。 
- 如果您使用 **Linux DSVM**，請在您的 DSVM 上建立 SSH 公開金鑰，並將它新增到您的群組 Azure DevOps Services。 如需 SSH 的詳細資訊，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)中的**建立 SSH 公開金鑰**一節。 


## <a name="1-create-a-project-repository-r5"></a>1.建立專案存放庫 (R5)

- 經由 *https://\<Azure DevOps Services 名稱\>.visualstudio.com* 登入您的群組 Azure DevOps Services。 
- 在 [最近使用的專案和小組] 底下，按一下 [瀏覽]。 快顯視窗會列出 Azure DevOps Services 上的所有專案。 

    ![2](./media/project-lead-tasks/project-leads-2-create-project-repo.png)

- 按一下您要在其中建立專案存放庫的專案名稱。 在此範例中，按一下 [MyTeam]。 
- 然後，按一下 [瀏覽] 以導向至專案 **MyTeam** 的首頁：

    ![3](./media/project-lead-tasks/project-leads-3-create-project-repo-2.png)

- 按一下 [在程式碼上共同作業] 以導向至專案的 git 首頁。  

    ![4](./media/project-lead-tasks/project-leads-4-create-project-repo-3.png)

- 按一下左上角的向下箭號，然後選取 [+ 新增存放庫]。 
    
    ![5](./media/project-lead-tasks/project-leads-5-create-project-repo-4.png)

- 在 [建立新的存放庫] 視窗中，輸入專案 git 存放庫的名稱。 請確定您選取 **Git** 作為存放庫類型。 在此範例中，我們會使用名稱 *DSProject1*。 

    ![6](./media/project-lead-tasks/project-leads-6-create-project-repo-5.png)

- 若要建立 ***DSProject1*** 專案 git 存放庫，請按一下 [建立]。


## <a name="2-seed-the-dsproject1-project-repository"></a>2.植入 DSProject1 專案存放庫

以下工作是從您的專案範本存放庫 (R3) 植入 **DSProject1** 專案存放庫 (R5)。 植入程序會使用本機 DSVM 上的目錄 D3 和 D5，作為中繼暫存網站。 綜上所述，植入路徑是：R3 -> D3 -> D5 -> R5。

如果您需要自訂 **DSProject1** 專案存放庫以符合一些特定專案需求，您要在下列程序的倒數第二個步驟執行這項操作。 以下是用來植入 **DSProject1** 專案存放庫內容的步驟摘要。 個別步驟對應至植入程序中的各小節：

- 將專案範本存放庫複製到本機目錄：Team R3 - 複製到 -> 本機 D3。
- 將 DSProject1 存放庫複製到本機目錄：Team R5 - 複製到 -> 本機 D5。
- 將複製的專案範本內容複製到 DSProject1 存放庫的本機複製：D3 - 內容複製到 -> D5。
- (選擇性) 自訂本機 D5。
- 將本機 DSProject1 內容推送到 Team 存放庫：D5 - 內容新增至 -> Team R5。


### <a name="clone-your-project-template-repository-r3-to-a-directory-d3-on-your-local-machine"></a>將您的專案範本存放庫 (R3) 複製到本機電腦上的目錄 (D3)。

在您的本機電腦上建立目錄：

- *C:\GitRepos\MyTeamCommon* (適用於 Windows) 
- *$home/GitRepos/MyTeamCommon* (適用於 Linux)

變更為該目錄。 然後，執行下列命令，將您的專案範本存放庫複製到本機電腦。 

**Windows**
            
    git clone <the HTTPS URL of the TeamProjectTemplate repository>
    
如果您使用 Azure DevOps 作為程式碼裝載平台，*專案範本存放庫的 HTTPS URL* 通常是：

 ***https://\<Azure DevOps Services 名稱\>.visualstudio.com/\<您的專案名稱\>/_git/\<您的專案範本存放庫名稱\>***。 

在此範例中，我們有：

***https://mysamplegroup.visualstudio.com/MyTeam/_git/MyTeamProjectTemplate***。 

![7](./media/project-lead-tasks/project-leads-7-clone-team-project-template.png)
            
**Linux**

    git clone <the SSH URL of the TeamProjectTemplate repository>
        
![8](./media/project-lead-tasks/project-leads-8-clone-team-project-template-linux.png)

如果您使用 Azure DevOps 作為程式碼裝載平台，*專案範本存放庫的 SSH URL* 通常是：

***ssh://\<Azure DevOps Services 名稱\>@\<Azure DevOps Services 名稱\>.visualstudio.com:22/\<您的專案名稱>/_git/\<您的專案範本存放庫名稱\>***。 

在此範例中，我們有：

***ssh://mysamplegroup@mysamplegroup.visualstudio.com:22/MyTeam/_git/MyTeamProjectTemplate***。 

### <a name="clone-dsproject1-repository-r5-to-a-directory-d5-on-your-local-machine"></a>將 DSProject1 存放庫 (R5) 複製到本機電腦上的目錄 (D5)

將目錄變更為 **GitRepos**，然後執行下列命令以將您的專案存放庫複製到本機電腦。 

**Windows**
            
    git clone <the HTTPS URL of the Project repository>

![9](./media/project-lead-tasks/project-leads-9-clone-project-repository.png)

如果您使用 Azure DevOps 作為程式碼裝載平台，_專案存放庫的 HTTPS URL_ 通常是 ***https://\<Azure DevOps Services 伺服器名稱\>.visualstudio.com/\<您的專案名稱>/_git/<您的專案存放庫名稱\>***。 在此範例中，我們有 ***https://mysamplegroup.visualstudio.com/MyTeam/_git/DSProject1***。

**Linux**

    git clone <the SSH URL of the Project repository>

![10](./media/project-lead-tasks/project-leads-10-clone-project-repository-linux.png)

如果您使用 Azure DevOps 作為程式碼裝載平台，_專案存放庫的 SSH URL_ 通常是 _ssh://<Azure DevOps Services 名稱\>@<Azure DevOps Services 名稱\>.visualstudio.com:22/<Your Project Name>/\_git/<您的專案存放庫名稱\>。 在此範例中，我們有 ***ssh://mysamplegroup@mysamplegroup.visualstudio.com:22/MyTeam/_git/DSProject1***。

### <a name="copy-contents-of-d3-to-d5"></a>將 D3 的內容複製到 D5 

現在在您的本機電腦上，您必須將 _D3_ 的內容複製到 _D5_，.git 目錄中的 git 中繼資料除外。 下列指令碼會執行作業。 請確定輸入目錄的正確且完整路徑。 來源資料夾是您的小組的其中一個 (_D3_)；目的地資料夾是您的專案的其中一個 (_D5_)。    

**Windows**
    
    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/tdsp_local_copy_win.ps1" -outfile "tdsp_local_copy_win.ps1"
    .\tdsp_local_copy_win.ps1 -role 3
    
![11](./media/project-lead-tasks/project-leads-11-local-copy-project-lead-new.png)

現在您可以查看 _DSProject1_ 資料夾，所有檔案 (包括 .git) 都是從 _MyTeamProjectTemplate_ 複製。

![12](./media/project-lead-tasks/project-leads-12-teamprojectTemplate_copied_to_local.png)

**Linux**
            
    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/tdsp_local_copy_linux.sh"
    bash tdsp_local_copy_linux.sh 3
        
![13](./media/project-lead-tasks/project-leads-13-local_copy_project_lead_linux_new.png)

現在您可以查看 _DSProject1_ 資料夾，所有檔案 (.git 中的中繼資料除外) 都是從 _MyTeamProjectTemplate_ 複製。

![14](./media/project-lead-tasks/project-leads-14-teamprojectTemplate_copied_to_local_linux_new.png)


### <a name="customize-d5-if-you-need-to-optional"></a>視需要自訂 D5 (選擇性)

如果您的專案需要一些特定目錄或文件，而不是您從專案範本取得的項目 (在上一個步驟中複製到 D5 目錄)，您可於此時自訂 D5 的內容。 

### <a name="add-contents-of-dsproject1-in-d5-to-r5-on-your-group-azure-devops-services"></a>將 D5 中 DSProject1 的內容新增至群組 Azure DevOps Services 上的 R5

現在，您必須將 **_DSProject1_** 中的內容推送至群組 Azure DevOps Services 上設定於專案中的 _R5_ 存放庫。 


- 將目錄變更為 **D5**。 
- 使用下列 git 命令，將 **D5** 中的內容新增至 **R5**。 適用於 Windows 和 Linux 系統的命令是相同的。 
    
    git status git add .
    git commit -m"push from win DSVM" git push
    
- 認可變更並推送。 

>[AZURE.NOTE] 如果這是第一次認可 Git 存放庫，您必須在執行 `git commit` 命令之前，設定全域參數 user.name 和 user.email。 執行下列兩個命令：
        
    git config --global user.name <your name>
    git config --global user.email <your email address>
 
> 如果您認可多個 Git 存放庫，請對於所有存放庫都使用相同名稱和電子郵件地址。 使用相同名稱和電子郵件地址，會讓您日後建置 PowerBI 儀表板以追蹤多個存放庫上的 Git 活動時，更為便利。

![15](./media/project-lead-tasks/project-leads-15-git-config-name.png)


## <a name="6-create-and-mount-azure-file-storage-as-project-resources-optional"></a>6.建立並掛接 Azure 檔案儲存體作為專案資源 (選擇性)

如果您想要建立 Azure 檔案儲存體以共用資料，例如專案未經處理資料或為您的專案產生的功能，讓所有專案成員都具有多個 DSVM 之相同資料集的存取權，請遵循[資料科學小組的小組負責人工作](team-lead-tasks.md)第 3 節和第 4 節中的指示。 


## <a name="next-steps"></a>後續步驟

以下是 Team 資料科學程序定義之角色和工作更詳細描述的連結：

- [資料科學小組的群組管理員工作](group-manager-tasks.md)
- [資料科學小組的小組負責人工作](team-lead-tasks.md)
- [資料科學小組的專案負責人工作](project-lead-tasks.md)
- [資料科學小組的專案個別參與者](project-ic-tasks.md)