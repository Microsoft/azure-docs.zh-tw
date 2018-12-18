---
title: 使用 Azure 彈性資料庫作業清除 SSISDB 記錄 | Microsoft Docs
description: 本文將說明如何使用 Azure 彈性資料庫作業來觸發用於清除 SSISDB 記錄的預存程序，進而清除 SSISDB 記錄
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/13/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: 5bd300a318008b34415f8bc9625cad3c38dec873
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "40106975"
---
# <a name="clean-up-ssisdb-logs-with-azure-elastic-database-jobs"></a>使用 Azure 彈性資料庫作業清除 SSISDB 記錄

本文將說明如何使用 Azure 彈性資料庫作業來觸發預存程序，以清除 SQL Server Integration Services 目錄資料庫 (`SSISDB`) 的記錄。

彈性資料庫作業是一項 Azure 服務，可讓您輕鬆地對一個資料庫或一組資料庫設定自動化作業或執行作業。 您可以使用 Azure 入口網站、Transact-SQL、PowerShell 或 REST API 來排程、執行和監視這些作業。 使用彈性資料庫作業來觸發一次清除記錄或依排程清除記錄的預存程序。 您可以根據 SSISDB 資源使用量選擇排程間隔，以避免資料庫負載過重。

如需詳細資訊，請參閱[使用彈性資料庫作業管理資料庫群組](../sql-database/elastic-jobs-overview.md)。

下列各節將說明如何觸發預存程序 `[internal].[cleanup_server_retention_window_exclusive]`，以將保留週期 (由系統管理員設定) 以外的 SSISDB 記錄移除。

## <a name="clean-up-logs-with-power-shell"></a>使用 Power Shell 清除記錄

下列 PowerShell 指令碼範例會建立新的彈性作業來觸發清除 SSISDB 記錄的預存程序。 如需詳細資訊，請參閱[使用 PowerShell 建立彈性作業代理程式](../sql-database/elastic-jobs-powershell.md)。

### <a name="create-parameters"></a>建立參數

``` powershell
# Parameters needed to create the Job Database
param(
$ResourceGroupName = $(Read-Host "Please enter an existing resource group name"),
$AgentServerName = $(Read-Host "Please enter the name of an existing Azure SQL server(for example, yhxserver) to hold the SSISDBLogCleanup job database"),
$SSISDBLogCleanupJobDB = $(Read-Host "Please enter a name for the Job Database to be created in the given SQL Server"),
# The Job Database should be a clean,empty,S0 or higher service tier. We set S0 as default.
$PricingTier = "S0",

# Parameters needed to create the Elastic Job agent
$SSISDBLogCleanupAgentName = $(Read-Host "Please enter a name for your new Elastic Job agent"),

# Parameters needed to create the job credential in the Job Databse to connect to SSISDB
$PasswordForSSISDBCleanupUser = $(Read-Host "Please provide a new password for SSISDBLogCleanup job user to connect to SSISDB database for log cleanup"),
# Parameters needed to create a login and a user in the SSISDB of the target server
$SSISDBServerEndpoint = $(Read-Host "Please enter the name of the target Azure SQL server which contains SSISDB you need to cleanup, for example, myserver") + '.database.windows.net',
$SSISDBServerAdminUserName = $(Read-Host "Please enter the target server admin username for SQL authentication"),
$SSISDBServerAdminPassword = $(Read-Host "Please enter the target server admin password for SQL authentication"),
$SSISDBName = "SSISDB",

# Parameters needed to set job scheduling to trigger execution of cleanup stored procedure
$RunJobOrNot = $(Read-Host "Please indicate whether you want to run the job to cleanup SSISDB logs outside the log retention window immediately(Y/N). Make sure the retention window is set appropriately before running the following powershell scripts. Those removed SSISDB logs cannot be recoverd"),
$IntervalType = $(Read-Host "Please enter the interval type for the execution schedule of SSISDB log cleanup stored procedure. For the interval type, Year, Month, Day, Hour, Minute, Second can be supported."),
$IntervalCount = $(Read-Host "Please enter the detailed interval value in the given interval type for the execution schedule of SSISDB log cleanup stored procedure"),
# StartTime of the execution schedule is set as the current time as default. 
$StartTime = (Get-Date)
```

### <a name="trigger-the-cleanup-stored-procedure"></a>觸發清除預存程序

```powershell
# Install the latest PackageManagement powershell package which PowershellGet v1.6.5 is dependent on
Find-Package PackageManagement -RequiredVersion 1.1.7.2 | Install-Package -Force
# You may need to restart the powershell session
# Install the latest PowershellGet module which adds the -AllowPrerelease flag to Install-Module
Find-Package PowerShellGet -RequiredVersion 1.6.5 | Install-Package -Force

# Place AzureRM.Sql preview cmdlets side by side with existing AzureRM.Sql version
Install-Module -Name AzureRM.Sql -AllowPrerelease -Force

# Sign in to your Azure account
Connect-AzureRmAccount

# Create a Job Database which is used for defining jobs of triggering SSISDB log cleanup stored procedure and tracking cleanup history of jobs
Write-Output "Creating a blank SQL database to be used as the SSISDBLogCleanup  Job Database ..."
$JobDatabase = New-AzureRmSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $AgentServerName -DatabaseName $SSISDBLogCleanupJobDB -RequestedServiceObjectiveName $PricingTier
$JobDatabase

# Enable the Elastic Jobs preview in your Azure subscription
Register-AzureRmProviderFeature -FeatureName sqldb-JobAccounts -ProviderNamespace Microsoft.Sql

# Create the Elastic Job agent
Write-Output "Creating the Elastic Job agent..."
$JobAgent = $JobDatabase | New-AzureRmSqlElasticJobAgent -Name $SSISDBLogCleanupAgentName
$JobAgent

# Create the job credential in the Job Database to connect to SSISDB database in the target server for log cleanup
Write-Output "Creating job credential to connect to SSISDB database..."
$JobCredSecure = ConvertTo-SecureString -String $PasswordForSSISDBCleanupUser -AsPlainText -Force
$JobCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList "SSISDBLogCleanupUser", $JobCredSecure
$JobCred = $JobAgent | New-AzureRmSqlElasticJobCredential -Name "SSISDBLogCleanupUser" -Credential $JobCred

# In the master database of the target SQL server which contains SSISDB to cleanup 
# - Create the job user login
Write-Output "Grant permissions on the master database of the target server..."
$Params = @{
  'Database' = 'master'
  'ServerInstance' = $SSISDBServerEndpoint
  'Username' = $SSISDBServerAdminUserName
  'Password' = $SSISDBServerAdminPassword
  'OutputSqlErrors' = $true
  'Query' = "CREATE LOGIN SSISDBLogCleanupUser WITH PASSWORD = '" + $PasswordForSSISDBCleanupUser + "'"
}
Invoke-SqlCmd @Params

# For SSISDB database of the target SQL server
# - Create the SSISDBLogCleanup user from the SSISDBlogCleanup user login
# - Grant permissions for the execution of SSISDB log cleanup stored procedure 
Write-Output "Grant appropriate permissions on SSISDB database..."
$TargetDatabase = $SSISDBName
$CreateJobUser = "CREATE USER SSISDBLogCleanupUser FROM LOGIN SSISDBLogCleanupUser"
$GrantStoredProcedureExecution = "GRANT EXECUTE ON internal.cleanup_server_retention_window_exclusive TO SSISDBLogCleanupUser"

$TargetDatabase | % {
  $Params.Database = $_
  $Params.Query = $CreateJobUser
  Invoke-SqlCmd @Params
  $Params.Query = $GrantStoredProcedureExecution
  Invoke-SqlCmd @Params
}

# Create a target group which includes SSISDB database needed to cleanup
Write-Output "Creating the target group including only SSISDB databse needed to cleanup ..."
$SSISDBTargetGroup = $JobAgent | New-AzureRmSqlElasticJobTargetGroup -Name "SSISDBTargetGroup"
$SSISDBTargetGroup | Add-AzureRmSqlElasticJobTarget -ServerName $SSISDBServerEndpoint -Database $SSISDBName 

# Create the job to trigger execution of SSISDB log cleanup stored procedure
Write-Output "Creating a new job to trigger execution of the stored procedure for SSISDB log cleanup"
$JobName = "CleanupSSISDBLog"
$Job = $JobAgent | New-AzureRmSqlElasticJob -Name $JobName -RunOnce
$Job

# Add the job step to execute internal.cleanup_server_retention_window_exclusive
Write-Output "Adding the job step for the cleanup stored procedure execution"
$SqlText = "EXEC internal.cleanup_server_retention_window_exclusive"
$Job | Add-AzureRmSqlElasticJobStep -Name "step to execute cleanup stored procedure" -TargetGroupName $SSISDBTargetGroup.TargetGroupName -CredentialName $JobCred.CredentialName -CommandText $SqlText

# Run the job to immediately start cleanup stored procedure execution for once
IF(($RunJobOrNot = "Y") -Or ($RunJobOrNot = "y"))
{
Write-Output "Start a new execution of the stored procedure for SSISDB log cleanup immediately..."
$JobExecution = $Job | Start-AzureRmSqlElasticJob
$JobExecution
}

# Schedule the job running to trigger stored procedure execution on schedule for removing SSISDB logs outside the retention window
Write-Output "Start the execution schedule of the stored procedure for SSISDB log cleanup..."
$Job | Set-AzureRmSqlElasticJob -IntervalType $IntervalType -IntervalCount $IntervalCount -StartTime $StartTime -Enable
```

## <a name="clean-up-logs-with-transact-sql"></a>使用 Transact-SQL 清除記錄

下列 Transact-SQL 指令碼範例會建立新的彈性作業來觸發清除 SSISDB 記錄的預存程序。 如需詳細資訊，請參閱[使用 Transact-SQL (T-SQL) 建立及管理彈性資料庫作業](../sql-database/elastic-jobs-tsql.md)。

1. 建立或識別作為 SSISDBCleanup 作業資料庫的空白 S0 或更高版本的 Azure SQL Database。 然後在 [Azure 入口網站](https://ms.portal.azure.com/#create/Microsoft.SQLElasticJobAgent)中建立彈性作業代理程式。

2. 在作業資料庫中，建立 SSISDB 記錄清除作業的認證。 此認證會用來連線到 SSISDB 資料庫，以清除記錄。

    ```sql
    -- Connect to the job database specified when creating the job agent
    -- Create a database master key if one does not already exist, using your own password.  
    CREATE MASTER KEY ENCRYPTION BY PASSWORD= '<EnterStrongPasswordHere>';  

    -- Create a credential for SSISDB log cleanup.  
    CREATE DATABASE SCOPED CREDENTIAL SSISDBLogCleanupCred WITH IDENTITY = 'SSISDBLogCleanupUser', SECRET = '<EnterStrongPasswordHere>'; 
    ```

3. 定義目標群組，以包含您要對其執行清除預存程序的 SSISDB 資料庫。

    ```sql
    -- Connect to the job database 
    -- Add a target group 
    EXEC jobs.sp_add_target_group 'SSISDBTargetGroup'

    -- Add SSISDB database into the target group
    EXEC jobs.sp_add_target_group_member 'SSISDBTargetGroup',
    @target_type = 'SqlDatabase',
    @server_name = '<EnterSSISDBTargetServerName>',
    @database_name = '<EnterSSISDBName>'

    --View the recently created target group and target group members
    SELECT * FROM jobs.target_groups WHERE target_group_name = 'SSISDBTargetGroup';
    SELECT * FROM jobs.target_group_members WHERE target_group_name = 'SSISDBTargetGroup';
    ```
4. 針對 SSISDB 資料庫授與適當的權限。 SSISDB 目錄必須具有適當的權限，才能讓預存程序順利地執行 SSISDB 記錄清除作業。 如需詳細指引，請參閱[管理登入](../sql-database/sql-database-manage-logins.md)。

    ```sql
    -- Connect to the master database in the target server including SSISDB 
    CREATE LOGIN SSISDBLogCleanupUser WITH PASSWORD = '<strong_password>';

    -- Connect to SSISDB database in the target server to cleanup logs
    CREATE USER SSISDBLogCleanupUser FROM LOGIN SSISDBLogCleanupUser;
    GRANT EXECUTE ON internal.cleanup_server_retention_window_exclusive TO SSISDBLogCleanupUser
    ```
5. 建立作業並新增作業步驟，以觸發預存程序執行清除 SSISDB 記錄的作業。

    ```sql
    --Connect to the job database 
    --Add the job for the execution of SSISDB log cleanup stored procedure.
    EXEC jobs.sp_add_job @job_name='CleanupSSISDBLog', @description='Remove SSISDB logs which are outside the retention window'

    --Add a job step to execute internal.cleanup_server_retention_window_exclusive
    EXEC jobs.sp_add_jobstep @job_name='CleanupSSISDBLog',
    @command=N'EXEC internal.cleanup_server_retention_window_exclusive',
    @credential_name='SSISDBLogCleanupCred',
    @target_group_name='SSISDBTargetGroup'
    ```
6. 繼續之前，請確定已適當地設定保留週期。 在週期外的 SSISDB 記錄會遭到刪除，並且無法復原。

   接著，您可以立即執行作業，以開始清除 SSISDB 記錄。

    ```sql
    --Connect to the job database 
    --Run the job immediately to execute the stored procedure for SSISDB log cleanup
    declare @je uniqueidentifier
    exec jobs.sp_start_job 'CleanupSSISDBLog', @job_execution_id = @je output

    --Watch the execution results for SSISDB log cleanup 
    select @je
    select * from jobs.job_executions where job_execution_id = @je
    ```
7. (選擇性) 排定作業執行，以根據排程移除保留週期以外的 SSISDB 記錄。 您可以使用類似的陳述式來更新作業參數。

    ```sql
    --Connect to the job database 
    EXEC jobs.sp_update_job
    @job_name='CleanupSSISDBLog',
    @enabled=1,
    @schedule_interval_type='<EnterIntervalType(Month,Day,Hour,Minute,Second)>',
    @schedule_interval_count='<EnterDetailedIntervalValue>',
    @schedule_start_time='<EnterProperStartTimeForSchedule>',
    @schedule_end_time='<EnterProperEndTimeForSchedule>'
    ```

## <a name="monitor-the-cleanup-job-in-the-azure-portal"></a>在 Azure 入口網站中監視清除作業

您可以在 Azure 入口網站中監視清除作業的執行。 針對每一次的執行，您會看到作業的狀態、開始時間和結束時間。

![在 Azure 入口網站中監視清除作業](media/how-to-clean-up-ssisdb-logs-with-elastic-jobs/monitor-cleanup-job-portal.png)

## <a name="monitor-the-cleanup-job-with-transact-sql"></a>使用 Transact-SQL 監視清除作業

您也可以使用 Transact-SQL 檢視清除作業的執行記錄。

```sql
--Connect to the job database 
--View all execution statuses for the job to cleanup SSISDB logs
SELECT * FROM jobs.job_executions WHERE job_name = 'CleanupSSISDBLog' 
ORDER BY start_time DESC

-- View all active executions
SELECT * FROM jobs.job_executions WHERE is_active = 1
ORDER BY start_time DESC
```

## <a name="next-steps"></a>後續步驟

如需管理和監視 Azure-SSIS Integration Runtime 的相關工作，請參閱下列文章。 Azure-SSIS IR 是 SSIS 套件的執行階段引擎，儲存在 Azure SQL Database 的 SSISDB 中。

-   [重新設定 Azure-SSIS 整合執行階段](manage-azure-ssis-integration-runtime.md)

-   [監視 Azure-SSIS 整合執行階段](monitor-integration-runtime.md#azure-ssis-integration-runtime)。
