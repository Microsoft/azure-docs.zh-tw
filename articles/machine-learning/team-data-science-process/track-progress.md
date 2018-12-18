---
title: 執行資料科學專案 - Azure Machine Learning | Microsoft Docs
description: 資料科學家可如何追蹤資料科學專案的進度。
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
ms.openlocfilehash: 32390b05d2ec258a68ed4f53135399675105a7e9
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302080"
---
# <a name="track-progress-of-data-science-projects"></a>追蹤資料科學專案的進度

資料科學群組管理員、小組負責人和專案負責人需要追蹤其專案的進度，哪些工作已完成、由誰完成，以及仍然在待辦事項清單上。 

## <a name="azure-devops-dashboards"></a>Azure DevOps 儀表板
如果您使用 Azure DevOps，您可以建置儀表板來追蹤與給定的敏捷式專案相關聯的活動和工作項目。 

如需有關如何在 Azure DevOps 上建立並自訂儀表板和小工具的詳細資訊，請參閱下列指示集：

- [新增及管理儀表板](https://docs.microsoft.com/azure/devops/report/dashboards/dashboards)
- [將小工具新增至儀表板](https://docs.microsoft.com/azure/devops/report/dashboards/add-widget-to-dashboard)。

## <a name="example-dashboard"></a>範例儀表板

以下是簡單的範例儀表板，建置來追蹤敏捷式資料科學專案的短期衝刺活動，以及相關聯存放庫的認可數。 **左上**窗格會顯示：

- 目前短期衝刺的倒數計時， 
- 過去 7 天內每個存放庫的認可數
- 特定使用者的工作項目。 

其餘窗格會顯示專案的累計流量圖 (CFD)、待執行工作和已完成工作：

- **左下方**：CFD 為給定狀態中的工作數量，以灰色顯示已核准，藍色顯示已認可，綠色顯示完成。
- **右上方**：待執行工作圖表為剩下未完成的工作與剩餘時間)。
- **右下方**：已完成工作圖表為已完成的工作與工作總數量。

![儀表板](./media/track-progress/dashboard.png)

如需如何建置這些圖表的說明，請參閱[儀表板](https://docs.microsoft.com/azure/devops/report/dashboards/)中的快速入門和教學課程。
 
## <a name="next-steps"></a>後續步驟

也會提供逐步解說，說明**特定案例**之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)文章中會列出這些逐步解說以及簡短說明的連結。 這些逐步解說說明如何將雲端、內部部署工具及服務組合成工作流程或管線，以建立智慧型應用程式。 
