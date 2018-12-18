---
title: '教學課程：使用 Apache Kafka Producer 和 Consumer API - Azure HDInsight '
description: 了解如何搭配 HDInsight 上的 Kafka 使用 Apache Kafka Producer 和 Consumer API。 在本教學課程中，您將了解如何從 Java 應用程式將這些 API 用於 HDInsight 上的 Kafka。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 04/16/2018
ms.openlocfilehash: c52f64c2508870bf061e144229cf26ab269c343b
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049231"
---
# <a name="tutorial-use-the-apache-kafka-producer-and-consumer-apis"></a>教學課程：使用 Apache Kafka Producer 和 Consumer API

了解如何將 Kafka Producer 和 Consumer API 用於 HDInsight 上的 Kafka。

Kafka Producer API 可讓應用程式將資料流傳送至 Kafka 叢集。 Kafka Consumer API 可讓應用程式從叢集讀取資料流。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定開發環境
> * 設定您的部署環境
> * 了解程式碼
> * 建置並部署應用程式
> * 在叢集上執行應用程式

如需 API 的詳細資訊，請參閱 [Producer API](https://kafka.apache.org/documentation/#producerapi) 和 [Consumer API](https://kafka.apache.org/documentation/#consumerapi) 的 Apache 說明文件。

## <a name="set-up-your-development-environment"></a>設定開發環境

您必須在開發環境中安裝下列元件：

* [Java JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或同等功能版本，例如 OpenJDK。

* [Apache Maven](http://maven.apache.org/)

* SSH 用戶端和 `scp` 命令。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

* 文字編輯器或 Java IDE。

當您在開發工作站上安裝 Java 和 JDK 時可能會設定下列環境變數。 不過，您應該檢查它們是否存在，以及它們是否包含您系統的正確值。

* `JAVA_HOME` - 應指向已安裝 JDK 的目錄。
* `PATH` - 應該包含下列路徑：
  
    * `JAVA_HOME` (或對等的路徑)。
    * `JAVA_HOME\bin` (或對等的路徑)。
    * 已安裝 Maven 的目錄。

## <a name="set-up-your-deployment-environment"></a>設定您的部署環境

本教學課程需要 HDInsight 3.6 上的 Kafka。 若要深入了解如何在 HDInsight 叢集上建立 Kafka，請參閱[開始使用 HDInsight 上的 Kafka](apache-kafka-get-started.md) 文件。

## <a name="understand-the-code"></a>了解程式碼

範例應用程式位於 [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started) 的 `Producer-Consumer` 子目錄中。 該應用程式主要包含四個檔案：

* `pom.xml`：此檔案會定義專案相依性、Java 版本和封裝方法。
* `Producer.java`：此檔案會使用 Producer API 將一百萬個 (1,000,000) 隨機句子傳送至 Kafka。
* `Consumer.java`：此檔案會使用 Consumer API 從 Kafka 讀取資料並將其發送至 STDOUT。
* `Run.java`：用來執行產生者和取用者程式碼的命令列介面。

### <a name="pomxml"></a>Pom.xml

在 `pom.xml` 檔案中務必要了解的事項包括：

* 相依性：此專案依存於 `kafka-clients` 套件所提供的 Producer 和 Consumer API。 下列 XML 程式碼會定義此相依性：

    ```xml
    <!-- Kafka client for producer/consumer operations -->
    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>${kafka.version}</version>
    </dependency>
    ```

    > [!NOTE]
    > `${kafka.version}` 項目會在 `pom.xml` 的 `<properties>..</properties>` 區段中進行宣告，並設定為 HDInsight 叢集的 Kafka 版本。

* 外掛程式：Maven 外掛程式可提供多種功能。 在此專案中，會使用下列外掛程式：

    * `maven-compiler-plugin`：用來將專案所使用的 Java 版本設為 8。 這是HDInsight 3.6 所使用的 Java 版本。
    * `maven-shade-plugin`：用來產生包含此應用程式以及任何相依性的 uber jar。 它也可用來設定應用程式的進入點，如此您即可直接執行 Jar 檔案，而不需要指定主要類別。

### <a name="producerjava"></a>Producer.java

產生者會與 Kafka 代理程式主機 (背景工作節點) 通訊，以將資料儲存到 Kafka 主題中。 下列程式碼片段來自於 `Producer.java` 檔案：

```java
package com.microsoft.example;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.Properties;
import java.util.Random;
import java.io.IOException;

public class Producer
{
    public static void produce(String brokers) throws IOException
    {

        // Set properties used to configure the producer
        Properties properties = new Properties();
        // Set the brokers (bootstrap servers)
        properties.setProperty("bootstrap.servers", brokers);
        // Set how to serialize key/value pairs
        properties.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        properties.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        // So we can generate random sentences
        Random random = new Random();
        String[] sentences = new String[] {
                "the cow jumped over the moon",
                "an apple a day keeps the doctor away",
                "four score and seven years ago",
                "snow white and the seven dwarfs",
                "i am at two with nature"
        };

        String progressAnimation = "|/-\\";
        // Produce a bunch of records
        for(int i = 0; i < 1000000; i++) {
            // Pick a sentence at random
            String sentence = sentences[random.nextInt(sentences.length)];
            // Send the sentence to the test topic
            producer.send(new ProducerRecord<String, String>("test", sentence));
            String progressBar = "\r" + progressAnimation.charAt(i % progressAnimation.length()) + " " + i;
            System.out.write(progressBar.getBytes());
        }
    }
}
```

此程式碼會連線至 Kafka 代理程式主機 (背景工作節點)，然後使用 Producer API 將 1,000,000 個句子傳送至 Kafka。

### <a name="consumerjava"></a>Consumer.java

取用者會與 Kafka 代理程式主機 (背景工作節點) 通訊，以迴圈方式讀取記錄。 下列程式碼片段來自於 `Consumer.java` 檔案：

```java
package com.microsoft.example;

import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import java.util.Properties;
import java.util.Arrays;

public class Consumer {
    public static void consume(String brokers, String groupId) {
        // Create a consumer
        KafkaConsumer<String, String> consumer;
        // Configure the consumer
        Properties properties = new Properties();
        // Point it to the brokers
        properties.setProperty("bootstrap.servers", brokers);
        // Set the consumer group (all consumers must belong to a group).
        properties.setProperty("group.id", groupId);
        // Set how to serialize key/value pairs
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        // When a group is first created, it has no offset stored to start reading from. This tells it to start
        // with the earliest record in the stream.
        properties.setProperty("auto.offset.reset","earliest");
        consumer = new KafkaConsumer<>(properties);

        // Subscribe to the 'test' topic
        consumer.subscribe(Arrays.asList("test"));

        // Loop until ctrl + c
        int count = 0;
        while(true) {
            // Poll for records
            ConsumerRecords<String, String> records = consumer.poll(200);
            // Did we get any?
            if (records.count() == 0) {
                // timeout/nothing to read
            } else {
                // Yes, loop over records
                for(ConsumerRecord<String, String> record: records) {
                    // Display record and count
                    count += 1;
                    System.out.println( count + ": " + record.value());
                }
            }
        }
    }
}
```

在此程式碼中，取用者會設定為從主題的開頭處讀取 (`auto.offset.reset` 設為 `earliest`)。

### <a name="runjava"></a>Run.java

`Run.java` 檔案會提供可執行產生者或取用者程式碼的命令列介面。 您必須提供 Kafka 代理程式主機資訊作為參數。 您可以選擇性地包含取用者程序所使用的群組識別碼值。 如果您使用相同的群組識別碼建立多個取用者執行個體，這些執行個體將會以負載平衡的方式讀取主題。

```java
package com.microsoft.example;

import java.io.IOException;
import java.util.UUID;

// Handle starting producer or consumer
public class Run {
    public static void main(String[] args) throws IOException {
        if(args.length < 2) {
            usage();
        }

        // Get the brokers
        String brokers = args[1];
        switch(args[0].toLowerCase()) {
            case "producer":
                Producer.produce(brokers);
                break;
            case "consumer":
                // Either a groupId was passed in, or we need a random one
                String groupId;
                if(args.length == 3) {
                    groupId = args[2];
                } else {
                    groupId = UUID.randomUUID().toString();
                }
                Consumer.consume(brokers, groupId);
                break;
            default:
                usage();
        }
        System.exit(0);
    }
    // Display usage
    public static void usage() {
        System.out.println("Usage:");
        System.out.println("kafka-example.jar <producer|consumer> brokerhosts [groupid]");
        System.exit(1);
    }
}
```

## <a name="build-and-deploy-the-example"></a>建置並部署範例

1. 從 [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started) 下載範例。

2. 將目錄變更為 `Producer-Consumer` 目錄的位置並使用下列命令︰

    ```
    mvn clean package
    ```

    此命令會建立名為 `target` 的目錄，其中包含名為 `kafka-producer-consumer-1.0-SNAPSHOT.jar` 的檔案。

3. 使用下列命令將 `kafka-producer-consumer-1.0-SNAPSHOT.jar` 檔案複製到 HDInsight 叢集：
   
    ```bash
    scp ./target/kafka-producer-consumer-1.0-SNAPSHOT.jar SSHUSER@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
    ```
   
    以叢集的 SSH 使用者取代 **USERNAME**，並以叢集的名稱取代 **CLUSTERNAME**。 出現提示時，請輸入 SSH 使用者的密碼。

## <a id="run"></a> 執行範例

1. 若要開啟與叢集的 SSH 連線，使用下列命令：

    ```bash
    ssh SSHUSER@CLUSTERNAME-ssh.azurehdinsight.net
    ```

    以叢集的 SSH 使用者取代 **USERNAME**，並以叢集的名稱取代 **CLUSTERNAME**。 出現提示時，請輸入 SSH 使用者帳戶的密碼。 如需搭配 HDInsight 使用 `scp` 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 若要建立本範例所使用的 Kafka 主題，請使用下列步驟：

    1. 將叢集名稱儲存到變數，並安裝 JSON 剖析公用程式 (`jq`)，請使用下列命令。 出現提示時，請輸入 Kafka 叢集名稱：
    
        ```bash
        sudo apt -y install jq
        read -p 'Enter your Kafka cluster name:' CLUSTERNAME
        ```
    
    2. 若要取得 Kafka 代理程式主機和 Zookeeper 主機，請使用下列命令。 出現提示時，輸入叢集登入 (admin) 帳戶的密碼。
    
        ```bash
        export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`; \
        export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`; \
        ```

    3. 若要建立 `test` 主題，請使用下列命令：

        ```bash
        /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
        ```
    4. 您也可以使用 jar 檔案來建立主題。 例如，若要建立 `test2` 主題，請使用下列命令：

        ```bash
        java -jar kafka-producer-consumer.jar create test2 $KAFKABROKERS
        ```

3. 若要執行產生器，並且將資料寫入至主題，請使用下列命令：

    ```bash
    java -jar kafka-producer-consumer.jar producer test $KAFKABROKERS
    ```

4. 產生器完成後，請使用下列命令從主題讀取︰
   
    ```bash
    java -jar kafka-producer-consumer.jar consumer test $KAFKABROKERS
    ```
   
    已讀取的記錄以及記錄計數隨即顯示。

5. 使用 __Ctrl + C__ 來結束取用者。

### <a name="multiple-consumers"></a>多個取用者

Kafka 取用者會在讀取記錄時使用取用者群組。 多個取用者使用相同群組會導致從主題讀取負載平衡。 群組中的每個取用者都會收到一部分的記錄。

取用者應用程式接受作為群組識別碼的參數。 例如，下列命令會使用 `mygroup` 的群組識別碼啟動取用者：
   
```bash
java -jar kafka-producer-consumer.jar consumer test $KAFKABROKERS mygroup
```

若要查看此程序的運作情況，請使用下列命令︰

```bash
tmux new-session 'java -jar kafka-producer-consumer.jar consumer test $KAFKABROKERS mygroup' \; split-w
indow -h 'java -jar kafka-producer-consumer.jar consumer test $KAFKABROKERS mygroup' \; attach
```

此命令會使用 `tmux` 將終端機分割成兩個資料行。 取用者會在每個資料行中啟動 (使用相同的群組識別碼值)。 取用者完成讀取後，請留意到每個取用者僅讀取了記錄的一部分。 請使用 __Ctrl + C __ 兩次來結束 `tmux`。

透過主題的資料分割處理相同群組內的用戶端取用。 稍早建立的 `test` 主題有 8 個資料分割。 如果您啟動八個取用者，則每個取用者都會從主題的單一分割區讀取記錄。

> [!IMPORTANT]
> 一個取用者群組中的取用者執行個體不得超過資料分割。 在此範例中，一個取用者群組可以包含最多 8 個取用者，因為這是主題中的資料分割數目。 或者，您可以有多個取用者群組，其各有不超過 8 個取用者。

Kafka 中儲存的記錄會依照其在資料分割內接收的順序儲存。 若要達到依序傳遞「資料分割內」的記錄，請建立取用者群組，其中的取用者執行個體數目與資料分割數目相符。 若要達到依序傳遞「主題內」的記錄，請建立只有一個取用者執行個體的取用者群組。

## <a name="next-steps"></a>後續步驟

在本文件中，您會了解如何搭配 HDInsight 上的 Kafka 使用 Kafka Producer and Consumer API。 使用下列各項來深入了解 Kafka 的使用方式︰

* [分析 Kafka 日誌](apache-kafka-log-analytics-operations-management.md)
* [在Kafka 叢集之間複寫資料](apache-kafka-mirroring.md)
* [採用 HDInsight 的 Kafka Streams API](apache-kafka-streams-api.md)
