---
title: 複製 StorSimple Virtual Array 備份 | Microsoft Docs
description: 了解如何從 StorSimple Virtual Array 複製備份和復原檔案。
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: af6e979c-55e3-477c-b53e-a76a697f80c9
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/21/2016
ms.author: alkohli
ms.openlocfilehash: 768c9a1c906999f4690c9c8f7d075743ab1678ff
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110106"
---
# <a name="clone-from-a-backup-of-your-storsimple-virtual-array"></a>從 StorSimple Virtual Array 的備份複製

## <a name="overview"></a>概觀

本文逐步說明如何複製 Microsoft Azure StorSimple Virtual Array 上的共用或磁碟區的備份組。 複製的備份用來復原已刪除或遺失的檔案。 本文也包含如何在設定為檔案伺服器的 StorSimple Virtual Array 上執行項目層級復原的詳細步驟。

## <a name="clone-shares-from-a-backup-set"></a>從備份組複製共用

**嘗試複製共用之前，請確定裝置上有足夠的空間可完成這項作業。** 若要從備份複製，請在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟。

#### <a name="to-clone-a-share"></a>複製共用

1. 瀏覽至 [裝置] 刀鋒視窗。 選取並按一下您的裝置，然後按一下 [共用]。 選取您想要複製的共用，以滑鼠右鍵按一下共用以叫用操作功能表。 選取 [複製]。
   
   ![複製備份](./media/storsimple-virtual-array-clone/cloneshare1.png)
2. 在 [複製] 刀鋒視窗中，按一下 [備份] > [選取]，然後執行下列動作︰ 
   
   a.    根據時間範圍篩選此裝置上的備份。 您可以選擇 [過去 7 天]、[過去 30 天] 和 [過去一年]。
   
   b.    在顯示的已篩選備份清單中，選取要複製的備份。
   
   c.    按一下 [SERVICEPRINCIPAL] 。
   
   ![複製備份](./media/storsimple-virtual-array-clone/cloneshare3.png)
3. 在 [複製] 刀鋒視窗中，按一下 [目標設定]，然後執行下列動作︰
   
   a.    提供共用名稱。 共用名稱必須包含 3-127 個字元。
   
   b.    對於複製的共用，選擇性地提供描述。
   
   c.    您無法變更還原的目標共用類型。 階層式共用會複製為階層式，固定在本機的共用則會還原為固定在本機。
   
   d.    容量設定為等於您複製的來源共用大小。
   
   e.    指派此共用的系統管理員。 複製完成後，您可以透過 [檔案總管] 修改共用內容。
   
   f.    按一下 [SERVICEPRINCIPAL] 。
   
   ![複製備份](./media/storsimple-virtual-array-clone/cloneshare6.png)

4. 按一下 [複製] 啟動複製作業。 作業完成後，複製作業會開始並通知您。 若要監視複製進度，請移至 [作業] 刀鋒視窗，按一下作業來檢視作業詳細資料。
5. 成功建立複製之後，瀏覽回到裝置的 [共用] 刀鋒視窗。
6. 您現在可以在裝置的共用清單中檢視剛複製的共用。 階層式共用會複製為階層式，固定在本機的共用則會複製為固定在本機的共用。
   
   ![複製備份](./media/storsimple-virtual-array-clone/cloneshare10.png)

## <a name="clone-volumes-from-a-backup-set"></a>從備份組複製磁碟區

若要從備份複製，您必須在 Azure 入口網站中執行類似於複製共用時的步驟。 複製作業會將備份複製到相同虛擬裝置上的新磁碟區；您無法複製到不同的裝置。

#### <a name="to-clone-a-volume"></a>若要複製磁碟區

1. 瀏覽至 [裝置] 刀鋒視窗。 選取並按一下您的裝置，然後按一下 [磁碟區]。 選取您想要複製的磁碟區，以滑鼠右鍵按一下磁碟區以叫用操作功能表。 選取 [複製]。
   
   ![複製磁碟區](./media/storsimple-virtual-array-clone/clonevolume1.png)
2. 在 [複製] 刀鋒視窗中，按一下 [備份]，然後執行下列動作︰ 
   
   a.    根據時間範圍篩選此裝置上的備份。 您可以選擇 [過去 7 天]、[過去 30 天] 和 [過去一年]。 
   
   b.    在顯示的已篩選備份清單中，選取要複製的備份。
   
   c.    按一下 [SERVICEPRINCIPAL] 。
   
   ![複製備份](./media/storsimple-virtual-array-clone/clonevolume3.png)
3. 在 [複製] 刀鋒視窗中，按一下 [目標磁碟區設定]，然後執行下列動作︰
   
   a. 裝置名稱會自動填入。
   
   b. 在 [複製的磁碟區] 中提供磁碟區名稱。 磁碟區名稱必須包含 3 到 127 個字元。
   
   c. 磁碟區類型會自動設定為原始磁碟區。 階層式磁碟區會複製為階層式，固定在本機的磁碟區則會複製為固定在本機。
   
   d. 在 [連接的主機] 中，按一下 [選取]。
   
   ![複製備份](./media/storsimple-virtual-array-clone/clonevolume4.png)
4. 在 [連接的主機] 刀鋒視窗中，選取現有的 ACR 或新增 ACR。 若要新增 ACR，您必須提供 ACR 名稱和主機 IQN。 按一下 [選取] 。
   
   ![複製備份](./media/storsimple-virtual-array-clone/clonevolume5.png)
5. 按一下 [複製] 啟動複製作業。
   
   ![複製備份](./media/storsimple-virtual-array-clone/clonevolume6.png)  
6. 建立複製作業之後將會開始複製。 建立複製後，就會顯示在裝置的 [磁碟區] 刀鋒視窗上。 請注意，階層式磁碟區會複製為階層式，固定在本機的磁碟區則會複製為固定在本機的磁碟區。
   
   ![複製備份](./media/storsimple-virtual-array-clone/clonevolume8.png)
7. 當磁碟區出現在線上的磁碟區清單上後，該磁碟區即可供使用。 在 iSCSI 啟動器主機上，重新整理 iSCSI 啟動器屬性視窗中的目標清單。 複製的磁碟區名稱所在的新目標，應該會在狀態資料行下方顯示為 [非作用中]。
8. 選取目標並按一下 [連接] 。 當啟動器連接到目標之後，狀態應會變更為 [已連接] 。
9. 在 [磁碟管理] 視窗中，將會出現掛接的磁碟區，如下圖所示。 以滑鼠右鍵按一下探索到的磁碟區 (按一下磁碟名稱)，然後按一下 [線上] 。

> [!IMPORTANT]
> 嘗試從備份組複製磁碟區或共用時，如果複製作業失敗，仍可能在入口網站中建立目標磁碟區或共用。 請務必在入口網站中刪除這個目標磁碟區或共用，以將未來這個項目所造成的任何問題降至最低。
> 
> 

## <a name="item-level-recovery-ilr"></a>項目層級復原 (ILR)

此版本在設定為檔案伺服器的 StorSimple Virtual Array 上引進項目層級復原 (ILR)。 此功能可讓您從 StorSimple 裝置上所有共用的雲端備份執行檔案和資料夾的細微復原。 您可以使用自助式模型，從最新的備份中擷取已刪除的檔案。

每個共用都有一個包含最新備份的 [.backups]  資料夾。 您可以導覽至想要的備份，並從備份中複製相關的檔案和資料夾，然後進行還原。 此功能不需要連絡系統管理員從備份中還原檔案。

1. 執行 ILR 時，您可以透過 [檔案總管] 來檢視備份。 按一下您想要查看備份的特定共用。 您將會在共用下方，看到建立來儲存所有備份的 [.backups]  資料夾。 展開 [.backups]  資料夾以檢視備份。 這個資料夾會顯示整個備份階層的展開檢視。 此檢視是依需求建立，而且通常只需要數秒的時間就能建立完成。
   
   最後五個備份會以這種方式顯示，而且可用來執行項目層級復原。 這五個最近的備份包括預設的排程備份和手動備份。
   
   * **排程備份**，命名為 &lt;裝置名稱&gt;DailySchedule-YYYYMMDD-HHMMSS-UTC。
   * **手動備份** ，命名為 Ad-hoc-YYYYMMDD-HHMMSS-UTC。
     
     ![](./media/storsimple-virtual-array-clone/image14.png)

2. 識別包含已刪除檔案之最新版本的備份。 在上述所有情況下，雖然資料夾名稱包含 UTC 時間戳記，但是資料夾建立時間是備份啟動時的實際裝置時間。 使用資料夾時間戳記來尋找並識別備份。

3. 在上一個步驟所識別的備份中，找到您想要還原的資料夾或檔案。 請注意，您只能檢視具備權限的檔案或資料夾。 如果您無法存取某些檔案或資料夾，請連絡共用系統管理員。 系統管理員可以使用 [檔案總管] 來編輯共用權限，並授權您存取特定的檔案或資料夾。 建議的最佳做法是共用系統管理員為使用者群組，而不是單一使用者。

4. 將檔案或資料夾複製到您的 StorSimple 檔案伺服器上的適當共用。

## <a name="next-steps"></a>後續步驟

深入了解如何 [使用本機 Web UI 管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

