---
title: 如何搭配 Ruby 使用 Azure 服務匯流排佇列 | Microsoft Docs
description: 了解如何使用 Azure 中的服務匯流排佇列。 程式碼範例以 Ruby 撰寫。
services: service-bus-messaging
documentationcenter: ruby
author: spelluru
manager: timlt
editor: ''
ms.assetid: 0a11eab2-823f-4cc7-842b-fbbe0f953751
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: ruby
ms.topic: article
ms.date: 08/10/2017
ms.author: spelluru
ms.openlocfilehash: 26015ec1a3bf294e466902992ff13f1bc4693f04
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702335"
---
# <a name="how-to-use-service-bus-queues-with-ruby"></a>如何將服務匯流排佇列搭配 Ruby 使用

[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

本指南將說明如何使用服務匯流排佇列。 這些範例均以 Ruby 撰寫，並使用 Azure gem。 本文說明的案例包括**建立佇列、傳送並接收訊息**，以及**刪除佇列**。 如需服務匯流排佇列的詳細資訊，請參閱[後續步驟](#next-steps)一節。

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]
   
[!INCLUDE [service-bus-ruby-setup](../../includes/service-bus-ruby-setup.md)]

## <a name="how-to-create-a-queue"></a>如何建立佇列
**Azure::ServiceBusService** 物件可讓您使用佇列。 若要建立佇列，請使用 `create_queue()` 方法。 下列範例會建立佇列，或列印出任何錯誤。

```ruby
azure_service_bus_service = Azure::ServiceBus::ServiceBusService.new(sb_host, { signer: signer})
begin
  queue = azure_service_bus_service.create_queue("test-queue")
rescue
  puts $!
end
```

您也可以使用其他選項傳遞 **Azure::ServiceBus::Queue** 物件，這可讓您覆寫訊息存留時間或佇列大小上限等預設佇列設定。 下列範例說明如何將佇列大小上限設為 5 GB，將存留時間設為 1 分鐘：

```ruby
queue = Azure::ServiceBus::Queue.new("test-queue")
queue.max_size_in_megabytes = 5120
queue.default_message_time_to_live = "PT1M"

queue = azure_service_bus_service.create_queue(queue)
```

## <a name="how-to-send-messages-to-a-queue"></a>如何傳送訊息至佇列
若要傳送訊息至服務匯流排佇列，應用程式將呼叫 **Azure::ServiceBusService** 物件上的 `send_queue_message()` 方法。 傳送至 (及接收自)「服務匯流排」佇列的訊息是 **Azure::ServiceBus::BrokeredMessage** 物件，而且具有一組標準屬性 (例如 `label` 和 `time_to_live`)、一個用來保存自訂應用程式特定屬性的字典，以及一堆任意的應用程式資料。 應用程式可用訊息的形式傳遞字串值以設定訊息內文，任何必要的標準屬性都會以預設值填入。

下列範例示範如何使用 `send_queue_message()` 將測試訊息傳送至名為 `test-queue` 的佇列：

```ruby
message = Azure::ServiceBus::BrokeredMessage.new("test queue message")
message.correlation_id = "test-correlation-id"
azure_service_bus_service.send_queue_message("test-queue", message)
```

服務匯流排佇列支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 佇列中所保存的訊息數目沒有限制，但佇列所保存的訊息大小總計會有最高限制。 此佇列大小會在建立時定義，上限是 5 GB。

## <a name="how-to-receive-messages-from-a-queue"></a>如何從佇列接收訊息
對於 **Azure::ServiceBusService** 物件使用 `receive_queue_message()` 方法即可從佇列接收訊息。 根據預設，在讀取及鎖定訊息後並不會將其從佇列中刪除。 但您可以將 `:peek_lock` 選項設為 **false**，而在讀取訊息後將其從佇列中刪除。

預設行為會使讀取和刪除變成兩階段作業，因此也可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成處理訊息 (或可靠地儲存此訊息以供未來處理) 之後，它可透過呼叫 `delete_queue_message()` 方法和以參數形式提供要刪除的訊息，完成接收程序的第二個階段。 `delete_queue_message()` 方法會將訊息標示為已取用，並將其從佇列中移除。

如果 `:peek_lock` 參數設為 **false**，讀取和刪除訊息將會變成最簡單的模型，且最適用於應用程式容許在發生失敗時不處理訊息的案例。 若要了解這一點，請考慮取用者發出接收要求，接著系統在處理此要求之前當機的案例。 因為服務匯流排已將訊息標示為已取用，當應用程式重新啟動並開始重新取用訊息時，它將會遺漏當機前已取用的訊息。

下列範例示範如何使用 `receive_queue_message()` 來接收和處理訊息。 此範例會先使用設為 **false** 的 `:peek_lock` 來接收及刪除訊息，然後再接收另一個訊息，接著使用 `delete_queue_message()` 刪除訊息：

```ruby
message = azure_service_bus_service.receive_queue_message("test-queue",
  { :peek_lock => false })
message = azure_service_bus_service.receive_queue_message("test-queue")
azure_service_bus_service.delete_queue_message(message)
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何處理應用程式當機與無法讀取的訊息
服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。 如果接收者應用程式因為某些原因無法處理訊息，它可以呼叫 **Azure::ServiceBusService** 物件上的 `unlock_queue_message()` 方法。 此呼叫將導致服務匯流排將佇列中的訊息解除鎖定，讓此訊息可以被相同取用應用程式或其他取用應用程式重新接收。

與在佇列內鎖定訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

如果應用程式在處理訊息之後，尚未呼叫 `delete_queue_message()` 方法時當機，則會在應用程式重新啟動時將訊息重新傳遞給該應用程式。 此程序通常稱為 *至少處理一次*，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，則應用程式開發人員應在其應用程式中加入其他邏輯，以處理重複的訊息傳遞。 通常您可使用訊息的 `message_id` 屬性來達到此目的，該方法在各個傳遞嘗試中保持不變。

## <a name="next-steps"></a>後續步驟
了解基本的服務匯流排佇列之後，請參考下列連結以取得更多資訊。

* [佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)概觀。
* 請造訪 GitHub 上的 [Azure SDK for Ruby](https://github.com/Azure/azure-sdk-for-ruby) 儲存機制。

若要比較本文所討論的 Azure 服務匯流排佇列與[如何使用 Ruby 的佇列儲存體](../storage/queues/storage-ruby-how-to-use-queue-storage.md)一文中討論的 Azure 佇列，請參閱 [Azure 佇列和 Azure 服務匯流排佇列 - 比較和對照](service-bus-azure-and-service-bus-queues-compared-contrasted.md)

