---
title: 使用 Azure HDInsight 為 HBase 進行疑難排解
description: 取得有關使用 HBase 和 Azure HDInsight 的常見問題解答。
services: hdinsight
ms.service: hdinsight
author: nitinver
ms.author: nitinver
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 7/7/2017
ms.openlocfilehash: e25a2dcaf9b7c820f5d7e0312fb2cb55fc558882
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39593894"
---
# <a name="troubleshoot-hbase-by-using-azure-hdinsight"></a>使用 Azure HDInsight 為 HBase 進行疑難排解

了解在 Apache Ambari 中使用 Apache HBase 承載時最常發生的問題及其解決方法。

## <a name="how-do-i-run-hbck-command-reports-with-multiple-unassigned-regions"></a>如何執行回報多個未指派區域的 hbck 命令？

當您執行 `hbase hbck` 命令時，可能會看到一個常見的錯誤訊息：「多個區域未獲指派或區域鏈結中出現漏洞」。

在 HBase Master UI 中，您可以看到所有區域伺服器中不對稱的區域數目。 然後，您可以執行 `hbase hbck` 命令來查看區域鏈結中的漏洞。

漏洞可能是由於離線區域所造成，因此請先修正指派。 

若要讓未指派的區域回復正常狀態，請完成下列步驟：

1. 使用 SSH 登入 HDInsight HBase 叢集。
2. 若要使用 ZooKeeper Shell 進行連線，請執行 `hbase zkcli` 命令。
3. 執行 `rmr /hbase/regions-in-transition` 命令或 `rmr /hbase-unsecure/regions-in-transition` 命令。
4. 若要結束 `hbase zkcli` Shell，請使用 `exit` 命令。
5. 開啟 Apache Ambari UI，然後重新啟動 Active HBase Master 服務。
6. 再次執行 `hbase hbck` 命令 (不含任何選項)。 檢查此命令的輸出，確定所有區域已獲指派。


## <a name="how-do-i-fix-timeout-issues-with-hbck-commands-for-region-assignments"></a>如何修正使用 hbck 命令進行區域指派時發生的逾時問題？

### <a name="issue"></a>問題

使用 `hbck` 命令時所發生的逾時問題，可能是由於數個區域長時間處於「轉換中」狀態所造成。 您可以在 HBase Master UI 中看到這些區域處於離線狀態。 由於有大量的區域嘗試進行轉換，HBase Master 可能會逾時，因而無法讓這些區域回復連線。

### <a name="resolution-steps"></a>解決步驟

1. 使用 SSH 登入 HDInsight HBase 叢集。
2. 若要使用 ZooKeeper Shell 進行連線，請執行 `hbase zkcli` 命令。
3. 執行 `rmr /hbase/regions-in-transition` 或 `rmr /hbase-unsecure/regions-in-transition` 命令。
4. 若要結束 `hbase zkcli` Shell，請使用 `exit` 命令。
5. 在 Ambari UI 中，重新啟動 Active HBase Master 服務。
6. 再次執行 `hbase hbck -fixAssignments` 命令。

## <a name="how-do-i-force-disable-hdfs-safe-mode-in-a-cluster"></a>如何在叢集中強制停用 HDFS 安全模式？

### <a name="issue"></a>問題

本機 Hadoop 分散式檔案系統 (HDFS) 在 HDInsight 叢集上卡在安全模式中。

### <a name="detailed-description"></a>詳細描述

此錯誤可能是由於您執行下列 HDFS 命令時發生失敗所造成：

```apache
hdfs dfs -D "fs.default.name=hdfs://mycluster/" -mkdir /temp
```

當您嘗試執行類似如下的命令時，可能會看到此錯誤：

```apache
hdiuser@hn0-spark2:~$ hdfs dfs -D "fs.default.name=hdfs://mycluster/" -mkdir /temp
17/04/05 16:20:52 WARN retry.RetryInvocationHandler: Exception while invoking ClientNamenodeProtocolTranslatorPB.mkdirs over hn0-spark2.2oyzcdm4sfjuzjmj5dnmvscjpg.dx.internal.cloudapp.net/10.0.0.22:8020. Not retrying because try once and fail.
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create directory /temp. Name node is in safe mode.
It was turned on manually. Use "hdfs dfsadmin -safemode leave" to turn safe mode off.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1359)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:4010)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:1102)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.mkdirs(ClientNamenodeProtocolServerSideTranslatorPB.java:630)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:640)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:982)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2313)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2309)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1724)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2307)
        at org.apache.hadoop.ipc.Client.getRpcResponse(Client.java:1552)
        at org.apache.hadoop.ipc.Client.call(Client.java:1496)
        at org.apache.hadoop.ipc.Client.call(Client.java:1396)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:233)
        at com.sun.proxy.$Proxy10.mkdirs(Unknown Source)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.mkdirs(ClientNamenodeProtocolTranslatorPB.java:603)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:278)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:194)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:176)
        at com.sun.proxy.$Proxy11.mkdirs(Unknown Source)
        at org.apache.hadoop.hdfs.DFSClient.primitiveMkdir(DFSClient.java:3061)
        at org.apache.hadoop.hdfs.DFSClient.mkdirs(DFSClient.java:3031)
        at org.apache.hadoop.hdfs.DistributedFileSystem$24.doCall(DistributedFileSystem.java:1162)
        at org.apache.hadoop.hdfs.DistributedFileSystem$24.doCall(DistributedFileSystem.java:1158)
        at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirsInternal(DistributedFileSystem.java:1158)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirs(DistributedFileSystem.java:1150)
        at org.apache.hadoop.fs.FileSystem.mkdirs(FileSystem.java:1898)
        at org.apache.hadoop.fs.shell.Mkdir.processNonexistentPath(Mkdir.java:76)
        at org.apache.hadoop.fs.shell.Command.processArgument(Command.java:273)
        at org.apache.hadoop.fs.shell.Command.processArguments(Command.java:255)
        at org.apache.hadoop.fs.shell.FsCommand.processRawArguments(FsCommand.java:119)
        at org.apache.hadoop.fs.shell.Command.run(Command.java:165)
        at org.apache.hadoop.fs.FsShell.run(FsShell.java:297)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:90)
        at org.apache.hadoop.fs.FsShell.main(FsShell.java:350)
mkdir: Cannot create directory /temp. Name node is in safe mode.
```

### <a name="probable-cause"></a>可能的原因

HDInsight 叢集已相應減少為極少數節點。 此節點數目低於或接近 HDFS 複寫因數。

### <a name="resolution-steps"></a>解決步驟 

1. 執行下列命令取得 HDInsight 叢集上的 HDFS 狀態：

   ```apache
   hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -report
   ```

   ```apache
   hdiuser@hn0-spark2:~$ hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -report
   Safe mode is ON
   Configured Capacity: 3372381241344 (3.07 TB)
   Present Capacity: 3138625077248 (2.85 TB)
   DFS Remaining: 3102710317056 (2.82 TB)
   DFS Used: 35914760192 (33.45 GB)
   DFS Used%: 1.14%
   Under replicated blocks: 0
   Blocks with corrupt replicas: 0
   Missing blocks: 0
   Missing blocks (with replication factor 1): 0

   -------------------------------------------------
   Live datanodes (8):

   Name: 10.0.0.17:30010 (10.0.0.17)
   Hostname: 10.0.0.17
   Decommission Status : Normal
   Configured Capacity: 421547655168 (392.60 GB)
   DFS Used: 5288128512 (4.92 GB)
   Non DFS Used: 29087272960 (27.09 GB)
   DFS Remaining: 387172253696 (360.58 GB)
   DFS Used%: 1.25%
   DFS Remaining%: 91.85%
   Configured Cache Capacity: 0 (0 B)
   Cache Used: 0 (0 B)
   Cache Remaining: 0 (0 B)
   Cache Used%: 100.00%
   Cache Remaining%: 0.00%
   Xceivers: 2
   Last contact: Wed Apr 05 16:22:00 UTC 2017
   ...

   ```
2. 您也可以使用下列命令檢查 HDInsight 叢集上的 HDFS 完整性：

   ```apache
   hdiuser@hn0-spark2:~$ hdfs fsck -D "fs.default.name=hdfs://mycluster/" /
   ```

   ```apache
   Connecting to namenode via http://hn0-spark2.2oyzcdm4sfjuzjmj5dnmvscjpg.dx.internal.cloudapp.net:30070/fsck?ugi=hdiuser&path=%2F
   FSCK started by hdiuser (auth:SIMPLE) from /10.0.0.22 for path / at Wed Apr 05 16:40:28 UTC 2017
   ....................................................................................................

   ....................................................................................................
   ..................Status: HEALTHY
   Total size:    9330539472 B
   Total dirs:    37
   Total files:   2618
   Total symlinks:                0 (Files currently being written: 2)
   Total blocks (validated):      2535 (avg. block size 3680686 B)
   Minimally replicated blocks:   2535 (100.0 %)
   Over-replicated blocks:        0 (0.0 %)
   Under-replicated blocks:       0 (0.0 %)
   Mis-replicated blocks:         0 (0.0 %)
   Default replication factor:    3
   Average block replication:     3.0
   Corrupt blocks:                0
   Missing replicas:              0 (0.0 %)
   Number of data-nodes:          8
   Number of racks:               1
   FSCK ended at Wed Apr 05 16:40:28 UTC 2017 in 187 milliseconds

   The filesystem under path '/' is HEALTHY
   ```

3. 如果您確定沒有遺漏、損毀或複寫中的區塊，或是這些區塊可以予以忽略，請執行下列命令使名稱節點脫離安全模式：

   ```apache
   hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -safemode leave
   ```


## <a name="how-do-i-fix-jdbc-or-sqlline-connectivity-issues-with-apache-phoenix"></a>如何修正與 Apache Phoenix 的 JDBC 或 SQLLine 連線問題？

### <a name="resolution-steps"></a>解決步驟

若要與 Phoenix 連線，您必須提供使用中 ZooKeeper 節點的 IP 位址。 請確定 sqlline.py 嘗試連線的 ZooKeeper 服務已啟動並執行。
1. 使用 SSH 登入 HDInsight 叢集。
2. 輸入下列命令：
                
   ```apache
           "/usr/hdp/current/phoenix-client/bin/sqlline.py <IP of machine where Active Zookeeper is running"
   ```

   > [!Note] 
   > 您可以從 Ambari UI 取得使用中 ZooKeeper 節點的 IP 位址。 移至 [HBase] > [快速連結] > [ZK\* (使用中)] > [Zookeeper 資訊]。 

3. 如果 sqlline.py 連線至 Phoenix，而且不會逾時，請執行下列命令以驗證 Phoenix 的可用性和健康狀態：

   ```apache
           !tables
           !quit
   ```      
4. 如果此命令能夠運作，就表示沒有問題。 使用者提供的 IP 位址可能不正確。 不過，如果此命令暫停太久並接著顯示下列錯誤，請繼續執行步驟 5。

   ```apache
           Error while connecting to sqlline.py (Hbase - phoenix) Setting property: [isolation, TRANSACTION_READ_COMMITTED] issuing: !connect jdbc:phoenix:10.2.0.7 none none org.apache.phoenix.jdbc.PhoenixDriver Connecting to jdbc:phoenix:10.2.0.7 SLF4J: Class path contains multiple SLF4J bindings. 
   ```

5. 從前端節點 (hn0) 執行下列命令，以診斷 Phoenix SYSTEM.CATALOG 資料表的狀況：

   ```apache
            hbase shell
                
           count 'SYSTEM.CATALOG'
   ```

   此命令應該傳回的錯誤大致如下： 

   ```apache
           ERROR: org.apache.hadoop.hbase.NotServingRegionException: Region SYSTEM.CATALOG,,1485464083256.c0568c94033870c517ed36c45da98129. is not online on 10.2.0.5,16020,1489466172189) 
   ```
6. 在 Ambari UI 中完成下列步驟，在所有 ZooKeeper 節點上重新啟動 HMaster 服務：

    1. 在 HBase 的 [摘要] 區段中，移至 [HBase] > [Active HBase Master]。 
    2. 在 [元件] 區段中，重新啟動 HBase Master 服務。
    3. 針對所有其餘的 **Standby HBase Master** 服務，重複這些步驟。 

HBase Master 服務可能需要五分鐘的時間，才能穩定和完成復原程序。 幾分鐘後，重複 sqlline.py 命令，以確認 SYSTEM.CATALOG 資料表已啟動且可供查詢。 

當 SYSTEM.CATALOG 資料表回復正常狀態時，應該會自動解決 Phoenix 的連線問題。


## <a name="what-causes-a-master-server-to-fail-to-start"></a>什麼情況導致主要伺服器無法啟動？

### <a name="error"></a>Error 

發生不可部分完成的重新命名失敗。

### <a name="detailed-description"></a>詳細描述

在啟動程序期間，HMaster 會完成許多初始化步驟。 這包括將資料從臨時 (.tmp) 資料夾移至數據資料夾。 HMaster 也會檢視預寫記錄檔 (WAL) 資料夾，以查看是否有任何沒有回應的區域伺服器等等。 

在啟動期間，HMaster 會對這些資料夾執行基本 `list` 命令。 如果任何時間 HMaster 在任一資料夾中看到未預期的檔案，則會擲回例外狀況，而且不會啟動。  

### <a name="probable-cause"></a>可能的原因

在區域伺服器記錄中，嘗試找出檔案建立的時間表，然後查看檔案建立前後是否存在處理序損毀 (請連絡 HBase 支援部門來協助您執行此動作)。這可協助我們提供更健全的機制，讓您可以避免發生這個錯誤 (bug)，並確保處理序順利關閉。

### <a name="resolution-steps"></a>解決步驟

請檢查呼叫堆疊，並嘗試判斷哪一個資料夾可能會造成問題 (例如，它可以是 WAL 資料夾或 .tmp 資料夾)。 接著，在 Cloud Explorer 中或使用 HDFS 命令嘗試找出問題檔案。 這通常是 \*-renamePending.json 檔案。 (\*-renamePending.json 檔案是用來在 WASB 驅動程式中實作不可部分完成之重新命名作業的日誌檔案。 由於此實作中發生錯誤 (bug)，因此這些檔案可能會在發生處理序損毀等狀況之後遺留下來。)請在 Cloud Explorer 中或使用 HDFS 命令強制刪除此檔案。 

這個位置有時候還可能有名稱類似 *$$$.$$$* 的暫存檔案。 您必須使用 HDFS `ls` 命令來查看此檔案；您無法在 Cloud Explorer 中查看此檔案。 若要刪除此檔案，請使用 HDFS 命令 `hdfs dfs -rm /\<path>\/\$\$\$.\$\$\$`。  

執行這些命令之後，HMaster 應立即啟動。 

### <a name="error"></a>Error

區域 xxx 的 *hbase: meta* 中未列出任何伺服器位址。

### <a name="detailed-description"></a>詳細描述

您可能會在 Linux 叢集上看到一則訊息，指出 *hbase: meta* 資料表不在線上。 執行 `hbck` 可能會回報「在任何區域上找不到 hbase: meta 資料表 replicaId 0」。 問題可能是 HMaster 無法在您重新啟動 HBase 之後初始化。 在 HMaster 記錄中，您可能會看到訊息：「區域 hbase: backup \<區域名稱\> 的 hbase: meta 中未列出任何伺服器位址」。  

### <a name="resolution-steps"></a>解決步驟

1. 在 HBase Shell 中，輸入下列命令 (視情況變更實際值)：  

   ```apache
   > scan 'hbase:meta'  
   ```

   ```apache
   > delete 'hbase:meta','hbase:backup <region name>','<column name>'  
   ```

2. 刪除 *hbase: namespace* 項目。 此項目可能是掃描 *hbase: namespace* 資料表時所回報的相同錯誤。

3. 若要讓 HBase 回復執行狀態，請在 Ambari UI 中重新啟動 Active HMaster 服務。  

4. 在 HBase Shell 中，若要啟動所有離線資料表，請執行下列命令：

   ```apache 
   hbase hbck -ignorePreCheckPermission -fixAssignments 
   ```

### <a name="additional-reading"></a>其他閱讀資料

[Unable to process the HBase table](http://stackoverflow.com/questions/4794092/unable-to-access-hbase-table) (無法處理 HBase 資料表)


### <a name="error"></a>Error

HMaster 逾時，並發生類似如下的嚴重例外狀況：「java.io.IOException：等待指派命名空間資料表時，逾時 300000 毫秒」。

### <a name="detailed-description"></a>詳細描述

如果您有許多資料表和區域在重新啟動 HMaster 服務時尚未排清，則可能會遇到此問題。 重新啟動可能會失敗，而且您會看到上述錯誤訊息。  

### <a name="probable-cause"></a>可能的原因

這是 HMaster 服務的已知問題。 一般叢集啟動工作可能需要很長的時間。 HMaster 因為尚未指派命名空間資料表而關閉。 這只會在存在大量未排清資料的情況下發生，因此五分鐘的逾時並不足夠。
  
### <a name="resolution-steps"></a>解決步驟

1. 在 Ambari UI 中，移至 [HBase] > [設定]。 在自訂 hbase-site.xml 檔案中，新增下列設定： 

   ```apache
   Key: hbase.master.namespace.init.timeout Value: 2400000  
   ```

2. 重新啟動必要的服務 (HMaster，以及其他可能的 HBase 服務)。  


## <a name="what-causes-a-restart-failure-on-a-region-server"></a>什麼情況導致區域伺服器上的重新啟動失敗？

### <a name="issue"></a>問題

區域伺服器上的重新啟動失敗可透過遵循最佳做法來加以防止。 建議您在計劃重新啟動 HBase 區域伺服器時暫停工作負載過重的活動。 如果進行關機時應用程式繼續與區域伺服器連線，區域伺服器重新啟動作業會變慢幾分鐘。 此外，先排清所有資料表也是不錯的做法。 如需如何排清資料表的參考，請參閱 [HDInsight HBase: How to improve the HBase cluster restart time by flushing tables](https://blogs.msdn.microsoft.com/azuredatalake/2016/09/19/hdinsight-hbase-how-to-improve-hbase-cluster-restart-time-by-flushing-tables/) (HDInsight HBase：如何藉由排清資料表來改善 HBase 叢集重新啟動時間)。

如果您從 Ambari UI 對 HBase 區域伺服器起始重新啟動作業，您會立即看到區域伺服器關機，但未立即重新啟動。 

以下是幕後發生的情況： 

1. Ambari 代理程式會傳送停止要求給區域伺服器。
2. Ambari 代理程式會等候 30 秒的時間，讓區域伺服器順利關機。 
3. 如果您的應用程式繼續與區域伺服器連線，則伺服器不會立即關機。 超過 30 秒逾時才會關機。 
4. 30 秒後，Ambari 代理程式會傳送強制終止 (`kill -9`) 命令給區域伺服器。 您可以在 ambari-agent 記錄 (位於個別背景工作節點的 /var/log/ 目錄) 中確認這一點：

   ```apache
           2017-03-21 13:22:09,171 - Execute['/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh --config /usr/hdp/current/hbase-regionserver/conf stop regionserver'] {'only_if': 'ambari-sudo.sh  -H -E t
           est -f /var/run/hbase/hbase-hbase-regionserver.pid && ps -p `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid` >/dev/null 2>&1', 'on_timeout': '! ( ambari-sudo.sh  -H -E test -
           f /var/run/hbase/hbase-hbase-regionserver.pid && ps -p `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid` >/dev/null 2>&1 ) || ambari-sudo.sh -H -E kill -9 `ambari-sudo.sh  -H 
           -E cat /var/run/hbase/hbase-hbase-regionserver.pid`', 'timeout': 30, 'user': 'hbase'}
           2017-03-21 13:22:40,268 - Executing '! ( ambari-sudo.sh  -H -E test -f /var/run/hbase/hbase-hbase-regionserver.pid && ps -p `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid` >
           /dev/null 2>&1 ) || ambari-sudo.sh -H -E kill -9 `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid`'. Reason: Execution of 'ambari-sudo.sh su hbase -l -s /bin/bash -c 'export  
           PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/var/lib/ambari-agent ; /usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh --config /usr/hdp/curre
           nt/hbase-regionserver/conf stop regionserver was killed due timeout after 30 seconds
           2017-03-21 13:22:40,285 - File['/var/run/hbase/hbase-hbase-regionserver.pid'] {'action': ['delete']}
           2017-03-21 13:22:40,285 - Deleting File['/var/run/hbase/hbase-hbase-regionserver.pid']
   ```
由於突然關機之故，即使區域伺服器處理序已停止，也可能不會釋放與處理序建立關聯的連接埠。 這種情況可能導致在啟動區域伺服器時發生 AddressBindException，如下列記錄中所示。 您可以在 region-server.log (位於區域伺服器失敗之背景工作節點的 /var/log/hbase 目錄) 中確認這一點。 

   ```apache

   2017-03-21 13:25:47,061 ERROR [main] regionserver.HRegionServerCommandLine: Region server exiting
   java.lang.RuntimeException: Failed construction of Regionserver: class org.apache.hadoop.hbase.regionserver.HRegionServer
   at org.apache.hadoop.hbase.regionserver.HRegionServer.constructRegionServer(HRegionServer.java:2636)
   at org.apache.hadoop.hbase.regionserver.HRegionServerCommandLine.start(HRegionServerCommandLine.java:64)
   at org.apache.hadoop.hbase.regionserver.HRegionServerCommandLine.run(HRegionServerCommandLine.java:87)
   at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76)
   at org.apache.hadoop.hbase.util.ServerCommandLine.doMain(ServerCommandLine.java:126)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.main(HRegionServer.java:2651)
        
   Caused by: java.lang.reflect.InvocationTargetException
   at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
   at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
   at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
   at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.constructRegionServer(HRegionServer.java:2634)
   ... 5 more
        
   Caused by: java.net.BindException: Problem binding to /10.2.0.4:16020 : Address already in use
   at org.apache.hadoop.hbase.ipc.RpcServer.bind(RpcServer.java:2497)
   at org.apache.hadoop.hbase.ipc.RpcServer$Listener.<init>(RpcServer.java:580)
   at org.apache.hadoop.hbase.ipc.RpcServer.<init>(RpcServer.java:1982)
   at org.apache.hadoop.hbase.regionserver.RSRpcServices.<init>(RSRpcServices.java:863)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.createRpcServices(HRegionServer.java:632)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.<init>(HRegionServer.java:532)
   ... 10 more
        
   Caused by: java.net.BindException: Address already in use
   at sun.nio.ch.Net.bind0(Native Method)
   at sun.nio.ch.Net.bind(Net.java:463)
   at sun.nio.ch.Net.bind(Net.java:455)
   at sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:223)
   at sun.nio.ch.ServerSocketAdaptor.bind(ServerSocketAdaptor.java:74)
   at org.apache.hadoop.hbase.ipc.RpcServer.bind(RpcServer.java:2495)
   ... 15 more
   ```

### <a name="resolution-steps"></a>解決步驟

1. 請嘗試減少 HBase 區域伺服器的負載，再起始重新啟動。 
2. 或者 (如果步驟 1 沒有作用)，請嘗試使用下列命令，手動重新啟動背景工作節點上的區域伺服器：

   ```apache
   sudo su - hbase -c "/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh stop regionserver"
   sudo su - hbase -c "/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh start regionserver"   
   ```

### <a name="see-also"></a>另請參閱
[使用 Azure HDInsight 進行疑難排解](../../hdinsight/hdinsight-troubleshoot-guide.md)
