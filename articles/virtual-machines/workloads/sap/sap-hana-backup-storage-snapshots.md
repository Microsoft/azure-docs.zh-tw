---
title: 儲存體快照集為基礎的 SAP HANA 備份 | Microsoft Docs
description: 備份 Azure 虛擬機器上的 SAP HANA 有兩種主要做法，本文涵蓋以儲存體快照集為基礎的 SAP HANA 備份
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
ms.openlocfilehash: 031cb10e476ba068f7e3d7baf3b19f7703caf170
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45580029"
---
# <a name="sap-hana-backup-based-on-storage-snapshots"></a>以儲存體快照集為基礎的 SAP HANA 備份

## <a name="introduction"></a>簡介

這是與 SAP HANA 備份相關的三篇系列文章的其中一篇。 [Azure 虛擬機器上 SAP HANA 的備份指南](sap-hana-backup-guide.md)提供開始使用的概觀和資訊，[檔案層級的 SAP HANA Azure 備份](sap-hana-backup-file-level.md)涵蓋以檔案為基礎的備份選項。

在單一執行個體全方位示範系統中使用 VM 備份功能時，應該考慮進行 VM 備份，而不是在作業系統層級管理 HANA 備份。 另一個替代方法是採用 Azure blob 快照集，建立連結至虛擬機器之個別虛擬磁碟的複本，並保留 HANA 資料檔案。 但關鍵點在於，當系統正在執行中，建立 VM 備份或磁碟快照集的一致性。 請參閱相關文章 [Azure 虛擬機器上 SAP HANA 的備份指南](sap-hana-backup-guide.md)中的＜建立儲存體快照集時，SAP HANA 資料的一致性＞。 SAP HANA 有個功能支援這類的儲存體快照集。

## <a name="sap-hana-snapshots-as-central-part-of-application-consistent-backups"></a>SAP HANA 快照集是應用程式保持備份一致性的關鍵所在

SAP HANA 中有個功能支援建立儲存體快照集。 單一容器系統有所限制。 多個租用戶使用 SAP HANA MCS 的情況下，則不支援這種 SAP HANA 資料庫快照集，請參閱[建立儲存體快照集 (SAP HANA Studio)](https://help.sap.com/saphelp_hanaplatform/helpdata/en/a0/3f8f08501e44d89115db3c5aa08e3f/content.htm)。

其運作如下：

- 起始 SAP HANA 快照集，為儲存體快照做準備
- 執行儲存體快照集 (例如 Azure blob 快照集)
- 確認 SAP HANA 快照集

![這個螢幕擷取畫面顯示透過 SQL 陳述式建立的 SAP HANA 資料快照集](media/sap-hana-backup-storage-snapshots/image011.png)

這個螢幕擷取畫面顯示透過 SQL 陳述式建立的 SAP HANA 資料快照集。

![然後快照集也會出現在 SAP HANA Studio 的備份目錄中](media/sap-hana-backup-storage-snapshots/image012.png)

然後快照集也會出現在 SAP HANA Studio 的備份目錄中。

![快照集會出現在磁碟上的 SAP HANA 資料目錄中](media/sap-hana-backup-storage-snapshots/image013.png)

快照集會出現在磁碟上的 SAP HANA 資料目錄中。

當 SAP HANA 處於快照集準備模式時，執行儲存體快照集之前，您必須確定也能保證檔案系統一致性。 請參閱相關文章 [Azure 虛擬機器上 SAP HANA 的備份指南](sap-hana-backup-guide.md)中的＜建立儲存體快照集時，SAP HANA 資料的一致性＞。

完成儲存體快照集時，務必確認 SAP HANA 快照集。 有個對應的 SQL 陳述式可以執行︰BACKUP DATA CLOSE SNAPSHOT (請參閱 [BACKUP DATA CLOSE SNAPSHOT陳述式 (備份和復原)](https://help.sap.com/saphelp_hanaplatform/helpdata/en/c3/9739966f7f4bd5818769ad4ce6a7f8/content.htm)。

> [!IMPORTANT]
> 確認 HANA 快照集。 由於「寫入時複製」&quot;&quot;的特性，SAP HANA 在快照集準備模式可能需要額外的磁碟空間，而且不確認 SAP HANA 快照集就不能啟動新的備份。

## <a name="hana-vm-backup-via-azure-backup-service"></a>透過 Azure 備份服務備份 HANA VM

Azure 備份服務的備份代理程式不再適用於 Linux VM。 Windows 有 VSS 適用的功能，但是 Linux 沒有類似的功能。  若要進行檔案/目錄層級的 Azure 備份，可以將 SAP HANA 備份檔案複製到 Windows VM，然後使用該備份代理程式。 

否則，只能透過 Azure 備份服務進行完整的 Linux VM 備份。 詳細資訊請參閱 [Azure 備份中的功能概觀](../../../backup/backup-introduction-to-azure-backup.md)。

Azure 備份服務提供備份和還原 VM 的選項。 關於此服務和運作方式的詳細資訊，可在[在 Azure 中規劃 VM 備份基礎結構](../../../backup/backup-azure-vms-introduction.md)中找到。

根據這篇文章，有兩個重要的考量︰

「Linux 虛擬機器則只能進行檔案一致的備份，因為 Linux 沒有相當於 VSS 的平台。」_&quot;&quot;_

「應用程式需要在還原的資料上實作自己的「修正」機制。」_&quot;&quot;&quot;&quot;_

因此，您必須確定備份啟動時，磁碟上的 SAP HANA 是處於一致的狀態。 請參閱本文稍早的＜SAP HANA 快照集＞。 但是當 SAP HANA 保持在此快照集準備模式中時，還有潛在的問題。 如需詳細資訊，請參閱[建立儲存體快照集 (SAP HANA Studio)](https://help.sap.com/saphelp_hanaplatform/helpdata/en/a0/3f8f08501e44d89115db3c5aa08e3f/content.htm)。

這篇文章中提到︰

_&quot;「強烈建議建立儲存體快照集之後，儘速確認或放棄它。準備或建立儲存體快照集時，與快照集相關的資料會被凍結。當快照集相關資料處於凍結，資料庫中仍然可以進行變更。這類變更不會使凍結的快照集相關資料跟著變更。相反地，變更會寫入資料區域中與儲存體快照集分開的位置。變更也會寫入記錄。不過，快照集相關資料凍結愈久，資料磁碟區可以成長愈多。」&quot;_

Azure 備份會透過 Azure VM 的擴充功能來處理檔案系統的一致性。 這些擴充功能沒有獨立版本，只能結合 Azure 備份服務使用。 不過，仍然需要由它提供指令碼來建立和刪除 SAP HANA 快照集，以確保應用程式一致性。

Azure 備份有四個主要階段︰

- 執行準備指令碼：需由指令碼建立 SAP HANA 快照集
- 建立快照集
- 執行後快照集指令碼：需由指令碼刪除準備指令碼所建立的 SAP HANA
- 將資料傳輸至保存庫

如需複製這些指令碼的詳細資訊和 Azure 備份實際如何運作的詳細資訊，請參閱下列文章：

- [在 Azure 中規劃 VM 備份基礎結構](https://docs.microsoft.com/azure/backup/backup-azure-vms-introduction)
- [Azure Linux VM 的應用程式一致備份](https://docs.microsoft.com/azure/backup/backup-azure-linux-app-consistent)



到目前為止，Microsoft 尚未發佈適用於 SAP HANA 的指令碼和後快照集指令碼。 若為客戶或系統整合商，會需要建立這些指令碼，並需根據上述文件設定程序。


## <a name="restore-from-application-consistent-backup-against-a-vm"></a>從針對虛擬機器進行的應用程式一致備份還原
[從 Azure 虛擬機器備份復原檔案](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm)一文中，說明了以 Azure 備份所產生的應用程式一致備份進行還原的流程。 

> [!IMPORTANT]
> [從 Azure 虛擬機器備份復原檔案](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm)一文中，有提供清單列出使用等量磁碟區組時的例外狀況和步驟。 等量磁碟區可能是 SAP HANA 的標準虛擬機器組態。 因此，務必參閱該文，並針對文中所列的情況測試還原流程。 



## <a name="hana-license-key-and-vm-restore-via-azure-backup-service"></a>透過 Azure 備份服務的 HANA 授權金鑰和 VM 還原

Azure 備份服務為了在還原期間建立新 VM 而設計。 目前還沒有&quot;就地&quot;還原現有 Azure VM 的規劃。

![此圖顯示 Azure 入口網站中 Azure 服務的還原選項](media/sap-hana-backup-storage-snapshots/image019.png)

此圖顯示 Azure 入口網站中 Azure 服務的還原選項。 可以選擇在還原期間建立 VM，或還原磁碟。 還原磁碟後，仍然必須在其之上建立新的 VM。 每次在 Azure 上建立新的 VM，唯一 VM 識別碼就會變更 (請參閱[存取和使用 Azure VM 唯一識別碼](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/))。

![此圖顯示透過 Azure 備份服務進行還原前後的 Azure VM 唯一識別碼](media/sap-hana-backup-storage-snapshots/image020.png)

此圖顯示透過 Azure 備份服務進行還原前後的 Azure VM 唯一識別碼。 SAP 硬體機碼 (用於 SAP 授權) 會使用這個唯一的 VM 識別碼。 因此，必須在 VM 還原後安裝新的 SAP 授權。

在編寫本備份指南的期間，預覽模式中有了新的 Azure 備份功能。 可以根據之前為了 VM 備份而建立的 VM 快照集，進行檔案層級的還原。 這樣就不必部署新的 VM，因此唯一 VM 識別碼維持不變，不需要新的 SAP HANA 授權金鑰。 有關這項功能的詳細文件，將會在經過完整測試之後提供。

Azure 備份最終將會允許備份個別 Azure 虛擬磁碟，加上虛擬機器內部的檔案和目錄。 Azure 備份的主要優點，是它可以管理所有備份，讓客戶不需要做這件事。 如果需要還原，Azure 備份會選取正確的備份來使用。

## <a name="sap-hana-vm-backup-via-manual-disk-snapshot"></a>透過手動磁碟快照集進行 SAP HANA VM 備份

除了使用 Azure 備份服務，您也可以設定個別備份解決方案，透過 PowerShell 手動建立 Azure VHD 的 blob 快照集。 如需步驟說明，請參閱[以 PowerShell 使用 blob 快照集](https://blogs.msdn.microsoft.com/cie/2016/05/17/using-blob-snapshots-with-powershell/)。

它提供更大的彈性，但不能解決本文件稍早所述的問題︰

- 您仍然必須建立 SAP HANA 快照集，確定 SAP HANA 處於一致的狀態
- 即使因為有「有租用存在」錯誤而將 VM 解除配置，仍然無法覆寫作業系統磁碟。 必須要在刪除 VM之後才能覆寫，而這會導致新的唯一 VM 識別碼，故需要安裝新的 SAP 授權。

![可以只還原 Azure VM 的資料磁碟](media/sap-hana-backup-storage-snapshots/image021.png)

可以只還原 Azure VM 的資料磁碟，避免取得新的唯一 VM 識別碼進而使 SAP 授權失效的問題︰

- 測試中將兩個 Azure 資料磁碟連結至 VM，並在其上定義軟體 RAID 
- 已經利用 SAP HANA 快照集功能確認的 SAP HANA 處於一致的狀態
- 檔案系統凍結 (請參閱相關文章 [Azure 虛擬機器上 SAP HANA 的備份指南](sap-hana-backup-guide.md)中的＜建立儲存體快照集時，SAP HANA 資料的一致性＞)
- 從這兩個資料磁碟建立 blob 快照集
- 檔案系統解除凍結
- 確認 SAP HANA 快照集
- 為了還原資料磁碟，已關閉 VM，並中斷兩個磁碟的連結
- 中斷連結磁碟之後，以先前的 blob 快照集覆寫磁碟
- 然後將還原後的虛擬磁碟再次連結至 VM
- 啟動 VM 後，軟體 RAID 上的所有項目正常運作，並已回到 blob 快照集時間的狀態
- HANA 已回到 HANA 快照集的狀態

如果可以在進行 blob 快照集之前關閉 SAP HANA，程序會比較不複雜。 在此情況下，可以略過 HANA 快照集，而且如果系統上沒有在進行其他作業，也可以略過檔案系統凍結。 當需要在所有項目都在線上時執行快照集，複雜度就會增加。 請參閱相關文章 [Azure 虛擬機器上 SAP HANA 的備份指南](sap-hana-backup-guide.md)中的＜建立儲存體快照集時，SAP HANA 資料的一致性＞。

## <a name="next-steps"></a>後續步驟
* [Azure 虛擬機器上 SAP HANA 的備份指南](sap-hana-backup-guide.md)提供快速入門的概觀和詳細資訊。
* [檔案層級的 SAP HANA 備份](sap-hana-backup-file-level.md)說明以檔案為基礎的備份選項。
* 若要了解如何建立高可用性並為 Azure 上的 SAP HANA 規劃災害復原，請參閱 [Azure 上的 SAP HANA (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md)。
