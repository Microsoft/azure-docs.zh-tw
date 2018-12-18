---
title: 在 Azure 中大規模備份 Azure VM
description: 本教學課程會詳細說明將多部 Azure 虛擬機器備份至復原服務保存庫。
services: backup
author: markgalioto
manager: carmonm
keywords: 虛擬機器備份; 備份虛擬機器; 備份與災害復原
ms.service: backup
ms.topic: tutorial
ms.date: 09/06/2017
ms.author: trinadhk
ms.custom: mvc
ms.openlocfilehash: 863960e012a8e345434459ad16526c8971f00b6b
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606999"
---
# <a name="back-up-azure-virtual-machines-in-azure-at-scale"></a>在 Azure 中大規模備份 Azure 虛擬機器

本教學課程會詳細說明如何將多部 Azure 虛擬機器備份至復原服務保存庫。 備份虛擬機器的工作多數是準備。 在備份 (或保護) 虛擬機器之前，您必須完成[必要條件](backup-azure-arm-vms-prepare.md)來備妥保護 VM 的環境。 

> [!IMPORTANT]
> 本教學課程假設您已經建立資源群組和 Azure 虛擬機器。

## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫

[復原服務保存庫](backup-azure-recovery-services-vault-overview.md)是一種容器，保留要備份項目的復原點。 復原服務保存庫是 Azure 資源，可以當成 Azure 資源群組的一部分來部署及管理。 在本教學課程中，您要在和受保護虛擬機器相同的資源群組中，建立復原服務保存庫。


第一次使用 Azure 備份時，您必須使用訂用帳戶註冊 Azure 復原服務提供者。 如已使用您的訂用帳戶註冊提供者，請移至下一個步驟。

```powershell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.RecoveryServices
```

使用 **New-AzureRmRecoveryServicesVault** 來建立復原服務保存庫。 設定要備份的虛擬機器時，請務必指定所使用的資源群組名稱和位置。 

```powershell
New-AzureRmRecoveryServicesVault -Name myRSvault -ResourceGroupName "myResourceGroup" -Location "EastUS"
```

許多 Azure 備份 Cmdlet 都需要將復原服務保存庫物件當做輸入。 基於這個理由，將備份復原服務保存庫物件儲存在變數中會是方便的做法。 然後使用 **Set-AzureRmRecoveryServicesBackupProperties** 將 **-BackupStorageRedundancy** 選項設定為[異地備援儲存體 (GRS)](../storage/common/storage-redundancy-grs.md)。 

```powershell
$vault1 = Get-AzureRmRecoveryServicesVault –Name myRSVault
Set-AzureRmRecoveryServicesBackupProperties  -vault $vault1 -BackupStorageRedundancy GeoRedundant
```

## <a name="back-up-azure-virtual-machines"></a>備份 Azure 虛擬機器

您必須先設定保存庫內容，才可以執行初始備份。 保存庫內容是保存庫中受保護的資料類型。 當您建立復原服務保存庫時，它會隨附預設的保護和保留原則。 預設保護原則會在每天的指定時間觸發備份作業。 預設保留原則會將每日復原點保留 30 天。 在本教學課程中，請接受預設原則。 

使用 **[Set-AzureRmRecoveryServicesVaultContext](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices/set-azurermrecoveryservicesvaultcontext)** 設定保存庫內容。 保存庫內容設定之後就會套用至所有後續的 Cmdlet。 

```powershell
Get-AzureRmRecoveryServicesVault -Name myRSVault | Set-AzureRmRecoveryServicesVaultContext
```

使用 **[Backup-AzureRmRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/backup-azurermrecoveryservicesbackupitem)** 觸發備份作業。 備份作業會建立復原點。 如果是初始備份，則復原點是完整備份。 後續備份會建立增量複本。

```powershell
$namedContainer = Get-AzureRmRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered -FriendlyName "V2VM"
$item = Get-AzureRmRecoveryServicesBackupItem -Container $namedContainer -WorkloadType AzureVM
$job = Backup-AzureRmRecoveryServicesBackupItem -Item $item
```

## <a name="delete-the-recovery-services-vault"></a>刪除復原服務保存庫

若要刪除復原服務保存庫，您必須先刪除保存庫所有復原點，再取消註冊保存庫。 下列命令將詳述這些步驟。 


```powershell
$Cont = Get-AzureRmRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered
$PI = Get-AzureRmRecoveryServicesBackupItem -Container $Cont[0] -WorkloadType AzureVm
Disable-AzureRmRecoveryServicesBackupProtection -RemoveRecoveryPoints $PI[0]
Unregister-AzureRmRecoveryServicesBackupContainer -Container $namedContainer
Remove-AzureRmRecoveryServicesVault -Vault $vault1
```

## <a name="troubleshooting-errors"></a>錯誤疑難排解
如果在備份虛擬機器時遇到問題，請參閱[備份 Azure 虛擬機器疑難排解文章](backup-azure-vms-troubleshoot.md)以取得說明。

## <a name="next-steps"></a>後續步驟
現在您的虛擬機器已受到保護，請參閱下列文章了解管理工作及如何從復原點還原虛擬機器。

* 若要修改備份原則，請參閱[使用 AzureRM.RecoveryServices.Backup Cmdlet 備份虛擬機器](backup-azure-vms-automation.md#create-a-protection-policy)。
* [管理和監視虛擬機器](backup-azure-manage-vms.md)
* [還原虛擬機器](backup-azure-arm-restore-vms.md)
