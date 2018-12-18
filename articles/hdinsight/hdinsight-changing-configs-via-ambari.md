---
title: 使用 Ambari - Azure HDInsight 將叢集組態最佳化
description: 使用 Ambari web UI 設定 HDInsight 叢集並使之最佳化。
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 07/09/2018
ms.author: ashish
ms.openlocfilehash: 73fdd3f221e35bc1e0b0904bdbbaa63525ba4be3
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43105468"
---
# <a name="use-ambari-to-optimize-hdinsight-cluster-configurations"></a>使用 Ambari 將 HDInsight 叢集設定最佳化

HDInsight 提供大規模資料處理應用程式的 Apache Hadoop 叢集。 管理、監視和最佳化這些複雜的多節點叢集相當有挑戰性。 [Apache Ambari](http://ambari.apache.org/) 是管理和監視 HDInsight Linux 叢集的 Web 介面。  若為 Windows 叢集，請使用 Ambari [REST API](hdinsight-hadoop-manage-ambari-rest-api.md)。

如需使用 Ambari Web UI 的說明，請參閱[使用 Ambari Web UI 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)

使用您的叢集認證登入在 `https://CLUSTERNAME.azurehdidnsight.net` 的 Ambari。 初始畫面會顯示概觀儀表板。

![Ambari 儀表板](./media/hdinsight-changing-configs-via-ambari/ambari-dashboard.png)

Ambari Web UI 可用來管理主機、服務、警示、設定及檢視。 Ambari 無法用來建立 HDInsight 叢集、升級服務、管理堆疊和版本、解除委任或重新委任主機，或將服務新增到叢集。

## <a name="manage-your-clusters-configuration"></a>管理您的叢集設定

組態設定可協助微調特定服務。 若要修改服務的組態設定，請選取 [服務] 資訊看板 (位於左邊) 中的服務，然後瀏覽至服務詳細資料頁面中的 [設定] 索引標籤。

![[服務] 資訊看板](./media/hdinsight-changing-configs-via-ambari/services-sidebar.png)

### <a name="modify-namenode-java-heap-size"></a>修改 NameNode Java 堆積大小

NameNode Java 堆積大小取決於許多因素，例如叢集的負載、檔案數和區塊數。 1 GB 的預設大小適用於大部分的叢集，而某些工作負載可能需要更多或更少的記憶體。 

修改 NameNode Java 堆積大小：

1. 選取[服務] 資訊看板中的 [HDFS]，並瀏覽至 [設定] 索引標籤。

    ![HDFS 設定](./media/hdinsight-changing-configs-via-ambari/hdfs-config.png)

1. 找出 [NameNode Java 堆積大小] 設定。 您也可以使用 [篩選] 文字方塊來輸入並尋找特定的設定。 選取設定名稱旁邊的**畫筆**圖示。

    ![NameNode Java 堆積大小](./media/hdinsight-changing-configs-via-ambari/java-heap-size.png)

1. 在文字方塊中輸入新值，然後按 **Enter** 儲存變更。

    ![編輯 NameNode Java 堆積大小](./media/hdinsight-changing-configs-via-ambari/java-heap-size-edit.png)

1. NameNode Java 堆積大小從 1GB 變更為 2 GB。

    ![編輯的 NameNode Java 堆積大小](./media/hdinsight-changing-configs-via-ambari/java-heap-size-edited.png)

1. 按一下設定畫面頂端的綠色 [儲存] 按鈕，儲存您的變更。

    ![儲存變更](./media/hdinsight-changing-configs-via-ambari/save-changes.png)

## <a name="hive-optimization"></a>Hive 最佳化

下節會說明將 Hive 整體效能最佳化的設定選項。

1. 若要修改 Hive 組態參數，請選取 [服務] 資訊看板中的 [Hive]。
1. 瀏覽至 [設定] 索引標籤。

### <a name="set-the-hive-execution-engine"></a>設定 Hive 執行引擎

Hive 提供兩個執行引擎：MapReduce 和 Tez。 Tez 比 MapReduce 更快。 HDInsight Linux 叢集有 Tez 做為預設執行引擎。 變更執行引擎：

1. 在 Hive [設定] 索引標籤的 [篩選] 方塊中，輸入**執行引擎**。

    ![搜尋執行引擎](./media/hdinsight-changing-configs-via-ambari/search-execution.png)

1. [最佳化] 屬性的預設值是 **Tez**。

    ![最佳化 - Tez](./media/hdinsight-changing-configs-via-ambari/optimization-tez.png)

### <a name="tune-mappers"></a>微調對應程式

Hadoop 會嘗試將單一檔案分割 (*對應*) 為多個檔案，並且平行處理產生的檔案。 對應程式的數目取決於分割的數目。 下列兩個組態參數會影響 Tez 執行引擎的分割數目：

* `tez.grouping.min-size`：群組分割大小下限，預設值是 16 MB (16777216 位元組)。
* `tez.grouping.max-size`：群組分割大小上限，預設值為 1 GB (1,073,741,824 位元組)。

根據效能的經驗法則，降低這兩個參數可改善延遲、增加更多輸送量。

例如，若要設定 128 MB 資料大小的四個對應程式工作，您會將這兩個參數分別設定為 32 MB (33,554,432 位元組)。

1. 若要修改限制參數，請瀏覽至 Tez 服務的 [設定] 索引標籤。 展開 [一般] 面板，並找出 `tez.grouping.max-size` 和 `tez.grouping.min-size` 參數。

1. 將這兩個參數設定為 **33,554,432** 位元組 (32 MB)。

    ![Tez 群組大小](./media/hdinsight-changing-configs-via-ambari/tez-grouping-size.png)
 
這些變更會影響伺服器的所有 Tez 作業。 若要得到最佳結果，請選擇適當的參數值。

### <a name="tune-reducers"></a>微調歸納器

ORC 和 Snappy 均可達到高效能。 不過，Hive 預設的歸納器數量可能會太少，因而導致瓶頸。

舉例而言，假設您有 50 GB 的輸入資料大小。 該資料為 ORC 格式，而且經過 Snappy 壓縮的處理後成為 1GB。 Hive 估計所需的歸納器數目為：(對應程式中的位元組輸入數目/`hive.exec.reducers.bytes.per.reducer`)。

根據預設設定，此範例為 4 個歸納器。

`hive.exec.reducers.bytes.per.reducer` 參數指定每個歸納器處理的位元組數目。 預設值是 64 MB。 降低這個值會增加平行處理原則，可能會改善效能。 過度降低也會產生過多歸納器，而可能嚴重影響效能。 此參數根據您的特定資料需求、壓縮設定和其他環境因素而定。

1. 若要修改參數，請瀏覽至 Hive [設定] 索引標籤，並尋找 [設定] 頁面上的 [各歸納器的資料] 參數。

    ![各歸納器的資料](./media/hdinsight-changing-configs-via-ambari/data-per-reducer.png)
 
1. 選取 [編輯] 將值修改為 128 MB (134,217,728 位元組)，然後按下 **Enter** 儲存。

    ![各歸納器的資料 - 已編輯](./media/hdinsight-changing-configs-via-ambari/data-per-reducer-edited.png)
  
    假設輸入大小為 1024 MB，而且各個歸納器有 128 MB 的資料，則共有八個歸納器 (1024/128)。

1. [各歸納器的資料] 參數的值不正確會導致歸納器的數目增加，而對於查詢效能造成負面影響。 若要限制歸納器數目上限，請將 `hive.exec.reducers.max` 設定為合適的值。 預設值是 1009。

### <a name="enable-parallel-execution"></a>啟用平行執行

在一個或多個階段中執行 Hive 查詢。 如果可以平行執行獨立階段，即可提升查詢效能。

1.  若要啟用平行查詢執行，請瀏覽至 Hive [設定] 索引標籤，並搜尋 `hive.exec.parallel` 屬性。 預設值為 False。 將值變更為 true，然後按下 **Enter** 儲存值。
 
1.  若要限制平行執行的作業數目，請修改 `hive.exec.parallel.thread.number` 屬性。 預設值為 8。

    ![Hive 執行平行](./media/hdinsight-changing-configs-via-ambari/hive-exec-parallel.png)


### <a name="enable-vectorization"></a>啟用向量化

Hive 會逐列處理資料。 向量化會指示 Hive 處理 1,024 列的資料，而非一次處理一個資料列。 向量化只適用於 ORC 檔案格式。

1. 若要啟用向量化查詢執行，請瀏覽至 Hive [設定] 索引標籤，並搜尋 `hive.vectorized.execution.enabled` 參數。 Hive 0.13.0 或更新版本的預設值為 true。
 
1. 若要對於查詢的歸納器端啟用向量化執行，將 `hive.vectorized.execution.reduce.enabled` 參數設定為 true。 預設值為 False。

    ![Hive 向量化執行](./media/hdinsight-changing-configs-via-ambari/hive-vectorized-execution.png)

### <a name="enable-cost-based-optimization-cbo"></a>啟用以成本為基礎的最佳化 (CBO)

Hive 預設會遵循一組規則，找出一個最佳的查詢執行計畫。 以成本為基礎的最佳化 (CBO) 會評估多個計畫來執行查詢，並將成本指派給每個計畫，然後定執行查詢的最低成本計畫。

若要啟用 CBO，請瀏覽至 Hive [設定] 索引標籤，並搜尋 `parameter hive.cbo.enable`，然後將切換按鈕切換為 [開]。

![CBO 設定](./media/hdinsight-changing-configs-via-ambari/cbo.png)

CBO 啟用時，下列的其他設定參數會提高 Hive 查詢效能：

* `hive.compute.query.using.stats`

    設定為 true 時，Hive 會使用儲存在其中繼存放區的統計資料，來回答類似 `count(*)` 的簡單查詢。

    ![CBO 統計資料](./media/hdinsight-changing-configs-via-ambari/hive-compute-query-using-stats.png)

* `hive.stats.fetch.column.stats`

    CBO 啟用時，會建立資料行統計資料。 Hive 會使用儲存在中繼存放區的資料將查詢最佳化。 資料行數目較高時，擷取每個資料行的資料行統計資料會花費較長的時間。 設定為 false 時，此設定會停用從中繼存放區擷取資料行統計資料。

    ![Hive 統計資料集合資料行統計資料](./media/hdinsight-changing-configs-via-ambari/hive-stats-fetch-column-stats.png)

* `hive.stats.fetch.partition.stats`

    基本的分割區統計資料 (例如資料列、資料大小和檔案大小) 儲存在中繼存放區。 設定為 true 時，會從中繼存放區擷取分割區統計資料。 若為 false，會從檔案系統擷取檔案大小，並且從資料列結構描述擷取資料列數目。

    ![Hive 統計資料集合分割區統計資料](./media/hdinsight-changing-configs-via-ambari/hive-stats-fetch-partition-stats.png)

### <a name="enable-intermediate-compression"></a>啟用中繼壓縮

對應工作會建立歸納器工作使用的中繼檔案。 中繼壓縮會壓縮中繼檔案大小。

Hadoop 工作通常出現 I/O 瓶頸。 壓縮資料可以加快 I/O 和整體網路傳輸。

可用的壓縮類型包括：

| 格式 | 工具 | 演算法 | 檔案副檔名 | 可分割？ |
| -- | -- | -- | -- | -- |
| Gzip | Gzip | DEFLATE | .gz | 否 |
| Bzip2 | Bzip2 | Bzip2 |.bz2 | 是 |
| LZO | Lzop | LZO | .lzo | 是，如果已編製索引 |
| Snappy | N/A | Snappy | Snappy | 否 |

一般而言，可分割的壓縮方法相當重要，否則只會建立極少量的對應程式。 如果輸入資料為文字，`bzip2` 是最佳選擇。 若為˙ ORC 格式，Snappy 是最快的壓縮選項。

1. 若要啟用中繼壓縮，請瀏覽至 Hive [設定] 索引標籤，然後將 `hive.exec.compress.intermediate` 參數設定為 true。 預設值為 False。

    ![Hive exec 壓縮中繼](./media/hdinsight-changing-configs-via-ambari/hive-exec-compress-intermediate.png)

    > [!NOTE]
    > 若要壓縮中繼檔案，即使轉碼器沒有高壓縮輸出，也請選擇較低 CPU 成本的壓縮轉碼器。

1. 若要設定中繼壓縮轉碼器，請將自訂屬性 `mapred.map.output.compression.codec` 新增至 `hive-site.xml` 或 `mapred-site.xml` 檔案。

1. 若要新增自訂設定：

    a. 瀏覽至 Hive [設定] 索引標籤，並選取 [進階] 索引標籤。

    b. 在 [進階] 索引標籤下，尋找並展開 [自訂 Hive 網站] 窗格。

    c. 按一下 [自訂 Hive 網站] 窗格底部的連結 [新增屬性]。

    d. 在 [新增屬性] 視窗中，輸入 `mapred.map.output.compression.codec` 做為索引鍵，並輸入 `org.apache.hadoop.io.compress.SnappyCodec` 做為值。

    e. 按一下 [新增] 。

    ![Hive 自訂屬性](./media/hdinsight-changing-configs-via-ambari/hive-custom-property.png)

    這會使用 Snappy 壓縮來壓縮中繼檔案。 新增屬性後，屬性會出現在 [自訂 Hive 網站] 窗格中。

    > [!NOTE]
    > 此程序會修改 `$HADOOP_HOME/conf/hive-site.xml` 檔案。

### <a name="compress-final-output"></a>壓縮最終輸出

最終 Hive 輸出也可壓縮。

1. 若要壓縮最終 Hive 輸出，請瀏覽至 Hive [設定] 索引標籤，然後將 `hive.exec.compress.output` 參數設定為 true。 預設值為 False。

1. 若要選擇輸出壓縮轉碼器，請將 `mapred.output.compression.codec` 自訂屬性新增至 [自訂 Hive 網站] 窗格，如上一節的步驟 3 中所述。

    ![Hive 自訂屬性](./media/hdinsight-changing-configs-via-ambari/hive-custom-property2.png)

### <a name="enable-speculative-execution"></a>啟用推測性執行

推測性執行會啟動特定數目的重複工作，以偵測緩慢執行的工作並列入封鎖清單，同時將個別工作的結果最佳化，藉以改善整體的工作執行效果。

對於有大量輸入的長時間執行 MapReduce 工作，不應該開啟推測性執行。

* 若要啟用推測性執行，請瀏覽至 Hive [設定] 索引標籤，然後將 `hive.mapred.reduce.tasks.speculative.execution` 參數設定為 true。 預設值為 False。

    ![Hive mapred 會減少推測性執行執行](./media/hdinsight-changing-configs-via-ambari/hive-mapred-reduce-tasks-speculative-execution.png)

### <a name="tune-dynamic-partitions"></a>微調動態分割區

Hive 能夠在將記錄插入資料表時建立動態分割區，不需要預先定義每個分割區。 這是相當強大的功能，不過可能會導致建立大量的分割區，且每個分割區都有大量的檔案。

1. 若要 Hive 執行動態分割，`hive.exec.dynamic.partition` 參數值應該為 true (預設值)。

1. 將動態分割模式變更為 [嚴格]。 在嚴格模式中，至少一個分割區必須為靜態。 這可防止在沒有分割區篩選器出現在 WHERE 子句的情況下進行查詢，也就是說，[嚴格] 會防止掃描所有分割區的查詢。 瀏覽至 Hive [設定] 索引標籤，然後將 `hive.exec.dynamic.partition.mode` 設定為 [嚴格]。 預設值為 **nonstrict**。
 
1. 若要限制建立的動態分割區數目，請修改 `hive.exec.max.dynamic.partitions` 參數。 預設值為 5000。
 
1. 若要限制每個節點的動態分割區總數，請修改 `hive.exec.max.dynamic.partitions.pernode`。 預設值為 2000。

### <a name="enable-local-mode"></a>啟用本機模式

本機模式可讓 Hive 執行單一電腦上一項作業的所有工作，有時可執行單一處理序中的所有工作。 如果輸入資料不大，而且啟動查詢工作的額外負荷可使用大量的整體查詢執行，則這可改善查詢效能。

若要啟用本機模式，請將 `hive.exec.mode.local.auto` 參數新增至 [自訂 Hive 網站] 面板，如「[啟用中繼壓縮](#enable-intermediate-compression)」一節的步驟 3 所述。

![Hive Exec 模式本機自動](./media/hdinsight-changing-configs-via-ambari/hive-exec-mode-local-auto.png)

### <a name="set-single-mapreduce-multigroup-by"></a>設定單一 MapReduce MultiGROUP BY

這個屬性設定為 true 時，一般群組依據索引鍵的 MultiGROUP BY 查詢會產生單一 MapReduce 工作。  

若要啟用此行為，請將 `hive.multigroupby.singlereducer` 參數新增至 [自訂 Hive 網站] 面板，如「[啟用中繼壓縮](#enable-intermediate-compression)」一節的步驟 3 所述。

![Hive 設定單一 MapReduce MultiGROUP BY](./media/hdinsight-changing-configs-via-ambari/hive-multigroupby-singlereducer.png)

### <a name="additional-hive-optimizations"></a>其他 Hive 最佳化

下列各節說明可以設定的其他 Hive 相關的最佳化。

#### <a name="join-optimizations"></a>聯結最佳化

Hive 的預設聯結類型是*隨機聯結*。 在 Hive 中，特殊對應程式會讀取輸入，並將聯結索引鍵/值組發送至中繼檔案。 Hadoop 會在隨機階段中將這些組排序並合併。 這個隨機階段需要相當高的成本。 依照您的資料選取正確的聯結可大幅提升效能。

| 聯結類型 | 當 | 方式 | Hive 設定 | 註解 |
| -- | -- | -- | -- | -- |
| 隨機聯結 | <ul><li>預設選擇</li><li>一律運作</li></ul> | <ul><li>讀取其中一個資料表的一部分</li><li>聯結索引鍵的貯體和排序</li><li>將一個貯體傳送至各項減少</li><li>在減少端完成聯結</li></ul> | 不需要特別的 Hive 設定 | 每次運作 |
| 對應聯結 | <ul><li>一個資料表可放入記憶體中</li></ul> | <ul><li>將小型資料表讀入記憶體雜湊表</li><li>透過大型檔案的一部分進行串流</li><li>聯結雜湊資料表中的每筆記錄</li><li>聯結由對應程式單獨完成</li></ul> | `hive.auto.confvert.join=true` | 非常快速，但有限 |
| 排序合併貯體 | 如果兩個資料表是： <ul><li>排序相同</li><li>貯體相同</li><li>對於排序/貯體資料行進行聯結</li></ul> | 每個處理序： <ul><li>讀取每個資料表中的貯體</li><li>處理最小值的資料列</li></ul> | `hive.auto.convert.sortmerge.join=true` | 非常有效率 |

#### <a name="execution-engine-optimizations"></a>執行引擎最佳化

最佳化 Hive 執行引擎的其他建議：

| 設定 | 建議 | HDInsight 預設 |
| -- | -- | -- |
| `hive.mapjoin.hybridgrace.hashtable` | True = 較安全、較慢；false = 較快 | false |
| `tez.am.resource.memory.mb` | 大部分適用的 4 GB 上限 | 自動調整 |
| `tez.session.am.dag.submit.timeout.secs` | 300+ | 300 |
| `tez.am.container.idle.release-timeout-min.millis` | 20000+ | 10000 |
| `tez.am.container.idle.release-timeout-max.millis` | 40000+ | 20000 |

## <a name="pig-optimization"></a>Pig 最佳化

可以從 Ambari Web UI 微調 Pig 查詢，以便修改 Pig 屬性。 直接從 Ambari 修改 Pig 屬性會修改 `/etc/pig/2.4.2.0-258.0/pig.properties` 檔案中的 Pig 屬性。

1. 若要修改 Pig 屬性，請瀏覽至 Pig [設定] 索引標籤，然後展開 [進階 pig 屬性] 窗格。

1. 尋找、取消註解並變更您想要修改的屬性值。

1. 選取視窗右上方的 [儲存] 儲存新值。 某些屬性可能需要服務重新啟動。

    ![進階 pig 屬性](./media/hdinsight-changing-configs-via-ambari/advanced-pig-properties.png)
 
> [!NOTE]
> 任何工作階段層級設定都會覆寫 `pig.properties` 檔案中的屬性值。

### <a name="tune-execution-engine"></a>調整執行引擎

兩個執行引擎可用來執行 Pig 指令碼：MapReduce 和 Tez。 Tez 是最佳化引擎，速度比 MapReduce 更快。

1. 若要修改執行引擎，請在 [進階 pig 屬性] 窗格中找出屬性 `exectype`。

1. 預設值為 **MapReduce**。 將它變更為 [Tez]。


### <a name="enable-local-mode"></a>啟用本機模式

和 Hive 類似，本機模式可用來加速資料量相對較小的作業。

1. 若要啟用本機模式，請將 `pig.auto.local.enabled` 設定為 **true**。 預設值為 False。

1. 輸入資料大小比 `pig.auto.local.input.maxbytes` 屬性值小的作業會被視為小型作業。 預設值為 1 GB。


### <a name="copy-user-jar-cache"></a>複製使用者 jar 快取

Pig 會將 UDF 需要的 JAR 檔案複製到分散式快取，以供工作節點使用。 這些 jar 不會經常變更。 如果啟用，`pig.user.cache.enabled` 設定可將 jar 放置在快取中，以供同一位使用者執行的工作重複使用。 這可小幅提升作業效能。

1. 若要啟用，請將 `pig.user.cache.enabled` 設定為 true。 預設值為 false。

1. 若要設定快取 jar 的基底路徑，請將 `pig.user.cache.location` 設定為基底路徑。 預設值為 `/tmp`。


### <a name="optimize-performance-with-memory-settings"></a>使用記憶體設定將效能最佳化

下列記憶體設定有助於將 Pig 指令碼效能最佳化。

* `pig.cachedbag.memusage`：配置於包的記憶體數量。 包是元組的集合。 元組是欄位的排序集合，欄位則是一段資料。 如果包中的資料超出配置的記憶體，則會溢出到磁碟。 預設值為 0.2，這表示可用記憶體的 20%。 這個記憶體由應用程式中的所有包共用。

* `pig.spill.size.threshold`：大於此溢出大小臨界值 (單位為位元組) 的包會溢出到磁碟。 預設值是 5 MB。


### <a name="compress-temporary-files"></a>壓縮暫存檔

Pig 會在作業執行期間產生暫存檔。 壓縮暫存檔會提升在磁碟中讀取或寫入檔案的效能。 下列設定可用來壓縮暫存檔。

* `pig.tmpfilecompression`：若為 true，啟用暫存檔壓縮。 預設值為 False。

* `pig.tmpfilecompression.codec`：壓縮轉碼器用來壓縮暫存檔。 為了降低 CPU 使用率，建議的壓縮轉碼器是 LZO 和 Snappy。

### <a name="enable-split-combining"></a>啟用分割合併

啟用時，小型檔案會合併，以減少對應工作。 對於包含大量小型檔案的作業，這可提升效率。 若要啟用，請將 `pig.noSplitCombination` 設定為 true。 預設值為 False。


### <a name="tune-mappers"></a>微調對應程式

修改屬性 `pig.maxCombinedSplitSize` 可控制對應程式的數目。 這會指定單一對應工作將處理的資料大小。 預設值是檔案系統的預設區塊大小。 增加這個值會減少對應程式工作的數目。


### <a name="tune-reducers"></a>微調歸納器

歸納器的數目是根據 `pig.exec.reducers.bytes.per.reducer` 參數計算而得。 該參數指定每個歸納器處理的位元組數目，預設為 1 GB。 若要限制歸納器數目上限，請設定 `pig.exec.reducers.max` 屬性，預設為 999。


## <a name="hbase-optimization-with-the-ambari-web-ui"></a>使用 Ambari Web UI 的 HBase 最佳化

從 [HBase 設定] 索引標籤可修改 HBase 設定。下列小節說明會影響 HBase 效能的一些重要組態設定。

### <a name="set-hbaseheapsize"></a>設定 HBASE_HEAPSIZE

HBase 堆積大小會指定*區域*和*主要*伺服器將使用的堆積最大數量 (單位為 MB)。 預設值是 1,000 MB。 應該對於叢集工作負載調整這個值。

1. 若要修改，請瀏覽至 HBase [設定] 索引標籤的 [進階 HBase-env] 窗格，然後尋找 `HBASE_HEAPSIZE` 設定。

1. 將預設值變更為 5,000 MB。

    ![HBASE_HEAPSIZE](./media/hdinsight-changing-configs-via-ambari/hbase-heapsize.png)


### <a name="optimize-read-heavy-workloads"></a>將大量讀取工作負載最佳化

下列設定對於改善大量讀取工作負載的效能很重要。

#### <a name="block-cache-size"></a>區塊快取大小

區塊快取是讀取快取。 其大小由 `hfile.block.cache.size` 參數控制。 預設值是 0.4，這是區域伺服器記憶體總計的 40%。 區塊快取大小愈大，隨機讀取愈快速。

1. 若要修改此參數，請瀏覽至 HBase [設定] 索引標籤的 [設定] 索引標籤，然後尋找 [配置於讀取緩衝區的 RegionServer 百分比]。

    ![HBase 區塊快取大小](./media/hdinsight-changing-configs-via-ambari/hbase-block-cache-size.png)
 
1. 若要變更該值，請選取 [編輯] 圖示。


#### <a name="memstore-size"></a>Memstore 大小

所有的編輯均儲存於稱為 *Memstore* 的記憶體緩衝區。 這會增加在單一作業中可以寫入磁碟的資料量總計，而且可加快最近編輯的後續存取速度。 Memstore 大小是由下列兩個參數所定義：

* `hbase.regionserver.global.memstore.UpperLimit`：定義合併的 Memstore 可使用的區域伺服器百分比上限。

* `hbase.regionserver.global.memstore.LowerLimit`：定義合併的 Memstore 可使用的區域伺服器百分比下限。

若要將隨機讀取最佳化，可以減少 Memstore 上限和下限。


#### <a name="number-of-rows-fetched-when-scanning-from-disk"></a>從磁碟掃描時擷取的資料列數目

`hbase.client.scanner.caching` 設定會定義在掃描器上呼叫 `next` 方法時，從磁碟讀取的資料列數目。  預設值是 100。 該數字愈高，從用戶端向區域伺服器進行的遠端呼叫愈少，因此可加快掃描。 不過，這也會增加用戶端的記憶體使用量。

![擷取資料列的 HBase 數目](./media/hdinsight-changing-configs-via-ambari/hbase-num-rows-fetched.png)

> [!IMPORTANT]
> 請勿將該值設定過高，以免造成掃描器引動 next 方法的相隔時間大於掃描程式逾時。 掃描器逾時持續期間是由 `hbase.regionserver.lease.period` 屬性所定義。


### <a name="optimize-write-heavy-workloads"></a>將大量寫入工作負載最佳化

下列設定對於改善大量寫入工作負載的效能很重要。


#### <a name="maximum-region-file-size"></a>區域檔案大小上限

HBase 會將資料儲存為稱為 *HFile* 的內部檔案格式。 屬性 `hbase.hregion.max.filesize` 會定義區域的單一 HFile 大小。  如果一個區域中的所有 HFiles 總和大於此設定，則一個區域會分割為兩個區域。
 
![HBase HRegion 最大檔案大小](./media/hdinsight-changing-configs-via-ambari/hbase-hregion-max-filesize.png)

區域檔案大小愈大，分割數目愈小。 您可以增加檔案大小，以判斷達到最大寫入效能的值。


#### <a name="avoid-update-blocking"></a>避免更新封鎖

* 屬性 `hbase.hregion.memstore.flush.size` 會定義 Memstore 排清至磁碟的大小。 預設大小為 128 MB。

* Hbase 區域區塊乘數是由 `hbase.hregion.memstore.block.multiplier` 定義。 預設值為 4。 允許的上限為 8。

* 如果 Memstore 是 (`hbase.hregion.memstore.flush.size` * `hbase.hregion.memstore.block.multiplier`) 位元組，HBase 區塊會更新。

    使用排清大小及區塊乘數的預設值時，在 Memstore 大小是 128 * 4 = 512 MB 的情況，將封鎖更新。 若要減少更新封鎖計數，請增加值 `hbase.hregion.memstore.block.multiplier`。

![HBase 區域區塊乘數](./media/hdinsight-changing-configs-via-ambari/hbase-hregion-memstore-block-multiplier.png)


### <a name="define-memstore-size"></a>定義 Memstore 大小

Memstore 大小是由 `hbase.regionserver.global.memstore.UpperLimit` 和 `hbase.regionserver.global.memstore.LowerLimit` 參數所定義。 設定這些彼此相等的值會減少寫入期間的暫停 (也會導致更頻繁清除)，並且可提升寫入效能。


### <a name="set-memstore-local-allocation-buffer"></a>設定 Memstore 本機配置緩衝區

Memstore 本機配置緩衝區的使用方式取決於屬性 `hbase.hregion.memstore.mslab.enabled`。 啟用 (true) 時，這可防止大量寫入作業期間的堆積分割。 預設值為 true。
 
![hbase.hregion.memstore.mslab.enabled](./media/hdinsight-changing-configs-via-ambari/hbase-hregion-memstore-mslab-enabled.png)


## <a name="next-steps"></a>後續步驟

* [使用 Ambari Web UI 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)
* [Ambari REST API](hdinsight-hadoop-manage-ambari-rest-api.md)
