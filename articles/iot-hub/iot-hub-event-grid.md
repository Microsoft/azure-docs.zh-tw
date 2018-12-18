---
title: Azure IoT 中樞和事件格線 | Microsoft Docs
description: 使用「Azure 事件格線」以根據「IoT 中樞」中發生的動作來觸發程序。
author: kgremban
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/14/2018
ms.author: kgremban
ms.openlocfilehash: 3c12e98137f44ac094adaae282b5d56d30061e60
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719846"
---
# <a name="react-to-iot-hub-events-by-using-event-grid-to-trigger-actions"></a>使用事件方格來觸發動作以回應 IoT 中樞事件

「Azure IoT 中樞」與「Azure 事件格線」整合，讓您能夠將事件通知傳送給其他服務並觸發下游程序。 請設定您的商務應用程式來接聽「IoT 中樞」事件，以便以可靠、彈性且安全的方式回應重大事件。 例如，建置應用程式來執行多個動作，像是更新資料庫、建立票證，以及在每次將新 IoT 裝置註冊至 IoT 中樞時傳遞電子郵件通知。 

[Azure 事件格線][lnk-eg-overview]是一個使用「發佈-訂閱」模型的完全受控事件路由服務。 「事件格線」針對 Azure 服務 (例如 [Azure Functions](../azure-functions/functions-overview.md) 和 [Azure Logic Apps](../logic-apps/logic-apps-what-are-logic-apps.md)) 內建支援，並可使用 Webhook 將事件警示傳遞給非 Azure 服務。 如需「事件格線」支援的事件處理常式完整清單，請參閱 [Azure 事件格線簡介][lnk-eg-overview]。 

![Azure 事件格線架構](./media/iot-hub-event-grid/event-grid-functional-model.png)

## <a name="regional-availability"></a>區域可用性

位於支援「事件格線」之區域中的 IoT 中樞都可使用「事件格線」整合。 如需最新的區域清單，請參閱 [Azure 事件格線簡介][lnk-eg-overview]。 

## <a name="event-types"></a>事件類型

「IoT 中樞」會發佈下列事件類型： 

| 事件類型 | 說明 |
| ---------- | ----------- |
| Microsoft.Devices.DeviceCreated | 向 IoT 中樞註冊裝置時發佈。 |
| Microsoft.Devices.DeviceDeleted | 從 IoT 中樞刪除裝置時發佈。 |
| Microsoft.Devices.DeviceConnected | 在裝置連線至 IoT 中樞時發佈。 |
| Microsoft.Devices.DeviceDisconnected | 在裝置從 IoT 中樞中斷連線時發佈。 |

請使用 Azure 入口網站或 Azure CLI 來設定要從每個 IoT 中樞發佈哪些事件。 例如，請試試[使用 Logic Apps 來傳送 Azure IoT 中樞事件的相關電子郵件通知](../event-grid/publish-iot-hub-events-to-logic-apps.md)教學課程。

## <a name="event-schema"></a>結構描述

「IoT 中樞」事件包含了回應裝置生命週期變更時所需的所有資訊。 您可以透過檢查 eventType 屬性的開頭是否為 **Microsoft.Devices** 來識別「IoT 中樞」事件。 如需有關如何使用「事件格線」事件屬性的詳細資訊，請參閱[事件格線事件結構描述](../event-grid/event-schema.md)。

### <a name="device-connected-schema"></a>裝置連線結構描述

下列範例說明裝置連線事件的結構描述： 

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
    "moduleId" : "DeviceModuleID",
  }, 
  "dataVersion": "1", 
  "metadataVersion": "1" 
}]
```

### <a name="device-created-schema"></a>已建立裝置結構描述

以下範例說明已建立裝置事件的結構描述： 

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
      "deviceEtag":"null",
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

如需每個屬性的詳細描述，請參閱[適用於 IoT 中樞的 Azure 事件格線事件結構描述](../event-grid/event-schema-iot-hub.md)

## <a name="filter-events"></a>篩選事件

「IoT 中樞」事件訂閱可以根據事件類型和裝置名稱來篩選事件。 事件方格中的主旨篩選會根據**開始於** (前置詞) 和**結尾是** (尾碼) 的相符項而運作。 篩選會使用 `AND` 運算子，讓主旨同時符合前置詞和尾碼的事件會傳遞給訂閱者。 

「IoT 事件」的主旨使用以下格式：

```json
devices/{deviceId}
```
## <a name="limitations-for-device-connected-and-device-disconnected-events"></a>裝置連線和裝置中斷連線事件的限制

若要接收裝置連線和裝置中斷連線事件，您必須為裝置開啟 D2C 連結或 C2D 連結。 如果您的裝置使用 MQTT 通訊協定，則 IoT 中樞會讓 C2D 連結保持開啟。 針對 AMQP，您可以藉由呼叫[接收非同步 API](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.deviceclient.receiveasync?view=azure-dotnet) 來開啟 C2D 連結。 如果您要傳送遙測資料，應開啟 D2C 連結。 如果裝置連線不斷變動 (也就是裝置會頻繁地連線和中斷連線)，我們將不會傳送每一個連線狀態，但會發佈每分鐘擷取快照的連線狀態。 IoT 中樞運作中斷時，我們將在中斷狀況結束後隨即發佈裝置連線狀態。 如果裝置在這段停止運作期間中斷連線，裝置中斷連線事件將在 10 分鐘內發佈。

## <a name="tips-for-consuming-events"></a>取用事件的秘訣

處理「IoT 中樞」事件的應用程式應該依循下列建議做法：

* 由於可以設定讓多個訂用帳戶將事件路由傳送至相同的事件處理常式，因此有一點相當重要，就是請勿假設事件來自特定的來源。 請一律檢查訊息主題，以確保訊息來自您預期的 IoT 中樞。 
* 請勿假設您收到的所有事件都是您預期的類型。 處理訊息之前，請一律先檢查 eventType。
* 訊息的抵達可能會不按順序或有延遲。 請使用 etag 欄位來了解物件的相關資訊是否為最新狀態。

## <a name="next-steps"></a>後續步驟

* [試試 IoT 中樞事件教學課程](../event-grid/publish-iot-hub-events-to-logic-apps.md)
* [了解如何排序裝置連線和中斷連線事件](../iot-hub/iot-hub-how-to-order-connection-state-events.md)
* [深入了解事件格線][lnk-eg-overview]
* [比較路由傳送 IoT 中樞事件與訊息之間的差異][lnk-eg-compare]

<!-- Links -->
[lnk-eg-overview]: ../event-grid/overview.md
[lnk-eg-compare]: iot-hub-event-grid-routing-comparison.md