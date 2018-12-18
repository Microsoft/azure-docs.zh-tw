---
title: 監視公用 Basic Load Balancer 的作業、事件和計數器 | Microsoft Docs
description: 了解如何啟用 Basic Load Balancer 的警示事件和探查健康情況狀態記錄
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
tags: azure-resource-manager
ms.assetid: 56656d74-0241-4096-88c8-aa88515d676d
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/10/2018
ms.author: kumud
ms.openlocfilehash: 5c196a24d5cc9e4343762e55b5885c1264c2fcad
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42144254"
---
# <a name="log-analytics-for-public-basic-load-balancer"></a>適用於公用 Basic Load Balancer 的 Log Analytics

>[!IMPORTANT] 
>Azure Load Balancer 支援兩種不同類型：基本和標準。 本文討論基本 Load Balancer。 如需有關 Standard Load Balancer 的詳細資訊，請參閱 [Standard Load Balancer 概觀](load-balancer-standard-overview.md)，其中會透過 Azure 監視器中的多維度計量來公開遙測資料。

您可以在 Azure 中使用不同類型的記錄來管理 Basic Load Balancer，並對其進行疑難排解。 透過入口網站可以存取其中一些記錄檔。 從 Azure Blob 儲存體可以擷取所有記錄，並且在不同的工具中進行檢視 (例如 Excel 和 PowerBI)。 您可以從下列清單進一步了解不同類型的記錄檔。

* **稽核記錄檔︰** 您可以使用 [Azure 稽核記錄檔](../monitoring-and-diagnostics/insights-debugging-with-events.md) (之前稱為「作業記錄檔」) 來檢視提交至您 Azure 訂用帳戶的所有作業及其狀態。 預設會啟用稽核記錄檔，並可在 Azure 入口網站中進行檢視。
* **警示事件記錄：** 您可以使用此記錄來檢視負載平衡器所引發的警示。 系統每五分鐘會收集一次負載平衡器的狀態。 只有在引發負載平衡器警示事件時，才會寫入此記錄檔。
* **健康狀態探查記錄︰** 您可以使用此記錄來檢視健康狀態探查所偵測到的問題，例如後端集區中因為健康狀態探查失敗而未從負載平衡器接收要求的執行個體數目。 健康狀態探查狀態發生變更時會寫入此記錄。

> [!IMPORTANT]
> 記錄分析目前僅適用於公用的 Basic 負載平衡器。 記錄檔僅適用於在資源管理員部署模型中部署的資源。 您無法將記錄檔使用於傳統部署模型中的資源。 如需這些部署模型的詳細資訊，請參閱[了解 Resource Manager 部署和傳統部署](../azure-resource-manager/resource-manager-deployment-model.md)。

## <a name="enable-logging"></a>啟用記錄

每個 Resource Manager 資源都會自動啟用稽核記錄。 您需要啟用事件和健全狀況探查記錄，才能開始收集可透過這些記錄檔取得的資料。 使用下列步驟以啟用記錄功能。

登入 [Azure 入口網站](http://portal.azure.com)。 如果您還沒有負載平衡器，請先 [建立負載平衡器](load-balancer-get-started-internet-arm-ps.md) 再繼續。

1. 在入口網站中，按一下 [瀏覽]。
2. 選取 [負載平衡器]。

    ![入口網站 - 負載平衡器](./media/load-balancer-monitor-log/load-balancer-browse.png)

3. 選取現有的負載平衡器 >> [所有設定]。
4. 在對話方塊的右側，於負載平衡器的名稱下方，捲動至 [監視]，按一下 [診斷]。

    ![入口網站 - 負載平衡器 - 設定](./media/load-balancer-monitor-log/load-balancer-settings.png)

5. 在 [診斷] 窗格中，在 [狀態] 下方，選取 [開啟]。
6. 按一下 [儲存體帳戶]。
7. 在 [記錄] 下方，選取現有的儲存體帳戶或建立一個新的。 使用滑桿來決定值得在事件記錄中儲存多少天的事件資料。 
8. 按一下 [檔案] 。

診斷資料會儲存在指定儲存體帳戶中的表格儲存體中。 如果未儲存記錄，則是因為沒有相關的記錄產生。

![入口網站 - 診斷記錄檔](./media/load-balancer-monitor-log/load-balancer-diagnostics.png)

> [!NOTE]
> 稽核記錄檔不需要個別的儲存體帳戶。 將儲存體用於事件和健全狀況探查記錄將會產生服務費用。

## <a name="audit-log"></a>稽核記錄檔

預設會產生稽核記錄檔。 記錄檔會在 Azure 的 [事件記錄檔] 存放區中保留 90 天。 閱讀 [檢視事件和稽核記錄檔](../monitoring-and-diagnostics/insights-debugging-with-events.md) 一文，進一步了解這些記錄檔。

## <a name="alert-event-log"></a>警示事件記錄檔

您必須對每一個負載平衡器進行啟用，才會產生此記錄檔。 事件會以 JSON 格式記錄，並儲存在您啟用記錄時所指定的儲存體帳戶中。 以下是事件的範例。

```json
{
    "time": "2016-01-26T10:37:46.6024215Z",
    "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
    "category": "LoadBalancerAlertEvent",
    "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
    "operationName": "LoadBalancerProbeHealthStatus",
    "properties": {
        "eventName": "Resource Limits Hit",
        "eventDescription": "Ports exhausted",
        "eventProperties": {
            "public ip address": "40.117.227.32"
        }
    }
}
```

JSON 輸出中會顯示 *eventname* 屬性，此屬性會描述負載平衡器建立警示的原因。 在此案例中，來源 IP NAT (SNAT) 限制導致了 TCP 連接埠耗盡，因此產生此警示。

## <a name="health-probe-log"></a>健全狀況探查記錄檔

如果您已如上所述對每一個負載平衡器進行啟用，才會產生此記錄檔。 資料會儲存在您啟用記錄時所指定的儲存體帳戶中。 系統會建立名為 'insights-logs-loadbalancerprobehealthstatus' 的容器，並記錄下列資料：

```json
{
    "records":[
    {
        "time": "2016-01-26T10:37:46.6024215Z",
        "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
        "category": "LoadBalancerProbeHealthStatus",
        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
        "operationName": "LoadBalancerProbeHealthStatus",
        "properties": {
            "publicIpAddress": "40.83.190.158",
            "port": "81",
            "totalDipCount": 2,
            "dipDownCount": 1,
            "healthPercentage": 50.000000
        }
    },
    {
        "time": "2016-01-26T10:37:46.6024215Z",
        "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
        "category": "LoadBalancerProbeHealthStatus",
        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
        "operationName": "LoadBalancerProbeHealthStatus",
        "properties": {
            "publicIpAddress": "40.83.190.158",
            "port": "81",
            "totalDipCount": 2,
            "dipDownCount": 0,
            "healthPercentage": 100.000000
        }
    }]
}
```

JSON 輸出在屬性欄位中顯示了探查健全狀況狀態的基本資訊。 *dipDownCount* 屬性會顯示後端上因為探查回應失敗而不會接收網路流量的執行個體總數。

## <a name="view-and-analyze-the-audit-log"></a>檢視和分析稽核記錄檔

您可以使用下列任何方法，檢視和分析稽核記錄檔資料：

* **Azure 工具︰** 透過 Azure PowerShell、Azure 命令列介面 (CLI)、Azure REST API 或 Azure Preview 入口網站，從稽核記錄擷取資訊。 [稽核作業與資源管理員](../azure-resource-manager/resource-group-audit.md) 一文會詳述每個方法的逐步指示。
* **Power BI︰** 如果您還沒有 [Power BI](https://powerbi.microsoft.com/pricing) 帳戶，您可以免費試用。 使用 [Power BI 的 Azure 稽核記錄檔內容套件](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-audit-logs)，您可以使用預先設定的儀表板來分析資料，或根據您的需求自訂檢視。

## <a name="view-and-analyze-the-health-probe-and-event-log"></a>檢視和分析健全狀況探查與事件記錄檔

您需要連接到儲存體帳戶並擷取事件和健全狀況探查記錄檔的 JSON 記錄項目。 下載 JSON 檔案後，您可以將它們轉換成 CSV 並在 Excel、PowerBI 或任何其他資料視覺化工具中檢視。

> [!TIP]
> 如果您熟悉 Visual Studio 以及在 C# 中變更常數和變數值的基本概念，您可以使用 GitHub 所提供的[記錄檔轉換器工具 (英文)](https://github.com/Azure-Samples/networking-dotnet-log-converter)。

## <a name="additional-resources"></a>其他資源

* [使用 Power BI 視覺化您的 Azure 稽核記錄檔](http://blogs.msdn.com/b/powerbi/archive/2015/09/30/monitor-azure-audit-logs-with-power-bi.aspx) 部落格文章。
* [在 Power BI 和其他工具中檢視和分析 Azure 稽核記錄](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/) 部落格文章。

## <a name="next-steps"></a>後續步驟

[了解負載平衡器偵查](load-balancer-custom-probe-overview.md)
