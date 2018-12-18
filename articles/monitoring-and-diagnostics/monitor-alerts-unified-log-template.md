---
title: 使用 Resource Manager 範本建立記錄警示
description: 了解如何使用 Azure Resource Manager 範本和 API 建立記錄警示。
author: msvijayn
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: vinagara
ms.component: alerts
ms.openlocfilehash: 6096833381db7ef0d2f011d517aaad4ae63ce4d6
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576862"
---
# <a name="create-a-log-alert-with-a-resource-manager-template"></a>使用 Resource Manager 範本建立記錄警示
本文將說明如何透過 [Azure Powershell](../azure-resource-manager/resource-group-template-deploy.md) 和 [Azure CLI](../azure-resource-manager/resource-group-template-deploy-cli.md)在 Azure 中使用 [Azure Resource Manager 範本](..//azure-resource-manager/resource-group-authoring-templates.md)，以程式設計方式管理大規模的[記錄警示](monitor-alerts-unified-log.md)。 Azure 警示目前支援來自 [Azure Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 和 [Azure Application Insights](../application-insights/app-insights-analytics-tour.md) 查詢上的記錄警示。

## <a name="managing-log-alert-on-log-analytics"></a>管理 Log Analytics 上的記錄警示
[Azure Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 的記錄警示已整合至[新的 Azure 警示體驗](monitoring-overview-unified-alerts.md)；但其仍可執行 Log Analytics API，並與早先用來[在 OMS 入口網站中管理警示](..//log-analytics/log-analytics-alerts-creating.md)的結構描述相容。

> [!NOTE]
> 從 2018 年 5 月 14 日開始，工作區中的所有警示都將自動開始延伸至 Azure。 在 2018 年 5 月 14 日之前，使用者可以主動起始將警示延伸至 Azure。 如需詳細資訊，請參閱[將警示從 OMS 延伸至 Azure](monitoring-alerts-extend.md)。 

### <a name="using-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本
Log Analytics 的記錄警示是由警示規則所建立，這些規則會定期執行已儲存的搜尋。 如果查詢的結果符合指定的準則，就會建立警示記錄，並執行一或多個動作。 

您可以在文件的 Log Analytics 一節中，取得記錄分析儲存的搜尋和記錄分析警示資源範本。 若要深入了解，請參閱[新增 Log Analytics 的已儲存搜尋和警示](../operations-management-suite/operations-management-suite-solutions-resources-searches-alerts.md)；其中包含說明範例與結構描述詳細資料。

### <a name="using-resource-template-via-apipowershell"></a>透過 API/Powershell 使用資源範本
Log Analytics 警示 REST API 是 RESTful，可透過 Azure Resource Manager REST API 來存取。 因此您可以從 PowerShell 命令列存取 API，API 會以 JSON 格式向您輸出搜尋結果，讓您以程式設計方式透過許多不同的方法使用結果。

深入了解[使用 REST API 建立及管理 Log Analytics 中的警示規則](../log-analytics/log-analytics-api-alerts.md)；包括從 Powershell 存取 API 的範例。

## <a name="managing-log-alert-on-application-insights"></a>管理 Application Insights 上的記錄警示
我們已將 Azure Application Insights 的記錄警示引進 Azure 監視器，作為新 Azure 警示的一部分。 因此，其會作為[排程的查詢規則](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/) REST 作業群組，執行於 Azure 監視器之下。

### <a name="using-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本
Application Insights 資源的記錄警示有一個類型是 `Microsoft.Insights/scheduledQueryRules/`。 如需有關此資源類型的詳細資訊，請參閱 [Azure 監視器 - 排程的查詢規則 API 參考](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/)。

以下結構用於根據資源範本[建立排程的查詢規則](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/createorupdate)，其中使用資料集範例作為變數。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0", 
    "parameters": {      
    },   
    "variables": {
    "alertLocation": "southcentralus",
    "alertName": "samplelogalert",
    "alertTag": "hidden-link:/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/myRG/providers/microsoft.insights/components/sampleAIapplication",
    "alertDesription": "Sample log search alert",
    "alertStatus": "true",
    "alertSource":{
        "Query":"requests",
        "SourceId": "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/myRG/providers/microsoft.insights/components/sampleAIapplication",
        "Type":"ResultCount"
         },
     "alertSchedule":{
         "Frequency": 15,
         "Time": 60
         },
     "alertActions":{
         "SeverityLevel": "4"
         },
      "alertTrigger":{
        "Operator":"GreaterThan",
        "Threshold":"1"
         },
       "actionGrp":{
        "ActionGroup": "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/myRG/providers/microsoft.insights/actiongroups/sampleAG",
        "Subject": "Customized Email Header",
        "Webhook": "{ \"alertname\":\"#alertrulename\", \"IncludeSearchResults\":true }"           
         }
  },
  "resources":[ {
    "name":"[variables('alertName')]",
    "type":"Microsoft.Insights/scheduledQueryRules",
    "apiVersion": "2018-04-16",
    "location": "[variables('alertLocation')]",
    "tags":{"[variables('alertTag')]": "Resource"},
    "properties":{
       "description": "[variables('alertDesription')]",
       "enabled": "[variables('alertStatus')]",
       "source": {
           "query": "[variables('alertSource').Query]",
           "dataSourceId": "[variables('alertSource').SourceId]",
           "queryType":"[variables('alertSource').Type]"
       },
      "schedule":{
           "frequencyInMinutes": "[variables('alertSchedule').Frequency]",
           "timeWindowInMinutes": "[variables('alertSchedule').Time]"    
       },
      "action":{
           "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.AlertingAction",
           "severity":"[variables('alertActions').SeverityLevel]",
           "aznsAction":{
               "actionGroup":"[array(variables('actionGrp').ActionGroup)]",
               "emailSubject":"[variables('actionGrp').Subject]",
               "customWebhookPayload":"[variables('actionGrp').Webhook]"
           },
       "trigger":{
               "thresholdOperator":"[variables('alertTrigger').Operator]",
               "threshold":"[variables('alertTrigger').Threshold]"
           }
       }
     }
   }
 ]
}
```
> [!IMPORTANT]
> [排程的查詢規則](https://docs.microsoft.com/rest/api/monitor/scheduledqueryrules/) API 呼叫或資源範本中，一定會使用具有目標資源隱藏連結的標籤欄位。 

基於本逐步解說的目的，上述 json 範例可儲存為 (假設) sampleScheduledQueryRule.json，而且可以使用 [Azure 入口網站中的 Azure Resource Manager](../azure-resource-manager/resource-group-template-deploy-portal.md#deploy-resources-from-custom-template) 進行部署。

### <a name="using-resource-template-via-clipowershell"></a>透過 CLI/Powershell 使用資源範本
Azure 監視器 - 已排程的查詢規則 API 是 REST API，並且可與 Azure Resource Manager REST API 完全相容。 因此它可以透過 Powershell 和 Azure CLI 搭配資源管理員 Cmdlet 來使用。

以下說明透過 Azure Resource Manager PowerShell Cmdlet 使用上述資源範本範例 (sampleScheduledQueryRule.json) 的方式：
```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile "D:\Azure\Templates\sampleScheduledQueryRule.json"
```
以下說明透過 Azure CLI 中的 Azure Resource Manager 命令，來使用上述資源範本範例 (sampleScheduledQueryRule.json) 的方式：

```azurecli
az group deployment create --resource-group myRG --template-file sampleScheduledQueryRule.json
```
成功完成作業後，系統會傳回 201，說明新警示規則的建立，或是傳回 200，表示現有的警示規則已修改。


## <a name="next-steps"></a>後續步驟
* 了解 [Webhook 動作記錄警示](monitor-alerts-unified-log-webhook.md)
* 了解新的 [Azure 警示](monitoring-overview-unified-alerts.md)
* 深入了解 [Application Insights](../application-insights/app-insights-analytics.md)
* 深入了解 [Log Analytics](../log-analytics/log-analytics-overview.md)。   
