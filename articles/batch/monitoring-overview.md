---
title: 監視 Azure Batch | Microsoft Docs
description: 了解 Azure 監視服務、計量、診斷記錄，以及 Azure Batch 的其他監視功能。
services: batch
author: dlepow
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.workload: na
ms.date: 04/05/2018
ms.author: danlep
ms.openlocfilehash: 885deca5efbd72b3d641c8c94837851340d752b9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957824"
---
# <a name="monitor-batch-solutions"></a>監視 Batch 解決方案

Azure 與 Batch 服務提供一系列服務、工具和 API 來監視您的 Batch 解決方案。 此概觀文件可協助您選擇符合您需求的監視方法。

如需可用來監視 Azure 資源的 Azure 元件和服務概觀，請參閱[監視 Azure 應用程式和資源](../monitoring-and-diagnostics/monitoring-overview.md)。

## <a name="subscription-level-monitoring"></a>訂用帳戶層級監視

在包含 Batch 帳戶的訂用帳戶層級上，[Azure 活動記錄](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)會收集[數種類別](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md#categories-in-the-activity-log)中的操作事件資料。

具體針對 Batch 帳戶來說，活動記錄會收集關於帳戶建立和刪除及金鑰管理的事件。

從活動記錄擷取事件的方法之一是使用 Azure 入口網站。 按一下 [所有服務] > [活動記錄]。 或是，使用 Azure CLI、PowerShell Cmdlet 或 Azure Monitor REST API 來查詢事件。 您也可以匯出活動記錄，或設定[活動記錄警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts-new-experience.md)。

## <a name="batch-account-level-monitoring"></a>Batch 帳戶層級監視

使用 [Azure 監視器](../azure-monitor/overview.md)的功能監視每個 Batch 帳戶。 Azure 監視器會收集 Batch 帳戶層級範圍內的資源[計量](../monitoring/monitoring-data-collection.md#metrics)和[診斷記錄 (選擇性)](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)，這些資源包括集區、作業和工作等。 以手動方式或程式設計方式收集和使用此資料，以監視您 Batch 帳戶中的活動並診斷問題。 如需詳細資料，請參閱[用於診斷評估和監視的 Batch 計量、警示和記錄](batch-diagnostics.md)。
 
> [!NOTE]
> 您的 Batch 帳戶中已預設可使用計量功能，無須其他設定，而且計量功能具有 30 天的累積記錄。 您必須啟用 Batch 帳戶的診斷記錄，但您可能需要為儲存或處理診斷記錄資料支付額外費用。 

## <a name="batch-resource-monitoring"></a>Batch 資源監視

在 Batch 應用程式中，使用 Batch API 監視或查詢像是作業、工作、節點和集區等資源的狀態。 例如︰

* [依照狀態計算工作和計算節點](batch-get-resource-counts.md)
* [建立查詢以便有效率地列出 Batch 資源](batch-efficient-list-queries.md)
* [建立工作相依性](batch-task-dependencies.md)
* 使用[作業管理員工作](/rest/api/batchservice/job/add#jobmanagertask)
* 監視[工作狀態](/rest/api/batchservice/task/list#taskstate)
* 監視[節點狀態](/rest/api/batchservice/computenode/list#computenodestate)
* 監視[集區狀態](/rest/api/batchservice/pool/get#poolstate)
* 監視[帳戶中的集區使用情形](/rest/api/batchservice/pool/listusagemetrics)
* [計算各種狀態的集區節點](/rest/api/batchservice/account/listpoolnodecounts)

## <a name="vm-performance-counters-and-application-monitoring"></a>VM 效能計數器和應用程式監視

* [Application Insights](../application-insights/app-insights-overview.md) 是一種 Azure 服務，可用來以程式設計方式監視 Batch 作業和工作的可用性、效能和使用情形。 輕鬆從計算節點 (VM) 取得效能計數器和自訂 VM 工作的資訊。 

  如需範例，請參閱[使用 Application Insights 監視 Batch .NET 應用程式並進行偵錯](monitor-application-insights.md)，以及隨附的[程式碼範例](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ApplicationInsights)。

  > [!NOTE]
  > 使用 Application Insights 可能會產生額外費用。 請參閱[定價選項](https://azure.microsoft.com/pricing/details/application-insights/)。 
  >

* [Batch Explorer](https://github.com/Azure/BatchExplorer)是免費、功能豐富、獨立用戶端的工具，可以協助建立、偵錯及監視 Azure Batch 應用程式。 下載適用於 Mac、Linux 或 Windows 的[安裝套件](https://azure.github.io/BatchExplorer/)。 (選擇性) 將您的 Batch 解決方案設定為[顯示 Application Insights 資料](https://github.com/Azure/batch-insights)，例如 Batch Explorer 中的 VM 效能計數器。


## <a name="next-steps"></a>後續步驟

* 了解可用來建置 Batch 解決方案的 [Batch API 和工具](batch-apis-tools.md)。
* 深入了解 Batch 的[診斷記錄](batch-diagnostics.md)。