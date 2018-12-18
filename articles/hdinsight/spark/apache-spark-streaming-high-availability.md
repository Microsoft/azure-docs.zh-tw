---
title: 在 YARN 中建立高可用性 Spark 串流作業 - Azure HDInsight
description: 如何針對高可用性案例設定 Spark 串流。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 01/26/2018
ms.openlocfilehash: 0725c70668ca3089028bff5fc1d8374c6bfdecde
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43051986"
---
# <a name="create-high-availability-spark-streaming-jobs-with-yarn"></a>使用 YARN 建立高可用性 Spark 串流作業

Spark 串流可讓您實作可擴充、高輸送量、容錯應用程式，以便進行資料流處理。 您可以將 HDInsight Spark 叢集上的 Spark 串流應用程式連線到各種資料來源，例如 Azure 事件中樞、Azure IoT 中樞、Kafka、Flume、Twitter、ZeroMQ、原始 TCP 通訊端，或藉由監視 HDFS 檔案系統的變更。 Spark 串流可支援容錯，並保證任何指定的事件只會處理一次，即使節點失敗亦然。

Spark 串流會建立長時間執行的作業，您可在這段期間將轉換套用至資料，然後將結果推送至檔案系統、資料庫、儀表板和主控台。 Spark 串流會先收集定義的時間間隔內的事件批次，藉此處理資料微批次。 接著會傳送該批次以進行處理和輸出。 批次時間間隔通常是以幾分之一秒定義。

![Spark Streaming](./media/apache-spark-streaming-high-availability/spark-streaming.png)

## <a name="dstreams"></a>DStreams

Spark 串流會使用「離散化串流」 (DStream) 來表示連續資料流。 此 DStream 可以從事件中樞或 Kafka 等輸入來源建立，或將轉換套用至另一個 DStream 來建立 DStream。 當事件抵達 Spark 串流應用程式時，事件會以可靠的方式儲存。 也就是會複寫事件資料，讓多個節點擁有其複本。 這可確保任何單一節點的錯誤不會造成事件遺失。

Spark 核心會使用「彈性分散式資料集」(RDD)。 RDD 會將資料分散於叢集中的多個節點，其中的每個節點通常都會在記憶體內部完整維護其資料，以達到最佳效能。 每個 RDD 代表在一段批次間隔內收集的事件。 當批次間隔消逝時，Spark 串流會產生一個新 RDD，其中包含該間隔內的所有資料。 這一組連續的 RDD 會被收集到 DStream。 Spark 串流應用程式會處理每個批次的 RDD 中儲存的資料。

![Spark DStream](./media/apache-spark-streaming-high-availability/DStream.png)

## <a name="spark-structured-streaming-jobs"></a>Spark 結構化串流作業

Spark 2.0 引進了 Spark 結構化串流，作為用於串流結構化資料的分析引擎。 Spark 結構化串流會使用 SparkSQL 批次引擎 API。 使用 Spark 串流，Spark 結構化串流會對持續送達的資料微批次執行其運算。 Spark 結構化串流代表會以內含無限多個資料列的輸入資料表來表示資料流。 也就是說，輸入資料表會隨新資料送達而持續成長。 此輸入資料表會由長時間執行的查詢持續進行處理，而結果會寫出至輸出資料表。

![Spark 結構化串流](./media/apache-spark-streaming-high-availability/structured-streaming.png)

在結構化串流中，資料會送達系統並立即內嵌到輸入資料表中。 您可撰寫查詢來對此輸入資料表執行作業。 查詢輸出會產生稱為「結果資料表」的其他資料表。 結果資料表包含您的查詢結果，您可以從中提取資料以傳送至外部資料存放區 (如關聯式資料庫)。 「觸發間隔」可設定從輸入資料表處理資料的時機。 根據預設，結構化串流會在資料送達時立即進行處理。 不過，您也可將觸發程序設定為以較長的間隔執行，從而以時間為基礎的批次方式處理串流資料。 每次有新資料時，結果資料表中的資料就會完全重新整理，使其包含自串流查詢開始後的所有輸出資料 (「完整模式」)，或只包含自上次處理查詢後的新資料 (「附加模式」)。

## <a name="create-fault-tolerant-spark-streaming-jobs"></a>建立容錯的 Spark 串流作業

若要為您的 Spark 串流作業建立高可用性環境，請從為個別作業編碼開始，以便在發生失敗時復原。 此種自我復原作業具有容錯特性。

RDD 有數個可協助高可用性和容錯的 Spark 串流作業的屬性：

* 儲存在 RDD 作為 DStream 的輸入資料批次會自動在記憶體中複寫，以供容錯之用。
* 背景工作失敗所造成的資料遺失，可從不同背景工作上複寫的輸入資料進行重新計算，但前題是可使用這些背景工作節點。
* 由於可透過記憶體中運算，從錯誤/落後的執行緒執行復原，因此一秒內即可執行錯誤快速復原。

### <a name="exactly-once-semantics-with-spark-streaming"></a>使用 Spark 串流的一次性語意

若要建立可處理每個事件一次 (且僅只一次) 的應用程式，請考慮在發生問題後如何重新啟動所有系統故障點，以及如何避免遺失資料。 一次性語意要求任何時間點都不會遺失任何資料，而且不論發生失敗的位置為何，都可以重新開始進行訊息處理。 請參閱[透過一次性事件處理來建立 Spark 串流作業](apache-spark-streaming-exactly-once.md)。

## <a name="spark-streaming-and-yarn"></a>Spark 串流和 YRAN

在 HDInsight 中，叢集工作是由 *Yet Another Resource Negotiator* (YARN) 協調。 設計高可用性的 Spark 串流時包含 Spark 串流技術，以及 YARN 元件技術。  使用 YARN 的組態範例如下所示。 

![YARN 架構](./media/apache-spark-streaming-high-availability/yarn-arch.png)

下列各節說明此組態的設計考量。

### <a name="plan-for-failures"></a>針對失敗進行規劃

若要建立高可用性的 YARN 組態，您應針對可能的執行程式或驅動程式失敗進行規劃。 有些 Spark 串流作業也包含需要額外設定和安裝的資料保證需求。 例如，串流應用程式可能有零資料遺失保證的業務需求，儘管主控串流系統或 HDInsight 叢集中發生任何錯誤時也是如此。

如果**執行程式**失敗，Spark 會自動重新啟動其工作和接收器，所以不需要變更組態。

不過，如果**驅動程式**失敗，則其所有相關聯的執行程式都會失敗，而且會失去所有收到的區塊和運算結果。 若要從驅動程式失敗中復原，請使用[透過一次性事件處理來建立 Spark 串流作業](apache-spark-streaming-exactly-once.md#use-checkpoints-for-drivers)中所述的「DStream 檢查點檢查」。 DStream 檢查點檢查會定期將 DStreams 的「有向非循環圖」 (DAG) 儲存至容錯儲存體，例如 Azure 儲存體。  檢查點檢查可讓 Spark 結構化串流從檢查點資訊重新啟動失敗的驅動程式。  重新啟動驅動程式即可啟動新的執行程式，也可重新啟動接收器。

若要使用 DStream 檢查點檢查來復原驅動程式：

* 使用 `yarn.resourcemanager.am.max-attempts` 組態設定，在 YARN 上設定驅動程式自動重新啟動。
* 使用 `streamingContext.checkpoint(hdfsDirectory)`，在 HDFS 相容的檔案系統中設定檢查點目錄。
* 重新建構原始程式碼以使用檢查點進行復原，例如：

    ```scala
        def creatingFunc() : StreamingContext = {
            val context = new StreamingContext(...)
            val lines = KafkaUtils.createStream(...)
            val words = lines.flatMap(...)
            ...
            context.checkpoint(hdfsDir)
        }

        val context = StreamingContext.getOrCreate(hdfsDir, creatingFunc)
        context.start()
    ```

* 藉由使用 `sparkConf.set("spark.streaming.receiver.writeAheadLog.enable","true")` 來啟用預寫記錄檔 (WAL)，設定遺失的資料復原，並使用 `StorageLevel.MEMORY_AND_DISK_SER` 來停用輸入 DStream 的記憶體中複寫。

總而言之，使用檢查點檢查、WAL 和可靠的接收器，您就能夠進行「至少一次」資料復原：

* 正好一次，前題是所接收的資料並未遺失，且輸出是等冪或交易輸出。
* 正好一次，使用新的 Kafka Direct 方法，該方法使用 Kafka 作為複寫的記錄，而不是使用接收器或 WAL。

### <a name="typical-concerns-for-high-availability"></a>高可用性的一般考量

* 監視串流作業比監視批次作業更加困難。 Spark 串流作業通常會長時間執行，而且 YARN 會等到作業完成才彙總記錄。  Spark 檢查點會在應用程式或 Spark 升級期間遺失，您必須在升級期間清除檢查點目錄。

* 將您的 YARN 叢集模式設定為即使用戶端失敗也會執行驅動程式。 若要設定驅動程式自動重新啟動：

    ```
    spark.yarn.maxAppAttempts = 2
    spark.yarn.am.attemptFailuresValidityInterval=1h
    ```

* Spark 和 Spark 串流 UI 都有可設定的計量系統。 您也可以使用其他程式庫 (例如 Graphite/Grafana) 來下載儀表板計量，例如「已處理的記錄數目」、「驅動程式與執行程式的記憶體/GC 使用量」、「總延遲時間」、「叢集的使用率」等等。 在結構化串流 2.1 版或更新版本中，您可以使用 `StreamingQueryListener` 來收集其他計量。

* 您應該分割長時間執行的作業。  將 Spark 串流應用程式提交至叢集時，必須定義執行作業的 YARN 佇列。 您可以使用 [YARN 容量排程器](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html)，將長時間執行的作業提交至不同的佇列。

* 正常關閉串流應用程式。 如果已知您的位移，且所有應用程式狀態都儲存在外部，您可以透過程式設計方式在適當位置停止串流應用程式。 其中有個技巧是在 Spark 中使用「執行緒勾點」，做法是每隔 *n* 秒檢查一次外部旗標。 您也可以使用啟動應用程式時在 HDFS 上建立的「標記檔」，然後在想要停止時移除。 若採用標記檔方法，請在 Spark 應用程序中使用個別的執行緒來呼叫類似下面的程式碼：

    ```scala
    streamingContext.stop(stopSparkContext = true, stopGracefully = true)
    // to be able to recover on restart, store all offsets in an external database
    ```

## <a name="next-steps"></a>後續步驟

* [Spark 串流概觀](apache-spark-streaming-overview.md)
* [透過一次性事件處理來建立 Spark 串流作業](apache-spark-streaming-exactly-once.md)
* [YARN 上長時間執行的 Spark 串流作業](http://mkuthan.github.io/blog/2016/09/30/spark-streaming-on-yarn/) 
* [結構化串流：容錯語意](http://spark.apache.org/docs/2.1.0/structured-streaming-programming-guide.html#fault-tolerance-semantics)
* [離散化串流：可擴充串流處理的容錯模型](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2012/EECS-2012-259.pdf)
