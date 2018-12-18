---
title: 搭配使用 Hive 與 Data Lake (Hadoop) Tools for Visual Studio - Azure HDInsight
description: 了解如何使用 Data Lake Tools for Visual Studio 在 Azure HDInsight 上搭配 Apache Hadoop 執行 Apache Hive 查詢。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: fd57e713a24fb83e42d46b4a1978530b706bf583
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049891"
---
# <a name="run-hive-queries-using-the-data-lake-tools-for-visual-studio"></a>使用 Data Lake Tools for Visual Studio 執行 Hive 查詢

了解如何使用 Data Lake Tools for Visual Studio 查詢 Apache Hive。 Data Lake Tools 可讓您在 Azure HDInsight 上輕鬆地建立、提交和監視對 Hadoop 的 Hive 查詢。

## <a id="prereq"></a>必要條件

* Azure HDInsight (HDInsight 上的 Hadoop) 叢集

  > [!IMPORTANT]
  > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* Visual Studio (下列其中一個版本)：

    * Visual Studio 2013 Community/Professional/Premium/Ultimate，含 Update 4

    * Visual Studio 2015 (任何版本)

    * Visual Studio 2017 (任何版本)

* HDInsight tools for Visual Studio 或 Azure Data Lake tools for Visual Studio。 如需有關安裝和設定工具的資訊，請參閱 [開始使用 Visual Studio Hadoop Tools for HDInsight](apache-hadoop-visual-studio-tools-get-started.md) 。

## <a id="run"></a> 使用 Visual Studio 執行 Hive 查詢

1. 開啟 **Visual Studio**，然後選取 [新增]  >  [專案]  >  [Azure Data Lake]  >  [HIVE]  >  [Hive 應用程式]。 提供此專案的名稱。

2. 開啟使用此專案所建立的 **Script.hql** 檔案，並貼入下列 HiveQL 陳述式中：

   ```hiveql
   set hive.execution.engine=tez;
   DROP TABLE log4jLogs;
   CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
   ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
   STORED AS TEXTFILE LOCATION '/example/data/';
   SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND  INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
   ```

    這些陳述式會執行下列動作：

   * `DROP TABLE`︰如果資料表存在，此陳述式將刪除它。

   * `CREATE EXTERNAL TABLE`在 Hive 中建立新的「外部」資料表。 外部資料表只會在 Hive 中儲存資料表定義 (資料會保留在原始位置)。

     > [!NOTE]
     > 當您預期會由外部來源來更新基礎資料時，請使用外部資料表。 例如，MapReduce 工作或 Azure 服務。
     >
     > 捨棄外部資料表並 **不會** 刪除資料，只會刪除資料表定義。

   * `ROW FORMAT`：告訴 Hive 如何格式化資料。 在此情況下，每個記錄中的欄位會以空格隔開。

   * `STORED AS TEXTFILE LOCATION`：將資料的儲存位置告訴 Hive (example/data 目錄)，且資料儲存為文字。

   * `SELECT`：選擇其資料欄 `t4` 包含值 `[ERROR]` 的所有資料列計數。 這個陳述式會傳回值 `3`，因為有三個資料列包含此值。

   * `INPUT__FILE__NAME LIKE '%.log'` - 告訴 Hive 我們只應該從檔名以 log 結尾的檔案中傳回資料。 這個子句會將搜尋限制為包含資料的 sample.log 檔案。

3. 從工具列中，選取您想要用於此查詢的 **Hdinsight 叢集**。 選取**提交**以 Hive 作業形式執行陳述式。

   ![提交列](./media/apache-hadoop-use-hive-visual-studio/toolbar.png)

4. [Hive 工作摘要] 將會出現並顯示執行中工作的相關資訊。 使用 [重新整理] 連結來重新整理工作資訊，直到 [工作狀態] 變更為 [已完成] 為止。

   ![顯示已完成作業的作業摘要](./media/apache-hadoop-use-hive-visual-studio/jobsummary.png)

5. 使用 [ **工作輸出** ] 連結檢視此工作的輸出。 它會顯示 `[ERROR] 3`，這是此查詢所傳回的值。

6. 您也可以執行 Hive 查詢，而不需建立專案。 使用 [伺服器總管]，展開 [Azure] > [HDInsight]，在 HDInsight 伺服器上按一下滑鼠右鍵，然後選取 [撰寫 Hive 查詢]。

7. 在出現的 **temp.hql** 文件，新增下列 HiveQL 陳述式：

   ```hiveql
   set hive.execution.engine=tez;
   CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
   INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
   ```

    這些陳述式會執行下列動作：

   * `CREATE TABLE IF NOT EXISTS`：建立資料表 (如果不存在)。 因為未使用 `EXTERNAL` 關鍵字，這個陳述式會建立內部資料表。 內部資料表儲存在 Hive 資料倉儲中，並受到 Hive 所管理。

     > [!NOTE]
     > 與 `EXTERNAL` 資料表不同之處在於，捨棄內部資料表也會刪除基礎資料。

   * `STORED AS ORC`：以最佳化資料列單欄式 (Optimized Row Columnar, ORC) 格式儲存資料。 ORC 是高度最佳化且有效率的 Hive 資料儲存格式。

   * `INSERT OVERWRITE ... SELECT`︰從含有 `[ERROR]`的 `log4jLogs` 資料表選取資料列，然後將資料插入 `errorLogs` 資料表。

8. 從工具列中，選取 [ **提交** ] 來執行工作。 使用 [ **工作狀態** ] 來判斷工作是否已順利完成。

9. 若要確認作業已建立資料表，請使用 [伺服器總管] 並展開 [Azure] > [HDInsight] > 您的 HDInsight 叢集 > [Hive 資料庫] > [預設]。 會列出 **errorLogs** 資料表和 **log4jLogs** 資料表。

## <a id="nextsteps"></a>後續步驟

如您所見，HDInsight tools for Visual Studio 提供簡單的方法，可在 HDInsight 上使用 Hive 查詢。

如需 HDInsight 中 Hive 的一般資訊：

* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)

如需您可以在 HDInsight 上使用 Hadoop 之其他方式的詳細資訊：

* [搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)

* [搭配使用 MapReduce 與 HDInsight 上的 Hadoop](hdinsight-use-mapreduce.md)

如需 HDInsight Tools for Visual Studio 的詳細資訊：

* [開始使用 HDInsight Tools for Visual Studio](apache-hadoop-visual-studio-tools-get-started.md)

[hdinsight-sdk-documentation]: http://msdnstage.redmond.corp.microsoft.com/library/dn479185.aspx

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://en.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: http://azure.microsoft.com/documentation/articles/hdinsight-connect-excel-power-query/


[hdinsight-use-oozie]: hdinsight-use-oozie.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md



[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md

[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]:submit-apache-hadoop-jobs-programmatically.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-get-started]:apache-hadoop-linux-tutorial-get-started.md

[powershell-here-strings]: http://technet.microsoft.com/library/ee692792.aspx

[image-hdi-hive-powershell]: ./media/hdinsight-use-hive/HDI.HIVE.PowerShell.png
[img-hdi-hive-powershell-output]: ./media/hdinsight-use-hive/HDI.Hive.PowerShell.Output.png
[image-hdi-hive-architecture]: ./media/hdinsight-use-hive/HDI.Hive.Architecture.png
