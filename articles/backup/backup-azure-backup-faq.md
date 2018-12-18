---
title: Azure 備份常見問題集
description: '有關以下常見問題的解答：包括復原服務保存庫、可以備份的項目、其運作方式、加密和限制等 Azure 備份功能。 '
services: backup
author: markgalioto
manager: carmonm
keywords: 備份和災害復原; 備份服務
ms.service: backup
ms.topic: conceptual
ms.date: 8/2/2018
ms.author: markgal
ms.openlocfilehash: 5fd0cb92bd35b1f238e4080d2c9e8caf781b8131
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39493863"
---
# <a name="questions-about-the-azure-backup-service"></a>關於 Azure 備份服務的問題
本文提供 Azure 備份元件的常見問題解答。 在某些答案中，有具有完整資訊的文章連結。 按一下 [註解] \(位於右側)，即可詢問 Azure 備份的相關問題。 註解會出現於文末。 需有 Livefyre 帳戶才能使用註解。 您也可以在 [論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazureonlinebackup)中張貼有關 Azure 備份服務的問題。

若要快速掃描本文中的各節內容，在**本文**下使用右方連結。


## <a name="recovery-services-vault"></a>復原服務保存庫

### <a name="is-there-any-limit-on-the-number-of-vaults-that-can-be-created-in-each-azure-subscription-br"></a>每個 Azure 訂用帳戶中可以建立的保存庫數目是否有任何限制？ <br/>
是。 您可以為每個訂用帳戶的 Azure 備份支援區域，最多建立 500 個復原服務保存庫。 如果您需要其他保存庫，請建立其他訂用帳戶。

### <a name="are-there-limits-on-the-number-of-serversmachines-that-can-be-registered-against-each-vault-br"></a>針對每個保存庫註冊的伺服器/電腦具有數目限制嗎？ <br/>
每個保存庫可以註冊最多 1000 部 Azure 虛擬機器。 如果您使用的是 MAB 代理程式，則每個保存庫可以註冊最多 50 個 MAB 代理程式。 而且您可以將 50 部 MAB 伺服器/DPM 伺服器註冊至保存庫。

### <a name="can-i-use-a-rest-api-to-query-the-size-of-protected-items-in-a-vault-br"></a>可以使用 REST API 來查詢保存庫中受保護項目的大小嗎？ <br/>
是，[使用方式 - 依保存庫列出](https://t.co/2lgIrIaF0J)一文列出您可以從復原服務保存庫取得的資訊。

### <a name="if-my-organization-has-one-vault-how-can-i-isolate-one-servers-data-from-another-server-when-restoring-databr"></a>如果我的組織有一個備份保存庫，如何在還原資料時隔離某一部伺服器與另一部伺服器的資料？<br/>
所有向相同保存庫註冊的伺服器都可以復原由「使用相同複雜密碼」 之其他伺服器所備份的資料。 如果您想要隔離您伺服器中的備份資料與您組織中的其他伺服器，請使用這些伺服器的指定複雜密碼。 例如，人力資源伺服器可能使用一組加密複雜密碼，而會計伺服器使用另一組，並且儲存體伺服器使用第三組。

### <a name="can-i-migrate-my-vault-between-subscriptions-br"></a>我是否可以在訂用帳戶之間移轉保存庫？ <br/>
否。 保存庫是於訂用帳戶層級上建立，且無法重新指派至其他訂用帳戶。

### <a name="can-i-migrate-backup-data-to-another-vault-br"></a>我是否可以將備份資料移轉至另一個保存庫？ <br/>
否。 儲存在保存庫中的備份資料無法移至不同的保存庫。

### <a name="recovery-services-vaults-are-resource-manager-based-are-backup-vaults-still-supported-br"></a>復原服務保存庫是以 Resource Manager 為基礎。 仍然支援備份保存庫嗎？ <br/>
備份保存庫都已轉換為復原服務保存庫。 如果您未將備份保存庫轉換為復原服務保存庫，則系統會為您將備份保存庫轉換為復原服務保存庫。 

### <a name="can-i-migrate-a-backup-vault-to-a-recovery-services-vault-br"></a>是否可以將備份保存庫移轉至復原服務保存庫？ <br/>
所有備份保存庫都已轉換為復原服務保存庫。 如果您未將備份保存庫轉換為復原服務保存庫，則系統會為您將備份保存庫轉換為復原服務保存庫。

## <a name="azure-backup-agent"></a>Azure 備份代理程式
問題的詳細清單會出現在 [Azure 檔案資料夾備份的常見問題集](backup-azure-file-folder-backup-faq.md)

## <a name="azure-vm-backup"></a>Azure VM 備份
問題的詳細清單會出現在 [Azure VM 備份的常見問題集](backup-azure-vm-backup-faq.md)

## <a name="back-up-vmware-servers"></a>備份到 VMware 伺服器

### <a name="can-i-back-up-vmware-vcenter-servers-to-azure"></a>我可以將 VMware vCenter 伺服器備份到 Azure 嗎？

是。 您可以使用 Azure 備份伺服器，將 VMware vCenter 和 ESXi 備份至 Azure。 如需支援 VMware 版本的資訊，請參閱 [Azure 備份伺服器保護矩陣](backup-mabs-protection-matrix.md)一文。 如需逐步指示，請參閱[使用 Azure 備份伺服器來備份 VMware 伺服器](backup-azure-backup-server-vmware.md)。

### <a name="do-i-need-a-separate-license-to-recover-a-full-on-premises-vmwarehyper-v-cluster-from-dpm-or-azure-backup-serverbr"></a>我需要個別授權，才能從 DPM 或 Azure 備份伺服器復原完整的內部部署 VMware/Hyper-V 叢集嗎？<br/>
您不需要針對 VMware/HYPER-V 保護進行個別授權。 如果您是 System Center 客戶，請使用 DPM 來保護 VMware VM。 如果您不是 System Center 客戶，您可使用 Azure 備份伺服器 (預付型方案) 來保護 VMware VM。

## <a name="azure-backup-server-and-system-center-data-protection-manager"></a>Azure 備份伺服器和 System Center Data Protection Manager
### <a name="can-i-use-azure-backup-server-to-create-a-bare-metal-recovery-bmr-backup-for-a-physical-server-br"></a>可以使用 Azure 備份伺服器來為實體伺服器建立裸機復原 (BMR) 備份嗎？ <br/>
是。

### <a name="can-i-register-my-dpm-server-to-multiple-vaults-br"></a>可以將我的 DPM 伺服器註冊至多個保存庫嗎？ <br/>
否。 DPM 或 MABS 伺服器只能註冊至一個保存庫。

### <a name="which-version-of-system-center-data-protection-manager-is-supported"></a>支援的 System Center Data Protection Manager 版本為何？

我們建議您在 System Center Data Protection Manager (DPM) 的最新更新彙總套件 (UR) 上安裝 [最新](http://aka.ms/azurebackup_agent) 的 Azure 備份代理程式。 
- 針對 System Center DPM 2012 R2，最新更新為[更新彙總套件 14](https://support.microsoft.com/help/4043315/update-rollup-14-for-system-center-2012-r2-data-protection-manager)。
- 針對 System Center DPM 2016，最新更新為[更新彙總套件 2](https://support.microsoft.com/en-us/help/3209593)。

### <a name="i-have-installed-azure-backup-agent-to-protect-my-files-and-folders-can-i-install-system-center-dpm-to-protect-on-premises-applicationvm-workloads-to-azure"></a>我已安裝 Azure 備份代理程式來保護我的檔案和資料夾。 是否可以安裝 System Center DPM 來保護內部部署應用程式/Azure 的 VM 工作負載？

是。 不過，若要搭配 System Center Data Protection Manager (DPM) 使用 Azure 備份，請先安裝 DPM，然後安裝 Azure 備份代理程式。 依此順序安裝 Azure 備份元件，可確保 Azure 備份代理程式能搭配 DPM 運作。 不建議或支援在安裝 DPM 之前安裝 Azure 備份代理程式。

### <a name="can-i-use-dpm-to-back-up-apps-in-azure-stack"></a>可以使用 DPM 備份 Azure Stack 中的應用程式嗎？

否。 雖然您可以使用 Azure 備份來保護 Azure Stack，但 Azure 備份目前不支援使用 DPM 來備份 Azure Stack 中的應用程式。

## <a name="how-azure-backup-works"></a>Azure 備份的運作方式
### <a name="if-i-cancel-a-backup-job-once-it-has-started-is-the-transferred-backup-data-deleted-br"></a>如果我在備份作業開始後予以取消，是否會刪除已傳輸的備份資料？ <br/>
否。 在備份作業取消之前﹐傳輸到保存庫的所有資料均保留於保存庫中。 Azure 備份會使用檢查點機制，在備份期間偶爾將檢查點加入至備份資料。 因為備份資料中有檢查點，所以下一個備份程序才可驗證檔案的完整性。 下一個備份工作將會增量到先前備份的資料。 增量備份只會傳輸新資料或變更的資料，相當於具有較佳的頻寬使用率。

如果您取消 Azure VM 的備份工作，則會忽略任何傳輸的資料。 下一個備份工作會傳輸自最後一個成功備份工作之後的增量資料。

### <a name="are-there-limits-on-when-or-how-many-times-a-backup-job-can-be-scheduledbr"></a>備份作業可排程的時間和次數是否有限制？<br/>
是。 您一天可以在 Windows Server 或 Windows 工作站上執行備份作業最多三次。 您一天可以在 System Center DPM 上執行備份作業最多兩次。 您一天可以執行 IaaS VM 的備份作業一次。 若要指定每日或每週排程，可以使用 Windows Server 或 Windows 工作站的排程原則。 使用 System Center DPM 時，您可以指定每日、每週、每月和每年排程。

### <a name="why-is-the-size-of-the-data-transferred-to-the-recovery-services-vault-smaller-than-the-data-i-backed-upbr"></a>為何傳輸到復原服務保存庫的資料大小會小於我備份的資料？<br/>
 從 Azure 備份代理程式、SCDPM 或 Azure 備份伺服器備份的所有資料都會在傳輸之前，經過壓縮和加密。 一旦套用壓縮和加密，復原服務保存庫中的資料會縮小 30-40%。

### <a name="can-i-delete-individual-files-from-a-recovery-point-in-the-vaultbr"></a>可以從保存庫中的復原點刪除個別的檔案嗎？<br/>
否，Azure 備份不支援從儲存的備份中刪除或清除個別項目。

## <a name="what-can-i-back-up"></a>我可以備份什麼
### <a name="which-operating-systems-does-azure-backup-support-br"></a>Azure 備份支援哪些作業系統？ <br/>
Azure 備份支援下列作業系統清單來備份：使用 Azure 備份伺服器和 System Center Data Protection Manager (DPM) 保護的檔案、資料夾及工作負載應用程式。

| 作業系統 | 平台 | SKU |
|:--- | --- |:--- |
| Windows 8 和最新的 SP |64 位元 |Enterprise、Pro |
| Windows 7 和最新的 SP |64 位元 |Ultimate、Enterprise、Professional、Home Premium、Home Basic、Starter |
| Windows 8.1 和最新的 SP |64 位元 |Enterprise、Pro |
| Windows 10 |64 位元 |企業版、專業版、家用版 |
| Windows Server 2016 |64 位元 |Standard、Datacenter、Essentials |
| Windows Server 2012 R2 和最新的 SP |64 位元 |Standard、Datacenter、Foundation |
| Windows Server 2012 和最新的 SP |64 位元 |Datacenter、Foundation、Standard |
| Windows Storage Server 2016 和最新的 SP |64 位元 |Standard、Workgroup | 
| Windows Storage Server 2012 R2 和最新的 SP |64 位元 |Standard、Workgroup |
| Windows Storage Server 2012 和最新的 SP |64 位元 |Standard、Workgroup |
| Windows Server 2012 R2 和最新的 SP |64 位元 |Essential |
| Windows Server 2008 R2 SP1 |64 位元 |Standard、Enterprise、Datacenter、Foundation |

**Azure VM 備份：**

* **Linux**：Azure 備份支援 [Azure 所背書的散發套件清單](../virtual-machines/linux/endorsed-distros.md) ，但核心作業系統 Linux 除外。  只要虛擬機器上有 VM 代理程式並且可支援 Python，其他「攜帶您自己的 Linux」散發套件可能也可以運作。
* **Windows Server**：不支援比 Windows Server 2008 R2 更舊的版本。


### <a name="is-there-a-limit-on-the-size-of-each-data-source-being-backed-up-br"></a>正在備份的每個資料來源是否有大小限制？ <br/>
Azure 備份會強制執行資料來源的大小上限，不過，來源的限額很大。 截至 2015 年 8 月，適用於支援作業系統的資料來源大小上限為：

| S.No | 作業系統 | 資料來源的大小上限 |
|:---:|:--- |:--- |
| 1 |Windows Server 2012 或更新版本 |54,400 GB |
| 2 |Windows 8 或更新版本 |54,400 GB |
| 3 |Windows Server 2008、Windows Server 2008 R2 |1700 GB |
| 4 |Windows 7 |1700 GB |

下表說明如何決定每個資料來源大小。

| 資料來源 | 詳細資料 |
|:---:|:--- |
| 磁碟區 |從伺服器或用戶端電腦的單一磁碟區備份的資料量。 |
| Hyper-V 虛擬機器 |要備份的虛擬機器上所有 VHD 的資料總和 |
| Microsoft SQL Server 資料庫 |要備份的單一 SQL 資料庫大小的大小 |
| Microsoft SharePoint |在要備份的 SharePoint 伺服陣列中內容和組態資料庫的總和 |
| Microsoft Exchange |在要備份的 Exchange Server 中所有 Exchange 資料庫的總和 |
| BMR/系統狀態 |在要備份的電腦上 BMR 或系統狀態的每個個別複本 |

如果是 Azure IaaS VM 備份，每個 VM 最多可有 16 個資料磁碟，每個資料磁碟最多為 4095 GB。

### <a name="is-there-a-limit-on-the-amount-of-data-held-in-a-recovery-services-vault"></a>復原服務保存庫容納的資料量是否有限制？
您可備份到復原服務保存庫的資料量並無任何限制。

## <a name="retention-policy-and-recovery-points"></a>保留原則和復原點
### <a name="is-there-a-difference-between-the-retention-policy-for-dpm-and-windows-serverclient-that-is-on-windows-server-without-dpmbr"></a>DPM 和 Windows Server/用戶端 (亦即在不含 DPM 的 Windows Server 上) 的保留原則是否具有差異？<br/>
否，DPM 和 Windows Server/用戶端均有每日、每週、每月和每年保留原則。

### <a name="can-i-configure-my-retention-policies-selectively--that-is-configure-weekly-and-daily-but-not-yearly-and-monthlybr"></a>我可以選擇性設定保留原則嗎 (例如設定每週和每日，但不包含每年和每月)？<br/>
是的，Azure 備份保留結構讓您可以依照您的需求，彈性定義保留原則。

### <a name="can-i-schedule-a-backup-at-6pm-and-specify-retention-policies-at-a-different-timebr"></a>我可以在下午 6:00「排程備份」，並在不同的時間指定「保留原則」嗎？<br/>
否。 保留原則僅能套用在復原點上。 在以下影像中，將於上午 12:00 和下午 6:00 進行備份時指定保留原則。 <br/>

![排程備份和保留](./media/backup-azure-backup-faq/Schedule.png)
<br/>

### <a name="if-a-backup-is-retained-for-a-long-duration-does-it-take-more-time-to-recover-an-older-data-point-br"></a>如果備份保留了很長一段時間，是否需要更多時間才能復原較舊的資料點？ <br/>
否，復原最舊或最新時間點所需的時間都相同。 每個復原點的功能就像一個完整的復原點。

### <a name="if-each-recovery-point-is-like-a-full-point-does-it-impact-the-total-billable-backup-storagebr"></a>若每個復原點就像一個完整的復原點，則其是否會影響可計費的備份儲存體總數？<br/>
典型的長期保留復原點產品會將備份資料儲存為完整的復原點。 完整的復原點是「效率不佳的」  儲存體，但可以更輕鬆且更快速地進行還原。 增量複本是「有效率的」  儲存體，但會要求您還原一連串的資料，而這會影響復原時間。 Azure 備份的儲存體架構透過最佳化儲存資料以進行快速還原，並降低儲存體成本支出，可讓您魚與熊掌兼得。 此資料儲存方法可確保有效率地使用您的輸入和輸出頻寬。 資料儲存體數量和復原資料所需的時間都會降到最低。 深入了解[增量備份](https://azure.microsoft.com/blog/microsoft-azure-backup-save-on-long-term-storage/)有何效用。

### <a name="is-there-a-limit-on-the-number-of-recovery-points-that-can-be-createdbr"></a>可建立的復原點數目有限制嗎？<br/>
您可以為每個受保護的執行個體最多建立 9999 個復原點。 受保護的執行個體可以是電腦、(實體或虛擬) 伺服器，或設定為將資料備份至 Azure 的工作負載。 如需詳細資訊，請參閱[備份和保留](./backup-introduction-to-azure-backup.md#backup-and-retention)和[什麼是受保護的執行個體？](./backup-introduction-to-azure-backup.md#what-is-a-protected-instance)

### <a name="how-many-recoveries-can-i-perform-on-the-data-that-is-backed-up-to-azurebr"></a>我可以在備份至 Azure 的資料上執行幾次復原？<br/>
從 Azure 備份進行復原的次數沒有任何限制。

### <a name="when-restoring-data-do-i-pay-for-the-egress-traffic-from-azure-br"></a>還原資料時，我需要支付來自 Azure 的輸出流量嗎？ <br/>
否。 您的復原作業完全免費，且輸出流量不計費。

### <a name="what-happens-when-i-change-my-backup-policy"></a>變更我的備份原則時會發生什麼狀況？
套用新原則後，就會遵循新原則的排程和保留期。 如果延長保留期，會標示現有的復原點，以依據新的原則加以保留。 如果縮短保留期，則會標示現有的復原點，以便在下次清除作業中剪除然後刪除。

## <a name="azure-backup-encryption"></a>Azure 備份加密
### <a name="is-the-data-sent-to-azure-encrypted-br"></a>傳送至 Azure 的資料會經過加密嗎？ <br/>
是。 資料會使用 AES256 在內部部署伺服器/用戶端/SCDPM 電腦上進行加密，且資料會透過安全的 HTTPS 連結進行傳送。

### <a name="is-the-backup-data-on-azure-encrypted-as-wellbr"></a>位於 Azure 的備份資料也會經過加密嗎？<br/>
是。 傳送至 Azure 的資料仍會維持加密 (靜止) 狀態。 Microsoft 不會解密在任何時間點所備份的資料。 備份 Azure VM 時，Azure 備份會依賴虛擬機器的加密。 例如，如果使用 Azure 磁碟加密或一些其他加密技術來加密您的 VM，Azure 備份會使用該加密來保護您的資料。

### <a name="what-is-the-minimum-length-of-encryption-key-used-to-encrypt-backup-data-br"></a>什麼是用來加密備份資料之加密金鑰的最小長度？ <br/>
當您使用 Azure 備份代理程式時，加密金鑰應該至少包含 16 個字元。 在 Azure VM 中，Azure KeyVault 所使用的金鑰長度沒有限制。 

### <a name="what-happens-if-i-misplace-the-encryption-key-can-i-recover-the-data-or-can-microsoft-recover-the-data-br"></a>如果我錯置加密金鑰，則會發生什麼情況？ 我可以復原資料 (或者) Microsoft 可以復原資料嗎？ <br/>
用來加密備份資料的金鑰僅存在於客戶組織內部。 Microsoft 不會維護 Azure 中的複本，且無法存取金鑰。 若客戶錯置金鑰，則 Microsoft 無法復原備份資料。
