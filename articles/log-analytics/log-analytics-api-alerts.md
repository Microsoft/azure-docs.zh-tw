---
title: 使用 OMS Log Analytics 警示 REST API
description: Log Analytics 警示 REST API 可讓您在 Log Analytics (其為 Operations Management Suite (OMS) 的一部分) 中建立及管理警示。  本文提供此 API 的詳細資料和幾個執行不同作業的範例。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 628ad256-7181-4a0d-9e68-4ed60c0f3f04
ms.service: log-analytics
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/10/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 7f55b762bda5ff0c7bbedf414b18465656496cbb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984580"
---
# <a name="create-and-manage-alert-rules-in-log-analytics-with-rest-api"></a>使用 REST API 在 Log Analytics 中建立及管理警示規則
Log Analytics 警示 REST API 可讓您在 Operations Management Suite (OMS) 中建立及管理警示。  本文提供此 API 的詳細資料和幾個執行不同作業的範例。

Log Analytics 搜尋 API 是 RESTful，可透過 Azure Resource Manager REST API 來存取。 在這份文件中，您會找到從 PowerShell 命令列使用 [ARMClient](https://github.com/projectkudu/ARMClient) (一個可簡化 Azure Resource Manager API 叫用流程的開放原始碼命令列工具) 來存取 API 的範例。 使用 ARMClient 和 PowerShell 是存取 Log Analytics 搜尋 API 的許多選項之一。 透過這些工具，您可以利用 RESTful Azure Resource Manager API 來呼叫 OMS 工作區，並在其中執行搜尋命令。 API 會以 JSON 格式向您輸出搜尋結果，讓您以程式設計方式透過許多不同的方法使用搜尋結果。

## <a name="prerequisites"></a>必要條件
目前，在 Log Analytics 中只能使用已儲存的搜尋來建立警示。  如需詳細資訊，請參閱 [記錄檔搜尋 REST API](log-analytics-log-search-api.md) 。

## <a name="schedules"></a>排程
一個已儲存的搜尋可以有一或多個排程。 排程中定義搜尋的執行頻率及識別準則的時間間隔。
排程具有下表中的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| 間隔 |執行搜尋的頻率。 以分鐘為單位。 |
| QueryTimeSpan |準則評估的時間間隔。 必須等於或大於 Interval。 以分鐘為單位。 |
| 版本 |所使用的 API 版本。  目前，這應該一律設為 1。 |

例如，假設事件查詢的 Interval 是 15 分鐘，而 Timespan 是 30 分鐘。 在此情況下，將會每隔 15 分鐘執行一次查詢，而如果準則在 30 分鐘內連續評估為 true，就會觸發警示。

### <a name="retrieving-schedules"></a>擷取排程
使用 Get 方法來擷取已儲存的搜尋的所有排程。

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules?api-version=2015-03-20

使用 Get 方法並指定排程識別碼，以擷取已儲存的搜尋的特定排程。

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20

以下是排程的回應範例。

```json
{
    "value": [{
        "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/MyWorkspace/savedSearches/0f0f4853-17f8-4ed1-9a03-8e888b0d16ec/schedules/a17b53ef-bd70-4ca4-9ead-83b00f2024a8",
        "etag": "W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\"",
        "properties": {
            "Interval": 15,
            "QueryTimeSpan": 15
        }
    }]
}
```

### <a name="creating-a-schedule"></a>建立排程
使用 Put 方法並指定唯一的排程識別碼，以建立新的排程。  請注意，兩個排程即使其已儲存的相關聯搜尋不同，也不能有相同的識別碼。  當您在 OMS 主控台中建立排程時，將會建立 GUID 做為排程識別碼。

> [!NOTE]
> Log Analytics API 所建立並儲存的所有搜尋、排程和動作，都必須使用小寫名稱。

    $scheduleJson = "{'properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Active':'true' } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson

### <a name="editing-a-schedule"></a>編輯排程
針對已儲存的相同搜尋，使用 Put 方法並指定現有的排程識別碼，以修改該排程。  要求的主體必須包含排程的 etag。

      $scheduleJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\""','properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Active':'true' } }"
      armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson


### <a name="deleting-schedules"></a>刪除排程
使用 Delete 方法並指定排程識別碼來刪除排程。

    armclient delete /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20


## <a name="actions"></a>動作
一個排程可以有多個動作。 一個動作可能定義一或多個處理序來執行，例如傳送郵件或啟動 Runbook，或也可能定義臨界值來判斷搜尋結果是否符合某些準則。  某些動作會同時定義這兩者，以便符合臨界值時執行處理序。

所有動作具有下表中的屬性。  不同類型的警示有不同的額外屬性，如下所述。

| 屬性 | 說明 |
|:--- |:--- |
| 類型 |動作的類型。  目前可能的值為 Alert 和 Webhook。 |
| 名稱 |警示的顯示名稱。 |
| 版本 |所使用的 API 版本。  目前，這應該一律設為 1。 |

### <a name="retrieving-actions"></a>擷取動作

> [!NOTE]
> 從 2018 年 5 月 14 日開始，Log Analytics 工作區的 Azure 公用雲端執行個體內所有警示都將自動延伸至 Azure。 在 2018 年 5 月 14 日之前，使用者可以主動起始將警示延伸至 Azure。 如需詳細資訊，請參閱[將警示從 OMS 延伸至 Azure](../monitoring-and-diagnostics/monitoring-alerts-extend.md)。 針對將警示延伸至 Azure 的使用者，現在便會以 Azure 動作群組來控制動作。 將工作區及其警示延伸至 Azure 之後，您便可以使用 [Azure 群組 API](https://docs.microsoft.com/rest/api/monitor/actiongroups) 來擷取或新增動作。

使用 Get 方法來擷取排程的所有動作。

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules/{Schedule ID}/actions?api-version=2015-03-20

使用 Put 方法並指定動作識別碼，擷取排程的特定動作。

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/actions/{Action ID}?api-version=2015-03-20

### <a name="creating-or-editing-actions"></a>建立或編輯動作
使用 Put 方法並指定排程特有的動作識別碼，以建立新的動作。  當您在 OMS 主控台中建立動作時，動作識別碼會使用 GUID。

> [!NOTE]
> Log Analytics API 所建立並儲存的所有搜尋、排程和動作，都必須使用小寫名稱。

針對已儲存的相同搜尋，使用 Put 方法並指定現有的動作識別碼，以修改該排程。  要求的主體必須包含排程的 etag。

建立新動作的要求格式依動作類型而不同，下列各節提供這些範例。

### <a name="deleting-actions"></a>刪除動作

> [!NOTE]
> 從 2018 年 5 月 14 日開始，Log Analytics 工作區的 Azure 公用雲端執行個體內所有警示都將自動延伸至 Azure。 在 2018 年 5 月 14 日之前，使用者可以主動起始將警示延伸至 Azure。 如需詳細資訊，請參閱[將警示從 OMS 延伸至 Azure](../monitoring-and-diagnostics/monitoring-alerts-extend.md)。 針對將警示延伸至 Azure 的使用者，現在便會以 Azure 動作群組來控制動作。 將工作區及其警示延伸至 Azure 之後，您便可以使用 [Azure 群組 API](https://docs.microsoft.com/rest/api/monitor/actiongroups) 來擷取或新增動作。

使用 Delete 方法並指定動作識別碼來刪除動作。

    armclient delete /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/Actions/{Action ID}?api-version=2015-03-20

### <a name="alert-actions"></a>警示動作
一個排程應該只有一個警示動作。  警示動作具有下表中的一或多個區段。  以下進一步詳細說明每一個區段。

| 區段 | 說明 | 使用量 |
|:--- |:--- |:--- |
| 閾值 |執行動作的準則。| 將警示延伸至 Azure 之前或之後，都必須為每個警示指定。 |
| 嚴重性 |用來在觸發時將警示分類的標籤。| 將警示延伸至 Azure 之前或之後，都必須為每個警示指定。 |
| 動作群組 |已指定所需動作之 Azure ActionGroup 的識別碼，例如 - 電子郵件、簡訊、語音通話、Webhook、自動化 Runbook、ITSM 連接器等。| 將警示延伸至 Azure 之後，就必須指定|
| 自訂動作|修改來自 ActionGroup 之所選動作的標準輸出| 就每個警示而言為選擇性，可在將警示延伸至 Azure 之後使用。 |
| EmailNotification |將郵件傳送給多位收件者。 | 如果將警示延伸至 Azure，則不須指定|
| 補救 |在 Azure 自動化中啟動 Runbook，以嘗試更正識別的問題。 |如果將警示延伸至 Azure，則不須指定|
| Webhook 動作 | 將來自警示的資料以 JSON 形式推送至所需的服務 |如果將警示延伸至 Azure，則不須指定|

> [!NOTE]
> 從 2018 年 5 月 14 日開始，Log Analytics 工作區的 Azure 公用雲端執行個體內所有警示都將自動延伸至 Azure。 在 2018 年 5 月 14 日之前，使用者可以主動起始將警示延伸至 Azure。 如需詳細資訊，請參閱[將警示從 OMS 延伸至 Azure](../monitoring-and-diagnostics/monitoring-alerts-extend.md)。

#### <a name="thresholds"></a>臨界值
一個警示動作應該只有一個臨界值。  當已儲存的搜尋結果符合與該搜尋相關聯動作中的臨界值時，將會執行該動作中的任何其他處理序。  動作也可以只包含臨界值，以便與不包含臨界值的其他類型動作一起搭配使用。

臨界值具有下表中的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| 運算子 |用於比較臨界值的運算子。 <br> gt = 大於 <br> lt = 小於 |
| 值 |臨界值。 |

例如，假設事件查詢的間隔是 15 分鐘、時間範圍是 30 分鐘，而臨界值大於 10。 在此情況下，將會每隔 15 分鐘執行一次查詢，而如果傳回 10 個在 30 分鐘內建立的事件，就會觸發警示。

以下是一個只包含臨界值的動作的回應範例。  

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My threshold action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "Version": 1
    }

使用 Put 方法並指定唯一的動作識別碼，以建立排程的新臨界值動作。  

    $thresholdJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson

使用 Put 方法並指定現有的動作識別碼，以修改排程的臨界值動作。  要求的主體必須包含動作的 etag。

    $thresholdJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson

#### <a name="severity"></a>嚴重性
Log Analytics 可讓您將警示分類成數個類別，以便於管理和分級。 已定義的警示嚴重性包括：資訊、警告及嚴重。 這些會以下列方式對應至「Azure 警示」的標準化嚴重性級別：

|Log Analytics 嚴重性等級  |Azure 警示嚴重性等級  |
|---------|---------|
|重要 |Sev 0|
|警告 |Sev 1|
|資訊 | Sev 2|

以下是一個只包含臨界值和嚴重性的動作範例回應。 

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My threshold action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "Severity": "critical",
        "Version": 1    }

使用 Put 方法搭配唯一動作識別碼，來為排程建立具有嚴重性的新動作。  

    $thresholdWithSevJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson

使用 Put 方法搭配現有的動作識別碼，以修改排程的嚴重性動作。  要求的主體必須包含動作的 etag。

    $thresholdWithSevJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson

#### <a name="action-groups"></a>動作群組
Azure 中的所有警示都使用「動作群組」作為處理動作的預設機制。 使用「動作群組」時，您可以指定您的動作一次，然後將動作群組與整個 Azure 中的多個警示建立關聯。 這樣就不需要一再地重複宣告相同的動作。 「動作群組」支援多個動作 - 包括電子郵件、、SMS、語音通話、ITSM 連線、自動化 Runbook、Webhook URI 等。 

針對將警示延伸至 Azure 的使用者 - 排程的「動作群組」詳細資料現在應該與閾值一起傳遞，才能建立警示。 必須先在「動作群組」內定義電子郵件詳細資料、Webhook URL、「Runbook 自動化」詳細資料及其他動作，才能建立警示；使用者可以在入口網站中[從 Azure 監視器建立動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)，或使用[動作群組 API](https://docs.microsoft.com/rest/api/monitor/actiongroups) 來建立動作群組。

若要將動作群組關聯新增至警示，請在警示定義中指定該動作群組的唯一 Azure Resource Manager 識別碼。 以下提供範例說明：

     "etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
      "properties": {
        "Type": "Alert",
        "Name": "test-alert",
        "Description": "I need to put a descriptio here",
        "Threshold": {
          "Operator": "gt",
          "Value": 12
        },
        "AzNsNotification": {
          "GroupIds": [
            "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
          ]
        },
        "Severity": "critical",
        "Version": 1
      },

使用 Put 方法搭配唯一動作識別碼，來為排程關聯已經存在的「動作群組」。  以下是使用方式的範例說明。

    $AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

使用 Put 方法搭配現有的動作識別碼，來為排程修改已關聯的「動作群組」。  要求的主體必須包含動作的 etag。

    $AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

#### <a name="customize-actions"></a>自訂動作
動作預設會依循通知的標準範本和格式。 但是使用者可以自訂一些動作，即使這些動作是由「動作群組」所控制。 目前，可以自訂「電子郵件主旨」和「Webhook 承載」。

##### <a name="customize-e-mail-subject-for-action-group"></a>自訂動作群組的電子郵件主旨
警示的電子郵件主旨預設為：<WorkspaceName> 的警示通知 <AlertName>。 但是您可以自訂此主旨，以便使用特定的文字或標籤 - 這樣可讓您在「收件匣」中輕鬆採用篩選規則。 自訂電子郵件標頭詳細資料必須一併傳送 ActionGroup 詳細資料，如以下範例所示。

     "etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
      "properties": {
        "Type": "Alert",
        "Name": "test-alert",
        "Description": "I need to put a descriptio here",
        "Threshold": {
          "Operator": "gt",
          "Value": 12
        },
        "AzNsNotification": {
          "GroupIds": [
            "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
          ],
          "CustomEmailSubject": "Azure Alert fired"
        },
        "Severity": "critical",
        "Version": 1
      },

使用 Put 方法搭配唯一動作識別碼，來為排程關聯含有自訂值的已存在「動作群組」。  以下是使用方式的範例說明。

    $AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

使用 Put 方法搭配現有的動作識別碼，來為排程修改已關聯的「動作群組」。  要求的主體必須包含動作的 etag。

    $AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired' }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

##### <a name="customize-webhook-payload-for-action-group"></a>自訂動作群組的 Webhook 承載
根據預設，透過「動作群組」傳送來進行記錄分析的 Webhook 會具有固定的結構。 但是使用者可以藉由使用所支援的特定變數來自訂 JSON 承載，以符合 Webhook 端點的需求。 如需詳細資訊，請參閱[記錄警示規則的 Webhook 動作](../monitoring-and-diagnostics/monitor-alerts-unified-log-webhook.md)。 

自訂 Webhook 詳細資料必須一併傳送 ActionGroup 詳細資料，並且將會套用至動作群組內所指定的所有 Webhook URI；如以下範例所示。

     "etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
      "properties": {
        "Type": "Alert",
        "Name": "test-alert",
        "Description": "I need to put a descriptio here",
        "Threshold": {
          "Operator": "gt",
          "Value": 12
        },
        "AzNsNotification": {
          "GroupIds": [
            "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
          ],
          "CustomWebhookPayload": "{\"field1\":\"value1\",\"field2\":\"value2\"}",
          "CustomEmailSubject": "Azure Alert fired"
        },
        "Severity": "critical",
        "Version": 1
      },

使用 Put 方法搭配唯一動作識別碼，來為排程關聯含有自訂值的已存在「動作群組」。  以下是使用方式的範例說明。

    $AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

使用 Put 方法搭配現有的動作識別碼，來為排程修改已關聯的「動作群組」。  要求的主體必須包含動作的 etag。

    $AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}' }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

#### <a name="email-notification"></a>電子郵件通知
「電子郵件通知」會將郵件傳送給一或多位收件者。  它們包含下表中的屬性。

> [!NOTE]
> 從 2018 年 5 月 14 日開始，Log Analytics 工作區的 Azure 公用雲端執行個體內所有警示都將自動延伸至 Azure。 在 2018 年 5 月 14 日之前，使用者可以主動起始將警示延伸至 Azure。 如需詳細資訊，請參閱[將警示從 OMS 延伸至 Azure](../monitoring-and-diagnostics/monitoring-alerts-extend.md)。 針對將警示延伸至 Azure 的使用者，現在便會以 Azure 動作群組來控制動作 (例如「電子郵件通知」)。 將工作區及其警示延伸至 Azure 之後，您便可以使用 [Azure 群組 API](https://docs.microsoft.com/rest/api/monitor/actiongroups) 來擷取或新增動作。
   

| 屬性 | 說明 |
|:--- |:--- |
| 收件者 |郵件地址清單。 |
| 主體 |郵件的主旨。 |
| 附件 |目前不支援附件，因此這個屬性的值一律為 “None”。 |

以下是一個包含臨界值的電子郵件通知動作的回應範例。  

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My email action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "EmailNotification": {
            "Recipients": [
                "recipient1@contoso.com",
                "recipient2@contoso.com"
            ],
            "Subject": "This is the subject",
            "Attachment": "None"
        },
        "Version": 1
    }

使用 Put 方法並指定唯一的動作識別碼，以建立排程的新電子郵件動作。  下列範例建立一個包含臨界值的電子郵件通知，當已儲存的搜尋結果超過臨界值時，就會傳送郵件。

    $emailJson = "{'properties': { 'Name': 'MyEmailAction', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'EmailNotification': {'Recipients': ['recipient1@contoso.com', 'recipient2@contoso.com'], 'Subject':'This is the subject', 'Attachment':'None'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myemailaction?api-version=2015-03-20 $emailJson

使用 Put 方法並指定現有的動作識別碼，以修改排程的電子郵件動作。  要求的主體必須包含動作的 etag。

    $emailJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'MyEmailAction', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'EmailNotification': {'Recipients': ['recipient1@contoso.com', 'recipient2@contoso.com'], 'Subject':'This is the subject', 'Attachment':'None'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myemailaction?api-version=2015-03-20 $emailJson

#### <a name="remediation-actions"></a>補救動作
「補救」會在 Azure 自動化中啟動 Runbook，嘗試更正警示所識別的問題。  您必須為補救動作中使用的 Runbook 建立 webhook，然後在 WebhookUri 屬性中指定 URI。  當您使用 OMS 主控台建立此動作時，將會自動為 Runbook 建立新的 webhook。

> [!NOTE]
> 從 2018 年 5 月 14 日開始，Log Analytics 工作區的 Azure 公用雲端執行個體內所有警示都將自動延伸至 Azure。 在 2018 年 5 月 14 日之前，使用者可以主動起始將警示延伸至 Azure。 如需詳細資訊，請參閱[將警示從 OMS 延伸至 Azure](../monitoring-and-diagnostics/monitoring-alerts-extend.md)。 針對將警示延伸至 Azure 的使用者，現在便會以 Azure 動作群組來控制動作 (例如使用 Runbook 的「補救」)。 將工作區及其警示延伸至 Azure 之後，您便可以使用 [Azure 群組 API](https://docs.microsoft.com/rest/api/monitor/actiongroups) 來擷取或新增動作。

「補救」包含下表中的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| RunbookName |Runbook 的名稱。 這必須符合自動化帳戶 (在 OMS 工作區的自動化方案中設定) 中發佈的 Runbook。 |
| WebhookUri |Webhook 的 URI。 |
| Expiry |webhook 的到期日期和時間。  如果 webhook 沒有到期日，這可以是任何有效的未來日期。 |

以下是一個包含臨界值的補救動作的回應範例。

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My remediation action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "Remediation": {
            "RunbookName": "My-Runbook",
            "WebhookUri": "https://s1events.azure-automation.net/webhooks?token=4jCibOjO3w4W2Cfg%2b2NkjLYdafnusaG6i8tnP8h%2fNNg%3d",
            "Expiry": "2018-02-25T18:27:20"
            },
        "Version": 1
    }

使用 Put 方法並指定唯一的動作識別碼，以建立排程的新補救動作。  下列範例建立一個包含臨界值的補救，當已儲存的搜尋結果超過臨界值時，就會啟動 Runbook。

    $remediateJson = "{'properties': { 'Type':'Alert', 'Name': 'My Remediation Action', 'Version':'1', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'Remediation': {'RunbookName': 'My-Runbook', 'WebhookUri':'https://s1events.azure-automation.net/webhooks?token=4jCibOjO3w4W2Cfg%2b2NkjLYdafnusaG6i8tnP8h%2fNNg%3d', 'Expiry':'2018-02-25T18:27:20Z'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myremediationaction?api-version=2015-03-20 $remediateJson

使用 Put 方法並指定現有的動作識別碼，以修改排程的補救動作。  要求的主體必須包含動作的 etag。

    $remediateJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Type':'Alert', 'Name': 'My Remediation Action', 'Version':'1', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'Remediation': {'RunbookName': 'My-Runbook', 'WebhookUri':'https://s1events.azure-automation.net/webhooks?token=4jCibOjO3w4W2Cfg%2b2NkjLYdafnusaG6i8tnP8h%2fNNg%3d', 'Expiry':'2018-02-25T18:27:20Z'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myremediationaction?api-version=2015-03-20 $remediateJson

#### <a name="example"></a>範例
以下是建立新電子郵件警示的完整範例。  這會建立新的排程及一個包含臨界值和電子郵件的動作。

    $subscriptionId = "3d56705e-5b26-5bcc-9368-dbc8d2fafbfc"
    $resourceGroup  = "MyResourceGroup"    
    $workspaceName    = "MyWorkspace"
    $searchId       = "MySearch"
    $scheduleId     = "MySchedule"
    $thresholdId    = "MyThreshold"
    $actionId       = "MyEmailAction"

    $scheduleJson = "{'properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Active':'true' }"
    armclient put /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.OperationalInsights/workspaces/$workspaceName/savedSearches/$searchId/schedules/$scheduleId/?api-version=2015-03-20 $scheduleJson

    $emailJson = "{'properties': { 'Name': 'MyEmailAction', 'Version':'1', 'Severity':'Warning', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'EmailNotification': {'Recipients': ['recipient1@contoso.com', 'recipient2@contoso.com'], 'Subject':'This is the subject', 'Attachment':'None'} }"
    armclient put /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.OperationalInsights/workspaces/$workspaceName/savedSearches/$searchId/schedules/$scheduleId/actions/$actionId/?api-version=2015-03-20 $emailJson

#### <a name="webhook-actions"></a>Webhook 動作
Webhook 動作會呼叫 URL 並選擇性地提供要傳送的承載，以啟動處理序。  這些動作類似於「補救」動作，不同之處在於它們用於可能叫用 Azure 自動化 Runbook 以外之處理序的 webhook。  它們還提供另一個選項，可指定要傳遞到遠端處理序的承載。

> [!NOTE]
> 從 2018 年 5 月 14 日開始，Log Analytics 工作區的 Azure 公用雲端執行個體內所有警示都將自動延伸至 Azure。 在 2018 年 5 月 14 日之前，使用者可以主動起始將警示延伸至 Azure。 如需詳細資訊，請參閱[將警示從 OMS 延伸至 Azure](../monitoring-and-diagnostics/monitoring-alerts-extend.md)。 針對將警示延伸至 Azure 的使用者，現在便會以 Azure 動作群組來控制動作 (例如 Webhook)。 將工作區及其警示延伸至 Azure 之後，您便可以使用 [Azure 群組 API](https://docs.microsoft.com/rest/api/monitor/actiongroups) 來擷取或新增動作。


Webhook 動作沒有臨界值，應該加入至具有警示動作和臨界值的排程。  

以下是 webhook 動作及一個包含臨界值的相關聯警示動作的回應範例。

    {
        "__metadata": {},
        "value": [
            {
                "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/bwren/savedSearches/2d1b30fb-7f48-4de5-9614-79ee244b52de/schedules/b80f5621-7217-4007-b32d-165d14377093/Actions/72884702-acf9-4653-bb67-f42436b342b4",
                "etag": "W/\"datetime'2016-02-26T20%3A25%3A00.6862124Z'\"",
                "properties": {
                    "Type": "Webhook",
                    "Name": "My Webhook Action",
                    "WebhookUri": "https://oaaswebhookdf.cloudapp.net/webhooks?token=VfkYTIlpk%2fc%2bJBP",
                    "CustomPayload": "{\"fielld1\":\"value1\",\"field2\":\"value2\"}",
                    "Version": 1
                }
            },
            {
                "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/bwren/savedSearches/2d1b30fb-7f48-4de5-9614-79ee244b52de/schedules/b80f5621-7217-4007-b32d-165d14377093/Actions/90a27cf8-71b7-4df2-b04f-54ed01f1e4b6",
                "etag": "W/\"datetime'2016-02-26T20%3A25%3A00.565204Z'\"",
                "properties": {
                    "Type": "Alert",
                    "Name": "Threshold for my webhook action",
                    "Threshold": {
                        "Operator": "gt",
                        "Value": 10
                    },
                    "Version": 1
                }
            }
        ]
    }

##### <a name="create-or-edit-a-webhook-action"></a>建立或編輯 webhook 動作
使用 Put 方法並指定唯一的動作識別碼，以建立排程的新 webhook 動作。  下列範例建立 Webhook 動作及一個包含臨界值的警示動作，當已儲存的搜尋結果超過臨界值時，就會啟動 webhook。

    $thresholdAction = "{'properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdAction

    $webhookAction = "{'properties': {'Type': 'Webhook', 'Name': 'My Webhook", 'WebhookUri': 'https://oaaswebhookdf.cloudapp.net/webhooks?token=VrkYTKlhk%2fc%2bKBP', 'CustomPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}', 'Version': 1 }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mywebhookaction?api-version=2015-03-20 $webhookAction

使用 Put 方法並指定現有的動作識別碼，以修改排程的 webhook 動作。  要求的主體必須包含動作的 etag。

    $webhookAction = "{'etag': 'W/\"datetime'2016-02-26T20%3A25%3A00.6862124Z'\"','properties': {'Type': 'Webhook', 'Name': 'My Webhook", 'WebhookUri': 'https://oaaswebhookdf.cloudapp.net/webhooks?token=VrkYTKlhk%2fc%2bKBP', 'CustomPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}', 'Version': 1 }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mywebhookaction?api-version=2015-03-20 $webhookAction


## <a name="next-steps"></a>後續步驟
* 在 Log Analytics 中使用 [REST API 執行記錄檔搜尋](log-analytics-log-search-api.md) 。
* 了解 [Azure 警示中的記錄警示](../monitoring-and-diagnostics/monitor-alerts-unified-log.md)

