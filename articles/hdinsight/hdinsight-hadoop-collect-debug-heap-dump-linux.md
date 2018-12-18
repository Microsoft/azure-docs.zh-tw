---
title: 在 HDInsight 上啟用 Hadoop 服務的堆積傾印 - Azure
description: 在以 Linux 為基礎的 HDInsight 叢集上啟用 Hadoop 服務的堆積傾印，以進行偵錯和分析。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: 7fa0ebb1946a856d02296b4ec752113ff8fecbd5
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43095577"
---
# <a name="enable-heap-dumps-for-hadoop-services-on-linux-based-hdinsight"></a>在以 Linux 為基礎的 HDInsight 上啟用 Hadoop 服務的堆積傾印

[!INCLUDE [heapdump-selector](../../includes/hdinsight-selector-heap-dump.md)]

堆積傾印含有應用程式記憶體的快照，其中包括建立傾印時的變數值。 因此它們有助於為執行階段發生的問題進行診斷。

> [!IMPORTANT]
> 本文件中的步驟只適用於使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="whichServices"></a>服務

您可以啟用下列服務的堆積傾印：

* **hcatalog** - tempelton
* **hive** - hiveserver2、metastore、derbyserver
* **mapreduce** - jobhistoryserver
* **yarn** - resourcemanager、nodemanager、timelineserver
* **hdfs** - datanode、secondarynamenode、namenode

您也可以針對 HDInsight 所執行的 map 和 reduce 處理序來啟用堆積傾印。

## <a name="configuration"></a>了解堆積傾印組態

堆積傾印的啟用方式，是在服務啟動時將選項 (有時稱為參數) 傳遞至 JVM。 就大部分的 Hadoop 服務而言，您可以修改用於啟動服務的殼層指令碼以略過這些選項。

在每個指令碼中，有一項 **\*\_OPTS** 匯出，其中含有傳遞至 JVM 的選項。 例如，在 **hadoop-env.sh** 指令碼中，以 `export HADOOP_NAMENODE_OPTS=` 為開頭的那一行即含有 NameNode 服務的選項。

map 和 reduce 處理序會稍有不同，因為這些作業是 MapReduce 服務的子處理序。 每個 map 或 reduce 處理序都會在一個子容器中執行，且有兩個含有 JVM 選項的項目。 兩者均包含在 **mapred-site.xml** 中：

* **mapreduce.admin.map.child.java.opts**
* **mapreduce.admin.reduce.child.java.opts**

> [!NOTE]
> 我們建議使用 Ambari 來修改指令碼和 mapred-site.xml 設定，因為 Ambari 會處理叢集中跨節點的複寫變更。 如需特定的步驟，請參閱 [使用 Ambari](#using-ambari) 一節。

### <a name="enable-heap-dumps"></a>啟用堆積傾印

以下選項可在發生 OutOfMemoryError 時啟用堆積傾印：

    -XX:+HeapDumpOnOutOfMemoryError

**+** 表示已啟用此選項。 預設值為停用。

> [!WARNING]
> 根據預設不會在 HDInsight 上啟用 Hadoop 服務的堆積傾印，因為傾印檔案可能會很龐大。 如果您為了進行疑難排解而啟用它們，請記得在重現問題並收集傾印檔案之後將其停用。

### <a name="dump-location"></a>傾印位置

傾印檔案的預設位置是目前的工作目錄。 您可以使用以下選項，控制檔案的儲存位置：

    -XX:HeapDumpPath=/path

例如，使用 `-XX:HeapDumpPath=/tmp` 會使傾印儲存在 /tmp 目錄中。

### <a name="scripts"></a>指令碼

您也可以在發生 **OutOfMemoryError** 時觸發指令碼。 例如觸發通知，讓您知道發生了錯誤。 使用下列選項可在發生 __OutOfMemoryError__ 時觸發指令碼：

    -XX:OnOutOfMemoryError=/path/to/script

> [!NOTE]
> 由於 Hadoop 是分散式系統，因此所有使用的指令碼都必須放置在服務所在叢集中的所有節點上。
> 
> 指令碼也必須位於服務所屬的帳戶可以存取的位置中，而且必須提供執行權限。 例如，您可能想要在 `/usr/local/bin` 中存放指令碼，並用 `chmod go+rx /usr/local/bin/filename.sh` 授與讀取和執行權限。

## <a name="using-ambari"></a>使用 Ambari

若要修改服務的組態，請依照下列步驟進行：

1. 開啟叢集的 Ambari Web UI。 URL 為 https://YOURCLUSTERNAME.azurehdinsight.net。

    出現提示時，請使用您叢集的 HTTP 帳戶名稱 (預設值：admin) 和密碼來向網站驗證。

   > [!NOTE]
   > Ambari 可能會再次提示您輸入使用者名稱和密碼。 此時，請輸入同一組帳戶名稱和密碼

2. 使用左側的清單，選取您想要修改的服務區域。 例如 **HDFS**。 在中間區域內，選取 [設定]  索引標籤。

    ![Ambari Web 圖片 (已選取 HDFS 設定索引標籤)](./media/hdinsight-hadoop-heap-dump-linux/serviceconfig.png)

3. 在 [篩選...] 項目中輸入 **opts**。 只會顯示包含此文字的項目。

    ![篩選的清單](./media/hdinsight-hadoop-heap-dump-linux/filter.png)

4. 針對您想要啟用堆積傾印的服務尋找 **\*\_OPTS** 項目，並新增您想要啟用的選項。 在下圖中，我將 `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/` 新增至 **HADOOP\_NAMENODE\_OPTS** 項目：

    ![含有 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/ 的 HADOOP_NAMENODE_OPTS](./media/hdinsight-hadoop-heap-dump-linux/opts.png)

   > [!NOTE]
   > 啟用 map 或 reduce 子處理序的堆積傾印時，請尋找名為 **mapreduce.admin.map.child.java.opts** 和 **mapreduce.admin.reduce.child.java.opts** 的欄位。

    使用 [儲存] 按鈕來儲存變更。 您可以輸入簡短的附註來說明所做的變更。

5. 套用變更後，一或多個服務旁邊就會出現**必須重新啟動**圖示。

    ![必須重新啟動圖示和重新啟動按鈕](./media/hdinsight-hadoop-heap-dump-linux/restartrequiredicon.png)

6. 選取每個需要重新啟動的服務，並使用 [服務動作] 按鈕**開啟維護模式**。 維護模式可避免您在重新啟動服務產生警示。

    ![開啟維護模式功能表](./media/hdinsight-hadoop-heap-dump-linux/maintenancemode.png)

7. 啟用維護模式後，請使用服務的 [重新啟動] 按鈕來**重新啟動所有受影響的項目**。

    ![重新啟動所有受影響的項目](./media/hdinsight-hadoop-heap-dump-linux/restartbutton.png)

   > [!NOTE]
   > 其他服務的 [重新啟動] 按鈕項目可能會有所不同。

8. 重新啟動服務後，請使用 [服務動作] 按鈕**關閉維護模式**。 這麼做可讓 Ambari 繼續監視服務是否有警示。

