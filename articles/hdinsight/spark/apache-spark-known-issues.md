---
title: 針對 Azure HDInsight 中的 Apache Spark 叢集問題進行疑難排解
description: 了解 Azure HDInsight 中的 Apache Spark 叢集相關問題，以及如何解決這些問題。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/21/2018
ms.author: jasonh
ms.openlocfilehash: 92baa28393100abe3d7694920e5ee327966db927
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048303"
---
# <a name="known-issues-for-apache-spark-cluster-on-hdinsight"></a>HDInsight 上的 Apache Spark 叢集已知問題

這份文件記錄 HDInsight Spark 公開預覽版本的所有已知問題。  

## <a name="livy-leaks-interactive-session"></a>Livy 會流失互動式工作階段
Livy 在互動式工作階段仍作用中的情況下重新啟動時 (從 Ambari 或是因為前端節點 0 虛擬機器重新開機)，互動式作業工作階段將會流失。 因此，新的作業可能會卡在「已接受」狀態中。

**緩和：**

請使用下列程序解決此問題：

1. Ssh 到前端節點。 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 執行下列命令，以尋找透過 Livy 啟動之互動式作業的應用程式識別碼。 
   
        yarn application –list
   
    如果使用 Livy 互動式工作階段啟動工作時沒有明確指定名稱，則預設作業名稱會是 Livy。 針對 Jupyter Notebook 啟動的 Livy 工作階段，作業名稱的開頭是 remotesparkmagics_*。 
3. 執行下列命令以刪除這些作業。 
   
        yarn application –kill <Application ID>

新的作業開始執行。 

## <a name="spark-history-server-not-started"></a>Spark 歷程記錄伺服器未啟動
叢集建立後，不會自動啟動 Spark 歷程記錄伺服器。  

**緩和：** 

請從 Ambari 手動啟動歷程記錄伺服器。

## <a name="permission-issue-in-spark-log-directory"></a>Spark 記錄檔目錄中的權限問題
hdiuser 在使用 spark-submit 提交作業時會發生下列錯誤：

```
java.io.FileNotFoundException: /var/log/spark/sparkdriver_hdiuser.log (Permission denied)
```
沒有任何驅動程式記錄寫入。 

**緩和：**

1. 將 hdiuser 新增至 Hadoop 群組。 
2. 在叢集建立之後，提供 /var/log/spark 的 777 權限。 
3. 使用 Ambari 將 Spark 記錄檔位置更新為具有 777 權限的目錄。  
4. 以 sudo 的身分執行 spark-submit。  

## <a name="spark-phoenix-connector-is-not-supported"></a>不支援 Spark-Phoenix 連接器

HDInsight Spark 叢集不支援 Spark-Phoenix 連接器。

**緩和：**

您必須改用 Spark-HBase 連接器。 如需指示，請參閱[如何使用 Spark-HBase 連接器](https://blogs.msdn.microsoft.com/azuredatalake/2016/07/25/hdinsight-how-to-use-spark-hbase-connector/)。

## <a name="issues-related-to-jupyter-notebooks"></a>Jupyter Notebook 的相關問題
以下是 Jupyter Notebook 的已知問題。

### <a name="notebooks-with-non-ascii-characters-in-filenames"></a>Notebook 在檔名中有非 ASCII 字元
請勿在 Jupyter Notebook 檔名中使用非 ASCII 字元。 如果您嘗試透過 Jupyter UI 上傳具有非 ASCII 檔名的檔案，則會上傳失敗，但沒有任何錯誤訊息。 Jupyter 不會讓您上傳檔案，但也不會擲回可見的錯誤。

### <a name="error-while-loading-notebooks-of-larger-sizes"></a>載入大型 Notebook 時發生錯誤
載入大型 Notebook 時，您可能會看到錯誤訊息 **`Error loading notebook`** 。  

**緩和：**

如果您收到這個錯誤，並不表示您的資料已損毀或遺失。  您的 Notebook 仍在磁碟的 `/var/lib/jupyter`中，您可以透過 SSH 連線到叢集來加以存取。 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

在您使用 SSH 連線到叢集之後，您可以從叢集中將 Notebook 複製到本機電腦 (使用 SCP 或 WinSCP) 來做為備份，以避免遺失 Notebook 中的重要資料。 您接著可以在連接埠 8001 以 SSH 通道連到前端節點，以存取 Jupyter 而不透過閘道。  您可以從該處清除 Notebook 的輸出，並將其重新儲存，以盡量縮減 Notebook 的大小。

若要防止日後再發生此錯誤，您必須遵循一些最佳作法：

* 務必讓 Notebook 保持小型規模。 會傳回到 Jupyter 的任何 Spark 作業輸出皆會保存在 Notebook 中。  一般來說，Jupyter 的最佳做法是避免在大型 RDD 或資料框架上執行 `.collect()`。相反地，如果您想要查看 RDD 的內容，請考慮執行 `.take()` 或 `.sample()`，這可讓輸出不會變得太大。
* 此外，當您儲存 Notebook 時，請清除所有輸出儲存格以減少大小。

### <a name="notebook-initial-startup-takes-longer-than-expected"></a>Notebook 的初始啟動比預期耗時
在使用 Spark magic 的 Jupyter Notebook 中，第一個程式碼陳述式可能需耗時一分鐘以上才能執行完畢。  

**說明：**

這會在執行第一個程式碼儲存格時發生。 它會在背景中起始設定工作階段組態，以及設定 SQL、Spark 和 Hive 內容。 設定這些內容後，第一個陳述式才會執行，因此會有陳述式會花很長時間完成的印象。

### <a name="jupyter-notebook-timeout-in-creating-the-session"></a>Jupyter Notebook 建立工作階段逾時
當 Spark 叢集的資源不足時，Jupyter Notebook 中的 Spark 和 PySpark 核心在嘗試建立工作階段時將會逾時。 

**緩和措施：** 

1. 藉由下列方式，釋出 Spark 叢集中的一些資源：
   
   * 移至 [關閉並停止] 功能表或按一下 Notebook 總管中的 [關閉]，以停止其他 Spark Notebook。
   * 從 YARN 停止其他 Spark 應用程式。
2. 重新啟動您先前嘗試啟動的 Notebook。 此時您應有足夠的資源可建立工作階段。

## <a name="see-also"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>建立及執行應用程式
* [使用 Scala 建立獨立應用程式](apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和擴充功能
* [Use HDInsight Tools Plugin for IntelliJ IDEA to create and submit Spark Scala applicatons (使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式)](apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)

