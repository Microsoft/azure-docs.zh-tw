---
title: 個別參與者的 Team 資料科學程序工作 - Azure | Microsoft Docs
description: 資料科學 Team 專案上個別參與者工作的概述。
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
ms.openlocfilehash: f21098381d75a4843e9300beaae687adc6ec107d
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303756"
---
# <a name="individual-contributor-tasks"></a>個別參與者工作

本主題概述希望個別參與者為其資料科學小組完成的工作。 目標是建立在 [Team 資料科學程序](overview.md) (TDSP) 上標準化的共同作業小組環境。 如需人員角色的概述，以及對此程序進行標準化之資料科學小組所處理相關聯工作的大綱，請參閱 [Team 資料科學程序角色和工作](roles-tasks.md)。

個別參與者 (資料科學家) 為所述之專案設定 TDSP 環境的專案工作，如下所示： 

![1](./media/project-ic-tasks/project-ic-1-tdsp-data-scientist.png)

- **GroupUtilities** 是您的群組會維護以在整個群組間共用有用公用程式的存放庫。 
- **TeamUtilities** 是您的小組特別為小組維護的存放庫。 

如需如何在 TDSP 底下執行資料科學專案的指示，請參閱[執行資料科學專案](project-execution.md)。 

>[AZURE.NOTE] 我們概述在下列指示中使用 Azure DevOps 設定 TDSP 小組環境所需的步驟。 我們會指定如何使用 Azure DevOps 完成這些工作，因為這是我們在 Microsoft 中實作 TDSP 的方式。 如果針對您的群組使用另一個程式碼裝載平台，必須由小組負責人完成的工作通常不會變更。 但是完成這些工作的方式將會不同。


## <a name="repositories-and-directories"></a>存放庫和目錄

本教學課程會使用存放庫和目錄的縮寫名稱。 這些名稱讓您更容易遵循存放庫和目錄之間的作業。 這個標記法 (**R** 適用於 Git 存放庫、**D** 適用於您 DSVM 上的本機目錄) 在下列各節中使用：

- **R2**：Git 上的 GroupUtilities 存放庫，由您的群組管理員設定於 Azure DevOps 群組伺服器上。
- **R4**：Git 上的 TeamUtilities 存放庫，您的小組負責人已設定。
- **R5**：Git 上的 Project 存放庫，您的專案負責人已設定。
- **D2**：從 R2 複製的本機目錄。
- **D4**：從 R4 複製的本機目錄。
- **D5**：從 R5 複製的本機目錄。


## <a name="step-0-prerequisites"></a>步驟-0：必要條件

完成[資料科學小組的群組管理員工作](group-manager-tasks.md)中所述，指派給您的群組管理員的工作，即可滿足必要條件。 簡單來說，就是在您開始小組負責人工作之前，必須符合下列需求： 
- 您的群組管理員已設定 **GroupUtilities** 存放庫 (如果有的話)。 
- 您的小組負責人已設定 **TeamUtilities** 存放庫 (如果有的話)。
- 專案負責人已設定專案存放庫。 
- 您的專案負責人已將您新增至專案存放庫，具備從專案存放庫複製以及推送回專案存放庫的權限。

第二個存放庫，**TeamUtilities** 存放庫，必要條件是選擇性的，取決於您的小組是否有小組專用的公用程式存放庫。 如果未完成其他三個必要條件中的任何一個，請連絡小組負責人、專案負責人或其代理人，藉由遵循[資料科學小組的小組負責人工作](team-lead-tasks.md)或[資料科學小組的專案負責人工作](project-lead-tasks.md)的指示，來進行設定。

- Git 必須安裝在您的機器上。 如果您使用資料科學虛擬機器 (DSVM)，則已預先安裝 Git，而您可以繼續作業。 否則，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)。  
- 如果您使用 **Windows DSVM**，您必須在機器上安裝 [Git 認證管理員 (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) \(英文\)。 在 README.md 檔案中，向下捲動至 [下載並安裝] 區段，然後按一下 [最新的安裝程式]。 這樣會帶您到最新的安裝程式分頁。 從這裡下載 .exe 安裝程式並執行它。 
- 如果您使用 **Linux DSVM**，請在您的 DSVM 上建立 SSH 公開金鑰，並將它新增到您的群組 Azure DevOps Services。 如需 SSH 的詳細資訊，請參閱[平台和工具附錄](platforms-and-tools.md#appendix)中的**建立 SSH 公開金鑰**一節。 
- 如果您的小組和/或專案負責人已建立一些您需要掛接至 DSVM 的 Azure 檔案儲存體，您應該從其中取得 Azure 檔案儲存體資訊。 

## <a name="step-1-3-clone-group-team-and-project-repositories-to-local-machine"></a>步驟 1-3：將群組、小組和專案存放庫複製到本機電腦

本章節提供完成專案個別參與者前三個工作的指示： 

- 將 **GroupUtilities** 存放庫 R2 複製到 D2
- 將 **TeamUtilities** 存放庫 R4 複製到 D4 
- 將 **Project** 存放庫 R5 複製到 D5。

在您的本機電腦上，建立目錄 ***C:\GitRepos*** (適用於 Windows) 或 ***$home/GitRepos*** (適用於 Linux)，然後變更為該目錄。 

執行下列其中一個命令 (視您的作業系統而定)，將 **GroupUtilities**、**TeamUtilities** 和 **Project** 存放庫複製到本機電腦上的目錄： 

**Windows**
    
    git clone <the URL of the GroupUtilities repository>
    git clone <the URL of the TeamUtilities repository>
    git clone <the URL of the Project repository>
    
![2](./media/project-ic-tasks/project-ic-2-clone-three-repo-to-ic.png)

確認您在專案目錄底下看到三個資料夾。

![3](./media/project-ic-tasks/project-ic-3-three-repo-cloned-to-ic.png)

**Linux**
    
    git clone <the SSH URL of the GroupUtilities repository>
    git clone <the SSH URL of the TeamUtilities repository>
    git clone <the SSH URL of the Project repository>

![4](./media/project-ic-tasks/project-ic-4-clone-three-repo-to_ic-linux.png)

確認您在專案目錄底下看到三個資料夾。

![5](./media/project-ic-tasks/project-ic-5-three-repo-cloned-to-ic-linux.png)

## <a name="step-4-5-mount-azure-file-storage-to-your-dsvm-optional"></a>步驟 4-5：(選擇性) 將 Azure 檔案儲存體掛接至 DSVM

若要將 Azure 檔案儲存體掛接至 DSVM，請參閱[資料科學小組的小組負責人工作](team-lead-tasks.md)中的第 4 節

## <a name="next-steps"></a>後續步驟

以下是 Team 資料科學程序定義之角色和工作更詳細描述的連結：

- [資料科學小組的群組管理員工作](group-manager-tasks.md)
- [資料科學小組的小組負責人工作](team-lead-tasks.md)
- [資料科學小組的專案負責人工作](project-lead-tasks.md)
- [資料科學小組的專案個別參與者](project-ic-tasks.md)

