---
title: Microsoft Azure StorSimple Virtual Array 備份教學課程 | Microsoft Docs
description: 說明如何備份 StorSimple Virtual Array 共用與磁碟區。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: e3cdcd9e-33b1-424d-82aa-b369d934067e
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c926f0c80ce56cac3106ad97ec3ec2e18a8e2cc6
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109566"
---
# <a name="back-up-shares-or-volumes-on-your-storsimple-virtual-array"></a>備份 StorSimple Virtual Array 上的共用或磁碟區

## <a name="overview"></a>概觀

StorSimple Virtual Array 是混合式雲端儲存體內部部署虛擬裝置，可設定為檔案伺服器或 iSCSI 伺服器。 虛擬陣列可讓使用者為裝置上的所有共用或磁碟區建立排程備份和手動備份。 設為檔案伺服器時，也可進行項目層級的復原。 本教學課程說明如何建立排程備份和手動備份，以及執行項目層級復原來還原虛擬陣列上已刪除的檔案。

本教學課程僅適用於 StorSimple Virtual Array。 如需 8000 系列的相關資訊，請參閱[建立 8000 系列裝置的備份](storsimple-manage-backup-policies-u2.md)

## <a name="back-up-shares-and-volumes"></a>備份共用和磁碟區

備份可提供共用和磁碟區的時間點保護、改善復原能力，同時讓還原時間降至最低。 您有兩種方法可以備份 StorSimple 裝置上的共用或磁碟區：「排程」或「手動」。 下列各節討論上述每一種方法。

## <a name="change-the-backup-start-time"></a>變更備份開始時間

> [!NOTE]
> 在此版本中，排程的備份由預設原則建立，該原則會每日在特定時間執行並備份裝置上的所有共用或磁碟區。 目前無法建立用於排程備份的自訂原則。


StorSimple Virtual Array 有預設的備份原則會在每日特定時間 (22:30) 啟動，每天備份一次裝置上的所有共用或磁碟區。 您可以變更備份開始的時間，但無法變更頻率及保留期 (指定備份保留的數量)。 這些在備份期間會備份整個虛擬裝置。 這可能會對影響裝置的效能和裝置上已部署的工作負載。 因此，建議您將這些備份排定在離峰時段。

 若要變更預設的備份開始時間，請在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟。

#### <a name="to-change-the-start-time-for-the-default-backup-policy"></a>變更預設備份原則的開始時間

1. 移至 [裝置]。 將會顯示已向您的 StorSimple 裝置管理員服務註冊的裝置清單。 
   
    ![瀏覽至裝置](./media/storsimple-virtual-array-backup/changebuschedule1.png)

2. 選取並按一下您的裝置。 [設定] 刀鋒視窗隨即顯示。 移至 [管理] > [備份原則]。
   
    ![選取您的裝置](./media/storsimple-virtual-array-backup/changebuschedule2.png)

3. 在 [備份原則] 刀鋒視窗中，預設開始時間為 22:30。 您可以在裝置時區中為每日排程指定新的開始時間。
   
    ![瀏覽至備份原則](./media/storsimple-virtual-array-backup/changebuschedule5.png)

4. 按一下 [檔案] 。

### <a name="take-a-manual-backup"></a>進行手動備份

除了排程備份，您隨時可以手動 (依需要) 備份裝置資料。

#### <a name="to-create-a-manual-backup"></a>建立手動備份

1. 移至 [裝置]。 選取您的裝置，然後以滑鼠右鍵按一下選取的資料列中最右邊的 [...]。 從操作能表中，選取 [進行備份]。
   
    ![瀏覽至進行備份](./media/storsimple-virtual-array-backup/takebackup1m.png)

2. 在 [進行備份] 刀鋒視窗中，按一下 [進行備份]。 這樣會備份檔案伺服器上的所有共用，或 iSCSI 伺服器上的所有磁碟區。 
   
    ![正在啟動備份](./media/storsimple-virtual-array-backup/takebackup2m.png)
   
    依需求備份開始執行，您會看到備份作業已啟動。
   
    ![正在啟動備份](./media/storsimple-virtual-array-backup/takebackup3m.png) 
   
    作業順利完成後會再次通知您。 備份程序接著開始。
   
    ![已建立備份工作](./media/storsimple-virtual-array-backup/takebackup4m.png)

3. 若要追蹤備份進度和查看作業詳細資料，請按一下通知。 這會帶您前往 [作業詳細資料]。
   
     ![備份作業詳細資料](./media/storsimple-virtual-array-backup/takebackup5m.png)

4. 備份完成後，請移至 [管理] > [備份目錄]。 您會看到裝置上所有共用 (或磁碟區) 的雲端快照。
   
    ![已完成的備份](./media/storsimple-virtual-array-backup/takebackup19m.png) 

## <a name="view-existing-backups"></a>檢視現有備份
若要檢視現有備份，請在 Azure 入口網站中執行下列步驟。

#### <a name="to-view-existing-backups"></a>檢視現有備份

1. 移至 [裝置] 刀鋒視窗。 選取並按一下您的裝置。 在 [設定] 刀鋒視窗中，移至 [管理] > [備份目錄]。
   
    ![瀏覽至備份目錄](./media/storsimple-virtual-array-backup/viewbackups1.png)
2. 指定下列準則以用於篩選︰
   
    - **時間範圍** – 可以是 [過去 1 小時]、[過去 24 小時]、[過去 7 天]、[過去 30 天]、[過去一年] 和 [自訂日期]。
    
    - **裝置** – 從已向 StorSimple 裝置管理員服務註冊的檔案伺服器或 iSCSI 伺服器清單中選擇。
   
    - **已起始** – 可以自動地 [已排程] \(依備份原則) 或 [手動] 起始 (由您執行)。
   
    ![篩選備份](./media/storsimple-virtual-array-backup/viewbackups2.png)

3. 按一下 [套用]。 已篩選的備份清單會顯示在 [備份目錄] 刀鋒視窗中。 請注意，永遠只會顯示 100 個備份項目。
   
    ![更新備份目錄](./media/storsimple-virtual-array-backup/viewbackups3.png)

## <a name="next-steps"></a>後續步驟

深入了解 [administering your StorSimple Virtual Array (管理 StorSimple Virtual Array)](storsimple-ova-web-ui-admin.md)。

