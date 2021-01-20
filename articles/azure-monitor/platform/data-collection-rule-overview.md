---
title: 'Azure 監視器 (預覽中的資料收集規則) '
description: 資料收集規則的總覽 (Dcr) Azure 監視器包括其內容和結構，以及如何建立和使用它們。
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/19/2021
ms.openlocfilehash: 7013a4ab1becd6108d30d8369f1f72bcb3e55c37
ms.sourcegitcommit: 8a74ab1beba4522367aef8cb39c92c1147d5ec13
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98611061"
---
# <a name="data-collection-rules-in-azure-monitor-preview"></a>Azure 監視器 (預覽中的資料收集規則) 
資料收集規則 (DCR) 定義進入 Azure 監視器的資料，並指定應該傳送或儲存資料的位置。 本文概述資料收集規則，包括其內容和結構，以及您可以如何建立和使用它們。

## <a name="input-sources"></a>輸入來源
資料收集規則目前支援下列輸入來源：

- 具有 Azure 監視器代理程式的 Azure 虛擬機器。 請參閱 [設定 Azure 監視器代理程式 (預覽) 的資料收集 ](data-collection-rule-azure-monitor-agent.md)。



## <a name="components-of-a-data-collection-rule"></a>資料集合規則的元件
資料收集規則包含下列元件。

| 元件 | 說明 |
|:---|:---|
| 資料來源 | 監視資料的唯一來源，其本身的格式和公開其資料的方法。 資料來源的範例包括 Windows 事件記錄檔、效能計數器和 syslog。 每個資料來源都符合如下所述的特定資料來源類型。 |
| 串流 | 唯一的控制碼，描述將轉換並架構化為一種類型的一組資料來源。 每個資料來源都需要一個或多個資料流程，而一個資料流程可能會由多個資料來源使用。 資料流程中的所有資料來源都會共用一個通用的架構。 例如，當您想要將特定資料來源傳送到相同 Log Analytics 工作區中的多個資料表時，請使用多個資料流程。 |
| 目的地 | 應傳送資料的目的地集合。 範例包括 Log Analytics 工作區、Azure 監視器計量和 Azure 事件中樞。 | 
| 資料流程 | 應將哪些資料流程傳送至哪些目的地的定義。 | 

下圖顯示資料集合規則及其關聯性的元件

[![DCR 的圖表](media/data-collection-rule/data-collection-rule-components.png)](media/data-collection-rule/data-collection-rule-components.png#lightbox)

### <a name="data-source-types"></a>資料來源類型
每個資料來源都有一種資料來源類型。 每個類型都會定義一組唯一的屬性，這些屬性必須針對每個資料來源指定。 下表顯示目前可用的資料來源類型。

| 資料來源類型 | 說明 | 
|:---|:---|
| 擴充功能 | 以 VM 延伸模組為基礎的資料來源 |
| performanceCounters | 適用于 Windows 和 Linux 的效能計數器 |
| syslog | Linux 上的 Syslog 事件 |
| windowsEventLogs | Windows 事件記錄檔 |


## <a name="limits"></a>限制
如需適用于每個資料收集規則的限制，請參閱 [Azure 監視器服務限制](../service-limits.md#data-collection-rules)。


## <a name="create-a-dcr"></a>建立 DCR
您目前可以使用下列任何一種方法來建立 DCR：

- [使用 Azure 入口網站](data-collection-rule-azure-monitor-agent.md) 建立資料集合規則，並將其與一或多部虛擬機器相關聯。
- 在 JSON 中直接編輯資料收集規則，並 [使用 REST API 提交](/rest/api/monitor/datacollectionrules)。
- 使用 [Azure CLI](https://github.com/Azure/azure-cli-extensions/blob/master/src/monitor-control-service/README.md)建立 DCR 和關聯。
- 使用 Azure PowerShell 建立 DCR 和關聯。
  - [AzDataCollectionRule](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/Get-AzDataCollectionRule.md)
  - [新 AzDataCollectionRule](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/New-AzDataCollectionRule.md)
  - [設定-AzDataCollectionRule](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/Set-AzDataCollectionRule.md)
  - [更新-AzDataCollectionRule](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/Update-AzDataCollectionRule.md)
  - [移除-AzDataCollectionRule](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/Remove-AzDataCollectionRule.md)
  - [AzDataCollectionRuleAssociation](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/Get-AzDataCollectionRuleAssociation.md)
  - [新 AzDataCollectionRuleAssociation](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/New-AzDataCollectionRuleAssociation.md)
  - [移除-AzDataCollectionRuleAssociation](https://github.com/Azure/azure-powershell/blob/master/src/Monitor/Monitor/help/Remove-AzDataCollectionRuleAssociation.md)

## <a name="sample-data-collection-rule"></a>範例資料收集規則
以下的範例資料收集規則適用于具有 Azure 管理代理程式的虛擬機器，且具有下列詳細資料：

- 效能資料
  - 每隔15秒會收集特定的處理器、記憶體、邏輯磁片和實體磁片計數器，並每分鐘上傳一次。
  - 每隔30秒會收集特定進程計數器，並每隔5分鐘上傳一次。
- Windows 事件
  - 收集 Windows 安全性事件，並每分鐘上傳。
  - 收集 Windows 應用程式和系統事件，並每隔5分鐘上傳一次。
- syslog
  - 從 cron 設備收集 Debug、Critical 和緊急事件。
  - 從 syslog 設備收集警示、重大和緊急事件。
- 目的地
  - 將所有資料傳送至名為 centralWorkspace 的 Log Analytics 工作區。

```json
{
    "location": "eastus",
    "properties": {
      "dataSources": {
        "performanceCounters": [
          {
            "name": "cloudTeamCoreCounters",
            "streams": [
              "Microsoft-Perf"
            ],
            "scheduledTransferPeriod": "PT1M",
            "samplingFrequencyInSeconds": 15,
            "counterSpecifiers": [
              "\\Processor(_Total)\\% Processor Time",
              "\\Memory\\Committed Bytes",
              "\\LogicalDisk(_Total)\\Free Megabytes",
              "\\PhysicalDisk(_Total)\\Avg. Disk Queue Length"
            ]
          },
          {
            "name": "appTeamExtraCounters",
            "streams": [
              "Microsoft-Perf"
            ],
            "scheduledTransferPeriod": "PT5M",
            "samplingFrequencyInSeconds": 30,
            "counterSpecifiers": [
              "\\Process(_Total)\\Thread Count"
            ]
          }
        ],
        "windowsEventLogs": [
          {
            "name": "cloudSecurityTeamEvents",
            "streams": [
              "Microsoft-WindowsEvent"
            ],
            "scheduledTransferPeriod": "PT1M",
            "xPathQueries": [
              "Security!*"
            ]
          },
          {
            "name": "appTeam1AppEvents",
            "streams": [
              "Microsoft-WindowsEvent"
            ],
            "scheduledTransferPeriod": "PT5M",
            "xPathQueries": [
              "System!*[System[(Level = 1 or Level = 2 or Level = 3)]]",
              "Application!*[System[(Level = 1 or Level = 2 or Level = 3)]]"
            ]
          }
        ],
        "syslog": [
          {
            "name": "cronSyslog",
            "streams": [
              "Microsoft-Syslog"
            ],
            "facilityNames": [
              "cron"
            ],
            "logLevels": [
              "Debug",
              "Critical",
              "Emergency"
            ]
          },
          {
            "name": "syslogBase",
            "streams": [
              "Microsoft-Syslog"
            ],
            "facilityNames": [
              "syslog"
            ],
            "logLevels": [
              "Alert",
              "Critical",
              "Emergency"
            ]
          }
        ]
      },
      "destinations": {
        "logAnalytics": [
          {
            "workspaceResourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.OperationalInsights/workspaces/my-workspace",
            "name": "centralWorkspace"
          }
        ]
      },
      "dataFlows": [
        {
          "streams": [
            "Microsoft-Perf",
            "Microsoft-Syslog",
            "Microsoft-WindowsEvent"
          ],
          "destinations": [
            "centralWorkspace"
          ]
        }
      ]
    }
  }
```


## <a name="next-steps"></a>後續步驟

- 使用 Azure 監視器代理程式，從虛擬機器[建立資料收集規則](data-collection-rule-azure-monitor-agent.md)和其關聯。