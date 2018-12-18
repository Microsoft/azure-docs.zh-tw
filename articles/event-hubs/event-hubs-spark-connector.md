---
title: 整合 Apache Spark 與 Azure 事件中樞 | Microsoft Docs
description: 整合 Apache Spark 以啟用事件中樞的結構化串流
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.service: event-hubs
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/21/2018
ms.author: shvija
ms.openlocfilehash: 688daedf29bbd68c7451be66b47ac90e404d4093
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746655"
---
# <a name="integrating-apache-spark-with-azure-event-hubs"></a>整合 Apache Spark 與 Azure 事件中樞

Azure 事件中樞會與 [Apache Spark](https://spark.apache.org/) 緊密整合，讓您建置分散式串流應用程式。 這項整合支援 [Spark Core](http://spark.apache.org/docs/latest/rdd-programming-guide.html)、[Spark Streaming](http://spark.apache.org/docs/latest/streaming-programming-guide.html) 和[結構化串流](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)。 適用於 Apache Spark 的事件中樞連接器可於 [GitHub](https://github.com/Azure/azure-event-hubs-spark) 上取得。 此文件庫也可用於 [Maven 中央存放庫](http://search.maven.org/#artifactdetails%7Ccom.microsoft.azure%7Cazure-eventhubs-spark_2.11%7C2.1.6%7C)中的 Maven 專案。

本文說明如何在 [Azure Databricks](https://azure.microsoft.com/services/databricks/) 中建立連續的應用程式。 雖然本文使用 Azure Databricks，但 Spark 叢集也可與 [HDInsight](../hdinsight/spark/apache-spark-overview.md) 搭配使用。

本文中的範例會使用兩個 Scala Notebook，一個用於串流事件中樞的事件，另一個用於將事件傳回給它。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有帳戶，請[建立免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
* 事件中樞執行個體。 如果您沒有，請[建立一個](event-hubs-create.md)。
* [Azure Databricks](https://azure.microsoft.com/services/databricks/) 執行個體。 如果您沒有，請[建立一個](../azure-databricks/quickstart-create-databricks-workspace-portal.md)。
* [使用 Maven 座標建立文件庫](https://docs.databricks.com/user-guide/libraries.html#upload-a-maven-package-or-spark-package)：`com.microsoft.azure:azure‐eventhubs‐spark_2.11:2.3.1`。

使用以下程式碼從事件中樞串流事件：

```scala
import org.apache.spark.eventhubs._

// To connect to an event hub, EntityPath is required as part of the connection string.
// Here, we assume that the connection string from the Azure portal does not have the EntityPath part.
val connectionString = ConnectionStringBuilder("{EVENT HUB CONNECTION STRING FROM AZURE PORTAL}")
  .setEventHubName("{EVENT HUB NAME}")
  .build 
val ehConf = EventHubsConf(connectionString)
  .setStartingPosition(EventPosition.fromEndOfStream)

// Create a stream that reads data from the specified Event Hub.
val reader = spark.readStream
  .format("eventhubs")
  .options(ehConf.toMap)
  .load()
val eventhubs = reader.select($"body" cast "string")

eventhubs.writeStream
  .format("console")
  .outputMode("append")
  .start()
  .awaitTermination()
```
下列程式碼會透過 Spark 批次 API 將事件傳送至您的事件中樞。 您也可以撰寫串流查詢，將事件傳送至事件中樞：

```scala
import org.apache.spark.eventhubs._
import org.apache.spark.sql.functions._

// To connect to an event hub, EntityPath is required as part of the connection string.
// Here, we assume that the connection string from the Azure portal does not have the EntityPath part.
val connectionString = ConnectionStringBuilder("{EVENT HUB CONNECTION STRING FROM AZURE PORTAL}")
  .setEventHubName("{EVENT HUB NAME}")
  .build
val ehConf = EventHubsConf(connectionString)

// Create random strings as the body of the message.
val bodyColumn = concat(lit("random nunmber: "), rand()).as("body")

// Write 200 rows to the specified event hub.
val df = spark.range(200).select(bodyColumn)
df.write
  .format("eventhubs")
  .options(ehConf.toMap)
  .save() 
```

## <a name="next-steps"></a>後續步驟

現在您知道如何使用適用於 Apache Spark 的事件中樞連接器設定可擴充且容錯的資料流。 請使用下列連結，深入了解如何搭配使用事件中樞與結構化串流和 Spark 串流：

* [結構化串流 + Azure 事件中樞整合指南](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md)
* [Spark 串流 + 事件中樞整合指南](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/spark-streaming-eventhubs-integration.md)
