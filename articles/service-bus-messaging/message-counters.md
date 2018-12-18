---
title: Azure 服務匯流排訊息計數 | Microsoft Docs
description: 擷取 Azure 服務匯流排訊息的計數。
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
ms.date: 09/26/2018
ms.author: spelluru
ms.openlocfilehash: bd9d1b813c90be09301eeff1de7580f22c15500b
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407581"
---
# <a name="message-counters"></a>訊息計數器

您可以使用 Azure Resource Manager 和 .NET Framework SDK 中的服務匯流排 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) API，擷取佇列和訂用帳戶中保留的訊息計數。

您可以透過 PowerShell 取得計數，如下所示：

```powershell
(Get-AzureRmServiceBusQueue -ResourceGroup mygrp -NamespaceName myns -QueueName myqueue).CountDetails
```

## <a name="message-count-details"></a>訊息計數詳細資料

得知作用中的訊息計數，有助於判斷佇列建置之待處理項目所需處理的資源是否比目前部署的資源還多。 [MessageCountDetails](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails) 類別提供以下計數器詳細資料：

-   [ActiveMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.activemessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_ActiveMessageCount)：佇列或訂用帳戶中處於作用中狀態，而且可供傳遞的訊息。
-   [DeadLetterMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.deadlettermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_DeadLetterMessageCount)：無效信件佇列中的訊息。
-   [ScheduledMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.scheduledmessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_ScheduledMessageCount)：處於已排程狀態的訊息。
-   [TransferDeadLetterMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.transferdeadlettermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_TransferDeadLetterMessageCount)：無法傳送到另一個佇列或主題，而且已移動到傳送無效信件佇列的訊息。
-   [TransferMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.transfermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_TransferMessageCount)：等待傳送到另一個佇列或主題的訊息。

如果應用程式想要依據佇列長度調整資源，它應該採用循序漸進的方式來進行。 取得訊息計數是在訊息代理程式中是項昂貴的作業，頻繁執行會直接對實體效能造成不良影響。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)