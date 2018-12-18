---
title: 使用 Akka Streams 搭配適用於 Apache Kafka 的 Azure 事件中樞 | Microsoft Docs
description: 將 Akka Streams 連線到已啟用 Kafka 的事件中樞
services: event-hubs
documentationcenter: ''
author: basilhariri
manager: timlt
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.custom: mvc
ms.date: 08/06/2018
ms.author: bahariri
ms.openlocfilehash: 7c09656f62f3a8a2efd889cf28f12bd5a42e309a
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42745491"
---
# <a name="using-akka-streams-with-event-hubs-for-apache-kafka"></a>使用 Akka Streams 搭配適用於 Apache Kafka 的事件中樞
本教學課程說明如何將 Akka Streams 連線至已啟用 Kafka 的事件中樞，而不需要變更通訊協定用戶端或執行您自己的叢集。 適用於 Kafka 的 Azure 事件中樞支援 [Apache Kafka 1.0 版](https://kafka.apache.org/10/documentation.html) \(英文\)。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 建立事件中樞命名空間
> * 複製範例專案
> * 執行 Flink 生產者 
> * 執行 Flink 取用者

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定您具有下列必要條件：

* 請參閱[適用於 Apache Kafka 的事件中樞](event-hubs-for-kafka-ecosystem-overview.md)一文。 
* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
* [Java Development Kit (JDK) 1.8+](http://www.oracle.com/technetwork/java/javase/downloads/index.html) \(英文\)
    * 在 Ubuntu 上，執行 `apt-get install default-jdk` 來安裝 JDK。
    * 務必設定 JAVA_HOME 環境變數，以指向 JDK 安裝所在的資料夾。
* [下載](http://maven.apache.org/download.cgi)並[安裝 ](http://maven.apache.org/install.html) Maven 二進位封存檔
    * 在 Ubuntu 上，您可以執行 `apt-get install maven` 來安裝 Maven。
* [Git](https://www.git-scm.com/downloads)
    * 在 Ubuntu 上，您可以執行 `sudo apt-get install git` 來安裝 Git。

## <a name="create-an-event-hubs-namespace"></a>建立事件中樞命名空間

您需要事件中樞命名空間，才能從任何事件中樞服務傳送或接收。 如需有關事件中樞 Kafka 端點的資訊，請參閱[建立已啟用 Kafka 的事件中樞](event-hubs-create-kafka-enabled.md)。 請務必複製事件中樞連接字串以供稍後使用。

## <a name="clone-the-example-project"></a>複製範例專案

既然您已經有已啟用 Kafka 的事件中樞連接字串，請複製 Azure 事件中樞存放庫，並瀏覽至 `akka` 子資料夾：

```shell
git clone https://github.com/Azure/azure-event-hubs.git
cd azure-event-hubs/samples/kafka/akka
```

## <a name="run-akka-streams-producer"></a>執行 Akka Streams 生產者

使用提供的 Akka Streams 生產者範例，傳送訊息到事件中樞服務。

### <a name="provide-an-event-hubs-kafka-endpoint"></a>提供事件中樞 Kafka 端點

#### <a name="producer-applicationconf"></a>生產者 application.conf

更新 `producer/src/main/resources/application.conf` 中 `bootstrap.servers` 和 `sasl.jaas.config` 的值，以使用正確的驗證將生產者導向至事件中樞 Kafka 端點。

```xml
akka.kafka.producer {
    #Akka Kafka producer properties can be defined here


    # Properties defined by org.apache.kafka.clients.producer.ProducerConfig
    # can be defined in this configuration section.
    kafka-clients {
        bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
        sasl.mechanism=PLAIN
        security.protocol=SASL_SSL
        sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

### <a name="run-producer-from-the-command-line"></a>從命令列執行生產者

若要從命令列執行生產者，請從 Maven 中產生 JAR 然後從中執行 (或是使用 Maven 產生 JAR，然後將必要的 Kafka JAR 加入至 classpath，在 Java 中執行)：

```shell
mvn clean package
mvn exec:java -Dexec.mainClass="AkkaTestProducer"
```

生產者會開始將事件傳送到已啟用 Kafka 的事件中樞 (位於主題 `test`)，並將事件印出至 stdout。

## <a name="run-akka-streams-consumer"></a>執行 Akka Streams 取用者

使用提供的取用者範例，接收來自已啟用 Kafka 之事件中樞的訊息。

### <a name="provide-an-event-hubs-kafka-endpoint"></a>提供事件中樞 Kafka 端點

#### <a name="consumer-applicationconf"></a>取用者 application.conf

更新 `consumer/src/main/resources/application.conf` 中 `bootstrap.servers` 和 `sasl.jaas.config` 的值，以使用正確的驗證將取用者導向至事件中樞 Kafka 端點。

```xml
akka.kafka.consumer {
    #Akka Kafka consumer properties defined here
    wakeup-timeout=60s

    # Properties defined by org.apache.kafka.clients.consumer.ConsumerConfig
    # defined in this configuration section.
    kafka-clients {
       request.timeout.ms=60000
       group.id=akka-example-consumer

       bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
       sasl.mechanism=PLAIN
       security.protocol=SASL_SSL
       sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

### <a name="run-consumer-from-the-command-line"></a>從命令列執行取用者

若要從命令列執行取用者，請從 Maven 中產生 JAR 然後從中執行 (或是使用 Maven 產生 JAR，然後將必要的 Kafka JAR 加入至 classpath，以在 Java 中執行)：

```shell
mvn clean package
mvn exec:java -Dexec.mainClass="AkkaTestConsumer"
```

如果已啟用 Kafka 的事件中樞有事件 (例如您的生產者也正在執行)，則取用者會開始接收來自主題 `test` 的事件。 

如需有關 Akka Streams 的詳細資訊，請參閱 [Akka Streams Kafka 指南](https://doc.akka.io/docs/akka-stream-kafka/current/home.html) \(英文\)。

## <a name="next-steps"></a>後續步驟
在本教學課中，您已了解如何將 Akka Streams 連線至已啟用 Kafka 的事件中樞，而不需要變更通訊協定用戶端或執行您自己的叢集。 適用於 Kafka 的 Azure 事件中樞支援 [Apache Kafka 1.0 版](https://kafka.apache.org/10/documentation.html)。 您在本教學課程中執行了下列動作： 

> [!div class="checklist"]
> * 建立事件中樞命名空間
> * 複製範例專案
> * 執行 Flink 生產者 
> * 執行 Flink 取用者

若要深入了解事件中樞和適用於 Kafka 的事件中樞，請參閱下列主題：  

* [了解事件中樞](event-hubs-what-is-event-hubs.md)
* [了解適用於 Kafka 的事件中樞](event-hubs-for-kafka-ecosystem-overview.md)
* 使用 [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330)，[將事件從 Kafka 內部部署串流至雲端上已啟用 Kafka 的事件中樞](event-hubs-kafka-mirror-maker-tutorial.md)。
* 了解如何使用[原生的 Kafka 應用程式](event-hubs-quickstart-kafka-enabled-event-hubs.md)或 [Apache Flink](event-hubs-kafka-flink-tutorial.md) 串流至已啟用 Kafka 的事件中樞
