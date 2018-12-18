---
title: 使用 Hive ODBC 驅動程式將 Excel 連線到 Hadoop - Azure HDInsight
description: 了解如何設定和使用 Excel 的 Microsoft Hive ODBC 驅動程式，從 Microsoft Excel 查詢 HDInsight 叢集中的資料。
keywords: hadoop excel, hive excel, hive odbc
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: b21863d7a91c14f9795d72a13575e33485ba7d2b
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041819"
---
# <a name="connect-excel-to-hadoop-in-azure-hdinsight-with-the-microsoft-hive-odbc-driver"></a>使用 Microsoft Hive ODBC 驅動程式將 Excel 連線到 Azure HDInsight 中的 Hadoop

[!INCLUDE [ODBC-JDBC-selector](../../../includes/hdinsight-selector-odbc-jdbc.md)]

Microsoft 巨量資料方案會將 Microsoft 商業智慧 (BI) 元件與 Azure HDInsight 所部署的 Apache Hadoop 叢集相整合。 舉例來說，此整合可讓您使用 Microsoft Hive 開放式資料庫連線能力 (ODBC) 驅動程式，將 Excel 連線到 HDInsight 中 Hadoop 叢集的 Hive 資料倉儲。

您也可以從 Excel 使用 Microsoft Power Query for Excel，連接與 HDInsight 叢集和其他資料來源 (包括其他 (非 HDInsight) Hadoop 叢集) 相關聯的資料。 如需安裝和使用 Power Query 的相關資訊，請參閱[使用 Power Query 將 Excel 連接到 HDInsight][hdinsight-power-query]。



**必要條件**：

開始閱讀本文之前，您必須有下列各項：

* **HDInsight 叢集**。 若要建立，請參閱[開始使用 Azure HDInsight](apache-hadoop-linux-tutorial-get-started.md)。
* **工作站** 。

## <a name="install-microsoft-hive-odbc-driver"></a>安裝 Microsoft Hive ODBC 驅動程式
從[下載中心][hive-odbc-driver-download]下載並安裝 Microsoft Hive ODBC 驅動程式。

此驅動程式可以安裝在 32 位元或 64 位元版本的 Windows 7、Windows 8、Windows 10、Windows Server 2008 R2 和 Windows Server 2012。 驅動程式可允許 Azure HDInsight 的連線。 您所安裝的版本必須與您要使用 ODBC 驅動程式的應用程式版本相符。 本教學課程將會從 Office Excel 使用此驅動程式。

## <a name="create-hive-odbc-data-source"></a>建立 Hive ODBC 資料來源
下列步驟將說明如何建立 Hive ODBC 資料來源。

1. 在 Windows 8 或 Windows 10 中，按視窗鍵以開啟 [開始] 畫面，然後輸入 **資料來源**。
2. 根據您的 Office 版本，按一下 [設定 ODBC 資料來源 (32 位元)] 或 [設定 ODBC 資料來源 (64 位元)]。 如果您使用 Windows 7，請從 [系統管理工具] 中選擇 [ODBC 資料來源 (32 位元)] 或 [ODBC 資料來源 (64 位元)]。 您應該會看到 [ODBC 資料來源管理員] 對話方塊。
   
    ![OBDC 資料來源管理員](./media/apache-hadoop-connect-excel-hive-odbc-driver/HDI.SimbaHiveOdbc.DataSourceAdmin1.png "使用 ODBC 資料來源管理員設定 DSN")

3. 在 [使用者 DNS] 中按一下 [新增]，以開啟 [建立新資料來源] 精靈。
4. 選取 [Microsoft Hive ODBC 驅動程式]，然後按一下 [完成]。 您應該會看到 [Microsoft Hive ODBC 驅動程式 DNS 設定] 對話方塊。
5. 輸入或選取下列值：
   
   | 屬性 | 說明 |
   | --- | --- |
   |  資料來源名稱 |為資料來源指定名稱 |
   |  Host |輸入 &lt;HDInsightClusterName>.azurehdinsight.net。 例如，myHDICluster.azurehdinsight.net |
   |  Port |使用 <strong>443</strong> (此連接埠已從 563 變更為 443)。 |
   |  資料庫 |使用<strong>預設值</strong> |
   |  機制 |選取 [Azure HDInsight 服務] |
   |  使用者名稱 |輸入 HDInsight 叢集 HTTP 使用者的使用者名稱。 預設的使用者名稱為 <strong>admin</strong>。 |
   |  密碼 |輸入 HDInsight 叢集使用者的密碼。 |
   
    </table>
   
    當您按一下 [進階選項] 時，您必須留意某些重要參數：
   
   | 參數 | 說明 |
   | --- | --- |
   |  使用原生查詢 |選取此選項時，ODBC 驅動程式不會嘗試將 TSQL 轉換為 HiveQL。 只有在百分之百確定您所提交的是純 HiveQL 陳述式時，才應使用此選項。 連接到 SQL Server 或 Azure SQL Database 時，您應將它保留為未勾選。 |
   |  每個區塊擷取的資料列 |在擷取大量記錄時，可能必須調整此參數，以確保最佳效能。 |
   |  預設字串資料行長度、二進位資料行長度、十進位資料行小數位數 |資料類型的長度和精確度可能會影響傳回資料的方式。 如果失去精確度且 (或) 發生截斷狀況，會傳回不正確的資訊。 |

    ![進階選項](./media/apache-hadoop-connect-excel-hive-odbc-driver/HDI.HiveOdbc.DataSource.AdvancedOptions1.png "進階 DSN 設定選項")

1. 按一下 [測試]  以測試資料來源。 資料來源正確設定時，會顯示 *「測試順利完成！」*。
2. 按一下 [確定]  以關閉 [測試] 對話方塊。 新的資料來源應會列示在 [ODBC 資料來源管理員] 中。
3. 按一下 [確定]  以結束精靈。

## <a name="import-data-into-excel-from-hdinsight"></a>從 HDInsight 將資料匯入 Excel 中
下列步驟將說明如何使用您在上一節中建立的 ODBC 資料來源，將資料從 Hive 資料表匯入 Excel 活頁簿中。

1. 在 Excel 中開啟新的或現有的活頁簿。
2. 從 [資料] 索引標籤，依序按一下 [取得資料] 和 [從其他來源]，然後按一下 [從 ODBC] 以啟動 [資料連接精靈]。
   
    ![開啟資料連線精靈](./media/apache-hadoop-connect-excel-hive-odbc-driver/HDI.SimbaHiveOdbc.Excel.DataConnection1.png "開啟資料連線精靈")
4. 選取您在上一節中建立的資料來源名稱，然後按一下 [確定]。
5. 輸入 Hadoop 使用者名稱 (預設名稱為 admin) 和密碼，然後按一下 [連接]。
6. 在導覽器中，依序展開 [HIVE] 和 [default]，按一下 [hivesampletable]，然後按一下 [載入]。 經過數秒後，資料即會匯入至 Excel。

    ![HDInsight Hive ODBC 導覽器](./media/apache-hadoop-connect-excel-hive-odbc-driver/hdinsight.hive.odbc.navigator.png "開啟資料連接精靈")


## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用 Microsoft Hive ODBC 驅動程式將 HDInsight 服務中的資料擷取至 Excel。 同樣地，您也可以將 HDInsight 服務中的資料擷取至 SQL Database。 此外也可以將資料上傳至 HDInsight 服務。 若要深入了解，請參閱：

* [在 Azure HDInsight 中使用 Microsoft Power BI 將 Hive 資料視覺化](apache-hadoop-connect-hive-power-bi.md)。
* [在 Azure HDInsight 中使用 Power BI 將互動式查詢 Hive 資料視覺化](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)。
* [使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢](./../hdinsight-connect-hive-zeppelin.md)。
* [使用 Power Query 將 Excel 連線到 Hadoop](apache-hadoop-connect-excel-power-query.md)。
* [使用 Data Lake Tools for Visual Studio 連線至 Azure HDInsight 及執行 Hive 查詢](apache-hadoop-visual-studio-tools-get-started.md)。
* [使用適用於 Visual Studio Code 的 Azure HDInsight 工具](../hdinsight-for-vscode.md)。
* [將資料上傳至 HDInsight](./../hdinsight-upload-data.md)。

[hdinsight-use-sqoop]:hdinsight-use-sqoop.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md
[hdinsight-use-hive]:hdinsight-use-hive.md
[hdinsight-upload-data]: ../hdinsight-upload-data.md
[hdinsight-power-query]: ../hdinsight-connect-excel-power-query.md
[hive-odbc-driver-download]: http://go.microsoft.com/fwlink/?LinkID=286698


