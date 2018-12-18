---
title: 回應 Azure 媒體服務事件 | Microsoft Docs
description: 使用 Azure 事件格線訂閱媒體服務事件。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: 3541a5b33aa0bb98d9381b51caefc63b6aa677ad
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49377543"
---
# <a name="handling-event-grid-events"></a>處理事件方格事件

媒體服務事件可讓應用程式利用現代無伺服器架構，針對不同事件做出反應，例如，工作狀態變更事件。 它不需要複雜的程式碼或昂貴且無效率的輪詢服務來執行此動作。 而是改為透過 [Azure 事件格線](https://azure.microsoft.com/services/event-grid/)，將事件推送給事件處理常式，例如 [Azure Functions](https://azure.microsoft.com/services/functions/)、[Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/)，甚至推送到您自己的Webhook，而且只有使用時，才須支付相關費用。 如需價格的資訊，請參閱[事件格線價格](https://azure.microsoft.com/pricing/details/event-grid/)。

媒體服務事件的可用性會繫結至事件格線[可用性](../../event-grid/overview.md)，並且將在其他區域中變成可用狀態，就像事件格線所做的一樣。  

## <a name="available-media-services-events"></a>可用的媒體服務事件

Event Grid 使用[事件訂閱](../../event-grid/concepts.md#event-subscriptions)將事件訊息路由至訂閱者。  目前，媒體服務事件訂閱可以包含下列事件：  

|活動名稱|說明|
|----------|-----------|
| Microsoft.Media.JobStateChange| 當工作的狀態變更時引發。 |
| Microsoft.Media.LiveEventConnectionRejected | 編碼器的連線嘗試遭到拒絕。 |
| Microsoft.Media.LiveEventEncoderConnected | 編碼器對即時事件建立連線。 |
| Microsoft.Media.LiveEventEncoderDisconnected | 編碼器中斷連線。 |
| Microsoft.Media.LiveEventIncomingDataChunkDropped | 媒體伺服器卸除資料區塊，原因是其來得太晚或有重疊的時間戳記 (新資料區塊的時間戳記小於先前資料區塊的結束時間)。 |
| Microsoft.Media.LiveEventIncomingStreamReceived | 媒體伺服器收到資料流或連線中每個資料軌的第一個資料區塊。 |
| Microsoft.Media.LiveEventIncomingStreamsOutOfSync | 媒體伺服器偵測到音訊和視訊資料流未同步。可作為警告，原因是使用者體驗可能不受影響。 |
| Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync | 媒體伺服器偵測到來自外部編碼器的兩個視訊資料流中，有任一個未同步。可作為警告，原因是使用者體驗可能不受影響。 |
| Microsoft.Media.LiveEventIngestHeartbeat | 當即時事件執行時，會針對每個資料軌每 20 秒發佈一次。 提供內嵌健康情況摘要。 |
| Microsoft.Media.LiveEventTrackDiscontinuityDetected | 媒體伺服器偵測到內送的資料軌發生中斷。 |

## <a name="event-schema"></a>事件結構描述

媒體事件包含了回應資料變更時所需的一切資訊。  因為 eventType 屬性開頭為 “Microsoft.Media”，所以您可以藉此識別出媒體服務事件。

如需詳細資訊，請參閱[媒體服務事件結構描述](media-services-event-schemas.md)。

## <a name="practices-for-consuming-events"></a>消費事件做法

處理媒體服務事件的應用程式，應該遵循幾個建議做法：

* 由於可設定多個訂用帳戶以將事件路由至相同的事件處理常式，因此重要的是，不要假設事件來自於特定來源，而要檢查訊息主題以確定其來自預期的儲存體帳戶。
* 同樣地，檢查 eventType 也是必須進行的步驟之一，而且不要假設您收到的所有事件都是您預期的類型。
* 請忽略您不了解的欄位。  此做法將有助於保持未來可能新增功能的彈性。
* 使用 "subject" 前置詞和後置詞相符，將事件限制為特定的事件。

## <a name="next-steps"></a>後續步驟

[取得工作狀態事件](job-state-events-cli-how-to.md)
