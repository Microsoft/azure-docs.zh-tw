---
title: 教學課程 - 透過 Azure CLI 使用發佈/訂閱通道和主題篩選來更新零售庫存商品 | Microsoft Docs
description: 在本教學課程中，您將了解如何從主題和訂用帳戶傳送及接收訊息，以及如何使用 Azure CLI 新增和使用篩選規則
services: service-bus-messaging
author: spelluru
manager: timlt
ms.author: spelluru
ms.date: 05/22/2018
ms.topic: tutorial
ms.service: service-bus-messaging
ms.custom: mvc
ms.openlocfilehash: 07ad8fc873a483b5d71d7ddd21f8f2a820bbfadc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46982370"
---
# <a name="tutorial-update-inventory-using-cli-and-topicssubscriptions"></a>教學課程：使用 CLI 和主題/訂用帳戶來更新庫存

Microsoft Azure 服務匯流排是一項多租用戶雲端傳訊服務，可在應用程式和服務之間傳送資訊。 非同步作業可讓您進行靈活的代理傳訊，搭配結構化的先進先出 (FIFO) 傳訊及發佈/訂閱功能。 本教學課程說明如何透過使用 Azure CLI 和 Java 的發佈/訂閱通道，在零售庫存案例中使用服務匯流排主題和訂用帳戶。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 使用 Azure CLI 建立服務匯流排主題，並建立該主題的一或多個訂用帳戶
> * 使用 Azure CLI 新增主題篩選
> * 建立含有不同內容的兩個訊息
> * 傳送訊息，並確認它們送達預期的訂用帳戶中
> * 從訂用帳戶接收訊息

此案例的範例是多個零售商店的庫存商品更新。 在此案例中，每個 (或每組) 商店都會收到預定要傳送給他們的訊息，用以更新其商品。 本教學課程將說明如何使用訂用帳戶和篩選實作此案例。 首先，您會以 3 個訂用帳戶建立主題、新增的某些規則和篩選條件，然後從主題和訂用帳戶來傳送和接收訊息。

![主題](./media/service-bus-tutorial-topics-subscriptions-cli/about-service-bus-topic.png)

如果您沒有 Azure 訂用帳戶，您可以在開始前建立[免費帳戶][]。

## <a name="prerequisites"></a>必要條件

若要使用 Java 開發服務匯流排應用程式，您必須安裝下列項目：

- 最新版本的 [Java Development Kit](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)。
- [Azure CLI](https://docs.microsoft.com/cli/azure)
- [Apache Maven](https://maven.apache.org) 3.0 版或更高版本。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="service-bus-topics-and-subscriptions"></a>服務匯流排主題和訂用帳戶

[主題的每個訂用帳戶](service-bus-messaging-overview.md#topics)都會接收到每則訊息的複本。 主題在通訊協定和語意上都與服務匯流排佇列完全相容。 服務匯流排主題支援具有篩選條件、並且可透過選用動作來設定或修改訊息屬性的多種選取規則。 每當有規則符合時，就會產生一則訊息。 若要深入了解規則、篩選條件和動作，請進入此[連結](topic-filters.md)。

## <a name="sign-in-to-azure"></a>登入 Azure

CLI 安裝後，請開啟命令提示字元並發出下列命令，以登入 Azure。 如果您使用 Cloud Shell，則不需要執行下列步驟：

1. 如果您在本機使用 Azure CLI，請執行下列命令以登入 Azure。 如果您在 Cloud Shell 中執行這些命令，則不需要執行此登入步驟：

   ```azurecli-interactive
   az login
   ```

2. 將目前的訂用帳戶內容設定為您想要使用的 Azure 訂用帳戶：

   ```azurecli-interactive
   az account set --subscription Azure_subscription_name
   ```

## <a name="use-cli-to-provision-resources"></a>使用 CLI 佈建資源

發出下列命令以佈建服務匯流排資源。 請務必將所有預留位置取代為適當的值。

```azurecli-interactive
# Create a resource group
az group create --name myResourcegroup --location eastus

# Create a Service Bus messaging namespace with a unique name
namespaceName=myNameSpace$RANDOM
az servicebus namespace create \
   --resource-group myResourceGroup \
   --name $namespaceName \
   --location eastus

# Create a Service Bus topic
az servicebus topic create --resource-group myResourceGroup \
   --namespace-name $namespaceName \
   --name myTopic

# Create subscription 1 to the topic
az servicebus subscription create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --name S1

# Create filter 1 - use custom properties
az servicebus rule create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S1 --name MyFilter --filter-sql-expression "StoreId IN ('Store1','Store2','Store3')"

# Create filter 2 - use custom properties
az servicebus rule create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S1 --name MySecondFilter --filter-sql-expression "StoreId = 'Store4'"

# Create subscription 2
az servicebus subscription create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --name S2

# Create filter 3 - use message header properties via IN list and 
# combine with custom properties.
az servicebus rule create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S2 --name MyFilter --filter-sql-expression "sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'"

# Create subscription 3
az servicebus subscription create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --name S3

# Create filter 4 - Get everything except messages for subscription 1 and 2. 
# Also modify and add an action; in this case set the label to a specified value. 
# Assume those stores might not be part of your main store, so you only add 
# specific items to them. For that, you flag them specifically.
az servicebus rule create --resource-group DemoGroup --namespace-name DemoNamespaceSB --topic-name tutorialtest1
 --subscription-name S3 --name MyFilter --filter-sql-expression "sys.To NOT IN ('Store1','Store2','Store3','Store4','Sto
re5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Stor
e8')" --action-sql-expression "SET sys.Label = 'SalesEvent'"

# Get the connection string
connectionString=$(az servicebus namespace authorization-rule keys list \
   --resource-group myResourceGroup \
   --namespace-name  $namespaceName \
   --name RootManageSharedAccessKey \
   --query primaryConnectionString --output tsv)
```

在最後一個命令執行之後，請將連接字串與您選取的佇列名稱複製並貼到暫存位置，例如 [記事本]。 您在下一個步驟將會用到這些資料。

## <a name="create-filter-rules-on-subscriptions"></a>在訂用帳戶上建立篩選規則

佈建命名空間和主題/訂用帳戶，且您已擁有必要的認證之後，您即可在訂用帳戶上建立篩選規則，然後傳送和接收訊息。 您可以在[此 GitHub 範例資料夾](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/quickstarts-and-tutorials/tutorial-topics-subscriptions-filters-java/src/main/java/com/microsoft/azure/)中查看程式碼。

## <a name="send-and-receive-messages"></a>傳送及接收訊息

1. 確定 Cloud Shell 已開啟並顯示 Bash 提示字元。

2. 發出下列命令，以複製[服務匯流排 GitHub 存放庫](https://github.com/Azure/azure-service-bus/)︰

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. 瀏覽至範例資料夾 `azure-service-bus/samples/Java/quickstarts-and-tutorials/quickstart-java/tutorial-topics-subscriptions-filters-java`。 請注意，Bash 殼層中的命令須區分大小寫，且路徑分隔符號必須是正斜線。

3. 發出下列命令以建置應用程式：
   
   ```shell
   mvn clean package -DskipTests
   ```
4. 若要執行程式，請發出下列命令。 請務必將預留位置取代為您在上一個步驟中取得的連接字串和主題名稱：

   ```shell
  java -jar .\target\tutorial-topics-subscriptions-filters-1.0.0-jar-with-dependencies.jar -c "myConnectionString" -t "myTopicName"
   ```

   觀察 10 個傳送至主題，和後續接收自個別訂用帳戶的訊息：

   ![程式輸出](./media/service-bus-tutorial-topics-subscriptions-cli/service-bus-tutorial-topics-subscriptions-cli.png)

## <a name="clean-up-resources"></a>清除資源

執行下列命令以移除資源群組、命名空間和所有相關資源：

```azurecli-interactive
az group delete --resource-group my-resourcegroup
```

## <a name="understand-the-sample-code"></a>了解範例程式碼

本節將詳細說明範例程式碼的功能。

### <a name="get-connection-string-and-queue"></a>取得連接字串和佇列

首先，程式碼會宣告一組變數，使程式完成其餘的執行：

```java
    public String ConnectionString = null;
    public String TopicName = null;
    static final String[] Subscriptions = {"S1","S2","S3"};
    static final String[] Store = {"Store1","Store2","Store3","Store4","Store5","Store6","Store7","Store8","Store9","Store10"};
    static final String SysField = "sys.To";
    static final String CustomField = "StoreId";    
    int NrOfMessagesPerStore = 1; // Send at least 1.
```

連接字串和主題名稱是唯一透過命令列參數新增並傳遞至 `main()` 的值。 實際的程式碼執行會在 `run()` 方法中觸發並傳送，然後從主題接收訊息：

```java
public static void main(String[] args) {
        TutorialTopicsSubscriptionsFilters app = new TutorialTopicsSubscriptionsFilters();
        try {
            app.runApp(args);
            app.run();
        } catch (Exception e) {
            System.out.printf("%s", e.toString());
        }
        System.exit(0);
    }
}

public void run() throws Exception {
        // Send sample messages.
        this.sendMessagesToTopic();

        // Receive messages from subscriptions.
        this.receiveAllMessages();
}
```

### <a name="create-topic-client-to-send-messages"></a>建立主題用戶端以傳送訊息

若要傳送和接收訊息，`sendMessagesToTopic()` 方法會建立主題用戶端執行個體，而此執行個體會使用連接字串和主題名稱，然後呼叫另一個方法以傳送訊息：

```java
public void sendMessagesToTopic() throws Exception, ServiceBusException {
         // Create client for the topic.
        TopicClient topicClient = new TopicClient(new ConnectionStringBuilder(ConnectionString, TopicName));

        // Create a message sender from the topic client.

        System.out.printf("\nSending orders to topic.\n");

        // Now we can start sending orders.
        CompletableFuture.allOf(
                SendOrders(topicClient,Store[0]),
                SendOrders(topicClient,Store[1]),
                SendOrders(topicClient,Store[2]),
                SendOrders(topicClient,Store[3]),
                SendOrders(topicClient,Store[4]),
                SendOrders(topicClient,Store[5]),
                SendOrders(topicClient,Store[6]),
                SendOrders(topicClient,Store[7]),
                SendOrders(topicClient,Store[8]),
                SendOrders(topicClient,Store[9])                
        ).join();

        System.out.printf("\nAll messages sent.\n");
    }

     public CompletableFuture<Void> SendOrders(TopicClient topicClient, String store) throws Exception {

        for(int i = 0;i<NrOfMessagesPerStore;i++) {
            Random r = new Random();
            final Item item = new Item(r.nextInt(5),r.nextInt(5),r.nextInt(5));         
            IMessage message = new Message(GSON.toJson(item,Item.class).getBytes(UTF_8)); 
            // We always set the Sent to field
            message.setTo(store);    
            final String StoreId = store;
            Double priceToString = item.getPrice();
            final String priceForPut = priceToString.toString();
            message.setProperties(new HashMap<String, String>() {{
                // Additionally we add a customer store field. In reality you would use sys.To or a customer property but not both. 
                // This is just for demo purposes.
                put("StoreId", StoreId);
                // Adding more potential filter / rule and action able fields
                put("Price", priceForPut);
                put("Color", item.getColor());
                put("Category", item.getItemCategory());
            }});
                        
            System.out.printf("Sent order to Store %s. Price=%f, Color=%s, Category=%s\n", StoreId, item.getPrice(), item.getColor(), item.getItemCategory());            
            topicClient.sendAsync(message);
        }
               
        return new CompletableFuture().completedFuture(null);         
    }
```

### <a name="receive-messages-from-the-individual-subscriptions"></a>從個別的訂用帳戶接收訊息

`receiveAllMessages()` 方法會呼叫 `receiveAllMessageFromSubscription()` 方法，繼而為每個呼叫建立一個訂用帳戶用戶端，並從個別的訂用帳戶接收訊息：

```java
public void receiveAllMessages() throws Exception {     
    System.out.printf("\nStart Receiving Messages.\n");
    
    CompletableFuture.allOf(
            receiveAllMessageFromSubscription(Subscriptions[0]),
            receiveAllMessageFromSubscription(Subscriptions[1]),
            receiveAllMessageFromSubscription(Subscriptions[2]) 
            ).join();
}

public CompletableFuture<Void> receiveAllMessageFromSubscription(String subscription) throws Exception {
        
        int receivedMessages = 0;

        // Create subscription client.
        IMessageReceiver subscriptionClient = ClientFactory.createMessageReceiverFromConnectionStringBuilder(new ConnectionStringBuilder(ConnectionString, TopicName+"/subscriptions/"+ subscription), ReceiveMode.PEEKLOCK);

        // Create a receiver from the subscription client and receive all messages.
        System.out.printf("\nReceiving messages from subscription %s.\n\n", subscription);

        while (true)
        {
            // This will make the connection wait for N seconds if new messages are available. 
            // If no additional messages come we close the connection. This can also be used to realize long polling.
            // In case of long polling you would obviously set it more to e.g. 60 seconds.
            IMessage receivedMessage = subscriptionClient.receive(Duration.ofSeconds(1));
            if (receivedMessage != null)
            {
                if ( receivedMessage.getProperties() != null ) {                                                                                
                    System.out.printf("StoreId=%s\n", receivedMessage.getProperties().get("StoreId"));                                                                                          
                    
                    // Show the label modified by the rule action
                    if(receivedMessage.getLabel() != null)
                        System.out.printf("Label=%s\n", receivedMessage.getLabel());   
                }
                
                byte[] body = receivedMessage.getBody();
                Item theItem = GSON.fromJson(new String(body, UTF_8), Item.class);
                System.out.printf("Item data. Price=%f, Color=%s, Category=%s\n", theItem.getPrice(), theItem.getColor(), theItem.getItemCategory());                          
                
                subscriptionClient.complete(receivedMessage.getLockToken());
                receivedMessages++;
            }
            else
            {
                // No more messages to receive.
                subscriptionClient.close();
                break;
            }
        }
        System.out.printf("\nReceived %s messages from subscription %s.\n", receivedMessages, subscription);
        
        return new CompletableFuture().completedFuture(null);
}
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已使用 Azure CLI 佈建資源，然後從服務匯流排主題及其訂用帳戶傳送和接收訊息。 您已了解如何︰

> [!div class="checklist"]
> * 使用 Azure 入口網站建立服務匯流排主題，並建立該主題的一或多個訂用帳戶
> * 使用 .NET 程式碼新增主題篩選
> * 建立含有不同內容的兩個訊息
> * 傳送訊息，並確認它們送達預期的訂用帳戶中
> * 從訂用帳戶接收訊息

如需傳送和接收訊息的範例，請開始使用 [GitHub 上的服務匯流排範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted)。

繼續進行下一個教學課程，以深入了解如何使用服務匯流排的發佈/訂閱功能。

> [!div class="nextstepaction"]
> [使用 PowerShell 和主題/訂用帳戶來更新庫存](service-bus-tutorial-topics-subscriptions-portal.md)

[免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Install the Azure CLI]: /cli/azure/install-azure-cli
[az group create]: /cli/azure/group#az_group_create