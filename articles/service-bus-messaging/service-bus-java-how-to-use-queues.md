---
title: 如何搭配 Java 使用 Azure 服務匯流排佇列 | Microsoft Docs
description: 了解如何使用 Azure 中的服務匯流排佇列。 程式碼範例以 Java 撰寫。
services: service-bus-messaging
documentationcenter: java
author: spelluru
manager: timlt
ms.assetid: f701439c-553e-402c-94a7-64400f997d59
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 09/13/2018
ms.author: spelluru
ms.openlocfilehash: 804e0dd4b510b40c1ebbc5790308a429c2715724
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45573309"
---
# <a name="how-to-use-service-bus-queues-with-java"></a>如何將服務匯流排佇列搭配 Java 使用
[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

此文章說明如何使用服務匯流排佇列。 相關範例是以 Java 撰寫，並且使用 [Azure SDK for Java][Azure SDK for Java]。 此文章說明的案例包括**建立佇列**、**傳送並接收訊息**，以及**刪除佇列**。

> [!NOTE]
> 您可以在 GitHub 的 [azure-service-bus repository](https://github.com/Azure/azure-service-bus/tree/master/samples/Java) 中找到 Java 範例。

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

## <a name="create-a-service-bus-namespace"></a>建立服務匯流排命名空間
[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="create-a-service-bus-queue"></a>建立服務匯流排佇列
[!INCLUDE [service-bus-create-queue-portal](../../includes/service-bus-create-queue-portal.md)]

## <a name="configure-your-application-to-use-service-bus"></a>設定應用程式以使用服務匯流排
在建置此範例之前，請先確定您已安裝 [Azure SDK for Java][Azure SDK for Java]。 如果您使用的是 Eclipse，則可以安裝包含 Azure SDK for Java 的[適用於 Eclipse 的 Azure 工具組][Azure Toolkit for Eclipse]。 然後您可以將 **Microsoft Azure Libraries for Java** 新增至您的專案：

![](./media/service-bus-java-how-to-use-queues/eclipselibs.png)

在 Java 檔案頂端新增下列 `import` 陳述式：

```java
// Include the following imports to use Service Bus APIs
import com.google.gson.reflect.TypeToken;
import com.microsoft.azure.servicebus.*;
import com.microsoft.azure.servicebus.primitives.ConnectionStringBuilder;
import com.google.gson.Gson;

import static java.nio.charset.StandardCharsets.*;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.*;

import org.apache.commons.cli.*;

```

## <a name="send-messages-to-a-queue"></a>傳送訊息至佇列
若要將訊息傳送至服務匯流排佇列，您的應用程式會將 **QueueClient** 物件具現化，並以非同步方式傳送訊息。 下列程式碼示範如何針對透過入口網站建立的佇列傳送訊息。

```java
public void run() throws Exception {
    // Create a QueueClient instance and then asynchronously send messages.
    // Close the sender once the send operation is complete.
    QueueClient sendClient = new QueueClient(new ConnectionStringBuilder(ConnectionString, QueueName), ReceiveMode.PEEKLOCK);
    this.sendMessageAsync(sendClient).thenRunAsync(() -> sendClient.closeAsync());

    sendClient.close();
}

    CompletableFuture<Void> sendMessagesAsync(QueueClient sendClient) {
        List<HashMap<String, String>> data =
                GSON.fromJson(
                        "[" +
                                "{'name' = 'Einstein', 'firstName' = 'Albert'}," +
                                "{'name' = 'Heisenberg', 'firstName' = 'Werner'}," +
                                "{'name' = 'Curie', 'firstName' = 'Marie'}," +
                                "{'name' = 'Hawking', 'firstName' = 'Steven'}," +
                                "{'name' = 'Newton', 'firstName' = 'Isaac'}," +
                                "{'name' = 'Bohr', 'firstName' = 'Niels'}," +
                                "{'name' = 'Faraday', 'firstName' = 'Michael'}," +
                                "{'name' = 'Galilei', 'firstName' = 'Galileo'}," +
                                "{'name' = 'Kepler', 'firstName' = 'Johannes'}," +
                                "{'name' = 'Kopernikus', 'firstName' = 'Nikolaus'}" +
                                "]",
                        new TypeToken<List<HashMap<String, String>>>() {}.getType());

        List<CompletableFuture> tasks = new ArrayList<>();
        for (int i = 0; i < data.size(); i++) {
            final String messageId = Integer.toString(i);
            Message message = new Message(GSON.toJson(data.get(i), Map.class).getBytes(UTF_8));
            message.setContentType("application/json");
            message.setLabel("Scientist");
            message.setMessageId(messageId);
            message.setTimeToLive(Duration.ofMinutes(2));
            System.out.printf("\nMessage sending: Id = %s", message.getMessageId());
            tasks.add(
                    sendClient.sendAsync(message).thenRunAsync(() -> {
                        System.out.printf("\n\tMessage acknowledged: Id = %s", message.getMessageId());
                    }));
        }
        return CompletableFuture.allOf(tasks.toArray(new CompletableFuture<?>[tasks.size()]));
    }

```

傳送至 (和接收自) 服務匯流排佇列的訊息是 [Message](/java/api/com.microsoft.azure.servicebus._message?view=azure-java-stable) 類別的執行個體。 Message 物件有一組標準屬性 (例如 Label 和 TimeToLive)、一個用來保存自訂應用程式專用屬性的字典，以及一堆任意應用程式資料。 應用程式可以設定訊息本文，方法是將任何可序列化物件傳遞到 Message 的建構函式，接著系統便會使用適當的序列化程式將物件序列化。 此外，您也可以提供 **java.IO.InputStream** 物件。


服務匯流排佇列支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 佇列中所保存的訊息數目沒有限制，但佇列所保存的訊息大小總計會有最高限制。 此佇列大小會在建立時定義，上限是 5 GB。

## <a name="receive-messages-from-a-queue"></a>從佇列接收訊息
自佇列接收訊息的主要方式是使用 **ServiceBusContract** 物件。 接收的訊息可在兩種不同的模式下運作：**ReceiveAndDelete** 和 **PeekLock**。

使用 **ReceiveAndDelete** 模式時，接收是一次性作業；也就是說，當服務匯流排在佇列中收到訊息的讀取要求時，它會將此訊息標示為已使用，並將它傳回應用程式。 **ReceiveAndDelete** 模式 (這是預設模式) 是最簡單的模型，且最適合可容許在發生失敗時不處理訊息的應用程式案例。 若要了解這一點，請考慮取用者發出接收要求，接著系統在處理此要求之前當機的案例。
因為服務匯流排已將訊息標示為已取用，所以，當應用程式重新啟動並開始重新取用訊息時，它會遺漏當機前已取用的訊息。

在 **PeekLock** 模式中，接收會變成兩階段作業，因此可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成處理訊息 (或可靠地儲存此訊息以供未來處理) 之後，它會在已接收的訊息上呼叫 **Delete**，以完成接收程序的第二個階段。 當服務匯流排看到 **Delete** 呼叫時，它會將訊息標示為已取用，並將它從佇列中移除。

以下範例示範如何使用 **PeekLock** 模式 (非預設模式) 來接收與處理訊息。 下列範例會建立一個無限迴圈，並處理 `TestQueue` 的訊息：

```java
    public void run() throws Exception {
        // Create a QueueClient instance for receiving using the connection string builder
        // We set the receive mode to "PeekLock", meaning the message is delivered
        // under a lock and must be acknowledged ("completed") to be removed from the queue
        QueueClient receiveClient = new QueueClient(new ConnectionStringBuilder(ConnectionString, QueueName), ReceiveMode.PEEKLOCK);
        this.registerReceiver(receiveClient);

        // shut down receiver to close the receive loop
        receiveClient.close();
    }
    void registerReceiver(QueueClient queueClient) throws Exception {
        // register the RegisterMessageHandler callback
        queueClient.registerMessageHandler(new IMessageHandler() {
        // callback invoked when the message handler loop has obtained a message
            public CompletableFuture<Void> onMessageAsync(IMessage message) {
            // receives message is passed to callback
                if (message.getLabel() != null &&
                    message.getContentType() != null &&
                    message.getLabel().contentEquals("Scientist") &&
                    message.getContentType().contentEquals("application/json")) {

                        byte[] body = message.getBody();
                        Map scientist = GSON.fromJson(new String(body, UTF_8), Map.class);

                        System.out.printf(
                            "\n\t\t\t\tMessage received: \n\t\t\t\t\t\tMessageId = %s, \n\t\t\t\t\t\tSequenceNumber = %s, \n\t\t\t\t\t\tEnqueuedTimeUtc = %s," +
                            "\n\t\t\t\t\t\tExpiresAtUtc = %s, \n\t\t\t\t\t\tContentType = \"%s\",  \n\t\t\t\t\t\tContent: [ firstName = %s, name = %s ]\n",
                            message.getMessageId(),
                            message.getSequenceNumber(),
                            message.getEnqueuedTimeUtc(),
                            message.getExpiresAtUtc(),
                            message.getContentType(),
                            scientist != null ? scientist.get("firstName") : "",
                            scientist != null ? scientist.get("name") : "");
                    }
                    return CompletableFuture.completedFuture(null);
                }

                // callback invoked when the message handler has an exception to report
                public void notifyException(Throwable throwable, ExceptionPhase exceptionPhase) {
                    System.out.printf(exceptionPhase + "-" + throwable.getMessage());
                }
        },
        // 1 concurrent call, messages are auto-completed, auto-renew duration
        new MessageHandlerOptions(1, true, Duration.ofMinutes(1)));
    }

```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何處理應用程式當機與無法讀取的訊息
服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。 如果接收者應用程式因為某些原因無法處理訊息，它可以在已接收的訊息上呼叫 **unlockMessage** 方法 (而不是 **deleteMessage** 方法)。 這將導致服務匯流排將佇列中的訊息解除鎖定，讓此訊息可以被相同取用應用程式或其他取用應用程式重新接收。

與在佇列內鎖定訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

在處理訊息之後和發出 **deleteMessage** 要求之前，如果應用程式當機，則會在應用程式重新啟動時，將訊息重新傳遞給該應用程式。 這通常稱為 *至少處理一次*，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，則應用程式開發人員應在其應用程式中加入其他邏輯，以處理重複的訊息傳遞。 通常您可使用訊息的 **getMessageId** 方法來達到此目的，該方法將在各個傳遞嘗試中保持不變。

## <a name="next-steps"></a>後續步驟
既然您已了解「服務匯流排」佇列的基本概念，請參閱[佇列、主題和訂用帳戶][Queues, topics, and subscriptions]來取得詳細資訊。

如需詳細資訊，請參閱 [Java 開發人員中心](https://azure.microsoft.com/develop/java/)。

[Azure SDK for Java]: http://azure.microsoft.com/develop/java/
[Azure Toolkit for Eclipse]: https://msdn.microsoft.com/library/azure/hh694271.aspx
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
