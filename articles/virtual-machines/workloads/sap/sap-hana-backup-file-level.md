---
title: 檔案層級的 SAP HANA Azure 備份 | Microsoft Docs
description: 備份 Azure 虛擬機器上的 SAP HANA 有兩種主要做法，本文涵蓋檔案層級的 SAP HANA Azure 備份
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ums.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/05/2018
ms.author: rclaus
ms.openlocfilehash: d3d1769766053b513a98df153cb635ae148f26b1
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37867365"
---
# <a name="sap-hana-azure-backup-on-file-level"></a>檔案層級的 SAP HANA Azure 備份

## <a name="introduction"></a>簡介

這是與 SAP HANA 備份相關的三篇系列文章的其中一篇。 [Azure 虛擬機器上 SAP HANA 的備份指南](./sap-hana-backup-guide.md)提供開始使用的概觀和資訊，[以儲存體快照集為基礎的 SAP HANA 備份](./sap-hana-backup-storage-snapshots.md)涵蓋以儲存體快照集為基礎的備份選項。

Azure 中的不同 VM 類型允許連接不同數目的 VHD。 確切的詳細資料記載於 [Azure 中的 Linux 虛擬機器大小](../../linux/sizes.md)。 在本文件所參照的測試中，我們使用允許 64 個已連接資料磁碟的 GS5 Azure VM。 在較大型的 SAP HANA 系統中，資料和記錄檔可能已佔據大量磁碟，可能還加上用於最佳磁碟 IO 輸送量的軟體等量。 如需 Azure VM 上 SAP HANA 部署之建議磁碟設定的詳細資料，請閱讀 [Azure 上的 SAP HANA 作業指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)一文。 所提供的建議也包括本機備份的磁碟空間建議。

目前 SAP HANA 備份還沒有與 Azure 備份服務整合。 在檔案層級管理備份/還原的標準方式，是透過 SAP HANA Studio 或 SAP HANA SQL 陳述式使用以檔案為基礎的備份。 詳細資訊請參閱 [SAP HANA SQL 和系統檢視參考](https://help.sap.com/hana/SAP_HANA_SQL_and_System_Views_Reference_en.pdf)。

![本圖顯示 SAP HANA Studio 中的備份的功能表對話方塊](media/sap-hana-backup-file-level/image022.png)

本圖顯示 SAP HANA Studio 中的備份的功能表對話方塊。 當選擇 [檔案]&quot;&quot; 類型，必須指定檔案系統中的路徑讓 SAP HANA 寫入備份檔案。 還原的運作方式亦同。

雖然這項選擇聽起來簡單又直接，但仍有一些事要考量。 之前有提到，Azure 虛擬機器可連結的資料磁碟數目有限制。 可能無法在 VM 的檔案系統上儲存 SAP HANA 備份檔案，取決於資料庫的大小和磁碟輸送量需求，其中可能牽涉跨多個資料磁碟的軟體等量。 在本文稍後提供多種選項，可用於在處理數 TB 資料時，移動這些備份檔案以及管理檔案大小限制和效能。

另一個可以不計總容量提供更多自由的選項是 Azure Blob 儲存體。 雖然單一 blob 也有 1 TB 的限制，單一 blob 容器的總容量目前為 500 TB。 此外，它讓客戶可以選擇較具成本效益的「非經常性」&quot;&quot;blob 儲存體。 如需非經常性 blob 儲存體的詳細資訊，請參閱 [Azure Blob 儲存體︰經常性存取與非經常性存取儲存層](../../../storage/blobs/storage-blob-storage-tiers.md)。

如果想要更安全，使用異地複寫儲存體帳戶來存放 SAP HANA 備份。 如需儲存體帳戶複寫相關的詳細資料，請參閱 [Azure 儲存體複寫](../../../storage/common/storage-redundancy.md)。

一個人將 SAP HANA 備份專用的 VHD 放在異地複寫的專用備份儲存體帳戶。 而另一個人可以將存放 SAP HANA 備份的 VHD 複製到異地複寫的儲存體帳戶中，或複製到不同區域中的儲存體帳戶。

## <a name="azure-backup-agent"></a>Azure 備份代理程式

Azure 備份提供的選項不僅可備份完整的 VM，但可透過備份代理程式 (必須在客體 OS 上安裝) 備份檔案和目錄。 但只有在 Windows 上才支援此代理程式 (請參閱[使用 Resource Manager 部署模型將 Windows Server 或用戶端備份至 Azure](../../../backup/backup-configure-vault.md))。

解決方法是先將 SAP HANA 備份檔案複製到 Azure 上的 Windows VM (例如，透過 SAMBA 共用)，然後從該處使用 Azure 備份代理程式。 雖然技術上可行，但可能會增加複雜度，並因為在 Linux 和 Windows VM 之間的複製而使備份或還原程序變慢許多。 不建議採取此做法。

## <a name="azure-blobxfer-utility-details"></a>Azure blobxfer 公用程式詳細資料

為了在 Azure 儲存體上儲存目錄和檔案，可以使用 CLI 或 PowerShell，或開發使用其中一種 [Azure SDK](https://azure.microsoft.com/downloads/) 的工具。 另外也有現成的公用程式 AzCopy，可將資料複製到 Azure 儲存體，但是它只適用於 Windows (請參閱[使用 AzCopy 命令列公用程式傳輸資料](../../../storage/common/storage-use-azcopy.md))。

因此，使用 Blobxfer 來複製 SAP HANA 備份檔案。 blobxfer 是開放原始碼，可從 [GitHub](https://github.com/Azure/blobxfer) 取得，許多客戶在生產環境中使用它。 此工具不允許將資料直接複製到 Azure blob 儲存體或 Azure 檔案共用。 它也提供各種實用的功能，例如 md5 雜湊、複製含多個檔案的目錄時可使用自動平行處理原則。

## <a name="sap-hana-backup-performance"></a>SAP HANA 備份的效能

![這個螢幕擷取畫面是 SAP HANA Studio 中的 SAP HANA 備份主控台](media/sap-hana-backup-file-level/image023.png)

這個螢幕擷取畫面是 SAP HANA Studio 中的 SAP HANA 備份主控台。 在連結到 HANA VM (使用 XFS 檔案系統) 的單一 Azure 標準儲存體磁碟上，備份 230 GB 的資料約需時 42 分鐘。

![這個螢幕擷取畫面是 SAP HANA 測試 VM 上的 YaST](media/sap-hana-backup-file-level/image024.png)

這個螢幕擷取畫面是 SAP HANA 測試 VM 上的 YaST。 如同先前所述，可以看到 SAP HANA 備份的單一 1 TB 磁碟。 備份 230 GB 花費約 42 分鐘。 此外，還連結了五個 200 GB 的磁碟並建立了軟體 RAID md0，而且在這五個 Azure 資料磁碟上串接。

![對軟體 RAID 重複相同的備份，並串接五個連結的 Azure 標準儲存體資料磁碟](media/sap-hana-backup-file-level/image025.png)

對軟體 RAID 重複相同的備份，並串接五個連結的 Azure 標準儲存體資料磁碟，將備份時間從 42 分鐘降到 10 分鐘。 磁碟已連結，且沒有快取至 VM。 由此可見，磁碟寫入輸送量對備份時間有多重要。 您可以改為切換到 Azure 進階儲存體，進一步加速程序，以獲得最佳效能。 一般而言，生產系統應該使用 Azure 進階儲存體。

## <a name="copy-sap-hana-backup-files-to-azure-blob-storage"></a>將 SAP HANA 備份檔案複製到 Azure Blob 儲存體

快速儲存 SAP HANA 備份檔案的另一個選項是 Azure Blob 儲存體。 使用 M32ts、M32ls、M64ls 和 GS5 VM 類型的 Azure，單一 Blob 容器的上限為 500 TB，對於一些較小的 SAP HANA 系統來說就已經足夠，可保留充足的 SAP HANA 備份。 客戶可在「經常性存取」&quot;&quot;和「非經常性存取」&quot;&quot;儲存體之間選擇 (請參閱 [Azure Blob 儲存體︰經常性存取與非經常性存取儲存層](../../../storage/blobs/storage-blob-storage-tiers.md)。

使用 blobxfer 工具，將 SAP HANA 備份檔案直接複製到 Azure Blob 儲存體很容易。

![在這裡可以看到完整的 SAP HANA 檔案備份的檔案](media/sap-hana-backup-file-level/image026.png)

在這裡可以看到完整的 SAP HANA 檔案備份的檔案。 有四個檔案，最大的一個約 230 GB。

![將 230 GB 複製至 Azure 的標準儲存體帳戶 blob 容器花了大約 3000 秒](media/sap-hana-backup-file-level/image027.png)

在初始測試中沒有使用 md5 雜湊，將 230 GB 複製至 Azure 的標準儲存體帳戶 blob 容器花了大約 3000 秒。

![在這個螢幕擷取畫面中，可以看到它在 Azure 入口網站中的樣子](media/sap-hana-backup-file-level/image028.png)

在這個螢幕擷取畫面中，可以看到它在 Azure 入口網站中的樣子。 建立了一個名為 &quot;sap-hana-backups&quot; 的 blob 容器，其中包含四個 blob，代表 SAP HANA 備份檔案。 其中一個的大小約為 230 GB。

HANA Studio 備份主控台可讓您限制 HANA 備份檔案的檔案大小上限。 在範例環境中，有多個較小的備份檔案，而不是一個大型的 230 GB 檔案，可增進效能。

![設定 HANA 端的備份檔案大小限制不會改善備份時間](media/sap-hana-backup-file-level/image029.png)

設定 HANA 端的備份檔案大小限制不會改善備份時間，因為檔案會循序寫入，如此圖中所示。 檔案大小上限設定為 60 GB，因此備份會建立四個大型資料檔案，而不是 230 GB 的單一檔案。 需要使用多個備份檔案來備份 HANA 資料庫，以利用較大型 Azure VM (例如 M64s、M64ms, M128s 和 M128ms) 的記憶體。

![為了測試 blobxfer 工具的平行處理原則，而將 HANA 備份的檔案大小上限設為 15 GB](media/sap-hana-backup-file-level/image030.png)

為了測試 blobxfer 工具的平行處理原則，而將 HANA 備份的檔案大小上限設為 15 GB，得到 19 個備份檔案。 如此設定，將 blobxfer 複製 230 GB 至 Azure Blob 儲存體的時間，從 3000 秒降至 875 秒 。

得到這個結果，是因為寫入 Azure blob 有每秒 60 MB 的限制。 透過多個 blob 實行平行處理原則可以解決瓶頸，但缺點是︰將 blobxfer 工具複製所有 HANA 備份檔案到 Azure Blob 儲存體的效能增加時，會加重 HANA VM 和網路的負載。 HANA 系統的作業會受影響。

## <a name="blob-copy-of-dedicated-azure-data-disks-in-backup-software-raid"></a>在備份軟體 RAID 中進行 Azure 專用資料磁碟的 Blob 複製

不同於手動 VM 資料磁碟備份，這個做法不會備份 VM 上的所有資料磁碟儲存整個 SAP 安裝，包括 HANA 資料、HANA 記錄檔和組態檔。 相反地，其概念是將專用的軟體 RAID 搭配串接多個 Azure 資料 VHD，來儲存完整的 SAP HANA 檔案備份。 可以只複製這些包含 SAP HANA 備份的磁碟。 它們可以輕鬆地保存在專用的 HANA 備份儲存體帳戶中，或連結到專用的「備份管理 VM」&quot;&quot;供進一步處理。

![使用 **start-azurestorageblobcopy** PowerShell 命令複製所有相關的 VHD](media/sap-hana-backup-file-level/image031.png)

備份至本機軟體 RAID 完成之後，使用 **start-azurestorageblobcopy** PowerShell 命令複製所有相關的 VHD (請參閱 [Start-AzureStorageBlobCopy](/powershell/module/azure.storage/start-azurestorageblobcopy))。 由於它只會影響保留備份檔案的專用檔案系統，無需考慮 SAP HANA 資料或磁碟上的記錄檔一致性。 這個命令的好處是，它可以在 VM 為線上狀態時運作。 若要確保沒有處理序寫入備份串接集，請務必在 blob 複本之前先它取消掛接，之後再將它掛接回來。 或者，可以使用適當的方式「凍結」&quot;&quot;檔案系統。 例如，在 XFS 檔案系統使用 xfs\_freeze。

![這個螢幕擷取畫面顯示 Azure 入口網站上 vhd 容器中的 blob 清單](media/sap-hana-backup-file-level/image032.png)

這個螢幕擷取畫面顯示 Azure 入口網站上 &quot;vhd&quot; 容器中的 blob 清單。 螢幕擷取畫面顯示五個 VHD，它們連結至 SAP HANA 伺服器 VM 作為軟體 RAID，以保留 SAP HANA 備份檔案。 畫面中也顯示 5 個複本，這些是經由 blob 複製命令得到的。

![基於測試目的，SAP HANA 備份軟體 RAID 磁碟的複本已連結至應用程式伺服器 VM](media/sap-hana-backup-file-level/image033.png)

基於測試目的，SAP HANA 備份軟體 RAID 磁碟的複本已連結至應用程式伺服器 VM。

![已關閉應用程式伺服器 VM 以連結磁碟複本](media/sap-hana-backup-file-level/image034.png)

已關閉應用程式伺服器 VM 以連結磁碟複本。 啟動 VM 之後，可正確找到磁碟和 RAID (透過 UUID 掛接)。 只有遺失透過 YaST partitioner 建立的掛接點。 之後，便可在 OS 層級看到 SAP HANA 備份檔案的複本。

## <a name="copy-sap-hana-backup-files-to-nfs-share"></a>將 SAP HANA 備份檔案複製到 NFS 共用

從效能或磁碟空間的觀點來看，若要減少對 SAP HANA 系統的潛在影響，可以考慮將 SAP HANA 備份檔案儲存在 NFS 共用上。 這技術上可行，但表示要使用第二個 Azure VM 作為 NFS 共用的主機。 因為 VM 網路頻寬的緣故，它不應該是小型的 VM 大小。 合理的做法是關閉此「備份 VM」&quot;&quot;，只讓它在進行 SAP HANA 備份時執行。 寫入 NFS 共用會加重網路的負載，影響 SAP HANA 系統，但只是事後管理「備份 VM」&quot;&quot;上的備份檔案，完全不會影響 SAP HANA 系統。

![另一個 Azure VM上的 NFS 共用掛接至 SAP HANA 伺服器 VM](media/sap-hana-backup-file-level/image035.png)

為了確認 NFS 使用案例，將另一個 Azure VM上的 NFS 共用掛接至 SAP HANA 伺服器 VM。 沒有套用任何特別的 NFS 微調。

![直接備份花了 1 小時 46 分鐘](media/sap-hana-backup-file-level/image036.png)

NFS 共用是快速串接集，就像 SAP HANA 伺服器上的一樣。 不過，當寫入本機串接集時，在 NFS 共用上進行直接備份花了 1 小時 46 分鐘來，而不是 10 分鐘。

![替代方法 1 小時 43 分鐘沒快多少](media/sap-hana-backup-file-level/image037.png)

對本機等量磁碟區執行備份，並複製到 OS 層級之 NFS 共用的替代方法 (簡單的 **cp -avr** 命令) 沒快多少， 花了 1 小時 43 分鐘。

它的確可行，但效能不如 230 GB 備份測試好。 若遇到數 TB 的資料，可能會更糟。

## <a name="copy-sap-hana-backup-files-to-azure-files"></a>將 SAP HANA 備份檔案複製到 Azure 檔案

可以將 Azure 檔案共用掛接在 Azure Linux VM 內部。 [如何搭配使用 Azure 檔案儲存體與 Linux](../../../storage/files/storage-how-to-use-files-linux.md) 一文提供作法的詳細資訊。 請記住，目前 Azure 檔案共用有 5 TB 配額的限制，以及每個檔案 1 TB 的檔案大小限制。 如需儲存體限制的詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](../../../storage/common/storage-scalability-targets.md) (英文)。

不過測試顯示，SAP HANA 備份目前不能直接搭配使用這種 CIFS 掛接。 在 [SAP Note 1820529](https://launchpad.support.sap.com/#/notes/1820529) (英文) 中也提到不建議使用 CIFS。

![本圖為 SAP HANA Studio 的備份對話方塊中顯示的錯誤](media/sap-hana-backup-file-level/image038.png)

本圖顯示嘗試直接備份到掛接 CIFS 的 Azure 檔案共用時，SAP HANA Studio 的備份對話方塊中顯示的錯誤。 因此必須執行標準 SAP HANA 備份先備份到 VM 檔案系統，然後將備份檔案從該處複製到 Azure 檔案服務。

![本圖顯示複製 19 個 SAP HANA 備份檔案需要約 929 秒](media/sap-hana-backup-file-level/image039.png)

本圖顯示將 19 個 SAP HANA 備份檔案 (總計大小約 230 GB) 複製到 Azure 檔案共用需要約 929 秒。

![在 SAP HANA VM 上的來源目錄結構已複製到 Azure 檔案共用](media/sap-hana-backup-file-level/image040.png)

在這個螢幕擷取畫面中可以看到，SAP HANA VM 上的來源目錄結構已複製到 Azure 檔案共用︰一個目錄 (hana\_backup\_fsl\_15gb) 和 19 個個別的備份檔案。

在未來，當 SAP HANA 檔案備份直接支援時，將 SAP HANA 備份檔案儲存在 Azure 檔案可能是個有趣的選項。 或者，當有可能透過 NFS 掛接 Azure 檔案，且配額上限大幅高於 5 TB 時。

## <a name="next-steps"></a>後續步驟
* [Azure 虛擬機器上 SAP HANA 的備份指南](sap-hana-backup-guide.md)提供快速入門的概觀和詳細資訊。
* [以儲存體快照集為基礎的 SAP HANA 備份](sap-hana-backup-storage-snapshots.md)描述以儲存體快照集為基礎的備份選項。
* 若要了解如何建立高可用性並為 Azure 上的 SAP HANA 規劃災害復原，請參閱 [Azure 上的 SAP HANA (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md)。
