---
title: 8000 系列裝置的 StorSimple 容錯移轉和災害復原 | Microsoft Docs
description: 了解如何將您的 StorSimple 裝置容錯移轉至相同的裝置。
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
ms.date: 06/23/2017
ms.author: alkohli
ms.openlocfilehash: acc8929dc3476e9590e8e4d9526b38b7c0719570
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108196"
---
# <a name="fail-over-your-storsimple-physical-device-to-same-device"></a>將您的 StorSimple 實體裝置容錯移轉至相同的裝置

## <a name="overview"></a>概觀

本教學課程說明出現災害時，將 StorSimple 8000 系列實體裝置容錯移轉至本身裝置所需要的步驟。 StorSimple 會使用裝置容錯移轉功能，將資料從資料中心的實體裝置來源移轉至另一個實體裝置。 本教學課程中的指導方針適用於執行軟體版本 Update 3 和更新版本的 StorSimple 8000 系列實體裝置。

若要深入了解裝置容錯移轉，以及如何用於從災害中復原，請移至 [ StorSimple 8000 系列裝置的容錯移轉與災害復原](storsimple-8000-device-failover-disaster-recovery.md)。

若要將實體裝置容錯移轉至另一個實體裝置，請移至[容錯移轉至相同的 StorSimple 實體裝置](storsimple-8000-device-failover-physical-device.md)。 若要將 StorSimple 實體裝置容錯移轉至 StorSimple 雲端設備，請移至[容錯移轉至 StorSimple 雲端設備](storsimple-8000-device-failover-cloud-appliance.md)。


## <a name="prerequisites"></a>先決條件

- 請確定您已檢閱裝置容錯移轉的注意事項。 如需詳細資訊，請移至[裝置容錯移轉的一般注意事項](storsimple-8000-device-failover-disaster-recovery.md)。


## <a name="steps-to-fail-over-to-the-same-device"></a>容錯移轉至相同裝置的步驟

如果您需要容錯移轉至相同的裝置，請執行下列步驟。

1. 建立裝置中所有磁碟區的雲端快照。 如需詳細資訊，請移至[使用 StorSimple 裝置管理員服務建立備份](storsimple-8000-manage-backup-policies-u2.md)。
2. 將裝置重設為原廠預設值。 請依照 [如何將 StorSimple 裝置重設為原廠預設值](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings)中的詳細指示執行。
3. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。 在 [裝置] 刀鋒視窗中，舊的裝置應會顯示成 [離線]。

    ![來源裝置離線](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev2.png)

4. 設定裝置，並再次使用 StorSimple 裝置管理員服務重新註冊裝置。 新註冊的裝置應該會顯示成 [準備好進行設定]。 新裝置的裝置名稱與舊裝置的相同，但會加上數字來表示裝置已重設為原廠預設值，並重新註冊。

    ![新註冊的裝置已準備好進行設定](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev3.png)
5. 請完成新裝置的裝置設定。 如需詳細資訊，請移至[步驟 4：完成基本裝置設定](storsimple-8000-deployment-walkthrough-u2.md#step-4-complete-minimum-device-setup)。 在 [裝置] 刀鋒視窗中，裝置的狀態會變更為 [線上]。

   > [!IMPORTANT]
   > **請先完成基本設定，否則您的 DR 可能會失敗。**

    ![新註冊的裝置狀態為線上](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev7.png)

6. 選取舊裝置 (離線狀態)，然後從命令列按一下 [容錯移轉]。 在 [容錯移轉] 刀鋒視窗中，選取舊裝置做為來源，並將目標裝置指定為新註冊的裝置。

    ![容錯移轉摘要](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev11.png)

    如需詳細指示，請參閱 [容錯移轉到另一個實體裝置](#fail-over-to-another-physical-device)。

7. 您可以從 [作業] 刀鋒視窗來監視建立的裝置還原作業。

8. 順利完成作業後，請存取新裝置，並瀏覽到 [磁碟區容器] 刀鋒視窗。 確認舊裝置的所有磁碟區容器已移轉到新的裝置。

   ![移轉磁碟區容器](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev13.png)

9. 完成容錯移轉之後，您可以停用舊裝置，並將其從入口網站中刪除。 選取舊裝置 (離線)，按一下滑鼠右鍵，然後選取 [停用]。 停用裝置之後，裝置的狀態也會一併更新。

     ![停用來源裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev14.png)

10. 選取停用的裝置，按一下滑鼠右鍵，然後選取 [刪除]。 如此會將裝置從裝置清單中刪除。

    ![刪除來源裝置](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev15.png)



## <a name="next-steps"></a>後續步驟

* 執行容錯移轉之後，您可能需要 [停用或刪除 StorSimple 裝置](storsimple-8000-deactivate-and-delete-device.md)。
* 如需如何使用 StorSimple 裝置管理員服務的相關資訊，請移至[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

