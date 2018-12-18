---
title: 將 Azure SQL 資料庫匯出到 BACPAC 檔案 | Microsoft Docs
description: 使用 Azure 入口網站將 Azure SQL Database 匯出到 BACPAC 檔案
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: a867a57e54efae6d735a9918a637fa3b4f9e37bc
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47160283"
---
# <a name="export-an-azure-sql-database-to-a-bacpac-file"></a>將 Azure SQL 資料庫匯出到 BACPAC 檔案

當您需要匯出資料庫以封存或移至另一個平台時，可以將資料庫結構描述和資料匯出至 [BACPAC](https://msdn.microsoft.com/library/ee210546.aspx#Anchor_4) 檔案。 BACPAC 檔案是一種副檔名為 BACPAC 的 ZIP 檔案，其包含來自 SQL Server Database 的中繼資料和資料。 BACPAC 檔案可以儲存在 Azure Blob 儲存體，或在內部部署位置的本機儲存體中，之後再匯入至 Azure SQL Database 或 SQL Server 內部部署安裝。 

> [!IMPORTANT] 
> Azure SQL Database 自動匯出已在 2017 年 3 月 1 日淘汰。 您可以使用[長期的備份保留](sql-database-long-term-retention.md
)或 [Azure 自動化](https://github.com/Microsoft/azure-docs/blob/2461f706f8fc1150e69312098640c0676206a531/articles/automation/automation-intro.md)，根據您選擇的排程使用 PowerShell 定期封存 SQL Database。 如需範例，請從 Github 下載[範例 PowerShell 指令碼](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-automation-automated-export)。
>

## <a name="considerations-when-exporting-an-azure-sql-database"></a>匯出 Azure SQL Database 時的考量

* 為了讓匯出處於交易一致狀態，您必須確定在匯出期間未發生任何寫入活動，或者是從 Azure SQL 資料庫的[交易一致性複本](sql-database-copy.md)匯出。
* 如果您要匯出至 blob 儲存體，BACPAC 檔案的大小上限為 200 GB。 若要封存較大的 BACPAC 檔案，請將匯出到本機儲存體。
* 不支援使用本文所討論的方法將 BACPAC 檔案匯出到 Azure 進階儲存體。
* 如果執行從 Azure SQL Database 匯出的作業超過 20 個小時，它可能會被取消。 若要增加匯出期間的效能，您可以︰
  * 暫時提高計算大小。
  * 在匯出期間停止所有讀取及寫入活動。
  * 在所有大型資料表上搭配使用 [叢集索引](https://msdn.microsoft.com/library/ms190457.aspx) 和非 null 值。 若沒有叢集索引，如果要花超過 6-12 小時，匯出可能會失敗。 這是因為匯出服務需要完成資料表掃描，以便嘗試匯出整份資料表。 有一個可判斷資料表是否已針對匯出進行最佳化的好方法，就是執行 **DBCC SHOW_STATISTICS**，並確定 *RANGE_HI_KEY* 不是 null 且其值具有良好的分佈。 如需詳細資料，請參閱 [DBCC SHOW_STATISTICS](https://msdn.microsoft.com/library/ms174384.aspx)。

> [!NOTE]
> BACPAC 並非用於備份和還原作業。 Azure SQL Database 會自動為每個使用者資料庫建立備份。 如需詳細資訊，請參閱[商務持續性概觀](sql-database-business-continuity.md)和 [SQL Database 備份](sql-database-automated-backups.md)。  
> 

## <a name="export-to-a-bacpac-file-using-the-azure-portal"></a>使用 Azure 入口網站匯出到 BACPAC 檔案

若要使用 [Azure 入口網站](https://portal.azure.com)匯出資料庫，請開啟資料庫頁面，然後按一下工具列上的 [匯出]。 指定 BACPAC 檔案名稱，並提供匯出的 Azure 儲存體帳戶和容器，然後提供認證以連線至來源資料庫。  

![資料庫匯出](./media/sql-database-export/database-export.png)

若要監視匯出作業的進度，請開啟包含匯出資料庫的邏輯伺服器頁面。 向下捲動至**作業**，然後按一下 [匯入/匯出歷程記錄] 。

![匯出記錄](./media/sql-database-export/export-history.png)
![匯出記錄狀態](./media/sql-database-export/export-history2.png)

## <a name="export-to-a-bacpac-file-using-the-sqlpackage-utility"></a>使用 SQLPackage 公用程式匯出到 BACPAC 檔案

若要使用 [SqlPackage](https://msdn.microsoft.com/library/hh550080.aspx) 命令列公用程式匯出 SQL 資料庫，請參閱[匯出參數和屬性](https://msdn.microsoft.com/library/hh550080.aspx#Export Parameters and Properties)。 SQLPackage 公用程式隨附於最新版的 [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 和 [SQL Server Data Tools for Visual Studio](https://msdn.microsoft.com/library/mt204009.aspx)，或者您也可以直接從 Microsoft 下載中心下載最新版的 [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876)。

針對大部分生產環境中的延展性和效能，我們建議您使用 SQLPackage 公用程式。 如需 SQL Server 客戶諮詢小組部落格中有關使用 BACPAC 檔案進行移轉的主題，請參閱[使用 BACPAC 檔案從 SQL Server 移轉至 Azure SQL Database](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)。

此範例會說明如何透過 Active Directory 通用驗證使用 SqlPackage.exe 匯出資料庫：

```cmd
SqlPackage.exe /a:Export /tf:testExport.bacpac /scs:"Data Source=apptestserver.database.windows.net;Initial Catalog=MyDB;" /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="export-to-a-bacpac-file-using-sql-server-management-studio-ssms"></a>使用 SQL Server Management Studio (SSMS) 匯出到 BACPAC 檔案

最新版的 SQL Server Management Studio 也提供精靈協助您將 Azure SQL Database 匯出到 BACPAC 檔案。 請參閱[匯出資料層應用程式](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/export-a-data-tier-application)。

## <a name="export-to-a-bacpac-file-using-powershell"></a>使用 PowerShell 匯出到 BACPAC 檔案

使用 [New-AzureRmSqlDatabaseExport](/powershell/module/azurerm.sql/new-azurermsqldatabaseexport) Cmdlet 來提交匯出資料庫要求至 Azure SQL Database 服務。 視資料庫大小而定，匯出作業可能需要一些時間才能完成。

 ```powershell
 $exportRequest = New-AzureRmSqlDatabaseExport -ResourceGroupName $ResourceGroupName -ServerName $ServerName `
   -DatabaseName $DatabaseName -StorageKeytype $StorageKeytype -StorageKey $StorageKey -StorageUri $BacpacUri `
   -AdministratorLogin $creds.UserName -AdministratorLoginPassword $creds.Password
 ```

若要查看匯出要求的狀態，請使用 [Get AzureRmSqlDatabaseImportExportStatus](/powershell/module/azurerm.sql/get-azurermsqldatabaseimportexportstatus)Cmdlet。 如果在要求後立即執行此 Cmdlet，通常會傳回 **Status : InProgress**。 當您看見 **Status: Succeeded** 時，便代表匯出已完成。

```powershell
$exportStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
[Console]::Write("Exporting")
while ($exportStatus.Status -eq "InProgress")
{
    Start-Sleep -s 10
    $exportStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
    [Console]::Write(".")   
}
[Console]::WriteLine("")
$exportStatus
```

## <a name="next-steps"></a>後續步驟

* 若要了解除了匯出的資料庫，還有 Azure SQL Database 備份的長期備份保留也可達到封存目的，請參閱[長期備份保留](sql-database-long-term-retention.md)。
- 如需 SQL Server 客戶諮詢小組部落格中有關使用 BACPAC 檔案進行移轉的主題，請參閱[使用 BACPAC 檔案從 SQL Server 移轉至 Azure SQL Database](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)。
* 若要了解如何將 BACPAC 匯入 SQL Server 資料庫，請參閱 [將 BACPAC 匯入 SQL Server 資料庫](https://msdn.microsoft.com/library/hh710052.aspx)。
* 若要了解如何將 BACPAC 從 SQL Server 資料庫匯出，請參閱[匯出資料層應用程式](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/export-a-data-tier-application)和[移轉您的第一個資料庫](sql-database-migrate-your-sql-server-database.md)。
* 如果您要從 SQL Server 匯出以準備移轉至 Azure SQL Database，請參閱[將 SQL Server 資料庫移轉至 Azure SQL Database](sql-database-cloud-migrate.md)。
* 若要了解如何管理和共用儲存體金鑰，以及安全地共用存取簽章，請參閱 [Azure 儲存體安全性指南](https://docs.microsoft.com/azure/storage/common/storage-security-guide)。
