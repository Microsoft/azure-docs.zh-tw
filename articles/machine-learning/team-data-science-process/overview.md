---
title: 什麼是 Team Data Science Process？ | Microsoft Docs
description: 提供資料科學方法，用以產出預測分析解決方案和智慧型應用程式。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: b1f677bb-eef5-4acb-9b3b-8a5819fb0e78
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/20/2017
ms.author: deguhath
ms.openlocfilehash: e6e6b8e03102825a15035adbea80ac5e9f3fa404
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996446"
---
# <a name="what-is-the-team-data-science-process"></a>什麼是 Team Data Science Process？

Team Data Science Process (TDSP) 是一種敏捷式反覆資料科學方法，可有效產出預測分析解決方案和智慧型應用程式。 TDSP 有助於改善團隊共同作業和學習。 其中包含 Microsoft 及業界其他公司的最佳做法和結構，有助於順利實作資料科學計劃。 目標是協助公司完全了解其分析程式的優點。

本文提供 TDSP 和其主要元件的概觀。 本文對於程序提供的一般描述可以使用各種工具實作。 其他連結的主題詳細描述流程的生命週期中涉及的專案工作與角色。 另外也指引如何使用我們團隊實作 TDSP 所用的一組特定 Microsoft 工具和基礎結構實作 TDSP。

## <a name="key-components-of-the-tdsp"></a>TDSP 的重要元件

TDSP 包含下列主要元件：

- **資料科學生命週期**定義
- **標準化專案結構**
- 資料科學專案的**基礎結構和資源**
- 專案執行的**工具和公用程式**


## <a name="data-science-lifecycle"></a>資料科學生命週期

Team Data Science Process (TDSP) 會提供建構資料科學專案開發的生命週期。 生命週期會概述專案在執行時通常會遵循之從開始到完成的步驟。

如果您正在使用另一個資料科學生命週期 (例如 [CRISP-DM](https://wikipedia.org/wiki/Cross_Industry_Standard_Process_for_Data_Mining)、[KDD](https://wikipedia.org/wiki/Data_mining#Process) 或您組織本身的自訂程序)，您仍可在這些開發生命週期的內容中使用工作型 TDSP。 大致而言，這些不同的方法有許多共通點。 

此生命週期是針對在智慧型應用程式中隨附的資料科學專案所設計。 這些應用程式會部署機器學習服務或人工智慧模型來做預測性分析。 探勘資料科學專案或臨機操作分析專案也可以從使用此程序而獲益。 但是，在這種情況下，可能不需要所述的某些步驟。    

生命週期可描繪出專案一般執行時 (通常會反覆進行) 的主要階段：

* **了解商務**
* **資料取得與認知**
* **模型化**
* **部署**
* **客戶接受度**

以下是 **Team Data Science Process 生命週期**的視覺化呈現。 

![TDSP-Lifecycle2](./media/overview/tdsp-lifecycle2.png) 

[小組資料科學程序生命週期](lifecycle.md)主題說明 TDSP 生命週期每個階段的目標、工作和文件構件。 這些工作和構件與專案角色相關聯：

- 解決方案架構師
- 專案經理
- 資料科學家
- 專案負責人 

下圖以格線檢視呈現這些角色 (水平軸) 的生命週期每個階段 (垂直軸) 相關聯的工作 (藍色) 和構件 (綠色)。 

[ ![TDSP 角色和工作](./media/overview/tdsp-tasks-by-roles.png) ](./media/overview/tdsp-tasks-by-roles.png#lightbox)

## <a name="standardized-project-structure"></a>標準化專案結構

使所有專案共用目錄結構，並使用專案文件的範本讓小組成員能輕易找到其專案的資訊。 Git、TFS 或 Subversion 之類的版本控制系統 (VCS) 儲存所有的程式碼和文件，有助於小組共同作業。 在 Jira、Rally 與 Azure DevOps 之類的敏捷式專案追蹤系統中的追蹤工作和功能，能夠更密切追蹤個別功能的程式碼。 這類追蹤也可讓小組獲得進一步的成本估計。 TDSP 建議在 VCS 上對於每個專案建立個別的儲存機制，以利版本控制、資訊安全和共同作業。 所有專案的標準化結構有助於累積整個組織的機構知識。

我們提供資料夾結構提供範本和標準位置的必要文件。 此資料夾結構包括內含資料探索和功能擷取程式碼的檔案，以及記錄模型反覆項目的檔案。 這些範本讓小組成員更容易了解其他人完成的工作，並且更容易將新成員加入至小組中。 Markdown 格式的文件範本很容易檢視和更新。 範本可用來對於每個專案提供重要問題的檢查清單，確保問題獲得妥善定義，而且交付項目符合預期的品質。 範例包括：

- 記錄商務問題和專案範圍的專案許可
- 記錄原始資料結構和統計資料的資料報告
- 記錄衍生功能的模型報告
- 模型效能計量，例如 ROC 曲線或 MSE


[ ![TDSP 目錄](./media/overview/tdsp-dir-structure.png) ](./media/overview/tdsp-dir-structure.png#lightbox)

可以從 [Github](https://github.com/Azure/Azure-TDSP-ProjectTemplate) 複製目錄結構。

## <a name="infrastructure-and-resources-for-data-science-projects"></a>資料科學專案的基礎結構和資源  

TDSP 提供管理共用分析和儲存體基礎結構的建議，例如：

- 儲存資料集的雲端檔案系統 
- 資料庫
- 巨量資料 (Hadoop 或 Spark) 叢集 
- 機器學習服務 

分析和儲存體基礎結構可以在雲端，也可以是內部部署。 這裡儲存著原始資料集和處理後的資料集。 這個基礎架構能夠進行可重現的分析。 它也可避免重複，重複可能會導致不一致和不必要的基礎結構成本。 提供的工具可用來佈建和追蹤共用的資源，並允許每個小組成員安全地連線到這些資源。 讓專案成員建立一致的計算環境也是個不錯的做法。 不同的小組成員可以複寫和驗證實驗。

以下是小組處理多個專案並共用各種雲端分析基礎結構元件的範例。

[ ![TDSP 基礎結構](./media/overview/tdsp-analytics-infra.png) ](./media/overview/tdsp-analytics-infra.png#lightbox) 


## <a name="tools-and-utilities-for-project-execution"></a>專案執行的工具和公用程式

在大部分的組織中引進流程是相當有挑戰性。 對於實作資料科學流程和生命週期所提供的工具有助於降低採用的障礙，並提升採用的一致性。 TDSP 提供一組初始的工具和指令碼，可供團隊快速採用 TDSP。 這也有助於資料科學開發週期中部分常見的工作自動進行，例如資料探索和基準模型。 提供妥善定義的結構，以供個人將共用的工具和公用程式貢獻到小組共用的程式碼儲存機制。 小組或組織內的其他專案接著即可使用這些資源。 TDSP 也計畫向整個社群貢獻工具和公用程式。 可以從 [Github](https://github.com/Azure/Azure-TDSP-Utilities) 複製 TDSP 公用程式。


## <a name="next-steps"></a>後續步驟

[小組資料科學流程：角色和工作](https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/roles-tasks.md)概述依據此流程進行標準化的資料科學團隊之中的重要人員角色及其相關工作。 
