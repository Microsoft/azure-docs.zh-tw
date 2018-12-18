---
title: 使用 Resource Manager 範本自動啟用診斷設定
description: 了解如何使用 Resource Manager 範本來建立診斷設定，以讓您將診斷記錄檔串流至事件中樞，或將它們儲存在儲存體帳戶中。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 3/26/2018
ms.author: johnkem
ms.component: ''
ms.openlocfilehash: e8af84467c008f5c576142fa094b2757cfd30387
ms.sourcegitcommit: 156364c3363f651509a17d1d61cf8480aaf72d1a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39248073"
---
# <a name="automatically-enable-diagnostic-settings-at-resource-creation-using-a-resource-manager-template"></a>使用 Resource Manager 範本在建立資源時自動啟用診斷設定
在本文中，我們示範如何在建立資源時使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md) 設定診斷設定。 這可讓您在建立資源時，自動開始將您的診斷記錄檔和度量串流至事件中樞、將它們封存在儲存體帳戶中，或將它們傳送至 Log Analytics。

> [!WARNING]
> 2018 年 11 月 1 日起，儲存體帳戶中的記錄資料格式將變更為 JSON 資料行。 [請參閱本文以了解影響的描述，以及如何更新您的工具，來處理新的格式。](./monitor-diagnostic-logs-append-blobs.md) 
>
> 

使用 Resource Manager 範本啟用診斷記錄檔的方法，取決於資源類型。

* **非計算** 資源 (例如，網路安全性群組、Logic Apps、自動化) 使用 [這篇文章中所述的診斷設定](monitoring-overview-of-diagnostic-logs.md#diagnostic-settings)。
* **計算** 資源 (以 WAD/LAD 為基礎) 使用 [本文中所述的WAD/LAD 組態檔](../vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md)。

在本文中，我們會說明如何使用這兩種方法來設定診斷。

基本步驟如下：

1. 建立一個描述如何建立資源的 JSON 檔案做為範本，然後啟用診斷功能。
2. [使用任何部署方法部署範本](../azure-resource-manager/resource-group-template-deploy.md)。

以下提供產生非計算和計算資源所需的範本 JSON 檔案的範例。

## <a name="non-compute-resource-template"></a>非計算資源範本
如是非計算資源，您需要做兩件事︰

1. 將參數新增至儲存體帳戶名稱、事件中樞授權規則識別碼，和/或 Log Analytics 工作區識別碼的參數 blob (可啟用儲存體帳戶中診斷記錄的封存、串流記錄至事件中樞，和/或將記錄傳送至 Log Analytics)。
   
    ```json
    "settingName": {
      "type": "string",
      "metadata": {
        "description": "Name for the diagnostic setting resource. Eg. 'archiveToStorage' or 'forSecurityTeam'."
      }
    },
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Storage Account in which Diagnostic Logs should be saved."
      }
    },
    "eventHubAuthorizationRuleId": {
      "type": "string",
      "metadata": {
        "description": "Resource ID of the event hub authorization rule for the Event Hubs namespace in which the event hub should be created or streamed to."
      }
    },
    "eventHubName": {
      "type": "string",
      "metadata": {
        "description": "Optional. Name of the event hub within the namespace to which logs are streamed. Without this, an event hub is created for each log category."
      }
    },
    "workspaceId":{
      "type": "string",
      "metadata": {
        "description": "Azure Resource ID of the Log Analytics workspace for the Log Analytics workspace to which logs will be sent."
      }
    }
    ```
2. 在您要啟用診斷記錄檔的資源的資源陣列中，加入 `[resource namespace]/providers/diagnosticSettings`類型的資源。
   
    ```json
    "resources": [
      {
        "type": "providers/diagnosticSettings",
        "name": "[concat('Microsoft.Insights/', parameters('settingName'))]",
        "dependsOn": [
          "[/*resource Id for which Diagnostic Logs will be enabled>*/]"
        ],
        "apiVersion": "2017-05-01-preview",
        "properties": {
          "name": "[parameters('settingName')]",
          "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
          "eventHubAuthorizationRuleId": "[parameters('eventHubAuthorizationRuleId')]",
          "eventHubName": "[parameters('eventHubName')]",
          "workspaceId": "[parameters('workspaceId')]",
          "logs": [ 
            {
              "category": "/* log category name */",
              "enabled": true,
              "retentionPolicy": {
                "days": 0,
                "enabled": false
              }
            }
          ],
          "metrics": [
            {
              "category": "AllMetrics",
              "enabled": true,
              "retentionPolicy": {
                "enabled": false,
                "days": 0
              }
            }
          ]
        }
      }
    ]
    ```

診斷設定的屬性 blob 遵循 [這篇文章中所述的格式](https://docs.microsoft.com/rest/api/monitor/diagnosticsettings/createorupdate)。 新增 `metrics` 屬性可讓您同時傳送資源計量到這些相同的輸出，但前提是[資源支援 Azure 監視器計量](monitoring-supported-metrics.md)。

以下的完整範例會建立邏輯應用程式，並開啟串流至事件中樞和儲存體帳戶中的儲存體。

```json

{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "logicAppName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Logic App that will be created."
      }
    },
    "testUri": {
      "type": "string",
      "defaultValue": "http://azure.microsoft.com/status/feed/"
    },
    "settingName": {
      "type": "string",
      "metadata": {
        "description": "Name of the setting. Name for the diagnostic setting resource. Eg. 'archiveToStorage' or 'forSecurityTeam'."
      }
    },
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Storage Account in which Diagnostic Logs should be saved."
      }
    },
    "eventHubAuthorizationRuleId": {
      "type": "string",
      "metadata": {
        "description": "Resource ID of the event hub authorization rule for the Event Hubs namespace in which the event hub should be created or streamed to."
      }
    },
    "eventHubName": {
      "type": "string",
      "metadata": {
        "description": "Optional. Name of the event hub within the namespace to which logs are streamed. Without this, an event hub is created for each log category."
      }
    },
    "workspaceId": {
      "type": "string",
      "metadata": {
        "description": "Log Analytics workspace ID for the Log Analytics workspace to which logs will be sent."
      }
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Logic/workflows",
      "name": "[parameters('logicAppName')]",
      "apiVersion": "2016-06-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "definition": {
          "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "testURI": {
              "type": "string",
              "defaultValue": "[parameters('testUri')]"
            }
          },
          "triggers": {
            "recurrence": {
              "type": "recurrence",
              "recurrence": {
                "frequency": "Hour",
                "interval": 1
              }
            }
          },
          "actions": {
            "http": {
              "type": "Http",
              "inputs": {
                "method": "GET",
                "uri": "@parameters('testUri')"
              },
              "runAfter": {}
            }
          },
          "outputs": {}
        },
        "parameters": {}
      },
      "resources": [
        {
          "type": "providers/diagnosticSettings",
          "name": "[concat('Microsoft.Insights/', parameters('settingName'))]",
          "dependsOn": [
            "[resourceId('Microsoft.Logic/workflows', parameters('logicAppName'))]"
          ],
          "apiVersion": "2017-05-01-preview",
          "properties": {
            "name": "[parameters('settingName')]",
            "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
            "eventHubAuthorizationRuleId": "[parameters('eventHubAuthorizationRuleId')]",
            "eventHubName": "[parameters('eventHubName')]",
            "workspaceId": "[parameters('workspaceId')]",
            "logs": [
              {
                "category": "WorkflowRuntime",
                "enabled": true,
                "retentionPolicy": {
                  "days": 0,
                  "enabled": false
                }
              }
            ],
            "metrics": [
              {
                "timeGrain": "PT1M",
                "enabled": true,
                "retentionPolicy": {
                  "enabled": false,
                  "days": 0
                }
              }
            ]
          }
        }
      ],
      "dependsOn": []
    }
  ]
}

```

## <a name="compute-resource-template"></a>計算資源範本
若要啟用計算資源 (例如虛擬機器或 Service Fabric 叢集) 的診斷功能，您需要︰

1. 將 Azure 診斷擴充功能加入 VM 資源定義。
2. 指定儲存體帳戶和/或事件中樞做為參數。
3. 將 WADCfg XML 檔案的內容加入 XMLCfg 屬性，適當地逸出所有 XML 字元。

> [!WARNING]
> 這最後一個步驟要做得正確可能會有點棘手。 [請參閱這篇文章](../virtual-machines/extensions/diagnostics-template.md#diagnostics-configuration-variables) 的範例，將診斷組態結構描述分割成正確逸出和格式正確的變數。
> 
> 

在 [本文件中](../virtual-machines/windows/extensions-diagnostics-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中會說明整個程序，包括範例。

## <a name="next-steps"></a>後續步驟
* [深入了解 Azure 診斷記錄檔](monitoring-overview-of-diagnostic-logs.md)
* [將 Azure 診斷記錄檔串流至事件中樞](monitoring-stream-diagnostic-logs-to-event-hubs.md)

