---
title: 在建立期間啟用 Azure VM 備份
description: 請參閱在建立程序期間啟用 Azure 虛擬機器備份的步驟。
services: backup, virtual-machines
author: markgalioto
manager: carmonm
tags: azure-resource-manager, virtual-machine-backup
ms.service: backup, virtual-machines
ms.topic: conceptual
ms.date: 01/08/2018
ms.author: trinadhk
ms.openlocfilehash: 928481f07875286a21f68dae6556f04eb2b6ae5c
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606115"
---
# <a name="enable-backup-during-azure-virtual-machine-creation"></a>在 Azure 虛擬機器建立期間啟用備份 

Azure 備份服務提供介面供您在雲端中建立和設定備份。 請定期建立備份 (稱為「復原點」) 以保護資料。 Azure 備份會建立復原點，其可儲存在異地備援復原保存庫中。 本文詳述當您在 Azure 入口網站中建立虛擬機器 (VM) 時，如何啟用備份。  

## <a name="log-in-to-azure"></a>登入 Azure 

如果您尚未登入帳戶，請登入 [Azure 入口網站](http://portal.azure.com)。
 
## <a name="create-virtual-machine-with-backup-configured"></a>建立已設定備份的虛擬機器 

1. 按一下 Azure 入口網站左上角的 [新增]。 

2. 選取 [計算]，然後選取虛擬機器的映像。   

3. 輸入虛擬機器資訊。 所提供的使用者名稱和密碼會用來登入虛擬機器。 完成時，按一下 [確定]。 

4. 選取 VM 的大小。  

5. 在 [設定] > [備份] 底下，按一下 [已啟用] 來啟動備份組態設定。 您可以接受預設值，然後按一下 [設定] 頁面上的 [確定]，繼續前往 [摘要] 頁面來建立 VM。 如果您要變更這些值，請遵循後續步驟。  

6. 建立或選取復原服務保存庫，以便保存虛擬機器的備份。 如果您要建立復原服務保存庫，則可以選擇保存庫的資源群組。  

    ![vm 建立頁面中的備份組態](./media/backup-during-vm-creation/create-vm-backup-config.png) 

    > [!NOTE] 
    > 復原服務保存庫的資源群組可與虛擬機器的資源群組不同。  
    > 
    > 

7. 根據預設，系統會為您選取備份原則以便快速保護 VM。 備份原則會指定建立備份快照的頻率，以及這些備份複本的保留時間長度。 您可以接受預設原則，也可以建立或選取不同的備份原則。 若要編輯備份原則，請選取 [備份原則] 並變更原則的值。  

8. 當備份組態值符合您的需要時，在 [設定] 頁面中按一下 [確定]。  

9. 在 [摘要] 頁面上，請於通過驗證後按一下 [建立]，以建立虛擬機器並讓其使用所設定的備份設定。 

## <a name="initiate-a-backup-after-creating-the-vm"></a>在建立 VM 之後起始備份 

雖然您已建立備份原則，但最好再建立初始備份。 若要在 VM 建立範本完成後檢視虛擬機器的備份詳細資料，請從左側功能表上的 [作業] 設定，按一下 [備份]。 您可以使用此選項來觸發隨選備份、還原完整的 VM 或所有磁碟、從 VM 備份還原檔案，或變更與虛擬機器相關聯的備份原則。  

## <a name="using-a-resource-manager-template-to-deploy-a-protected-vm"></a>使用 Resource Manager 範本部署受保護的 VM

先前的步驟說明如何使用 Azure 入口網站來建立虛擬機器，並保護它到復原服務保存庫。 如果您想要快速部署一或多部虛擬機器，並保護它們到復原服務保存庫，請參閱下列範本：[部署 Windows VM 並啟用備份](https://azure.microsoft.com/resources/templates/101-recovery-services-create-vm-and-configure-backup/)。

## <a name="frequently-asked-questions"></a>常見問題集 

### <a name="which-vm-images-enable-backup-at-the-time-of-vm-creation"></a>哪些 VM 映像可在建立 VM 時啟用備份？ 

Microsoft 所發佈的以下核心映像清單支援在建立 VM 期間啟用備份。 至於其他 VM，則可以在建立 VM 後啟用備份。 深入了解如何[在建立 VM 後啟用備份](quick-backup-vm-portal.md) 

- **Windows** - Windows Server 2016 Data center、Windows Server 2016 Data Center core、Windows Server 2012 DataCenter、Windows Server 2012 R2 DataCenter、Windows Server 2008 R2 SP1 
- **Ubuntu** - Ubuntu Server 1710、Ubuntu Server 1704、UUbuntu Server 1604(LTS)、Ubuntu Server 1404(LTS) 
- **Redhat** - RHEL 6.7、6.8、6.9、7.2、7.3、7.4 
- **SUSE** - SUSE Linux Enterprise Server 11 SP4、12 SP2、12 SP3 
- **Debian** - Debian 8、Debian 9 
- **CentOS** - CentOS 6.9、CentOS 7.3 
- **Oracle Linux** - Oracle Linux 6.7、6.8、6.9、7.2、7.3 
 
### <a name="is-backup-cost-included-in-the-vm-cost"></a>VM 成本中是否已納入備份成本？ 

否，備份成本與虛擬機器成本不同，兩者會分開計算。 如需備份定價的詳細資訊，請參閱[備份定價網站](https://azure.microsoft.com/pricing/details/backup/)。
 
### <a name="which-permissions-are-required-to-enable-backup-on-a-vm"></a>需要哪些權限才能在 VM 上啟用備份？ 

如果您是虛擬機器參與者，就能在 VM 上啟用備份。 如果您使用自訂角色，則需要下列權限才能成功地在 VM 上啟用備份。 

- Microsoft.RecoveryServices/Vaults/write 
- Microsoft.RecoveryServices/Vaults/read 
- Microsoft.RecoveryServices/locations/* 
- Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/*/read 
- Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read 
- Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/write 
- Microsoft.RecoveryServices/Vaults/backupFabrics/backupProtectionIntent/write 
- Microsoft.RecoveryServices/Vaults/backupPolicies/read 
- Microsoft.RecoveryServices/Vaults/backupPolicies/write 
 
如果復原服務保存庫和虛擬機器有不同的資源群組，請確保您具有復原服務保存庫資源群組的寫入權限。  

## <a name="next-steps"></a>後續步驟 

現在您已能保護您的 VM，請參閱下列文章，以了解 VM 管理工作及如何還原 VM。 

- [管理和監視虛擬機器](backup-azure-manage-vms.md) 
- [還原虛擬機器](backup-azure-arm-restore-vms.md) 
