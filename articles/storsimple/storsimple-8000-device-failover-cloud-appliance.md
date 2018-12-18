---
title: StorSimple 容錯移轉、災害復原至 StorSimple 雲端設備| Microsoft Docs
description: 了解如何將您的 StorSimple 8000 系列實體裝置容錯移轉至雲端設備。
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
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: ec8bebf2854e84a37e84b45564e80fc20b63d8d8
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108326"
---
# <a name="fail-over-to-your-storsimple-cloud-appliance"></a>容錯移轉至您的 StorSimple 雲端設備

## <a name="overview"></a>概觀

本教學課程說明當發生災害時，將 StorSimple 8000 系列實體裝置容錯移轉至 StorSimple 雲端設備所需要的步驟。 StorSimple 會使用裝置容錯移轉功能，將資料從資料中心的實體裝置來源移轉至 Azure 中執行的雲端設備。 本教學課程中的指導方針適用於 StorSimple 8000 系列實體裝置和執行軟體版本為 Update 3 和更新版本的雲端設備。

若要深入了解裝置容錯移轉，以及如何用於從災害中復原，請移至 [ StorSimple 8000 系列裝置的容錯移轉與災害復原](storsimple-8000-device-failover-disaster-recovery.md)。

若要將 StorSimple 實體裝置容錯移轉至另一個實體裝置，請移至[容錯移轉至 StorSimple 實體裝置](storsimple-8000-device-failover-physical-device.md)。 若要將裝置容錯移轉至本身的裝置，請移至[容錯移轉至相同的 StorSimple 實體裝置](storsimple-8000-device-failover-same-device.md)。

## <a name="prerequisites"></a>先決條件

- 請確定您已檢閱裝置容錯移轉的注意事項。 如需詳細資訊，請移至[裝置容錯移轉的一般注意事項](storsimple-8000-device-failover-disaster-recovery.md)。

- 執行此程序之前，您必須已建立並設定 StorSimple 雲端設備。 如果執行的是 Update 3 的軟體版本或更新版本，請考慮針對 DR 使用 8020 雲端設備。 8020 型號具有 64 TB 容量，且使用的是進階儲存體。 如需詳細資訊，請移至[部署和管理 StorSimple 雲端設備](storsimple-8000-cloud-appliance-u2.md)。

## <a name="steps-to-fail-over-to-a-cloud-appliance"></a>容錯移轉至雲端設備的步驟

請執行下列步驟以將裝置還原至目標 StorSimple 雲端設備。

1.  確認您要容錯移轉的磁碟區容器是否具有相關聯的雲端快照。 如需詳細資訊，請移至[使用 StorSimple 裝置管理員服務建立備份](storsimple-8000-manage-backup-policies-u2.md)。
2. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。 在 [裝置] 刀鋒視窗中，移至與您的服務連接的裝置清單。
    ![選取裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev1.png)
3. 選取並按一下您的來源裝置。 來源裝置具有您想要容錯移轉的磁碟區容器。 移至 [設定] > [磁碟區容器]。

    ![選取裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev2.png)
    
4. 選取您要容錯移轉至另一個裝置的磁碟區容器。 按一下磁碟區容器，以顯示此容器內的磁碟區清單。 選取磁碟區，按一下滑鼠右鍵，然後按一下 [離線]，讓磁碟區離線。

    ![選取裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev5.png)

5. 針對磁碟區容器中的所有磁碟區，重複執行這個程序。

     ![選取裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev7.png)

6. 針對您要容錯移轉至另一個裝置的所有磁碟區容器，重複執行前一個步驟。

7. 返回 [裝置] 刀鋒視窗。 從命令列中，按一下 [容錯移轉]。

    ![按一下 [容錯移轉]](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev8.png)
8. 在 [容錯移轉] 刀鋒視窗中，執行下列步驟︰
   
    1. 按一下 [來源]。 選取要容錯移轉的磁碟區容器。 **只會顯示與雲端快照集和離線磁碟區相關聯的磁碟區容器。**
        ![選取來源](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev11.png)
    2. 按一下 [目標]。 從可用裝置的下拉式清單中，選取目標雲端設備。 **清單中只會顯示擁有足夠容量容納來源磁碟區容器的裝置。**

        ![選取目標](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev12.png)

    3. 檢閱 [摘要] 的容錯移轉設定，並選取核取方塊，代表選取的磁碟區容器內的磁碟區已離線。 

        ![檢閱容錯移轉設定](./media/storsimple-8000-device-failover-disaster-recovery/failover-cloud-dev13.png)

9. 已建立容錯移轉作業。 若要監視容錯移轉作業，請按一下 [作業通知]。

    ![監視容錯移轉作業](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev13.png)

10. 完成容錯移轉後，返回 [裝置] 刀鋒視窗。

    1. 為容錯移轉選取要用來做為目標的裝置。

       ![選取裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev14.png)

    2. 按一下 [磁碟區容器]。 此時應會列出所有磁碟區容器以及舊裝置中的磁碟區。

       如果您容錯移轉的磁碟區容器含有固定在本機的磁碟區，這些磁碟區會容錯移轉成為分層磁碟區。 StorSimple 雲端設備不支援已固定在本機的磁碟區。

       ![檢視目標磁碟區容器](./media/storsimple-8000-device-failover-disaster-recovery/failover-phy-dev17.png)


## <a name="next-steps"></a>後續步驟

* 執行容錯移轉之後，您可能需要 [停用或刪除 StorSimple 裝置](storsimple-8000-deactivate-and-delete-device.md)。

* 如需如何使用 StorSimple 裝置管理員服務的相關資訊，請移至[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

