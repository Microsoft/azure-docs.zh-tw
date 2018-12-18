---
title: 使用 Resource Manager 範本設定 Azure 資源健康狀態警示 | Microsoft Docs
description: 以程式設計方式建立警示，在您的 Azure 資源變成無法使用時通知您。
author: shawntabrizi
manager: scotthit
editor: ''
services: service-health
documentationcenter: service-health
ms.assetid: ''
ms.service: service-health
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 9/4/2018
ms.author: shtabriz
ms.openlocfilehash: b19a840e5d0d3434d3e3ff91e0d7af037d64fb5a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46975355"
---
# <a name="configure-resource-health-alerts-using-resource-manager-templates"></a>使用 Resource Manager 範本設定資源健康狀態警示

本文將示範如何使用 Azure Resource Manager 範本和 Azure PowerShell，以程式設計方式建立資源健康狀態活動記錄警示。

Azure 資源健康狀態會隨時通知您 Azure 資源目前和過去的健康狀態。 Azure 資源健康狀態警示會在這些資源的健康狀態變更時，以近乎即時的方式通知您。 以程式設計方式建立資源健康狀態警示，讓使用者能夠建立並自訂大量警示。

## <a name="prerequisites"></a>必要條件

若要遵循此頁面上的指示，您將必須事先設定幾件事：

1. 您必須安裝 [Azure PowerShell 模組](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) (`AzureRm`)
2. 您需要[建立或重複使用動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)，該動作群組會設定來通知您

## <a name="instructions"></a>範例的指示
1. 使用 PowerShell、使用您的帳戶登入 Azure，然後選取您想要與之互動的訂用帳戶

        Login-AzureRmAccount
        Select-AzureRmSubscription -Subscription <subscriptionId>

    > 您可以使用 `Get-AzureRmSubscription` 來列出您有權存取的訂用帳戶。

2. 針對您的動作群組尋找並儲存完整的 Azure Resource Manager 識別碼

        (Get-AzureRmActionGroup -ResourceGroupName <resourceGroup> -Name <actionGroup>).Id

3. 建立適用於資源健康狀態警示的 Resource Manager 範本並儲存為 `resourcehealthalert.json` ([請參閱下方的詳細資料](#resource-manager-template-for-resource-health-alerts))

4. 使用此範本建立新的 Azure Resource Manager 部署

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName <resourceGroup> -TemplateFile <path\to\resourcehealthalert.json>

5. 系統將提示您輸入警示名稱和您稍早複製的動作群組資源識別碼：

        Supply values for the following parameters:
        (Type !? for Help.)
        activityLogAlertName: <Alert Name>
        actionGroupResourceId: /subscriptions/<subscriptionId>/resourceGroups/<resouceGroup>/providers/microsoft.insights/actionGroups/<actionGroup>

6. 如果一切均順利執行，您將會在 PowerShell 中收到確認訊息

        DeploymentName          : ExampleDeployment
        ResourceGroupName       : <resourceGroup>
        ProvisioningState       : Succeeded
        Timestamp               : 11/8/2017 2:32:00 AM
        Mode                    : Incremental
        TemplateLink            :
        Parameters              :
                                Name                     Type       Value
                                ===============          =========  ==========
                                activityLogAlertName     String     <Alert Name>
                                activityLogAlertEnabled  Bool       True
                                actionGroupResourceId    String     /...
        
        Outputs                 :
        DeploymentDebugLogLevel :

請注意，如果您打算將此程序完全自動化，只需編輯 Resource Manager 範本，使其不提示您輸入步驟 5 中的值。

## <a name="resource-manager-template-for-resource-health-alerts"></a>適用於資源健康狀態警示的 Resource Manager 範本

您可以使用這個基底範本，做為建立資源健康狀態警示的起點。 此範本將如撰寫般運作，而且會將您註冊，以便在訂用帳戶中的所有資源上接收所有新啟動資源健康狀態事件的警示。

> 我們也會在本文底部提供一個更複雜的警示範本，相較於此範本，該範本應該會提高資源健康狀態警示的訊號雜訊比率。

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
        "enabled": true,
        "scopes": [
            "[subscription().id]"
        ],        
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "ResourceHealth"
            },
            {
              "field": "status",
              "equals": "Active"
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

不過，通常不建議使用與此類似的廣泛警示。 了解如何縮小此警示的範圍，以將焦點放在我們所關心的事件上。

### <a name="adjusting-the-alert-scope"></a>調整警示範圍

資源健康狀態警示可設定來監視三個不同範圍的事件：

 * 訂用帳戶層級
 * 資源群組層級
 * 資源層級

警示範本已設定於訂用帳戶層級，但如果您想要設定警示，以便僅通知您特定資源或特定資源群組內資源的相關事項，您只需修改上一個範本中的 `scopes` 區段。

如果是資源群組層級範圍，scopes 區段應該看起來如下：
```json
"scopes": [
    "/subscriptions/<subscription id>/resourcegroups/<resource group>"
],
```

此外，如果是資源層級範圍，則 scope 區段應該看起來如下：

```json
"scopes": [
    "/subscriptions/<subscription id>/resourcegroups/<resource group>/providers/<resource>"
],
```

例如：`"/subscriptions/d37urb3e-ed41-4670-9c19-02a1d2808ff9/resourcegroups/myRG/providers/microsoft.compute/virtualmachines/myVm"`

> 您可以移至 Azure 入口網站，並在檢視您的 Azure 資源時查看 URL 以取得此字串。

### <a name="adjusting-the-resource-types-which-alert-you"></a>調整要警示您的資源類型

位於訂用帳戶或資源群組層級的警示可能具有不同種類的資源。 如果您想要將警示限制為只來自特定子集的資源類型，您可以在範本的 `condition` 區段中定義該情況，如下：

```json
"condition": {
    "allOf": [
        ...,
        {
            "anyOf": [
                {
                    "field": "resourceType",
                    "equals": "Microsoft.Compute/virtualMachines",
                    "containsAny": null
                },
                {
                    "field": "resourceType",
                    "equals": "Microsoft.Storage/storageAccounts",
                    "containsAny": null
                },
                ...
            ]
        }
    ]
},
```

我們在這裡使用 `anyOf` 包裝函式，讓資源健康情況警示能夠符合我們所指定的任何一個條件，從而允許用於目標為特定資源類型的警示。

### <a name="adjusting-the-resource-health-events-that-alert-you"></a>調整要警示您的資源健康狀態事件
當資源經歷健康情況事件時，它們會經過一系列代表健康情況事件狀態的階段：`Active`、`InProgress`、`Updated` 和 `Resolved`。

您可能只想在資源變為狀況不良時收到通知，在此情況下，您希望將警示設定為只有在 `status` 是 `Active` 時才進行通知。 但是，如果您也想要在其他階段收到通知，您可以新增如下的詳細資料：

```json
"condition": {
    "allOf": [
        ...,
        {
            "anyOf": [
                {
                    "field": "status",
                    "equals": "Active"
                },
                {
                    "field": "status",
                    "equals": "InProgress"
                },
                        {
                    "field": "status",
                    "equals": "Resolved"
                }
            ]
        }
    ]
}
```

如果您想要收到所有四個階段之健康情況事件的通知，您可以一併移除此條件，不論 `status` 屬性為何，警示都將通知您。

### <a name="adjusting-the-resource-health-alerts-to-avoid-unknown-events"></a>調整資源健康狀態警示以避免「未知」事件

Azure 資源健康狀態可以使用測試執行器持續監視資源，以向您報告資源的最新健康情況。 相關報告的健康狀態是：「可用」、「無法使用」和「已降級」。 不過，在執行器和 Azure 資源無法通訊的情況下，會針對資源報告「未知」的健康狀態，並將其視為「作用中」的健康情況事件。

不過，當資源報告「未知」時，很可能其健康狀態自上次正確的報告之後並未變更。 如果您想要消除關於「未知」事件的警示，您可以在範本中指定該邏輯：

```json
"condition": {
    "allOf": [
        ...,
        {
            "anyOf": [
                {
                    "field": "properties.currentHealthStatus",
                    "equals": "Available",
                    "containsAny": null
                },
                {
                    "field": "properties.currentHealthStatus",
                    "equals": "Unavailable",
                    "containsAny": null
                },
                {
                    "field": "properties.currentHealthStatus",
                    "equals": "Degraded",
                    "containsAny": null
                }
            ]
        },
        {
            "anyOf": [
                {
                    "field": "properties.previousHealthStatus",
                    "equals": "Available",
                    "containsAny": null
                },
                {
                    "field": "properties.previousHealthStatus",
                    "equals": "Unavailable",
                    "containsAny": null
                },
                {
                    "field": "properties.previousHealthStatus",
                    "equals": "Degraded",
                    "containsAny": null
                }
            ]
        },
    ]
},
```

在此範例中，我們只會通知目前和先前健康狀態沒有「未知」的事件。 如果您的警示會直接傳送到您的行動電話或電子郵件，則此變更可能是很有用的新增項目。

### <a name="adjusting-the-alert-to-avoid-user-initiated-events"></a>調整警示以避免由使用者初始化的事件

資源健康狀態事件可以透過由平台初始化和由使用者初始化的事件來觸發。 只有健康情況事件是由 Azure 平台所導致時才傳送通知，這或許是可行的。

您可以輕鬆地將警示設定為只篩選這些種類的事件：

```json
"condition": {
    "allOf": [
        ...,
        {
            "field": "properties.cause",
            "equals": "PlatformInitiated",
            "containsAny": null
        }
    ]
}
```

## <a name="recommended-resource-health-alert-template"></a>建議的資源健康狀態警示範本

使用上一節所述的不同調整，我們可以建立完整的警示範本，並設定為要將訊號雜訊比率最大化。

以下是我們建議您使用的內容：
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
                "enabled": true,
                "scopes": [
                    "[subscription().id]"
                ],
                "condition": {
                    "allOf": [
                        {
                            "field": "category",
                            "equals": "ResourceHealth",
                            "containsAny": null
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "properties.currentHealthStatus",
                                    "equals": "Available",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.currentHealthStatus",
                                    "equals": "Unavailable",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.currentHealthStatus",
                                    "equals": "Degraded",
                                    "containsAny": null
                                }
                            ]
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "properties.previousHealthStatus",
                                    "equals": "Available",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.previousHealthStatus",
                                    "equals": "Unavailable",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.previousHealthStatus",
                                    "equals": "Degraded",
                                    "containsAny": null
                                }
                            ]
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "properties.cause",
                                    "equals": "PlatformInitiated",
                                    "containsAny": null
                                }
                            ]
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "status",
                                    "equals": "Active",
                                    "containsAny": null
                                },
                                {
                                    "field": "status",
                                    "equals": "Resolved",
                                    "containsAny": null
                                },
                                {
                                    "field": "status",
                                    "equals": "InProgress",
                                    "containsAny": null
                                }
                            ]
                        }
                    ]
                },
                "actions": {
                    "actionGroups": [
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

不過，您將最了解哪些設定適合您使用，因此，請使用本文件教您的工具來進行自己的自訂。

## <a name="next-steps"></a>後續步驟

深入了解資源健康狀態：
-  [Azure 資源健康狀態概觀](Resource-health-overview.md)
-  [可透過 Azure 資源健康狀態使用的資源類型和健康檢查](resource-health-checks-resource-types.md)

建立服務健康狀態警示：
-  [設定適用於服務健康情況的警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts-on-service-notifications.md) 
