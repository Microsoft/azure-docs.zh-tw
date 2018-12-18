---
title: 敏捷式資料科學專案開發 - Azure Machine Learning | Microsoft Docs
description: 開發人員如何藉由使用 Team Data Science Process，在專案小組中以系統化、版本控制，以及共同作業的方式執行資料科學專案。
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
ms.openlocfilehash: a032127d249f944d08cc6578a03f1a7e5a658361
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44301084"
---
# <a name="agile-development-of-data-science-projects"></a>敏捷式資料科學專案開發

本文件說明開發人員如何藉由使用 [Team Data Science Process](overview.md) (TDSP)，在專案小組中以系統化、版本控制，以及共同作業的方式執行資料科學專案。 TDSP 是 Microsoft 開發的架構，提供結構化的一系列活動，以有效率地執行雲端式、預測性分析解決方案。 如需人員角色的大綱，以及對此程序進行標準化之資料科學小組所處理相關聯工作的大綱，請參閱 [Team 資料科學程序角色和工作](roles-tasks.md)。 

本文章包含如何執行以下作業的指示： 

1. 針對專案中涉及的工作項目進行**短期衝刺計劃**。<br> 如果您不熟悉短期衝刺計劃，可以在[這裡](https://en.wikipedia.org/wiki/Sprint_(software_development) "這裡")找到詳細資料和一般資訊。 
2. **新增工作項目**至短期衝刺。 

> [!NOTE]
> 下列指示集中概述了使用 Azure DevOps Services 設定 TDSP 小組環境所需的步驟。 他們會指定如何使用 Azure DevOps Services 完成這些工作，因為這是在 Microsoft 中實作 TDSP 的方式。  上述清單中的項目 (3) 和 (4) 是您選擇使用 Azure DevOps Services 時自然得到的權益。 如果針對您的群組使用另一個程式碼裝載平台，必須由小組負責人完成的工作通常不會變更。 但是完成這些工作的方式將會不同。 例如，區段 6 中的項目：**連結工作項目與 Git 分支**，可能無法像在 Azure DevOps Services 上一樣簡單。
>
>

下圖說明實作資料科學專案時涉及的典型短期衝刺計劃、編碼，以及原始檔控制工作流程：

![1](./media/agile-development/1-project-execute.png)


##  1.<a name='Terminology-1'></a>術語 

在 TDSP 短期衝刺計劃架構中，有四種常用的**工作項目**類型：**功能**、**使用者劇本**、**工作**和 **Bug**。 每個專案都會為所有工作項目維護單一待辦項目。 在專案下的 Git 存放庫層級沒有任何待辦項目。 其定義如下：

- **功能**：對應至專案參與的功能。 用戶端的不同參與會被視為不同的功能。 同樣地，最好將用戶端專案的不同階段視為不同的功能。 如果您選擇例如 ***ClientName-EngagementName*** 的結構描述來為功能命名，則您可以輕易地從名稱本身辨識專案/參與的內容。
- **劇本**：劇本是與端對端完成功能 (專案) 所需的工作項目不同的工作項目。 劇本的範例包括：
    - 取得資料 
    - 瀏覽資料 
    - 產生功能
    - 建立模型
    - 運用模型 
    - 重新定型模型
- **工作**：工作是可指派的程式碼或文件工作項目或其他活動，必須完成這些項目才能完成特定劇本。 例如，劇本「取得資料」中的工作可能是：
    -  取得 SQL Server 的認證 
    -  將資料上傳至 SQL 資料倉儲。 
- **Bug**：Bug 通常是指完成工作時完成之現有程式碼或文件所需的修正。 如果 BUG 是因為分別遺漏階段或工作所造成，它可以提升為劇本或工作。 

> [!NOTE]
> 我們從資料科學中使用的軟體程式碼管理 (SCM) 借用功能、劇本、工作及 BUG 的概念。 它們可能會與傳統 SCM 定義稍有不同。
>
>

> [!NOTE]
> 資料科學家可能會覺得較喜歡使用與 TDSP 生命週期階段一致的敏捷式範本。 我們已經針對這一點，建立了衍生的敏捷式短期衝刺計劃範本，其中的 Epics、劇本等等會由 TDSP 生命週期階段或子階段取代。 如需如何建立敏捷式軟體開發的範本的指示，請參閱[設定敏捷式軟體開發資料科學程序，在 Visual Studio Online](agile-development.md#set-up-agile-dsp-6)。
>
>

## 2.<a name='SprintPlanning-2'></a>短期衝刺計劃 

短期衝刺計劃對於專案優先順序，以及資源規劃和配置相當有用。 許多資料科學家正在參與許多專案，每個專案都可能需要幾個月才能完成。 專案通常是以不同的步調進行。 在 Azure DevOps Services 上，您可以輕鬆地建立、管理和追蹤專案中的工作項目，並且執行短期衝刺計劃，以確保您的專案如預期般進行。 

請遵循[此連結](https://www.visualstudio.com/en-us/docs/work/scrum/sprint-planning)以取得 Azure DevOps Services 中短期衝刺計劃的逐步指示。 


## 3.<a name='AddFeature-3'></a>新增功能  

在專案下建立專案存放庫之後，請移至小組 [概觀] 分頁，然後按一下 [管理工作]。

![2](./media/agile-development/2-sprint-team-overview.png)

若要在待處理項目中包含功能，請按一下 [待處理項目]  -->  [功能]  -->  [新增]，輸入功能**標題** (通常是您的專案名稱)，然後按一下 [新增]。

![3](./media/agile-development/3-sprint-team-add-work.png)

按兩下您建立的功能。 填入描述、指派此功能的小組成員，以及設定此功能的計劃參數。 

您也可以將此功能連結至專案存放庫。 按一下 [開發] 區段底下的 [新增連結]。 完成編輯功能之後，按一下 [儲存並關閉] 以結束。


## 4.<a name='AddStoryunderfeature-4'></a>在功能底下新增劇本 

在功能底下可以新增劇本以描述完成 (功能) 專案所需的主要步驟。 若要新增劇本，請按一下待處理項目檢視中功能左側的 **+** 符號。  

![4](./media/agile-development/4-sprint-add-story.png)

您可以在快顯視窗中編輯劇本的詳細資料，例如狀態、描述、註解、計劃和優先順序。

![5](./media/agile-development/5-sprint-edit-story.png)

您也可以按一下 [開發] 底下的 [+ 新增連結]，將此劇本連結至現有的存放庫。 

![6](./media/agile-development/6-sprint-link-existing-branch.png)


## 5.<a name='AddTaskunderstory-5'></a>將工作新增至劇本 

工作是完成各個劇本所需的特定詳細步驟。 完成劇本的所有工作之後，劇本應該也會完成。 

若要將工作新增至劇本，請按一下劇本項目旁邊的 **+** 符號，選取 [工作]，然後在快顯視窗中填入此工作的詳細資訊。

![7](./media/agile-development/7-sprint-add-task.png)

建立功能、劇本和工作之後，您可以在 [待處理項目] 或 [面板] 檢視中檢視它們，來追蹤其狀態。

![8](./media/agile-development/8-sprint-backlog-view.png)

![9](./media/agile-development/9-link-to-a-new-branch.png)


## 6.<a name='set-up-agile-dsp-6'></a>在 Visual Studio Online 中設定敏捷式 TDSP 工作範本

本文說明如何設定敏捷式資料科學程序範本，使用 TDSP 資料科學生命週期階段並使用 Visual Studio Online (vso) 來追蹤工作項目。 下列步驟逐步解說設定資料科學專屬敏捷式程序範本的範例 AgileDataScienceProcess，並示範如何以範本作為基礎來建立資料科學工作項目。

### <a name="agile-data-science-process-template-setup"></a>敏捷式資料科學程序範本設定

1. 瀏覽至伺服器首頁，[設定] -> [程序]。

    ![10](./media/agile-development/10-settings.png) 

2. 瀏覽至 [敏捷式] 下的 [所有程序] -> [程序]，然後按一下 [建立繼承的程序]。 然後將程序命名為 "AgileDataScienceProcess"，並按一下 [建立程序]。

    ![11](./media/agile-development/11-agileds.png)

3. 在 [AgileDataScienceProcess] -> [工作項目類型] 索引標籤下，停用 [Epic]、[功能]、[使用者劇本]，和 [工作] 等工作項目類型，方法為 [設定] -> [停用]

    ![12](./media/agile-development/12-disable.png)

4. 瀏覽至 [AgileDataScienceProcess] -> [待辦項目層級] 索引標籤。按一下 [設定] -> [編輯/重新命名]，將 [Epics] 重新命名為「TDSP 專案」。 在相同的對話方塊中，按一下「資料科學專案」中的 [+ 新增工作項目類型]，然後將 [預設工作項目類型] 的值設為「TDSP 專案」 

    ![13](./media/agile-development/13-rename.png)  

5. 同樣地，將待辦項目名稱 [功能] 變更為「TDSP 階段」，並將下列項目新增至 [新工作項目類型]：

    - 了解商務
    - 資料擷取
    - 模型化
    - 部署

6. 將 [使用者劇本] 重新命名為「TDSP 次階段」，預設工作項目類型設定為新建立的「TDSP 次階段」類型。

7. 將「工作」設定為新建立的工作項目類型「TDSP 工作」 

8. 在這些步驟之後，待辦項目層級看起來應該像這樣：

    ![14](./media/agile-development/14-template.png)  

 
### <a name="create-data-science-work-items"></a>建立資料科學工作項目

建立資料科學程序範本之後，您就可以建立及追蹤對應至 TDSP 生命週期的資料科學工作項目。

1. 當您建立新的專案時，請選取 "Agile\AgileDataScienceProcess" 作為 [工作項目程序]：

    ![15](./media/agile-development/15-newproject.png)

2. 瀏覽至新建立的專案，然後按一下 [工作] -> [待辦項目]。

3. 按一下 [設定小組設定] 並檢查「TDSP 專案」來顯示「TDSP 專案」；然後儲存。

    ![16](./media/agile-development/16-enabledsprojects.png)

4. 現在您可以開始建立資料科學專屬工作項目。

    ![17](./media/agile-development/17-dsworkitems.png)

5. 資料科學專案工作項目的顯示方式範例如下：

    ![18](./media/agile-development/18-workitems.png)


## <a name="next-steps"></a>後續步驟

[使用 Git 共同作業撰寫程式碼](collaborative-coding-with-git.md)描述如何使用 Git 作為共用的程式碼開發架構來執行資料科學專案之共同作業程式碼開發，以及如何將這些撰寫程式碼活動連結到使用敏捷式程序規劃的工作。

以下是敏捷式程序的其他資源連結。

- 敏捷式流程   [https://www.visualstudio.com/en-us/docs/work/guidance/agile-process](https://www.visualstudio.com/en-us/docs/work/guidance/agile-process)
- 敏捷式流程工作項目類型和工作流程   [https://www.visualstudio.com/en-us/docs/work/guidance/agile-process-workflow](https://www.visualstudio.com/en-us/docs/work/guidance/agile-process-workflow)


也會提供逐步解說，說明**特定案例**之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)文章中會列出這些逐步解說以及簡短說明的連結。 這些逐步解說說明如何將雲端、內部部署工具及服務組合成工作流程或管線，以建立智慧型應用程式。 
