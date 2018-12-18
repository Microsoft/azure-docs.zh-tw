---
title: 如何使用服務匯流排主題 (Ruby) | Microsoft Docs
description: 了解如何在 Azure 使用服務匯流排主題及訂用帳戶。 程式碼範例專為 Ruby 應用程式撰寫。
services: service-bus-messaging
documentationcenter: ruby
author: spelluru
manager: timlt
editor: ''
ms.assetid: 3ef2295e-7c5f-4c54-a13b-a69c8045d4b6
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: ruby
ms.topic: article
ms.date: 08/10/2018
ms.author: spelluru
ms.openlocfilehash: 2bde0661f57acc9507b1f26f6ceb442cefee7947
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406476"
---
# <a name="how-to-use-service-bus-topics-and-subscriptions-with-ruby"></a>如何透過 Ruby 使用服務匯流排主題和訂用帳戶
 
[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

此文章說明如何從 Ruby 應用程式使用服務匯流排主題和訂用帳戶。 涵蓋的案例包括**建立主題和訂用帳戶、建立訂用帳戶篩選器、傳送訊息**至主題、**接收訂用帳戶的訊息**，以及**刪除主題和訂用帳戶**。 如需主題和訂用帳戶的詳細資訊，請參閱[後續步驟](#next-steps)一節。

[!INCLUDE [howto-service-bus-topics](../../includes/howto-service-bus-topics.md)]

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

[!INCLUDE [service-bus-ruby-setup](../../includes/service-bus-ruby-setup.md)]

## <a name="create-a-topic"></a>建立主題
**Azure::ServiceBusService** 物件可讓您使用主題。 下列程式碼將建立 **Azure::ServiceBusService** 物件。 若要建立主題，請使用 `create_topic()` 方法。 下列範例會建立主題，或列印出任何錯誤。

```ruby
azure_service_bus_service = Azure::ServiceBus::ServiceBusService.new(sb_host, { signer: signer})
begin
  topic = azure_service_bus_service.create_queue("test-topic")
rescue
  puts $!
end
```

您也可以使用其他選項傳遞 **Azure::ServiceBus::Topic** 物件，這可讓您覆寫訊息存留時間或佇列大小上限等預設主題設定。 下列範例說明將佇列大小上限設為 5 GB，將存留時間設為 1 分鐘的設定：

```ruby
topic = Azure::ServiceBus::Topic.new("test-topic")
topic.max_size_in_megabytes = 5120
topic.default_message_time_to_live = "PT1M"

topic = azure_service_bus_service.create_topic(topic)
```

## <a name="create-subscriptions"></a>建立訂用帳戶
**Azure::ServiceBusService** 物件也能用來建立主題訂用帳戶。 訂閱是具名的，它們能擁有選用的篩選器，以限制傳遞至訂閱之虛擬佇列的訊息集合。

訂用帳戶是持續性的， 會持續到本身或相關的主題遭到刪除為止。 如果應用程式含有建立訂用帳戶的邏輯，它應該會先使用 getSubscription 方法檢查訂用帳戶是否存在。

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>使用預設 (MatchAll) 篩選器建立訂用帳戶
如果在建立新的訂用帳戶時未指定篩選器，將會使用 **MatchAll** 篩選器 (預設)。 使用 **MatchAll** 篩選器時，所有發佈至主題的訊息都會被置於訂用帳戶的虛擬佇列中。 下列範例將建立名為「all-messages」的訂用帳戶，並使用預設的 **MatchAll** 篩選器。

```ruby
subscription = azure_service_bus_service.create_subscription("test-topic", "all-messages")
```

### <a name="create-subscriptions-with-filters"></a>使用篩選器建立訂用帳戶
您也可以定義篩選器，讓您指定傳送至主題的哪些訊息應顯示在特定訂用帳戶中。

在訂用帳戶支援的篩選器中，實作 SQL92 子集的 **Azure::ServiceBus::SqlFilter** 是最具彈性的類型。 SQL 篩選器會對發佈至主題之訊息的屬性運作。 如需可與 SQL 篩選條件搭配使用的運算式詳細資料，請檢閱 [SqlFilter](service-bus-messaging-sql-filter.md) 語法。

您可以使用 **Azure::ServiceBusService** 物件的 `create_rule()` 方法將篩選器新增至訂用帳戶。 此方法可讓您將篩選器新增至現有的訂用帳戶中。

由於預設篩選器會自動套用至所有新訂用帳戶，因此您必須先移除預設篩選器，否則 **MatchAll** 會覆寫您指定的其他任何篩選器。 您可以使用 **Azure::ServiceBusService** 物件上的 `delete_rule()` 方法移除預設規則。

以下範例將建立名為 "high-messages" 的訂用帳戶，而且所含的 **Azure::ServiceBus::SqlFilter** 只選取自訂 `message_number` 屬性大於 3 的訊息：

```ruby
subscription = azure_service_bus_service.create_subscription("test-topic", "high-messages")
azure_service_bus_service.delete_rule("test-topic", "high-messages", "$Default")

rule = Azure::ServiceBus::Rule.new("high-messages-rule")
rule.topic = "test-topic"
rule.subscription = "high-messages"
rule.filter = Azure::ServiceBus::SqlFilter.new({
  :sql_expression => "message_number > 3" })
rule = azure_service_bus_service.create_rule(rule)
```

同樣地，下列範例將建立名為 `low-messages` 的訂用帳戶，而且所含的 **Azure::ServiceBus::SqlFilter** 只會選取 `message_number` 屬性小於或等於 3 的訊息：

```ruby
subscription = azure_service_bus_service.create_subscription("test-topic", "low-messages")
azure_service_bus_service.delete_rule("test-topic", "low-messages", "$Default")

rule = Azure::ServiceBus::Rule.new("low-messages-rule")
rule.topic = "test-topic"
rule.subscription = "low-messages"
rule.filter = Azure::ServiceBus::SqlFilter.new({
  :sql_expression => "message_number <= 3" })
rule = azure_service_bus_service.create_rule(rule)
```

當訊息傳送至 `test-topic` 時，一律會將該訊息傳遞至已訂閱 `all-messages` 主題訂用帳戶的接收者，並選擇性地傳遞至已訂閱 `high-messages` 和 `low-messages` 主題訂用帳戶的接收者 (視訊息內容而定)。

## <a name="send-messages-to-a-topic"></a>傳送訊息至主題
若要將訊息傳送至服務匯流排主題，應用程式必須使用 **Azure::ServiceBusService** 物件的 `send_topic_message()` 方法。 傳送至服務匯流排主題的訊息是 **Azure::ServiceBus::BrokeredMessage** 物件的執行個體。 **Azure::ServiceBus::BrokeredMessage** 物件具有一組標準屬性 (例如 `label` 和 `time_to_live`)、一個用來保存自訂應用程式特定屬性的字典，以及一堆字串資料。 應用程式能將字串值傳遞至 `send_topic_message()` 方法以設定訊息本文，而且系統會將預設值填入任何需要的標準屬性中。

下列範例說明如何將五個測試訊息傳送至 `test-topic`。 迴圈反覆運算上每個訊息的 `message_number` 自訂屬性值會有變化 (這可判斷接收訊息的訂用帳戶為何)：

```ruby
5.times do |i|
  message = Azure::ServiceBus::BrokeredMessage.new("test message " + i,
    { :message_number => i })
  azure_service_bus_service.send_topic_message("test-topic", message)
end
```

服務匯流排主題支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 主題中所保存的訊息數目沒有限制，但主題所保存的訊息大小總計會有最高限制。 此主題大小會在建立時定義，上限是 5 GB。

## <a name="receive-messages-from-a-subscription"></a>自訂用帳戶接收訊息
對於 **Azure::ServiceBusService** 物件使用 `receive_subscription_message()` 方法即可從佇列接收訊息。 根據預設，在讀取 (查看) 及鎖定訊息後，並不會從訂用帳戶中刪除訊息。 您可以將 `peek_lock` 選項設為 **false**，而在讀取訊息後從訂用帳戶中刪除訊息。

預設行為會使讀取和刪除變成兩階段作業，因此也可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成處理訊息 (或可靠地儲存此訊息以供未來處理) 之後，它可透過呼叫 `delete_subscription_message()` 方法和以參數形式提供要刪除的訊息，完成接收程序的第二個階段。 `delete_subscription_message()` 方法會將訊息標示為已取用，並將其從訂用帳戶中移除。

如果 `:peek_lock` 參數設為 **false**，讀取和刪除訊息將會變成最簡單的模型，且最適用於應用程式容許在發生失敗時不處理訊息的案例。 設想取用者發出接收要求，然後系統在處理此要求之前當機的案例。 因為服務匯流排已將訊息標示為已取用，所以，當應用程式重新啟動並開始重新取用訊息時，它會遺漏當機前已取用的訊息。

以下範例將示範如何使用預設的 `receive_subscription_message()` 模式來接收與處理訊息。 此範例會先使用設為 **false** 的 `:peek_lock` 接收及刪除來自 `low-messages` 訂用帳戶的訊息，然後接收另一個來自 `high-messages` 的訊息，接著使用 `delete_subscription_message()` 刪除該訊息：

```ruby
message = azure_service_bus_service.receive_subscription_message(
  "test-topic", "low-messages", { :peek_lock => false })
message = azure_service_bus_service.receive_subscription_message(
  "test-topic", "high-messages")
azure_service_bus_service.delete_subscription_message(message)
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何處理應用程式當機與無法讀取的訊息
服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。 如果接收者應用程式因為某些原因無法處理訊息，它可以呼叫 **Azure::ServiceBusService** 物件上的 `unlock_subscription_message()` 方法。 這導致服務匯流排將訂用帳戶中的訊息解除鎖定，讓此訊息可以由相同取用應用程式或其他取用應用程式重新接收。

與在訂用帳戶內鎖定訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

如果應用程式在處理訊息之後，尚未呼叫 `delete_subscription_message()` 方法時當機，則會在應用程式重新啟動時將訊息重新傳遞給該應用程式。 這通常稱為*至少處理一次*，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，則應用程式開發人員應在其應用程式中加入其他邏輯，以處理重複的訊息傳遞。 您通常可以使用訊息的 `message_id` 屬性來達到此邏輯，這在各個傳遞嘗試中保持不變。

## <a name="delete-topics-and-subscriptions"></a>刪除主題和訂用帳戶
主題和訂用帳戶是持續性的，您必須透過 [Azure 入口網站][Azure portal]或以程式設計方式明確地刪除它們。 下列範例示範如何刪除名為 `test-topic` 的主題。

```ruby
azure_service_bus_service.delete_topic("test-topic")
```

刪除主題也將會刪除對主題註冊的任何訂用帳戶。 您也可以個別刪除訂用帳戶。 下列程式碼將示範如何將名為 `high-messages` 的訂用帳戶從 `test-topic` 主題中刪除：

```ruby
azure_service_bus_service.delete_subscription("test-topic", "high-messages")
```

## <a name="next-steps"></a>後續步驟
了解基本的服務匯流排主題之後，請參考下列連結以取得更多資訊。

* 請參閱[佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)。
* [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter#microsoft_servicebus_messaging_sqlfilter) 的 API 參考資料。
* 請造訪 GitHub 上的 [Azure SDK for Ruby](https://github.com/Azure/azure-sdk-for-ruby) 存放庫。

[Azure portal]: https://portal.azure.com
