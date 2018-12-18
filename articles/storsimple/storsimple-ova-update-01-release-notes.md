---
title: StorSimple Virtual Array 更新版本資訊 | Microsoft Docs
description: 描述執行 Update 0.2 和 0.1 的 StorSimple Virtual Array 的重大未決問題和解決方式。
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 3993864d-2ddd-4302-a2f1-8d737fba6eab
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/16/2016
ms.author: alkohli
ms.openlocfilehash: c4ccde9635b3874864baa9d4d262ff5ddcf2a425
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "23109976"
---
# <a name="storsimple-virtual-array-update-02-and-01-release-notes"></a>StorSimple Virtual Array Update 0.2 和 0.1 版本資訊
## <a name="overview"></a>概觀
下列版本資訊指出 Microsoft Azure StorSimple Virtual Array 更新的重大未決問題和已解決問題。 (Microsoft Azure StorSimple Virtual Array 也稱為 StorSimple 內部部署虛擬裝置或 StorSimple 虛擬裝置。) 

版本資訊會持續更新，並在發現需要提出因應措施的重大問題時有所增補。 部署 StorSimple 虛擬裝置之前，請仔細檢閱版本資訊中所含的資訊。

Update 0.2 對應到軟體版本 **10.0.10280.0**；Update 0.1 是版本 **10.0.10279.0**。 下列各節列出每個更新的變更。 

> [!NOTE]
> 更新是干擾性的，將重新啟動您的裝置。 如果 I/O 正在進行中，裝置就會停機。
> 
> 

## <a name="issues-fixed-in-the-update-02"></a>Update 0.2 中修正的問題
除了下表所述的修正之外，Update 0.2 還包含來自 Update 0.1 的所有變更：

| 功能 | 問題 |
| --- | --- |
| 更新 |在上個版本中，無法在 Azure 傳統入口網站中自動偵測到更新，因此您必須使用本機 Web UI 來安裝更新。 此版本已經修正這個問題。 安裝 Update 0.2 之後，您可以使用 Azure 傳統入口網站安裝未來的更新。 |

## <a name="whats-new-in-the-update-01"></a>Update 0.1 的新功能
Update 0.1 包含下列的錯誤修復和改善。 

* **改善雲端中斷恢復功能**︰此版本提供數個錯誤修正來解決發生雲端連線中斷時的災害復原、備份、還原及分層問題。 
* **改善還原效能**︰本版的錯誤修正大幅減少還原作業的完成時間。
* **自動空間回收最佳化**：在精簡佈建的磁碟區上刪除資料時，需回收未使用的儲存區塊。 此版本已經改善雲端空間回收程序，相較於之前的版本，此版本可讓未使用的空間更快變成可用空間。
* **新的虛擬磁碟映像**︰新的 VHD、VHDX 和 VMDK 現在已可透過 Azure 傳統入口網站取得。 您可以下載這些映像來佈建新的 Update 0.1 裝置。
* **提高入口網站的作業狀態精確度**︰在舊版軟體中，入口網站的作業狀態報告不夠細緻。 本版已經修正這個問題。
* **網域加入體驗**：提供與裝置的網域加入及重新命名相關的錯誤修正。

## <a name="issues-fixed-in-the-update-01"></a>Update 0.1 中修正的問題
下表提供本版已修正問題的摘要。

| 編號 | 功能 | 問題 |
| --- | --- | --- |
| 1 |VMDK |過去在某些 VMware 版本中，OS 磁碟視為疏鬆，從而導致警示並會中斷正常作業。 本版已修正這個問題。 |
| 2 |iSCSI 伺服器 |在上個版本中，使用者需要為 StorSimple 虛擬裝置每個已啟用的網路介面指定閘道器。 本版已變更這種行為，所以使用者必須為所有已啟用的網路介面設定至少一個閘道器。 |
| 3 |支援封裝 |在舊版的軟體中，當封裝大小超過 1 GB 時，封裝集合支援就會失敗。 此版本已經修正這個問題。 |
| 4 |雲端存取 |在上個版本中，如果 StorSimple Virtual Array 沒有網路連線且重新啟動，本機 UI 就會有連線問題。 本版已修正這個問題。 |
| 5 |監視圖表 |在上個版本中，接在裝置容錯移轉之後，雲端容量使用率圖表在 Azure 傳統入口網站中顯示的值不正確。 目前的版本已修正這個問題。 |

## <a name="known-issues-in-the-update-01"></a>Update 0.1 的已知問題
下表提供 StorSimple Virtual Array 的已知問題摘要，並包含舊版所列的問題。 **本版所列的問題會以星號標示。這份清單中所有的問題幾乎都是自 GA 版的 StorSimple Virtual Array 即已存在。**

| 不用。 | 功能 | 問題 | 因應措施/註解 |
| --- | --- | --- | --- |
| **1.** |更新 |預覽版中所建立的虛擬裝置無法更新為支援的正式運作版本。 |必須針對正式運作版本使用災害復原 (DR) 工作流程容錯移轉這些虛擬裝置。 |
| **2.** |佈建的資料磁碟 |佈建特定指定大小的資料磁碟並建立對應的 StorSimple 虛擬裝置之後，不得展開或壓縮資料磁碟。 嘗試執行此作業可能會導致裝置之本機層中的所有資料遺失。 | |
| **3.** |群組原則 |若裝置已加入網域，套用群組原則可能會對裝置運作產生不良的影響。 |確定您的虛擬陣列位於它自己的 Active Directory 組織單位 (OU) 中，且沒有套用群組原則物件 (GPO)。 |
| **4.** |本機 Web UI |Internet Explorer (IE ESC) 如有啟用增強式安全性功能，部分本機 Web UI 頁面 (例如 [疑難排解] 或 [維護]) 可能無法正常運作。 這些頁面上的按鈕也可能無法運作。 |關閉 Internet Explorer 中的增強式安全性功能。 |
| **5.** |本機 Web UI |在 Hyper-V 虛擬機器中，Web UI 中的網路介面會顯示為 10 Gbps 介面。 |這個行為是 Hyper-V 的反射。 Hyper-V 一律會將虛擬網路介面卡顯示為 10 Gbps。 |
| **6.** |階層式磁碟區或共用 |不支援使用 StorSimple 階層式磁碟區之應用程式的位元組範圍鎖定。 如果啟用位元組範圍鎖定，則 StorSimple 分層無法運作。 |建議的方法包括︰ <br></br>關閉應用程式邏輯中的位元組範圍鎖定。<br></br>選擇將這個應用程式的資料放在固定在本機的磁碟區中，而非階層式磁碟區。<br></br>警告：如果使用固定在本機的磁碟區，並啟用位元組範圍鎖定，請注意固定在本機的磁碟區可以上線，即使在還原完成之前也是一樣。 在這種情況下，如果正在還原，則必須等待還原完成。 |
| **7.** |階層式共用 |使用大型檔案可能會導致緩慢分層輸出。 |使用大型檔案時，建議最大檔案不要超過共用大小的 3%。 |
| **8.** |共用的已使用容量 |您可能會在共用上沒有任何資料時看到共用耗用。 原因是共用的已使用容量包括中繼資料。 | |
| **9.** |災害復原 |您只能對與來源裝置網域相同的網域執行檔案伺服器的災害復原。 這個版本不支援另一個網域中目標裝置的災害復原。 |新版本將會實作這個功能。 |
| **10.** |Azure PowerShell |在這版本中，無法透過 Azure PowerShell 管理 StorSimple 虛擬裝置。 |所有虛擬裝置的管理應該透過 Azure 傳統入口網站和本機 Web UI 來完成。 |
| **11.** |密碼變更 |虛擬陣列裝置主控台僅接受美式鍵盤格式的輸入。 | |
| **12.** |CHAP |CHAP 認證一經建立，即無法移除。 此外，若是修改了 CHAP 認證，必須先讓磁碟區離線再上線，變更才會生效。 |這些問題在新版本中都會獲得解決。 |
| **13.** |iSCSI 伺服器 |顯示在 iSCSI 磁碟區的 [使用的儲存體] 在 StorSimple Manager 服務與 iSCSI 主機中可能不同。 |ISCSI 主機具有檔案系統檢視。<br></br>裝置會在達到磁碟區大小上限時，看到所配置的區塊。 |
| **14.** |檔案伺服器* |如果資料夾中的檔案中有與其相關聯的替代資料流 (ADS)，就不會透過災害復原、複製和項目層級復原來備份或還原 ADS。 | |

## <a name="next-step"></a>後續步驟
[安裝更新](storsimple-ova-install-update-01.md) 。

