---
title: Azure 事件中樞診斷記錄 | Microsoft Docs
description: 了解如何為 Azure 中的事件中樞設定診斷記錄檔。
keywords: ''
documentationcenter: ''
services: event-hubs
author: ShubhaVijayasarathy
manager: ''
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-services
ms.date: 07/02/2018
ms.author: shvija
ms.openlocfilehash: d4ed53cf1f0d21e488631434b60309602169b59a
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40004645"
---
# <a name="event-hubs-diagnostic-logs"></a>事件中樞診斷記錄檔

您可以檢視 Azure 事件中樞的兩種記錄檔類型：

* **[活動記錄](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)**：這些記錄包含對作業執行的操作相關資訊。 系統一律會啟用這些記錄。
* **[診斷記錄](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)**：您可以設定診斷記錄，以深入檢視隨著作業發生的所有事件。 診斷記錄涵蓋從建立工作到刪除工作期間的活動，包括工作執行時發生的更新與活動。

## <a name="enable-diagnostic-logs"></a>啟用診斷記錄

診斷記錄預設為停用。 若要啟用診斷記錄，請依照下列步驟操作：

1.  在 [Azure 入口網站](https://portal.azure.com)的 [監視 + 管理] 下，按一下 [診斷記錄]。

    ![瀏覽到診斷記錄的窗格](./media/event-hubs-diagnostic-logs/image1.png)

2.  按一下您想要監視的資源。

3.  按一下 [開啟診斷]。

    ![開啟診斷記錄](./media/event-hubs-diagnostic-logs/image2.png)

4.  針對 [狀態]，按一下 [開啟]。

    ![變更診斷記錄檔的狀態](./media/event-hubs-diagnostic-logs/image3.png)

5.  設定您想要的封存目標，例如儲存體帳戶、事件中樞或 Azure Log Analytics。

6.  儲存新的診斷設定。

新的設定大約會在 10 分鐘內生效。 之後，記錄就會在 [診斷記錄] 窗格內的已設定封存目標中顯示。

如需設定診斷的詳細資訊，請參閱 [Azure 診斷記錄概觀](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)。

## <a name="diagnostic-logs-categories"></a>診斷記錄類別

事件中樞會擷取兩種類別的診斷記錄檔：

* **封存記錄**：與「事件中樞」封存相關的記錄，具體而言，就是與封存錯誤相關的記錄。
* **作業記錄**：與「事件中樞」作業期間發生的事件有關的資訊，具體而言，就是作業類型 (包括事件中樞的建立)、使用的資源及作業狀態。

## <a name="diagnostic-logs-schema"></a>診斷記錄結構描述

所有的記錄檔都會以 JavaScript 物件標記法 (JSON) 格式儲存。 每個項目都具有字串欄位，這些欄位會使用下列小節所述的格式。

### <a name="archive-logs-schema"></a>封存記錄檔結構描述

封存記錄檔 JSON 字串包括下表所列的元素：

Name | 描述
------- | -------
TaskName | 失敗工作的描述。
ActivityId | 用於追蹤的內部識別碼。
trackingId | 用於追蹤的內部識別碼。
ResourceId | Azure Resource Manager 資源識別碼。
eventHub | 事件中樞完整名稱 (包括命名空間名稱)。
partitionId | 要寫入的事件中樞資料分割。
archiveStep | ArchiveFlushWriter
startTime | 失敗開始時間。
failures | 發生失敗的次數。
durationInSeconds | 失敗持續時間。
Message | 錯誤訊息。
category | ArchiveLogs

下列程式碼是封存記錄 JSON 字串的範例：

```json
{
   "TaskName": "EventHubArchiveUserError",
   "ActivityId": "21b89a0b-8095-471a-9db8-d151d74ecf26",
   "trackingId": "21b89a0b-8095-471a-9db8-d151d74ecf26_B7",
   "resourceId": "/SUBSCRIPTIONS/854D368F-1828-428F-8F3C-F2AFFA9B2F7D/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
   "eventHub": "fbettati-opera-eventhub:eventhub:eh123~32766",
   "partitionId": "1",
   "archiveStep": "ArchiveFlushWriter",
   "startTime": "9/22/2016 5:11:21 AM",
   "failures": 3,
   "durationInSeconds": 360,
   "message": "Microsoft.WindowsAzure.Storage.StorageException: The remote server returned an error: (404) Not Found. ---> System.Net.WebException: The remote server returned an error: (404) Not Found.\r\n   at Microsoft.WindowsAzure.Storage.Shared.Protocol.HttpResponseParsers.ProcessExpectedStatusCodeNoException[T](HttpStatusCode expectedStatusCode, HttpStatusCode actualStatusCode, T retVal, StorageCommandBase`1 cmd, Exception ex)\r\n   at Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob.<PutBlockImpl>b__3e(RESTCommand`1 cmd, HttpWebResponse resp, Exception ex, OperationContext ctx)\r\n   at Microsoft.WindowsAzure.Storage.Core.Executor.Executor.EndGetResponse[T](IAsyncResult getResponseResult)\r\n   --- End of inner exception stack trace ---\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.StorageAsyncResult`1.End()\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.AsyncExtensions.<>c__DisplayClass4.<CreateCallbackVoid>b__3(IAsyncResult ar)\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.",
   "category": "ArchiveLogs"
}
```

### <a name="operational-logs-schema"></a>作業記錄結構描述

作業記錄 JSON 字串包括下表所列的元素：

Name | 說明
------- | -------
ActivityId | 用於追蹤目的的內部識別碼。
EventName | 作業名稱。  
ResourceId | Azure Resource Manager 資源識別碼。
SubscriptionId | 訂用帳戶 ID。
EventTimeString | 作業時間。
EventProperties | 作業屬性。
狀態 | 作業狀態。
呼叫者 | 作業呼叫者 (Azure 入口網站或管理用戶端)。
category | OperationalLogs

下列程式碼是作業記錄 JSON 字串的範例：

```json
Example:
{
   "ActivityId": "6aa994ac-b56e-4292-8448-0767a5657cc7",
   "EventName": "Create EventHub",
   "resourceId": "/SUBSCRIPTIONS/1A2109E3-9DA0-455B-B937-E35E36C1163C/RESOURCEGROUPS/DEFAULT-SERVICEBUS-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/SHOEBOXEHNS-CY4001",
   "SubscriptionId": "1a2109e3-9da0-455b-b937-e35e36c1163c",
   "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
   "EventProperties": "{\"SubscriptionId\":\"1a2109e3-9da0-455b-b937-e35e36c1163c\",\"Namespace\":\"shoeboxehns-cy4001\",\"Via\":\"https://shoeboxehns-cy4001.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
   "Status": "Succeeded",
   "Caller": "ServiceBus Client",
   "category": "OperationalLogs"
}
```

## <a name="next-steps"></a>後續步驟
* [事件中樞簡介](event-hubs-what-is-event-hubs.md)
* [事件中樞 API 概觀](event-hubs-api-overview.md)
* [開始使用事件中心](event-hubs-dotnet-standard-getstarted-send.md)
