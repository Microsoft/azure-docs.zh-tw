---
title: 復原服務保存庫概觀
description: 復原服務保存庫和 Azure 備份保存庫之間的概觀與比較。
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 8/10/2018
ms.author: markgal
ms.openlocfilehash: 04b755d7ca1ec72b075ecaa4b32a35f5009a938b
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42141673"
---
# <a name="recovery-services-vaults-overview"></a>復原服務保存庫概觀

本文說明復原服務保存庫的功能。 復原服務保存庫是 Azure 中裝載資料的儲存體實體。 資料通常是資料的副本，或是虛擬機器 (VM)、工作負載、伺服器或工作站的設定資訊。 您可以使用復原服務保存庫來保存各種 Azure 服務 (例如 IaaS VM (Linux 或 Windows) 和 Azure SQL Database) 的備份資料。 復原服務保存庫支援 System Center DPM、Windows Server、Azure 備份伺服器等等。 復原服務保存庫可輕鬆地組織您的備份資料，同時可減輕管理負擔。 

在 Azure 訂用帳戶內，您可以為每個區域最多建立 500 個復原服務保存庫。

## <a name="comparing-recovery-services-vaults-and-backup-vaults"></a>比較復原服務保存庫與備份保存庫

如果您仍有「備份」保存庫，它們會自動被升級為「復原服務」保存庫。 到了 2017 年11 月，所有備份保存庫都會自動被升級為復原服務保存庫。 

復原服務保存庫是以 Azure 的 Azure Resource Manager 模型作為基礎，而備份保存庫則是以 Azure Service Manager 模型為作為基礎。 當您將備份保存庫升級至復原服務保存庫時，備份資料在升級程序期間及升級程序之後都會維持不變。 復原服務保存庫可提供備份保存庫無法使用的功能，例如︰

- **可協助保護備份資料安全的增強功能**︰透過復原服務保存庫，Azure 備份能提供安全性功能來保護雲端備份。 這些安全性功能可確保您能保護您的備份，並安全地將資料復原，即使生產和備份伺服器遭到入侵也一樣。 [深入了解](backup-azure-security-feature.md)

- **將您的混合式 IT 環境集中監視**︰透過復原服務保存庫，您不只可以監視 [Azure IaaS VM](backup-azure-manage-vms.md)，還可以從中央入口網站監視[內部部署資產](backup-azure-manage-windows-server.md#manage-backup-items)。 [深入了解](http://azure.microsoft.com/blog/alerting-and-monitoring-for-azure-backup)

- **角色型存取控制 (RBAC)**：RBAC 提供 Azure 中的更細緻存取權管理。 [Azure 提供各種內建角色](../role-based-access-control/built-in-roles.md)，且 Azure Backup 有三個[內建的角色可用來管理復原點](backup-rbac-rs-vault.md)。 復原服務保存庫與 RBAC 相容，且會對一組定義之使用者角色的備份和還原存取權限加以限制。 [深入了解](backup-rbac-rs-vault.md)

- **保護 Azure 虛擬機器的所有設定**︰復原服務保存庫會保護以 Resource Manager 為基礎的 VM，包括進階磁碟、受控磁碟及加密的 VM。 將備份保存庫升級至復原服務保存庫，讓您有機會可將以 Service Manager 為基礎的 VM 升級至以 Resource Manager 為基礎的 VM。 在升級保存庫時，您可以保留以 Service Manager 為基礎的 VM 復原點，並設定已升級 (已啟用 Resource Manager) 的 VM 保護。 [深入了解](http://azure.microsoft.com/blog/azure-backup-recovery-services-vault-ga)

- **IaaS VM 的立即還原**︰您可以使用復原服務保存庫，從 IaaS VM 還原檔案和資料夾，而非還原整個 VM，這樣可加速還原時間。 IaaS VM 的立即還原適用於 Windows 和 Linux VM。 [深入了解](http://azure.microsoft.com/blog/instant-file-recovery-from-azure-linux-vm-backup-using-azure-backup-preview)

## <a name="managing-your-recovery-services-vaults-in-the-portal"></a>在入口網站中管理復原服務保存庫
建立和管理 Azure 入口網站中的復原服務保存庫很輕鬆，因為備份服務已整合至其他 Azure 服務。 這項整合代表您可以在目標服務的內容中，建立或管理復原服務保存庫。 例如，若要檢視 VM 的復原點，請選取您的 VM，然後按一下 [作業] 功能表中的 [備份]。 

![復原服務保存庫詳細資料 VM](./media/backup-azure-recovery-services-vault-overview/rs-vault-in-context-vm.png)

如果 VM 沒有設定備份，則會出現要您設定備份的提示。 如果已設定備份，您會看到與 VM 相關的備份資訊，其中包括還原點清單。  

![復原服務保存庫詳細資料 VM](./media/backup-azure-recovery-services-vault-overview/vm-recovery-point-list.png)

在先前範例中，**ContosoVM** 是虛擬機器的名稱。 **ContosoVM demovault** 是復原服務保存庫的名稱。 您不需要記住儲存復原點的復原服務保存庫名稱，您可以從虛擬機器存取這項資訊。  

如果一個復原服務保存庫保護著多部伺服器，則查看復原服務保存庫可能更具邏輯。 您可以搜尋訂用帳戶中所有的復原服務保存庫，並從清單中選擇一個。

下列各節所包含的文章連結，說明如何在每一個活動類型中使用復原服務保存庫。

### <a name="back-up-data"></a>備份資料
- [備份 Azure VM](backup-azure-vms-first-look-arm.md)
- [備份 Windows Server 或 Windows 工作站](backup-try-azure-backup-in-10-mins.md)
- [將 DPM 工作負載備份到 Azure](backup-azure-dpm-introduction.md)
- [準備使用 Azure 備份伺服器來備份工作負載](backup-azure-microsoft-azure-backup.md)

### <a name="manage-recovery-points"></a>管理復原點
- [管理 Azure VM 備份](backup-azure-manage-vms.md)
- [管理檔案和資料夾](backup-azure-manage-windows-server.md)

### <a name="restore-data-from-the-vault"></a>從保存庫還原資料
- [復原 Azure VM 中的個別檔案](backup-azure-restore-files-from-vm.md)
- [還原 Azure VM](backup-azure-arm-restore-vms.md)

### <a name="secure-the-vault"></a>保護保存庫
- [保護復原服務保存庫中的雲端備份資料](backup-azure-security-feature.md)



## <a name="next-steps"></a>後續步驟
使用下列文章進行：</br>
[備份 IaaS VM](backup-azure-arm-vms-prepare.md)</br>
[備份 Azure 備份伺服器](backup-azure-microsoft-azure-backup.md)</br>
[備份 Windows Server](backup-configure-vault.md)
