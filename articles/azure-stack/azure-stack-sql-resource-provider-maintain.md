---
title: 在 Azure Stack 上維護 SQL 資源提供者 | Microsoft Docs
description: 了解如何在 Azure Stack 上維護 SQL 資源提供者服務。
services: azure-stack
documentationCenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2018
ms.author: jeffgilb
ms.reviewer: jeffgo
ms.openlocfilehash: ad899739dab1dc51d64368d2136ab87f73f6f3a0
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "36300905"
---
# <a name="sql-resource-provider-maintenance-operations"></a>SQL 資源提供者維護作業

SQL 資源提供者會在鎖定的虛擬機器上執行。 若要啟用維護作業，您必須更新虛擬機器的安全性。 若要使用最低權限原則執行此作業，您可以使用 [PowerShell Just Enough Administration (JEA)](https://docs.microsoft.com/powershell/jea/overview) 端點 *DBAdapterMaintenance*。 資源提供者安裝套件包含這項作業的指令碼。

## <a name="patching-and-updating"></a>修補和更新

SQL 資源提供者不會隨著 Azure Stack 提供，因為它是附加元件。 Microsoft 會視需要將更新提供給 SQL 資源提供者。 釋出更新的 SQL 配接器時，將提供指令碼來套用更新。 此指令碼會建立新的資源提供者 VM，將舊提供者 VM 的狀態移轉至新的 VM。 如需詳細資訊，請參閱[更新 SQL 資源提供者](azure-stack-sql-resource-provider-update.md)。

### <a name="provider-virtual-machine"></a>提供者虛擬機器

因為資源提供者會在「使用者」虛擬機器上執行，所以您需要在必要的修補程式和更新釋出時加以套用。 您可以使用隨著修補和更新週期提供的 Windows 更新套件，將更新套用至 VM。

## <a name="backuprestoredisaster-recovery"></a>備份/還原/災害復原

 因為這是附加元件，所以不會在 Azure Stack 商務持續性災害復原 (BCDR) 過程中備份 SQL 資源提供者。 系統會提供下列作業的指令碼：

- 備份狀態資訊 (儲存在 Azure Stack 儲存體帳戶中)。
- 還原資源提供者 (如果需要完整堆疊復原)。

>[!NOTE]
>如果您必須進行復原，則必須先復原資料庫伺服器，才能復原資源提供者。

## <a name="updating-sql-credentials"></a>更新 SQL 認證

您需負責建立及維護 SQL 伺服器上的系統管理員帳戶。 資源提供者需要具有這些權限的帳戶，才能為使用者管理資料庫，但不需要存取使用者的資料。 如果需要更新 SQL Server 的系統管理員密碼，您可以使用資源提供者的系統管理員介面，變更已儲存的密碼。 這些密碼會儲存在 Azure Stack 執行個體上的金鑰保存庫中。

若要修改設定，請選取 [瀏覽] &gt; [管理資源] &gt; [SQL 主控伺服器] &gt; [SQL 登入] 並選取使用者名稱。 必須先在 SQL 執行個體上 (必要時還需在任何複本上) 進行變更。在 [設定] 底下，選取 [密碼]。

![更新管理員密碼](./media/azure-stack-sql-rp-deploy/sqlrp-update-password.PNG)

## <a name="secrets-rotation"></a>祕密輪替

這些指示只適用於 Azure Stack 整合系統 1804 版或更新版本。請勿在早於 1804 的 Azure Stack 版本上嘗試輪替秘密。

搭配使用 SQL 和 MySQL 資源提供者與 Azure Stack 整合系統時，您可以輪替下列基礎結構 (部署) 祕密：

- [部署期間所提供的](azure-stack-pki-certs.md)外部 SSL 憑證。
- 部署期間所提供的資源提供者 VM 本機系統管理員帳戶密碼。
- 資源提供者診斷使用者 (dbadapterdiag) 密碼。

### <a name="powershell-examples-for-rotating-secrets"></a>輪替祕密的 PowerShell 範例

**同時變更所有祕密。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    –DiagnosticsUserPassword $passwd `
    -DependencyFilesLocalPath $certPath `
    -DefaultSSLCertificatePassword $certPasswd  `
    -VMLocalCredential $localCreds
```

**變更診斷使用者密碼。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    –DiagnosticsUserPassword  $passwd
```

**變更 VM 本機系統管理員帳戶密碼。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    -VMLocalCredential $localCreds
```

**變更 SSL 憑證密碼。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    -DependencyFilesLocalPath $certPath `
    -DefaultSSLCertificatePassword $certPasswd
```

### <a name="secretrotationsqlproviderps1-parameters"></a>SecretRotationSQLProvider.ps1 參數

|參數|說明|
|-----|-----|
|AzCredential|Azure Stack 服務系統管理員帳戶認證。|
|CloudAdminCredential|Azure Stack 雲端管理網域帳戶認證。|
|PrivilegedEndpoint|用來存取 Get-AzureStackStampInformation 的特殊權限端點。|
|DiagnosticsUserPassword|診斷使用者帳戶密碼。|
|VMLocalCredential|MySQLAdapter VM 上的本機系統管理員帳戶。|
|DefaultSSLCertificatePassword|預設 SSL 憑證 (*pfx) 密碼。|
|DependencyFilesLocalPath|相依性檔案本機路徑。|
|     |     |

### <a name="known-issues"></a>已知問題

**問題**：祕密輪替記錄。<br>
如果祕密輪替自訂指令碼在執行時失敗，就不會自動收集祕密輪替記錄。

**因應措施**：<br>
使用 Get-AzsDBAdapterLogs Cmdlet 來收集所有資源提供者記錄，包括 C:\Log 中儲存的 AzureStack.DatabaseAdapter.SecretRotation.ps1_*.log。

## <a name="update-the-virtual-machine-operating-system"></a>更新虛擬機器作業系統

使用下列其中一種方法來更新虛擬機器作業系統。

- 使用目前已修補的 Windows Server 2016 Core 映像來安裝最新的資源提供者套件。
- 在安裝或更新資源提供者期間，安裝 Windows Update 套件。

## <a name="update-the-virtual-machine-windows-defender-definitions"></a>更新虛擬機器 Windows Defender 定義

若要更新 Windows Defender 定義：

1. 從 [Windows Defender 定義](https://www.microsoft.com/en-us/wdsi/definitions) \(英文\) 下載 Windows Defender 定義更新。

   在定義更新頁面上，向下捲動至「手動下載並安裝定義」。 下載「適用於 Windows 10 和 Windows 8.1 的 Windows Defender Antivirus」64 位元檔案。

   或者，使用[此直接連結](https://go.microsoft.com/fwlink/?LinkID=121721&arch=x64)來下載/執行 fpam-fe.exe 檔案。

2. 建立連至 SQL 資源提供者配接器虛擬機器維護端點的 PowerShell 工作階段。

3. 使用維護端點工作階段，將定義更新檔案複製到虛擬機器。

4. 在維護 PowerShell 工作階段上，執行 *Update-DBAdapterWindowsDefenderDefinitions* 命令。

5. 在您安裝定義之後，建議您使用 *Remove-ItemOnUserDrive* 命令來刪除定義更新檔案。

**用於更新定義的 PowerShell 指令碼範例。**

您可以編輯並執行下列指令碼來更新 Defender 定義。 以您環境中的值取代指令碼中的值。

```powershell
# Set credentials for local admin on the resource provider VM.
$vmLocalAdminPass = ConvertTo-SecureString "<local admin user password>" -AsPlainText -Force
$vmLocalAdminUser = "<local admin user name>"
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential `
    ($vmLocalAdminUser, $vmLocalAdminPass)

# Provide the public IP address for the adapter VM.
$databaseRPMachine  = "<RP VM IP address>"
$localPathToDefenderUpdate = "C:\DefenderUpdates\mpam-fe.exe"

# Download the Windows Defender update definitions file from https://www.microsoft.com/en-us/wdsi/definitions.
Invoke-WebRequest -Uri 'https://go.microsoft.com/fwlink/?LinkID=121721&arch=x64' `
    -Outfile $localPathToDefenderUpdate

# Create a session to the maintenance endpoint.
$session = New-PSSession -ComputerName $databaseRPMachine `
    -Credential $vmLocalAdminCreds -ConfigurationName DBAdapterMaintenance
# Copy the defender update file to the adapter virtual machine.
Copy-Item -ToSession $session -Path $localPathToDefenderUpdate `
     -Destination "User:\"
# Install the update definitions.
Invoke-Command -Session $session -ScriptBlock `
    {Update-AzSDBAdapterWindowsDefenderDefinition -DefinitionsUpdatePackageFile "User:\mpam-fe.exe"}
# Cleanup the definitions package file and session.
Invoke-Command -Session $session -ScriptBlock `
    {Remove-AzSItemOnUserDrive -ItemPath "User:\mpam-fe.exe"}
$session | Remove-PSSession
```

## <a name="collect-diagnostic-logs"></a>收集診斷記錄

若要從鎖定的虛擬機器收集記錄，您可以使用 PowerShell Just Enough Administration (JEA) 端點 *DBAdapterDiagnostics*。 此端點會提供下列命令：

- **Get-AzsDBAdapterLog**。 此命令會建立資源提供者診斷記錄的 zip 套件，並將檔案儲存在工作階段的使用者磁碟機上。 您可以執行此命令 (不使用任何參數)，並收集過去四個小時的記錄。
- **Remove-AzsDBAdapterLog**。 此命令會移除資源提供者 VM 上現有的記錄套件。

### <a name="endpoint-requirements-and-process"></a>端點需求和程序

安裝或更新資源提供者時，會建立 **dbadapterdiag** 使用者帳戶。 您將使用此帳戶來收集診斷記錄。

>[!NOTE]
>dbadapterdiag 帳戶密碼與提供者部署或更新期間所建立的虛擬機器上本機系統管理員使用的密碼相同。

若要使用 *DBAdapterDiagnostics* 命令，請建立可連至資源提供者虛擬機器的遠端 PowerShell 工作階段，然後執行 **Get-AzsDBAdapterLog** 命令。

使用 **FromDate** 和 **ToDate** 參數，設定記錄收集的時間範圍。 如果您未指定上述一或兩個參數，則會使用下列預設值：

- FromDate 為目前時間的前四小時。
- ToDate 為目前時間。

**用於收集記錄的 PowerShell 指令碼範例。**

下列指令碼示範如何從資源提供者 VM 收集診斷記錄。

```powershell
# Create a new diagnostics endpoint session.
$databaseRPMachineIP = '<RP VM IP address>'
$diagnosticsUserName = 'dbadapterdiag'
$diagnosticsUserPassword = '<Enter Diagnostic password>'

$diagCreds = New-Object System.Management.Automation.PSCredential `
        ($diagnosticsUserName, (ConvertTo-SecureString -String $diagnosticsUserPassword -AsPlainText -Force))
$session = New-PSSession -ComputerName $databaseRPMachineIP -Credential $diagCreds `
        -ConfigurationName DBAdapterDiagnostics

# Sample that captures logs from the previous hour.
$fromDate = (Get-Date).AddHours(-1)
$dateNow = Get-Date
$sb = {param($d1,$d2) Get-AzSDBAdapterLog -FromDate $d1 -ToDate $d2}
$logs = Invoke-Command -Session $session -ScriptBlock $sb -ArgumentList $fromDate,$dateNow

# Copy the logs to the user drive.
$sourcePath = "User:\{0}" -f $logs
$destinationPackage = Join-Path -Path (Convert-Path '.') -ChildPath $logs
Copy-Item -FromSession $session -Path $sourcePath -Destination $destinationPackage

# Cleanup the logs.
$cleanup = Invoke-Command -Session $session -ScriptBlock {Remove- AzsDBAdapterLog }
# Close the session.
$session | Remove-PSSession
```

## <a name="next-steps"></a>後續步驟

[新增 SQL Server 主控伺服器](azure-stack-sql-resource-provider-hosting-servers.md)
