---
title: Azure SQL Database 受控執行個體的 T-SQL 差異 | Microsoft Docs
description: 本文將討論 Azure SQL Database 受控執行個體與 SQL Server之間的 T-SQL 差異。
services: sql-database
author: jovanpop-msft
ms.reviewer: carlrab, bonova
ms.service: sql-database
ms.custom: managed instance
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: jovanpop
manager: craigg
ms.openlocfilehash: 89544ea72a4356fb8d4f3a192e6fc546eb6b3cff
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39092005"
---
# <a name="azure-sql-database-managed-instance-t-sql-differences-from-sql-server"></a>Azure SQL Database 受控執行個體的 T-SQL 差異 

Azure SQL Database 受控執行個體 (預覽) 可與內部部署 SQL Server 資料庫引擎高度相容。 受控執行個體支援大部分的 SQL Server 資料庫引擎功能。 由於仍有一些語法和行為上的差異，因此本文將摘要說明這些差異。
 - [T-SQL 差異與不支援的功能](#Differences)
 - [受控執行個體中具有不同行為的功能](#Changes)
 - [暫時性限制與已知問題](#Issues)

## <a name="Differences"></a>與 SQL Server 之間的 T-SQL 差異 

本節摘要說明受控執行個體和內部部署 SQL Server 資料庫引擎在 T-SQL 語法與行為上的主要差異，以及不支援的功能。

### <a name="always-on-availability"></a>Always-On 可用性

受控執行個體內建[高可用性](sql-database-high-availability.md)，並且無法由使用者控制。 不支援下列陳述式︰
 - [CREATE ENDPOINT … FOR DATABASE_MIRRORING](https://docs.microsoft.com/sql/t-sql/statements/create-endpoint-transact-sql)
 - [CREATE AVAILABILITY GROUP](https://docs.microsoft.com/sql/t-sql/statements/create-availability-group-transact-sql)
 - [ALTER AVAILABILITY GROUP](https://docs.microsoft.com/sql/t-sql/statements/alter-availability-group-transact-sql)
 - [DROP AVAILABILITY GROUP](https://docs.microsoft.com/sql/t-sql/statements/drop-availability-group-transact-sql)
 - [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql)陳述式的 [SET HADR](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-hadr) 子句

### <a name="auditing"></a>稽核 
 
SQL 稽核在受控執行個體、Azure SQL Database 和 SQL Server 內部部署之間的主要差異如下：
- 在受控執行個體中，SQL 稽核會在伺服器層級運作，並將 `.xel` 檔案存放在 Azure Blob 儲存體帳戶。  
- 在 Azure SQL Database 中，SQL 稽核則在資料庫層級運作。
- 在 SQL Server 內部部署 / 虛擬機器中，SQL 稽核會在伺服器層級運作，但會將事件儲存在檔案系統/windows 事件記錄。  
  
受控執行個體中的 XEvent 稽核支援 Azure Blob 儲存體目標。 不支援檔案與 windows 記錄。    
 
向 Azure Blob 儲存體進行稽核的 `CREATE AUDIT` 語法有一個主要差異：
- 已提供新的 `TO URL` 語法，可讓您指定放置 `.xel` 檔案的 Azure Blob 儲存體容器 URL 
- `TO FILE` 語法不受支援，因為受控執行個體無法存取 Windows 檔案共用。 
 
如需詳細資訊，請參閱  
- [CREATE SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/create-server-audit-transact-sql)  
- [ALTER SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/alter-server-audit-transact-sql) 
- [稽核](https://docs.microsoft.com/sql/relational-databases/security/auditing/sql-server-audit-database-engine)     

### <a name="backup"></a>Backup  

受控執行個體有自動備份，且可讓使用者建立完整資料庫的 `COPY_ONLY` 備份。 不支援差異、記錄及檔案快照集備份。  
- 受控執行個體只可將資料庫備份至 Azure Blob 儲存體帳戶： 
 - 只支援 `BACKUP TO URL` 
 - 不支援 `FILE`、`TAPE`及備份裝置  
- 支援大部分一般的 `WITH` 選項 
 - `COPY_ONLY` 是必要的
 - 不支援 `FILE_SNAPSHOT`  
 - 磁帶選項：不支援 `REWIND`、`NOREWIND`、`UNLOAD` 和 `NOUNLOAD` 
 - 日誌專用選項：不支援 `NORECOVERY`、`STANDBY` 和 `NO_TRUNCATE` 
 
限制：  
- 受控執行個體可以將資料庫備份至最多具有 32 個條帶 (Stripe) 的備份，這足夠最多 4 TB 的資料庫使用 (如果使用備份壓縮)。
- 條帶 (Stripe) 大小的上限為 195 GB (Blob 大小的上限)。 在備份命令中增加條帶 (Stripe) 數目，可減少個別條帶 (Stripe) 的大小並維持在此限制內。 

> [!TIP]
> 若要在內部部署中解決此限制，請備份至 `DISK` (而不是備份至 `URL`)，並上傳至 Blob，然後進行還原。 還原作業支援更大的檔案，因為使用不同的 Blob 類型。  

如需使用 T-SQL 進行備份的相關資訊，請參閱[備份](https://docs.microsoft.com/sql/t-sql/statements/backup-transact-sql)。

### <a name="buffer-pool-extension"></a>緩衝集區延伸 
 
- [不支援緩衝集區延伸](https://docs.microsoft.com/sql/database-engine/configure-windows/buffer-pool-extension)。
- 不支援 `ALTER SERVER CONFIGURATION SET BUFFER POOL EXTENSION`。 請參閱 [ALTER SERVER CONFIGURATION](https://docs.microsoft.com/sql/t-sql/statements/alter-server-configuration-transact-sql)。 
 
### <a name="bulk-insert--openrowset"></a>大量插入 / openrowset

受控執行個體無法存取檔案共用及 Windows 資料夾，因此必須從 Azure Blob 儲存體匯入這些檔案。
- 從 Azure Blob 儲存體匯入檔案時，`BULK INSERT` 命令中需要 `DATASOURCE`。 請參閱[大量插入](https://docs.microsoft.com/sql/t-sql/statements/bulk-insert-transact-sql)。
- 當您從 Azure Blob 儲存體讀取檔案的內容時，`OPENROWSET` 函式需要 `DATASOURCE`。 請參閱 [OPENROWSET](https://docs.microsoft.com/sql/t-sql/functions/openrowset-transact-sql)。
 
### <a name="certificates"></a>憑證 

受控執行個體無法存取檔案共用及 Windows 資料夾，因此會有下列限制： 
- 憑證不支援 `CREATE FROM`/`BACKUP TO` 檔案
- 不支援來自 `FILE`/`ASSEMBLY` 的 `CREATE`/`BACKUP` 憑證。 無法使用私密金鑰檔案。  
 
請參閱 [CREATE CERTIFICATE](https://docs.microsoft.com/sql/t-sql/statements/create-certificate-transact-sql) 和 [BACKUP CERTIFICATE](https://docs.microsoft.com/sql/t-sql/statements/backup-certificate-transact-sql)。  
  
> [!TIP]
> 因應措施：指令碼憑證/私密金鑰，儲存為 .sql 檔案，並從二進位檔建立： 
> 
> ``` 
CREATE CERTIFICATE  
 FROM BINARY = asn_encoded_certificate    
WITH PRIVATE KEY ( <private_key_options> ) 
>```   
 
### <a name="clr"></a>CLR 

受控執行個體無法存取檔案共用及 Windows 資料夾，因此會有下列限制： 
- 只支援 `CREATE ASSEMBLY FROM BINARY`。 請參閱 [CREATE ASSEMBLY FROM BINARY](https://docs.microsoft.com/sql/t-sql/statements/create-assembly-transact-sql)。  
- 不支援 `CREATE ASSEMBLY FROM FILE`。 請參閱 [CREATE ASSEMBLY FROM FILE](https://docs.microsoft.com/sql/t-sql/statements/create-assembly-transact-sql)。
- `ALTER ASSEMBLY` 無法參考檔案。 請參閱 [ALTER ASSEMBLY](https://docs.microsoft.com/sql/t-sql/statements/alter-assembly-transact-sql)。
 
### <a name="compatibility-levels"></a>相容性層級 
 
- 支援的相容性層級：100、110、120、130、140  
- 不支援低於 100 的相容性層級。 
- 新資料庫的預設相容性層級為 140。 針對已還原的資料庫，如果相容性層級為 100 或更高，則維持不變。

請參閱 [ALTER DATABASE 相容性層級](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level)。
 
### <a name="credential"></a>認證 
 
僅支援Azure Key Vault 和 `SHARED ACCESS SIGNATURE` 身分識別。 不支援 Windows 使用者。
 
請參閱 [CREATE CREDENTIAL](https://docs.microsoft.com/sql/t-sql/statements/create-credential-transact-sql) 和 [ALTER CREDENTIAL](https://docs.microsoft.com/sql/t-sql/statements/alter-credential-transact-sql)。 
 
### <a name="cryptographic-providers"></a>密碼編譯提供者

受控執行個體無法存取檔案，所以無法建立密碼編譯提供者：
- 不支援 `CREATE CRYPTOGRAPHIC PROVIDER`。 請參閱 [CREATE CRYPTOGRAPHIC PROVIDER](https://docs.microsoft.com/sql/t-sql/statements/create-cryptographic-provider-transact-sql)。
- 不支援 `ALTER CRYPTOGRAPHIC PROVIDER`。 請參閱 [ALTER CRYPTOGRAPHIC PROVIDER](https://docs.microsoft.com/sql/t-sql/statements/alter-cryptographic-provider-transact-sql)。 

### <a name="collation"></a>Collation 
 
伺服器定序是 `SQL_Latin1_General_CP1_CI_AS`，且無法變更。 請參閱[定序](https://docs.microsoft.com/sql/t-sql/statements/collations)。
 
### <a name="database-options"></a>資料庫選項 
 
- 不支援多個記錄檔。 
- 「一般用途」服務層中不支援記憶體內部物件。  
- 限制每個執行個體 280 個檔案，也就是每個資料庫最多 280 個檔案。 資料和記錄檔都會計入此限制。  
- 資料庫不能包含具有 Filestream 資料的檔案群組。  如果 .bak 包含 `FILESTREAM` 資料，還原將會失敗。  
- 每個檔案都位於 Azure 進階儲存體中。 每個檔案的 IO 和輸送量取決於每個個別檔案的大小，如同用於 Azure 進階儲存體磁碟的方式。 請參閱 [Azure 進階磁碟效能](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage-performance#premium-storage-disk-sizes)  
 
#### <a name="create-database-statement"></a>CREATE DATABASE 陳述式

以下是 `CREATE DATABASE` 的限制： 
- 無法定義檔案和檔案群組。  
- 不支援 `CONTAINMENT` 選項。  
- 不支援 `WITH` 選項。  
   > [!TIP]
   > 因應措施是在 `CREATE DATABASE` 之後使用 `ALTER DATABASE` 來設定資料庫選項以新增檔案，或設定內含項目。  

- 不支援 `FOR ATTACH` 選項 
- 不支援 `AS SNAPSHOT OF` 選項 

如需詳細資訊，請參閱 [CREATE DATABASE](https://docs.microsoft.com/sql/t-sql/statements/create-database-sql-server-transact-sql)。

#### <a name="alter-database-statement"></a>ALTER DATABASE 陳述式

有些檔案屬性無法設定或變更：
- `ALTER DATABASE ADD FILE (FILENAME='path')` T-SQL 陳述式中無法指定檔案路徑。 從指令碼中移除 `FILENAME`，因為受控執行個體會自動放置這些檔案。  
- 無法使用 `ALTER DATABASE` 陳述式變更檔案名稱。

以下為預設選項且無法變更： 
- `MULTI_USER` 
- `ENABLE_BROKER ON` 
- `AUTO_CLOSE OFF` 

下列選項無法修改： 
- `AUTO_CLOSE` 
- `AUTOMATIC_TUNING(CREATE_INDEX=ON|OFF)` 
- `AUTOMATIC_TUNING(DROP_INDEX=ON|OFF)` 
- `DISABLE_BROKER` 
- `EMERGENCY` 
- `ENABLE_BROKER` 
- `FILESTREAM` 
- `HADR`   
- `NEW_BROKER` 
- `OFFLINE` 
- `PAGE_VERIFY` 
- `PARTNER` 
- `READ_ONLY` 
- `RECOVERY BULK_LOGGED` 
- `RECOVERY_SIMPLE` 
- `REMOTE_DATA_ARCHIVE`  
- `RESTRICTED_USER` 
- `SINGLE_USER` 
- `WITNESS`

不支援修改名稱。

如需詳細資訊，請參閱 [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-file-and-filegroup-options)。

### <a name="database-mirroring"></a>資料庫鏡像

不支援資料庫鏡像。
 - 不支援 `ALTER DATABASE SET PARTNER` 和 `SET WITNESS` 選項。
 - 不支援 `CREATE ENDPOINT … FOR DATABASE_MIRRORING`。

如需詳細資訊，請參閱 [ALTER DATABASE SET PARTNER and SET WITNESS](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-database-mirroring) 和 [CREATE ENDPOINT … FOR DATABASE_MIRRORING](https://docs.microsoft.com/sql/t-sql/statements/create-endpoint-transact-sql)。

### <a name="dbcc"></a>DBCC 
 
受控執行個體中不支援已在 SQL Server 中啟用但無文件說明的 DBCC 陳述式。
- 不支援 `Trace Flags`。 請參閱[追蹤旗標](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql)。
- 不支援 `DBCC TRACEOFF`。 請參閱 [DBCC TRACEOFF](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-traceoff-transact-sql)。
- 不支援 `DBCC TRACEON`。 請參閱 [DBCC TRACEON](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-traceon-transact-sql)。

### <a name="distributed-transactions"></a>分散式交易

受控執行個體中目前不支援 MSDTC 和[彈性交易](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-transactions-overview)。

### <a name="extended-events"></a>擴充事件 

不支援 XEvent 的某些 Windows 特定目標：
- 不支援 `etw_classic_sync target`。 將 `.xel` 檔案儲存在 Azure Blob 儲存體中。 請參閱 [etw_classic_sync 目標](https://docs.microsoft.com/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#etwclassicsynctarget-target)。 
- 不支援 `event_file target`。 將 `.xel` 檔案儲存在 Azure Blob 儲存體中。 請參閱 [event_file 目標](https://docs.microsoft.com/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#eventfile-target)。

### <a name="external-libraries"></a>外部程式庫

尚不支援資料庫內部 R 與 Python 的外部程式庫。 請參閱 [SQL Server Machine Learning 服務](https://docs.microsoft.com/sql/advanced-analytics/r/sql-server-r-services)。

### <a name="filestream-and-filetable"></a>Filestream 和 Filetable

- 不支援 Filestream 資料。 
- 資料庫不能包含具有 `FILESTREAM` 資料的檔案群組
- 不支援 `FILETABLE`
- 資料表不能有 `FILESTREAM` 類型
- 不支援下列案函式：
 - `GetPathLocator()` 
 - `GET_FILESTREAM_TRANSACTION_CONTEXT()` 
 - `PathName()` 
 - `GetFileNamespacePath()` 
 - `FileTableRootPath()` 

如需詳細資訊，請參閱 [FILESTREAM](https://docs.microsoft.com/sql/relational-databases/blob/filestream-sql-server) 和 [Filetable](https://docs.microsoft.com/sql/relational-databases/blob/filetables-sql-server)。

### <a name="full-text-semantic-search"></a>全文檢索的語意搜尋

不支援[語意搜尋](https://docs.microsoft.com/sql/relational-databases/search/semantic-search-sql-server)。

### <a name="linked-servers"></a>連結的伺服器
 
在受控執行個體中，連結伺服器支援的目標數有限： 
- 支援的目標：SQL Server 及 SQL Database
- 不支援目標：檔案、Analysis Services 和其他 RDBMS。

作業

- 不支援跨執行個體寫入交易。
- 支援使用 `sp_dropserver` 卸除連結的伺服器。 請參閱 [sp_dropserver](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-dropserver-transact-sql)。
- `OPENROWSET` 函式只可用來在 SQL Server 執行個體 (受控、內部部署或在虛擬機器中) 上執行查詢。 請參閱 [OPENROWSET](https://docs.microsoft.com/sql/t-sql/functions/openrowset-transact-sql)。
- `OPENDATASOURCE` 函式只可用來在 SQL Server 執行個體 (受控、內部部署或在虛擬機器中) 上執行查詢。 只支援使用 `SQLNCLI`、`SQLNCLI11` 和 `SQLOLEDB` 值作為提供者。 例如： `SELECT * FROM OPENDATASOURCE('SQLNCLI', '...').AdventureWorks2012.HumanResources.Employee` 。 請參閱 [OPENDATASOURCE](https://docs.microsoft.com/sql/t-sql/functions/opendatasource-transact-sql)。
 
### <a name="logins--users"></a>登入 / 使用者 

- 不支援 `FROM CERTIFICATE`、`FROM ASYMMETRIC KEY` 和 `FROM SID` 建立的 SQL 登入。 請參閱 [CREATE LOGIN](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql)。
- 不支援使用 `CREATE LOGIN ... FROM WINDOWS` 語法建立的 Windows 登入。
- 建立執行個體的 Azure Active Directory (Azure AD) 使用者具有[不受限制的系統管理員權限](https://docs.microsoft.com/azure/sql-database/sql-database-manage-logins#unrestricted-administrative-accounts)。
- 您可以使用 `CREATE USER ... FROM EXTERNAL PROVIDER` 語法建立非系統管理員的 Azure Active Directory (Azure AD) 資料庫層級使用者。 請參閱 [CREATE USER ...FROM EXTERNAL PROVIDER](https://docs.microsoft.com/azure/sql-database/sql-database-manage-logins#non-administrator-users)
 
### <a name="polybase"></a>Polybase

不支援參考 HDFS 或 Azure Blob 儲存體內部檔案的外部資料表。 如需有關 Polybase 的資訊，請參閱 [Polybase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide)。

### <a name="replication"></a>複寫 
 
受控執行個體上支援複寫功能。 如需有關複寫的資訊，請參閱 [SQL Server 複寫](http://review.docs.microsoft.com/sql/relational-databases/replication/replication-with-sql-database-managed-instance)。
 
### <a name="restore-statement"></a>RESTORE 陳述式 
 
- 支援的語法  
   - `RESTORE DATABASE` 
   - `RESTORE FILELISTONLY ONLY` 
   - `RESTORE HEADER ONLY` 
   - `RESTORE LABELONLY ONLY` 
   - `RESTORE VERIFYONLY ONLY` 
- 不支援的語法 
   - `RESTORE LOG ONLY` 
   - `RESTORE REWINDONLY ONLY`
- 來源  
 - `FROM URL` (Azure Blob 儲存體) 是唯一支援的選項。
 - 不支援 `FROM DISK`/`TAPE`/備份裝置。
 - 不支援備份組。 
- 不支援 `WITH` 選項 (沒有 `DIFFERENTIAL`、`STATS` 等項目)。     
- `ASYNC RESTORE` - 即使用戶端連線中斷，還原作業仍會繼續。 如果您的連線中斷，您可以檢查 `sys.dm_operation_status` 檢視，了解還原作業的狀態 (也適用於 CREATE (建立) 和 DROP (捨棄) 資料庫作業)。 請參閱 [sys.dm_operation_status](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database)。  
 
系統會設定/覆寫下列資料庫選項，而且之後無法變更：  
- `NEW_BROKER` (如果 broker.bak 檔案中未啟用 Broker)  
- `ENABLE_BROKER` (如果 broker.bak 檔案中未啟用 Broker)  
- `AUTO_CLOSE=OFF` (如果.bak 檔案中的資料庫具有 `AUTO_CLOSE=ON`)  
- `RECOVERY FULL` (如果.bak 檔案中的資料庫具有 `SIMPLE` 或 `BULK_LOGGED` 鎖定模式)
- 新增記憶體最佳化檔案群組並呼叫 XTP (如果來源 .bak 檔案中沒有的話)  
- 任何現有的記憶體最佳化檔案群組已重新命名為 XTP  
- `SINGLE_USER` 和 `RESTRICTED_USER` 選項會轉換成 `MULTI_USER`   
限制：  
- 無法還原包含多個備份組的 `.BAK` 檔案。 
- 無法還原包含多個記錄檔的 `.BAK` 檔案。 
- 如果 .bak 包含 `FILESTREAM` 資料，還原將會失敗。
- 若備份中包含的資料庫具有作用中的記憶體內部物件，則目前無法還原該備份。  
- 若備份中包含的資料庫在某時間點上有記憶體內部物件，則目前無法還原該備份。   
- 若備份中包含的資料庫處於唯讀模式中，則目前無法還原該備份。 這項限制很快將會移除。   
 
如需有關 Restore 陳述式的資訊，請參閱 [陳述式](https://docs.microsoft.com/sql/t-sql/statements/restore-statements-transact-sql)。

### <a name="service-broker"></a>Service broker 
 
- 不支援跨執行個體的 Service Broker 
 - `sys.routes`必要條件：從 sys.routes 選取位址。 每個路由上的位址必須是 LOCAL。 請參閱 [sys.routes](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-routes-transact-sql)。
 - `CREATE ROUTE` - 您只能使用 `LOCAL` 的 `ADDRESS` 來執行 `CREATE ROUTE`。 請參閱 [CREATE ROUTE](https://docs.microsoft.com/sql/t-sql/statements/create-route-transact-sql)。
 - `ALTER ROUTE` - 您只能使用 `LOCAL` 的 `ADDRESS` 來執行 `ALTER ROUTE`。 請參閱 [ALTER ROUTE](https://docs.microsoft.com/sql/t-sql/statements/alter-route-transact-sql)。  
 
### <a name="service-key-and-service-master-key"></a>服務金鑰和服務主要金鑰 
 
- 不支援[主要金鑰備份](https://docs.microsoft.com/sql/t-sql/statements/backup-master-key-transact-sql) (由 SQL Database 服務管理) 
- 不支援[主要金鑰還原](https://docs.microsoft.com/sql/t-sql/statements/restore-master-key-transact-sql) (由 SQL Database 服務管理) 
- 不支援[服務主要金鑰備份](https://docs.microsoft.com/sql/t-sql/statements/backup-service-master-key-transact-sql) (由 SQL Database 服務管理) 
- 不支援[服務主要金鑰還原](https://docs.microsoft.com/sql/t-sql/statements/restore-service-master-key-transact-sql) (由 SQL Database 服務管理) 
 
### <a name="stored-procedures-functions-triggers"></a>預存程序、函式、觸發程序 
 
- 目前不支援 `NATIVE_COMPILATION`。 
- 不支援下列 [sp_configure](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql) 選項： 
 - `allow polybase export` 
 - `allow updates` 
 - `filestream_access_level` 
 - `max text repl size` 
 - `remote data archive` 
 - `remote proc trans` 
- 不支援 `sp_execute_external_scripts`。 請參閱 [sp_execute_external_scripts](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql#examples)。
- 不支援 `xp_cmdshell`。 請參閱 [xp_cmdshell](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql)。
- 不支援 `Extended stored procedures`，包括 `sp_addextendedproc` 和 `sp_dropextendedproc`。 請參閱[擴充預存程序](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/general-extended-stored-procedures-transact-sql)
- 未支援 `sp_attach_db`、`sp_attach_single_file_db` 和 `sp_detach_db`。 請參閱 [sp_attach_db](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-attach-db-transact-sql)、[sp_attach_single_file_db](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-attach-single-file-db-transact-sql) 和 [sp_detach_db](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-detach-db-transact-sql)。
- 不支援 `sp_renamedb`。 請參閱 [sp_renamedb](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-renamedb-transact-sql)。

### <a name="sql-server-agent"></a>SQL Server 代理程式 
 
- SQL 代理程式設定是唯讀狀態。 受控執行個體中不支援 `sp_set_agent_properties` 程序。  
- 作業 - 目前支援 T-SQL 作業步驟 (公開預覽期間將會新增更多步驟)。
 - 尚不支援 SSIS。 
 - 尚不支援複寫  
  - 尚不支援交易記錄讀取器。  
  - 尚不支援快照集。  
  - 尚不支援散發者。  
  - 不支援合併。  
  - 不支援佇列讀取器。  
 - 尚不支援命令殼層。 
  - 受控執行個體無法存取外部資源 (例如，透過 robocopy 的網路共用)。  
 - 尚不支援 PowerShell。
 - 不支援 Analysis Services。  
- 部分支援通知。
 - 支援電子郵件通知，但必須設定「資料庫郵件」設定檔。 可能只有一個資料庫郵件設定檔，而且在公開預覽中一定名為 `AzureManagedInstance_dbmail_profile` (暫時性限制)。  
 - 不支援呼叫器。  
 - 不支援 NetSend。 
 - 尚不支援警示。
 - 不支援 Proxy。  
- 不支援 Eventlog。 
 
目前不支援下列功能，但未來將會啟用：  
- Proxy
- 在閒置的 CPU 上排程工作 
- 啟用/停用代理程式
- 警示

如需有關 SQL Server Agent 的資訊，請參閱 [SQL Server Agent](https://docs.microsoft.com/sql/ssms/agent/sql-server-agent)。
 
### <a name="tables"></a>資料表 

不支援下列項目： 
- `FILESTREAM` 
- `FILETABLE` 
- `EXTERNAL TABLE`
- `MEMORY_OPTIMIZED`  

如需有關建立和更改資料表的資訊，請參閱 [CREATE TABLE](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql) 和 [ALTER TABLE](https://docs.microsoft.com/sql/t-sql/statements/alter-table-transact-sql)。

## <a name="Changes"></a> 行為變更 
 
下列變數、函式和檢視會傳回不同的結果：  
- `SERVERPROPERTY('EngineEdition')` 傳回值「8」。 此屬性只會識別出受控執行個體。 請參閱 [SERVERPROPERTY](https://docs.microsoft.com/sql/t-sql/functions/serverproperty-transact-sql)。
- `SERVERPROPERTY('InstanceName')` 會傳回 NULL，因為執行個體基於 SQL Server 而存在的概念不適用於受控執行個體。 請參閱 [SERVERPROPERTY('InstanceName')](https://docs.microsoft.com/sql/t-sql/functions/serverproperty-transact-sql)。
- `@@SERVERNAME` 會傳回完整的 DNS「可連線」名稱，例如 my-managed-instance.wcus17662feb9ce98.database.windows.net。 請參閱 [@@SERVERNAME](https://docs.microsoft.com/sql/t-sql/functions/servername-transact-sql)。  
- `SYS.SERVERS` - 傳回完整的 DNS「可連線」名稱，例如 'name' 和 'data_source' 屬性的 `myinstance.domain.database.windows.net`。 請參閱 [SYS.SERVERS](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-servers-transact-sql)。 
- `@@SERVICENAME` 會傳回 NULL，因為服務基於 SQL Server 而存在的概念不適用於受控執行個體。 請參閱 [@@SERVICENAME](https://docs.microsoft.com/sql/t-sql/functions/servicename-transact-sql)。   
- 支援 `SUSER_ID`。 如果 AAD 登入不是 sys.syslogins，則傳回 NULL。 請參閱 [SUSER_ID](https://docs.microsoft.com/sql/t-sql/functions/suser-id-transact-sql)。  
- 不支援 `SUSER_SID`。 傳回錯誤資料 (暫時的已知問題)。 請參閱 [SUSER_SID](https://docs.microsoft.com/sql/t-sql/functions/suser-sid-transact-sql)。 
- `GETDATE()` 和其他內建的日期/時間函式一律會傳回時間 (UTC 時區)。 請參閱 [GETDATE](https://docs.microsoft.com/sql/t-sql/functions/getdate-transact-sql)。

## <a name="Issues"></a>已知問題與限制

### <a name="tempdb-size"></a>TEMPDB 大小

`tempdb` 分割成 12 檔案，每個檔案最大為 14 GB。 無法變更每個檔案大小的上限，而且無法將新檔案新增至 `tempdb`。 這項限制很快將會移除。 如果某些查詢需要 168 GB 以上的 `tempdb`，則可能會傳回錯誤。

### <a name="exceeding-storage-space-with-small-database-files"></a>小型資料庫檔案造成儲存空間超出限制

每個「受控執行個體」最多會為「Azure 進階磁碟」空間保留 35 TB 的儲存體，且每個資料庫檔案都會放在個別的實體磁碟上。 磁碟大小可以是 128 GB、256 GB、512 GB、1 TB 或 4 TB。 針對磁碟上未使用的空間，並不收費，但「Azure 進階磁碟」大小的總和不可超過 35 TB。 在某些情況下，總計不需 8 TB 的「受控執行個體」可能會因內部分散的緣故而超過 35 TB 的 Azure 儲存體大小限制。 

例如，「受控執行個體」可能有一個大小為 1.2 TB 而放置在 4 TB 磁碟上的檔案，以及 248 個各為 1 GB 大小而放置在其他 128 GB 磁碟上的檔案。 在此範例中， 
* 磁碟儲存體大小總計為 1 x 4 TB + 248 x 128 GB = 35 TB。 
* 為執行個體上的資料庫保留的大小總計為 1 x 1.2 TB + 248 x 1 GB = 1.4 TB。
這說明了在特性情況下，由於非常特定的檔案散發方式，「受控執行個體」可能在您未預期的情形下，達到為所連結「Azure 進階磁碟」保留的 35 TB。 

在此範例中，現有資料庫會繼續運作，只要不新增檔案，就可正常成長而不會有任何問題。 不過，因為沒有足夠空間可供新的磁碟機使用，所以無法建立或還原新的資料庫，即使所有資料庫的大小總計未達到執行個體大小限制也是如此。 在該情況下所傳回的錯誤將不清楚。

### <a name="incorrect-configuration-of-sas-key-during-database-restore"></a>還原資料庫期間 SAS 金鑰設定不正確

如果 `CREDENTIAL` 中的共用存取簽章不正確，讀取 .bak 檔案的 `RESTORE DATABASE` 可能會不斷重試讀取 .bak 檔案，並在很長一段時間之後傳回錯誤。 請在還原資料庫前執行 RESTORE HEADERONLY，以確定 SAS 金鑰正確無誤。
請確定您已從 Azure 入口網站所產生的 SAS 金鑰中移除 `?` 前置符號。

### <a name="tooling"></a>工具

SQL Server Management Studio 和 SQL Server Data Tools 在存取受控執行個體時可能會有一些問題。 所有工具問題將會在正式運作之前解決。

### <a name="incorrect-database-names"></a>不正確的資料庫名稱

在還原期間或在某些錯誤訊息中，受控執行個體可能會顯示 GUID 值，而非資料庫名稱。 這些問題將會在正式運作之前修正。

### <a name="database-mail-profile"></a>資料庫郵件 XP
可能只有一個資料庫郵件設定檔，而且一定名為 `AzureManagedInstance_dbmail_profile`。 這是暫時性的限制，很快將會移除。

## <a name="next-steps"></a>後續步驟

- 如需受控執行個體的詳細資訊，請參閱[受控執行個體是什麼？](sql-database-managed-instance.md)
- 如需功能與比較清單，請參閱 [SQL 的一般功能](sql-database-features.md)。
- 如需示範如何新建受控執行個體的教學課程，請參閱[建立受控執行個體](sql-database-managed-instance-create-tutorial-portal.md)。
