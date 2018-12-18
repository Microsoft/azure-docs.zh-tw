---
title: 使用 Power Query 將 Excel 連接到 Hadoop - Azure HDInsight
description: 了解如何利用商業智慧元件和使用 Power Query for Excel 來存取 HDInsight 上的 Hadoop 中儲存的資料。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.openlocfilehash: 45014592cd648ee1a7b75fa59babf8571c151269
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042296"
---
# <a name="connect-excel-to-hadoop-by-using-power-query"></a>使用 Power Query 將 Excel 連接到 Hadoop
Microsoft 巨量資料方案的主要功能之一，是將 Microsoft 商業智慧 (BI) 元件與 Azure HDInsight 上的 Hadoop 叢集相整合。 舉例來說，可讓您使用 Microsoft Power Query for Excel 增益集，將 Excel 連接到包含 Hadoop 叢集相關聯資料的 Azure 儲存體帳戶。 本文將逐步解說如何設定及使用 Power Query，以查詢受 HDInsight 管理的 Hadoop 叢集相關聯資料。

### <a name="prerequisites"></a>必要條件
開始閱讀本文之前，您必須有下列各項：

* **HDInsight 叢集**。 若要設定叢集，請參閱 [開始使用 Azure HDInsight][hdinsight-get-started]。
* **工作站** 。
* **Office 2016、Office 2013 專業增強版、Office 365 專業增強版、Excel 2013 獨立版或 Office 2010 專業增強版**。

## <a name="install-power-query"></a>安裝 Power Query
Power Query 可匯入在 HDInsight 叢集上執行的 Hadoop 工作所匯出或產生的資料。

在 Excel 2016 中，Power Query 已整合至 [Get & Transform] \(取得並轉換\) 區段下方的 [Data] \(資料\) 功能區。 如果是舊的 Excel 版本，請從 [Microsoft 下載中心][powerquery-download]下載並安裝 Microsoft Power Query for Excel。

## <a name="import-hdinsight-data-into-excel"></a>將 HDInsight 資料匯入 Excel 中
Power Query add-in for Excel 可協助您將 HDInsight 叢集中的資料匯入至 Excel，以便使用 PowerPivot 和 Power Map 等 BI 工具來檢查、分析及呈現資料。

**從 HDInsight 叢集匯入資料**

1. 開啟 Excel。
2. 建立新的空白活頁簿。
3. 依據 Excel 版本執行下列步驟：

    - Excel 2016

        - 按一下 [Data] \(資料\) 功能表，從 [Get & Transform Data] \(取得並轉換資料\) 功能區中按一下 [Get Data] \(取得資料\)，按一下 [From Azure] \(從 Azure\)，然後按一下 [From Azure HDInsight(HDFS)] \(從 Azure HDInsight(HDFS)\)。

        ![HDI.PowerQuery.SelectHdiSource](./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.selecthdisource.excel2016.png)

    - Excel 2013/2010

        - 依序按一下 [Power Query] 功能表、[從其他來源]、[從 Microsoft Azure HDInsight]。
   
        ![HDI.PowerQuery.SelectHdiSource][image-hdi-powerquery-hdi-source]
       
        **附註：** 如果 [Power Query] 功能表未顯示，請移至 [檔案] > [選項] > [增益集]，然後從頁面底部的下拉式 [管理] 方塊中，選取 [COM 增益集]。 選取 [移至...] 按鈕，並驗證 Power Query for Excel 增益集的方塊已勾選。
       
        **附註：** Power Query 也可讓您從 HDFS 匯入資料，方法是按一下 [從其他來源]。
4. 對於 [帳戶名稱]，輸入叢集相關 Azure Blob 儲存體帳戶的名稱，然後按一下 [確定]。 此帳戶可以是[預設儲存體帳戶](../hdinsight-administer-use-management-portal.md#find-the-default-storage-account)或連結儲存體帳戶。  其格式為 *https://&lt;StorageAccountName>.blob.core.windows.net/*。
5. 對於 [帳戶金鑰]，輸入 Blob 儲存體帳戶的金鑰，然後按一下 [儲存]。 (只有在您第一次存取這個存放區時才必須輸入帳戶資訊)。
6. 在 [查詢編輯器] 左側的 [ **瀏覽器** ] 窗格中，按兩下 Blob 儲存體容器名稱。 依預設，容器名稱與叢集名稱相同。
7. 在 [名稱] 欄中找出 **HiveSampleData.txt** (資料夾路徑為 **../hive/warehouse/hivesampletable/**)，然後按一下 HiveSampleData.txt 左側的 [二進位]。 HiveSampleData.txt 隨附於所有叢集。 您也可以選擇使用您自己的檔案。
   
    ![HDI.PowerQuery.ImportData][image-hdi-powerquery-importdata]
8. 如有需要，您可以將欄名稱重新命名。 請在準備就緒後，按一下 [關閉並載入]。  資料已載入至您的活頁簿：
   
    ![HDI.PowerQuery.ImportedTable][image-hdi-powerquery-imported-table]

## <a name="next-steps"></a>後續步驟
在本文中，您已了解到如何使用 Power Query 將 HDInsight 中的資料擷取至 Excel。 同樣地，您也可以將 HDInsight 中的資料擷取至 Azure SQL Database。 此外也可以將資料上傳至 HDInsight。 若要深入了解，請參閱下列文章：

* [在 Azure HDInsight 中使用 Microsoft Power BI 將 Hive 資料視覺化](apache-hadoop-connect-hive-power-bi.md)。
* [在 Azure HDInsight 中使用 Power BI 將互動式查詢 Hive 資料視覺化](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)。
* [使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢](./../hdinsight-connect-hive-zeppelin.md)。
* [使用 Microsoft Hive ODBC 驅動程式將 Excel 連線到 HDInsight](apache-hadoop-connect-excel-hive-odbc-driver.md)。
* [使用 Data Lake Tools for Visual Studio 連線至 Azure HDInsight 及執行 Hive 查詢](apache-hadoop-visual-studio-tools-get-started.md)。
* [使用適用於 Visual Studio Code 的 Azure HDInsight 工具](../hdinsight-for-vscode.md)。
* [將資料上傳至 HDInsight](./../hdinsight-upload-data.md)。

[image-hdi-powerquery-hdi-source]: ./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.selecthdisource.png
[image-hdi-powerquery-importdata]: ./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.importdata.png
[image-hdi-powerquery-imported-table]: ./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.importedtable.PNG

[powerquery-download]: http://go.microsoft.com/fwlink/?LinkID=286689
