---
title: 搭配 Azure HDInsight 使用互動式查詢
description: 了解如何搭配 HDInsight 使用互動式查詢 (Hive LLAP)。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/22/2018
ms.openlocfilehash: a90ec3102f3ce821193d58b6d14ca119f6d7e916
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46959677"
---
# <a name="use-interactive-query-with-hdinsight"></a>搭配 HDInsight 使用互動式查詢
互動式查詢 (又稱為 Hive LLAP 或[低延遲分析處理](https://cwiki.apache.org/confluence/display/Hive/LLAP)) 是一種 Azure HDInsight [叢集類型](../hdinsight-hadoop-provision-linux-clusters.md#cluster-types)。 互動式查詢支援記憶體內快取，可讓 Hive 查詢速度更快且更具互動性。

[!INCLUDE [hdinsight-price-change](../../../includes/hdinsight-enhancements.md)] 

互動式查詢叢集與 Hadoop 叢集不同。 它只包含 Hive 服務。 

> [!NOTE]
> 您只能透過 Ambari Hive 檢視、Beeline 和 Microsoft Hive 開放式資料庫連線驅動程式 (Hive ODBC)，存取互動式查詢叢集中的 Hive 服務。 您無法透過 Hive 主控台、Templeton、Azure 傳統 CLI 或 Azure PowerShell 來存取它。 
> 
> 

## <a name="create-an-interactive-query-cluster"></a>建立互動式查詢叢集
如需有關建立 HDInsight 叢集的資訊，請參閱[在 HDInsight 中建立 Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。 選擇互動式查詢叢集類型。

## <a name="execute-hive-queries-from-interactive-query"></a>從互動式查詢執行 Hive 查詢
若要執行 Hive 查詢，您有下列選項：

* 使用 Power BI

    請參閱[在 Azure HDInsight 中使用 Power BI 將互動式查詢 Hive 資料視覺化](./apache-hadoop-connect-hive-power-bi-directquery.md) 請參閱[在 Azure HDInsight 中使用 Power BI 將巨量資料視覺化](../hadoop/apache-hadoop-connect-hive-power-bi.md)。
 
* 使用 Zeppelin

    請參閱[使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢](../hdinsight-connect-hive-zeppelin.md)。

* 使用 Visual Studio

    請參閱[使用 Data Lake Tools for Visual Studio 連線至 Azure HDInsight 及執行 Hive 查詢](../hadoop/apache-hadoop-visual-studio-tools-get-started.md#run-interactive-hive-queries)。

* 使用 Visual Studio Code

    請參閱[使用適用於 Hive、LLAP 或 pySpark 的 Visual Studio Code](../hdinsight-for-vscode.md)。
* 使用 Ambari Hive 檢視執行 Hive。
  
    請參閱[在 Azure HDInsight 中搭配 Hadoop 使用 Hive 檢視](../hadoop/apache-hadoop-use-hive-ambari-view.md)。
* 使用 Beeline 執行 Hive。
  
    請參閱[利用 Beeline 搭配使用 Hive 與 HDInsight 中的 Hadoop](../hadoop/apache-hadoop-use-hive-beeline.md)。
  
    您可以從前端節點或空白邊緣節點使用 Beeline。 我們的建議是從空白邊緣節點使用 Beeline。 如需使用空白邊緣節點建立 HDInsight 叢集的詳細資訊，請參閱[在 HDInsight 中使用空白邊緣節點](../hdinsight-apps-use-edge-node.md)。
* 使用 Hive ODBC 執行 Hive。
  
    請參閱[使用 Microsoft Hive ODBC 驅動程式將 Excel 連接到 Hadoop](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)。

若要尋找 Java 資料庫連線 (JDBC) 連接字串：

1. 使用下列 URL 登入 Ambari： https://\<cluster name\>.AzureHDInsight.net。
2. 在左側功能表中，選取 **Hive**。
3. 若要複製 URL，選取剪貼簿圖示：
   
   ![HDInsight Hadoop 互動式查詢 LLAP JDBC](./media/apache-interactive-query-get-started/hdinsight-hadoop-use-interactive-hive-jdbc.png)

## <a name="next-steps"></a>後續步驟

* 了解如何[在 HDInsight 中建立互動式查詢叢集](../hdinsight-hadoop-provision-linux-clusters.md)。
* 了解如何[在 Azure HDInsight 中使用 Power BI 將巨量資料視覺化](../hadoop/apache-hadoop-connect-hive-power-bi.md)。
* 了解如何[使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢](../hdinsight-connect-hive-zeppelin.md)。
* 了解如何[使用 Data Lake Tools for Visual Studio 執行 Hive 查詢](../hadoop/apache-hadoop-visual-studio-tools-get-started.md#run-interactive-hive-queries)。
* 了解如何[使用 HDInsight Tools for Visual Studio Code](../hdinsight-for-vscode.md)。
* 了解如何[在 HDInsight 中搭配 Hadoop 使用 Hive 檢視](../hadoop/apache-hadoop-use-hive-ambari-view.md)
* 了解如何[使用 Beeline 在 HDInsight 中執行 Hive 查詢](../hadoop/apache-hadoop-use-hive-beeline.md)。
* 了解如何[使用 Microsoft Hive ODBC 驅動程式將 Excel 連線到 Hadoop](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)。

