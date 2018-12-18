---
title: Azure Batch 的計量、警示和診斷記錄 | Microsoft Docs
description: 記錄並分析 Azure Batch 帳戶資源 (如集區和工作) 的診斷記錄事件。
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: batch
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: big-compute
ms.date: 04/05/2018
ms.author: danlep
ms.custom: ''
ms.openlocfilehash: 98bd088d4995bd1444b059531dc5b19d0283f551
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965519"
---
# <a name="batch-metrics-alerts-and-logs-for-diagnostic-evaluation-and-monitoring"></a>用於診斷評估和監視的 Batch 計量、警示和記錄

 
本文說明如何使用 [Azure 監視器](../azure-monitor/overview.md)的功能來監視 Batch 帳戶。 Azure 監視器會收集您 Batch 帳戶中的資源[計量](../monitoring/monitoring-data-collection.md#metrics)和[診斷記錄](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)。 透過各種方式收集及使用此資料，以監視您的 Batch 帳戶和診斷問題。 您也可以設定[計量警示](../monitoring-and-diagnostics/monitoring-overview-alerts.md)，在計量達到指定值時接收通知。 

## <a name="batch-metrics"></a>Batch 計量

計量是指 Azure 遙測資料 (也稱為效能計數器)，由 Azure 監視器服務取用的 Azure 資源所發出。 Batch 帳戶中的範例計量包括：集區建立事件、低優先順序的節點計數及工作完成事件。 

請參閱[支援的 Batch 計量清單](../monitoring-and-diagnostics/monitoring-supported-metrics.md#microsoftbatchbatchaccounts)。

計量：

* 在每個 Batch 帳戶中預設啟用，不需要額外設定
* 每隔 1 分鐘產生
* 不會自動保存，但是有 30 天的累積記錄。 您可以保存活動計量以作為[診斷記錄](#work-with-diagnostic-logs)的一部分。

### <a name="view-metrics"></a>檢視計量

在 Azure 入口網站中檢視 Batch 帳戶的計量。 帳戶的**概觀**頁面會預設顯示索引鍵節點、核心和工作計量。 

檢視所有 Batch 帳戶計量： 

1. 在入口網站中，按一下 [所有服務] > [Batch 帳戶]，然後按一下您的 Batch 帳戶名稱。
2. 在 [監視] 下，按一下 [計量]。
3. 選取一或多個計量。 如有需要，可使用 [訂閱]、[資源群組]、[資源類型]和 [資源] 下拉式清單，來選取其他資源計量。

    ![Batch 計量](media/batch-diagnostics/metrics-portal.png)

若要以程式設計方式擷取計量，請使用 Azure 監視器 API。 例如，請參閱[使用 .NET 擷取 Azure 監視器計量](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/)。

## <a name="batch-metric-alerts"></a>Batch 計量警示

(選擇性) 設定幾近即時的「計量警示」，當指定的計量值超出您指派的閾值時，就會觸發此警示。 當警示為「已啟動」時 (超出閾值且符合警示條件時)，以及當警示為「已解決」時 (再次超出閾值且不再符合條件時)，您選擇的警示會產生[通知](../monitoring-and-diagnostics/insights-alerts-portal.md)。 

例如，您可以在低優先順序核心計數降至特定層級時設定計量警示，以便您調整集區的構成要素。

在入口網站中設定計量警示：

1. 按一下 [所有服務] > [Batch 帳戶]，然後按一下您的 Batch 帳戶名稱。
2. 在 [監視] 下方，按一下 [警示規則] > [新增計量警示]。
3. 選取計量、警示條件 (例如計量在某個期間超過特定值)，以及一個或多個通知。

您也可以使用 [REST API](https://docs.microsoft.com/rest/api/monitor/) 設定幾近即時的警示。 如需詳細資訊，請參閱[警示概觀](../monitoring-and-diagnostics/monitoring-overview-alerts.md)

## <a name="batch-diagnostics"></a>Batch 診斷

診斷記錄包含 Azure 資源發出的資訊，其會描述每個資源的作業。 針對 Batch，您可以收集下列記錄：

* 在個別 Batch 資源 (如集區或工作) 存留期間，由 Azure Batch 服務發出的**服務記錄**事件。 

* 帳戶層級上的**計量**記錄。 

啟用診斷記錄收集的設定並不會預設為啟用。 請為每個您想要監視的 Batch 帳戶明確啟用診斷設定。

### <a name="log-destinations"></a>記錄目的地

常見案例是選取 Azure 儲存體帳戶作為記錄目的地。 若要在 Azure 儲存體中儲存記錄，請在啟用記錄收集之前建立帳戶。 如果您有與 Batch 帳戶產生關聯的儲存體帳戶，您可以選擇該帳戶作為記錄目的地。 

診斷記錄目的地的其他選項：

* 將 Batch 診斷記錄事件串流至 [Azure 事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)。 事件中樞每秒可輸入數百萬個事件，您可以使用任何即時分析提供者來轉換和儲存。 

* 將診斷記錄傳送至 [Azure Log Analytics](../log-analytics/log-analytics-overview.md)，您可以在 Operations Management Suite (OMS) 入口網站中分析它們，或是匯出它們以在 Power BI 或 Excel 中分析。

> [!NOTE]
> 使用 Azure 服務儲存或處理診斷記錄資料可能會產生額外費用。 
>

### <a name="enable-collection-of-batch-diagnostic-logs"></a>啟用 Batch 診斷記錄的收集

1. 在入口網站中，按一下 [所有服務] > [Batch 帳戶]，然後按一下您的 Batch 帳戶名稱。
2. 在 [監視] 下方，按一下 [診斷記錄] > [開啟診斷]。
3. 在 [診斷設定] 中輸入設定的名稱，然後選擇記錄目的地 (現有儲存體帳戶、事件中樞或 Log Analytics)。 選取 **ServiceLog** 或/和 **AllMetrics**。

    當您選取儲存體帳戶時，可選擇性地設定保留原則。 如果您未指定保留天數，資料的保留期限會是儲存體帳戶的存留期間。

4. 按一下 [檔案] 。

    ![Batch 診斷](media/batch-diagnostics/diagnostics-portal.png)

啟用記錄收集的其他選項包括：使用入口網站中的 Azure 監視器設定診斷設定、使用 [Resource Manager 範本](../monitoring-and-diagnostics/monitoring-enable-diagnostic-logs-using-template.md)或使用 Azure PowerShell 或 Azure CLI。 請參閱[收集並取用來自 Azure 資源的記錄資料](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md#how-to-enable-collection-of-diagnostic-logs)。


### <a name="access-diagnostics-logs-in-storage"></a>存取儲存體中的診斷記錄

如果您將 Batch 診斷記錄封存在儲存體帳戶中，當發生相關事件時，儲存體帳戶中就會建立儲存體容器。 Blob 會根據下列命名模式建立：

```
insights-{log category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/
RESOURCEGROUPS/{resource group name}/PROVIDERS/MICROSOFT.BATCH/
BATCHACCOUNTS/{batch account name}/y={four-digit numeric year}/
m={two-digit numeric month}/d={two-digit numeric day}/
h={two-digit 24-hour clock hour}/m=00/PT1H.json
```
範例：

```
insights-metrics-pt1m/resourceId=/SUBSCRIPTIONS/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/
RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.BATCH/
BATCHACCOUNTS/MYBATCHACCOUNT/y=2018/m=03/d=05/h=22/m=00/PT1H.json
```
每個 PT1H.json blob 檔案都會包含 JSON 格式的事件，這是在 Blob URL 指定時數內 (例如 h = 12) 發生的事件。 在目前這一小時，事件一發生就會附加到 PT1H.json 檔案。 分鐘值 (m = 00) 一定是 00，因為診斷記錄事件是分成每小時的個別 blob。 (所有時間都是採用 UTC 格式。)


若要深入了解儲存體帳戶中的診斷記錄結構描述，請參閱[封存 Azure 診斷記錄](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md#schema-of-diagnostic-logs-in-the-storage-account)。

若要以程式設計方式存取您儲存體帳戶中的記錄，請使用儲存體 API。 

### <a name="service-log-events"></a>服務記錄檔事件
已收集的 Azure Batch 服務記錄中包含在個別 Batch 資源 (如集區或工作) 存留期間，由 Azure Batch 服務發出的事件。 Batch 發出的每個事件都會以 JSON 格式記錄。 例如，**集區建立事件**範例的主體為：

```json
{
    "poolId": "myPool1",
    "displayName": "Production Pool",
    "vmSize": "Small",
    "cloudServiceConfiguration": {
        "osFamily": "5",
        "targetOsVersion": "*"
    },
    "networkConfiguration": {
        "subnetId": " "
    },
    "resizeTimeout": "300000",
    "targetDedicatedComputeNodes": 2,
    "maxTasksPerNode": 1,
    "vmFillType": "Spread",
    "enableAutoscale": false,
    "enableInterNodeCommunication": false,
    "isAutoPool": false
}
```

Batch 服務目前會發出下列服務記錄檔事件。 這份清單可能不完整，因為自上次更新此文件後可能已新增額外的事件。

| **服務記錄檔事件** |
| --- |
| [建立集區](batch-pool-create-event.md) |
| [開始刪除集區](batch-pool-delete-start-event.md) |
| [完成集區刪除](batch-pool-delete-complete-event.md) |
| [開始調整集區大小](batch-pool-resize-start-event.md) |
| [完成集區大小調整](batch-pool-resize-complete-event.md) |
| [開始工作](batch-task-start-event.md) |
| [完成工作](batch-task-complete-event.md) |
| [工作失敗](batch-task-fail-event.md) |



## <a name="next-steps"></a>後續步驟

* 了解可用來建置 Batch 解決方案的 [Batch API 和工具](batch-apis-tools.md)。
* 深入了解[監視 Batch 解決方案](monitoring-overview.md)。
