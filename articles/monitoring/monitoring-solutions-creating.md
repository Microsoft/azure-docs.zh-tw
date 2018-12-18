---
title: 在 Azure 中建置管理解決方案 | Microsoft Docs
description: 管理解決方案包含 Azure 中客戶可新增至其 Log Analytics 工作區的已封裝管理案例。  這篇文章提供詳細資料，說明如何建立要用於自己的環境中或可供客戶使用的管理解決方案。
services: monitoring
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 1915e204-ba7e-431b-9718-9eb6b4213ad8
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2017
ms.author: bwren
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a07a17105b4d84b51689e9636cfacc7a3b5428ad
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528022"
---
# <a name="design-and-build-a-management-solution-in-azure-preview"></a>在 Azure 中設計和建置管理解決方案 (預覽)
> [!NOTE]
> 這是在 Azure 中建立管理解決方案 (目前處於預覽狀態) 的預備文件。 以下所述的任何結構描述可能會有所變更。

[管理解決方案]( monitoring-solutions.md)提供客戶可新增至 Log Analytics 工作區的已封裝管理案例。  本文介紹的基本程序，可用來設計和建置適用於最常見需求的管理解決方案。  如果您不了解如何建置管理解決方案，則可以先使用此程序，之後再隨著需求的發展，運用相關概念來建置更複雜的解決方案。

## <a name="what-is-a-management-solution"></a>何謂管理解決方案？

管理解決方案包含可共同運作以達成特殊監視案例的 Azure 資源。  解決方案會實作為[資源管理範本](../azure-resource-manager/resource-manager-template-walkthrough.md)，範本中包含如何在安裝解決方案時安裝及設定其內含資源的詳細資料。

基本策略是藉由在 Azure 環境中建置個別元件來開始您的管理解決方案。  當功能正常運作後，您就可以開始將這些元件封裝到[管理解決方案檔]( monitoring-solutions-solution-file.md)。 


## <a name="design-your-solution"></a>設計您的解決方案
下圖所示的是最常見的管理解決方案模式。  下面會討論此模式的不同元件。

![管理解決方案概觀](media/monitoring-solutions-creating/solution-overview.png)


### <a name="data-sources"></a>資料來源
設計解決方案時的第一個步驟，是決定您需要從 Log Analytics 存放庫取得的資料。  這項資料可由[資料來源](../log-analytics/log-analytics-data-sources.md)或[另一個解決方案]( monitoring-solutions.md)所收集，否則您的解決方案就可能需要提供資料收集程序。

有數種方式可以將資料來源收集在 Log Analytics 存放庫中，您可以在 [Log Analytics 中的資料來源](../log-analytics/log-analytics-data-sources.md)中看到其說明。  這包括 Windows 事件記錄中的事件或 Syslog 所產生的事件，以及 Windows 和 Linux 用戶端的效能計數器。  您也可以從 Azure 監視器所收集的 Azure 資源來收集資料。  

如果您需要的資料無法透過任何可用的資料來源取得，可以使用 [HTTP 資料收集器 API](../log-analytics/log-analytics-data-collector-api.md)，這個 API 可讓您從任何可以呼叫 REST API 的用戶端，將資料寫入 Log Analytics 存放庫。  若要在管理解決方案中自訂資料收集，最常見的方法是建立 [Azure 自動化中的 Runbook](../automation/automation-runbook-types.md)，以從 Azure 或外部資源收集所需的資料，並使用資料收集器 API 將資料寫入存放庫中。  

### <a name="log-searches"></a>記錄檔搜尋
[記錄搜尋](../log-analytics/log-analytics-log-searches.md)可用來擷取和分析 Log Analytics 存放庫中的資料。  除了讓使用者能夠對存放庫中的資料執行特定分析外，記錄搜尋也可供檢視和警示使用。  

您應該定義您認為會對使用者有幫助的任何查詢，即使沒有任何檢視或警示使用這些查詢也是如此。  在入口網站中，這些查詢會以「已儲存搜尋」的形式提供給使用者來使用，而您也可以將這些查詢包含在自訂檢視中的[查詢清單視覺效果部分](../log-analytics/log-analytics-view-designer-parts.md#list-of-queries-part)。

### <a name="alerts"></a>警示
[Log Analytics 中的警示](../log-analytics/log-analytics-alerts.md)可透過對存放庫中的資料進行[記錄搜尋](#log-searches)來識別問題。  它們會通知使用者，或自動執行回應中的動作。 您應該識別應用程式的不同警示條件，並在解決方案檔中包含對應的警示規則。

如果問題有可能透過自動化程序加以修正，您通常會在 Azure 自動化中建立 Runbook 以執行此修復。  大部分 Azure 服務都可使用 [Cmdlet](/powershell/azure/overview) 來管理，Runbook 會運用這些 Cmdlet 來執行這類功能。

如果您的解決方案需要外部功能以對警示做出回應，您可以使用 [Webhook 回應](../log-analytics/log-analytics-alerts-actions.md)。  這可讓您呼叫外部 Web 服務，以從警示傳送資訊。

### <a name="views"></a>Views
Log Analytics 中的檢視可以視覺方式呈現 Log Analytics 儲存機制中的資料。  每個解決方案通常會有一個檢視，這個檢視具有[圖格](../log-analytics/log-analytics-view-designer-tiles.md)，並且會顯示在使用者的主要儀表板上。  檢視可以包含任意數目的[視覺效果部分](../log-analytics/log-analytics-view-designer-parts.md)，以便為使用者提供不同的視覺效果來呈現收集到的資料。

您可以[使用檢視設計工具來建立自訂檢視](../log-analytics/log-analytics-view-designer.md)，並於稍後匯出以納入解決方案檔中。  


## <a name="create-solution-file"></a>建立解決方案檔
在設定並測試要成為解決方案一部分的元件後，您可以[建立解決方案檔]( monitoring-solutions-solution-file.md)。  您會將解決方案元件實作在 [Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)中，此範本所包含的[解決方案資源]( monitoring-solutions-solution-file.md#solution-resource)會與檔案中的其他資源有所關聯。  


## <a name="test-your-solution"></a>測試解決方案
在開發解決方案的同時，您必須將其安裝在工作區中並進行測試。  您可以使用任何可用來[測試和安裝 Resource Manager 範本](../azure-resource-manager/resource-group-template-deploy.md)的方法來進行此操作。

## <a name="publish-your-solution"></a>發佈您的解決方案
解決方案完成並通過測試後，您可以透過下列來源將它提供給客戶。

- **Azure 快速入門範本**。  [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/)是一組由社群成員透過 GitHub 所貢獻的 Resource Manager 範本。  您可以藉由遵循[參與指南](https://github.com/Azure/azure-quickstart-templates/tree/master/1-CONTRIBUTION-GUIDE)中的資訊，來提供您的解決方案。
- **Azure Marketplace**。  [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) 可讓您將解決方案散發和販售給其他開發人員、ISV 和 IT 專業人員。  您可以在[如何在 Azure Marketplace 中發佈和管理供應項目](../marketplace/marketplace-publishers-guide.md)中，了解如何將解決方案發佈至 Azure Marketplace。



## <a name="next-steps"></a>後續步驟
* 了解如何為您的管理解決方案[建立解決方案檔]( monitoring-solutions-solution-file.md)。
* 了解[編寫 Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)的詳細資料。
* 搜尋 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates)不同 Resource Manager 範本的範例。