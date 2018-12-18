---
title: 搭配 HDInsight 來使用 Ambari Tez 檢視 - Azure
description: 了解如何在 HDInsight 上使用 Ambari Tez 檢視來為 Tez 作業偵錯。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: 576460f4b68d670e534e0ddeed920f7ac99e1458
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43108881"
---
# <a name="use-ambari-views-to-debug-tez-jobs-on-hdinsight"></a>在 HDInsight 上使用 Ambari 檢視來為 Tez 作業偵錯

適用於 HDInsight 的 Ambari Web UI 包含 Tez 檢視，可用來了解使用 Tez 的作業，和為該類型作業偵錯。 Tez 檢視可讓您把作業視覺化有已連接項目的圖表、深入每個項目、取得統計資料，以及記錄資訊。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 元件版本設定](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="prerequisites"></a>必要條件

* 以 Linux 為基礎的 HDInsight 叢集。 如需建立叢集的步驟，請參閱[開始使用以 Linux 為基礎的 HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)。
* 支援 HTML5 的新式網頁瀏覽器。

## <a name="understanding-tez"></a>了解 Tez

Tez 是 Hadoop 中資料處理用的可延伸架構，資料處理的速度比傳統的 MapReduce 處理方式還要快。 而對於 Linux 型 HDInsight 叢集來說，Tez 是 Hive 的預設引擎。

Tez 會建立有向非循環圖 (DAG)，說明作業所需動作的順序。 個別的動作稱為頂點，它會執行整體作業的一部分。 而由端點所描述的實際工作 (Work) 執行程序稱為工作 (Task)，且可能會分散到叢集中的多個節點上。

### <a name="understanding-the-tez-view"></a>了解 Tez 檢視

Tez 檢視同時提供歷程記錄資訊和正在執行之處理序的資訊。 這項資訊顯示作業如何分散到叢集。 它也會顯示工作和頂點使用的計數器，以及與作業相關的錯誤資訊。 它可能會提供下列案例中的有用資訊：

* 監視長期執行的處理序、檢視對應進度，以及減少工作。
* 分析成功或失敗的處理序歷史資料，以便了解如何改進處理序，或是處理序失敗的原因。

## <a name="generate-a-dag"></a>產生 DAG

Tez 檢視只有在使用 Tez 引擎的作業目前正在執行，或先前曾執行過才會包含資料。 簡單的 Hive 查詢可不使用 Tez 進行解析。 複雜的查詢會執行諸如篩選、群組、排序、聯結等等 則需使用 Tez 引擎。

使用下列步驟來執行使用 Tez 的 Hive 查詢：

1. 在網頁瀏覽器中瀏覽至 https://CLUSTERNAME.azurehdinsight.net，其中 **CLUSTERNAME** 是 HDInsight 叢集的名稱。

2. 在頁面頂端的功能表中，選取 [檢視] 圖示。 此圖示看起來像一系列正方形。 在隨後出現的下拉式清單中，選取 [Hive 檢視]。

    ![選取 [Hive 檢視]](./media/hdinsight-debug-ambari-tez-view/selecthive.png)

3. 當 Hive 檢視載入之後，在查詢編輯器中貼上下列查詢，然後按一下 [執行]。

        select market, state, country from hivesampletable where deviceplatform='Android' group by market, country, state;

    當作業完成之後，您應該會在 [查詢處理程序結果] 區段看到輸出。 結果應該會與下列文字類似：

        market  state       country
        en-GB   Hessen      Germany
        en-GB   Kingston    Jamaica

4. 選取 [記錄] 索引標籤。您會看到類似下列文字的資訊：

        INFO : Session is already open
        INFO :

        INFO : Status: Running (Executing on YARN cluster with App id application_1454546500517_0063)

    請儲存 **App id** 的值，因為您會在下一節中使用它。

## <a name="use-the-tez-view"></a>使用 Tez 檢視

1. 在頁面頂端的功能表中，選取 [檢視] 圖示。 在隨後出現的下拉式清單中，選取 [Tez 檢視]。

    ![選取 [Tez 檢視]](./media/hdinsight-debug-ambari-tez-view/selecttez.png)

2. 當 Tez 檢視載入時，您會看到正在或曾經在叢集上執行的 Hive 查詢清單。

    ![所有 DAG](./media/hdinsight-debug-ambari-tez-view/tez-view-home.png)

3. 如果您只有一個項目，則它是您在上一節中執行的查詢。 如果您有多個項目，可以使用頁面頂端的欄位搜尋。

4. 選取 Hive 查詢的 [Query ID] \(查詢識別碼\)。 關於查詢的詳細資訊隨即顯示。

    ![DAG 詳細資料](./media/hdinsight-debug-ambari-tez-view/query-details.png)

5. 此頁面上的索引標籤可讓您檢視下列資訊：

    * **查詢詳細資料**：Hive 查詢的相關詳細資料。
    * **時間軸**：每個處理階段所耗費時間的相關資訊。
    * **設定**：此查詢使用的設定。

    從 [Query Details] \(查詢詳細資料\)，您可以使用連結來尋找和此查詢的 [Application] \(應用程式\) 或 [DAG] 相關的資訊。
    
    * [Application] \(應用程式\) 連結會顯示此查詢的 YARN 應用程式相關資訊。 您可以從此處存取 YARN 應用程式記錄檔。
    * [DAG] 連結會顯示此查詢的有向非循環圖相關資訊。 您可以從此處以圖形化的方式檢視 DAG。 您也可以尋找關於 DAG 內之頂點的資訊。

## <a name="next-steps"></a>後續步驟

既然您已了解如何使用 Tez 檢視，請深入了解 [在 HDInsight 上使用 Hive](hadoop/hdinsight-use-hive.md)。

如需 Tez 的詳細技術資訊，請參閱 [Hortonworks 的 Tez 頁面](http://hortonworks.com/hadoop/tez/)。

如需如何搭配 HDInsight 來使用 Ambari 的詳細資訊，請參閱 [使用 Ambari Web UI 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)
