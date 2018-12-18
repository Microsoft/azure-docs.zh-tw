---
title: StorSimple Snapshot Manager 及磁碟區 | Microsoft Docs
description: 描述如何使用 StorSimple Snapshot Manager MMC 嵌入式管理單元來檢視和管理磁碟區及設定備份。
services: storsimple
documentationcenter: NA
author: SharS
manager: carmonm
editor: ''
ms.assetid: 78896323-e57c-431e-bbe2-0cbde1cf43a2
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 04/18/2016
ms.author: v-sharos
ms.openlocfilehash: 2c0b211bced99d272a73a7b018a22f99d8d58aa9
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23111716"
---
# <a name="use-storsimple-snapshot-manager-to-view-and-manage-volumes"></a>使用 StorSimple Snapshot Manager 來檢視和管理磁碟區
## <a name="overview"></a>概觀
您可以使用 StorSimple Snapshot Manager 的 [磁碟區] 節點 (在 [範圍] 窗格上)，選取磁碟區並檢視其相關資訊。 磁碟區會呈現為對應至主機所掛接磁碟區的磁碟機。 [磁碟區]  節點會顯示 StorSimple 所支援的本機磁碟區和磁碟區類型，包括透過使用 iSCSI 及裝置探索到的磁碟區。 

如需支援之磁碟區的詳細資訊，請移至《 [支援多個磁碟區類型](storsimple-what-is-snapshot-manager.md#support-for-multiple-volume-types)》。

![在 [結果] 窗格中的磁碟區清單](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Volume_node.png)

[磁碟區]  節點也可讓您在 StorSimple Snapshot Manager 探索到磁碟區之後，重新掃描或刪除磁碟區。 

本教學課程說明如何掛載、初始化和格式化磁碟區，然後使用 StorSimple Snapshot Manager：

* 檢視磁碟區的相關資訊 
* 刪除磁碟區
* 重新掃描磁碟區 
* 設定基本磁碟區並將其備份
* 設定動態鏡像磁碟區並將其備份

> [!NOTE]
> 所有 [磁碟區] 節點動作也可以在 [動作] 窗格中取得。
> 
> 

## <a name="mount-volumes"></a>掛接磁碟區
請使用下列程序，來掛接、初始化和格式化 StorSimple 磁碟區。 此程序使用 [磁碟管理]，這是一種系統公用程式，用於管理硬碟及其相對應的磁碟區或磁碟分割。 如需 [磁碟管理] 的詳細資訊，請移至 Microsoft TechNet 網站上的 [磁碟管理](https://technet.microsoft.com/library/cc770943.aspx) 。

#### <a name="to-mount-volumes"></a>若要掛接磁碟區
1. 在您的主機電腦上，啟動 Microsoft iSCSI 啟動器。
2. 提供其中一個介面 IP 位址作為目標入口網站，或探索 IP 位址，然後連接至裝置。 在連接裝置之後，您的 Windows 系統將可存取磁碟區。 如需使用 Microsoft iSCSI 啟動器的詳細資訊，請移至[安裝和設定 Microsoft iSCSI 啟動器][1]中的〈連接至 iSCSI 目標裝置〉一節。
3. 請使用下列任一個選項來啟動 [磁碟管理]：
   
   * 在 [執行]  方塊中輸入 Diskmgmt.msc。
   * 啟動 [伺服器管理員]，展開 [儲存體] 節點，然後選取 [磁碟管理]。
   * 啟動 [系統管理工具]，展開 [電腦管理] 節點，然後選取 [磁碟管理]。 
     
     > [!NOTE]
     > 您必須使用系統管理員權限，才能執行 [磁碟管理]。
     > 
     > 
4. 將磁碟區連線：
   
   1. 在 [磁碟管理] 中，以滑鼠右鍵按一下任何標示為 [離線] 的磁碟區。
   2. 按一下 [重新啟動磁碟] 。 磁碟重新啟動之後，應該標示為 [線上]  。
5. 初始化磁碟區：
   
   1. 以滑鼠右鍵按一下已探索到的磁碟區。
   2. 在功能表上，選取 [初始化磁碟] 。
   3. 在 [初始化磁碟] 對話方塊中，選取您要初始化的磁碟，然後按一下 [確定]。
6. 格式化簡單磁碟區：
   
   1. 以滑鼠右鍵按一下您想要格式化的磁碟區。
   2. 在功能表上，選取 [新增簡單磁碟區] 。
   3. 使用 [新增簡單磁碟區] 精靈來格式化磁碟區：
      
      * 指定磁碟區大小。
      * 提供磁碟機代號。
      * 選取 NTFS 檔案系統。
      * 指定 64 KB 的配置單位大小。
      * 執行快速格式化。
7. 格式化多重磁碟分割磁碟區。 如需指示，請移至《 [實作磁碟管理](https://msdn.microsoft.com/library/dd163556.aspx)》的〈磁碟分割與磁碟區〉一節。

## <a name="view-information-about-your-volumes"></a>檢視您磁碟區的相關資訊
請使用下列程序，來檢視本機和 Azure StorSimple 磁碟區的相關資訊。

#### <a name="to-view-volume-information"></a>若要檢視磁碟區資訊
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。 
2. 在 [範圍] 窗格中，按一下 [磁碟區] 節點。 本機與掛接的磁碟區 (包括所有的 Azure StorSimple 磁碟區) 的清單會出現在 [結果] 窗格中。 您可以設定 [結果] 窗格中的資料行。 (以滑鼠右鍵按一下 [磁碟區] 節點，選取 [檢視]，然後選取 [新增/移除資料行]。)
   
    ![設定資料行](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_View_volumes.png)
   
   | 結果資料行 | 說明 |
   |:--- |:--- |
   |  Name |[名稱]  資料行包含已指派至每個已探索到之磁碟區的磁碟機代號。 |
   |  裝置 |[裝置]  資料行包含已連接到主機電腦之裝置的 IP 位址。 |
   |  裝置磁碟區名稱 |[裝置磁碟區名稱]  資料行包含所選磁碟區所屬之裝置磁碟區的名稱。 這是 Azure 入口網站中針對該特定磁碟區定義的磁碟區名稱。 |
   |  存取路徑 |[存取路徑]  資料行會顯示磁碟區的存取路徑。 這是可在主機電腦上存取磁碟區的磁碟機代號或掛接點。 |

## <a name="delete-a-volume"></a>刪除磁碟區
請使用下列程序，從 StorSimple Snapshot Manager 中刪除磁碟區。

> [!NOTE]
> 如果磁碟區是任何磁碟區群組的一部分，則無法刪除。 (磁碟區若是磁碟區群組的成員，無法對其使用刪除選項。)您必須刪除整個磁碟區群組，才能刪除磁碟區。

#### <a name="to-delete-a-volume"></a>若要刪除磁碟區
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，按一下 [磁碟區] 節點。 
3. 在 [結果]  窗格中，以滑鼠右鍵按一下您想要刪除的磁碟區。
4. 在功能表上，按一下 [刪除] 。 
   
    ![刪除磁碟區](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Delete_volume.png) 
5. [刪除磁碟區] 對話方塊隨即出現。 在文字方塊中輸入 **Confirm**，然後按一下 [確定]。
6. 根據預設，StorSimple Snapshot Manager 會先備份磁碟區，再將其刪除。 此預防措施可讓您在意外刪除時避免資料損失。 StorSimple Snapshot Manager 會在備份磁碟區時顯示 [自動快照]  進度訊息。 
   
    ![自動快照訊息](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Automatic_snap.png) 

## <a name="rescan-volumes"></a>重新掃描磁碟區
請使用下列程序，來重新掃描已連接至 StorSimple Snapshot Manager 的磁碟區。

#### <a name="to-rescan-the-volumes"></a>若要重新掃描磁碟區
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，以滑鼠右鍵按一下 [磁碟區]，然後按一下 [重新掃描磁碟區]。
   
    ![重新掃描磁碟區](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Rescan_volumes.png)
   
    此程序會同步處理磁碟區清單與 StorSimple Snapshot Manager。 任何變更 (例如新增的磁碟區或已刪除的磁碟區) 將會反映在結果中。

## <a name="configure-and-back-up-a-basic-volume"></a>設定和備份基本磁碟區
請使用下列程序，來設定基本磁碟區的備份，然後立即啟動備份，或建立一個原則進行排程的備份。

### <a name="prerequisites"></a>先決條件
開始之前：

* 確定已正確設定 StorSimple 裝置和主機電腦。 如需詳細資訊，請移至 [部署內部部署 StorSimple 裝置](storsimple-deployment-walkthrough-u2.md)。
* 安裝和設定 StorSimple Snapshot Manager。 如需詳細資訊，請移至 [部署 StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md)。

#### <a name="to-configure-backup-of-a-basic-volume"></a>若要設定基本磁碟區的備份
1. 在 StorSimple 裝置上建立基本磁碟區。
2. 依 [掛接磁碟區](#mount-volumes)所述掛接、初始化和格式化磁碟區。 
3. 按一下桌面上的 StorSimple Snapshot Manager 圖示。 StorSimple Snapshot Manager 視窗隨即出現。 
4. 在 [範圍] 窗格中，以滑鼠右鍵按一下 [磁碟區] 節點，然後選取 [重新掃描磁碟區]。 掃描完成時，磁碟區清單應該會出現在 [結果]  窗格中。 
5. 在 [結果] 窗格中，以滑鼠右鍵按一下磁碟區，然後選取 [建立磁碟區群組]。 
   
    ![建立磁碟區群組](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Create_volume_group.png) 
6. 在 [建立磁碟區群組] 對話方塊中，輸入磁碟區群組的名稱，並為其指派磁碟區，然後按一下 [確定]。
7. 在 [範圍] 窗格中，展開 [磁碟區群組] 節點。 新的磁碟區群組應該會出現在 [磁碟區群組]  節點之下。 
8. 以滑鼠右鍵按一下磁碟區群組名稱。
   
   * 若要以互動方式 (依需求) 啟動備份作業，請按一下 [進行備份] 。 
   * 若要排程自動備份，請按一下 [建立備份原則] 。 在 [一般]  頁面上，從清單中選取磁碟區群組。 在 [排程]  頁面上，輸入排程詳細資料。 完成時，請按一下 [確定] 。 
9. 若要確認備份作業已啟動，請展開 [範圍] 窗格中的 [作業] 節點，然後按一下 [執行中] 節點。 目前執行中的工作清單會出現在 [結果]  窗格中。 

## <a name="configure-and-back-up-a-dynamic-mirrored-volume"></a>設定和備份動態鏡像磁碟區
請完成下列步驟來設定動態鏡像磁碟區的備份：

* 步驟 1：使用 [磁碟管理] 來建立動態鏡像磁碟區。 
* 步驟 2：使用 StorSimple Snapshot Manager 來設定備份。

### <a name="prerequisites"></a>先決條件
開始之前：

* 確定已正確設定 StorSimple 裝置和主機電腦。 如需詳細資訊，請移至 [部署內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)。
* 安裝和設定 StorSimple Snapshot Manager。 如需詳細資訊，請移至 [部署 StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md)。
* 在 StorSimple 裝置上設定兩個磁碟區。 (在這些範例中，可用的磁碟區為 [磁碟 1] 和 [磁碟 2]。) 

### <a name="step-1-use-disk-management-to-create-a-dynamic-mirrored-volume"></a>步驟 1：使用 [磁碟管理] 來建立動態鏡像磁碟區
[磁碟管理] 是一種系統公用程式，用於管理硬碟及其包含的磁碟區或磁碟分割。 如需 [磁碟管理] 的詳細資訊，請移至 Microsoft TechNet 網站上的 [磁碟管理](https://technet.microsoft.com/library/cc770943.aspx) 。

#### <a name="to-create-a-dynamic-mirrored-volume"></a>若要建立動態鏡像磁碟區
1. 請使用下列任一個選項來啟動 [磁碟管理]： 
   
   * 開啟 [執行] 方塊，輸入 **Diskmgmt.msc**，然後按下 Enter 鍵。
   * 啟動 [伺服器管理員]，展開 [儲存體] 節點，然後選取 [磁碟管理]。 
   * 啟動 [系統管理工具]，展開 [電腦管理] 節點，然後選取 [磁碟管理]。 
2. 確定您在 StorSimple 裝置上有兩個可用的磁碟區。 (在此範例中，可用的磁碟區為 [磁碟 1] 和 [磁碟 2])。 
3. 在 [磁碟管理] 視窗中，於下方窗格右邊的資料行中，以滑鼠右鍵按一下 [磁碟 1]，然後選取 [新增鏡像磁碟區]。 
   
    ![新鏡像磁碟區](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_New_mirrored_volume.png) 
4. 在 [新增鏡像磁碟區] 精靈頁面上，按 [下一步]。
5. 在 [選取磁碟] 頁面上，於 [已選取] 窗格中選取 [磁碟 2]，按一下 [新增]，然後按一下 [下一步]。 
6. 在 [指派磁碟機代號或路徑] 頁面上，接受預設值，然後按一下 [下一步]。 
7. 在 [格式化磁碟區] 頁面上，於 [配置單位大小] 方塊中，選取 [64K]。 選取 [執行快速格式化] 核取方塊，然後按一下 [下一步]。 
8. 在 [完成新增鏡像磁碟區] 頁面上，檢閱您的設定，然後按一下 [完成]。 
9. 訊息會出現，以指出基本磁碟將會轉換成動態磁碟。 按一下 [是] 。
   
    ![動態磁碟轉換訊息](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Disk_management_msg.png) 
10. 在磁碟管理中，確認磁碟區 1 與磁碟區 2 顯示為動態鏡像磁碟區。 (狀態欄中應該會顯示 [動態]，容量條顏色應變為紅色表示鏡像磁碟區。) 
    
    ![磁碟管理的鏡像動態磁碟](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Verify_dynamic_disks_2.png) 

### <a name="step-2-use-storsimple-snapshot-manager-to-configure-backup"></a>步驟 2：使用 StorSimple Snapshot Manager 來設定備份
請使用下列程序，來設定動態鏡像磁碟區，然後立即啟動備份，或建立一個原則進行排程的備份。

#### <a name="to-configure-backup-of-a-dynamic-mirrored-volume"></a>若要設定動態鏡像磁碟區的備份
1. 按一下桌面上的 StorSimple Snapshot Manager 圖示。 StorSimple Snapshot Manager 視窗隨即出現。 
2. 在 [範圍] 窗格中，以滑鼠右鍵按一下 [磁碟區] 節點，然後選取 [重新掃描磁碟區]。 掃描完成時，磁碟區清單應該會出現在 [結果]  窗格中。 動態鏡像磁碟區會列為單一磁碟區。 
3. 在 [結果] 窗格中，以滑鼠右鍵按一下動態鏡像磁碟區，然後按一下 [建立磁碟區群組]。 
4. 在 [建立磁碟區群組] 對話方塊中，輸入磁碟區群組的名稱，將動態鏡像磁碟區指派給這個群組，然後按一下 [確定]。 
5. 在 [範圍] 窗格中，展開 [磁碟區群組] 節點。 新的磁碟區群組應該會出現在 [磁碟區群組] 節點之下。 
6. 以滑鼠右鍵按一下磁碟區群組名稱。 
   
   * 若要以互動方式 (依需求) 啟動備份作業，請按一下 [進行備份] 。 
   * 若要排程自動備份，請按一下 [建立備份原則] 。 在 [一般]  頁面上，從清單中選取該磁碟區群組。 在 [排程]  頁面上，輸入排程詳細資料。 完成時，請按一下 [確定] 。 
7. 您可以在備份作業執行時加以監視。 在 [範圍] 窗格中，展開 [作業] 節點，然後按一下 [執行中]。作業詳細資料會出現在 [結果] 窗格中。 備份作業完成時，詳細資料會傳輸至 [過去 24 小時] 作業清單。 

## <a name="next-steps"></a>後續步驟
* 了解如何 [使用 StorSimple Snapshot Manager 來管理您的 StorSimple 解決方案](storsimple-snapshot-manager-admin.md)。
* 了解如何 [使用 StorSimple Snapshot Manager 來建立和管理磁碟區群組](storsimple-snapshot-manager-manage-volume-groups.md)。

<!--Reference links-->
[1]: https://msdn.microsoft.com/library/ee338480(v=ws.10).aspx
