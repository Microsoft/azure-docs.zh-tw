---
title: PowerShell 範例 - 更新 SQL 資料同步的同步結構描述 | Microsoft Docs
description: 更新 SQL 資料同步之同步結構描述的 Azure PowerShell 範例指令碼
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
ms.date: 01/10/2018
ms.author: xiwu
ms.reviewer: douglasl
ms.openlocfilehash: c121204e054618eef1435e64c28d959c0fe50ea9
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39617899"
---
# <a name="use-powershell-to-update-the-sync-schema-in-an-existing-sync-group"></a>使用 PowerShell 更新現有同步群組中的同步結構描述

此 PowerShell 範例會更新現有「SQL 資料同步」同步群組中的同步結構描述。 當您要同步處理多個資料表時，此指令碼可協助您有效率地更新同步結構描述。

此範例會示範 **UpdateSyncSchema** 指令碼的使用方式，這個指令碼在 GitHub 上會以 [UpdateSyncSchema.ps1](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/sql-data-sync/UpdateSyncSchema.ps1) 提供使用。

如需 SQL 資料同步的概觀，請參閱[使用 Azure SQL 資料同步，跨多個雲端和內部部署資料庫同步處理資料](../sql-database-sync-data.md)。
## <a name="prerequisites"></a>必要條件

此範例需要 Azure PowerShell 模組 4.2 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。
 
執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。

## <a name="examples"></a>範例

### <a name="example-1---add-all-tables-to-the-sync-schema"></a>範例 1 - 將所有的資料表新增至同步結構描述

下列範例會重新整理資料庫結構描述，並將中樞資料庫中所有有效的資料表，新增至同步結構描述。

```powershell
UpdateSyncSchema.ps1 -SubscriptionId <subscription_id> -ResourceGroupName <resource_group_name> -ServerName <server_name> -DatabaseName <database_name> -SyncGroupName <sync_group_name> -RefreshDatabaseSchema $true -AddAllTables $true
```

### <a name="example-2---add-and-remove-tables-and-columns"></a>範例 2 - 新增和移除資料表和資料行

下列範例會將 `[dbo].[Table1]` 和 `[dbo].[Table2].[Column1]` 新增至同步結構描述，並移除 `[dbo].[Table3]`。

```powershell
UpdateSyncSchema.ps1 -SubscriptionId <subscription_id> -ResourceGroupName <resource_group_name> -ServerName <server_name> -DatabaseName <database_name> -SyncGroupName <sync_group_name> -TablesAndColumnsToAdd "[dbo].[Table1],[dbo].[Table2].[Column1]" -TablesAndColumnsToRemove "[dbo].[Table3]"
```

## <a name="script-parameters"></a>指令碼參數

**UpdateSyncSchema** 指令碼具有下列參數︰

| 參數 | 注意 |
|---|---|
| $SubscriptionId | 其中建立同步群組的訂閱。 |
| $ResourceGroupName | 其中建立同步群組的資源群組。|
| $ServerName | 中樞資料庫的伺服器名稱。|
| $DatabaseName | 中樞資料庫名稱。 |
| $SyncGroupName | 同步群組名稱。 |
| $MemberName | 如果您希望從同步成員而非從中樞資料庫載入資料庫結構描述，請指定成員名稱。 如果您希望從中樞資料庫載入資料庫結構描述，請將此參數保留空白。 |
| $TimeoutInSeconds | 當指令碼重新整理資料庫結構描述時等候逾時。 預設值為 900 秒。 |
| $RefreshDatabaseSchema | 指定指令碼是否需要重新整理資料庫結構描述。 如果您的資料庫結構描述與先前的設定有所變更 - 例如，如果您已新增新的資料表或者新的資料行，您必須重新整理結構描述，才能重新設定它。 預設值為 false。 |
| $AddAllTables | 如果此值為 true，所有有效的資料表和資料行會新增到同步結構描述中。 $TablesAndColumnsToAdd and $TablesAndColumnsToRemove 的值會被忽略。 |
| $TablesAndColumnsToAdd | 指定要新增到同步結構描述中的資料表或資料行。 每個資料表或資料行名稱必須使用分隔符號與結構描述名稱完全分隔。 例如：`[dbo].[Table1]`, `[dbo].[Table2].[Column1]`。 可以指定多個資料表或資料行名稱，並以以逗號 (,) 分隔。 |
| $TablesAndColumnsToRemove | 指定要從同步結構描述中移除的資料表或資料行。 每個資料表或資料行名稱必須使用分隔符號與結構描述名稱完全分隔。 例如：`[dbo].[Table1]`, `[dbo].[Table2].[Column1]`。 可以指定多個資料表或資料行名稱，並以以逗號 (,) 分隔。 |
|||

## <a name="script-explanation"></a>指令碼說明

**UpdateSyncSchema** 指令碼具有下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmSqlSyncGroup](https://docs.microsoft.com/powershell/module/azurerm.sql/get-azurermsqlsyncgroup) | 傳回同步群組的相關資訊。 |
| [Update-AzureRmSqlSyncGroup](https://docs.microsoft.com/powershell/module/azurerm.sql/update-azurermsqlsyncgroup) | 更新同步群組。 |
| [Get-AzureRmSqlSyncMember](https://docs.microsoft.com/powershell/module/azurerm.sql/get-azurermsqlsyncmember) | 傳回同步成員的相關資訊。 |
| [Get-AzureRmSqlSyncSchema](https://docs.microsoft.com/powershell/module/azurerm.sql/get-azurermsqlsyncschema) | 傳回同步結構描述的相關資訊。 |
| [Update-AzureRmSqlSyncSchema](https://docs.microsoft.com/powershell/module/azurerm.sql/update-azurermsqlsyncschema) | 更新同步結構描述。 |
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
    -   [使用 PowerShell 在多個 Azure SQL Database 之間進行同步處理](sql-database-sync-data-between-sql-databases.md)
    -   [使用 PowerShell 設定「資料同步」在內部部署的 Azure SQL Database 和 SQL Server 之間進行同步處理](sql-database-sync-data-between-azure-onprem.md)

如需 SQL Database 的詳細資訊，請參閱：

-   [SQL Database 概觀](../sql-database-technical-overview.md)
-   [資料庫生命週期管理](https://msdn.microsoft.com/library/jj907294.aspx)
