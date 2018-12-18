---
title: StorSimple 8000 系列 Update 3 版本資訊 | Microsoft Docs
description: 說明「StorSimple 8000 系列 Update 3」的新功能、問題及因應措施。
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: 2158aa7a-4ac3-42ba-8796-610d1adb984d
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/09/2018
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 9308602b77903e553e26d6f0586d963a4c58230d
ms.sourcegitcommit: d28bba5fd49049ec7492e88f2519d7f42184e3a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34057820"
---
# <a name="update-3-release-notes-for-your-storsimple-8000-series-device"></a>StorSimple 8000 系列裝置的 Update 3 版本資訊

## <a name="overview"></a>概觀
下列版本資訊說明「StorSimple 8000 系列 Update 3」的新功能，並識別未決的重要問題。 當中也包含此版本中隨附之 StorSimple 軟體更新的清單。 

Update 3 可以套用至任何執行 Release (GA) 或 Update 0.1 到 Update 2.2 的 StorSimple 裝置。 與 Update 3 關聯的裝置版本為 6.3.9600.17759。

在 StorSimple 方案中部署更新之前，請檢閱版本資訊中所包含的資訊。

> [!IMPORTANT]
> * Update 3 包含裝置軟體、LSI 驅動程式和韌體以及 Storport 和 Spaceport 的更新。 安裝此更新，大約需要花費 1.5-2 小時。 
> * 如果是新版本，您可能不會立即看到更新，因為我們會分階段推出更新。 請等候數天然後再次掃描更新，因為很快就會提供更新。
> 
> 

## <a name="whats-new-in-update-3"></a>Update 3 的新功能
Update 3 包含以下重要的改良功能和錯誤修正。

* **自動化空間回收變更** – 從 Update 3 開始，空間回收演算法是在待命控制器上執行，使得執行更快速。 如需有關與空間回收搭配運作所需之連接埠的詳細資訊，請參閱 [StorSimple 的網路需求](storsimple-8000-system-requirements.md#networking-requirements-for-your-storsimple-device)。
* **效能增強功能** – Update 3 已改善對雲端的讀寫效能。
* **移轉相關的改良** – 在此版本中，針對從 5000/7000 系列裝置到 8000 系列裝置的移轉功能，進行了數個錯誤修正和改良。 如需有關如何使用移轉功能的詳細資訊，請移至[從 5000/7000 系列裝置的移轉到 8000 系列裝置](https://gallery.technet.microsoft.com/Azure-StorSimple-50007000-c1a0460b)。 
* **監視相關的修正** - 在此版本中，與監視圖表、服務儀表板及裝置儀表板相關的錯誤均已修正。

## <a name="issues-fixed-in-update-3"></a>Update 3 中修正的問題
下表提供 Update 3 中所修正問題的摘要。    

| 否 | 功能 | 問題 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- |
| 1 |主機端資料移轉 |在舊版本中，進行主機端資料移轉時，StorSimple Cloud Appliance 會離線。 此版本已經修正這個問題。 |否 |yes |
| 2 |固定在本機的磁碟區 |在先前版本中，固定在本機的磁碟區有與 I/O 失敗、磁碟區轉換失敗及資料路徑失敗相關的問題。 此版本已找出這些問題的根本原因並加以修正。 |yes |否 |
| 3 |監視 |有多個與報表單位和監視以及裝置儀表板圖表相關的問題，其中針對固定在本機的磁碟區顯示了不正確的資訊。 此版本已經修正這些問題。 |yes |否 |
| 4 |大量寫入 I/O |當使用 StorSimple 來處理涉及大量寫入的工作負載時，使用者會遇到工作集被分層到雲端的罕見錯誤。 此版本已經修正這個錯誤。 |yes |yes |
| 5 |備份 |在某些罕見的情況下，於舊版軟體中，當使用者進行遠端複本的備份時，會遇到雲端錯誤而使得作業發生錯誤。在此版本中，已修正此問題且作業順利完成。 |yes |yes |
| 6 |備份原則 |在某些罕見的情況下，於舊版軟體中，有與刪除備份原則相關的錯誤。 此版本已經修正這個問題。 |yes |yes |

## <a name="known-issues-in-update-3"></a>Update 3 中的已知問題
下表提供此版本的已知問題摘要。

| 編號 | 功能 | 問題 | 註解 / 因應措施 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- | --- |
| 1 |磁碟仲裁 |在罕見情況下，如果 8600 裝置的 EBOD 機箱中大部分的磁碟都已中斷連線，而導致沒有磁碟仲裁，那麼將會使存放集區離線。 即使已重新連接磁碟時，它依然會保持離線。 |您必須重新啟動裝置。 如果問題持續發生， 請連絡 Microsoft 支援以進行後續步驟。 |yes |否 |
| 2 |不正確的控制器識別碼 |進行控制器更換時，控制器 0 可能顯示為控制器 1。 在控制器更換期間從對等節點載入影像時，控制器識別碼一開始可能會顯示為對等控制器的識別碼。 在罕見情況下，可能會在系統重新開機後出現這種行為。 |因此，使用者不需要採取任何動作。 控制器更換完成之後，會自行解決這種情況。 |yes |否 |
| 3 |儲存體帳戶 |不支援使用儲存體服務刪除儲存體帳戶的案例。 這會導致無法擷取使用者資料的情況。 | |yes |yes |
| 4 |裝置容錯移轉 |不支援從相同來源裝置將某個磁碟區容器多次容錯移轉至不同的目標裝置。 從單一失效裝置容錯移轉到多個裝置，會讓第一個容錯移轉裝置上的磁碟區容器失去資料擁有權。 進行這類容錯移轉之後，當您在 Azure 傳統入口網站中檢視這些磁碟區容器時，會發現它們的外觀或行為有所不同。 | |yes |否 |
| 5 |安裝 |在 StorSimple Adapter for SharePoint 安裝其間，您必須提供裝置 IP，才能順利完成安裝。 | |yes |否 |
| 6 |Web Proxy |如果您的 Web Proxy 組態設定將 HTTPS 做為指定的通訊協定，您的裝置對服務通訊將會受到影響並使裝置離線。 同時會在程序中產生支援封裝，耗用裝置上的大量資源。 |請確定 Web Proxy URL 指定的通訊協定為 HTTP。 如需詳細資訊，請參閱 [設定裝置的 Web Proxy](storsimple-8000-configure-web-proxy.md)。 |yes |否 |
| 7 |Web Proxy |如果您在註冊的裝置上設定並啟用 Web Proxy，將需要重新啟動裝置上的主動控制器。 | |yes |否 |
| 8 |雲端高延遲與高 I/O 工作負載 |當 StorSimple 裝置同時出現雲端延遲情況嚴重 (大約數秒) 和 I/O 工作負載高的情況時，裝置磁碟區會進入降級的狀態，而且 I/O 可能會失敗，發生「裝置未就緒」錯誤。 |您必須以手動方式將裝置控制器重新開機，或或執行裝置容錯移轉，才能從這種情況下復原。 |yes |否 |
| 9 |Azure PowerShell |當您使用 StorSimple Cmdlet **Get-AzureStorSimpleStorageAccountCredential &#124; Select-Object -First 1 -Wait** 選取第一個物件，讓您可以建立新的 **VolumeContainer** 物件時，此 Cmdlet 會傳回所有物件。 |將此 Cmdlet 以括號括住，如下所示：**(Get-Azure-StorSimpleStorageAccountCredential) &#124; Select-Object -First 1 -Wait** |yes |yes |
| 10 |移轉 |傳遞多個磁碟區容器以進行移轉時，只有第一個磁碟區容器的最新備份的 ETA 正確。 此外，在移轉第一個磁碟區容器中的前 4 個備份之後，將會開始進行平行移轉。 |建議您一次移轉一個磁碟區容器。 |yes |否 |
| 11 |移轉 |還原之後，不會將磁碟區新增至備份原則或虛擬磁碟群組。 |您必須將這些磁碟區新增至備份原則，才能建立備份。 |yes |yes |
| 12 |移轉 |完成移轉之後，5000/7000 系列裝置不得存取移轉的資料容器。 |建議您在移轉完成並認可之後，刪除移轉的資料容器。 |yes |否 |
| 13 |複製和 DR |執行 Update 1 的 StorSimple 裝置無法對執行 Update 1 前軟體的裝置進行複製或執行災害復原。 |您必須將目標裝置更新為 Update 1，以允許這些作業 |yes |yes |
| 14 |移轉 |當磁碟區群組中沒有相關聯的磁碟區時，5000-7000 系列裝置上用於移轉的設定備份可能會失敗。 |刪除不含相關聯磁碟區的所有空磁碟區群組，然後重試設定備份。 |yes |否 |
| 15 |Azure PowerShell Cmdlet 和固定在本機的磁碟區 |您無法透過 Azure PowerShell Cmdlet 建立固定在本機的磁碟區。 (您透過 Azure PowerShell 建立的任何磁碟區都會分層。) |一律使用 StorSimple Manager 服務來設定固定在本機的磁碟區。 |yes |否 |
| 16 |固定在本機的磁碟區的可用空間 |如果您刪除固定在本機的磁碟區，新的磁碟區的可用空間可能不會立即更新。 StorSimple Manager 服務會每小時更新大致的本機可用空間。 |等候 1 小時，然後再嘗試建立新的磁碟區。 |yes |否 |
| 17 |固定在本機的磁碟區 |您的還原作業會在「備份目錄」中顯示暫時快照集備份，但僅會在還原作業期間顯示。 此外，它也會在 [備份原則] 頁面上顯示前面加上 **tmpCollection** 的虛擬磁碟群組，但僅會在還原作業期間顯示。 |如果還原作業僅有固定在本機的磁碟區或是混合了固定在本機的磁碟區與分層磁碟區，就可能就會發生此問題。 如果還原作業僅包含分層磁碟區，將不會發生此問題。 不需使用者介入。 |yes |否 |
| 18 |固定在本機的磁碟區 |如果您取消還原作業，而之後立即發生控制器容錯移轉，還原作業將會顯示 **Failed** 而不是 **Canceled**。 如果還原作業失敗，而之後立即發生控制器容錯移轉，還原作業將會顯示 **Canceled** 而不是 **Failed**。 |如果還原作業僅有固定在本機的磁碟區或是混合了固定在本機的磁碟區與分層磁碟區，就可能就會發生此問題。 如果還原作業僅包含分層磁碟區，將不會發生此問題。 不需使用者介入。 |yes |否 |
| 19 |固定在本機的磁碟區 |如果您取消還原作業，或還原失敗且發生控制器容錯移轉， **工作** 頁面上會顯示額外的還原作業。 |如果還原作業僅有固定在本機的磁碟區或是混合了固定在本機的磁碟區與分層磁碟區，就可能就會發生此問題。 如果還原作業僅包含分層磁碟區，將不會發生此問題。 不需使用者介入。 |yes |否 |
| 20 |固定在本機的磁碟區 |如果您嘗試將分層磁碟區 (使用 Update 1.2 或更早版本所建立及複製) 轉換為固定在本機的磁碟機，而您的裝置空間不足或是雲端空間不足，則複製可能損毀。 |此問題只發生於使用 Update 2.1 之前的軟體所建立及複製的磁碟機。 這應該是不常見的案例。 | | |
| 21 |磁碟區轉換 |請勿在磁碟區轉換 (從分層轉換為固定在本機，反之亦然) 正在進行時，更新連接到磁碟區的 ACR。 更新 ACR 可能會導致資料損毀。 |如有需要，請在磁碟區轉換之前更新 ACR，並請勿在轉換時進行任何更進一步的 ACR 更新。 | | |
| 22 |更新 |套用更新 3 時，Azure 傳統入口網站中的 [維護] 頁面會顯示與更新 2 相關的下列訊息 -「StorSimple 8000 系列更新 2 包含當我們偵測到潛在問題時，Microsoft 主動從您的裝置收集記錄資訊的能力」。 這會產生誤導，因為它指出裝置會更新為更新 2。 裝置成功更新為更新 3 之後，此訊息會消失。 |在未來版本中將會修正這個行為。 |yes |否 |

## <a name="controller-and-firmware-updates-in-update-3"></a>Update 3 中的控制器和韌體更新
此版本包含 LSI 驅動程式和韌體更新。 如需有關如何安裝 LSI 驅動程式和韌體更新的詳細資訊，請參閱 [在 StorSimple 裝置上安裝 Update 3](storsimple-install-update-3.md) 。

## <a name="virtual-device-updates-in-update-3"></a>Update 3 中的虛擬裝置更新
這項更新無法套用至 StorSimple Cloud Appliance (也稱為虛擬裝置)。 將需要建立新的虛擬裝置。 

## <a name="next-step"></a>後續步驟
了解如何在 StorSimple 裝置上 [安裝 Update 3](storsimple-install-update-3.md) 。

