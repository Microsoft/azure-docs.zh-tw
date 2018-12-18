---
title: 何謂 Azure 備份？
description: 使用 Azure 備份，從 Windows Server、Windows 工作站、System Center DPM 伺服器及 Azure 虛擬機器，備份和還原資料與工作負載。
services: backup
author: markgalioto
manager: carmonm
keywords: 備份與還原；復原服務；備份解決方案
ms.service: backup
ms.topic: overview
ms.date: 8/2/2018
ms.author: markgal
ms.custom: mvc
ms.openlocfilehash: 0a5b9e6cdb5329705cb3c6d4676dfc8d987119e4
ms.sourcegitcommit: fc5555a0250e3ef4914b077e017d30185b4a27e6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39480968"
---
# <a name="overview-of-the-features-in-azure-backup"></a>Azure 備份中的功能概觀
Azure 備份是您可用來備份 (或保護) 和還原 Microsoft Cloud 資料的 Azure 服務。 Azure 備份將以一個可靠、安全及具成本競爭力的雲端架構解決方案，取代您現有的內部部署或異地備份解決方案。 Azure 備份提供多個元件，您可以下載並部署在適當的電腦、伺服器或雲端中。 您部署的元件或代理程式，取決於您想要保護的項目。 所有 Azure 備份的元件 (無論您要保護的是內部部署或雲端資料) 都可以將資料備份至 Azure 中的復原服務保存庫。 請參閱 [Azure 備份元件資料表](backup-introduction-to-azure-backup.md#which-azure-backup-components-should-i-use) (稍後於本文提及) 以取得該使用哪個元件來保護特定資料、應用程式或工作負載的資訊。

[觀看 Azure 備份的影片概觀](https://azure.microsoft.com/documentation/videos/what-is-azure-backup/)

## <a name="why-use-azure-backup"></a>為何使用 Azure 備份？
傳統的備份解決方案已演變成將雲端視為端點或靜態儲存體目的地，類似於磁碟或磁帶。 此方法雖然簡單，但卻受到限制並且未能充分利用基礎的雲端平台。換句話說，是昂貴且沒有效率的解決方案。 其他解決方案很昂貴的原因，是因為您最後花錢購買的是錯誤的儲存體類型或不需要的儲存體。 通常其他解決方案效率不佳，是因為並未提供您所需的儲存體類型或數量，或系統管理工作需要太多時間。 相反地，Azure 備份則提供下列主要優點：

**自動儲存管理** - 混合式環境通常需要異質性儲存體 - 部份在內部部署，部份在雲端。 使用 Azure 備份，使用內部部署儲存體裝置無需成本。 Azure 備份會自動配置和管理備份儲存體，且採用使用時付費制。 使用時付費表示您只需為有用到的儲存體付費。 如需詳細資訊，請參閱 [Azure 定價文章](https://azure.microsoft.com/pricing/details/backup)。

**無限制的調整** - Azure 備份使用 Azure 雲端的基礎功能及無限制調整來提供高可用性 - 沒有維護或監控的額外負荷。 您可以設定提供相關事件資訊的警示，但您不必擔心雲端資料的高可用性問題。

**多個儲存體選項** - 高可用性的其中一環是儲存體複寫。 Azure 備份提供兩種複寫類型：[本地備援儲存體](../storage/common/storage-redundancy-lrs.md)和[異地備援儲存體](../storage/common/storage-redundancy-grs.md)。 可根據需求選擇備份儲存體：

* 本地備援儲存體 (LRS) 會將資料複寫至資料中心的儲存體縮放單位三次 (建立三個資料複本)。 此資料的所有複本都存在於相同的區域內。 LRS 是保護資料免於本機硬體失敗的低成本選項。

* 異地備援儲存體 (GRS) 是預設且建議的複寫選項。 GRS 會將資料複寫到次要地區 (與來源資料主要位置距離數百英哩)。 GRS 的價格高於 LRS，但它為您的資料提供更高層級的持久性，即使遭受區域性停電也不影響。

**無限制的資料傳輸** -Azure 備份不會限制輸入或輸出資料的傳輸。 Azure 備份也不會對傳輸資料收取費用。 不過，如果您使用 Azure 匯入/匯出服務匯入大量的資料，會有與輸入資料關聯的費用。 如需這項成本的詳細資訊，請參閱 [Azure Backup 中的離線備份工作流程](backup-azure-backup-import-export.md)。 輸出資料是指還原作業期間傳輸自復原服務保存庫的資料。

**資料加密** - 資料加密可安全傳輸與儲存您在公用雲端中的資料。 加密複雜密碼會儲存在本機，並且永遠不會傳送至或儲存在 Azure 中。 如果需要還原任何資料，只有您有加密複雜密碼或金鑰。

**應用程式一致備份** - 應用程式一致備份表示復原點具有所有必要的資料來還原備份複本。 Azure 備份提供應用程式一致備份，確保資料還原不需要其他修正程式。 還原應用程式一致的資料會減少還原時間，讓您能夠快速回到執行狀態。

**長期保留** - 您可以使用復原服務保存庫進行短期和長期資料保留。 Azure 不會限制您可在復原服務保存庫中保留資料的時間長度。 只要您喜歡，就可以將資料保留在保存庫中。 Azure 備份的每個受保護執行個體上限為 9999 個復原點。 請參閱本文的[備份和保留](backup-introduction-to-azure-backup.md#backup-and-retention)一節，以了解這項限制對您的備份需求有何影響。

## <a name="which-azure-backup-components-should-i-use"></a>我該使用哪一個 Azure 備份元件？
請使用下表，了解每個 Azure 備份元件的防護相關資訊。 

| 元件 | 優點 | 限制 | 什麼會受到保護？ | 備份會儲存在哪裡？ |
| --- | --- | --- | --- | --- |
| Azure 備份 (MARS) 代理程式 |<li>備份的檔案和資料夾儲存在實體或虛擬 Windows OS (VM 可以位於內部部署或 Azure 中)<li>不需任何個別的備份伺服器。 |<li>每天備份 3 次 <li>不會感知應用程式；僅有檔案、資料夾和磁碟區層級還原， <li>  不支援 Linux。 |<li>檔案、 <li>資料夾、 <li>系統狀態 |復原服務保存庫 |
| System Center DPM |<li>應用程式感知快照集 (VSS)<li>具有隨時備份的彈性<li>復原細微度 (全部)<li>可以使用復原服務保存庫<li>Hyper-V 和 VMware VM 的 Linux 支援 <li>使用 DPM 2012 R2 備份並還原 VMware VM |無法備份 Oracle 工作負載。|<li>檔案、 <li>資料夾、<li> 磁碟區、 <li>VM、<li> 應用程式、<li> 工作負載 <li>系統狀態 |<li>復原服務保存庫、<li> 本機連接的磁碟、<li>  磁帶 (僅限內部部署) |
| Azure 備份伺服器 |<li>應用程式感知快照集 (VSS)<li>具有隨時備份的彈性<li>復原細微度 (全部)<li>可以使用復原服務保存庫<li>Hyper-V 和 VMware VM 的 Linux 支援<li>備份和還原 VMware VM <li>不需要 System Center 授權 |<li>無法備份 Oracle 工作負載。<li>一律需要即時 Azure 訂用帳戶<li>不支援磁帶備份 |<li>檔案、 <li>資料夾、<li> 磁碟區、 <li>VM、<li> 應用程式、<li> 工作負載、 <li>系統狀態 |<li>復原服務保存庫、<li> 本機連接的磁碟 |
| Azure IaaS VM 備份 |<li>應用程式感知快照集 (VSS)<li>適用於 Windows/Linux 的原生備份<li>不需使用特定代理程式安裝<li>不需要備份基礎結構的網狀架構層級備份 |<li>VM 每天備份一次 <li>只在磁碟層級還原 VM<li>無法備份內部部署 |<li>VM、 <li>所有磁碟 (使用 PowerShell) |<p>復原服務保存庫</p> |

## <a name="what-are-the-deployment-scenarios-for-each-component"></a>每個元件的部署案例為何？
| 元件 | 可以在 Azure 中部署嗎？ | 可以在內部部署嗎？ | 支援的目標儲存體 |
| --- | --- | --- | --- |
| Azure 備份 (MARS) 代理程式 |<p>**是**</p> <p>您可以在 Azure 中執行的任何 Windows Server VM 上部署 Azure 備份代理程式。</p> |<p>**是**</p> <p>您可以在 Windows Server VM 或實體機器上部署備份代理程式。</p> |<p>復原服務保存庫</p> |
| System Center DPM |<p>**是**</p><p>深入了解[如何使用 System Center DPM 保護 Azure 中的工作負載](backup-azure-dpm-introduction.md)。</p> |<p>**是**</p> <p>深入了解[如何保護工作負載和資料中心內的 VM](https://technet.microsoft.com/system-center-docs/dpm/data-protection-manager)。</p> |<p>本機連接的磁碟、</p> <p>復原服務保存庫、</p> <p>磁帶 (僅限內部部署)</p> |
| Azure 備份伺服器 |<p>**是**</p><p>深入了解[如何使用 Azure 備份伺服器保護 Azure 中的工作負載](backup-azure-microsoft-azure-backup.md)。</p> |<p>**是**</p> <p>深入了解[如何使用 Azure 備份伺服器保護 Azure 中的工作負載](backup-azure-microsoft-azure-backup.md)。</p> |<p>本機連接的磁碟、</p> <p>復原服務保存庫</p> |
| Azure IaaS VM 備份 |<p>**是**</p><p>Azure 網狀架構的一部分</p><p>適用於[備份 Azure 基礎結構即服務 (IaaS) 虛擬機器](backup-azure-vms-introduction.md)。</p> |<p>**否**</p> <p>請使用 System Center DPM 備份資料中心內的虛擬機器。</p> |<p>復原服務保存庫</p> |

## <a name="which-applications-and-workloads-can-be-backed-up"></a>可以備份哪些應用程式和工作負載？
下表所提供的矩陣，顯示使用 Azure 備份可保護的資料和工作負載。 Azure 備份方案的資料行已連結至該方案的部署文件。 

| 資料或工作負載 | 來源環境 | Azure 備份方案 |
| --- | --- | --- |
| 檔案和資料夾 |Windows Server |<p>[Azure 備份代理程式](backup-configure-vault.md)、</p> <p>[System Center DPM](backup-azure-dpm-introduction.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| 檔案和資料夾 |Windows 電腦 |<p>[Azure 備份代理程式](backup-configure-vault.md)、</p> <p>[System Center DPM](backup-azure-dpm-introduction.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| Hyper-V 虛擬機器 (Windows) |Windows Server |<p>[System Center DPM](backup-azure-backup-sql.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| Hyper-V 虛擬機器 (Linux) |Windows Server |<p>[System Center DPM](backup-azure-backup-sql.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| VMware 虛擬機器 |Windows Server |<p>[System Center DPM](backup-azure-backup-sql.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| 連接字串 |Windows Server |<p>[System Center DPM](backup-azure-backup-sql.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| Microsoft SharePoint |Windows Server |<p>[System Center DPM](backup-azure-backup-sql.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| Microsoft Exchange |Windows Server |<p>[System Center DPM](backup-azure-backup-sql.md) (+ Azure 備份代理程式)、</p> <p>[Azure 備份伺服器](backup-azure-microsoft-azure-backup.md) (包括 Azure 備份代理程式)</p> |
| Azure IaaS VM (Windows) |在 Azure 中執行 |[Azure 備份 (VM 延伸模組)](backup-azure-vms-introduction.md) |
| Azure IaaS VM (Linux) |在 Azure 中執行 |[Azure 備份 (VM 延伸模組)](backup-azure-vms-introduction.md) |

## <a name="linux-support"></a>支援 Linux
下表顯示支援 Linux 的 Azure 備份元件。  

| 元件 | Linux (Azure 背書) 支援 |
| --- | --- |
| Azure 備份 (MARS) 代理程式 |否 (僅限 Windows 代理程式) |
| System Center DPM |<li> Hyper-V 和 VMWare 上 Linux 來賓 VM 的檔案一致性備份<br/> <li> Hyper-V 和 VMWare Linux 來賓 VM 的 VM 還原 </br> </br>  *不適用於 Azure VM 的檔案一致性備份* <br/> |
| Azure 備份伺服器 |<li>Hyper-V 和 VMWare 上 Linux 來賓 VM 的檔案一致性備份<br/> <li> Hyper-V 和 VMWare Linux 來賓 VM 的 VM 還原 </br></br> *不適用於 Azure VM 的檔案一致性備份*  |
| Azure IaaS VM 備份 |使用[前置指令碼和後置指令碼架構](backup-azure-linux-app-consistent.md)的應用程式一致備份<br/> [細微檔案復原](backup-azure-restore-files-from-vm.md)<br/> [還原所有的 VM 磁碟](backup-azure-arm-restore-vms.md#restore-backed-up-disks)<br/> [VM 還原](backup-azure-arm-restore-vms.md#create-a-new-vm-from-a-restore-point) |

## <a name="using-premium-storage-vms-with-azure-backup"></a>使用進階儲存體 VM 與 Azure 備份
Azure 備份可保護進階儲存體 VM。 Azure 進階儲存體是一個以固態硬碟 (SSD) 為基礎的儲存體產品，專門設計用來支援需大量 I/O 的工作負載。 進階儲存體非常適合用於虛擬機器 (VM) 工作負載。 如需有關進階儲存體的詳細資訊，請參閱此文章：[進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../virtual-machines/windows/premium-storage.md)。

### <a name="back-up-premium-storage-vms"></a>備份進階儲存體 VM
在備份進階儲存體 VM 時，備份服務會在進階儲存體帳戶中建立臨時預備位置，名為 "AzureBackup-"。 預備位置的大小等於復原點快照集的大小。 請確定進階儲存體帳戶有適當的可用空間可容納暫存的預備位置。 如需詳細資訊，請參閱[進階儲存體限制](../virtual-machines/windows/premium-storage.md#scalability-and-performance-targets)一文。 備份作業完成後，就會刪除預備位置。 用於預備位置之儲存體的價格在所有 [進階儲存體價格](../virtual-machines/windows/premium-storage.md#pricing-and-billing)中皆一致。

> [!NOTE]
> 請勿修改或編輯預備位置。
>
>

### <a name="restore-premium-storage-vms"></a>還原進階儲存體 VM
您可以將進階儲存體 VM 還原到進階儲存體或標準儲存體。 將進階儲存體 VM 的復原點還原到進階儲存體是常見的程序。 不過，如果您需要 VM 中的部分檔案，將進階儲存體 VM 的復原點還原到標準儲存體比較符合成本效益。

## <a name="using-managed-disk-vms-with-azure-backup"></a>搭配使用受控磁碟 VM 與 Azure 備份
Azure 備份可保護受控磁碟 VM。 受控磁碟可讓您免於管理虛擬機器的儲存體帳戶，並可大幅簡化 VM 佈建。

### <a name="back-up-managed-disk-vms"></a>備份受控磁碟 VM
受控磁碟上的 VM 備份與 Resource Manager VM 備份沒有任何差異。 在 Azure 入口網站中，您可以直接從 [虛擬機器] 檢視或 [復原服務保存庫] 檢視來設定備份工作。 您可以透過以受控磁碟為基礎的 RestorePoint 集合來備份受控磁碟上的 VM。 Azure 備份也支援備份使用 Azure 磁碟加密 (ADE) 加密的受控磁碟 VM。

### <a name="restore-managed-disk-vms"></a>還原受控磁碟 VM
Azure 備份可讓您還原具有受控磁碟的完整 VM﹐或將受控磁碟還原到儲存體帳戶。 Azure 會在還原程序期間管理受控磁碟。 您 (客戶) 管理在還原程序期間建立的儲存體帳戶。 還原受控已加密 VM 時，在開始還原作業之前，VM 的金鑰和祕密應已經存在於 Key Vault 中。

## <a name="what-are-the-features-of-each-backup-component"></a>每個備份元件的功能為何？
下列各節提供每個 Azure 備份元件的可用性，或各種其所支援功能的彙總資料表。 請參閱下列各個資料表，以了解額外支援或詳細資料。

### <a name="storage"></a>儲存體
| 功能 | Azure 備份代理程式 | System Center DPM | Azure 備份伺服器 | Azure IaaS VM 備份 |
| --- | --- | --- | --- | --- |
| 復原服務保存庫 |![是][green] |![yes][green] |![yes][green] |![是][green] |
| 磁碟儲存體 | |![是][green] |![是][green] | |
| 磁帶儲存體 | |![是][green] | | |
| 壓縮 <br/>(在復原服務保存庫中) |![是][green] |![yes][green] |![是][green] | |
| 增量備份 |![是][green] |![yes][green] |![yes][green] |![是][green] |
| 磁碟重複資料刪除 | |![部分][yellow] |![部分][yellow] | | |

![資料表索引鍵](./media/backup-introduction-to-azure-backup/table-key.png)

復原服務保存庫是所有元件慣用的儲存體目標。 System Center DPM 和 Azure 備份伺服器也提供產生本機磁碟複本的選項。 不過，System Center DPM 會提供將資料寫入磁帶儲存裝置的選項。

#### <a name="compression"></a>壓縮
備份會經過壓縮以減少所需的儲存空間。 唯一不進行壓縮的元件為 VM 延伸模組。 VM 延伸模組會將所有備份資料從您的儲存體帳戶複製到相同區域中的復原服務保存庫。 傳輸資料時，不使用任何壓縮。 傳輸資料而不壓縮會需要稍微多一點儲存體空間。 不過，儲存未經壓縮的資料可讓還原時間加快，您需要該復原點。


#### <a name="disk-deduplication"></a>磁碟重複資料刪除
當您在 [HYPER-V 虛擬機器上](http://blogs.technet.com/b/dpm/archive/2015/01/06/deduplication-of-dpm-storage-reduce-dpm-storage-consumption.aspx)部署 System Center DPM 或 Azure 備份伺服器時，可以充分利用重複資料刪除。 Windows Server 重複資料刪除會在當作備份儲存體連接至虛擬機器的虛擬硬碟 (VHD) 上，執行重複資料刪除 (主機層級)。

> [!NOTE]
> 重複資料刪除不適用於任何在 Azure 中的備份元件。 當 System Center DPM 和備份伺服器部署在 Azure 中，連接至 VM 的儲存體磁碟不能進行重複資料刪除。
>
>

### <a name="incremental-backup-explained"></a>增量備份的說明
不論目標儲存體 (磁碟、磁帶、復原服務保存庫) 為何，每個 Azure 備份元件都支援增量備份。 增量備份可確保備份符合儲存和時間效率，因為它只會傳輸自上次備份後所做的變更。

#### <a name="comparing-full-differential-and-incremental-backup"></a>比較完整、差異和增量備份

每一種備份方法的儲存體耗用量、復原時間目標 (RTO) 以及網路耗用量都有所不同。 若要降低備份的擁有權總成本 (TCO)，您需要了解如何選擇最佳的備份解決方案。 下圖比較完整備份、差異備份和增量備份。 在影像中，資料來源 A 由 A1-A10 的 10 個儲存體區塊所組成，每月備份一次。 區塊 A2、A3、A4 和 A9 在第一個月有所變更，而 A5 則在下個月有所變更。

![圖中顯示備份方法的比較](./media/backup-introduction-to-azure-backup/backup-method-comparison.png)

使用「完整備份」，每個備份複本都包含整個資料來源。 完整備份會在每次傳輸備份複本時，耗用大量網路頻寬和儲存體。

「差異備份」只儲存自從初始完整備份以來有所變更的區塊，這樣會使網路和儲存體耗用量較少。 差異備份不會保留未變更資料的備援複本。 不過，由於會傳輸並儲存後續備份之間保持不變的資料區塊，使得差異備份效率不佳。 在第二個月會備份有所變更的區塊 A2、A3、A4 和 A9。 在第三個月會再次備份這一些相同區塊，以及有所變更的區塊 A5。 在執行下次完整備份之前，都會繼續備份有所變更的區塊。

「增量備份」藉由只儲存自從上次備份後有所變更的資料區塊，達到高儲存體和網路效率。 使用增量備份，就不需要定期執行完整備份。 在此範例中，於第一個月執行完整備份之後，區塊 A2、A3、A4、和 A9 會標示為已變更，並移轉至第二個月。 在第三個月，只標示區塊 A5 有所變更，而加以傳輸。 移動較少的資料可節省儲存體和網路資源，進而降低 TCO。

### <a name="security"></a>安全性
| 功能 | Azure 備份代理程式 | System Center DPM | Azure 備份伺服器 | Azure IaaS VM 備份 |
| --- | --- | --- | --- | --- |
| 網路安全性<br/> (至 Azure) |![是][green] |![yes][green] |![yes][green] |![是][green] |
| 資料安全性<br/> (在 Azure 中) |![是][green] |![yes][green] |![yes][green] |![是][green] |

![資料表索引鍵](./media/backup-introduction-to-azure-backup/table-key.png)

#### <a name="network-security"></a>網路安全性
從您的伺服器到復原服務保存庫的所有備份流量，皆使用進階加密標準 256 加密。 備份資料會透過安全的 HTTPS 連結來傳送。 備份資料也會在復原服務保存庫中以加密格式儲存。 只有您 (Azure 的客戶) 持有的複雜密碼可以解除鎖定此資料。 Microsoft 無法解密在任何時間點所備份的資料。

> [!WARNING]
> 您一旦建立復原服務保存庫，只有您能夠存取加密金鑰。 Microsoft 永遠不會持有您加密金鑰的備份，也沒有存取金鑰的權限。 如果錯置金鑰，則 Microsoft 無法復原備份資料。
>
>

#### <a name="data-security"></a>資料安全性
備份 Azure VM 時，需要在虛擬機器「內」  設定加密。 Azure 備份支援 Azure 磁碟加密，其會在 Windows 虛擬機器上使用 BitLocker，而在 Linux 虛擬機器上使用 **dm-crypt**。 在後端，Azure 備份會使用 [Azure 儲存體服務加密](../storage/common/storage-service-encryption.md)，以保護待用資料。

### <a name="network"></a>網路
| 功能 | Azure 備份代理程式 | System Center DPM | Azure 備份伺服器 | Azure IaaS VM 備份 |
| --- | --- | --- | --- | --- |
| 網路壓縮 <br/>(至**備份伺服器**) | |![是][green] |![是][green] | |
| 網路壓縮 <br/>(到**復原服務保存庫**) |![是][green] |![yes][green] |![是][green] | |
| 網路通訊協定 <br/>(至**備份伺服器**) | |TCP |TCP | |
| 網路通訊協定 <br/>(到**復原服務保存庫**) |HTTPS |HTTPS |HTTPS |HTTPS |

![資料表索引鍵](./media/backup-introduction-to-azure-backup/table-key-2.png)

VM 延伸模組 (在 IaaS VM 上) 會透過儲存體網路直接從 Azure 儲存體帳戶讀取資料，所以不需要壓縮此流量。

如果您使用 System Center DPM 或 Azure 備份伺服器作為次要備份伺服器，請壓縮要從主要伺服器送往備份伺服器的資料。 將資料備份到 DPM 或 Azure 備份伺服器之前，壓縮資料可以節省頻寬。

#### <a name="network-throttling"></a>網路節流
Azure 備份代理程式提供網路節流，可讓您控制在資料傳輸期間使用網路頻寬的方式。 如果您需要在上班時間內備份資料，但不希望備份程序干擾其他網際網路流量，節流會很有幫助。 資料傳輸的節流適用於備份和還原活動。

## <a name="backup-and-retention"></a>備份和保留

Azure 備份每個*受保護的執行個體*上限為 9999 個復原點 (也稱為備份複本或快照集)。 受保護的執行個體可以是電腦、(實體或虛擬) 伺服器，或設定為將資料備份至 Azure 的工作負載。 如需詳細資訊，請參閱[什麼是受保護的執行個體](backup-introduction-to-azure-backup.md#what-is-a-protected-instance)一節。 儲存資料的備份複本之後，執行個體就會受到保護。 資料的備份複本就是保護。 如果來源資料已遺失或損毀，備份可以還原來源資料。 下表顯示每個元件的最大備份頻率。 備份原則設定決定您多快會用完復原點。 比方說，如果您每天建立復原點，則在用完之前可以保留復原點 27 年。如果您採取每月建立復原點，則在用完之前可以保留復原點 833 年。備份服務不會對復原點設定到期時間限制。

|  | Azure 備份代理程式 | System Center DPM | Azure 備份伺服器 | Azure IaaS VM 備份 |
| --- | --- | --- | --- | --- |
| 備份頻率<br/> (到復原服務保存庫) |每天備份 3 次 |每天備份 2 次 |每天備份 2 次 |每天備份 1 次 |
| 備份頻率<br/> (至磁碟) |不適用 |<li>每隔 15 分鐘 (SQL Server) <li>每隔 1 小時 (其他工作負載) |<li>每隔 15 分鐘 (SQL Server) <li>每隔 1 小時 (其他工作負載)</p> |不適用 |
| 保留選項 |每日、每週、每月、每年 |每日、每週、每月、每年 |每日、每週、每月、每年 |每日、每週、每月、每年 |
| 每個受保護執行個體的最大復原點 |9999|9999|9999|9999|
| 最大保留期間 |依照備份頻率而定 |依照備份頻率而定 |依照備份頻率而定 |依照備份頻率而定 |
| 本機磁碟上的復原點 |不適用 |<li>64 (檔案伺服器)、<li>448 (應用程式伺服器) |<li>64 (檔案伺服器)、<li>448 (應用程式伺服器) |不適用 |
| 磁帶上的復原點 |不適用 |無限 |不適用 |不適用 |

## <a name="what-is-a-protected-instance"></a>什麼是受保護的執行個體
受保護的執行個體一般是指 Windows 電腦、(實體或虛擬) 伺服器，或已設定為備份至 Azure 的 SQL 資料庫。 一旦您設定電腦、伺服器或資料庫的備份原則，並建立資料的備份複本之後，執行個體就會受到保護。 受保護執行個體備份資料後續的複本 (稱為復原點)，會增加取用的儲存體數量。 針對一個受保護的執行個體，您可以建立最多 9999 個復原點。 如果您從儲存體將復原點刪除，就不會算入 9999 的復原點總數。
受保護執行個體的常見範例是虛擬機器、應用程式伺服器、資料庫，以及執行 Windows 作業系統的個人電腦。 例如︰

* 執行 Hyper-V 或 Azure IaaS Hypervisor 網狀架構的虛擬機器。 虛擬機器的客體作業系統可以是 Windows Server 或 Linux。
* 應用程式伺服器︰應用程式伺服器可以是需要備份其資料，且執行 Windows Server 和工作負載的實體或虛擬機器。 一般工作負載為 Microsoft SQL Server、Microsoft Exchange Server、Microsoft SharePoint Server 及 Windows Server 的檔案伺服器角色。 若要備份這些工作負載，您需要 System Center Data Protection Manager (DPM) 或 Azure 備份伺服器。
* 執行 Windows 作業系統的個人電腦、工作站或膝上型電腦。


## <a name="what-is-a-recovery-services-vault"></a>什麼是復原服務保存庫？
復原服務保存庫是 Azure 中的線上儲存實體，用來保存備份副本、復原點及備份原則等資料。 您可以使用復原服務保存庫來保存 Azure 服務及內部部署伺服器和工作站的備份資料。 復原服務保存庫可輕鬆地組織您的備份資料，同時可減輕管理負擔。 在每個 Azure 訂用帳戶內，您可以為每個 Azure 區域最多建立 500 個復原服務保存庫。 在考慮儲存資料的位置時，並非所有區域的考量都一樣。 如需區域配對和額外儲存體考量的相關資訊，請參閱[異地備援儲存體](../storage/common/storage-redundancy-grs.md)。

備份保存庫是第一個保存庫版本，並以 Azure 服務管理員為基礎。 復原服務保存庫是第二個保存庫版本，並新增 Azure Resource Manager 模型功能。 如需功能差異的完整說明，請參閱[復原服務保存庫概觀](backup-azure-recovery-services-vault-overview.md)一文。 您無法再建立備份保存庫，而且所有現有的備份保存庫都已升級為復原服務保存庫。 您可以使用 Azure 入口網站來管理已升級為復原服務保存庫的保存庫。

## <a name="how-does-azure-backup-differ-from-azure-site-recovery"></a>Azure 備份與 Azure Site Recovery 有何不同？
Azure 備份與 Azure Site Recovery 類似，這兩個服務皆可備份資料及還原資料。 不過，這些服務各有不同的用途，可在您的企業中提供業務持續性和災害復原。 使用 Azure 備份來保護及還原更細微的資料。 例如，如果筆記型電腦上的簡報損毀，您會使用 Azure 備份來還原簡報。 如果您想要跨另一個資料中心複寫 VM 上的組態和資料，請使用 Azure Site Recovery。

Azure 備份會在內部部署和雲端保護資料。 Azure Site Recovery 可協調虛擬機器和實體伺服器的複寫、容錯移轉及容錯回復。 這兩項服務都很重要，因為您的災害復原解決方案必須讓資料保持安全且可復原 (備份)，「並且」  在服務中斷時讓您的工作負載可供使用 (Site Recovery)。

下列概念可協助您做出有關備份和災害復原的重要決策。

| 概念 | 詳細資料 | Backup  | 災害復原 (DR) |
| --- | --- | --- | --- |
| 復原點目標 (RPO) |在需要進行復原的情況下可接受的資料遺失數量。 |備份解決方案在其可接受的 RPO 有寬廣的變化性。 虛擬機器備份的 RPO 通常為 1 天，而資料庫備份的 RPO 只需 15 分鐘。 |災害復原解決方案的 RPO 很低。 DR 複本可以落後幾秒鐘或幾分鐘的時間。 |
| 復原時間目標 (RTO) |完成復原或還原所需的時間量。 |由於 RPO 較大，備份解決方案需要處理的資料量通常更多，這會導致 RTO 較長。 例如，根據從異地傳輸磁帶所需的時間，從磁帶還原資料可能需要數天的時間。 |因為災害復原解決方案與來源比較能保持同步，所以其 RTO 比較小。 需要處理的變更也比較少。 |
| 保留 |資料需要儲存多久？ |在需要作業復原的情況下 (資料損毀、不小心刪除檔案、作業系統失敗)，備份資料通常會保留 30 天或更短。<br>從法規遵循觀點來看，資料可能需要儲存達數個月或甚至數年。 在這類情況下，最理想的方式是將備份資料封存。 |災害復原只需要作業復原資料，通常是幾個小時或最多一天。 因為 DR 解決方案中使用的細部資料擷取，所以不建議使用 DR 資料進行長期保留。 |

## <a name="next-steps"></a>後續步驟
下列教學課程可以獲取詳細的逐步指示，協助您保護 Windows Server 上的資料，或在 Azure 中保護虛擬機器 (VM)︰

* [備份檔案和資料夾](backup-try-azure-backup-in-10-mins.md)
* [備份 Azure 虛擬機器](backup-azure-vms-first-look-arm.md)

如需保護其他工作負載的詳細資訊，請參閱這些文件︰

* [備份 Windows 伺服器](backup-configure-vault.md)
* [備份應用程式工作負載](backup-azure-microsoft-azure-backup.md)
* [備份 Azure IaaS VM](backup-azure-arm-vms-prepare.md)

[green]: ./media/backup-introduction-to-azure-backup/green.png
[yellow]: ./media/backup-introduction-to-azure-backup/yellow.png
[red]: ./media/backup-introduction-to-azure-backup/red.png
