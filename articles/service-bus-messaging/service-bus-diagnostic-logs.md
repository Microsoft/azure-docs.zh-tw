---
title: Azure 服務匯流排診斷記錄 | Microsoft Docs
description: 了解如何為 Azure 中的「服務匯流排」設定診斷記錄。
keywords: ''
documentationcenter: .net
services: service-bus-messaging
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-services
ms.date: 02/05/2018
ms.author: spelluru
ms.openlocfilehash: 3c2528634dea5c75e4a0e35b7e1a6a30de8d96c1
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696151"
---
# <a name="service-bus-diagnostic-logs"></a>服務匯流排診斷記錄

您可以檢視「Azure 服務匯流排」的兩種記錄：
* **[活動記錄檔](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)**。 這些記錄包含在工作上執行之操作的相關資訊。 系統一律會啟用這些記錄。
* **[診斷記錄](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)**。 您可以設定診斷記錄，以取得在工作內所發生之所有事件的更詳細資訊。 診斷記錄涵蓋從建立工作到刪除工作期間的活動，包括工作執行時發生的更新與活動。

## <a name="turn-on-diagnostic-logs"></a>開啟診斷記錄

診斷記錄預設為停用。 若要啟用診斷記錄，請執行下列步驟：

1.  在 [Azure 入口網站](https://portal.azure.com)的 [監視 + 管理] 下，按一下 [診斷記錄]。

    ![瀏覽到診斷記錄的刀鋒視窗](./media/service-bus-diagnostic-logs/image1.png)

2. 按一下您想要監視的資源。  

3.  按一下 [開啟診斷]。

    ![開啟診斷記錄](./media/service-bus-diagnostic-logs/image2.png)

4.  針對 [狀態]，按一下 [開啟]。

    ![變更診斷記錄狀態](./media/service-bus-diagnostic-logs/image3.png)

5.  設定您想要的封存目標，例如儲存體帳戶、事件中樞或 Azure Log Analytics。

6.  儲存新的診斷設定。

新的設定大約會在 10 分鐘內生效。 之後，記錄就會顯示在 [診斷記錄] 刀鋒視窗上已設定的封存目標中。

如需設定診斷的詳細資訊，請參閱 [Azure 診斷記錄概觀](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)。

## <a name="diagnostic-logs-schema"></a>診斷記錄結構描述

所有的記錄檔都會以 JavaScript 物件標記法 (JSON) 格式儲存。 每個項目都具有字串欄位，這些欄位會使用下列小節所述的格式。

## <a name="operational-logs-schema"></a>作業記錄結構描述

**OperationalLogs** 類別中的記錄會擷取「服務匯流排」作業期間發生的事項。 具體而言，這些記錄會擷取作業類型，包括建立佇列、使用的資源，以及作業狀態。

作業記錄 JSON 字串包括下表所列的元素：

名稱 | 說明
------- | -------
ActivityId | 用於追蹤的內部識別碼
EventName | 作業名稱           
ResourceId | Azure Resource Manager 資源識別碼
SubscriptionId | 訂用帳戶識別碼
EventTimeString | 作業時間
EventProperties | 作業屬性
狀態 | 作業狀態
呼叫者 | 作業呼叫者 (Azure 入口網站或管理用戶端)
category | OperationalLogs

以下是作業記錄 JSON 字串的範例：

```json
{
  "ActivityId": "6aa994ac-b56e-4292-8448-0767a5657cc7",
  "EventName": "Create Queue",
  "resourceId": "/SUBSCRIPTIONS/1A2109E3-9DA0-455B-B937-E35E36C1163C/RESOURCEGROUPS/DEFAULT-SERVICEBUS-CENTRALUS/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/SHOEBOXEHNS-CY4001",
  "SubscriptionId": "1a2109e3-9da0-455b-b937-e35e36c1163c",
  "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
  "EventProperties": "{\"SubscriptionId\":\"1a2109e3-9da0-455b-b937-e35e36c1163c\",\"Namespace\":\"shoeboxehns-cy4001\",\"Via\":\"https://shoeboxehns-cy4001.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
  "Status": "Succeeded",
  "Caller": "ServiceBus Client",
  "category": "OperationalLogs"
}
```

## <a name="next-steps"></a>後續步驟

請參閱下列連結以深入了解服務匯流排：

* [服務匯流排簡介](service-bus-messaging-overview.md)
* [開始使用服務匯流排](service-bus-dotnet-get-started-with-queues.md)
