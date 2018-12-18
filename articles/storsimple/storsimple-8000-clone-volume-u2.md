---
title: 複製 StorSimple 8000 系列的磁碟區 | Microsoft Docs
description: 說明不同的複製類型以及使用方式，並說明如何使用備份組來複製 StorSimple 8000 系列裝置上的個別磁碟區。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 12/05/2017
ms.author: alkohli
ms.openlocfilehash: d3965c74d51996b3e4144130fad01fae9e265897
ms.sourcegitcommit: 5d3e99478a5f26e92d1e7f3cec6b0ff5fbd7cedf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2017
ms.locfileid: "26333197"
---
# <a name="use-the-storsimple-device-manager-service-in-azure-portal-to-clone-a-volume"></a>在 Azure 入口網站中，使用 StorSimple 裝置管理員服務來複製磁碟區

## <a name="overview"></a>概觀

本教學課程說明如何透過 [備份類別目錄] 刀鋒視窗使用備份組來複製個別磁碟區。 它也會說明「暫時性」與「永久」複製之間的差異。 本教學課程中的指導適用於執行 Update 3 或更新版本的所有 StorSimple 8000 系列裝置。

StorSimple 裝置管理員服務 [備份類別目錄]  刀鋒視窗會顯示在進行手動或自動備份時所建立的所有備份組。 然後您可以在備份組中選取要複製的磁碟區。

 ![備份組清單](./media/storsimple-8000-clone-volume-u2/bucatalog.png)

## <a name="considerations-for-cloning-a-volume"></a>複製磁碟區的考量

複製磁碟區之前，請考量下列資訊。

- 複製的磁碟區行為方式與一般磁碟區相同。 在磁碟區上可進行的任何作業皆可用於複製磁碟區。

- 複製的磁碟區上會自動停用監視和預設備份。 進行任何備份時您都必須設定複製磁碟區。

- 固定在本機的磁碟區會複製為分層磁碟區。 如果您需要將複製的磁碟區固定在本機，可以在複製作業成功完成之後，再將複製品轉換為固定在本機的磁碟區。 如需將分層磁碟區轉換為固定在本機的磁碟區的詳細資訊，請移至 [變更磁碟區類型](storsimple-8000-manage-volumes-u2.md#change-the-volume-type)。

- 如果您嘗試在複製之後 (當其仍為暫時性複製) 立即將分層磁碟區轉換為固定在本機的磁碟區，則轉換會失敗並出現下列錯誤訊息：

    `Unable to modify the usage type for volume {0}. This can happen if the volume being modified is a transient clone and hasn’t been made permanent. Take a cloud snapshot of this volume and then retry the modify operation.`

    只有在您複製到不同裝置時才會收到這個錯誤。 先將暫時性複製品轉換為永久複製品，即可成功將磁碟區轉換為固定在本機的磁碟區。 請拍下暫時性複製的雲端快照，以將其轉換為永久複製。

## <a name="create-a-clone-of-a-volume"></a>建立磁碟區複製

您可以使用本機或雲端快照，在相同的裝置、另一個裝置，甚或雲端設備上建立複製。

下列程序描述如何從備份類別目錄建立複製。  起始複製的替代方法是移至 [磁碟區]，選取磁碟區，然後按一下滑鼠右鍵以叫用操作功能表，然後選取 [複製]。

執行下列步驟，以從備份類別目錄中建立您的磁碟區的複製。

#### <a name="to-clone-a-volume"></a>若要複製磁碟區

1. 移至 StorSimple 裝置管理員服務，然後按一下 [備份類別目錄]。

2. 選取備份組，如下所示：
   
   1. 選取適當的裝置。
   2. 在下拉式清單中，針對要選取的備份選擇磁碟區或備份原則。
   3. 指定時間範圍。
   4. 按一下 [套用] 以執行此查詢。

    與選取的磁碟區或備份原則相關聯的備份應該會出現在備份組清單中。
   
    ![備份組清單](./media/storsimple-8000-clone-volume-u2/bucatalog.png)
     
3. 展開備份組以檢視相關聯的磁碟區，並選取備份組中的磁碟區。 以滑鼠右鍵按一下，然後從操作功能表中選取 [複製]。

   ![備份組清單](./media/storsimple-8000-clone-volume-u2/clonevol3b.png) 

3. 在 [複製] 刀鋒視窗中，執行下列步驟︰
   
    1. 識別目標裝置。 這是即將建立複製的位置。 您可以選擇相同的裝置，或指定另一個裝置。

      > [!NOTE]
      > 確定要複製的容量小於目標裝置中可用的容量。
       
    2. 為複製指定唯一的磁碟區名稱。 此名稱必須包含 3 到 127 個字元。
      
        > [!NOTE]
        > 即使您複製了本機釘選磁碟區，[將磁碟區複製為] 欄位仍然會**分層**。 您無法變更此設定，但如果您需要在本機釘選複製的磁碟區，您可以在成功建立複製之後，將複製轉換到本機釘選的磁碟區。 如需將分層磁碟區轉換為固定在本機的磁碟區的詳細資訊，請移至 [變更磁碟區類型](storsimple-8000-manage-volumes-u2.md#change-the-volume-type)。
          
    3. 在 [已連線的主機] 下，指定該複製的存取控制記錄 (ACR)。 您可以加入新的 ACR，或從現有清單中選擇。 ACR 會決定可存取此複製的主機。
      
        ![備份組清單](./media/storsimple-8000-clone-volume-u2/clonevol3a.png) 

    4. 按一下 [複製]，以完成作業。

4. 複製作業隨即起始，並在成功建立複製時通知您。 按一下作業通知，或移至 [作業] 刀鋒視窗以監控複製作業。

    ![備份組清單](./media/storsimple-8000-clone-volume-u2/clonevol5.png)

7. 複製作業完成後，移至您的裝置，然後按一下 [磁碟區]。 在磁碟區清單中，您應該會看見剛剛在相同磁碟區容器中建立的複製具有來源磁碟區。

    ![備份組清單](./media/storsimple-8000-clone-volume-u2/clonevol6.png)

以這種方式建立的複製就是暫時性複製。 如需複製類型的詳細資訊，請參閱 [暫時性與永久複製](#transient-vs-permanent-clones)。


## <a name="transient-vs-permanent-clones"></a>暫時性與永久複製
只有在您複製到另一個裝置時，才會建立暫時性複製。 您可以從備份組複製特定磁碟區到受 StorSimple 裝置管理員管理的不同裝置。 暫時性複製有原始磁碟區中資料的參考，並使用該資料在本機的目標裝置上讀取和寫入。

在建立暫時性複製的雲端快照之後，產生的複製就是「永久」 複製。 在此程序中，會在雲端建立資料複本，而複製此資料的時機則由資料大小和 Azure 的延遲來決定 (這是 Azure 對 Azure 複製)。 此程序可能需要數天至數週。 暫時性複製會成為永久複製，且沒有對所複製原始磁碟區資料的任何參考。

## <a name="scenarios-for-transient-and-permanent-clones"></a>暫時性複製與永久複製的案例
下列各節將說明可使用暫時性複製與永久複製的範例情況。

### <a name="item-level-recovery-with-a-transient-clone"></a>使用暫時性複製復原項目層級
您要復原過去一年的 Microsoft PowerPoint 簡報檔案。 IT 系統管理員會辨識該時間內的特定備份，然後篩選磁碟區。 系統管理員接著再複製磁碟區，找出您要找的檔案再提供給您。 在此案例中，使用的是暫時性複製。

### <a name="testing-in-the-production-environment-with-a-permanent-clone"></a>利用永久複製在實際執行環境中進行測試
您需要確認生產環境中的測試錯誤。 您在生產環境中建立磁碟區的複製，並採用這個複製的雲端快照來建立獨立的複製磁碟區。 在此案例中，使用的是永久複製。

## <a name="next-steps"></a>後續步驟
* 了解如何 [從備份組還原 StorSimple 磁碟區](storsimple-8000-restore-from-backup-set-u2.md)。
* 了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

