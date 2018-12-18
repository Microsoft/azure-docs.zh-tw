---
title: PowerShell 範例 - 在多個 Azure SQL Database 之間進行同步處理 | Microsoft Docs
description: 在多個 Azure SQL Database 之間進行同步處理的 Azure PowerShell 範例指令碼
services: sql-database
documentationcenter: sql-database
author: allenwux
manager: craigg
editor: ''
tags: ''
ms.assetid: ''
ms.service: sql-database
ms.custom: load & move data, mvc
ms.devlang: PowerShell
ms.topic: sample
ms.tgt_pltfrm: sql-database
ms.workload: database
ms.date: 04/01/2018
ms.author: xiwu
ms.reviewer: douglasl
ms.openlocfilehash: 3091d772c9650c816e8ec56ea4a01b3819deabc9
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44715783"
---
# <a name="use-powershell-to-sync-between-multiple-sql-databases"></a>使用 PowerShell 在多個 SQL Database 之間進行同步處理
 
此 PowerShell 範例會設定「資料同步」，以在多個 Azure SQL Database 之間進行同步處理。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

如需 SQL 資料同步的概觀，請參閱[使用 Azure SQL 資料同步，跨多個雲端和內部部署資料庫同步處理資料](../sql-database-sync-data.md)。

## <a name="sample-script"></a>範例指令碼

```powershell-interactive
# prerequisites: 
# 1. Create an Azure Database from AdventureWorksLT sample database as hub database
# 2. Create an Azure Database in the same region as sync database
# 3. Create an Azure Database as member database
# 4. Update the parameters below before running the sample
#
using namespace Microsoft.Azure.Commands.Sql.DataSync.Model
using namespace System.Collections.Generic

# Hub database info
# Subscription id for hub database
$SubscriptionId = "subscription_guid"
# Resource group name for hub database
$ResourceGroupName = "ResourceGroup"
# Server name for hub database
$ServerName = "Server"
# Database name for hub database
$DatabaseName = "AdventureWorks"

# Sync database info
# Resource group name for sync database
$SyncDatabaseResourceGroupName = "ResourceGroup"
# Server name for sync database
$SyncDatabaseServerName = "Server"
# Sync database name
$SyncDatabaseName = "SyncDatabase"

# Sync group info
# Sync group name
$SyncGroupName = "SampleSyncGroup1"
# Conflict resolution Policy. Value can be HubWin or MemberWin
$ConflictResolutionPolicy = "HubWin"
# Sync interval in seconds. Value must be no less than 300
$IntervalInSeconds = 300

# Member database info
# Member name
$SyncMemberName = "member"
# Member server name
$MemberServerName = "MemberServer"
# Member database name
$MemberDatabaseName = "SyncDatabase1"
# Member database type. Value can be AzureSqlDatabase or SqlServerDatabase
$MemberDatabaseType = "AzureSqlDatabase"
# Sync direction. Value can be Bidirectional, Onewaymembertohub, Onewayhubtomember
$SyncDirection = "Bidirectional"

# Other info
# Temp file to save the sync schema
$TempFile = $env:TEMP+"\syncSchema.json"

# List of included columns and tables in quoted name
$IncludedColumnsAndTables =  "[SalesLT].[Address].[AddressID]",
                             "[SalesLT].[Address].[AddressLine2]",
                             "[SalesLT].[Address].[rowguid]",
                             "[SalesLT].[Address].[PostalCode]",
                             "[SalesLT].[ProductDescription]"
$MetadataList = [System.Collections.ArrayList]::new($IncludedColumnsAndTables)


Connect-AzureRmAccount 
select-azurermsubscription -SubscriptionId $SubscriptionId

# Use this section if it is safe to show password in the script.
# Otherwise, use the PromptForCredential
# $User = "username"
# $PWord = ConvertTo-SecureString -String "Password" -AsPlainText -Force
# $Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $User, $PWord

$Credential = $Host.ui.PromptForCredential("Need credential", 
              "Please enter your user name and password for server "+$ServerName+".database.windows.net", 
              "", 
              "")

# Create a new sync group
Write-Host "Creating Sync Group"$SyncGroupName
New-AzureRmSqlSyncGroup   -ResourceGroupName $ResourceGroupName `
                            -ServerName $ServerName `
                            -DatabaseName $DatabaseName `
                            -Name $SyncGroupName `
                            -SyncDatabaseName $SyncDatabaseName `
                            -SyncDatabaseServerName $SyncDatabaseServerName `
                            -SyncDatabaseResourceGroupName $SyncDatabaseResourceGroupName `
                            -ConflictResolutionPolicy $ConflictResolutionPolicy `
                            -DatabaseCredential $Credential

# Use this section if it is safe to show password in the script.
#$User = "username"
#$Password = ConvertTo-SecureString -String "password" -AsPlainText -Force
#$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $User, $Password

$Credential = $Host.ui.PromptForCredential("Need credential", 
              "Please enter your user name and password for server "+$MemberServerName, 
              "", 
              "")

# Add a new sync member
Write-Host "Adding member"$SyncMemberName" to the sync group"
New-AzureRmSqlSyncMember   -ResourceGroupName $ResourceGroupName `
                            -ServerName $ServerName `
                            -DatabaseName $DatabaseName `
                            -SyncGroupName $SyncGroupName `
                            -Name $SyncMemberName `
                            -MemberDatabaseCredential $Credential `
                            -MemberDatabaseName $MemberDatabaseName `
                            -MemberServerName ($MemberServerName + ".database.windows.net") `
                            -MemberDatabaseType $MemberDatabaseType `
                            -SyncDirection $SyncDirection

# Refresh database schema from hub database
# Specify the -SyncMemberName parameter if you want to refresh schema from the member database
Write-Host "Refreshing database schema from hub database"
$StartTime= Get-Date
Update-AzureRmSqlSyncSchema   -ResourceGroupName $ResourceGroupName `
                              -ServerName $ServerName `
                              -DatabaseName $DatabaseName `
                              -SyncGroupName $SyncGroupName


#Waiting for successful refresh

$StartTime=$StartTime.ToUniversalTime()
$timer=0
$timeout=90
# Check the log and see if refresh has gone through
Write-Host "Check for successful refresh"
$IsSucceeded = $false
While ($IsSucceeded -eq $false)
{
    Start-Sleep -s 10
    $timer=$timer+10
    $Details = Get-AzureRmSqlSyncSchema -SyncGroupName $SyncGroupName -ServerName $ServerName -DatabaseName $DatabaseName -ResourceGroupName $ResourceGroupName
    if ($Details.LastUpdateTime -gt $StartTime)
      {
        Write-Host "Refresh was successful"
        $IsSucceeded = $true
      }
    if ($timer -eq $timeout) 
      {
              Write-Host "Refresh timed out"
        break;
      }
}



# Get the database schema 
Write-Host "Adding tables and columns to the sync schema"
$databaseSchema = Get-AzureRmSqlSyncSchema   -ResourceGroupName $ResourceGroupName `
                                             -ServerName $ServerName `
                                             -DatabaseName $DatabaseName `
                                             -SyncGroupName $SyncGroupName `

$databaseSchema | ConvertTo-Json -depth 5 -Compress | Out-File "c:\tmp\databaseSchema"     
$newSchema = [AzureSqlSyncGroupSchemaModel]::new()
$newSchema.Tables = [List[AzureSqlSyncGroupSchemaTableModel]]::new();

# Add columns and tables to the sync schema
foreach ($tableSchema in $databaseSchema.Tables)
{
    $newTableSchema = [AzureSqlSyncGroupSchemaTableModel]::new()
    $newTableSchema.QuotedName = $tableSchema.QuotedName
    $newTableSchema.Columns = [List[AzureSqlSyncGroupSchemaColumnModel]]::new();
    $addAllColumns = $false
    if ($MetadataList.Contains($tableSchema.QuotedName))
    {
        if ($tableSchema.HasError)
        {
            $fullTableName = $tableSchema.QuotedName
            Write-Host "Can't add table $fullTableName to the sync schema" -foregroundcolor "Red"
            Write-Host $tableSchema.ErrorId -foregroundcolor "Red"
            continue;
        }
        else
        {
            $addAllColumns = $true
        }
    }
    foreach($columnSchema in $tableSchema.Columns)
    {
        $fullColumnName = $tableSchema.QuotedName + "." + $columnSchema.QuotedName
        if ($addAllColumns -or $MetadataList.Contains($fullColumnName))
        {
            if ((-not $addAllColumns) -and $tableSchema.HasError)
            {
                Write-Host "Can't add column $fullColumnName to the sync schema" -foregroundcolor "Red"
                Write-Host $tableSchema.ErrorId -foregroundcolor "Red"c            }
            elseif ((-not $addAllColumns) -and $columnSchema.HasError)
            {
                Write-Host "Can't add column $fullColumnName to the sync schema" -foregroundcolor "Red"
                Write-Host $columnSchema.ErrorId -foregroundcolor "Red"
            }
            else
            {
                Write-Host "Adding"$fullColumnName" to the sync schema"
                $newColumnSchema = [AzureSqlSyncGroupSchemaColumnModel]::new()
                $newColumnSchema.QuotedName = $columnSchema.QuotedName
                $newColumnSchema.DataSize = $columnSchema.DataSize
                $newColumnSchema.DataType = $columnSchema.DataType
                $newTableSchema.Columns.Add($newColumnSchema)
            }
        }
    }
    if ($newTableSchema.Columns.Count -gt 0)
    {
        $newSchema.Tables.Add($newTableSchema)
    }
}

# Convert sync schema to Json format
$schemaString = $newSchema | ConvertTo-Json -depth 5 -Compress

# workaround a powershell bug
$schemaString = $schemaString.Replace('"Tables"', '"tables"').Replace('"Columns"', '"columns"').Replace('"QuotedName"', '"quotedName"').Replace('"MasterSyncMemberName"','"masterSyncMemberName"')

# Save the sync schema to a temp file
$schemaString | Out-File $TempFile

# Update sync schema
Write-Host "Updating the sync schema"
Update-AzureRmSqlSyncGroup  -ResourceGroupName $ResourceGroupName `
                            -ServerName $ServerName `
                            -DatabaseName $DatabaseName `
                            -Name $SyncGroupName `
                            -Schema $TempFile

$SyncLogStartTime = Get-Date

# Trigger sync manually
Write-Host "Trigger sync manually"
Start-AzureRmSqlSyncGroupSync  -ResourceGroupName $ResourceGroupName `
                               -ServerName $ServerName `
                               -DatabaseName $DatabaseName `
                               -SyncGroupName $SyncGroupName

# Check the sync log and wait until the first sync succeeded
Write-Host "Check the sync log"
$IsSucceeded = $false
For ($i = 0; ($i -lt 300) -and (-not $IsSucceeded); $i = $i + 10)
{
    Start-Sleep -s 10
    $SyncLogEndTime = Get-Date
    $SyncLogList = Get-AzureRmSqlSyncGroupLog  -ResourceGroupName $ResourceGroupName `
                                           -ServerName $ServerName `
                                           -DatabaseName $DatabaseName `
                                           -SyncGroupName $SyncGroupName `
                                           -StartTime $SyncLogStartTime.ToUniversalTime() `
                                           -EndTime $SyncLogEndTime.ToUniversalTime()
    if ($SynclogList.Length -gt 0)
    {
        foreach ($SyncLog in $SyncLogList)
        {
            if ($SyncLog.Details.Contains("Sync completed successfully"))
            {
                Write-Host $SyncLog.TimeStamp : $SyncLog.Details
                $IsSucceeded = $true
            }
        }
    }
}

if ($IsSucceeded)
{
    # Enable scheduled sync
    Write-Host "Enable the scheduled sync with 300 seconds interval"
    Update-AzureRmSqlSyncGroup  -ResourceGroupName $ResourceGroupName `
                                -ServerName $ServerName `
                                -DatabaseName $DatabaseName `
                                -Name $SyncGroupName `
                                -IntervalInSeconds $IntervalInSeconds
}
else
{
    # Output all log if sync doesn't succeed in 300 seconds
    $SyncLogEndTime = Get-Date
    $SyncLogList = Get-AzureRmSqlSyncGroupLog  -ResourceGroupName $ResourceGroupName `
                                           -ServerName $ServerName `
                                           -DatabaseName $DatabaseName `
                                           -SyncGroupName $SyncGroupName `
                                           -StartTime $SyncLogStartTime.ToUniversalTime() `
                                           -EndTime $SyncLogEndTime.ToUniversalTime()
    if ($SynclogList.Length -gt 0)
    {
        foreach ($SyncLog in $SyncLogList)
        {
            Write-Host $SyncLog.TimeStamp : $SyncLog.Details
        }
    }
}

# Clean up deployment 
# Remove-AzureRmResourceGroup -ResourceGroupName $resourcegroupname
# Remove-AzureRmResourceGroup -ResourceGroupName $SyncDatabaseResourceGroupName

```

## <a name="clean-up-deployment"></a>清除部署

執行範例指令碼之後，您可以執行下列命令以移除資源群組及與其相關聯的所有資源。

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName $ResourceGroupName
Remove-AzureRmResourceGroup -ResourceGroupName $SyncDatabaseResourceGroupName
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmSqlSyncAgent](/powershell/module/azurerm.sql/New-AzureRmSqlSyncAgent) |  建立新的同步代理程式 |
| [New-AzureRmSqlSyncAgentKey](/powershell/module/azurerm.sql/New-AzureRmSqlSyncAgentKey) |  產生與同步代理程式相關聯的代理程式金鑰 |
| [Get-AzureRmSqlSyncAgentLinkedDatabase](/powershell/module/azurerm.sql/Get-AzureRmSqlSyncAgentLinkedDatabase) |  取得同步代理程式的所有資訊 |
| [New-AzureRmSqlSyncMember](/powershell/module/azurerm.sql/New-AzureRmSqlSyncMember) |  在同步群組中新增成員 |
| [Update-AzureRmSqlSyncSchema](/powershell/module/azurerm.sql/Update-AzureRmSqlSyncSchema) |  重新整理資料庫結構描述資訊 |
| [Get-AzureRmSqlSyncSchema](https://docs.microsoft.com/powershell/module/azurerm.sql/Get-AzureRmSqlSyncSchema?view=azurermps-6.8.1) |  取得資料庫結構描述資訊 |
| [Update-AzureRmSqlSyncGroup](/powershell/module/azurerm.sql/Update-AzureRmSqlSyncGroup) |  更新同步群組 |
| [Start-AzureRmSqlSyncGroupSync](/powershell/module/azurerm.sql/Start-AzureRmSqlSyncGroupSync) | 觸發同步處理 |
| [Get-AzureRmSqlSyncGroupLog](/powershell/module/azurerm.sql/Get-AzureRmSqlSyncGroupLog) |  檢查同步記錄 |
|||

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure SQL Database PowerShell 指令碼](../sql-database-powershell-samples.md)中找到其他 SQL Database PowerShell 指令碼範例。

如需 SQL 資料同步的詳細資訊，請參閱：

-   [使用 Azure SQL 資料同步，跨多個雲端和內部部署資料庫同步處理資料](../sql-database-sync-data.md)
-   [設定 Azure SQL 資料同步](../sql-database-get-started-sql-data-sync.md)
-   [Azure SQL 資料同步最佳做法](../sql-database-best-practices-data-sync.md)
-   [透過 Log Analytics 監視 Azure SQL 資料同步](../sql-database-sync-monitor-oms.md)
-   [對 Azure SQL 資料同步的問題進行疑難排解](../sql-database-troubleshoot-data-sync.md)

-   示範如何設定 SQL 資料同步的完整 PowerShell 範例：
    -   [使用 PowerShell 設定「資料同步」在內部部署的 Azure SQL Database 和 SQL Server 之間進行同步處理](sql-database-sync-data-between-azure-onprem.md)

如需 SQL Database 的詳細資訊，請參閱：

-   [SQL Database 概觀](../sql-database-technical-overview.md)
-   [資料庫生命週期管理](https://msdn.microsoft.com/library/jj907294.aspx)
