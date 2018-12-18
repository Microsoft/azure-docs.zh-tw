---
title: 將管理事件的 Azure 警示移轉至活動記錄警示
description: 管理事件的警示將於 10 月 1 日移除。 藉由移轉現有警示來做準備。
author: johnkemnetz
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 08/14/2017
ms.author: johnkem
ms.component: alerts
ms.openlocfilehash: 9e4302b780d0c08afbc791a0aec6bfd806aba161
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35263699"
---
# <a name="migrate-azure-alerts-on-management-events-to-activity-log-alerts"></a>將管理事件的 Azure 警示移轉至活動記錄警示


> [!WARNING]
> 管理事件的警示將於 10 月 1 日或之後關閉。 使用下列指示來了解您是否有這些警示並加以移轉 (如果有的話)。
>
> 

## <a name="what-is-changing"></a>變更內容

Azure 監視器 (先前稱為 Azure Insights) 提供了建立警示的功能，而警示會觸發管理事件並產生通知以傳送至 webhook URL 或電子郵件地址。 您已使用任何一種方法建立下列其中一種警示：
* 在特定資源類型的 Azure 入口網站中，在 [監視] -> [警示]-> [新增警示] 之下，其中 [警示對象] 設定為 [事件]
* 執行 Add-AzureRmLogAlertRule PowerShell Cmdlet
* 直接使用[警示 REST API](http://docs.microsoft.com/rest/api/monitor/alertrules)，其 odata.type = “ManagementEventRuleCondition” 和 dataSource.odata.type = “RuleManagementEventDataSource”
 
下列 PowerShell 指令碼會傳回您的訂用帳戶中管理事件的所有警示清單，以及每個警示上設定的條件。

```powershell
Connect-AzureRmAccount
$alerts = $null
foreach ($rg in Get-AzureRmResourceGroup ) {
  $alerts += Get-AzureRmAlertRule -ResourceGroup $rg.ResourceGroupName
}
foreach ($alert in $alerts) {
  if($alert.Properties.Condition.DataSource.GetType().Name.Equals("RuleManagementEventDataSource")) {
    "Alert Name: " + $alert.Name
    "Alert Resource ID: " + $alert.Id
    "Alert conditions:"
    $alert.Properties.Condition.DataSource
    "---------------------------------"
  }
} 
```

如果您沒有管理事件的警示，上述 PowerShell Cmdlet 會輸出一系列的警告訊息，如下所示：

`WARNING: The output of this cmdlet will be flattened, i.e. elimination of the properties field, in a future release to improve the user experience.`

您可以忽略這些警告訊息。 如果您有管理事件的警示，此 PowerShell Cmdlet 的輸出如下所示：

```
Alert Name: webhookEvent1
Alert Resource ID: /subscriptions/<subscription-id>/resourceGroups/<resourcegroup-name>/providers/microsoft.insights/alertrules/webhookEvent1
Alert conditions:

EventName            : 
EventSource          : 
Level                : 
OperationName        : microsoft.web/sites/start/action
ResourceGroupName    : 
ResourceProviderName : 
Status               : succeeded
SubStatus            : 
Claims               : Microsoft.Azure.Management.Monitor.Management.Models.RuleManagementEventClaimsDataSource
ResourceUri          : /subscriptions/<subscription-id>/resourceGroups/<resourcegroup-name>/providers/Microsoft.Web/sites/samplealertapp

---------------------------------
Alert Name: someclilogalert
Alert Resource ID: /subscriptions/<subscription-id>/resourceGroups/<resourcegroup-name>/providers/microsoft.insights/alertrules/someclilogalert
Alert conditions:

EventName            : 
EventSource          : 
Level                : 
OperationName        : Start
ResourceGroupName    : 
ResourceProviderName : 
Status               : 
SubStatus            : 
Claims               : Microsoft.Azure.Management.Monitor.Management.Models.RuleManagementEventClaimsDataSource
ResourceUri          : /subscriptions/<subscription-id>/resourceGroups/<resourcegroup-name>/providers/Microsoft.Compute/virtualMachines/Seaofclouds

---------------------------------
```

每個警示會以虛線分隔，其詳細資料包括警示的資源識別碼和受監視的特定規則。

這項功能已轉換為 [Azure 監視器活動記錄警示](monitoring-activity-log-alerts.md)。 這些新警示可讓您在活動記錄事件上設定條件，並且在新事件符合條件時收到通知。 管理事件的警示也有好幾項改善功能：
* 您可以使用[動作群組](monitoring-action-groups.md)，將您的通知收件者 (「動作」) 群組重複使用於許多警示，以降低變更誰應收到通知的複雜度。
* 您可以使用簡訊搭配動作群組，直接在電話上接收通知。
* 您可以[使用 Resource Manager 範本建立活動記錄警示](monitoring-create-activity-log-alerts-with-resource-manager-template.md)。
* 您可以建立具有更大彈性和複雜性的條件，以符合您的特定需求。
* 更快速地傳遞通知。
 
## <a name="how-to-migrate"></a>如何移轉
 
若要建立新的活動記錄警示，您可以：
* 遵循[有關如何在 Azure 入口網站中建立警示的指南](monitoring-activity-log-alerts.md)
* 了解[如何使用 Resource Manager 範本建立警示](monitoring-create-activity-log-alerts-with-resource-manager-template.md)
 
您先前建立之管理事件的警示，將不會自動移轉至活動記錄警示。 您需要使用上述 PowerShell 指令碼，列出您目前已設定之管理事件的警示，並以手動方式將其重新建立為活動記錄警示。 這必須在 10 月 1 日之前完成，之後您的 Azure 訂用帳戶將不再顯示管理事件的警示。 其他類型的 Azure 警示，包括 Azure 監視器計量警示、Application Insights 警示，以及不受此變更影響的 Log Analytics 警示。 如果您有任何問題，請張貼於下列的註解中。


## <a name="next-steps"></a>後續步驟

* 深入了解[活動記錄](monitoring-overview-activity-logs.md)
* [透過 Azure 入口網站設定活動記錄警示](monitoring-activity-log-alerts.md)
* [透過 Resource Manager 設定活動記錄警示](monitoring-create-activity-log-alerts-with-resource-manager-template.md)
* 檢閱[活動記錄警示 webhook 結構描述](monitoring-activity-log-alerts-webhook.md)
* 深入了解[服務通知](monitoring-service-notifications.md)
* 深入了解[動作群組](monitoring-action-groups.md)
