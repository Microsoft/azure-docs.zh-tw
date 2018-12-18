---
title: Azure VM 備份常見問題集
description: 有關以下常見問題的解答：Azure VM 備份的運作方式、限制，以及原則變更時會發生什麼情況
services: backup
author: trinadhk
manager: shreeshd
keywords: azure vm 備份, azure vm 還原, 備份原則
ms.service: backup
ms.topic: conceptual
ms.date: 8/16/2018
ms.author: trinadhk
ms.openlocfilehash: 58b0622da2ef617e652c8bb9dacbf7daa2d79966
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42141084"
---
# <a name="questions-about-the-azure-vm-backup-service"></a>Azure VM 備份服務的相關問題
本文包含常見問題的解答，可協助您快速了解 Azure VM 備份元件。 在某些答案中，有具有完整資訊的文章連結。 您也可以在 [論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazureonlinebackup)中張貼有關 Azure 備份服務的問題。

## <a name="configure-backup"></a>設定備份
### <a name="do-recovery-services-vaults-support-classic-vms-or-resource-manager-based-vms-br"></a>復原服務保存庫是否支援傳統 VM 或以 Resource Manager 為基礎的 VM？ <br/>
復原服務保存庫支援兩種模式。  您可以將傳統 VM 或 Resource Manager VM 備份至復原服務保存庫。

### <a name="what-configurations-are-not-supported-by-azure-vm-backup"></a>Azure VM 備份不支援哪些組態？
請瀏覽[支援的作業系統](backup-azure-arm-vms-prepare.md#supported-operating-systems-for-backup)和 [VM 備份的限制](backup-azure-arm-vms-prepare.md#limitations-when-backing-up-and-restoring-a-vm)。

### <a name="why-cant-i-see-my-vm-in-configure-backup-wizard"></a>在設定備份精靈中為何看不到我的 VM？
在「設定備份精靈」中，Azure 備份只會列出這種 VM：
  * 尚未保護 - 移至 VM 刀鋒視窗，並從 [設定] 功能表檢查備份狀態，即可驗證 VM 的備份狀態。 進一步了解如何[檢查 VM 的備份狀態](backup-azure-vms-first-look-arm.md#configure-the-backup-job-from-the-vm-operations-menu)
  * 屬於與 VM 相同的區域

## <a name="backup"></a>Backup 
### <a name="will-on-demand-backup-job-follow-same-retention-schedule-as-scheduled-backups"></a>隨選備份作業是否會遵循與排定備份相同的保留排程？
否。 您應該指定隨選備份作業的保留範圍。 根據預設，若從入口網站觸發，會保留 30 天。 

### <a name="i-recently-enabled-azure-disk-encryption-on-some-vms-will-my-backups-continue-to-work"></a>我在最近一些 VM 上啟用了 Azure 磁碟加密。 我的備份是否會繼續運作？
您需要提供 Azure 備份服務存取 Key Vault 的權限。 您可以使用 [PowerShell](backup-azure-vms-automation.md) 文件的「啟用備份」一節中所述的步驟，在 PowerShell 中提供這些權限。

### <a name="i-migrated-disks-of-a-vm-to-managed-disks-will-my-backups-continue-to-work"></a>我已將 VM 的磁碟移轉到受控磁碟。 我的備份是否會繼續運作？
是，備份會順暢運作，而且不需要重新設定備份。 

### <a name="my-vm-is-shut-down-will-an-on-demand-or-a-scheduled-backup-work"></a>我的 VM 關閉了。 隨選或排程的備份工作會運作嗎？
是。 即使機器已經關閉，且備份工作和復原點標示為「當機時保持一致」。 如需詳細資訊，請參閱[這篇文章](backup-azure-vms-introduction.md#how-does-azure-back-up-virtual-machines)的＜資料一致性＞。

### <a name="can-i-cancel-an-in-progress-backup-job"></a>我是否可以取消進行中的備份作業？
是。 如果備份作業處於「正在建立快照」階段，您可以將它取消。 **如果正在從快照傳輸資料，則無法取消作業**。 

### <a name="i-enabled-resource-group-lock-on-my-backed-up-managed-disk-vms-will-my-backups-continue-to-work"></a>我已在已備份的受控磁碟 VM 上啟用資源群組鎖定。 我的備份是否會繼續運作？
如果使用者鎖定資源群組，備份服務就無法刪除較舊的還原點。 原因是這個新的備份因後端最多只能有 18 個還原點而開始失敗。 如果您的備份在 RG 鎖定之後，因為內部錯誤而發生失敗，請遵循下列[步驟來移除還原點集合](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#backup-service-does-not-have-permission-to-delete-the-old-restore-points-due-to-resource-group-lock)。

### <a name="does-backup-policy-take-daylight-saving-timedst-into-account"></a>備份原則是否會將日光節約時間 (DST) 列入考量？
否。 請留意本機電腦上顯示的日期和時間是您的當地時間，加上目前的日光節約時間偏差。 因此，排程備份設定的時間可能會因為 DST 而與您的當地時間不同。

## <a name="restore"></a>Restore
### <a name="how-do-i-decide-between-restoring-disks-versus-full-vm-restore"></a>如何在還原磁碟與完整 VM 還原之間做決定？
將 Azure VM 完整還原視為快速建立選項。 還原 VM 選項會變更磁碟的名稱、這些磁碟使用的容器、公用 IP 位址和網路介面名稱。 在 VM 建立期間，需要這些變更來維持建立之資源的唯一性。 但變更不會將 VM 加入可用性設定組。 

使用還原磁碟：
  * 自訂從時間點組態建立的 VM，例如變更大小
  * 新增備份時不存在的組態 
  * 控制建立資源時的命名慣例
  * 將 VM 新增至可用性設定組
  * 任何其他只使用 PowerShell/宣告式範本定義即可達成的組態
  
### <a name="can-i-use-backups-of-unmanaged-disk-vm-to-restore-after-i-upgrade-my-disks-to-managed-disks"></a>我的磁碟在升級為受控磁碟後，是否可以使用非受控磁碟 VM 的備份來進行還原？
是，您可以使用在將磁碟從非受控磁碟移轉為受控磁碟之前所擷取的備份。 根據預設，還原 VM 作業將會使用非受控磁碟來建立 VM。 您可以使用還原磁碟功能來還原磁碟，並用這些磁碟在受控磁碟上r建立 VM。 

### <a name="what-is-the-procedure-to-restore-a-vm-to-a-restore-point-taken-before-the-conversion-from-unmanaged-to-managed-disks-was-done-for-a-vm"></a>若要將虛擬機器的非受控磁碟轉換至受控磁碟，要如何將虛擬機器還原至還原點？其程序為何？
在此案例中，根據預設，還原虛擬機器的作業將會使用非受控磁碟來建立虛擬機器。 若要使用受控磁碟建立虛擬機器：
1. [還原至非受控磁碟](tutorial-restore-disk.md#restore-a-vm-disk)
2. [將還原的磁碟轉換至受控磁碟](tutorial-restore-disk.md#convert-the-restored-disk-to-a-managed-disk)
3. [使用受控磁碟建立虛擬機器](tutorial-restore-disk.md#create-a-vm-from-the-restored-disk) <br>
若是 Powershell Cmdlet，請參閱[此處](backup-azure-vms-automation.md#restore-an-azure-vm)。

### <a name="can-i-restore-the-vm-if-my-vm-is-deleted"></a>如果 VM 已刪除，是否可還原 VM？
是。 VM 的生命週期和其對應的備份項目不同。 因此，即使您刪除 VM，仍可移至復原服務保存庫中的對應備份項目，並使用其中一個復原點觸發還原作業。 

## <a name="manage-vm-backups"></a>管理 VM 備份
### <a name="what-happens-when-i-change-a-backup-policy-on-vms"></a>變更 VM 的備份原則時會發生什麼狀況？
在 VM 上套用新原則後，就會遵循新原則的排程和保留期。 如果延長保留期，會標示現有的復原點，以依據新的原則加以保留。 如果縮短保留期，則會標示現有的復原點，以便在下次清除作業中剪除然後刪除。 

### <a name="how-can-i-move-a-vm-enrolled-in-azure-backup-between-resource-groups"></a>如何在資源群組之間移動在 Azure 備份中註冊的 VM？
請遵循下列步驟，將已備份的 VM 成功移到目標資源群組 
1. 暫時停止備份並保留備份資料
2. 將 VM 移到目標資源群組
3. 使用相同/新的保存庫重新保護它

使用者可以從移動作業前建立的可用還原點進行還原。


