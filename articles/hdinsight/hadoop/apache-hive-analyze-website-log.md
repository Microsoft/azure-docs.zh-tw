---
title: 使用 Hive 和 Hadoop 進行網站記錄分析 - Azure HDInsight
description: 了解如何使用 HDInsight 上的 Hive 來分析網站記錄。 您將使用記錄檔做為 HDInsight 資料表的輸入，然後使用 HiveQL 來查詢資料。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/17/2016
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 47175ed4bc01a43d1f21cbfc84b8a788f11b7338
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048019"
---
# <a name="use-hive-with-windows-based-hdinsight-to-analyze-logs-from-websites"></a>使用 Windows 型 HDInsight 上的 Hive 分析網站的記錄
了解如何使用 HDInsight 上的 HiveQL 來分析網站的記錄。 網站記錄分析可用於根據類似活動來區隔對象、依人口統計將網站造訪者分類，以及找出他們檢視的內容、他們來自的網站等。

> [!IMPORTANT]
> 本文件的步驟只適用於 Windows HDInsight 叢集。 Windows 上的 HDInsight 只提供低於 HDInsight 3.4 的版本。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

在此範例中，您將使用 HDInsight 叢集來分析網站記錄檔，以深入了解一天內從外部網站的網站造訪頻率。 也將產生使用者遇到的網站錯誤摘要。 您會了解如何：

* 連接至包含網站記錄檔的 Azure Blob 儲存體。
* 建立 Hive 資料表以查詢這些記錄。
* 建立 Hive 查詢以分析資料。
* 使用 Microsoft Excel 連接到 HDInsight (使用開放式資料庫連接，ODBC) 擷取已分析的資料。

![HDI.Samples.Website.Log.Analysis](./media/apache-hive-analyze-website-log/hdinsight-weblogs-sample.png)

## <a name="prerequisites"></a>必要條件
* 您必須已在 Azure HDInsight 上佈建 Hadoop 叢集。 如需指示，請參閱[佈建 HDInsight 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。
* 您必須已安裝 Microsoft Excel 2013 或 Excel 2010。
* 您必須有 [Microsoft Hive ODBC 驅動程式](http://www.microsoft.com/download/details.aspx?id=40886) ，才能從 Hive 將資料匯入 Excel 中。

## <a name="to-run-the-sample"></a>執行範例
1. 從 [Azure 入口網站](https://portal.azure.com/)的「開始面板」(若您已在該處釘選叢集)，按一下您要在其中執行範例的叢集磚。
2. 在叢集刀鋒視窗的 [快速連結] 之下，按一下 [叢集儀表板]，然後從 [叢集儀表板] 刀鋒視窗按一下 [HDInsight 叢集儀表板]。 或者，您也可以使用下列 URL 直接開啟儀表板：

         https://<clustername>.azurehdinsight.net

    出現提示時，使用您佈建叢集時所用的系統管理員使用者名稱和密碼來進行驗證。
3. 從開啟的網頁中，按一下 [快速啟用資源庫] 索引標籤，然後在 [搭配範例資料的方案] 類別下，按一下 [網站記錄分析] 範例。
4. 依照網頁上提供的指示完成範例。

## <a name="next-steps"></a>後續步驟
嘗試下列範例： [使用 Hive 和 HDInsight 分析感應器資料](apache-hive-analyze-sensor-data.md)。

[hdinsight-sensor-data-sample]: ../hdinsight-use-hive-sensor-data-analysis.md
