---
title: StorSimple Virtual Array 更新版本資訊 | Microsoft Docs
description: 描述執行 Update 0.3 之 StorSimple Virtual Array 的重大未決問題和解決方式。
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: b197651a-3c40-4185-b23d-4c8f22cfa8f4
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/15/2016
ms.author: alkohli
ms.openlocfilehash: fe9d4f6b232e9abcf1fe9fc5657044b6c72fedb8
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109336"
---
# <a name="storsimple-virtual-array-update-03-release-notes"></a>StorSimple Virtual Array Update 0.3 版本資訊
## <a name="overview"></a>概觀
下列版本資訊指出 Microsoft Azure StorSimple Virtual Array 更新的重大未決問題和已解決問題。

版本資訊會持續更新，並在發現需要提出因應措施的重大問題時有所增補。 部署 StorSimple Virtual Array 之前，請仔細檢閱版本資訊中所含的資訊。

Update 0.3 與軟體版本 **10.0.10288.0**對應。

> [!NOTE]
> 更新是干擾性作業，會重新啟動您的裝置。 如果 I/O 正在進行中，裝置就會停機。
> 
> 

## <a name="whats-new-in-the-update-03"></a>Update 0.3 的新功能
Update 0.3 主要是錯誤修正組建。 在此版本中，已修正數個在舊版中導致備份失敗的錯誤。

## <a name="issues-fixed-in-the-update-03"></a>Update 0.3 中修正的問題
下表提供本版已修正問題的摘要。

| 編號 | 功能 | 問題 |
| --- | --- | --- |
| 1 |備份 |在舊版中發生無法完成檔案共用備份的問題。 如果發生此問題，備份作業將會失敗並在 StorSimple Manager 服務引發重大警示來通知使用者。 這個問題不會影響共用上的資料或對該資料的存取。 此版本已找出根本原因並加以修正。 <br></br> 此修正無法溯及既往地套用到已發生此問題的共用。 發生此問題的客戶應該先套用 Update 0.3，然後連絡「Microsoft 支援服務」來執行完整的系統備份，以修正此問題。 客戶也可以不連絡「Microsoft 支援服務」，而是針對受影響的共用，從狀況良好備份還原到新的共用。 |
| 2 |iSCSI |在舊版中發生將資料複製到 StorSimple Virtual Array 上的磁碟區時磁碟區會消失的問題。 此版本已經修正這個問題。 <br></br> 這些修正只有在新建立的磁碟區上才會生效。 這些修正無法溯及既往地套用到已發生此問題的磁碟區。 建議客戶透過 Azure 傳統入口網站讓受影響磁碟區上線、備份這些磁碟區，然後將這些磁碟區還原到新的磁碟區。 |

## <a name="known-issues-in-the-update-03"></a>Update 0.3 中的已知問題
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

## <a name="next-step"></a>後續步驟
[安裝 Update 0.3](storsimple-ova-install-update-01.md) 。

## <a name="references"></a>參考
要尋找舊版本資訊嗎？ 請移至： 

* [StorSimple Virtual Array Update 0.1 和 0.2 版本資訊](storsimple-ova-update-01-release-notes.md)
* [StorSimple Virtual Array 正式運作版版本資訊](storsimple-ova-pp-release-notes.md)

