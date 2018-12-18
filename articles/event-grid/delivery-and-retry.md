---
title: Azure Event Grid 傳遞與重試
description: 說明 Azure Event Grid 如何傳遞事件，以及如何處理未傳遞的訊息。
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: tomfitz
ms.openlocfilehash: 2a9ff23e5182c8cb7c91ad93e368f61f258c84f8
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43841587"
---
# <a name="event-grid-message-delivery-and-retry"></a>Event Grid 訊息傳遞與重試 

本文章說明未確認傳遞時，Azure Event Grid 如何處理事件。

Event Grid 提供持久的傳遞。 它針對每個訂用帳戶傳遞每則訊息至少一次。 事件會立即傳送到每個訂用帳戶的已註冊 Webhook。 如果 Webhook 在第一次嘗試傳遞的 60 秒內未確認接收事件，事件格線會重試傳遞事件。 

目前，事件方格會將每個事件個別傳送給訂閱者。 訂閱者會收到包含單一事件的陣列。

## <a name="message-delivery-status"></a>訊息傳遞狀態

Event Grid 使用 HTTP 回應碼以確認接收事件。 

### <a name="success-codes"></a>成功碼

下列 HTTP 回應碼表示事件已成功傳遞到您的 Webhook。 Event Grid 會將傳遞視為完成。

- 200 確定
- 202 已接受

### <a name="failure-codes"></a>失敗碼

下列 HTTP 回應碼表示事件傳遞嘗試失敗。

- 400 不正確的要求
- 401 未經授權
- 404 找不到
- 408 要求逾時
- 413 要求實體太大
- 414 URI 太長
- 429 要求太多
- 500 內部伺服器錯誤
- 503 服務無法使用
- 504 閘道逾時

如果您[已設定無效信件端點](manage-event-delivery.md)，當事件方格接收到 400 或 413 回應碼，它會立即將該事件傳送至無效信件端點。 否則，事件方格會重試所有錯誤。

## <a name="retry-intervals-and-duration"></a>重試間隔和持續時間

Event Grid 針對事件傳遞使用指數輪詢重試原則。 如果您的 Webhook 沒有回應，或是傳回失敗碼，Event Grid 會於下列排程重試傳遞：

1. 10 秒
2. 30 秒
3. 1 分鐘
4. 5 分鐘
5. 10 分鐘
6. 30 分鐘
7. 1 小時

Event Grid 會將小型隨機加入所有重試間隔。 一個小時之後，事件傳遞會每小時重試一次。

根據預設，事件格線會讓所有未在 24 小時內傳遞的事件變成過期事件。 您可以在建立事件訂用帳戶時[自訂重試原則](manage-event-delivery.md)。 您可以提供傳遞嘗試次數上限 (預設為 30 次) 和事件存留時間 (預設為 1440 分鐘)。

## <a name="dead-letter-events"></a>無效信件事件

當事件方格無法傳遞事件時，它可以將未傳遞事件傳送到儲存體帳戶。 這個程序稱為無效信件處理。 若要查看未傳遞的事件，您可以從無效信件位置提取它們。 如需詳細資訊，請參閱[無效信件與重試原則](manage-event-delivery.md)。

## <a name="next-steps"></a>後續步驟

* 若要檢視事件傳遞的狀態，請參閱[監視 Event Grid 訊息傳遞](monitor-event-delivery.md)。
* 若要自訂事件傳遞選項，請參閱[管理事件方格傳遞設定](manage-event-delivery.md)。
* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)。
* 若要快速地開始使用 Event Grid，請參閱[使用 Azure Event Grid 建立和路由傳送自訂事件](custom-event-quickstart.md)。