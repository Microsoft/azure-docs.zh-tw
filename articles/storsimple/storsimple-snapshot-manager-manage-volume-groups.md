---
title: StorSimple Snapshot Manager 磁碟區群組 | Microsoft Docs
description: 描述如何使用 StorSimple Snapshot Manager MMC 嵌入式管理單元，來建立和理磁碟區群組。
services: storsimple
documentationcenter: NA
author: SharS
manager: carmonm
editor: ''
ms.assetid: 7a232414-6a28-4b81-bd7b-cf61e28b33d7
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: v-sharos
ms.openlocfilehash: 6067a88cd42d29c3d2f4b74580095424de77561e
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "23109846"
---
# <a name="use-storsimple-snapshot-manager-to-create-and-manage-volume-groups"></a>使用 StorSimple Snapshot Manager 來建立和管理磁碟區群組
## <a name="overview"></a>概觀
您可以使用 [範圍] 窗格上的 [磁碟區群組] 節點，將磁碟區指派給磁碟區群組、檢視磁碟區群組的相關資訊、排定備份，以及編輯磁碟區群組。

磁碟區群組是用來確保應用程式具有一致備份之相關磁碟區的集區。 如需詳細資訊，請參閱[磁碟區和磁碟區群組](storsimple-what-is-snapshot-manager.md#volumes-and-volume-groups)，以及[與 Windows 磁碟區陰影複製服務整合](storsimple-what-is-snapshot-manager.md#integration-with-windows-volume-shadow-copy-service)。

> [!IMPORTANT]
> * 磁碟區群組中的所有磁碟區必須來自單一雲端服務提供者。
> * 當設定磁碟區群組時，請勿在相同的磁碟區群組中混用叢集共用磁碟區 (CSV) 和非 CSV。 StorSimple Snapshot Manager 不支援在相同快照中混用 CSV 和非 CSV。

![磁碟區群組節點](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Volume_groups.png)

**圖 1：StorSimple Snapshot Manager 磁碟區群組節點** 

本教學課程說明如何使用 StorSimple Snapshot Manager：

* 檢視磁碟區群組的相關資訊
* 建立磁碟區群組
* 備份磁碟區群組
* 編輯磁碟區群組
* 刪除磁碟區群組

所有這些動作也可在 [動作]  窗格上進行。

## <a name="view-volume-groups"></a>檢視磁碟區群組
如果您按一下 [磁碟區群組] 節點，[結果] 窗格會顯示每個磁碟區群組的下列相關資訊，視您選擇的資料行而定。 您可以設定 [結果] 窗格中的資料行。 (以滑鼠右鍵按一下 [磁碟區] 節點，選取 [檢視]，然後選取 [新增/移除資料行]。)

| 結果資料行 | 說明 |
|:--- |:--- |
| Name |[名稱]  資料行包含磁碟區群組的名稱。 |
| Application |[應用程式]  資料行會顯示 Windows 主機上目前已安裝且正在執行的 VSS 寫入器數目。 |
| 已選取 |[已選取]  資料行會顯示磁碟區群組中所包含的磁碟區數目。 零 (0) 表示沒有任何應用程式與磁碟區群組中的磁碟區相關聯。 |
| 已匯入 |[已匯入] 資料行會顯示已匯入的磁碟區數目。 當設定為 **True** 時，此資料行會指出已從 Azure 入口網站匯入磁碟區群組，而不是在 StorSimple Snapshot Manager 中建立它。 |

> [!NOTE]
> StorSimple Snapshot Manager 磁碟區群組也會顯示在 Azure 入口網站的 [備份原則] 索引標籤上。
> 
> 

## <a name="create-a-volume-group"></a>建立磁碟區群組
請使用下列程序來建立磁碟區群組。

#### <a name="to-create-a-volume-group"></a>若要建立磁碟區群組
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，以滑鼠右鍵按一下 [磁碟區群組]，然後按一下 [建立磁碟區群組]。
   
    ![建立磁碟區群組](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Create_volume_group.png)
   
    [建立磁碟區群組]  對話方塊隨即出現。
   
    ![建立磁碟區群組對話方塊](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_CreateVolumeGroup_dialog.png)
3. 輸入以下資訊：
   
   1. 在 [名稱]  方塊中，輸入新的磁碟區群組的唯一名稱。
   2. 在 [應用程式]  方塊中，選取與您將新增至磁碟區群組之磁碟區相關聯的應用程式。
      
       [應用程式]  方塊僅會列出那些使用 StorSimple 磁碟區，並對它們啟用 VSS 寫入器的應用程式。 只在寫入器注意的所有磁碟區都是 StorSimple 磁碟區時，才會啟用 VSS 寫入器。 如果 [應用程式] 方塊是空的，則不會安裝任何使用 Azure StorSimple 磁碟區，並具有支援之 VSS 寫入器的應用程式。 (目前，Azure StorSimple 支援 Microsoft Exchange 和 SQL Server)。如需 VSS 寫入器的詳細資訊，請參閱[與 Windows 磁碟區陰影複製服務整合](storsimple-what-is-snapshot-manager.md#integration-with-windows-volume-shadow-copy-service)。
      
       如果選取應用程式，則會自動選取所有與其相關聯的磁碟區。 相反地，如果選取與特定應用程式相關聯的磁碟區，則會在 [應用程式]  方塊中自動選取該應用程式。 
   3. 在 [磁碟區]  方塊中，選取要新增到磁碟區群組的 StorSimple 磁碟區。 
      
      * 您可以包含具有單一或多個磁碟分割的磁碟區。 (多個磁碟分割磁碟區可以是具有多個磁碟分割的動態磁碟或基本磁碟)。包含多個磁碟分割的磁碟區會被視為單一單位。 因此，如果您只將其中一個磁碟分割新增到磁碟區群組，則所有其他磁碟分割會同時自動新增到該磁碟區群組。 在將多個磁碟分割磁碟區新增到磁碟區群組之後，多個磁碟分割磁碟區會繼續被視為單一單位。
      * 您可以建立空的磁碟區群組，方法是不將任何磁碟區指派給它們。 
      * 請勿在相同的磁碟區群組中混用叢集共用磁碟區 (CSV) 和非 CSV。 StorSimple Snapshot Manager 不支援在相同快照中混用 CSV 磁碟區和非 CSV 磁碟區。
4. 按一下 [確定]  儲存磁碟區群組。

## <a name="back-up-a-volume-group"></a>備份磁碟區群組
請使用下列程序來備份磁碟區群組。

#### <a name="to-back-up-a-volume-group"></a>若要備份磁碟區群組
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，展開 [磁碟區群組] 節點，以滑鼠右鍵按一下磁碟區群組名稱，然後按一下 [取得備份]。
   
    ![立即備份磁碟區群組](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Take_backup.png)
3. 在 [取得備份] 對話方塊中，選取 [本機快照] 或 [雲端快照]，然後按一下 [建立]。
   
    ![取得備份對話方塊](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_TakeBackup_dialog.png)
4. 若要確認備份執行中，請展開 [作業] 節點，然後按一下 [執行中]。 應該會列出備份。
5. 若要檢視已完成的快照集，請展開 [備份目錄] 節點，展開磁碟區群組名稱，然後按一下 [本機快照] 或 [雲端快照]。 如果順利完成，將會列出備份。

## <a name="edit-a-volume-group"></a>編輯磁碟區群組
請使用下列程序來編輯磁碟區群組。

#### <a name="to-edit-a-volume-group"></a>若要編輯磁碟區群組
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，展開 [磁碟區群組] 節點，以滑鼠右鍵按一下磁碟區群組名稱，然後按一下 [編輯]。
3. [建立磁碟區群組] 對話方塊隨即出現。 您可以變更 [名稱]、[應用程式] 和 [磁碟區] 項目。
4. 按一下 [確定]  以儲存變更。

## <a name="delete-a-volume-group"></a>刪除磁碟區群組
請使用下列程序來刪除磁碟區群組。 

> [!WARNING]
> 這也會刪除所有與磁碟區群組相關聯的備份。
> 
> 

#### <a name="to-delete-a-volume-group"></a>若要刪除磁碟區群組
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，展開 [磁碟區群組] 節點，以滑鼠右鍵按一下磁碟區群組名稱，然後按一下 [刪除]。
3. [刪除磁碟區群組] 對話方塊隨即出現。 在文字方塊中輸入 **Confirm**，然後按一下 [確定]。
   
    刪除的磁碟區群組會從 [結果]  窗格中的清單消失，而且所有與該磁碟區群組相關聯的備份都會從備份目錄中刪除。

## <a name="next-steps"></a>後續步驟
* 了解如何 [使用 StorSimple Snapshot Manager 來管理您的 StorSimple 解決方案](storsimple-snapshot-manager-admin.md)。
* 了解如何 [使用 StorSimple Snapshot Manager 建立和管理備份原則](storsimple-snapshot-manager-manage-backup-policies.md)。

