---
title: 使用 Azure 備份來備份和還原已加密的 VM
description: 本文討論使用「Azure 磁碟加密」加密之 VM 的備份與還原經驗。
services: backup
author: sogup
manager: vijayts
ms.service: backup
ms.topic: conceptual
ms.date: 7/10/2018
ms.author: sogup
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4b060fc3d273a0243271d2c38f90e81f83857e79
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39420323"
---
# <a name="back-up-and-restore-encrypted-virtual-machines-with-azure-backup"></a>使用 Azure 備份來備份及還原加密的虛擬機器
本文討論使用 Azure 備份來備份和還原虛擬機器 (VM) 的步驟。 它也提供有關支援的案例、必要條件的詳細資料，以及的錯誤案例的疑難排解步驟。

## <a name="supported-scenarios"></a>支援的案例

 只有使用 Azure Resource Manager 部署模型的 VM 支援備份及還原加密的 VM。 使用傳統部署模型的 VM 不支援。 使用 Azure 磁碟加密的 Windows 和 Linux VM 皆支援備份及還原加密的 VM。 磁碟加密使用 Windows 的業界標準 BitLocker 功能和 Linux 的 dm-crypt 功能來提供磁碟的加密。 下表列出了虛擬機器的加密類型和支援。

   |  | BEK + KEK VM | 僅限 BEK VM |
   | --- | --- | --- |
   | **非受控 VM**  | 是 | 是  |
   | **受控 VM**  | 是 | 是  |

## <a name="prerequisites"></a>必要條件
* VM 是使用 [Azure 磁碟加密](../security/azure-security-disk-encryption.md)來加密。

* 已建立復原服務保存庫，並且藉由遵循[準備環境以便備份](backup-azure-arm-vms-prepare.md)中的步驟來設定儲存體複寫。

* 備份已獲得[存取金鑰保存庫的權限](#provide-permissions-to-backup)，該保存庫包含已加密 VM 的金鑰和祕密。

## <a name="backup-encrypted-vm"></a>備份加密的 VM
使用下列步驟來設定備份目標、定義原則、設定項目和觸發備份。

### <a name="configure-backup"></a>設定備份
1. 如果您已開啟復原服務保存庫，請繼續下一個步驟。 如果您並未開啟復原服務保存庫，但位於 Azure 入口網站中，請選取 [所有服務]。

   a. 在資源清單中輸入 **復原服務**。

   b. 當您開始輸入時，清單會根據您輸入的文字進行篩選。 當您看到 [復原服務保存庫] 時，請選取它。

      ![復原服務保存庫](./media/backup-azure-vms-encryption/browse-to-rs-vaults.png) <br/>

    c. 隨即會出現 [復原服務保存庫] 清單。 從清單中選取保存庫。

     選取的保存庫儀表板隨即開啟。
1. 從出現在保存庫下方的項目清單中，選取 [備份] 以開始備份加密 VM。

      ![備份刀鋒視窗](./media/backup-azure-vms-encryption/select-backup.png)
1. 在 [備份] 圖格中，選取 [備份目標]。

      ![案例刀鋒視窗](./media/backup-azure-vms-encryption/select-backup-goal-one.png)
1. 在 [工作負載的執行位置?] 底下，選取 [Azure]。 在 [您要備份什麼?] 底下，選取 [虛擬機器]。 然後選取 [確定]。

   ![開啟 [案例] 刀鋒視窗](./media/backup-azure-vms-encryption/select-backup-goal-two.png)
1. 在 [選擇備份原則] 底下，選取您要套用至保存庫的備份原則。 然後選取 [確定]。

      ![選取備份原則](./media/backup-azure-vms-encryption/setting-rs-backup-policy-new.png)

    預設原則的詳細資料隨即列出。 如果您想要建立原則，請在下拉式清單中選取 [建立新的]。 選取 [確定] 之後，備份原則便會與保存庫建立關聯。

1. 選擇要與指定的原則建立關聯的已加密 VM，然後選取 [確定]。

      ![選取加密的 VM](./media/backup-azure-vms-encryption/selected-encrypted-vms.png)
1. 此頁面會顯示與所選加密的 VM 相關聯的金鑰保存庫相關訊息。 備份需要金鑰保存庫中金鑰和密碼的唯讀存取權。 它會使用這些權限來備份金鑰和密碼，以及相關聯的 VM。<br>
如果您是**成員使用者**，「啟用備份」程序將會以無縫接軌的方式取得金鑰保存庫的存取權來備份加密的 VM，而不需要任何使用者介入操作。

   ![加密的 VM 訊息](./media/backup-azure-vms-encryption/member-user-encrypted-vm-warning-message.png)

   針對**來賓使用者**，您則必須將金鑰保存庫的存取權限提供給備份服務，才能進行備份。 您可以依照[下一節所述的步驟](#provide-permissions-to-backup)來提供這些權限

   ![加密的 VM 訊息](./media/backup-azure-vms-encryption/guest-user-encrypted-vm-warning-message.png)
 
    現在您已定義保存庫的所有設定，接下來選取分頁底部的 [啟用備份]。 [啟用備份] 會將原則部署至保存褲和 VM。
  
1. 下一個階段的準備作業是安裝 VM 代理程式，或確定 VM 代理程式已安裝。 若要執行相同的動作，請依照[準備環境以進行備份](backup-azure-arm-vms-prepare.md)中的步驟。

### <a name="trigger-a-backup-job"></a>觸發備份作業
遵循[將 Azure VM 備份至復原服務保存庫](backup-azure-arm-vms.md)中的步驟來觸發備份作業。

### <a name="continue-backups-of-already-backed-up-vms-with-encryption-enabled"></a>將啟用加密之已備份 VM 繼續備份  
如果您的 VM 已在復原服務保存庫中進行備份，並於稍後已啟用加密，您必須將備份的權限提供給金鑰保存庫，才可繼續進行備份。 您可以遵循[下一節的步驟](#provide-permissions-to-azure-backup)來提供這些權限。 或者，您可以遵循 [PowerShell 文件](backup-azure-vms-automation.md)之「啟用備份」一節中的 PowerShell 步驟。 

## <a name="provide-permissions-to-backup"></a>對備份提供權限
使用下列步驟來提供相關權限給備份，以供其存取金鑰保存庫並執行已加密 VM 的備份。
1. 選取 [所有服務]，然後搜尋**金鑰保存庫**。

    ![金鑰保存庫](./media/backup-azure-vms-encryption/search-key-vault.png)
    
1. 從金鑰保存庫清單中選取與已加密之 VM 相關聯的金鑰保存庫，以讓其進行備份。

     ![金鑰保存庫選取項目](./media/backup-azure-vms-encryption/select-key-vault.png)
     
1. 選取 [存取原則]，然後選取 [新增]。

    ![新增](./media/backup-azure-vms-encryption/select-key-vault-access-policy.png)
    
1. 選取 [選取主體]，然後在搜尋方塊中輸入**備份管理服務**。 

    ![備份服務搜尋](./media/backup-azure-vms-encryption/search-backup-service.png)
    
1. 選取 [備份管理服務]，然後選取 [選取]。

    ![備份服務選取項目](./media/backup-azure-vms-encryption/select-backup-service.png)
    
1. 在 [從範本設定 (選擇性)] 底下，選取 [Azure 備份]。 該備份會在 [金鑰權限] 和 [祕密權限] 預先填入必要的權限。 如果您是使用**僅 BE** 加密您的 VM，則需要機密資料的權限，因此您必須移除**金鑰權限**的選取項目。

    ![Azure 備份選取項目](./media/backup-azure-vms-encryption/select-backup-template.png)
    
1. 選取 [確定] 。 請注意「備份管理服務」會新增到 [存取原則] 中。 

    ![存取原則](./media/backup-azure-vms-encryption/backup-service-access-policy.png)
    
1. 選取 [儲存] 對備份提供必要權限。

    ![備份存取原則](./media/backup-azure-vms-encryption/save-access-policy.png)

順利提供權限之後，您可以繼續為已加密的 VM 啟用備份。

## <a name="restore-an-encrypted-vm"></a>還原已加密的 VM
若要還原加密的 VM，請先遵循[選擇 VM 還原組態](backup-azure-arm-restore-vms.md#choose-a-vm-restore-configuration)之「還原備份的磁碟」一節中的步驟來還原磁碟。 之後，您可以使用下列其中一個選項：

* 遵循[從還原的磁碟建立 VM](backup-azure-vms-automation.md#create-a-vm-from-restored-disks) 中的 PowerShell 步驟，從還原的磁碟建立完整的 VM。
* 或者，[使用範本來自訂還原的 VM](backup-azure-arm-restore-vms.md#use-templates-to-customize-a-restored-vm)，以從還原的磁碟建立 VM。 只可將範本用於 2017 年 4 月 26 日之後建立的復原點。

## <a name="troubleshooting-errors"></a>錯誤疑難排解
| 作業 | 錯誤詳細資料 | 解決方案 |
| --- | --- | --- |
|Backup  | 備份沒有足夠的金鑰保存庫權限可以進行加密 VM 的備份。 | 備份應該依照[上一節中的步驟](#provide-permissions-to-azure-backup)來提供這些權限。 或者，您可以依照[使用 AzureRM.RecoveryServices.Backup Cmdlet 來備份虛擬機器](backup-azure-vms-automation.md#back-up-azure-vms)之 PowerShell 文件中「啟用保護」一節的 PowerShell 步驟。 |  
| Restore |您無法還原這部已加密的 VM，因為與此 VM 相關聯的金鑰保存庫不存在。 |利用[開始使用 Azure Key Vault](../key-vault/key-vault-get-started.md)，建立金鑰保存庫。 請參閱[使用 Azure 備份來還原金鑰保存庫金鑰和密碼](backup-azure-restore-key-secret.md)，來還原金鑰和密碼 (如果不存在)。 |
| Restore |您無法還原這部已加密的 VM，因為與此 VM 相關聯的金鑰和密碼不存在。 |請參閱[使用 Azure 備份來還原金鑰保存庫金鑰和密碼](backup-azure-restore-key-secret.md)，來還原金鑰和密碼 (如果不存在)。 |
| Restore |備份無權存取您訂用帳戶中的資源。 |如先前所述，請先遵循[選擇 VM 還原組態](backup-azure-arm-restore-vms.md#choose-a-vm-restore-configuration)之「還原備份的磁碟」一節中的步驟來還原磁碟。 之後，使用 PowerShell 來[從還原的磁碟建立 VM](backup-azure-vms-automation.md#create-a-vm-from-restored-disks)。 |
