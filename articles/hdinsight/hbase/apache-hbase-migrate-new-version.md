---
title: 將 HBase 叢集移轉至新的版本 - Azure HDInsight
description: 如何將 HBase 叢集移轉至新的版本。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: ashishth
ms.openlocfilehash: 64b3762c40cc2e01944d78c546ebe267503526a7
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049325"
---
# <a name="migrate-an-hbase-cluster-to-a-new-version"></a>將 HBase 叢集移轉至新的版本

將 Spark 和 Hadoop 等作業型叢集進行升級相當簡單，請參閱[將 HDInsight 叢集升級為更新的版本](../hdinsight-upgrade-cluster.md)：

1. 將暫時性 (在本機儲存的) 資料備份。
2. 刪除現有的叢集。
3. 在相同的 VNET 子網路中建立新的叢集。
4. 將暫時性資料匯入。
5. 在新叢集上啟動作業並繼續處理。

升級 HBase 叢集需要一些額外的步驟，如本文中所述。

> [!NOTE]
> 升級時的停機時間不應太長，為大約幾分鐘。 這個停機時間是因排清所有記憶體內部資料的步驟，接著在新叢集上設定及重新啟動服務的時間所造成。 您的結果會依節點數目、資料量和其他變數而有所不同。

## <a name="review-hbase-compatibility"></a>檢閱 HBase 相容性

在升級 HBase 之前，請確認來源和目的地叢集上的 HBase 版本相容。 如需詳細資訊，請參閱[可以搭配 HDInsight 使用的 Hadoop 元件和版本](../hdinsight-component-versioning.md)。

> [!NOTE]
> 強烈建議您檢閱 [HBase 書籍](https://hbase.apache.org/book.html#upgrading)中的版本相容性矩陣。

以下是版本相容性矩陣的範例，其中 Y 表示相容性，N 則表示可能的不相容：

| 相容性類型 | 主要版本| 次要版本 | 修補程式 |
| --- | --- | --- | --- |
| 用戶端-伺服器連線相容性 | N | Y | Y |
| 伺服器-伺服器相容性 | N | Y | Y |
| 檔案格式相容性 | N | Y | Y |
| 用戶端 API 相容性 | N | Y | Y |
| 用戶端二進位相容性 | N | N | Y |
| **伺服器端有限的 API 相容性** |  |  |  |
| Stable | N | Y | Y |
| 發展中 | N | N | Y |
| 不穩定 | N | N | N |
| 相依性相容性 | N | Y | Y |
| 作業相容性 | N | N | Y |

> [!NOTE]
> 任何重大的不相容都需在 HBase 版本的版本資訊中加以說明。

## <a name="upgrade-with-same-hbase-major-version"></a>以相同的 HBase 主要版本進行升級

下列案例適用於使用相同的 HBase 主要版本從 HDInsight 3.4 升級至 3.6 (皆隨附 Apache HBase 1.1.2)。 只要來源與目的地版本之間沒有相容性問題，其他的版本升級都很類似。

1. 請確定您的應用程式與新的版本相容，如 HBase 相容性矩陣和版本資訊中所示。 在執行 HDInsight 和 HBase 目標版本的叢集中測試您的應用程式。

2. 使用包含不同容器名稱的相同儲存體帳戶來[設定新的目的地 HDInsight 叢集](../hdinsight-hadoop-provision-linux-clusters.md)：

    ![使用相同的儲存體帳戶，但建立不同的容器](./media/apache-hbase-migrate-new-version/same-storage-different-container.png)

3. 排清來源 HBase 叢集。 這是您要開始升級的叢集。 HBase 會將內送資料寫入名為 _memstore_ 的記憶體內部存放區中。 memstore 觸達特定的大小之後，系統會將 memstore 排清到磁碟，以長期儲存在叢集的儲存體帳戶中。 將舊的叢集刪除時，會回收 memstores，可能還會遺失資料。 若要將每個資料表的 memstore 手動排清至磁碟，請執行下列指令碼。 此指令碼的最新版本位於 Azure 的 [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/scripts/flush_all_tables.sh)。

    ```bash
    #!/bin/bash
    
    #-------------------------------------------------------------------------------#
    # SCRIPT TO FLUSH ALL HBASE TABLES.
    #-------------------------------------------------------------------------------#
    
    LIST_OF_TABLES=/tmp/tables.txt
    HBASE_SCRIPT=/tmp/hbase_script.txt
    TARGET_HOST=$1
    
    usage ()
    {
        if [[ "$1" == "-h" ]] || [[ "$1" == "--help" ]]
        then
            cat << ...
    
    Usage: 
    
    $0 [hostname]
    
    Providing hostname is optional and not required when the script is executed within HDInsight cluster with access to 'hbase shell'.
    
    However hostname should be provided when executing the script as a script-action from HDInsight portal.
    
    For Example:
    
        1.  Executing script inside HDInsight cluster (where 'hbase shell' is 
            accessible):
    
            $0 
    
            [No need to provide hostname]
    
        2.  Executing script from HDinsight Azure portal:
    
            Provide Script URL.
    
            Provide hostname as a parameter (i.e. hn0, hn1 or wn2 etc.).
    ...
            exit
        fi
    }
    
    validate_machine ()
    {
        THIS_HOST=`hostname`
    
        if [[ ! -z "$TARGET_HOST" ]] && [[ $THIS_HOST  != $TARGET_HOST* ]]
        then
            echo "[INFO] This machine '$THIS_HOST' is not the right machine ($TARGET_HOST) to execute the script."
            exit 0
        fi
    }
    
    get_tables_list ()
    {
    hbase shell << ... > $LIST_OF_TABLES 2> /dev/null
        list
        exit
    ...
    }
    
    add_table_for_flush ()
    {
        TABLE_NAME=$1
        echo "[INFO] Adding table '$TABLE_NAME' to flush list..."
        cat << ... >> $HBASE_SCRIPT
            flush '$TABLE_NAME'
    ...
    }
    
    clean_up ()
    {
        rm -f $LIST_OF_TABLES
        rm -f $HBASE_SCRIPT
    }
    
    ########
    # MAIN #
    ########
    
    usage $1
    
    validate_machine
    
    clean_up
    
    get_tables_list
    
    START=false
    
    while read LINE 
    do 
        if [[ $LINE == TABLE ]] 
        then
            START=true
            continue
        elif [[ $LINE == *row*in*seconds ]]
        then
            break
        elif [[ $START == true ]]
        then
            add_table_for_flush $LINE
        fi
    
    done < $LIST_OF_TABLES
    
    cat $HBASE_SCRIPT
    
    hbase shell $HBASE_SCRIPT << ... 2> /dev/null
    exit
    ...
    
    ```
    
4. 停止擷取舊的 HBase 叢集。
5. 若要確定系統已排清 memstore 中的任何新資料，請再次執行上述指令碼。
6. 在舊叢集上登入 Ambari (https://OLDCLUSTERNAME.azurehdidnsight.net) 並停止 HBase 服務。 當系統提示您確認是否要停止服務時，請核取方塊以開啟 HBase 的維護模式。 如需連線至 Ambari 及使用 Ambari 的詳細資訊，請參閱[使用 Ambari Web UI 管理 HDInsight 叢集](../hdinsight-hadoop-manage-ambari.md)。

    ![在 Ambari 中，依序按一下 [服務] 索引標籤、左側功能表中的 [HBase]，及 [服務動作] 底下的 [停止]](./media/apache-hbase-migrate-new-version/stop-hbase-services.png)

    ![勾選 [開啟 HBase 的維護模式] 核取方塊，然後確認](./media/apache-hbase-migrate-new-version/turn-on-maintenance-mode.png)

7. 在新的 HDInsight 叢集上登入 Ambari。 變更 `fs.defaultFS` HDFS 設定，以指向原始叢集所使用的容器名稱。 此設定位於 [HDFS] > [設定] > [進階] > [進階核心網站] 之下。

    ![在 Ambari 中，依序按一下 [服務] 索引標籤、左側功能表中的 [HDFS]、[設定] 索引標籤，及下方的 [進階] 索引標籤](./media/apache-hbase-migrate-new-version/hdfs-advanced-settings.png)

    ![在 Ambari 中，變更容器名稱](./media/apache-hbase-migrate-new-version/change-container-name.png)

8. 儲存您的變更。
9. 如 Ambari 所示，重新啟動所有必要的服務。
10. 將您的應用程式指向新的叢集。

    > [!NOTE]
    > 您應用程式的靜態 DNS 會在升級時變更。 您不必將此 DNS 寫入程式碼，而是可以在網域名稱的 DNS 設定中設定 CNAME，以指向叢集的名稱。 另一個選項是使用您應用程式的組態檔，以便您無須重新部署即可進行更新。

11. 開始擷取，以查看一切是否皆如預期運作。
12. 如果新的叢集令人滿意，請將原始叢集刪除。

## <a name="next-steps"></a>後續步驟

若要深入了解 HBase 和升級 HDInsight 叢集，請參閱下列文章：

* [將 HDInsight 叢集升級為更新的版本](../hdinsight-upgrade-cluster.md)
* [使用 Ambari Web UI 監視和管理 Azure HDInsight](../hdinsight-hadoop-manage-ambari.md)
* [Hadoop 元件和版本](../hdinsight-component-versioning.md)
* [使用 Ambari 將組態最佳化](../hdinsight-changing-configs-via-ambari.md#hbase-optimization-with-the-ambari-web-ui)
