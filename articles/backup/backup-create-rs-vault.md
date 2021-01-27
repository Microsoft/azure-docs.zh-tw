---
title: 建立和設定復原服務保存庫
description: 在本文中，您將瞭解如何建立及設定復原服務保存庫，以儲存備份和復原點。 瞭解如何在次要區域中使用跨區域還原來還原。
ms.topic: conceptual
ms.date: 05/30/2019
ms.custom: references_regions
ms.openlocfilehash: 51d095ab68bc9277cb610bd226039ffbde8d7ddf
ms.sourcegitcommit: 436518116963bd7e81e0217e246c80a9808dc88c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98915721"
---
# <a name="create-and-configure-a-recovery-services-vault"></a>建立和設定復原服務保存庫

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="set-storage-redundancy"></a>設定儲存體備援

Azure 備份會自動處理保存庫的儲存體。 您必須指定該儲存體的複寫方式。

> [!NOTE]
> 在 **保存** 庫中設定備份之前，您必須先完成針對復原服務保存庫變更儲存體複寫類型 (本地區域冗余/地理位置多餘的) 。 設定備份之後，將會停用修改的選項。
>
>- 如果您還沒有設定備份，請 [遵循下列步驟](#set-storage-redundancy) 來檢查和修改設定。
>- 如果您已設定備份，且必須從 GRS 移至 LRS，請 [參閱這些](#how-to-change-from-grs-to-lrs-after-configuring-backup)因應措施。

1. 從 [復原 **服務保存庫** ] 窗格中，選取新的保存庫。 在 [ **設定** ] 區段下，選取 [  **屬性**]。
1. 在 [ **屬性**] 中的 [ **備份** 設定] 下，選取 [ **更新**]。

1. 選取儲存體複寫類型，然後選取 [**儲存]。**

     ![為新保存庫設定儲存體組態](./media/backup-create-rs-vault/recovery-services-vault-backup-configuration.png)

   - 如果您使用 Azure 作為主要的備份儲存體端點，建議您繼續使用預設的 **地理區域冗余** 設定。
   - 如果您未使用 Azure 做為主要的備份儲存體端點，則選擇 [本地備援]，以減少 Azure 儲存體成本。
   - 深入瞭解 [地理](../storage/common/storage-redundancy.md#geo-redundant-storage) 和 [本機](../storage/common/storage-redundancy.md#locally-redundant-storage) 冗余。
   - 如果您需要在區域中沒有停機的資料可用性，請保證資料存放區，然後選擇 [區域多餘的儲存體](../storage/common/storage-redundancy.md#zone-redundant-storage)。

>[!NOTE]
>保存庫的儲存體複寫設定與 Azure 檔案共用備份無關，因為目前的解決方案是以快照集為基礎，而且沒有任何資料傳輸至保存庫。 快照集會儲存在與備份檔案共用相同的儲存體帳戶中。

## <a name="set-cross-region-restore"></a>設定跨區域還原

還原選項 **跨區域還原 (CRR)** 可讓您在次要的 [Azure 配對區域](../best-practices-availability-paired-regions.md)中還原資料。

它支援下列資料來源：

- Azure Vm (正式運作) 
- 裝載于 Azure Vm 上的 SQL 資料庫 (預覽版) 
- Azure Vm 上裝載的 SAP Hana 資料庫 (preview) 

使用跨區域還原可讓您：

- 在有審核或合規性需求時進行演練
- 如果主要區域發生損毀，請還原資料

還原 VM 時，您可以還原 VM 或其磁片。 如果您要從裝載于 Azure Vm 上的 SQL/SAP Hana 資料庫還原，則可以還原資料庫或其檔案。

若要選擇這項功能，請選取 [**備份** 設定] 窗格中的 [**啟用跨區域還原**]。

因為此程式位於儲存體層級，所以會 [影響價格](https://azure.microsoft.com/pricing/details/backup/)。

>[!NOTE]
>開始之前：
>
>- 請參閱 [支援矩陣](backup-support-matrix.md#cross-region-restore) ，以取得支援的 managed 類型和區域清單。
>- Azure Vm 的跨區域還原 (的 CRR) 功能現已在所有 Azure 公用區域正式推出。
>- 適用于 SQL 和 SAP Hana 資料庫的跨區域還原在所有 Azure 公用區域都處於預覽狀態。
>- 針對任何 GRS 保存庫，CRR 都是保存庫層級的加入宣告功能， (預設關閉) 。
>- 加入宣告後，最多可能需要48小時的時間，才能在次要區域中使用備份專案。
>- 目前只有 azure Resource manager Azure Vm 支援 Azure Vm 的 CRR。 不支援傳統 Azure Vm。  當其他管理類型支援 CRR 時，系統就會 **自動** 註冊它們。
>- 當第一次起始保護時， **無法將** 跨區域還原還原回 GRS 或 LRS。

### <a name="configure-cross-region-restore"></a>設定跨區域還原

使用 GRS 冗余建立的保存庫包含設定跨區域還原功能的選項。 每個 GRS 保存庫都會有橫幅，其會連結至檔。 若要設定保存庫的 CRR，請移至 [備份設定] 窗格，其中包含啟用這項功能的選項。

 ![備份設定橫幅](./media/backup-azure-arm-restore-vms/banner.png)

1. 從入口網站中，移至您的復原服務保存 **庫 > [** **設定** ]) 下的 [內容] (。
1. 在 [ **備份** 設定] 下，選取 [ **更新**]。
1. 選取 [ **在此保存庫中啟用跨區域還原** ] 以啟用此功能。

   ![啟用跨區域還原](./media/backup-azure-arm-restore-vms/backup-configuration.png)

請參閱下列文章，以取得使用 CRR 進行備份和還原的詳細資訊：

- [Azure Vm 的跨區域還原](backup-azure-arm-restore-vms.md#cross-region-restore)
- [SQL 資料庫的跨區域還原](restore-sql-database-azure-vm.md#cross-region-restore)
- [SAP Hana 資料庫的跨區域還原](sap-hana-db-restore.md#cross-region-restore)

## <a name="set-encryption-settings"></a>設定加密設定

根據預設，復原服務保存庫中的資料會使用平臺管理的金鑰進行加密。 您的端不需要明確的動作就能啟用此加密，並且會套用到所有備份到復原服務保存庫的工作負載。  您可以選擇攜帶您自己的金鑰，將此保存庫中的備份資料加密。 這稱為客戶管理的金鑰。 如果您想要使用您自己的金鑰來加密備份資料，則必須在此保存庫保護任何專案之前，指定加密金鑰。 一旦您使用金鑰啟用加密，就無法反轉。

### <a name="configuring-a-vault-to-encrypt-using-customer-managed-keys"></a>將保存庫設定為使用客戶管理的金鑰進行加密

若要將您的保存庫設定為使用客戶管理的金鑰進行加密，必須依照下列順序遵循這些步驟：

1. 啟用復原服務保存庫的受控識別

1. 將許可權指派給保存庫，以存取 Azure Key Vault 中的加密金鑰

1. 啟用 Azure Key Vault 的虛刪除和清除保護

1. 將加密金鑰指派給復原服務保存庫

您可以 [在本文中](encryption-at-rest-with-cmk.md#configuring-a-vault-to-encrypt-using-customer-managed-keys)找到每個步驟的指示。

## <a name="modifying-default-settings"></a>修改預設設定

強烈建議您先檢閱 [儲存體複寫類型] 和 [安全性設定] 的預設設定，再於保存庫中設定備份。

- 依預設，**儲存體複寫類型** 會設定為 **異地冗余** (GRS) 。 設定備份之後，[修改] 選項就會停用。
  - 如果您還沒有設定備份，請 [遵循下列步驟](#set-storage-redundancy) 來檢查和修改設定。
  - 如果您已設定備份，且必須從 GRS 移至 LRS，請 [參閱這些](#how-to-change-from-grs-to-lrs-after-configuring-backup)因應措施。

- 預設會在新建立的保存庫上 **啟用** 虛 **刪除**，以防止備份資料遭到意外或惡意刪除。 [請遵循下列步驟](./backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) 來檢查和修改設定。

### <a name="how-to-change-from-grs-to-lrs-after-configuring-backup"></a>設定備份之後如何從 GRS 變更為 LRS

在決定從 GRS 移至本機冗余儲存體 (LRS) 之前，請先查看符合您案例的低成本和更高資料耐久性之間的取捨。 如果您必須從 GRS 移至 LRS，您有兩個選擇。 它們取決於您的業務需求，以保留備份資料：

- [不需要保留先前的備份資料](#dont-need-to-preserve-previous-backed-up-data)
- [必須保留先前的備份資料](#must-preserve-previous-backed-up-data)

#### <a name="dont-need-to-preserve-previous-backed-up-data"></a>不需要保留先前的備份資料

若要保護新 LRS 保存庫中的工作負載，您必須在 GRS 保存庫中刪除目前的保護和資料，然後再重新設定備份。

>[!WARNING]
>下列作業是破壞性的作業，無法復原。 所有與受保護伺服器相關聯的備份資料和備份專案都會永久刪除。 請謹慎進行。

停止並刪除 GRS 保存庫上的目前保護：

1. 停用 GRS 保存庫屬性中的虛刪除。 請遵循下列 [步驟](backup-azure-security-feature-cloud.md#disabling-soft-delete-using-azure-portal) 來停用虛刪除。

1. 停止保護並刪除現有 GRS 保存庫中的備份。 在保存庫儀表板功能表中，選取 [ **備份專案**]。 此處所列的專案必須移至 LRS 保存庫，並與其備份資料一起移除。 瞭解如何 [在雲端中刪除受保護的專案](backup-azure-delete-vault.md#delete-protected-items-in-the-cloud) ，以及 [刪除內部部署的受保護專案](backup-azure-delete-vault.md#delete-protected-items-on-premises)。

1. 如果您打算將 AFS (Azure 檔案共用) 、SQL server 或 SAP Hana 伺服器，則您也需要將其取消註冊。 在保存庫儀表板功能表中，選取 [ **備份基礎結構**]。 瞭解如何 [取消註冊 SQL server](manage-monitor-sql-database-backup.md#unregister-a-sql-server-instance)、 [取消登錄與 Azure 檔案共用相關聯的儲存體帳戶](manage-afs-backup.md#unregister-a-storage-account)，以及 [取消註冊 SAP Hana 實例](sap-hana-db-manage.md#unregister-an-sap-hana-instance)。

1. 從 GRS 保存庫中移除之後，請繼續在新的 LRS 保存庫中設定工作負載的備份。

#### <a name="must-preserve-previous-backed-up-data"></a>必須保留先前的備份資料

如果您需要將目前受保護的資料保留在 GRS 保存庫中，並在新的 LRS 保存庫中繼續保護，則某些工作負載的選項有限：

- 針對 MARS，您可以 [停止保護保留資料](backup-azure-manage-mars.md#stop-protecting-files-and-folder-backup) ，並在新的 LRS 保存庫中註冊代理程式。

  - Azure 備份服務會繼續保留 GRS 保存庫的所有現有復原點。
  - 您必須付費，才能將復原點保留在 GRS 保存庫中。
  - 您只可以針對 GRS 保存庫中未到期的復原點還原已備份的資料。
  - 您必須在 LRS 保存庫上建立資料的新初始複本。

- 針對 Azure VM，您可以停止保護 GRS 保存庫中的 VM [保留資料](backup-azure-manage-vms.md#stop-protecting-a-vm) 、將 vm 移至另一個資源群組，然後在 LRS 保存庫中保護 vm。 請參閱將 VM 移至另一個資源群組的 [指導方針和限制](../azure-resource-manager/management/move-limitations/virtual-machines-move-limitations.md) 。

  虛擬機器一次只能在一個保存庫中受到保護。 不過，您可以在 LRS 保存庫中保護新資源群組中的 VM，因為它被視為不同的 VM。

  - Azure 備份服務會保留已在 GRS 保存庫上備份的復原點。
  - 您必須付費，才能將復原點保留在 GRS 保存庫 (如需詳細資料) ，請參閱 [Azure 備份定價](azure-backup-pricing.md) 。
  - 您將能夠從 GRS 保存庫還原 VM （如有需要）。
  - 在新資源中，VM LRS 保存庫上的第一個備份會是初始複本。

## <a name="next-steps"></a>後續步驟

[深入瞭解](backup-azure-recovery-services-vault-overview.md) 復原服務保存庫。
[深入瞭解](backup-azure-delete-vault.md) 刪除復原服務保存庫。
