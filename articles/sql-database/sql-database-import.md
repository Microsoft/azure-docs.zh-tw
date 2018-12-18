---
title: 匯入 BACPAC 檔案以建立 Azure SQL Database | Microsoft Docs
description: 匯入 BACPAC 檔案以建立新的 Azure SQL Database。
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
ms.openlocfilehash: 9de7fe9972f1ae0fca1c4e527f718b31fddf4294
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161513"
---
# <a name="import-a-bacpac-file-to-a-new-azure-sql-database"></a>將 BACPAC 檔案匯入到新的 Azure SQL Database

當您需要從封存匯入資料庫或從另一個平台進行移轉時，您可以從 [BACPAC](https://msdn.microsoft.com/library/ee210546.aspx#Anchor_4) 檔案匯入資料庫結構描述和資料。 BACPAC 檔案是一種副檔名為 BACPAC 的 ZIP 檔案，其包含來自 SQL Server Database 的中繼資料和資料。 BACPAC 檔案可以從 Azure Blob 儲存體 (僅限標準儲存體) 匯入，或從內部部署位置中的本機儲存體匯入。 若要使匯入速度最大化，我們建議您指定較高的服務層級與計算大小 (例如 P6)，然後在匯入成功後，適當地向下調整。 此外，匯入之後的資料庫相容性等級會以來源資料庫的相容性等級為基礎。 

> [!IMPORTANT] 
> 您將資料庫移轉至 Azure SQL Database 後，可選擇於目前的相容性等級 (針對 AdventureWorks2008R2 資料庫為等級 100) 或更高等級運作資料庫。 如需於特定相容性層級操作資料庫的含意與選項詳細資訊，請參閱 [ALTER DATABASE 相容性層級](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level)。 如需相容性層級其他相關資料庫等級設定的資訊，另請參閱 [ALTER DATABASE 範圍組態 (ALTER DATABASE SCOPED CONFIGURATION)](https://docs.microsoft.com/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql)。   >

## <a name="import-from-a-bacpac-file-using-azure-portal"></a>使用 Azure 入口網站從 BACPAC 檔案匯入

本文提供的指示將說明如何使用 [Azure 入口網站](https://portal.azure.com)，從 BACPAC 檔案 (儲存於 Azure Blob 儲存體中) 建立 Azure SQL Database。 使用 Azure 入口網站匯入的方式只支援從 Azure Blob 儲存體匯入 BACPAC 檔案。

若要使用 Azure 入口網站匯入資料庫，請開啟伺服器頁面 (而非資料庫頁面) 來建立對資料庫的關聯，然後按一下工具列上的 [匯入]。 指定儲存體帳戶和容器，然後選取您要匯入的 BACPAC 檔案。 選取新資料庫的大小 (通常與來源相同)，並提供目的地 SQL Server 認證。  

   ![資料庫匯入](./media/sql-database-import/import.png)

若要監視匯入作業的進度，請開啟包含匯入資料庫的邏輯伺服器頁面。 向下捲動至**作業**，然後按一下 [匯入/匯出歷程記錄] 。

> [!NOTE]
> [Azure SQL Database 受控執行個體](sql-database-managed-instance.md)已支援使用本文中的其他方法從 BACPAC 檔案匯入，但目前不支援使用 Azure 入口網站移轉 。

### <a name="monitor-the-progress-of-an-import-operation"></a>監視匯入作業的進度

若要監視匯入作業的進度，請將邏輯伺服器的頁面開啟為要匯入的資料庫。 向下捲動至 [作業]，然後按一下 [匯入/匯出歷程記錄]。
   
   ![匯入](./media/sql-database-import/import-history.png) ![匯入狀態](./media/sql-database-import/import-status.png)

確認伺服器上的資料庫為線上狀態，請按一下 [SQL 資料庫]，並確認新的資料庫為 [線上]。

## <a name="import-from-a-bacpac-file-using-sqlpackage"></a>使用 SQLPackage 從 BACPAC 檔案匯入

若要使用 [SqlPackage](https://msdn.microsoft.com/library/hh550080.aspx) 命令列公用程式匯入 SQL Database，請參閱[匯入參數和屬性](https://msdn.microsoft.com/library/hh550080.aspx#Import Parameters and Properties)。 SQLPackage 公用程式隨附於最新版的 [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 和 [SQL Server Data Tools for Visual Studio](https://msdn.microsoft.com/library/mt204009.aspx)，或者您也可以直接從 Microsoft 下載中心下載最新版的 [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876)。

針對大部分生產環境中的延展性和效能，我們建議您使用 SQLPackage 公用程式。 如需 SQL Server 客戶諮詢小組部落格中有關使用 BACPAC 檔案進行移轉的主題，請參閱[使用 BACPAC 檔案從 SQL Server 移轉至 Azure SQL Database](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)。

請參閱下列 SQLPackage 命令，以取得如何將 **AdventureWorks2008R2** 資料庫從本機儲存體匯入到 Azure SQL Database 邏輯伺服器 (在此範例中稱為 **mynewserver20170403**) 的指令碼範例。 此指令碼會示範如何建立名為 **myMigratedDatabase** 的新資料庫，並搭配 **Premium (進階)** 服務層和 **P6** 服務目標。 將這些值變更為適合您環境的值。

```cmd
SqlPackage.exe /a:import /tcs:"Data Source=mynewserver20170403.database.windows.net;Initial Catalog=myMigratedDatabase;User Id=ServerAdmin;Password=<change_to_your_password>" /sf:AdventureWorks2008R2.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> Azure SQL Database 邏輯伺服器會接聽連接埠 1433。 如果您嘗試從公司防火牆連線至 Azure SQL Database 邏輯伺服器，則必須在公司防火牆中開啟此連接埠，您才能成功連線。
>

此範例會說明如何透過 Active Directory 通用驗證使用 SqlPackage.exe 匯入資料庫：

```cmd
SqlPackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="import-from-a-bacpac-file-using-powershell"></a>使用 PowerShell 從 BACPAC 檔案匯入

使用 [New-AzureRmSqlDatabaseImport](/powershell/module/azurerm.sql/new-azurermsqldatabaseimport) Cmdlet 來提交匯入資料庫要求至 Azure SQL Database 服務。 視資料庫大小而定，匯入作業可能需要一些時間才能完成。

 ```powershell
 $importRequest = New-AzureRmSqlDatabaseImport -ResourceGroupName "myResourceGroup" `
    -ServerName $servername `
    -DatabaseName "MyImportSample" `
    -DatabaseMaxSizeBytes "262144000" `
    -StorageKeyType "StorageAccessKey" `
    -StorageKey $(Get-AzureRmStorageAccountKey -ResourceGroupName "myResourceGroup" -StorageAccountName $storageaccountname).Value[0] `
    -StorageUri "http://$storageaccountname.blob.core.windows.net/importsample/sample.bacpac" `
    -Edition "Standard" `
    -ServiceObjectiveName "P6" `
    -AdministratorLogin "ServerAdmin" `
    -AdministratorLoginPassword $(ConvertTo-SecureString -String "ASecureP@assw0rd" -AsPlainText -Force)

 ```

若要查看匯入要求的狀態，請使用 [Get-AzureRmSqlDatabaseImportExportStatus](/powershell/module/azurerm.sql/get-azurermsqldatabaseimportexportstatus) Cmdlet。 如果在要求後立即執行此 Cmdlet，通常會傳回 **Status : InProgress**。 當您看見 **Status: Succeeded** 時，便代表匯入已完成。

```powershell
$importStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
[Console]::Write("Importing")
while ($importStatus.Status -eq "InProgress")
{
    $importStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
    [Console]::Write(".")
    Start-Sleep -s 10
}
[Console]::WriteLine("")
$importStatus
```

> [!TIP]
如需其他指令碼範例，請參閱[從 BACPAC 檔案匯入資料庫](scripts/sql-database-import-from-bacpac-powershell.md)。

## <a name="limitations"></a>限制
- 不支援將資料庫匯入彈性集區。 您可以將資料匯入到單一資料庫，然後將資料庫移到集區。

## <a name="import-using-other-methods"></a>使用其他方法匯入

您也可以使用這些精靈：

- [SQL Server Management Studio 中的匯入資料層應用程式精靈](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database#using-the-import-data-tier-application-wizard)。
- [SQL Server 匯入和匯出精靈](https://docs.microsoft.com/sql/integration-services/import-export-data/start-the-sql-server-import-and-export-wizard)。

## <a name="next-steps"></a>後續步驟
* 若要了解如何連線並查詢匯入的 SQL Database，請參閱[使用 SQL Server Management Studio 連接到 SQL Database 並執行範例 T-SQL 查詢](sql-database-connect-query-ssms.md)。
* 如需 SQL Server 客戶諮詢小組部落格中有關使用 BACPAC 檔案進行移轉的主題，請參閱[使用 BACPAC 檔案從 SQL Server 移轉至 Azure SQL Database](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)。
* 如需有關整個 SQL Server 資料庫移轉程序的討論，包括效能建議，請參閱[將 SQL Server 資料庫移轉至 Azure SQL Database](sql-database-cloud-migrate.md)。
* 若要了解如何管理和共用儲存體金鑰，以及安全地共用存取簽章，請參閱 [Azure 儲存體安全性指南](https://docs.microsoft.com/azure/storage/common/storage-security-guide)。 


  

