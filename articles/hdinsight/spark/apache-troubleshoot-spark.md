---
title: 在 Azure HDInsight 中對 Spark 進行疑難排解
description: 取得有關使用 Apache Spark 和 Azure HDInsight 的常見問題解答。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.topic: conceptual
ms.date: 11/2/2017
ms.openlocfilehash: 7c7f89864d9394ff4527f9a0354b9276f7c01c49
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39591732"
---
# <a name="troubleshoot-spark-by-using-azure-hdinsight"></a>使用 Azure HDInsight 為 Spark 進行疑難排解

了解在 Apache Ambari 中使用 Apache Spark 承載時最常發生的問題及其解決方法。

## <a name="how-do-i-configure-a-spark-application-by-using-ambari-on-clusters"></a>如何使用 Ambari 在叢集上設定 Spark 應用程式？

### <a name="resolution-steps"></a>解決步驟

此程序的設定值之前已在 HDInsight 中設定過。 若要判斷需要設定哪些 Spark 設定以及設為哪些值，請參閱[造成 Spark 應用程式 OutOfMemoryError 例外狀況的原因](#what-causes-a-spark-application-outofmemoryerror-exception)。 

1. 在叢集清單中選取 [Spark2]。

    ![從清單中選取叢集](./media/apache-troubleshoot-spark/update-config-1.png)

2. 選取 [ **設定** ] 索引標籤。

    ![選取 [設定] 索引標籤](./media/apache-troubleshoot-spark/update-config-2.png)

3. 在設定清單中選取 [Custom-spark2-defaults]。

    ![選取 [Custom-spark2-defaults]](./media/apache-troubleshoot-spark/update-config-3.png)

4. 尋找需要調整的設定值，例如 **spark.executor.memory**。 在本例中，值 **4608m** 太高。

    ![選取 [spark.executor.memory] 欄位](./media/apache-troubleshoot-spark/update-config-4.png)

5. 將值設為建議的設定。 這項設定的建議值為 **2048m**。

    ![將值變更為 2048m](./media/apache-troubleshoot-spark/update-config-5.png)

6. 儲存此值，然後儲存設定。 在工具列上，選取 [儲存]。

    ![儲存設定值和設定](./media/apache-troubleshoot-spark/update-config-6a.png)

    如有任何設定需要注意，您會收到通知。 記下項目，然後選取 [仍要繼續]。 

    ![選取 [仍要繼續]](./media/apache-troubleshoot-spark/update-config-6b.png)

    註記設定變更，然後選取 [儲存]。

    ![輸入所做變更的附註內容](./media/apache-troubleshoot-spark/update-config-6c.png)

7. 每次儲存設定時，系統都會提示您重新啟動服務。 選取 [重新啟動]。

    ![選取 [重新啟動]](./media/apache-troubleshoot-spark/update-config-7a.png)

    確認重新啟動。

    ![選取 [確認全部重新啟動]](./media/apache-troubleshoot-spark/update-config-7b.png)

    您可以檢閱執行中的處理序。

    ![檢閱執行中的處理序](./media/apache-troubleshoot-spark/update-config-7c.png)

8. 您可以新增設定。 在設定清單中選取 [Custom-spark2-defaults]，然後選取 [新增屬性]。

    ![選取 [新增屬性]](./media/apache-troubleshoot-spark/update-config-8.png)

9. 定義新的屬性。 您可以使用特定設定 (例如資料類型) 的對話方塊定義單一屬性。 或者，您也可以使用一行一定義的方式定義多個屬性。 

    在本例中，**spark.driver.memory** 屬性是以值 **4g** 定義。

    ![定義新的屬性](./media/apache-troubleshoot-spark/update-config-9.png)

10. 儲存設定，然後重新啟動服務，如步驟 6 和 7 中所述。

這些變更為全叢集的，但可在提交 Spark 作業時予以覆寫。

### <a name="additional-reading"></a>其他閱讀資料

[提交 HDInsight 叢集上的 Spark 作業](https://blogs.msdn.microsoft.com/azuredatalake/2017/01/06/spark-job-submission-on-hdinsight-101/)


## <a name="how-do-i-configure-a-spark-application-by-using-a-jupyter-notebook-on-clusters"></a>如何使用 Jupyter 筆記本在叢集上設定 Spark 應用程式？

### <a name="resolution-steps"></a>解決步驟

1. 若要判斷需要設定哪些 Spark 設定以及設為哪些值，請參閱[造成 Spark 應用程式 OutOfMemoryError 例外狀況的原因](#what-causes-a-spark-application-outofmemoryerror-exception)。

2. 在 Jupyter 筆記本第一個資料格中的 **%%configure** 指示詞後面，以有效的 JSON 格式指定 Spark 設定。 請視需要變更實際值：

    ![新增設定](./media/apache-troubleshoot-spark/add-configuration-cell.png)

### <a name="additional-reading"></a>其他閱讀資料

[提交 HDInsight 叢集上的 Spark 作業](https://blogs.msdn.microsoft.com/azuredatalake/2017/01/06/spark-job-submission-on-hdinsight-101/)


## <a name="how-do-i-configure-a-spark-application-by-using-livy-on-clusters"></a>如何使用 Livy 在叢集上設定 Spark 應用程式？

### <a name="resolution-steps"></a>解決步驟

1. 若要判斷需要設定哪些 Spark 設定以及設為哪些值，請參閱[造成 Spark 應用程式 OutOfMemoryError 例外狀況的原因](#what-causes-a-spark-application-outofmemoryerror-exception)。 

2. 使用 cURL 等 REST 用戶端，將 Spark 應用程式提交給 Livy。 使用類似下列的命令。 請視需要變更實際值：

    ```apache
    curl -k --user 'username:password' -v -H 'Content-Type: application/json' -X POST -d '{ "file":"wasb://container@storageaccountname.blob.core.windows.net/example/jars/sparkapplication.jar", "className":"com.microsoft.spark.application", "numExecutors":4, "executorMemory":"4g", "executorCores":2, "driverMemory":"8g", "driverCores":4}'  
    ```

### <a name="additional-reading"></a>其他閱讀資料

[提交 HDInsight 叢集上的 Spark 作業](https://blogs.msdn.microsoft.com/azuredatalake/2017/01/06/spark-job-submission-on-hdinsight-101/)


## <a name="how-do-i-configure-a-spark-application-by-using-spark-submit-on-clusters"></a>如何使用 spark-submit 在叢集上設定 Spark 應用程式？

### <a name="resolution-steps"></a>解決步驟

1. 若要判斷需要設定哪些 Spark 設定以及設為哪些值，請參閱[造成 Spark 應用程式 OutOfMemoryError 例外狀況的原因](#what-causes-a-spark-application-outofmemoryerror-exception)。

2. 使用類似下列的命令啟動 spark-shell。 請視需要變更設定的實際值： 

    ```apache
    spark-submit --master yarn-cluster --class com.microsoft.spark.application --num-executors 4 --executor-memory 4g --executor-cores 2 --driver-memory 8g --driver-cores 4 /home/user/spark/sparkapplication.jar
    ```

### <a name="additional-reading"></a>其他閱讀資料

[提交 HDInsight 叢集上的 Spark 作業](https://blogs.msdn.microsoft.com/azuredatalake/2017/01/06/spark-job-submission-on-hdinsight-101/)


## <a name="what-causes-a-spark-application-outofmemoryerror-exception"></a>造成 Spark 應用程式 OutofMemoryError 例外狀況的原因為何？

### <a name="detailed-description"></a>詳細描述

Spark 應用程式因下列無法攔截的例外狀況類型而失敗：

```apache
ERROR Executor: Exception in task 7.0 in stage 6.0 (TID 439) 

java.lang.OutOfMemoryError 
    at java.io.ByteArrayOutputStream.hugeCapacity(Unknown Source) 
    at java.io.ByteArrayOutputStream.grow(Unknown Source) 
    at java.io.ByteArrayOutputStream.ensureCapacity(Unknown Source) 
    at java.io.ByteArrayOutputStream.write(Unknown Source) 
    at java.io.ObjectOutputStream$BlockDataOutputStream.drain(Unknown Source) 
    at java.io.ObjectOutputStream$BlockDataOutputStream.setBlockDataMode(Unknown Source) 
    at java.io.ObjectOutputStream.writeObject0(Unknown Source) 
    at java.io.ObjectOutputStream.writeObject(Unknown Source) 
    at org.apache.spark.serializer.JavaSerializationStream.writeObject(JavaSerializer.scala:44) 
    at org.apache.spark.serializer.JavaSerializerInstance.serialize(JavaSerializer.scala:101) 
    at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:239) 
    at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source) 
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source) 
    at java.lang.Thread.run(Unknown Source) 
```

```apache
ERROR SparkUncaughtExceptionHandler: Uncaught exception in thread Thread[Executor task launch worker-0,5,main] 

java.lang.OutOfMemoryError 
    at java.io.ByteArrayOutputStream.hugeCapacity(Unknown Source) 
    at java.io.ByteArrayOutputStream.grow(Unknown Source) 
    at java.io.ByteArrayOutputStream.ensureCapacity(Unknown Source) 
    at java.io.ByteArrayOutputStream.write(Unknown Source) 
    at java.io.ObjectOutputStream$BlockDataOutputStream.drain(Unknown Source) 
    at java.io.ObjectOutputStream$BlockDataOutputStream.setBlockDataMode(Unknown Source) 
    at java.io.ObjectOutputStream.writeObject0(Unknown Source) 
    at java.io.ObjectOutputStream.writeObject(Unknown Source) 
    at org.apache.spark.serializer.JavaSerializationStream.writeObject(JavaSerializer.scala:44) 
    at org.apache.spark.serializer.JavaSerializerInstance.serialize(JavaSerializer.scala:101) 
    at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:239) 
    at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source) 
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source) 
    at java.lang.Thread.run(Unknown Source) 
```

### <a name="probable-cause"></a>可能的原因

此例外狀況最可能發生的原因，就是未配置足夠的堆積記憶體給 Java 虛擬機器 (JVM)。 這些 JVM 會當作 Spark 應用程式一部分的執行程式或驅動程式啟動。 

### <a name="resolution-steps"></a>解決步驟

1. 決定 Spark 應用程式可處理的資料大小上限。 您可以依輸入資料的大小上限、由轉換輸入資料所產生的中繼資料，以及應用程式進一步轉換中繼資料時所產生的輸出資料粗略估算。 如果您無法進行正式的初始估算，也可以設定為反覆的程序。 

2. 請確定您要使用的 HDInsight 叢集具有足夠的記憶體資源及核心，才能採用 Spark 應用程式。 若要判斷這點，請檢視 YARN UI 的叢集計量一節，查看**已使用記憶體**和**記憶體總計**，以及**已使用的 VCore** 和**VCore 總計**的值。

3. 將下列 Spark 設定設為適當的值，不要超過可用記憶體及核心的 90%。 這些值應該在 Spark 應用程式的記憶體需求範圍內： 

    ```apache
    spark.executor.instances (Example: 8 for 8 executor count) 
    spark.executor.memory (Example: 4g for 4 GB) 
    spark.yarn.executor.memoryOverhead (Example: 384m for 384 MB) 
    spark.executor.cores (Example: 2 for 2 cores per executor) 
    spark.driver.memory (Example: 8g for 8GB) 
    spark.driver.cores (Example: 4 for 4 cores)   
    spark.yarn.driver.memoryOverhead (Example: 384m for 384MB) 
    ```

    若要計算所有執行程式使用的記憶體總計： 
    
    ```apache
    spark.executor.instances * (spark.executor.memory + spark.yarn.executor.memoryOverhead) 
    ```
   若要計算驅動程式使用的記憶體總計：
    
    ```apache
    spark.driver.memory + spark.yarn.driver.memoryOverhead
    ```

### <a name="additional-reading"></a>其他閱讀資料

- [Spark 記憶體管理概觀](http://spark.apache.org/docs/latest/tuning.html#memory-management-overview)
- [Debug a Spark application on an HDInsight cluster](https://blogs.msdn.microsoft.com/azuredatalake/2016/12/19/spark-debugging-101/) (在 HDInsight 叢集上偵錯 Spark 應用程式)


### <a name="see-also"></a>另請參閱
[使用 Azure HDInsight 進行疑難排解](../../hdinsight/hdinsight-troubleshoot-guide.md)

