---
title: 2018 年 6 月 Azure SQL 資料倉儲版本資訊 | Microsoft Docs
description: Azure SQL 資料倉儲的版本資訊。
services: sql-data-warehouse
author: twounder
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 07/23/2018
ms.author: twounder
ms.reviewer: twounder
ms.openlocfilehash: 6dff2d1886a560e3a7e2758816a884782e33787f
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287965"
---
# <a name="whats-new-in-azure-sql-data-warehouse-june-2018"></a>Azure SQL 資料倉儲有哪些最新功能？ 2018 年 6 月
Azure SQL 資料倉儲會持續改進。 本文說明 2018 年 6 月導入的新功能和變更。 

## <a name="user-defined-restore-points"></a>使用者定義的還原點
SQL 資料倉儲每 8 小時自動拍攝資料倉儲的快照集，保證八小時復原點目標 (RPO)。 雖然此自動快照集可以減輕執行資料倉儲的管理負擔，但仍需要根據業務需求，在關鍵時刻拍攝快照集。 例如，在重要資料載入之前拍攝快照集或將新指令碼部署到資料倉儲中，以在作業之前啟用還原點。 

SQL 資料倉儲目前透過 [New-AzureRmSqlDatabaseRestorePoint](https://docs.microsoft.com/powershell/module/azurerm.sql/new-azurermsqldatabaserestorepoin) Cmdlet 支援[使用者定義的還原點](https://azure.microsoft.com/blog/quick-recovery-time-with-sql-data-warehouse-using-user-defined-restore-points/)。

```PowerShell
New-AzureRmSqlDatabaseRestorePoint
    -ResourceGroupName $ResourceGroupName
    -ServerName $ServerName
    -DatabaseName $DatabaseName
    -RestorePointLabel $RestorePointName
```

## <a name="column-level-security"></a>資料行層級安全性
管理資料倉儲中的資料存取和安全性，對於建立客戶和合作夥伴之間的信任至關重要。 SQL 資料倉儲[目前支援資料行層級安全性 (CLS)](https://azure.microsoft.com/blog/column-level-security-is-now-supported-in-azure-sql-data-warehouse/)，可讓您透過限制使用者對資料表中特定資料行的存取權，來調整檢視敏感性資料的權限，而無需重新設計資料倉儲。

CLS 可讓您使用標準的 [GRANT](https://docs.microsoft.com/azure/sql-data-warehouse/column-level-security) T-SQL 陳述式，根據使用者的執行內容或其群組成員資格來控制資料表資料行的存取。 存取限制邏輯位於資料庫層本身，而不是遠離另一個應用程式中的資料，簡化了整體安全性實作。


```sql
CREATE USER Nurse WITHOUT LOGIN;   
GRANT SELECT ON Membership(MemberID, FirstName, LastName, Phone, Email) TO Nurse;   
EXECUTE AS USER = 'Nurse';
SELECT * FROM Membership;

Msg 230, Level 14, State 1, Line 12 
The SELECT permission was denied on the column 'SSN' of the object 'Membership', database 'CLS_TestDW', schema 'dbo'.
```

## <a name="objectschemaname"></a>OBJECT_SCHEMA_NAME
[OBJECT_SCHEMA_NAME()]() 函式會傳回結構描述範圍物件的資料庫結構描述名稱。 在進行物件結構描述驗證時，此函式在一般 ETL 工具中很常見。 

```sql
SELECT
    OBJECT_SCHEMA_NAME([object_id]) [table_schema]
    , [name]                        [table_name]
FROM
    [sys].[tables];
```

**範例結果**
```
table_schema    | table_name
-----------------------------
dbo               customer
dbo               lineitem
dbo               nation
dbo               orders
```

## <a name="support-for-the-systimezoneinfo-view"></a>支援 sys.time_zone_info 檢視
[sys.time_zone_info](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-time-zone-info-transact-sql) 檢視會傳回 Azure SQL 資料倉儲中支援時區的相關資訊。

```sql
SELECT * FROM [sys].[time_zone_info];
```

**範例結果**
```
name                            | current_utc_offset | is_currently_dst
-------------------------------------------------------------------------
Pacific Standard Time (Mexico)    -07:00               1
US Mountain Standard Time         -07:00               0
Mountain Standard Time (Mexico)   -06:00               1
Central Standard Time             -05:00               1
```

## <a name="auto-stats-operations-appear-in-sysdmpdwexecrequests-behavior-change"></a>Auto Stats 作業會顯示在 sys.dm_pdw_exec_requests (行為變更)

隨著[自動建立統計資料](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-statistics#automatic-creation-of-statistics)的推出，Azure SQL 資料倉儲將產生統計資料以最佳化查詢執行。 2018 年 6 月版新增監控何時自動產生統計資料的功能，只要執行任何 [CREATE STATISTICS](https://docs.microsoft.com/sql/t-sql/statements/create-statistics-transact-sql) 作業，就會將記錄新增至 [sys.dm_pdw_exec_requests](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql) 檢視中。

```sql
SELECT
    [start_time]
    , [end_time]
    , [command]
FROM
    [sys].[dm_pdw_exec_requests]
WHERE
    [command] LIKE 'CREATE STATISTICS _WA_Sys%';
```
**範例結果**
```
start_time                | end_time                | command
------------------------------------------------------------------------------------------------------------------------------
2018-06-06 19:06:26.173    2018-06-06 19:06:26.173    CREATE STATISTICS _WA_Sys_00000001_63D998CC ON dbo.LineItem(l_orderkey);
```

## <a name="next-steps"></a>後續步驟
現在您已稍微了解 SQL 資料倉儲，請了解如何快速[建立 SQL 資料倉儲][create a SQL Data Warehouse]。 如果您不熟悉 Azure，您可能會發現 [Azure 詞彙][Azure glossary]在您遇到新術語時很有幫助。 或者，也可以看一下其中一些其他 SQL 資料倉儲資源。  

* [客戶成功案例]
* [部落格]
* [功能要求]
* [影片]
* [客戶諮詢小組部落格]
* [Stack Overflow 論壇]
* [Twitter]


[部落格]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[客戶諮詢小組部落格]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[客戶成功案例]: https://azure.microsoft.com/case-studies/?service=sql-data-warehouse
[功能要求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[Stack Overflow 論壇]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[影片]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
[create a SQL Data Warehouse]: ./create-data-warehouse-portal.md
[Azure glossary]: ../azure-glossary-cloud-terminology.md