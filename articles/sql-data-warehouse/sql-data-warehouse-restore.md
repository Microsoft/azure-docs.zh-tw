---
title: 還原 Azure SQL 資料倉儲 | Microsoft Docs
description: 還原 Azure SQL 資料倉儲的做法指南。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 08/29/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 6eba50fbe7c2a7a40b08e37a96adac66583b8251
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43781855"
---
# <a name="restoring-azure-sql-data-warehouse"></a>還原 Azure SQL 資料倉儲 
在本文中，您將了解如何在 Azure 入口網站和 PowerShell 中執行下列作業：

- 建立還原點
- 從自動還原點或使用者定義的還原點進行還原
- 從已刪除的資料庫還原
- 從異地備份還原
- 從使用者定義的還原點建立資料倉儲副本

> [!NOTE]
> 自 8/27 起，跨伺服器還原已因已知的迴歸而停用。 我們以最高優先順序積極修正中。 很抱歉造成您的不便。 在此同時，您可以使用[異地備份](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-restore#restore-from-an-azure-geographical-region)進行跨伺服器還原。  
>

## <a name="before-you-begin"></a>開始之前
**請驗證您的 DTU 容量。** 每個 SQL 資料倉儲均由具有預設 DTU 配額的 SQL 伺服器裝載 (例如 myserver.database.windows.net)。  在您還原 SQL 資料倉儲之前，請確認您的 SQL 伺服器有足夠的剩餘 DTU 配額供要還原的資料庫使用。 若要了解如何計算所需 DTU 或要求更多 DTU，請參閱[要求 DTU 配額變更][Request a DTU quota change]。

## <a name="restore-through-powershell"></a>透過 PowerShell 還原

## <a name="install-powershell"></a>安裝 PowerShell
若要搭配使用 Azure Powershell 與 SQL 資料倉儲，您需要安裝 Azure PowerShell 1.0 版或更高版本。  您可以執行 **Get-Module -ListAvailable -Name AzureRM**來檢查您的版本。  可透過 [Microsoft Web Platform Installer][Microsoft Web Platform Installer]安裝最新的版本。  如需安裝最新版本的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell][How to install and configure Azure PowerShell]。

## <a name="restore-an-active-or-paused-database-using-powershell"></a>使用 PowerShell 還原作用中或已暫停的資料庫
若要從還原點還原資料庫，請使用 [Restore-AzureRmSqlDatabase][Restore-AzureRmSqlDatabase] PowerShell Cmdlet。

1. 開啟 Windows PowerShell。

2. 連接到您的 Azure 帳戶，然後列出與您帳戶關聯的所有訂用帳戶。

3. 選取包含要還原之資料庫的訂用帳戶。

4. 列出資料庫的還原點。

5. 使用 RestorePointCreationDate 挑選出想要的還原點。

   > [!NOTE]
   > 還原時，您可以指定不同的 ServiceObjectiveName (DWU) 或位在不同區域的不同伺服器。

6. 將資料庫還原到想要的還原點。

7. 確認還原的資料庫在線上。

```Powershell

$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"

Connect-AzureRmAccount
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName $SubscriptionName

# List the last 10 database restore points
((Get-AzureRMSqlDatabaseRestorePoints -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName ($DatabaseName)).RestorePointCreationDate)[-10 .. -1]

# Or list all restore points
Get-AzureRmSqlDatabaseRestorePoints -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Get the specific database to restore
$Database = Get-AzureRmSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Pick desired restore point using RestorePointCreationDate
$PointInTime="<RestorePointCreationDate>"  

# Restore database from a restore point
$RestoredDatabase = Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $Database.ResourceID

# Verify the status of restored database
$RestoredDatabase.status

```

> [!NOTE]
> 還原完成後，您可以遵循[在復原之後設定資料庫][Configure your database after recovery]來設定復原的資料庫。
>

## <a name="copy-your-data-warehouse-with-user-defined-restore-points-using-powershell"></a>使用 PowerShell 透過使用者定義的還原點來複製資料倉儲
若要從使用者定義的還原點還原資料庫，請使用 [Restore-AzureRmSqlDatabase][Restore-AzureRmSqlDatabase] PowerShell Cmdlet。

1. 開啟 Windows PowerShell。
2. 連接到您的 Azure 帳戶，然後列出與您帳戶關聯的所有訂用帳戶。
3. 選取包含要還原之資料庫的訂用帳戶。
4. 建立用來立即複製資料庫的還原點
5. 將您的資料庫重新命名為暫時的名稱。
6. 透過指定的 RestorePointLabel 擷取最新還原點。
7. 取得資料庫的資源識別碼以起始還原作業
8. 將資料庫還原到想要的還原點。
9. 確認還原的資料庫在線上。

```Powershell

$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
$DatabaseName="<YourDatabaseName>"
$TempDatabaseName = "<YourTemporaryDatabaseName>"
$Label = "<YourRestorePointLabel"

Connect-AzureRmAccount
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName $SubscriptionName

# Create a restore point of the original database
New-AzureRmSqlDatabaseRestorePoint -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName -RestorePointLabel $Label

# Rename the database to a temporary name
Set-AzureRmSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName -NewName $TempDatabaseName

# Get the most recent restore point with the specified label
$LabelledRestorePoint = Get-AzureRmSqlDatabaseRestorePoints -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $TempDatabaseName | where {$_.RestorePointLabel -eq $Label} | sort {$_.RestorePointCreationDate} | select -Last 1

# Get the resource id of the database
$ResourceId = (Get-AzureRmSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $TempDatabaseName).ResourceId

# Restore the database to its original name from the labelled restore point from the temporary database
$RestoredDatabase = Restore-AzureRmSqlDatabase -FromPointInTimeBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -ResourceId $ResourceId -PointInTime $LabelledRestorePoint.RestorePointCreationDate -TargetDatabaseName $DatabaseName

# Verify the status of restored database
$RestoredDatabase.status

# The original temporary database can be deleted at this point

```

## <a name="restore-a-deleted-database-using-powershell"></a>使用 PowerShell 還原已刪除的資料庫
若要還原已刪除的資料庫，請使用 [Restore-AzureRmSqlDatabase][Restore-AzureRmSqlDatabase] Cmdlet。

1. 開啟 Windows PowerShell。
2. 連接到您的 Azure 帳戶，然後列出與您帳戶關聯的所有訂用帳戶。
3. 選取包含要還原之已刪除資料庫的訂用帳戶。
4. 取得已刪除的特定資料庫。
5. 還原已刪除的資料庫。
6. 確認還原的資料庫在線上。

```Powershell
$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"

Connect-AzureRmAccount
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName $SubscriptionName

# Get the deleted database to restore
$DeletedDatabase = Get-AzureRmSqlDeletedDatabaseBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Restore deleted database
$RestoredDatabase = Restore-AzureRmSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID

# Verify the status of restored database
$RestoredDatabase.status
```

> [!NOTE]
> 還原完成後，您可以遵循[在復原之後設定資料庫][Configure your database after recovery]來設定復原的資料庫。
>

## <a name="restore-from-an-azure-geographical-region-using-powershell"></a>使用 PowerShell 從 Azure 地理區域還原
若要還原資料庫，請使用 [Restore-AzureRmSqlDatabase][Restore-AzureRmSqlDatabase] Cmdlet。

> [!NOTE]
> 您可以執行異地還原來還原至 Gen2！ 若要這麼做，請指定 Gen2 ServiceObjectiveName (例如 DW1000**c**) 作為選擇性參數。
>

1. 開啟 Windows PowerShell。
2. 連接到您的 Azure 帳戶，然後列出與您帳戶關聯的所有訂用帳戶。
3. 選取包含要還原之資料庫的訂用帳戶。
4. 取得您想要復原的資料庫。
5. 建立資料庫的復原要求。
6. 確認異地還原資料庫的狀態。

```Powershell
Connect-AzureRmAccount
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

# Get the database you want to recover
$GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

# Recover database
$GeoRestoredDatabase = Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourTargetServer>" -TargetDatabaseName "<NewDatabaseName>" –ResourceId $GeoBackup.ResourceID -ServiceObjectiveName "<YourTargetServiceLevel>"

# Verify that the geo-restored database is online
$GeoRestoredDatabase.status
```

> [!NOTE]
> 若要在還原完成之後設定資料庫，請參閱[在復原之後設定資料庫][Configure your database after recovery]。
>

如果來源資料庫是啟用 TDE，則復原的資料庫將是啟用 TDE。

## <a name="restore-through-the-azure-portal"></a>透過 Azure 入口網站還原

## <a name="create-a-user-defined-restore-point-using-the-azure-portal"></a>使用 Azure 入口網站建立使用者定義的還原點
1. 登入 [Azure 入口網站][Azure portal]。

2. 瀏覽至您想要為其建立還原點的 SQL 資料倉儲。

3. 在 [概觀] 刀鋒視窗頂端，選取 [+ 新增還原點]。

    ![新增還原點](./media/sql-data-warehouse-restore-database-portal/creating_restore_point_0.png)

4. 指定還原點的名稱。

    ![還原點名稱](./media/sql-data-warehouse-restore-database-portal/creating_restore_point_1.png)

## <a name="restore-an-active-or-paused-database-using-the-azure-portal"></a>使用 Azure 入口網站還原作用中或已暫停的資料庫
1. 登入 [Azure 入口網站][Azure portal]。
2. 瀏覽至您想要還原的 SQL 資料倉儲。
3. 在 [概觀] 刀鋒視窗頂端，選取 [還原]。

    ![ 還原概觀](./media/sql-data-warehouse-restore-database-portal/restoring_0.png)

4. 選取 [自動還原點] 或 [使用者定義的還原點]。

    ![自動還原點](./media/sql-data-warehouse-restore-database-portal/restoring_1.png)

5. 針對使用者定義的還原點，**選取還原點**或**建立新的使用者定義還原點**。

    ![使用者定義的還原點](./media/sql-data-warehouse-restore-database-portal/restoring_2_udrp.png)

## <a name="restore-a-deleted-database-using-the-azure-portal"></a>使用 Azure 入口網站還原已刪除的資料庫
1. 登入 [Azure 入口網站][Azure portal]。
2. 瀏覽至裝載已刪除資料庫的 SQL 伺服器。
3. 在目錄中選取已刪除資料庫的圖示。

    ![已刪除的資料庫](./media/sql-data-warehouse-restore-database-portal/restoring_deleted_0.png)

4. 選取您想要還原的已刪除資料庫。

    ![選取已刪除的資料庫](./media/sql-data-warehouse-restore-database-portal/restoring_deleted_1.png)

5. 指定新的資料庫名稱。

    ![指定資料庫名稱](./media/sql-data-warehouse-restore-database-portal/restoring_deleted_2.png)

<!--Image references-->

<!--Article references-->
[Azure SQL Database business continuity overview]: ../sql-database/sql-database-business-continuity.md
[Request a DTU quota change]: ./sql-data-warehouse-get-started-create-support-ticket.md
[Configure your database after recovery]: ../sql-database/sql-database-disaster-recovery.md#configure-your-database-after-recovery
[How to install and configure Azure PowerShell]: /powershell/azureps-cmdlets-docs
[Overview]: ./sql-data-warehouse-restore-database-overview.md
[Portal]: ./sql-data-warehouse-restore-database-portal.md
[PowerShell]: ./sql-data-warehouse-restore-database-powershell.md
[REST]: ./sql-data-warehouse-restore-database-rest-api.md
[Configure your database after recovery]: ../sql-database/sql-database-disaster-recovery.md#configure-your-database-after-recovery

<!--MSDN references-->
[Restore-AzureRmSqlDatabase]: https://docs.microsoft.com/powershell/module/azurerm.sql/restore-azurermsqldatabase

<!--Other Web references-->
[Azure Portal]: https://portal.azure.com/
[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps
