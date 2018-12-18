---
title: 採用 Hadoop 的 Apache Sqoop - Azure HDInsight
description: 了解如何使用 Apache Sqoop 在 Hadoop on HDInsight 與 Azure SQL Database 之間進行匯入和匯出。
keywords: hadoop sqoop,sqoop
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 03/26/2018
ms.openlocfilehash: e9ee4ceb51b2de58010f3e6cf7feba9df64b9bad
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43050164"
---
# <a name="use-apache-sqoop-to-import-and-export-data-between-hadoop-on-hdinsight-and-sql-database"></a>使用 Apache Sqoop 在 HDInsight 與 SQL Database 的 Hadoop 間匯入及匯出資料

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

了解如何使用 Sqoop，在 Azure HDInsight 中的 Hadoop 叢集與 Azure SQL Database 或 Microsoft SQL Server 資料庫之間進行匯入和匯出。 本文件中的步驟直接從 Hadoop 叢集的前端節點使用 `sqoop` 命令。 您可以使用 SSH 連接至前端節點，並執行本文件中的命令。

> [!IMPORTANT]
> 本文件中的步驟只適用於使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

> [!WARNING]
> 本文件的步驟假設您已建立名為 `sqooptest` 的 Azure SQL Database。
>
> 本文件提供了 T-SQL 陳述式，用於在 SQL Database 中建立和查詢資料表。 有許多用戶端可以搭配使用這些陳述式與 SQL Database。 建議使用下列用戶端：
>
> * [SQL Server Management Studio](../../sql-database/sql-database-connect-query-ssms.md)
> * [Visual Studio Code](../../sql-database/sql-database-connect-query-vscode.md)
> * [sqlcmd](https://docs.microsoft.com/sql/tools/sqlcmd-utility) 公用程式

## <a name="create-the-table-in-sql-database"></a>在 SQL Database 中建立資料表

> [!IMPORTANT]
> 如果您使用在[建立叢集與 SQL Database](hdinsight-use-sqoop.md) 中建立的 HDInsight 叢集和 SQL Database，請略過這一節的步驟。 在[建立叢集與 SQL database](hdinsight-use-sqoop.md) 文件的步驟中所建立的資料庫和資料表。

使用 SQL 用戶端連線至 SQL Database 中的 `sqooptest` 資料庫。 然後使用下列 T-SQL 建立名為 `mobiledata` 的資料表：

```sql
CREATE TABLE [dbo].[mobiledata](
[clientid] [nvarchar](50),
[querytime] [nvarchar](50),
[market] [nvarchar](50),
[deviceplatform] [nvarchar](50),
[devicemake] [nvarchar](50),
[devicemodel] [nvarchar](50),
[state] [nvarchar](50),
[country] [nvarchar](50),
[querydwelltime] [float],
[sessionid] [bigint],
[sessionpagevieworder] [bigint])
GO
CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(clientid)
GO
```

## <a name="sqoop-export"></a>Sqoop export

1. 使用 SSH 連接到 HDInsight 叢集。 例如，下列命令可連接至名為 `mycluster` 之叢集的主要前端節點：

    ```bash
    ssh mycluster-ssh.azurehdinsight.net
    ```

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 若要確認 Sqoop 是否看得見您的 SQL Database，請使用下列命令：

    ```bash
    sqoop list-databases --connect jdbc:sqlserver://<serverName>.database.windows.net:1433 --username <adminLogin> -P
    ```
    出現提示時，請輸入 SQL Database 的登入密碼。

    此命令會傳回一份資料庫清單，其中包含稍早使用的 **sqooptest** 資料庫。

3. 若要在 SQL Databse 中將資料從 Hive **hivesampletable** 資料表匯出至 **mobiledata** 資料表，請使用下列命令：

    ```bash
    sqoop export --connect 'jdbc:sqlserver://<serverName>.database.windows.net:1433;database=sqooptest' --username <adminLogin> -P -table 'mobiledata' --hcatalog-table hivesampletable
    ```

4. 若要確認是否已匯出資料，請從 SQL 用戶端使用下列查詢，以檢視匯出的資料：

    ```sql
    SET ROWCOUNT 50;
    SELECT * FROM mobiledata;
    ```

    此命令會列出 50 個匯入至資料表的資料列。

## <a name="sqoop-import"></a>Sqoop import

1. 使用下列命令將資料從 SQL Database 中的 **mobiledata** 資料表匯入至 HDInsight 上的 **wasb:///tutorials/usesqoop/importeddata** 目錄：

    ```bash
    sqoop import --connect 'jdbc:sqlserver://<serverName>.database.windows.net:1433;database=sqooptest' --username <adminLogin> -P --table 'mobiledata' --target-dir 'wasb:///tutorials/usesqoop/importeddata' --fields-terminated-by '\t' --lines-terminated-by '\n' -m 1
    ```

    資料中的欄位是以定位字元分隔，行是以換行字元終止。

    > [!IMPORTANT]
    > `wasb:///` 路徑會與叢集搭配運作，而叢集會使用 Azure 儲存體作為預設叢集存放區。 如果是使用 Azure Data Lake Store 的叢集，請改為使用 `adl:///`。

2. 匯入完成後，使用下列命令來列出新目錄中的資料：

    ```bash
    hdfs dfs -text /tutorials/usesqoop/importeddata/part-m-00000
    ```

## <a name="using-sql-server"></a>使用 SQL Server

您也可以使用 Sqoop 從 SQL Server 匯入和匯出資料。 使用 SQL Database 和 SQL Server 之間的差異如下：

* HDInsight 與 SQL Server 必須位於相同的 Azure 虛擬網路。

    如需範例，請參閱[將 HDInsight 連線至內部部署網路](./../connect-on-premises-network.md)文件。

    如需使用 HDInsight 搭配 Azure 虛擬網路的詳細資訊，請參閱[使用 Azure 虛擬網路擴充 HDInsight](../hdinsight-extend-hadoop-virtual-network.md)文件。 如需 Azure 虛擬網路的詳細資訊，請參閱[虛擬網路概觀](../../virtual-network/virtual-networks-overview.md)文件。

* SQL Server 也必須設定為允許 SQL 驗證。 如需詳細資訊，請參閱[選擇驗證模式](https://msdn.microsoft.com/ms144284.aspx)文件。

* 您可能必須設定 SQL Server 以接受遠端連線。 如需詳細資訊，請參閱[如何疑難排解 SQL Server Database Engine 連線](http://social.technet.microsoft.com/wiki/contents/articles/2102.how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx)文件 (英文)。

* 使用下列 Transact-SQL 陳述式建立 **mobiledata** 資料表︰

    ```sql
    CREATE TABLE [dbo].[mobiledata](
    [clientid] [nvarchar](50),
    [querytime] [nvarchar](50),
    [market] [nvarchar](50),
    [deviceplatform] [nvarchar](50),
    [devicemake] [nvarchar](50),
    [devicemodel] [nvarchar](50),
    [state] [nvarchar](50),
    [country] [nvarchar](50),
    [querydwelltime] [float],
    [sessionid] [bigint],
    [sessionpagevieworder] [bigint])
    ```

* 從 HDInsight 連線到 SQL Server 時，您可能必須使用 SQL Server 的 IP 位址。 例如︰

    ```bash
    sqoop import --connect 'jdbc:sqlserver://10.0.1.1:1433;database=sqooptest' --username <adminLogin> -P -table 'mobiledata' --target-dir 'wasb:///tutorials/usesqoop/importeddata' --fields-terminated-by '\t' --lines-terminated-by '\n' -m 1
    ```

## <a name="limitations"></a>限制

* 大量匯出 - 使用 Linux 型 HDInsight，Sqoop 連接器用來將資料匯出至 Microsoft SQL Server 或 Azure SQL Database，不支援大量插入。

* 批次處理 - 使用 Linux 型 HDInsight，執行插入時若使用 `-batch` 參數，Sqoop 將會執行多個插入，而不是批次處理插入作業。

## <a name="next-steps"></a>後續步驟

現在，您已了解如何使用 Sqoop。 若要深入了解，請參閱：

* [搭配 HDInsight 使用 Oozie](../hdinsight-use-oozie.md)：在 Oozie 工作流程中使用 Sqoop 動作。
* [使用 HDInsight 分析航班延誤資料](../hdinsight-analyze-flight-delay-data.md)：使用 Hive 分析航班誤點資料，然後使用 Sqoop 將資料匯出至 Azure SQL Database。
* [將資料上傳至 HDInsight](../hdinsight-upload-data.md)：尋找可將資料上傳至 HDInsight/Azure Blob 儲存體的其他方法。

[hdinsight-versions]:  ../hdinsight-component-versioning.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-get-started]:apache-hadoop-linux-tutorial-get-started.md
[hdinsight-storage]: ../hdinsight-hadoop-use-blob-storage.md
[hdinsight-submit-jobs]:submit-apache-hadoop-jobs-programmatically.md
[sqldatabase-get-started]: ../sql-database-get-started.md
[sqldatabase-create-configue]: ../sql-database-create-configure.md

[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-install]: /powershell/azureps-cmdlets-docs
[powershell-script]: http://technet.microsoft.com/library/ee176949.aspx

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html
