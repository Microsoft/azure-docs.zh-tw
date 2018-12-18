---
title: StorSimple Virtual Array Update 0.4 版本資訊| Microsoft Docs
description: 描述執行 Update 0.4 之 StorSimple Virtual Array 的重大未決問題和解決方式。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/05/2017
ms.author: alkohli
ms.openlocfilehash: cc2b025b7f3e28954c7f95409ffab03e5cbcf13d
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110596"
---
# <a name="storsimple-virtual-array-update-04-release-notes"></a>StorSimple Virtual Array Update 0.4 版本資訊

## <a name="overview"></a>概觀

下列版本資訊指出 Microsoft Azure StorSimple Virtual Array 更新的重大未決問題和已解決問題。

版本資訊會持續更新，並在發現需要提出因應措施的重大問題時有所增補。 部署 StorSimple Virtual Array 之前，請仔細檢閱版本資訊中所含的資訊。

Update 0.4 與軟體版本 **10.0.10289.0** 對應。

> [!NOTE]
> 更新是干擾性作業，會重新啟動您的裝置。 如果 I/O 正在進行中，裝置就會停機。


## <a name="whats-new-in-the-update-04"></a>Update 0.4 的新增功能
Update 0.4 是主要的錯誤修正組建，而且包含一些加強功能。 在此版本中，已修正數個在舊版中導致備份失敗的錯誤。 主要加強功能錯誤修正如下：

- **備份效能加強功能** - 此版本已進行數個重大加強功能，以改進備份效能。 因此，牽涉到大量檔案備份的時間將能大幅縮短，不論是完整備份或增量備份都是這樣。

- **加強的還原效能** - 此版本包含大幅改進大量檔案還原效能的加強功能。 若使用 2 - 4 百萬個檔案，建議您佈建具有 16 GB RAM 的虛擬陣列，看看改良功能的表現。 使用少於 2 百萬個檔案時，虛擬機器的最低需求仍然是 8 GB RAM。

- **對支援套件的改良** - 改良包括將磁碟、CPU、記憶體、網路與雲端的統計資料記錄到支援套件，進而改進診斷/偵錯裝置問題的程序。

- **限制本機固定 iSCSI 磁碟區為 200 GB** - 對於本機固定磁碟區，建議您在 StorSimple Virtual Array 上限制為 200 GB iSCSI 磁碟區。 分層磁碟區的本機保留仍為已佈建磁碟區大小的 10 %，但上限為 200 GB。 

- **備份相關錯誤修正** - 在舊版軟體中，有一些備份相關問題會導致備份失敗。 此版本已修正這些錯誤。


## <a name="issues-fixed-in-the-update-04"></a>Update 0.4 中修正的問題

下表提供本版已修正問題的摘要。

| 編號 | 功能 | 問題 |
| --- | --- | --- |
| 1 |備份效能|在舊版中，牽涉到大量檔案的備份可能需要非常長的時間才能完成 (數以天計)。 在此版本中，完整與增量備份的完成時間將會大幅縮短。 |
| 2 |支援封裝|磁碟、CPU、記憶體、網路與雲端統計資料現在會記錄到支援記錄檔，讓支援套件在支援人員針對任何裝置問題進行疑難排解時發揮效用。|
| 3 |Backup  |在舊版中，長時間執行的備份會導致裝置空間耗盡，進而造成備份失敗。 此版本已解決此錯誤，因為現在會將一次可排入佇列的備份數目限制為 5 個。|
| 4 |iSCSI | 在舊版中，分層或本機固定磁碟區的保留是已佈建磁碟區大小的 10%。 在此版本中，所有 iSCSI 磁碟區 (本機固定或分層) 的本機保留限制為 10 %，且上限為 200 GB (對於大於 2 TB 的分層磁碟區)，因此可讓本機磁碟有更多空間。 建議您在此版本中將本機固定磁碟區限制為 200 GB。|


## <a name="known-issues-in-the-update-04"></a>Update 0.4 的已知問題

下表提供 StorSimple Virtual Array 的已知問題摘要，並包含舊版所列的問題。 

| 編號 | 功能 | 問題 | 因應措施/註解 |
| --- | --- | --- | --- |
| **1.** |更新 |預覽版中所建立的虛擬裝置無法更新為支援的正式運作版本。 |必須針對正式運作版本使用災害復原 (DR) 工作流程容錯移轉這些虛擬裝置。 |
| **2.** |佈建的資料磁碟 |佈建特定指定大小的資料磁碟並建立對應的 StorSimple 虛擬裝置之後，不得展開或壓縮資料磁碟。 嘗試執行會導致裝置本機層中的所有資料遺失。 | |
| **3.** |群組原則 |若裝置已加入網域，套用群組原則可能會對裝置運作產生不良的影響。 |確定您的虛擬陣列位於它自己的 Active Directory 組織單位 (OU) 中，且沒有套用群組原則物件 (GPO)。 |
| **4.** |本機 Web UI |Internet Explorer (IE ESC) 如有啟用增強式安全性功能，部分本機 Web UI 頁面 (例如 [疑難排解] 或 [維護]) 可能無法正常運作。 這些頁面上的按鈕也可能無法運作。 |關閉 Internet Explorer 中的增強式安全性功能。 |
| **5.** |本機 Web UI |在 Hyper-V 虛擬機器中，Web UI 中的網路介面會顯示為 10 Gbps 介面。 |這個行為是 Hyper-V 的反射。 Hyper-V 一律會將虛擬網路介面卡顯示為 10 Gbps。 |
| **6.** |階層式磁碟區或共用 |不支援使用 StorSimple 階層式磁碟區之應用程式的位元組範圍鎖定。 如果啟用位元組範圍鎖定，則 StorSimple 分層無法運作。 |建議的方法包括︰ <br></br>關閉應用程式邏輯中的位元組範圍鎖定。<br></br>選擇將這個應用程式的資料放在固定在本機的磁碟區中，而非階層式磁碟區。<br></br>警告：使用固定在本機的磁碟區，並啟用位元組範圍鎖定時，固定在本機的磁碟區可以上線，即使在還原完成之前也是一樣。 在這種情況下，如果正在還原，則必須等待還原完成。 |
| **7.** |階層式共用 |使用大型檔案可能會導致緩慢分層輸出。 |使用大型檔案時，建議最大檔案不要超過共用大小的 3%。 |
| **8.** |共用的已使用容量 |您可能會在共用上沒有任何資料時看到共用耗用量。 原因是共用的已使用容量包括中繼資料。 | |
| **9.** |災害復原 |您只能對與來源裝置網域相同的網域執行檔案伺服器的災害復原。 這個版本不支援另一個網域中目標裝置的災害復原。 |新版本將會實作這個功能。 |
| **10.** |Azure PowerShell |在這版本中，無法透過 Azure PowerShell 管理 StorSimple 虛擬裝置。 |所有虛擬裝置的管理應該透過 Azure 傳統入口網站和本機 Web UI 來完成。 |
| **11.** |密碼變更 |虛擬陣列裝置主控台僅接受美式鍵盤格式的輸入。 | |
| **12.** |CHAP |CHAP 認證一經建立，即無法移除。 此外，如果您修改 CHAP 認證，就必須先讓磁碟區離線再上線，變更才會生效。 |新版本將會解決此問題。 |
| **13.** |iSCSI 伺服器 |顯示在 iSCSI 磁碟區的 [使用的儲存體] 在 StorSimple Manager 服務與 iSCSI 主機中可能不同。 |ISCSI 主機具有檔案系統檢視。<br></br>裝置會在達到磁碟區大小上限時，看到所配置的區塊。 |
| **14.** |檔案伺服器 |如果資料夾中的檔案中有與其相關聯的替代資料流 (ADS)，就不會透過災害復原、複製和項目層級復原來備份或還原 ADS。 | |
| **15.** |檔案伺服器 |不支援符號連結。 | |
| **16.** |檔案伺服器 |如果將 Windows 加密檔案系統 (EFS) 保護的檔案複製或儲存到 StorSimple 虛擬陣列檔案伺服器，將會導致不支援的組態。  | |

## <a name="next-step"></a>後續步驟
在您的 StorSimple Virtual Array 上[安裝 Update 0.4](storsimple-virtual-array-install-update-04.md)。

## <a name="references"></a>參考
要尋找舊版本資訊嗎？ 請移至： 

* [StorSimple Virtual Array Update 0.3 版本資訊](storsimple-ova-update-03-release-notes.md)
* [StorSimple Virtual Array Update 0.1 和 0.2 版本資訊](storsimple-ova-update-01-release-notes.md)
* [StorSimple Virtual Array 正式運作版版本資訊](storsimple-ova-pp-release-notes.md)

