---
title: 使用 Sqoop 在 Azure Data Lake Storage Gen1 和 Azure SQL Database 之間複製資料 | Microsoft Docs
description: 使用 Sqoop 在 Azure SQL Database 和 Azure Data Lake Storage Gen1 之間複製資料
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: 3f914b2a-83cc-4950-b3f7-69c921851683
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 958171a8d1091254588aef250406b968009eb968
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391737"
---
# <a name="copy-data-between-azure-data-lake-storage-gen1-and-azure-sql-database-using-sqoop"></a>使用 Sqoop 在 Azure Data Lake Storage Gen1 和 Azure SQL Database 之間複製資料
了解如何使用 Apache Sqoop 在 Azure SQL Database 和 Azure Data Lake Storage Gen1 之間匯入及匯出資料。

## <a name="what-is-sqoop"></a>什麼是 Sqoop？
巨量資料應用程式是處理非結構化和半結構化資料 (例如記錄和檔案)，很自然的一個選擇。 不過，也有可能需要處理儲存在關聯式資料庫中的結構化資料。

[Apache Sqoop](https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html) 是一個專門設計來在關聯式資料庫和巨量資料儲存機制 (例如 Azure Data Lake Storage Gen1) 之間傳送資料的工具。 您可以使用它從像是 Azure SQL Database 這類的關聯式資料庫管理系統 (RDBMS)，匯入資料至 Azure Data Lake Storage Gen1。 您可以使用巨量資料工作負載來轉換和分析資料，然後重新將資料匯出到 RDBMS。 在本教學課程中，您會使用 Azure SQL Database 做為匯入/匯出之來源關聯式資料庫。

## <a name="prerequisites"></a>必要條件
開始閱讀本文之前，您必須符合下列必要條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure Data Lake Storage Gen1 帳戶**。 如需如何建立帳戶的指示，請參閱[開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* 可存取 Data Lake Storage Gen1 帳戶的 **Azure HDInsight 叢集**。 請參閱[建立搭配 Data Lake Storage Gen1 的 HDInsight 叢集](data-lake-store-hdinsight-hadoop-use-portal.md)。 本文假設您已使用 Azure Data Lake Storage Gen1 存取 HDInsight Linux 叢集。
* **Azure SQL Database**。 如需建立方式的指示，請參閱 [建立 Azure SQL Database](../sql-database/sql-database-get-started.md)

## <a name="do-you-learn-fast-with-videos"></a>使用影片快速學習？
請[觀看這部影片](https://mix.office.com/watch/1butcdjxmu114)，主題是關於如何使用 DistCp 在 Azure 儲存體 Blob 與 Data Lake Storage Gen1 之間複製資料。

## <a name="create-sample-tables-in-the-azure-sql-database"></a>在 Azure SQL Database 中建立範例資料表
1. 若要開始，請在 Azure SQL Database 中建立兩個範例資料表。 使用 [SQL Server Management Studio](../sql-database/sql-database-connect-query-ssms.md) 或 Visual Studio 連接至 Azure SQL Database，然後執行下列查詢。

    **建立 Table1**

        CREATE TABLE [dbo].[Table1](
        [ID] [int] NOT NULL,
        [FName] [nvarchar](50) NOT NULL,
        [LName] [nvarchar](50) NOT NULL,
         CONSTRAINT [PK_Table_1] PRIMARY KEY CLUSTERED
            (
                   [ID] ASC
            )
        ) ON [PRIMARY]
        GO

    **建立 Table2**

        CREATE TABLE [dbo].[Table2](
        [ID] [int] NOT NULL,
        [FName] [nvarchar](50) NOT NULL,
        [LName] [nvarchar](50) NOT NULL,
         CONSTRAINT [PK_Table_2] PRIMARY KEY CLUSTERED
            (
                   [ID] ASC
            )
        ) ON [PRIMARY]
        GO
2. 在 **Table1**中，新增一些範例資料。 保留 **Table2** 空白。 我們從 **Table1** 匯入資料至 Data Lake Storage Gen1。 然後，我們會從 Data Lake Storage Gen1 將資料匯出到 **Table2**。 執行下列程式碼片段。

        INSERT INTO [dbo].[Table1] VALUES (1,'Neal','Kell'), (2,'Lila','Fulton'), (3, 'Erna','Myers'), (4,'Annette','Simpson');


## <a name="use-sqoop-from-an-hdinsight-cluster-with-access-to-data-lake-storage-gen1"></a>從使用 Data Lake Storage Gen1 存取的 HDInsight Linux 叢集使用 Sqoop
HDInsight 叢集已有可用的 Sqoop 套件。 如果您已設定 HDInsight 叢集使用 Data Lake Storage Gen1 做為額外的儲存體，您可以使用 Sqoop (不需要任何組態變更) 在關聯式資料庫 (本範例中為 Azure SQL Database) 與 Data Lake Storage Gen1 帳戶之間匯入/匯出資料。

1. 在本教學課程中，我們假設您已經建立 Linux 叢集，因此您應該使用 SSH 來連線至叢集。 請參閱 [連線至以 Linux 為基礎的 HDInsight 叢集](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md)。
2. 請確認您是否可從叢集存取 Data Lake Storage Gen1 帳戶。 從 SSH 提示字元執行下列命令：

        hdfs dfs -ls adl://<data_lake_storage_gen1_account>.azuredatalakestore.net/

    這應會提供 Data Lake Storage Gen1 帳戶中的檔案/資料夾清單。

### <a name="import-data-from-azure-sql-database-into-data-lake-storage-gen1"></a>從 Azure SQL Database 將資料匯入至 Data Lake Storage Gen1
1. 瀏覽至提供 Sqoop 封裝的目錄。 一般而言，這會在 `/usr/hdp/<version>/sqoop/bin`。
2. 從 **Table1** 將資料匯入至 Data Lake Storage Gen1。 使用下列語法：

        sqoop-import --connect "jdbc:sqlserver://<sql-database-server-name>.database.windows.net:1433;username=<username>@<sql-database-server-name>;password=<password>;database=<sql-database-name>" --table Table1 --target-dir adl://<data-lake-storage-gen1-name>.azuredatalakestore.net/Sqoop/SqoopImportTable1

    請注意， **sql-database-server-name** 預留位置代表正在執行 Azure SQL Database 之伺服器的名稱。 **sql-database-name** 預留位置代表實際的資料庫名稱。

    例如，


        sqoop-import --connect "jdbc:sqlserver://mysqoopserver.database.windows.net:1433;username=nitinme@mysqoopserver;password=<password>;database=mysqoopdatabase" --table Table1 --target-dir adl://myadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1

1. 請確認資料已被傳輸至 Data Lake Storage Gen1 帳戶。 執行以下命令：

        hdfs dfs -ls adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/

    您應該會看見下列輸出。


        -rwxrwxrwx   0 sshuser hdfs          0 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/_SUCCESS
        -rwxrwxrwx   0 sshuser hdfs         12 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00000
        -rwxrwxrwx   0 sshuser hdfs         14 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00001
        -rwxrwxrwx   0 sshuser hdfs         13 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00002
        -rwxrwxrwx   0 sshuser hdfs         18 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00003

    每個 **part-m-*** 檔案會對應至來源資料表 **Table1**中的資料列。 您可以檢視 part-m-* 檔案的內容來確認。


### <a name="export-data-from-data-lake-storage-gen1-into-azure-sql-database"></a>從 Data Lake Storage Gen1 將資料匯出到 Azure SQL Database
1. 從 Data Lake Storage Gen1 帳戶將資料匯出到 Azure SQL Database 中的空白資料表 **Table2**。 使用下列語法。

        sqoop-export --connect "jdbc:sqlserver://<sql-database-server-name>.database.windows.net:1433;username=<username>@<sql-database-server-name>;password=<password>;database=<sql-database-name>" --table Table2 --export-dir adl://<data-lake-storage-gen1-name>.azuredatalakestore.net/Sqoop/SqoopImportTable1 --input-fields-terminated-by ","

    例如，


        sqoop-export --connect "jdbc:sqlserver://mysqoopserver.database.windows.net:1433;username=nitinme@mysqoopserver;password=<password>;database=mysqoopdatabase" --table Table2 --export-dir adl://myadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1 --input-fields-terminated-by ","

1. 確認資料已上傳至 SQL Database 資料表。 使用 [SQL Server Management Studio](../sql-database/sql-database-connect-query-ssms.md) 或 Visual Studio 連接至 Azure SQL Database，然後執行下列查詢。

        SELECT * FROM TABLE2

    這應該會有下列的輸出。

         ID  FName   LName
        ------------------
        1    Neal    Kell
        2    Lila    Fulton
        3    Erna    Myers
        4    Annette    Simpson

## <a name="performance-considerations-while-using-sqoop"></a>使用 Sqoop 時的效能考量

關於 Sqoop 作業將資料複製到 Data Lake Storage Gen1 時的效能調整，請參閱 [Sqoop 效能文件](https://blogs.msdn.microsoft.com/bigdatasupport/2015/02/17/sqoop-job-performance-tuning-in-hdinsight-hadoop/)。

## <a name="see-also"></a>另請參閱
* [將資料從 Azure 儲存體 Blob 複製到 Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
* [搭配 Data Lake Storage Gen1 使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [搭配 Data Lake Storage Gen1 使用 Azure HDInsight](data-lake-store-hdinsight-hadoop-use-portal.md)
