---
title: 管理您的 StorSimple 備份類別目錄 | Microsoft Docs
description: 說明如何使用 StorSimple 裝置管理員服務的 [備份類別目錄] 頁面列出、選取和刪除備份組。
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
ms.date: 06/29/2017
ms.author: alkohli
ms.openlocfilehash: ac2577c6cd350d6d437d55e61ec73d954cb24893
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108376"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-your-backup-catalog"></a>使用 StorSimple 裝置管理員服務管理備份類別目錄
## <a name="overview"></a>概觀
StorSimple 裝置管理員服務的 [備份類別目錄]  刀鋒視窗會顯示在進行手動或排程備份時所建立的所有備份組。 您可以使用此頁面來列出備份原則或磁碟區的所有備份、選取或刪除備份，或是使用備份來還原或複製磁碟區。

本教學課程說明如何列出、選取和刪除備份組。 若要了解如何透過備份還原裝置內容，請移至 [從備份組還原裝置](storsimple-8000-restore-from-backup-set-u2.md)。 若要了解如何複製磁碟區，請移至 [複製 StorSimple 磁碟區](storsimple-8000-clone-volume-u2.md)。

![備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog.png) 

[備份類別目錄] 刀鋒視窗提供查詢，可讓您縮小備份組選取範圍。 您可以篩選根據下列參數擷取的備份組：

* **裝置** - 建立備份組所在的裝置。
* **備份原則或磁碟區** – 與此備份組相關聯的備份原則或磁碟區。
* **起迄** – 建立備份組的日期和時間範圍。

接著會根據下列屬性，將篩選的備份組列表顯示：

* **名稱** - 與備份組相關聯的備份原則或磁碟區的名稱。
* **大小** - 備份組的實際大小。
* **建立日期** - 建立備份的日期和時間。 
* **類型** - 備份組可以是本機快照集或雲端快照集。 本機快照是本機儲存於裝置上的所有磁碟區資料備份，而雲端快照是指位於雲端的磁碟區資料備份。 本機快照可提供更快速的存取，而雲端快照是選擇來進行資料復原。
* **啟動者** – 備份可由排程自動啟動，或由使用者手動啟動。 您可以使用備份原則排程備份。 或者，可以使用 [進行備份]  選項來進行手動備份。

## <a name="list-backup-sets-for-a-backup-policy"></a>列出備份原則的備份組
完成下列步驟，以列出備份原則的所有備份。

#### <a name="to-list-backup-sets"></a>若要列出備份組
1. 移至 StorSimple 裝置管理員服務，然後按一下 [備份類別目錄]。

2. 如下方所示，篩選選取項目：
   
   1. 指定時間範圍。
   2. 選取適當的裝置。
   3. 依 [備份原則] 篩選，以檢視對應的備份。
   3. 從 [備份原則] 下拉式清單中，選擇 [全部] 以檢視所選取裝置上的所有備份。
   4. 按一下 [套用] 以執行此查詢。
      
      與選取的備份原則相關聯的備份應該會出現在備份組清單中。

      ![移至備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog1.png)

## <a name="select-a-backup-set"></a>選取備份組
完成下列步驟，選取磁碟區的備份組或備份原則。

#### <a name="to-select-a-backup-set"></a>若要選取備份組
1. 移至 StorSimple 裝置管理員服務，然後按一下 [備份類別目錄]。
2. 如下方所示，篩選選取項目：
   
   1. 指定時間範圍。 
   2. 選取適當的裝置。 
   3. 針對要選取的備份，依磁碟區或備份原則篩選。
   4. 按一下 [套用] 以執行此查詢。
      
      與選取的磁碟區或備份原則相關聯的備份應該會出現在備份組清單中。

      ![移至備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog1.png)

3. 選取並展開備份組。 您現在可以看到根據其包含之磁碟區細分的備份組。 [還原] 與 [刪除] 選項可透過備份組的操作功能表 (按一下滑鼠右鍵) 取得。 您可以在選取的備份組上執行以上任一動作。

    ![移至備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog2.png)

## <a name="delete-a-backup-set"></a>刪除備份組
不再需要保留與備份相關的資料時，請將備份刪除。 執行下列步驟以刪除備份組。

#### <a name="to-delete-a-backup-set"></a>若要刪除備份組
 移至 StorSimple 裝置管理員服務，然後按一下 [備份類別目錄]。
2. 如下方所示，篩選選取項目：
   
   1. 指定時間範圍。 
   2. 選取適當的裝置。 
   3. 針對要選取的備份，依磁碟區或備份原則篩選。
   4. 按一下 [套用] 以執行此查詢。
      
      與選取的磁碟區或備份原則相關聯的備份應該會出現在備份組清單中。

      ![移至備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog1.png)

3. 選取並展開備份組。 您現在可以看到根據其包含之磁碟區細分的備份組。 [還原] 與 [刪除] 選項可透過備份組的操作功能表 (按一下滑鼠右鍵) 取得。 以滑鼠右鍵按一下選取的備份組，然後從操作功能表中選取 [刪除]。

    ![移至備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog3.png)

4. 當系統提示確認時，請檢閱顯示的資訊，然後按一下 [刪除]。 選取的備份會永久刪除。

    ![移至備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog4.png)  

5. 刪除進行中和順利完成時，您會收到通知。 刪除完成之後，重新整理此頁面上的查詢。 已刪除的備份組將不會再顯示於備份組清單中。

    ![移至備份類別目錄](./media/storsimple-8000-manage-backup-catalog/bucatalog7.png)

## <a name="next-steps"></a>後續步驟
* 了解如何使用備份類別目錄以 [從備份組還原裝置](storsimple-8000-restore-from-backup-set-u2.md)。
* 了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

