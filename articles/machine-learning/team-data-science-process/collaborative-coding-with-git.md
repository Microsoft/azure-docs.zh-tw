---
title: 使用 Git 共同作業撰寫程式碼 - Azure Machine Learning | Microsoft Docs
description: 如何使用 Git 搭配敏捷式計劃進行資料科學專案的共同作業程式碼開發。
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
ms.date: 11/28/2017
ms.author: deguhath
ms.openlocfilehash: 78bbdb244d9bd52a06623f7a6fa3bca123ef3828
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44300064"
---
# <a name="collaborative-coding-with-git"></a>使用 Git 共同作業撰寫程式碼

本文中我們會說明如何使用 Git 作為共用的程式碼開發架構，進行資料科學專案的共同作業程式碼開發。 其中涵蓋如何將這些撰寫程式碼活動連結至[敏捷式開發](agile-development.md)中規劃的工作，以及如何執行程式碼檢閱。


## 1.<a name='Linkaworkitemwithagitbranch-1'></a>連結工作項目與 Git 分支 

Azure DevOps Services 提供連結工作項目 (劇本或工作) 與 Git 分支的便利方式。 這可讓您將劇本或工作直接連接至與它相關聯的程式碼。 

若要將工作項目連接至新分支，請按兩下工作項目，然後在快顯視窗中按一下 [+ 新增連結] 底下的 [建立新的分支]。  

![1](./media/collaborative-coding-with-git/1-sprint-board-view.png)

提供這個新分支的資訊，例如分支名稱、基底 Git 存放庫和分支。 選擇的 Git 存放庫必須是工作項目所屬的相同專案下的存放庫。 基底分支可以是主要分支或其他某些現有的分支。

![2](./media/collaborative-coding-with-git/2-create-a-branch.png)

為每個劇本工作項目建立 Git 分支是很好的做法。 然後，您可以根據劇本分支，為每個工作工作項目建立分支。 以對應到劇本-工作關係的這種階層方式組織分支，在您有許多人員使用相同專案的不同劇本時，或者有許多人員使用相同劇本的不同工作時，都很有幫助。 當每個小組成員使用不同的分支時，以及當共用分支的時候每個成員使用不同程式碼或其他構件時，衝突能夠降至最低。 

下圖說明建議的 TDSP 分支策略。 您可能不需要像這裡示範的這麼多分支，特別是當您只有一個或兩個人員使用相同專案，或只有一個人員使用劇本的所有工作。 但是將開發分支與主要分支分隔開來，永遠是良好的作法。 這可協助防止發行分支被開發活動中斷。 可以在[成功的 Git 分支模型](http://nvie.com/posts/a-successful-git-branching-model/)中找到 Git 分支模型的完整說明。

![3](./media/collaborative-coding-with-git/3-git-branches.png)

若要切換至您想要使用的分支，請在殼層命令中執行下列命令 (Windows 或 Linux)。 

    git checkout <branch name>

將 <分支名稱\> 變更為「主要」，會讓您切換回**主要**分支。 切換至工作分支之後，您可以開始使用該工作項目，開發完成此項目所需的程式碼或文件構件。 

您也可以將工作項目連結至現有的分支。 在工作項目的 [詳細資料] 分頁中，不是按一下 [建立新的分支]，而是按一下 [+ 新增連結]。 然後，選取您想要連結至工作項目的分支。 

![4](./media/collaborative-coding-with-git/4-link-to-an-existing-branch.png)

您也可以使用 Git Bash 命令建立新的分支。 如果 <基底分支名稱\> 遺漏，<新的分支名稱\> 會根據_主要_分支。 
    
    git checkout -b <new branch name> <base branch name>


## 2.<a name='WorkonaBranchandCommittheChanges-2'></a>使用分支並且認可變更 

現在假設您對工作項目的 data\_ingestion 分支進行一些變更，例如在本機機器中的分支上新增 R 檔案。 假設您是在 Git 殼層的該分支中，使用下列 Git 命令，可以將新增的 R 檔案認可至此工作項目的分支：

    git status
    git add .
    git commit -m"added a R scripts"
    git push origin data_ingestion

![5](./media/collaborative-coding-with-git/5-sprint-push-to-branch.png)

## 3.<a name='CreateapullrequestonVSTS-3'></a>在 Azure DevOps Services 上建立提取要求 

當您在一些認可和推送之後準備就緒時，若要將目前的分支合併至其基底分支，您可以在 Azure DevOps Services 上建立**提取要求**。 

請移至專案的主頁面，然後按一下 [程式碼]。 選取要合併的分支和分支合併所在的 Git 存放庫名稱。 然後按一下 [提取要求]、按一下 [新的提取要求]，在分支上的工作合併至其基底分支之前，建立提取要求檢閱。

![6](./media/collaborative-coding-with-git/6-spring-create-pull-request.png)

填入有關這項提取要求的一些描述、新增檢閱者，然後送出。

![7](./media/collaborative-coding-with-git/7-spring-send-pull-request.png)

## 4.<a name='ReviewandMerge-4'></a>檢閱及合併 

建立提取要求時，您的檢閱者會取得電子郵件通知，以檢閱提取要求。 檢閱者必須檢查變更是否有作用，以及在可行時測試要求者的變更。 根據其評量，檢閱者可以核准或拒絕提取要求。 

![8](./media/collaborative-coding-with-git/8-add_comments.png)

![9](./media/collaborative-coding-with-git/9-spring-approve-pullrequest.png)

完成檢閱之後，按一下 [完成] 按鈕，工作分支便會合併到其基底分支。 您可以選擇在工作分支合併之後將其刪除。 

![10](./media/collaborative-coding-with-git/10-spring-complete-pullrequest.png)

確認左上角要求標示為「已完成」。 

![11](./media/collaborative-coding-with-git/11-spring-merge-pullrequest.png)

當您返回 [程式碼] 底下的存放庫時，系統會告訴您，您已切換到主要分支。

![12](./media/collaborative-coding-with-git/12-spring-branch-deleted.png)

您也可以使用下列 Git 命令，將工作分支合併到其基底分支，並且在合併之後刪除工作分支：

    git checkout master
    git merge data_ingestion
    git branch -d data_ingestion

![13](./media/collaborative-coding-with-git/13-spring-branch-deleted-commandline.png)


 
## <a name="next-steps"></a>後續步驟

[資料科學工作的執行](execute-data-science-tasks.md)示範如何使用公用程式來完成幾個一般資料科學工作，例如互動式資料探索、資料分析、報告和模型建立。

還會提供逐步解說，說明**特定案例**之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)文章中會列出這些逐步解說以及簡短說明的連結。 這些逐步解說說明如何將雲端、內部部署工具及服務組合成工作流程或管線，以建立智慧型應用程式。 

