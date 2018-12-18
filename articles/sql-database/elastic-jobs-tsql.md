---
title: 使用 Transact-SQL (T-SQL) 建立及管理 Azure SQL 彈性資料庫作業 | Microsoft Docs
description: 使用 Transact-SQL (T-SQL) 透過彈性資料庫作業代理程式跨多個資料庫執行指令碼。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
ms.author: jaredmoo
author: jaredmoo
ms.reviewer: ''
manager: craigg
ms.date: 06/14/2018
ms.openlocfilehash: 49fe1fc79ac94b798cb257b961c36a6258fb00d9
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056782"
---
# <a name="use-transact-sql-t-sql-to-create-and-manage-elastic-database-jobs"></a>使用 Transact-SQL (T-SQL) 建立及管理彈性資料庫作業

本文提供許多範例案例，說明如何透過 T-SQL 開始使用彈性作業。

這些範例使用[*作業資料庫*](elastic-jobs-overview.md#job-database)中提供的[預存程序](#job-stored-procedures)和[檢視](#job-views)。

Transact-SQL (T-SQL) 可用來建立、設定、執行和管理作業。 目前不支援以 T-SQL 建立彈性作業代理程式，因此您必須先使用入口網站或 [PowerShell](elastic-jobs-powershell.md#create-the-elastic-job-agent) 建立*彈性作業代理程式*。


## <a name="create-a-credential-for-job-execution"></a>建立執行作業所需的認證

認證可用來連線至執行指令碼的目標資料庫。 認證必須具有適當權限 (對於目標群組所指定的資料庫)，才能成功執行指令碼。 使用伺服器和/或集區目標群組成員時，強烈建議您建立主要認證，以在作業執行期間，於擴充伺服器和/或集區之前用來重新整理認證。 資料庫範圍認證會建立在作業代理程式資料庫中。 相同的認證必須用來在目標資料庫上*建立登入*和*從登入建立要授與登入資料庫權限的使用者*。


```sql
--Connect to the job database specified when creating the job agent

-- Create a db master key if one does not already exist, using your own password.  
CREATE MASTER KEY ENCRYPTION BY PASSWORD='<EnterStrongPasswordHere>';  
  
-- Create a database scoped credential.  
CREATE DATABASE SCOPED CREDENTIAL myjobcred WITH IDENTITY = 'jobcred',
    SECRET = '<EnterStrongPasswordHere>'; 
GO

-- Create a database scoped credential for the master database of server1.
CREATE DATABASE SCOPED CREDENTIAL mymastercred WITH IDENTITY = 'mastercred',
    SECRET = '<EnterStrongPasswordHere>'; 
GO
```

## <a name="create-a-target-group-servers"></a>建立目標資料庫 (伺服器)

下列範例說明如何對伺服器中的所有資料庫執行作業。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：


```sql
-- Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group 'ServerGroup1'

-- Add a server target member
EXEC jobs.sp_add_target_group_member
'ServerGroup1',
@target_type = 'SqlServer',
@refresh_credential_name='mymastercred', --credential required to refresh the databases in server
@server_name='server1.database.windows.net'

--View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name='ServerGroup1';
SELECT * FROM jobs.target_group_members WHERE target_group_name='ServerGroup1';
```


## <a name="exclude-a-single-database"></a>執行單一資料庫

下列範例說明如何對伺服器中的所有資料庫 (名為 *MappingDB* 的資料庫除外) 執行作業。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC [jobs].sp_add_target_group N'ServerGroup'
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name=N'London.database.windows.net'
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name='server2.database.windows.net'
GO

--Excude a database target member from the server target group
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@membership_type = N'Exclude',
@target_type = N'SqlDatabase',
@server_name = N'server1.database.windows.net',
@database_name =N'MappingDB'
GO

--View the recently created target group and target group members
SELECT * FROM [jobs].target_groups WHERE target_group_name = N'ServerGroup';
SELECT * FROM [jobs].target_group_members WHERE target_group_name = N'ServerGroup';
```


## <a name="create-a-target-group-pools"></a>建立目標資料庫 (集區)

下列範例說明如何以一或多個彈性集區中的所有資料庫作為目標。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing pool(s)
EXEC jobs.sp_add_target_group 'PoolGroup'

-- Add an elastic pool(s) target member
EXEC jobs.sp_add_target_group_member
'PoolGroup',
@target_type = 'SqlElasticPool',
@refresh_credential_name='mymastercred', --credential required to refresh the databases in server
@server_name='server1.database.windows.net',
@elastic_pool_name='ElasticPool-1'

-- View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name = N'PoolGroup';
SELECT * FROM jobs.target_group_members WHERE target_group_name = N'PoolGroup';
```


## <a name="deploy-new-schema-to-many-databases"></a>將新的結構描述部署至多個資料庫

下列範例說明如何將新的結構描述部署至所有資料庫。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：


```sql
--Connect to the job database specified when creating the job agent

--Add job for create table
EXEC jobs.sp_add_job @job_name='CreateTableTest', @description='Create Table Test'

-- Add job step for create table
EXEC jobs.sp_add_jobstep @job_name='CreateTableTest',
@command=N'IF NOT EXISTS (SELECT * FROM sys.tables 
            WHERE object_id = object_id(''Test''))
CREATE TABLE [dbo].[Test]([TestId] [int] NOT NULL);',
@credential_name='myjobcred',
@target_group_name='PoolGroup'
```


## <a name="data-collection-using-built-in-parameters"></a>使用內建參數的資料收集

在許多資料收集案例中，納入以下部分指令碼變數都可能有助於對作業的結果進行後處理。

- $(job_name)
- $(job_id)
- $(job_version)
- $(step_id)
- $(step_name)
- $(job_execution_id)
- $(job_execution_create_time)
- $(target_group_name)

例如，若要將執行相同作業所產生的所有結果分為同一群組，請使用 *$(job_execution_id)*，如下列命令所示：


```sql
@command= N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());'
```


## <a name="monitor-database-performance"></a>監視資料庫效能

下列範例會建立從多個資料庫收集效能資料的新作業。

根據預設，作業代理程式會建立資料表將傳回的結果儲存於其中。 因此，與用於輸出認證相關聯的登入必須要有足夠的權限能執行此作業。 如果您想要事先手動建立資料表，則必須具有下列屬性：
1. 結果集使用正確名稱和資料類型的資料行。
2. internal_execution_id 的其他資料行 (資料類型為 uniqueidentifier)。
3. internal_execution_id 資料行上名為 "IX_<TableName>_Internal_Execution_ID" 的非叢集索引。

連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

-- Add a job to collect perf results
EXEC jobs.sp_add_job @job_name ='ResultsJob', @description='Collection Performance data from all customers'

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name='ResultsJob',
@command= N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());',
@credential_name='myjobcred',
@target_group_name='PoolGroup',
@output_type='SqlDatabase',
@output_credential_name=’myjobcred’,
@output_server_name=’server1.database.windows.net',
@output_database_name=’<resultsdb>',
@output_table_name='<resutlstable>'
Create a job to monitor pool performance
--Connect to the job database specified when creating the job agent

-- Add a target group containing master database
EXEC jobs.sp_add_target_group 'MasterGroup'

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name='MasterGroup',
@target_type='SqlDatabase',
@server_name='server1.database.windows.net',
@database_name='master'

-- Add a job to collect perf results
EXEC jobs.sp_add_job
@job_name='ResultsPoolsJob',
@description='Demo: Collection Performance data from all pools',
@schedule_interval_type='Minutes',
@schedule_interval_count=15

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name='ResultsPoolsJob',
@command=N'declare @now datetime
DECLARE @startTime datetime
DECLARE @endTime datetime
DECLARE @poolLagMinutes datetime
DECLARE @poolStartTime datetime
DECLARE @poolEndTime datetime
SELECT @now = getutcdate ()
SELECT @startTime = dateadd(minute, -15, @now)
SELECT @endTime = @now
SELECT @poolStartTime = dateadd(minute, -30, @startTime)
SELECT @poolEndTime = dateadd(minute, -30, @endTime)

SELECT elastic_pool_name , end_time, elastic_pool_dtu_limit, avg_cpu_percent, avg_data_io_percent, avg_log_write_percent, max_worker_percent, max_session_percent,
        avg_storage_percent, elastic_pool_storage_limit_mb FROM sys.elastic_pool_resource_stats
        WHERE end_time > @poolStartTime and end_time <= @poolEndTime;
'),
@credential_name='myjobcred',
@target_group_name='MasterGroup',
@output_type='SqlDatabase',
@output_credential_name='myjobcred',
@output_server_name=’server1.database.windows.net',
@output_database_name=’resultsdb',
@output_table_name='resutlstable'
```


## <a name="view-job-definitions"></a>檢視作業定義

下列範例說明如何檢視目前的作業定義。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

-- View all jobs
SELECT * FROM jobs.jobs

-- View the steps of the current version of all jobs
SELECT js.* FROM jobs.jobsteps js
JOIN jobs.jobs j 
  ON j.job_id = js.job_id AND j.job_version = js.job_version

-- View the steps of all versions of all jobs
select * from jobs.jobsteps
```


## <a name="begin-ad-hoc-execution-of-a-job"></a>開始執行臨機操作作業

下列範例說明如何立即啟動作業。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

-- Execute the latest version of a job
EXEC jobs.sp_start_job 'CreateTableTest'

-- Execute the latest version of a job and receive the execution id
declare @je uniqueidentifier
exec jobs.sp_start_job 'CreateTableTest', @job_execution_id = @je output
select @je

select * from jobs.job_executions where job_execution_id = @je

-- Execute a specific version of a job (e.g. version 1)
exec jobs.sp_start_job 'CreateTableTest', 1
```


## <a name="schedule-execution-of-a-job"></a>排程作業執行時間

下列範例說明如何排程要在未來執行的作業。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_update_job
@job_name='ResultsJob',
@enabled=1,
@schedule_interval_type='Minutes',
@schedule_interval_count=15
```

## <a name="monitor-job-execution-status"></a>監視作業執行狀態

下列範例說明如何檢視所有作業的執行狀態詳細資料。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

--View top-level execution status for the job named ‘ResultsPoolJob’
SELECT * FROM jobs.job_executions 
WHERE job_name = 'ResultsPoolsJob' and step_id IS NULL
ORDER BY start_time DESC

--View all top-level execution status for all jobs
SELECT * FROM jobs.job_executions WHERE step_id IS NULL
ORDER BY start_time DESC

--View all execution statuses for job named ‘ResultsPoolsJob’
SELECT * FROM jobs.job_executions 
WHERE job_name = 'ResultsPoolsJob' 
ORDER BY start_time DESC

-- View all active executions
SELECT * FROM jobs.job_executions 
WHERE is_active = 1
ORDER BY start_time DESC
```


## <a name="cancel-a-job"></a>取消工作

下列範例說明如何取消作業。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

-- View all active executions to determine job execution id
SELECT * FROM jobs.job_executions 
WHERE is_active = 1 AND job_name = 'ResultPoolsJob'
ORDER BY start_time DESC
GO

-- Cancel job execution with the specified job execution id
EXEC jobs.sp_stop_job '01234567-89ab-cdef-0123-456789abcdef'
```


## <a name="delete-old-job-history"></a>刪除舊作業歷程記錄

下列範例說明如何刪除特定日期之前的作業歷程記錄。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

-- Delete history of a specific job’s executions older than the specified date
EXEC jobs.sp_purge_jobhistory @job_name='ResultPoolsJob', @oldest_date='2016-07-01 00:00:00'

--Note: job history is automatically deleted if it is >45 days old
```

## <a name="delete-a-job-and-all-its-job-history"></a>刪除作業及其所有的作業歷程記錄

下列範例說明如何刪除作業和所有相關的作業歷程記錄。  
連線至[*作業資料庫*](elastic-jobs-overview.md#job-database)，然後執行下列命令：

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_delete_job @job_name='ResultsPoolsJob'

--Note: job history is automatically deleted if it is >45 days old
```




## <a name="job-stored-procedures"></a>作業預存程序

下列預存程序位於[作業資料庫](elastic-jobs-overview.md#job-database)中。



|預存程序  |說明  |
|---------|---------|
|[sp_add_job](#spaddjob)     |     新增作業。    |
|[sp_update_job ](#spupdatejob)    |      更新現有作業。   |
|[sp_delete_job](#spdeletejob)     |      刪除現有作業。   |
|[sp_add_jobstep](#spaddjobstep)    |    將步驟新增至作業。     |
|[sp_update_jobstep](#spupdatejobstep)     |     更新作業步驟。    |
|[sp_delete_jobstep](#spdeletejobstep)     |     刪除作業步驟。    |
|[sp_start_job](#spstartjob)    |  開始執行作業。       |
|[sp_stop_job](#spstopjob)     |     停止作業執行。   |
|[sp_add_target_group](#spaddtargetgroup)    |     新增目標群組。    |
|[sp_delete_target_group](#spdeletetargetgroup)     |    刪除目標群組。     |
|[sp_add_target_group_member](#spaddtargetgroupmember)     |    將資料庫或資料庫群組新增至目標群組。     |
|[sp_delete_target_group_member](#spdeletetargetgroupmember)     |     從目標群組中移除目標群組成員。    |
|[sp_purge_jobhistory ](#sppurgejobhistory)    |    移除作業的歷程記錄。     |





### <a name="spaddjob"></a>sp_add_job

新增作業。 
  
#### <a name="syntax"></a>語法  
  

```sql
[jobs].sp_add_job [ @job_name = ] 'job_name'  
    [ , [ @description = ] 'description' ]   
    [ , [ @enabled = ] enabled ]
    [ , [ @schedule_interval_type = ] schedule_interval_type ]  
    [ , [ @schedule_interval_count = ] schedule_interval_count ]   
    [ , [ @schedule_start_time = ] schedule_start_time ]   
    [ , [ @schedule_end_time = ] schedule_end_time ]   
    [ , [ @job_id = ] job_id OUTPUT ]
```

  
#### <a name="arguments"></a>引數  

[ **@job_name =** ] 'job_name'  
作業的名稱。 名稱必須是唯一的，且不可包含百分比 (%) 字元。 job_name 是 nvarchar(128)，沒有預設值。

[ **@description =** ] 'description'  
作業的描述。 description 是 nvarchar (512)，預設值為 NULL。 如果省略 description，則會使用空字串。

[ **@enabled =** ] enabled  
指定是否啟用作業的排程。 Enabled 是位元，預設值是 0 (停用)。 如果為 0，則不會啟用作業，且不會依據排程執行作業；但您可以手動執行作業。 如果為 1，則會依據排程執行作業，且您可以手動執行作業。

[ **@schedule_interval_type =**] schedule_interval_type  
此值表示要執行作業的時機。 schedule_interval_type 是 nvarchar(50)，預設值為 [單次]，而其值可以是下列其中之一：
- 「單次」、
- 「分鐘」、
- 「小時」、
- 「天」、
- 「週」、
- 「月」、

[ **@schedule_interval_count =** ] schedule_interval_count  
每次執行作業之間所發生的 schedule_interval_count 期間數目。 schedule_interval_count 是 int，預設值為 1。 此值必須大於或等於 1。

[ **@schedule_start_time =** ] schedule_start_time  
可開始執行作業的日期。 schedule_start_time 是 DATETIME2，預設值為 0001-01-01 00:00:00.0000000。

[ **@schedule_end_time =** ] schedule_end_time  
可停止執行作業的日期。 schedule_end_time 是 DATETIME2，預設值為 9999-12-31 11:59:59.0000000。 

[ **@job_id =** ] job_id OUTPUT  
在作業成功建立後指派給作業的識別碼。 job_id 是 uniqueidentifier 類型的輸出變數。

#### <a name="return-code-values"></a>傳回碼值

0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
sp_add_job 必須從建立作業代理程式時所指定的作業代理程式資料庫執行。
在執行 sp_add_job 以新增作業後，可以使用 sp_add_jobstep 新增相關步驟，以執行作業的活動。 作業的初始版本號碼為 0，而其後會在新增第一個步驟時累加為 1。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：

- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。

### <a name="spupdatejob"></a>sp_update_job

更新現有作業。

#### <a name="syntax"></a>語法

```sql
[jobs].sp_update_job [ @job_name = ] 'job_name'  
    [ , [ @new_name = ] 'new_name' ]
    [ , [ @description = ] 'description' ]   
    [ , [ @enabled = ] enabled ]
    [ , [ @schedule_interval_type = ] schedule_interval_type ]  
    [ , [ @schedule_interval_count = ] schedule_interval_count ]   
    [ , [ @schedule_start_time = ] schedule_start_time ]   
    [ , [ @schedule_end_time = ] schedule_end_time ]   
```

#### <a name="arguments"></a>引數
[ **@job_name =** ] 'job_name'  
要更新的作業名稱。 job_name 是 nvarchar(128)。

[ **@new_name =** ] 'new_name'  
作業的新名稱。 new_name 是 nvarchar(128)。

[ **@description =** ] 'description'  
作業的描述。 description 是 nvarchar(512)。

[ **@enabled =** ] enabled  
指定要啟用 (1) 還是不啟用 (0) 作業的排程。 Enabled 是位元。

[ **@schedule_interval_type=** ] schedule_interval_type  
此值表示要執行作業的時機。 schedule_interval_type 是 nvarchar(50)，而其值可以是下列其中之一：

- 「單次」、
- 「分鐘」、
- 「小時」、
- 「天」、
- 「週」、
- 「月」

[ **@schedule_interval_count=** ] schedule_interval_count  
每次執行作業之間所發生的 schedule_interval_count 期間數目。 schedule_interval_count 是 int，預設值為 1。 此值必須大於或等於 1。

[ **@schedule_start_time=** ] schedule_start_time  
可開始執行作業的日期。 schedule_start_time 是 DATETIME2，預設值為 0001-01-01 00:00:00.0000000。

[ **@schedule_end_time=** ] schedule_end_time  
可停止執行作業的日期。 schedule_end_time 是 DATETIME2，預設值為 9999-12-31 11:59:59.0000000。 

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
在執行 sp_add_job 以新增作業後，可以使用 sp_add_jobstep 新增相關步驟，以執行作業的活動。 作業的初始版本號碼為 0，而其後會在新增第一個步驟時累加為 1。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。



### <a name="spdeletejob"></a>sp_delete_job

刪除現有作業。

#### <a name="syntax"></a>語法

```sql
[jobs].sp_delete_job [ @job_name = ] 'job_name'
    [ , [ @force = ] force ]
```

#### <a name="arguments"></a>引數
[ **@job_name =** ] 'job_name'  
要刪除的作業名稱。 job_name 是 nvarchar(128)。

[ **@force =** ] force  
指定是要在有任何作業正在執行時刪除並取消所有執行中的作業 (1)，還是要使任何執行中的作業成為失敗的作業 (0)。 force 是位元。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
作業歷程記錄會在作業刪除時自動刪除。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。



### <a name="spaddjobstep"></a>sp_add_jobstep

將步驟新增至作業。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_add_jobstep [ @job_name = ] 'job_name'   
     [ , [ @step_id = ] step_id ]   
     [ , [ @step_name = ] step_name ]   
     [ , [ @command_type = ] 'command_type' ]   
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]   
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]   
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]   
     [ , [ @retry_attempts = ] retry_attempts ]   
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]   
     [ , [ @output_type = ] 'output_type' ]   
     [ , [ @output_credential_name = ] 'output_credential_name' ]   
     [ , [ @output_subscription_id = ] 'output_subscription_id' ]   
     [ , [ @output_resource_group_name = ] 'output_resource_group_name' ]   
     [ , [ @output_server_name = ] 'output_server_name' ]   
     [ , [ @output_database_name = ] 'output_database_name' ]   
     [ , [ @output_schema_name = ] 'output_schema_name' ]   
     [ , [ @output_table_name = ] 'output_table_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>引數

[ **@job_name =** ] 'job_name'  
要新增步驟的作業名稱。 job_name 是 nvarchar(128)。

[ **@step_id =** ] step_id  
作業步驟的序列識別碼。 步驟識別碼從 1 開始，並漸次遞增而不會跳號。 如果現有的步驟已具有此識別碼，則該步驟和所有後續步驟將會有累加的識別碼，而使這個新步驟可插入序列中。 若未指定，則會將 step_id 自動指派給步驟序列中的最後一個項目。 step_id 是 int。

[ **@step_name =** ] step_name  
步驟的名稱。 必須指定，但具有預設名稱 'JobStep' 的作業中採用的第一個步驟除外 (以利操作)。 step_name 是 nvarchar(128)。

[ **@command_type =** ] 'command_type'  
此作業步驟所執行的命令類型。 command_type 是 nvarchar(50)，預設值為 TSql，表示 @command_type 參數的值為 T-SQL 指令碼。

如果指定，則值必須是 TSql。

[ **@command_source =** ] 'command_source'  
用來存放命令的位置類型。 command_source 是 nvarchar(50)，預設值為內嵌，表示 @command_source 參數的值為命令的常值文字。

如果指定，則值必須是內嵌。

[ **@command =** ] 'command'  
command 必須是有效的 T-SQL 指令碼，且後續會由此作業步驟執行。 command 是 nvarchar(max)，預設值為 NULL。

[ **@credential_name =** ] ‘credential_name’  
存放在此作業控制資料庫中的資料庫範圍認證名稱，在此步驟執行時用來連線至目標群組內的每個目標資料庫。 credential_name 是 nvarchar(128)。

[ **@target_group_name =** ] ‘target-group_name'  
將執行作業步驟的目標資料庫所屬的目標群組名稱。 target_group_name 是 nvarchar(128)。

[ **@initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
作業步驟的初始執行嘗試失敗時，在第一次重試之前的延遲時間。 initial_retry_interval_seconds 是 int，預設值為 1。

[ **@maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
重試之間的延遲上限。 如果重試之間的延遲時間超出此值，該延遲將會限定為此值。 maximum_retry_interval_seconds 是 int，預設值為 120。

[ **@retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
在多個作業步驟執行嘗試失敗時要套用至重試延遲的乘數。 例如，如果第一次重試的延遲為 5 秒，降速乘數為 2.0，則第二次重試的延遲將是 10 秒，第三次重試的延遲將是 20 秒。 retry_interval_backoff_multiplier 是實數，預設值為 2.0。

[ **@retry_attempts =** ] retry_attempts  
初始嘗試失敗時的重試執行次數。 例如，如果 retry_attempts 值為 10，則將會有 1 次初始嘗試和 10 次重試，共計 11 次嘗試。 在最後一次重試失敗後，作業執行將會終止，且生命週期會失敗。 retry_attempts 是 int，預設值為 10。

[ **@step_timeout_seconds =** ] step_timeout_seconds  
允許執行步驟的時間長度上限。 如果超出此時間，作業執行將會終止，且生命週期會逾時。 step_timeout_seconds 是 int，預設值為 43,200 秒 (12 小時)。

[ **@output_type =** ] 'output_type'  
如果不是 Null，則為命令的第一個結果集寫入的目的地類型。 output_type 是 nvarchar(50)，預設值為 NULL。

如果指定，則值必須是 SqlDatabase。

[ **@output_credential_name =** ] 'output_credential_name'  
如果不是 Null，則為用來連線至輸出目的地資料庫的資料庫範圍認證的名稱。 如果 output_type 是 SqlDatabase，則必須指定。 output_credential_name 是 nvarchar(128)，預設值為 NULL。

[ **@output_subscription_id =** ] 'output_subscription_id'  
需要描述。

[ **@output_resource_group_name =** ] 'output_resource_group_name'  
需要描述。

[ **@output_server_name =** ] 'output_server_name'  
如果不是 Null，則為包含輸出目的地資料庫的伺服器所具備的完整 DNS 名稱。 如果 output_type 是 SqlDatabase，則必須指定。 output_server_name 是 nvarchar(256)，預設值為 NULL。

[ **@output_database_name =** ] 'output_database_name'  
如果不是 Null，則為包含輸出目的地資料表的資料庫名稱。 如果 output_type 是 SqlDatabase，則必須指定。 output_database_name 是 nvarchar(128)，預設值為 NULL。

[ **@output_schema_name =** ] 'output_schema_name'  
如果不是 Null，則為包含輸出目的地資料表的 SQL 結構描述名稱。 如果 output_type 是 SqlDatabase，則預設值為 dbo。 output_schema_name 是 nvarchar(128)。

[ **@output_table_name =** ] 'output_table_name'  
如果不是 Null，則為命令的第一個結果集將會寫入的目標資料表名稱。 如果資料表尚不存在，則會根據傳回的結果集所具備的結構描述建立資料表。 如果 output_type 是 SqlDatabase，則必須指定。 output_table_name 是 nvarchar(128)，預設值為 NULL。

[ **@job_version =** ] job_version OUTPUT  
將會被指派新的作業版本號碼的輸出參數。 job_version 是 int。

[ **@max_parallelism =** ] max_parallelism OUTPUT  
每個彈性集區的平行處理原則最大層級。 如果設定，則作業步驟將會限定為最多僅為每個彈性集區執行該數量的資料庫。 此項目會套用至直接包含在目標群組中的每個彈性集區，或目標群組所含伺服器內的每個彈性集區。 max_parallelism 是 int。


#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
當 sp_add_jobstep 成功時，作業目前的版本號碼將會累加。 在下次執行作業時，將會使用新版本。 如果作業目前正在執行，該執行將不會包含新的步驟。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：  

- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。



### <a name="spupdatejobstep"></a>sp_update_jobstep

更新作業步驟。

#### <a name="syntax"></a>語法

```sql
[jobs].sp_update_jobstep [ @job_name = ] 'job_name'   
     [ , [ @step_id = ] step_id ]   
     [ , [ @step_name = ] 'step_name' ]   
     [ , [ @new_id = ] new_id ]   
     [ , [ @new_name = ] 'new_name' ]   
     [ , [ @command_type = ] 'command_type' ]   
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]   
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]   
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]   
     [ , [ @retry_attempts = ] retry_attempts ]   
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]   
     [ , [ @output_type = ] 'output_type' ]   
     [ , [ @output_credential_name = ] 'output_credential_name' ]   
     [ , [ @output_server_name = ] 'output_server_name' ]   
     [ , [ @output_database_name = ] 'output_database_name' ]   
     [ , [ @output_schema_name = ] 'output_schema_name' ]   
     [ , [ @output_table_name = ] 'output_table_name' ]   
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>引數
[ **@job_name =** ] 'job_name'  
步驟所屬作業的名稱。 job_name 是 nvarchar(128)。

[ **@step_id =** ] step_id  
要修改的作業步驟識別碼。 必須指定 step_id 或 step_name。 step_id 是 int。

[ **@step_name =** ] 'step_name'  
要修改的步驟名稱。 必須指定 step_id 或 step_name。 step_name 是 nvarchar(128)。

[ **@new_id =** ] new_id  
作業步驟的新序列識別碼。 步驟識別碼從 1 開始，並漸次遞增而不會跳號。 如果某個步驟重新排序，其他步驟將會自動重新編碼。

[ **@new_name =** ] 'new_name'  
步驟的新名稱。 new_name 是 nvarchar(128)。

[ **@command_type =** ] 'command_type'  
此作業步驟所執行的命令類型。 command_type 是 nvarchar(50)，預設值為 TSql，表示 @command_type 參數的值為 T-SQL 指令碼。

如果指定，則值必須是 TSql。

[ **@command_source =** ] 'command_source'  
用來存放命令的位置類型。 command_source 是 nvarchar(50)，預設值為內嵌，表示 @command_source 參數的值為命令的常值文字。

如果指定，則值必須是內嵌。

[ **@command =** ] 'command'  
command 必須是有效的 T-SQL 指令碼，且後續會由此作業步驟執行。 command 是 nvarchar(max)，預設值為 NULL。

[ **@credential_name =** ] ‘credential_name’  
存放在此作業控制資料庫中的資料庫範圍認證名稱，在此步驟執行時用來連線至目標群組內的每個目標資料庫。 credential_name 是 nvarchar(128)。

[ **@target_group_name =** ] ‘target-group_name'  
將執行作業步驟的目標資料庫所屬的目標群組名稱。 target_group_name 是 nvarchar(128)。

[ **@initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
作業步驟的初始執行嘗試失敗時，在第一次重試之前的延遲時間。 initial_retry_interval_seconds 是 int，預設值為 1。

[ **@maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
重試之間的延遲上限。 如果重試之間的延遲時間超出此值，該延遲將會限定為此值。 maximum_retry_interval_seconds 是 int，預設值為 120。

[ **@retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
在多個作業步驟執行嘗試失敗時要套用至重試延遲的乘數。 例如，如果第一次重試的延遲為 5 秒，降速乘數為 2.0，則第二次重試的延遲將是 10 秒，第三次重試的延遲將是 20 秒。 retry_interval_backoff_multiplier 是實數，預設值為 2.0。

[ **@retry_attempts =** ] retry_attempts  
初始嘗試失敗時的重試執行次數。 例如，如果 retry_attempts 值為 10，則將會有 1 次初始嘗試和 10 次重試，共計 11 次嘗試。 在最後一次重試失敗後，作業執行將會終止，且生命週期會失敗。 retry_attempts 是 int，預設值為 10。

[ **@step_timeout_seconds =** ] step_timeout_seconds  
允許執行步驟的時間長度上限。 如果超出此時間，作業執行將會終止，且生命週期會逾時。 step_timeout_seconds 是 int，預設值為 43,200 秒 (12 小時)。

[ **@output_type =** ] 'output_type'  
如果不是 Null，則為命令的第一個結果集寫入的目的地類型。 若要將 output_type 的值重設為 NULL，請將此參數的值設為 '' (空字串)。 output_type 是 nvarchar(50)，預設值為 NULL。

如果指定，則值必須是 SqlDatabase。

[ **@output_credential_name =** ] 'output_credential_name'  
如果不是 Null，則為用來連線至輸出目的地資料庫的資料庫範圍認證的名稱。 如果 output_type 是 SqlDatabase，則必須指定。 若要將 output_credential_name 的值重設為 NULL，請將此參數的值設為 '' (空字串)。 output_credential_name 是 nvarchar(128)，預設值為 NULL。

[ **@output_server_name =** ] 'output_server_name'  
如果不是 Null，則為包含輸出目的地資料庫的伺服器所具備的完整 DNS 名稱。 如果 output_type 是 SqlDatabase，則必須指定。 若要將 output_server_name 的值重設為 NULL，請將此參數的值設為 '' (空字串)。 output_server_name 是 nvarchar(256)，預設值為 NULL。

[ **@output_database_name =** ] 'output_database_name'  
如果不是 Null，則為包含輸出目的地資料表的資料庫名稱。 如果 output_type 是 SqlDatabase，則必須指定。 若要將 output_database_name 的值重設為 NULL，請將此參數的值設為 '' (空字串)。 output_database_name 是 nvarchar(128)，預設值為 NULL。

[ **@output_schema_name =** ] 'output_schema_name'  
如果不是 Null，則為包含輸出目的地資料表的 SQL 結構描述名稱。 如果 output_type 是 SqlDatabase，則預設值為 dbo。 若要將 output_schema_name 的值重設為 NULL，請將此參數的值設為 '' (空字串)。 output_schema_name 是 nvarchar(128)。

[ **@output_table_name =** ] 'output_table_name'  
如果不是 Null，則為命令的第一個結果集將會寫入的目標資料表名稱。 如果資料表尚不存在，則會根據傳回的結果集所具備的結構描述建立資料表。 如果 output_type 是 SqlDatabase，則必須指定。 若要將 output_server_name 的值重設為 NULL，請將此參數的值設為 '' (空字串)。 output_table_name 是 nvarchar(128)，預設值為 NULL。

[ **@job_version =** ] job_version OUTPUT  
將會被指派新的作業版本號碼的輸出參數。 job_version 是 int。

[ **@max_parallelism =** ] max_parallelism OUTPUT  
每個彈性集區的平行處理原則最大層級。 如果設定，則作業步驟將會限定為最多僅為每個彈性集區執行該數量的資料庫。 此項目會套用至直接包含在目標群組中的每個彈性集區，或目標群組所含伺服器內的每個彈性集區。 若要將 max_parallelism 的值重設為 Null，請將此參數的值設為 -1。 max_parallelism 是 int。


#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
任何執行中的作業都不會受到影響。 當 sp_update_jobstep 成功時，作業的版本號碼將會累加。 在下次執行作業時，將會使用新版本。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：

- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。




### <a name="spdeletejobstep"></a>sp_delete_jobstep

從作業移除作業步驟。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_delete_jobstep [ @job_name = ] 'job_name'   
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] 'step_name' ]   
     [ , [ @job_version = ] job_version OUTPUT ]
```

#### <a name="arguments"></a>引數
[ **@job_name =** ] 'job_name'  
要從中移除步驟的作業名稱。 job_name 是 nvarchar(128)，沒有預設值。

[ **@step_id =** ] step_id  
要刪除的作業步驟識別碼。 必須指定 step_id 或 step_name。 step_id 是 int。

[ **@step_name =** ] 'step_name'  
要刪除的步驟名稱。 必須指定 step_id 或 step_name。 step_name 是 nvarchar(128)。

[ **@job_version =** ] job_version OUTPUT  
將會被指派新的作業版本號碼的輸出參數。 job_version 是 int。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
任何執行中的作業都不會受到影響。 當 sp_update_jobstep 成功時，作業的版本號碼將會累加。 在下次執行作業時，將會使用新版本。

其他作業步驟將會自動重新編碼，以填補已刪除的作業步驟所留下的空號。
 
#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。






### <a name="spstartjob"></a>sp_start_job

開始執行作業。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_start_job [ @job_name = ] 'job_name'   
     [ , [ @job_execution_id = ] job_execution_id OUTPUT ]   
```

#### <a name="arguments"></a>引數
[ **@job_name =** ] 'job_name'  
要從中移除步驟的作業名稱。 job_name 是 nvarchar(128)，沒有預設值。

[ **@job_execution_id =** ] job_execution_id OUTPUT  
將會被指派作業執行識別碼的輸出參數。job_version 是 uniqueidentifier。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
無。
 
#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。

### <a name="spstopjob"></a>sp_stop_job

停止作業執行。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_stop_job [ @job_execution_id = ] ' job_execution_id '
```


#### <a name="arguments"></a>引數
[ **@job_execution_id =** ] job_execution_id  
要停止的作業執行識別碼。 job_execution_id 是 uniqueidentifier，識別碼為 NULL。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
無。
 
#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。


### <a name="spaddtargetgroup"></a>sp_add_target_group

新增目標群組。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_add_target_group [ @target_group_name = ] 'target_group_name'   
     [ , [ @target_group_id = ] target_group_id OUTPUT ]
```


#### <a name="arguments"></a>引數
[ **@target_group_name =** ] 'target_group_name'  
要建立的目標群組名稱。 target_group_name 是 nvarchar(128)，沒有預設值。

[ **@target_group_id =** ] target_group_id OUTPUT 在作業成功建立後指派給作業的目標群組識別碼。 target_group_id 是 uniqueidentifier 類型的輸出變數，預設值為 NULL。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
目標群組可讓您輕鬆地設定資料庫集合的作業。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。

### <a name="spdeletetargetgroup"></a>sp_delete_target_group

刪除目標群組。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_delete_target_group [ @target_group_name = ] 'target_group_name'
```


#### <a name="arguments"></a>引數
[ **@target_group_name =** ] 'target_group_name'  
要刪除的目標群組名稱。 target_group_name 是 nvarchar(128)，沒有預設值。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
無。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。

### <a name="spaddtargetgroupmember"></a>sp_add_target_group_member

將資料庫或資料庫群組新增至目標群組。

#### <a name="syntax"></a>語法

```sql
[jobs].sp_add_target_group_member [ @target_group_name = ] 'target_group_name'
         [ @membership_type = ] ‘membership_type’ ]   
        [ , [ @target_type = ] ‘target_type’ ]   
        [ , [ @refresh_credential_name = ] ‘refresh_credential_name’ ]   
        [ , [ @server_name = ] ‘server_name’ ]   
        [ , [ @database_name = ] ‘database_name’ ]   
        [ , [ @elastic_pool_name = ] ‘elastic_pool_name’ ]   
        [ , [ @shard_map_name = ] ‘shard_map_name’ ]   
        [ , [ @target_id = ] ‘target_id’ OUTPUT ]
```

#### <a name="arguments"></a>引數
[ **@target_group_name =** ] 'target_group_name'  
要新增成員的目標群組名稱。 target_group_name 是 nvarchar(128)，沒有預設值。

[ **@membership_type =** ] 'membership_type'  
指定是否要包含或排除目標群組成員。 target_group_name 是 nvarchar(128)，預設值為「包含」。 target_group_name 的有效值為「包含」或「排除」。

[ **@target_type =** ] 'target_type'  
包含伺服器中的所有資料庫、彈性集區中的所有資料庫、分區對應中的所有資料庫或個別資料庫的目標資料庫或資料庫集合的類型。 target_type 是 nvarchar(128)，沒有預設值。 Target_type 的有效值為 'SqlServer'、'SqlElasticPool'、'Sql Database' 或 'SqlShardMap'。 

[ **@refresh_credential_name =** ] 'refresh_credential_name'  
邏輯伺服器的名稱。 refresh_credential_name 是 nvarchar(128)，沒有預設值。

[ **@server_name =** ] 'server_name'  
應新增至指定目標群組的邏輯伺服器名稱。 當 target_type 為 ‘SqlServer’ 時，即應指定 server_name。 server_name 是 nvarchar(128)，沒有預設值。

[ **@database_name =** ] 'database_name'  
應新增至指定目標群組的資料庫名稱。 當 target_type 為 ‘SqlDatabase’ 時，即應指定 database_name。 database_name 是 nvarchar(128)，沒有預設值。

[ **@elastic_pool_name =** ] ‘elastic_pool_name'  
應新增至指定目標群組的彈性集區名稱。 當 target_type 為 ‘SqlElasticPool’ 時，即應指定 elastic_pool_name。 elastic_pool_name 是 nvarchar(128)，沒有預設值。

[ **@shard_map_name =** ] ‘shard_map_name'  
應新增至指定目標群組的分區對應集區名稱。 當 target_type 為 ‘SqlSqlShardMap’ 時，即應指定 elastic_pool_name。 shard_map_name 是 nvarchar(128)，沒有預設值。

[ **@target_id =** ] target_group_id OUTPUT  
在目標群組成員新增至目標群組時指派給該成員的目標識別碼。 target_id 是 uniqueidentifier 類型的輸出變數，預設值為 NULL。
傳回碼值 0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
當目標群組中包含邏輯伺服器或彈性集區時，在執行期間會對伺服器或彈性集區內的所有資料庫執行作業。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。

#### <a name="examples"></a>範例
下列範例會將 London 和 NewYork 伺服器中的所有資料庫新增至「維護客戶資訊的伺服器」群組。 您必須連線至在建立作業代理程式時所指定的作業資料庫，在此案例中為 ElasticJobs。

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs ; 
GO

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group @target_group_name =  N'Servers Maintaining Customer Information'
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name=N'London.database.windows.net' ;
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name=N'NewYork.database.windows.net' ;
GO

--View the recently added members to the target group
SELECT * FROM [jobs].target_group_members WHERE target_group_name= N'Servers Maintaining Customer Information';
GO
```

### <a name="spdeletetargetgroupmember"></a>sp_delete_target_group_member

從目標群組中移除目標群組成員。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_delete_target_group_member [ @target_group_name = ] 'target_group_name'
        [ , [ @target_id = ] ‘target_id’]
```



Arguments [ @target_group_name = ] 'target_group_name'  
要從中移除目標群組成員的目標群組名稱。 target_group_name 是 nvarchar(128)，沒有預設值。

[ @target_id = ] target_id  
 指派給要移除的目標群組成員的目標識別碼。 target_id 是 uniqueidentifier，預設值為 NULL。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗)

#### <a name="remarks"></a>備註
目標群組可讓您輕鬆地設定資料庫集合的作業。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。

#### <a name="examples"></a>範例
下列範例會從「維護客戶資訊的伺服器」群組中移除 London 伺服器。 您必須連線至在建立作業代理程式時所指定的作業資料庫，在此案例中為 ElasticJobs。

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs ; 
GO

-- Retrieve the target_id for a target_group_members
declare @tid uniqueidentifier
SELECT @tid = target_id FROM [jobs].target_group_members WHERE target_group_name = 'Servers Maintaining Customer Information' and server_name = 'London.database.windows.net'

-- Remove a target group member of type server
EXEC jobs.sp_delete_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_id = @tid
GO
```

### <a name="sppurgejobhistory"></a>sp_purge_jobhistory

移除作業的歷程記錄。

#### <a name="syntax"></a>語法


```sql
[jobs].sp_purge_jobhistory [ @job_name = ] 'job_name'   
      [ , [ @job_id = ] job_id ]
      [ , [ @oldest_date = ] oldest_date []
```

#### <a name="arguments"></a>引數
[ **@job_name =** ] 'job_name'  
要刪除歷程記錄的作業名稱。 job_name 是 nvarchar(128)，預設值為 NULL。 必須指定 job_id 或 job_name，但不可同時指定兩者。

[ **@job_id =** ] job_id  
 要刪除記錄的作業識別碼。 job_id 是 uniqueidentifier，預設值為 NULL。 必須指定 job_id 或 job_name，但不可同時指定兩者。

[ **@oldest_date =** ] oldest_date  
 歷程記錄中要保存的最舊記錄。 oldest_date 是 DATETIME2，預設值為 NULL。 指定了 oldest_date 時，sp_purge_jobhistory 只會移除比指定的值還舊的記錄。

#### <a name="return-code-values"></a>傳回碼值
0 (成功) 或 1 (失敗) 備註：目標群組可讓您輕鬆地設定資料庫集合的作業。

#### <a name="permissions"></a>權限
根據預設，系統管理員固定伺服器角色的成員可執行此預存程序。 他們會將使用者限定為只能監視作業，而您可以為使用者授與權限，使其在建立作業代理程式時所指定的作業代理程式資料庫中成為下列資料庫角色的一部分：
- jobs_reader

如需與這些角色的權限有關的詳細資訊，請參閱本文件中的「權限」一節。 只有系統管理員的成員可以使用此預存程序來編輯其他使用者所擁有的作業屬性。

#### <a name="examples"></a>範例
下列範例會將 London 和 NewYork 伺服器中的所有資料庫新增至「維護客戶資訊的伺服器」群組。 您必須連線至在建立作業代理程式時所指定的作業資料庫，在此案例中為 ElasticJobs。

```sql
--Connect to the jobs database specified when creating the job agent

EXEC sp_delete_target_group_member   
    @target_group_name = N'Servers Maintaining Customer Information',  
    @server_name = N'London.database.windows.net';  
GO
```


## <a name="job-views"></a>作業檢視

[作業資料庫](elastic-jobs-overview.md#job-database)中提供下列檢視。


|檢視  |說明  |
|---------|---------|
|[jobs_executions](#jobsexecutions-view)     |  顯示作業執行歷程記錄。      |
|[jobs](#jobs-view)     |   顯示所有作業。      |
|[job_versions](#jobversions-view)     |   顯示所有作業版本。      |
|[jobsteps](#jobsteps-view)     |     顯示每項作業的目前版本中包含的所有步驟。    |
|[jobstep_versions](#jobstepversions-view)     |     顯示每項作業的所有版本中包含的所有步驟。    |
|[target_groups](#targetgroups-view)     |      顯示所有目標群組。   |
|[target_group_members](#targetgroups-view)     |   顯示所有目標群組的所有成員。      |


### <a name="jobsexecutions-view"></a>jobs_executions 檢視

[jobs].[jobs_executions]

顯示作業執行歷程記錄。


|資料行名稱|   資料類型   |說明|
|---------|---------|---------|
|**job_execution_id**   |uniqueidentifier|  作業執行的執行個體唯一識別碼。
|**job_name**   |nvarchar(128)  |作業的名稱。
|**job_id** |uniqueidentifier|  作業的唯一識別碼。
|**job_version**    |int    |作業的版本 (會在作業每次修改時自動更新)。
|**step_id**    |int|   步驟的唯一識別碼 (用於此作業)。 NULL 表示這是父作業執行。
|**is_active**| bit |表示資訊是使用中還是非使用中。 1 表示使用中的作業，0 表示非使用中。
|**lifecycle**| nvarchar(50)|表示作業狀態的值：‘Created’、‘In Progress’、‘Failed’、‘Succeeded’、‘Skipped’、'SucceededWithSkipped’|
|**create_time**|   datetime2(7)|   建立作業的日期和時間。
|**start_time** |datetime2(7)|  作業開始執行的日期和時間。 如果作業尚未執行，則為 NULL。
|**end_time**|  datetime2(7)    |作業執行完成的日期和時間。 如果作業尚未執行或尚未完成執行，則為 NULL。
|**current_attempts**   |int    |重試步驟的次數。 父作業將是 0，子作業執行將是 1 或更大的值，視執行原則而定。
|**current_attempt_start_time** |datetime2(7)|  作業開始執行的日期和時間。 NULL 表示這是父作業執行。
|**last_message**   |nvarchar(max)| 作業或步驟歷程記錄訊息。 
|**target_type**|   nvarchar(128)   |包含伺服器中的所有資料庫、彈性集區中的所有資料庫或個別資料庫的目標資料庫或資料庫集合的類型。 Target_type 的有效值為 'SqlServer'、'SqlElasticPool' 或 'SqlDatabase'。 NULL 表示這是父作業執行。
|**target_id**  |uniqueidentifier|  目標群組成員的唯一識別碼。  NULL 表示這是父作業執行。
|**target_group_name**  |nvarchar(128)  |目標群組的名稱。 NULL 表示這是父作業執行。
|**target_server_name**|    nvarchar(256)|  目標群組中包含的邏輯伺服器名稱。 只有在 target_type 為 ‘SqlServer’ 時才須指定。 NULL 表示這是父作業執行。
|**target_database_name**   |nvarchar(128)| 目標群組中包含的資料庫名稱。 只有在 target_type 為 ‘SqlDatabase’ 時才須指定。 NULL 表示這是父作業執行。


### <a name="jobs-view"></a>作業檢視

[jobs].[jobs]

顯示所有作業。

|資料行名稱|   資料類型|  說明|
|------|------|-------|
|**job_name**|  nvarchar(128)   |作業的名稱。|
|**job_id**|    uniqueidentifier    |作業的唯一識別碼。|
|**job_version**    |int    |作業的版本 (會在作業每次修改時自動更新)。|
|**description**    |nvarchar(512)| 作業的描述。 enabled bit 會指出作業是啟用還是停用。 1 表示啟用的作業，0 表示停用的作業。|
|**schedule_interval_type** |nvarchar(50)   |指出何時要執行作業的值：「單次、「分鐘」、「小時」、「天」、「週」、「月」
|**schedule_interval_count**|   int|    每次執行作業之間所發生的 schedule_interval_type 期間數目。|
|**schedule_start_time**    |datetime2(7)|  作業上次開始執行的日期和時間。|
|**schedule_end_time**| datetime2(7)|   作業上次執行完成的日期和時間。|


### <a name="jobversions-view"></a>job_versions 檢視

[jobs].[job_verions]

顯示所有作業版本。

|資料行名稱|   資料類型|  說明|
|------|------|-------|
|**job_name**|  nvarchar(128)   |作業的名稱。|
|**job_id**|    uniqueidentifier    |作業的唯一識別碼。|
|**job_version**    |int    |作業的版本 (會在作業每次修改時自動更新)。|


### <a name="jobsteps-view"></a>作業步驟檢視

[jobs].[jobsteps]

顯示每項作業的目前版本中包含的所有步驟。

|資料行名稱    |資料類型| 說明|
|------|------|-------|
|**job_name**   |nvarchar(128)| 作業的名稱。|
|**job_id** |uniqueidentifier   |作業的唯一識別碼。|
|**job_version**|   int|    作業的版本 (會在作業每次修改時自動更新)。|
|**step_id**    |int    |步驟的唯一識別碼 (用於此作業)。|
|**step_name**  |nvarchar(128)  |步驟的唯一名稱 (用於此作業)。|
|**command_type**   |nvarchar(50)   |要在作業步驟中執行的命令類型。 針對 v1，值必須等於並預設為 ‘TSql’。|
|**command_source** |nvarchar(50)|  命令的位置。 針對 v1，「內嵌」是預設且唯一可接受的值。|
|**command**|   nvarchar(max)|  彈性作業要透過 command_type 執行的命令。|
|**credential_name**|   nvarchar(128)   |用來執行作業的資料庫範圍認證名稱。|
|**target_group_name**| nvarchar(128)   |目標群組的名稱。|
|**target_group_id**|   uniqueidentifier|   目標群組的唯一識別碼。|
|**initial_retry_interval_seconds**|    int |第一次重試之前的延遲時間。 預設值為 1。|
|**maximum_retry_interval_seconds** |int|   重試之間的延遲上限。 如果重試之間的延遲時間超出此值，該延遲將會限定為此值。 預設值為 120。|
|**retry_interval_backoff_multiplier**  |real|  在多個作業步驟執行嘗試失敗時要套用至重試延遲的乘數。 預設值為 2.0。|
|**retry_attempts** |int|   此步驟失敗時嘗試使用的重試次數。 預設值為 10，表示不會重試。|
|**step_timeout_seconds**   |int|   重試間隔的時間長度 (秒)。 預設值為 0，表示 0 分鐘的間隔。|
|**output_type**    |nvarchar(11)|  命令的位置。 在目前的預覽版中，「內嵌」是預設且唯一可接受的值。|
|**output_credential_name**|    nvarchar(128)   |要用來連線至目的地伺服器以存放結果集的認證名稱。|
|**output_subscription_id**|    uniqueidentifier|   查詢執行結果集的目的地伺服器\資料庫的訂用帳戶唯一識別碼。|
|**output_resource_group_name** |nvarchar(128)| 目的地伺服器所在的資源群組名稱。|
|**output_server_name**|    nvarchar(256)   |結果集的目的地伺服器名稱。|
|**output_database_name**   |nvarchar(128)| 結果集的目的地資料庫名稱。|
|**output_schema_name** |nvarchar(max)| 目的地結構描述的名稱。 若未指定，則預設為 dbo。|
|**output_table_name**| nvarchar(max)|  用來存放查詢結果的結果集資料表名稱。 如果資料表尚不存在，則會根據結果集的結構描述自動建立資料表。 結構描述必須符合結果集的結構描述。|
|**max_parallelism**|   int|    每個彈性集區將逐一執行作業步驟的資料庫數目上限。 預設值為 NULL，表示沒有限制。 |


### <a name="jobstepversions-view"></a>jobstep_versions 檢視

[jobs].[jobstep_versions]

顯示每項作業的所有版本中包含的所有步驟。 結構描述等同於[作業步驟](#jobsteps-view)。

### <a name="targetgroups-view"></a>target_groups 檢視

[jobs].[target_groups]

列出所有目標群組。

|資料行名稱|資料類型| 說明|
|-----|-----|-----|
|**target_group_name**| nvarchar(128)   |要刪除的目標群組 (資料庫集合) 的名稱。 
|**target_group_id**    |uniqueidentifier   |目標群組的唯一識別碼。

### <a name="targetgroupsmembers-view"></a>target_groups_members 檢視

[jobs].[target_groups_members]

顯示所有目標群組的所有成員。

|資料行名稱|資料類型| 說明|
|-----|-----|-----|
|**target_group_name**  |nvarchar(128|要刪除的目標群組 (資料庫集合) 的名稱。 |
|**target_group_id**    |uniqueidentifier   |目標群組的唯一識別碼。|
|**membership_type**    |int|   指定是否要在目標群組中包含或排除目標群組成員。 target_group_name 的有效值為「包含」或「排除」。|
|**target_type**    |nvarchar(128)| 包含伺服器中的所有資料庫、彈性集區中的所有資料庫或個別資料庫的目標資料庫或資料庫集合的類型。 Target_type 的有效值為 'SqlServer'、'SqlElasticPool'、'Sql Database' 或 'SqlShardMap'。|
|**target_id**  |uniqueidentifier|  目標群組成員的唯一識別碼。|
|**refresh_credential_name**    |nvarchar(128)  |用來連線至目標群組成員的資料庫範圍認證的名稱。|
|subscription_id    |uniqueidentifier|  訂用帳戶的唯一識別碼。|
|**resource_group_name**    |nvarchar(128)| 目標群組成員所在的資源群組名稱。|
|**server_name**    |nvarchar(128)  |目標群組中包含的邏輯伺服器名稱。 只有在 target_type 為 ‘SqlServer’ 時才須指定。 |
|**database_name**  |nvarchar(128)  |目標群組中包含的資料庫名稱。 只有在 target_type 為 ‘SqlDatabase’ 時才須指定。|
|**elastic_pool_name**  |nvarchar(128)| 目標群組中包含的彈性集區名稱。 只有在 target_type 為 ‘SqlElasticPool’ 時才須指定。|
|**shard_map_name** |nvarchar(128)| 目標群組中包含的分區對應名稱。 只有在 target_type 為 ‘SqlShardMap’ 時才須指定。|


## <a name="resources"></a>資源

 - ![主題連結圖示](https://docs.microsoft.com/sql/database-engine/configure-windows/media/topic-link.gif "主題連結圖示") [Transact-SQL 語法慣例](https://docs.microsoft.com/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql)  


## <a name="next-steps"></a>後續步驟

- [使用 PowerShell 建立及管理彈性作業](elastic-jobs-powershell.md)
- [SQL Server 中的授權和權限](https://docs.microsoft.com/dotnet/framework/data/adonet/sql/authorization-and-permissions-in-sql-server)
  
