---
title: '教學課程：使用 Azure HDInsight 中的 Apache Spark 處理來自 Azure 事件中樞的資料 '
description: 將 Azure HDInsight 中的 Apache Spark 連線至 Azure 事件中樞，並處理串流資料。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive,mvc
ms.topic: conceptual
ms.date: 06/14/2018
ms.openlocfilehash: 9cdb5ae31e2743b5ebe877ddd8d6680423e3d9b2
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046247"
---
# <a name="tutorial-process-tweets-using-azure-event-hubs-and-spark-in-hdinsight"></a>教學課程：使用 Azure 事件中樞與 HDInsight 中的 Spark 處理推文

在本教學課程中，您會了解如何建立 Apache Spark 串流應用程式以將推文傳送至 Azure 事件中樞，並建立另一個應用程式以便從事件中樞讀取推文。 如需 Spark 串流的詳細說明，請參閱 [Apache Spark 串流概觀](http://spark.apache.org/docs/latest/streaming-programming-guide.html#overview)。 HDInsight 會將相同的串流功能帶入 Azure 上的 Spark 叢集。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 將訊息傳送至 Azure 事件中樞
> * 從 Azure 事件中樞讀取訊息

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

* **完成本文[教學課程：在 Azure HDInsight 中的 Apache Spark 叢集上載入資料和執行查詢](./apache-spark-load-data-run-query.md)**。

## <a name="create-a-twitter-application"></a>建立 Twitter 應用程式

若要收到推文的串流，您必須在 Twitter 中建立應用程式。 依照下列指示建立 Twitter 應用程式，並寫下完成本教學課程所需的值。

1. 瀏覽至 [Twitter 應用程式管理](https://apps.twitter.com/) \(英文\)。
2. 選取 [Create New App] \(建立新的應用程式\)。
3. 提供下列值：

    - 名稱：提供應用程式名稱。 針對本教學課程所使用的值是 **HDISparkStreamApp0423**。 此名稱必須是唯一的名稱。
    - 描述：提供應用程式的簡短描述。 針對本教學課程所使用的值是**簡單的 HDInsight Spark 串流應用程式**。
    - 網站：提供應用程式的網站。 不需是有效的網站。  針對本教學課程所使用的值是 **http://www.contoso.com**。
    - 回呼 URL：您可以保留空白。

4. 選取 [Yes, I have read and agree to the Twitter Developer Agreement] \(是，我已閱讀並同意 Twitter 開發人員合約\)，然後選取 [Create your Twitter application] \(建立 Twitter 應用程式\)。
5. 選取 [ **金鑰和存取權杖** ] 索引標籤。
6. 選取頁面底部的 [Create my access token] \(建立我的存取權杖\)。
7. 寫下頁面上的下列值。  在本教學課程後續的內容中，您會需要這些值：

    - **取用者金鑰 (API 金鑰)**    
    - **取用者祕密 (API 祕密)**  
    - **存取權杖**
    - **存取權杖祕密**   

## <a name="create-an-azure-event-hub"></a>建立 Azure 事件中樞

您會使用此事件中樞來儲存推文。

1. 登入 [Azure 入口網站](https://ms.portal.azure.com)。
2. 選取螢幕左上方的 [建立資源]。
3. 選取 [物聯網]，然後選取 [事件中樞]。

    ![建立 Spark 串流範例的事件中樞](./media/apache-spark-eventhub-streaming/hdinsight-create-event-hub-for-spark-streaming.png "建立 Spark 串流範例的事件中樞")
4. 針對新的事件中樞命名空間，輸入下列值：

    - **名稱**：輸入事件中樞的名稱。  針對本教學課程所使用的值是 **myeventhubns20180403**。
    - **定價層**：選取 [標準]。
    - **資源群組**：您可以選擇建立新的或選取適用於 Spark 叢集的資源群組。 
    - **位置**：選取與 HDInsight 中 Apache Spark 叢集相同的**位置**，以降低延遲和成本。

    ![提供 Spark 串流範例的事件中樞名稱](./media/apache-spark-eventhub-streaming/hdinsight-provide-event-hub-name-for-spark-streaming.png "提供 Spark 串流範例的事件中樞名稱")
5. 選取 [建立] 來建立命名空間。

6. 使用下列指示來開啟事件中樞命名空間：

    1. 從入口網站，選取 [所有服務]。
    2. 在篩選方塊中，輸入**事件中樞**。
    3. 按兩下您建立的命名空間。
    4. 選取 [+ 事件中樞]。

6. 在 [事件中樞] 命名空間清單中，按一下新建立的命名空間。      
5. 選取 [事件中樞]，然後選取 [+ 事件中樞] 以建立新的事件中樞。
  

6. 輸入下列值：

    - 名稱：提供事件中樞的名稱。
    - 分割區計數：10
    - 訊息保留期：1。 
   
    ![提供 Spark 串流範例的事件中樞詳細資料](./media/apache-spark-eventhub-streaming/hdinsight-provide-event-hub-details-for-spark-streaming-example.png "提供 Spark 串流範例的事件中樞詳細資料")

7. 選取 [建立] 。
8. 針對命名空間選取 [共用存取原則] (請注意，不是事件中樞共用存取原則)，然後選取 [RootManageSharedAccessKey]。
    
     ![設定 Spark 串流範例的事件中樞原則](./media/apache-spark-eventhub-streaming/hdinsight-set-event-hub-policies-for-spark-streaming-example.png "設定 Spark 串流範例的事件中樞原則")

9. 儲存**主索引鍵**和**連接字串-主索引鍵**值，以便稍後在本教學課程中使用。

     ![檢視 Spark 串流範例的事件中樞原則金鑰](./media/apache-spark-eventhub-streaming/hdinsight-view-event-hub-policy-keys.png "檢視 Spark 串流範例的事件中樞原則金鑰")


## <a name="send-tweets-to-the-event-hub"></a>將推文傳送至事件中樞

您需要建立一個 Jupyter Notebook，並將其命名為 **SendTweetsToEventHub**。 

1. 執行下列程式碼，以新增外部 Maven 程式庫：

    ```
    %%configure
    {"conf":{"spark.jars.packages":"com.microsoft.azure:azure-eventhubs-spark_2.11:2.2.0,org.twitter4j:twitter4j-core:4.0.6"}}
    ```

2. 執行下列程式碼，以將推文傳送到您的事件中樞：

    ```
    import java.util._
    import scala.collection.JavaConverters._
    import java.util.concurrent._
    
    import org.apache.spark._
    import org.apache.spark.streaming._
    import org.apache.spark.eventhubs.ConnectionStringBuilder

    // Event hub configurations
    // Replace values below with yours        
    val eventHubName = "<Event hub name>"
    val eventHubNSConnStr = "<Event hub namespace connection string>"
    val connStr = ConnectionStringBuilder(eventHubNSConnStr).setEventHubName(eventHubName).build 
    
    import com.microsoft.azure.eventhubs._
    val pool = Executors.newFixedThreadPool(1)
    val eventHubClient = EventHubClient.create(connStr.toString(), pool)
    
    def sendEvent(message: String) = {
          val messageData = EventData.create(message.getBytes("UTF-8"))
          eventHubClient.get().send(messageData)
          println("Sent event: " + message + "\n")
    }
    
    import twitter4j._
    import twitter4j.TwitterFactory
    import twitter4j.Twitter
    import twitter4j.conf.ConfigurationBuilder

    // Twitter application configurations
    // Replace values below with yours   
    val twitterConsumerKey = "<CONSUMER KEY>"
    val twitterConsumerSecret = "<CONSUMER SECRET>"
    val twitterOauthAccessToken = "<ACCESS TOKEN>"
    val twitterOauthTokenSecret = "<TOKEN SECRET>"
    
    val cb = new ConfigurationBuilder()
    cb.setDebugEnabled(true).setOAuthConsumerKey(twitterConsumerKey).setOAuthConsumerSecret(twitterConsumerSecret).setOAuthAccessToken(twitterOauthAccessToken).setOAuthAccessTokenSecret(twitterOauthTokenSecret)
    
    val twitterFactory = new TwitterFactory(cb.build())
    val twitter = twitterFactory.getInstance()

    // Getting tweets with keyword "Azure" and sending them to the Event Hub in realtime!
    
    val query = new Query(" #Azure ")
    query.setCount(100)
    query.lang("en")
    var finished = false
    while (!finished) {
      val result = twitter.search(query)
      val statuses = result.getTweets()
      var lowestStatusId = Long.MaxValue
      for (status <- statuses.asScala) {
        if(!status.isRetweet()){
          sendEvent(status.getText())
        }
        lowestStatusId = Math.min(status.getId(), lowestStatusId)
        Thread.sleep(2000)
      }
      query.setMaxId(lowestStatusId - 1)
    }
    
    // Closing connection to the Event Hub
    eventHubClient.get().close()
    ```

3. 在 Azure 入口網站中開啟事件中樞。  在 [概觀] 上，您應該會看到某些圖表顯示傳送至事件中樞的訊息。

## <a name="read-tweets-from-the-event-hub"></a>從事件中樞讀取推文

您需要建立另一個 Jupyter Notebook，並將其命名為 **ReadTweetsFromEventHub**。 

1. 執行下列程式碼，以新增外部 Maven 程式庫：

    ```
    %%configure -f
    {"conf":{"spark.jars.packages":"com.microsoft.azure:azure-eventhubs-spark_2.11:2.2.0"}}
    ```
2. 執行下列程式碼，以從您的事件中樞讀取推文：

    ```
    import org.apache.spark.eventhubs._
    // Event hub configurations
    // Replace values below with yours        
    val eventHubName = "<Event hub name>"
    val eventHubNSConnStr = "<Event hub namespace connection string>"
    val connStr = ConnectionStringBuilder(eventHubNSConnStr).setEventHubName(eventHubName).build 
    
    val customEventhubParameters = EventHubsConf(connStr).setMaxEventsPerTrigger(5)
    val incomingStream = spark.readStream.format("eventhubs").options(customEventhubParameters.toMap).load()
    //incomingStream.printSchema    
    
    import org.apache.spark.sql.types._
    import org.apache.spark.sql.functions._
    
    // Event Hub message format is JSON and contains "body" field
    // Body is binary, so you cast it to string to see the actual content of the message
    val messages = incomingStream.withColumn("Offset", $"offset".cast(LongType)).withColumn("Time (readable)", $"enqueuedTime".cast(TimestampType)).withColumn("Timestamp", $"enqueuedTime".cast(LongType)).withColumn("Body", $"body".cast(StringType)).select("Offset", "Time (readable)", "Timestamp", "Body")
    
    messages.printSchema
    
    messages.writeStream.outputMode("append").format("console").option("truncate", false).start().awaitTermination()
    ```

## <a name="clean-up-resources"></a>清除資源

利用 HDInsight，會將您的資料儲存於 Azure 儲存體或 Azure Data Lake Store 中，以便您在未使用叢集時安全地進行刪除。 您也需支付 HDInsight 叢集的費用 (即使未使用)。 由於叢集費用是儲存體費用的許多倍，所以刪除未使用的叢集符合經濟效益。 如果您打算立即進行下一個教學課程，則可能想要保留叢集。

在 Azure 入口網站中開啟叢集，然後選取 [刪除]。

![刪除 HDInsight 叢集](./media/apache-spark-load-data-run-query/hdinsight-azure-portal-delete-cluster.png "刪除 HDInsight 叢集")

您也可以選取資源群組名稱來開啟資源群組頁面，然後選取 [刪除資源群組]。 刪除資源群組時，會同時刪除 HDInsight Spark 叢集及預設儲存體帳戶。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

* 從事件中樞讀取訊息。
前進到下一篇文章，以查看您如何建立機器學習應用程式。 

> [!div class="nextstepaction"]
> [建立機器學習應用程式](./apache-spark-ipython-notebook-machine-learning.md)


