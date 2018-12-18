---
title: 如何搭配 Python 使用 Azure 服務匯流排主題 | Microsoft Docs
description: 了解如何從 Python 使用 Azure 服務匯流排主題和訂用帳戶。
services: service-bus-messaging
documentationcenter: python
author: spelluru
manager: timlt
editor: ''
ms.assetid: c4f1d76c-7567-4b33-9193-3788f82934e4
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: python
ms.topic: article
ms.date: 09/20/2018
ms.author: spelluru
ms.openlocfilehash: ba95eb7742af611cc3365fdb07b75c785ba42681
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406884"
---
# <a name="how-to-use-service-bus-topics-and-subscriptions-with-python"></a>如何透過 Python 使用服務匯流排主題和訂用帳戶

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

此文章說明如何使用服務匯流排主題和訂用帳戶。 相關範例是以 Python 撰寫，並且使用 [Azure Python SDK 套件][Azure Python package]。 涵蓋的案例包括**建立主題和訂用帳戶**、**建立訂用帳戶篩選器**、**傳送訊息至主題**、**接收訂用帳戶的訊息**，以及**刪除主題和訂用帳戶**。 如需主題和訂用帳戶的詳細資訊，請參閱[後續步驟](#next-steps)一節。

[!INCLUDE [howto-service-bus-topics](../../includes/howto-service-bus-topics.md)]

> [!NOTE] 
> 如果您需要安裝 Python 或 [Azure Python 套件][Azure Python package]，請參閱 [Python 安裝指南](../python-how-to-install.md)。

## <a name="create-a-topic"></a>建立主題

**ServiceBusService** 物件可讓您使用主題。 將下列程式碼新增至您想要在其中以程式設計方式存取服務匯流排之任何 Python 檔案內的頂端附近：

```python
from azure.servicebus import ServiceBusService, Message, Topic, Rule, DEFAULT_RULE_NAME
```

下列程式碼將建立 **ServiceBusService** 物件。 請使用真實的命名空間、共用存取簽章 (SAS) 金鑰名稱和金鑰值來取代 `mynamespace`、`sharedaccesskeyname` 和 `sharedaccesskey`。

```python
bus_service = ServiceBusService(
    service_namespace='mynamespace',
    shared_access_key_name='sharedaccesskeyname',
    shared_access_key_value='sharedaccesskey')
```

您可以從 [Azure 入口網站][Azure portal]取得 SAS 金鑰名稱和值的值。

```python
bus_service.create_topic('mytopic')
```

`create_topic` 方法也支援其他選項，而可讓您覆寫訊息存留時間或主題大小上限等預設主題設定。 下列範例會將主題大小上限設為 5 GB，並將存留時間 (TTL) 的值設為 1 分鐘：

```python
topic_options = Topic()
topic_options.max_size_in_megabytes = '5120'
topic_options.default_message_time_to_live = 'PT1M'

bus_service.create_topic('mytopic', topic_options)
```

## <a name="create-subscriptions"></a>建立訂用帳戶

**ServiceBusService** 物件也能用來建立主題的訂用帳戶。 訂閱是具名的，它們能擁有選用的篩選器，以限制傳遞至訂閱之虛擬佇列的訊息集合。

> [!NOTE]
> 訂用帳戶是持續性的，它們會持續存在，直到本身或它們訂閱的主題遭到刪除為止。
> 
> 

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>使用預設 (MatchAll) 篩選器建立訂用帳戶

如果在建立新的訂用帳戶時未指定篩選器，將會使用 **MatchAll** 篩選器 (預設)。 使用 **MatchAll** 篩選器時，所有發佈至主題的訊息都會被置於訂用帳戶的虛擬佇列中。 下列範例將建立名為 `AllMessages` 的訂用帳戶，並使用預設的 **MatchAll** 篩選條件。

```python
bus_service.create_subscription('mytopic', 'AllMessages')
```

### <a name="create-subscriptions-with-filters"></a>使用篩選器建立訂用帳戶

您也可以定義篩選器，讓您指定傳送至主題的哪些訊息應顯示在特定主題訂用帳戶中。

在訂用帳戶支援的篩選器中，實作 SQL92 子集的 **SqlFilter** 是最具彈性的類型。 SQL 篩選器會對發佈至主題之訊息的屬性運作。 如需可與 SQL 篩選器搭配使用的運算式詳細資訊，請參閱 [SqlFilter.SqlExpression][SqlFilter.SqlExpression] 語法。

您可以使用 **ServiceBusService** 物件的 **create\_rule** 方法將篩選器新增至訂用帳戶。 此方法可讓您將篩選器新增至現有的訂用帳戶中。

> [!NOTE]
> 由於預設篩選器會自動套用至所有新的訂用帳戶，因此您必須先移除預設篩選器，否則 **MatchAll** 將會覆寫您指定的任何其他篩選器。 您可以使用 **ServiceBusService** 物件的 `delete_rule` 方法移除預設規則。
> 
> 

以下範例將建立名為 `HighMessages` 的訂用帳戶，而且所含的 **SqlFilter** 只會選取自訂 `messagenumber` 屬性大於 3 的訊息：

```python
bus_service.create_subscription('mytopic', 'HighMessages')

rule = Rule()
rule.filter_type = 'SqlFilter'
rule.filter_expression = 'messagenumber > 3'

bus_service.create_rule('mytopic', 'HighMessages', 'HighMessageFilter', rule)
bus_service.delete_rule('mytopic', 'HighMessages', DEFAULT_RULE_NAME)
```

同樣地，下列範例將建立名為 `LowMessages` 的訂用帳戶，而且所含的 **SqlFilter** 只會選取 `messagenumber` 屬性小於或等於 3 的訊息：

```python
bus_service.create_subscription('mytopic', 'LowMessages')

rule = Rule()
rule.filter_type = 'SqlFilter'
rule.filter_expression = 'messagenumber <= 3'

bus_service.create_rule('mytopic', 'LowMessages', 'LowMessageFilter', rule)
bus_service.delete_rule('mytopic', 'LowMessages', DEFAULT_RULE_NAME)
```

現在當訊息傳送到 `mytopic` 時，一律會將該訊息傳遞到已訂閱 **AllMessages** 主題訂用帳戶的接收者，並選擇性地將它傳遞到已訂閱 **HighMessages** 和 **LowMessages** 主題訂用帳戶的接收者 (視訊息內容而定)。

## <a name="send-messages-to-a-topic"></a>傳送訊息至主題

若要將訊息傳送至服務匯流排主題，應用程式必須使用 **ServiceBusService** 物件的 `send_topic_message` 方法。

下列範例說明如何將五個測試訊息傳送至 `mytopic`。 迴圈反覆運算上每個訊息的 `messagenumber` 屬性值會有變化 (這可判斷接收訊息的訂用帳戶為何)：

```python
for i in range(5):
    msg = Message('Msg {0}'.format(i).encode('utf-8'), custom_properties={'messagenumber':i})
    bus_service.send_topic_message('mytopic', msg)
```

服務匯流排主題支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 主題中所保存的訊息數目沒有限制，但主題所保存的訊息大小總計會有最高限制。 此主題大小會在建立時定義，上限是 5 GB。 如需有關配額的詳細資訊，請參閱[服務匯流排配額][Service Bus quotas]。

## <a name="receive-messages-from-a-subscription"></a>自訂用帳戶接收訊息

對於 **ServiceBusService** 物件使用 `receive_subscription_message` 方法即可從佇列接收訊息：

```python
msg = bus_service.receive_subscription_message('mytopic', 'LowMessages', peek_lock=False)
print(msg.body)
```

將 `peek_lock` 參數設為 **False** 時，則當讀取訊息後，訊息便會從訂用帳戶中刪除。 您可以將參數 `peek_lock` 設為 **True**，來讀取 (查看) 並鎖定訊息，避免系統從佇列刪除訊息。

隨著接收作業讀取及刪除訊息之行為是最簡單的模型，且最適合可容許在發生失敗時不處理訊息的應用程式案例。 若要了解此行為，請考慮取用者發出接收要求，接著系統在處理此要求之前當機的案例。 因為服務匯流排已將訊息標示為已取用，所以，當應用程式重新啟動並開始重新取用訊息時，它會遺漏當機前已取用的訊息。

如果您將 `peek_lock` 參數設為 **True**，接收會變成兩階段作業，因此可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成訊息處理 (或可靠地儲存此訊息以供未來處理) 之後，它會在 **Message** 物件上呼叫 `delete` 方法，以完成接收程序的第二個階段。 `delete` 方法會將訊息標示為已取用，並將其從訂用帳戶中移除。

```python
msg = bus_service.receive_subscription_message('mytopic', 'LowMessages', peek_lock=True)
if msg.body is not None:
print(msg.body)
msg.delete()
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何處理應用程式當機與無法讀取的訊息

服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。 如果接收者應用程式因為某些原因無法處理訊息，它可以在 **Message** 物件上呼叫 `unlock` 方法。 此方法導致服務匯流排將訂閱中的訊息解除鎖定，讓此訊息可以被相同取用應用程式或其他取用應用程式重新接收。

與在訂閱內鎖定訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

如果應用程式在處理訊息之後，尚未呼叫 `delete` 方法時當機，則會在應用程式重新啟動時將訊息重新傳遞給該應用程式。 這通常稱為 「至少處理一次」*，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，則應用程式開發人員應在其應用程式中加入其他邏輯，以處理重複的訊息傳遞。 您可使用訊息的 **MessageId** 屬性來完成此操作，該屬性在各個傳遞嘗試中會保持不變。

## <a name="delete-topics-and-subscriptions"></a>刪除主題和訂用帳戶

主題和訂用帳戶是持續性的，您必須透過 [Azure 入口網站][Azure portal]或以程式設計方式明確地刪除它們。 下列範例示範如何刪除名為 `mytopic` 的主題：

```python
bus_service.delete_topic('mytopic')
```

刪除主題也將會刪除對主題註冊的任何訂用帳戶。 您也可以個別刪除訂用帳戶。 下列程式碼示範如何從 `mytopic` 主題刪除名為 `HighMessages` 的訂用帳戶：

```python
bus_service.delete_subscription('mytopic', 'HighMessages')
```

## <a name="next-steps"></a>後續步驟

了解基本的服務匯流排主題之後，請參考下列連結以取得更多資訊。

* 請參閱[佇列、主題和訂用帳戶][Queues, topics, and subscriptions]。
* [SqlFilter.SqlExpression][SqlFilter.SqlExpression] 的參考資料。

[Azure portal]: https://portal.azure.com
[Azure Python package]: https://pypi.python.org/pypi/azure  
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[SqlFilter.SqlExpression]: service-bus-messaging-sql-filter.md
[Service Bus quotas]: service-bus-quotas.md 
