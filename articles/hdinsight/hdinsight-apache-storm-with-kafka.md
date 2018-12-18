---
title: '教學課程：在 HDInsight 上搭配使用 Apache Kafka 與 Storm - Azure '
description: 了解如何在 HDInsight 上搭配使用 Apache Storm 與 Apache Kafka 以建立串流管線。 在本教學課程中，您會使用 KafkaBolt 與 KafkaSpout 元件從 Kafka 串流處理資料。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 05/21/2018
ms.openlocfilehash: 7aa8f0b62459c376113bca5a0c58cc7dd3b5280c
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43094884"
---
# <a name="tutorial-use-apache-storm-with-kafka-on-hdinsight"></a>教學課程：在 HDInsight 上搭配使用 Apache Storm 與 Kafka

本教學課程說明如何使用 Apache Storm 拓撲，對 HDInsight 上的 Apache Kafka 讀取和寫入資料。 本教學課程也會說明如何將資料保存至 Storm 叢集上與 HDFS 相容的儲存體。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * Storm 和 Kafka
> * 了解程式碼
> * 建立 Kafka 和 Storm 叢集
> * 建置拓撲
> * 設定拓撲
> * 建立 Kafka 主題
> * 啟動拓撲
> * 停止拓撲
> * 清除資源

## <a name="prerequisites"></a>必要條件

* 熟悉如何建立 Kafka 主題。 如需詳細資訊，請參閱 [HDInsight 上的 Kafka 快速入門](./kafka/apache-kafka-get-started.md)文件。

* 熟悉如何建置和部署 Storm 解決方案 (拓撲)。 具體來說，即使用 Flux 架構的拓撲。 如需詳細資訊，請參閱[以 Java 建立 Storm 拓撲](./storm/apache-storm-develop-java-topology.md)文件。

* [Java JDK 1.8](http://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html) 或更新版本。 HDInsight 3.5 或更新版本需要 Java 8。

* [Maven 3.x](https://maven.apache.org/download.cgi)

* SSH 用戶端 (您需要 `ssh` 和 `scp` 命令) - 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

當您在開發工作站上安裝 Java 和 JDK 時可能會設定下列環境變數。 不過，您應該檢查它們是否存在，以及它們是否包含您系統的正確值。

* `JAVA_HOME` - 應指向已安裝 JDK 的目錄。
* `PATH` - 應該包含下列路徑：
  
    * `JAVA_HOME` (或對等的路徑)。
    * `JAVA_HOME\bin` (或對等的路徑)。
    * 已安裝 Maven 的目錄。

> [!IMPORTANT]
> 在本文件的步驟中，Azure 資源群組必須包含 HDInsight 上的 Storm 和 HDInsight 叢集上的 Kafka。 這兩個叢集都位於 Azure 虛擬網路中，可讓 Storm 叢集直接與 Kafka 叢集通訊。
> 
> 為了方便您使用，本文件會連結至可建立所有必要 Azure 資源的範本。 
>
> 如需在虛擬網路中使用 HDInsight 的詳細資訊，請參閱[使用 Azure 虛擬網路擴充 HDInsight](hdinsight-extend-hadoop-virtual-network.md) 文件。

## <a name="storm-and-kafka"></a>Storm 和 Kafka

Apache Storm 提供數個用來處理 Kafka 的元件。 在本教學課程中會使用下列元件：

* `org.apache.storm.kafka.KafkaSpout`：此元件會從 Kafka 讀取資料。 此元件依存於下列元件：

    * `org.apache.storm.kafka.SpoutConfig`：提供 Spout 元件的組態。

    * `org.apache.storm.spout.SchemeAsMultiScheme` 和 `org.apache.storm.kafka.StringScheme`：來自 Kafka 的資料如何轉換為 Storm Tuple。

* `org.apache.storm.kafka.bolt.KafkaBolt`：此元件會將資料寫入至 Kafka。 此元件依存於下列元件：

    * `org.apache.storm.kafka.bolt.selector.DefaultTopicSelector`：說明寫入的目標主題。

    * `org.apache.kafka.common.serialization.StringSerializer`：設定 Bolt 以將資料序列化為字串值。

    * `org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper`：從在 Strom 拓樸內使用的 Tuple 資料結構對應至儲存在 Kafka 中的欄位。

這些是 `org.apache.storm : storm-kafka` 套件中提供的元件。 請使用與 Storm 版本相符的套件版本。 對於 HDInsight 3.6，適用的 Storm 版本為 1.1.0。
您也需要 `org.apache.kafka : kafka_2.10` 套件，其中包含其他 Kafka 元件。 請使用與 Kafka 版本相符的套件版本。 對於 HDInsight 3.6，適用的 Kafka 版本為 0.10.0.0。

下列 XML 是 `pom.xml` 中對於 Maven 專案的相依性宣告：

```xml
<!-- Storm components for talking to Kafka -->
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-kafka</artifactId>
    <version>1.1.0</version>
</dependency>
<!-- needs to be the same Kafka version as used on your cluster -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.10</artifactId>
    <version>0.10.0.0</version>
    <!-- Exclude components that are loaded from the Storm cluster at runtime -->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## <a name="understanding-the-code"></a>了解程式碼

您可以在 [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka) 上找到此文件中使用的程式碼。

本教學課程中提供兩種拓撲：

* Kafka 寫入器：產生隨機的句子，並將其儲存至 Kafka。

* Kafka 讀取器：從 Kafka 讀取資料，然後將其儲存至 Storm 叢集中與 HDFS 相容的檔案存放區。

    > [!WARNING] 
    > 若要讓 Storm 能夠使用 HDInsight 所使用的 HDFS 相容儲存體，必須執行指令碼動作。 此指令碼會將數個 jar 檔案安裝到 Storm 的 `extlib` 路徑。 本教學課程中的範本會在叢集建立期間自動使用此指令碼。
    >
    > 如果您未使用此文件中的範本建立 Storm 叢集，則必須手動將指令碼動作套用至您的叢集。
    >
    > 此指令碼動作位於 `https://hdiconfigactions2.blob.core.windows.net/stormextlib/stormextlib.sh`，且會套用至 Storm 叢集的監督員和 nimbus 節點。 如需使用指令碼動作的詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight](hdinsight-hadoop-customize-cluster-linux.md) 文件。

拓撲是使用 [Flux](https://storm.apache.org/releases/1.1.2/flux.html) 來定義的。 Storm 0.10.x 引進了 Flux，可讓您區隔拓撲組態與程式碼。 若為使用 Flux 架構的拓撲，拓撲定義於 YAML 檔案中。 YAML 檔案可以納入為拓撲的一部分。 它也可以是您提交拓撲時使用的獨立檔案。 Flux 也支援執行階段的變數替代 (在此範例中使用)。

在執行階段會針對這些拓撲設定下列參數：

* `${kafka.topic}`：拓樸讀取/寫入的 Kafka 主題名稱。

* `${kafka.broker.hosts}`：Kafka 訊息代理程式執行所在的主機。 KafkaBolt 在寫入 Kafka 時會使用訊息代理程式資訊。

* `${kafka.zookeeper.hosts}`：Kafka 叢集中 Zookeeper 執行所在的主機。

* `${hdfs.url}`：HDFSBolt 元件的檔案系統 URL。 指出資料會寫入至 Azure 儲存體帳戶還是 Azure Data Lake Store。

* `${hdfs.write.dir}`：寫入資料的目標目錄。

如需有關 Flux 拓撲的詳細資訊，請參閱 [https://storm.apache.org/releases/1.1.2/flux.html](https://storm.apache.org/releases/1.1.2/flux.html)。

### <a name="kafka-writer"></a>Kafka 寫入器

在 Kafka 寫入器拓撲中，Kafka Bolt 元件會以兩個字串值作為參數。 這些參數會指出 Bolt 會將哪些 Tuple 欄位傳送至 Kafka 作為__索引鍵__和__訊息__值。 索引鍵可用來分割 Kafka 中的資料。 訊息是要儲存的資料。

在此範例中，`com.microsoft.example.SentenceSpout` 元件會發出包含兩個欄位的 Tuple，即 `key` 和 `message`。 Kafka Bolt 會擷取這些欄位，並將其資料傳送至 Kafka。

欄位不需要使用名稱 `key` 和 `message`。 這些名稱會用於此專案中，使對應更容易了解。

下列 YAML 是 Kafka 寫入器元件的定義：

```yaml
# kafka-writer
---

# topology definition
# name to be used when submitting
name: "kafka-writer"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Topic selector for KafkaBolt
  - id: "topicSelector"
    className: "org.apache.storm.kafka.bolt.selector.DefaultTopicSelector"
    constructorArgs:
      - "${kafka.topic}"

  # Mapper for KafkaBolt
  - id: "kafkaMapper"
    className: "org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper"
    constructorArgs:
      - "key"
      - "message"

  # Producer properties for KafkaBolt
  - id: "producerProperties"
    className: "java.util.Properties"
    configMethods:
      - name: "put"
        args:
          - "bootstrap.servers"
          - "${kafka.broker.hosts}"
      - name: "put"
        args:
          - "acks"
          - "1"
      - name: "put"
        args:
          - "key.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
      - name: "put"
        args:
          - "value.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
 

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "com.microsoft.example.SentenceSpout"
    parallelism: 8

# Bolt definitions
bolts:
  - id: "kafka-bolt"
    className: "org.apache.storm.kafka.bolt.KafkaBolt"
    parallelism: 8
    configMethods:
    - name: "withProducerProperties"
      args: [ref: "producerProperties"]
    - name: "withTopicSelector"
      args: [ref: "topicSelector"]
    - name: "withTupleToKafkaMapper"
      args: [ref: "kafkaMapper"]

# Stream definitions

streams:
  - name: "spout --> kafka" # Streams data from the sentence spout to the Kafka bolt
    from: "sentence-spout"
    to: "kafka-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="kafka-reader"></a>Kafka 讀取器

在 Kafka 讀取器拓撲中，Spout 元件會從 Kafka 讀取字串值形式的資料。 這項資料隨後會由記錄元件寫入至 Storm 記錄，並且由 HDFS Bolt 元件寫入至 Storm 叢集的 HDFS 相容檔案系統。

```yaml
# kafka-reader
---

# topology definition
# name to be used when submitting
name: "kafka-reader"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Convert data from Kafka into string tuples in storm
  - id: "stringScheme"
    className: "org.apache.storm.kafka.StringScheme"
  - id: "stringMultiScheme"
    className: "org.apache.storm.spout.SchemeAsMultiScheme"
    constructorArgs:
      - ref: "stringScheme"

  - id: "zkHosts"
    className: "org.apache.storm.kafka.ZkHosts"
    constructorArgs:
      - "${kafka.zookeeper.hosts}"

  # Spout configuration
  - id: "spoutConfig"
    className: "org.apache.storm.kafka.SpoutConfig"
    constructorArgs:
      # brokerHosts
      - ref: "zkHosts"
      # topic
      - "${kafka.topic}"
      # zkRoot
      - ""
      # id
      - "readerid"
    properties:
      - name: "scheme"
        ref: "stringMultiScheme"

    # How often to sync files to HDFS; every 1000 tuples.
  - id: "syncPolicy"
    className: "org.apache.storm.hdfs.bolt.sync.CountSyncPolicy"
    constructorArgs:
      - 1

  # Rotate files when they hit 5 MB
  - id: "rotationPolicy"
    className: "org.apache.storm.hdfs.bolt.rotation.FileSizeRotationPolicy"
    constructorArgs:
      - 5
      - "KB"

  # File format; read the directory from filters at run time, and use a .txt extension when writing.
  - id: "fileNameFormat"
    className: "org.apache.storm.hdfs.bolt.format.DefaultFileNameFormat"
    configMethods:
      - name: "withPath"
        args: ["${hdfs.write.dir}"]
      - name: "withExtension"
        args: [".txt"]

  # Internal file format; fields delimited by `|`.
  - id: "recordFormat"
    className: "org.apache.storm.hdfs.bolt.format.DelimitedRecordFormat"
    configMethods:
      - name: "withFieldDelimiter"
        args: ["|"]

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "kafka-spout"
    className: "org.apache.storm.kafka.KafkaSpout"
    constructorArgs:
      - ref: "spoutConfig"
    # Set to the number of partitions for the topic
    parallelism: 8

# Bolt definitions
bolts:
  - id: "logger-bolt"
    className: "com.microsoft.example.LoggerBolt"
    parallelism: 1
  
  - id: "hdfs-bolt"
    className: "org.apache.storm.hdfs.bolt.HdfsBolt"
    configMethods:
      - name: "withConfigKey"
        args: ["hdfs.config"]
      - name: "withFsUrl"
        args: ["${hdfs.url}"]
      - name: "withFileNameFormat"
        args: [ref: "fileNameFormat"]
      - name: "withRecordFormat"
        args: [ref: "recordFormat"]
      - name: "withRotationPolicy"
        args: [ref: "rotationPolicy"]
      - name: "withSyncPolicy"
        args: [ref: "syncPolicy"]
    parallelism: 1

# Stream definitions

streams:
  # Stream data to log
  - name: "kafka --> log" # name isn't used (placeholder for logging, UI, etc.)
    from: "kafka-spout"
    to: "logger-bolt"
    grouping:
      type: SHUFFLE
  
  # stream data to file
  - name: "kafka --> hdfs"
    from: "kafka-spout"
    to: "hdfs-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="property-substitutions"></a>屬性替代

專案包含名為 `dev.properties` 的檔案，用來傳遞拓撲所使用的參數。 它會定義下列屬性：

| dev.properties 檔案 | 說明 |
| --- | --- |
| `kafka.zookeeper.hosts` | Kafka 叢集的 Zookeeper 主機。 |
| `kafka.broker.hosts` | Kafka 代理程式主機 (背景工作節點)。 |
| `kafka.topic` | 拓撲所使用的 Kafka 主題。 |
| `hdfs.write.dir` | Kafka 讀取器拓撲寫入的目標目錄。 |
| `hdfs.url` | Storm 叢集所使用的檔案系統。 對於 Azure 儲存體帳戶，請使用 `wasb:///` 值。 對於 Azure Data Lake Store，請使用 `adl:///` 值。 |

## <a name="create-the-clusters"></a>建立叢集

Apache Kafka on HDInsight 不提供透過公用網際網路存取 Kafka 訊息代理程式。 任何使用 Kafka 的項目都必須位於相同的 Azure 虛擬網路中。 在本教學課程中，Kafka 和 Storm 叢集位於相同的 Azure 虛擬網路中。 

下圖顯示 Storm 與 Kafka 之間的通訊流程︰

![Azure 虛擬網路中的 Storm 和 Kafka 叢集圖表](./media/hdinsight-apache-storm-with-kafka/storm-kafka-vnet.png)

> [!NOTE]
> 叢集上的其他服務 (例如 SSH 和 Ambari) 可以透過網際網路存取。 如需有關適用於 HDInsight 的公用連接埠詳細資訊，請參閱 [HDInsight 所使用的連接埠和 URI](hdinsight-hadoop-port-settings-for-services.md)。

若要建立 Azure 虛擬網路，然後在其中建立 Kafka 和 Storm 叢集，請使用下列步驟：

1. 使用以下按鈕，在 Azure 入口網站中登入 Azure 並開啟範本。
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-storm-java-kafka%2Fmaster%2Fcreate-kafka-storm-clusters-in-vnet.json" target="_blank"><img src="./media/hdinsight-apache-storm-with-kafka/deploy-to-azure.png" alt="Deploy to Azure"></a>
   
    Azure Resource Manager 範本位於 **https://github.com/Azure-Samples/hdinsight-storm-java-kafka/blob/master/create-kafka-storm-clusters-in-vnet.json**。 它會建立下列資源︰
    
    * Azure 資源群組
    * Azure 虛擬網路
    * Azure 儲存體帳戶
    * HDInsight 版本 3.6 上的 Kafka (三個背景工作角色節點)
    * HDInsight 版本 3.6 上的 Storm (三個背景工作角色節點)

  > [!WARNING]
  > 若要保證 Kafka 在 HDInsight 上的可用性，您的叢集必須包含至少三個背景工作角色節點。 此範本會建立包含三個背景工作角色節點的 Kafka 叢集。

2. 使用下列指引來填入 [自訂部署] 區段上的項目︰

    2. 使用下列資訊，填入 [自訂範本] 區段上的項目︰

    | 設定 | 值 |
    | --- | --- |
    | 訂用帳戶 | 您的 Azure 訂用帳戶 |
    | 資源群組 | 包含資源的資源群組。 |
    | 位置 | 資源建立所在的 Azure 區域。 |
    | Kafka 叢集名稱 | Kafka 叢集的名稱。 |
    | Storm 叢集名稱 | Storm 叢集的名稱。 |
    | 叢集登入使用者名稱 | 叢集的管理員使用者名稱。 |
    | 叢集登入密碼 | 叢集的管理員使用者密碼。 |
    | SSH 使用者名稱 | 要為叢集建立的 SSH 使用者。 |
    | SSH 密碼 | SSH 使用者的密碼。 |
   
    ![範本參數的圖片](./media/hdinsight-apache-storm-with-kafka/storm-kafka-template.png)

3. 讀取**條款及條件**，然後選取 [我同意上方所述的條款及條件]。

4. 最後，核取 [釘選到儀表板]，然後選取 [購買]。

> [!NOTE]
> 建立叢集可能需要 20 分鐘的時間。

## <a name="build-the-topology"></a>建置拓撲

1. 在開發環境中，從 [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka) 下載專案，開啟命令列，並將目錄變更為您下載專案的位置。

2. 從 **hdinsight-storm-java-kafka** 目錄，使用下列命令來編譯專案並建立部署套件︰

  ```bash
  mvn clean package
  ```

    封裝程序會在 `target` 目錄中建立名為 `KafkaTopology-1.0-SNAPSHOT.jar` 的檔案。

3. 使用下列命令將套件複製到 Storm on HDInsight 叢集。 將 `sshuser` 取代為叢集的 SSH 使用者名稱。 將 `stormclustername` 取代為 __Storm__ 叢集的名稱。

  ```bash
  scp ./target/KafkaTopology-1.0-SNAPSHOT.jar sshuser@stormclustername-ssh.azurehdinsight.net:KafkaTopology-1.0-SNAPSHOT.jar
  ```

    出現提示時，請輸入您在建立叢集時所使用的密碼。

## <a name="configure-the-topology"></a>設定拓撲

1. 使用下列其中一種方法來探索 HDInsight 叢集上 **Kafka** 的 Kafka 訊息代理程式主機：

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $brokerHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($brokerHosts -join ":9092,") + ":9092"
    ```

    > [!IMPORTANT]
    > 下列 Bash 範例假設 `$CLUSTERNAME` 包含 __Kafka__ 叢集的名稱。 它同時假設已安裝 [jq](https://stedolan.github.io/jq/) 1.5 版或更新版本。 出現提示時，輸入叢集登入帳戶的密碼。

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER" | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2
    ```

    傳回的值類似下列文字︰

        wn0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,wn1-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092

    > [!IMPORTANT]
    > 雖然叢集可能有兩個以上的訊息代理程式主機，您並不需要提供客戶端完整的主機名單。 列出一兩個主機便已足夠。

2. 使用下列其中一種方法來探索 HDInsight 叢集上 __Kafka__ 的 Zookeeper 主機：

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $zookeeperHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($zookeeperHosts -join ":2181,") + ":2181"
    ```

    > [!IMPORTANT]
    > 下列 Bash 範例假設 `$CLUSTERNAME` 包含 __Kafka__ 叢集的名稱。 它也假設已安裝 [jq](https://stedolan.github.io/jq/)。 出現提示時，輸入叢集登入帳戶的密碼。

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2
    ```

    傳回的值類似下列文字︰

        zk0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,zk2-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181

    > [!IMPORTANT]
    > 雖然可能有兩個以上的 Zookeeper 節點，您並不需要提供客戶端完整的主機名單。 列出一兩個主機便已足夠。

    儲存這個值以便稍後使用。

3. 編輯專案根目錄中的 `dev.properties` 檔案。 請將 __Kafka__ 叢集的訊息代理程式和 Zookeeper 主機資訊新增至此檔案中相符的行。 下例是使用先前步驟中的範例值所設定：

        kafka.zookeeper.hosts: zk0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,zk2-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181
        kafka.broker.hosts: wn0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,wn1-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092
        kafka.topic: stormtopic

    > [!IMPORTANT]
    > 為使用 Azure 儲存體帳戶的叢集設定 `hdfs.url` 項目。 若要將此拓撲用於使用 Data Lake Store 的 Storm 叢集，請將此值從 `wasb` 變更為 `adl`。

4. 儲存 `dev.properties` 檔案，然後使用下列命令將它上傳至 **Storm** 叢集：

     ```bash
    scp dev.properties USERNAME@storm-BASENAME-ssh.azurehdinsight.net:dev.properties
    ```

    將 **USERNAME** 替換為叢集的 SSH 使用者名稱。 將 **BASENAME** 替換為您在建立叢集時使用的基底名稱。

## <a name="create-the-kafka-topic"></a>建立 Kafka 主題

Kafka 會將資料儲存到_主題_中。 在啟動 Storm 拓撲之前，您必須先建立主題。 若要建立拓撲，請使用下列步驟：

1. 使用下列命令透過 SSH 連線至 __Kafka__ 叢集。 將 `sshuser` 替換為建立叢集時所使用的 SSH 使用者名稱。 將 `kafkaclustername` 取代為 Kafka 叢集的名稱：

    ```bash
    ssh sshuser@kafkaclustername-ssh.azurehdinsight.net
    ```

    出現提示時，請輸入您在建立叢集時所使用的密碼。
   
    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 若要建立 Kafka 主題，請使用下列命令。 請將 `$KAFKAZKHOSTS` 取代為您在設定拓撲時所使用的 Zookeeper 主機資訊：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic stormtopic --zookeeper $KAFKAZKHOSTS
    ```

    此命令會連線至 Kafka 叢集的 Zookeeper，並建立名為 `stormtopic` 的新主題。 Storm 拓撲會使用此主題。

## <a name="start-the-writer"></a>開始寫入器

1. 使用下列命令來透過 SSH 連線到 **Storm** 叢集。 將 `sshuser` 替換為建立叢集時所使用的 SSH 使用者名稱。 請將 `stormclustername` 取代為 Storm 叢集的名稱：

    ```bash
    ssh sshuser@stormclustername-ssh.azurehdinsight.net
    ```

    出現提示時，請輸入您在建立叢集時所使用的密碼。
   
    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 在連往 Storm 叢集的 SSH 連線中，使用下列命令來啟動寫入器拓撲：

    ```bash
    storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writer.yaml --filter dev.properties
    ```

    此命令使用的參數如下：

    * `org.apache.storm.flux.Flux`︰使用 Flux 來設定及執行此拓撲。

    * `--remote`：將拓撲提交至 Nimbus。 拓撲會分散於叢集中的背景工作節點。

    * `-R /writer.yaml`︰使用 `writer.yaml` 檔案來設定拓撲。 `-R` 表示此資源包含在 jar 檔案中。 剛檔案位於 jar 的根目錄中，所以 `/writer.yaml` 是它的路徑。

    * `--filter`：使用 `dev.properties` 檔案的值填入 `writer.yaml` 拓撲的項目。 例如，使用檔案的 `kafka.topic` 項目值取代拓撲定義的 `${kafka.topic}` 項目。

## <a name="start-the-reader"></a>開始讀取器

1. 在 Storm 叢集的 SSH 工作階段中，使用下列命令來啟動讀取器拓撲：

  ```bash
  storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /reader.yaml --filter dev.properties
  ```

2. 稍候片刻，然後使用下列命令檢視讀取器拓撲所建立的檔案：

    ```bash
    hdfs dfs -ls /stormdata
    ```

    輸出大致如下：

        Found 173 items
        -rw-r--r--   1 storm supergroup       5137 2018-04-09 19:00 /stormdata/hdfs-bolt-4-0-1523300453088.txt
        -rw-r--r--   1 storm supergroup       5128 2018-04-09 19:00 /stormdata/hdfs-bolt-4-1-1523300453624.txt
        -rw-r--r--   1 storm supergroup       5131 2018-04-09 19:00 /stormdata/hdfs-bolt-4-10-1523300455170.txt
        ...

3. 若要檢視檔案內容，請使用下列命令。 請將 `filename.txt` 取代為檔案名稱：

    ```bash
    hdfs dfs -cat /stormdata/filename.txt
    ```

    以下文字是檔案內容的範例：

        four score and seven years ago
        snow white and the seven dwarfs
        i am at two with nature
        snow white and the seven dwarfs
        i am at two with nature
        four score and seven years ago
        an apple a day keeps the doctor away

## <a name="stop-the-topologies"></a>停止拓撲

在連往 Storm 叢集的 SSH 工作階段中，使用下列命令來停止 Storm 拓撲：

  ```bash
  storm kill kafka-writer
  storm kill kafka-reader
  ```

## <a name="clean-up-resources"></a>清除資源

若要清除本教學課程所建立的資源，您可以刪除資源群組。 刪除資源群組也會刪除相關聯的 HDInsight 叢集，以及與資源群組相關聯的任何其他資源。

若要使用 Azure 入口網站移除資源群組：

1. 在 Azure 入口網站中展開左側功能表，以開啟服務的功能表，然後選擇 [資源群組] 以顯示資源群組的清單。
2. 找出要刪除的資源群組，然後以滑鼠右鍵按一下清單右側的 [更多] 按鈕 (...)。
3. 選取 [刪除資源群組]，並加以確認。

> [!WARNING]
> HDInsight 叢集的計費起自叢集建立時，終至叢集刪除時。 計費是以每分鐘按比例計算，因此不再使用時，請一律刪除您的叢集。
> 
> 刪除 HDInsight 叢集上的 Kafka，也會刪除 Kafka 中儲存的任何資料。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Storm 拓撲對 HDInsight 上的 Kafka 進行寫入和讀取。 您也已了解如何將資料儲存至 HDInsight 所使用的 HDFS 相容儲存體。

若要深入了解如何使用 HDInsight 上的 Kafka，請參閱[使用 Kafka Producer 和 Consumer API](kafka/apache-kafka-producer-consumer-api.md) 文件。

如需部署和監視以 Linux 為基礎的 HDInsight 上的拓撲相關資訊，請參閱[部署和管理以 Linux 為基礎的 HDInsight 上的 Apache Storm 拓撲](storm/apache-storm-deploy-monitor-topology-linux.md)
