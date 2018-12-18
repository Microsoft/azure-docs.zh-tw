---
title: 使用 Resource Manager 範本建立動作群組
description: 了解如何使用 Azure Resource Manager 範本建立動作群組。
author: dkamstra
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 02/16/2018
ms.author: dukek
ms.component: alerts
ms.openlocfilehash: 9b49d21dad9bb1e48194cc31940c5cd53c909dc0
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35263094"
---
# <a name="create-an-action-group-with-a-resource-manager-template"></a>使用 Resource Manager 範本建立動作群組
本文章將說明如何使用 [Azure Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates)設定動作群組。 您可以使用範本自動設定可在特定類型的警示中重複使用的動作群組。 這些動作群組能確定觸發警示時，所有正確的對象都會收到通知。

基本步驟為：

1. 建立一個描述如何建立動作群組的範本作為 JSON 檔案。

2. [使用任何部署方法部署範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)。

首先，我們會說明如何建立動作群組的 Resource Manager 範本，其中的動作定義是在範本中硬式編碼。 其次，我們會說明如何建立範本部署時，採用 Webhook 組態資訊作為輸入參數的範本。

## <a name="resource-manager-templates-for-an-action-group"></a>動作群組的 Resource Manager 範本

若要使用 Resource Manager 範本建立動作群組，您要建立 `Microsoft.Insights/actionGroups` 類型的資源。 然後要填入所有相關的屬性。 以下是兩個建立動作群組的範例範本。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroupName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Action group."
      }
    },
    "actionGroupShortName": {
      "type": "string",
      "metadata": {
        "description": "Short name (maximum 12 characters) for the Action group."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Insights/actionGroups",
      "apiVersion": "2018-03-01",
      "name": "[parameters('actionGroupName')]",
      "location": "Global",
      "properties": {
        "groupShortName": "[parameters('actionGroupShortName')]",
        "enabled": true,
        "smsReceivers": [
          {
            "name": "contosoSMS",
            "countryCode": "1",
            "phoneNumber": "5555551212"
          },
          {
            "name": "contosoSMS2",
            "countryCode": "1",
            "phoneNumber": "5555552121"
          }
        ],
        "emailReceivers": [
          {
            "name": "contosoEmail",
            "emailAddress": "devops@contoso.com"
          },
          {
            "name": "contosoEmail2",
            "emailAddress": "devops2@contoso.com"
          }
        ],
        "webhookReceivers": [
          {
            "name": "contosoHook",
            "serviceUri": "http://requestb.in/1bq62iu1"
          },
          {
            "name": "contosoHook2",
            "serviceUri": "http://requestb.in/1bq62iu2"
          }
        ]
      }
    }
  ],
  "outputs":{
      "actionGroupId":{
          "type":"string",
          "value":"[resourceId('Microsoft.Insights/actionGroups',parameters('actionGroupName'))]"
      }
  }
}
```

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroupName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Action group."
      }
    },
    "actionGroupShortName": {
      "type": "string",
      "metadata": {
        "description": "Short name (maximum 12 characters) for the Action group."
      }
    },
    "webhookReceiverName": {
      "type": "string",
      "metadata": {
        "description": "Webhook receiver service Name."
      }
    },    
    "webhookServiceUri": {
      "type": "string",
      "metadata": {
        "description": "Webhook receiver service URI."
      }
    }    
  },
  "resources": [
    {
      "type": "Microsoft.Insights/actionGroups",
      "apiVersion": "2018-03-01",
      "name": "[parameters('actionGroupName')]",
      "location": "Global",
      "properties": {
        "groupShortName": "[parameters('actionGroupShortName')]",
        "enabled": true,
        "smsReceivers": [
        ],
        "emailReceivers": [
        ],
        "webhookReceivers": [
          {
            "name": "[parameters('webhookReceiverName')]",
            "serviceUri": "[parameters('webhookServiceUri')]"
          }
        ]
      }
    }
  ],
  "outputs":{
      "actionGroupResourceId":{
          "type":"string",
          "value":"[resourceId('Microsoft.Insights/actionGroups',parameters('actionGroupName'))]"
      }
  }
}
```


## <a name="next-steps"></a>後續步驟
* 深入了解[動作群組](monitoring-action-groups.md)。
* 深入了解[警示](monitoring-overview-alerts.md)。
* 了解如何[使用 Resource Manager 範本新增警示](monitoring-create-activity-log-alerts-with-resource-manager-template.md)。
