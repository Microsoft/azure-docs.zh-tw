---
title: 使用 Azure 監視器來監視 Data Factory | Microsoft Docs
description: 了解如何藉由啟用 Azure Data Factory 資訊的診斷記錄，以使用 Azure 監視器來監視 Data Factory 管線。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/22/2018
ms.author: shlo
ms.openlocfilehash: 2e8c5b3d9624d3a622f16d770f68bc8614993d36
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387477"
---
# <a name="alert-and-monitor-data-factories-using-azure-monitor"></a>使用 Azure 監視器提出警示及監視 Data Factory
雲端應用程式相當複雜，且具有許多移動組件。 監視會提供資料，以確保應用程式持續運作並以健全的狀態執行。 它也可協助您預防潛在問題，或是針對過去所發生的問題進行疑難排解。 除此之外，您還可以使用監視資料來取得應用程式的深入解析。 這項知識可協助您提升應用程式效能或維護性，或是將原本需要手動介入的動作自動化。

Azure 監視器可針對 Microsoft Azure 中的大多數服務提供基本等級的基礎結構計量與記錄。 如需詳細資訊，請參閱[監視概觀](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor)。 Azure 診斷記錄是資源所發出的記錄，會經常提供有關該資源之作業的豐富資料。 Data Factory 會在 Azure 監視器中輸出診斷記錄。

## <a name="persist-data-factory-data"></a>保存 Data Factory 資料
Data Factory 只會儲存管線執行資料 45 天。 如果您想要保存管線執行資料 45 天以上，可以使用 Azure 監視器；如此一來，您不僅可以路由診斷記錄以進行分析，還可以將這些資料保存在儲存體帳戶中，並擁有所選擇持續時間的處理站資訊。

## <a name="diagnostic-logs"></a>診斷記錄檔

* 將診斷記錄檔儲存到 **儲存體帳戶** 以利稽核或手動檢查。 您可以使用診斷設定來指定保留時間 (以天為單位)。
* 將診斷記錄串流至**事件中樞**，以供第三方服務或自訂的分析解決方案 (如 PowerBI) 擷取。
* 使用 **Log Analytics** 分析診斷記錄

您可以使用並非與發出記錄之資源位於同一個訂用帳戶的儲存體帳戶或事件中樞命名空間。 進行此設定的使用者必須具有這兩個訂用帳戶的適當角色型存取控制 (RBAC) 存取權。

## <a name="set-up-diagnostic-logs"></a>設定診斷記錄

### <a name="diagnostic-settings"></a>診斷設定
非計算資源的診斷記錄檔要使用診斷設定來設定。 資源的診斷設定會控制：

* 診斷記錄傳送至何處 (儲存體帳戶、事件中樞和/或 Log Analytics)。
* 傳送何種類別的記錄。
* 每個記錄類別應該在儲存體帳戶中保留多久
* 保留期為 0 天表示會永遠保留記錄。 否則，此值可以是 1 到 2147483647 之間的任意天數。
* 如果有設定保留原則，但將儲存體帳戶的記錄儲存停用 (例如，只選取事件中樞或 Log Analytics 選項)，保留原則不會有任何作用。
* 保留原則是每天套用，因此在一天結束時 (UTC)，這一天超過保留原則的記錄會被刪除。 例如，如果您的保留原則為一天，在今天一開始，昨天之前的記錄檔會被刪除。

### <a name="enable-diagnostic-logs-via-rest-apis"></a>透過 REST API 啟用診斷記錄

在 Azure 監視器 REST API 中建立或更新診斷設定

**要求**
```
PUT
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

**標頭**
* 將 `{api-version}` 取代為 `2016-09-01`。
* 將 `{resource-id}` 取代為您要編輯診斷設定之資源的資源識別碼。 如需詳細資訊，請參閱[使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-portal.md)。
* 將 `Content-Type` 標頭設定為 `application/json`。
* 將授權標頭設定為您從 Azure Active Directory 取得的 JSON Web 權杖。 如需詳細資訊，請參閱[驗證要求](../active-directory/develop/authentication-scenarios.md)。

**內文**
```json
{
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "metrics": [
        ],
        "logs": [
                {
                    "category": "PipelineRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "TriggerRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "ActivityRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                }
            ]
    },
    "location": ""
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| storageAccountId |字串 | 要作為診斷記錄傳送目的地之儲存體帳戶的資源識別碼 |
| serviceBusRuleId |字串 | 您為了串流診斷記錄而想要在其中建立事件中樞之服務匯流排命名空間的服務匯流排規則識別碼。 規則識別碼的格式為："{服務匯流排資源識別碼}/authorizationrules/{金鑰名稱}"。|
| workspaceId | 複雜類型 | 計量時間粒紋和其保留原則的陣列。 此屬性目前是空的。 |
|metrics| 要傳遞給已叫用之管線的管線執行參數值| 將參數名稱對應到引數值的 JSON 物件 |
| logs| 複雜類型| 資源類型之診斷記錄類別的名稱。 若要取得資源之診斷記錄類別的清單，請先執行 GET 診斷設定作業。 |
| category| 字串| 記錄類別和其保留原則的陣列 |
| timeGrain | 字串 | 以 ISO 8601 持續時間格式擷取的計量細微性。 必須是 PT1M (一分鐘)|
| 已啟用| BOOLEAN | 指定是否要為此資源啟用該計量或記錄類別的收集|
| retentionPolicy| 複雜類型| 描述計量或記錄類別的保留原則。 僅用於儲存體帳戶選項。|
| days| int| 計量或記錄的保留天數。 值為 0 會無限期地保留記錄。 僅用於儲存體帳戶選項。 |

**回應**

200 確定


```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```

在 Azure 監視器 REST API 中取得診斷設定的相關資訊

**要求**
```
GET
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

**標頭**
* 將 `{api-version}` 取代為 `2016-09-01`。
* 將 `{resource-id}` 取代為您要編輯診斷設定之資源的資源識別碼。 如需詳細資訊，請參閱＜使用資源群組管理您的 Azure 資源＞。
* 將 `Content-Type` 標頭設定為 `application/json`。
* 將授權標頭設定為您從 Azure Active Directory 取得的 JSON Web 權杖。 如需詳細資訊，請參閱＜驗證要求＞。

**回應**

200 確定

```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.Storage/storageAccounts/azmonlogs",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.EventHub/namespaces/shloeventhub/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/ADF/providers/Microsoft.OperationalInsights/workspaces/mihaipie",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```
[更多資訊](https://docs.microsoft.com/rest/api/monitor/diagnosticsettings)

## <a name="schema-of-logs--events"></a>記錄和事件的結構描述

### <a name="activity-run-logs-attributes"></a>活動執行記錄屬性

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "activityRunId":"",
   "pipelineRunId":"",
   "resourceId":"",
   "category":"ActivityRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "activityName":"",
   "start":"",
   "end":"",
   "properties:"
       {
          "Input": "{
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "BlobSink"
              }
           }",
          "Output": "{"dataRead":121,"dataWritten":121,"copyDuration":5,
               "throughput":0.0236328132,"errors":[]}",
          "Error": "{
              "errorCode": "null",
              "message": "null",
              "failureType": "null",
              "target": "CopyBlobtoBlob"
        }
    }
}
```

| 屬性 | 類型 | 說明 | 範例 |
| --- | --- | --- | --- |
| Level |字串 | 診斷記錄的層級。 層級 4 一律是活動執行記錄的情況。 | `4`  |
| correlationId |字串 | 用來完整追蹤特定要求的唯一識別碼 | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| 分析 | 字串 | 時間範圍內所發生之事件的時間 (UTC 格式) | `YYYY-MM-DDTHH:MM:SS.00000Z` | `2017-06-28T21:00:27.3534352Z` |
|activityRunId| 字串| 活動執行的識別碼 | `3a171e1f-b36e-4b80-8a54-5625394f4354` |
|pipelineRunId| 字串| 管線執行的識別碼 | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|ResourceId| 字串 | 資料處理站資源的相關聯資源識別碼 | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|category| 字串 | 診斷記錄的類別。 請將此屬性設定為 "ActivityRuns" | `ActivityRuns` |
|層級| 字串 | 診斷記錄的層級。 請將此屬性設定為 "Informational" | `Informational` |
|operationName| 字串 |具有狀態之活動的名稱。 如果狀態是啟動活動訊號，則此屬性是 `MyActivity -`。 如果狀態是結束活動訊號，則此屬性是具有最終狀態的 `MyActivity - Succeeded` | `MyActivity - Succeeded` |
|pipelineName| 字串 | 管線的名稱 | `MyPipeline` |
|activityName| 字串 | 活動的名稱 | `MyActivity` |
|start| 字串 | 時間範圍內之活動執行的開始時間 (UTC 格式) | `2017-06-26T20:55:29.5007959Z`|
|end| 字串 | 時間範圍內之活動執行的結束時間 (UTC 格式) 如果活動尚未結束 (活動開始的診斷記錄)，則會設定預設值 `1601-01-01T00:00:00Z`。  | `2017-06-26T20:55:29.5007959Z` |


### <a name="pipeline-run-logs-attributes"></a>管線執行記錄屬性

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "runId":"",
   "resourceId":"",
   "category":"PipelineRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "start":"",
   "end":"",
   "status":"",
   "properties":
    {
      "Parameters": {
        "<parameter1Name>": "<parameter1Value>"
      },
      "SystemParameters": {
        "ExecutionStart": "",
        "TriggerId": "",
        "SubscriptionId": ""
      }
    }
}
```

| 屬性 | 類型 | 說明 | 範例 |
| --- | --- | --- | --- |
| Level |字串 | 診斷記錄的層級。 層級 4 是活動執行記錄的情況。 | `4`  |
| correlationId |字串 | 用來完整追蹤特定要求的唯一識別碼 | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| 分析 | 字串 | 時間範圍內所發生之事件的時間 (UTC 格式) | `YYYY-MM-DDTHH:MM:SS.00000Z` | `2017-06-28T21:00:27.3534352Z` |
|runId| 字串| 管線執行的識別碼 | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|ResourceId| 字串 | 資料處理站資源的相關聯資源識別碼 | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|category| 字串 | 診斷記錄的類別。 請將此屬性設定為 "PipelineRuns" | `PipelineRuns` |
|層級| 字串 | 診斷記錄的層級。 請將此屬性設定為 "Informational" | `Informational` |
|operationName| 字串 |具有狀態之管線的名稱。 在管線執行完成時具有最終狀態的 "Pipeline - Succeeded"| `MyPipeline - Succeeded` |
|pipelineName| 字串 | 管線的名稱 | `MyPipeline` |
|start| 字串 | 時間範圍內之活動執行的開始時間 (UTC 格式) | `2017-06-26T20:55:29.5007959Z`|
|end| 字串 | 時間範圍內之活動執行的結束時間 (UTC 格式) 如果活動尚未結束 (活動開始的診斷記錄)，則會設定預設值 `1601-01-01T00:00:00Z`。  | `2017-06-26T20:55:29.5007959Z` |
|status| 字串 | 管線執行的最終狀態 (成功或失敗) | `Succeeded`|


### <a name="trigger-run-logs-attributes"></a>觸發程序執行記錄屬性

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "triggerId":"",
   "resourceId":"",
   "category":"TriggerRuns",
   "level":"Informational",
   "operationName":"",
   "triggerName":"",
   "triggerType":"",
   "triggerEvent":"",
   "start":"",
   "status":"",
   "properties":
   {
      "Parameters": {
        "TriggerTime": "",
       "ScheduleTime": ""
      },
      "SystemParameters": {}
    }
}

```

| 屬性 | 類型 | 說明 | 範例 |
| --- | --- | --- | --- |
| Level |字串 | 診斷記錄的層級。 若為活動執行記錄，則設定為層級 4。 | `4`  |
| correlationId |字串 | 用來完整追蹤特定要求的唯一識別碼 | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| 分析 | 字串 | 時間範圍內所發生之事件的時間 (UTC 格式) | `YYYY-MM-DDTHH:MM:SS.00000Z` | `2017-06-28T21:00:27.3534352Z` |
|triggerId| 字串| 觸發程序執行的識別碼 | `08587023010602533858661257311` |
|ResourceId| 字串 | 資料處理站資源的相關聯資源識別碼 | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|category| 字串 | 診斷記錄的類別。 請將此屬性設定為 "PipelineRuns" | `PipelineRuns` |
|層級| 字串 | 診斷記錄的層級。 請將此屬性設定為 "Informational" | `Informational` |
|operationName| 字串 |無論是否成功引發，具有最終狀態之觸發程序的名稱。 若活動訊號為成功時，則為 "MyTrigger - Succeeded"| `MyTrigger - Succeeded` |
|triggerName| 字串 | 觸發程序的名稱 | `MyTrigger` |
|triggerType| 字串 | 觸發程序的類型 (手動觸發程序或排程觸發程序) | `ScheduleTrigger` |
|triggerEvent| 字串 | 觸發程序的事件 | `ScheduleTime - 2017-07-06T01:50:25Z` |
|start| 字串 | 時間範圍內所引發之觸發程序的開始時間 (UTC 格式) | `2017-06-26T20:55:29.5007959Z`|
|status| 字串 | 無論是否成功引發觸發程序的最終狀態 (成功或失敗) | `Succeeded`|

## <a name="metrics"></a>度量

Azure 監視器可讓您取用遙測來查看您 Azure 工作負載的效能與健全狀況。 Azure 遙測資料最重要的類型是由大多數 Azure 資源所發出的度量 (也稱為效能計數器)。 Azure 監視器提供數種方式可設定及取用這些度量進行監視與疑難排解。

ADFV2 會發出下列計量

| **度量**           | **計量顯示名稱**         | **單位** | **彙總類型** | **說明**                                       |
|----------------------|---------------------------------|----------|----------------------|-------------------------------------------------------|
| PipelineSucceededRun | 成功的管線執行計量 | Count    | 總計                | 為期一分鐘的時間內成功的管線執行總計 |
| PipelineFailedRuns   | 失敗的管線執行計量    | Count    | 總計                | 為期一分鐘的時間內失敗的管線執行總計    |
| ActivitySucceededRuns | 成功的活動執行計量 | Count    | 總計                | 為期一分鐘的時間內成功的活動執行總計  |
| ActivityFailedRuns   | 失敗的活動執行計量    | Count    | 總計                | 為期一分鐘的時間內失敗的活動執行總計     |
| TriggerSucceededRuns | 成功的觸發程序執行計量  | Count    | 總計                | 為期一分鐘的時間內成功的觸發程序執行總計   |
| TriggerFailedRuns    | 失敗的觸發程序執行計量     | Count    | 總計                | 為期一分鐘的時間內失敗的觸發程序執行總計      |

若要存取計量，請遵循文章中的指示操作 - https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics

## <a name="monitor-data-factory-metrics-with-azure-monitor"></a>使用 Azure 監視器監視 Data Factory 計量

您可以使用 Azure Data Factory 與 Azure 監視器的整合，將資料路由傳送至 Azure 監視器。 此整合在下列案例中很實用 ：

1.  您想要對由 Data Factory 發佈至 Azure 監視器的一組豐富計量，撰寫複雜的查詢。 您也可以透過 Azure 監視器建立這些查詢的自訂警示。

2.  您想要監視所有資料處理站。 您可以將資料從多個資料處理站傳送至單一 Azure 監視器工作區。

如需此功能的 7 分鐘簡介與示範，請觀看下列影片：

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Monitor-Data-Factory-pipelines-using-Operations-Management-Suite-OMS/player]

### <a name="configure-diagnostic-settings-and-workspace"></a>設定診斷設定和工作區

啟用資料處理站的 [診斷設定]。

1.  選取 [Azure 監視器] -> [診斷設定] -> 選取資料處理站 -> 開啟診斷。

    ![monitor-oms-image1.png](media/data-factory-monitor-oms/monitor-oms-image1.png)

2.  提供診斷設定，包括工作區的組態。

    ![monitor-oms-image2.png](media/data-factory-monitor-oms/monitor-oms-image2.png)

### <a name="install-azure-data-factory-analytics-from-azure-marketplace"></a>從 Azure Marketplace 安裝 Azure Data Factory Analytics

![monitor-oms-image3.png](media/data-factory-monitor-oms/monitor-oms-image3.png)

![monitor-oms-image4.png](media/data-factory-monitor-oms/monitor-oms-image4.png)

按一下 [建立]，然後選取工作區和工作區設定。

![monitor-oms-image5.png](media/data-factory-monitor-oms/monitor-oms-image5.png)

### <a name="monitor-data-factory-metrics"></a>監視 Data Factory 計量

安裝 **Azure Data Factory Analytics**，可建立能啟用下列計量的預設檢視集：

- ADF 執行- 1) 依 Data Factory 的管線執行

- ADF 執行- 2) 依 Data Factory 的活動執行

- ADF 執行- 3) 依 Data Factory 的觸發程序執行

- ADF 錯誤- 1) 依 Data Factory 的前 10 個管線錯誤

- ADF 錯誤- 2) 依 Data Factory 的前 10 個活動執行

- ADF 錯誤- 3) 依 Data Factory 的前 10 個觸發程序執行

- ADF 統計資料- 1) 依類型的活動執行

- ADF 統計資料- 2) 依類型的觸發程序執行

- ADF 統計資料- 3) 最大管線執行持續時間

![monitor-oms-image6.png](media/data-factory-monitor-oms/monitor-oms-image6.png)

![monitor-oms-image7.png](media/data-factory-monitor-oms/monitor-oms-image7.png)

您可以將上述計量視覺化、查看這些計量背後的查詢、編輯查詢、建立警示等等。

![monitor-oms-image8.png](media/data-factory-monitor-oms/monitor-oms-image8.png)

## <a name="alerts"></a>警示

您可以針對 Data Factory 中支援的計量提出警示。 請在 Data Factory [監視] 頁面中按一下 [警示] 按鈕。

![警示選項](media/monitor-using-azure-monitor/alerts_image1.png)

帶您前往 [警示]。

![警示頁面](media/monitor-using-azure-monitor/alerts_image2.png)

您也可以登入 Azure 入口網站，然後按一下 [監視] -&gt; [警示] 來直接前往 [警示] 頁面。

![入口網站功能表中的警示](media/monitor-using-azure-monitor/alerts_image3.png)

### <a name="create-alerts"></a>建立警示

1.  按一下 [+ 新增警示規則] 來新建警示。

    ![新增警示規則](media/monitor-using-azure-monitor/alerts_image4.png)

2.  定義**警示條件**。

    > [!NOTE]
    > 請務必在 [依資源類型篩選] 中選取 [全部]。

    ![警示條件畫面 3 之 1](media/monitor-using-azure-monitor/alerts_image5.png)

    ![警示條件畫面 3 之 2](media/monitor-using-azure-monitor/alerts_image6.png)

    ![警示條件畫面 3 之 3](media/monitor-using-azure-monitor/alerts_image7.png)

3.  定義**警示詳細資料**。

    ![警示詳細資料](media/monitor-using-azure-monitor/alerts_image8.png)

4.  定義**動作群組**。

    ![動作群組畫面 4 之 1](media/monitor-using-azure-monitor/alerts_image9.png)

    ![動作群組畫面 4 之 2](media/monitor-using-azure-monitor/alerts_image10.png)

    ![動作群組畫面 4 之 3](media/monitor-using-azure-monitor/alerts_image11.png)

    ![動作群組畫面 4 之 4](media/monitor-using-azure-monitor/alerts_image12.png)

## <a name="next-steps"></a>後續步驟
請參閱[以程式設計方式監視和管理管線](monitor-programmatically.md)文章，以了解如何藉由執行來監視和管理管線。
