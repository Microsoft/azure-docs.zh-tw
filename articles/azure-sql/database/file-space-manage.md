---
title: Azure SQL Database 檔案空間管理
description: 此頁面描述如何管理 Azure SQL Database 中的單一和集區資料庫的檔案空間，並提供如何判斷您是否需要壓縮單一或集區資料庫，以及如何執行資料庫壓縮作業的程式碼範例。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: jrasnick, sstein
ms.date: 12/22/2020
ms.openlocfilehash: 08cab806d6ad8b75821a92994dde0fa07db8b960
ms.sourcegitcommit: c7153bb48ce003a158e83a1174e1ee7e4b1a5461
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2021
ms.locfileid: "98233588"
---
# <a name="manage-file-space-for-databases-in-azure-sql-database"></a>在 Azure SQL Database 中管理資料庫的檔案空間
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

本文說明 Azure SQL Database 中資料庫的不同儲存空間類型，以及當設定檔案空間需要明確管理時可採取的步驟。

> [!NOTE]
> 本文「不」適用於 Azure SQL Database 受控執行個體。

## <a name="overview"></a>總覽

使用 Azure SQL Database 時，會有工作負載模式，而資料庫的基礎資料檔案配置可能會變得大於使用的資料頁量。 當使用的空間增加，隨後卻將資料刪除時，就會發生這種狀況。 原因是因為在資料刪除後，並不會自動回收已配置的檔案空間。

在下列情況中，可能需要監視檔案空間使用量和壓縮資料檔案：

- 當配置給檔案資料庫的檔案空間達到集區大小上限時，允許彈性集區中的資料成長。
- 允許減少單一資料庫或彈性集區的大小上限。
- 允許將單一資料庫或彈性集區變更為大小上限較低的不同服務層級或效能層級。

### <a name="monitoring-file-space-usage"></a>監視檔案空間使用量

在 Azure 入口網站和下列 API 中顯示的大部分儲存體空間計量只會測量已使用資料頁面的大小：

- Azure Resource Manager 型計量 API 包括 PowerShell [get-metrics](/powershell/module/az.monitor/get-azmetric)
- T-SQL：[sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)

不過，下列 API 也會測量配置給資料庫和彈性集區的空間大小：

- T-SQL：[sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)
- T-SQL：[sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database)

### <a name="shrinking-data-files"></a>壓縮資料檔案

Azure SQL Database 不會自動壓縮資料檔案，以回收未使用的配置空間，因為可能會對資料庫效能造成影響。  不過，客戶可以依照「 [回收未使用](#reclaim-unused-allocated-space)的配置空間」中所述的步驟，在其選擇時，透過自助服務來壓縮資料檔案。

> [!NOTE]
> 不同于資料檔案，Azure SQL Database 會自動壓縮記錄檔，因為該作業不會影響資料庫效能。

## <a name="understanding-types-of-storage-space-for-a-database"></a>了解資料庫的儲存體空間類型

要管理資料庫的檔案空間，務必要了解下列儲存體空間數量。

|資料庫數量|定義|註解|
|---|---|---|
|**已使用的資料空間**|以 8 KB 分頁用來儲存資料庫資料的空間量。|一般而言，使用的空間會在插入 (刪除) 時增加 (減少)。 在某些情況下，根據作業與任何分割中涉及的資料量和模式而定，使用的空間並不會隨插入或刪除而變更。 例如，從每個資料頁刪除一個資料列，不見得會減少使用的空間。|
|**已配置的資料空間**|可用以儲存資料庫資料的格式化檔案空間量。|配置的空間量會自動成長，但絕不會在刪除後減少。 此行為可確保未來能更快地插入，因為不需要重新格式化空間。|
|**已配置但未使用的資料空間**|已配置的資料空間量與已使用的資料空間之間的差異。|此數量代表可藉由壓縮資料檔案而回收的可用空間量上限。|
|**資料大小上限**|可用來儲存資料庫資料的空間量上限。|配置的資料空間量不得成長超過資料大小上限。|

下圖說明資料庫的不同儲存體空間類型之間的關聯性。

![儲存體空間類型和關聯性](./media/file-space-manage/storage-types.png)

## <a name="query-a-single-database-for-storage-space-information"></a>查詢資料庫來取得儲存體空間資訊

下列查詢可用來判斷單一資料庫的儲存體空間數量。  

### <a name="database-data-space-used"></a>使用的資料庫資料空間

修改下列查詢，以傳回資料庫已使用的資料空間量。  查詢結果以 MB 為單位。

```sql
-- Connect to master
-- Database data space used in MB
SELECT TOP 1 storage_in_megabytes AS DatabaseDataSpaceUsedInMB
FROM sys.resource_stats
WHERE database_name = 'db1'
ORDER BY end_time DESC;
```

### <a name="database-data-space-allocated-and-unused-allocated-space"></a>配置的資料庫資料空間與已配置但未使用的空間

使用下列查詢，以傳回已配置的資料庫資料空間量與已配置但未使用的空間量。  查詢結果以 MB 為單位。

```sql
-- Connect to database
-- Database data space allocated in MB and database data space allocated unused in MB
SELECT SUM(size/128.0) AS DatabaseDataSpaceAllocatedInMB,
SUM(size/128.0 - CAST(FILEPROPERTY(name, 'SpaceUsed') AS int)/128.0) AS DatabaseDataSpaceAllocatedUnusedInMB
FROM sys.database_files
GROUP BY type_desc
HAVING type_desc = 'ROWS';
```

### <a name="database-data-max-size"></a>資料庫資料大小上限

修改下列查詢，以傳回資料庫資料大小上限。  查詢結果以位元組為單位。

```sql
-- Connect to database
-- Database data max size in bytes
SELECT DATABASEPROPERTYEX('db1', 'MaxSizeInBytes') AS DatabaseDataMaxSizeInBytes;
```

## <a name="understanding-types-of-storage-space-for-an-elastic-pool"></a>了解彈性集區的儲存體空間類型

要管理彈性集區的檔案空間，務必要了解下列儲存體空間數量。

|彈性集區數量|定義|註解|
|---|---|---|
|**已使用的資料空間**|彈性集區中的所有資料庫所使用的資料空間總和。||
|**已配置的資料空間**|彈性集區中的所有資料庫所配置的資料空間總和。||
|**已配置但未使用的資料空間**|已配置的資料空間量與彈性集區中的所有資料庫已使用的資料空間之間的差異。|此數量代表已為彈性集區配置、且可藉由壓縮資料檔案而回收的空間量上限。|
|**資料大小上限**|彈性集區可用於其所有資料庫的資料空間量上限。|為彈性集區配置的空間不應超過彈性集區大小上限。  如果發生此狀況，則可藉由壓縮資料庫資料檔案來回收已配置但未使用的空間。|

> [!NOTE]
> [彈性集區已達到儲存空間限制] 錯誤訊息表示已配置足夠的空間來符合彈性集區儲存體限制，但是資料空間配置中可能有未使用的空間。 請考慮增加彈性集區的儲存空間限制，或使用下面的「 [**回收未使用**](#reclaim-unused-allocated-space) 的配置空間」一節，以較短期的解決方案來釋出資料空間。 您也應該留意壓縮資料庫檔案的潛在負面效能影響，請參閱下面的 [**重建索引**](#rebuild-indexes) 一節。

## <a name="query-an-elastic-pool-for-storage-space-information"></a>查詢彈性集區，以取得儲存體空間資訊

下列查詢可用來判斷彈性集區的儲存體空間數量。  

### <a name="elastic-pool-data-space-used"></a>使用的彈性集區資料空間

修改下列查詢，以傳回已使用的彈性集區資料空間量。  查詢結果以 MB 為單位。

```sql
-- Connect to master
-- Elastic pool data space used in MB  
SELECT TOP 1 avg_storage_percent / 100.0 * elastic_pool_storage_limit_mb AS ElasticPoolDataSpaceUsedInMB
FROM sys.elastic_pool_resource_stats
WHERE elastic_pool_name = 'ep1'
ORDER BY end_time DESC;
```

### <a name="elastic-pool-data-space-allocated-and-unused-allocated-space"></a>配置的彈性集區資料空間與已配置但未使用的空間

修改下列範例，以傳回資料表，其中列出彈性集區中每個資料庫所配置的空間和未使用的配置空間。 此資料表會根據資料庫已配置但未使用的空間量，從最大至最小來排序資料庫。  查詢結果以 MB 為單位。  

以查詢判斷為集區中的每個資料庫配置的空間所產生的結果，可在加總後用來判斷為彈性集區配置的總空間。 配置的彈性集區空間不應超過彈性集區大小上限。  

> [!IMPORTANT]
> Azure SQL Database 仍然支援 PowerShell Azure Resource Manager 模組，但所有未來的開發都是針對 Az.Sql 模組。 AzureRM 模組在至少 2020 年 12 月之前都還會持續收到 Bug 修正。 Az 模組和 AzureRm 模組中命令的引數本質上完全相同。 如需其相容性的詳細資訊，請參閱[新的 Azure PowerShell Az 模組簡介](/powershell/azure/new-azureps-module-az)。

PowerShell 指令碼需要 SQL Server PowerShell 模組 – 請參閱[下載 PowerShell 模組](/sql/powershell/download-sql-server-ps-module)以便安裝。

```powershell
$resourceGroupName = "<resourceGroupName>"
$serverName = "<serverName>"
$poolName = "<poolName>"
$userName = "<userName>"
$password = "<password>"

# get list of databases in elastic pool
$databasesInPool = Get-AzSqlElasticPoolDatabase -ResourceGroupName $resourceGroupName `
    -ServerName $serverName -ElasticPoolName $poolName
$databaseStorageMetrics = @()

# for each database in the elastic pool, get space allocated in MB and space allocated unused in MB
foreach ($database in $databasesInPool) {
    $sqlCommand = "SELECT DB_NAME() as DatabaseName, `
    SUM(size/128.0) AS DatabaseDataSpaceAllocatedInMB, `
    SUM(size/128.0 - CAST(FILEPROPERTY(name, 'SpaceUsed') AS int)/128.0) AS DatabaseDataSpaceAllocatedUnusedInMB `
    FROM sys.database_files `
    GROUP BY type_desc `
    HAVING type_desc = 'ROWS'"
    $serverFqdn = "tcp:" + $serverName + ".database.windows.net,1433"
    $databaseStorageMetrics = $databaseStorageMetrics + 
        (Invoke-Sqlcmd -ServerInstance $serverFqdn -Database $database.DatabaseName `
            -Username $userName -Password $password -Query $sqlCommand)
}

# display databases in descending order of space allocated unused
Write-Output "`n" "ElasticPoolName: $poolName"
Write-Output $databaseStorageMetrics | Sort -Property DatabaseDataSpaceAllocatedUnusedInMB -Descending | Format-Table
```

下列螢幕擷取畫面是指令碼輸出的其中一個範例：

![彈性集區的配置空間和未使用配置空間範例](./media/file-space-manage/elastic-pool-allocated-unused.png)

### <a name="elastic-pool-data-max-size"></a>彈性集區資料大小上限

修改下列 T-SQL 查詢，以傳回上次記錄的彈性集區資料大小上限。  查詢結果以 MB 為單位。

```sql
-- Connect to master
-- Elastic pools max size in MB
SELECT TOP 1 elastic_pool_storage_limit_mb AS ElasticPoolMaxSizeInMB
FROM sys.elastic_pool_resource_stats
WHERE elastic_pool_name = 'ep1'
ORDER BY end_time DESC;
```

## <a name="reclaim-unused-allocated-space"></a>回收未使用的配置空間

> [!NOTE]
> 壓縮命令會影響資料庫在執行時的效能，並盡可能在低使用量期間執行。

### <a name="dbcc-shrink"></a>DBCC 壓縮

在識別哪些資料庫要回收已配置但未使用的空間後，請修改下列命令中的資料庫名稱，以壓縮每個資料庫的資料檔案。

```sql
-- Shrink database data space allocated.
DBCC SHRINKDATABASE (N'db1');
```

壓縮命令會影響資料庫在執行時的效能，並盡可能在低使用量期間執行。  

您也應該留意壓縮資料庫檔案的潛在負面效能影響，請參閱下面的 [**重建索引**](#rebuild-indexes) 一節。

如需有關此命令的詳細資訊，請參閱 [SHRINKDATABASE](/sql/t-sql/database-console-commands/dbcc-shrinkdatabase-transact-sql.md)。

### <a name="auto-shrink"></a>自動壓縮

或者，可以啟用資料庫自動壓縮。  自動壓縮會減少檔案管理複雜度，且比起 `SHRINKDATABASE` 或 `SHRINKFILE` 較不影響資料庫效能。  自動壓縮對於管理包含許多資料庫的彈性集區可能特別有用。  不過，自動壓縮比起 `SHRINKDATABASE` 和 `SHRINKFILE` 在回收檔案空間上可能較沒效率。
根據預設，針對大部分的資料庫，預設會停用自動壓縮。 如需詳細資訊，請參閱 [AUTO_SHRINK 的考慮](/troubleshoot/sql/admin/considerations-autogrow-autoshrink#considerations-for-auto_shrink)。

若要啟用自動壓縮，請修改下列命令中的資料庫名稱。

```sql
-- Enable auto-shrink for the database.
ALTER DATABASE [db1] SET AUTO_SHRINK ON;
```

如需有關此命令的詳細資訊，請參閱 [DATABASE SET](/sql/t-sql/statements/alter-database-transact-sql-set-options) 選項。

### <a name="rebuild-indexes"></a>重建索引

在資料庫資料檔案壓縮後，索引可能會變得分散而失去效能最佳化效益。 如果發生效能降低的情形，請考慮重建資料庫索引。 如需索引分散和如何重建索引的詳細資訊，請參閱[重新組織與重建索引](/sql/relational-databases/indexes/reorganize-and-rebuild-indexes)。

## <a name="next-steps"></a>後續步驟

- 如需資料庫大小上限的相關資訊，請參閱：
  - [適用於單一資料庫的 Azure SQL Database 以虛擬核心為基礎的購買模型限制](resource-limits-vcore-single-databases.md)
  - [使用以 DTU 為基礎的購買模型的單一資料庫資源限制](resource-limits-dtu-single-databases.md)
  - [針對彈性集區，Azure SQL Database 以虛擬核心為基礎的購買模型限制](resource-limits-vcore-elastic-pools.md)
  - [使用以 DTU 為基礎的購買模型的彈性集區資源限制](resource-limits-dtu-elastic-pools.md)
- 如需有關 `SHRINKDATABASE` 命令的詳細資訊，請參閱 [SHRINKDATABASE](/sql/t-sql/database-console-commands/dbcc-shrinkdatabase-transact-sql)。
- 如需索引分散和如何重建索引的詳細資訊，請參閱[重新組織與重建索引](/sql/relational-databases/indexes/reorganize-and-rebuild-indexes)。