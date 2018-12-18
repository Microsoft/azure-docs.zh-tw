---
title: 包含檔案
description: 包含檔案
services: storage
author: luywang
ms.service: storage
ms.topic: include
ms.date: 06/05/2018
ms.author: luywang
ms.custom: include file
ms.openlocfilehash: 7f093a1878bc3cf7e91cc14ec7a68b1a84764a49
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39485906"
---
# <a name="backup-and-disaster-recovery-for-azure-iaas-disks"></a>Azure IaaS 磁碟的備份和災害復原

本文說明如何在 Azure 中規劃 IaaS 虛擬機器 (VM) 和磁碟的備份和災害復原 (DR)。 本文件涵蓋受控磁碟和非受控磁碟。

首先，我們會討論 Azure 平台中內建的容錯功能，該功能有助於防範本機失敗。 接著會討論內建功能未能完全涵蓋的災害案例。 我們也會示範幾個工作負載案例範例，其中可能會有不同的備份和 DR 考量。 接著檢閱 IaaS 磁碟的可能 DR 解決方案。 

## <a name="introduction"></a>簡介

Azure 平台使用各種備援和容錯方法，協助保護客戶防範當地語系化硬體失敗。 本機失敗可能是指儲存部分虛擬磁碟資料的 Azure 儲存體伺服器電腦發生問題，或是該伺服器上的 SSD 或 HDD 失敗。 這類個別硬體元件失敗可能發生在正常作業期間。 

Azure 平台的設計可從這些失敗中復原。 重大災害可能會導致許多儲存體伺服器或甚至整個資料中心失敗或無法存取。 雖然您的 VM 和磁碟通常可防範當地語系化失敗，但若要保護工作負載防範可能影響 VM 和磁碟的區域全面重大失敗 (例如重大災害)，則需要採取額外步驟。

除了可能發生平台失敗之外，也可能發生客戶應用程式或資料問題。 例如，您的新版應用程式可能會不小心變更資料以致發生中斷。 在此情況下，您可能想要將應用程式和資料還原為含有已知良好狀態的舊版。 這需要維護定期備份。

針對區域性災害復原，您必須將 IaaS VM 磁碟備份至不同區域。 

讓我們先回顧可用來處理當地語系化失敗的一些方法，再檢視備份和 DR 選項。

### <a name="azure-iaas-resiliency"></a>Azure IaaS 復原

「復原」是指硬體元件中所發生之一般失敗的容錯。 復原是從失敗中復原並繼續運作的能力。 它不是避免失敗，而是以避免停機或資料遺失的方式來回應失敗。 復原的目標是讓應用程式在失敗後完全回到正常運作的狀態。 Azure 虛擬機器和磁碟設計成可從常見的硬體故障中復原。 讓我們看看 Azure IaaS 平台如何提供這項復原功能。

虛擬機器主要包含兩部分：計算伺服器和永續性磁碟。 這兩者都會影響虛擬機器的容錯。

如果裝載 VM 的 Azure 計算主機伺服器發生硬體失敗 (這很罕見)，Azure 的設計可自動在另一部伺服器上還原 VM。 在這種情況下，電腦會重新開機，而 VM 會在一段時間後進行備份。 Azure 會自動偵測這類硬體失敗並執行復原，協助確保客戶的 VM 盡快恢復可用。

至於 IaaS 磁碟，資料持久性是永續性儲存體平台的關鍵。 Azure 客戶在 IaaS 上執行重要的商務應用程式，而且需要持續提供資料。 Azure 設計這些 IaaS 磁碟保護的方式是在本機儲存三份資料的備援複本。 這些複本可提供高持久性以防範本機失敗。 如果其中一個保留磁碟的硬體元件失敗，由於還有兩個額外的複本支援磁碟要求，因此您的 VM 不會受到影響。 即使兩個支援磁碟的不同硬體元件同時失敗 (很罕見)，也沒問題。 

為了確保您永遠都有三份複本，如果三份複本中有一份無法使用，Azure 儲存體會在背景自動繁衍新的資料複本。 因此，您不需要對 Azure 磁碟使用 RAID 以取得容錯功能。 如果需要建立更大的磁碟區，簡單的 RAID 0 設定應該就足以分割磁碟。

由於此結構，Azure 為 IaaS 磁碟一致地提供企業級持久性，其[年度失敗率](https://en.wikipedia.org/wiki/Annualized_failure_rate)為零，領先業界。

計算主機或儲存體平台的當地語系化硬體故障有時會導致 VM 暫時無法使用，如 VM 可用性的 [Azure SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/) 中所述。 Azure 也提供領先業界的 SLA，適用於使用 Azure 進階 SSD 磁碟的單一 VM 執行個體。

若要防止應用程式工作負載由於磁碟或 VM 暫時無法使用而停機，客戶可以使用[可用性設定組](../articles/virtual-machines/windows/manage-availability.md)。 可用性設定組中的兩個或多個虛擬機器提供應用程式的備援。 Azure 會接著在具有不同電源、網路和伺服器元件的個別容錯網域中，建立這些 VM 和磁碟。 

因為有個別的容錯網域，當地語系化的硬體失敗通常不會同時影響組合中的多個 VM。 擁有個別的容錯網域可提高應用程式的可用性。 使用可用性設定組是在需要高可用性時的最佳做法。 下一節說明災害復原方面。

### <a name="backup-and-disaster-recovery"></a>備份和災害復原

災害復原是從罕見但重大的事件中復原的能力。 這些事件包括非暫時性且規模廣泛的失敗，例如影響整個區域的服務中斷。 災害復原包括資料備份和封存，而且可能包括手動操作，例如從備份中還原資料庫。

Azure 平台對當地語系化失敗的內建保護，在發生導致大規模中斷的重大災害時，可能無法完全保護 VM/磁碟。 這些大規模中斷包括資料中心遇到颶風、地震、火災，或大規模硬體裝置故障等重大事件。 此外，您可能會由於應用程式或資料問題而發生失敗。

為協助防止您的 IaaS 工作負載中斷，您應該規劃備援並擁有備份才能進行復原。 若要進行災害復原，您應該在與主要網站不同的地理位置進行備份。 這種方法有助於確保您的備份不會受到與原本影響 VM 或磁碟的相同事件所影響。 如需詳細資訊，請參閱 [Azure 應用程式的災害復原](/azure/architecture/resiliency/disaster-recovery-azure-applications)。

您的 DR 考量可能包括下列層面：

- 高可用性：可讓應用程式繼續在狀況良好狀態下執行，而不需要長期停機。 「狀況良好狀態」這個狀態是指應用程式有回應，而且使用者可以連線到應用程式並與其互動。 某些任務關鍵性應用程式和資料庫可能必須永遠可供使用，即使平台發生失敗也一樣。 針對這些工作負載，您可能需要規劃應用程式及資料的備援。

- 資料持久性：在某些情況下，主要考量是確保發生災害時會保留資料。 因此，您可能需要在不同的網站上有資料備份。 針對這類工作負載，您可能不需要對應用程式進行完整備援，而只要定期備份磁碟。

## <a name="backup-and-dr-scenarios"></a>備份和 DR 案例

讓我們看看應用程式工作負載案例的一些典型範例，以及規劃災害復原的考量。

### <a name="scenario-1-major-database-solutions"></a>案例 1：主要資料庫解決方案

假設有一個可支援高可用性的生產環境資料庫伺服器，例如 SQL Server 或 Oracle。 重要的生產環境應用程式和使用者都相依於此資料庫。 此系統的災害復原方案可能需要支援下列需求：

- 資料必須是受保護且可復原的。
- 伺服器必須可供使用。

災害復原方案可能需要在與備份不同的區域中維護資料庫複本。 根據伺服器可用性和資料復原的需求，此解決方案的涵蓋範圍從「主動對主動」或「主動對被動」複本網站，到定期離線備份資料。 SQL Server 和 Oracle 等關聯式資料庫提供各種複寫選項。 若是 SQL Server，請使用 [SQL Server Always On 可用性群組](https://msdn.microsoft.com/library/hh510230.aspx)來取得高可用性。

MongoDB 之類的 NoSQL 資料庫也支援以[複本](https://docs.mongodb.com/manual/replication/)進行備援。 使用高可用性複本。

### <a name="scenario-2-a-cluster-of-redundant-vms"></a>案例 2：備援 VM 的叢集

假設有一個由 VM 的叢集所處理的工作負載提供備援和負載平衡。 其中一個範例是區域中所部署的 Cassandra 叢集。 此架構類型已在該區域內提供高層級的備援。 不過，為保護工作負載防範區域層級失敗，您應該考慮將叢集分散到兩個區域，或定期備份至另一個區域。

### <a name="scenario-3-iaas-application-workload"></a>案例 3：IaaS 應用程式工作負載

讓我們看看 IaaS 應用程式工作負載。 例如，此應用程式可能是在 Azure VM 上執行的一般生產環境工作負載。 它可能是保留網站內容和其他資源的 Web 伺服器或檔案伺服器。 也可能是量身打造的商務應用程式，該應用程式會在 VM 上執行，並將其資料、資源和應用程式狀態儲存在 VM 磁碟上。 在此情況下，請務必定期進行備份。 備份頻率應該根據 VM 工作負載的本質。 例如，如果應用程式每天執行並修改資料，則應該每小時進行備份。

另一個範例是，報表伺服器從其他來源提取資料並產生彙總報表。 遺失此 VM 或磁碟可能會導致遺失報表。 不過，您可重新執行報告程序並重新產生輸出。 在此情況下，即使報表伺服器遇到災害，您實際上也不會遺失資料。 由於遺失的是報表伺服器上的部分資料，因此您可能會有更高層級的容錯。 在此情況下，為降低成本，可選擇較不頻繁的備份。

### <a name="scenario-4-iaas-application-data-issues"></a>案例 4：IaaS 應用程式資料問題

IaaS 應用程式資料問題是另一種可能性。 請考慮一個可計算、維護及提供重要商業資料 (例如價格資訊) 的應用程式。 您的新版應用程式有軟體錯誤，未能正確地計算價格，而且平台所提供的現有商用資料已損毀。 此時的最佳做法是還原至舊版應用程式和資料。 若要這麼做，請定期備份您的系統。

## <a name="disaster-recovery-solution-azure-backup"></a>災害復原解決方案：Azure 備份 

[Azure 備份](https://azure.microsoft.com/services/backup/)可用來進行備份和 DR，並適用於[受控磁碟](../articles/virtual-machines/windows/managed-disks-overview.md)和[非受控磁碟](../articles/virtual-machines/windows/about-disks-and-vhds.md#unmanaged-disks)。 您可以建立具有以時間為基礎的備份、簡易 VM 還原，以及備份保留原則的備份工作。 

如果您使用[進階 SSD 磁碟](../articles/virtual-machines/windows/premium-storage.md)、[受控磁碟](../articles/virtual-machines/windows/managed-disks-overview.md)或啟用[本地備援儲存體](../articles/storage/common/storage-redundancy-lrs.md)選項的其他磁碟類型，定期 DR 備份尤其重要。 Azure 備份會將資料儲存在復原服務保存庫中，以長期保留。 針對備份復原服務保存庫，選擇[異地備援儲存體](../articles/storage/common/storage-redundancy-grs.md)選項。 該選項可確保備份會複寫至不同的 Azure 區域，以防範區域性災害。

若是[非受控磁碟](../articles/virtual-machines/windows/about-disks-and-vhds.md#unmanaged-disks)，您可以針對 IaaS 磁碟使用本地備援儲存體類型，但請確定 Azure 備份已啟用復原服務保存庫的異地備援儲存體選項。

> [!NOTE]
> 如果您針對非受控磁碟使用[異地備援儲存體](../articles/storage/common/storage-redundancy-grs.md)或[具有讀取權限的異地備援儲存體](../articles/storage/common/storage-redundancy-grs.md#read-access-geo-redundant-storage)選項，您仍然需要一致性快照集才能進行備份和 DR。 請使用 [Azure 備份](https://azure.microsoft.com/services/backup/)或[一致性快照集](#alternative-solution-consistent-snapshots)。

 下表是 DR 可用解決方案的摘要。

| 案例 | 自動複寫 | DR 解決方案 |
| --- | --- | --- |
| 進階 SSD 磁碟 | 本機 ([本地備援儲存體](../articles/storage/common/storage-redundancy-lrs.md)) | [Azure 備份](https://azure.microsoft.com/services/backup/) |
| 受控磁碟 | 本機 ([本地備援儲存體](../articles/storage/common/storage-redundancy-lrs.md)) | [Azure 備份](https://azure.microsoft.com/services/backup/) |
| 非受控本地備援儲存體磁碟 | 本機 ([本地備援儲存體](../articles/storage/common/storage-redundancy-lrs.md)) | [Azure 備份](https://azure.microsoft.com/services/backup/) |
| 非受控異地備援儲存體磁碟 | 跨區域 ([異地備援儲存體](../articles/storage/common/storage-redundancy-grs.md)) | [Azure 備份](https://azure.microsoft.com/services/backup/)<br/>[一致性快照](#alternative-solution-consistent-snapshots) |
| 非受控具有讀取權限的異地備援儲存體磁碟 | 跨區域 ([具有讀取權限的異地備援儲存體](../articles/storage/common/storage-redundancy-grs.md#read-access-geo-redundant-storage)) | [Azure 備份](https://azure.microsoft.com/services/backup/)<br/>[一致性快照](#alternative-solution-consistent-snapshots) |

使用可用性設定組中的受控磁碟及 Azure 備份，即可達到最佳的高可用性。 如果您使用非受控磁碟，您仍然可以使用 Azure 備份進行 DR。 如果您無法使用 Azure 備份，替代的備份和 DR 解決方案是擷取[一致性快照](#alternative-solution-consistent-snapshots) (如稍後章節所述)。

應用程式或基礎結構層級的高可用性、備份和 DR 選項如下所示：

| Level |   高可用性   | 備份或 DR |
| --- | --- | --- |
| Application | SQL Server AlwaysOn | Azure 備份 |
| 基礎結構    | 可用性設定組  | 使用一致性快照的異地備援儲存體 |

### <a name="using-azure-backup"></a>使用 Azure 備份 

[Azure 備份](../articles/backup/backup-azure-vms-introduction.md)可將執行 Windows 或 Linux 的 VM 備份至 Azure 復原服務保存庫。 備份與還原業務重要資料之所以複雜，原因在於資料必須在產生資料的應用程式執行時進行備份。 

為了解決此問題，Azure 備份會對 Microsoft 工作負載提供應用程式一致備份。 它會使用磁碟區陰影服務來確保資料正確地寫入至儲存體。 Linux VM 則只能進行檔案一致備份，因為 Linux 沒有相當於磁碟區陰影服務的功能。

![Azure 備份流程][1]

Azure 備份在排定的時間起始備份工作時，會觸發 VM 中所安裝的備份延伸模組擷取時間點快照集。 快照集是與磁碟區陰影服務搭配擷取的，可在不需將虛擬機器中磁碟關閉的狀況下，取得所有磁碟一致的快照集。 VM 中的備份延伸模組會排清所有寫入，再擷取所有磁碟的一致性快照集。 擷取快照集之後，Azure 備份會將資料傳輸至備份保存庫。 為了讓備份程序更有效率，服務會識別上次備份之後變更的資料區塊，並只傳輸這些區塊。

若要還原，您可以透過 Azure 備份檢視可用的備份，再起始還原。 您可以透過 [Azure 入口網站](https://portal.azure.com/)、[使用 PowerShell](../articles/backup/backup-azure-vms-automation.md) 或使用 [Azure CLI](/cli/azure/)，建立並還原 Azure 備份。 

### <a name="steps-to-enable-a-backup"></a>啟用備份的步驟

使用下列步驟，透過 [Azure 入口網站](https://portal.azure.com/)啟用 VM 的備份。 視您的實際情況而定，會有一些變化。 請參閱 [Azure 備份](../articles/backup/backup-azure-vms-introduction.md)文件以取得完整詳細資料。 Azure 備份也[支援 VM 與受控磁碟](https://azure.microsoft.com/blog/azure-managed-disk-backup/)。

1.  建立 VM 的復原服務保存庫：

    a. 在 [Azure 入口網站](https://portal.azure.com/)中，瀏覽 [所有資源] 並尋找 [復原服務保存庫]。

    b. 在 [復原服務保存庫] 功能表上，按一下 [新增]，然後遵循步驟以在與 VM 相同的區域中建立新的保存庫。 例如，如果您的 VM 位於美國西部地區，請選取美國西部作為保存庫。

1.  確認新建立保存庫的儲存體複寫。 在 [復原服務保存庫] 之下存取該保存庫，然後移至 [設定] > [備份設定]。 確定預設已選取 [異地備援儲存體] 選項。 此選項可確保您的保存庫會自動複寫至次要資料中心。 例如，您在美國西部的保存庫會自動複寫至美國東部。

1.  設定備份原則，然後從相同的 UI 中選取 VM。

1.  確認已在 VM 上安裝備份代理程式。 如果使用 Azure 資源庫映像建立 VM，則已安裝備份代理程式。 否則 (也就是，如果您使用自訂映像)，請使用相關指示[在虛擬機器上安裝 VM 代理程式](../articles/backup/backup-azure-arm-vms-prepare.md#install-the-vm-agent-on-the-virtual-machine)。

1.  確定 VM 允許網路連線，備份服務才能運作正常。 請遵循[網路連線](../articles/backup/backup-azure-arm-vms-prepare.md#establish-network-connectivity)的指示。

1.  完成上述步驟之後，備份就會依照備份原則中指定的間隔定期執行。 如有必要，您可以從 Azure 入口網站上的保存庫儀表板，以手動方式觸發第一個備份。

如需使用指令碼將 Azure 備份自動化，請參閱 [VM 備份的 PowerShell Cmdlet](../articles/backup/backup-azure-vms-automation.md)。

### <a name="steps-for-recovery"></a>復原步驟

如果您需要修復或重建 VM，您可以從保存庫中的任何備份復原點還原 VM。 有幾個不同的選項可執行復原：

-   您可以建立新的 VM 作為已備份 VM 的時間點表示。

-   您可以還原磁碟，然後使用 VM 的範本來自訂及重建還原的 VM。 

如需詳細資訊，請參閱指示以[使用 Azure 入口網站來還原虛擬機器](../articles/backup/backup-azure-arm-restore-vms.md)。 本文件也說明在主要資料中心發生災害時，使用異地備援備份保存庫將備份的 VM 還原至成對資料中心的特定步驟。 在此情況下，Azure 備份會使用次要區域的計算服務來建立還原的虛擬機器。

您也可以使用 PowerShell 來[還原 VM](../articles/backup/backup-azure-arm-restore-vms.md#restore-a-vm-during-an-azure-datacenter-disaster)，或[從還原的磁碟建立新的 VM](../articles/backup/backup-azure-vms-automation.md#create-a-vm-from-restored-disks)。

## <a name="alternative-solution-consistent-snapshots"></a>替代解決方案：一致性快照集

如果您無法使用 Azure 備份，您可以使用快照集實作自己的備份機制。 為 VM 使用的所有磁碟建立一致性快照集，然後將這些快照集複寫至另一個區域很複雜。 因此，Azure 認為利用備份服務，會是比建置自訂解決方案更好的選擇。 

如果針對磁碟使用具有讀取權限的異地備援儲存體/異地備援儲存體，則會自動將快照集複寫至次要資料中心。 如果針對磁碟使用本地備援儲存體，則必須自行複寫資料。 如需詳細資訊，請參閱[以增量快照備份 Azure 非受控 VM 磁碟](../articles/virtual-machines/windows/incremental-snapshots.md)。

快照集是物件在特定時間點的表示。 快照集會針對其保留的資料大小增量予以收費。 如需詳細資訊，請參閱[建立 Blob 快照集](../articles/storage/blobs/storage-blob-snapshots.md)。

### <a name="create-snapshots-while-the-vm-is-running"></a>在 VM 執行時建立快照集

雖然您可以隨時擷取快照集，但如果 VM 正在執行，仍然會有資料串流處理到磁碟。 快照集所包含的部分作業可能在進行傳輸。 此外，如果涉及多個磁碟，則可能在不同時間發生不同磁碟的快照集。 這些案例可能會導致快照集未經協調。 這種缺乏協調的狀況對於等量磁碟區而言會特別有問題，如果在備份期間發生變更，其中所包含的檔案可能會損毀。

為避免這種情況，備份程序必須實作下列步驟：

1.  凍結所有磁碟。

1.  排清所有擱置的寫入。

1.  為所有磁碟[建立 Blob 快照集](../articles/storage/blobs/storage-blob-snapshots.md)。

某些 Windows 應用程式 (例如 SQL Server) 會透過磁碟區陰影服務提供一致備份機制，以建立應用程式一致備份。 在 Linux 上，您可以使用 fsfreeze 之類的工具來一致處理磁碟。 此工具會提供檔案一致備份，但不提供應用程式一致性快照集。 此程序很複雜，因此您應該考慮使用 [Azure 備份](../articles/backup/backup-azure-vms-introduction.md)，或是已實作此程序的第三方備份解決方案。

上述程序會產生所有 VM 磁碟的一致性快照集合，代表 VM 的特定時間點檢視。 這是 VM 的備份還原點。 您可以在排程的間隔重複此程序，以建立定期備份。 請參閱[將備份複製到另一個區域](#copy-the-snapshots-to-another-region)中將快照集複製到另一個區域以進行 DR 的步驟。

### <a name="create-snapshots-while-the-vm-is-offline"></a>在 VM 離線時建立快照集

建立一致備份的另一個選擇是關閉 VM 並擷取每個磁碟的 Blob 快照集。 擷取 Blob 快照集比協調執行中 VM 的快照集更容易，但是需要停機幾分鐘。

1. 關閉 VM。

1. 建立每個虛擬硬碟 Blob 的快照集，只需幾秒鐘的時間。

    若要建立快照集，您可以使用 [PowerShell](../articles/storage/common/storage-powershell-guide-full.md)、[Azure 儲存體 REST API](https://msdn.microsoft.com/library/azure/ee691971.aspx)、[Azure CLI](/cli/azure/)，或其中一個 Azure 儲存體用戶端程式庫，例如[適用於 .NET 的儲存體用戶端程式庫](https://msdn.microsoft.com/library/azure/hh488361.aspx)。

1. 啟動 VM，這會結束停機時間。 整個程序通常會在幾分鐘內完成。

此程序會產生所有磁碟的一致性快照集合，為 VM 提供備份還原點。

### <a name="copy-the-snapshots-to-another-region"></a>將快照集複製到另一個區域

光是建立快照集可能不足以進行 DR。 您也必須將快照集備份複寫至另一個區域。

如果針對磁碟使用異地備援儲存體或具有讀取權限的異地備援儲存體，則會自動將快照集複寫至次要地區。 複寫前可能有幾分鐘的延遲。 如果主要資料中心在快照集完成複寫前關閉，您就無法從次要資料中心存取快照集。 這個可能性很小。

> [!NOTE] 
> 只在異地備援儲存體或具有讀取權限的異地備援儲存體帳戶中保留磁碟，並無法保護 VM 防範災害。 您還必須建立一致性快照或使用 Azure 備份。 若要將 VM 復原至一致的狀態，則需要這樣做。

如果您使用本地備援儲存體，則必須在建立快照集之後，立即將快照集複製到不同的儲存體帳戶。 複製目標可能是不同區域中的本地備援儲存體帳戶，而導致複本位於遠端區域。 您也可以將快照集複製到相同區域中具有讀取權限的異地備援儲存體帳戶。 在此情況下，快照集會延遲複寫至遠端次要區域。 複製和複寫完成後，您的備份就可在主要網站發生災害時受到保護。

若要有效率地複製增量快照以進行 DR，請檢閱[以增量快照備份 Azure 非受控 VM 磁碟](../articles/virtual-machines/windows/incremental-snapshots.md)中的指示。

![以遞增快照集備份 Azure 非受控 VM 磁碟][2]

### <a name="recovery-from-snapshots"></a>從快照集復原

若要擷取快照集，請加以複製以建立新的 Blob。 如果您要從主要帳戶複製快照集，您可以將快照集複製到快照集的基底 Blob。 此程序會將磁碟還原至快照集。 這個程序稱為升級快照集。 如果您要從次要帳戶 (在本例中為存取權限異地備援儲存體) 複製快照集備份，則必須將它複製到主要帳戶。 您可以[使用 PowerShell](../articles/storage/common/storage-powershell-guide-full.md) 或使用 AzCopy 公用程式來複製快照集。 如需詳細資訊，請參閱[使用 AzCopy 命令列公用程式傳輸資料](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy)。

如果 VM 有多個磁碟，您必須複製屬於同一個一致還原點的所有快照集。 將快照集複製到可寫入的 VHD Blob 後，您就可以使用 Blob 透過 VM 的範本重新建立 VM。

## <a name="other-options"></a>其他選項

### <a name="sql-server"></a>SQL Server

在 VM 中執行的 SQL Server 有其本身內建的功能，可將您的 SQL Server 資料庫備份至 Azure Blob 儲存體或檔案共用。 如果儲存體帳戶是異地備援儲存體或存取權限異地備援儲存體，您可以在發生災害時，存取儲存體帳戶之次要資料中心內的備份，其限制如先前所述。 如需詳細資訊，請參閱 [Azure 虛擬機器中 SQL Server 的備份與還原](../articles/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery.md)。 除了備份和還原之外，[SQL Server Always On 可用性群組](../articles/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr.md)可以維護資料庫的次要複本。 這項功能可大幅縮短災害復原時間。

## <a name="other-considerations"></a>其他考量

本文已說明如何備份您的 VM 及其磁碟或擷取快照集以支援災害復原，以及如何使用這些備份或快照集來復原資料。 在 Azure Resource Manager 模型中，許多人使用範本在 Azure 中建立其 VM 和其他基礎結構。 您可以使用範本來建立 VM，以便每次都有相同的設定。 如果您使用自訂映像來建立 VM，您還必須使用存取權限異地備援儲存體帳戶儲存映像，以確保映像受到保護。

因此，備份程序可能包含下列兩項：

- 備份資料 (磁碟)。
- 備份組態 (範本和自訂映像)。

視您選擇的備份選項而定，您可能必須處理資料和設定的備份，或備份服務可能會為您處理所有作業。

## <a name="appendix-understanding-the-impact-of-data-redundancy"></a>附錄：了解資料備援的影響

對於 Azure 中的儲存體帳戶而言，有三種資料備援類型可視為與災害復原相關，那就是本地備援、異地備援或具有讀取權限的異地備援。 

本地備援儲存體會在同一個資料中心保留三份資料複本。 VM 寫入資料時，會更新所有三個複本再將成功傳回呼叫者，因此您知道這些複本完全相同。 您的磁碟可防範本機失敗，因為不太可能同時影響所有三個複本。 在本地備援儲存體的案例中，沒有異地備援，因此磁碟無法防範可能影響整個資料中心或儲存單位的重大失敗。

使用異地備援儲存體和具有讀取權限的異地備援儲存體，您的資料會有三個複本保留在您所選取的主要區域中。 您的資料會有另外三個複本保留在 Azure 所設定的對應次要區域。 例如，如果您在美國西部儲存資料，資料會複寫至美國東部。 複本保留會以非同步方式完成，而主要與次要網站更新之間有些延遲。 次要網站上的磁碟複本在每個磁碟上都是一致的 (有所延遲)，但多個作用中磁碟的複本可能不會彼此同步。 若要讓多個磁碟之間有一致複本，則需要一致性快照。

異地備援儲存體和具有讀取權限的異地備援儲存體之間的主要差異在於，採用具有讀取權限的異地備援儲存體，您可以隨時讀取次要複本。 如果發生問題導致無法存取主要區域中的資料，Azure 團隊會盡全力還原存取。 雖然主要區域已關閉，但如果您啟用具有讀取權限的異地備援儲存體，您可以存取次要資料中心內的資料。 因此，如果您打算在無法存取主要時從複本讀取，則應該考慮使用具有讀取權限的異地備援儲存體。

如果結果是嚴重的中斷，Azure 團隊可能會觸發異地容錯移轉，然後變更主要 DNS 項目以指向次要儲存體。 此時，如果您啟用異地備援儲存體或具有讀取權限的異地備援儲存體，您可以存取之前為次要區域中的資料。 換句話說，如果您的儲存體帳戶是具有讀取權限的異地備援儲存體並發生問題，只有在具異地容錯移轉時才能存取次要儲存體。

如需詳細資訊，請參閱[如果 Azure 儲存體發生中斷怎麼辦](../articles/storage/common/storage-disaster-recovery-guidance.md)。 

>[!NOTE] 
>Microsoft 會控制是否發生容錯移轉。 容錯移轉不是針對每個儲存體帳戶所控制，所以不會由個別客戶來決定。 若要實作特定儲存體帳戶或虛擬機器磁碟的災害復原，您必須使用本文稍早所述的技術。



[1]: ./media/virtual-machines-common-backup-and-disaster-recovery-for-azure-iaas-disks/backup-and-disaster-recovery-for-azure-iaas-disks-1.png
[2]: ./media/virtual-machines-common-backup-and-disaster-recovery-for-azure-iaas-disks/backup-and-disaster-recovery-for-azure-iaas-disks-2.png
