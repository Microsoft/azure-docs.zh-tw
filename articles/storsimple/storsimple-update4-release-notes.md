---
title: StorSimple 8000 系列 Update 4 版本資訊 | Microsoft Docs
description: 說明 StorSimple 8000 系列 Update 4 的新功能、問題及因應措施。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/23/2018
ms.author: alkohli
ms.openlocfilehash: f4db5a49b71b0c384da7af2b90bd5426a4f35c72
ms.sourcegitcommit: d28bba5fd49049ec7492e88f2519d7f42184e3a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34058143"
---
# <a name="storsimple-8000-series-update-4-release-notes"></a>StorSimple 8000 系列 Update 4 版本資訊

## <a name="overview"></a>概觀

下列版本資訊說明 StorSimple 8000 系列 Update 4 的新功能，並識別未決的重要問題。 當中也包含此版本中隨附之 StorSimple 軟體更新的清單。 

Update 4 可以套用至任何執行 Release (GA) 或 Update 0.1 到 Update 3.1 的 StorSimple 裝置。 與 Update 4 關聯的裝置版本為 6.3.9600.17820。

在 StorSimple 方案中部署更新之前，請檢閱版本資訊中所包含的資訊。

> [!IMPORTANT]
> * Update 4 包含裝置軟體、USM 韌體、LSI 驅動程式與韌體、磁碟韌體、Storport 和 Spaceport、 安全性及其他 OS 更新。 安裝此更新，大約需要花費 4 小時。 磁碟韌體更新是干擾性更新，會導致您的裝置停機。 建議您套用 Update 4，讓您的裝置維持在最新狀態。 
> * 如果是新版本，您可能不會立即看到更新，因為我們會分階段推出更新。 請等候數天然後再次掃描更新，因為很快就會提供更新。

## <a name="whats-new-in-update-4"></a>Update 4 的新功能

Update 4 包含以下重要的改良功能和錯誤修正。

* **更聰明的自動化空間回收演算法** – 在 Update 4 中，自動化空間回收演算法已加強，可根據雲端中預期可用的回收空間，以調整空間回收週期。 

* **本機固定磁碟區的效能增強功能** – Update 4 已改善大量資料擷取情況下 (資料與磁碟區大小相當) 本機固定磁碟區的效能。

* **以熱度圖為基礎的還原** - 在舊版中，進行災害復原 (DR) 之後，將根據存取模式從雲端還原資料，導致效能變慢。 

    Update 4 中實作新的功能，可在 DR 之前於裝置正在使用時，追蹤經常存取的資料來建立熱度圖 (最常用的資料區塊有較高熱度，而較少用的區塊有較低熱度)。 在 DR 之後，StorSimple 會使用熱度圖自動還原和解除凍結雲端中的資料。 

    所有還原現在都是以熱度圖為基礎的還原。 如需有關如何查詢和取消以熱度圖為基礎的還原和解除凍結作業的詳細資訊，請移至 [Windows PowerShell for StorSimple Cmdlet 參考文件](https://technet.microsoft.com/library/dn688168.aspx)。

* **StorSimple 診斷工具** – Update 4 中推出 StorSimple 診斷工具，對於系統、網路、效能和硬體元件健康狀態的相關問題，可讓您輕鬆診斷和疑難排解。 此工具是透過 Windows PowerShell for StorSimple 執行。 如需詳細資訊，請移至[使用 StorSimple 診斷工具進行疑難排解](storsimple-8000-diagnostics.md)。

* **以 UI 為基礎的 StorSimple 移轉工具** - 在此版本之前，如果要從 5000-7000 系列移轉資料，使用者必須在移轉工作流程中利用 Azure PowerShell 介面來執行移轉。 在此版本中，提供簡單好用以 UI 為基礎的 StorSimple 移轉工具給支援人員，以利於進行相同的移轉工作流程。 此工具也可用來合併復原貯體。 

* **FIPS 相關變更** - 從這個版本開始，針對 Microsoft Azure Government 和 Azure 公用雲端帳戶，所有 StorSimple 8000 系列裝置上依預設都啟用 FIPS。

* **更新變更** - 此版本已修正與更新失敗相關的錯誤。

* **磁碟失敗的警示** - 此版本中增加新的警示，可向使用者警告磁碟即將失敗。 如果您遇到此警示，請連絡 Microsoft 支援服務以運送替換磁碟。 如需詳細資訊，請移至 [StorSimple 裝置上的硬體警示](storsimple-8000-manage-alerts.md#hardware-alerts)。

* **控制器更換變更** - 此版本中增加 Cmdlet，可讓使用者查詢控制器更換程序的狀態。 如需詳細資訊，請移至[用於查詢控制器更換狀態的 Cmdlet](https://technet.microsoft.com/library/dn688168.aspx)。


## <a name="issues-fixed-in-update-4"></a>Update 4 中修正的問題

下表提供 Update 4 中已修正之問題的摘要。    

| 否 | 功能 | 問題 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- |
| 1 |容錯移轉 |在舊版中，容錯移轉之後，已在客戶方面發現有關清除的問題。 此版本已經修正這個問題。 |yes |yes |
| 2 |固定在本機的磁碟區 |在舊版中，建立本機固定磁碟區方面有問題，導致建立磁碟區失敗。 此版本已找出此問題的根本原因並加以修正。 |yes |否 |
| 3 |支援封裝 |在舊版中，支援套件方面有問題，造成 System.OutOfMemory 例外狀況或其他錯誤，導致建立支套件失敗。 此版本已經修正這些錯誤。 |yes |yes |
| 4 |監視 |在舊版中，本機固定磁碟區的監視圖表有問題，顯示的耗用量是以 EB 為單位。 此版本已經解決這個錯誤。 |yes |yes |
| 5 |移轉 |在舊版中，從 5000-7000 系列移轉至 8000 系列裝置的可靠性有幾個問題。 此版本已經解決這些問題。 |yes |yes |
| 6 |更新 |在舊版中，如果發生更新失敗，控制器會進入復原模式，造成使用者無法繼續更新，並且需要連絡 Microsoft 支援服務。 <br> 這個行為在此版本中已變更。 如果使用者在兩個控制器執行相同版本 (Update 4) 之後發生更新失敗，控制器不會進入復原模式。 如果使用者遇到此失敗，我們建議他們先稍候一陣子，再重試更新。 重試應該能成功。 如果重試失敗，他們便應該連絡 Microsoft 支援服務。 |yes |yes |


## <a name="known-issues-in-update-4-from-previous-releases"></a>Update 4 中舊版的已知問題

Update 4 中沒有新的已知問題。 如需 Update 4 中舊版遺留的問題清單，請移至 [Update 3 版本資訊](storsimple-update3-release-notes.md#known-issues-in-update-3)。

## <a name="serial-attached-scsi-sas-controller-and-firmware-updates-in-update-4"></a>Update 4 中的序列連結 SCSI (SAS) 控制器和韌體更新

此版本包含 SAS 控制器和 LSI 驅動程式與韌體的更新。 如需有關如何安裝這些更新的詳細資訊，請參閱[在 StorSimple 裝置上安裝 Update 4](storsimple-install-update-4.md) 。

## <a name="virtual-device-updates-in-update-4"></a>Update 4 中的虛擬裝置更新

這項更新無法套用至 StorSimple Cloud Appliance (也稱為虛擬裝置)。 將需要建立新的虛擬裝置。 

## <a name="next-step"></a>後續步驟

了解如何[在 StorSimple 裝置上安裝 Update 4](storsimple-install-update-4.md)。

