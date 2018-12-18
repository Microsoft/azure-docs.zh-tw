---
title: 8000 系列裝置的 StorSimple 容錯移轉和災害復原 | Microsoft Docs
description: 了解如何將 StorSimple 裝置容錯移轉至其本身、另一個實體裝置或雲端設備。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/03/2017
ms.author: alkohli
ms.openlocfilehash: 5dc4a98bf889d38c62c76364289c2d58c14d771e
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108476"
---
# <a name="failover-and-disaster-recovery-for-your-storsimple-8000-series-device"></a>StorSimple 8000 系列裝置的容錯移轉和災害復原

## <a name="overview"></a>概觀

本文說明 StorSimple 8000 系列裝置的裝置容錯移轉功能，以及發生災害時，如何使用此功能來復原 StorSimple 裝置。 StorSimple 會使用裝置容錯移轉將資料從資料中心的來源裝置移轉至另一個目標裝置。 本文中的指導方針適用於 StorSimple 8000 系列實體裝置和執行軟體版本 Update 3 和更新版本的雲端設備。

發生災害時，StorSimple 會使用 [裝置] 刀鋒視窗來啟動裝置容錯移轉功能。 此刀鋒視窗會列出所有連接至 StorSimple 裝置管理員服務的 StorSimple 裝置。

![[裝置] 刀鋒視窗](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev1.png)


## <a name="disaster-recovery-dr-and-device-failover"></a>災害復原 (DR) 與裝置容錯移轉

在災害復原 (DR) 案例中，主要裝置會停止運作。 StorSimple 會使用主要裝置做為_來源_，並將相關聯的雲端資料移至另一個_目標_裝置。 這個程序就稱為「容錯移轉」 。 下圖說明容錯移轉的流程。

![裝置容錯移轉時會發生什麼事？](./media/storsimple-8000-device-failover-disaster-recovery/failover-dr-flow.png)

容錯移轉的目標裝置可能是實體裝置，甚至是雲端設備。 目標裝置所處的地理位置可能與來源裝置相同，也可能不同。

容錯移轉期間，可選擇要用來進行移轉的磁碟區容器。 接著，StorSimple 會將這些磁碟區容器的擁有權從來源裝置變更為目標裝置。 在磁碟區容器變更擁有權之後，StorSimple 會從來源裝置中刪除這些容器。 完成刪除之後，您便可以容錯回復目標裝置。 _容錯回復_會將擁有權轉移回原始的來源裝置。

### <a name="cloud-snapshot-used-during-device-failover"></a>裝置容錯移轉時使用的雲端快照

在 DR 之後，最新的雲端備份會用來將資料還原至目標裝置。 如需有關雲端快照的詳細資訊，請參閱[使用 StorSimple 裝置管理員服務來進行手動備份](storsimple-8000-manage-backup-policies-u2.md#take-a-manual-backup)。

在 StorSimple 8000 系列上，備份原則與備份有關係。 如果相同磁碟區有多個備份原則，StorSimple 會選擇磁碟區數目最多的備份原則。 接著，StorSimple 會使用所選的備份原則中最新的備份，以還原目標裝置上的資料。

假設有兩個備份原則，defaultPol 與 customPol：

* defaultPol：一個磁碟區 (vol1)，會於每日晚上 10:30 開始執行。
* customPol：四個磁碟區 (vol1、vol2、vol3、vol4)，會於每日晚上 10:00 開始執行。

在此情況下，StorSimple 會針對損毀一致性來排定優先順序，並使用 customPol，因其擁有的磁碟區較多。 此原則的最新備份可用來還原資料。 如需如何建立及管理備份原則的詳細資訊，請移至[使用 StorSimple 裝置管理員服務來管理備份原則](storsimple-8000-manage-backup-policies-u2.md)。

## <a name="common-considerations-for-device-failover"></a>裝置容錯移轉的一般注意事項

容錯移轉裝置之前，請檢閱下列資訊：

* 裝置容錯移轉開始前，所有磁碟區容器中的磁碟區必須離線。 在非計劃性容錯移轉時，StotSimple 磁碟區會自動離線。 但是，如果您正在執行計劃性容錯移轉 (用於測試 DR)，則必須將所有磁碟區離線。
* 僅會列出具有相關聯的雲端快照的磁碟區容器，以供 DR 使用。 必須至少有一個擁有相關聯的雲端快照的磁碟區容器，才可復原資料。
* 如果有跨多個磁碟區容器的複數雲端快照，StorSimple 會將這些磁碟區容器容錯移轉為一個集合。 在極少數情況下，如果有跨多個磁碟區容器的本機快照，但沒有相關聯的雲端快照，StorSimple 會容錯移轉本機快照，而本機資料會在 DR 之後消失。
* 適用於 DR 的可用目標裝置是空間足以容納選定磁碟區容器的裝置。 沒有足夠空間的裝置，皆不會列為目標裝置。
* DR (一段有限時間) 之後，資料存取效能會受到大幅影響，因為裝置必須從雲端存取資料，然後儲存到本機。

#### <a name="device-failover-across-software-versions"></a>跨軟體版本的裝置容錯移轉

部署中的 StorSimple 裝置管理員服務可能會有多個實體和雲端裝置，且全都執行不一樣的軟體版本。

使用下表可判斷您是否可以容錯移轉或容錯回復至另一個執行不同軟體版本的裝置，以及各種磁碟區類型在 DR 期間會如何行動。

#### <a name="failover-and-failback-across-software-versions"></a>跨軟體版本的容錯移轉與容錯回復

| 容錯移轉/容錯回復來源 | 實體裝置 | 雲端設備 |
| --- | --- | --- |
| Update 3 至 Update 4 |階層式磁碟區以分層的方式進行容錯移轉。 <br></br>本機固定磁碟區以固定在本機的方式進行容錯移轉。 <br></br> 在 Update 4 裝置上拍下快照進行容錯移轉後，會啟動[熱度圖式追蹤](storsimple-update4-release-notes.md#whats-new-in-update-4)。 |本機固定磁碟區以分層的方式進行容錯移轉。 |
| Update 4 至 Update 3 |階層式磁碟區以分層的方式進行容錯移轉。 <br></br>本機固定磁碟區以固定在本機的方式進行容錯移轉。 <br></br> 用來還原的備份會保留熱度圖中繼資料。 <br></br>容錯回復後，無法在 Update 3 使用熱度圖式追蹤。 |本機固定磁碟區以分層的方式進行容錯移轉。 |


## <a name="device-failover-scenarios"></a>裝置容錯移轉情節

萬一發生災害，您可以選擇將 StorSimple 裝置容錯移轉到：

* [實體裝置](storsimple-8000-device-failover-physical-device.md)。
* [本身](storsimple-8000-device-failover-same-device.md)。
* [雲端設備](storsimple-8000-device-failover-cloud-appliance.md)。

上述文章詳述了以上每種容錯移轉情況的步驟。


## <a name="failback"></a>容錯回復

針對 Update 3 和更新版本，StorSimple 也支援容錯回復。 容錯回復為容錯移轉的反向運作，目標變成來源，而容錯轉移期間的原始來源裝置如今變成目標裝置。 

在容錯回復期間，StorSimple 會將資料重新同步回主要位置、停止 I/O 與應用程式活動，並轉換回原始位置。

容錯移轉完成之後，StorSimple 會執行下列動作：

* StorSimple 會將容錯移轉完畢的磁碟區容器從來源裝置中清除。
* 在來源裝置上，StorSimple 會針對每一磁碟區容器起始一項背景作業 (容錯移轉)。 如果您嘗試在作業進行中時進行容錯回復，您會收到該項影響的通知。 必須等到作業完成後才能開始容錯回復。
* 完成磁碟區容器刪除所需的時間取決於各種因素，例如資料量、資料存留期、備份數目，以及作業可用的網路頻寬。

如果您正在規劃為容錯移轉或容錯回復進行測試，建議您以含較少資料 (單位為 Gb) 的磁碟區容器進行測試。 通常您可以在容錯移轉完成之後 24 小時開始進行容錯回復。

## <a name="frequently-asked-questions"></a>常見問題集

問： **如果 DR 失敗或只有部分成功，會發生什麼狀況？**

A. 如果 DR 失敗，建議您再試一次。 第二次裝置容錯移轉作業可感知第一次作業的進度，並從該點開始進行。

問： **正在進行裝置容錯移轉時，是否可以刪除裝置？**

A. 您無法在 DR 正在進行時刪除裝置。 只有在 DR 完成之後，您才能刪除裝置。 您可以在 [作業] 刀鋒視窗中監視裝置容錯移轉作業進度。

問： **何時開始收集來源裝置上的廢棄項目，以便刪除來源裝置上的本機資料？**

A. 只有在完全清理裝置之後，才會在來源裝置上啟用記憶體回收。 清理作業包括清理已從來源裝置容錯移轉的物件，例如磁碟區、備份物件 (非資料)、磁碟區容器，以及原則。

問： **如果與來源裝置中磁碟區容器關聯的刪除作業失敗，會發生什麼情況？**

A.  如果刪除作業失敗，您可以手動刪除磁碟區容器。 在 [裝置] 刀鋒視窗中，選取您的來源裝置，然後按一下 [磁碟區容器]。 選取您已進行容錯移轉的磁碟區容器，然後在刀鋒視窗底部按一下 [刪除]。 刪除來源裝置上所有已容錯移轉的磁碟區容器之後，您便可以開始容錯回復。 如需詳細資訊，請移至[刪除磁碟區容器](storsimple-8000-manage-volume-containers.md#delete-a-volume-container)。

## <a name="business-continuity-disaster-recovery-bcdr"></a>業務持續性災害復原 (BCDR)

當整個 Azure 資料中心停止運作時，就構成業務持續性災害復原 (BCDR) 狀況。 這情況會影響您的 StorSimple 裝置管理員服務和相關聯的 StorSimple 裝置。

如果 StorSimple 裝置在發生災害的前一刻才剛註冊，則此裝置可能需要進行原廠重設。 災害發生後，StorSimple 裝置在 Azure 入口網站中會顯示為離線。 必須將此裝置從入口網站中刪除。 請將裝置重設為原廠預設值，並再次註冊服務。

## <a name="next-steps"></a>後續步驟

如果已準備好執行裝置容錯移轉，請在下列情節中擇一，取得詳細說明：

- [容錯移轉至另一個實體裝置](storsimple-8000-device-failover-physical-device.md)
- [容錯移轉至相同裝置](storsimple-8000-device-failover-same-device.md)
- [容錯移轉至 StorSimple 雲端設備](storsimple-8000-device-failover-cloud-appliance.md)

如果已將裝置容錯移轉，請在下列選項中擇一：

* [停用或刪除 StorSimple 裝置](storsimple-8000-deactivate-and-delete-device.md)。
* [使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

