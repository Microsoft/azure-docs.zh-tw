---
title: Microsoft Azure StorSimple Virtual Array 概觀 | Microsoft Docs
description: 說明 StorSimple Virtual Array，這是一個整合式儲存體解決方案，可管理內部部署虛擬陣列與 Microsoft Azure 雲端儲存體之間的儲存體工作。
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: 169c639b-1124-46a5-ae69-ba9695525b77
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 08/02/2018
ms.author: alkohli
ms.openlocfilehash: 78ed53e5e2f5d04943e6c32ddfedf037cb9e1f73
ms.sourcegitcommit: fc5555a0250e3ef4914b077e017d30185b4a27e6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39480949"
---
# <a name="introduction-to-the-storsimple-virtual-array"></a>StorSimple Virtual Array 簡介

## <a name="overview"></a>概觀

Microsoft Azure StorSimple Virtual Array 是一個整合式儲存體解決方案，可管理 Hypervisor 中執行之內部部署虛擬陣列與 Microsoft Azure 雲端儲存體之間的儲存體工作。 虛擬陣列是一個有效率、符合成本效益且易於管理的檔案伺服器或 iSCSI 伺服器解決方案，可減少許多與企業儲存體和資料保護相關聯的問題和支出。 虛擬陣列特別適合不常存取之封存資料的儲存體。

本文章提供虛擬陣列的概觀 - 以下是一些其他資源︰

* 如需最佳做法，請參閱 [StorSimple Virtual Array 的最佳做法](storsimple-ova-best-practices.md)。
* 如需 StorSimple 8000 系列裝置的概觀，請移至 [StorSimple 8000 系列：混合式雲端解決方案](storsimple-overview.md)。
* 如需 StorSimple 5000/7000 系列裝置的相關資訊，請移至 [StorSimple 線上說明](http://onlinehelp.storsimple.com/)。

虛擬陣列支援 iSCSI 或伺服器訊息區 (SMB) 通訊協定。 它會在現有 Hypervisor 基礎結構上執行，並提供雲端、雲端備份、快速還原、項目層級復原和災害復原功能的分層。

下表總結 StorSimple Virtual Array 的重要功能。

| 功能 | StorSimple Virtual Array |
| --- | --- |
| 安裝需求 |使用虛擬化基礎結構 (Hyper-V 或 VMware) |
| 可用性 |單一節點 |
| 總容量 (包括雲端) |每個虛擬陣列最多 64 TB 可用容量 |
| 本機容量 |每個虛擬陣列有 390 GB 到 6.4 TB 可用容量 (需要佈建 500 GB 到 8 TB 的磁碟空間) |
| 原生通訊協定 |iSCSI 或 SMB |
| 復原時間目標 (RTO) |iSCSI：少於 2 分鐘 (不論大小為何) |
| 復原點目標 (RPO) |每日備份和隨選備份 |
| 儲存體分層 |使用熱度對應來判斷應該分層輸入或分層輸出資料 |
| 支援 |供應商所支援的虛擬化基礎結構 |
| 效能 |會視基礎結構而異 |
| 資料行動性 |可以還原至相同裝置，或執行項目層級復原 (檔案伺服器) |
| 儲存層 |本機 Hypervisor 儲存體和雲端 |
| 共用大小 |階層式：最多 20 TB；固定在本機：最多 2 TB |
| 磁碟區大小 |階層式：500 GB 到 5 TB；固定在本機：50 GB 到 200 GB <br> 階層式磁碟區的最大本機保留為 200 GB。 |
| 快照集 |當機時保持一致 |
| 項目層級復原 |是；使用者可以從共用還原 |

## <a name="why-use-storsimple"></a>為何要使用 StorSimple？

StorSimple 會在數分鐘內將使用者和伺服器連接到 Azure 儲存體，而不需要進行任何應用程式修改。

下表說明一些 StorSimple Virtual Array 解決方案所提供的主要優點。

| 功能 | 優點 |
| --- | --- |
| 透明整合 |虛擬陣列支援 iSCSI 或 SMB 通訊協定。 本機層和雲端層之間的資料移動十分順暢，讓使用者不致察覺。 |
| 降低儲存成本 |您可以使用 StorSimple 來佈建足夠的本機儲存體，以符合目前對於最常使用的熱資料需求。 隨著儲存體需求的成長，StorSimple 會將冷資料分層至符合成本效益的雲端儲存體。 資料會在傳送到雲端之前進行重複資料刪除並加以壓縮，以進一步降低儲存體需求和費用。 |
| 簡化儲存管理 |StorSimple 會在雲端中使用 StorSimple 裝置管理員提供集中式管理來管理多個裝置。 |
| 改進災害復原和提高合規性 |StorSimple 藉由立即還原中繼資料，並視需要還原資料，有助於更快速地進行災害復原。 這表示正常作業可以在最少干擾的情況下繼續執行。 |
| 資料行動性 |您可以從其他站台存取分層至雲端的資料，以供復原和移轉使用。 請注意，您只可以將資料還原到原始虛擬陣列。 不過，您可以使用災害復原功能將整個虛擬陣列還原到另一個虛擬陣列。 |

## <a name="storsimple-workload-summary"></a>StorSimple 工作負載摘要

下表顯示所支援 StorSimple 工作負載的摘要。

|案例     |工作負載     |支援      |限制               |
|-------------|-------------|---------------|---------------------------|
|遠端辦公室/分公司 (ROBO)  |檔案共用     |是      |請參閱[檔案伺服器的上限](storsimple-ova-limits.md)。<br></br>請參閱[支援 SMB 版的系統需求](storsimple-ova-system-requirements.md)。| 所有版本     |
|雲端封存  |封存檔案共用     |是      |請參閱[檔案伺服器的上限](storsimple-ova-limits.md)。<br></br>請參閱[支援 SMB 版的系統需求](storsimple-ova-system-requirements.md)。| 所有版本     |

StorSimple Virtual Array 最適合不常存取的資料。 雖然虛擬陣列具有本機快取可提高效能，但使用者應該假設裝置為儲存體 (雲端) 最低層的檔案提供服務。 每個虛擬陣列都能以大約 100 Mbps 的速度寫入及讀取 Azure 儲存體。 該連結會在進入裝置的所有要求之間共用，並會形成瓶頸，如下圖所示。

![雲端封存](./media/storsimple-ova-overview/cloud-archiving.png)

當多個使用者同時存取虛擬陣列時，他們會共用 Azure 連線，導致效能降低。 不保證每個使用者的效能，且裝置會在個別要求到達時處理它們。

StorSimple Virtual Array 不適合需要高可用性的工作負載。 虛擬陣列是安裝軟體更新時會遇到停機時間的單一節點裝置。 系統管理員應該規劃每年 3-4 次 30 分鐘的維護期間。

## <a name="workflows"></a>工作流程

StorSimple Virtual Array 特別適用於下列工作流程：

* [雲端儲存體管理](#cloud-based-storage-management)
* [與位置無關的備份](#location-independent-backup)
* [資料保護和災害復原](#data-protection-and-disaster-recovery)

### <a name="cloud-based-storage-management"></a>雲端儲存體管理
您可以使用 Azure 入口網站中執行的 StorSimple 裝置管理員服務，來管理多個裝置上和多個位置中所儲存的資料。 這在分散式分公司案例中特別有用。 請注意，您必須建立個別的 StorSimple 裝置管理員服務執行個體來管理虛擬陣列和實體 StorSimple 裝置。 另請注意，虛擬陣列現在會使用新的 Azure 入口網站而不是 Azure 傳統入口網站。

![雲端儲存體管理](./media/storsimple-ova-overview/cloud-based-storage-management.png)

### <a name="location-independent-backup"></a>與位置無關的備份
運用虛擬陣列，雲端快照集可針對磁碟區或共用提供與位置無關的時間點複本。 預設會啟用雲端快照集，並且無法停用。 所有磁碟區和共用都是透過單一每日備份原則同時備份，需要時，也可以採用其他臨機操作備份。

### <a name="data-protection-and-disaster-recovery"></a>資料保護和災害復原
虛擬陣列支援下列資料保護和災害復原案例：

* **磁碟區或共用還原** – 使用還原作為新的工作流程來復原磁碟區或共用。 這種方法可用來復原整個磁碟區或共用。
* **項目層級復原** – 共用可簡化對最新備份的存取。 您可以輕鬆地從雲端中可用的特殊 .backup 資料夾復原個別檔案。 這個還原功能是由使用者所驅動，而且不需要系統管理介入。
* **災害復原** – 使用容錯移轉功能來將所有磁碟區或共用復原到新的虛擬陣列。 您可以建立新的虛擬陣列，並向 StorSimple 裝置管理員服務進行註冊，然後容錯移轉原始虛擬陣列。 新的虛擬陣列之後將採用佈建的資源。

## <a name="storsimple-virtual-array-components"></a>StorSimple Virtual Array 元件

虛擬陣列包括下列元件：

* [虛擬陣列](#virtual-array) – 一種以虛擬化環境或 Hypervisor 中所佈建之虛擬機器為基礎的混合式雲端存放裝置。
* [StorSimple 裝置管理員服務](#storsimple-device-manager-service) – 一種 Azure 入口網站擴充功能，可讓您透過可從不同地理位置存取的單一 Web 介面，管理一或多部 StorSimple 裝置。 您可以使用 StorSimple 裝置管理員服務來建立和管理服務、檢視和管理裝置及警示，以及管理磁碟區、共用和現有快照集。
* [本機 Web 使用者介面](#local-web-user-interface) – 一種 Web 型 UI，可用來設定裝置，讓它可以連線到區域網路，然後向「StorSimple 裝置管理員」服務註冊裝置。 
* [命令列介面](#command-line-interface) – 一種 Windows PowerShell 介面，可用來在虛擬陣列上啟動支援工作階段。
  下列各節將更詳細描述每個元件，並說明解決方案如何排列資料、配置儲存體、加快儲存體管理速度以及資料保護。

### <a name="virtual-array"></a>虛擬陣列

虛擬陣列是一種單一節點儲存體解決方案，可提供主要儲存體、管理與雲端儲存體的通訊，並協助確保所有儲存在裝置之資料的安全性和機密性。

在可供下載的模型中提供虛擬陣列。 裝置上的虛擬陣列最大容量是 6.4 TB (基礎儲存體需求為 8 TB) 和 64 TB (包括雲端儲存體)。

虛擬陣列具有下列功能：

* 符合成本效益。 使用現有的虛擬化基礎結構，而且可以部署於現有 Hyper-V 或 VMware Hypervisor 上。
* 位於資料中心，而且可以設定為 iSCSI 伺服器或檔案伺服器。
* 與雲端整合。
* 備份儲存在雲端中，可以加速災害復原並簡化項目層級復原 (ILR)。
* 您可以將更新套用至虛擬陣列，就像將它們套用至實體裝置一樣。

> [!NOTE]
> 無法展開虛擬陣列。 因此，建立虛擬陣列時，請一定要佈建足夠的儲存體。

### <a name="storsimple-device-manager-service"></a>StorSimple 裝置管理員服務

Microsoft Azure StorSimple 提供可讓您集中管理 StorSimple 儲存體的 Web 架構使用者介面 (StorSimple 裝置管理員服務)。 您可以使用 StorSimple 裝置管理員服務來執行下列工作：

* 透過單一服務管理多個 StorSimple Virtual Array。
* 進行和管理 StorSimple Virtual Array 的安全性設定。 (雲端中的加密與 Microsoft Azure API 相依。)
* 設定儲存體帳戶認證和屬性。
* 設定和管理磁碟區或共用。
* 備份和還原磁碟區或共用上的資料。
* 監視效能。
* 檢閱系統設定，並識別可能發生的問題。

您可以使用 StorSimple 裝置管理員服務來執行虛擬陣列的每日管理。

如需詳細資訊，請移至[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-virtual-array-manager-service-administration.md)。

### <a name="local-web-user-interface"></a>本機 Web 使用者介面

虛擬陣列包括一個 Web 式 UI，用於單次設定裝置，並向 StorSimple 裝置管理員服務註冊裝置。 您可以使用它來關閉和重新啟動虛擬陣列、執行診斷測試、更新軟體、變更裝置系統管理員密碼、檢視系統記錄檔，以及連絡 Microsoft 支援服務發出服務要求。

如需有關使用 Web 式 UI 的資訊，請移至 [使用 Web 式 UI 來管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

### <a name="command-line-interface"></a>命令列介面

包括的 Windows PowerShell 介面可讓您起始與 Microsoft 支援服務的支援工作階段，讓他們協助您疑難排解並解決虛擬陣列上可能發生的問題。

## <a name="storage-management-technologies"></a>儲存體管理技術

除了虛擬陣列和其他元件以外，StorSimple 解決方案還會使用下列軟體技術來提供重要資料的快速存取、減少儲存體使用，以及保護虛擬陣列上所儲存的資料：

* [自動儲存體分層](#automatic-storage-tiering) 
* [固定在本機的共用和磁碟區](#locally-pinned-shares-and-volumes)
* [分層或備份至雲端之資料的重複資料刪除和壓縮](#deduplication-and-compression-for-data-tiered/backed-up-to-the-cloud) 
* [排程和隨選備份](#scheduled-and-on-demand-backups)

### <a name="automatic-storage-tiering"></a>自動儲存體分層
虛擬陣列使用新的分層機制來管理跨虛擬陣列和雲端儲存的資料。 只有兩層：本機虛擬陣列和 Azure 雲端儲存體。 StorSimple Virtual Array 會根據熱度圖自動排列分層中的資料，以追蹤目前使用量、使用年限以及與其他資料的關聯性。 最常使用 (最熱) 的資料會儲存在本機，而不常使用和非使用中資料會自動移轉至雲端  (所有備份都是儲存在雲端中)。StorSimple 會隨著使用模式變更而調整並重新排列資料和儲存體指派。 例如，某些資訊在經過一段時間之後可能會變成不常使用。 資訊逐漸變成不常使用時，會分層輸出至雲端。 如果相同的資料再次變成使用中，則會分層輸入至儲存體陣列。

特定階層式共用或磁碟區的資料保證其自身本機層的空間 (大約為該共用或磁碟區已佈建總空間的 10%)。 雖然這會降低虛擬陣列上該共用或磁碟區的可用儲存體，但是可確保某個共用或磁碟區的分層不受其他共用或磁碟區的分層需求的影響。 因此，某個共用或磁碟區上的極忙碌工作負載無法將所有其他工作負載強制到雲端。

無論磁碟區大小為何，針對 iSCSI 所建立之分層磁碟區的最大本機保留區均為 200 GB。

![自動儲存體分層](./media/storsimple-ova-overview/automatic-storage-tiering.png)

> [!NOTE]
> 您可以指定本機固定的磁碟區，在此情況下，資料會保留在虛擬陣列上，而且絕不會分層至雲端。 如需詳細資訊，請移至 [固定在本機的共用和磁碟區](#locally-pinned-shares-and-volumes)。


### <a name="locally-pinned-shares-and-volumes"></a>固定在本機的共用和磁碟區

您可以建立適當的固定在本機共用和磁碟區。 這項功能可確保重要應用程式所需的資料保留在虛擬陣列中，而且絕不會分層至雲端。 固定在本機的共用和磁碟區具有下列功能：

* 不受限於雲端延遲或連線問題。
* 依然受益於 StorSimple 雲端備份和災害復原功能。

您可以將固定在本機的共用或磁碟區還原為階層式，或將階層式共用或磁碟區還原為固定在本機。 

如需有關固定在本機的磁碟區的詳細資訊，請移至[使用 StorSimple 裝置管理員服務來管理磁碟區](storsimple-virtual-array-manage-volumes.md)。

### <a name="deduplication-and-compression-for-data-tiered-or-backed-up-to-the-cloud"></a>分層或備份至雲端之資料的重複資料刪除和壓縮

StorSimple 會使用重複資料刪除和資料壓縮，來進一步降低雲端中的儲存體需求。 重複資料刪除減少整體儲存資料量的方式是刪除儲存資料集中的重複資料。 當資訊變更時，StorSimple 會忽略未變更的資料，而只擷取變更。 此外，StorSimple 還會透過識別並移除重複資訊來減少儲存的資料量。

> [!NOTE]
> 虛擬陣列上所儲存的資料不會進行重複資料刪除或壓縮。 所有重複資料刪除和壓縮只會在將資料傳送至雲端之前發生。

### <a name="scheduled-and-on-demand-backups"></a>排程和隨選備份

StorSimple 資料保護功能可讓您建立隨選備份。 此外，預設備份排程可確保每日備份資料。 備份採用儲存在雲端的累加快照形式。 快照集 (僅記錄自上次備份後的變更) 可以快速進行建立和還原。 這些快照集在災害復原案例中至關重要，因為這些快照集會取代次要儲存體系統 (例如磁帶備份)，並讓您將資料還原到資料中心或在必要時還原至其他網站。

## <a name="managing-personal-information"></a>管理個人資訊

虛擬系列的 StorSimple 裝置管理員會收集兩個主要執行個體中的個人資訊：
 - 警示使用者設定，其中會設定使用者的電子郵件地址。 系統管理員可以清除此資訊。 
 - 可以存取位於共用上的資料的使用者。 顯示可存取共用資料之使用者的清單，並且可匯出。 刪除共用時，也會一併刪除這份清單。

如需詳細資訊，請在信任中心[檢閱 Microsoft 隱私權原則](https://www.microsoft.com/trustcenter)。

## <a name="next-steps"></a>後續步驟

了解如何[準備虛擬陣列入口網站](storsimple-virtual-array-deploy1-portal-prep.md)。
