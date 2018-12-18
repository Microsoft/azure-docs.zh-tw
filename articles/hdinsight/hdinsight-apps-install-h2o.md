---
title: 安裝已發佈的應用程式 - H2O Sparkling Water - Azure HDInsight
description: 安裝及使用 H2O Sparkling Water 第三方 Hadoop 應用程式。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: ashish
ms.openlocfilehash: e5a9505c41c14016768a5da42f9ac1836240b98f
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43095013"
---
# <a name="install-published-application---h2o-sparkling-water"></a>安裝已發佈的應用程式 - H2O Sparkling Water

本文說明如何在 Azure HDInsight 上安裝及執行 [H20 Sparkling Water](http://www.h2o.ai/) 已發佈的 Hadoop 應用程式。 如需 HDInsight 應用程式平台的概觀，以及可用獨立軟體廠商 (ISV) 已發佈的應用程式清單，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。 如需您自己的應用程式的安裝指示，請參閱[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。

## <a name="about-h2o-sparkling-water"></a>關於 H2O Sparkling Water

H20 Sparkling Water 是開放原始碼形式的完全分散式記憶體內部機器學習平台，具有線性延展性。 H2O Sparkling Water 可讓您將快速且可調整的 H2O 機器學習演算法與 Spark 功能合併。 使用者可以使用 Sparkling Water 利用 H2O Flow UI 從 Scala、R 和 Python 驅動計算。

H2O Sparkling Water 提供：

* **方便使用的 WebUI 和熟悉的介面** – 使用 H2O 的直覺式 Web 型 Flow GUI 或程式設計環境 (例如 R、Python、Java、Scala、JSON 和 H2O API)，快速地設定及開始使用。
* **適用於所有常見資料庫和檔案類型的不限資料支援** – 輕鬆地從 Microsoft Excel、R Studio、Tableau 等工具探索巨量資料並建構其模型。 從 HDFS、S3、SQL 和 NoSQL 資料來源連線到資料。
* **可大幅調整的巨量資料改寫和分析** – H2O Big Joins 的執行速度比 R data.table 作業快上 7 倍，並且可以線性方式調整至 100 億 x 100 億個資料列聯結。
* **即時資料評分** – 使用普通 Java 物件 (Plain-Old Java Objects, POJO)、模型最佳化 Java 物件 (Model-Optimized Java Objects, MOJO) 或 H2O REST API，將模型快速部署至生產環境。

### <a name="resource-links"></a>資源連結

* [H2O.ai 工程藍圖](http://jira.h2o.ai/)
* [H2O.ai 首頁](http://www.h2o.ai/)
* [H2O.ai 文件](http://docs.h2o.ai/)
* [H2O.ai 支援](https://support.h2o.ai/)
* [H2O.ai 開放原始碼程式碼基底](https://github.com/h2oai/)

## <a name="prerequisites"></a>必要條件

若要在新的 HDInsight 叢集或現有叢集上安裝此應用程式，您必須具有下列設定：

* 叢集層：標準或進階
* 叢集類型：Spark
* 叢集版本：3.5 或 3.6

## <a name="install-the-h2o-sparkling-water-published-application"></a>安裝 H2O Sparkling Water 已發佈的應用程式

如需有關安裝這個 ISV 應用程式和其他可用 ISV 應用程式的逐步指示，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。

## <a name="launch-h2o-sparkling-water"></a>啟動 H2O Sparkling Water

1. 安裝之後，您可以藉由開啟 Jupyter Notebook (`https://<ClusterName>.azurehdinsight.net/jupyter`)，從 Azure 入口網站中的叢集開始使用 H2O Sparkling Water (h2o-sparklingwater)。 您也可以藉由選取入口網站中叢集窗格的 [叢集儀表板]，然後選取 [Jupyter Notebook]，以移至 Jupyter。 系統會提示您輸入認證。 輸入在建立叢集時所指定的叢集 Hadoop 認證。

2. 在 Jupyter 中，您會看到三個資料夾：H2O-PySparkling-Examples、PySpark Examples 和 Scala Examples。 選取 [H2O-PySparkling-Examples] 資料夾。

    ![Jupyter Notebook 首頁](./media/hdinsight-apps-install-h2o/jupyter-home.png)

3. 建立新 Notebook 的第一個步驟是設定 Spark 環境。 此資訊包含在 **Sentiment_analysis_with_Sparkling_Water** 範例中。 設定 Spark 環境時，請務必使用正確的 jar，並指定第一個資料格之輸出所提供的 IP 位址。

    ![Jupyter Notebook 首頁](./media/hdinsight-apps-install-h2o/spark-config.png)

4. 啟動 H2O 叢集。

    ![啟動叢集](./media/hdinsight-apps-install-h2o/start-cluster.png)

5. H2O 叢集啟動並執行之後，藉由前往 **`https://<ClusterName>-h2o.apps.azurehdinsight.net:443`** 來開啟 H2O Flow。

    > [!NOTE]
    > 如果您無法開啟 H2O Flow，請嘗試清除瀏覽器快取。 如果您仍然無法與它連線，可能是叢集上沒有足夠的資源。 請嘗試在叢集窗格中的 [調整叢集] 選項底下增加背景工作角色節點的數目。

    ![H2O Flow 儀表板](./media/hdinsight-apps-install-h2o/h2o-flow.png)

6. 從右邊的功能表選取 [Million_Songs.flow] 範例。 出現警告提示時，按一下 [載入 Notebook]。 根據設計，此示範能在幾分鐘內就使用實際資料來執行。 其目標是要使用二元分類，從資料中預測歌曲是在 2004 年之前還是之後發行的。

    ![選取 Million_Songs.flow](./media/hdinsight-apps-install-h2o/million-songs.png)

7. 尋找包含 **milsongs-cls-train.csv.gz** 的路徑，並且以 **https://h2o-public-test-data.s3.amazonaws.com/bigdata/laptop/milsongs/milsongs-cls-train.csv.gz** 取代整個路徑。

8. 尋找包含 **milsongs-cls-test.csv.gz** 的路徑，並且以 **https://h2o-public-test-data.s3.amazonaws.com/bigdata/laptop/milsongs/milsongs-cls-test.csv.gz** 加以取代。

9. 若要執行 Notebook 資料格內的所有陳述式，請選取工具列上的 [全部執行] 按鈕。

    ![全部執行](./media/hdinsight-apps-install-h2o/run-all.png)

10. 幾分鐘之後，您應該就會看到如下所示的輸出。

    ![輸出](./media/hdinsight-apps-install-h2o/output.png)

就這麼簡單！ 您已經在短短幾分鐘內駕馭 Spark 中的人工智慧。 您現在可以探索 H2O Flow 中的更多範例，這些範例會示範不同類型的機器學習演算法。

## <a name="next-steps"></a>後續步驟

* [H2O 文件](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/index.html)
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
* [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)︰了解如何使用空白邊緣節點來存取 HDInsight 叢集，以及測試和裝載 HDInsight 應用程式。
