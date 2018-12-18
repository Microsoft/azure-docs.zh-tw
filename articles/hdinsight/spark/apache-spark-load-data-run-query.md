---
title: '教學課程：在 Azure HDInsight 中的 Apache Spark 叢集上載入資料和執行查詢 '
description: 了解如何在 Azure HDInsight 中的 Spark 叢集上載入資料和執行互動式查詢。
services: azure-hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,mvc
ms.topic: tutorial
ms.author: jasonh
ms.date: 05/17/2018
ms.openlocfilehash: d59f04c5dde522f3d193f345ac59147ece9d86f0
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047552"
---
# <a name="tutorial-load-data-and-run-queries-on-an-apache-spark-cluster-in-azure-hdinsight"></a>教學課程：在 Azure HDInsight 中的 Apache Spark 叢集上載入資料和執行查詢

在本教學課程中，您將了解如何從 csv 檔案建立資料框架，以及如何在 Azure HDInsight 中對 Apache Spark 叢集執行互動式 Spark SQL 查詢。 在 Spark 中，資料框架是組織成具名資料行的分散式資料集合。 資料框架在概念上等同於關聯式資料庫中的資料表或 R/Python 中的資料框架。
 
在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 從 csv 檔案建立資料框架
> * 在資料框架上執行查詢

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

* 完成[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

## <a name="create-a-dataframe-from-a-csv-file"></a>從 csv 檔案建立資料框架

應用程式可使用 SQLContext 物件，從現有的彈性分散式資料集 (RDD)、Hive 資料表或資料來源建立資料框架。 下列螢幕擷取畫面顯示本教學課程中使用之 HVAC.csv 檔案的快照集。 該 csv 檔案會隨附於所有 HDInsight Spark 叢集。 資料會擷取相同建築物的溫度變化。
    
![互動式 Spark SQL 查詢的資料快照集](./media/apache-spark-load-data-run-query/hdinsight-spark-sample-data-interactive-spark-sql-query.png "互動式 Spark SQL 查詢的資料快照集")


1. 開啟您在必要條件一節中建立的 Jupyter Notebook。
2. 將以下程式碼貼入 Notebook 的空白儲存格，然後按 **SHIFT + ENTER** 以執行此程式碼。 此程式碼會匯入此案例所需的類型：

    ```PySpark
    from pyspark.sql import *
    from pyspark.sql.types import *
    ```

    在 Jupyter 中執行互動式查詢時，網頁瀏覽器視窗或索引標籤標題都會顯示 [(忙碌)] 狀態，以及 Notebook 的標題。 您也會在右上角的 **PySpark** 文字旁看到一個實心圓。 作業完成後，實心圓將變成空心圓。

    ![互動式 Spark SQL 查詢的狀態](./media/apache-spark-load-data-run-query/hdinsight-spark-interactive-spark-query-status.png "互動式 Spark SQL 查詢的狀態")

3. 執行下列程式碼，以建立資料框架和暫存資料表 (**hvac**)。 

    ```PySpark
    # Create an RDD from sample data
    csvFile = spark.read.csv('wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv', header=True, inferSchema=True)
    csvFile.write.saveAsTable("hvac")
    ```

    > [!NOTE]
    > 透過使用 PySpark 核心來建立 Notebook，當您執行第一個程式碼儲存格時，系統便會自動為您建立 SQL 內容。 您不需要明確建立任何內容。


## <a name="run-queries-on-the-dataframe"></a>在資料框架上執行查詢

建立資料表之後，您可以對資料執行互動式查詢。

1. 在 Notebook 的空白儲存格中，執行下列程式碼：

    ```PySpark
    %%sql
    SELECT buildingID, (targettemp - actualtemp) AS temp_diff, date FROM hvac WHERE date = \"6/1/13\"
    ```

   由於 PySpark 核心用於 Notebook 中，因此您現在可直接在暫存資料表 **hvac** 上執行互動式 SQL 查詢。

   此時會顯示下列表格式輸出。

     ![互動式 Spark 查詢結果的資料表輸出](./media/apache-spark-load-data-run-query/hdinsight-interactive-spark-query-result.png "互動式 Spark 查詢結果的資料表輸出")

3. 您也可以查看其他視覺效果中的結果。 若要查看相同輸出的區域圖表，請選取 [區域]，然後設定其他值，如下所示。

    ![互動式 Spark 查詢結果的區域圖表](./media/apache-spark-load-data-run-query/hdinsight-interactive-spark-query-result-area-chart.png "互動式 Spark 查詢結果的區域圖表")

10. 從 Notebook 的 [檔案] 功能表中，選取 [儲存與檢查點]。 

11. 如果您現在要開始進行[下一個教學課程](apache-spark-use-bi-tools.md)，請讓 Notebook 保持開啟。 如果尚不進行，請關閉 Notebook 以釋出叢集資源：從 Notebook 上的 [檔案] 功能表，選取 [關閉並終止]。

## <a name="clean-up-resources"></a>清除資源

利用 HDInsight，會將您的資料儲存於 Azure 儲存體或 Azure Data Lake Store 中，以便您在未使用叢集時安全地進行刪除。 您也需支付 HDInsight 叢集的費用 (即使未使用)。 由於叢集費用是儲存體費用的許多倍，所以刪除未使用的叢集符合經濟效益。 如果您打算立即進行下一個教學課程，則可能想要保留叢集。

在 Azure 入口網站中開啟叢集，然後選取 [刪除]。

![刪除 HDInsight 叢集](./media/apache-spark-load-data-run-query/hdinsight-azure-portal-delete-cluster.png "刪除 HDInsight 叢集")

您也可以選取資源群組名稱來開啟資源群組頁面，然後選取 [刪除資源群組]。 刪除資源群組時，會同時刪除 HDInsight Spark 叢集及預設儲存體帳戶。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

* 建立 Spark 資料框架。
* 對資料框架執行 Spark SQL。

前往下一篇文章，以查看如何將您在 Spark 中註冊的資料提取至 BI 分析工具，例如 Power BI。 
> [!div class="nextstepaction"]
> [使用 BI 工具分析資料](apache-spark-use-bi-tools.md)

