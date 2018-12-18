---
title: StorSimple Virtual Array 的災害復原和裝置容錯移轉 | Microsoft Docs
description: 深入了解如何容錯移轉 StorSimple Virtual Array。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 3c1f9c62-af57-4634-a0d8-435522d969aa
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 12079f8dbc409afe5acc274fa08bda878c90b76e
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110746"
---
# <a name="disaster-recovery-and-device-failover-for-your-storsimple-virtual-array-via-azure-portal"></a>透過 Azure 入口網站進行 StorSimple Virtual Array 的災害復原和裝置容錯移轉

## <a name="overview"></a>概觀
本文說明 Microsoft Azure StorSimple Virtual Array 的災害復原，包括容錯移轉至另一個虛擬陣列的詳細步驟。 容錯移轉可讓您將資料從資料中心的「來源」裝置移至「目標」裝置。 目標裝置可能位於相同或不同的地理位置。 裝置容錯移轉適用於整個裝置。 在容錯移轉期間，來源裝置的雲端資料會將擁有權變更為目標裝置雲端資料。

本文僅適用於 StorSimple Virtual Array。 若要容錯移轉 8000 系列裝置，請參閱 [StorSimple 裝置的裝置容錯移轉和災害復原](storsimple-device-failover-disaster-recovery.md)。

## <a name="what-is-disaster-recovery-and-device-failover"></a>什麼是災害復原和裝置容錯移轉？

在災害復原 (DR) 案例中，主要裝置會停止運作。 在此情況下，您可以將與故障裝置相關聯的雲端資料移至另一個裝置。 您可以使用主要裝置當做「來源」，並指定另一個裝置做為「目標」。 這個程序就稱為「容錯移轉」 。 在容錯移轉期間，來源裝置的所有磁碟區或共用都會變更擁有權，並移轉到目標裝置。 不允許篩選資料。

使用熱度圖分層和追蹤，可將 DR 模型化為完整裝置還原。 熱度圖的定義方式，是根據讀取和寫入模式，將熱度值指派給資料。 這個熱度圖接著先將最低熱度資料區塊分層至雲端，同時將高熱 (最常用) 資料區塊保留在本機層中。 在 DR 期間，StorSimple 會使用熱度圖來還原和解除凍結雲端中的資料。 裝置會擷取最新備份 (如內部決定) 中的所有磁碟區/共用，並從該備份執行還原。 虛擬陣列會協調整個 DR 程序。

> [!IMPORTANT]
> 裝置容錯移轉結束時會刪除來源裝置，因此不支援容錯回復。
> 
> 

災害復原是透過裝置容錯移轉功能來策劃，而且是從 [裝置] 刀鋒視窗起始。 此刀鋒視窗會以表格列出所有連接至 StorSimple 裝置管理員服務的 StorSimple 裝置。 您可以看到每個裝置的易記名稱、狀態、已佈建和最大容量、類型及模型。

## <a name="prerequisites-for-device-failover"></a>裝置容錯移轉需求

### <a name="prerequisites"></a>先決條件

對於裝置容錯移轉，請確定符合下列必要條件：

* 來源裝置需要處於 [已停用]  狀態。
* 在 Azure 入口網站中，目標裝置必須顯示為 [準備好進行設定] 。 佈建容量相同或更大的目標虛擬陣列。 使用本機 Web UI 來設定並成功註冊目標虛擬陣列。
  
  > [!IMPORTANT]
  > 請不要嘗試透過服務來設定已註冊的虛擬裝置。 您不應該透過服務執行任何裝置設定。
  > 
  > 
* 目標裝置不能與來源裝置同名。
* 來源和目標裝置的類型必須相同。 您只能將設定為檔案伺服器的虛擬陣列容錯移轉至另一個檔案伺服器。 這適用於 iSCSI 伺服器。
* 針對檔案伺服器 DR，建議您將目標裝置加入與來源相同的網域。 此設定可確保自動解析共用權限。 只容錯移轉至相同網域中的目標裝置。
* DR 的可用目標裝置是容量與來源裝置相同或更高的裝置。 已連接至服務但不符合足夠空間條件的裝置無法做為目標裝置。

### <a name="other-considerations"></a>其他考量

* 規劃的容錯移轉 
  
  * 建議您將來源裝置上的所有磁碟區或共用離線。
  * 建議您先備份裝置，再繼續容錯移轉，以儘可能避免資料遺失。 
* 如果是未規劃的容錯移轉，裝置會使用最新備份來還原資料。

### <a name="device-failover-prechecks"></a>裝置容錯移轉前置檢查

DR 開始之前，裝置會執行前置檢查。 這些檢查有助於確保 DR 開始時不會發生任何錯誤。 前置檢查包括：

* 驗證儲存體帳戶。
* 檢查 Azure 的雲端連線。
* 檢查目標裝置上的可用空間。
* 檢查 iSCSI 伺服器來源裝置磁碟區是否有
  
  * 有效的 ACR 名稱。
  * 有效的 IQN (不超過 220 個字元)。
  * 有效的 CHAP 密碼 (長度為 12-16 個字元)。

如果上述任何前置檢查失敗，則無法繼續進行 DR。 解決這些問題，然後重試 DR。

DR 順利完成之後，來源裝置上雲端資料的擁有權會移轉給目標裝置。 來源裝置就無法再於入口網站中使用。 會封鎖對來源裝置上所有磁碟區/共用的存取，而目標裝置會變成使用中。

> [!IMPORTANT]
> 雖然裝置無法再使用，但是您在主機系統上佈建的虛擬機器仍然會耗用資源。 DR 順利完成之後，您就可以從主機系統中刪除此虛擬機器。
> 
> 

## <a name="fail-over-to-a-virtual-array"></a>容錯移轉至虛擬陣列

執行此程序之前，建議您先佈建、設定另一個 StorSimple Virtual Array 並向 StorSimple 裝置管理員服務註冊。

> [!IMPORTANT]
> 
> * 您無法從 StorSimple 8000 系列裝置容錯移轉至 1200 虛擬裝置。
> * 您可以從一個通過美國聯邦資訊處理標準 (FIPS) 的虛擬裝置，容錯移轉至另一個通過 FIPS 的裝置，或容錯移轉至部署於政府入口網站中的非 FIPS 裝置。


請執行下列步驟以將裝置還原至目標 StorSimple 虛擬裝置。

1. 佈建及設定符合[裝置容錯移轉求](#prerequisites)的目標裝置。 透過本機 Web UI 完成裝置設定，並向 StorSimple 裝置管理員服務註冊裝置。 如果是建立檔案伺服器，請移至[設定為檔案伺服器](storsimple-virtual-array-deploy3-fs-setup.md#step-1-complete-the-local-web-ui-setup-and-register-your-device)的步驟 1。 如果是建立 iSCSI 伺服器，請移至[設定為 iSCSI 伺服器](storsimple-virtual-array-deploy3-iscsi-setup.md#step-1-complete-the-local-web-ui-setup-and-register-your-device)的步驟 1。

2. 將主機上的磁碟區/共用離線。 若要讓磁碟區/共用離線，請參閱依作業系統而定的主機指示。 如果尚未離線，您需要執行下列動作，讓裝置上的所有磁碟區/共用離線。
   
    1. 移至 [裝置] 刀鋒視窗，然後選取您的裝置。
   
    2. 移至 [設定] > [管理] > [共用] \(或 [設定] > [管理] > [磁碟區])。 
   
    3. 選取共用/磁碟區，按一下滑鼠右鍵並選取 [離線]。 
   
    4. 提示您確認時，請勾選 [我了解讓此共用離線的影響]。 
   
    5. 按一下 [離線]。

3. 在 StorSimple 裝置管理員服務中，移至 [管理] > [裝置]。 在 [裝置] 刀鋒視窗中，選取並按一下您的來源裝置。

4. 在 [裝置儀表板] 刀鋒視窗中，按一下 [停用]。

5. 在 [停用] 刀鋒視窗中，系統會提示您進行確認。 裝置停用是無法復原的「永久性」程序。 系統也會提醒您讓主機上的共用/磁碟區離線。 輸入裝置名稱加以確認，然後按一下 [停用]。
   
    ![](./media/storsimple-virtual-array-failover-dr/failover1.png)
6. 開始停用。 停用順利完成之後，您將會收到通知。
   
    ![](./media/storsimple-virtual-array-failover-dr/failover2.png)
7. 在 [裝置] 頁面上，裝置狀態現在會變更為 [已停用]。
    ![](./media/storsimple-virtual-array-failover-dr/failover3.png)
8. 在 [裝置] 刀鋒視窗上，選取並按一下已停用的裝置來進行容錯移轉。 
9. 在 [裝置儀表板] 刀鋒視窗中，按一下 [容錯移轉]。 
10. 在 [容錯移轉裝置] 刀鋒視窗中，執行下列動作︰
    
    1. [來源裝置] 欄位會自動填入。 請注意來源裝置的資料大小總計。 資料大小應該小於目標裝置上的可用容量。 檢閱與來源裝置相關聯的詳細資料，例如裝置名稱、容量總計，以及要容錯移轉的共用名稱。

    2. 從可用裝置的下拉式清單中，選擇 [目標裝置]。 下拉式清單中只會顯示具有足夠容量的裝置。

    3. 勾選 [我了解這項作業會將資料容錯移轉至目標裝置]。 

    4. 按一下 [容錯移轉]。
    
        ![](./media/storsimple-virtual-array-failover-dr/failover4.png)
11. 容錯移轉工作起始，您會收到通知。 移至 [裝置] > [作業] 來監視容錯移轉。
    
     ![](./media/storsimple-virtual-array-failover-dr/failover5.png)
12. 在 [作業] 刀鋒視窗中，您會看到針對來源裝置所建立的容錯移轉作業。 此工作會執行 DR 前置檢查。
    
    ![](./media/storsimple-virtual-array-failover-dr/failover6.png)
    
     DR 前置檢查成功之後，容錯移轉工作會產生來源裝置上每個共用/磁碟區的還原作業。
    
    ![](./media/storsimple-virtual-array-failover-dr/failover7.png)
13. 完成容錯移轉後，移至 [裝置] 刀鋒視窗。
    
    1. 選取並按一下已做為容錯移轉程序目標裝置的 StorSimple 裝置。
    2. 移至 [設定] > [管理] > [共用] \(如果是 iSCSI 伺服器，則移至 [磁碟區])。 在 [共用] 刀鋒視窗中，您可以檢視來自舊裝置的所有共用 (磁碟區)。
        ![](./media/storsimple-virtual-array-failover-dr/failover9.png)
14. 您將需要[建立 DNS 別名](https://support.microsoft.com/kb/168322)，讓所有嘗試連接的應用程式都可以重新導向至新的裝置。

## <a name="errors-during-dr"></a>DR 期間發生錯誤

**DR 期間雲端連線能力中斷**

從 DR 啟動之後到裝置還原完成之前，如果雲端連線中斷，DR 將會失敗。 您會收到失敗通知。 DR 的目標裝置會標示為 [無法使用]。 您無法將相同的目標裝置用於未來的 DR。

**沒有相容的目標裝置**

如果可用目標裝置的空間不足，您會看到錯誤，指出沒有相容的目標裝置。

**前置檢查失敗**

如果其中一項前置檢查未通過，您會看到前置檢查失敗。

## <a name="business-continuity-disaster-recovery-bcdr"></a>業務持續性災害復原 (BCDR)

當整個 Azure 資料中心停止運作時，就構成業務持續性災害復原 (BCDR) 狀況。 這會影響您的 StorSimple 裝置管理員服務和相關聯的 StorSimple 裝置。

如果 StorSimple 裝置在發生災害的前一刻才剛註冊，則可能需要刪除這些 StorSimple 裝置。 災害之後，您可以重建並設定這些裝置。

## <a name="next-steps"></a>後續步驟

深入了解如何 [使用本機 Web UI 管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

