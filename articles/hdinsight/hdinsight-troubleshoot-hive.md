---
title: 使用 Azure HDInsight 為 Hive 進行疑難排解
description: 取得有關使用 Apache Hive 和 Azure HDInsight 的常見問題解答。
keywords: Azure HDInsight, Hive, 常見問題集, 疑難排解指南, 常見問題
services: hdinsight
ms.service: hdinsight
author: dharmeshkakadia
ms.author: dharmeshkakadia
ms.topic: conceptual
ms.date: 11/2/2017
ms.openlocfilehash: 832fab6c4f183ddad512c5e6e4309d70938a316b
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39600018"
---
# <a name="troubleshoot-hive-by-using-azure-hdinsight"></a>使用 Azure HDInsight 為 Hive 進行疑難排解

了解在 Apache Ambari 中使用 Apache Hive 承載時最常發生的問題及其解決方法。


## <a name="how-do-i-export-a-hive-metastore-and-import-it-on-another-cluster"></a>如何匯出 Hive 中繼存放區並匯入另一個叢集？


### <a name="resolution-steps"></a>解決步驟

1. 使用安全殼層 (SSH) 用戶端連線到 HDInsight 叢集。 如需詳細資訊，請參閱[其他閱讀資料](#additional-reading-end)。

2. 在您要匯出中繼存放區的 HDInsight 叢集上執行下列命令：

    ```apache
    for d in `hive -e "show databases"`; do echo "create database $d; use $d;" >> alltables.sql ; for t in `hive --database $d -e "show tables"` ; do ddl=`hive --database $d -e "show create table $t"`; echo "$ddl ;" >> alltables.sql ; echo "$ddl" | grep -q "PARTITIONED\s*BY" && echo "MSCK REPAIR TABLE $t ;" >> alltables.sql ; done; done
    ```

  此命令會產生名為 allatables.sql 的檔案。

3. 將 alltables.sql 檔案複製到 HDInsight 叢集，然後執行下列命令：

  ```apache
  hive -f alltables.sql
  ```

解決步驟中的程式碼會假設新叢集上的資料路徑與舊叢集上的資料路徑相同。 如果資料路徑不同，您可以手動編輯產生的 alltables.sql 檔案以反映任何變更。

### <a name="additional-reading"></a>其他閱讀資料

- [使用 SSH 連線到 HDInsight 叢集](hdinsight-hadoop-linux-use-ssh-unix.md)


## <a name="how-do-i-locate-hive-logs-on-a-cluster"></a>如何在叢集上尋找 Hive 記錄？

### <a name="resolution-steps"></a>解決步驟

1. 使用 SSH 連線到 HDInsight 叢集。 如需詳細資訊，請參閱**其他閱讀資料**。

2. 若要檢視 Hive 用戶端記錄，請使用下列命令：

  ```apache
  /tmp/<username>/hive.log 
  ```

3. 若要檢視 Hive 中繼存放區記錄，請使用下列命令：

  ```apache
  /var/log/hive/hivemetastore.log 
  ```

4. 若要檢視 Hive 伺服器記錄，請使用下列命令：

  ```apache
  /var/log/hive/hiveserver2.log 
  ```

### <a name="additional-reading"></a>其他閱讀資料

- [使用 SSH 連線到 HDInsight 叢集](hdinsight-hadoop-linux-use-ssh-unix.md)


## <a name="how-do-i-launch-the-hive-shell-with-specific-configurations-on-a-cluster"></a>如何在叢集上以特定設定啟動 Hive Shell？

### <a name="resolution-steps"></a>解決步驟

1. 當您啟動 Hive Shell 時，請指定設定機碼值組。 如需詳細資訊，請參閱[其他閱讀資料](#additional-reading-end)。

  ```apache
  hive -hiveconf a=b 
  ```

2. 若要列出 Hive Shell 上所有有效的設定，使用下列命令：

  ```apache
  hive> set;
  ```

  例如，使用下列命令，以啟動 Hive Shell 並在主控台上啟用偵錯記錄：

  ```apache
  hive -hiveconf hive.root.logger=ALL,console 
  ```

### <a name="additional-reading"></a>其他閱讀資料

- [Hive 設定屬性](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties)


## <a name="how-do-i-analyze-tez-dag-data-on-a-cluster-critical-path"></a>如何分析叢集關鍵路徑上的 Tez DAG 資料？


### <a name="resolution-steps"></a>解決步驟
 
1. 若要分析叢集關鍵圖表上的 Apache Tez 有向非循環圖 (DAG)，請使用 SSH 連線到 HDInsight 叢集。 如需詳細資訊，請參閱[其他閱讀資料](#additional-reading-end)。

2. 在命令提示字元中，執行下列命令：
   
  ```apache
  hadoop jar /usr/hdp/current/tez-client/tez-job-analyzer-*.jar CriticalPath --saveResults --dagId <DagId> --eventFileName <DagData.zip> 
  ```

3. 若要列出可用來分析 Tez DAG 的其他分析器，請使用下列命令：

  ```apache
  hadoop jar /usr/hdp/current/tez-client/tez-job-analyzer-*.jar
  ```

  您必須提供範例程式作為第一個引數。

  有效的程式名稱包括：
    - **ContainerReuseAnalyzer**：列印 DAG 中的容器重複使用詳細資料
    - **CriticalPath**：尋找 DAG 的關鍵路徑
    - **LocalityAnalyzer**：列印 DAG 中的位置詳細資料
    - **ShuffleTimeAnalyzer**：分析 DAG 中的隨機播放時間詳細資料 
    - **SkewAnalyzer**：分析 DAG 中的扭曲詳細資料
    - **SlowNodeAnalyzer**：列印 DAG 中的節點詳細資料
    - **SlowTaskIdentifier**：列印 DAG 中的低速工作詳細資料
    - **SlowestVertexAnalyzer**：列印 DAG 中最慢頂點詳細資料
    - **SpillAnalyzer**：列印 DAG 中的溢出詳細資料
    - **TaskConcurrencyAnalyzer**：列印 DAG 中的工作並行詳細資料
    - **VertexLevelCriticalPathAnalyzer**：在 DAG 的頂點層級中尋找關鍵路徑


### <a name="additional-reading"></a>其他閱讀資料

- [使用 SSH 連線到 HDInsight 叢集](hdinsight-hadoop-linux-use-ssh-unix.md)


## <a name="how-do-i-download-tez-dag-data-from-a-cluster"></a>如何從叢集下載 Tez DAG 資料？


#### <a name="resolution-steps"></a>解決步驟

有兩種方式可以收集 Tez DAG 資料：

- 從命令列：
 
    使用 SSH 連線到 HDInsight 叢集。 在命令提示字元中，執行下列命令：

  ```apache
  hadoop jar /usr/hdp/current/tez-client/tez-history-parser-*.jar org.apache.tez.history.ATSImportTool -downloadDir . -dagId <DagId> 
  ```

- 使用 Ambari Tez 檢視：
   
  1. 移至 Ambari。 
  2. 移至 Tez 檢視 (在右上角的磚圖示下)。 
  3. 選取您要檢視的 DAG。
  4. 選取 [下載資料]。

### <a name="additional-reading-end"></a>其他閱讀資料

[使用 SSH 連線到 HDInsight 叢集](hdinsight-hadoop-linux-use-ssh-unix.md)


### <a name="see-also"></a>另請參閱
[使用 Azure HDInsight 進行疑難排解](hdinsight-troubleshoot-guide.md)




