---
title: 了解 Azure IoT 中樞的雲端到裝置傳訊 | Microsoft Docs
description: 開發人員指南 - 如何使用 IoT 中樞的雲端到裝置傳訊。 包含訊息生命週期和設定選項的相關資訊。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/15/2018
ms.author: dobett
ms.openlocfilehash: d3d8df0d1e00fdff4d0e1e93715e1a408116d1e7
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34632470"
---
# <a name="send-cloud-to-device-messages-from-iot-hub"></a>從 IoT 中樞傳送雲端到裝置訊息

若要從您的解決方案後端傳送單向通知到裝置應用程式，請從您的 IoT 中樞傳送雲端對裝置訊息到您的裝置。 如需 IoT 中樞所支援之其他雲端到裝置選項的討論，請參閱[雲端對裝置通訊指引][lnk-c2d-guidance]。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

您可以透過面向服務的端點 (**/messages/devicebound**) 來傳送雲端到裝置訊息。 裝置接著會透過裝置專用的端點 (**/devices/{deviceId}/messages/devicebound**) 來接收訊息。

為了讓每個雲端到裝置訊息都以單一裝置作為目標，IoT 中樞會將 **to** 屬性設定為 **/devices/{deviceId}/messages/devicebound**。

每個裝置佇列最多保留 50 則雲端到裝置訊息。 嘗試將更多訊息傳送到相同的裝置會導致錯誤。

## <a name="the-cloud-to-device-message-lifecycle"></a>雲端到裝置訊息生命週期

為了保證至少一次訊息傳遞，IoT 中樞會在每個裝置佇列保留雲端到裝置訊息。 裝置必須明確地認可「完成」  ，如此一來，IoT 中樞便能從佇列中移除它們。 此方法保證連線失敗和裝置故障能夠恢復。

下圖顯示 IoT 中樞之雲端到裝置訊息的生命週期狀態圖。

![雲端到裝置的訊息生命週期][img-lifecycle]

當 IoT 中樞服務傳送訊息至裝置時，該服務會將訊息狀態設定為 [已加入佇列]。 當裝置想要「接收」訊息時，IoT 中樞會「鎖定」該訊息 (藉由將狀態設為 [不可見])，以便讓裝置上的其他執行緒開始接收其他訊息。 當裝置執行緒完成處理訊息時，它會藉由「完成」  訊息來通知 IoT 中樞。 IoT 中樞接著會將狀態設定為 [已完成]。

裝置也可以選擇：

* 「拒絕」訊息，這會導致「IoT 中樞」將它設定為 [無法寄出] 狀態。 透過 MQTT 通訊協定連線的裝置無法拒絕雲端到裝置訊息。
* *放棄*訊息，這會導致 IoT 中樞將訊息放回佇列，並將狀態設定為**已排入佇列**。 透過 MQTT 通訊協定連線的裝置無法放棄雲端到裝置訊息。

執行緒可能無法處理訊息，且不會通知 IoT 中樞。 在此情況下，訊息會在過了 [可見性 (或鎖定) 逾時] 之後，自動從**不可見**狀態轉換回**已加入佇列**狀態。 此逾時的預設值是一分鐘。

IoT 中樞上的**最大傳遞計數**屬性會決定訊息可以在**已加入佇列**與**不可見**狀態之間轉換的次數上限。 達到該轉換次數之後，「IoT 中樞」就會將訊息的狀態設定為 [無法寄出]。 同樣地，「IoT 中樞」在訊息的到期時間過後 (請參閱[存留時間][lnk-ttl])，也會將訊息的狀態設定為 [無法寄出]。

[如何使用 IoT 中樞傳送雲端到裝置訊息][lnk-c2d-tutorial]示範如何從雲端傳送雲端到裝置訊息，並在裝置上接收訊息。

通常只要遺失訊息不會影響應用程式邏輯，裝置就應該完成雲端到裝置訊息。 例如，當裝置已在本機保存訊息內容，或已成功執行作業時。 訊息可能也會攜帶暫時性資訊，遺失該訊息並不會影響應用程式的功能。 對於長時間執行的工作，您有時可以：

* 在將作業描述保存於本機儲存體之後，完成雲端到裝置訊息。
* 在作業進度的各個階段，利用一或多個裝置到雲端訊息來通知解決方案後端。

## <a name="message-expiration-time-to-live"></a>訊息到期 (存留時間)

每個雲端到裝置的訊息都有到期時間。 此時間是由下列其中一項所設定：

* 服務中的 **ExpiryTimeUtc** 屬性。
* IoT 中樞，使用預設*存留時間*來指定作為 IoT 中樞屬性。

請參閱[雲端到裝置的設定選項][lnk-c2d-configuration]。

利用訊息到期和避免將訊息傳送至已中斷連線裝置的常見方法是設定較短的存留時間值。 這個做法可達到與維護裝置連線狀態相同的效果，同時更具效率。 當您要求訊息通知時，IoT 中樞會對您通知下列狀態的裝置：

* 能夠接收訊息。
* 未上線或已失敗。

## <a name="message-feedback"></a>訊息意見反應

當您傳送雲端到裝置的訊息時，服務可以要求傳遞每則訊息的意見反應 (關於該訊息的最終狀態)。

| Ack 屬性 | 行為 |
| ------------ | -------- |
| **positive** | 如果雲端到裝置訊息達到 [已完成] 狀態，IoT 中樞就會產生意見反應訊息。 |
| **negative** | 如果雲端到裝置訊息達到 [無法寄出] 狀態，「IoT 中樞」就會產生意見反應訊息。 |
| **full**     | IoT 中樞在任一情況下都會產生意見反應訊息。 |

如果 **Ack** 是 **full**，而且您沒有收到意見反應訊息，這表示該意見反應訊息已過期。 服務無法得知原始訊息發生了什麼事。 實際上，服務應該確保它可以在回饋到期之前對其進行處理。 最長到期時間是兩天，因此如果服務發生失敗，您會有充裕的時間可以讓服務再次執行。

如[端點][lnk-endpoints]中所述，IoT 中樞會透過面向服務的端點 (**/messages/servicebound/feedback**) 利用訊息來傳遞意見反應。 接收意見反應的語意與雲端到裝置訊息的接收語意相同。 可能的話，訊息意見反應會放入單一訊息中，其格式如下：

| 屬性     | 說明 |
| ------------ | ----------- |
| EnqueuedTime | 時間戳記，指出中樞收到意見反應訊息的時間。 |
| UserId       | `{iot hub name}` |
| ContentType  | `application/vnd.microsoft.iothub.feedback.json` |

主體是記錄的 JSON 序列化陣列，而每筆記錄都具有下列屬性︰

| 屬性           | 說明 |
| ------------------ | ----------- |
| EnqueuedTimeUtc    | 指出訊息的結果出現時的時間戳記。 例如，中樞收到意見反應訊息或原始訊息已過期。 |
| OriginalMessageId  | 此意見反應資訊相關之雲端到裝置訊息的 **MessageId**。 |
| StatusCode         | 必要字串。 用於 IoT 中樞所產生的回饋訊息中。 <br/> 「成功」 <br/> 「已過期」 <br/> 「DeliveryCountExceeded」 <br/> 「已拒絕」 <br/> 「已清除」 |
| 說明        | **StatusCode**的字串值。 |
| deviceId           | 此意見反應資訊相關之雲端到裝置訊息的目標裝置 **DeviceId**。 |
| DeviceGenerationId | 此意見反應資訊相關之雲端到裝置訊息的目標裝置 **DeviceGenerationId**。 |

服務必須指定雲端到裝置訊息的 **MessageId** ，才能使其意見反應與原始訊息相互關聯。

下列範例顯示意見反應訊息的本文。

```json
[
  {
    "OriginalMessageId": "0987654321",
    "EnqueuedTimeUtc": "2015-07-28T16:24:48.789Z",
    "StatusCode": 0,
    "Description": "Success",
    "DeviceId": "123",
    "DeviceGenerationId": "abcdefghijklmnopqrstuvwxyz"
  },
  {
    ...
  },
  ...
]
```

## <a name="cloud-to-device-configuration-options"></a>雲端到裝置組態選項

每個 IoT 中樞都會針對雲端到裝置傳訊公開下列設定選項：

| 屬性                  | 說明 | 範圍和預設值 |
| ------------------------- | ----------- | ----------------- |
| defaultTtlAsIso8601       | 雲端到裝置訊息的預設 TTL。 | ISO_8601 間隔高達 2D (最小為 1 分鐘)。 預設值︰1 小時。 |
| maxDeliveryCount          | 每個裝置佇列的雲端到裝置最大傳遞計數。 | 1 到 100。 預設值：10 |
| feedback.ttlAsIso8601     | 保留服務繫結的意見反應訊息。 | ISO_8601 間隔高達 2D (最小為 1 分鐘)。 預設值︰1 小時。 |
| feedback.maxDeliveryCount |意見反應佇列的最大傳遞計數。 | 1 到 100。 預設值 = 100。 |

如需如何設定這些設定選項的詳細資訊，請參閱[建立 IoT 中樞][lnk-portal]。

## <a name="next-steps"></a>後續步驟

如需您可以用來接收雲端對裝置訊息之 SDK 的資訊，請參閱 [Azure IoT SDK][lnk-sdks]。

若要嘗試接收雲端對裝置訊息，請參閱 [傳送雲端到裝置][lnk-c2d-tutorial] 教學課程。

[img-lifecycle]: ./media/iot-hub-devguide-messages-c2d/lifecycle.png

[lnk-portal]: iot-hub-create-through-portal.md
[lnk-c2d-guidance]: iot-hub-devguide-c2d-guidance.md
[lnk-endpoints]: iot-hub-devguide-endpoints.md
[lnk-sdks]: iot-hub-devguide-sdks.md
[lnk-ttl]: #message-expiration-time-to-live
[lnk-c2d-configuration]: #cloud-to-device-configuration-options
[lnk-lifecycle]: #message-lifecycle
[lnk-c2d-tutorial]: iot-hub-csharp-csharp-c2d.md
