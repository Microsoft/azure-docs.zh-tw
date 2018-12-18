---
title: 停用及刪除 StorSimple 8000 系列裝置 | Microsoft Docs
description: 描述如何停用然後刪除 StorSimple 裝置，將其從服務中移除。
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
ms.date: 07/23/2018
ms.author: alkohli
ms.openlocfilehash: a2b764e76cd1987c83e7be38d365c1dfa8513db9
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39214783"
---
# <a name="deactivate-and-delete-a-storsimple-device"></a>停用及刪除 StorSimple 裝置

## <a name="overview"></a>概觀

本文說明如何停用及刪除已連線至 StorSimple 裝置管理員服務的 StorSimple 裝置。 本文中的指導僅適用於包含 StorSimple 雲端設備的 StorSimple 8000 系列裝置。 如果您使用 StorSimple Virtual Array，請移至[停用及刪除 StorSimple Virtual Array](storsimple-virtual-array-deactivate-and-delete-device.md)。

停用會切斷裝置與相對應 StorSimple 裝置管理員服務之間的連接。 您可能會想讓某個 StorSimple 裝置停止提供服務 (舉例來說，當您在替換或升級裝置時，或是當您不再使用 StorSimple 時)。 如果是此情況，您必須在刪除該裝置之前，先停用裝置。

當您停用裝置時，無法再存取以本機方式儲存在裝置上的任何資料。 只有儲存於雲端之裝置的相關聯資料可以復原。

> [!WARNING]
> 停用是永久性的作業，而且無法復原。 停用的裝置除非重設為原廠預設，否則無法向 StorSimple 裝置管理員服務登錄。
>
> 原廠重設程序會刪除以本機方式儲存在裝置上的所有資料。 因此，在您停用裝置之前，您必須擷取所有資料的雲端快照。 此雲端快照可讓您在稍後的階段中復原所有資料。

閱讀本教學課程之後，您將能夠：

* 停用裝置並刪除資料。
* 停用裝置並保留資料。

> [!NOTE]
> 在您停用 StorSimple 實體裝置或雲端設備之前，請停止或刪除相依於該裝置的用戶端和主機。


## <a name="deactivate-and-delete-data"></a>停用及刪除資料

如果您想要完全刪除裝置，且不要保留裝置中的資料，請完成下列步驟。

#### <a name="to-deactivate-the-device-and-delete-the-data"></a>如何停用裝置並刪除資料

1. 在停用裝置之前，您必須刪除和裝置相關聯的所有磁碟區容器 (和磁碟區)。 但您在刪除磁碟區容器之前，必須先刪除相關聯的備份。

    > [!NOTE]
    > 停用 StorSimple 實體裝置或雲端應用裝置之前，請確定已刪除的磁碟區容器中的資料已確實從裝置上刪除。 您可以監視雲端耗用量圖表，當您看到雲端使用量因為您刪除備份而下降時，就可以繼續進行停用裝置。 如果您在使用量下降之前就停用裝置，資料會受困在儲存體帳戶中，導致費用產生。

2. 請依照下列步驟來停用裝置：
   
   1. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。 在 [裝置] 刀鋒視窗中，選取您想要停用的裝置、按一下滑鼠右鍵，然後按一下 [停用]。

        ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate1.png)
   2. 在 [停用] 刀鋒視窗中，輸入裝置名稱以進行確認，然後按一下 [停用]。 停用程序開始，需要幾分鐘才能完成。

        ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate2.png)

3. 停用之後，您就可以完全刪除裝置。 刪除裝置會將其從與服務連接的裝置清單移除。 服務將不再管理已刪除的裝置。 請使用下列步驟來刪除裝置：
   
   1. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。 在 [裝置] 刀鋒視窗中，選取您想要刪除的已停用裝置、按一下滑鼠右鍵，然後按一下 [刪除]。

        ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate5.png)
   2. 在 [刪除] 刀鋒視窗中，輸入裝置名稱以進行確認，然後按一下 [刪除]。 刪除需要幾分鐘才能完成。

        ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate6.png)
   3. 順利完成刪除之後，您會收到通知。 裝置清單也會更新以反映刪除。

## <a name="deactivate-and-retain-data"></a>停用並保留資料

如果您想要刪除裝置，但要保留資料，請完成下列步驟：

#### <a name="to-deactivate-a-device-and-retain-the-data"></a>如何停用裝置並保留資料
1. 停用裝置。 裝置的所有磁碟區容器及快照都會保留。
   
   1. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。 在 [裝置] 刀鋒視窗中，選取您想要停用的裝置、按一下滑鼠右鍵，然後按一下 [停用]。

         ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate1.png)
   2. 在 [停用] 刀鋒視窗中，輸入裝置名稱以進行確認，然後按一下 [停用]。 停用程序開始，需要幾分鐘才能完成。

         ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate2.png)
2. 您現在可以容錯移轉磁碟區容器和相關聯的快照集。 關於程序，請移至 [StorSimple 裝置的容錯移轉和災害復原](storsimple-8000-device-failover-disaster-recovery.md)。
3. 停用和容錯移轉之後，您就可以完全刪除裝置。 刪除裝置會將其從與服務連接的裝置清單移除。 服務將不再管理已刪除的裝置。 若要刪除裝置，請完成下列步驟：
   
   1. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。 在 [裝置] 刀鋒視窗中，選取您想要刪除的已停用裝置、按一下滑鼠右鍵，然後按一下 [刪除]。

       ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate5.png)
   2. 在 [刪除] 刀鋒視窗中，輸入裝置名稱以進行確認，然後按一下 [刪除]。 刪除需要幾分鐘才能完成。

       ![停用 StorSimple 裝置](./media/storsimple-8000-deactivate-and-delete-device/deactivate6.png)
   3. 順利完成刪除之後，您會收到通知。 裝置清單也會更新以反映刪除。

     
## <a name="deactivate-and-delete-a-cloud-appliance"></a>停用及刪除雲端設備

針對 StorSimple 雲端設備，從入口網站停用會解除配置並刪除虛擬機器，以及佈建時所建立的資源。 停用雲端設備之後，就無法將它還原為先前的狀態。

![停用 StorSimple 雲端設備](./media/storsimple-8000-deactivate-and-delete-device/deactivate7.png)

停用會導致下列動作發生：

* StorSimple 雲端設備會從服務中移除。
* 刪除 StorSimple 雲端設備的虛擬機器。
* 保留針對 StorSimple 雲端設備建立的作業系統磁碟和資料磁碟。 如果您不使用這些項目，就應該手動加以刪除。
* 保留在佈建期間建立的託管服務和虛擬網路。 如果您不使用這些項目，就應該手動加以刪除。
* 保留 StorSimple 雲端設備建立的雲端快照。

雲端設備停用之後，您可以將其從裝置清單中刪除。 選取停用的裝置、按一下滑鼠右鍵，然後按一下 [刪除]。 刪除裝置之後，StorSimple 會通知您，而且裝置清單也會隨即更新。

## <a name="next-steps"></a>後續步驟

* 若要將已停用的裝置還原為原廠預設值，請移至 [將裝置重設為原廠預設設定](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings)。
* 如需技術協助， [請連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。
* 若要深入了解如何使用 StorSimple 裝置管理員服務，請移至[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

