---
title: 使用 Resource Manager 範本建立度量警示
description: 了解如何使用 Resource Manager 範本建立計量警示。
author: snehithm
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 9/27/2018
ms.author: snmuvva
ms.component: alerts
ms.openlocfilehash: 3739ad3d90dd9d97d755bc0e2fa41e0962396cb1
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49354235"
---
# <a name="create-a-metric-alert-with-a-resource-manager-template"></a>使用 Resource Manager 範本建立度量警示

本文章將說明如何使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)在 Azure 監視器中設定[新版計量警示](monitoring-near-real-time-metric-alerts.md)。 Resource Manager 範本可讓您以程式設計方式，在環境中以一致且可重現的方式設定警示。 新版計量警示目前可在[這組資源類型](monitoring-near-real-time-metric-alerts.md#metrics-and-dimensions-supported)上使用。

> [!IMPORTANT]
> 為計量警示指定的 Resource Manager 範本不適用於資源類型：Microsoft.OperationalInsights/workspaces，因為 Log Analytics 的計量支援為預覽狀態。 對資源範本預覽功能有興趣的使用者，請連絡 [Azure 警示意見反應](mailto:azurealertsfeedback@microsoft.com)


基本步驟如下：

1. 使用下列範本之一，作為描述如何建立警示的 JSON 檔案。
2. 編輯和使用 JSON 形式的對應參數檔案以自訂警示
3. 使用[任何部署方法](../azure-resource-manager/resource-group-template-deploy.md)部署範本。


## <a name="resource-manager-template-for-a-simple-metric-alert"></a>簡單計量警是的 Resource Manager 範本

若要使用 Resource Manager 範本建立警示，您要建立 `Microsoft.Insights/metricAlerts` 類型的資源，並填入所有相關的屬性。 以下是建立計量警示規則的範例範本。

根據本逐步解說的目的，請將以下的 JSON 儲存為 simplemetricalert.json。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
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
    "variables": {  },
    "resources": [
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
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

警示規則的結構描述和屬性說明 [請見這裡](https://docs.microsoft.com/rest/api/monitor/metricalerts/createorupdate)。

您可以在命令列上或透過參數檔案，設定參數的值。 範例參數檔案如下所示。 

將以下的 JSON 儲存為 simplemetricalert.parameters.json，並視需要進行修改。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "alertName": {
            "value": "New Metric Alert"
        },
        "alertDescription": {
            "value": "New metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resourceGroup-name/providers/Microsoft.Compute/virtualMachines/replace-with-resource-name"
        },
        "metricName": {
            "value": "Percentage CPU"
        },
        "operator": {
          "value": "GreaterThan"
        },
        "threshold": {
            "value": "80"
        },
        "timeAggregation": {
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/replace-with-subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Insights/actionGroups/replace-with-action-group"
        }
    }
}
```


您可以透過 PowerShell 或 Azure CLI 使用範本和參數檔案來建立計量警示。

使用 Azure PowerShell

```powershell
Connect-AzureRmAccount

Select-AzureRmSubscription -SubscriptionName <yourSubscriptionName>
 
New-AzureRmResourceGroupDeployment -Name AlertDeployment -ResourceGroupName ResourceGroupofTargetResource `
  -TemplateFile simplemetricalert.json -TemplateParameterFile simplemetricalert.parameters.json
```


使用 Azure CLI
```azurecli
az login

az group deployment create \
    --name AlertDeployment \
    --resource-group ResourceGroupofTargetResource \
    --template-file simplemetricalert.json \
    --parameters @simplemetricalert.parameters.json
```

> [!NOTE]
>
> 雖然計量警示能夠建立在與目標資源不同的資源群組中，但仍建議使用與目標資源相同的資源群組。

## <a name="resource-manager-template-for-a-more-advanced-metric-alert"></a>更進階的計量警示 Resource Manager 範本

新版計量警示支援多維度計量警示以及支援多項準則。 您可以使用下列範本，針對維度計量建立更進階的計量警示並指定多項準則。

根據本逐步解說的目的，請將以下的 JSON 儲存為 advancedmetricalert.json。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "alertName": {
            "type": "string",
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
            "defaultValue": "",
            "metadata": {
                "description": "Resource ID of the resource emitting the metric that will be used for the comparison."
            }
        },
        "criterion1":{
            "type": "object",
            "metadata": {
                "description": "Criterion includes metric name, dimension values, threshold and an operator. The alert rule fires when ALL criteria are met"
            }
        },
        "criterion2": {
            "type": "object",
            "metadata": {
                "description": "Criterion includes metric name, dimension values, threshold and an operator. The alert rule fires when ALL criteria are met"
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
        "criterion1": "[array(parameters('criterion1'))]",
        "criterion2": "[array(parameters('criterion2'))]",
        "criteria": "[concat(variables('criterion1'),variables('criterion2'))]"
     },
    "resources": [
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('resourceId')]"],
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria",
                    "allOf": "[variables('criteria')]"
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

您可以使用上述的範本以及下面提供的參數檔案。 

根據本逐步解說的目的，請將以下的 JSON 儲存為 advancedmetricalert.parameters.json 並修改。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "alertName": {
            "value": "New Multi-dimensional Metric Alert (Replace with your alert name)"
        },
        "alertDescription": {
            "value": "New multi-dimensional metric alert created via template (Replace with your alert description)"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resourcegroup-name/providers/Microsoft.Storage/storageAccounts/replace-with-storage-account"
        },
        "criterion1": {
            "value": {
                    "name": "1st criterion",
                    "metricName": "Transactions",
                    "dimensions": [
                        {
                            "name":"ResponseType",
                            "operator": "Include",
                            "values": ["Success"]
                        },
                        {
                            "name":"ApiName",
                            "operator": "Include",
                            "values": ["GetBlob"]
                        }
                    ],
                    "operator": "GreaterThan",
                    "threshold": "5",
                    "timeAggregation": "Total"
                }
        },
        "criterion2": {
            "value":{
                "name": "2nd criterion",
                "metricName": "SuccessE2ELatency",
                "dimensions": [
                    {
                        "name":"ApiName",
                        "operator": "Include",
                        "values": ["GetBlob"]
                    }
                ],
                "operator": "GreaterThan",
                "threshold": "250",
                "timeAggregation": "Average"
            }      
        },          
        "actionGroupId": {
            "value": "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name/providers/Microsoft.Insights/actionGroups/replace-with-actiongroup-name"
        }
    }    
}
```


您可以從目前的工作資料夾，透過 PowerShell 或 Azure CLI 使用範本和參數檔案來建立計量警示

使用 Azure PowerShell
```powershell
Connect-AzureRmAccount

Select-AzureRmSubscription -SubscriptionName <yourSubscriptionName>
 
New-AzureRmResourceGroupDeployment -Name AlertDeployment -ResourceGroupName ResourceGroupofTargetResource `
  -TemplateFile advancedmetricalert.json -TemplateParameterFile advancedmetricalert.parameters.json
```



使用 Azure CLI
```azurecli
az login

az group deployment create \
    --name AlertDeployment \
    --resource-group ResourceGroupofTargetResource \
    --template-file advancedmetricalert.json \
    --parameters @advancedmetricalert.parameters.json
```

>[!NOTE]
>
> 雖然計量警示能夠建立在與目標資源不同的資源群組中，但仍建議使用與目標資源相同的資源群組。

## <a name="resource-manager-template-for-metric-alert-that-monitors-multiple-resources"></a>監視多個資源的計量警示 Resource Manager 範本

上一節已說明用來建立計量警示以監視單一資源的 Azure Resource Manager 範本範例。 Azure 監視器現在支援以單一計量警示規則監視多個資源。 此預覽功能目前只能透過 Azure Resource Manager 範本和 REST API 提供，並僅適用於虛擬機器。 不久後，入口網站也將可支援建立和編輯這類規則。

針對以單一規則監視多個資源的功能，本節將說明三種此類案例適用的 Azure Resource Manager 範本。

- 監視一個或多個資源群組中的所有虛擬機器 (位於一個 Azure 區域)。
- 監視一個訂用帳戶中的所有虛擬機器 (位於一個 Azure 區域)
- 監視一個訂用帳戶中的一系列虛擬機器 (位於一個 Azure 區域)。

### <a name="all-virtual-machines-in-one-or-more-resource-groups"></a>一個或多個資源群組中的所有虛擬機器

此範本會為一個或多個資源群組中的所有虛擬機器 (位於一個 Azure 區域)，建立可監視 CPU 百分比的計量警示規則。

根據本逐步解說的目的，請將以下的 JSON 儲存為 all-vms-in-resource-group.json。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
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
        "targetResourceGroup":{
            "type": "array",
            "minLength": 1,
            "metadata": {
                "description": "Full path of the resource group(s) where target resources to be monitored are in. For example - /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroups/ResourceGroupName"
            }
        },
        "targetResourceRegion":{
            "type": "string",
            "allowedValues": [
                "EastUS",
                "EastUS2",
                "CentralUS",
                "NorthCentralUS",
                "SouthCentralUS",
                "WestCentralUS",
                "WestUS",
                "WestUS2",
                "CanadaEast",
                "CanadaCentral",
                "BrazilSouth",
                "NorthEurope",
                "WestEurope",
                "FranceCentral",
                "FranceSouth",
                "UKWest",
                "UKSouth",
                "GermanyCentral",
                "GermanyNortheast",
                "GermanyNorth",
                "GermanyWestCentral",
                "SwitzerlandNorth",
                "SwitzerlandWest",
                "NorwayEast",
                "NorwayWest",
                "SoutheastAsia",
                "EastAsia",
                "AustraliaEast",
                "AustraliaSoutheast",
                "AustraliaCentral",
                "AustraliaCentral2",
                "ChinaEast",
                "ChinaNorth",
                "ChinaEast2",
                "ChinaNorth2",
                "CentralIndia",
                "WestIndia",
                "SouthIndia",
                "JapanEast",
                "JapanWest",
                "KoreaCentral",
                "KoreaSouth",
                "SouthAfricaWest",
                "SouthAfricaNorth",
                "UAECentral",
                "UAENorth"
            ],
            "metadata": {
                "description": "Azure region in which target resources to be monitored are in (without spaces). For example: EastUS"
            }
        },
        "targetResourceType": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Resource type of target resources to be monitored. Currently only supported resource type is Microsoft.Compute/virtualMachines"
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
    "variables": {  },
    "resources": [
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": "[parameters('targetResourceGroup')]",
                "targetResourceType": "[parameters('targetResourceType')]",
                "targetResourceRegion": "[parameters('targetResourceRegion')]",
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria",
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

您可以使用上述範本及下列參數檔案。
根據本逐步解說的目的，請將以下的 JSON 儲存為 all-vms-in-resource-group.parameters.json 並修改。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "alertName": {
            "value": "Multi-resource metric alert via Azure Resource Manager template"
        },
        "alertDescription": {
            "value": "New Multi-resource metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "targetResourceGroup":{
            "value": [
                "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name1",
                "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name2"
            ]
        },
        "targetResourceRegion":{
            "value": "SouthCentralUS"
        },
        "targetResourceType":{
            "value": "Microsoft.Compute/virtualMachines"
        },
        "metricName": {
            "value": "Percentage CPU"
        },
        "operator": {
          "value": "GreaterThan"
        },
        "threshold": {
            "value": "0"
        },
        "timeAggregation": {
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name/providers/Microsoft.Insights/actionGroups/replace-with-action-group-name"
        }
    }
}
```

您可以從目前的工作資料夾，透過 PowerShell 或 Azure CLI 使用範本和參數檔案來建立計量警示。

使用 Azure PowerShell

```powershell
Connect-AzureRmAccount

Select-AzureRmSubscription -SubscriptionName <yourSubscriptionName>

New-AzureRmResourceGroupDeployment -Name MultiResourceAlertDeployment -ResourceGroupName ResourceGroupWhereRuleShouldbeSaved `
  -TemplateFile all-vms-in-resource-group.json -TemplateParameterFile all-vms-in-resource-group.parameters.json
```

使用 Azure CLI

```azurecli
az login

az group deployment create \
    --name MultiResourceAlertDeployment \
    --resource-group ResourceGroupWhereRuleShouldbeSaved \
    --template-file all-vms-in-resource-group.json \
    --parameters @all-vms-in-resource-group.parameters.json
```

### <a name="all-virtual-machines-in-a-subscription"></a>一個訂用帳戶中的所有虛擬機器

此範本會為一個訂用帳戶中的所有虛擬機器 (位於一個 Azure 區域)，建立可監視 CPU 百分比的計量警示規則。

根據本逐步解說的目的，請將以下的 JSON 儲存為 all-vms-in-subscription.json。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
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
        "targetSubscription":{
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Azure Resource Manager path up to subscription ID. For example - /subscriptions/00000000-0000-0000-0000-0000-00000000"
            }
        },
        "targetResourceRegion":{
            "type": "string",
            "allowedValues": [
                "EastUS",
                "EastUS2",
                "CentralUS",
                "NorthCentralUS",
                "SouthCentralUS",
                "WestCentralUS",
                "WestUS",
                "WestUS2",
                "CanadaEast",
                "CanadaCentral",
                "BrazilSouth",
                "NorthEurope",
                "WestEurope",
                "FranceCentral",
                "FranceSouth",
                "UKWest",
                "UKSouth",
                "GermanyCentral",
                "GermanyNortheast",
                "GermanyNorth",
                "GermanyWestCentral",
                "SwitzerlandNorth",
                "SwitzerlandWest",
                "NorwayEast",
                "NorwayWest",
                "SoutheastAsia",
                "EastAsia",
                "AustraliaEast",
                "AustraliaSoutheast",
                "AustraliaCentral",
                "AustraliaCentral2",
                "ChinaEast",
                "ChinaNorth",
                "ChinaEast2",
                "ChinaNorth2",
                "CentralIndia",
                "WestIndia",
                "SouthIndia",
                "JapanEast",
                "JapanWest",
                "KoreaCentral",
                "KoreaSouth",
                "SouthAfricaWest",
                "SouthAfricaNorth",
                "UAECentral",
                "UAENorth"
            ],
            "metadata": {
                "description": "Azure region in which target resources to be monitored are in (without spaces). For example: EastUS"
            }
        },
        "targetResourceType": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Resource type of target resources to be monitored. Currently only supported resource type is Microsoft.Compute/virtualMachines"
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
    "variables": {  },
    "resources": [
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('targetSubscription')]"],
                "targetResourceType": "[parameters('targetResourceType')]",
                "targetResourceRegion": "[parameters('targetResourceRegion')]",
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria",
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

您可以使用上述範本及下列參數檔案。
根據本逐步解說的目的，請將以下的 JSON 儲存為 all-vms-in-subscription.parameters.json 並修改。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "alertName": {
            "value": "Multi-resource sub level metric alert via Azure Resource Manager template"
        },
        "alertDescription": {
            "value": "New Multi-resource sub level metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "targetSubscription":{
            "value": "/subscriptions/replace-with-subscription-id"
        },
        "targetResourceRegion":{
            "value": "SouthCentralUS"
        },
        "targetResourceType":{
            "value": "Microsoft.Compute/virtualMachines"
        },        
        "metricName": {
            "value": "Percentage CPU"
        },
        "operator": {
          "value": "GreaterThan"
        },
        "threshold": {
            "value": "0"
        },
        "timeAggregation": {
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name/providers/Microsoft.Insights/actionGroups/replace-with-action-group-name"
        }
    }
}
```

您可以從目前的工作資料夾，透過 PowerShell 或 Azure CLI 使用範本和參數檔案來建立計量警示。

使用 Azure PowerShell

```powershell
Connect-AzureRmAccount

Select-AzureRmSubscription -SubscriptionName <yourSubscriptionName>

New-AzureRmResourceGroupDeployment -Name MultiResourceAlertDeployment -ResourceGroupName ResourceGroupWhereRuleShouldbeSaved `
  -TemplateFile all-vms-in-subscription.json -TemplateParameterFile all-vms-in-subscription.parameters.json
```

使用 Azure CLI

```azurecli
az login

az group deployment create \
    --name MultiResourceAlertDeployment \
    --resource-group ResourceGroupWhereRuleShouldbeSaved \
    --template-file all-vms-in-subscription.json \
    --parameters @all-vms-in-subscription.parameters.json
```

### <a name="a-list-of-virtual-machines"></a>一系列虛擬機器

此範本會為一個訂用帳戶中的一系列虛擬機器 (位於一個 Azure 區域)，建立可監視 CPU 百分比的計量警示規則。

根據本逐步解說的目的，請將以下的 JSON 儲存為 list-of-vms.json。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
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
        "targetResourceId":{
            "type": "array",
            "minLength": 1,
            "metadata": {
                "description": "array of Azure resource Ids. For example - /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroup/resource-group-name/Microsoft.compute/virtualMachines/vm-name"
            }
        },
        "targetResourceRegion":{
            "type": "string",
            "allowedValues": [
                "EastUS",
                "EastUS2",
                "CentralUS",
                "NorthCentralUS",
                "SouthCentralUS",
                "WestCentralUS",
                "WestUS",
                "WestUS2",
                "CanadaEast",
                "CanadaCentral",
                "BrazilSouth",
                "NorthEurope",
                "WestEurope",
                "FranceCentral",
                "FranceSouth",
                "UKWest",
                "UKSouth",
                "GermanyCentral",
                "GermanyNortheast",
                "GermanyNorth",
                "GermanyWestCentral",
                "SwitzerlandNorth",
                "SwitzerlandWest",
                "NorwayEast",
                "NorwayWest",
                "SoutheastAsia",
                "EastAsia",
                "AustraliaEast",
                "AustraliaSoutheast",
                "AustraliaCentral",
                "AustraliaCentral2",
                "ChinaEast",
                "ChinaNorth",
                "ChinaEast2",
                "ChinaNorth2",
                "CentralIndia",
                "WestIndia",
                "SouthIndia",
                "JapanEast",
                "JapanWest",
                "KoreaCentral",
                "KoreaSouth",
                "SouthAfricaWest",
                "SouthAfricaNorth",
                "UAECentral",
                "UAENorth"
            ],
            "metadata": {
                "description": "Azure region in which target resources to be monitored are in (without spaces). For example: EastUS"
            }
        },
        "targetResourceType": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Resource type of target resources to be monitored. Currently only supported resource type is Microsoft.Compute/virtualMachines"
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
    "variables": {  },
    "resources": [
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": "[parameters('targetResourceId')]",
                "targetResourceType": "[parameters('targetResourceType')]",
                "targetResourceRegion": "[parameters('targetResourceRegion')]",
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria",
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

您可以使用上述範本及下列參數檔案。
根據本逐步解說的目的，請將以下的 JSON 儲存為 list-of-vms.parameters.json 並修改。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "alertName": {
            "value": "Multi-resource metric alert by list via Azure Resource Manager template"
        },
        "alertDescription": {
            "value": "New Multi-resource metric alert by list created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "targetResourceId":{
            "value": [
                "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name1/Microsoft.Compute/virtualMachines/replace-with-vm-name1",
                "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name2/Microsoft.Compute/virtualMachines/replace-with-vm-name2"
            ]
        },
        "targetResourceRegion":{
            "value": "SouthCentralUS"
        },
        "targetResourceType":{
            "value": "Microsoft.Compute/virtualMachines"
        },
        "metricName": {
            "value": "Percentage CPU"
        },
        "operator": {
          "value": "GreaterThan"
        },
        "threshold": {
            "value": "0"
        },
        "timeAggregation": {
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/replace-with-subscription-id/resourceGroups/replace-with-resource-group-name/providers/Microsoft.Insights/actionGroups/replace-with-action-group-name"
        }
    }
}
```

您可以從目前的工作資料夾，透過 PowerShell 或 Azure CLI 使用範本和參數檔案來建立計量警示。

使用 Azure PowerShell

```powershell
Connect-AzureRmAccount

Select-AzureRmSubscription -SubscriptionName <yourSubscriptionName>

New-AzureRmResourceGroupDeployment -Name MultiResourceAlertDeployment -ResourceGroupName ResourceGroupWhereRuleShouldbeSaved `
  -TemplateFile list-of-vms.json -TemplateParameterFile list-of-vms.parameters.json
```

使用 Azure CLI

```azurecli
az login

az group deployment create \
    --name MultiResourceAlertDeployment \
    --resource-group ResourceGroupWhereRuleShouldbeSaved \
    --template-file list-of-vms.json \
    --parameters @list-of-vms.parameters.json
```

## <a name="next-steps"></a>後續步驟
* 深入了解 [Azure 中的警示](monitoring-overview-unified-alerts.md)
* 了解如何[使用 Resource Manager 範本建立動作群組](monitoring-create-action-group-with-resource-manager-template.md)
