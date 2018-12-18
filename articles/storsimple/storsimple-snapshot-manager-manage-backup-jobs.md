---
title: StorSimple Snapshot Manager 備份工作 | Microsoft Docs
description: 描述如何使用 StorSimple Snapshot Manager MMC 嵌入式管理單元來檢視和管理已排程、目前執行中和已完成的備份作業。
services: storsimple
documentationcenter: NA
author: SharS
manager: timlt
editor: ''
ms.assetid: bf4dcff6-c819-4766-b9d9-9922831cb200
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: v-sharos
ms.openlocfilehash: 03e306b62250f2bb033cc14e856a59760b5406c3
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109856"
---
# <a name="use-storsimple-snapshot-manager-to-view-and-manage-backup-jobs"></a>使用 StorSimple Snapshot Manager 來檢視和管理備份作業

## <a name="overview"></a>概觀
[範圍] 窗格中的 [作業] 節點會顯示您以互動方式或透過已設定之原則起始的 [已排程]、[過去 24 小時] 和 [執行中] 備份工作。 

本教學課程說明如何使用 [ **作業** ] 節點，以顯示已排程、最近和目前執行中之備份作業的相關資訊。 (作業清單和對應資訊會出現在 [結果] 窗格中)。此外，您也可以以滑鼠右鍵按一下列出的作業，並查看內容功能表，其中列出可用的動作。

## <a name="view-scheduled-jobs"></a>檢視已排程的作業
請使用下列程序來檢視已排程的備份作業。

#### <a name="to-view-scheduled-jobs"></a>若要檢視已排程的作業
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。 
2. 在 [範圍] 窗格中，展開 [作業] 節點，然後按一下 [已排程]。 下列資訊會出現在 [ **結果** ] 窗格中：
   
   * **名稱**  – 已排定之快照的名稱
   * **下次執行**  – 下次排定之快照的日期和時間
   * **上次執行**  – 最新排定之快照的日期和時間
     
     > [!NOTE]
     > 若為僅一次快照，[下次執行] 和 [上次執行] 將相同。
     
     ![排定的備份工作](./media/storsimple-snapshot-manager-manage-backup-jobs/HCS_SSM_Jobs_scheduled.png) 
3. 若要在特定工作上執行其他動作，請以滑鼠右鍵按一下 [ **結果** ] 窗格中的作業名稱，然後選取功能表選項。

## <a name="view-recent-jobs"></a>檢視最近的作業
請使用下列程序，來檢視過去 24 小時內完成的備份和還原作業。

#### <a name="to-view-recent-jobs"></a>若要檢視最近的作業
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，展開 [作業] 節點，然後按一下 [過去 24 小時]。 [ **結果** ] 窗格會顯示過去 24 小時的備份作業 (最多可顯示 64 個作業)。 下列資訊會出現在 [結果] 窗格中，視您指定的 [檢視] 選項而定：
   
   * **名稱**  – 已排定之快照的名稱。
   * **已啟動**  – 快照開始的日期和時間。
   * **已停止**  – 快照完成或已終止的日期和時間。
   * [經過時間] – [已啟動] 時間與 [已停止] 時間之間的時間量。
   * **狀態**  – 最近完成之工作的狀態。 **成功**  指出已順利建立備份。 **失敗**  指出作業並未成功執行。
   * **資訊**  – 失敗的原因。
   * **已處理的位元組數 (MB)**  – 來自磁碟區群組已處理的資料量 (以 MB 為單位)。 
     
     ![過去 24 小時內執行的工作](./media/storsimple-snapshot-manager-manage-backup-jobs/HCS_SSM_Jobs_Last_24_hours.png) 
3. 若要在特定工作上執行其他動作，請以滑鼠右鍵按一下 [ **結果** ] 窗格中的作業名稱，然後選取功能表選項。
   
    ![刪除工作](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Delete_backup.png)

## <a name="view-currently-running-jobs"></a>檢視目前執行中的作業
請使用下列程序來檢視目前執行中的作業。

#### <a name="to-view-currently-running-jobs"></a>若要檢視目前執行中的工作
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，展開 [作業] 節點，然後按一下 [執行中]。 視您指定的 [檢視] 選項而定，下列資訊會出現在 [結果] 窗格中：
   
   * **名稱**  – 已排定之快照的名稱。
   * **已啟動**  – 快照開始的日期和時間。
   * **檢查點**  – 備份的目前動作。
   * **狀態**  – 完成百分比。
   * **經過時間**  – 從備份開始後所經過的時間量。 
   * **平均輸送量 (MB)** – 處理的總資料位元組與處理所花時間的比率 (以 MB 為單位)。
   * **處理的位元組 (MB)** – 處理的總資料位元組 (以 MB 為單位)。
   * **寫入的位元組 (MB)** – 寫入的總資料位元組 (以 MB 為單位)。 它包含資料以及中繼資料，因此通常大於處理的位元組。
     
     ![目前執行中的工作](./media/storsimple-snapshot-manager-manage-backup-jobs/HCS_SSM_Jobs_running.png)
3. 若要在特定工作上執行其他動作，請以滑鼠右鍵按一下 [ **結果** ] 窗格中的作業名稱，然後選取功能表選項。

## <a name="next-steps"></a>後續步驟
* 了解如何 [使用 StorSimple Snapshot Manager 來管理您的 StorSimple 解決方案](storsimple-snapshot-manager-admin.md)。
* 了解如何 [使用 StorSimple Snapshot Manager 管理備份目錄](storsimple-snapshot-manager-manage-backup-catalog.md)。

