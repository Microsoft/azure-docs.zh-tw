---
title: Azure SQL 資料倉儲的 PowerShell Cmdlet
description: 尋找 Azure SQL 資料倉儲的前幾個 PowerShell Cmdlet，包括如何暫停和繼續資料庫。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 222bc8ee15fdc8802dacd5a5b74cfd84961aa397
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43300748"
---
# <a name="powershell-cmdlets-and-rest-apis-for-sql-data-warehouse"></a>適用於 SQL 資料倉儲的 PowerShell Cmdlet 和 REST API
您可使用 Azure PowerShell Cmdlet 或 REST API 管理許多 SQL 資料倉儲系統管理工作。  下列為一些在 SQL 資料倉儲中使用 PowerShell 命令來自動化一般工作的範例。  如需一些良好的 REST 範例，請參閱[使用 REST 管理延展性][Manage scalability with REST]一文。

> [!NOTE]
> 若要搭配使用 Azure Powershell 與 SQL 資料倉儲，需要有 Azure PowerShell 1.0.3 版或更高版本。  您可以執行 **Get-Module -ListAvailable -Name Azure**來檢查您的版本。  可透過 [Microsoft Web Platform Installer][Microsoft Web Platform Installer]安裝最新的版本。  如需安裝最新版本的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell][How to install and configure Azure PowerShell]。
> 
> 

## <a name="get-started-with-azure-powershell-cmdlets"></a>開始使用 Azure PowerShell Cmdlet
1. 開啟 Windows PowerShell。
2. 在 PowerShell 提示中，執行下列命令來登入 Azure Resource Manager，並選取您的訂用帳戶。
   
    ```PowerShell
    Connect-AzureRmAccount
    Get-AzureRmSubscription
    Select-AzureRmSubscription -SubscriptionName "MySubscription"
    ```

## <a name="pause-sql-data-warehouse-example"></a>暫停 SQL 資料倉儲範例
暫停 "Server01" 伺服器上託管的 "Database02" 資料庫。  此伺服器位於「ResourceGroup1」這個 Azure 資源群組。

```Powershell
Suspend-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
```
在以下另一種變化的範例中，會將擷取的物件輸送到 [Suspend-AzureRmSqlDatabase][Suspend-AzureRmSqlDatabase]。  結果就是暫停資料庫。 最終的命令會顯示結果。

```Powershell
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Suspend-AzureRmSqlDatabase
$resultDatabase
```

## <a name="start-sql-data-warehouse-example"></a>啟動 SQL 資料倉儲範例
繼續 "Server01" 伺服器上託管之 "Database02" 資料庫的作業。 此伺服器包含在「ResourceGroup1」這個資源群組中。

```Powershell
Resume-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" -DatabaseName "Database02"
```

一種變化，此範例會從 "ResourceGroup1" 資源群組包含的 "Server01" 伺服器中，擷取 "Database02" 資料庫。 它將擷取的物件輸送到 [Resume-AzureRmSqlDatabase][Resume-AzureRmSqlDatabase]。

```Powershell
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Resume-AzureRmSqlDatabase
```

> [!NOTE]
> 請注意，如果您的伺服器是 foo.database.windows.net，請使用 "foo" 作為 PowerShell Cmdlet 中的 -ServerName。
> 
> 

## <a name="other-supported-powershell-cmdlets"></a>其他支援的 PowerShell Cmdlet
這些 PowerShell Cmdlet 皆由 Azure SQL 資料倉儲所支援。

* [Get-AzureRmSqlDatabase][Get-AzureRmSqlDatabase]
* [Get-AzureRmSqlDeletedDatabaseBackup][Get-AzureRmSqlDeletedDatabaseBackup]
* [Get-AzureRmSqlDatabaseRestorePoints][Get-AzureRmSqlDatabaseRestorePoints]
* [New-AzureRmSqlDatabase][New-AzureRmSqlDatabase]
* [Remove-AzureRmSqlDatabase][Remove-AzureRmSqlDatabase]
* [Restore-AzureRmSqlDatabase][Restore-AzureRmSqlDatabase]
* [Resume-AzureRmSqlDatabase][Resume-AzureRmSqlDatabase]
* [Select-AzureRmSubscription][Select-AzureRmSubscription]
* [Set-AzureRmSqlDatabase][Set-AzureRmSqlDatabase]
* [Suspend-AzureRmSqlDatabase][Suspend-AzureRmSqlDatabase]

## <a name="next-steps"></a>後續步驟
如需更多 PowerShell 範例，請參閱：

* [使用 PowerShell 建立 SQL 資料倉儲][Create a SQL Data Warehouse using PowerShell]
* [資料庫還原][Database restore]

如需可以使用 PowerShell 進行自動化的其他工作，請參閱 [Azure SQL Database Cmdlet][Azure SQL Database Cmdlets]。 請注意，Azure SQL 資料倉儲並沒有支援所有的 Azure SQL Database Cmdlet。  如需可以使用 REST 來自動化的工作清單，請參閱 [Azure SQL Database 的作業][Operations for Azure SQL Databases]。

<!--Image references-->

<!--Article references-->
[How to install and configure Azure PowerShell]: /powershell/azureps-cmdlets-docs
[Create a SQL Data Warehouse using PowerShell]: ./create-data-warehouse-powershell.md
[Database restore]: ./sql-data-warehouse-restore-database-powershell.md
[Manage scalability with REST]: ./sql-data-warehouse-manage-compute-rest-api.md

<!--MSDN references-->
[Azure SQL Database Cmdlets]: https://docs.microsoft.com/powershell/module/azurerm.sql
[Operations for Azure SQL Databases]: https://msdn.microsoft.com/library/azure/dn505719.aspx
[Get-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/get-azurermsqldatabase
[Get-AzureRmSqlDeletedDatabaseBackup]: https://docs.microsoft.com/powershell/module/azurerm.sql/get-AzureRmSqlDeletedDatabaseBackup
[Get-AzureRmSqlDatabaseRestorePoints]: https://docs.microsoft.com/powershell/module/azurerm.sql/get-AzureRmSqlDatabaseRestorePoints
[New-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/New-AzureRmSqlDatabase
[Remove-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/Remove-AzureRmSqlDatabase
[Restore-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/Restore-AzureRmSqlDatabase
[Resume-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/Resume-AzureRmSqlDatabase
<!-- It appears that Select-AzureRmSubscription isn't documented, so this points to Select-AzureSubscription -->
[Select-AzureRmSubscription]: https://msdn.microsoft.com/library/dn722499.aspx
[Set-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/Set-AzureRmSqlDatabase
[Suspend-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/Suspend-AzureRmSqlDatabase

<!--Other Web references-->
[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps
