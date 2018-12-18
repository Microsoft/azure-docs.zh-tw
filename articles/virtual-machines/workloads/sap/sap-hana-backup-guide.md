---
title: Azure 虛擬機器上的 SAP HANA 備份指南 | Microsoft Docs
description: SAP HANA 備份指南說明備份 Azure 虛擬機器上 SAP HANA 的兩種主要做法
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
ms.openlocfilehash: 9d72bc885bdaaed521042df236dd722b80533186
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37866996"
---
# <a name="backup-guide-for-sap-hana-on-azure-virtual-machines"></a>Azure 虛擬機器上的 SAP HANA 備份指南

## <a name="getting-started"></a>開始使用

Azure 虛擬機器上執行的 SAP HANA 備份指南只描述 Azure 相關主題。 如需一般 SAP HANA 備份的相關主題，請閱讀 SAP HANA 文件 (請參閱本文稍後的＜SAP HANA 備份文件＞)。

本文的重點是備份 Azure 虛擬機器上 SAP HANA 的兩種主要做法：

- HANA 備份至 Azure Linux 虛擬機器中的檔案系統 (請參閱[檔案層級的 SAP HANA Azure 備份](sap-hana-backup-file-level.md))
- 手動使用 Azure 儲存體 Blob 快照集功能或使用 Azure 備份服務，進行以儲存體快照集為基礎的 HANA 備份 (請參閱[以儲存體快照集為基礎的 HANA 備份](sap-hana-backup-storage-snapshots.md))

SAP HANA 提供備份 API，可讓第三方備份工具直接與 SAP HANA 整合。 (這已超出本指南的範圍。)根據此 API，目前 SAP HANA 與 Azure 備份服務沒有任何直接整合。

各種 Azure VM 類型都正式支援 SAP HANA (例如 Azure M 系列)。 如需完整 SAP HANA 認證 Azure VM 清單，請參閱 [Find Certified IaaS Platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) (尋找認證 IaaS 平台)。 這篇文章將會更新為新的供應項目，您就可以使用 Azure 上的 SAP HANA。

Azure 上另外還有一個 SAP HANA 的混合式解決方案，其中的 SAP HANA 是在實體伺服器上非虛擬化的執行。 不過，此 SAP HANA Azure 備份指南涵蓋純 Azure 環境，即 SAP HANA 在 Azure VM 中執行，而不是 SAP HANA 在「大型執行個體」&quot;&quot;上執行。針對此備份解決方案在「大型執行個體」&quot;&quot;上執行以儲存體快照集為基礎的備份，如需詳細資訊請參閱 [Azure 上的 SAP HANA (大型執行個體) 概觀和架構](hana-overview-architecture.md)。

Azure 上支援的 SAP 產品資訊可在 [SAP Note 1928533](https://launchpad.support.sap.com/#/notes/1928533) 之中找到。

接下來的三張圖提供目前使用原生 Azure 功能的 SAP HANA 備份選項的概觀，也顯示三個可能的未來備份案例。 相關的文章[檔案層級的 SAP Hana Azure 備份](sap-hana-backup-file-level.md)和[以儲存體快照集為基礎的 SAP Hana 備份](sap-hana-backup-storage-snapshots.md)說明這些選項的詳細資料，包括數 TB 的 SAP Hana 備份的大小和效能考量。

![此圖顯示儲存目前 VM 狀態的兩種做法](media/sap-hana-backup-guide/image001.png)

此圖顯示儲存目前 VM 狀態的兩種做法：透過 Azure 備份服務或 VM 磁碟的手動快照集。 採取這個做法，不需要管理 SAP HANA 備份。 磁碟快照集案例的挑戰，是檔案系統的一致性，和應用程式一致的磁碟狀態。 一致性主題會在本文稍後的＜建立儲存體快照集時，SAP HANA 資料的一致性＞一節中討論。 與 SAP HANA 備份相關的 Azure 備份服務功能和限制也會在本文稍後討論。

![此圖顯示在 VM 中建立 SAP HANA 檔案備份的選項](media/sap-hana-backup-guide/image002.png)

此圖顯示在 VM 中建立 SAP HANA 檔案備份的選項，然後再使用不同的工具將 HANA 備份檔案儲存在其他地方。 建立 HANA 備份需要的時間比以快照集為基礎的備份解決方案更多，但具有整體性與一致性的優點。 本文稍後將提供更詳細的說明。

![此圖顯示未來可能的 SAP HANA 備份案例](media/sap-hana-backup-guide/image003.png)

此圖顯示未來可能的 SAP HANA 備份案例。 如果 SAP HANA 允許從次要複寫建立備份，會加入額外的備份策略選項。 根據 SAP HANA wiki 文章，目前仍不可能辦到︰

_&quot;是否可能在次要端建立備份？_

_否，目前您只能在主要端上建立資料和記錄的備份。若有啟用自動記錄備份，在次要端接管之後，會自動將記錄備份寫入那裡。&quot;_

## <a name="sap-resources-for-hana-backup"></a>HANA 備份的 SAP 資源

### <a name="sap-hana-backup-documentation"></a>SAP HANA 備份文件

- [SAP HANA 管理簡介](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.00/en-US)
- [規劃備份和復原策略](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm)
- [使用 ABAP DBACOCKPIT 排程 HANA 備份](http://www.hanatutorials.com/p/schedule-hana-backup-using-abap.html)
- [排程資料備份 (SAP HANA Cockpit)](http://help.sap.com/saphelp_hanaplatform/helpdata/en/6d/385fa14ef64a6bab2c97a3d3e40292/frameset.htm)
- [SAP Note 1642148](https://launchpad.support.sap.com/#/notes/1642148) 中的 SAP HANA 備份常見問題集
- [SAP Note 2039883](https://launchpad.support.sap.com/#/notes/2039883) 中的 SAP HANA 資料庫和儲存體快照集的常見問題集
- [SAP Note 1820529](https://launchpad.support.sap.com/#/notes/1820529) 中的不適用於備份和復原的網路檔案系統

### <a name="why-sap-hana-backup"></a>為什麼要使用 SAP HANA 備份？

Azure 儲存體提供現成的可用性和可靠性 (如需有關 Azure 儲存體的詳細資訊，請參閱 [Microsoft Azure 儲存體簡介](../../../storage/common/storage-introduction.md))。

備份&quot;&quot;的下限取決於 Azure SLA，將 SAP HANA 資料和記錄檔案保留在連結至 SAP HANA 伺服器 VM 的 Azure VHD 上。 這個方法可涵蓋 VM 失敗，但不包含對 SAP HANA 資料和記錄檔案可能造成的損害，或意外刪除資料或檔案之類的邏輯錯誤。 備份也是基於合規性或法律因素而有其必要。 簡單地說，一定需要 SAP HANA 備份。

### <a name="how-to-verify-correctness-of-sap-hana-backup"></a>如何確認 SAP HANA 備份的正確性
使用儲存體快照集時，建議在不同的系統上執行測試還原。 這個方法能確保備份正確，以及備份和還原的內部程序如預期般運作。 這對內部部署來說是重大的障礙，但在雲端則能輕鬆達成，只要針對此目的暫時提供必要的資源。

請記住，光是簡單執行還原，然後確認 HANA 有在執行是不夠的。 理想情況是執行資料表一致性檢查，以確保還原的資料庫沒問題。 SAP HANA 提供數種一致性檢查，如 [SAP Note 1977584](https://launchpad.support.sap.com/#/notes/1977584) 中所述。

有關資料表一致性檢查的相關資訊，也可以在 SAP 網站上的[資料表和目錄一致性檢查](http://help.sap.com/saphelp_hanaplatform/helpdata/en/25/84ec2e324d44529edc8221956359ea/content.htm#loio9357bf52c7324bee9567dca417ad9f8b)找到。

若是進行標準檔案備份，則不需要測試還原。 有兩個 SAP HANA 工具可協助您檢查還原可使用的備份︰hdbbackupdiag 和 hdbbackupcheck。 如需有關這些工具的資訊，請參閱[手動檢查是否有可能復原](https://help.sap.com/saphelp_hanaplatform/helpdata/en/77/522ef1e3cb4d799bab33e0aeb9c93b/content.htm)。

### <a name="pros-and-cons-of-hana-backup-versus-storage-snapshot"></a>比較 HANA 備份與儲存體快照集的優缺點

SAP 不特別偏好 HANA 備份或儲存體快照集之中的任何一個。 它會列出其各別的優缺點，讓您視情況和可用的儲存技術決定要使用哪一個 (請參閱[規劃備份和復原策略](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm))。

在 Azure 上，請注意一件事：Azure blob 快照集功能不保證檔案系統的一致性 (請參閱[以 PowerShell 使用 blob 快照集](https://blogs.msdn.microsoft.com/cie/2016/05/17/using-blob-snapshots-with-powershell/))。 下一節＜建立儲存體快照集時，SAP HANA 資料的一致性＞將討論此功能的考量事項。

此外，您必須了解經常使用 blob 快照集對費用的影響，如這篇文章中所述︰[了解快照如何產生費用](/rest/api/storageservices/understanding-how-snapshots-accrue-charges)— 它不像使用 Azure 虛擬磁碟那麼明顯。

### <a name="sap-hana-data-consistency-when-taking-storage-snapshots"></a>建立儲存體快照集時，SAP HANA 資料的一致性

建立儲存體快照集時，檔案系統和應用程式的一致性是複雜的問題。 若要避免問題，最簡單的方法是關閉 SAP HANA，或甚至是整台虛擬機器。 關機在示範或原型，或甚至開發系統上也許可行，但不是生產系統的選項。

在 Azure 上，必須記住：Azure blob 快照集功能不保證檔案系統的一致性。 不過，只要僅有單一虛擬磁碟，使用SAP HANA 快照集功能便能讓一致性正常運作。 但即使是使用單一磁碟，也必須檢查其他項目。 [SAP Note 2039883](https://launchpad.support.sap.com/#/notes/2039883) 中有關於透過儲存體快照集進行 SAP HANA 備份的重要資訊。 例如其中提到，若使用 XFS 檔案系統，就必須在開始儲存體快照集之前執行 **xfs\_freeze**，才可以確保一致性 (如需有關 **xfs\_freeze** 的詳細資訊，請參閱 [xfs\_freeze(8) -Linux man page](https://linux.die.net/man/8/xfs_freeze))。

當單一檔案系統跨多個磁碟/磁碟區的情況下，一致性的主題會變得更具挑戰性。 例如，使用 mdadm 或 LVM 和串接。 上述 SAP Note 中描述︰

「但是請記住，在建立每個 SAP HANA 資料磁碟區的儲存體快照集時，儲存體系統必須保證 I/O 一致性，也就是 SAP HANA 服務專用資料磁碟區的快照必須是不可部分完成的作業。」_&quot;&quot;_

假設有個橫跨四個 Azure 虛擬磁碟的 XFS 檔案系統，下列步驟提供可表示 HANA 資料區域的一致快照集︰

- 準備 HANA 快照集
- 凍結檔案系統 (例如，使用 **xfs\_freeze**)
- 在 Azure 上建立所有必要的 blob 快照集
- 取消凍結檔案系統
- 確認 HANA 快照集

為了安全起見，建議在所有情況下使用上述程序，不論是哪種檔案系統。 或者，如果是單一磁碟或串接，透過 mdadm 或 LVM 跨多個磁碟。

確認 HANA 快照集很重要。 由於「複製時寫入」的特性&quot;&quot;，在此快照集準備模式中，SAP HANA 可能不需要額外的磁碟空間。 而且除非確認 SAP HANA 快照集，否則不能開始新的備份。

Azure Backup 服務使用 Azure VM 的擴充功能來處理檔案系統的一致性。 這些 VM 擴充功能無法獨立使用。 您還是必須管理 SAP HANA 一致性。 如需詳細資訊，請參閱相關文章[檔案層級的 SAP HANA Azure 備份](sap-hana-backup-file-level.md)。

### <a name="sap-hana-backup-scheduling-strategy"></a>SAP HANA 備份排程策略

SAP HANA 文章[規劃備份和復原策略](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm)說明執行備份的基本規劃︰

- 儲存體快照集 (每天)
- 使用檔案或 bacint 格式進行完整的資料備份 (每週一次)
- 自動記錄備份

您可以選擇完全不進行儲存體快照集；可利用 HANA 差別備份來準備它們，例如增量或差異備份 (請參閱[差別備份](https://help.sap.com/saphelp_hanaplatform/helpdata/en/c3/bb7e33bb571014a03eeabba4e37541/content.htm))。

HANA 管理指南中提供範例清單。 它建議用以下的備份順序來將 SAP HANA 復原至特定時間點︰

1. 完整資料備份
2. 差異備份
3. 增量備份 1
4. 增量備份 2
5. 記錄備份

至於應該何時進行特定類型的備份以及發生頻率的實際排程，不太可能有通用的指導方針 — 這完全因客戶而異，而且與系統中發生多少資料變更有關。 就 SAP 而言，基本建議是 - 可視為一般性準則 - 每週進行一次完整的 HANA 備份。
有關記錄備份，請參閱 SAP HANA 文件[記錄備份](https://help.sap.com/saphelp_hanaplatform/helpdata/en/c3/bb7e33bb571014a03eeabba4e37541/content.htm)。

SAP 也建議您「打掃」備份目錄，以免它無止盡地增加 (請參閱[打掃備份目錄和備份儲存體](http://help.sap.com/saphelp_hanaplatform/helpdata/en/ca/c903c28b0e4301b39814ef41dbf568/content.htm))。

### <a name="sap-hana-configuration-files"></a>SAP HANA 組態檔

如 [SAP Note 1642148](https://launchpad.support.sap.com/#/notes/1642148) 中的常見問題集所述，SAP HANA 組態檔不是標準的 HANA 備份的一部分。 它們不是還原系統的必要項目。 在還原之後，可以手動變更 HANA 組態。 如果想要在還原程序中取得相同的自訂設定，就必須另外備份 HANA 組態檔。

如果標準 HANA 備份是放在專用的 HANA 備份檔案系統，也可以將組態檔複製到相同的備份檔案系統，然後將所有東西一起複製的到最終的目的地儲存體，例如非經常性存取的 blob 儲存體。

### <a name="sap-hana-cockpit"></a>SAP HANA Cockpit

SAP HANA Cockpit 提供透過瀏覽器監視和管理 SAP HANA 的可能性。 它也允許您處理 SAP HANA 備份，因此，可作為 SAP HANA Studio 和 ABAP DBACOCKPIT 的替代方案 (如需詳細資訊請參閱 [SAP HANA Cockpit](https://help.sap.com/saphelp_hanaplatform/helpdata/en/73/c37822444344f3973e0e976b77958e/content.htm))。

![此圖顯示 SAP HANA Cockpit 資料庫系統管理畫面](media/sap-hana-backup-guide/image004.png)

此圖顯示 SAP HANA Cockpit 資料庫系統管理畫面，左側是備份圖格。 查看備份圖格需有具備適當使用者權限的登入帳戶。

![備份正在進行時，可以在 SAP HANA Cockpit 中監視備份](media/sap-hana-backup-guide/image005.png)

備份正在進行時，可以在 SAP HANA Cockpit 中監視備份，而且備份完成後會提供所有的備份詳細資料。

![在 Azure SLES 12 VM 及 Gnome 桌面上使用 Firefox 的範例](media/sap-hana-backup-guide/image006.png)

之前的螢幕擷取畫面是從 Azure Windows VM 上取得。 這一個是 Gnome 桌面搭配 Azure SLES 12 VM 使用 Firefox 的範例。 圖中顯示在 SAP HANA Cockpit 中定義 SAP HANA 備份排程的選項。 您也可以看到，它建議以日期/時間作為備份檔案的前置詞。 在 SAP HANA Studio 中，執行完整檔案備份時的預設前置詞是 &quot;COMPLETE\_DATA\_BACKUP&quot;。 建議使用唯一的前置詞。

### <a name="sap-hana-backup-encryption"></a>SAP HANA 備份的加密

SAP HANA 提供加密資料和記錄的功能。 如果 SAP HANA 資料和記錄未加密，則其備份也不會加密。 客戶可自由決定是否使用其他的第三方解決方案來加密 SAP HANA 備份。 若要深入了解 SAP HANA 加密，請參閱[資料和記錄磁碟區加密](https://help.sap.com/saphelp_hanaplatform/helpdata/en/dc/01f36fbb5710148b668201a6e95cf2/content.htm)。

在 Microsoft Azure 上，客戶可以使用 IaaS VM 加密功能進行加密。 例如，可以使用連結至 VM 的專用資料磁碟 (只用來儲存 SAP HANA 備份)，然後製作這些磁碟的複本。

Azure 備份服務可以處理加密過的 VM/磁碟 (請參閱[如何使用 Azure 備份來備份及還原加密的虛擬機器](../../../backup/backup-azure-vms-encryption.md))。

另一個選項是維護未經加密的 SAP HANA VM 及其磁碟，並將 SAP HANA 備份檔案儲存在已啟用加密的儲存體帳戶 (請參閱 [Azure 儲存體服務待用資料加密](../../../storage/common/storage-service-encryption.md))。

## <a name="test-setup"></a>測試設定

### <a name="test-virtual-machine-on-azure"></a>測試 Azure 上的虛擬機器

若要執行測試，下列備份/還原測試使用 Azure GS5 VM 中的 SAP HANA 安裝。 原則會與 M 系列 VM 相同。

![此圖為 HANA 測試 VM 的 Azure 入口網站部分概觀](media/sap-hana-backup-guide/image007.png)

此圖為 HANA 測試 VM 的 Azure 入口網站部分概觀。

### <a name="test-backup-size"></a>測試備份大小

![此圖取自 HANA Studio 中的備份主控台，顯示 HANA 索引伺服器的備份檔案大小為 229 GB。](media/sap-hana-backup-guide/image008.png)

為了產生實際的效能資料，虛擬的資料表已填滿資料，資料大小總計超過 200 GB。 圖片取自 HANA Studio 中的備份主控台，顯示 HANA 索引伺服器的備份檔案大小為 229 GB。 測試中使用 SAP HANA Studio 的預設備份前置詞 "COMPLETE_DATA_BACKUP"。 在實際生產系統中，應該定義更實用的前置詞。 SAP HANA Cockpit 的建議是日期/時間。

### <a name="test-tool-to-copy-files-directly-to-azure-storage"></a>測試工具直接將檔案複製到 Azure 儲存體

為了將 SAP HANA 備份檔案直接傳送到 Azure blob 儲存體或 Azure 檔案共用，我們使用 blobxfer 工具，因為這兩個目標它都支援，而且因為它有命令列介面可以輕鬆地整合到自動化指令碼。 blobxfer 工具可在 [GitHub](https://github.com/Azure/blobxfer) 取得。

### <a name="test-backup-size-estimation"></a>測試備份大小估計

請務必估計 SAP HANA 備份的大小。 這項估計有助於增進效能，因為檔案複製期間採用平行處理原則，藉由定義若干備份檔案的大小上限，可以改善效能。 (詳情稍後在本文中會加以說明。)您也必須決定要執行完整備份或差別備份 (增量或差異)。

幸運的是，有個簡單的 SQL 陳述式可以估計備份檔案的大小︰**select \* from M\_BACKUP\_SIZE\_ESTIMATIONS** (請參閱[估計檔案系統資料備份所需的空間](https://help.sap.com/saphelp_hanaplatform/helpdata/en/7d/46337b7a9c4c708d965b65bc0f343c/content.htm))。

![這個 SQL 陳述式的輸出幾乎完全符合磁碟上完整資料備份的實際大小](media/sap-hana-backup-guide/image009.png)

在測試系統中，這個 SQL 陳述式的輸出幾乎完全符合磁碟上完整資料備份的實際大小。

### <a name="test-hana-backup-file-size"></a>測試 HANA 備份檔案大小

![HANA Studio 備份主控台可讓您限制 HANA 備份檔案的檔案大小上限](media/sap-hana-backup-guide/image010.png)

HANA Studio 備份主控台可讓您限制 HANA 備份檔案的檔案大小上限。 在範例環境中，此功能可以得到多個較小的備份檔案，而不是一個大型的 230 GB 備份檔案。 較小的檔案大小對效能的影響顯著 (請參閱相關文章[檔案層級的 SAP HANA Azure 備份](sap-hana-backup-file-level.md))。

## <a name="summary"></a>總結

根據測試結果，下表列出備份 Azure 虛擬機器上執行的 SAP HANA 資料庫時，使用不同方案的優缺點。

**先將 SAP HANA 備份至檔案系統，再將備份檔案複製到備份目的地**

|解決方法                                           |優點                                 |缺點                                  |
|---------------------------------------------------|-------------------------------------|--------------------------------------|
|將 HANA 備份保留在 VM 磁碟上                      |沒有額外的管理工作     |耗盡本機 VM 磁碟空間           |
|用 blobxfer 工具將備份檔案複製到 blob 儲存體 |以平行處理原則複製多個檔案、可選擇使用非經常性存取的 blob 儲存體 | 需額外維護工具和自訂指令碼 | 
|透過 Powershell 或 CLI 進行 blob 複製                    |不需要額外的工具，透過 Azure Powershell 或 CLI 即可完成 |手動程序、客戶必須負責處理指令碼和管理複製的 blob 以供還原|
|複製到 NFS 共用                                  |在其他 VM 上進行備份檔案的後置處理，不會影響 HANA 伺服器|複製程序緩慢|
|用 blobxfer 複製到 Azure 檔案服務                |不會耗盡本機 VM 磁碟上的空間|HANA 備份不支援直接寫入、檔案共用目前的大小限制為 5 TB|
|Azure 備份代理程式                                 | 會是較佳的解決方案         | 目前無法在 Linux 上使用    |



**以儲存體快照集為基礎的 SAP HANA 備份**

|解決方法                                           |優點                                 |缺點                                  |
|---------------------------------------------------|-------------------------------------|--------------------------------------|
|Azure 備份服務                               | 允許以 blob 快照集為基礎備份 VM | 如果不使用檔案層級還原，則需要為還原程序建立新的 VM，這表示需要新的 SAP HANA 授權金鑰|
|手動 blob 快照集                              | 可彈性地建立和還原特定 VM 磁碟，而不需變更唯一 VM 識別碼|必須由客戶完成所有的手動作業|

## <a name="next-steps"></a>後續步驟
* [檔案層級的 SAP HANA Azure 備份](sap-hana-backup-file-level.md)說明以檔案為基礎的備份選項。
* [以儲存體快照集為基礎的 SAP HANA 備份](sap-hana-backup-storage-snapshots.md)描述以儲存體快照集為基礎的備份選項。
* 若要了解如何建立高可用性並為 Azure 上的 SAP HANA 規劃災害復原，請參閱 [Azure 上的 SAP HANA (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md)。
