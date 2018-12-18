---
title: StorSimple 8000 系列 Update 1.2 版本資訊 | Microsoft Docs
description: 說明 StorSimple 8000 系列 Update 1.2 的新功能、問題及因應措施。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 6c9aae87-6f77-44b8-b7fa-ebbdc9d8517c
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7810027e4ab9df6742b4431d12daf9ba2b678bf6
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37061804"
---
# <a name="update-12-release-notes-for-your-storsimple-8000-series-device"></a>StorSimple 8000 系列裝置的 Update 1.2 版本資訊

## <a name="overview"></a>概觀
下列版本資訊說明 StorSimple 8000 系列 Update 1.2 的新功能，並識別未決的重要問題。 當中也包含此版本中隨附之 StorSimple 軟體、驅動程式與磁碟韌體更新的清單。 

Update 1.2 可以套用至任何執行 Release (GA)、Update 0.1、Update 0.2 或 Update 0.3 軟體的 StorSimple 裝置。 如果您的裝置是執行 Update 1 或 Update 1.1，就無法使用 Update 1.2。 如果您的裝置是執行 Release (GA)，請 [連絡 Microsoft 支援服務](storsimple-contact-microsoft-support.md) 協助您安裝此更新。

下表列出 Updates 1、1.1 和 1.2 的對應裝置軟體版本。

| 如果執行更新... | 這是您裝置的軟體版本。 |
| --- | --- |
| Update 1.2 |6.3.9600.17584 |
| Update 1.1 |6.3.9600.17521 |
| Update 1.0 |6.3.9600.17491 |

在 StorSimple 方案中部署更新之前，請檢閱版本資訊中所包含的資訊。 如需詳細資訊，請參閱如何 [在 StorSimple 裝置上安裝 Update 1.2](storsimple-install-update-1.md)。 

> [!IMPORTANT]
> * 安裝此更新大約需要 5-10 小時 (包括 Windows Update)。 
> * Update 1.2 具有軟體、LSI 驅動程式和磁碟韌體更新。 若要安裝，請遵循 [在 StorSimple 裝置上安裝 Update 1.2](storsimple-install-update-1.md)中的指示。
> * 如果是新版本，您可能不會立即看到更新，因為我們會分階段推出更新。 請在數天內再次掃描更新，因為很快就會提供更新。
> 
> 

## <a name="whats-new-in-update-12"></a>Update 1.2 的新功能
這些功能在 Update 1 初次發行時，僅供有限的使用者使用。 而在 Update 1.2 版本中，大多數 StorSimple 使用者都可發現下列新功能與改進功能：

* **從 5000-7000 系列移轉至 8000 系列裝置** – 此版本導入新的移轉功能，可讓 StorSimple 5000-7000 系列應用裝置使用者將其資料移轉至 StorSimple 8000 系列實體應用裝置或虛擬應用裝置。 移轉功能包含兩個重要的價值主張：                                                                  
  
  * **業務持續性**：透過允許將 5000-7000 系列應用裝置上的現有資料移轉到 8000 系列應用裝置。
  * 
  **8000 系列應用裝置的改進功能供應項目**：例如透過 StorSimple Manager 服務對多個應用裝置進行有效率的集中式管理、更高等級的硬體和更新的韌體、虛擬應用裝置、資料行動力，以及未來藍圖中的功能。
    
    如需如何將 StorSimple 5000-7000 [系列移轉至](https://gallery.technet.microsoft.com/Azure-StorSimple-50007000-c1a0460b) 8000 系列裝置的詳細資訊，請參閱《移轉指南》 。 
* **Azure Government 入口網站的可用性** – Azure Government 入口網站現已提供 StorSimple。 請參閱如何 [在 Azure Government 入口網站中部署 StorSimple 裝置](storsimple-deployment-walkthrough-gov.md)。
* **其他雲端服務提供者的支援** – 支援的其他雲端服務提供者包括 Amazon S3、含 RRS 的 Amazon S3、HP 以及 OpenStack (beta)。
* **最新儲存 API 的更新** – StorSimple 已經透過這個版本，更新為最新的 Azure 儲存體服務 API。 執行 Update 1 前軟體版本 (發行、0.1、0.2 和 0.3 版本) 的 StorSimple 8000 系列裝置，是使用 2009 年 7 月 17 日之前的 Azure 儲存體服務 API 版本。 如更新的 [移除儲存體服務版本相關公告](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/10/19/microsoft-azure-storage-service-version-removal-update-extension-to-2016.aspx)中所述，這些 API 將於 2016 年 8 月 1 日被取代。 請務必在 2016 年 8 月 1 日之前，套用 StorSimple 8000 Series Update 1。 如果您無法執行這項操作，StorSimple 裝置將會停止正常運作。
* **區域備援儲存體 (ZRS) 的支援** – 升級至最新版的儲存體 API 之後，StorSimple 8000 系列除了本地備援儲存體 (LRS) 和異地備援儲存體 (GRS) 之外，也將支援區域備援儲存體 (ZRS)。 如需 ZRS 的詳細資訊，請參閱本文件的 [Azure 儲存體備援選項](../storage/common/storage-redundancy.md) 。
* **增強的初始部署和更新經驗** – 在此版本中，已增強安裝和更新程序。 安裝精靈安裝已獲得改善，如果網路組態和防火牆設定不正確，可提供意見給使用者。 已提供其他診斷 Cmdlet 來協助您為裝置的網路連線進行疑難排解。 如需有關用於疑難排解的新診斷 Cmdlet 的詳細資訊，請參閱 [疑難排解部署文件](storsimple-troubleshoot-deployment.md) 。

## <a name="issues-fixed-in-update-12"></a>在 Update 1.2 中修正的問題
下表提供 Update 1.2、1.1 及 1 中已修正問題的摘要。    

| 否。 | 功能 | 問題 | 在 Update 中修正的問題 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- | --- |
| 1 |Windows PowerShell for StorSimple |當使用者從遠端使用 Windows PowerShell for StorSimple 存取 StorSimple 裝置，然後再啟動安裝精靈時，只要輸入 Data 0 IP，就會當機。 這個 Bug 現在已在 Update 1 中修正。 |Update 1 |yes |yes |
| 2 |恢復出廠預設值 |在某些情況下，當您要恢復出廠預設值時，StorSimple 裝置會卡住，並顯示此訊息：「正在重設為原廠預設值 (階段 8)」 。 如果正在進行 Cmdlet 時，按 CTRL+C，就會發生這種情況。 這個 Bug 現在已修正。 |Update 1 |yes |否 |
| 3 |恢復出廠預設值 |雙控制器恢復出廠預設值失敗之後，允許您繼續進行裝置註冊。 這會產生不支援的系統組態。 在 Update 1 中，會顯示錯誤訊息，而且在恢復出廠預設值失敗的裝置上，會阻止進行註冊。 |Update 1 |yes |否 |
| 4 |恢復出廠預設值 |在某些情況下，會引發誤判的不相符警示。 在執行 Update 1 的裝置上，將不會再產生不正確的不相符警示。 |Update 1 |yes |否 |
| 5 |恢復出廠預設值 |如果恢復出廠預設值在完成前遭到中斷，裝置會進入復原模式，且不允許您存取 Windows PowerShell for StorSimple。 這個 Bug 現在已修正。 |Update 1 |yes |否 |
| 6 |災害復原 |災害復原 (DR) 在目標裝置上探索備份期間失敗的災害復原 (DR) Bug 已修正。 |Update 1 |yes |yes |
| 7 |監控 LED |在某些情況下，應用裝置背面的監控 LED 並未指出正確的狀態。 藍色 LED 已熄滅。 即使在未設定這些介面時，DATA 0 和 DATA 1 LED 也在閃爍。 此問題已修正，且監控 LED 現在會指出正確的狀態。 |Update 1 |yes |否 |
| 8 |監控 LED |在某些情況下，套用 Update 1 之後，作用中控制器上的藍色燈號會關閉，導致難以識別作用中控制器。 此修補程式版本已修正此問題。 |Update 1.2 |yes |否 |
| 9 |網路介面 |在舊版中，使用無法路由的閘道設定的 StorSimple 裝置可能會離線。 在此版本中，已將 Data 0 的路由計量設為最低，因此，即使其他網路介面都具備雲端功能，來自裝置的所有雲端流量還是會透過 Data 0 路由。 |Update 1 |yes |yes |
| 10 |備份 |修補程式版本 Update 1.1 已修正了 Update 1 中，導致備份在建立 24 天後損毀的錯誤。 |Update 1.1 |yes |yes |
| 11 |備份 |舊版中的錯誤導致雲端快照集變更率低，且效能不佳。 此修補程式版本已修正此問題。 |Update 1.2 |yes |yes |
| 12 |更新 |此修補程式版本已修正 Update 1 會報告升級失敗，並導致控制器進入修復模式的錯誤。 |Update 1.2 |yes |yes |

## <a name="known-issues-in-update-12"></a>Update 1.2 中的已知問題
下表提供此版本的已知問題摘要。

| 否。 | 功能 | 問題 | 註解/因應措施 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- | --- |
| 1 |磁碟仲裁 |在罕見情況下，如果 8600 裝置的 EBOD 機箱中大部分的磁碟都已中斷連線，而導致沒有磁碟仲裁，那麼將會使存放集區離線。 即使已重新連接磁碟時，它依然會保持離線。 |您必須重新啟動裝置。 如果問題持續發生， 請連絡 Microsoft 支援以進行後續步驟。 |yes |否 |
| 2 |不正確的控制器識別碼 |進行控制器更換時，控制器 0 可能顯示為控制器 1。 在控制器更換期間從對等節點載入影像時，控制器識別碼一開始可能會顯示為對等控制器的識別碼。 在罕見情況下，可能會在系統重新開機後出現這種行為。 |因此，使用者不需要採取任何動作。 控制器更換完成之後，會自行解決這種情況。 |yes |否 |
| 3 |儲存體帳戶 |不支援使用儲存體服務刪除儲存體帳戶的案例。 這會導致無法擷取使用者資料的情況。 |yes |yes | |
| 4 |裝置容錯移轉 |不支援從相同來源裝置將某個磁碟區容器多次容錯移轉至不同的目標裝置。 當裝置容錯移轉是從單一失效裝置容錯移轉到多個裝置時，會讓第一個容錯移轉裝置上的磁碟區容器失去資料擁有權。 進行這類容錯移轉之後，當您在 Azure 傳統入口網站中檢視這些磁碟區容器時，會發現它們的外觀或行為有所不同。 | |yes |否 |
| 5 |安裝 |在 StorSimple Adapter for SharePoint 安裝其間，您必須提供裝置 IP，才能順利完成安裝。 | |yes |否 |
| 6 |Web Proxy |如果您的 Web Proxy 組態設定將 HTTPS 做為指定的通訊協定，您的裝置對服務通訊將會受到影響並使裝置離線。 同時會在程序中產生支援封裝，耗用裝置上的大量資源。 |請確定 Web Proxy URL 指定的通訊協定為 HTTP。 如需詳細資訊，請參閱 [設定裝置的 Web Proxy](storsimple-configure-web-proxy.md)。 |yes |否 |
| 7 |Web Proxy |如果您在註冊的裝置上設定並啟用 Web Proxy，將需要重新啟動裝置上的主動控制器。 | |yes |否 |
| 8 |雲端高延遲與高 I/O 工作負載 |當 StorSimple 裝置同時出現雲端延遲情況嚴重 (大約數秒) 和 I/O 工作負載高的情況時，裝置磁碟區會進入降級的狀態，而且 I/O 可能會失敗，發生「裝置未就緒」錯誤。 |您必須以手動方式將裝置控制器重新開機，或或執行裝置容錯移轉，才能從這種情況下復原。 |yes |否 |
| 9 |Azure PowerShell |當您使用 StorSimple Cmdlet **Get-AzureStorSimpleStorageAccountCredential &#124; Select-Object -First 1 -Wait** 選取第一個物件，讓您可以建立新的 **VolumeContainer** 物件時，此 Cmdlet 會傳回所有物件。 |將此 Cmdlet 以括號括住，如下所示：**(Get-Azure-StorSimpleStorageAccountCredential) &#124; Select-Object -First 1 -Wait** |yes |yes |
| 10 |移轉 |傳遞多個磁碟區容器以進行移轉時，只有第一個磁碟區容器的最新備份的 ETA 正確。 此外，在移轉第一個磁碟區容器中的前 4 個備份之後，將會開始進行平行移轉。 |建議您一次移轉一個磁碟區容器。 |yes |否 |
| 11 |移轉 |還原之後，不會將磁碟區新增至備份原則或虛擬磁碟群組。 |您必須將這些磁碟區新增至備份原則，才能建立備份。 |yes |yes |
| 12 |移轉 |完成移轉之後，5000/7000 系列裝置不得存取移轉的資料容器。 |建議您在移轉完成並認可之後，刪除移轉的資料容器。 |yes |否 |
| 13 |複製和 DR |執行 Update 1 的 StorSimple 裝置無法對執行 Update 1 前軟體的裝置複製或執行災害復原。 |您必須將目標裝置更新為 Update 1，以允許這些作業 |yes |yes |
| 14 |移轉 |當磁碟區群組中沒有相關聯的磁碟區時，5000-7000 系列裝置上用於移轉的設定備份可能會失敗。 |刪除不含相關聯磁碟區的所有空磁碟區群組，然後重試設定備份。 |yes |否 |

## <a name="physical-device-updates-in-update-12"></a>Update 1.2 中的實體裝置更新
如果將修補程式 Update 1.2 套用至執行 Update 1 之前版本的實體裝置，軟體版本會變更為 6.3.9600.17584。

## <a name="controller-and-firmware-updates-in-update-12"></a>Update 1.2 中的控制器和韌體更新
這個版本會更新裝置上的驅動程式和磁碟韌體。

* 如需有關 SAS 控制器更新的詳細資訊，請參閱 [適用於 Microsoft Azure StorSimple 應用裝置的 LSI SAS 控制器的 Update 1](https://support.microsoft.com/kb/3043005)。 
* 如需有關磁碟韌體更新的詳細資訊，請參閱 [適用於 Microsoft Azure StorSimple 應用裝置的磁碟韌體 Update 1](https://support.microsoft.com/kb/3063416)。

## <a name="virtual-device-updates-in-update-12"></a>Update 1.2 中的虛擬裝置更新
這項更新無法套用至虛擬裝置。 將需要建立新的虛擬裝置。 

## <a name="next-steps"></a>後續步驟
* [在您的裝置上安裝 Update 1.2](storsimple-install-update-1.md)。

