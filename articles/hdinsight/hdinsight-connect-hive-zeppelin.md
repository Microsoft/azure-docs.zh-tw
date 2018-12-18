---
title: 使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢
description: 了解如何使用 Zeppelin 來執行 Hive 查詢。
keywords: hdinsight,hadoop,hive,互動式查詢,LLAP
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: jasonh
ms.openlocfilehash: 3064c9cd141458307891f666bd5af9aa738cc021
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43093163"
---
# <a name="use-zeppelin-to-run-hive-queries-in-azure-hdinsight"></a>使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢 

HDInsight 互動式查詢叢集納入了 Zeppelin 筆記本，以供您用來執行互動式 Hive 查詢。 在本文中，您將會了解如何使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢。 

## <a name="prerequisites"></a>必要條件
在閱讀本文之前，您必須有下列各項：

* **HDInsight 互動式查詢叢集**。 請參閱[建立叢集](hadoop/apache-hadoop-linux-tutorial-get-started.md#create-cluster)以建立 HDInsight 叢集。  請務必選擇互動式查詢類型。 

## <a name="create-a-zeppelin-note"></a>建立 Zeppelin 記事

1. 瀏覽至下列 URL：

        https://CLUSTERNAME.azurehdinsight.net/zeppelin
    將 **CLUSTERNAME** 取代為您叢集的名稱。

2. 輸入您的 Hadoop 使用者名稱和密碼。 透過 [Zeppelin] 頁面，您可以建立新的記事或開啟現有記事。 HiveSample 包含一些 Hive 查詢範例。  

    ![HDInsight 互動式查詢 Zeppelin](./media/hdinsight-connect-hive-zeppelin/hdinsight-hive-zeppelin.png)
3. 按一下 [建立新記事]。
4. 輸入或選取下列值：

    - 記事名稱：輸入記事的名稱。
    - 預設解譯器：選取 [JDBC]。

5. 按一下 [建立記事]。
6. 執行下列 Hive 查詢：

        %jdbc(hive)
        show tables

    ![HDInsight 互動式查詢 Zeppelin 執行查詢](./media/hdinsight-connect-hive-zeppelin/hdinsight-hive-zeppelin-query.png)

    第一行裡面的 **%jdbc(hive)** 陳述式會指示筆記本使用 Hive JDBC 解譯器。

    此查詢應該會傳回一個 Hive 資料表，其名稱為 hivesampletable。

    以下是您可以對 hivesampletable 執行的另外兩個 Hive 查詢。 

        %jdbc(hive)
        select * from hivesampletable limit 10

        %jdbc(hive)
        select ${group_name}, count(*) as total_count
        from hivesampletable
        group by ${group_name=market,market|deviceplatform|devicemake}
        limit ${total_count=10}

    相較於傳統的 Hive，查詢結果的傳回速度會快很多。


## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用 Power BI 將 HDInsight 中的資料視覺化。  若要深入了解，請參閱下列文章：

* [在 Azure HDInsight 中使用 Microsoft Power BI 將 Hive 資料視覺化](hadoop/apache-hadoop-connect-hive-power-bi.md)。
* [在 Azure HDInsight 中使用 Power BI 將互動式查詢 Hive 資料視覺化](./interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)。
* [使用 Microsoft Hive ODBC 驅動程式將 Excel 連線到 HDInsight](hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)。
* [使用 Power Query 將 Excel 連線到 Hadoop](hadoop/apache-hadoop-connect-excel-power-query.md)。
* [使用 Data Lake Tools for Visual Studio 連線至 Azure HDInsight 及執行 Hive 查詢](hadoop/apache-hadoop-visual-studio-tools-get-started.md)。
* [使用適用於 Visual Studio Code 的 Azure HDInsight 工具](hdinsight-for-vscode.md)。
* [將資料上傳至 HDInsight](./hdinsight-upload-data.md)。
