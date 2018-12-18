---
title: 安裝並使用 HDInsight (Hadoop) 上的 Giraph
description: 在本主題中，您將學習如何使用指令碼動作在以 Linux 為基礎的 HDInsight 叢集上安裝 Giraph。 透過變更叢集組態或自訂安裝服務與公用程式，指令碼動作可讓您在叢集建立期間自訂叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: jasonh
ms.openlocfilehash: f5d7a5587d47f7601f8dc3f65318a6b7d486f58e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953288"
---
# <a name="install-giraph-on-hdinsight-hadoop-clusters-and-use-giraph-to-process-large-scale-graphs"></a>在 HDInsight Hadoop 叢集上安裝 Giraph，以及使用 Giraph 來處理大規模圖形

了解如何在 HDInsight 叢集上安裝 Apache Giraph。 HDInsight 的指令碼動作功能可讓您執行 bash 指令碼來自訂叢集。 在叢集建立期間和之後，可利用指令碼來自訂叢集。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="whatis"></a>什麼是 Giraph

[Apache Giraph](http://giraph.apache.org/) 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。 圖形可以為物件之間的關聯性建立模型。 例如，大型網路 (例如網際網路) 上各路由器之間的連線，或社交網路上的人際關係。 圖形處理可讓您分析圖形中物件之間的關聯，例如：

* 根據目前的人際關係找出可能的朋友。

* 識別網路中兩台電腦之間的最短路線。

* 計算網頁的頁面排名。

> [!WARNING]
> 完全支援 HDInsight 叢集隨附的元件 - Microsoft 支援服務會協助釐清與解決這些元件的相關問題。
>
> 自訂元件 (例如 Giraph) 則獲得商務上合理的支援，協助您進一步對問題進行疑難排解。 Microsoft 支援服務可能有能力解決問題。 如果無法解決，則您必須諮詢開放原始碼社群，以尋求該項技術的深厚專業知識。 例如，有許多社群網站可供使用，例如：[MSDN 的 HDInsight 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=hdinsight) \(英文\)、[http://stackoverflow.com](http://stackoverflow.com) \(英文\)。 此外，Apache 專案在 [http://apache.org](http://apache.org) 上也有專案網站，例如 [Hadoop](http://hadoop.apache.org/)。


## <a name="what-the-script-does"></a>指令碼會執行哪些作業

此指令碼會執行下列動作：

* 將 Giraph 安裝至 `/usr/hdp/current/giraph`

* 將 `giraph-examples.jar` 檔案複製到您的叢集的預設儲存體 (WASB)：`/example/jars/giraph-examples.jar`

## <a name="install"></a>使用指令碼動作安裝 Giraph

在 HDInsight 叢集上安裝 Giraph 的範例指令碼位於下列位置：

    https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh

本節提供有關如何在使用 Azure 入口網站建立叢集時使用範例指令碼的指示。

> [!NOTE]
> 您可以使用下列任何方法套用指令碼動作︰
> * Azure PowerShell
> * Azure 傳統 CLI
> * HDInsight .NET SDK
> * Azure 資源管理員範本
> 
> 您也可以將指令碼動作套用到執行中的叢集上。 如需詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

1. 使用 [建立以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)中的步驟開始建立叢集，但是不完成建立。

2. 在 [選用組態] 區段中，選取 [指令碼動作]，並提供下列資訊：

   * **名稱**：輸入指令碼動作的易記名稱。

   * **指令碼 URI**： https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh

   * **HEAD**：勾選此項目

   * **WORKER**：保持不勾選此項目

   * **ZOOKEEPER**：保持不勾選此項目

   * **參數**：將此欄位保留空白

3. 在 [指令碼動作] 底部，使用 [選取] 按鈕以儲存組態。 最後，使用 [選用組態] 區段底部的 [選取] 按鈕，儲存選用組態資訊。

4. 繼續如 [建立以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)中所述建立叢集。

## <a name="usegiraph"></a>如何在 HDInsight 中使用 Giraph？

建立叢集之後，請使用下列步驟執行 Giraph 隨附的 SimpleShortestPathsComputation 範例。 此範例會使用基本 [Pregel](http://people.apache.org/~edwardyoon/documents/pregel.pdf) 實作，在圖表中找出物件之間的最短路徑。

1. 使用 SSH 連線到 HDInsight 叢集

    ```bash
    ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    ```

    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 使用以下命令建立名為 **tiny_graph.txt** 的檔案：

    ```bash
    nano tiny_graph.txt
    ```

    使用下列文字做為此檔案的內容：

    ```text
    [0,0,[[1,1],[3,3]]]
    [1,0,[[0,1],[2,2],[3,1]]]
    [2,0,[[1,2],[4,4]]]
    [3,0,[[0,3],[1,1],[4,4]]]
    [4,0,[[3,4],[2,4]]]
    ```

    此資料使用 `[source_id, source_value,[[dest_id], [edge_value],...]]` 格式，描述有向圖形中物件之間的關聯。 每一行代表 `source_id` 物件和一或多個 `dest_id` 物件之間的關聯性。 `edge_value` 可以視為 `source_id` 和 `dest\_id` 之間的連線強度或距離。

    使用值 (或權數) 當作物件之間的距離繪製出來後，資料可能如下圖所示：

    ![tiny_graph.txt drawn as circles with lines of varying distance between](./media/hdinsight-hadoop-giraph-install-linux/giraph-graph.png)

3. 若要儲存檔案，依序使用 **Ctrl+X**、**Y**，最後按 **Enter** 以接受檔案名稱。

4. 使用下列項目以將資料儲存至您的 HDInsight 叢集的主要儲存體：

    ```bash
    hdfs dfs -put tiny_graph.txt /example/data/tiny_graph.txt
    ```

5. 使用下列命令執行 SimpleShortestPathsComputation 範例：

    ```bash
    yarn jar /usr/hdp/current/giraph/giraph-examples.jar org.apache.giraph.GiraphRunner org.apache.giraph.examples.SimpleShortestPathsComputation -ca mapred.job.tracker=headnodehost:9010 -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vip /example/data/tiny_graph.txt -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op /example/output/shortestpaths -w 2
    ```

    下表說明與此命令搭配使用的參數：

   | 參數 | 作用 |
   | --- | --- |
   | `jar` |包含範例的 jar 檔案。 |
   | `org.apache.giraph.GiraphRunner` |用於啟動範例的類別。 |
   | `org.apache.giraph.examples.SimpleShortestPathsCoputation` |所使用的範例。 在此範例中，它會計算圖表中識別碼 1 和所有其他識別碼之間的最短路徑。 |
   | `-ca mapred.job.tracker` |叢集的前端節點。 |
   | `-vif` |用於輸入資料的輸入格式。 |
   | `-vip` |輸入資料檔案。 |
   | `-vof` |輸出格式。 在此範例中，識別碼和值是純文字。 |
   | `-op` |輸出位置。 |
   | `-w 2` |要使用的背景工作角色數目。 在此範例中是 2。 |

    如需這些項目以及與 Giraph 範例搭配使用之其他參數的詳細資訊，請參閱 [Giraph 快速入門](http://giraph.apache.org/quick_start.html)。

6. 一旦作業完成，結果會儲存在 **/example/out/shotestpaths** 目錄中。 輸出檔的名稱會以 **part-m-** 開頭，結束的數字表示是第一個、第二個檔案，依此類推。 使用下列命令來檢視輸出：

    ```bash
    hdfs dfs -text /example/output/shortestpaths/*
    ```

    此輸出看起來類似下列文字：

        0    1.0
        4    5.0
        2    2.0
        1    0.0
        3    1.0

    SimpleShortestPathComputation 範例已刻意設計成從物件識別碼 1 開始，尋找前往其他物件的最短路徑。 輸出的格式為 `destination_id` 和 `distance`。 `distance` 是物件識別碼 1 與目標識別碼之間經過之邊緣的值 (或權數)。

    將此資料視覺化之後，您可以走過識別碼 1 和其他所有物件之間的最短路徑來驗證結果。 識別碼 1 和識別碼 4 之間的最短路徑為 5。 此值為<span style="color:orange">識別碼 1 和 3</span> 之間加上<span style="color:red">識別碼 3 和 4</span> 之間的總距離。

    ![Drawing of objects as circles with shortest paths drawn between](./media/hdinsight-hadoop-giraph-install-linux/giraph-graph-out.png)

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 叢集上安裝及使用 Hue](hdinsight-hadoop-hue-linux.md)。

* [在 HDInsight 叢集上安裝 Solr](hdinsight-hadoop-solr-install-linux.md)。
