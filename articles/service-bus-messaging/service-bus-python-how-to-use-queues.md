---
title: 如何搭配 Python 使用 Azure 服務匯流排佇列 | Microsoft Docs
description: 了解如何從 Python 使用 Azure 服務匯流排佇列。
services: service-bus-messaging
documentationcenter: python
author: spelluru
manager: timlt
editor: ''
ms.assetid: b95ee5cd-3b31-459c-a7f3-cf8bcf77858b
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: python
ms.topic: article
ms.date: 08/30/2018
ms.author: spelluru
ms.openlocfilehash: cc8fc62811fe019dcaf613a1c0b50d89183e6555
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406187"
---
# <a name="how-to-use-service-bus-queues-with-python"></a>如何將服務匯流排佇列搭配 Python 使用

[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

本文說明如何使用服務匯流排佇列。 相關範例是以 Python 撰寫，並且使用 [Python Azure 服務匯流排封裝][Python Azure Service Bus package]。 本文說明的案例包括**建立佇列、傳送並接收訊息**，以及**刪除佇列**。

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

> [!NOTE]
> 若要安裝 Python 或 [Python Azure 服務匯流排封裝][Python Azure Service Bus package]，請參閱 [Python 安裝指南](../python-how-to-install.md)。
> 
> 

## <a name="create-a-queue"></a>建立佇列
**ServiceBusService** 物件可讓您使用佇列。 將下列程式碼新增至您想要在其中以程式設計方式存取服務匯流排之任何 Python 檔案內的頂端附近：

```python
from azure.servicebus import ServiceBusService, Message, Queue
```

下列程式碼將建立 **ServiceBusService** 物件。 請使用您的命名空間、共用存取簽章 (SAS) 金鑰名稱和值來取代 `mynamespace`、`sharedaccesskeyname` 和 `sharedaccesskey`。

```python
bus_service = ServiceBusService(
    service_namespace='mynamespace',
    shared_access_key_name='sharedaccesskeyname',
    shared_access_key_value='sharedaccesskey')
```

SAS 金鑰名稱和值的值可以在 [Azure 入口網站][Azure portal]連線資訊中找到，或在 Visual Studio 的 [伺服器總管] 中選取「服務匯流排」命名空間時於 [屬性] 窗格中找到 (如上一節所示)。

```python
bus_service.create_queue('taskqueue')
```

`create_queue` 方法也支援其他選項，而可讓您覆寫訊息存留時間 (TTL) 或佇列大小上限等預設佇列設定。 下列範例會將佇列大小上限設為 5GB，並將 TTL 時間設為 1 分鐘：

```python
queue_options = Queue()
queue_options.max_size_in_megabytes = '5120'
queue_options.default_message_time_to_live = 'PT1M'

bus_service.create_queue('taskqueue', queue_options)
```

## <a name="send-messages-to-a-queue"></a>傳送訊息至佇列
若要傳送訊息至服務匯流排佇列，應用程式將呼叫 **ServiceBusService** 物件上的 `send_queue_message` 方法。

下列範例示範如何使用 `send_queue_message` 將測試訊息傳送至名為 `taskqueue` 的佇列：

```python
msg = Message(b'Test Message')
bus_service.send_queue_message('taskqueue', msg)
```

服務匯流排佇列支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 佇列中所保存的訊息數目沒有限制，但佇列所保存的訊息大小總計會有最高限制。 此佇列大小會在建立時定義，上限是 5 GB。 如需有關配額的詳細資訊，請參閱[服務匯流排配額][Service Bus quotas]。

## <a name="receive-messages-from-a-queue"></a>從佇列接收訊息
對於 **ServiceBusService** 物件使用 `receive_queue_message` 方法即可從佇列接收訊息：

```python
msg = bus_service.receive_queue_message('taskqueue', peek_lock=False)
print(msg.body)
```

如果將 `peek_lock` 參數設為 **False**，則當讀取訊息後，訊息便會從佇列中刪除。 您可以將參數 `peek_lock` 設為 **True**，來讀取 (查看) 並鎖定訊息，避免系統從佇列刪除訊息。

隨著接收作業讀取及刪除訊息之行為是最簡單的模型，且最適合可容許在發生失敗時不處理訊息的應用程式案例。 若要了解這一點，請考慮取用者發出接收要求，接著系統在處理此要求之前當機的案例。 因為服務匯流排會將訊息標示為已取用，當應用程式重新啟動並開始重新取用訊息時，它將會遺漏當機前已取用的訊息。

如果您將 `peek_lock` 參數設為 **True**，接收會變成兩階段作業，因此可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成訊息處理 (或可靠地儲存此訊息以供未來處理) 之後，它會在 **Message** 物件上呼叫 **delete** 方法，以完成接收程序的第二個階段。 **delete** 方法會將訊息標示為已取用，並將其從佇列中移除。

```python
msg = bus_service.receive_queue_message('taskqueue', peek_lock=True)
print(msg.body)

msg.delete()
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何處理應用程式當機與無法讀取的訊息
服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。 如果接收者應用程式因為某些原因無法處理訊息，它可以在 **Message** 物件上呼叫 **unlock** 方法。 這將導致服務匯流排將佇列中的訊息解除鎖定，讓此訊息可以被相同取用應用程式或其他取用應用程式重新接收。

與在佇列內鎖定訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排將自動解除鎖定訊息，並讓訊息可以被重新接收。

如果應用程式在處理訊息之後，尚未呼叫 **delete** 方法時當機，則會在應用程式重新啟動時將訊息重新傳遞給該應用程式。 這通常稱為**至少處理一次**，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，則應用程式開發人員應在其應用程式中加入其他邏輯，以處理重複的訊息傳遞。 通常您可使用訊息的 **MessageId** 屬性來達到此目的，該屬性將在各個傳遞嘗試中會保持不變。

## <a name="next-steps"></a>後續步驟
既然您已經了解「服務匯流排」佇列的基本概念，您現在可以閱讀下列文章來深入了解。

* [佇列、主題和訂用帳戶][Queues, topics, and subscriptions]

[Azure portal]: https://portal.azure.com
[Python Azure Service Bus package]: https://pypi.python.org/pypi/azure-servicebus  
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[Service Bus quotas]: service-bus-quotas.md

