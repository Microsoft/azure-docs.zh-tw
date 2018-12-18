---
title: 設定 HBase 和 Phoenix 備份和複寫 - Azure HDInsight
description: 設定 HBase 和 Phoenix 的備份和複寫。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: ashishth
ms.openlocfilehash: 0dfb1cf5ce16e9aa30bb7f9fcc43bd24ccb90d76
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042214"
---
# <a name="set-up-backup-and-replication-for-hbase-and-phoenix-on-hdinsight"></a>設定 HDInsight 上 HBase 和 Phoenix 的備份和複寫

HBase 支援數種防範資料遺失的方法：

* 複製 `hbase` 資料夾
* 匯出然後匯入
* 複製資料表
* 快照集
* 複寫

> [!NOTE]
> Apache Phoenix 會將其中繼資料儲存在 HBase 資料表中，因此該中繼資料會在您備份 HBase 系統目錄資料表時備份。

下列各節說明這些方法的使用案例。

## <a name="copy-the-hbase-folder"></a>複製 hbase 資料夾

使用這個方法時，您會複製所有 HBase 資料，無法選擇小部分的資料表或資料行系列。 後續方法可提供更好的控制。

HDInsight 中的 HBase 會使用建立叢集時選取的預設儲存體，即 Azure 儲存體 blob 或 Azure Data Lake Store。 不管是哪一種，HBase 都會將其資料和中繼資料檔案儲存在下列路徑中：

    /hbase

* 在 Azure 儲存體帳戶中，`hbase` 資料夾位於 blob 容器的根目錄：

    ```
    wasbs://<containername>@<accountname>.blob.core.windows.net/hbase
    ```

* 在 Azure Data Lake Store 中，`hbase` 資料夾位於佈建叢集時所指定的根路徑之下。 此根路徑通常會有 `clusters` 資料夾，其中包含一個以您的 HDInsight 叢集命名的子資料夾：

    ```
    /clusters/<clusterName>/hbase
    ```

在任一種情況下，`hbase` 資料夾都會包含 HBase 排清至磁碟的所有資料，但不包含記憶體中的資料。 您必須先關閉叢集，才能信任此資料夾能精確呈現 HBase 資料。

刪除叢集之後，您可以讓資料留在原處，或將資料複製到新的位置：

* 建立指向目前儲存位置的新 HDInsight 執行個體。 以所有現有的資料建立新的執行個體。

* 將 `hbase` 資料夾複製到其他 Azure 儲存體 blob 容器或 Data Lake Store 位置，然後建立包含該資料的新叢集。 對 Azure 儲存體使用 [AzCopy](../../storage/common/storage-use-azcopy.md)，而對 Data Lake Store 使用 [AdlCopy](../../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md)。

## <a name="export-then-import"></a>匯出然後匯入

在來源 HDInsight 叢集上，使用匯出公用程式 (隨附於 HBase) 將資料從來源資料表匯出至預設連結的儲存體。 您可以接著將匯出的資料夾複製到目的地儲存位置，然後在目的地 HDInsight 叢集上執行匯入公用程式。

若要匯出資料表，先透過 SSH 連線到來源 HDInsight 叢集的前端節點，然後執行下列 `hbase` 命令：

    hbase org.apache.hadoop.hbase.mapreduce.Export "<tableName>" "/<path>/<to>/<export>"

若要匯出資料表，請透過 SSH 連線到目的地 HDInsight 叢集的前端節點，然後執行下列 `hbase` 命令：

    hbase org.apache.hadoop.hbase.mapreduce.Import "<tableName>" "/<path>/<to>/<export>"

指定完整匯出路徑，可以是預設儲存體或任何連結的儲存體選項。 例如，在 Azure 儲存體中：

    wasbs://<containername>@<accountname>.blob.core.windows.net/<path>

在 Azure Data Lake Store 中，語法是：

    adl://<accountName>.azuredatalakestore.net:443/<path>

這個方法可提供資料表層級的細微性。 您也可以指定要包含的資料列日期範圍，這可讓您以累加方式執行此程序。 每個日期都是以 Unix Epoch 以來的毫秒數表示。

    hbase org.apache.hadoop.hbase.mapreduce.Export "<tableName>" "/<path>/<to>/<export>" <numberOfVersions> <startTimeInMS> <endTimeInMS>

請注意，您必須指定要匯出之每個資料列的版本數目。 若要涵括日期範圍中的所有版本，請將 `<numberOfVersions>` 設定為大於資料列版本最大可能數目的值，例如 100000。

## <a name="copy-tables"></a>複製資料表

CopyTable 公用程式會將資料從來源資料表逐列複製到現有的目的地資料表，並保持與來源相同的結構描述。 目的地資料表可以位於相同叢集或不同的 HBase 叢集上。

若要在叢集內使用 CopyTable，請透過 SSH 連線到來源 HDInsight 叢集的前端節點，然後執行下列 `hbase` 命令：

    hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=<destTableName> <srcTableName>

若要使用 CopyTable 複製到不同叢集上的資料表，請新增 `peer` 參數與目的地叢集的位址：

    hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=<destTableName> --peer.adr=<destinationAddress> <srcTableName>

目的地位址由下列三個部分組成：

    <destinationAddress> = <ZooKeeperQuorum>:<Port>:<ZnodeParent>

* `<ZooKeeperQuorum>` 是以逗號分隔的 ZooKeeper 節點清單，例如：

    zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net

* HDInsight 上的 `<Port>` 會預設為 2181，而 `<ZnodeParent>` 為 `/hbase-unsecure`，因此完整 `<destinationAddress>` 會是：

    zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net:2181:/hbase-unsecure

如需有關如何為您的 HDInsight 叢集擷取這些值的詳細資訊，請參閱本文中的[手動收集 ZooKeeper 仲裁清單](#manually-collect-the-zookeeper-quorum-list)。

CopyTable 公用程式也支援以參數指定要複製的資料列時間範圍，以及以指定資料表中要複製的資料行系列子集。 若要查看 CopyTable 支援的完整參數清單，請在不用任何參數的情形下執行 CopyTable：

    hbase org.apache.hadoop.hbase.mapreduce.CopyTable

CopyTable 會掃描即將複製到目的地資料表的整個來源資料表內容。 CopyTable 執行時，這可能會降低 HBase 叢集的效能。

> [!NOTE]
> 若要將資料表間的資料複製自動化，請參閱 GitHub 上 [Azure HBase Utils](https://github.com/Azure/hbase-utils/tree/master/replication) 存放庫中的 `hdi_copy_table.sh` 指令碼。

### <a name="manually-collect-the-zookeeper-quorum-list"></a>手動收集 ZooKeeper 仲裁清單

當兩個 HDInsight 叢集位於相同虛擬網路時，如先前所述，系統會自動解析內部主機名稱。 若要對經由 VPN 閘道連線的兩個不同虛擬網路中的 HDInsight 叢集使用 CopyTable，您必須在仲裁中提供 Zookeeper 節點的主機 IP 位址。

若要取得仲裁主機名稱，請執行下列 curl 命令：

    curl -u admin:<password> -X GET -H "X-Requested-By: ambari" "https://<clusterName>.azurehdinsight.net/api/v1/clusters/<clusterName>/configurations?type=hbase-site&tag=TOPOLOGY_RESOLVED" | grep "hbase.zookeeper.quorum"

curl 命令會擷取包含 HBase 組態資訊的 JSON 文件，而 grep 命令只會傳回 "hbase.zookeeper.quorum" 項目，例如：

    "hbase.zookeeper.quorum" : "zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net"

仲裁主機名稱值是冒號右邊的整個字串。

若要擷取這些主機的 IP 位址，對上述清單中的每個主機使用下列 curl 命令：

    curl -u admin:<password> -X GET -H "X-Requested-By: ambari" "https://<clusterName>.azurehdinsight.net/api/v1/clusters/<clusterName>/hosts/<zookeeperHostFullName>" | grep "ip"

在此 curl 命令中，`<zookeeperHostFullName>` 是 ZooKeeper 主機的完整 DNS 名稱，例如 `zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net` 範例。 此命令的輸出包含所指定主機的 IP 位址，例如：

    100    "ip" : "10.0.0.9",

在您收集仲裁中 ZooKeeper 節點的所有 IP 位址之後，重建的目的地位址：

    <destinationAddress>  = <Host_1_IP>,<Host_2_IP>,<Host_3_IP>:<Port>:<ZnodeParent>

在我們的範例中：

    <destinationAddress> = 10.0.0.9,10.0.0.8,10.0.0.12:2181:/hbase-unsecure

## <a name="snapshots"></a>快照集

快照集可讓您建立 HBase 資料存放區在某個時間點的資料備份。 快照集的額外負荷最低，幾秒就能完成，因為快照集作業實際上是擷取該瞬間儲存體中所有檔案名稱的中繼資料作業。 建立快照集時，不會真的複製任何資料。 快照集依賴 HDFS 中儲存之資料的不可變本質 ，HDFS 中的更新、刪除、插入都會以新資料表示。 您可以在同一個叢集上還原 (複製) 快照集，或將快照集匯出至其他叢集。

若要建立快照集，請透過 SSH 連線至 HDInsight HBase 叢集的前端節點並啟動 `hbase` Shell：

    hbase shell

在 hbase Shell 中，使用 snapshot 命令搭配資料表名稱和這個快照集的名稱：

    snapshot '<tableName>', '<snapshotName>'

若要在 `hbase` Shell 中依名稱還原快照集，請先停用資料表，然後還原快照集，再重新啟用資料表：

    disable '<tableName>'
    restore_snapshot '<snapshotName>'
    enable '<tableName>'

若要將快照集還原為新資料表，請使用 clone_snapshot：

    clone_snapshot '<snapshotName>', '<newTableName>'

若要將快照集匯出到 HDFS 讓其他叢集使用，請先如前述建立快照集，然後使用 ExportSnapshot 公用程式。 在 SSH 連線至前端節點的工作階段內執行此公用程式，而不是在 `hbase` Shell 內：

     hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot <snapshotName> -copy-to <hdfsHBaseLocation>

`<hdfsHBaseLocation>` 可以是來源叢集可存取的任何儲存體位置，而且應指向目的地叢集所使用的 hbase 資料夾。 例如，如果來源叢集附加了次要 Azure 儲存體帳戶，而且該帳戶可供存取目的地叢集的預設儲存體所使用的容器，您可以使用此命令：

    hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot 'Snapshot1' -copy-to 'wasbs://secondcluster@myaccount.blob.core.windows.net/hbase'

匯出快照集後，透過 SSH 連線到目的地叢集的前端節點，然後如先前所述使用 restore_snapshot 命令還原快照集。

快照集可提供資料表在執行 `snapshot` 命令時的完整備份。 快照集不能依時段執行增量快照集，也不能指定快照集中要包含哪些資料行系列的子集。

## <a name="replication"></a>複寫

HBase 複寫會使用非同步機制自動將交易從來源叢集推送至目的地叢集，這對來源叢集的額外負荷最少。 在 HDInsight 中，您可以在下列情況下設定叢集之間的複寫：

* 來源和目的地叢集位於相同虛擬網路中。
* 來源和目的地叢集位於以 VPN 閘道連線的不同虛擬網路中，且這兩個叢集都位於相同的地理位置。
* 來源和目的地叢集位於以 VPN 閘道連線的不同虛擬網路中，且這兩個叢集位於不同的地理位置。

設定複寫的一般步驟如下：

1. 在來源叢集上，建立資料表並填入資料。
2. 在目的地叢集上，建立空的目的地資料表，並具有來源資料表的結構描述。
3. 將目的地叢集註冊為來源叢集的同儕節點。
4. 啟用所需來源資料表的複寫。
5. 將現有的資料從來源資料表複製到目的地資料表。
6. 複寫會自動將來源資料表上新的資料修改複製到目的地資料表。

若要在 HDInsight 上啟用複寫，請對執行中的來源 HDInsight 叢集套用指令碼動作。 如需在叢集中啟用複寫，或在使用 Azure Resource Management 範本在虛擬網路中建立的範例叢集上試驗複寫的逐步解說，請參閱[設定 HBase 複寫](apache-hbase-replication.md)。 這篇文章也包含啟用 Phoenix 中繼資料複寫的指示。

## <a name="next-steps"></a>後續步驟

* [設定 HBase 複寫](apache-hbase-replication.md)
