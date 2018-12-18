---
title: 使用 Apache Storm 從 Azure 事件中樞接收事件 | Microsoft Docs
description: 開始使用 Apache Storm 從事件中樞接收事件
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: java
ms.devlang: multiple
ms.topic: article
ms.date: 04/12/2018
ms.author: shvija
ms.openlocfilehash: 3880ffe1b61b59e4d05e594a34e1119188177b56
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40002758"
---
# <a name="receive-events-from-event-hubs-using-apache-storm"></a>使用 Apache Storm 從事件中樞接收事件

[Apache Storm](https://storm.incubator.apache.org) 是分散式即時運算系統，可簡化未繫結資料串流的可靠處理。 本節說明如何使用「Azure 事件中樞」Storm Spout 來接收來自「事件中樞」的事件。 使用 Apache Storm，您可以將事件分割到多個裝載於不同節點的處理序。 事件中心與 Storm 的整合透過使用 Storm 的 Zookeeper 安裝透明地設定檢查點以檢查其進度、管理持續檢查點以及來自事件中心的平行接收，以簡化事件的使用。

如需事件中樞接收模式的詳細資訊，請參閱 [事件中樞概觀][Event Hubs overview]。

## <a name="create-project-and-add-code"></a>建立專案並新增程式碼

本教學課程使用 [HDInsight Storm][HDInsight Storm] 安裝，其包含在已可使用的事件中樞 Spout 中。

1. 請遵循 [HDInsight Storm - 入門](../hdinsight/storm/apache-storm-overview.md) 程序來建立新的 HDInsight 叢集，並透過遠端桌面與其連線。
2. 將 `%STORM_HOME%\examples\eventhubspout\eventhubs-storm-spout-0.9-jar-with-dependencies.jar` 檔案複製到本機開發環境。 這包含 events-storm-spout。
3. 使用下列命令將封裝安裝到本機 Maven 存放區。 這樣可讓您在稍後的步驟中將它加入 Storm 專案中做為參考。

    ```shell
    mvn install:install-file -Dfile=target\eventhubs-storm-spout-0.9-jar-with-dependencies.jar -DgroupId=com.microsoft.eventhubs -DartifactId=eventhubs-storm-spout -Dversion=0.9 -Dpackaging=jar
    ```
4. 在 Eclipse 中，建立新的 Maven 專案 (依序按一下 [檔案]、[新增]、[專案])。
   
    ![][12]
5. 選取 [使用預設工作區位置]，然後按 [下一步]。
6. 選取 [maven-archetype-quickstart] 原型，然後按 [下一步]。
7. 插入 **GroupId** 和 **ArtifactId**，然後按一下 [完成]。
8. 在 **pom.xml** 中，於 `<dependency>` 節點中新增下列相依性。

    ```xml  
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-core</artifactId>
        <version>0.9.2-incubating</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.microsoft.eventhubs</groupId>
        <artifactId>eventhubs-storm-spout</artifactId>
        <version>0.9</version>
    </dependency>
    <dependency>
        <groupId>com.netflix.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>1.3.3</version>
        <exclusions>
            <exclusion>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
            </exclusion>
        </exclusions>
        <scope>provided</scope>
    </dependency>
    ```

9. 在 **src** 資料夾中，建立名為 **Config.properties** 的檔案，複製下列內容，並取代 `receive rule key` 與 `event hub name` 的值：

    ```java
    eventhubspout.username = ReceiveRule
    eventhubspout.password = {receive rule key}
    eventhubspout.namespace = ioteventhub-ns
    eventhubspout.entitypath = {event hub name}
    eventhubspout.partitions.count = 16
       
    # if not provided, will use storm's zookeeper settings
    # zookeeper.connectionstring=localhost:2181
       
    eventhubspout.checkpoint.interval = 10
    eventhub.receiver.credits = 10
    ```
    **eventhub.receiver.credits** 的值可決定批次處理多少事件之後，才將它們釋放到 Storm 管線。 為了簡單起見，此範例會將此值設定為 10。 在生產環境中，它通常應設定為較高的值。例如，1024年。
10. 使用下列程式碼，建立稱為 **LoggerBolt** 的新類別：
    
    ```java
    import java.util.Map;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import backtype.storm.task.OutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseRichBolt;
    import backtype.storm.tuple.Tuple;
    
    public class LoggerBolt extends BaseRichBolt {
        private OutputCollector collector;
        private static final Logger logger = LoggerFactory
                  .getLogger(LoggerBolt.class);
    
        @Override
        public void execute(Tuple tuple) {
            String value = tuple.getString(0);
            logger.info("Tuple value: " + value);
   
            collector.ack(tuple);
        }
   
        @Override
        public void prepare(Map map, TopologyContext context, OutputCollector collector) {
            this.collector = collector;
            this.count = 0;
        }
        
        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            // no output fields
        }
    
    }
    ```
    
    此 Storm Bolt 會記錄已接收事件的內容。 這可以輕鬆地擴充以將 Tuple 儲存至儲存體服務。 [採用事件中樞的 HDInsight Storm 範例] 使用此相同方法將資料儲存到 Azure 儲存體和 Power BI 中。
11. 使用下列程式碼，建立稱為 **LogTopology** 的類別：
    
    ```java
    import java.io.FileReader;
    import java.util.Properties;
    import backtype.storm.Config;
    import backtype.storm.LocalCluster;
    import backtype.storm.StormSubmitter;
    import backtype.storm.generated.StormTopology;
    import backtype.storm.topology.TopologyBuilder;
    import com.microsoft.eventhubs.samples.EventCount;
    import com.microsoft.eventhubs.spout.EventHubSpout;
    import com.microsoft.eventhubs.spout.EventHubSpoutConfig;
        
    public class LogTopology {
        protected EventHubSpoutConfig spoutConfig;
        protected int numWorkers;
        
        protected void readEHConfig(String[] args) throws Exception {
            Properties properties = new Properties();
            if (args.length > 1) {
                properties.load(new FileReader(args[1]));
            } else {
                properties.load(EventCount.class.getClassLoader()
                        .getResourceAsStream("Config.properties"));
            }
        
            String username = properties.getProperty("eventhubspout.username");
            String password = properties.getProperty("eventhubspout.password");
            String namespaceName = properties
                    .getProperty("eventhubspout.namespace");
            String entityPath = properties.getProperty("eventhubspout.entitypath");
            String zkEndpointAddress = properties
                    .getProperty("zookeeper.connectionstring"); // opt
            int partitionCount = Integer.parseInt(properties
                    .getProperty("eventhubspout.partitions.count"));
            int checkpointIntervalInSeconds = Integer.parseInt(properties
                    .getProperty("eventhubspout.checkpoint.interval"));
            int receiverCredits = Integer.parseInt(properties
                    .getProperty("eventhub.receiver.credits")); // prefetch count
                                                                // (opt)
            System.out.println("Eventhub spout config: ");
            System.out.println("  partition count: " + partitionCount);
            System.out.println("  checkpoint interval: "
                    + checkpointIntervalInSeconds);
            System.out.println("  receiver credits: " + receiverCredits);
     
            spoutConfig = new EventHubSpoutConfig(username, password,
                    namespaceName, entityPath, partitionCount, zkEndpointAddress,
                    checkpointIntervalInSeconds, receiverCredits);
        
            // set the number of workers to be the same as partition number.
            // the idea is to have a spout and a logger bolt co-exist in one
            // worker to avoid shuffling messages across workers in storm cluster.
            numWorkers = spoutConfig.getPartitionCount();
        
            if (args.length > 0) {
                // set topology name so that sample Trident topology can use it as
                // stream name.
                spoutConfig.setTopologyName(args[0]);
            }
        }
        
        protected StormTopology buildTopology() {
            TopologyBuilder topologyBuilder = new TopologyBuilder();
       
            EventHubSpout eventHubSpout = new EventHubSpout(spoutConfig);
            topologyBuilder.setSpout("EventHubsSpout", eventHubSpout,
                    spoutConfig.getPartitionCount()).setNumTasks(
                    spoutConfig.getPartitionCount());
            topologyBuilder
                    .setBolt("LoggerBolt", new LoggerBolt(),
                            spoutConfig.getPartitionCount())
                    .localOrShuffleGrouping("EventHubsSpout")
                    .setNumTasks(spoutConfig.getPartitionCount());
            return topologyBuilder.createTopology();
        }
        
        protected void runScenario(String[] args) throws Exception {
            boolean runLocal = true;
            readEHConfig(args);
            StormTopology topology = buildTopology();
            Config config = new Config();
            config.setDebug(false);
        
            if (runLocal) {
                config.setMaxTaskParallelism(2);
                LocalCluster localCluster = new LocalCluster();
                localCluster.submitTopology("test", config, topology);
                Thread.sleep(5000000);
                localCluster.shutdown();
            } else {
                config.setNumWorkers(numWorkers);
                StormSubmitter.submitTopology(args[0], config, topology);
            }
        }
        
        public static void main(String[] args) throws Exception {
            LogTopology topology = new LogTopology();
            topology.runScenario(args);
        }
    }
    ```

    這個類別會建立新的事件中樞 Spout，方法是使用組態檔中的屬性來進行具現化。 請務必注意此範例所建立的 Spout 工作數目會與事件中樞內的磁碟分割數目相同，這是為了發揮該事件中樞所允許的平行處理原則上限。

## <a name="next-steps"></a>後續步驟
您可以造訪下列連結以深入了解事件中樞︰

* [事件中樞概觀][Event Hubs overview]
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

<!-- Links -->
[Event Hubs overview]: event-hubs-what-is-event-hubs.md
[HDInsight Storm]: ../hdinsight/storm/apache-storm-overview.md
[採用事件中樞的 HDInsight Storm 範例]: https://azure.microsoft.com/resources/samples/hdinsight-java-storm-eventhub/

<!-- Images -->

[12]: ./media/event-hubs-get-started-receive-storm/create-storm1.png
