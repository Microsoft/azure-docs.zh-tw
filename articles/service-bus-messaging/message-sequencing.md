---
title: Azure 服務匯流排訊息排序和時間戳記 | Microsoft 文件
description: 利用時間戳記保留服務匯流排訊息序列和順序
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
ms.openlocfilehash: 7f31adcd7ebdd51cf930fcaf0cd2f214c7566565
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699364"
---
# <a name="message-sequencing-and-timestamps"></a>訊息排序和時間戳記

排序和時間戳記是隨時可以在所有服務匯流排實體和介面上啟用的兩項功能，您可以透過接收或瀏覽之訊息的 [SequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sequencenumber) 和 [EnqueuedTimeUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedtimeutc) 屬性予以啟用。

在重視訊息絕對順序和/或客戶需要值得信賴之訊息唯一識別碼的情況下，訊息代理程式會為訊息加上與佇列或主題相關的連貫遞增序號。 對於分割的實體，發行的序號與分割區相關。

**SequenceNumber**是指派給訊息的唯一 64 位元整數，會由訊息代理程式和函式所接收並儲存，以作為內部識別碼。 對於分割的實體，最前面的 16 位元會反映分割區識別碼。 序號會在 48/64 位元範圍用盡時回復為 0。

序號由中央和中立的權力機構指派 (而非用戶端)，因此如唯一識別碼般值得信賴。 它也代表真正的抵達順序，而且在當作順序準則時還比時間戳記來得精準。因為在極端訊息速率的情況下，時間戳記的解析度可能會不夠精確，另外當訊息代理程式擁有權在節點間轉換時，時間戳記可能會受到時鐘誤差的影響 (雖然影響很細微)。

在諸如商品數量有限，先買先贏、賣完為止的商務案例中，絕對抵達順序至關重要。演唱會門票的銷售即是一例。

時間戳記功能可成為中性、值得信賴的依據，其精準地記錄訊息抵達時的 UTC 時間，並反映在 **EnqueuedTimeUtc**。 這個值對於仰賴截止日期的商務案例來說很實用，例如工作項目已在某天的午夜之前提交，不過處理進度卻遠遠落後佇列的待處理項目。

## <a name="scheduled-messages"></a>排定的訊息

您可以將訊息提交到佇列或主題，以利延遲處理。例如，排定作業讓系統在某個時間處理。 這項功能實現可靠的分散式時間型排程器。

排定之訊息在定義的加入佇列時間到來之前，不會在佇列中具體化。 在時間到來之前，您可以取消排定的訊息。 取消即是刪除訊息。

在透過一般傳送路徑傳送訊息時，您可以藉由設定 [ScheduledEnqueueTimeUtc](/dotnet/api/microsoft.azure.servicebus.message.scheduledenqueuetimeutc) 屬性來排定訊息，或明確地利用 [ScheduleMessageAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.schedulemessageasync#Microsoft_Azure_ServiceBus_QueueClient_ScheduleMessageAsync_Microsoft_Azure_ServiceBus_Message_System_DateTimeOffset_) API 排定訊息。 後者能立即傳回排定之訊息的 **SequenceNumber**，稍後若有需要，您可以用來取消排定的訊息。 您也可以使用[訊息瀏覽](message-browsing.md)來探索排定的訊息與其序號。

排定之訊息的 **SequenceNumber** 唯有在訊息仍處於該狀態時有效。 當訊息轉換成作用中狀態時，系統會將訊息附加到佇列中，如同在當下立刻加入佇列，其中包括指派新的 **SequenceNumber**。

由於該功能會錨定在個別訊息上，而訊息只能加入佇列一次，因此服務匯流排不支援訊息的週期性排程。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)