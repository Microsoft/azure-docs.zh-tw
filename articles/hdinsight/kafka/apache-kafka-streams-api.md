---
title: '教學課程：使用 Apache Kafka 串流 API - Azure HDInsight '
description: 了解如何搭配 HDInsight 上的 Kafka 使用 Apache Kafka 串流 API。 此 API 可讓您在 Kafka 主題之間執行串流處理。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 04/17/2018
ms.openlocfilehash: 0c1b45d7db53bd2eb7c9f058eb1c44c762886b80
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049829"
---
# <a name="tutorial-apache-kafka-streams-api"></a>教學課程：Apache Kafka 串流 API

了解如何建立應用程式，該應用程式會使用 Kafka 串流 AP，並且與 HDInsight 上的 Kafka 搭配使用。 

本教學課程中使用的應用程式是串流字數統計程式。 它會讀取 Kafka 主題中的文字資料、擷取個別文字，然後將文字和字數儲存到另一個 Kafka 主題中。

> [!NOTE]
> Kafka 串流處理通常會使用 Apache Spark 或 Storm 來完成。 Kafka 0.10.0 版 (在 HDInsight 3.5 和 3.6 中) 導入了 Kafka 串流 API。 此 API 可讓您轉換輸入和輸出主題之間的資料流。 在某些情況下，這可能是建立 Spark 或 Storm 串流解決方案的替代方案。 
>
> 如需有關 Kafka 串流的詳細資訊，請參閱 Apache.org 上的[串流簡介](https://kafka.apache.org/10/documentation/streams/)文件。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定開發環境
> * 了解程式碼
> * 建置並部署應用程式
> * 設定 Kafka 主題
> * 執行程式碼

## <a name="prerequisites"></a>必要條件

* HDInsight 3.6 叢集上的 Kafka。 若要深入了解如何在 HDInsight 叢集上建立 Kafka，請參閱[開始使用 HDInsight 上的 Kafka](apache-kafka-get-started.md) 文件。

* 完成 [Kafka Consumer 和 Producer API](apache-kafka-producer-consumer-api.md) 文件中的步驟。 此文件中的步驟會在本教學課程中建立的範例應用程式和主題。

## <a name="set-up-your-development-environment"></a>設定開發環境

您必須在開發環境中安裝下列元件：

* [Java JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或同等功能版本，例如 OpenJDK。

* [Apache Maven](http://maven.apache.org/)

* SSH 用戶端和 `scp` 命令。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

## <a name="understand-the-code"></a>了解程式碼

範例應用程式位於 [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started) 的 `Streaming` 子目錄中。 該應用程式包含兩個檔案：

* `pom.xml`：此檔案會定義專案相依性、Java 版本和封裝方法。
* `Stream.java`：此檔案會實作串流邏輯。

### <a name="pomxml"></a>Pom.xml

在 `pom.xml` 檔案中務必要了解的事項包括：

* 相依性：此專案依存於 `kafka-clients` 套件所提供的 Kafka 串流 API。 下列 XML 程式碼會定義此相依性：

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

    * `maven-compiler-plugin`：用來將專案所使用的 Java 版本設為 8。 HDInsight 3.6 需要 Java 8。
    * `maven-shade-plugin`：用來產生包含此應用程式以及任何相依性的 uber jar。 它也可用來設定應用程式的進入點，如此您即可直接執行 Jar 檔案，而不需要指定主要類別。

### <a name="streamjava"></a>Stream.java

`Stream.java` 檔案會使用串流 API 來實作字數統計應用程式。 它會從名為 `test` 的 Kafka 主題中讀取資料，並將字數統計寫入名為 `wordcounts` 的主題中。

下列程式碼會定義字數統計應用程式：

```java
package com.microsoft.example;

import org.apache.kafka.common.serialization.Serde;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KStreamBuilder;

import java.util.Arrays;
import java.util.Properties;

public class Stream
{
    public static void main( String[] args ) {
        Properties streamsConfig = new Properties();
        // The name must be unique on the Kafka cluster
        streamsConfig.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-example");
        // Brokers
        streamsConfig.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, args[0]);
        // SerDes for key and values
        streamsConfig.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        streamsConfig.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        // Serdes for the word and count
        Serde<String> stringSerde = Serdes.String();
        Serde<Long> longSerde = Serdes.Long();

        KStreamBuilder builder = new KStreamBuilder();
        KStream<String, String> sentences = builder.stream(stringSerde, stringSerde, "test");
        KStream<String, Long> wordCounts = sentences
                .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
                .map((key, word) -> new KeyValue<>(word, word))
                .countByKey("Counts")
                .toStream();
        wordCounts.to(stringSerde, longSerde, "wordcounts");

        KafkaStreams streams = new KafkaStreams(builder, streamsConfig);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```


## <a name="build-and-deploy-the-example"></a>建置並部署範例

若要建置專案，並將其部署至 HDInsight 叢集上的 Kafka，請使用下列步驟：

1. 從 [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started) 下載範例。

2. 將目錄變更為 `Streaming` 目錄，然後使用下列命令以建立 jar 套件：

    ```bash
    mvn clean package
    ```

    此命令會在 `target/kafka-streaming-1.0-SNAPSHOT.jar` 上建立套件。

3. 使用下列命令將 `kafka-streaming-1.0-SNAPSHOT.jar` 檔案複製到 HDInsight 叢集：
   
    ```bash
    scp ./target/kafka-streaming-1.0-SNAPSHOT.jar sshuser@clustername-ssh.azurehdinsight.net:kafka-streaming.jar
    ```
   
    將 `sshuser` 取代為叢集的 SSH 使用者，並將 `clustername` 取代為叢集的名稱。 出現提示時，請輸入 SSH 使用者帳戶的密碼。 如需搭配 HDInsight 使用 `scp` 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="create-kafka-topics"></a>建立 Kafka 主題

1. 若要開啟與叢集的 SSH 連線，使用下列命令：

    ```bash
    ssh sshuser@clustername-ssh.azurehdinsight.net
    ```

    將 `sshuser` 取代為叢集的 SSH 使用者，並將 `clustername` 取代為叢集的名稱。 出現提示時，請輸入 SSH 使用者帳戶的密碼。 如需搭配 HDInsight 使用 `scp` 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 將叢集名稱儲存到變數，並安裝 JSON 剖析公用程式 (`jq`)，請使用下列命令。 出現提示時，請輸入 Kafka 叢集名稱：

    ```bash
    sudo apt -y install jq
    read -p 'Enter your Kafka cluster name:' CLUSTERNAME
    ```

3. 若要取得 Kafka 代理程式主機和 Zookeeper 主機，請使用下列命令。 出現提示時，輸入叢集登入 (admin) 帳戶的密碼。 系統會提示您輸入密碼兩次。

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`; \
    export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`; \
    ```

4. 若要建立串流作業所使用的主題，請使用下列命令：

    > [!NOTE]
    > 您可能會收到指出 `test` 主題已存在的錯誤。 這是正常的，因為該主題可能已在「Producer 和 Consumer API」教學課程中建立。

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcounts --zookeeper $KAFKAZKHOSTS

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
    ```

    這些主題的用途如下：

    * `test`：此主題是接收記錄之處。 串流應用程式會從中進行讀取。
    * `wordcounts`：此主題是串流應用程式儲存其輸出之處。
    * `RekeyedIntermediateTopic`：此主題可在 `countByKey` 運算子更新計數時用來重新分割資料。
    * `wordcount-example-Counts-changelog`：此主題是 `countByKey` 作業所使用的狀態存放區

    > [!IMPORTANT]
    > HDInsight 上的 Kafka 也可設定為自動建立主題。 如需詳細資訊，請參閱[設定自動建立主題功能](apache-kafka-auto-create-topics.md)文件。

## <a name="run-the-code"></a>執行程式碼

1. 若要以背景程序的形式啟動串流應用程式，請使用下列命令：

    ```bash
    java -jar kafka-streaming.jar $KAFKABROKERS $KAFKAZKHOSTS &
    ```

    > [!NOTE]
    > 您可能會收到關於 log4j 的警告。 您可以忽略此警告。

2. 若要將記錄傳送至 `test` 主題，請使用下列命令啟動產生器應用程式：

    ```bash
    java -jar kafka-producer-consumer.jar producer test $KAFKABROKERS
    ```

3. 產生器執行完成後，請使用下列命令檢視 `wordcounts` 主題中儲存的資訊：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic wordcounts --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer --from-beginning
    ```

    > [!NOTE]
    > `--property` 參數會告知主控台取用者列印索引鍵 (字組) 和計數 (值)。 這個參數也會設定從 Kafka 讀取這些值時要使用的還原序列化。

    輸出大致如下：
   
        dwarfs  13635
        ago     13664
        snow    13636
        dwarfs  13636
        ago     13665
        a       13803
        ago     13666
        a       13804
        ago     13667
        ago     13668
        jumped  13640
        jumped  13641
   
    > [!NOTE]
    > 參數 `--from-beginning` 會設定讓取用者從主題中儲存的記錄開頭處開始執行。 每遇到一個字，字數就會增加一個，因此主題中會包含每個字的多個項目，且計數會遞增。

7. 使用 __Ctrl + C__ 來結束產生者。 繼續使用 __Ctrl + C__ 來結束應用程式和取用者。

## <a name="next-steps"></a>後續步驟

在本文件中，您會了解如何搭配 HDInsight 上的 Kafka 使用 Kafka 串流 API。 使用下列各項來深入了解 Kafka 的使用方式︰

* [分析 Kafka 日誌](apache-kafka-log-analytics-operations-management.md)
* [在Kafka 叢集之間複寫資料](apache-kafka-mirroring.md)
