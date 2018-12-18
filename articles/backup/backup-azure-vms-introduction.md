---
title: 在 Azure 中規劃 VM 備份基礎結構
description: 在 Azure 中備份虛擬機器時的重要考量
services: backup
author: markgalioto
manager: carmonm
keywords: 備份 VM, 備份虛擬機器
ms.service: backup
ms.topic: conceptual
ms.date: 8/29/2018
ms.author: markgal
ms.openlocfilehash: 9e2ef16cffb044409b6f7f8e7785010097bcda87
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286647"
---
# <a name="plan-your-vm-backup-infrastructure-in-azure"></a>在 Azure 中規劃 VM 備份基礎結構
本文提供效能和資源方面的建議，以協助您規劃 VM 備份基礎結構。 本文也會定義備份服務的重要層面；這些層面對於決定架構、容量規劃及排程來說相當重要。 如果您已經[準備好環境](backup-azure-arm-vms-prepare.md)，則規劃是您開始[備份 VM](backup-azure-arm-vms.md) 之前的下一個步驟。 如果您需要 Azure 虛擬機器的詳細資訊，請參閱 [虛擬機器文件](https://azure.microsoft.com/documentation/services/virtual-machines/)。 

> [!NOTE]
> 本文適用於受控磁碟和非受控磁碟。 如果您使用非受控磁碟，則有一些儲存體帳戶建議。 如果您使用 [Azure 受控磁碟](../virtual-machines/windows/managed-disks-overview.md)，則不必擔心效能或資源使用率問題。 Azure 會為您最佳化儲存體使用率。
>

## <a name="how-does-azure-back-up-virtual-machines"></a>Azure 如何備份虛擬機器？
Azure 備份服務在排定的時間開始備份工作時，服務會觸發備份擴充功能以建立時間點快照集。 Azure 備份服務在 Windows 中使用 _VMSnapshot_ 延伸模組，在 Linux 中使用 _VMSnapshotLinux_ 延伸模組。 延伸模組會在進行第一個 VM 備份期間安裝。 若要安裝延伸模組，VM 必須正在執行中。 如果 VM 未在執行中，則備份服務會擷取基礎儲存體的快照集 (因為 VM 停止時不會發生任何應用程式寫入)。

當擷取 Windows VM 的快照集時，備份服務會與磁碟區陰影複製服務 (VSS) 協調，以取得虛擬機器磁碟一致的快照集。 如果您正在備份 Linux VM，您可以撰寫自己的自訂指令碼，以確保擷取 VM 快照集時會有一致性。 本文稍後會提供叫用這些指令碼的詳細資料。

Azure 備份服務擷取快照集之後，資料會傳輸至保存庫。 為了能更有效率，服務只會找出並傳輸自上次備份之後有變更的資料區塊。

![Azure 虛擬機器備份架構](./media/backup-azure-vms-introduction/vmbackup-architecture.png)

資料傳輸完畢後，系統會移除快照集並建立復原點。

> [!NOTE]
> 1. 在備份程序進行期間，Azure 備份不會包含連接至虛擬機器的暫存磁碟。 如需詳細資訊，請參閱[暫存儲存空間](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)部落格文章 (英文)。
> 2. Azure 備份會擷取儲存體層級的快照集，並將快照集傳輸至保存庫。 在備份作業完成前，請勿變更儲存體帳戶金鑰。
>

### <a name="data-consistency"></a>資料一致性
備份與還原業務重要資料之所以複雜，原因在於資料必須在產生資料的應用程式執行時進行備份。 為了解決此問題，Azure 備份同時針對 Windows 和 Linux VM 支援應用程式一致備份
#### <a name="windows-vm"></a>Windows VM
Azure 備份會在 Windows VM 上執行 VSS 完整備份 (深入了解 [VSS 完整備份](http://blogs.technet.com/b/filecab/archive/2008/05/21/what-is-the-difference-between-vss-full-backup-and-vss-copy-backup-in-windows-server-2008.aspx))。 若要啟用 VSS 複製備份，需要在 VM 上設定下列登錄機碼。

```
[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
"USEVSSCOPYBACKUP"="TRUE"
```

#### <a name="linux-vms"></a>Linux VM

Azure 備份提供指令碼處理架構，可用來控制備份工作流程和環境。 若要確保應用程式一致的 Linux VM 備份，請使用指令碼處理架構來建立自訂前置指令碼和後置指令碼。 擷取虛擬機器快照集之前先叫用前指令碼，然後在虛擬機器快照集作業完成之後叫用後指令碼。 如需詳細資訊，請參閱[應用程式一致的 Linux VM 備份](https://docs.microsoft.com/azure/backup/backup-azure-linux-app-consistent)一文。

> [!NOTE]
> Azure 備份只會叫用客戶撰寫的前指令碼和後指令碼。 如果前指令碼和後指令碼順利執行，Azure 備份會將復原點標記為應用程式一致。 但是，客戶要對使用自訂指令碼時的應用一致性負最終責任。
>

下表說明一致性的類型，以及發生一致性的條件。

| 一致性 | 以 VSS 基礎 | 說明和詳細資料 |
| --- | --- | --- |
| 應用程式一致性 |是 - 適用於 Windows|對於工作負載而言，最理想的情況是應用程式一致性，因為它確保：<ol><li> VM 啟動。 <li>不會毀損。 <li>不會遺失資料。<li> 資料會與使用資料的應用程式保持一致，方法是在備份期間使用 VSS 或前/後指令碼將應用程式納入。</ol> <li>*Windows VM*- 大部分 Microsoft 工作負載都有 VSS 寫入器，負責執行與資料一致性有關的工作負載特定動作。 例如，SQL Server VSS 寫入器可確保正確寫入交易記錄檔和資料庫。 對於 IaaS Windows VM 備份，若要建立應用程式一致的復原點，備份擴充功能必須叫用 VSS 工作流程，並在建立虛擬機器快照之前完成它。 若要取得精確的 Azure VM 快照集，也必須完成所有 Azure VM 應用程式的 VSS 寫入器。 (了解 [VSS 的基本知識](http://blogs.technet.com/b/josebda/archive/2007/10/10/the-basics-of-the-volume-shadow-copy-service-vss.aspx)及深入了解[其運作方式](https://technet.microsoft.com/library/cc785914%28v=ws.10%29.aspx)的詳細資料)。 </li> <li> *Linux VM*- 客戶可以執行[自訂前指令碼和後指令碼以確保應用程式一致性](https://docs.microsoft.com/azure/backup/backup-azure-linux-app-consistent)。 </li> |
| 檔案系統一致性 |是 - 適用於 Windows 電腦 |有兩種案例的復原點可以讓「檔案系統保持一致」：<ul><li>Azure 中 Linux VM 的備份，沒有前指令碼/後指令碼，或是前指令碼/後指令碼失敗。 <li>在 Azure 中備份 Windows VM 期間發生的 VSS 失敗。</li></ul> 在這兩種情況中，可採取的最佳作法是確保： <ol><li> VM 啟動。 <li>不會毀損。<li>不會遺失資料。</ol> 應用程式需要在還原的資料上實作自己的「修正」機制。 |
| 損毀一致性 |否 |此狀況相當於虛擬機器「當機」(經由軟體重設或硬體重設)。 當機時保持一致性通常發生於 Azure 虛擬機器在備份期間關閉時。 當機時保持一致的復原點不保證儲存媒體上資料的一致性 -- 無論從作業系統還是應用程式的觀點來說都一樣。 只會擷取備份時已存在磁碟上的資料，並加以備份。 <br/> <br/> 儘管不提供保證，但作業系統通常會啟動，後面接著進行磁碟檢查程序 (例如 chkdsk)，以修正任何損毀錯誤。 任何未傳輸到磁碟的記憶體內部資料或寫入都會遺失。 如果需要進行資料復原，應用程式通常會接著進行其本身的驗證機制。 <br><br>例如，如果交易記錄中有項目不存在資料庫中，則資料庫軟體會執行復原，直到資料變成一致為止。 當資料跨多個虛擬磁碟時 (例如跨距磁碟區)，損毀一致復原點不保證資料的正確性。 |

## <a name="performance-and-resource-utilization"></a>效能與資源使用率
和內部部署的備份軟體一樣，在 Azure 中備份 VM 時也應該規劃容量和資源使用率需求。 [Azure 儲存體限制](../azure-subscription-service-limits.md#storage-limits) 會定義如何建構 VM 部署，以發揮最大效能並盡可能不影響執行中的工作負載。

### <a name="number-of-disks"></a>磁碟數量
備份程序會嘗試盡快完成備份作業。 過程中，它會竭盡所能地取用資源。 不過，所有 I/O 作業都受限於 *單一 Blob 的目標輸送量*，上限為每秒 60MB。 為了達到最大速度，備份程序會嘗試「平行」 備份每個 VM 的磁碟。 如果 VM 有四個磁碟，服務便會嘗試平行備份全部 4 個磁碟。 判斷儲存體帳戶備份流量時，要備份的**磁碟數量**是最重要的因素。

### <a name="backup-schedule"></a>備份排程
影響效能的另一因素是 **備份排程**。 如果您設定原則，讓所有 VM 在同一時間進行備份，您便排定了一場流量壅塞災難。 備份程序會嘗試平行備份所有磁碟。 若要減少備份流量，請在同一天的不同時段備份不同的 VM，時間不要重疊。

## <a name="capacity-planning"></a>容量規劃
下載 [VM 備份容量規劃 Excel 試算表](https://gallery.technet.microsoft.com/Azure-Backup-Storage-a46d7e33) ，以了解磁碟和備份排程選項所產生的影響。

### <a name="backup-throughput"></a>備份輸送量
針對每個要備份的磁碟，Azure 備份會讀取磁碟上的區塊，只儲存已變更的資料 (增量備份)。 下表顯示備份服務輸送量的平均值。 使用下列資料，您可以預估備份特定大小的磁碟需花費的時間量。

| 備份作業 | 最佳輸送量 |
| --- | --- |
| 初始備份 |160 Mbps |
| 增量備份 (DR) |640 Mbps  <br><br> 如果將變更的資料 (需要備份) 分散於磁碟上，則輸送量就會顯著降低。|

## <a name="total-vm-backup-time"></a>VM 備份時間總計
雖然大部分的備份時間都花費在讀取和複製資料，但備份 VM 所需的全部時間還包含其他作業：

* [安裝或更新備份擴充功能](backup-azure-arm-vms.md)所需要的時間。
* 快照時間，即觸發快照所需的時間。 快照會在接近排定的備份時間觸發。
* 佇列等候時間。 備份服務程序作業同時來自多個客戶，因此快照集資料可能不會立即複製到復原服務保存庫。 在尖峰負載時刻，最多可能需要八小時才能處理備份。 不過，針對每日備份原則，VM 備份時間總計會小於 24 小時。
小於 24 小時的備份時間總計適用於增量備份，但可能不適合第一次備份。 根據資料大小和製作備份的時間，第一次備份所需的時間會按比例增加，並可能大於 24 小時。
* 資料傳輸時間，備份服務計算上次備份的增量變更，並將這些變更傳輸到保存庫儲存體所需的時間。

### <a name="why-are-backup-times-longer-than-12-hours"></a>為何備份時間會超過 12 小時？

備份包含兩個階段：擷取快照集，以及將快照集傳輸到保存庫。 備份服務會針對儲存體進行最佳化。 將快照集資料傳輸到保存庫時，服務只會傳輸自上一個快照集以來的累加變更。  為了判斷累加變更，服務會計算區塊的總和檢查碼。 如果區塊已變更，就會將該區塊識別為要傳送到保存庫的區塊。 接著，服務會深入探索每個已識別的區塊，尋找機會將要傳輸的資料降到最低。 在評估所有已變更的區塊之後，服務會聯合所做的變更，然後將它們傳送到保存庫。 對部分舊版應用程式來說，小型的分散式寫入並非最適合用於儲存體。 如果快照集包含許多小型分散式寫入，則服務會花更多時間來處理應用程式所寫入的資料。 對於在 VM 內執行的應用程式，建議的應用程式寫入區塊大小下限為 8 KB。 如果您的應用程式使用小於 8 KB 的區塊，則備份效能會受到影響。 如需調整您的應用程式以改善備份效能的說明，請參閱[使用 Azure 儲存體調整應用程式以取得最佳化效能](../virtual-machines/windows/premium-storage-performance.md)。 雖然這篇關於備份效能的文章使用的是進階儲存體範例，但指引適用於標準儲存體磁碟。

## <a name="total-restore-time"></a>總計還原時間

還原作業是由兩項主要工作所組成︰將資料從保存庫複製回所選擇的客戶儲存體帳戶，以及建立虛擬機器。 從保存庫複製資料所需的時間，取決於備份儲存在 Azure 中的位置，以及客戶儲存體帳戶的位置。 複製資料所花費的時間取決於︰
* 佇列等待時間 - 由於服務會同時處理來自多位客戶的還原作業，因此會將還原要求放入佇列中。
* 資料複製時間 - 資料會從保存庫複製到客戶儲存體帳戶。 還原時間取決於 Azure 備份服務在所選客戶儲存體帳戶上取得的 IOPS 和輸送量。 若要減少還原程序期間的複製時間，請選取未使用其他應用程式寫入或讀取載入的儲存體帳戶。

## <a name="best-practices"></a>最佳作法

在設定所有虛擬機器的備份時，建議您遵循下列做法：

* 避免將超過 10 個來自相同雲端服務的傳統虛擬機器排程在同一時間內進行備份。 如果您想要備份多個來自相同雲端服務的虛擬機器，建議您將備份開始時間錯開一小時。
* 避免將一個保存庫的超過 100 個虛擬機器排程在同一時間內進行備份。
* 安排 VM 備份在非尖峰時段進行。 如此一來，備份服務會使用 IOPS 將資料從客戶儲存體帳戶傳輸到保存庫。
* 請確定為了備份而啟用的 Linux VM 具有 Python 2.7 版或更新版本。

### <a name="best-practices-for-vms-with-unmanaged-disks"></a>非受控磁碟虛擬機器的適用最佳做法

下列建議僅適用於使用非受控磁碟的虛擬機器。 如果您的虛擬機器使用受控磁碟，備份服務會處理所有的儲存體管理活動。

* 請務必將備份原則套用於分散到多個儲存體帳戶的虛擬機器。 單一儲存體帳戶中受到同個備份排程保護的磁碟數量，最多不應超過 20 個。 如果您的儲存體帳戶中有超過 20 個磁碟，請將這些 VM 分散到多個原則，以在備份程序的傳輸階段取得所需的 IOPS。
* 請勿將進階儲存體上執行的 VM 還原至相同的儲存體帳戶。 如果還原作業程序與備份作業一致，將會減少備份可用的 IOPS。
* 對於虛擬機器備份堆疊 V1 上的進階虛擬機器備份，您應該只配置 50% 的總儲存體帳戶空間，讓備份服務可以將快照集複製到儲存體帳戶，並從儲存體帳戶將資料傳送到到保存庫。


## <a name="data-encryption"></a>資料加密
Azure 備份不會在備份過程中加密資料。 不過，您可以在 VM 中加密資料，並順暢地備份受保護的資料 (深入了解 [加密資料的備份](backup-azure-vms-encryption.md))。

## <a name="calculating-the-cost-of-protected-instances"></a>計算受保護執行個體的成本
透過 Azure 備份進行備份的 Azure 虛擬機器受限於 [Azure 備份價格](https://azure.microsoft.com/pricing/details/backup/)。 受保護的執行個體是根據虛擬機器的*實際*大小來計算，也就是虛擬機器中暫存空間除外的所有資料總和。

備份 VM 的定價方式並非以連接到虛擬機器的每個資料磁碟其支援大小上限為基礎。 而是根據儲存在資料磁碟中的實際資料來定價。 同樣地，備份儲存體帳單也是根據 Azure 備份所儲存的資料計費，也就是每個復原點所儲存的實際資料總和。

例如，A2 標準大小的虛擬機器擁有兩個額外資料磁碟，每個大小上限為 4 TB。 下表提供上述每個磁碟實際儲存的資料：

| 磁碟類型 | 大小上限 | 實際儲存的資料 |
| --------- | -------- | ----------- |
| 作業系統磁碟 |4095 GB |17 GB |
| 本機磁碟/暫存磁碟 |135 GB |5 GB (未包含備份) |
| 資料磁碟 1 |4095 GB |30 GB |
| 資料磁碟 2 |4095 GB |0 GB |

在此案例中，虛擬機器的實際容量為 17 GB + 30 GB + 0 GB = 47 GB。 這個受保護的執行個體大小 (47 GB) 會成為每月計費的依據。 當虛擬機器的資料量增加時，用於計費之受保護的執行個體大小也會隨之變更。

第一次成功備份完成後才會開始計費。 儲存體和受保護的執行個體也會在此同時開始計費。 只要虛擬機器有任何備份資料儲存在保存庫，就會繼續計費。 如果您停止虛擬機器上的保護功能，但是虛擬機器備份資料仍在保存庫中，就會繼續計費。

只有在停止保護且刪除全部備份資料時，才會對指定的虛擬機器停止計費。 當保護停止且沒有任何作用中的備份作業時，最後一個成功 VM 備份的大小會成為每月帳單所使用的受保護執行個體大小。

## <a name="questions"></a>有疑問嗎？
如果您有問題，或希望我們加入任何功能，請 [傳送意見反應給我們](http://aka.ms/azurebackup_feedback)。

## <a name="next-steps"></a>後續步驟
* [備份虛擬機器](backup-azure-arm-vms.md)
* [管理虛擬機器備份](backup-azure-manage-vms.md)
* [還原虛擬機器](backup-azure-arm-restore-vms.md)
* [疑難排解 VM 備份問題](backup-azure-vms-troubleshoot.md)
