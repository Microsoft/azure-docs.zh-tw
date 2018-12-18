---
title: Team 資料科學程序小組負責人工作 - Azure | Microsoft Docs
description: 資料科學 Team 專案上小組負責人工作的概述。
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
ms.openlocfilehash: 86ab49cb0acd9ffee47fb1f8f531c3a0cd6e6730
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297956"
---
# <a name="team-lead-tasks"></a>小組負責人工作

本主題概述希望小組負責人為其資料科學小組完成的工作。 目標是建立在 [Team 資料科學程序](overview.md) (TDSP) 上標準化的共同作業小組環境。 TDSP 是一種敏捷式反覆資料科學方法，可有效產出預測分析解決方案和智慧型應用程式。 它的設計目的是協助改進共同作業和小組學習。 程序是從 Microsoft 以及產業淬練出來的最佳作法和結構，是成功實作資料科學不可或缺的，以便協助公司完全實現其分析程式的優點。 如需人員角色的大綱，以及對此程序進行標準化之資料科學小組所處理相關聯工作的大綱，請參閱 [Team 資料科學程序角色和工作](roles-tasks.md)。

**小組負責人**會管理企業資料科學單位中的小組。 小組是由多個資料科學家所組成。 若為只有少數資料科學家的資料科學單位，**群組管理員**和**小組負責人**可能是同一人，或者可以將其工作委派給代理人。 但是，工作本身不會變更。 由小組負責人完成以便設定此環境之工作的工作流程，如下圖所述：

![1](./media/team-lead-tasks/team-leads-1-creating-teams.png)

>[AZURE.NOTE] 如果您使用 Azure DevOps 作為程式碼裝載平台，而且您想要讓自己的小組有個別的 Azure DevOps 專案，則需要上圖區塊 1 和 2 中的工作。 一旦這些工作完成，就可以在此專案底下建立您小組的所有存放庫。 

由群組管理員滿足後續章節中指定的數個必要條件工作之後，有五個您要在此教學課程完成的主體工作 (有些是選擇性的)。 這些工作對應本主題的主要編號章節：

1. 在群組的 Azure DevOps Services 伺服器上建立**專案**，以及在專案中建立兩個小組存放庫：
    - **ProjectTemplate 存放庫** 
    - **TeamUtilities 存放庫**
2. 從 **GroupProjectTemplate** 存放庫植入 **ProjectTemplate** 存放庫，前者已由群組管理員設定。 
3. 建立小組資料與分析資源：
    - 將小組特定公用程式新增至 **TeamUtilities** 存放庫。 
    - (選擇性) 建立 **Azure 檔案儲存體**，用來儲存對整個小組而言很實用的資料資產。 
4. (選擇性) 將 Azure 檔案儲存體掛接至小組負責人的**資料科學虛擬機器** (DSVM) 並在其上新增資料資產。
5. 藉由新增小組成員和設定其權限來設定**安全性控制**。

>[AZURE.NOTE] 我們概述在下列指示中使用 Azure DevOps 設定 TDSP 小組環境所需的步驟。 我們會指定如何使用 Azure DevOps 完成這些工作，因為這是我們在 Microsoft 中實作 TDSP 的方式。 如果針對您的群組使用另一個程式碼裝載平台，必須由小組負責人完成的工作通常不會變更。 但是完成這些工作的方式將會不同。

## <a name="repositories-and-directories"></a>存放庫和目錄

本主題會使用存放庫和目錄的縮寫名稱。 這些名稱讓您更容易遵循存放庫和目錄之間的作業。 這個標記法 (**R** 適用於 Git 存放庫、**D** 適用於您 DSVM 上的本機目錄) 在下列各節中使用：

- **R1**：Git 上的 **GroupProjectTemplate** 存放庫，由您的群組管理員設定於 Azure DevOps 群組伺服器上。
- **R3**：您設定之 Git 上的小組 **ProjectTemplate** 存放庫。
- **R4**：您設定之 Git 上的 **TeamUtilities** 存放庫。
- **D1**：從 R1 複製並且複製到 D3 的本機目錄。
- **D3**：從 R3 複製、自訂，並且複製回 R3 的本機目錄。
- **D4**：從 R4 複製、自訂，並且複製回 R4 的本機目錄。

本教學課程中為存放庫和目錄指定的名稱，均假設您的目標是為了要在較大的資料科學群組中為您自己的小組建立個別專案。 但是還有其他選項為身為小組負責人的您開放：

- 整個群組可以選擇建立單一專案。 其後，所有資料科學小組的所有專案都會在這個單一專案底下。 為此，您可以指定 git 系統管理員須遵循這些指示來建立單一專案。 這個案例對於下列情形有效，例如：
    -  沒有多個資料科學小組的小型資料科學群組 
    -  具有多個資料科學小組的較大資料科學群組，想要使用例如群組層級短期衝刺計劃的活動來最佳化小組間共同作業。 
- 小組可以選擇讓整個群組的單一專案下包含小組特定專案範本或小組特定公用程式。 在此情況下，小組負責人應該在相同的專案下建立專案範本存放庫和/或小組公用程式存放庫。 將這些存放庫命名為 <TeamName\>ProjectTemplate 和 <TeamName\>Utilities，例如，TeamJohnProjectTemplate 和 TeamJohnUtilities。 

在任何情況下，小組負責人都必須讓小組成員知道當他們設定及複製專案和公用程式存放庫時，要採用哪個範本和公用程式存放庫。 專案負責人應遵循[資料科學小組的專案負責人工作](project-lead-tasks.md)以建立專案存放庫，無論是在個別專案下還是單一專案下。 


## <a name="0-prerequisites"></a>0.必要條件

完成[資料科學小組的群組管理員工作](group-manager-tasks.md)中所述，指派給您的群組管理員的工作，即可滿足必要條件。 簡單來說，就是在您開始小組負責人工作之前，必須符合下列需求： 

- 您的**群組 Azure DevOps Services** (或其他某些程式碼裝載平台上的群組帳戶) 已由您的群組管理員設定。
- 您的 **GroupProjectTemplate 存放庫** (R1) 已由您計劃使用之程式碼裝載平台上的群組管理員，在群組帳戶上設定。
- 您已經被群組管理員**授權**，為您的小組建立存放庫。
- Git 必須安裝在您的機器上。 如果您使用資料科學虛擬機器 (DSVM)，則已預先安裝 Git，而您可以繼續作業。 否則，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)。  
- 如果您使用 **Windows DSVM**，您必須在機器上安裝 [Git 認證管理員 (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) \(英文\)。 在 README.md 檔案中，向下捲動至 [下載並安裝] 區段，然後按一下 [最新的安裝程式]。 這樣會帶您到最新的安裝程式分頁。 從這裡下載 .exe 安裝程式並執行它。 
- 如果您使用 **Linux DSVM**，請在您的 DSVM 上建立 SSH 公開金鑰，並將它新增到您的群組 Azure DevOps Services。 如需 SSH 的詳細資訊，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)中的**建立 SSH 公開金鑰**一節。 
    
## <a name="1-create-a-project-and-repositories"></a>1.建立專案和存放庫

如果您使用 Azure DevOps 作為版本設定和共同作業的程式碼裝載平台，請完成這個步驟。 在本節中，您會在群組的 Azure DevOps Services 中建立三個成品：

- Azure DevOps 中的 **MyTeam** 專案
- Git 上的 **MyProjectTemplate** 存放庫 (**R3**)
- Git 上的 **MyTeamUtilities** 存放庫 (**R4**)

### <a name="create-the-myteam-project"></a>建立 MyTeam 專案

- 移至您群組的 Azure DevOps Services 首頁，位於 URL：`https://<Azure DevOps Services Name\>.visualstudio.com`。 
- 按一下 [新增] 以建立專案。 

    ![2](./media/team-lead-tasks/team-leads-2-create-new-team.png)

- [建立專案] 視窗會要求您輸入專案名稱 (在此範例中為 **MyTeam**)。 請確定您選取 **Agile** 作為**程序範本**，以及選取 **Git** 作為**版本控制**。 

    ![3](./media/team-lead-tasks/team-leads-3-create-new-team-2.png)

- 按一下 [建立專案]。 您的專案 **MyTeam** 會在 1 分鐘內建立。 

- 專案 **MyTeam** 建立之後，請按一下 [瀏覽至專案] 按鈕，以導向專案的首頁。 

    ![4](./media/team-lead-tasks/team-leads-4-create-new-team-3.png)

- 如果您看到 [恭喜！] 快顯視窗，請按一下 [新增程式碼] (紅色方塊中的按鈕)。 否則，請按一下 [程式碼] (黃色方塊中)。 這樣會將您導向至專案的 Git 存放庫頁面。 

    ![5](./media/team-lead-tasks/team-leads-5-team-project-home.png)

### <a name="create-the-myprojecttemplate-repository-r3-on-git"></a>在 Git 上建立 MyProjectTemplate 存放庫 (R3)

- 在專案的 Git 存放庫頁面上，按一下存放庫名稱 **MyTeam** 旁的向下箭號，然後選取 [管理存放庫...]。

    ![6](./media/team-lead-tasks/team-leads-6-rename-team-project-repo.png)

- 在專案的控制台上，按一下 [版本控制] 索引標籤上的 [MyTeam]，然後選取 [重新命名存放庫...]。 

    ![7](./media/team-lead-tasks/team-leads-7-rename-team-project-repo-2.png)

- 在 [重新命名 MyTeam 存放庫] 視窗中，輸入存放庫的新名稱。 在此範例中為 *MyTeamProjectTemplate*。 您可以選擇類似 <您的小組名稱\>ProjectTemplate 這樣的名稱。 按一下 [重新命名] 繼續作業。

    ![8](./media/team-lead-tasks/team-leads-8-rename-team-project-repo-3.png)

### <a name="create-the-myteamutilities-repository-r4-on-git"></a>在 Git 上建立 MyTeamUtilities 存放庫 (R4)

- 若要在專案下建立新的存放庫 <您的小組名稱\>Utilities，請在專案控制台的 [版本控制] 索引標籤上，按一下 [新增存放庫...]。  

    ![9](./media/team-lead-tasks/team-leads-9-create-team-utilities.png)

- 在快顯的 [建立新存放庫] 視窗中，提供這個存放庫的名稱。 在此範例中，我們將其命名為 MyTeamUtilities，在我們的標記法中它是 **R4**。 選擇類似 <您的小組名稱\>Utilities 這樣的名稱。 請確定您針對 **類型**選取 **Git**。 接著，按一下 [建立] 繼續作業。

    ![10](./media/team-lead-tasks/team-leads-10-create-team-utilities-2.png)

- 確認您看到兩個新的 Git 存放庫建立於專案 **MyTeam** 下。 在此範例中： 

- **MyTeamProjectTemplate** (R3) 
- **MyTeamUtilities** (R4)。

    ![11](./media/team-lead-tasks/team-leads-11-two-repo-in-team.png)


## <a name="2-seed-your-projecttemplate-and-teamutilities-repositories"></a>2.植入您的 ProjectTemplate 和 TeamUtilities 存放庫

植入程序會使用本機 DSVM 上的目錄作為中繼暫存網站。 如果您需要自訂 **ProjectTemplate** 和 **TeamUtilities** 存放庫以符合一些特定小組需求，您要在下列程序的倒數第二個步驟執行這項操作。 以下是用來為資料科學小組植入 **MyTeamProjectTemplate** 和 **MyTeamUtilities** 存放庫內容的步驟摘要。 個別步驟對應至植入程序中的各小節：

- 將群組存放庫複製到本機目錄：小組 R1 - 複製到 -> 本機 D1
- 將小組存放庫複製到本機目錄：小組 R3 & R4 - 複製到 -> 本機 D3 & D4
- 將群組專案範本內容複製到本機小組資料夾：D1 - 內容複製到 -> D3
- (選擇性) 自訂本機 D3 & D4
- 將本機目錄內容推送至小組存放庫：D3 & D4 - 內容新增至 -> team R3 & R4


### <a name="initialize-the-team-repositories"></a>初始化小組存放庫

在此步驟中，您會從群組專案範本存放庫初始化專案範本存放庫：

- 來自 **GroupProjectTemplate** (**R1**) 存放庫的 **MyTeamProjectTemplate** 存放庫 (**R3**)


### <a name="clone-group-repositories-into-local-directories"></a>將群組存放庫複製到本機目錄

若要開始此程序：

- 在您的本機電腦上建立目錄：
    - 針對 **Windows**：**C:\GitRepos\GroupCommon** 和 **C:\GitRepos\MyTeam**
    - 針對 **Linux**：主目錄上的 **GitRepos\GroupCommon** 和 **GitRepos\MyTeam** 
- 將目錄變更為 **GitRepos\GroupCommon**。
- 視需要在本機電腦作業系統上執行下列命令。

**Windows**

    git clone https://<Your Azure DevOps Services name>.visualstudio.com/GroupCommon/_git/GroupProjectTemplate
    

![12](./media/team-lead-tasks/team-leads-12-create-two-group-repos.png)

**Linux**
    
    git clone ssh://<Your Azure DevOps Services name>@<Your Azure DevOps Services name>.visualstudio.com:22/GroupCommon/_git/GroupProjectTemplate
    
    
![13](./media/team-lead-tasks/team-leads-13-clone_two_group_repos_linux.png)

這些命令會將群組 Azure DevOps Services 上的 **GroupProjectTemplate** (R1) 存放庫，複製到本機電腦上 **GitRepos\GroupCommon** 中的本機目錄。 在複製之後，目錄 **GroupProjectTemplate** (D1) 會在目錄 **GitRepos\GroupCommon** 中建立。 在此，我們假設您的群組管理員已建立專案 **GroupCommon**，且 **GroupProjectTemplate** 存放庫位於此專案下。 


### <a name="clone-your-team-repositories-into-local-directories"></a>將小組存放庫複製到本機目錄

這些命令會將群組 Azure DevOps Services 上的專案 **MyTeam** 下的 **MyTeamProjectTemplate** (R3) 和 **MyTeamUtilities** (R4) 存放庫，複製到本機電腦上 **GitRepos\MyTeam** 中的 **MyTeamProjectTemplate** (D3) 和 **MyTeamUtilities** (D4) 目錄。 

- 將目錄變更為 **GitRepos\MyTeam**
- 視需要在本機電腦作業系統上執行下列命令。 

**Windows**

    git clone https://<Your Azure DevOps Services name>.visualstudio.com/<Your Team Name>/_git/MyTeamProjectTemplate
    git clone https://<Your Azure DevOps Services name>.visualstudio.com/<Your Team Name>/_git/MyTeamUtilities

![14](./media/team-lead-tasks/team-leads-14-clone_two_empty_team_repos.png)
        
**Linux**
    
    git clone ssh://<Your Azure DevOps Services name>@<Your Azure DevOps Services name>.visualstudio.com:22/<Your Team Name>/_git/MyTeamProjectTemplate
    git clone ssh://<Your Azure DevOps Services name>@<Your Azure DevOps Services name>.visualstudio.com:22/<Your Team Name>/_git/MyTeamUtilities
    
![15](./media/team-lead-tasks/team-leads-15-clone_two_empty_team_repos_linux.png)

在複製之後，兩個目錄 **MyTeamProjectTemplate** (D3) 和 **MyTeamUtilities** (D4) 會在 **GitRepos\MyTeam** 會在目錄中建立。 在此，我們假設您已將專案範本和公用程式存放庫命名為 **MyTeamProjectTemplate** 和 **MyTeamUtilities**。 

### <a name="copy-the-group-project-template-content-to-the-local-project-template-directory"></a>將群組專案範本內容複製到本機專案範本目錄

若要將本機 **GroupProjectTemplate** (D1) 資料夾的內容複製到本機 **MyTeamProjectTemplate** (D3)，請執行下列其中一個殼層指令碼： 

#### <a name="from-the-powershell-command-line-for-windows"></a>從適用於 Windows 的 PowerShell 命令列       

    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/tdsp_local_copy_win.ps1" -outfile "tdsp_local_copy_win.ps1"
    .\tdsp_local_copy_win.ps1 2

    
![16](./media/team-lead-tasks/team-leads-16-local_copy_team_lead_new.png)

#### <a name="from-the-linux-shell-for-the-linux-dsvm"></a>從適用於 **Linux DSVM** 的 Linux 殼層
    
    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/tdsp_local_copy_linux.sh"
    bash tdsp_local_copy_linux.sh 2
    
![17](./media/team-lead-tasks/team-leads-17-local-copy-team-lead-linux-new.png)

指令碼排除 .git 目錄的內容。 指令碼會提示您提供來源目錄 D1 和目的地目錄 D3 的**完整路徑**。
        

### <a name="customize-your-project-template-or-team-utilities-optional"></a>自訂您的專案範本或小組公用程式 (選擇性)

視需要在設定程序的這個階段，自訂 **MyTeamProjectTemplate** (D3) 和 **MyTeamUtilities** (D4)。 

- 如果您想要自訂 D3 的內容以符合小組的特定需求，您可以修改範本文件或變更目錄結構。

- 如果您的小組已開發一些您想要與整個小組共用的公用程式，請將這些公用程式複製並貼上到目錄 D4。 


### <a name="push-local-directory-content-to-team-repositories"></a>將本機目錄內容推送至小組存放庫

若要將 (選擇性自訂) 本機目錄 D3 和 D4 的內容新增至小組存放庫 R3 和 R4，請從 Windows PowerShell 主控台或 Linux 殼層執行下列 git bash 命令。 從 **GitRepos\MyTeam\MyTeamProjectTemplate** 目錄執行命令。

    git status
    git add .
    git commit -m"push from DSVM"
    git push
    
![18](./media/team-lead-tasks/team-leads-18-push-to-group-server-2.png)

在此指令碼執行時，群組 Azure DevOps Services 的 MyTeamProjectTemplate 存放庫會以幾近即時的方式同步其檔案。

![19](./media/team-lead-tasks/team-leads-19-push-to-group-server-showed-up.png)

現在從 **GitRepos\MyTeam\MyTeamUtilities** 目錄執行相同一組的四個 git 命令。 

> [AZURE.NOTE]如果這是第一次認可 Git 存放庫，您必須在執行 `git commit` 命令之前，設定全域參數 user.name 和 user.email。 執行下列兩個命令：
        
    git config --global user.name <your name>
    git config --global user.email <your email address>
 
> 如果您認可多個 Git 存放庫，請在認可各個存放庫時使用相同名稱和電子郵件地址。 使用相同名稱和電子郵件地址，會讓您日後建置 PowerBI 儀表板以追蹤多個存放庫上的 Git 活動時，更為便利。

![20](./media/team-lead-tasks/team-leads-20-git-config-name.png)


## <a name="3-create-team-data-and-analytics-resources-optional"></a>3.建立小組資料與分析資源 (選擇性)

與您的整個小組共用資料與分析資源，有效能和成本的優點：小組成員可以在共用的資源上執行其專案、節省預算，以及更有效率地共同作業。 在本節中，我們提供了有關如何建立 Azure 檔案儲存體的指示。 在下一節中，我們會提供如何將 Azure 檔案儲存體掛接至本機電腦的指示。 如需有關共用其他資源，例如 Azure 資料科學虛擬機器、Azure HDInsight Spark 叢集的詳細資訊，請參閱[平台和工具](platforms-and-tools.md)。 本主題提供從資料科學觀點選取適合您需求的資源的指引，產品分頁的連結，以及我們已發佈之其他相關且有用的教學課程。

>[AZURE.NOTE] 若要避免資料在資料中心之間傳輸，因為這麼做既緩慢又耗費成本，請確定資源群組、儲存體帳戶和 Azure VM (例如，DSVM) 是在相同的 Azure 資料中心。 

執行下列指令碼，為您的小組建立 Azure 檔案儲存體。 小組的 Azure 檔案儲存體可以用來儲存對整個小組而言很實用的資料資產。 指令碼會提示您 Azure 帳戶和訂用帳戶資訊，讓這些認證準備就緒以便輸入。 

### <a name="create-azure-file-storage-with-powershell-from-windows"></a>從 Windows 使用 PowerShell 建立 Azure 檔案儲存體

從 PowerShell 命令列執行此指令碼：

    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/CreateFileShare.ps1" -outfile "CreateFileShare.ps1"
    .\CreateFileShare.ps1

![21](./media/team-lead-tasks/team-leads-21-create-fileshare-win.png)   

當系統提示時，登入您的 Microsoft Azure 帳戶：

![22](./media/team-lead-tasks/team-leads-22-file-create-s1.png)

選取您要使用的 Azure 訂用帳戶：

![23](./media/team-lead-tasks/team-leads-23-file-create-s2.png)

選取要使用儲存體帳戶，或是在選取的訂用帳戶底下建立一個新帳戶：

![24](./media/team-lead-tasks/team-leads-24-file-create-s3.png)

輸入要建立之 Azure 檔案儲存體的名稱。 只接受小寫字元、數字和 -：

![25](./media/team-lead-tasks/team-leads-25-file-create-s4.png)

為了協助在建立這個儲存體之後掛接及共用，將 Azure 檔案儲存體資訊儲存到文字檔並記下其位置路徑。 您在下一個章節特別需要此檔案，以將您的 Azure 檔案儲存體掛接至 Azure 虛擬機器。 

將此文字檔簽入 ProjectTemplate 存放庫中，是很好的作法。 我們建議您放入目錄 **Docs\DataDictionaries**。 因此，這個資料資產可以被小組中的所有專案存取。 

![26](./media/team-lead-tasks/team-leads-26-file-create-s5.png)


### <a name="create-azure-file-storage-with-a-linux-script"></a>使用 Linux 指令碼建立 Azure 檔案儲存體

從 Linux 殼層中執行此指令碼：

    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/CreateFileShare.sh"
    bash CreateFileShare.sh

遵循此畫面上的指示，登入 Microsoft Azure 帳戶：

![27](./media/team-lead-tasks/team-leads-27-file-create-linux-s1.png)

選取您要使用的 Azure 訂用帳戶：

![28](./media/team-lead-tasks/team-leads-28-file-create-linux-s2.png)

選取要使用儲存體帳戶，或是在選取的訂用帳戶底下建立一個新帳戶：

![29](./media/team-lead-tasks/team-leads-29-file-create-linux-s3.png)

輸入要建立之 Azure 檔案儲存體的名稱，只接受小寫字元、數字和 -：

![30](./media/team-lead-tasks/team-leads-30-file-create-linux-s4.png)

為了協助在建立這個儲存體之後進行存取，將 Azure 檔案儲存體資訊儲存到文字檔並記下其位置路徑。 您在下一個章節特別需要此檔案，以將您的 Azure 檔案儲存體掛接至 Azure 虛擬機器。

將此文字檔簽入 ProjectTemplate 存放庫中，是很好的作法。 我們建議您放入目錄 **Docs\DataDictionaries**。 因此，這個資料資產可以被小組中的所有專案存取。 

![31](./media/team-lead-tasks/team-leads-31-file-create-linux-s5.png)


## <a name="4-mount-azure-file-storage-optional"></a>4.掛接 Azure 檔案儲存體 (選擇性)

在成功建立 Azure 檔案儲存體之後，可以使用下列其中一個 PowerShell 或 Linux 指令碼，將它掛接至您的本機電腦。

### <a name="mount-azure-file-storage-with-powershell-from-windows"></a>從 Windows 使用 PowerShell 掛接 Azure 檔案儲存體

    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/AttachFileShare.ps1" -outfile "AttachFileShare.ps1"
    .\AttachFileShare.ps1
    
如果您尚未登入，系統會要求您先登入。 

當系統詢問您是否有 Azure 檔案儲存體資訊檔案時，按一下 **Enter** 鍵或 **y** 鍵以繼續，然後輸入您在上一個步驟中建立之檔案的 ***完整路徑和名稱**。 掛接 Azure 檔案儲存體的資訊會直接從該檔案讀取，您已經準備就緒可以進行下一個步驟。

![32](./media/team-lead-tasks/team-leads-32-attach-s1.png)

> [AZURE.NOTE] 如果您沒有包含 Azure 檔案儲存體資訊的檔案，從鍵盤輸入資訊的步驟會在此章節結尾提供。

然後，系統會要求您輸入要新增至虛擬機器之磁碟機的名稱。 現有磁碟機名稱的清單會列印在畫面上。 您應該提供清單上沒有的磁碟機名稱。

![33](./media/team-lead-tasks/team-leads-33-attach-s2.png)

確認新的 F 磁碟機已成功掛接到您的電腦。

![34](./media/team-lead-tasks/team-leads-34-attach-s3.png)

**如何以手動方式輸入 Azure 檔案儲存體資訊：** 如果您的 Azure 檔案儲存體資訊不在文字檔中，您可以遵循以下畫面的指示，輸入必要的訂用帳戶、儲存體帳戶，及 Azure 檔案儲存體資訊：

![35](./media/team-lead-tasks/team-leads-35-attach-s4.png)

輸入您的 Azure 訂用帳戶名稱、選取建立 Azure 檔案儲存體所在的儲存體帳戶，以及輸入 Azure 檔案儲存體名稱：

![36](./media/team-lead-tasks/team-leads-36-attach-s5.png)

### <a name="mount-azure-file-storage-with-a-linux-script"></a>使用 Linux 指令碼掛接 Azure 檔案儲存體

    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/AttachFileShare.sh"
    bash AttachFileShare.sh

![37](./media/team-lead-tasks/team-leads-37-attach-s1-linux.png)

如果您尚未登入，系統會要求您先登入。 

當系統詢問您是否有 Azure 檔案儲存體資訊檔案時，按一下 **Enter** 鍵或 **y** 鍵以繼續，然後輸入您在上一個步驟中建立之檔案的 ***完整路徑和名稱**。 掛接 Azure 檔案儲存體的資訊會直接從該檔案讀取，您已經準備就緒可以進行下一個步驟。

![38](./media/team-lead-tasks/team-leads-38-attach-s2-linux.png)

然後，系統會要求您輸入要新增至虛擬機器之磁碟機的名稱。 現有磁碟機名稱的清單會列印在畫面上。 您應該提供清單上沒有的磁碟機名稱。

![39](./media/team-lead-tasks/team-leads-39-attach-s3-linux.png)

確認新的 F 磁碟機已成功掛接到您的電腦。

![40](./media/team-lead-tasks/team-leads-40-attach-s4-linux.png)

**如何以手動方式輸入 Azure 檔案儲存體資訊：** 如果您的 Azure 檔案儲存體資訊不在文字檔中，您可以遵循以下畫面的指示，輸入必要的訂用帳戶、儲存體帳戶，及 Azure 檔案儲存體資訊：

- 輸入 **n**。
- 選取訂用帳戶名稱的索引，該訂用帳戶是在上一個步驟中建立 Azure 檔案儲存體的位置：

    ![41](./media/team-lead-tasks/team-leads-41-attach-s5-linux.png)

- 選取訂用帳戶底下的儲存體帳戶，並且輸入 Azure 檔案儲存體名稱：

    ![42](./media/team-lead-tasks/team-leads-42-attach-s6-linux.png)

- 輸入要新增至電腦的磁碟機名稱，該名稱應該與現有的任何磁碟機名稱有所區隔：

    ![43](./media/team-lead-tasks/team-leads-43-attach-s7-linux.png)


## <a name="5-set-up-security-control-policy"></a>5.設定安全性控制原則 

群組 Azure DevOps Services 首頁的右上角，按一下您的使用者名稱旁邊的**齒輪圖示**，然後選取 [安全性] 索引標籤。您可以在這裡將成員新增至您的小組，並且授與各種權限。

![44](./media/team-lead-tasks/team-leads-44-add-team-members.png)

## <a name="next-steps"></a>後續步驟

以下是 Team 資料科學程序定義之角色和工作更詳細描述的連結：

- [資料科學小組的群組管理員工作](group-manager-tasks.md)
- [資料科學小組的小組負責人工作](team-lead-tasks.md)
- [資料科學小組的專案負責人工作](project-lead-tasks.md)
- [資料科學小組的專案個別參與者](project-ic-tasks.md)