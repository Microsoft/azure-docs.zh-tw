---
title: 將 Azure HDInsight 中的 Hive 查詢最佳化
description: 了解如何在 HDInsight 中最佳化 Hadoop 的 Hive 查詢。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: jasonh
ms.openlocfilehash: 1fd3ff89fc8428f03d22f4aa195dabf0e988ef57
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106482"
---
# <a name="optimize-hive-queries-in-azure-hdinsight"></a>將 Azure HDInsight 中的 Hive 查詢最佳化

根據預設，Hadoop 叢集不會為了效能進行最佳化。 本文涵蓋幾個最常見 Hive 效能最佳化方法，您可將這些方法套用於我們的查詢。

## <a name="scale-out-worker-nodes"></a>相應放大背景工作節點

增加叢集中的背景工作節點數目，即可運用更多平行執行的對應器和歸納器。 在 HDInsight 中您有兩種方法可相應放大：

* 在佈建階段，您可以使用 Azure 入口網站、Azure PowerShell 或跨平台命令列介面來指定背景工作節點的數目。  如需詳細資訊，請參閱[建立 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)。 下列畫面顯示 Azure 入口網站上的背景工作節點組態：
  
    ![scaleout_1][image-hdi-optimize-hive-scaleout_1]
* 在執行階段，您可以也相應放大叢集，而不需重新一個叢集：

    ![scaleout_1][image-hdi-optimize-hive-scaleout_2]

如需 HDInsight 支援之各種虛擬機器的相關資訊，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)。

## <a name="enable-tez"></a>啟用 Tez

[Apache Tez](http://hortonworks.com/hadoop/tez/) 是 MapReduce 引擎的替代執行引擎：

![tez_1][image-hdi-optimize-hive-tez_1]

Tez 比較迅速，因為：

* **在 MapReduce 引擎中執行有向非循環圖 (DAG) 作為單一作業**。 DAG 要求每一組對應程式後面有一組歸納器。 這會導致多個 MapReduce 工作針對每個 Hive 查詢而分拆。 Tez 沒有此種條件約束，並可將複雜的 DAG 當作一項工作處理，因而將工作啟動的額外負荷降至最低。
* **避免不必要的寫入**。 由於 MapReduce 引擎中有多項作業針對相同的 Hive 查詢運作，每項作業的輸出都會寫入中繼資料的 HDFS。 Tez 可以將每個 Hive 查詢的工作數目降至最低，所以能夠避免不必要的寫入。
* **將啟動延遲最小化**。 Tez 會減少需要啟動的對應器數目，同時提升整個最佳化，因此較能夠將啟動延遲降到最低。
* **重複使用容器**。 Tez 會儘可能重複使用容器，確保減少因為啟動容器而產生的延遲。
* **連續最佳化技巧**。 習慣上，是在編譯階段進行最佳化。 但是有更多關於輸入的資訊可用，所以在執行階段進行最佳化比較理想。 Tez 會使用連續最佳化技巧，進一步在執行階段將計劃最佳化。

如需這些概念的詳細資訊，請參閱 [Apache TEZ](http://hortonworks.com/hadoop/tez/)。

在查詢的前面加上以下設定，即可啟用任何 Hive 查詢 Tez：

    set hive.execution.engine=tez;

Linux 的 HDInsight 叢集預設會啟用 Tez。


## <a name="hive-partitioning"></a>Hive 分割

I/O 作業是執行 Hive 查詢的主要效能瓶頸。 如果可以減少需要讀取的資料量，即可改善效能。 根據預設，Hive 查詢會掃描整個 Hive 資料表。 這很適合資料表掃描之類的查詢。 但是，對於只需要掃描少量資料的查詢 (例如具有篩選的查詢)，這種行為就會產生不必要的額外負荷。 Hive 分割可讓 Hive 查詢只存取 Hive 資料表中所需的資料量。

Hive 分割的實作方法是將未經處理的資料重新整理成新的目錄，而每個分割區都有自己的目錄 - 其中的分割區是由使用者定義。 下圖說明如何依據 *年度*資料行來分割 Hive 資料表。 每年都會建立新的目錄。

![分割][image-hdi-optimize-hive-partitioning_1]

一些分割考量：

* **請勿分割不足** - 依據只有少數幾個值的資料行進行分割，可能會造成很少的分割區。 例如，依據性別進行分割，只會建立兩個分割區 (男性和女性)，因此只會降低最多一半的延遲。
* **請勿過度分割** - 另一方面，在具有唯一值 (例如，使用者識別碼) 的資料行建立分割區會造成多個分割區。 過度分割會在叢集 namenode 上造成太多壓力，因為它必須處理大量目錄。
* **避免資料扭曲** - 明智地選擇分割索引鍵，讓所有分割區的大小平均。 例如，依據 *州* 進行分割可能造成加州的記錄數目幾乎是佛蒙特州的記錄數目的 30 倍 (因為人口差異)。

若要建立分割資料表，請使用 *Partitioned By* 子句：

    CREATE TABLE lineitem_part
        (L_ORDERKEY INT, L_PARTKEY INT, L_SUPPKEY INT,L_LINENUMBER INT,
         L_QUANTITY DOUBLE, L_EXTENDEDPRICE DOUBLE, L_DISCOUNT DOUBLE,
         L_TAX DOUBLE, L_RETURNFLAG STRING, L_LINESTATUS STRING,
         L_SHIPDATE_PS STRING, L_COMMITDATE STRING, L_RECEIPTDATE            STRING, L_SHIPINSTRUCT STRING, L_SHIPMODE STRING,
         L_COMMENT STRING)
    PARTITIONED BY(L_SHIPDATE STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    STORED AS TEXTFILE;

建立分割資料表後，您可以建立靜態分割或動態分割。

* **靜態分割** 表示您已在適當的目錄中將資料分區，而且可以手動要求以目錄位置為基礎的 Hive 分割區。 下列範例為程式碼片段。
  
        INSERT OVERWRITE TABLE lineitem_part
        PARTITION (L_SHIPDATE = ‘5/23/1996 12:00:00 AM’)
        SELECT * FROM lineitem 
        WHERE lineitem.L_SHIPDATE = ‘5/23/1996 12:00:00 AM’
  
        ALTER TABLE lineitem_part ADD PARTITION (L_SHIPDATE = ‘5/23/1996 12:00:00 AM’))
        LOCATION ‘wasb://sampledata@ignitedemo.blob.core.windows.net/partitions/5_23_1996/'
* **動態分割** 表示您要 Hive 為您自動建立分割區。 我們已從暫存資料表建立分割資料表，所以我們只需要將資料插入至分割資料表：
  
        SET hive.exec.dynamic.partition = true;
        SET hive.exec.dynamic.partition.mode = nonstrict;
        INSERT INTO TABLE lineitem_part
        PARTITION (L_SHIPDATE)
        SELECT L_ORDERKEY as L_ORDERKEY, L_PARTKEY as L_PARTKEY , 
             L_SUPPKEY as L_SUPPKEY, L_LINENUMBER as L_LINENUMBER,
              L_QUANTITY as L_QUANTITY, L_EXTENDEDPRICE as L_EXTENDEDPRICE,
             L_DISCOUNT as L_DISCOUNT, L_TAX as L_TAX, L_RETURNFLAG as           L_RETURNFLAG, L_LINESTATUS as L_LINESTATUS, L_SHIPDATE as           L_SHIPDATE_PS, L_COMMITDATE as L_COMMITDATE, L_RECEIPTDATE as      L_RECEIPTDATE, L_SHIPINSTRUCT as L_SHIPINSTRUCT, L_SHIPMODE as      L_SHIPMODE, L_COMMENT as L_COMMENT, L_SHIPDATE as L_SHIPDATE FROM lineitem;

如需詳細資訊，請參閱[資料分割資料表](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables) \(英文\)。

## <a name="use-the-orcfile-format"></a>使用 ORCFile 格式
Hive 支援不同的檔案格式。 例如︰

* **文字**：這是預設檔案格式，適用於大部分的案例
* **Avro**：適用於互通性案例
* **ORC/Parquet**：最適合處理效能

ORC (最佳化的資料列單欄式) 格式是儲存 Hive 資料的高效率方式。 相較於其他格式，ORC 具有下列優點：

* 支援複雜的類型，包括 DateTime 和複雜和半結構化類型
* 高達 70% 的壓縮
* 每 10,000 個資料列的索引可讓您略過一些資料列
* 執行階段的執行大幅減少

若要啟用 ORC 格式，請先使用子句 *Stored as ORC*建立資料表：

    CREATE TABLE lineitem_orc_part
        (L_ORDERKEY INT, L_PARTKEY INT,L_SUPPKEY INT, L_LINENUMBER INT,
         L_QUANTITY DOUBLE, L_EXTENDEDPRICE DOUBLE, L_DISCOUNT DOUBLE,
         L_TAX DOUBLE, L_RETURNFLAG STRING, L_LINESTATUS STRING,
         L_SHIPDATE_PS STRING, L_COMMITDATE STRING, L_RECEIPTDATE STRING,
         L_SHIPINSTRUCT STRING, L_SHIPMODE STRING, L_COMMENT      STRING)
    PARTITIONED BY(L_SHIPDATE STRING)
    STORED AS ORC;

接著，將資料從暫存資料表插入至 ORC 資料表。 例如︰

    INSERT INTO TABLE lineitem_orc
    SELECT L_ORDERKEY as L_ORDERKEY, 
           L_PARTKEY as L_PARTKEY , 
           L_SUPPKEY as L_SUPPKEY,
           L_LINENUMBER as L_LINENUMBER,
            L_QUANTITY as L_QUANTITY, 
           L_EXTENDEDPRICE as L_EXTENDEDPRICE,
           L_DISCOUNT as L_DISCOUNT,
           L_TAX as L_TAX,
           L_RETURNFLAG as L_RETURNFLAG,
           L_LINESTATUS as L_LINESTATUS,
           L_SHIPDATE as L_SHIPDATE,
           L_COMMITDATE as L_COMMITDATE,
           L_RECEIPTDATE as L_RECEIPTDATE, 
           L_SHIPINSTRUCT as L_SHIPINSTRUCT,
           L_SHIPMODE as L_SHIPMODE,
           L_COMMENT as L_COMMENT
    FROM lineitem;

您可以在 [這裡](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC)進一步了解 ORC 格式。

## <a name="vectorization"></a>向量化

向量化可讓 Hive 以批次方式同時處理 1024 個資料列，而不是一次處理一個資料列。 這表示，因為需要執行的內部程式碼較少，所以簡單的作業會更快完成。

若要啟用向量化，請在 Hive 查詢的前面加上以下列設定：

    set hive.vectorized.execution.enabled = true;

如需詳細資訊，請參閱 [向量化查詢執行](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)。

## <a name="other-optimization-methods"></a>其他最佳化方法
您有更多最佳化方法可以考慮，例如：

* **Hive 值區：** 能將大型資料集叢集化或分段以最佳化查詢效能的技術。
* **聯結最佳化：** Hive 的查詢執行計劃最佳化，可改善聯結的效率並減少使用者提示的需求。 如需詳細資訊，請參閱 [聯結最佳化](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+JoinOptimization#LanguageManualJoinOptimization-JoinOptimization)。
* **增加歸納器**。

## <a name="next-steps"></a>後續步驟
在本文中，您學到幾種常見的 Hive 查詢最佳化方法。 若要深入了解，請參閱下列文章：

* [在 HDInsight 中使用 Apache Hive](hadoop/hdinsight-use-hive.md)
* [在 HDInsight 中使用 Hive 分析航班延誤資料](hdinsight-analyze-flight-delay-data.md)
* [在 HDInsight 中使用 Hive 分析 Twitter 資料](hdinsight-analyze-twitter-data.md)
* [在 HDInsight 的 Hadoop 上使用 Hive 查詢主控台分析感應器資料](hadoop/apache-hive-analyze-sensor-data.md)
* [使用 HDInsight 上的 Hive 分析網站的記錄](hadoop/apache-hive-analyze-website-log.md)

[image-hdi-optimize-hive-scaleout_1]: ./media/hdinsight-hadoop-optimize-hive-query/scaleout_1.png
[image-hdi-optimize-hive-scaleout_2]: ./media/hdinsight-hadoop-optimize-hive-query/scaleout_2.png
[image-hdi-optimize-hive-tez_1]: ./media/hdinsight-hadoop-optimize-hive-query/tez_1.png
[image-hdi-optimize-hive-partitioning_1]: ./media/hdinsight-hadoop-optimize-hive-query/partitioning_1.png
