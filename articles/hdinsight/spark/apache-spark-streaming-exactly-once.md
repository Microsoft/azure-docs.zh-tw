---
title: 透過一次性事件處理來建立 Spark 串流作業 - Azure HDInsight
description: 如何設定 Spark 串流來一次處理一個事件，且只處理一次。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/26/2018
ms.openlocfilehash: ae170e90cede26bd6a43fcc10b93fcd7490d838f
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39618816"
---
# <a name="create-spark-streaming-jobs-with-exactly-once-event-processing"></a>透過一次性事件處理來建立 Spark 串流作業

在系統發生錯誤後，串流處理應用程式會採取不同的方法來處理再處理 (re-processing) 訊息：

* 至少一次：保證會處理每則訊息，但可能會處理一次以上。
* 最多一次：不一定會處理每則訊息。 如果已處理訊息，則只會處理一次。
* 正好一次：保證會處理每則訊息一次，且只處理一次。

本文說明如何設定 Spark 串流以進行一次性處理。

## <a name="exactly-once-semantics-with-spark-streaming"></a>使用 Spark 串流的一次性語意

首先，請考慮發生問題後如何重新啟動所有系統故障點，以及如何避免遺失資料。 Spark 串流應用程式具有：

* 輸入來源
* 從輸入來源提取資料的一或多個接收器流程
* 可處理資料的工作
* 輸出接收端
* 管理長時間執行之作業的驅動程式程序

一次性語意要求任何時間點都不會遺失任何資料，而且不論發生失敗的位置為何，都可以重新開始進行訊息處理。

### <a name="replayable-sources"></a>可重新使用的來源

Spark 串流應用程式讀取事件的來源必須「可重新使用」。 這表示在已擷取訊息，但接著在可保存或處理訊息之前系統卻發生失敗的情況下，來源必須再次提供相同的訊息。

在 Azure 中，HDInsight 上的 Azure 事件中樞和 Kafka 都會提供可重新使用的來源。 可重新使用來源的另一個例子是容錯檔案系統，例如 HDFS、Azure 儲存體 Blob 或 Azure Data Lake Store，其中所有資料都會永久保留，您隨時都可以重新讀取完整的資料。

### <a name="reliable-receivers"></a>可靠的接收器

在 Spark 串流中，Event Hub 和 Kafka 之類的來源具有「可靠的接收器」，而每個接收器都可追蹤其讀取來源的進度。 可靠的接收器會將其狀態保存在容錯儲存體中 (在寫入 HDFS 的 ZooKeeper或 Spark 串流檢查點中)。 如果這類收件器故障且稍後重新啟動，它可以挑選其中斷的位置。

### <a name="use-the-write-ahead-log"></a>使用預寫記錄檔

Spark 串流支援使用預寫記錄檔，其中每個收到的事件會先寫入容錯儲存體中的 Spark 的檢查點目錄，然後儲存在彈性分散式資料集 (RDD) 中。 在 Azure 中，容錯儲存體是 Azure 儲存體或 Azure Data Lake Store 所支援的 HDFS。 在 Spark 串流應用程式中，將 `spark.streaming.receiver.writeAheadLog.enable` 組態設定設為 `true`，即可為所有接收器啟用預寫記錄檔。 預寫記錄檔會為失敗的驅動程式和執行程式提供容錯。

對於針對事件資料執行工作的背景工作，每個 RDD 都會定義為在多個背景工作間進行複寫與散發。 如果工作因執行它的背景工作損毀而失敗，則此工作會在另一個具有事件資料複本的背景工作上重新啟動，因此不會遺失事件。

### <a name="use-checkpoints-for-drivers"></a>使用驅動程式的檢查點

作業驅動程式必須可重新啟動。 如果執行 Spark 串流應用程式的驅動程式損毀，則會關閉其所有執行中的接收器、工作以及儲存事件資料的任何 RDD。 在此情況下，您必須能夠儲存作業的進度，以便稍後繼續執行。 定期向容錯儲存體執行 DStream 的有向非循環圖 (DAG) 檢查點即可完成此工作。 DAG 中繼資料包含用來建立串流應用程式的組態、用來定義應用程式的作業，以及已排入佇列但尚未完成的任何批次。 此中繼資料可讓失敗的驅動程式從檢查點資訊重新啟動。 驅動程式重新啟動時，它會啟動新的接收器，讓它們自行將事件資料從預寫記錄檔復原回到 RDD。

在 Spark 串流中以兩個步驟啟用檢查點。 

1. 在 StreamingContext 物件中，設定檢查點的儲存體路徑：

    val ssc = new StreamingContext(spark, Seconds(1))  ssc.checkpoint("/path/to/checkpoints")

    在 HDInsight 中，這些檢查點應會儲存到您的叢集所連結的預設儲存體，即 Azure 儲存體或 Azure Data Lake Store。

2. 接下來，在 DStream 上指定檢查點間隔 (以秒為單位)。 在每個間隔中，衍生自輸入事件的狀態資料會保存到儲存體。 保存的狀態資料可減少從來源事件重建狀態時所需的運算。

    val lines = ssc.socketTextStream("hostname", 9999)  lines.checkpoint(30)  ssc.start()  ssc.awaitTermination()

### <a name="use-idempotent-sinks"></a>使用等冪接收端

您的作業寫入結果的目的地接收端必須能夠處理多次提供相同結果的情況。 接收端必須能夠偵測這類重複結果並予以忽略。 您可以在不變更狀態的情況下，使用相同的資料多次呼叫「等冪」接收端。

您可以實作先檢查資料存放區中是否存在連入結果的邏輯，從而建立等冪接收端。 如果結果已經存在，從您的 Spark 作業的觀點來看，寫入應該成功，但實際上資料存放區卻忽略了重複的資料。 如果結果不存在，則接收端應將此新結果插入其儲存體中。 

例如，您可以使用預存程序與 Azure SQL Database，在資料表中插入事件。 此預存程序會先依索引鍵欄位尋找事件，而只有在找不到相符的事件時，才會將記錄插入資料表中。

另一個範例是使用已分割的檔案系統，例如 Azure 儲存體 Blob 或 Azure Data Lake 存放區。 在此情況下，您的接收邏輯不必檢查檔案是否存在。 如果代表事件的檔案存在，則只會使用相同的資料來覆寫。 否則會在計算的路徑上建立新檔案。

## <a name="next-steps"></a>後續步驟

* [Spark 串流概觀](apache-spark-streaming-overview.md)
* [在 YARN 中建立高可用性 Spark 串流作業](apache-spark-streaming-high-availability.md)
