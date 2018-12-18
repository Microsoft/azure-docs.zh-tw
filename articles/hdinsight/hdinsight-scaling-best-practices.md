---
title: 調整叢集大小 - Azure HDInsight
description: 調整 HDInsight 叢集以因應工作負載。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/02/2018
ms.author: ashish
ms.openlocfilehash: 93eb6fb0da86909dfc880db2a9bb2331abe4418a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948112"
---
# <a name="scale-hdinsight-clusters"></a>調整 HDInsight 叢集

HDInsight 具有彈性，可讓您選擇相應增加和相應減少叢集中的背景工作節點數。 此彈性能夠讓您在下班後或是週末時縮小叢集，並於業務需求達到高峰的期間擴大叢集。

例如，如果您有一些發生頻率為一天一次或一個月一次的批次處理，HDInsight 叢集可以在該排程事件開始前幾分鐘相應增加，以便提供足夠的記憶體和 CPU 計算能力。 您可以使用 PowerShell Cmdlet [`Set–AzureRmHDInsightClusterSize`](hdinsight-administer-use-powershell.md#scale-clusters) 來自動進行調整。  之後，當處理完成且使用量再次降低時，您可以將 HDInsight 叢集相應減少為較少的背景工作節點。

* 若要透過 [PowerShell](hdinsight-administer-use-powershell.md) 調整叢集：

    ```powershell
    Set-AzureRmHDInsightClusterSize -ClusterName <Cluster Name> -TargetInstanceCount <NewSize>
    ```
    
* 若要透過 [Azure 傳統 CLI](hdinsight-administer-use-command-line.md) 調整叢集：

    ```
    azure hdinsight cluster resize [options] <clusterName> <Target Instance Count>
    ```

[!INCLUDE [classic-cli-warning](../../includes/requires-classic-cli.md)]
    
* 若要透過 [Azure 入口網站](https://portal.azure.com)調整叢集，請開啟 [HDInsight 叢集] 窗格，選取左側功能表中的 [調整叢集]，然後在 [調整叢集] 窗格中輸入背景工作節點數，再選取 [儲存]。

    ![調整叢集](./media/hdinsight-scaling-best-practices/scale-cluster-blade.png)

只要使用上述任一方法，您即可在幾分鐘內相應增加或相應減少 HDInsight 叢集。

## <a name="scaling-impacts-on-running-jobs"></a>調整對於執行中作業的影響

當您**新增**節點至執行中的 HDInsight 叢集時，不論是擱置還是執行中的作業都不會受到影響。 此外，在調整程序執行的同時，您仍可放心地提交新的作業。 如果調整作業因為任何原因而失敗，系統會以適當方式處理失敗，而讓叢集保持正常運作狀態。

不過，如果您是透過**移除**節點的方式來相應減少叢集，當調整作業完成時，所有擱置或執行中的作業都會失敗。 此失敗的發生原因是程序進行期間會重新啟動某些服務。

若要解決此問題，您可以等候作業完成再相應減少叢集、手動終止作業，或在調整作業結束後重新提交作業。

若要查看擱置和執行中作業的清單，您可以使用 YARN ResourceManager UI，並遵循下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側功能表中，依序選取 [瀏覽]、[HDInsight 叢集] 和您的叢集。
3. 從 [HDInsight 叢集] 窗格中，選取頂端功能表中的 [儀表板] 以開啟 Ambari UI。 輸入叢集登入認證。
4. 在左側功能表中的服務清單上按一下 [YARN]。 在 [YARN] 頁面上選取 [快速連結]、將滑鼠停留在作用中的前端節點上，然後按一下 [ResourceManager UI]。

    ![ResourceManager UI](./media/hdinsight-scaling-best-practices/resourcemanager-ui.png)

您可以使用 `https://<HDInsightClusterName>.azurehdinsight.net/yarnui/hn/cluster` 直接存取 ResourceManager UI。

您會看到作業及其目前狀態的清單。 螢幕擷取畫面中有一項目前正在執行的作業：

![ResourceManager UI 應用程式](./media/hdinsight-scaling-best-practices/resourcemanager-ui-applications.png)

若要手動終止該執行中的應用程式，請從 SSH 殼層執行下列命令：

```bash
yarn application -kill <application_id>
```

例如︰

```bash
yarn application -kill "application_1499348398273_0003"
```

## <a name="rebalancing-an-hbase-cluster"></a>重新平衡 HBase 叢集

區域伺服器會在完成調整作業的數分鐘之內自動取得平衡。 若要手動平衡區域伺服器，請使用下列步驟：

1. 使用 SSH 連線到 HDInsight 叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 啟動 HBase 殼層：

        hbase shell

3. 使用下列命令手動平衡區域伺服器：

        balancer

## <a name="scale-down-implications"></a>相應減少影響

如先前所述，當相應減少作業完成時，所有擱置或執行中的作業都會終止。 不過，相應減少作業可能還會發生其他潛在的影響。

## <a name="hdinsight-name-node-stays-in-safe-mode-after-scaling-down"></a>進行相應減少後 HDInsight 名稱節點仍會處於安全模式

![調整叢集](./media/hdinsight-scaling-best-practices/scale-cluster.png)

如果您如上圖所示將叢集縮小為最小值 (也就是只有一個背景工作節點)，當背景工作節點因為進行修補而重新啟動時，HDFS 可能會無法脫離安全模式，而在調整作業完成當下也會發生此情形。

此問題的主要原因是 Hive 會使用一些 `scratchdir` 檔案，而且根據預設，每個區塊應該都會有三個複本，但如果您相應減少為最小值 (也就是只有一個背景工作節點)，則只可能有一個複本。 因此，`scratchdir` 中的檔案會變成「複寫不足」的狀態。 這可能會導致服務在調整作業完成後進行重新啟動時，HDFS 仍處於安全模式。

當您嘗試進行相應減少時，HDInsight 會依靠 Ambari 管理介面先將不要的多餘背景工作節點解除委任 (此動作會將這些節點的 HDFS 區塊複寫至其他線上背景工作節點)，然後才放心地將叢集相應減少。 HDFS 在維護期間會進入安全模式，調整完成時則應該會退出該模式。 HDFS 正是可能會在此時無法脫離安全模式。

HDFS 的 `dfs.replication` 設定會設定為 3。 因此，每當線上的背景工作節點少於三個時，暫存檔案的區塊就會有複寫不足的情形，原因是每個檔案區塊可用的複本數並非如預期般地有三個。

您可以執行命令讓 HDFS 脫離安全模式。 例如，如果您知道安全模式會開啟的唯一理由是暫存檔案有複寫不足的情形，您就可以放心地離開安全模式。 這是因為複寫不足的檔案屬於 Hive 暫存檔案。

```bash
hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -safemode leave
```

離開安全模式後，您可以手動移除暫存檔案，也可以等候 Hive 最終自動將其清除。

### <a name="example-errors-when-safe-mode-is-turned-on"></a>安全模式開啟時的錯誤範例

* H070 無法開啟 Hive 工作階段。 org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.RetriableException): org.apache.hadoop.hdfs.server.namenode.SafeModeException: **無法建立目錄** /tmp/hive/hive/819c215c-6d87-4311-97c8-4f0b9d2adcf0。 **名稱節點處於安全模式**。 所報告的區塊 75 個還需要 12 個區塊才能觸達閾值 0.9900 (總計 87 個區塊)。 即時資料節點數目 10 個已觸達最小值 0 個。 一旦觸達閾值，安全模式就會自動關閉。

* H100 無法提交陳述式顯示資料庫: org.apache.thrift.transport.TTransportException: org.apache.http.conn.HttpHostConnectException: 連線至 hn0-clustername.servername.internal.cloudapp.net:10001 [hn0-clustername.servername。 internal.cloudapp.net/1.1.1.1] 失敗: **拒絕連線**

* H020 無法連線至 hn0-hdisrv.servername.bx.internal.cloudapp.net:10001: org.apache.thrift.transport.TTransportException: 無法以 http 連線至 http://hn0-hdisrv.servername.bx.internal.cloudapp.net:10001/。 org.apache.http.conn.HttpHostConnectException: 連線至 hn0-hdisrv.servername.bx.internal.cloudapp.net:10001 [hn0-hdisrv.servername.bx.internal.cloudapp.net/10.0.0.28] 失敗: 拒絕連線: org.apache.thrift.transport.TTransportException: 無法以 http 連線至 http://hn0-hdisrv.servername.bx.internal.cloudapp.net:10001/。 org.apache.http.conn.HttpHostConnectException: 連線至 hn0-hdisrv.servername.bx.internal.cloudapp.net:10001 [hn0-hdisrv.servername.bx.internal.cloudapp.net/10.0.0.28] 失敗: **拒絕連線**

* 來自 Hive 記錄: WARN [main]: server.HiveServer2 (HiveServer2.java:startHiveServer2(442)) – 在第 21 次嘗試啟動 HiveServer2 時發生錯誤，將在 60 秒後重試 java.lang.RuntimeException: 對 hive 組態套用授權原則時發生錯誤: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.RetriableException): org.apache.hadoop.hdfs.server.namenode.SafeModeException: **無法建立目錄** /tmp/hive/hive/70a42b8a-9437-466e-acbe-da90b1614374。 **名稱節點處於安全模式**。
    所報告的區塊 0 個還需要 9 個區塊才能觸達閾值 0.9900 (總計 9 個區塊)。
    即時資料節點數目 10 個已觸達最小值 0 個。 **一旦觸達閾值，安全模式就會自動關閉**。
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1324)

您可以從 `/var/log/hadoop/hdfs/` 資料夾檢閱接近叢集調整時的名稱節點記錄，以查看其進入安全模式的時間。 記錄檔名稱為 `Hadoop-hdfs-namenode-hn0-clustername.*`。

先前錯誤的根本原因是 Hive 在執行查詢時會仰賴 HDFS 中的暫存檔案。 當 HDFS 進入安全模式時，Hive 因為無法寫入到 HDFS 而無法執行查詢。 HDFS 中的暫存檔案位於掛接到個別背景工作節點 VM 的本機磁碟機，並且會在其他背景工作節點之間複寫到至少三個複本。

Hive 中的 `hive.exec.scratchdir` 參數會設定於 `/etc/hive/conf/hive-site.xml` 內：

```xml
<property>
    <name>hive.exec.scratchdir</name>
    <value>hdfs://mycluster/tmp/hive</value>
</property>
```

### <a name="view-the-health-and-state-of-your-hdfs-file-system"></a>檢視 HDFS 檔案系統的健康情況和狀態

您可以檢視每個名稱節點的狀態報告以查看節點是否處於安全模式。 若要檢視報告，請以 SSH 連線到每個前端節點，並執行下列命令：

```
hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -safemode get
```

![安全模式關閉](./media/hdinsight-scaling-best-practices/safe-mode-off.png)

> [!NOTE]
> `-D` 是必要參數，原因是 HDInsight 中的預設檔案系統是 Azure 儲存體或 Azure Data Lake Store。 `-D` 會指定讓命令對本機 HDFS 檔案系統來執行。

接下來，您可以檢視報告，報告中會顯示 HDFS 狀態的詳細資料：

```
hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -report
```

此命令會在狀況良好的叢集上產生下列結果，其中此叢集上的所有區塊都會複寫至預期的程度：

![安全模式關閉](./media/hdinsight-scaling-best-practices/report.png)

HDFS 支援使用 `fsck` 命令來檢查各種檔案的不一致，例如檔案中的遺漏區塊或複寫不足的區塊。 若要對 `scratchdir` (暫存磁碟) 檔案執行 `fsck` 命令：

```
hdfs fsck -D 'fs.default.name=hdfs://mycluster/' /tmp/hive/hive
```

在沒有複寫不足區塊的狀況良好 HDFS 檔案系統上執行時，您會看到類似下面的輸出：

```
Connecting to namenode via http://hn0-scalin.name.bx.internal.cloudapp.net:30070/fsck?ugi=sshuser&path=%2Ftmp%2Fhive%2Fhive
FSCK started by sshuser (auth:SIMPLE) from /10.0.0.21 for path /tmp/hive/hive at Thu Jul 06 20:07:01 UTC 2017
..Status: HEALTHY
 Total size:    53 B
 Total dirs:    5
 Total files:   2
 Total symlinks:                0 (Files currently being written: 2)
 Total blocks (validated):      2 (avg. block size 26 B)
 Minimally replicated blocks:   2 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       0 (0.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     3.0
 Corrupt blocks:                0
 Missing replicas:              0 (0.0 %)
 Number of data-nodes:          4
 Number of racks:               1
FSCK ended at Thu Jul 06 20:07:01 UTC 2017 in 3 milliseconds


The filesystem under path '/tmp/hive/hive' is HEALTHY
```

相反地，在具有某些複寫不足區塊的 HDFS 檔案系統上執行 `fsck` 命令時，輸出會類似下面：

```
Connecting to namenode via http://hn0-scalin.name.bx.internal.cloudapp.net:30070/fsck?ugi=sshuser&path=%2Ftmp%2Fhive%2Fhive
FSCK started by sshuser (auth:SIMPLE) from /10.0.0.21 for path /tmp/hive/hive at Thu Jul 06 20:13:58 UTC 2017
.
/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/inuse.info:  Under replicated BP-1867508080-10.0.0.21-1499348422953:blk_1073741826_1002. Target Replicas is 3 but found 1 live replica(s), 0 decommissioned replica(s) and 0 decommissioning replica(s).
.
/tmp/hive/hive/e7c03964-ff3a-4ee1-aa3c-90637a1f4591/inuse.info: CORRUPT blockpool BP-1867508080-10.0.0.21-1499348422953 block blk_1073741825

/tmp/hive/hive/e7c03964-ff3a-4ee1-aa3c-90637a1f4591/inuse.info: MISSING 1 blocks of total size 26 B.Status: CORRUPT
 Total size:    53 B
 Total dirs:    5
 Total files:   2
 Total symlinks:                0 (Files currently being written: 2)
 Total blocks (validated):      2 (avg. block size 26 B)
  ********************************
  UNDER MIN REPL'D BLOCKS:      1 (50.0 %)
  dfs.namenode.replication.min: 1
  CORRUPT FILES:        1
  MISSING BLOCKS:       1
  MISSING SIZE:         26 B
  CORRUPT BLOCKS:       1
  ********************************
 Minimally replicated blocks:   1 (50.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       1 (50.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     0.5
 Corrupt blocks:                1
 Missing replicas:              2 (33.333332 %)
 Number of data-nodes:          1
 Number of racks:               1
FSCK ended at Thu Jul 06 20:13:58 UTC 2017 in 28 milliseconds


The filesystem under path '/tmp/hive/hive' is CORRUPT
```

您也可以選取左側的 **HDFS** 服務或使用 `https://<HDInsightClusterName>.azurehdinsight.net/#/main/services/HDFS/summary`，來檢視 Ambari UI 中的 HDFS 狀態。

![Ambari HDFS 狀態](./media/hdinsight-scaling-best-practices/ambari-hdfs.png)

您也可能會在作用中或待命 NameNode 上看到一或多個嚴重錯誤。 若要檢視 NameNode 區塊健康情況，請選取警示旁的 NameNode 連結。

![NameNode 區塊健康情況](./media/hdinsight-scaling-best-practices/ambari-hdfs-crit.png)

若要清除暫存檔案，從而移除區塊複寫錯誤，請以 SSH 連線到 每個前端節點，然後執行下列命令：

```
hadoop fs -rm -r -skipTrash hdfs://mycluster/tmp/hive/
```

> [!NOTE]
> 如果仍有某些執行中的作業，此命令可能會中斷 Hive。

### <a name="how-to-prevent-hdinsight-from-getting-stuck-in-safe-mode-due-to-under-replicated-blocks"></a>如何防止 HDInsight 因為複寫不足的區塊而無法脫離安全模式

有數種方式可避免讓 HDInsight 無法脫離安全模式：

* 先停止所有 Hive 作業，再相應減少 HDInsight。 或者，對相應減少程序進行排程，以免與執行中的 Hive 作業發生衝突。
* 先手動清除 Hive 在 HDFS 中的暫存 `tmp` 目錄檔案，再相應減少。
* 只將 HDInsight 相應減少為三個背景工作節點 (最小值)。 避免減少到只剩一個背景工作節點。
* 如有需要，則執行命令來脫離安全模式。

下列幾節會說明這些選項。

#### <a name="stop-all-hive-jobs"></a>停止所有 Hive 作業

先停止所有 Hive 作業，再相應減少為一個背景工作節點。 如果您已將工作負載排程，則請在 Hive 工作完成後執行相應減少。

這有助於將 tmp 資料夾中的暫存檔案 (如果有的話) 數量減到最少。

#### <a name="manually-clean-up-hives-scratch-files"></a>手動清除 Hive 的暫存檔案

如果 Hive 有遺留暫存檔案，則可以先手動清除那些檔案再相應減少，以避開安全模式。

1. 停止 Hive 服務，並確定所有查詢和作業皆已完成。

2. 列出 `hdfs://mycluster/tmp/hive/` 目錄的內容，以查看它是否包含任何檔案：

    ```
    hadoop fs -ls -R hdfs://mycluster/tmp/hive/hive
    ```
    
    以下是有檔案存在時的輸出範例：

    ```
    sshuser@hn0-scalin:~$ hadoop fs -ls -R hdfs://mycluster/tmp/hive/hive
    drwx------   - hive hdfs          0 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c
    drwx------   - hive hdfs          0 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/_tmp_space.db
    -rw-r--r--   3 hive hdfs         27 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/inuse.info
    -rw-r--r--   3 hive hdfs          0 2017-07-06 13:40 hdfs://mycluster/tmp/hive/hive/4f3f4253-e6d0-42ac-88bc-90f0ea03602c/inuse.lck
    drwx------   - hive hdfs          0 2017-07-06 20:30 hdfs://mycluster/tmp/hive/hive/c108f1c2-453e-400f-ac3e-e3a9b0d22699
    -rw-r--r--   3 hive hdfs         26 2017-07-06 20:30 hdfs://mycluster/tmp/hive/hive/c108f1c2-453e-400f-ac3e-e3a9b0d22699/inuse.info
    ```

3. 如果您知道 Hive 不會再使用這些檔案，則可將其移除。 請查看 Yarn ResourceManager UI 頁面，確定 Hive 沒有任何執行中的查詢。

    從 HDFS 中移除檔案的命令列範例：

    ```
    hadoop fs -rm -r -skipTrash hdfs://mycluster/tmp/hive/
    ```
    
#### <a name="scale--hdinsight-to-three-worker-nodes"></a>將 HDInsight 調整為三個背景工作節點

如果一直有無法脫離安全模式的問題，而且不想選擇使用前面的步驟，則建議您只相應減少為三個背景工作節點，以免發生這個問題。 相較於相應減少為一個節點，這可能不是最好的做法，原因是會有成本限制。 不過，如果只有一個背景工作節點，HDFS 將無法保證叢集會有三個資料複本可以使用。

#### <a name="run-the-command-to-leave-safe-mode"></a>執行命令來脫離安全模式

最後一個選項則是留意 HDFS 是否有進入安全模式的罕見情況，若有就執行脫離安全模式的命令。 一旦您確定 HDFS 之所以進入安全模式是因為 Hive 檔案複寫不足，就執行下列命令來脫離安全模式：

* Linux 上的 HDInsight：

    ```bash
    hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -safemode leave
    ```
    
* Windows 上的 HDInsight：

    ```bash
    hdfs dfsadmin -fs hdfs://headnodehost:9000 -safemode leave
    ```
    
## <a name="next-steps"></a>後續步驟

* [Azure HDInsight 簡介](hadoop/apache-hadoop-introduction.md)
* [調整叢集](hdinsight-administer-use-portal-linux.md#scale-clusters)
* [使用 Ambari Web UI 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)
