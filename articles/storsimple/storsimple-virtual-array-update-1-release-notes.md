---
title: StorSimple Virtual Array Update 1.0 版本資訊 | Microsoft Docs
description: 描述執行 Update 1.0 之 StorSimple Virtual Array 的重大未決問題和解決方式。
services: storsimple
documentationcenter: ''
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: alkohli
ms.openlocfilehash: 777718c4893f1edab3613be5dc941e2716d4c972
ms.sourcegitcommit: 295ec94e3332d3e0a8704c1b848913672f7467c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/06/2017
ms.locfileid: "24010861"
---
# <a name="storsimple-virtual-array-update-10-release-notes"></a>StorSimple Virtual Array Update 1.0 版本資訊

## <a name="overview"></a>概觀

下列版本資訊指出 Microsoft Azure StorSimple Virtual Array 更新的重大未決問題和已解決問題。

版本資訊會持續更新，並在發現需要提出因應措施的重大問題時有所增補。 部署 StorSimple Virtual Array 之前，請仔細檢閱版本資訊中所含的資訊。

Update 1.0 與軟體版本 **10.0.10296.0** 對應。

> [!IMPORTANT]
> - 更新是干擾性作業，會重新啟動您的裝置。 如果 I/O 正在進行中，裝置就會停機。 如需有關如何套用更新的詳細指示，請移至[安裝 Update 1.0](storsimple-virtual-array-install-update-1.md)。
>
> - 只有在您的裝置執行的是 Update 0.6 時，才會透過 Azure 入口網站提供 Update 1。

## <a name="whats-new-in-update-10"></a>Update 1.0 的新功能

**Update 1.0 包含 StorSimple 裝置管理員服務驗證的相關變更，應該儘快部署。** 此更新包含下列增強功能與錯誤修正：

 - **使用 Azure Active Directory (AAD) 來向 StorSimple 裝置管理員服務驗證** – 從 Update 1.0 起，系統會使用 Azure Active Directory 來向 StorSimple 裝置管理員服務驗證。 舊的驗證機制會在 2017 年 12 月後淘汰。 所有使用者都必須在其防火牆規則中包含新的驗證 URL。 如需詳細資訊，請移至 [StorSimple Virtual Array 網路需求](storsimple-ova-system-requirements.md)所列出的驗證 URL。
 
    如果未將驗證 URL 包含在防火牆規則內，使用者會看到重大警示，其內容指出他們的 StorSimple 裝置無法向服務驗證。 如果使用者看到此警示，就必須包含新的驗證 URL。 如需詳細資訊，請移至 [StorSimple 網路警示](storsimple-virtual-array-manage-alerts.md)。

 - **效能改善** - 已修正幾個錯誤，以提高雲端讀取、分層加入與分層挑出的速度。 因此，iSCSI 與檔案伺服器裝置的備份與還原效能皆獲改善。

 - **記憶體回收改善** - 此版本已修正錯誤，可在裝置和儲存體帳戶位於兩個遠距區域時，改善記憶體回收週期的效能。

 - **記錄改善** - 此版本改看了記憶體回收和 I/O 路徑相關的記錄功能。


## <a name="issues-fixed-in-update-10"></a>Update 1.0 中修正的問題

下表提供本版已修正問題的摘要。

| 編號 | 功能 | 問題 |
| --- | --- | --- |
| 1 |AAD 式驗證| 此版本經過變更，可讓 AAD 使用 StorSimple 裝置管理員進行驗證。|
| 2 |記憶體回收| 裝置和儲存體帳戶處於不同區域的客戶站台回報了此問題，而且客戶回報有間歇性網路錯誤，因=而影響到計費。 此版本已經修正這個問題。 |
| 3 |效能| 這個版本包含變更，可改善還原/雲端讀取/分層加入/分層挑出的效能。|
| 4 |更新| 舊版中的更新出現問題，導致客戶站台備份失敗。 此版本已經修正這個問題。|

## <a name="known-issues-in-update-10"></a>Update 1.0 中的已知問題

下表提供 StorSimple Virtual Array 的已知問題摘要，並包含舊版所列的問題。

| 編號 | 功能 | 問題 | 因應措施/註解 |
| --- | --- | --- | --- |
| **1.** |更新 |預覽版中所建立的虛擬陣列無法更新為支援的正式運作版本。 |必須針對正式運作版本使用災害復原 (DR) 工作流程容錯移轉這些虛擬陣列。 |
| **2.** |佈建的資料磁碟 |佈建特定指定大小的資料磁碟並建立對應的 StorSimple Virtual Array 之後，不得展開或壓縮資料磁碟。 嘗試執行會導致裝置本機層中的所有資料遺失。 | |
| **3.** |群組原則 |若裝置已加入網域，套用群組原則可能會對裝置運作產生不良的影響。 |確定您的虛擬陣列位於它自己的 Active Directory 組織單位 (OU) 中，且沒有套用群組原則物件 (GPO)。 |
| **4.** |本機 Web UI |Internet Explorer (IE ESC) 如有啟用增強式安全性功能，部分本機 Web UI 頁面 (例如 [疑難排解] 或 [維護]) 可能無法正常運作。 這些頁面上的按鈕也可能無法運作。 |關閉 Internet Explorer 中的增強式安全性功能。 |
| **5.** |本機 Web UI |在 Hyper-V 虛擬機器中，Web UI 中的網路介面會顯示為 10 Gbps 介面。 |這個行為是 Hyper-V 的反射。 Hyper-V 一律會將虛擬網路介面卡顯示為 10 Gbps。 |
| **6.** |階層式磁碟區或共用 |不支援使用 StorSimple 階層式磁碟區之應用程式的位元組範圍鎖定。 如果啟用位元組範圍鎖定，則 StorSimple 分層無法運作。 |建議的方法包括︰ <br></br>關閉應用程式邏輯中的位元組範圍鎖定。<br></br>選擇將這個應用程式的資料放在固定在本機的磁碟區中，而非階層式磁碟區。<br></br>警告：使用固定在本機的磁碟區，並啟用位元組範圍鎖定時，固定在本機的磁碟區可以上線，即使在還原完成之前也是一樣。 在這種情況下，如果正在還原，則必須等待還原完成。 |
| **7.** |階層式共用 |使用大型檔案可能會導致緩慢分層輸出。 |使用大型檔案時，建議最大檔案不要超過共用大小的 3%。 |
| **8.** |共用的已使用容量 |您可能會在共用上沒有任何資料時看到共用耗用量。 這是因為共用的已使用容量包括中繼資料。 | |
| **9.** |災害復原 |您只能對與來源裝置網域相同的網域執行檔案伺服器的災害復原。 這個版本不支援另一個網域中目標裝置的災害復原。 |新版本將會實作這個功能。 如需詳細資訊，請移至 [StorSimple Virtual Array 的容錯移轉和災害復原](storsimple-virtual-array-failover-dr.md) |
| **10.** |Azure PowerShell |在這版本中，無法透過 Azure PowerShell 管理 StorSimple Virtual Array。 |所有虛擬裝置管理都應該透過 Azure 入口網站和本機 Web UI 來完成。 |
| **11.** |密碼變更 |虛擬陣列裝置主控台僅接受美式鍵盤格式的輸入。 | |
| **12.** |CHAP |CHAP 認證一經建立，即無法移除。 此外，如果您修改 CHAP 認證，就必須先讓磁碟區離線再上線，變更才會生效。 |新版本將會解決此問題。 |
| **13.** |iSCSI 伺服器 |針對 iSCSI 磁碟區顯示的 [已使用的儲存體] 在 StorSimple Device Manager 服務與 iSCSI 主機中可能不同。 |ISCSI 主機具有檔案系統檢視。<br></br>裝置會在達到磁碟區大小上限時，看到所配置的區塊。 |
| **14.** |檔案伺服器 |如果資料夾中的檔案中有與其相關聯的替代資料流 (ADS)，就不會透過災害復原、複製和項目層級復原來備份或還原 ADS。 | |
| **15.** |檔案伺服器 |不支援符號連結。 | |
| **16.** |檔案伺服器 |如果將 Windows 加密檔案系統 (EFS) 保護的檔案複製或儲存到 StorSimple 虛擬陣列檔案伺服器，將會導致不支援的組態。  | |
| **17.** |更新 |如果透過本機 UI 嘗試安裝 Hotfix 時看到錯誤代碼：2359302 (十六進位 0x240006)，表示您的裝置上已安裝了 Hotfix。   | |
| **18.** |更新 |如果您使用本機 Web UI 在虛擬陣列上安裝 Update 1，您必須確定正在執行 Update 0.6。 如果您執行的是 Update 0.6 以下版本，您必須先安裝 Update 0.6，然後再套用 Update 1。 如果您直接從 Update 0.6 之前的版本安裝 Update 1.0，則會遺失部分更新，而且監視圖表將無法運作。   | |


## <a name="next-steps"></a>後續步驟
在 StorSimple Virtual Array 上[安裝 Update 1.0](storsimple-virtual-array-install-update-1.md)。

## <a name="references"></a>參考
要尋找舊版本資訊嗎？ 請移至：
*  [StorSimple Virtual Array Update 0.6 版本資訊](storsimple-virtual-array-update-06-release-notes.md)
* [StorSimple Virtual Array Update 0.5 版本資訊](storsimple-virtual-array-update-05-release-notes.md)
* [StorSimple Virtual Array Update 0.4 版本資訊](storsimple-virtual-array-update-04-release-notes.md)
* [StorSimple Virtual Array Update 0.3 版本資訊](storsimple-ova-update-03-release-notes.md)
* [StorSimple Virtual Array Update 0.1 和 0.2 版本資訊](storsimple-ova-update-01-release-notes.md)
* [StorSimple Virtual Array 正式運作版版本資訊](storsimple-ova-pp-release-notes.md)
