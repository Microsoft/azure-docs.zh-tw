---
title: 將 Azure HDInsight 3.6 Apache Storm 遷移至 HDInsight 4.0 Apache Spark
description: 將 Apache Storm 工作負載遷移至 Spark 串流或 Spark 結構化串流的差異和遷移流程。
ms.service: hdinsight
ms.topic: how-to
ms.date: 01/16/2019
ms.openlocfilehash: aa57c01558cfdcf069b17fad9e86f7640553dcfd
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98944777"
---
# <a name="migrate-azure-hdinsight-36-apache-storm-to-hdinsight-40-apache-spark"></a>將 Azure HDInsight 3.6 Apache Storm 遷移至 HDInsight 4.0 Apache Spark

本檔說明如何將 HDInsight 3.6 上的 Apache Storm 工作負載遷移至 HDInsight 4.0。 HDInsight 4.0 不支援 Apache Storm 叢集類型，因此您將需要遷移至另一個串流資料平臺。 有兩個適當的選項 Apache Spark 串流和 Spark 結構化串流。 本檔說明這些平臺之間的差異，也建議工作流程來遷移 Apache Storm 工作負載。

## <a name="storm-migration-paths-in-hdinsight"></a>HDInsight 中的風暴遷移路徑

如果您想要從 HDInsight 3.6 上的 Apache Storm 進行遷移，您有多個選項：

* HDInsight 4.0 上的 Spark 串流
* HDInsight 4.0 上的 Spark 結構化串流
* Azure 串流分析

本檔提供從 Apache Storm 遷移至 Spark 串流和 Spark 結構化串流的指南。

> [!div class="mx-imgBorder"]
> ![HDInsight 風暴遷移路徑](./media/migrate-storm-to-spark/storm-migration-path.png)

## <a name="comparison-between-apache-storm-and-spark-streaming-spark-structured-streaming"></a>Apache Storm 和 Spark 串流、Spark 結構化串流之間的比較

Apache Storm 可以提供不同程度的訊息處理保證。 例如，基本的 Storm 應用程式可以保證至少處理一次，而 [Trident](https://storm.apache.org/releases/current/Trident-API-Overview.html) 可以保證只處理一次。 Spark 串流和 Spark 結構化串流保證任何輸入事件都只會處理一次，即使發生節點失敗也一樣。 風暴具有可處理每個單一事件的模型，而且您也可以搭配 Trident 使用微批次模型。 Spark 串流和 Spark 結構化串流提供 Micro-Batch 處理模型。

|  |Storm |Spark 串流 | Spark 結構化串流|
|---|---|---|---|
|**事件處理保證**|至少一次 <br> 只 (Trident)  |[剛好一次](https://spark.apache.org/docs/latest/streaming-programming-guide.html)|[剛好一次](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)|
|**處理模型**|即時 <br> 微批次 (Trident)  |微批次 |微批次 |
|**事件時間支援**|[是](https://storm.apache.org/releases/2.0.0/Windowing.html)|否|[是](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)|
|**語言**|JAVA 等等。|Scala、JAVA、Python|Python、R、Scala、JAVA、SQL|

### <a name="spark-streaming-vs-spark-structured-streaming"></a>Spark 串流與 Spark 結構化串流

Spark 結構化串流會取代 Spark 串流 (DStreams) 。 結構化串流將繼續接收增強功能和維護，而 DStreams 只會處於維護模式。 **注意：需要強調此點的連結**。 結構化串流的功能和 DStreams 的來源和接收器所提供的功能一樣多，因此請評估您的需求，以選擇適當的 Spark 串流處理選項。

## <a name="streaming-single-event-processing-vs-micro-batch-processing"></a>串流處理 (單一事件) 處理與 Micro-Batch 處理

風暴提供可處理每個單一事件的模型。 這表示所有的內送記錄都會在抵達時立即處理。 Spark 串流應用程式必須等待幾毫秒，才可收集每個微批次事件，然後才能傳送該批次以進行處理。 相較之下，事件驅動的應用程式會立即處理每個事件。 而 Spark 串流通常會延遲幾秒鐘。 微批次方法的優點是可更有效率的處理資料和更簡易的彙總運算。

> [!div class="mx-imgBorder"]
> ![串流處理和微批次處理](./media/migrate-storm-to-spark/streaming-and-micro-batch-processing.png)

## <a name="storm-architecture-and-components"></a>風暴架構與元件

Storm 拓撲是由有向非循環圖 (DAG) 中排列的多個元件所組成。 圖形中元件之間的資料流程。 每個元件會取用一或多個資料流，並可選擇性地發出一或多個資料流。

|元件 |描述 |
|---|---|
|壺 嘴|將資料帶入拓撲中。 它們會將一或多個串流發出至拓撲。|
|螺栓|取用從 spout 或其他螺栓發出的串流。 Bolt 可以選擇性地將串流發出至拓撲。 Bolt 也負責將資料寫入外部服務或儲存體，例如 HDFS、Kafka 或 HBase。|

> [!div class="mx-imgBorder"]
> ![風暴元件的互動](./media/migrate-storm-to-spark/apache-storm-components.png)

風暴是由下列三個守護程式所組成，可讓風暴叢集正常運作。

|精靈 |描述 |
|---|---|
|Nimbus|與 Hadoop JobTracker 相似之處在于，它負責將程式碼散發到叢集，以及將工作指派給機器和監視失敗。|
|Zookeeper|用於叢集協調。|
|監督員|接聽指派給其電腦的工作，並根據 Nimbus 的指示來啟動和停止工作者進程。 每個背景工作進程都會執行拓撲的子集。 使用者的應用程式邏輯 (Spout 和螺栓) 在這裡執行。|

> [!div class="mx-imgBorder"]
> ![nimbus、zookeeper 和監督員守護程式](./media/migrate-storm-to-spark/nimbus-zookeeper-supervisor.png)

## <a name="spark-streaming-architecture-and-components"></a>Spark 串流架構和元件

下列步驟摘要說明元件如何在 Spark 串流中一起運作 (DStreams) 和 Spark 結構化串流：

* 啟動 Spark 串流時，驅動程式會在執行程式中啟動工作。
* 執行程式會從串流資料來源接收資料流程。
* 當執行程式接收資料流程時，會將資料流程分割成區塊，並將其保留在記憶體中。
* 資料區塊會複寫到其他執行程式。
* 然後，已處理的資料會儲存在目標資料存放區中。

> [!div class="mx-imgBorder"]
> ![輸出的 spark 串流路徑](./media/migrate-storm-to-spark/spark-streaming-to-output.png)

## <a name="spark-streaming-dstream-workflow"></a>Spark 串流 (DStream) 工作流程

當每個批次間隔過去後，新的 RDD 就會產生，並包含該間隔中的所有資料。 系統會將連續的 Rdd 集合收集到 DStream。 例如，如果批次間隔長度為一秒，您的 DStream 就會每秒發出包含一個 RDD 的批次，該 RDD 會包含在這一秒期間內嵌的所有資料。 處理 DStream 時，溫度事件就會出現在這些批次的其中一個。 Spark 串流應用程式會處理包含事件的批次，並在最後處理儲存在每個 RDD 中的資料。

> [!div class="mx-imgBorder"]
> ![spark 串流處理批次](./media/migrate-storm-to-spark/spark-streaming-batches.png)

如需 Spark 串流可用之不同轉換的詳細資訊，請參閱 [DStreams 上的轉換](https://spark.apache.org/docs/latest/streaming-programming-guide.html#transformations-on-dstreams)。

## <a name="spark-structured-streaming"></a>Spark 結構化串流

Spark 結構化串流以非深入的資料表形式表示資料流程。 當新資料到達時，資料表會持續成長。 長時間執行的查詢會持續處理此輸入資料表，並將結果傳送至輸出資料表。

在結構化串流中，資料會送達系統並立即內嵌到輸入資料表中。 您可撰寫查詢 (使用 DataFrame 和資料集 API) 來對此輸入資料表執行作業。

查詢輸出會產生包含查詢結果的 *結果資料表*。 您可以從外部資料存放區（例如關係資料庫）的結果資料表中繪製資料。

「觸發間隔」會控制從輸入資料表處理資料的時機。 根據預設，觸發間隔為零，所以結構化串流會在資料到達時立即嘗試處理資料。 實際上，這表示只要是結構化串流處理完前一個查詢的執行後，就會針對任何新收到的資料啟動另一個處理執行。 您可以將觸發程序設定為依據間隔執行，從而以時間為基礎的批次方式處理串流資料。

> [!div class="mx-imgBorder"]
> ![結構化串流中的資料處理](./media/migrate-storm-to-spark/structured-streaming-data-processing.png)

> [!div class="mx-imgBorder"]
> ![結構化串流的程式設計模型](./media/migrate-storm-to-spark/structured-streaming-model.png)

## <a name="general-migration-flow"></a>一般遷移流程

從風暴到 Spark 的建議遷移流程採用下列初始架構：

* Kafka 可作為串流資料來源
* Kafka 和風暴是部署在相同的虛擬網路上
* 由風暴處理的資料會寫入至資料接收，例如 Azure 儲存體或 Azure Data Lake Storage Gen2。

    > [!div class="mx-imgBorder"]
    > ![假設目前的環境圖表](./media/migrate-storm-to-spark/presumed-current-environment.png)

若要將您的應用程式從風暴遷移至其中一個 Spark 串流 Api，請執行下列動作：

1. **部署新的叢集。** 在相同的虛擬網路中部署新的 HDInsight 4.0 Spark 叢集，並在其上部署 Spark 串流或 Spark 結構化串流應用程式，並徹底測試。

    > [!div class="mx-imgBorder"]
    > ![HDInsight 中的新 spark 部署](./media/migrate-storm-to-spark/new-spark-deployment.png)

1. **停止在舊的風暴叢集中使用。** 在現有的風暴中，停止取用串流資料來源中的資料，並等候資料完成寫入至目標接收。

    > [!div class="mx-imgBorder"]
    > ![停止在目前的叢集中使用](./media/migrate-storm-to-spark/stop-consuming-current-cluster.png)

1. **開始在新的 Spark 叢集上使用。** 從新部署的 HDInsight 4.0 Spark 叢集中開始串流資料。 目前，此程式會藉由從最新的 Kafka 位移取用來處理。

    > [!div class="mx-imgBorder"]
    > ![開始在新叢集上使用](./media/migrate-storm-to-spark/start-consuming-new-cluster.png)

1. **視需要移除舊的叢集。** 一旦切換完成且正常運作，請視需要移除舊的 HDInsight 3.6 風暴叢集。

    > [!div class="mx-imgBorder"]
    > ![視需要移除舊的 HDInsight 叢集](./media/migrate-storm-to-spark/remove-old-clusters1.png)

## <a name="next-steps"></a>後續步驟

如需有關風暴、Spark 串流和 Spark 結構化串流的詳細資訊，請參閱下列檔：

* [Apache Spark 串流概觀](../spark/apache-spark-streaming-overview.md)
* [Apache Spark 結構化串流的概觀](../spark/apache-spark-structured-streaming-overview.md)
