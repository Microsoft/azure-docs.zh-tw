---
title: 如何搭配 Java 使用 Azure 服務匯流排主題 | Microsoft Docs
description: 使用 Azure 中的服務匯流排主題和訂用帳戶。
services: service-bus-messaging
documentationcenter: java
author: spelluru
manager: timlt
editor: ''
ms.assetid: 63d6c8bd-8a22-4292-befc-545ffb52e8eb
ms.service: service-bus-messaging
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 09/17/2018
ms.author: spelluru
ms.openlocfilehash: 0be5f9842cd3aa90d82f3efe44451e624ed5d371
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47395673"
---
# <a name="how-to-use-service-bus-topics-and-subscriptions-with-java"></a>如何透過 Java 使用服務匯流排主題和訂用帳戶

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

本指南說明如何使用服務匯流排主題和訂閱。 相關範例是以 Java 撰寫並使用 [Azure SDK for Java][Azure SDK for Java]。 涵蓋的案例包括**建立主題和訂用帳戶**、**建立訂用帳戶篩選器**、**傳送訊息至主題**、**接收訂用帳戶的訊息**，以及**刪除主題和訂用帳戶**。

## <a name="what-are-service-bus-topics-and-subscriptions"></a>什麼是服務匯流排主題和訂用帳戶？
服務匯流排主題和訂用帳戶支援「發佈/訂閱」  訊息通訊模型。 使用主題和訂用帳戶時，分散式應用程式的元件彼此不直接通訊，相反的，他們會透過扮演中繼角色的主題來交換訊息。

![主題概念](./media/service-bus-java-how-to-use-topics-subscriptions/sb-topics-01.png)

有別於服務匯流排佇列，服務匯流排佇列中的每個訊息只會由單一取用者處理，主題和訂用帳戶採用發佈/訂閱模式，提供一對多的通訊形式。 一個主題可以登錄多個訂用帳戶。 當訊息傳送至主題時，每個訂用帳戶都可取得訊息來個別處理。

主題的訂用帳戶類似於虛擬佇列，同樣可接收已傳送到主題的訊息複本。 您可以選擇為個別訂用帳戶登錄主題的篩選規則，以篩選或限制主題的哪些訊息由哪些主題訂用帳戶接收。

服務匯流排主題和訂用帳戶可讓您擴大處理非常多使用者和應用程式上大量的訊息。

## <a name="create-a-service-namespace"></a>建立服務命名空間
若要開始在 Azure 中使用服務匯流排主題和訂用帳戶，您必須先建立*命名空間*，它能針對在應用程式內處理服務匯流排資源提供範圍容器。

若要建立命名空間：

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="configure-your-application-to-use-service-bus"></a>設定應用程式以使用服務匯流排
在建置此範例之前，請先確定您已安裝 [Azure SDK for Java][Azure SDK for Java]。 如果您使用的是 Eclipse，則可以安裝包含 Azure SDK for Java 的[適用於 Eclipse 的 Azure 工具組][Azure Toolkit for Eclipse]。 然後您可以將 **Microsoft Azure Libraries for Java** 新增至您的專案：

![](media/service-bus-java-how-to-use-topics-subscriptions/eclipselibs.png)

在 Java 檔案頂端新增下列 `import` 陳述式：

```java
import com.microsoft.windowsazure.services.servicebus.*;
import com.microsoft.windowsazure.services.servicebus.models.*;
import com.microsoft.windowsazure.core.*;
import javax.xml.datatype.*;
```

將 Azure Libraries for Java 新增至您的建置路徑，並將其納入專案部署組件中。

## <a name="create-a-topic"></a>建立主題
服務匯流排主題的管理作業可透過 **ServiceBusContract** 類別來執行。 **ServiceBusContract** 物件是利用使用權限來封裝 SAS 權杖以進行管理的適當組態所建構，而 **ServiceBusContract** 類別是唯一可與 Azure 通訊的點。

**ServiceBusService** 類別會提供建立、列舉及刪除主題的方法。 下列範例顯示如何使用 **ServiceBusService** 物件來建立名為 `TestTopic` 且命名空間為 `HowToSample` 的主題：

```java
Configuration config =
    ServiceBusConfiguration.configureWithSASAuthentication(
      "HowToSample",
      "RootManageSharedAccessKey",
      "SAS_key_value",
      ".servicebus.windows.net"
      );

ServiceBusContract service = ServiceBusService.create(config);
TopicInfo topicInfo = new TopicInfo("TestTopic");
try  
{
    CreateTopicResult result = service.createTopic(topicInfo);
}
catch (ServiceException e) {
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
```

**TopicInfo** 有相關方法可讓您設定主題的屬性 (例如，針對要在傳送至主題的訊息所套用的存留時間 (TTL) 設定預設值)。 下列範例將示範如何使用大小上限為 5 GB 的設定，來建立名為 `TestTopic` 的主題：

```java
long maxSizeInMegabytes = 5120;  
TopicInfo topicInfo = new TopicInfo("TestTopic");  
topicInfo.setMaxSizeInMegabytes(maxSizeInMegabytes);
CreateTopicResult result = service.createTopic(topicInfo);
```

您可以在 **ServiceBusContract** 物件上使用 **listTopics** 方法，來檢查服務命名空間內是否已有指定名稱的主題存在。

## <a name="create-subscriptions"></a>建立訂用帳戶
**ServiceBusService** 類別也能用來建立主題的訂用帳戶。 為訂閱命名，且能包含選擇性篩選器，以用來限制傳遞至訂閱的虛擬佇列的訊息集合。

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>使用預設 (MatchAll) 篩選器建立訂用帳戶
如果在建立新的訂用帳戶時未指定篩選器，**MatchAll** 篩選器就會是預設使用的篩選器。 使用 **MatchAll** 篩選器時，所有發佈至主題的訊息都會被置於訂用帳戶的虛擬佇列中。 下列範例將建立名為 `AllMessages` 的訂用帳戶，並使用預設的 `MatchAll` 篩選器。

```java
SubscriptionInfo subInfo = new SubscriptionInfo("AllMessages");
CreateSubscriptionResult result =
    service.createSubscription("TestTopic", subInfo);
```

### <a name="create-subscriptions-with-filters"></a>使用篩選器建立訂用帳戶
您也可以建立篩選器，讓您界定傳送至主題的哪些訊息應出現在特定主題訂用帳戶中。

訂用帳戶所支援的最具彈性篩選器類型是實作 SQL92 子集的 [SqlFilter][SqlFilter]。 SQL 篩選器會對發佈至主題之訊息的屬性運作。 如需可與 SQL 篩選器搭配使用的運算式詳細資料，請檢閱 [SqlFilter.SqlExpression][SqlFilter.SqlExpression] 語法。

以下範例將建立名為 `HighMessages` 的訂用帳戶，而且所含的 [SqlFilter][SqlFilter] 物件只會選取自訂 **MessageNumber** 屬性大於 3 的訊息：

```java
// Create a "HighMessages" filtered subscription  
SubscriptionInfo subInfo = new SubscriptionInfo("HighMessages");
CreateSubscriptionResult result = service.createSubscription("TestTopic", subInfo);
RuleInfo ruleInfo = new RuleInfo("myRuleGT3");
ruleInfo = ruleInfo.withSqlExpressionFilter("MessageNumber > 3");
CreateRuleResult ruleResult = service.createRule("TestTopic", "HighMessages", ruleInfo);
// Delete the default rule, otherwise the new rule won't be invoked.
service.deleteRule("TestTopic", "HighMessages", "$Default");
```

同樣地，下列範例將建立名為 `LowMessages` 的訂用帳戶，而且所含的 [SqlFilter][SqlFilter] 物件只會選取 **MessageNumber** 屬性小於或等於 3 的訊息：

```java
// Create a "LowMessages" filtered subscription
SubscriptionInfo subInfo = new SubscriptionInfo("LowMessages");
CreateSubscriptionResult result = service.createSubscription("TestTopic", subInfo);
RuleInfo ruleInfo = new RuleInfo("myRuleLE3");
ruleInfo = ruleInfo.withSqlExpressionFilter("MessageNumber <= 3");
CreateRuleResult ruleResult = service.createRule("TestTopic", "LowMessages", ruleInfo);
// Delete the default rule, otherwise the new rule won't be invoked.
service.deleteRule("TestTopic", "LowMessages", "$Default");
```

現在，當訊息傳送至 `TestTopic` 時，一律會將該訊息傳遞至已訂閱 `AllMessages` 訂用帳戶的接收者，並選擇性地傳遞至已訂閱 `HighMessages` 和 `LowMessages` 訂用帳戶 (視訊息內容而定) 的接收者。

## <a name="send-messages-to-a-topic"></a>傳送訊息至主題
若要將訊息傳送至服務匯流排主題，應用程式會取得 **ServiceBusContract** 物件。 下列程式碼示範如何針對先前在 `HowToSample` 命名空間內建立的 `TestTopic` 主題傳送訊息：

```java
BrokeredMessage message = new BrokeredMessage("MyMessage");
service.sendTopicMessage("TestTopic", message);
```

傳送至服務匯流排主題的訊息是 [BrokeredMessage][BrokeredMessage] 類別的執行個體。 [BrokeredMessage][BrokeredMessage]* 物件具有一組標準方法 (例如 **setLabel** 和 **TimeToLive**)、一個用來保存自訂應用程式特定屬性的字典，以及一個任意應用程式資料的主體。 應用程式可設定訊息內文，方法是將任何可序列化的物件傳遞到 [BrokeredMessage][BrokeredMessage] 的建構函式，接著系統會使用適當的 **DataContractSerializer** 來將物件序列化。 或者，也可以提供 **java.io.InputStream**。

下列範例示範如何將五個測試訊息傳送至上述程式碼片段中所取得的 `TestTopic` **MessageSender**。
請注意，迴圈反覆運算上每個訊息的 **MessageNumber** 屬性值會產生哪種變化 (此值可判斷哪些訂用帳戶會接收訊息)：

```java
for (int i=0; i<5; i++)  {
// Create message, passing a string message for the body
BrokeredMessage message = new BrokeredMessage("Test message " + i);
// Set some additional custom app-specific property
message.setProperty("MessageNumber", i);
// Send message to the topic
service.sendTopicMessage("TestTopic", message);
}
```

服務匯流排主題支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 主題中所保存的訊息數目沒有限制，但主題所保存的訊息大小總計會有限制。 此主題大小會在建立時定義，上限是 5 GB。

## <a name="how-to-receive-messages-from-a-subscription"></a>如何自訂用帳戶接收訊息
若要從訂用帳戶接收訊息，請使用 **ServiceBusContract** 物件。 接收的訊息可在兩種不同的模式下運作：**ReceiveAndDelete** 和 **PeekLock** (預設值)。

使用 **ReceiveAndDelete** 模式時，接收是一次性作業；也就是說，當服務匯流排收到訊息的讀取要求時，它會將此訊息標示為已使用，並將它傳回應用程式。 **ReceiveAndDelete** 模式是最簡單的模型，且最適合可容許在發生失敗時不處理訊息的應用程式案例。 例如，設想取用者發出接收要求，然後系統在處理此要求之前當機的案例。 因為服務匯流排已將訊息標示為已取用，所以，當應用程式重新啟動並開始重新取用訊息時，它會遺漏當機前已取用的訊息。

在 **PeekLock** 模式中，接收會變成兩階段作業，因此可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成處理訊息 (或可靠地儲存此訊息以供未來處理) 之後，它會在已接收的訊息上呼叫 **Delete**，以完成接收程序的第二個階段。 當服務匯流排看到 **Delete** 呼叫時，它會將訊息標示為已取用，並將它從主題中移除。

以下範例示範如何使用 **PeekLock** (預設模式) 來接收與處理訊息。 此範例會執行迴圈，並處理 `HighMessages` 訂用帳戶中的訊息，然後在沒有任何訊息時結束 (您也可以將其設為等待新訊息)。

```java
try
{
    ReceiveMessageOptions opts = ReceiveMessageOptions.DEFAULT;
    opts.setReceiveMode(ReceiveMode.PEEK_LOCK);

    while(true)  {
        ReceiveSubscriptionMessageResult  resultSubMsg =
            service.receiveSubscriptionMessage("TestTopic", "HighMessages", opts);
        BrokeredMessage message = resultSubMsg.getValue();
        if (message != null && message.getMessageId() != null)
        {
            System.out.println("MessageID: " + message.getMessageId());
            // Display the topic message.
            System.out.print("From topic: ");
            byte[] b = new byte[200];
            String s = null;
            int numRead = message.getBody().read(b);
            while (-1 != numRead)
            {
                s = new String(b);
                s = s.trim();
                System.out.print(s);
                numRead = message.getBody().read(b);
            }
            System.out.println();
            System.out.println("Custom Property: " +
                message.getProperty("MessageNumber"));
            // Delete message.
            System.out.println("Deleting this message.");
            service.deleteMessage(message);
        }  
        else  
        {
            System.out.println("Finishing up - no more messages.");
            break;
            // Added to handle no more messages.
            // Could instead wait for more messages to be added.
        }
    }
}
catch (ServiceException e) {
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
catch (Exception e) {
    System.out.print("Generic exception encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何處理應用程式當機與無法讀取的訊息
服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。 如果接收者應用程式因為某些原因無法處理訊息，它可以在已接收的訊息上呼叫 **unlockMessage** 方法 (而不是 **deleteMessage** 方法)。 此方法呼叫會導致服務匯流排將主題中的訊息解除鎖定，讓此訊息可由相同的取用應用程式或其他取用應用程式重新接收。

與主題內鎖定之訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，若應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

在處理訊息之後和發出 **deleteMessage** 要求之前，如果應用程式當機，則會在應用程式重新啟動時，將訊息重新傳遞給該應用程式。 此程序通常稱為**至少處理一次**，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，則應用程式開發人員應在其應用程式中加入其他邏輯，以處理重複的訊息傳遞。 您可以使用訊息的 **getMessageId** 方法來達到此目的，該方法將在各個傳遞嘗試中保持不變。

## <a name="delete-topics-and-subscriptions"></a>刪除主題和訂用帳戶
刪除主題和訂用帳戶的主要方式，是使用 **ServiceBusContract** 物件。 刪除主題也將會刪除對主題註冊的任何訂用帳戶。 您也可以個別刪除訂用帳戶。

```java
// Delete subscriptions
service.deleteSubscription("TestTopic", "AllMessages");
service.deleteSubscription("TestTopic", "HighMessages");
service.deleteSubscription("TestTopic", "LowMessages");

// Delete a topic
service.deleteTopic("TestTopic");
```

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱[服務匯流排佇列、主題和訂用帳戶][Service Bus queues, topics, and subscriptions]。

[Azure SDK for Java]: http://azure.microsoft.com/develop/java/
[Azure Toolkit for Eclipse]: ../azure-toolkit-for-eclipse.md
[Service Bus queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[SqlFilter]: /dotnet/api/microsoft.azure.servicebus.sqlfilter
[SqlFilter.SqlExpression]: /dotnet/api/microsoft.azure.servicebus.sqlfilter.sqlexpression
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage

[0]: ./media/service-bus-java-how-to-use-topics-subscriptions/sb-queues-13.png
[2]: ./media/service-bus-java-how-to-use-topics-subscriptions/sb-queues-04.png
[3]: ./media/service-bus-java-how-to-use-topics-subscriptions/sb-queues-09.png
