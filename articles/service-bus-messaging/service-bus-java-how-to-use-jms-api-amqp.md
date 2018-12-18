---
title: 如何使用 AMQP 1.0 與 Java 服務匯流排 API 搭配 | Microsoft Docs
description: 如何搭配 Azure 服務匯流排和 Advanced Message Queuing Protodol (AMQP) 1.0 使用 Java Message Service (JMS)。
services: service-bus-messaging
documentationcenter: java
author: spelluru
manager: timlt
editor: ''
ms.assetid: be766f42-6fd1-410c-b275-8c400c811519
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 08/10/2017
ms.author: spelluru
ms.openlocfilehash: bfab0c374e4b20b09167f37363fe0681144426ac
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699340"
---
# <a name="how-to-use-the-java-message-service-jms-api-with-service-bus-and-amqp-10"></a>如何搭配使用 Java 訊息服務 (JMS) API 與服務匯流排和 AMQP 1.0
進階訊息佇列通訊協定 (AMQP) 1.0 是一個有效率且可靠的有線等級訊息通訊協定，可以用來建置強大的跨平台訊息應用程式。

在服務匯流排中支援 AMQP 1.0 代表您現在能夠從一組平台中使用有效率的二進位通訊協定，來運用佇列與發佈/訂閱代理傳訊功能。 此外，您還可以建置使用混合語言、架構及作業系統所建置之元件所組成的應用程式。

本文說明如何利用常用的 Java 訊息服務 (JMS) API 標準，從 Java 應用程式使用服務匯流排傳訊功能 (佇列和發佈/訂閱主題)。 這是說明如何使用服務匯流排 .NET API 達到相同效用的[附屬文章](service-bus-amqp-dotnet.md)。 您可以同時使用這兩個指南了解使用 AMQP 1.0 的跨平台訊息。

## <a name="get-started-with-service-bus"></a>開始使用服務匯流排
本指南假設您已經有服務匯流排命名空間，其中包含名稱為 **queue1** 的佇列。 如果沒有，您可以使用 [Azure 入口網站](service-bus-create-namespace-portal.md)建立[命名空間和佇列](https://portal.azure.com)。 如需有關如何建立服務匯流排命名空間和佇列的相關詳細資訊，請參閱[開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)。

> [!NOTE]
> 分割的佇列和主題也支援 AMQP。 如需詳細資訊，請參閱[分割傳訊實體](service-bus-partitioning.md)及[服務匯流排分割佇列和主題的 AMQP 1.0 支援](service-bus-partitioned-queues-and-topics-amqp-overview.md)。
> 
> 

## <a name="downloading-the-amqp-10-jms-client-library"></a>下載 AMQP 1.0 JMS 用戶端程式庫
如需可從哪裡下載最新版 Apache Qpid JMS AMQP 1.0 用戶端程式庫的相關資訊，請造訪 [https://qpid.apache.org/download.html](https://qpid.apache.org/download.html)。

您使用服務匯流排建立和執行 JMS 應用程式時，必須從 Apache Qpid JMS AMQP 1.0 散發封裝將下列 4 個 JAR 檔加入 Java CLASSPATH：

* geronimo-jms\_1.1\_spec-1.0.jar
* qpid-amqp-1-0-client-[version].jar
* qpid-amqp-1-0-client-jms-[version].jar
* qpid-amqp-1-0-common-[version].jar

## <a name="coding-java-applications"></a>編寫 Java 應用程式
### <a name="java-naming-and-directory-interface-jndi"></a>Java 命名及目錄介面 (JNDI)
JMS 使用 Java 命名及目錄介面 (JNDI) 建立邏輯名稱與實際名稱之間的區別。 使用 JNDI 可以解析兩種 JMS 物件：ConnectionFactory 和 Destination。 JNDI 使用提供者模型，您可以在其中插入不同的目錄服務處理名稱解析作業。 Apache Qpid JMS AMQP 1.0 程式庫提供使用下列格式的內容檔案設定的簡單內容檔案型 JNDI 提供者：

```
# servicebus.properties - sample JNDI configuration

# Register a ConnectionFactory in JNDI using the form:
# connectionfactory.[jndi_name] = [ConnectionURL]
connectionfactory.SBCF = amqps://[SASPolicyName]:[SASPolicyKey]@[namespace].servicebus.windows.net

# Register some queues in JNDI using the form
# queue.[jndi_name] = [physical_name]
# topic.[jndi_name] = [physical_name]
queue.QUEUE = queue1
```

#### <a name="configure-the-connectionfactory"></a>設定 ConnectionFactory
在 Qpid 屬性檔案 JNDI 提供者中用來定義 **ConnectionFactory** 的項目使用下列格式：

```
connectionfactory.[jndi_name] = [ConnectionURL]
```

其中 **[jndi_name]** 及 **[ConnectionURL]** 具有下列意義：

* **[jndi_name]**：ConnectionFactory 的邏輯名稱。 這是使用 JNDI IntialContext.lookup() 方法在 Java 應用程式中解析的名稱。
* **[ConnectionURL]**：將包含所需資訊的 JMS 程式庫提供給 AMQP 訊息代理程式的 URL。

**ConnectionURL** 的格式如下：

```
amqps://[SASPolicyName]:[SASPolicyKey]@[namespace].servicebus.windows.net
```
其中 **[namespace]**、**[SASPolicyName]** 和 **[SASPolicyKey]** 具有下列意義：

* **[namespace]**：服務匯流排命名空間。
* **[SASPolicyName]**：佇列共用存取簽章原則名稱。
* **[SASPolicyKey]**：佇列共用存取簽章原則金鑰。

> [!NOTE]
> 您必須手動使用 URL 將密碼編碼。 實用的 URL 編碼公用程式可於 [http://www.w3schools.com/tags/ref_urlencode.asp](http://www.w3schools.com/tags/ref_urlencode.asp) 取得。
> 
> 

#### <a name="configure-destinations"></a>設定目的地
在 Qpid 屬性檔案 JNDI 提供者中用來定義目的地的項目使用下列格式：

```
queue.[jndi_name] = [physical_name]
```

或

```
topic.[jndi_name] = [physical_name]
```

其中 **[jndi\_name]** 及 **[physical\_name]** 具有下列意義：

* **[jndi_name]**：目的地的邏輯名稱。 這是使用 JNDI IntialContext.lookup() 方法在 Java 應用程式中解析的名稱。
* **[physical_name]**：應用程式傳送或接收訊息的服務匯流排實體名稱。

> [!NOTE]
> 從服務匯流排主題訂用帳戶收到在 JNDI 中指定的實體名稱應該是主題的名稱。 以 JMS 應用程式程式碼建立持續性訂用帳戶時，將建立訂用帳戶名稱。 [服務匯流排 AMQP 1.0 開發人員指南](service-bus-amqp-dotnet.md)提供處理 JMS 服務匯流排主題的詳細資料。
> 
> 

### <a name="write-the-jms-application"></a>撰寫 JMS 應用程式
對於服務匯流排使用 JMS 時，不需要特別 API 或選項。 不過，後續將說明一些限制。 和任何 JMS 應用程式一樣，首先需要設定 JNDI 環境，才能夠解析 **ConnectionFactory** 和目的地。

#### <a name="configure-the-jndi-initialcontext"></a>設定 JNDI InitialContext
將組態資訊的雜湊表傳遞到 javax.naming.InitialContext 類別的建構函式，將設定 JNDI 環境。 雜湊表中的兩個所需項目是 Initial Context Factory 和 Provider URL 的類別名稱。 下列程式碼顯示如何使用名稱為 **servicebus.properties** 的內容檔案，設定 JNDI 環境使用 Qpid 內容檔案型 JNDI 提供者。

```java
Hashtable<String, String> env = new Hashtable<String, String>(); 
env.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
env.put(Context.PROVIDER_URL, "servicebus.properties"); 
InitialContext context = new InitialContext(env);
``` 

### <a name="a-simple-jms-application-using-a-service-bus-queue"></a>使用服務匯流排佇列的簡單 JMS 應用程式
下列範例程式將 JMS TextMessages 傳送到 JNDI 邏輯名稱為 QUEUE 的服務匯流排佇列，並收到傳回的訊息。

```java
// SimpleSenderReceiver.java

import javax.jms.*;
import javax.naming.Context;
import javax.naming.InitialContext;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Hashtable;
import java.util.Random;

public class SimpleSenderReceiver implements MessageListener {
    private static boolean runReceiver = true;
    private Connection connection;
    private Session sendSession;
    private Session receiveSession;
    private MessageProducer sender;
    private MessageConsumer receiver;
    private static Random randomGenerator = new Random();

    public SimpleSenderReceiver() throws Exception {
        // Configure JNDI environment
        Hashtable<String, String> env = new Hashtable<String, String>();
        env.put(Context.INITIAL_CONTEXT_FACTORY, 
                   "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory");
        env.put(Context.PROVIDER_URL, "servicebus.properties");
        Context context = new InitialContext(env);

        // Look up ConnectionFactory and Queue
        ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
        Destination queue = (Destination) context.lookup("QUEUE");

        // Create Connection
        connection = cf.createConnection();

        // Create sender-side Session and MessageProducer
        sendSession = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        sender = sendSession.createProducer(queue);

        if (runReceiver) {
            // Create receiver-side Session, MessageConsumer,and MessageListener
            receiveSession = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
            receiver = receiveSession.createConsumer(queue);
            receiver.setMessageListener(this);
            connection.start();
        }
    }

    public static void main(String[] args) {
        try {

            if ((args.length > 0) && args[0].equalsIgnoreCase("sendonly")) {
                runReceiver = false;
            }

            SimpleSenderReceiver simpleSenderReceiver = new SimpleSenderReceiver();
            System.out.println("Press [enter] to send a message. Type 'exit' + [enter] to quit.");
            BufferedReader commandLine = new java.io.BufferedReader(new InputStreamReader(System.in));

            while (true) {
                String s = commandLine.readLine();
                if (s.equalsIgnoreCase("exit")) {
                    simpleSenderReceiver.close();
                    System.exit(0);
                } else {
                    simpleSenderReceiver.sendMessage();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private void sendMessage() throws JMSException {
        TextMessage message = sendSession.createTextMessage();
        message.setText("Test AMQP message from JMS");
        long randomMessageID = randomGenerator.nextLong() >>>1;
        message.setJMSMessageID("ID:" + randomMessageID);
        sender.send(message);
        System.out.println("Sent message with JMSMessageID = " + message.getJMSMessageID());
    }

    public void close() throws JMSException {
        connection.close();
    }

    public void onMessage(Message message) {
        try {
            System.out.println("Received message with JMSMessageID = " + message.getJMSMessageID());
            message.acknowledge();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}    
```

### <a name="run-the-application"></a>執行應用程式
執行應用程式將產生表單的輸出：

```
> java SimpleSenderReceiver
Press [enter] to send a message. Type 'exit' + [enter] to quit.

Sent message with JMSMessageID = ID:2867600614942270318
Received message with JMSMessageID = ID:2867600614942270318

Sent message with JMSMessageID = ID:7578408152750301483
Received message with JMSMessageID = ID:7578408152750301483

Sent message with JMSMessageID = ID:956102171969368961
Received message with JMSMessageID = ID:956102171969368961
exit
```

## <a name="cross-platform-messaging-between-jms-and-net"></a>JMS 與 .NET 之間的跨平台訊息
本指南說明使用 JMS 傳送和接收服務匯流排的訊息。 不過，AMQP 1.0 的其中一個主要優點是能夠從不同語言撰寫的元件建立應用程式，並確實完整交換訊息。

使用上述的範例 JMS 應用程式和取自隨附文章[搭配使用 .NET 的服務匯流排與 AMQP 1.0](service-bus-amqp-dotnet.md) 的類似 .NET 應用程式，即可交換 .NET 與 Java 之間的訊息。 如需使用服務匯流排與 AMQP 1.0 傳送跨平台訊息的詳細資訊，請參閱本文。

### <a name="jms-to-net"></a>JMS 到 .NET
示範 JMS 到 .NET 的訊息：

* 不使用任何命令列引數啟動 .NET 範例應用程式。
* 使用「sendonly」命令列引數啟動 Java 範例應用程式。 在此模式中，應用程式將不會接收來自佇列的訊息，它只會傳送訊息。
* 在 Java 應用程式主控台多次按下 **Enter** 鍵，這將傳送訊息。
* 這些訊息將由 .NET 應用程式所接收。

#### <a name="output-from-jms-application"></a>JMS 應用程式的輸出
```
> java SimpleSenderReceiver sendonly
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Sent message with JMSMessageID = ID:4364096528752411591
Sent message with JMSMessageID = ID:459252991689389983
Sent message with JMSMessageID = ID:1565011046230456854
exit
```

#### <a name="output-from-net-application"></a>.NET 應用程式的輸出
```
> SimpleSenderReceiver.exe    
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Received message with MessageID = 4364096528752411591
Received message with MessageID = 459252991689389983
Received message with MessageID = 1565011046230456854
exit
```

### <a name="net-to-jms"></a>.NET 到 JMS
示範從 .NET 到 JMS 的傳訊操作：

* 使用「sendonly」命令列引數啟動 .NET 範例應用程式。 在此模式中，應用程式將不會接收來自佇列的訊息，它只會傳送訊息。
* 不使用任何命令列引數啟動 Java 範例應用程式。
* 在 .NET 應用程式主控台多次按下 **Enter** 鍵，這將傳送訊息。
* 這些訊息將由 Java 應用程式所接收。

#### <a name="output-from-net-application"></a>.NET 應用程式的輸出
```
> SimpleSenderReceiver.exe sendonly
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Sent message with MessageID = d64e681a310a48a1ae0ce7b017bf1cf3    
Sent message with MessageID = 98a39664995b4f74b32e2a0ecccc46bb
Sent message with MessageID = acbca67f03c346de9b7893026f97ddeb
exit
```

#### <a name="output-from-jms-application"></a>JMS 應用程式的輸出
```
> java SimpleSenderReceiver    
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Received message with JMSMessageID = ID:d64e681a310a48a1ae0ce7b017bf1cf3
Received message with JMSMessageID = ID:98a39664995b4f74b32e2a0ecccc46bb
Received message with JMSMessageID = ID:acbca67f03c346de9b7893026f97ddeb
exit
```

## <a name="unsupported-features-and-restrictions"></a>不支援的功能和限制
對於服務匯流排使用 JMS 而不使用 AMQP 1.0 會有下列限制：

* 對於各個**工作階段**僅允許一個 **MessageProducer** 或 **MessageConsumer**。 如果您需要在應用程式中建立多個 **MessageProducers** 或 **MessageConsumers**，請分別建立專用的**工作階段**。
* 目前不支援可變更的主題訂閱。
* 目前不支援 **MessageSelectors**。
* 目前不支援 **TemporaryQueue** 和 **TemporaryTopic** 這些暫時目的地，也不支援用到這些目的地的 **QueueRequestor** 和 **TopicRequestor** API。
* 不支援交易式工作階段和分散式交易。

## <a name="summary"></a>總結
本作法指南說明如何以常用的 JMS API 和 AMQP 1.0 從 Java 使用服務匯流排代理訊息功能 (佇列和發佈/訂閱主題)。

您也可以使用包括 .NET、C、Python 和 PHP 在內的其他語言所撰寫的 Service Bus AMQP 1.0。 使用這些不同的語言撰寫的元件可使用服務匯流排中的 AMQP 1.0 支援確實完整交換訊息。

## <a name="next-steps"></a>後續步驟
* [Azure 服務匯流排中的 AMQP 1.0 支援](service-bus-amqp-overview.md)
* [如何透過服務匯流排 .NET API 使用 AMQP 1.0](service-bus-dotnet-advanced-message-queuing.md)
* [服務匯流排 AMQP 1.0 開發人員指南](service-bus-amqp-dotnet.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [Java 開發人員中心](https://azure.microsoft.com/develop/java/)

