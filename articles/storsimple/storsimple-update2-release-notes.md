---
title: StorSimple 8000 系列 Update 2 版本資訊 | Microsoft Docs
description: 說明 StorSimple 8000 系列 Update 2 的新功能、問題及因應措施。
services: storsimple
documentationcenter: NA
author: SharS
manager: carmonm
editor: ''
ms.assetid: e2c8bffd-7fc5-4b77-b632-a4f59edacc3a
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: v-sharos
ms.openlocfilehash: 0667c62e2ee2039c2b96b8c4ba81619969c3611b
ms.sourcegitcommit: d28bba5fd49049ec7492e88f2519d7f42184e3a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34057990"
---
# <a name="storsimple-8000-series-update-2-release-notes"></a>StorSimple 8000 系列 Update 2 版本資訊

## <a name="overview"></a>概觀
下列版本資訊說明 StorSimple 8000 系列 Update 2 的新功能，並識別未決的重要問題。 當中也包含此版本中隨附之 StorSimple 軟體、驅動程式與磁碟韌體更新的清單。 

Update 2 可以套用至任何執行 Release (GA)、Update 0.1 到 Update 1.2 的 StorSimple 裝置。 與 Update 2 相關聯的裝置版本為 6.3.9600.17673。

在 StorSimple 方案中部署更新之前，請檢閱版本資訊中所包含的資訊。

> [!IMPORTANT]
> * 安裝此更新大約需要 4-7 小時 (包括 Windows Update)。 
> * Update 2 具有軟體、LSI 驅動程式和 SSD 韌體更新。
> * 如果是新版本，您可能不會立即看到更新，因為我們會分階段推出更新。 請等候數天然後再次掃描更新，因為很快就會提供更新。
> 
> 

## <a name="whats-new-in-update-2"></a>Update 2 的新功能
Update 2 導入下列新功能：

* **固定在本機的磁碟區** – 在舊版的 StorSimple 8000 系列中，資料區塊根據使用量分層至雲端。 無法保證區塊會保持在本機上。 在 Update 2 中，當您建立磁碟區時，您可以將磁碟區指定為固定在本機，該磁碟區的主要資料不會分層至雲端。 固定在本機的磁碟區的快照仍然會複製到雲端進行備份，讓雲端可以用於資料行動力和災害復原。 此外，您可以變更磁碟區類型 (也就是將分層磁碟區轉換為固定在本機的磁碟區，以及將固定在本機的磁碟區轉換為分層)。 
* **StorSimple 虛擬裝置改進** – 以前 StorSimple 8000 系列將虛擬裝置定位為災害復原或開發/測試解決方案。 只有一個虛擬裝置模型 (模型 1100)。 Update 2 導入兩個虛擬裝置模型： 
  
  * 8010 (先前稱為 1100) – 不變更，具有 30 TB 的容量，並且使用 Azure 標準儲存體。
  * 8020 – 具有 64 TB 的容量，並且使用 Azure 進階儲存體以改善效能。
    
    兩個虛擬裝置模型 (8010/8020) 有單一 VHD。 當您第一次啟動虛擬裝置時，它會偵測平台參數，並且套用正確的模型版本。
* **網路改進** – Update 2 包含下列網路改進：
  
  * 可以針對雲端啟用多個 NIC，在某個 NIC 失敗時進行容錯移轉。
  * 路由改進，具有已啟用雲端區塊的固定度量。
  * 在容錯移轉之前的失敗資源線上重試。
  * 服務失敗的新警示。
* **更新改進** – 在更新 1.2 及更早版本中，StorSimple 8000 系列透過兩個管道更新：針對叢集、iSCSI 等等的 Windows Update，以及針對二進位檔和韌體的 Microsoft Update。
    Update 2 會針對所有更新套件使用 Microsoft Update。 這應該會讓修補或進行容錯移轉的時間減少。 
* **韌體更新** – 包含下列韌體更新：
  
  * LSI：lsi_sas2.sys 產品版本 2.00.72.10
  * 只有 SSD (沒有 HDD 更新)：XMGG、XGEG、KZ50、F6C2 和 VR08
* **主動支援** – Update 2 讓 Microsoft 從裝置提取其他診斷資訊。 當我們的作業小組識別有問題的裝置時，我們更有能力從裝置收集資訊並診斷問題。 **接受 Update 2，表示您允許我們提供此主動支援**。    

## <a name="issues-fixed-in-update-2"></a>在 Update 2 中修正的問題
下表提供 Update 2 中修正之問題的摘要。    

| 編號 | 功能 | 問題 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- |
| 1 |網路介面 |升級至 Update 1 之後，StorSimple Manager 服務報告在一個控制器上的 Data2 和 Data3 連接埠失敗。 已修正此問題。 |yes |否 |
| 2 |更新 |升級至 Update 1 之後，在多個裝置上的 Azure 傳統入口網站中發生有聲警報器警示。 已修正此問題。 |yes |否 |
| 3 |Openstack 驗證 |使用 Openstack 做為您的雲端服務提供者時，您可能會收到錯誤，表示您的雲端驗證字串過長。 已修正此問題。 |yes |否 |

## <a name="known-issues-in-update-2"></a>Update 2 中的已知問題
下表提供此版本的已知問題摘要。

| 編號 | 功能 | 問題 | 註解 / 因應措施 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- | --- |
| 1 |磁碟仲裁 |在罕見情況下，如果 8600 裝置的 EBOD 機箱中大部分的磁碟都已中斷連線，而導致沒有磁碟仲裁，那麼將會使存放集區離線。 即使已重新連接磁碟時，它依然會保持離線。 |您必須重新啟動裝置。 如果問題持續發生， 請連絡 Microsoft 支援以進行後續步驟。 |yes |否 |
| 2 |不正確的控制器識別碼 |進行控制器更換時，控制器 0 可能顯示為控制器 1。 在控制器更換期間從對等節點載入影像時，控制器識別碼一開始可能會顯示為對等控制器的識別碼。 在罕見情況下，可能會在系統重新開機後出現這種行為。 |因此，使用者不需要採取任何動作。 控制器更換完成之後，會自行解決這種情況。 |yes |否 |
| 3 |儲存體帳戶 |不支援使用儲存體服務刪除儲存體帳戶的案例。 這會導致無法擷取使用者資料的情況。 | |yes |yes |
| 4 |裝置容錯移轉 |不支援從相同來源裝置將某個磁碟區容器多次容錯移轉至不同的目標裝置。 從單一失效裝置容錯移轉到多個裝置，會讓第一個容錯移轉裝置上的磁碟區容器失去資料擁有權。 進行這類容錯移轉之後，當您在 Azure 傳統入口網站中檢視這些磁碟區容器時，會發現它們的外觀或行為有所不同。 | |yes |否 |
| 5 |安裝 |在 StorSimple Adapter for SharePoint 安裝其間，您必須提供裝置 IP，才能順利完成安裝。 | |yes |否 |
| 6 |Web Proxy |如果您的 Web Proxy 組態設定將 HTTPS 做為指定的通訊協定，您的裝置對服務通訊將會受到影響並使裝置離線。 同時會在程序中產生支援封裝，耗用裝置上的大量資源。 |請確定 Web Proxy URL 指定的通訊協定為 HTTP。 如需詳細資訊，請參閱 [設定裝置的 Web Proxy](storsimple-configure-web-proxy.md)。 |yes |否 |
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
| 20 |固定在本機的磁碟區 |如果您嘗試將分層磁碟區 (使用 Update 1.2 或更早版本所建立及複製) 轉換為固定在本機的磁碟機，而您的裝置空間不足或是雲端空間不足，則複製可能損毀。 |此問題只發生於使用 Update 2 之前的軟體所建立及複製的磁碟機。 這應該是不常見的案例。 | | |
| 21 |磁碟區轉換 |請勿在磁碟區轉換 (從分層轉換為固定在本機，反之亦然) 正在進行時，更新連接到磁碟區的 ACR。 更新 ACR 可能會導致資料損毀。 |如有需要，請在磁碟區轉換之前更新 ACR，並請勿在轉換時進行任何更進一步的 ACR 更新。 | | |

## <a name="controller-and-firmware-updates-in-update-2"></a>Update 2 中的控制器和韌體更新
這個版本會更新裝置上的驅動程式和磁碟韌體。

* 如需 LSI 韌體更新的詳細資訊，請參閱 Microsoft 知識庫文章 3121900。 
* 如需磁碟韌體更新的詳細資訊，請參閱 Microsoft 知識庫文章 3121899。

## <a name="virtual-device-updates-in-update-2"></a>Update 2 中的虛擬裝置更新
這項更新無法套用至虛擬裝置。 將需要建立新的虛擬裝置。 

## <a name="next-step"></a>後續步驟
了解如何在 StorSimple 裝置上 [安裝 Update 2](storsimple-install-update-2.md) 。

