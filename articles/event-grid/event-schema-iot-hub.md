---
title: IoT 中樞的 Azure Event Grid 結構描述 | Microsoft Docs
description: IoT 中樞之事件結構描述格式與屬性的參考頁面
services: iot-hub
documentationcenter: ''
author: kgremban
manager: timlt
editor: ''
ms.service: event-grid
ms.topic: reference
ms.date: 08/17/2018
ms.author: kgremban
ms.openlocfilehash: a86b22b3327b2353dd37a9f9863337d12a009434
ms.sourcegitcommit: a1140e6b839ad79e454186ee95b01376233a1d1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43143568"
---
# <a name="azure-event-grid-event-schema-for-iot-hub"></a>IoT 中樞的 Azure Event Grid 事件結構描述

本文提供 Azure IoT 中樞事件的屬性和結構描述。 如需事件結構描述的簡介，請參閱 [Azure Event Grid 事件結構描述](event-schema.md)。 

如需範例指令碼和教學課程的清單，請參閱 [IoT 中樞事件來源](event-sources.md#iot-hub)。

## <a name="available-event-types"></a>可用的事件類型

Azure IoT 中樞會發出下列事件類型：

| 事件類型 | 說明 |
| ---------- | ----------- |
| Microsoft.Devices.DeviceCreated | 向 IoT 中樞註冊裝置時發佈。 |
| Microsoft.Devices.DeviceDeleted | 從 IoT 中樞刪除裝置時發佈。 | 
| Microsoft.Devices.DeviceConnected | 在裝置連線至 IoT 中樞時發佈。 |
| Microsoft.Devices.DeviceDisconnected | 在裝置從 IoT 中樞中斷連線時發佈。 | 

## <a name="example-event"></a>事件範例

DeviceConnected 和 DeviceDisconnected 事件的結構描述具有相同的結構。 此事件範例顯示裝置連線至 IoT 中樞註冊時引發的事件結構描述：

```json
[{
  "id": "f6bbf8f4-d365-520d-a878-17bf7238abd8", 
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>", 
  "subject": "devices/LogicAppTestDevice", 
  "eventType": "Microsoft.Devices.DeviceConnected", 
  "eventTime": "2018-06-02T19:17:44.4383997Z", 
  "data": {
    "deviceConnectionStateEventInfo": {
      "sequenceNumber":
        "000000000000000001D4132452F67CE200000002000000000000000000000001"
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice",
    "moduleId" : "DeviceModuleID"
  }, 
  "dataVersion": "1", 
  "metadataVersion": "1" 
}]
```

DeviceCreated 和 DeviceDeleted 事件的結構描述具有相同的結構。 此事件範例顯示向 IoT 中樞註冊裝置時引發之事件的結構描述：

```json
[{
  "id": "56afc886-767b-d359-d59e-0da7877166b2",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceCreated",
  "eventTime": "2018-01-02T19:17:44.4383997Z",
  "data": {
    "twin": {
      "deviceId": "LogicAppTestDevice",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag": "null",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        },
        "reported": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        }
      }
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="event-properties"></a>事件屬性

所有事件都包含相同的最高層級資料： 

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| id | 字串 | 事件的唯一識別碼。 |
| 主題 | 字串 | 事件來源的完整資源路徑。 此欄位不可寫入。 Event Grid 提供此值。 |
| 主旨 | 字串 | 發行者定義事件主體的路徑。 |
| eventType | 字串 | 此事件來源已註冊的事件類型之一。 |
| eventTime | 字串 | 事件產生的時間，以提供者之 UTC 時間為準。 |
| data | 物件 | IoT 中樞事件資料。  |
| dataVersion | 字串 | 資料物件的結構描述版本。 發行者會定義結構描述版本。 |
| metadataVersion | 字串 | 事件中繼資料的結構描述版本。 Event Grid 會定義最上層屬性的結構描述。 Event Grid 提供此值。 |

對於所有 IoT 中樞事件，資料物件都會包含下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| hubName | 字串 | 已建立或刪除裝置的 IoT 中樞名稱。 |
| deviceId | 字串 | 裝置的唯一識別碼。 此區分大小寫的字串最長為 128 個字元，並支援 ASCII 7 位元英數字元和下列特殊字元：`- : . + % _ # * ? ! ( ) , = @ ; $ '`。 |

每個事件發行者有不同的資料物件內容。 對於**裝置連線**和**裝置中斷連線** IoT 中樞事件，資料物件會包含下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| moduleId | 字串 | 模組的唯一識別碼。 針對模組裝置才會輸出此欄位。 此區分大小寫的字串最長為 128 個字元，並支援 ASCII 7 位元英數字元和下列特殊字元：`- : . + % _ # * ? ! ( ) , = @ ; $ '`。 |
| deviceConnectionStateEventInfo | 物件 | 裝置連線狀態事件資訊
| sequenceNumber | 字串 | 一個號碼，有助於指出裝置連線或裝置中斷連線事件的順序。 最新的事件會有高於前一個事件的序號。 此號碼的變動有可能超過 1，但只會增加不會減少。 請參閱[如何使用序號](../iot-hub/iot-hub-how-to-order-connection-state-events.md)。 |

每個事件發行者有不同的資料物件內容。 對於**裝置建立**和**裝置刪除** IoT 中樞事件，資料物件會包含下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| twin | 物件 | 裝置對應項的相關資訊，也就是應用程式裝置中繼資料的雲端表示。 | 
| deviceID | 字串 | 裝置對應項的唯一識別碼。 | 
| etag | 字串 | 可確保裝置對應項的更新具有一致性的驗證程式。 對每個裝對應項而言，每個 etag 保證都是唯一的。 |  
| deviceEtag| 字串 | 可確保裝置登錄的更新具有一致性的驗證程式。 對每個裝置登錄而言，每個 deviceEtag 保證都是唯一的。 |
| status | 字串 | 已啟用或停用裝置對應項。 | 
| statusUpdateTime | 字串 | 上次更新裝置對應項狀態的 ISO8601 時間戳記。 |
| connectionState | 字串 | 裝置已連線或已中斷連線。 | 
| lastActivityTime | 字串 | 上次活動的 ISO8601 時間戳記。 | 
| cloudToDeviceMessageCount | integer | 傳送到此裝置的雲端到裝置訊息計數。 | 
| authenticationType | 字串 | 用於這個裝置的驗證類型：`SAS`、`SelfSigned` 或 `CertificateAuthority`。 |
| x509Thumbprint | 字串 | 指紋是 x509 憑證的唯一值，通常用來尋找憑證存放區中的特定憑證。 指紋是使用 SHA1 演算法所動態產生的，並未實際存在於憑證中。 | 
| primaryThumbprint | 字串 | x509 憑證的主要指紋。 |
| secondaryThumbprint | 字串 | x509 憑證的次要指紋。 | 
| version | integer | 每次更新裝置對應項時，整數就會遞增 1。 |
| desired | 物件 | 只能由應用程式後端寫入，並可由裝置讀取的屬性部分。 | 
| reported | 物件 | 只能由裝置寫入，並可由應用程式後端讀取的屬性部分。 |
| lastUpdated | 字串 | 上次更新裝置對應項屬性的 ISO8601 時間戳記。 | 

## <a name="next-steps"></a>後續步驟

* 如需 Azure Event Grid 的簡介，請參閱[什麼是 Event Grid？](overview.md)
* 若要深入了解 IoT 中樞與 Event Grid 一起運作的方式，請參閱[使用 Event Grid 觸發動作來回應 IoT 中樞事件](../iot-hub/iot-hub-event-grid.md)。