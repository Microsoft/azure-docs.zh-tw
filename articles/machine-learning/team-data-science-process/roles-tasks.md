---
title: Team Data Science Process 角色和工作 - Azure | Microsoft Docs
description: 資料科學小組專案的重要元件、人員角色和相關工作概述。
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
services: machine-learning
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/04/2017
ms.author: deguhath
ms.openlocfilehash: 3e0e9a59d7f77355077fc1f7781eb3be4dbe0532
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44296698"
---
# <a name="team-data-science-process-roles-and-tasks"></a>Team Data Science Process 角色和工作

Team Data Science Process 是一種由 Microsof 開發的架構，可提供結構化方法來有效建立預測分析解決方案和智慧型應用程式。 本文概述重要人員角色，以及對此程序進行標準化之資料科學小組所處理的相關工作。 

本簡介會連結至教學課程，其中提供如何針對整個資料科學團隊、資料科學小組和專案設定 TDSP 環境的相關指示。 也會提供在教學課程中使用 Azure DevOps 的詳細指引。 Azure DevOps 提供程式碼託管平台和敏捷式規劃工具，以便您管理小組工作、控制存取權及管理存放庫。 

您可以使用此資訊在自己的程式碼託管和敏捷式規劃工具上實作 TDSP。 

## <a name="structures-of-data-science-groups-and-teams"></a>資料科學團隊和小組的結構

企業中的資料科學功能通常會組成下列階層：

1. ***資料科學團隊***

2. ***團隊中的資料科學小組***

在這種結構中，將會有團隊和小組負責人。 一般而言，資料科學專案是由資料科學小組進行，小組成員可能包含專案負責人 (負責專案管理和控管工作) 和資料科學家或工程師 (個別參與者 / 技術人員，將會執行專案的資料科學和資料工程部分)。 在執行之前，設定和控管是由團隊、小組或專案負責人進行。

## <a name="definition-of-four-tdsp-roles"></a>四個 TDSP 角色定義
在上述假設下，小組人員有四個不同的角色：

1. ***團隊管理員***。 團隊管理員是在企業中整個資料科學單位的管理員。 資料科學單位可能有多個小組，而每個小組負責不同商業垂直市場中的多個資料科學專案。 團隊管理員可能會將他們的工作委派給代理人，但是與角色相關的工作會改變。

2. ***小組負責人***。 小組負責人需管理企業資料科學單位中的小組。 小組是由多個資料科學家所組成。 若為只有少數資料科學家的資料科學單位，團隊管理員和小組負責人可能是同一人。

3. ***專案負責人***。 專案負責人會管理特定資料科學專案的個別資料科學家日常活動。

4. ***專案個別參與者***。 資料科學家、商務分析師、資料工程師、架構師等。專案個別參與者可執行資料科學專案。 


> [!NOTE]
> 視企業結構而定，一個人可能扮演一個以上的角色，或可能有多個人扮演同一個角色。 這種情況常見於小型企業，或資料科學組織人員很少的企業。

## <a name="tasks-to-be-completed-by-four-personnel"></a>由上述四種人完成的工作

下圖按照以 Microsoft 的概念採用與實作 Team Data Science Process 的角色，描述人員的主要工作。 

![角色和工作概觀](./media/roles-tasks/overview-tdsp-top-level.png)

此結構描述以及指派給 TDSP 中每個角色之工作的下列詳細概述，可協助您根據您在組織中的職責選擇適當的教學課程。

> [!NOTE]
> 下列指示會說明如何設定 TDSP 環境，以及在 Azure DevOps 中完成其他資料科學工作的步驟。 我們會指定如何使用 Azure DevOps 完成這些工作，因為我們將它用來在 Microsoft 實作 TDSP。 Azure DevOps 藉由整合工作項目的管理以追蹤各項工作和程式碼託管服務 (其用於共用公用程式、組織版本及提供角色型安全性)，來達成共同作業。 您也可以視情況選擇其他平台來實作 TDSP 所述的工作。 但視您的平台而定，某些透過 Azure DevOps 使用的功能可能無法使用。 
>
>這裡的指示也會使用 Azure 雲端上的[資料科學虛擬機器 (DSVM)](http://aka.ms/dsvm)作為分析桌面，該桌面已預先設定數個熱門的資料科學工具並與各種 Microsoft 軟體和 Azure 服務整合。 您可以使用 DSVM 或任何其他開發環境來實作 TDSP。 


## <a name="group-manager-tasks"></a>團隊管理員工作

團隊管理員 (或指定的 TDSP 系統管理員) 會完成下列工作以採用 TDSP：

- 在程式碼託管平台 (如 Github、Git、Azure DevOps 或其他平台) 上建立**團隊帳戶**
- 在團隊帳戶上建立**專案範本存放庫**，並從 Microsoft TDSP 小組所開發的專案範本存放庫植入該存放庫。 Microsoft 的 TDSP 專案範本存放庫 
    - 提供**標準化目錄結構**，包括資料、程式碼和文件的目錄。 
    - 提供一組**標準化文件範本**，以引導有效率的資料科學程序。 
- 建立**公用程式存放庫**，並從 Microsoft TDSP 小組所開發的公用程式存放庫植入該存放庫。 Microsoft 的 TDSP 公用程式存放庫會提供 
    - 一組實用的公用程式，讓資料科學家的工作更有效率，其中包括用於互動式資料探勘、分析和報告，以及基準模型化和報告的公用程式。
- 在團隊帳戶上設定這兩個存放庫的**安全性控制原則**。  

如需詳細的逐步指示，請參閱[資料科學小組的團隊管理員工作](group-manager-tasks.md)。 


## <a name="team-lead-tasks"></a>小組負責人工作

小組負責人 (或指定的專案管理員) 會完成下列工作以採用 TDSP：

- 若已選取 Azure DevOps 作為版本控制和共同作業的程式碼託管平台，請在團隊的 Azure DevOps Services 上建立**專案**。 否則，可以略過此工作。
- 請在專案底下建立**專案範本存放庫**，並從群組管理員或管理員代表設定的群組專案範本存放庫植入此存放庫。 
- 建立**小組公用程式存放庫**，並將小組專用公用程式新增至存放庫。 
- (選擇性)建立 **[Azure 檔案儲存體](https://azure.microsoft.com/services/storage/files/)**，用來儲存對整個小組而言很實用的資料資產。 其他小組成員可以在其分析桌面上掛接此共用雲端檔案存放區。
- (選擇性) 將 Azure 檔案儲存體掛接至小組負責人的**資料科學虛擬機器** (DSVM) 並在其上新增資料資產。
- 藉由新增小組成員和設定其權限來設定**安全性控制**。 

如需詳細的逐步指示，請參閱[資料科學小組的小組負責人工作](team-lead-tasks.md)。  


## <a name="project-lead-tasks"></a>專案負責人工作

專案負責人會完成下列工作以採用 TDSP：

- 在專案之下建立**專案存放庫**，並從專案範本存放庫植入該存放庫。 
- (選擇性)建立 **Azure 檔案儲存體**，用來儲存專案的資料資產。 
- (選擇性) 將 Azure 檔案儲存體掛接至專案負責人的**資料科學虛擬機器** (DSVM) 並在其上新增專案資料資產。
- 藉由新增專案成員和設定其權限來設定**安全性控制**。 

如需詳細的逐步指示，請參閱[資料科學小組的專案負責人工作](project-lead-tasks.md)。 

## <a name="project-individual-contributor-tasks"></a>專案個別參與者工作

專案個別參與者 (通常是資料科學家) 會完成下列工作，以使用 TDSP 進行資料科學專案：

- 複製專案負責人所設定的**專案存放庫**。 
- (選擇性) 在**資料科學虛擬機器** (DSVM) 上掛接小組和專案的共用 **Azure 檔案儲存體**。
- 執行專案。 

 
如需參與專案的詳細逐步指示，請參閱[資料科學小組的專案個別參與者](project-ic-tasks.md)。 


## <a name="data-science-project-execution"></a>資料科學專案執行
 
資料科學家、專案負責人和小組負責人可以遵循一組相關的指示來建立工作項目，以追蹤專案從開始到結束所需的工作和階段。 使用 git 也可促進資料科學家之間的共同作業，並確保在專案執行期間產生的構件會由所有專案成員進行版本控制和共用。

已根據工作項目和專案 git 存放庫均位於 Azure DevOps 的這項假設，開發為了執行專案所提供的指示。 將 Azure DevOps 用於兩者，可讓您連結工作項目與您專案存放庫的 Git 分支。 如此一來，您可以輕鬆地追蹤已對工作項目進行的作業。 

下圖概述使用 TDSP 執行專案的工作流程。

![典型的資料科學專案執行](./media/roles-tasks/overview-project-execute.png)

工作流程包含可分為三個活動的步驟：

- 短期衝刺計劃 (專案負責人)
- 在 Git 分支上開發構件來處理工作項目 (資料科學家)
- 檢閱程式碼以及合併分支與主要分支 (專案負責人或其他小組成員)

如需專案執行工作流程的詳細逐步指示，請參閱[執行資料科學專案](project-execution.md)。

## <a name="project-structure"></a>專案結構

使用此[專案範本存放庫](https://github.com/Azure/Azure-TDSP-ProjectTemplate)來支援有效率的專案執行和共同作業。 此存放庫提供可用於自有 TDSP 專案的標準化目錄結構和文件範本。

## <a name="next-steps"></a>後續步驟

深入探索 Team Data Science Process 定義的角色和工作描述：

- [資料科學小組的群組管理員工作](group-manager-tasks.md)
- [資料科學小組的小組負責人工作](team-lead-tasks.md)
- [資料科學小組的專案負責人工作](project-lead-tasks.md)
- [資料科學小組的專案個別參與者](project-ic-tasks.md)