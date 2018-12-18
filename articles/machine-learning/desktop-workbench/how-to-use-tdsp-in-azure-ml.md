---
title: 使用 Team Data Science Process 範本來建構專案 | Microsoft Docs
description: 如何在 Azure Machine Learning 中將 Team Data Science Process (TDSP) 範本具現化，以建構專案來進行共同作業
services: machine-learning
documentationcenter: ''
author: deguhath
ms.author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 2fd0fe55b975010a1eaafeb455d61a6cb3ebff77
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970918"
---
# <a name="structure-projects-with-the-team-data-science-process-template"></a>使用 Team Data Science Process 範本來建構專案

本文件說明如何在較早版本的 Azure Machine Learning 中使用 Team Data Science Process (TDSP) 範本來建立資料科學專案。 這些範本可協助建構專案來進行共同作業和提供重現性。 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

## <a name="what-is-the-team-data-science-process"></a>什麼是 Team Data Science Process？
TDSP 是一個靈活、反覆的資料科學程序，可用於執行和提供進階的分析解決方案。 其設計目的是要改善企業組織中資料科學小組的共同作業與效率。 它透過四個主要元件來支援這些目標：

   * 標準的[資料科學生命週期](../team-data-science-process/lifecycle.md) \(英文\) 定義。
   * 標準化的專案結構、[專案文件和報告範本](https://github.com/Azure/Azure-TDSP-ProjectTemplate) \(英文\)。
   * 適用於專案執行的基礎結構與資源，例如分別是計算和儲存體基礎結構，以及程式碼存放區。
   * 適用於專案工作的[工具與公用程式](https://github.com/Azure/Azure-TDSP-Utilities) \(英文\)，例如：
      - 共同作業版本控制
      - 程式碼檢閱
      - 資料探索和模型化
      - 工作規劃

如需 TDSP 的更完整討論，請參閱 [Team Data Science Process 概觀](../team-data-science-process/overview.md) \(英文\)。

## <a name="why-should-you-use-the-tdsp-structure-and-templates"></a>為什麼您應該使用 TDSP 結構和範本？
標準化的結構、生命週期，以及資料科學專案的文件，是在資料科學團隊之間促進有效共同作業的關鍵。 使用 TDSP 範本建立機器學習專案，可為協調的團隊工作提供適用的架構。

我們先前發行了[適用於 TDSP 專案結構和範本的 GitHub 儲存機制](https://github.com/Azure/Azure-TDSP-ProjectTemplate) \(英文\)，可協助您達成這些目標。 但到目前為止，仍無法在資料科學工具中將 TDSP 結構和範本具現化。 您現在能夠建立機器學習專案，將 TDSP 結構和文件範本具現化。 

## <a name="things-to-note-before-creating-a-new-project"></a>建立新專案前需注意的事項
建立新專案「之前」，請檢閱下列各項：
* 檢閱 TDSP Machine Learning [範本](https://aka.ms/tdspamlgithubrepo) \(英文\)。
* 內容 ([docs] 資料夾中既有的內容除外) 大小必須小於 25 MB。 請參閱本清單後面的附註。
* [sample\_data] 資料夾只適用於小型資料檔案 (小於 5 MB)，您可以使用這類檔案來測試程式碼或進行早期開發。
* 儲存檔案 (例如 Word 和 PowerPoint 檔案) 會大幅增加 [docs] 資料夾的大小。 建議您尋找一個共同作業的 Wiki、[SharePoint](https://products.office.com/sharepoint/collaboration) 或其他共同作業資源來儲存這類檔案。
* 若要了解如何處理 Machine Learning 中的大型檔案和輸出，請參閱[保存變更和處理大型檔案](http://aka.ms/aml-largefiles)。

> [!NOTE]
> 所有在專案執行期間「不」使用的文件相關內容 (文字、Markdown、影像及其他文件檔案，readme.md 檔案除外) 都必須位於名為 "docs" (全部小寫) 的資料夾中。 [docs] 資料夾是 Machine Learning 執行時會忽略的特殊資料夾，如此才不會將此資料夾中的內容不必要地複製到計算目標中。 此資料夾中的物件也不計入專案大小的 25 MB 上限中。 舉例來說，[docs] 資料夾是儲存您文件所需大型影像檔的地方。 Git 仍會透過執行歷程記錄來追蹤這些檔案。 

## <a name="instantiate-the-tdsp-structure-and-templates-from-the-machine-learning-template-gallery"></a>從 Machine Learning 範本庫將 TDSP 結構和範本具現化
若要使用 TDSP 結構和文件範本來建立新專案，請完成下列程序。

### <a name="create-a-new-project"></a>建立新專案
若要建立新專案，請開啟 Azure Machine Learning。 在左上方窗格上的 [專案] 底下，選取加號 (**+**)，然後選取 [新增專案]。

![新增專案](./media/how-to-use-tdsp-in-azure-ml/instantiation-1.png)


### <a name="create-a-new-tdsp-structured-project"></a>建立新的 TDSP 結構化專案
   1. 在相關的方塊或清單中指定參數和資訊：

      - 專案名稱
      - 專案目錄
      - 專案描述
      - 空的 Git 存放庫路徑
      - 工作區名稱

   2. 然後在 [搜尋] 方塊中，輸入 **TDSP**。 
   3. 出現 [使用 TDSP 建構專案] 選項時，選取該範本。 
   4. 選取 [建立] 按鈕，以建立含 TDSP 結構的新專案。 如果您在建立專案期間 (在適當的方塊中) 提供空的 Git 儲存機制，則會在建立專案之後，使用專案結構和內容來填入該儲存機制。

![建立 TDSP 專案](./media/how-to-use-tdsp-in-azure-ml/instantiation-2.png)


## <a name="examine-the-tdsp-project-structure"></a>檢查 TDSP 專案結構
建立新專案之後，您可以檢查它的結構 (請參閱下圖中的左面板)。 它包含可供了解業務的所有標準化文件層面。 這些項目包括 TDSP 生命週期的各個階段、資料位置、定義，以及此文件範本的架構。 

所顯示的結構衍生自 [TDSP 專案結構、文件及成品範本](https://github.com/Azure/Azure-TDSP-ProjectTemplate) \(英文\) 中所發佈的 TDSP 結構，但有一些修改。 例如，將數個文件範本合併成一個 Markdown，也就是 [ProjectReport](https://aka.ms/tdspamlgithubrepoprojectreport)。 

### <a name="project-folder-structure"></a>專案資料夾結構
TDSP 專案範本包含下列最上層資料夾：
   - **code**：包含程式碼。
   - **docs**：包含專案相關的必要文件 (例如 Markdown 檔案和相關媒體)。
   - **sample_data**：包含可用來進行早期開發或測試的**範例 (小型)** 資料。 通常，這些集合不會超過幾個 (5) MB。 此資料夾不適用於完整或大型的資料集。

![範例資料](./media/how-to-use-tdsp-in-azure-ml/instantiation-3.png)


## <a name="use-the-tdsp-structure-and-templates"></a>使用 TDSP 結構和範本
您必須在結構和範本中新增專案特定資訊。 您應該使用執行和提供專案所需的程式碼和資訊來填入這些項目。 [ProjectReport](https://aka.ms/tdspamlgithubrepoprojectreport) 檔案是一個範本，您必須使用您專案的相關資訊來修改此範本。 它隨附一組問題，可協助您針對 [TDSP 生命週期](../team-data-science-process/lifecycle.md) \(英文\) 四個階段中的每個階段填寫資訊。

下圖的左面板中提供一個範例，顯示專案結構在執行期間或完成後看起來的樣子。 此專案來自 [Team Data Science Process 範例專案：在 Azure Machine Learning 中將美國人口普查資料的收入分類](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome) \(英文\) 範例專案。

![範例專案結構](./media/how-to-use-tdsp-in-azure-ml/instantiation-4.png)

## <a name="document-your-project"></a>為您的專案提供文件
如需有關如何為您專案提供文件的資訊，請參考 [TDSP 文件範本](https://github.com/Azure/Azure-TDSP-ProjectTemplate) \(英文\)。 在目前的 Machine Learning TDSP 文件範本中，建議您在 [ProjectReport](https://aka.ms/tdspamlgithubrepoprojectreport) \(英文\) 檔案中包含所有資訊。 您應該使用專案特定的資訊來填寫這個範本。 

我們也提供 [ProjectLearnings](https://aka.ms/tdspamlgithubrepoprojectlearnings) 範本。 您可以使用此範本來包含主要專案文件中未包含但仍值得記錄的任何資訊。 

### <a name="example-project-report"></a>範例專案報告
您可以取得一份[範例專案報告](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome/blob/master/docs/deliverable_docs/ProjectReport.md)。 這份[美國收入分類範例專案](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome) \(英文\) 專案報告說明如何針對資料科學專案具現化和使用 TDSP 範本。

## <a name="next-steps"></a>後續步驟
為了協助您了解如何在 Machine Learning 專案中使用 TDSP 結構和範本，我們在 Machine Learning 的文件中提供數個已完成的專案範例：

- 如需說明如何在 Machine Learning 中建立 TDSP 專案的範例，請參閱 [Team Data Science Process 範例專案：在 Azure Machine Learning 中將美國人口普查資料的收入分類](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome) \(英文\)。
- 如需在 Machine Learning 之已將 TDSP 具現化的專案中，於自然語言處理 (NLP) 中使用深入學習的範例，請參閱[使用自然語言處理搭配深入學習來進行生物醫學實體辨識](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction) \(英文\)。

