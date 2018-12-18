---
title: '教學課程：搭配 Kafka 進行 Apache Spark 結構化串流 - Azure HDInsight '
description: 了解如何使用 Apache Spark 串流將資料傳入或傳出 Apache Kafka。 在此教學課程中，您會使用 Jupyter Notebook 從 HDInsight 上的 Spark 串流處理資料。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 05/08/2018
ms.author: jasonh
ms.openlocfilehash: 11169e45d5fbdc5e51b81c06d814524a7b09c614
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43108284"
---
# <a name="tutorial-use-spark-structured-streaming-with-kafka-on-hdinsight"></a>教學課程：搭配使用 Spark 結構化串流與 HDInsight 上的 Kafka

本教學課程說明如何使用 Spark 結構化串流，對 Azure HDInsight 上的 Apache Kafka 讀取和寫入資料。

Spark 結構化串流是建置在 Spark SQL 上的串流處理引擎。 它允許您進行與靜態資料批次計算相同的串流計算。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 搭配 Kafka 使用結構化串流
> * 建立 Kafka 和 Spark 叢集
> * 將 Notebook 上傳至 Spark
> * 使用 Notebook
> * 清除資源

當您完成本文件中的步驟時，請記得刪除叢集，以避免產生過多的費用。

## <a name="prerequisites"></a>必要條件

* 熟悉如何搭配使用 Jupyter Notebook 和 HDInsight 上的 Spark。 如需詳細資訊，請參閱[使用 HDInsight 上的 Spark 載入資料及執行查詢](spark/apache-spark-load-data-run-query.md)文件。

* 熟悉 [Scala](https://www.scala-lang.org/) 程式設計語言。 教學課程中使用的程式碼是以 Scala 撰寫的。

* 熟悉如何建立 Kafka 主題。 如需詳細資訊，請參閱 [HDInsight 上的 Kafka 快速入門](kafka/apache-kafka-get-started.md)文件。

> [!IMPORTANT]
> 在本文件的步驟中，Azure 資源群組必須包含 HDInsight 上的 Spark 和 HDInsight 叢集上的 Kafka。 這兩個叢集都位於 Azure 虛擬網路中，可讓 Spark 叢集直接與 Kafka 叢集通訊。
> 
> 為了方便您使用，本文件會連結至可建立所有必要 Azure 資源的範本。 
>
> 如需在虛擬網路中使用 HDInsight 的詳細資訊，請參閱[使用 Azure 虛擬網路擴充 HDInsight](hdinsight-extend-hadoop-virtual-network.md) 文件。

## <a name="structured-streaming-with-kafka"></a>搭配 Kafka 使用結構化串流

Spark 結構化串流是建置在 Spark SQL 引擎上的串流處理引擎。 使用結構化串流時，您可以比照撰寫批次查詢的方式來撰寫串流查詢。

下列程式碼片段說明如何讀取 Kafka 和儲存至檔案。 第一個程式碼片段是批次作業，第二個則是串流作業：

```scala
// Read a batch from Kafka
val kafkaDF = spark.read.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()
// Select data and write to file
kafkaDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .write
                .format("parquet")
                .option("path","/example/batchtripdata")
                .option("checkpointLocation", "/batchcheckpoint")
                .save()
```

```scala
// Stream from Kafka
val kafkaStreamDF = spark.readStream.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()
// Select data from the stream and write to file
kafkaStreamDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .writeStream
                .format("parquet")
                .option("path","/example/streamingtripdata")
                .option("checkpointLocation", "/streamcheckpoint")
                .start.awaitTermination(30000)
```

在這兩個程式碼片段中，都是從 Kafka 讀取資料並寫入至檔案。 這兩個範例的差異如下：

| Batch | 串流 |
| --- | --- |
| `read` | `readStream` |
| `write` | `writeStream` |
| `save` | `start` |

串流作業也會使用 `awaitTermination(30000)`，而會在 30000 毫秒之後停止串流。 

若要搭配使用結構化串流和 Kafka，您的專案必須具有對 `org.apache.spark : spark-sql-kafka-0-10_2.11` 套件的相依性。 此套件的版本應與 HDInsight 上的 Spark 版本相符。 對於 Spark 2.2.0 (適用於 HDInsight 3.6)，您可以在 [https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar](https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar) 上找到不同專案類型的相依性資訊。

對於本教學課程中提供的 Jupyter Notebook，下列資料格會載入此套件相依性：

```
%%configure -f
{
    "conf": {
        "spark.jars.packages": "org.apache.spark:spark-sql-kafka-0-10_2.11:2.2.0",
        "spark.jars.excludes": "org.scala-lang:scala-reflect,org.apache.spark:spark-tags_2.11"
    }
}
```

## <a name="create-the-clusters"></a>建立叢集

Apache Kafka on HDInsight 不提供透過公用網際網路存取 Kafka 訊息代理程式。 任何使用 Kafka 的項目都必須位於相同的 Azure 虛擬網路中。 在本教學課程中，Kafka 和 Spark 叢集位於相同的 Azure 虛擬網路中。 

下圖顯示 Spark 與 Kafka 之間的通訊流程︰

![Azure 虛擬網路中的 Spark 和 Kafka 叢集圖表](./media/hdinsight-apache-spark-with-kafka/spark-kafka-vnet.png)

> [!NOTE]
> Kafka 服務僅限於虛擬網路內的通訊。 叢集上的其他服務 (例如 SSH 和 Ambari) 可以透過網際網路存取。 如需有關適用於 HDInsight 的公用連接埠詳細資訊，請參閱 [HDInsight 所使用的連接埠和 URI](hdinsight-hadoop-port-settings-for-services.md)。

若要建立 Azure 虛擬網路，然後在其中建立 Kafka 和 Spark 叢集，請使用下列步驟：

1. 使用以下按鈕，在 Azure 入口網站中登入 Azure 並開啟範本。
    
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-spark-kafka-structured-streaming%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apache-spark-with-kafka/deploy-to-azure.png" alt="Deploy to Azure"></a>
    
    Azure Resource Manager 範本位於 **https://raw.githubusercontent.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming/master/azuredeploy.json**。

    此範本會建立下列資源：

    * HDInsight 3.6 叢集上的 Kafka。
    * HDInsight 3.6 叢集上的 Spark 2.2.0。
    * Azure 虛擬網路，其中包含 HDInsight 叢集。

    > [!IMPORTANT]
    > 本教學課程中使用的結構化串流 Notebook 需要 HDInsight 3.6 上的 Spark 2.2.0。 如果您在 HDInsight 上使用較早版本的 Spark，當使用 Notebook 時會收到錯誤。

2. 使用下列資訊，填入 [自訂範本] 區段上的項目︰

    | 設定 | 值 |
    | --- | --- |
    | 訂用帳戶 | 您的 Azure 訂用帳戶 |
    | 資源群組 | 包含資源的資源群組。 |
    | 位置 | 資源建立所在的 Azure 區域。 |
    | Spark 叢集名稱 | Spark 叢集的名稱。 前六個字元必須與 Kafka 叢集名稱不同。 |
    | Kafka 叢集名稱 | Kafka 叢集的名稱。 前六個字元必須與 Spark 叢集名稱不同。 |
    | 叢集登入使用者名稱 | 叢集的管理員使用者名稱。 |
    | 叢集登入密碼 | 叢集的管理員使用者密碼。 |
    | SSH 使用者名稱 | 要為叢集建立的 SSH 使用者。 |
    | SSH 密碼 | SSH 使用者的密碼。 |
   
    ![自訂範本的螢幕擷取畫面](./media/hdinsight-apache-kafka-spark-structured-streaming/spark-kafka-template.png)

3. 讀取**條款及條件**，然後選取 [我同意上方所述的條款及條件]

4. 最後，核取 [釘選到儀表板]，然後選取 [購買]。 

> [!NOTE]
> 建立叢集可能需要 20 分鐘的時間。

## <a name="upload-the-notebook"></a>上傳 Notebook

若要將專案中的 Notebook 上傳至您在 HDInsight 叢集上的 Spark，請使用下列步驟：

1. 從 [https://github.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming](https://github.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming) 下載專案。

1. 在網頁瀏覽器中，連線到您 Spark 叢集上的 Jupyter Notebook。 在下列 URL 中，將 `CLUSTERNAME`取代為您的 __Spark__ 叢集名稱：

        https://CLUSTERNAME.azurehdinsight.net/jupyter

    出現提示時，輸入您在建立叢集時所使用的叢集登入 (admin) 和密碼。

2. 從頁面右上方，使用 [上傳] 按鈕將 __spark-structured-streaming-kafka.ipynb__ 檔案上傳至叢集。 選取 [開啟] 以開始上傳。

    ![使用上傳按鈕來選取和上傳 Notebook](./media/hdinsight-apache-kafka-spark-structured-streaming/upload-button.png)

    ![選取 KafkaStreaming.ipynb 檔案](./media/hdinsight-apache-kafka-spark-structured-streaming/select-notebook.png)

3. 在 Notebook 清單中尋找 __spark-structured-streaming-kafka.ipynb__ 項目，然後選取旁邊的 [上傳] 按鈕。

    ![若要上傳 Notebook，請對 KafkaStreaming.ipynb 項目使用 [上傳] 按鈕](./media/hdinsight-apache-kafka-spark-structured-streaming/upload-notebook.png)


## <a name="use-the-notebook"></a>使用 Notebook

上傳檔案之後，請選取 __spark-structured-streaming-kafka.ipynb__ 項目以開啟 Notebook。 若想了解如何在 HDInsight 上使用 Spark 與 Kafka 的結構化串流，請遵循 Notebook 中的指示。

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

在本教學課程中，您已了解如何使用 Spark 結構化串流對 HDInsight 上的 Kafka 寫入及讀取資料。 請使用下列連結了解如何搭配使用 Storm 與 Kafka。

> [!div class="nextstepaction"]
> [搭配使用 Apache Storm 與 Kafka](hdinsight-apache-storm-with-kafka.md)
