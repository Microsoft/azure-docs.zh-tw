---
title: Team 資料科學程序群組管理員工作 - Azure | Microsoft Docs
description: 資料科學 Team 專案上群組管理員工作的概述。
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
ms.openlocfilehash: a89c0f916f67de1bae81a5e1b3dcfc2cae41d248
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304181"
---
# <a name="group-manager-tasks"></a>群組管理員工作

本主題概述希望群組管理員為其資料科學組織完成的工作。 目標是建立在 [Team 資料科學程序](overview.md) (TDSP) 上標準化的共同作業群組環境。 如需人員角色的概述，以及對此程序進行標準化之資料科學小組所處理相關聯工作的大綱，請參閱 [Team 資料科學程序角色和工作](roles-tasks.md)。

**群組管理員**是在企業中整個資料科學單位的管理員。 一個資料科學單位可能有多個小組，而每個小組負責不同商業垂直市場中的多個資料科學專案。 群組管理員可能會將他們的工作委派給代理人，但與角色相關聯的工作則一樣。 有六個主要工作，如下圖所示：

![0](./media/group-manager-tasks/tdsp-group-manager.png)


>[AZURE.NOTE] 我們概述在下列指示中使用 Azure DevOps Services 來設定 TDSP 群組環境所需的步驟。 我們會指定如何使用 Azure DevOps Services 完成這些工作，因為這是我們在 Microsoft 中實作 TDSP 的方式。 如果針對您的群組使用另一個程式碼裝載平台，必須由群組管理員完成的工作通常不會變更。 但是完成這些工作的方式將會不同。

1. 設定群組的 **Azure DevOps Services**。
2. 在 Azure DevOps Services 上建立**群組專案** (針對 Azure DevOps Services 使用者)
3. 建立 **GroupProjectTemplate** 存放庫
4. 建立 **GroupUtilities** 存放庫
5. 針對含有 TDSP 存放庫內容的 Azure DevOps Services，植入 **GroupProjectTemplate** 和 **GroupUtilities** 存放庫。
6. 為小組成員設定**安全性控制**，以存取 GroupProjectTemplate 和 GroupUtilities 存放庫。

以下將詳細說明上述各步驟。 但首先，我們要讓您熟悉各個縮寫，並討論使用存放庫的必要條件。

### <a name="abbreviations-for-repositories-and-directories"></a>存放庫和目錄的縮寫

本教學課程會針對存放庫和目錄使用縮寫名稱。 這些定義讓您更容易遵循存放庫和目錄之間的作業。 在下列各節中，會使用這個標記法：

- **G1**：由 Microsoft 的 TDSP 小組所開發與管理的專案範本存放庫。
- **G2**：由 Microsoft 的 TDSP 小組所開發與管理的公用程式存放庫。
- **R1**：Git 上的 GroupProjectTemplate 存放庫，由您設定於 Azure DevOps Services 群組伺服器上。
- **R2**：Git 上的 GroupUtilities 存放庫，由您設定於 Azure DevOps Services 群組伺服器上。
- **LG1** 和 **LG2**：您機器上的本機目錄，您會分別將 G1 和 G2 複製到其中。
- **LR1** 和 **LR2**：您機器上的本機目錄，您會分別將 R1 和 R2 複製到其中。

### <a name="pre-requisites-for-cloning-repositories-and-checking-code-in-and-out"></a>複製存放庫以及簽入和簽出程式碼的必要條件
 
- Git 必須安裝在您的機器上。 如果您使用資料科學虛擬機器 (DSVM)，則已預先安裝 Git，而您可以繼續作業。 否則，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)。  
- 如果您使用 **Windows DSVM**，您必須在機器上安裝 [Git 認證管理員 (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) \(英文\)。 在 README.md 檔案中，向下捲動至 [下載並安裝] 區段，然後按一下 [最新的安裝程式]。 這個步驟會帶您到最新的安裝程式頁面。 從這裡下載 .exe 安裝程式並執行它。 
- 如果您使用 **Linux DSVM**，請在您的 DSVM 上建立 SSH 公開金鑰，並將它新增到您的群組 Azure DevOps Services。 如需 SSH 的詳細資訊，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)中的**建立 SSH 公開金鑰**一節。 


## <a name="1-create-account-on-azure-devops-services"></a>1.建立 Azure DevOps Services 的帳戶

Azure DevOps Services 會裝載下列存放庫：

- **群組通用的存放庫**：一般用途的存放庫，可由群組內的多個小組針對多個資料科學專案加以採用。 例如，*GroupProjectTemplate* 和 *GroupUtilities* 存放庫。
- **小組存放庫**：群組內特定小組的存放庫。 這些存放庫是針對小組的需求而定，可能會由該小組執行的多個專案所採用，但通常不足以供資料科學群組內的多個小組使用。 
- **專案存放庫**：適用於特定專案的存放庫。 這類存放庫通常不足以供由小組執行的多個專案，以及資料科學群組中的多個小組使用。


### <a name="setting-up-the-azure-devops-services-sign-into-your-microsoft-account"></a>設定 Azure DevOps Services 登入您的 Microsoft 帳戶
    
移至 [Visual Studio Online](https://www.visualstudio.com/)、按一下右上角的 [登入]，然後登入您的 Microsoft 帳戶。 
    
![1](./media/group-manager-tasks/login.PNG)

如果您沒有 Microsoft 帳戶，按一下 [立即註冊] 來建立 Microsoft 帳戶，然後使用此帳戶登入。 

如果您的組織具有 Visual Studio/MSDN 訂用帳戶，按一下綠色的 [使用您的工作或學校帳戶登入] 方塊，然後使用與此訂用帳戶相關聯的認證登入。 
        
![2](./media/group-manager-tasks/signin.PNG)


        
登入之後，按一下右上角的 [建立新帳戶]，如下圖所示：
        
![3](./media/group-manager-tasks/create-account-1.PNG)
        
使用下列值，為您想要在 [建立您的帳戶] 精靈中建立的 Azure DevOps Services 填入資訊： 

- **伺服器 URL**：使用您自己的「伺服器名稱」取代 mysamplegroup。 伺服器的 URL 將會是： https://\<servername\>.visualstudio.com。 
- **使用以下項目管理程式碼：** 選取 [Git]。
- **專案名稱：** 輸入 *GroupCommon*。 
- **工作的組織方式：** 選擇 [Agile]。
- **您專案的裝載位置：** 選擇地理位置。 在此範例中，我們選擇 [美國中南部]。 
        
![4](./media/group-manager-tasks/fill-in-account-information.png)

>[AZURE.NOTE] 如果您在按一下 [建立新帳戶] 之後看到下列快顯視窗，則需按一下 [變更詳細資料] 來顯示分項的所有欄位。

![5](./media/group-manager-tasks/create-account-2.png)


按一下 [繼續]。 

## <a name="2-groupcommon-project"></a>2.GroupCommon 專案

[GroupCommon] 頁面 (https://\<servername\>.visualstudio.com/GroupCommon) 會在您的 Azure DevOps Services 建立之後開啟。
                            
![6](./media/group-manager-tasks/server-created-2.PNG)

## <a name="3-create-the-grouputilities-r2-repository"></a>3.建立 GroupUtilities (R2) 存放庫

若要在 Azure DevOps Services 下建立 **GroupUtilities** (R2) 存放庫：

- 若要開啟 [建立新的存放庫] 精靈，請在專案的 [版本控制] 索引標籤上按一下 [新增存放庫]。 

![7](./media/group-manager-tasks/create-grouputilities-repo-1.png) 

- 選取 [Git] 作為**類型**，並輸入 *GroupUtilities* 作為**名稱**，然後按一下 [建立]。 

![8](./media/group-manager-tasks/create-grouputilities-repo-2.png)
                
現在您應該會在 [版本控制] 頁面的左側資料行中看見兩個 Git 存放庫 **GroupProjectTemplate** 和 **GroupUtilities**： 

![9](./media/group-manager-tasks/two-repo-under-groupCommon.PNG)


## <a name="4-create-the-groupprojecttemplate-r1-repository"></a>4.建立 GroupProjectTemplate (R1) 存放庫

Azure DevOps 群組伺服器適用的儲存庫設定包含兩個工作：

- 將預設的 **GroupCommon** 存放庫重新命名為 ***GroupProjectTemplate***。
- 在專案 **GroupCommon** 下方的 Azure DevOps Services 上建立 **GroupUtilities** 存放庫。 

本節會在有關重新命名慣例或我們的存放庫和目錄的備註之後包含第一個工作的指示。 第二個工作的指示則包含於下一節的步驟 4 中。

### <a name="rename-the-default-groupcommon-repository"></a>重新命名預設的 GroupCommon 存放庫

將預設的 **GroupCommon** 存放庫重新命名為 GroupProjectTemplate (在本教學課程中稱為 **R1**)：
    
- 按一下 [GroupCommon] 專案頁面上的 [在程式碼上共同作業]。 這會帶您前往專案 **GroupCommon** 的預設 Git 存放庫頁面。 此 Git 存放庫目前是空的。 

![10](./media/group-manager-tasks/rename-groupcommon-repo-3.png)
        
- 在 [GroupCommon] 之 Git 存放庫頁面上，按一下左上角的 [GroupCommon] (下圖中使用紅色方塊反白顯示的選項)，然後選取 [管理存放庫] (下圖中使用綠色方塊反白顯示的選項)。 此程序會開啟 [控制台]。 
- 選取專案的 [版本控制] 索引標籤。 

![11](./media/group-manager-tasks/rename-groupcommon-repo-4.png)

- 在左面板中，按一下 [GroupCommon] 存放庫右邊的 [...]，然後選取 [重新命名存放庫]。 

![12](./media/group-manager-tasks/rename-groupcommon-repo-5.png)
        
- 在快顯的 [重新命名 GroupCommon 存放庫] 精靈中，於 [存放庫名稱] 方塊中輸入GroupProjectTemplate，然後按一下 [重新命名]。 

![13](./media/group-manager-tasks/rename-groupcommon-repo-6.png)



## <a name="5-seed-the-r1--r2-repositories-on-the-azure-devops-services"></a>5.在 Azure DevOps Services 上植入 R1 和 R2 存放庫

在程序的這個階段中，您會植入您在上一節中設定的 *GroupProjectTemplate* (R1) 和 *GroupUtilities* (R2) 存放庫。 這些存放庫是使用由 Microsoft 針對 Team 資料科學程序所管理的 ***ProjectTemplate*** (**G1**) 和 ***Utilities*** (**G2**) 存放庫來植入的。 完成此植入時：

- R1 存放庫所擁有的目錄與文件範本組合，將與 G1 擁有的一樣
- 您的 R2 存放庫將包含 Microsoft 所開發的資料科學公用程式組合。

植入程序會使用本機 DSVM 上的目錄作為中繼暫存網站。 以下是本節中遵循的步驟：

- G1 & G2 - 複製到 -> LG1 & LG2
- R1 & R2 - 複製到 -> LR1 & LR2
- LG1 & LG2 - 檔案複製到 -> LR1 & LR2
- (選擇性) 自訂 LR1 & LR2
- LR1 & LR2 - 內容新增到 -> R1 & R2


### <a name="clone-g1--g2-repositories-to-your-local-dsvm"></a>將 G1 & G2 存放庫複製到您的本機 DSVM

在此步驟中，您會將 Team 資料科學程序 (TDSP) ProjectTemplate 存放庫 (G1) 和 Utilities (G2) 從 TDSP github 存放庫複製到您本機 DSVM 中的資料夾作為 LG1 和 LG2：

- 建立目錄作為根目錄，來裝載您對存放庫所做的所有複製。 
    -  在 Windows DSVM 中，建立 C:\GitRepos\TDSPCommon 目錄。 
    -  在 Linux DSVM 中，於主目錄中建立 *GitRepos\TDSPCommon* 目錄。 

- 從 *GitRepos\TDSPCommon* 目錄執行下列命令組合。

    `git clone https://github.com/Azure/Azure-TDSP-ProjectTemplate`<br>
    `git clone https://github.com/Azure/Azure-TDSP-Utilities`
        
![14](./media/group-manager-tasks/two-folder-cloned-from-TDSP-windows.PNG)

- 使用我們的縮寫存放庫名稱，以下是這些指令碼已達成的目標： 
    - G1 - 複製到 -> LG1
    - G2 - 複製到 -> LG2
- 完成複製之後，您應該可以在 **GitRepos\TDSPCommon** 目錄下看到兩個目錄：ProjectTemplate 和 Utilities。 

### <a name="clone-r1--r2-repositories-to-your-local-dsvm"></a>將 R1 & R2 存放庫複製到您的本機 DSVM

在此步驟中，您會在 DSVM 上的 **GitRepos\GroupCommon** 下方，於本機目錄上複製 GroupProjectTemplate 存放庫 (R1) 和 GroupUtilities 存放庫 (R2) (分別稱為 LR1 和 LR2)。

- 若要取得 R1 和 R2 存放庫的 URL，請移至 Azure DevOps Services 的 **GroupCommon** 首頁。 這通常會有 URL https://\<您的 Azure DevOps Services 名稱\>.visualstudio.com/GroupCommon。 
- 按一下 [程式碼]。 
- 選擇 [GroupProjectTemplate] 和 [GroupUtilities] 存放庫。 從 [複製 URL] 元素中複製並儲存每個 URL (如果是 Windows 為 HTTPS；如果是 Linux 則為 SSH)，接著在下列指令碼中使用：  

![15](./media/group-manager-tasks/find_https_ssh_2.PNG)

- 變更為您的 Windows 或 Linux DSVM 上的 **GitRepos\GroupCommon** 目錄，然後執行下列命令組合來將 R1 和 R2 複製到該目錄。
        
以下是 Windows 和 Linux 的指令碼：

    # Windows DSVM

    git clone <the HTTPS URL of the GroupProjectTemplate repository>
    git clone <the HTTPS URL of the GroupUtilities repository>

![16](./media/group-manager-tasks/clone-two-empty-group-reo-windows-2.PNG)

    # Linux DSVM

    git clone <the SSH URL of the GroupProjectTemplate repository>
    git clone <the SSH URL of the GroupUtilities repository>

![17](./media/group-manager-tasks/clone-two-empty-group-reo-linux-2.PNG)

>[AZURE.NOTE] 預期會收到 LR1 和 LR2 為空的警告訊息。    

- 使用我們的縮寫存放庫名稱，以下是這些指令碼已達成的目標： 
    - R1 - 複製到 -> LR1
    - R2 - 複製到 -> LR2   


### <a name="seed-your-groupprojecttemplate-lr1-and-grouputilities-lr2"></a>植入您的 GroupProjectTemplate (LR1) 和 GroupUtilities (LR2)

接下來，在您的本機電腦上，將 GitRepos\TDSPCommon 下方 ProjectTemplate 和 Utilities 目錄的內容 (除了 .git 目錄中的中繼資料) 複製到您在 **GitRepos\GroupCommon** 下的 GroupProjectTemplate 和 GroupUtilities 目錄。 以下是要在此步驟中完成的兩個工作：

- 將 GitRepos\TDSPCommon\ProjectTemplate (**LG1**) 中的檔案複製到 GitRepos\GroupCommon\GroupProjectTemplate (**LR1**) 
- 將 GitRepos\TDSPCommon\Utilities (**LG2**) 中的檔案複製到 GitRepos\GroupCommon\Utilities (**LR2**)。 

若要完成這兩個工作，請在 PowerShell 主控台 (Windows) 或殼層指令碼主控台 (Linux) 中執行下列指令碼。 系統會提示您輸入 LG1、LR1、LG2 和 LR2 的完整路徑。 系統會驗證您輸入的路徑。 如果您輸入的目錄不存在，系統會要求您再次輸入。 

    # Windows DSVM      
    
    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/tdsp_local_copy_win.ps1" -outfile "tdsp_local_copy_win.ps1"
    .\tdsp_local_copy_win.ps1 1

![18](./media/group-manager-tasks/copy-two-folder-to-group-folder-windows-2.PNG)

現在您可以看到已將目錄 LG1 和 LG1 中的檔案 (除了.git 目錄中的檔案) 分別複製到 LR1 和 LR2。

![19](./media/group-manager-tasks/copy-two-folder-to-group-folder-windows.PNG)

    # Linux DSVM

    wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/tdsp_local_copy_linux.sh"
    bash tdsp_local_copy_linux.sh 1

![20](./media/group-manager-tasks/copy-two-folder-to-group-folder-linux-2.PNG)
        
現在您會看到已將這兩個資料夾中的檔案 (除了.git 目錄中的檔案) 分別複製到 GroupProjectTemplate 和 GroupUtilities。
    
![21](./media/group-manager-tasks/copy-two-folder-to-group-folder-linux.PNG)

- 使用我們的縮寫存放庫名稱，以下是這些指令碼已達成的目標：
    - LG1 - 檔案複製到 -> LR1
    - LG2 - 檔案複製到 -> LR2

### <a name="option-to-customize-the-contents-of-lr1--lr2"></a>自訂 LR1 & LR2 內容的選項
    
如果您想要自訂 LR1 和 LR2 的內容以符合您群組的特定需求，則適合在程序的這個階段進行。 您可以修改範本文件、變更目錄結構，並新增您群組已開發或有助於整個群組的現有公用程式。 

### <a name="add-the-contents-in-lr1--lr2-to-r1--r2-on-group-server"></a>將 LR1 & LR2 中的內容新增到群組伺服器上的 R1 & R2

現在，您需要將 LR1 和 LR2 中的內容新增到存放庫 R1 和 R2。 以下是您可在 Windows PowerShell 或 Linux 中執行的 git bash 指令。 

從 GitRepos\GroupCommon\GroupProjectTemplate 目錄執行下列命令：

    git status
    git add .
    git commit -m"push from DSVM"
    git push

-m 選項可讓您設定適用於 git 認可的訊息。

![22](./media/group-manager-tasks/push-to-group-server-2.PNG)

您可以在群組的 Azure DevOps Services 中看到 GroupProjectTemplate 存放庫中的檔案立即同步。

![23](./media/group-manager-tasks/push-to-group-server-showed-up-2.PNG)

最後，變更至 **GitRepos\GroupCommon\GroupUtilities** 目錄，然後執行同一組 git bash命令：

    git status
    git add .
    git commit -m"push from DSVM"
    git push

>[AZURE.NOTE] 如果這是第一次認可 Git 存放庫，您必須在執行 `git commit` 命令之前，設定全域參數 user.name 和 user.email。 執行下列兩個命令：
        
    git config --global user.name <your name>
    git config --global user.email <your email address>
 
>如果您認可多個 Git 存放庫，請在認可各個存放庫時使用相同名稱和電子郵件地址。 使用相同名稱和電子郵件地址，會讓您日後建置 PowerBI 儀表板以追蹤多個存放庫上的 Git 活動時，更為便利。


- 使用我們的縮寫存放庫名稱，以下是這些指令碼已達成的目標：
    - LR1 - 內容新增到 -> R1
    - LR2 - 內容新增到 -> R2

## <a name="6-add-group-members-to-the-group-server"></a>6.將群組成員新增到群組伺服器

群組 Azure DevOps Services 首頁的右上角，按一下您的使用者名稱旁邊的**齒輪圖示**，然後選取 [安全性] 索引標籤。您可以在這裡將成員新增至您的群組，並授與各種權限。

![24](./media/group-manager-tasks/add_member_to_group.PNG) 


## <a name="next-steps"></a>後續步驟

以下是 Team 資料科學程序定義之角色和工作更詳細描述的連結：

- [資料科學小組的群組管理員工作](group-manager-tasks.md)
- [資料科學小組的小組負責人工作](team-lead-tasks.md)
- [資料科學小組的專案負責人工作](project-lead-tasks.md)
- [資料科學小組的專案個別參與者](project-ic-tasks.md)