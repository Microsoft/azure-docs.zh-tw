---
title: Azure 服務匯流排重複訊息偵測 | Microsoft 文件
description: 偵測重複的服務匯流排訊息
services: service-bus-messaging
documentationcenter: ''
author: clemensv
manager: timlt
editor: ''
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/25/2018
ms.author: spelluru
ms.openlocfilehash: 7402fcf01078ea3934d1b6794a9190947fe339c2
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696620"
---
# <a name="duplicate-detection"></a>重複偵測

如果應用程式在傳送訊息後立即發生嚴重錯誤，而重新啟動後的應用程式執行個體錯誤地認為先前的訊息傳遞並未發生，後續的傳送會導致同樣的訊息在系統中出現兩次。

這也有可能是用戶端或網路層級在稍早發生錯誤，在通知未能成功返回用戶端的情況下，讓已傳送的訊息提交到佇列中。 這個案例會讓用戶端對傳送作業的結果產生疑慮。

透過讓寄件者重新傳送同樣的訊息，重複偵測能將這些情況下的疑慮排除，而佇列或主題則會捨棄任何重複的複本。

對於在指定時間範圍內傳送到佇列或主題的所有訊息，啟用重複偵測有助於追蹤由應用程式控制的 *MessageId*。 對於任何已傳送的新訊息，如果訊息附帶的 *MessageId* 已在時間範圍內記錄下來，系統便會將訊息回報為已接受 (傳送作業成功)，不過新傳送的訊息會立即遭到忽略或捨棄。 除了 *MessageId* 之外，系統不會考慮訊息的其他部分。

由應用程式控制識別碼是必要措施，唯有這樣才能讓應用程式將 *MessageId* 與商務程序內容繫結，在發生故障時運用可預測的方法重建。

對於在處理某些應用程式內容期間傳送多則訊息的商務程序，*MessageId* 可以是應用程式層級內容識別碼 (如訂單號碼) 和訊息主旨 (如 **12345.2017/付款**) 的複合。

*MessageId* 永遠都可以是某個 GUID，不過將識別碼錨定在商務程序中會產生可預測的重複性，這是有效運用重複偵測功能不可或缺的一環。

## <a name="enable-duplicate-detection"></a>啟用重複偵測

在入口網站中，於建立實體時勾選 [啟用重複偵測] 核取方塊可開啟該功能 (預設為關閉)。 建立新主題的設定也是如此。

![][1]

從程式設計方面來說，您可以利用 [QueueDescription.requiresDuplicateDetection](/dotnet/api/microsoft.servicebus.messaging.queuedescription.requiresduplicatedetection#Microsoft_ServiceBus_Messaging_QueueDescription_RequiresDuplicateDetection) 屬性在完整架構的 .NET API 上設定旗標。 透過 Azure Resource Manager API，您可以利用 [queueProperties.requiresDuplicateDetection](/azure/templates/microsoft.servicebus/namespaces/queues#property-values) 屬性設定值。

佇列和主題的重複偵測時間記錄預設為 30 秒，最大值為 7 天。 您可以在 Azure 入口網站的佇列和主題屬性視窗中變更這項設定。

![][2]

從程式設計方面來說，您可以使用 [QueueDescription.DuplicateDetectionHistoryTimeWindow](/dotnet/api/microsoft.servicebus.messaging.queuedescription.duplicatedetectionhistorytimewindow#Microsoft_ServiceBus_Messaging_QueueDescription_DuplicateDetectionHistoryTimeWindow) 屬性搭配完整的 .NET Framework API，設定保留訊息識別碼之重複偵測時間範圍的大小。 透過 Azure Resource Manager API，您可以使用 [queueProperties.duplicateDetectionHistoryTimeWindow](/azure/templates/microsoft.servicebus/namespaces/queues#property-values) 屬性設定值。

請注意，啟用重複偵測和時間範圍的大小會直接影響佇列 (和主題) 輸送量，因為所有記錄下來的訊息識別碼必須與新提交的訊息識別碼相符。

維持較短的時間範圍，意味著要保留及比對的訊息識別碼比較少，對輸送量的影響也比較小。 對於需要重複偵測的高輸送量實體，您應盡可能縮短時間範圍。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[1]: ./media/duplicate-detection/create-queue.png
[2]: ./media/duplicate-detection/queue-prop.png
