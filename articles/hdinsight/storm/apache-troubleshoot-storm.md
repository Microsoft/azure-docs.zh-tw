---
title: 使用 Azure HDInsight 為 Storm 進行疑難排解
description: 取得有關使用 Apache Storm 和 Azure HDInsight 的常見問題解答。
keywords: Azure HDInsight, Storm, 常見問題集, 疑難排解指南, 常見問題
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonwhowell
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 11/2/2017
ms.openlocfilehash: cad9e6ceb22ab66b5a3fe00358eb6b6170a3f29f
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049582"
---
# <a name="troubleshoot-storm-by-using-azure-hdinsight"></a>使用 Azure HDInsight 為 Storm 進行疑難排解

了解在 Apache Ambari 中使用 Apache Storm 承載時最常發生的問題及其解決方法。

## <a name="how-do-i-access-the-storm-ui-on-a-cluster"></a>如何在叢集上存取 Storm UI？
您有兩個選項可從瀏覽器存取 Storm UI：

### <a name="ambari-ui"></a>Ambari UI
1. 移至 Ambari 儀表板。
2. 在服務清單中，選取 [Storm]。
3. 在 [快速連結] 功能表中，選取 [Storm UI]。

### <a name="direct-link"></a>直接連結
您可以在下列 URL 存取 Storm UI：

https://\<叢集 DNS 名稱\>/stormui

範例：

 https://stormcluster.azurehdinsight.net/stormui

## <a name="how-do-i-transfer-storm-event-hub-spout-checkpoint-information-from-one-topology-to-another"></a>如何將 Storm 事件中樞 Spout 檢查點資訊從一個拓撲傳輸到另一個拓撲？

當您使用 HDInsight Storm 事件中樞 Spout .jar 檔案開發讀取自 Azure 事件中樞的拓撲時，您必須在新的叢集上部署具有相同名稱的拓撲。 不過，您必須在舊的叢集上保留認可至 Apache ZooKeeper 的檢查點資料。

### <a name="where-checkpoint-data-is-stored"></a>檢查點資料的儲存位置
事件中樞 Spout 會將位移的檢查點資料儲存在 ZooKeeper 的兩個根路徑中：
- 非交易式 Spout 檢查點會儲存在 /eventhubspout 中。
- 交易式 Spout 檢查點資料會儲存在 /transactional 中。

### <a name="how-to-restore"></a>還原方式
若要取得您用來從 ZooKeeper 匯出資料，再以新名稱將資料匯回 ZooKeeper 的指令碼和程式庫，請參閱 [HDInsight Storm 範例](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/tools/zkdatatool-1.0)。

lib 資料夾的 .jar 檔案包含匯出/匯入作業的實作。 bash 資料夾的範例指令碼，示範如何從舊叢集的 ZooKeeper 伺服器匯出資料，再將它匯回新叢集的 ZooKeeper 伺服器。

從 ZooKeeper 節點執行 [stormmeta.sh](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/tools/zkdatatool-1.0/bash/stormmeta.sh) 指令碼以匯出再匯入資料。 將指令碼更新為正確的 Hortonworks Data Platform (HDP) 版本。 (我們正致力於讓這些指令碼在 HDInsight 中成為泛型。 泛型指令碼可以從叢集的任何節點執行，不需要使用者修改。)

匯出命令會將中繼資料寫入您設定位置的 Apache Hadoop 分散式檔案系統 (HDFS) 路徑 (Azure Blob 儲存體或 Azure Data Lake Store 存放區中)。

### <a name="examples"></a>範例

#### <a name="export-offset-metadata"></a>匯出位移中繼資料
1. 使用 SSH 移至叢集的 ZooKeeper 叢集，該舊叢集需要匯出檢查點位移。
2. 執行下列命令 (在您更新 HDP 版本字串之後)，將 ZooKeeper 位移資料匯出至 /stormmetadta/zkdata HDFS 路徑：

    ```apache   
    java -cp ./*:/etc/hadoop/conf/*:/usr/hdp/2.5.1.0-56/hadoop/*:/usr/hdp/2.5.1.0-56/hadoop/lib/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/lib/*:/etc/failover-controller/conf/*:/etc/hadoop/* com.microsoft.storm.zkdatatool.ZkdataImporter export /eventhubspout /stormmetadata/zkdata
    ```

#### <a name="import-offset-metadata"></a>匯入位移中繼資料
1. 使用 SSH 移至叢集的 ZooKeeper 叢集，該舊叢集需要匯入檢查點位移。
2. 執行下列命令 (在您更新 HDP 版本字串之後)，將 ZooKeeper 位移資料從 HDFS 路徑 /stormmetadata/zkdata 匯入至目標叢集的 ZooKeeper 伺服器：

    ```apache
    java -cp ./*:/etc/hadoop/conf/*:/usr/hdp/2.5.1.0-56/hadoop/*:/usr/hdp/2.5.1.0-56/hadoop/lib/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/lib/*:/etc/failover-controller/conf/*:/etc/hadoop/* com.microsoft.storm.zkdatatool.ZkdataImporter import /eventhubspout /home/sshadmin/zkdata
    ```
   
#### <a name="delete-offset-metadata-so-that-topologies-can-start-processing-data-from-the-beginning-or-from-a-timestamp-that-the-user-chooses"></a>刪除位移中繼資料，以便拓撲可以從頭開始處理資料，或從使用者選擇的時間戳記開始處理
1. 使用 SSH 移至叢集的 ZooKeeper 叢集，該舊叢集需要刪除檢查點位移。
2. 執行下列命令 (在您更新 HDP 版本字串之後)，刪除目前叢集中的所有 ZooKeeper 位移資料：

    ```apache
    java -cp ./*:/etc/hadoop/conf/*:/usr/hdp/2.5.1.0-56/hadoop/*:/usr/hdp/2.5.1.0-56/hadoop/lib/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/lib/*:/etc/failover-controller/conf/*:/etc/hadoop/* com.microsoft.storm.zkdatatool.ZkdataImporter delete /eventhubspout
    ```

## <a name="how-do-i-locate-storm-binaries-on-a-cluster"></a>如何找出叢集上的 Storm 二進位檔？
目前 HDP 堆疊的 Storm 二進位檔位於 /usr/hdp/current/storm-client 中。 前端節點和背景工作節點使用相同的位置。
 
/usr/hdp 中可能會有特定 HDP 版本的多個二進位檔 (例如 /usr/hdp/2.5.0.1233/storm)。 /usr/hdp/current/storm-client 資料夾是以符號連結至叢集上執行的最新版本。

如需詳細資訊，請參閱[使用 SSH 連線到 HDInsight 叢集](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix)和 [Storm](http://storm.apache.org/)。
 
## <a name="how-do-i-determine-the-deployment-topology-of-a-storm-cluster"></a>如何判斷 Storm 叢集的部署拓撲？
首先，找出所有與 HDInsight Storm 一起安裝的元件。 Storm 叢集包含四個節點類別：

* 閘道節點
* 前端節點
* ZooKeeper 節點
* 背景工作節點
 
### <a name="gateway-nodes"></a>閘道節點
閘道節點是閘道與反向 Proxy 服務，可讓您啟用使用中的 Ambari 管理服務公用存取。 它也可以處理 Ambari 前置選擇。
 
### <a name="head-nodes"></a>前端節點
Storm 前端節點執行下列服務：
* Nimbus
* Ambari 伺服器
* Ambari 計量伺服器
* Ambari 計量收集器
 
### <a name="zookeeper-nodes"></a>ZooKeeper 節點
HDInsight 隨附一個三節點的 ZooKeeper 仲裁。 仲裁大小是固定的，且無法重新設定。
 
叢集中的 Storm 服務已設定為自動使用 ZooKeeper 仲裁。
 
### <a name="worker-nodes"></a>背景工作節點
Storm 背景工作節點執行下列服務：
* 監督員
* 背景工作 Java 虛擬機器 (JVM)，用於執行拓樸
* Ambari 代理程式
 
## <a name="how-do-i-locate-storm-event-hub-spout-binaries-for-development"></a>如何找出用於開發的 Storm 事件中樞 Spout 二進位檔？
 
如需對您的拓撲使用 Storm 事件中樞 Spout .jar 檔案的詳細資訊，請參閱下列資源。
 
### <a name="java-based-topology"></a>Java 型拓撲
[使用 Storm on HDInsight 處理 Azure 事件中樞的事件 (Java)](https://docs.microsoft.com/azure/hdinsight/hdinsight-storm-develop-java-event-hub-topology)
 
### <a name="c-based-topology-mono-on-hdinsight-34-linux-storm-clusters"></a>C# 型拓撲 (HDInsight 3.4+ Linux Storm 叢集上的 Mono)
[利用 Storm on HDInsight 處理 Azure 事件中樞的事件 (C#)](https://docs.microsoft.com/azure/hdinsight/hdinsight-storm-develop-csharp-event-hub-topology)
 
### <a name="latest-storm-event-hub-spout-binaries-for-hdinsight-35-linux-storm-clusters"></a>適用於 HDInsight 3.5+ Linux Storm 叢集的最新 Storm 事件中樞 Spout 二進位檔
若要了解如何使用適用於 HDInsight 3.5+ Linux Storm 叢集的最新 Storm 事件中樞 Spout，請參閱 mvn-repo [readme 檔案](https://github.com/hdinsight/mvn-repo/blob/master/README.md)。
 
### <a name="source-code-examples"></a>原始程式碼範例
請參閱[範例](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub)，以了解如何使用 Azure HDInsight 叢集上的 Apache Storm 拓撲 (以 Java 撰寫)，來從 Azure 事件中樞進行讀取和寫入。
 
## <a name="how-do-i-locate-storm-log4j-configuration-files-on-clusters"></a>如何找出叢集上的 Storm Log4J 組態檔？
 
識別 Storm 服務的 Apache Log4J 設定檔。
 
### <a name="on-head-nodes"></a>在前端節點上
從 /usr/hdp/\<HDP version\>/storm/log4j2/cluster.xml 可讀取 Nimbus Log4J 設定。
 
### <a name="on-worker-nodes"></a>在背景工作節點上
從 /usr/hdp/\<HDP version\>/storm/log4j2/cluster.xml 可讀取監督員 Log4J 設定。
 
從 /usr/hdp/\<HDP version\>/storm/log4j2/worker.xml 可讀取背景工作角色 Log4J 設定。
 
範例：/usr/hdp/2.6.0.2-76/storm/log4j2/cluster.xml /usr/hdp/2.6.0.2-76/storm/log4j2/worker.xml

### <a name="see-also"></a>另請參閱
[使用 Azure HDInsight 進行疑難排解](../../hdinsight/hdinsight-troubleshoot-guide.md)
