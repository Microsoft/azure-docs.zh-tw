---
title: 使用 Resource Manager 範本建立活動記錄警示
description: 在 Azure 資源建立時取得通知。
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 07/06/2017
ms.author: ancav
ms.component: alerts
ms.openlocfilehash: a1e28f08231ae1fbef3e0d0306e986c1dc9d1d1c
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35262985"
---
# <a name="create-an-activity-log-alert-with-a-resource-manager-template"></a>使用 Resource Manager 範本建立活動記錄警示
本文章將說明如何使用 [Azure Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates)設定活動記錄警示。 您可以使用範本輕鬆地設定許多警示，會以您自動部署程序中特定的活動記錄事件條件作為基礎加以啟動。

基本步驟為：

1. 建立一個以描述如何建立活動記錄警示的 JSON 檔案為形式的範本。

2. [使用任何部署方法部署範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)。

## <a name="resource-manager-template-for-an-activity-log-alert"></a>活動記錄警示的 Resource Manager 範本
若要使用 Resource Manager 範本建立活動記錄警示，您要建立 `microsoft.insights/activityLogAlerts` 類型的資源。 然後要填入所有相關的屬性。 以下是建立活動記錄警示的範本。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "activityLogAlertName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Activity log alert."
      }
    },
    "activityLogAlertEnabled": {
      "type": "bool",
      "defaultValue": true,
      "metadata": {
        "description": "Indicates whether or not the alert is enabled."
      }
    },
    "actionGroupResourceId": {
      "type": "string",
      "metadata": {
        "description": "Resource Id for the Action group."
      }
    }
  },
  "resources": [   
    {
      "type": "Microsoft.Insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlertName')]",      
      "location": "Global",
      "properties": {
        "enabled": "[parameters('activityLogAlertEnabled')]",
        "scopes": [
            "[subscription().id]"
        ],        
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Administrative"
            },
            {
              "field": "operationName",
              "equals": "Microsoft.Resources/deployments/write"
            },
            {
              "field": "resourceType",
              "equals": "Microsoft.Resources/deployments"
            }
          ]
        },
        "actions": {
          "actionGroups":
          [
            {
              "actionGroupId": "[parameters('actionGroupResourceId')]"
            }
          ]
        }
      }
    }
  ]
}
```

請瀏覽我們的 [Azure 快速入門資源庫](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Insights)，取得活動記錄警示範本的一些範例。

> [!NOTE]

> 您也可以使用增強的使用者體驗，在 [監視] >[警示 (預覽)](monitoring-overview-unified-alerts.md) 中建立活動記錄警示規則。 如需有關如何建立這些規則的詳細資訊，請參閱[本文](monitoring-activity-log-alerts-new-experience.md)。

## <a name="next-steps"></a>後續步驟
- 深入了解[警示](monitoring-overview-alerts.md)。
- 了解如何[使用 Resource Manager 範本新增動作群組](monitoring-create-action-group-with-resource-manager-template.md)。
- 了解如何[建立活動記錄警示以監視訂用帳戶的所有自動調整引擎作業](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert)。
- 了解如何[建立活動記錄警示以監視訂用帳戶中所有失敗的相應縮小/相應放大自動調整作業](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert)。
