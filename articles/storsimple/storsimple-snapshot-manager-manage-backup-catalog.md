---
title: StorSimple Snapshot Manager 備份目錄 | Microsoft Docs
description: 說明如何使用 StorSimple Snapshot Manager MMC 嵌入式管理單元來檢視和管理備份磁碟區。
services: storsimple
documentationcenter: NA
author: SharS
manager: timlt
editor: ''
ms.assetid: 6abdbfd2-22ce-45a5-aa15-38fae4c8f4ec
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: v-sharos
ms.openlocfilehash: b97753e6f1b67e3c8d247281c5e5208033a56eca
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23111286"
---
# <a name="use-storsimple-snapshot-manager-to-manage-the-backup-catalog"></a>使用 StorSimple Snapshot Manager 來管理備份目錄

## <a name="overview"></a>概觀
StorSimple Snapshot Manager 的主要功能是可讓您以快照集的形式建立應用程式一致的 StorSimple 磁碟區備份複本。 快照集會列在稱為 *備份目錄*的 XML 檔案中。 備份目錄會根據磁碟區群組、然後根據本機快照集或雲端快照集組織快照集。

本教學課程描述如何使用 **備份目錄** 節點來完成下列工作：

* 還原磁碟區
* 複製磁碟區或磁碟區群組
* 刪除備份
* 復原檔案
* 還原 Storsimple Snapshot Manager 資料庫

您可以藉由展開 [範圍] 窗格中的**備份目錄**節點來檢視備份目錄，然後再展開磁碟區群組。

* 如果您按一下磁碟區群組名稱，[ **結果** ] 窗格會顯示磁碟區群組可用之本機快照集和雲端快照集的數目。 
* 如果您按一下 [本機快照集] 或 [雲端快照集]，[結果] 窗格會顯示下列每個備份快照集的資訊 (取決於您的**檢視**設定)：
  
  * **名稱** – 擷取快照集的時間。
  * **類型** – 這是本機快照集或雲端快照集。
  * **擁有者** – 內容擁有者。 
  * **可用** – 目前是否為可用的快照集。 **True** 表示快照集可供使用，且可還原；**False** 表示快照集已經無法使用。 
  * **匯入** – 是否已匯入備份。 **True** 表示裝置在 StorSimple Snapshot Manager 中設定時已從 StorSimple 裝置管理員服務匯入備份；**False** 表示備份尚未匯入，但已由 StorSimple Snapshot Manager 建立。 (您可以輕鬆地識別已匯入的磁碟區群組，因為已匯入磁碟區群組的裝置會新增尾碼以供識別。)
    
    ![備份類別目錄](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Backup_catalog.png)
* 如果您展開**本機快照集**或**雲端快照集**，然後按一下個別快照集名稱，[結果] 窗格會顯示您所選取之快照集的下列相關資訊：
  
  * **名稱** – 根據磁碟機代號識別的磁碟區。 
  * **本機名稱** – 磁碟機的本機名稱 (如果有的話)。 
  * **裝置** – 磁碟區所在的裝置名稱。 
  * **可用** – 目前是否為可用的快照集。 **True** 表示快照集可供使用，且可還原；**False** 表示快照集已經無法使用。 

## <a name="restore-a-volume"></a>還原磁碟區
使用下列程序從備份還原磁碟區。

#### <a name="prerequisites"></a>先決條件
如果您尚未這樣做，請建立磁碟區和磁碟區群組，然後再刪除磁碟區。 根據預設，StorSimple Snapshot Manager 會先備份磁碟區，才允許將它刪除。 如果不小心刪除磁碟區，或資料需要因為任何原因復原，這個預防措施就可以防止資料遺失。 

StorSimple Snapshot Manager 在建立預防措施備份時，會顯示下列訊息。

![自動快照訊息](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Automatic_snap.png) 

> [!IMPORTANT]
> 您無法刪除磁碟區，因為它是磁碟區群組的一部分的。 無法使用刪除選項。 <br>
> 
> 

#### <a name="to-restore-a-volume"></a>還原磁碟區
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。 
2. 在 [範圍] 窗格中，展開**備份目錄**節點，展開磁碟區群組，然後按一下 [本機快照集] 或 [雲端快照集]。 備份快照集清單會出現在 [ **結果** ] 窗格中。
3. 尋找您想要還原的備份，以滑鼠右鍵按一下，然後再按一下 [ **還原**]。
   
    ![還原備份目錄](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Restore_BU_catalog.png) 
4. 在確認頁面上，檢閱詳細資料，輸入**確認**，然後按一下 [確定]。 StorSimple Snapshot Manager 會使用備份來還原磁碟區。
   
    ![還原確認訊息](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Restore_volume_msg.png) 
5. 您可以在執行還原動作時加以監視。 在 [範圍] 窗格中，展開**作業**節點，然後按一下 [執行]。 工作詳細資料會出現在 [ **結果** ] 窗格中。 還原工作完成時，工作詳細資料會傳送到 **過去 24 小時** 清單。

## <a name="clone-a-volume-or-volume-group"></a>複製磁碟區或磁碟區群組
使用下列程序來建立磁碟區或磁碟區群組的重複 (複製)。

#### <a name="to-clone-a-volume-or-volume-group"></a>複製磁碟區或磁碟區群組
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，展開**備份目錄**節點，展開磁碟區群組，然後按一下 [雲端快照集]。 備份清單會出現在 [ **結果** ] 窗格中。
3. 尋找您想要複製的磁碟區或磁碟區群組，以滑鼠右鍵按一下磁碟區或磁碟區群組名稱，然後按一下 [ **複製**]。 [ **複製雲端快照集** ] 對話方塊隨即出現。
   
    ![複製雲端快照集](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Clone.png) 
4. 完成 [ **複製雲端快照集** ] 對話方塊，如下所示： 
   
   1. 在 [ **名稱** ] 文字方塊中，輸入複製之磁碟區的名稱。 這個名稱會出現在 **磁碟區** 節點中。 
   2. (選擇性) 選取 [ **磁碟機**]，然後從下拉式清單中選取磁碟機代號。
   3. (選擇性) 選取 [ **資料夾 (NTFS)**]，然後輸入資料夾路徑或按一下 [瀏覽] 並選取資料夾的位置。 
   4. 按一下頁面底部的 [新增] 。
5. 複製程序完成時，您必須初始化已複製的磁碟區。 啟動伺服器管理員，然後啟動磁碟管理。 如需詳細指示，請參閱 [掛接磁碟區](storsimple-snapshot-manager-manage-volumes.md#mount-volumes)。 初始化之後，磁碟區會列於 [範圍] 窗格中的**磁碟區**節點下。 如果您看不到列出的磁碟區，請重新整理磁碟區清單 (以滑鼠右鍵按一下**磁碟區**節點，然後再按一下 [重新整理])。

## <a name="delete-a-backup"></a>刪除備份
使用下列程序從備份目錄刪除快照集。 

> [!NOTE]
> 刪除快照集會刪除和快照集相關聯的備份資料。 不過，從雲端清除資料的程序可能需要一些時間。<br>


#### <a name="to-delete-a-backup"></a>刪除備份
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，展開**備份目錄**節點，展開磁碟區群組，然後按一下 [本機快照集] 或 [雲端快照集]。 快照集清單會出現在 [ **結果** ] 窗格中。
3. 以滑鼠右鍵按一下您想要刪除的快照集，然後按一下 [ **刪除**]。
4. 確認訊息出現時，按一下 [ **確定**]。

## <a name="recover-a-file"></a>復原檔案
如果不小心從磁碟區刪除檔案，您可以擷取刪除日以前的快照集、使用快照集建立磁碟區的複製，並從複製的磁碟區將檔案複製到原始磁碟區，即可復原檔案。

#### <a name="prerequisites"></a>先決條件
在您開始之前，請確定您有磁碟區群組的最新備份。 接著，刪除儲存在該磁碟區群組中磁碟區上的其中一個檔案。 最後，使用下列步驟從您的備份還原已刪除的檔案。 

#### <a name="to-recover-a-deleted-file"></a>復原已刪除的檔案
1. 按一下桌面上的 StorSimple Snapshot Manager 圖示。 StorSimple Snapshot Manager 主控台視窗隨即出現。 
2. 在 [範圍] 窗格中，展開**備份目錄**節點，然後瀏覽至包含已刪除檔案的快照集。 一般而言，您應該選取剛好在刪除之前建立的快照集。
3. 尋找您想要複製的磁碟區，以滑鼠右鍵按一下，然後按一下 [ **複製**]。 [ **複製雲端快照集** ] 對話方塊隨即出現。
   
    ![複製雲端快照集](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_Clone.png) 
4. 完成 [ **複製雲端快照集** ] 對話方塊，如下所示： 
   
   1. 在 [ **名稱** ] 文字方塊中，輸入複製之磁碟區的名稱。 這個名稱會出現在 **磁碟區** 節點中。 
   2. (選擇性) 選取 [ **磁碟機**]，然後從下拉式清單中選取磁碟機代號。 
   3. (選擇性) 選取 [資料夾 (NTFS)]，然後輸入資料夾路徑或按一下 [瀏覽] 並選取資料夾的位置。 
   4. 按一下頁面底部的 [新增] 。 
5. 複製程序完成時，您必須初始化已複製的磁碟區。 啟動伺服器管理員，然後啟動磁碟管理。 如需詳細指示，請參閱 [掛接磁碟區](storsimple-snapshot-manager-manage-volumes.md#mount-volumes)。 初始化之後，磁碟區會列於 [範圍] 窗格中的**磁碟區**節點下。 
   
    如果您看不到列出的磁碟區，請重新整理磁碟區清單 (以滑鼠右鍵按一下**磁碟區**節點，然後再按一下 [重新整理])。
6. 開啟包含已複製之磁碟區的 NTFS 資料夾，展開 **磁碟區** 節點，然後再開啟複製的磁碟區。 尋找您想要復原的檔案，並將它複製到主要磁碟區。
7. 還原檔案之後，您可以刪除包含已複製磁碟區的 NTFS 資料夾。

## <a name="restore-the-storsimple-snapshot-manager-database"></a>還原 Storsimple Snapshot Manager 資料庫
您應該在主機電腦上定期備份 StorSimple Snapshot Manager 資料庫。 如果發生災害，或主機電腦因任何原因故障，您就可以將它從備份中還原。 建立資料庫備份是手動程序。

#### <a name="to-back-up-and-restore-the-database"></a>備份和還原資料庫
1. 停止 Microsoft StorSimple 管理服務：
   
   1. 啟動伺服器管理員。
   2. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   3. 在 [服務] 視窗中，選取 [Microsoft StorSimple 管理服務]。
   4. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [停止服務]。
2. 在主機電腦上，瀏覽至 C:\ProgramData\Microsoft\StorSimple\BACatalog。 
   
   > [!NOTE]
   > ProgramData 是隱藏的資料夾。
   > 
   > 
3. 尋找目錄 XML 檔案、複製檔案，並將複本儲存在安全位置或雲端中。 如果主機失敗，您可以使用這個備份檔協助復原您在 StorSimple Snapshot Manager 中建立的備份原則。
   
    ![Azure StorSimple 備份目錄檔案](./media/storsimple-snapshot-manager-manage-backup-catalog/HCS_SSM_bacatalog.png)
4. 重新啟動 Microsoft StorSimple 管理服務： 
   
   1. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   2. 在 [服務] 視窗中，選取 [Microsoft StorSimple 管理服務]。
   3. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [重新啟動服務]。
5. 在主機電腦上，瀏覽至 C:\ProgramData\Microsoft\StorSimple\BACatalog。 
6. 刪除目錄 XML 檔案，並以您所建立的備份版本取代它。 
7. 按一下桌面的 StorSimple Snapshot Manager 圖示以啟動 StorSimple Snapshot Manager。 

## <a name="next-steps"></a>後續步驟
* 深入了解如何 [使用 StorSimple Snapshot Manager 來管理您的 StorSimple 解決方案](storsimple-snapshot-manager-admin.md)。
* 深入了解 [StorSimple Snapshot Manager 工作和工作流程](storsimple-snapshot-manager-admin.md#storsimple-snapshot-manager-tasks-and-workflows)。

