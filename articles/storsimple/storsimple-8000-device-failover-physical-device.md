---
title: StorSimple 容錯移轉、災害復原至 StorSimple 8000 系列實體裝置| Microsoft Docs
description: 了解如何將您的 StorSimple 8000 系列實體裝置容錯移轉至另一個實體裝置。
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
ms.openlocfilehash: f3ac9545a341fc24ca12c9f2547805d6956cd98a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108266"
---
# <a name="fail-over-to-a-storsimple-8000-series-physical-device"></a>容錯移轉至 StorSimple 8000 系列實體裝置

## <a name="overview"></a>概觀

本教學課程說明出現災害時，將 StorSimple 8000 系列實體裝置容錯移轉至另一個 StorSimple 實體裝置所需要的步驟。 StorSimple 會使用裝置容錯移轉功能，將資料從資料中心的實體裝置來源移轉至另一個實體裝置。 本教學課程中的指導方針適用於執行軟體版本 Update 3 和更新版本的 StorSimple 8000 系列實體裝置。

若要深入了解裝置容錯移轉，以及如何用於從災害中復原，請移至 [ StorSimple 8000 系列裝置的容錯移轉與災害復原](storsimple-8000-device-failover-disaster-recovery.md)。

若要將 StorSimple 實體裝置容錯移轉至 StorSimple 雲端設備，請移至[容錯移轉至 StorSimple 雲端設備](storsimple-8000-device-failover-cloud-appliance.md)。 若要將實體裝置容錯移轉至本身的裝置，請移至[容錯移轉至相同的 StorSimple 實體裝置](storsimple-8000-device-failover-same-device.md)。


## <a name="prerequisites"></a>先決條件

- 請確定您已檢閱裝置容錯移轉的注意事項。 如需詳細資訊，請移至[裝置容錯移轉的一般注意事項](storsimple-8000-device-failover-disaster-recovery.md)。

- 必須已在資料中心內部署 StorSimple 8000 系列實體裝置。 裝置必須執行 Update 3 或更新的軟體版本。 如需詳細資訊，請移至 [部署內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)。


## <a name="steps-to-fail-over-to-a-physical-device"></a>容錯移轉到實體裝置的步驟

請執行下列步驟以將裝置還原至目標實體裝置。

1. 確認您要容錯移轉的磁碟區容器是否具有相關聯的雲端快照。 如需詳細資訊，請移至[使用 StorSimple 裝置管理員服務建立備份](storsimple-8000-manage-backup-policies-u2.md)。
2. 移至您的 StorSimple 裝置管理員，然後按一下 [裝置]。 在 [裝置] 刀鋒視窗中，移至與您的服務連接的裝置清單。
    ![選取裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev1.png)
3. 選取並按一下您的來源裝置。 來源裝置具有您想要容錯移轉的磁碟區容器。 移至 [設定] > [磁碟區容器]。
4. 選取您要容錯移轉至另一個裝置的磁碟區容器。 按一下磁碟區容器，以顯示此容器內的磁碟區清單。 選取磁碟區，按一下滑鼠右鍵，然後按一下 [離線]，讓磁碟區離線。 針對磁碟區容器中的所有磁碟區，重複執行這個程序。
5. 針對您要容錯移轉至另一個裝置的所有磁碟區容器，重複執行前一個步驟。
6. 返回 [裝置] 刀鋒視窗。 從命令列中，按一下 [容錯移轉]。
    按一下 [容錯移轉]![](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev2.png)
    
7. 在 [容錯移轉] 刀鋒視窗中，執行下列步驟︰
   
   1. 按一下 [來源]。 隨即顯示含有與雲端快照集相關聯之磁碟區的磁碟區容器。 只有顯示的容器才可進行容錯移轉。 在磁碟區容器清單中，選取您要容錯移轉的磁碟區容器。 **只會顯示與雲端快照集和離線磁碟區相關聯的磁碟區容器。**

       ![選取來源](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev5.png)
   2. 按一下 [目標]。 從可用裝置的下拉式清單中選取目標裝置，供上一步選取的磁碟區容器使用。 清單中只會顯示擁有足夠容量容納來源磁碟區容器的裝置。

        ![選取目標](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev6.png)

   3. 最後，檢閱 [摘要] 下的所有容錯移轉設定。 檢閱設定之後，選取核取方塊，代表所選取磁碟區容器中的磁碟區已離線。 按一下 [SERVICEPRINCIPAL] 。

       ![檢閱容錯移轉設定](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev8.png)
  
8. StorSimple 會建立容錯移轉作業。 按一下 [作業通知]，以透過 [作業] 刀鋒視窗監視容錯移轉作業。

    如果要容錯移轉的磁碟區容器包含本機磁碟區，則您會看到容器中每個本機磁碟區 (除分層磁碟區外) 的還原作業。 這些還原作業可能需要一段時間才能完成。 容錯移轉作業可能會較早完成。 還原作業完成之後這些磁碟區才有本機保證。

    ![監視容錯移轉作業](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev13.png)

9. 完成容錯移轉後，移至 [裝置] 刀鋒視窗。
   
   1. 為容錯移轉程序選取要用來做為目標裝置的裝置。

       ![選取裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev14.png)

   2. 移至 [磁碟區容器] 刀鋒視窗。 此時應會列出所有磁碟區容器以及舊裝置中的磁碟區。

       ![檢視目標磁碟區容器](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev16.png)


## <a name="next-steps"></a>後續步驟

* 執行容錯移轉之後，您可能需要 [停用或刪除 StorSimple 裝置](storsimple-8000-deactivate-and-delete-device.md)。
* 如需如何使用 StorSimple 裝置管理員服務的相關資訊，請移至[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

