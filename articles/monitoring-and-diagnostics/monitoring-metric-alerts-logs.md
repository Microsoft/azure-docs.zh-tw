---
title: 為 Azure 監視器中的記錄建立計量警示
description: 針對常用記錄分析資料建立近乎即時之計量警示的教學課程。
author: msvijayn
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/17/2018
ms.author: vinagara
ms.component: alerts
ms.openlocfilehash: bed0c15504323aba4ebf680273870720a8ff833a
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388292"
---
# <a name="create-metric-alerts-for-logs-in-azure-monitor"></a>為 Azure 監視器中的記錄建立計量警示  

## <a name="overview"></a>概觀
Azure 監視器支援的[計量警示類型](monitoring-near-real-time-metric-alerts.md)具有優於[傳統警示](insights-alerts-portal.md)的優點。 計量適用於 [Azure 服務的大型清單](monitoring-supported-metrics.md)。 此文章說明 `Microsoft.OperationalInsights/workspaces` 資源子集的使用量。 

您可以將計量警示使用在「來自記錄的計量」程序所擷取作為計量的常用 Log Analytics 記錄上，包括 Azure 或內部部署中的資源。 以下列出支援的 Log Analytics 解決方案：
- Windows 和 Linux 電腦的[效能計數器](../log-analytics/log-analytics-data-sources-performance-counters.md)
- [代理程式健全狀況的活動訊號記錄](../operations-management-suite/oms-solution-agenthealth.md)
- [更新管理](../operations-management-suite/oms-solution-update-management.md)記錄
- [事件資料](../log-analytics/log-analytics-data-sources-windows-events.md)記錄
 
在 Azure 中對以查詢為基礎的[記錄警示](alert-log.md)使用**記錄的計量警示**有許多優點；以下列出一部分優點：
- 計量警示提供近乎即時的監視功能，以及來自記錄來源的記錄計量警示分支資料，以確保都相同
- 計量警示具狀態，只會在引發警示和解決警示時各通知一次；和記錄警示相反，其為無狀態，而且在每個間隔只要警示條件符合就會引發
- 記錄的計量警示提供多個維度，允許篩選較簡單且不需要在分析中釘選查詢的特定值，例如電腦、作業系統類型等

> [!NOTE]
> 只有在所選期間中有特定計量和/或維度的資料時，才會顯示特定計量和/或維度。 這些計量可供有 Azure Log Analytics 工作區的客戶使用。

## <a name="metrics-and-dimensions-supported-for-logs"></a>記錄支援的計量和維度
 計量警示支援針對使用維度的計量發出警示。 您可以使用維度來將計量篩選到正確層級。 跨支援的解決方案列出來自 [Log Analytics 工作區](monitoring-supported-metrics.md#microsoftoperationalinsightsworkspaces)的記錄所支援計量的完整清單。

> [!NOTE]
> 若要檢視透過 [Azure 監視器 - 計量](monitoring-metric-charts.md)從 Log Analytics 工作區擷取的支援計量，則必須為所述計量建立記錄的計量警示。 在記錄的計量警示中選擇的維度，只會透過 [Azure 監視器 - 計量] 顯示以供探索。

# <a name="creating-metric-alert-for-log-analytics"></a>建立 Log Analytics 的計量警示
來自常用記錄的計量資料會先輸送至 [Azure 監視器 - 計量]，再於 Log Analytics 中處理。 這可讓使用者運用計量平台以及計量警示的功能，包括最低頻率為 1 分鐘的警示。 以下列出記錄的計量警示製作方法。

## <a name="prerequisites-for-metric-alert-for-logs"></a>記錄的計量警示必要條件
在針對 Log Analytics 資料所收集記錄的計量發揮作用之前，必須先設定並提供下列各項：
1. **使用中的 Log Analytics 工作區**：有效且使用中的 Log Analytics 工作區必須存在。 如需詳細資訊，請參閱[在 Azure 入口網站中建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)。
2. **為 Log Analytics 工作區設定代理程式**：需要為 Azure VM (和/或) 內部部署 VM 設定代理程式，才能將資料傳送至先前步驟中使用的 Log Analytics 工作區。 如需詳細資訊，請參閱 [Log Analytics - 代理程式概觀](../monitoring/monitoring-overview-azure-agents.md)。
3. **已安裝支援的 Log Analytics 解決方案**：應設定 Log Analytics 解決方案並將資料傳送至 Log Analytics 工作區 - 支援的解決方案為 [Windows 和 Linux 效能計數器](../log-analytics/log-analytics-data-sources-performance-counters.md)、[代理程式健康情況的活動訊號記錄](../operations-management-suite/oms-solution-agenthealth.md)、更新管理以及[事件資料](../log-analytics/log-analytics-data-sources-windows-events.md)。
4. **設定 Log Analytics 解決方案以傳送記錄**：Log Analytis 解決方案應啟用對應至 [Log Analytics 工作區所支援計量](monitoring-supported-metrics.md#microsoftoperationalinsightsworkspaces)的必要記錄/資料。 例如，您必須先在[效能計數器](../log-analytics/log-analytics-data-sources-performance-counters.md)解決方案中設定其 *% Available Memory* 計數器。

## <a name="configuring-metric-alert-for-logs"></a>設定記錄的計量警示
 您可以使用 Azure 入口網站、Resource Manager 範本、REST API、PowerShell 與 Azure CLI 來建立及管理計量警示。 由於記錄的計量警示是計量警示變體，所以完成必要條件之後，可為指定的 Log Analytics 工作區建立記錄的計量警示。 [計量警示](monitoring-near-real-time-metric-alerts.md)的所有特性與功能同樣都適用於記錄的計量警示；包括承載結構描述、適用配額限制及計費價格。

如需逐步詳細資料與範例，請參閱[建立及管理計量警示](https://aka.ms/createmetricalert)。 具體對記錄的計量警示而言，請依照指示來管理計量警示，並確定下列各項：
- 計量警示的目標是有效的「Log Analytics 工作區」
- 為所選取「Log Analytics 工作區」的計量警示選擇的訊號是 [計量] 類型
- 使用維度篩選篩選特定條件或資源；記錄的計量是多維度的
- 當設定「訊號邏輯」時，可建立訊號警示，以延伸維度 (像是電腦) 的多個值
- 如果**未**使用 Azure 入口網站來為選取的*Log Analytics 工作區*建立計量警示，則使用者必須使用 [Azure 監視器 - 排程的查詢規則](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules
) \(英文\)，先建立將記錄資料轉換為計量的明確規則。

> [!NOTE]
> 透過 Azure 入口網站為 Log Analytics 工作區建立計量警示時，透過 [Azure 監視器 - 排程的查詢規則](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules) \(英文\) 將記錄資料轉換為計量的對應規則會在背景自動建立，完全不需要使用者介入或採取行動。 如需使用 Azure 入口網站以外的方法所建立記錄的計量警示，請參閱[記錄的計量警示所適用資源範本](#resource-template-for-metric-alerts-for-logs)一節中，在建立計量警示之前，對計量轉換規則建立 ScheduledQueryRule 記錄的示範方法，否則對記錄建立的計量警示不會有任何資料。

## <a name="resource-template-for-metric-alerts-for-logs"></a>記錄的計量警示所適用資源範本
如稍早所述，從記錄建立計量警示的程序有二：
1. 使用 scheduledQueryRule API 建立從支援的記錄擷取計量的規則
2. 針對從記錄 (步驟 1) 與 Log Analytics 工作區擷取作為目標資源的計量，建立計量警示

若要達到一模一樣，您可以使用下面的範例 Azure Resource Manager 範本，其建立的計量警示取決於透過 scheduledQueryRule 從記錄擷取計量的規則建立成功與否。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the rule to convert log to metric"
            }
        },
        "convertRuleDescription": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Description for log converted to metric"
            }
        },
        "convertRuleRegion": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the region used by workspace"
            }
        },
        "convertRuleStatus": {
            "type": "string",
            "defaultValue": "true",
            "metadata": {
                "description": "Specifies whether the log convertion rule is enabled"
            }
        },
        "convertRuleMetric": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric once extraction done from logs."
            }
        },
        "alertName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the alert"
            }
        },
        "alertDescription": {
            "type": "string",
            "defaultValue": "This is a metric alert",
            "metadata": {
                "description": "Description of alert"
            }
        },
        "alertSeverity": {
            "type": "int",
            "defaultValue": 3,
            "allowedValues": [
                0,
                1,
                2,
                3,
                4
            ],
            "metadata": {
                "description": "Severity of alert {0,1,2,3,4}"
            }
        },
        "isEnabled": {
            "type": "bool",
            "defaultValue": true,
            "metadata": {
                "description": "Specifies whether the alert is enabled"
            }
        },
        "resourceId": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Full Resource ID of the resource emitting the metric that will be used for the comparison. For example /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroups/ResourceGroupName/providers/Microsoft.compute/virtualMachines/VM_xyz"
            }
        },
        "metricName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric used in the comparison to activate the alert."
            }
        },
        "operator": {
            "type": "string",
            "defaultValue": "GreaterThan",
            "allowedValues": [
                "Equals",
                "NotEquals",
                "GreaterThan",
                "GreaterThanOrEqual",
                "LessThan",
                "LessThanOrEqual"
            ],
            "metadata": {
                "description": "Operator comparing the current value with the threshold value."
            }
        },
        "threshold": {
            "type": "string",
            "defaultValue": "0",
            "metadata": {
                "description": "The threshold value at which the alert is activated."
            }
        },
        "timeAggregation": {
            "type": "string",
            "defaultValue": "Average",
            "allowedValues": [
                "Average",
                "Minimum",
                "Maximum",
                "Total"
            ],
            "metadata": {
                "description": "How the data that is collected should be combined over time."
            }
        },
        "windowSize": {
            "type": "string",
            "defaultValue": "PT5M",
            "metadata": {
                "description": "Period of time used to monitor alert activity based on the threshold. Must be between five minutes and one day. ISO 8601 duration format."
            }
        },
        "evaluationFrequency": {
            "type": "string",
            "defaultValue": "PT1M",
            "metadata": {
                "description": "how often the metric alert is evaluated represented in ISO 8601 duration format"
            }
        },
        "actionGroupId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The ID of the action group that is triggered when the alert is activated or deactivated"
            }
        }
    },
    "variables": {
        "convertRuleTag": "hidden-link:/subscriptions/1234-56789-1234-567a/resourceGroups/resouceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName",
        "convertRuleSourceWorkspace": {
            "SourceId": "/subscriptions/1234-56789-1234-567a/resourceGroups/resouceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        }
    },
    "resources": [
        {
            "name": "[parameters('convertRuleName')]",
            "type": "Microsoft.Insights/scheduledQueryRules",
            "apiVersion": "2018-04-16",
            "location": "[parameters('convertRuleRegion')]",
            "tags": {
                "[variables('convertRuleTag')]": "Resource"
            },
            "properties": {
                "description": "[parameters('convertRuleDescription')]",
                "enabled": "[parameters('convertRuleStatus')]",
                "source": {
                    "dataSourceId": "[variables('convertRuleSourceWorkspace').SourceId]"
                },
                "action": {
                    "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.LogToMetricAction",
                    "criteria": [{
                            "metricName": "[parameters('convertRuleMetric')]",
                            "dimensions": []
                        }
                    ]
                }
            }
        },
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "dependsOn":["[resourceId('Microsoft.Insights/scheduledQueryRules',parameters('convertRuleName'))]"],
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('resourceId')]"],
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria",
                    "allOf": [
                        {
                            "name" : "1st criterion",
                            "metricName": "[parameters('metricName')]",
                            "dimensions":[],   
                            "operator": "[parameters('operator')]",
                            "threshold" : "[parameters('threshold')]",
                            "timeAggregation": "[parameters('timeAggregation')]"
                        }
                    ]
                },
                "actions": [
                    {
                        "actionGroupId": "[parameters('actionGroupId')]"                
                    }
                ]
            }
        }
    ]
}

```
假設上述的 JSON 儲存為 metricfromLogsAlert.json，其能與參數 JSON 檔案搭配，以供根據資源範本來建立。 以下列出範例參數 JSON 檔案：

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "value": "TestLogtoMetricRule" 
        },
        "convertRuleDescription": {
            "value": "Test rule to extract metrics from logs via template"
        },
        "convertRuleRegion": {
            "value": "West Central US"
        },
        "convertRuleStatus": {
            "value": "true"
        },
        "convertRuleMetric": {
            "value": "Average_% Idle Time"
        },
        "alertName": {
            "value": "TestMetricAlertonLog"
        },
        "alertDescription": {
            "value": "New multi-dimensional metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        },
        "metricName":{
            "value": "Average_% Idle Time"
        },
        "operator": {
            "value": "GreaterThan" 
        },
        "threshold":{
            "value": "1"
        },
        "timeAggregation":{
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/microsoft.insights/actionGroups/actionGroupName"
        }
    }    
}
```
假設上述參數檔案儲存為 metricfromLogsAlert.parameters.json，您可以使用[在 Azure 入口網站中用於建立的資源範本](../azure-resource-manager/resource-group-template-deploy-portal.md)，為記錄建立計量警示。 

此外，您也可以使用下列 Azure Powershell 命令：
```PowerShell
New-AzureRmResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile metricfromLogsAlert.json TemplateParameterFile metricfromLogsAlert.parameters.json
```

或，使用 Azure CLI 來使用部署資源範本：
```CLI
az group deployment create --resource-group myRG --template-file metricfromLogsAlert.json --parameters @metricfromLogsAlert.parameters.json
```

## <a name="next-steps"></a>後續步驟

* 深入了解[計量警示](https://aka.ms/createmetricalert)。
* 了解 [Azure 中的記錄警示](monitor-alerts-unified-log.md)。
* 深入了解 [Azure 中的警示](monitoring-overview-alerts.md)。
