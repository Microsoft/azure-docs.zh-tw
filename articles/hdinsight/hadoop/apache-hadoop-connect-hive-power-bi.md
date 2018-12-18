---
title: 在 Azure HDInsight 中使用 Power BI 將巨量資料視覺化
description: 了解如何使用 Microsoft Power BI 將 Azure HDInsight 所處理的 Hive 資料視覺化。
keywords: hdinsight,hadoop,hive,interactive query,interactive hive,LLAP,odbc
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: c24818f6b746754111540bae5fbf7f21d22c3a61
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041899"
---
# <a name="visualize-hive-data-with-microsoft-power-bi-using-odbc-in-azure-hdinsight"></a>在 Azure HDInsight 中使用 ODBC 藉由 Microsoft Power BI 將 Hive 資料視覺化

了解如何使用 ODBC 將 Microsoft Power BI 連線至 Azure HDInsight，然後將 Hive 資料視覺化。 

>[!IMPORTANT]
> 您可以利用 Hive ODBC 驅動程式，透過 Power BI Desktop 中的一般 ODBC 連接器執行匯入作業。 不過，對於 BI 工作負載則不建議使用此方式，因為 Hive 查詢引擎具有非互動式的本質。 為發揮其效能，[HDInsight 互動式查詢連接器](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)和 [HDInsight Spark 連接器](https://docs.microsoft.com/power-bi/spark-on-hdinsight-with-direct-connect)會是較好的選擇。

在本教學課程中，您可將資料從 hivesampletable Hive 資料表載入至 Power BI。 Hive 資料表包含某些行動電話使用量資料。 然後您可在世界地圖上繪製使用量資料：

![HDInsight Power BI 地圖報告](./media/apache-hadoop-connect-hive-power-bi/hdinsight-power-bi-visualization.png)

此資訊也適用於新的[互動式查詢](../interactive-query/apache-interactive-query-get-started.md)叢集類型。 若要了解如何使用直接查詢來連線到 HDInsight 互動式查詢，請參閱[在 Azure HDInsight 中使用直接查詢藉由 Microsoft Power BI 將互動式查詢 Hive 資料視覺化](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)。



## <a name="prerequisites"></a>必要條件
在閱讀本文之前，您必須有下列各項：

* **HDInsight 叢集**。 此叢集可以是含有 Hive 的 HDInsight 叢集或新發行的互動式查詢叢集。 如需建立叢集，請參閱[建立叢集](apache-hadoop-linux-tutorial-get-started.md#create-cluster)。
* **[Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/)**。 您可以從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=45331)下載一份。

## <a name="create-hive-odbc-data-source"></a>建立 Hive ODBC 資料來源

請參閱[建立 Hive ODBC 資料來源](apache-hadoop-connect-excel-hive-odbc-driver.md#create-hive-odbc-data-source)。

## <a name="load-data-from-hdinsight"></a>從 HDInsight 載入資料

所有 HDInsight 叢集都隨附 hivesampletable Hive 資料表。

1. 登入 Power BI Desktop。
2. 按一下 [首頁] 索引標籤，從 [外部資料] 功能區按一下 [取得資料]，然後選取 [更多]。

    ![HDInsight Power BI 開放式資料](./media/apache-hadoop-connect-hive-power-bi/hdinsight-power-bi-open-odbc.png)
3. 從 [取得資料] 窗格，按一下左邊的 [其他]，按一下右邊的 [ODBC]，然後按一下底部的 [連線]。
4. 在 [從 ODBC] 窗格中，選取您在上一節中建立的資料來源名稱，然後按一下 [確定]。
5. 從 [導覽器] 窗格，展開 [ODBC -> HIVE -> 預設值]，選取 [hivesampletable]，然後按一下 [載入]。

## <a name="visualize-data"></a>顯現資料

從上一個程序繼續進行。

1. 從 [視覺效果] 窗格中選取 [地圖]。  它是地球圖示。

    ![HDInsight Power BI 自訂報告](./media/apache-hadoop-connect-hive-power-bi/hdinsight-power-bi-customize.png)
2. 從 [欄位] 窗格選取 **country** 和 **devicemake**。 您可以看到地圖上繪製的資料。
3. 展開地圖。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用 Power BI 將 HDInsight 中的資料視覺化。  若要深入了解，請參閱下列文章：

* [使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢](./../hdinsight-connect-hive-zeppelin.md)。
* [使用 Microsoft Hive ODBC 驅動程式將 Excel 連線到 HDInsight](./apache-hadoop-connect-excel-hive-odbc-driver.md)。
* [使用 Power Query 將 Excel 連線到 Hadoop](apache-hadoop-connect-excel-power-query.md)。
* [使用 Data Lake Tools for Visual Studio 連線至 Azure HDInsight 及執行 Hive 查詢](apache-hadoop-visual-studio-tools-get-started.md)。
* [使用適用於 Visual Studio Code 的 Azure HDInsight 工具](../hdinsight-for-vscode.md)。
* [將資料上傳至 HDInsight](./../hdinsight-upload-data.md)。
