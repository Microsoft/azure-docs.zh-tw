---
title: 在 Microsoft Azure PowerShell 中使用 Azure 資料庫移轉服務模組，將 SQL Server 內部部署移轉至 Azure SQL DB MI | Microsoft Docs
description: 了解如何使用 Azure PowerShell 從內部部署 SQL Server 移轉至 Azure SQL DB MI。
services: database-migration
author: HJToland3
ms.author: jtoland
manager: ''
ms.reviewer: ''
ms.service: database-migration
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 08/13/2018
ms.openlocfilehash: 7bd7e7a4cb78cf8a9f818936c980b47a2e7865e7
ms.sourcegitcommit: 0fcd6e1d03e1df505cf6cb9e6069dc674e1de0be
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "40099653"
---
# <a name="migrate-sql-server-on-premises-to-azure-sql-db-using-azure-powershell"></a>使用 Azure PowerShell 將 SQL Server 內部部署移轉至 Azure SQL DB
在本文中，您會使用 Microsoft Azure PowerShell，將已還原至內部部署 SQL Server 2005 (或更新版本) 執行個體的 **Adventureworks2012** 資料庫移轉至 Azure SQL Database。 您可以使用 Microsoft Azure PowerShell 中的 `AzureRM.DataMigration` 模組，將資料庫從內部部署 SQL Server 執行個體移轉至 Azure SQL Database。

在本文中，您將了解：
> [!div class="checklist"]
> * 建立資源群組。
> * 建立 Azure 資料庫移轉服務的執行個體。
> * 在 Azure 資料庫移轉服務執行個體中建立移轉專案。
> * 執行移轉。

## <a name="prerequisites"></a>必要條件
若要完成這些步驟，您需要：

- [SQL Server 2016 或更新版本](https://www.microsoft.com/sql-server/sql-server-downloads) (任何版本)
- 啟用 SQL Server Express 安裝預設已停用的 TCP/IP 通訊協定。 依照[啟用或停用伺服器網路通訊協定](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure)一文的說明，啟用 TCP/IP 通訊協定。
- 設定[用於 Database Engine 存取的 Windows 防火牆](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
- Azure SQL Database 執行個體。 您可以依照[在 Azure 入口網站中建立 Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal) 一文中的詳細資料來建立 Azure SQL Database 執行個體。
- [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) \(英文\) 的 v3.3 或更新版本。
- 使用 Azure Resource Manager 部署模型建立 VNET，讓 Azure 資料庫移轉服務能夠使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 對您的內部部署來源伺服器進行站對站連線。
- 使用資料移轉小幫手完成內部部署資料庫和結構描述移轉的評估，如同[執行 SQL Server 移轉評估](https://docs.microsoft.com/sql/dma/dma-assesssqlonprem)一文所說明
- 使用 [Install-Module PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/powershellget/Install-Module?view=powershell-5.1)，從 PowerShell 資源庫下載並安裝 AzureRM.DataMigration 模組
- 確定用來連線至來源 SQL Server 執行個體的認證具有 [CONTROL SERVER](https://docs.microsoft.com/sql/t-sql/statements/grant-server-permissions-transact-sql) 權限。
- 確定用來連線至目標 Azure SQL DB 執行個體的認證，在目標 Azure SQL Database 資料庫上具有 CONTROL DATABASE 權限。
- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費](https://azure.microsoft.com/free/)帳戶。

## <a name="log-in-to-your-microsoft-azure-subscription"></a>登入您的 Microsoft Azure 訂用帳戶
根據[登入 Azure PowerShell](https://docs.microsoft.com/powershell/azure/authenticate-azureps?view=azurermps-4.4.1)一文中的指示，使用 PowerShell 來登入您的 Azure 訂用帳戶。

## <a name="create-a-resource-group"></a>建立資源群組
Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 建立虛擬機器之前，先建立資源群組。

使用 [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroup?view=azurermps-4.4.1) 命令建立資源群組。 

下列範例會在 EastUS 地區建立名為 myResourceGroup 的資源群組。

```powershell
New-AzureRmResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```
## <a name="create-an-instance-of-the-azure-database-migration-service"></a>建立 Azure 資料庫移轉服務的執行個體 
您可以使用 `New-AzureRmDataMigrationService` Cmdlet，來建立新的 Azure 資料庫移轉服務執行個體。 此 Cmdlet 預期有下列必要參數：
- Azure 資源群組名稱。 您可以使用 [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroup?view=azurermps-4.4.1) 命令，來建立如先前所示的 Azure 資源群組，並將其名稱作為參數使用。
- 服務名稱。 該字串會對應至 Azure 資料庫移轉服務所需的唯一服務名稱 
- *位置*。 指定服務的位置。 指定 Azure 資料中心位置，例如美國西部或東南亞
- SKU。 此參數會對應至 DMS SKU 名稱。 目前支援的 SKU 名稱為 Basic_1vCore、Basic_2vCores、GeneralPurpose_4vCores
- 虛擬子網路識別碼。 您可以使用 [New-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig?view=azurermps-4.4.1) Cmdlet 來建立子網路。 

下列範例會使用名為 *MyVNET* 的虛擬網路和名為 *MySubnet* 的子網路，在 *MyDMSResourceGroup* 資源群組中建立名為 *MyDMS* 的服務 (位於美國東部地區)。

```powershell
 $vNet = Get-AzureRmVirtualNetwork -ResourceGroupName MyDMSResourceGroup -Name MyVNET

$vSubNet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vNet -Name MySubnet

$service = New-AzureRmDms -ResourceGroupName myResourceGroup `
  -ServiceName MyDMS `
  -Location EastUS `
  -Sku Basic_2vCores `  
  -VirtualSubnetId $vSubNet.Id`
```

## <a name="create-a-migration-project"></a>建立移轉專案
建立 Azure 資料庫移轉服務執行個體之後，建立移轉專案。 Azure 資料庫移轉服務專案需要來源和目標執行個體的連線資訊，以及您想要移轉的資料庫清單。

### <a name="create-a-database-connection-info-object-for-the-source-and-target-connections"></a>建立來源和目標連線的資料庫連接資訊物件
您可以使用 `New-AzureRmDmsConnInfo` Cmdlet 來建立資料庫連接資訊物件。 此 Cmdlet 預期有下列參數：
- ServerType。 要求的資料庫連接類型，例如 SQL、Oracle 或 MySQL。 使用 SQL Server 的 SQL 和 Azure SQL。
- DataSource。 SQL Server 執行個體或 Azure SQL 資料庫的名稱或 IP。
- AuthType。 連線的驗證類型，可以是 SqlAuthentication 或 WindowsAuthentication。
- TrustServerCertificate 參數會設定一個值，指出通道是否會加密，同時繞過驗證信任的信任鏈結。 值可以是 true 或 false。

下列範例會使用 SQL 驗證，為名為 MySourceSQLServer 的來源 SQL Server 建立連線資訊物件： 

```powershell
$sourceConnInfo = New-AzureRmDmsConnInfo -ServerType SQL `
  -DataSource MySourceSQLServer `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$true
```

下一個範例會說明如何使用 sql 驗證，為名為 ‘targetmanagedinstance.database.windows.net’ 的 Azure SQL Database 受控執行個體伺服器建立連線資訊：

```powershell
$targetConnInfo = New-AzureRmDmsConnInfo -ServerType SQL `
  -DataSource "targetmanagedinstance.database.windows.net" `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$false
```

### <a name="provide-databases-for-the-migration-project"></a>提供移轉專案所用的資料庫
建立 `AzureRmDataMigrationDatabaseInfo` 物件清單，該清單會指定資料庫作為 Azure 資料庫移轉專案的一部份，並提供給建立物件所需的參數使用。 `New-AzureRmDataMigrationDatabaseInfo` Cmdlet 可以用來建立 AzureRmDataMigrationDatabaseInfo。 

下列範例會為 **AdventureWorks** 資料庫建立 `AzureRmDataMigrationDatabaseInfo` 專案，並將其新增至清單 (提供給建立物件所需的參數使用)。

```powershell
$dbInfo1 = New-AzureRmDataMigrationDatabaseInfo -SourceDatabaseName AdventureWorks
$dbList = @($dbInfo1)
```

### <a name="create-a-project-object"></a>建立專案物件
最後，您可以使用 `New-AzureRmDataMigrationProject` 建立位在美國東部且名為 MyDMSProject 的 Azure 資料庫移轉專案，並新增先前建立的來源和目標連線及資料庫清單來進行移轉。

```powershell
$project = New-AzureRmDataMigrationProject -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName MyDMSProject `
  -Location EastUS `
  -SourceType SQL `
  -TargetType SQLMI `
  -SourceConnection $sourceConnInfo `
  -TargetConnection $targetConnInfo `
  -DatabaseInfo $dbList
```

## <a name="create-and-start-a-migration-task"></a>建立並啟動移轉工作
最後，建立並啟動 Azure 資料庫移轉工作。 除了專案在作為必要元件建立時提供的資訊外，Azure 資料庫移轉工作還需要來源和目標的連線認證資訊，以及要移轉的資料庫清單資料表。 

### <a name="create-credential-parameters-for-source-and-target"></a>建立來源和目標的認證參數
連線安全性認證可建立為 [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0) \(英文\) 物件。 

下列範例示範如何將密碼提供為 $sourcePassword 和 $targetPassword 字串變數，以建立來源和目標的 PSCredential 物件。 

```powershell
$secpasswd = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCred = New-Object System.Management.Automation.PSCredential ($sourceUserName, $secpasswd)
$secpasswd = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCred = New-Object System.Management.Automation.PSCredential ($targetUserName, $secpasswd)
```

### <a name="create-backup-fileshare-object"></a>建立備份 FileShare 物件
現在，請建立 FileShare 物件，用以代表 Azure 資料庫移轉服務可使用 New-AzureRmDmsFileShare Cmdlet 移入來源資料庫備份的本機 SMB 網路共用。

```powershell
$backupPassword = ConvertTo-SecureString -String $password -AsPlainText -Force
$backupCred = New-Object System.Management.Automation.PSCredential ($backupUserName, $backupPassword)

$backupFileSharePath="\\10.0.0.76\SharedBackup"
$backupFileShare = New-AzureRmDmsFileShare -Path $backupFileSharePath -Credential $backupCred
```

### <a name="create-selected-database-object"></a>建立選取的資料庫物件
下一個步驟是使用 New-AzureRmDmsSelectedDB Cmdlet 選取來源和目標資料庫，如下列範例所示：

```powershell
$selectedDbs = @()
$selectedDbs += New-AzureRmDmsSelectedDB -MigrateSqlServerSqlDbMi `
  -Name AdventureWorks2016 `
  -TargetDatabaseName AdventureWorks2016 `
  -BackupFileShare $backupFileShare `
```

### <a name="sas-uri-for-azure-storage-container"></a>Azure 儲存體容器的 SAS URI
建立包含 SAS URI 的變數，讓 Azure 資料庫管理服務能夠存取此服務將備份檔案上傳到的儲存體帳戶容器。

```powershell
$blobSasUri="https://mystorage.blob.core.windows.net/test?st=2018-07-13T18%3A10%3A33Z&se=2019-07-14T18%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=qKlSA512EVtest3xYjvUg139tYSDrasbftY%3D"
```

### <a name="select-logins"></a>選取登入
依照下列範例的說明建立要進行移轉的登入清單：請注意，DMS 目前僅支援移轉 SQL 登入。 

```powershell
$selectedLogins = @("user1", "user2")
```

### <a name="select-agent-jobs"></a>選取代理程式作業
依照下列範例的說明建立要移轉的代理程式作業清單：

>[!NOTE]
>DMS 目前僅支援以 T-SQL 子系統作業步驟來執行作業。

```powershell
$selectedAgentJobs = @("agentJob1", "agentJob2")
```

### <a name="create-and-start-a-migration-task"></a>建立並啟動移轉工作

使用 `New-AzureRmDataMigrationTask` Cmdlet 建立並啟動移轉工作。 此 Cmdlet 預期有下列參數：
- TaskType。 要為從 SQL Server 到 Azure SQL Database 受控執行個體的移轉建立的移轉工作類型，預期為 *MigrateSqlServerSqlDbMi*。 
- 資源群組名稱。 要在其中建立工作的資源群組名稱。
- ServiceName。 要在其中建立工作的 Azure 資料庫移轉服務執行個體。
- ProjectName。 要在其中建立工作的 Azure 資料庫移轉服務專案名稱。 
- TaskName。 要建立的工作名稱。 
- *SourceConnection*。 代表來源 SQL Server 連線的 AzureRmDmsConnInfo 物件。
- *TargetConnection*。 代表目標 Azure SQL Database 受控執行個體連線的 AzureRmDmsConnInfo 物件。
- SourceCred。 連線至來源伺服器所用的 [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0) 物件。
- TargetCred。 連線至目標伺服器所用的 [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0) 物件。
- SelectedDatabase。 代表來源和目標資料庫對應的 AzureRmDataMigrationSelectedDB 物件。
- *BackupFileShare*。 此為 FileShare 物件，代表 Azure 資料庫移轉服務可移入來源資料庫備份的本機網路共用。
- *BackupBlobSasUri*。 此為 SAS URI，可讓 Azure 資料庫移轉服務存取此服務會將備份檔案上傳到的儲存體帳戶容器。 了解如何取得 Blob 容器的 SAS URI。
- *SelectedLogins*。 已選取要移轉的登入清單。
- *SelectedAgentJobs*。 已選取要移轉的代理程式作業清單。

下列範例會建立並啟動名為 myDMSTask 的移轉工作：

```powershell
$migTask = New-AzureRmDataMigrationTask -TaskType MigrateSqlServerSqlDbMi `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs`
  -BackupFileShare $backupFileShare `
  -BackupBlobSasUri $blobSasUri `
  -SelectedLogins $selectedLogins `
  -SelectedAgentJobs $selectedJobs `
```

## <a name="monitor-the-migration"></a>監視移轉
您可以透過查詢工作的狀態屬性來監視移轉工作執行，如下列範例所示：

```powershell
if (($mytask.ProjectTask.Properties.State -eq "Running") -or ($mytask.ProjectTask.Properties.State -eq "Queued"))
{
  write-host "migration task running"
}
```

## <a name="next-steps"></a>後續步驟
- 在 Microsoft [資料庫移轉指南](https://datamigration.microsoft.com/) \(英文\) 中檢閱移轉指引。