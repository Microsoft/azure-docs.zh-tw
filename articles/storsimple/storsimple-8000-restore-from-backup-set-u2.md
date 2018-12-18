---
title: 在 StorSimple 8000 系列上從備份還原磁碟區 | Microsoft Docs
description: 說明如何使用 StorSimple 裝置管理員服務的 [備份類別目錄]，從備份組還原 StorSimple 磁碟區。
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
ms.date: 05/23/2017
ms.author: alkohli
ms.openlocfilehash: aff0710ead4f76bb80c38e2d88fe9cd3ce6a7b48
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109256"
---
# <a name="restore-a-storsimple-volume-from-a-backup-set"></a>從備份組還原 StorSimple 磁碟區

## <a name="overview"></a>概觀

本教學課程說明在 StorSimple 8000 系列裝置上使用現有的備份組執行的還原作業。 使用 [備份類別目錄] 刀鋒視窗可從本機或雲端備份還原磁碟區。 [備份類別目錄] 刀鋒視窗顯示在產生手動或自動備份時建立的所有備份組。 當資料在背景下載時，從備份組進行的還原作業會立即讓磁碟區連線。

啟動還原的一個替代方法是移至 [裝置] > [您的裝置] > [磁碟區]。 在 [磁碟區] 刀鋒視窗中，選取磁碟區，以滑鼠右鍵按一下以叫用操作功能表，然後選取 [還原]。

## <a name="before-you-restore"></a>還原之前

在開始還原之前，請先檢閱下列注意事項：

* **必須使磁碟區離線** – 起始還原作業之前，同時讓主機和裝置上的磁碟區離線。 雖然還原作業會自動使裝置上的磁碟區連線，但您必須手動讓主機上的裝置連線。 在裝置上的磁碟區連線之後，您就可以立即讓主機上的磁碟區連線。 (不需要等到還原作業完成。)如需程序，請前往[使磁碟區離線](storsimple-8000-manage-volumes-u2.md#take-a-volume-offline)。

* **還原後的磁碟區類型** – 已刪除的磁碟區會根據快照集中的類型還原；亦即，固定在本機的磁碟機會還原為固定在本機的磁碟機，而分層磁碟機會還原為分層磁碟機。

    針對現有的磁碟區，磁碟機目前的使用類型會覆寫快照集中所還原的類型。 例如，如果您從原本磁碟區類型為分層，而現在磁碟區類型為固定在本機 (因為已執行的轉換作業) 時所採取的快照集還原磁碟區，則磁碟區會還原成固定在本機的磁碟區。 同樣地，如果現有的固定在本機的磁碟區已擴充，且後續從過去磁碟區仍較小的舊快照集還原，還原的磁碟區將會保留目前擴充的大小。

    當磁碟區正在還原時，您無法從分層磁碟區轉換成固定在本機的磁碟區，或從固定在本機的磁碟區轉換成分層磁碟區。 請等候還原作業完成，您就能將磁碟區轉換為其他類型。 如需轉換磁碟區的詳細資訊，請移至 [變更磁碟區類型](storsimple-8000-manage-volumes-u2.md#change-the-volume-type)。 

* **磁碟區大小會反映在還原的磁碟區中** - 如果您要還原已刪除的固定在本機的磁碟區，這是很重要的考量 (因為要完整佈建固定在本機的磁碟區)。 請確定您有足夠的空間，然後再嘗試還原之前已刪除的固定在本機的磁碟區。

* **正在還原時，您無法擴充磁碟區** – 等候還原作業完成，才能嘗試擴充磁碟區。 如需有關擴充磁碟區的資訊，請移至 [修改磁碟區](storsimple-8000-manage-volumes-u2.md#modify-a-volume)。

* **還原本機磁碟區時，您可以執行備份** – 如需程序，請移至[使用 StorSimple 裝置管理員服務來管理備份原則](storsimple-8000-manage-backup-policies-u2.md)。

* **可以取消還原作業** – 如果您取消還原工作，磁碟區將會回復到起始還原作業之前的狀態。 如需程序，請移至 [取消工作](storsimple-8000-manage-jobs-u2.md#cancel-a-job)。

## <a name="how-does-restore-work"></a>還原的運作方式

如果是執行 Update 4 或更新版本的裝置，就會實作熱度圖式還原。 當存取資料的主機要求到達裝置時，即會追蹤這些要求並建立熱度圖。 高要求率會產生具有更高熱度的資料區塊，而較低的要求率會轉譯為熱度較低的區塊。 您必須存取資料至少兩次，才能將其標示為「熱」。 已修改的檔案也會標示為「熱」。 一旦您起始還原之後，接著會根據熱度圖進行資料的主動式水化。 如果版本早於 Update 4，就只會在還原期間根據存取下載資料。

下列注意事項適用於熱度圖式還原：

* 熱度圖追蹤只適用於階層式磁碟區，不支援固定在本機的磁碟區。

* 將磁碟區複製到另一個裝置時，不支援熱度圖式還原。 

* 如果有就地還原且裝置上目前有要還原之磁碟區的本機快照集，則我們不會解除凍結 (因為資料已經可在本機使用)。 

* 根據預設，當您還原時，會起始解除凍結作業，以根據熱度圖主動解除凍結資料。 

在 Update 4 中，Windows PowerShell Cmdlet 可用來查詢執行中的解除凍結作業、取消解除凍結作業，以及取得解除凍結作業的狀態。

* `Get-HcsRehydrationJob` -此 Cmdlet 會取得解除凍結作業的狀態。 單一解除凍結作業會針對某一個磁碟區來觸發。

* `Set-HcsRehydrationJob` - 此 Cmdlet 可讓您在解除凍結正在進行時暫停、停止、繼續執行解除凍結作業。

如需解除凍結 Cmdlet 的詳細資訊，請移至[適用於 StorSimple 的 Windows PowerShell Cmdlet 參考](https://technet.microsoft.com/library/dn688168.aspx)。

透過自動解除凍結功能，通常可預期較高的暫時性讀取效能。 改進的實際範圍取決於各種因素，例如存取模式、資料變換及資料類型。 

若要取消解除凍結作業，您可以使用 PowerShell Cmdlet。 如果您希望針對所有未來的還原作業永久停用解除凍結作業，請[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。

## <a name="how-to-use-the-backup-catalog"></a>如何使用備份類別目錄

[備份類別目錄] 刀鋒視窗提供查詢，可協助您縮小備份組選取範圍。 您可以篩選根據下列參數擷取的備份組：

* **時間範圍** – 建立備份組的日期和時間範圍。
* **裝置** - 建立備份組所在的裝置。
* **備份原則**或**磁碟區** – 與此備份組相關聯的備份原則或磁碟區。

接著會根據下列屬性，將篩選的備份組列表顯示：

* **名稱** - 與備份組相關聯的備份原則或磁碟區的名稱。
* **類型** - 備份組可以是本機快照集或雲端快照集。 本機快照是本機儲存於裝置上的所有磁碟區資料備份，而雲端快照是指位於雲端的磁碟區資料備份。 本機快照可提供更快速的存取，而雲端快照是選擇來進行資料復原。
* **大小** - 備份組的實際大小。
* **建立日期** - 建立備份的日期和時間。 
* **磁碟區** - 與備份組相關聯的磁碟區數目。
* **起始方式** - 備份可根據排程自動起始，或由使用者手動起始。 (您可以使用備份原則來排程備份。 或者，可以使用 [進行備份] 選項來進行互動式或隨選備份。)

## <a name="how-to-restore-your-storsimple-volume-from-a-backup"></a>如何從備份還原您的 StorSimple 磁碟區

您可以使用 [備份類別目錄] 刀鋒視窗，從特定的備份還原 StorSimple 磁碟區。 不過，請記住，還原磁碟區會將磁碟區的狀態還原為其在取得備份時的狀態。 在備份作業之後新增的所有資料都將遺失。

> [!WARNING]
> 從備份還原將從備份取代現有的磁碟區。 這可能會造成在取得備份之後寫入的所有資料遺失。


### <a name="to-restore-your-volume"></a>還原您的磁碟區
1. 移至 StorSimple 裝置管理員服務，然後按一下 [備份類別目錄]。

2. 選取備份組，如下所示：
   
   1. 指定時間範圍。
   2. 選取適當的裝置。
   3. 在下拉式清單中，針對要選取的備份選擇磁碟區或備份原則。
   4. 按一下 [套用] 以執行此查詢。

    與選取的磁碟區或備份原則相關聯的備份應該會出現在備份組清單中。
   
    ![備份組清單](./media/storsimple-8000-restore-from-backup-set-u2/bucatalog.png)     
     
3. 展開備份組以檢視相關聯的磁碟區。 您必須先在主機和裝置上將這些磁碟區離線，才能還原它們。 存取您裝置上 [磁碟區] 刀鋒視窗上的磁碟區，然後依照[讓磁碟區離線](storsimple-8000-manage-volumes-u2.md#take-a-volume-offline)中的步驟來讓它們離線。
   
   > [!IMPORTANT]
   > 確定您已先讓主機上的磁碟區離線，然後再讓裝置上的磁碟區離線。 如果您並未讓主機上的磁碟區離線，可能會導致資料損毀。
   
4. 瀏覽回到 [備份類別目錄]  索引標籤，並選取備份組。 以滑鼠右鍵按一下，然後從操作功能表中選取 [還原]。

    ![備份組清單](./media/storsimple-8000-restore-from-backup-set-u2/restorebu1.png)

5. 系統將提示您進行確認。 檢閱還原資訊，然後選取 [確認] 核取方塊。
   
    ![確認電子郵件](./media/storsimple-8000-restore-from-backup-set-u2/restorebu2.png)

7.  按一下 [還原]。 這將會起始還原工作，而您可以存取 [工作] 頁面來檢視。

    ![確認電子郵件](./media/storsimple-8000-restore-from-backup-set-u2/restorebu5.png)

8. 還原完成之後，請確認磁碟區的內容已由備份的磁碟區所取代。


## <a name="if-the-restore-fails"></a>如果還原失敗

如果還原作業因為任何原因而失敗，您會收到警示。 如果發生這種情況，請重新整理備份清單，以確認備份仍然有效。 如果備份有效，且您是從雲端還原，則連線問題可能會造成問題。

若要完成還原作業，請使主機上的磁碟區離線，然後重試還原作業。 請注意，在還原程序期間所執行的磁碟區資料修改將會遺失。

## <a name="next-steps"></a>後續步驟
* 了解如何 [管理 StorSimple 磁碟區](storsimple-8000-manage-volumes-u2.md)。
* 了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

