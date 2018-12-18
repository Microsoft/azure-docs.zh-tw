---
title: Azure 最佳做法中的管理解決方案 | Microsoft Docs
description: ''
services: operations-management-suite
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 1915e204-ba7e-431b-9718-9eb6b4213ad8
ms.service: operations-management-suite
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2017
ms.author: bwren
ms.openlocfilehash: 0bd5e19e00dbae1d0ece27d0498a1f599dba05b7
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33887388"
---
# <a name="best-practices-for-creating-management-solutions-in-azure-preview"></a>在 Azure 中建立管理解決方案的最佳做法 (預覽)
> [!NOTE]
> 這是在 Azure 中建立管理解決方案 (目前處於預覽狀態) 的預備文件。 以下所述的任何結構描述可能會有所變更。  

本文提供在 Azure 中[建立管理解決方案檔](monitoring-solutions-solution-file.md)的最佳做法。  本資訊會在識別出其他最佳作法時更新。

## <a name="data-sources"></a>資料來源
- 資料來源可以[使用 Resource Manager 範本設定](../log-analytics/log-analytics-template-workspace-configuration.md)，但不應該將資源來源包含在解決方案檔中。  原因是設定資料來源目前並非等冪，這表示您的解決方案可能會覆寫使用者工作區中現有的設定。<br><br>例如，您的解決方案可能需要來自應用程式事件記錄檔的警告和錯誤事件。  如果您在解決方案中將此指定為資料來源，在使用者已在其工作區中設定此項的情況下，會有移除資訊事件的風險。  如果您包含所有的事件，則可能會收集使用者工作區中過多的資訊事件。

- 如果您的解決方案需要來自其中一個標準資料來源的資料，則您應該將此定義為必要條件。  請在文件中表示客戶必須自行設定資料來源。  
- 將[資料流程驗證](../log-analytics/log-analytics-view-designer-tiles.md)訊息新增至解決方案中的任何檢視，以指示使用者關於需要設定以收集必要資料的資料來源資訊。  找不到必要資料時，此訊息會顯示在檢視的圖格中。


## <a name="runbooks"></a>Runbook
- 為您解決方案中每個需要依排程執行的 Runbook 新增[自動化排程](../automation/automation-schedules.md)。
- 在您的解決方案中包含 [IngestionAPI 模組 (英文)](https://www.powershellgallery.com/packages/OMSIngestionAPI/1.5)，以供將資料寫入 Log Analytics 存放庫的 Runbook 使用。  將解決方案設定為[參考](monitoring-solutions-solution-file.md#solution-resource)此資源，讓它在移除解決方案後仍能保留。  這可讓多個解決方案共用模組。
- 使用[自動化變數](../automation/automation-schedules.md)來將值提供給使用者於稍後可能會想要變更的解決方案。  即使解決方案已設定為包含變數，其值仍可以變更。

## <a name="views"></a>Views
- 所有的解決方案都應該包含會在使用者的入口網站中顯示的單一檢視。  檢視可以包含多個[視覺效果組件](../log-analytics/log-analytics-view-designer-parts.md)，以說明不同的資料集。
- 將[資料流程驗證](../log-analytics/log-analytics-view-designer-tiles.md)訊息新增至解決方案中的任何檢視，以指示使用者關於需要設定以收集必要資料的資料來源資訊。
- 將解決方案設定為[包含](monitoring-solutions-solution-file.md#solution-resource)該檢視，讓檢視會隨著解決方案一起移除。

## <a name="alerts"></a>警示
- 將收件者清單定義為解決方案檔中的參數，讓使用者可以在安裝解決方案時定義它們。
- 將解決方案設定為[參考](monitoring-solutions-solution-file.md#solution-resource)警示規則，讓使用者可以變更其設定。  他們可能會想要做出如修改收件者清單、變更警示閾值，或停用警示規則等變更。 


## <a name="next-steps"></a>後續步驟
* 逐步了解[設計與建置管理解決方案](monitoring-solutions-creating.md)的基本程序。
* 了解如何[建立解決方案檔](monitoring-solutions-solution-file.md)。
* 在您的管理解決方案中[新增儲存的搜尋和警示](monitoring-solutions-resources-searches-alerts.md)。
* 在您的管理解決方案中[新增檢視](monitoring-solutions-resources-views.md)。
* 在您的管理解決方案中[新增自動化 Runbook 及其他資源](monitoring-solutions-resources-automation.md)。

