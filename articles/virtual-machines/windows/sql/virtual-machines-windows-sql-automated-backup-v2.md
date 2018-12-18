---
title: SQL Server 2016/2017 Azure VM 的自動備份 v2 | Microsoft Docs
description: 說明在 Azure 中執行之 SQL Server 2016/2017 VM 的「自動備份」功能。 本文是專門針對使用 Resource Manager 的 VM。
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: craigg
tags: azure-resource-manager
ms.assetid: ebd23868-821c-475b-b867-06d4a2e310c7
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/03/2018
ms.author: jroth
ms.openlocfilehash: 4619c26e34c90f58702ad286f76a999f83f49cc4
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33894504"
---
# <a name="automated-backup-v2-for-azure-virtual-machines-resource-manager"></a>Azure 虛擬機器的自動備份 v2 (Resource Manager)

> [!div class="op_single_selector"]
> * [SQL Server 2014](virtual-machines-windows-sql-automated-backup.md)
> * [SQL Server 2016/2017](virtual-machines-windows-sql-automated-backup-v2.md)

「自動備份 v2」會針對執行 SQL Server 2016/2017 Standard、Enterprise 或 Developer 版本之 Azure VM 上所有現有和新的資料庫，自動設定[受控備份至 Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx)。 這可讓您設定採用持久性 Azure Blob 儲存體的一般資料庫備份。 「自動備份 v2」依存於 [SQL Server IaaS 代理程式擴充功能](virtual-machines-windows-sql-server-agent-extension.md)。

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>先決條件
若要使用「自動備份 v2」，請檢閱下列必要條件：

**作業系統**：

- Windows Server 2012 R2
- Windows Server 2016

**SQL Server 版本**：

- SQL Server 2016：Developer、Standard 或 Enterprise
- SQL Server 2017：Developer、Standard 或 Enterprise

> [!IMPORTANT]
> 「自動備份 v2」可與 SQL Server 2016 或更新版本搭配運作。 如果您使用的是 SQL Server 2014，則可以使用「自動備份 v1」來備份您的資料庫。 如需詳細資訊，請參閱 [SQL Server 2014 Azure 虛擬機器的自動備份](virtual-machines-windows-sql-automated-backup.md)。

**資料庫組態**：

- 目標資料庫必須使用完整復原模型。 如需有關完整復原模型對備份之影響的詳細資訊，請參閱[在完整復原模式下備份](https://technet.microsoft.com/library/ms190217.aspx)。
- 系統資料庫不一定要使用完整復原模型。 不過，如果您要求針對 Model 或 MSDB 進行記錄備份，就必須使用完整復原模型。
- 目標資料庫必須位於預設的 SQL Server 執行個體上。 「SQL Server IaaS 擴充功能」並不支援具名執行個體。

> [!NOTE]
> 「自動備份」依存於「SQL Server IaaS 代理程式擴充功能」。 目前的 SQL 虛擬機器資源庫映像預設會新增這項擴充。 如需詳細資訊，請參閱 [SQL Server IaaS 代理程式擴充](virtual-machines-windows-sql-server-agent-extension.md)。

## <a name="settings"></a>設定
下表說明可以為「自動備份 v2」設定的選項。 實際的設定步驟會依據您是使用 Azure 入口網站或 Azure Windows PowerShell 命令而有所不同。

### <a name="basic-settings"></a>基本設定

| 設定 | 範圍 (預設值) | 說明 |
| --- | --- | --- |
| **自動備份** | 啟用/停用 (已停用) | 針對執行 SQL Server 2016/2017 Developer、Standard 或 Enterprise 的 Azure VM，啟用或停用自動備份。 |
| **保留期限** | 1-30 天 (30 天) | 保留備份的天數。 |
| **儲存體帳戶** | Azure 儲存體帳戶 | 將自動備份檔案儲存在 Blob 儲存體中時，所使用的 Azure 儲存體帳戶。 這個位置會建立一個容器來儲存所有備份檔案。 備份檔案命名慣例包括日期、時間和資料庫 GUID。 |
| **加密** |啟用/停用 (已停用) | 啟用或停用加密。 啟用加密時，用來還原備份的憑證會放在指定的儲存體帳戶中。 它會使用相同的 **automaticbackup** 容器與相同的命名慣例。 如果密碼變更，就會以該密碼產生新的憑證，但是舊的憑證還是會保留，以還原先前的備份。 |
| **密碼** |密碼文字 | 加密金鑰的密碼。 唯有啟用加密時，才需要此密碼。 若要還原加密的備份，您必須要有建立備份時所使用的正確密碼和相關憑證。 |

### <a name="advanced-settings"></a>進階設定

| 設定 | 範圍 (預設值) | 說明 |
| --- | --- | --- |
| **系統資料庫備份** | 啟用/停用 (已停用) | 當啟用時，此功能會一併備份系統資料庫：Master、MSDB 及 Model。 針對 MSDB 和 Model 資料庫，如果您想要進行記錄備份，請確認它們處於完整復原模式。 針對 Master 是一律不進行記錄備份。 而針對 TempDB 則不進行任何備份。 |
| **備份排程** | 手動/自動化 (自動化) | 預設會根據記錄的成長情況自動決定備份排程。 手動備份排程可讓使用者指定備份的時間範圍。 在此情況下，只會以指定的頻率且在特定一天的指定時間範圍期間進行備份。 |
| **完整備份頻率** | 每天/每週 | 完整備份的頻率。 在這兩種情況下，完整備份都會在下一個排定的時間範圍期間開始進行。 選取 [每週] 時，備份可以跨多天，直到所有資料庫都已順利備份為止。 |
| **完整備份開始時間** | 00:00 – 23:00 (01:00) | 可進行完整備份的特定一天開始時間期間。 |
| **完整備份時間範圍** | 1 – 23 小時 (1 小時) | 可進行完整備份的特定一天時間範圍持續時間。 |
| **記錄備份頻率** | 5 – 60 分鐘 (60 分鐘) | 記錄備份的頻率。 |

## <a name="understanding-full-backup-frequency"></a>了解完整備份頻率
了解每天與每週備份之間的差異相當重要。 請考慮下列兩個案例範例。

### <a name="scenario-1-weekly-backups"></a>案例 1：每週備份
您有一部包含一些大型資料庫的 SQL Server VM。

在星期一，您以下列設定啟用「自動備份 v2」：

- 備份排程：**自動**
- 完整備份頻率：**每週**
- 完整備份開始時間：**01:00**
- 完整備份時間範圍：**1 小時**

這意謂著下一個可用的備份時間範圍是星期二上午 1 點，持續時間為 1 小時。 到該時間，「自動備份」會開始逐一備份您的資料庫。 在此案例中，由於您的資料庫相當大，因此會完成前幾個資料庫的完整備份。 不過，在一小時之後，並非所有資料庫都已完成備份。

如果發生這種情況，「自動備份」就會在下一天 (星期三) 的上午 1 點備份剩餘的資料庫，持續時間為一小時。 如果在該時間未完成所有資料庫的備份，就會在下一天的相同時間再次嘗試。 這會持續直到順利備份所有資料庫為止。

再次到了星期二後，「自動備份」就會重新開始將所有資料庫備份一遍。

此案例說明「自動備份」只會在指定的時間範圍內運作，且每個資料庫每週會備份一次。 這也說明在無法一天內完成所有備份的情況下，可以跨多天執行備份。

### <a name="scenario-2-daily-backups"></a>案例 2：每天備份
您有一部包含一些大型資料庫的 SQL Server VM。

在星期一，您以下列設定啟用「自動備份 v2」：

- 備份排程：手動
- 完整備份頻率：每天
- 完整備份開始時間：22:00
- 完整備份時間範圍：6 小時

這意謂著下一個可用的備份時間範圍是星期一下午 10 點，持續時間為 6 小時。 到該時間，「自動備份」會開始逐一備份您的資料庫。

然後，在星期二下午 10 點，會重新開始進行所有資料庫的完整備份，持續時間為 6 小時。

> [!IMPORTANT]
> 排定每天備份時，建議您排定較大的時間範圍，以確保可在此時間內完成所有資料庫的備份。 在您有大量資料需要備份的情況下，這點尤其重要。

## <a name="configure-in-the-portal"></a>在入口網站中設定

您可以在佈建期間或針對現有的 SQL Server 2016/2017 VM，使用 Azure 入口網站來設定「自動備份 v2」。

## <a name="configure-for-new-vms"></a>設定新的 VM

以 Resource Manager 部署模型建立新的「SQL Server 2016 或 2017 虛擬機器」時，請使用 Azure 入口網站來設定「自動備份 v2」。

在 [SQL Server 設定] 窗格中，選取 [自動備份]。 下列的 Azure 入口網站螢幕擷取畫面顯示 [SQL 自動備份] 設定。

![在 Azure 入口網站中設定 SQL 自動備份](./media/virtual-machines-windows-sql-automated-backup-v2/automated-backup-blade.png)

> [!NOTE]
> 預設會停用「自動備份 v2」。

## <a name="configure-existing-vms"></a>設定現有的虛擬機器

如果是現有的 SQL Server 虛擬機器，請選取您的 SQL Server 虛擬機器。 然後選取虛擬機器 [設定] 的 [SQL Server 組態] 區段。

![現有 VM 的 SQL 自動備份](./media/virtual-machines-windows-sql-automated-backup-v2/sql-server-configuration.png)

在 [SQL Server 組態] 設定中，按一下 [自動備份] 區段中的 [編輯] 按鈕。

![設定現有 VM 的 SQL 自動備份](./media/virtual-machines-windows-sql-automated-backup-v2/sql-server-configuration-edit.png)

完成時，按一下 [SQL Server 組態] 設定底部的 [確定] 按鈕，以儲存您的變更。

如果這是您第一次啟用「自動備份」，Azure 就會在背景中設定 SQL Server IaaS Agent。 在此期間，Azure 入口網站可能不會顯示已設定自動備份。 請等候幾分鐘的時間來安裝及設定代理程式。 之後，Azure 入口網站將會反映新的設定。

## <a name="configure-with-powershell"></a>以 PowerShell 設定

您可以使用 PowerShell 來設定「自動備份 v2」。 開始進行之前，您必須：

- [下載並安裝最新的 Azure PowerShell](http://aka.ms/webpi-azps)。
- 開啟 Windows PowerShell，並使用 **Connect-AzureRmAccount** 命令，將其與您的帳戶建立關聯。

### <a name="install-the-sql-iaas-extension"></a>安裝 SQL IaaS 擴充功能
如果您是從 Azure 入口網站佈建 SQL Server 虛擬機器，則「SQL Server IaaS 擴充功能」應該已經安裝妥當。 您可以透過呼叫 **Get-AzureRmVM** 命令並檢查 **Extensions** 屬性，來判斷是否已為您的 VM 安裝該擴充功能。

```powershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"

(Get-AzureRmVM -Name $vmname -ResourceGroupName $resourcegroupname).Extensions 
```

如果已安裝「SQL Server IaaS 代理程式」擴充功能，您應該會看到它以 “SqlIaaSAgent” 或 “SQLIaaSExtension” 的形式列出。 該擴充功能的 **ProvisioningState** 應該也顯示為 “Succeeded”。 

如果未安裝或無法佈建該擴充功能，您可以使用下列命令來安裝它。 除了 VM 名稱和資源群組之外，您還必須指定 VM 所在的區域 (**$region**)。

```powershell
$region = “EASTUS2”
Set-AzureRmVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "1.2" -Location $region 
```

### <a id="verifysettings"></a> 確認目前的設定
如果您已在佈建期間啟用自動備份，您可以使用 PowerShell 來檢查目前的組態。 請執行 **Get-AzureRmVMSqlServerExtension** 命令並檢查 **AutoBackupSettings** 屬性：

```powershell
(Get-AzureRmVMSqlServerExtension -VMName $vmname -ResourceGroupName $resourcegroupname).AutoBackupSettings
```

您應該會看到類似以下的輸出：

```
Enable                      : True
EnableEncryption            : False
RetentionPeriod             : 30
StorageUrl                  : https://test.blob.core.windows.net/
StorageAccessKey            :  
Password                    : 
BackupSystemDbs             : False
BackupScheduleType          : Manual
FullBackupFrequency         : WEEKLY
FullBackupStartTime         : 2
FullBackupWindowHours       : 2
LogBackupFrequency          : 60
```

如果您的輸出顯示 **Enable** 是設定為 **False**，您就必須啟用自動備份。 好消息是，啟用和設定「自動備份」的方式是相同的。 如需這項資訊，請參閱下一節。

> [!NOTE] 
> 如果您在進行變更後立即檢查設定，可能會得到舊的組態值。 只要等幾分鐘後再重新檢查設定，即可確定已套用您的變更。

### <a name="configure-automated-backup-v2"></a>設定自動備份 v2
您可以使用 PowerShell 來啟用「自動備份」，也可以隨時修改其組態和行為。 

首先，為備份檔案選取或建立儲存體帳戶。 以下指令碼會選取或建立儲存體帳戶 (如果不存在)。

```powershell
$storage_accountname = “yourstorageaccount”
$storage_resourcegroupname = $resourcegroupname

$storage = Get-AzureRmStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzureRmStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region } 
```

> [!NOTE]
> 「自動備份」不支援將備份存放在進階儲存體中，但是可以從使用「進階儲存體」的 VM 磁碟進行備份。

接著，使用 **New-AzureRmVMSqlServerAutoBackupConfig** 命令來啟用及設定「自動備份 v2」設定，以將備份存放在 Azure 儲存體帳戶中。 在此範例中，是將備份設定為保留 10 天。 系統資料庫備份已啟用。 完整備份是排定為每週執行，時間範圍是從 20:00 開始並持續 2 小時。 記錄備份是排定為每 30 分鐘執行一次。 第二個命令 **Set-AzureRmVMSqlServerExtension** 會使用這些設定來更新指定的 Azure VM。

```powershell
$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname 
```

可能需要幾分鐘的時間來安裝及設定 SQL Server IaaS 代理程式。 

若要啟用加密，請修改先前的指令碼，為 **CertificatePassword** 參數傳遞 **EnableEncryption** 參數和密碼 (安全字串)。 下列指令碼會啟用上述範例中的自動備份設定，並新增加密。

```powershell
$password = "P@ssw0rd"
$encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force  

$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -Enable `
    -EnableEncryption -CertificatePassword $encryptionpassword `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

若要確認是否已套用您的設定，請[確認自動備份組態](#verifysettings)。

### <a name="disable-automated-backup"></a>停用自動備份
若要停用「自動備份」，請執行相同的指令碼，但不要對 **New-AzureRmVMSqlServerAutoBackupConfig** 命令使用 **-Enable** 參數。 沒有 **-Enable** 參數時，即表示通知命令停用此功能。 和安裝一樣，可能需要幾分鐘的時間來停用自動備份。

```powershell
$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -ResourceGroupName $storage_resourcegroupname

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

### <a name="example-script"></a>範例指令碼
以下指令碼提供一組變數，您可以自訂這些變數來為您的 VM 啟用及設定「自動備份」。 在您的案例中，您可能需要根據您的需求自訂此指令碼。 例如，如果您想要停用系統資料庫備份或啟用加密，您就必須進行變更。

```powershell
$vmname = "yourvmname"
$resourcegroupname = "vmresourcegroupname"
$region = “Azure region name such as EASTUS2”
$storage_accountname = “storageaccountname”
$storage_resourcegroupname = $resourcegroupname
$retentionperiod = 10
$backupscheduletype = "Manual"
$fullbackupfrequency = "Weekly"
$fullbackupstarthour = "20"
$fullbackupwindow = "2"
$logbackupfrequency = "30"

# ResourceGroupName is the resource group which is hosting the VM where you are deploying the SQL IaaS Extension 

Set-AzureRmVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "1.2" -Location $region

# Creates/use a storage account to store the backups

$storage = Get-AzureRmStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzureRmStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }

# Configure Automated Backup settings

$autobackupconfig = New-AzureRmVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays $retentionperiod -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType $backupscheduletype -FullBackupFrequency $fullbackupfrequency `
    -FullBackupStartHour $fullbackupstarthour -FullBackupWindowInHours $fullbackupwindow `
    -LogBackupFrequencyInMinutes $logbackupfrequency

# Apply the Automated Backup settings to the VM

Set-AzureRmVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

## <a name="monitoring"></a>監視

若要監控 SQL Server 2016/2017 上的自動備份，您有兩個主要選項。 因為自動備份使用 SQL Server 受控備份功能，所以相同的監控技術適用於兩者。

首先，您可以藉由呼叫 [msdb.managed_backup.sp_get_backup_diagnostics](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/managed-backup-sp-get-backup-diagnostics-transact-sql) 來輪詢狀態。 或查詢 [msdb.managed_backup.fn_get_health_status](https://docs.microsoft.com/sql/relational-databases/system-functions/managed-backup-fn-get-health-status-transact-sql) 資料表值函式。

另一個選項是利用內建的 Database Mail 功能進行通知。

1. 呼叫 [msdb.managed_backup.sp_set_parameter](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/managed-backup-sp-set-parameter-transact-sql) 預存程序，以將電子郵件地址指派至 **SSMBackup2WANotificationEmailIds** 參數。 
1. 啟用 [SendGrid](../../../sendgrid-dotnet-how-to-send-email.md)，以從 Azure VM 傳送電子郵件。
1. 使用 SMTP 伺服器與使用者名稱以設定 Database Mail。 您可以在 SQL Server Management Studio 中或使用 Transact-SQL 命令設定 Database Mail。 如需詳細資訊，請參閱 [Database Mail](https://docs.microsoft.com/sql/relational-databases/database-mail/database-mail)。
1. [設定 SQL Server Agent 以使用 Database Mail](https://docs.microsoft.com/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail)。
1. 確認是否允許 SMTP 連接埠經過本機虛擬機器防火牆和虛擬機器的網路安全性群組。

## <a name="next-steps"></a>後續步驟
「自動備份 v2」會在 Azure VM 上設定受控備份。 因此，請務必 [檢閱受控備份的文件](https://msdn.microsoft.com/library/dn449496.aspx) ，以了解其行為和隱含意義。

您可以在下列文章中找到 Azure VM 上 SQL Server 的其他備份和還原指引：[Azure 虛擬機器中的 SQL Server 備份和還原](virtual-machines-windows-sql-backup-recovery.md)。

如需有關其他可用之自動化工作的資訊，請參閱 [SQL Server IaaS 代理程式擴充功能](virtual-machines-windows-sql-server-agent-extension.md)。

如需有關在 Azure VM 上執行 SQL Server 的詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](virtual-machines-windows-sql-server-iaas-overview.md)。

