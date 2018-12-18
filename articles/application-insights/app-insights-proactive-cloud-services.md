---
title: 使用 Azure 診斷與 Azure Application Insights 的整合對 Azure 雲端服務的問題發出警示 | Microsoft Docs
description: 使用 Azure Application Insights，監視 Azure 雲端服務中如啟動失敗、當機和角色回收迴圈等問題
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 06/07/2018
ms.reviewer: harelbr
ms.author: mbullwin
ms.openlocfilehash: cdb395a590fb200a24c68e56728a270b968e53b5
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633026"
---
# <a name="alert-on-issues-in-azure-cloud-services-using-the-azure-diagnostics-integration-with-azure-application-insights"></a>使用 Azure 診斷與 Azure Application Insights 的整合對 Azure 雲端服務的問題發出警示

在本文中，我們將描述如何設定警示規則，以監視 Azure 雲端服務 (Web 和背景工作角色) 中如啟動失敗、當機和角色回收迴圈等問題。

本文中所述的方法是根據 [Azure 診斷與 Application Insights 的整合](https://azure.microsoft.com/blog/azure-diagnostics-integration-with-application-insights/)，以及最近發行的 [Application Insights 的記錄檔警示](https://azure.microsoft.com/blog/log-alerts-for-application-insights-preview/)功能。

## <a name="define-a-base-query"></a>定義基本查詢

開始進行之前，我們會定義一個基本查詢，以從 Windows Azure 通道擷取 Windows 事件記錄檔的事件，這些事件會當作追蹤記錄擷取到 Application Insights 中。
這些記錄可用於在 Azure 雲端服務中偵測各種問題，例如啟動失敗、執行階段失敗和回收迴圈。

> [!NOTE]
> 下列的基本查詢會在 30 分的時間範圍內檢查是否有問題，並假設內嵌遙測記錄的延遲時間為 10 分鐘。 您可以根據您的需要設定這些預設值。

```
let window = 30m;
let endTime = ago(10m);
let EventLogs = traces
| where timestamp > endTime - window and timestamp < endTime
| extend channel = tostring(customDimensions.Channel), eventId = tostring(customDimensions.EventId)
| where channel == 'Windows Azure' and isnotempty(eventId)
| where tostring(customDimensions.DeploymentName) !contains 'deployment' // discard records captured from local machines
| project timestamp, channel, eventId, message, cloud_RoleInstance, cloud_RoleName, itemCount;
```

## <a name="check-for-specific-event-ids"></a>檢查是否有特定的事件識別碼

擷取 Windows 事件記錄檔的事件之後，可以藉由檢查是否有其各自的事件識別碼和訊息屬性 (請參閱以下範例) 來偵測特定的問題。
只需合併上述基本查詢與以下其中一個查詢，並在定義記錄檔警示規則時使用該合併查詢。

> [!NOTE]
> 在下列範例中，如果在分析的時間範圍內找到三個以上的事件，將會偵測到問題。 您可以設定此預設值來變更警示規則的敏感度。

```
// Detect failures in the OnStart method
EventLogs
| where eventId == '2001'
| where message contains '.OnStart()'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect failures during runtime
EventLogs
| where eventId == '2001'
| where message contains '.Run()'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect failures when running a startup task
EventLogs
| where eventId == '1000'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect recycle loops
EventLogs
| where eventId == '1006'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

## <a name="create-an-alert"></a>建立警示

在 Application Insights 資源內的導覽功能表中，移至 [警示]，然後選取 [新增警示規則]。

![建立規則的螢幕擷取畫面](./media/app-insights-proactive-cloud-services/001.png)

在 [建立規則] 視窗的 [定義警示條件] 區段底下，按一下 [新增準則]，然後選取 [自訂記錄檔搜尋]。

![定義警示之條件準則的螢幕擷取畫面](./media/app-insights-proactive-cloud-services/002.png)

在 [搜尋查詢] 方塊中，貼入您在上一個步驟中備妥的合併查詢。

然後，繼續前往 [閾值] 方塊，並將其值設定為 0。 您可能會選擇調整 [期間] 和 [頻率] 欄位。
按一下 [完成] 。

![設定訊號邏輯查詢的螢幕擷取畫面](./media/app-insights-proactive-cloud-services/003.png)

在 [定義警示詳細資料] 區段底下，提供警示規則的 [名稱] 和 [描述]，並設定其 [嚴重性]。
另外，請確定 [在建立時啟用規則] 按鈕設定為 [是]。

![警示詳細資料的螢幕擷取畫面](./media/app-insights-proactive-cloud-services/004.png)

在 [定義動作群組] 區段底下，您可以選取現有的 [動作群組]，或建立一個新的動作群組。
您可以選擇讓動作群組包含各種類型的多個動作。

![螢幕擷取畫面的動作群組](./media/app-insights-proactive-cloud-services/005.png)

定義動作群組之後，請確認您的變更，然後按一下 [建立警示規則]。

## <a name="next-steps"></a>後續步驟

深入了解自動偵測：

[失敗異常](app-insights-proactive-failure-diagnostics.md)
[記憶體流失](app-insights-proactive-potential-memory-leak.md)
[效能異常](app-insights-proactive-performance-diagnostics.md)

