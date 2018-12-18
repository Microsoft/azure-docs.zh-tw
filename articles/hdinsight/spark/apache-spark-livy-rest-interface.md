---
title: 使用 Livy Spark 將作業提交至 Azure HDInsight 上的 Spark 叢集
description: 了解如何使用 Apache Spark REST API 從遠端將 Spark 作業提交至 Azure HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 07/18/2018
ms.openlocfilehash: 677c7d27d34725b75c5dfed70cc377735f5d7d61
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045207"
---
# <a name="use-apache-spark-rest-api-to-submit-remote-jobs-to-an-hdinsight-spark-cluster"></a>使用 Apache Spark REST API 將遠端作業提交至 HDInsight Spark 叢集

了解如何使用可將遠端作業提交至 Azure HDInsight Spark 叢集的 Livy (也就是 Apache Spark REST API)。 如需詳細文件，請參閱[http://livy.incubator.apache.org/](http://livy.incubator.apache.org/)。

您可以使用 Livy 執行互動式 Spark 殼層，或提交要在 Spark 上執行的批次作業。 本文將討論如何使用 Livy 提交批次作業。 本文中的程式碼片段會使用 cURL 向 Livy Spark 端點發出 REST API 呼叫。

**必要條件：**

* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

* [cURL](http://curl.haxx.se/)。 本文使用 cURL 示範如何對 HDInsight Spark 叢集進行 REST API 呼叫。

## <a name="submit-a-livy-spark-batch-job"></a>提交 Livy Spark 批次作業
在提交批次作業之前，您必須將應用程式 jar 上傳至與叢集相關聯的叢集儲存體。 您可以使用命令列公用程式 [**AzCopy**](../../storage/common/storage-use-azcopy.md) 來執行此動作。 此外也有各種用戶端可用來上傳資料。 您可以在 [在 HDInsight 上將 Hadoop 作業的資料上傳](../hdinsight-upload-data.md)中找到其詳細資訊。

    curl -k --user "<hdinsight user>:<user password>" -v -H <content-type> -X POST -d '{ "file":"<path to application jar>", "className":"<classname in jar>" }' 'https://<spark_cluster_name>.azurehdinsight.net/livy/batches' -H "X-Requested-By: admin"

**範例**：

* 如果 jar 檔案位於叢集儲存體 (WASB) 上
  
        curl -k --user "admin:mypassword1!" -v -H 'Content-Type: application/json' -X POST -d '{ "file":"wasb://mycontainer@mystorageaccount.blob.core.windows.net/data/SparkSimpleTest.jar", "className":"com.microsoft.spark.test.SimpleFile" }' "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
* 如果您需要在輸入檔案 (在此範例中為 input.txt) 中傳遞 jar 檔案名稱和類別名稱
  
        curl -k  --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"

## <a name="get-information-on-livy-spark-batches-running-on-the-cluster"></a>取得在叢集上執行之 Livy Spark 批次的相關資訊
    curl -k --user "<hdinsight user>:<user password>" -v -X GET "https://<spark_cluster_name>.azurehdinsight.net/livy/batches"

**範例**：

* 如果您想要擷取在叢集上執行的所有 Livy Spark 批次：
  
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches" 
* 如果您想要擷取具有指定批次識別碼的特定批次
  
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches/{batchId}"

## <a name="delete-a-livy-spark-batch-job"></a>將 Livy Spark 批次作業刪除
    curl -k --user "<hdinsight user>:<user password>" -v -X DELETE "https://<spark_cluster_name>.azurehdinsight.net/livy/batches/{batchId}"

**範例**：

    curl -k --user "admin:mypassword1!" -v -X DELETE "https://mysparkcluster.azurehdinsight.net/livy/batches/{batchId}"

## <a name="livy-spark-and-high-availability"></a>Livy Spark 與高可用性
Livy 可為在叢集上執行的 Spark 作業提供高可用性。 以下是一些範例。

* 如果在您從遠端將作業提交給 Spark 叢集之後，Livy 服務當機，作業將會繼續在背景執行。 當 Livy 恢復運作時，它會還原作業的狀態並回報。
* 適用於 HDInsight 的 Jupyter Notebook 是由 Livy 在後端提供技術支援。 如果在 Notebook 執行 Spark 作業時，Livy 服務重新啟動，Notebook 就會繼續執行程式碼單元。 

## <a name="show-me-an-example"></a>請舉例說明
在本節中，我們將透過範例了解如何使用 Livy Spark 來提交批次作業、監視作業的進度，然後加以刪除。 我們在此範例中使用的應用程式，就是 [建立獨立 Scala 應用程式，並在 HDInsight Spark 叢集上執行](apache-spark-create-standalone-application.md)一文中所開發的應用程式。 這裡的步驟假設：

* 您已將應用程式 jar 複製到與叢集相關聯的儲存體帳戶。
* 您已將 CuRL 安裝在要嘗試這些步驟的電腦上。

執行下列步驟：

1. 我們先確認 Livy Spark 正在叢集上執行。 我們可以取得執行中的批次清單，加以確認。 如果您是第一次使用 Livy 執行作業，輸出應會傳回零。
   
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches"
   
    您應該會看到如下列程式碼片段的輸出：
   
        < HTTP/1.1 200 OK
        < Content-Type: application/json; charset=UTF-8
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Fri, 20 Nov 2015 23:47:53 GMT
        < Content-Length: 34
        <
        {"from":0,"total":0,"sessions":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    請留意到輸出的最後一行顯示為 **total:0**，這表示沒有執行中的批次。

2. 現在，我們要提交批次作業。 下列程式碼片段會使用輸入檔案 (input.txt) 傳遞 jar 名稱和類別名稱來作為參數。 如果您要從 Windows 電腦執行這些步驟，建議您採用輸出檔案這個方法。
   
        curl -k --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
   
    檔案 **input.txt** 中的參數定義如下：
   
        { "file":"wasb:///example/jars/SparkSimpleApp.jar", "className":"com.microsoft.spark.example.WasbIOTest" }
   
    您應該會看到如下列程式碼片段的輸出：
   
        < HTTP/1.1 201 Created
        < Content-Type: application/json; charset=UTF-8
        < Location: /0
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Fri, 20 Nov 2015 23:51:30 GMT
        < Content-Length: 36
        <
        {"id":0,"state":"starting","log":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    請留意到輸出的最後一行顯示為 **state:starting**。 此外也顯示 **id:0**。 在這裡，批次識別碼是 **0**。

3. 現在，您可以使用批次識別碼來擷取此批次的狀態。
   
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches/0"
   
    您應該會看到如下列程式碼片段的輸出：
   
        < HTTP/1.1 200 OK
        < Content-Type: application/json; charset=UTF-8
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Fri, 20 Nov 2015 23:54:42 GMT
        < Content-Length: 509
        <
        {"id":0,"state":"success","log":["\t diagnostics: N/A","\t ApplicationMaster host: 10.0.0.4","\t ApplicationMaster RPC port: 0","\t queue: default","\t start time: 1448063505350","\t final status: SUCCEEDED","\t tracking URL: http://hn0-myspar.lpel1gnnvxne3gwzqkfq5u5uzh.jx.internal.cloudapp.net:8088/proxy/application_1447984474852_0002/","\t user: root","15/11/20 23:52:47 INFO Utils: Shutdown hook called","15/11/20 23:52:47 INFO Utils: Deleting directory /tmp/spark-b72cd2bf-280b-4c57-8ceb-9e3e69ac7d0c"]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    輸出此時顯示 **state:success**，這表示作業已順利完成。

4. 現在，您可以視需要刪除批次。
   
        curl -k --user "admin:mypassword1!" -v -X DELETE "https://mysparkcluster.azurehdinsight.net/livy/batches/0"
   
    您應該會看到如下列程式碼片段的輸出：
   
        < HTTP/1.1 200 OK
        < Content-Type: application/json; charset=UTF-8
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Sat, 21 Nov 2015 18:51:54 GMT
        < Content-Length: 17
        <
        {"msg":"deleted"}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    輸出的最後一行顯示批次已成功刪除。 當作業執行時將它刪除，也會清除作業。 如果您刪除已完成的作業，無論成功與否，這將會完全刪除作業資訊。

## <a name="using-livy-spark-on-hdinsight-35-clusters"></a>在 HDInsight 3.5 叢集上使用 Livy Spark

根據預設，HDInsight 3.5 叢集會停用使用本機檔案路徑，以存取範本資料檔案或 jar。 建議您使用 `wasb://` 路徑，而不是從叢集存取 jar 或範本資料檔案。 如果您確定要使用本機路徑，您就必須同時更新 Ambari 組態。 若要這樣做：

1. 移至叢集的 Ambari 入口網站。 Ambari Web UI 位在您的 HDInsight 叢集的 https://**CLUSTERNAME**.azurehdidnsight.net，其中 CLUSTERNAME 是您的叢集的名稱。

2. 在左側導覽中，按一下 [Livy]，然後按一下 [設定]。

3. 在 [livy-default] 底下新增屬性名稱 `livy.file.local-dir-whitelist`，如果您想要允許存取整個檔案系統，可將其值設為 **"/"**。 如果您只想要允許存取特定目錄，請將值設為該目錄的路徑。

## <a name="submitting-livy-jobs-for-a-cluster-within-an-azure-virtual-network"></a>在 Azure 虛擬網路內提交叢集的 Livy 作業

如果您是從 Azure 虛擬網路內連線到 HDInsight Spark 叢集，可以直接連線到叢集上的 Livy。 在此案例中，Livy 端點的 URL 是 `http://<IP address of the headnode>:8998/batches`。 在這裡，**8998** 是 Livy 在叢集前端節點上執行的連接埠。 如需有關在非公用連接埠上存取服務的詳細資訊，請參閱 [HDInsight 上 Hadoop 服務所使用的連接埠](../hdinsight-hadoop-port-settings-for-services.md)。

## <a name="troubleshooting"></a>疑難排解

以下是一些使用 Livy 進行對 Spark 叢集的遠端作業提交時可能遇到的問題。

### <a name="using-an-external-jar-from-the-additional-storage-is-not-supported"></a>不支援從其他儲存體使用外部 jar

**問題︰** 如果您的 Livy Spark 作業是參考與叢集相關聯的其他儲存體帳戶之外部 jar，則作業將會失敗。

**解決方式︰** 請確定您想要使用的 jar 位於與 HDInsight 叢集相關聯的預設儲存體中。





## <a name="next-step"></a>後續步驟

* [Livy REST API 文件](http://livy.incubator.apache.org/docs/latest/rest-api.html)
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)

