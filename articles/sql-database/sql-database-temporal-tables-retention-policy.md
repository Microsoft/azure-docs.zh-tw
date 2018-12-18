---
title: 使用保留原則管理時態表中的歷史資料 | Microsoft Docs
description: 了解如何使用時態保留原則來控制歷史資料。
services: sql-database
author: bonova
manager: craigg
ms.service: sql-database
ms.custom: develop databases
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: bonova
ms.openlocfilehash: f65f7ec44ccbeb6f64d43d20b1bd7a77329fa97f
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34649018"
---
# <a name="manage-historical-data-in-temporal-tables-with-retention-policy"></a>使用保留原則管理時態表中的歷史資料
時態表會讓資料庫變得更大，特別是當您保留一段很長時間的歷史資料時，而一般資料表還不至於讓資料庫變得這麼大。 因此，在規劃及管理每個時態表的生命週期時，歷史資料的保留原則很重要。 Azure SQL Database 中的時態表隨附方便使用的保留機制，可協助您完成這項工作。

使用者可以在個別的資料表層級設定時態記錄保留，以建立有彈性的過時清除原則。 套用時態保留很簡單︰只需要在建立資料表或變更結構描述時設定一個參數。

定義保留原則之後，Azure SQL Database 會開始定期檢查是否有適合自動清除資料的歷史資料列。 系統排定和執行的背景工作會自動識別相符的資料列，並從記錄資料表中移除。 將會根據代表 SYSTEM_TIME 時段結束的資料行，檢查記錄資料表資料列的過時條件。 例如，如果保留期限設定為六個月，則適合清除的資料表資料列滿足下列條件︰

````
ValidTo < DATEADD (MONTH, -6, SYSUTCDATETIME())
````

在上述範例中，我們假設 **ValidTo** 資料行對應於 SYSTEM_TIME 時段結束。

## <a name="how-to-configure-retention-policy"></a>如何設定保留原則？
在設定時態表的保留原則之前，請先檢查是否已在「資料庫層級」啟用時態記錄保留。

````
SELECT is_temporal_history_retention_enabled, name
FROM sys.databases
````

根據預設，資料庫旗標 **is_temporal_history_retention_enabled** 會設為 ON，但使用者可以利用 ALTER DATABASE 陳述式變更它。 它也會在[還原時間點](sql-database-recovery-using-backups.md)作業之後自動設為 OFF。 若要對資料庫啟用時態記錄保留清除，請執行下列陳述式︰

````
ALTER DATABASE <myDB>
SET TEMPORAL_HISTORY_RETENTION  ON
````

> [!IMPORTANT]
> 即使 **is_temporal_history_retention_enabled** 為 OFF，您還是可以設定時態表的保留，但在此情況下，不會觸發自動清除過時的資料列。
> 
> 

在建立資料表期間，您可以指定 HISTORY_RETENTION_PERIOD 參數的值來設定保留原則︰

````
CREATE TABLE dbo.WebsiteUserInfo
(  
    [UserID] int NOT NULL PRIMARY KEY CLUSTERED
  , [UserName] nvarchar(100) NOT NULL
  , [PagesVisited] int NOT NULL
  , [ValidFrom] datetime2 (0) GENERATED ALWAYS AS ROW START
  , [ValidTo] datetime2 (0) GENERATED ALWAYS AS ROW END
  , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
 )  
 WITH
 (
     SYSTEM_VERSIONING = ON
     (
        HISTORY_TABLE = dbo.WebsiteUserInfoHistory,
        HISTORY_RETENTION_PERIOD = 6 MONTHS
     )
 );
````

Azure SQL Database 可讓您使用不同的時間單位來指定保留期限︰DAYS、WEEKS、MONTHS 年 YEARS。 如果省略 HISTORY_RETENTION_PERIOD，則假設為 INFINITE 保留。 您也可以明確使用 INFINITE 關鍵字。

在某些情況下，您可能想要在建立資料表之後設定保留，或變更先前設定的值。 在此情況下，請使用 ALTER TABLE 陳述式︰

````
ALTER TABLE dbo.WebsiteUserInfo
SET (SYSTEM_VERSIONING = ON (HISTORY_RETENTION_PERIOD = 9 MONTHS));
````

> [!IMPORTANT]
> 將 SYSTEM_VERSIONING 設定為 OFF「不會保留」保留期限值。 如果將 SYSTEM_VERSIONING 設為 ON，但未明確指定 HISTORY_RETENTION_PERIOD，則會形成 INFINITE 保留期限。
> 
> 

若要檢閱保留原則的目前狀態，請使用下列查詢，其中結合資料庫層級的時態保留啟用旗標和個別資料表的保留期限︰

````
SELECT DB.is_temporal_history_retention_enabled,
SCHEMA_NAME(T1.schema_id) AS TemporalTableSchema,
T1.name as TemporalTableName,  SCHEMA_NAME(T2.schema_id) AS HistoryTableSchema,
T2.name as HistoryTableName,T1.history_retention_period,
T1.history_retention_period_unit_desc
FROM sys.tables T1  
OUTER APPLY (select is_temporal_history_retention_enabled from sys.databases
where name = DB_NAME()) AS DB
LEFT JOIN sys.tables T2   
ON T1.history_table_id = T2.object_id WHERE T1.temporal_type = 2
````


## <a name="how-sql-database-deletes-aged-rows"></a>SQL Database 如何刪除過時資料列？
清除程序取決於記錄資料表的索引配置。 請務必注意，具有叢集索引 (B 型樹狀目錄或資料行存放區) 的記錄資料表才能設定有限保留原則。 對於設定有限保留期限的所有時態表，將會建立背景工作來清除過時資料。
資料列存放區 (B 型樹狀目錄) 叢集索引的清除邏輯會以較小區塊刪除過時資料列 (最多 10K)，以儘量減輕資料庫記錄檔和 IO 子系統的壓力。 雖然清除邏輯採用必要的 B 型樹狀目錄索引，但在刪除比保留期限更舊的資料列時，就無法確實保證刪除順序。 因此，在您的應用程式中，絕對不要依賴清除順序。

叢集資料行存放區的清除工作會一次移除整個[資料列群組](https://msdn.microsoft.com/library/gg492088.aspx) (每個通常包含 1 百萬個資料列)，這非常有效率，特別當歷史資料快速產生時。

![叢集資料行存放區保留](./media/sql-database-temporal-tables-retention-policy/cciretention.png)

當您的工作負載快速產生大量記錄資料時，絕佳的資料壓縮和有效率的保留清除，使得叢集資料行存放區索引成為最佳選擇。 此模式通常用於需要[使用時態表的大量交易處理工作負載](https://msdn.microsoft.com/library/mt631669.aspx)，以追蹤和稽核變更、分析趨勢，或擷取 IoT 資料。

## <a name="index-considerations"></a>索引考量
對於具有資料列存放區叢集索引的資料表，清除工作需有從對應 SYSTEM_TIME 時段結束的資料行開始的索引。 如果沒有這種索引，您就無法設定有限保留期限：

*訊息 13765，層級 16，狀態 1 <br></br> 在由系統設定版本的時態表 'temporalstagetestdb.dbo.WebsiteUserInfo' 上，設定有限的保留期限失敗，因為記錄資料表 'temporalstagetestdb.dbo.WebsiteUserInfoHistory' 不包含必要的叢集索引。在記錄資料表上，請考慮建立從符合 SYSTEM_TIME 時段結束的資料行開始的叢集資料行存放區或 B 型樹狀目錄索引。*

請務必注意，Azure SQL Database 建立的預設記錄資料表已有相容於保留原則的叢集索引。 如果您嘗試在設定有限保留期限的資料表上移除該索引，作業會失敗並傳回下列錯誤︰

*訊息 13766，層級 16，狀態 1 <br></br> 無法卸除叢集索引 'WebsiteUserInfoHistory.IX_WebsiteUserInfoHistory'，因為它正用於自動清除過時資料。如果您需要卸除此索引，請在由系統設定版本的對應時態表上，考慮將 HISTORY_RETENTION_PERIOD 設定為 INFINITE。*

如果歷史資料列依遞增順序插入 (依時段結束資料行排序)，完全以 SYSTEM_VERSIONIOING 機制填入記錄資料表時就是如此，則會以最佳方式清除叢集資料行存放區索引。 如果記錄資料表中的資料列未依時段結束資料行排序 (可能是您已移轉現有的歷史資料)，您應該在已正確排序的 B 型樹狀目錄資料列存放區索引上，重新建立叢集資料行存放區索引，以發揮最佳效能。

在設定有限保留期限的記錄資料表上，請避免重建叢集資料行存放區索引，因為這可能會變更系統版本控制作業在資料列群組中自然強加的順序。 如果您需要在記錄資料表上重建叢集資料行存放區索引，請在相容的 B 型樹狀目錄索引之外重建，讓需要定期清除資料的資料列群組中可以保留順序。 如果您使用具有叢集資料行索引但不保證資料順序的現有記錄資料表來建立時態表，則應該採取相同的方法︰

````
/*Create B-tree ordered by the end of period column*/
CREATE CLUSTERED INDEX IX_WebsiteUserInfoHistory ON WebsiteUserInfoHistory (ValidTo)
WITH (DROP_EXISTING = ON);
GO
/*Re-create clustered columnstore index*/
CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory ON WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON);
````

當具有叢集資料行存放區索引的記錄資料表已設定有限保留期限時，您無法在該資料表上建立其他非叢集 B 型樹狀目錄索引︰

````
CREATE NONCLUSTERED INDEX IX_WebHistNCI ON WebsiteUserInfoHistory ([UserName])
````

嘗試執行上述陳述式失敗並傳回下列錯誤︰

*訊息 13772，層級 16，狀態 1 <br></br> 無法在時態記錄資料表 'WebsiteUserInfoHistory' 上建立非叢集索引，因為它己定義有限保留期限和叢集資料行存放區索引。*

## <a name="querying-tables-with-retention-policy"></a>使用保留原則來查詢資料表
時態表上的所有查詢會自動篩選掉符合有限保留原則的歷史資料列，以避免非預期和不一致的結果，因為清除工作可以「隨時以任意順序」刪除過時資料列。

下圖顯示簡單查詢的查詢計劃︰

````
SELECT * FROM dbo.WebsiteUserInfo FOR SYSTEM_TIME ALL;
````

查詢計劃包含額外的篩選，在「叢集索引掃描」運算子中套用到記錄資料表 (反白顯示) 的時段結束資料行 (ValidTo)。 這個範例假設 WebsiteUserInfo 資料表上已設定一個 MONTH 保留期限。

![保留查詢篩選](./media/sql-database-temporal-tables-retention-policy/queryexecplanwithretention.png)

不過，如果您直接查詢記錄資料表，您可能會看到超過指定保留期限的資料列，但完全不保證會得到重複的查詢結果。 下圖顯示在記錄資料表上查詢的查詢執行計劃，但未套用額外篩選︰

![在不使用保留篩選的情況下查詢記錄](./media/sql-database-temporal-tables-retention-policy/queryexecplanhistorytable.png)

請勿依賴商務邏輯來讀取超出保留期限的記錄資料表，否則可能會得到不一致或非預期的結果。 建議以 FOR SYSTEM_TIME 子句來使用暫時查詢，以分析時態表中的資料。

## <a name="point-in-time-restore-considerations"></a>還原時間點考量
當您[將現有資料庫還原到特定時間點](sql-database-recovery-using-backups.md)來建立新的資料庫時，資料庫層級上會停用時態保留  (**is_temporal_history_retention_enabled** 旗標設為 OFF)。 這項功能可讓您在還原時檢查所有歷史資料列，而不必擔心過時資料列在您開始查詢之前就被移除。 這可用來「檢查超出設定保留期限的歷史資料」。

假設時態表已指定一個 MONTH 保留期限。 如果是在高階服務層中建立資料庫，您可以使用過去最多 35 天的資料庫狀態來建立資料庫副本。 實際上，這可讓您直接查詢記錄資料表，以分析過去最多 65 天的歷史資料列。

如果您想要啟動時態保留清除，請在還原時間點之後執行下列 TRANSACT-SQL 陳述式︰

````
ALTER DATABASE <myDB>
SET TEMPORAL_HISTORY_RETENTION  ON
````

## <a name="next-steps"></a>後續步驟
若要了解如何在應用程式中使用時態表，請參閱[開始使用 Azure SQL Database 中的時態表](sql-database-temporal-tables.md)。

瀏覽 Channel 9，聽聽[真實客戶的時態表實作成功案例](https://channel9.msdn.com/Blogs/jsturtevant/Azure-SQL-Temporal-Tables-with-RockStep-Solutions)，並觀看[時態表的即時示範](https://channel9.msdn.com/Shows/Data-Exposed/Temporal-in-SQL-Server-2016)。

如需有關時態表的詳細資訊，請檢閱 [MSDN 文件](https://msdn.microsoft.com/library/dn935015.aspx)。

