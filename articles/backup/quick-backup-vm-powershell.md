---
title: Azure 快速入門 - 使用 PowerShell 來備份 VM
description: 了解如何使用 Azure PowerShell 來備份虛擬機器
services: backup
author: markgalioto
manager: carmonm
tags: azure-resource-manager, virtual-machine-backup
ms.service: backup
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 2/14/2018
ms.author: markgal
ms.custom: mvc
ms.openlocfilehash: 4161b11e88d2b3201e18e095e13db864e25d7bfc
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34607531"
---
# <a name="back-up-a-virtual-machine-in-azure-with-powershell"></a>使用 PowerShell 在 Azure 中備份虛擬機器
Azure PowerShell 模組用於從命令列或在指令碼中建立和管理 Azure 資源。 您可以定期建立備份以保護您的資料。 Azure 備份會建立復原點，其可儲存在異地備援復原保存庫中。 本文詳述如何使用 Azure PowerShell 模組備份虛擬機器 (VM)。 您也可以使用 [Azure CLI](quick-backup-vm-cli.md) 或 [Azure 入口網站](quick-backup-vm-portal.md)來執行這些步驟。

本快速入門能夠在現有的 Azure VM 上進行備份。 如果您需要建立 VM，您可以[使用 Azure PowerShell 來建立 VM](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fpowershell%2fmodule%2ftoc.json)。

本快速入門需要 Azure PowerShell 模組 4.4 版或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。


## <a name="log-in-to-azure"></a>登入 Azure
使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

```powershell
Connect-AzureRmAccount
```

第一次使用 Azure 備份時，您必須使用 [Register-AzureRmResourceProvider](/powershell/module/AzureRM.Resources/Register-AzureRmResourceProvider) 在您的訂用帳戶中註冊 Azure 復原服務提供者。

```powershell
Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
```


## <a name="create-a-recovery-services-vaults"></a>建立復原服務保存庫
復原服務保存庫是一個邏輯容器，可儲存每個受保護資源 (例如 Azure VM) 的備份資料。 執行受保護資源的備份作業時，它會在復原服務保存庫內建立復原點。 然後您可以使用其中一個復原點，將資料還原到指定的時間點。

使用 [New-AzureRmRecoveryServicesVault](/powershell/module/azurerm.recoveryservices/new-azurermrecoveryservicesvault) 來建立復原服務保存庫。 指定與您想要保護的 VM 相同的資源群組和位置。 如果您使用了[範例指令碼](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fpowershell%2fmodule%2ftoc.json)來建立 VM，則資源群組會命名為 *myResourceGroup*、VM 會命名為 *myVM*，而資源位於 *WestEurope* 位置。

```powershell
New-AzureRmRecoveryServicesVault `
    -ResourceGroupName "myResourceGroup" `
    -Name "myRecoveryServicesVault" `
    -Location "WestEurope"
```

根據預設，已針對異地備援儲存體設定保存庫。 若要進一步保護您的資料，此儲存體備援層級可確保備份資料會複寫到與主要地區距離數百英哩的次要 Azure 地區。

若要在剩餘的步驟中使用此保存庫，請使用 [Set-AzureRmRecoveryServicesVaultContext](/powershell/module/AzureRM.RecoveryServices/Set-AzureRmRecoveryServicesVaultContext) 設定保存庫內容。

```powershell
Get-AzureRmRecoveryServicesVault `
    -Name "myRecoveryServicesVault" | Set-AzureRmRecoveryServicesVaultContext
```


## <a name="enable-backup-for-an-azure-vm"></a>啟用 Azure VM 的備份
您可以建立和使用相關原則，來定義備份作業的執行時以及復原點的儲存時間長度。 預設保護原則會每天執行備份作業並將復原點保留 30 天。 您可以使用這些預設原則值來快速保護您的 VM。 首先，使用 [Get-AzureRmRecoveryServicesBackupProtectionPolicy](/powershell/module/AzureRM.RecoveryServices.Backup/Get-AzureRmRecoveryServicesBackupProtectionPolicy) 設定預設原則：

```powershell
$policy = Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"
```

若要啟用 VM 的備份保護，請使用 [Enable-AzureRmRecoveryServicesBackupProtection](/powershell/module/AzureRM.RecoveryServices.Backup/Enable-AzureRmRecoveryServicesBackupProtection)。 指定要使用的原則，然後指定要保護的資源群組和 VM：

```powershell
Enable-AzureRmRecoveryServicesBackupProtection `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Policy $policy
```


## <a name="start-a-backup-job"></a>開始備份作業
若要立即開始備份，而非等候預設原則在排定的時間執行此作業，請使用 [Backup-AzureRmRecoveryServicesBackupItem](/powershell/module/azurerm.recoveryservices.backup/backup-azurermrecoveryservicesbackupitem)。 這第一個備份作業會建立完整復原點。 此初始備份之後的每個備份作業會建立增量復原點。 增量復原點符合儲存和時間效率，因為它只會傳輸自上次備份後所做的變更。

在下列命令中，您可使用 [Get-AzureRmRecoveryServicesBackupContainer](/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupcontainer) 指定復原服務保存庫中用於保留備份資料的容器。 要備份的每個 VM 都會被視為一個項目。 若要開始備份作業，請使用 [Get-AzureRmRecoveryServicesBackupItem](/powershell/module/AzureRM.RecoveryServices.Backup/Get-AzureRmRecoveryServicesBackupItem) 取得 VM 項目的相關資訊。

```powershell
$backupcontainer = Get-AzureRmRecoveryServicesBackupContainer `
    -ContainerType "AzureVM" `
    -FriendlyName "myVM"

$item = Get-AzureRmRecoveryServicesBackupItem `
    -Container $backupcontainer `
    -WorkloadType "AzureVM"

Backup-AzureRmRecoveryServicesBackupItem -Item $item
```

因為這第一個備份作業會建立完整復原點，所以程序可能需要長達 20 分鐘。


## <a name="monitor-the-backup-job"></a>監視備份作業
若要監視備份作業的狀態，請使用 [Get-AzureRmRecoveryservicesBackupJob](/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupjob)：

```powershell
Get-AzureRmRecoveryservicesBackupJob
```

輸出會類似下列範例，其顯示的備份作業為 InProgress︰

```
WorkloadName   Operation         Status       StartTime              EndTime                JobID
------------   ---------         ------       ---------              -------                -----
myvm           Backup            InProgress   9/18/2017 9:38:02 PM                          9f9e8f14
myvm           ConfigureBackup   Completed    9/18/2017 9:33:18 PM   9/18/2017 9:33:51 PM   fe79c739
```

當備份作業的「狀態」回報 Completed，您的 VM 已受復原服務保護並已儲存完整復原點。


## <a name="clean-up-deployment"></a>清除部署
不再需要時，您可以停用 VM 的保護功能，移除還原點和復原服務保存庫，然後刪除資源群組和相關聯的 VM 資源。 如果您使用現有的 VM，可以略過最後的 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令，讓資源群組和 VM 留在原處。

如果您要繼續進行說明如何為您的 VM 還原資料的備份教學課程，請略過本節中的步驟並前往[後續步驟](#next-steps)。 

```powershell
Disable-AzureRmRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzureRmRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzureRmRecoveryServicesVault -Vault $vault
Remove-AzureRmResourceGroup -Name "myResourceGroup"
```


## <a name="next-steps"></a>後續步驟
在本快速入門中，您已建立復原服務保存庫、啟用 VM 的保護功能，並建立初始復原點。 若要深入了解 Azure 備份和復原服務，請繼續進行教學課程。

> [!div class="nextstepaction"]
> [備份多個 Azure VM](./tutorial-backup-vm-at-scale.md)
