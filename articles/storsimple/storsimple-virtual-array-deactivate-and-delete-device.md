---
title: 停用和刪除 Microsoft Azure StorSimple Virtual Array | Microsoft Docs
description: 描述如何停用然後刪除 StorSimple 裝置，將其從服務中移除。
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: a929f5bc-03e2-4b01-b925-973db236f19f
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: alkohli
ms.openlocfilehash: 8dea36f92b034f8c6cdb6875634848d37f4c6606
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109876"
---
# <a name="deactivate-and-delete-a-storsimple-virtual-array"></a>停用及刪除 StorSimple Virtual Array

## <a name="overview"></a>概觀

當您停用 StorSimple Virtual Array 時，您會切斷裝置與相對應 StorSimple 裝置管理員服務之間的連接。 本教學課程說明如何：

* 停用裝置 
* 刪除或停用裝置

本文中的資訊僅適用於 StorSimple Virtual Array。 如需 8000 系列的相關資訊，請參閱如何[停用或刪除裝置](storsimple-deactivate-and-delete-device.md)。

## <a name="when-to-deactivate"></a>何時停用？

停用是永久性的作業，而且無法復原。 您無法將已停用的裝置重新向 StorSimple 裝置管理員服務註冊。 在下列情況下，您可能需要停用和刪除 StorSimple Virtual Array：

* **規劃的容錯移轉**：您的裝置已上線，而且您打算容錯移轉裝置。 如果您打算升級至更大的裝置，您可能需要容錯移轉您的裝置。 在資料擁有權轉移及容錯移轉完成之後，就會自動刪除來源裝置。
* **未規劃的容錯移轉**：您的裝置已離線，而且您需要容錯移轉裝置。 在資料中心停止運作且主要裝置已關閉的災害期間，可能會發生此情況。 您計劃將裝置容錯移轉至次要裝置。 在資料擁有權轉移及容錯移轉完成之後，就會自動刪除來源裝置。
* **解除委任**：您想要解除委任裝置。 這需要您先停用裝置，然後刪除它。 當您停用裝置時，您將無法再存取儲存在本機的任何資料。 您只能存取和復原雲端中儲存的資料。 如果您計劃在停用之後保留裝置資料，則您應該在停用裝置之前擷取所有資料的雲端快照集。 此雲端快照可讓您在稍後的階段中復原所有資料。

## <a name="deactivate-a-device"></a>停用裝置

若要停用裝置，請執行下列步驟。

#### <a name="to-deactivate-the-device"></a>停用裝置

1. 在您的服務中，移至 [管理] > [裝置]。 在 [裝置] 刀鋒視窗中，按一下並選取您想要停用的裝置。
   
    ![選取要停用的裝置](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete7.png)
2. 在 [裝置儀表板] 刀鋒視窗中，按一下 [...更多]，然後從清單中選取 [停用]。
   
    ![按一下停用](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete8.png)
3. 在 [停用] 刀鋒視窗中，輸入裝置名稱，然後按一下 [停用]。 
   
    ![確認停用](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete1.png)
   
    停用程序開始，需要幾分鐘才能完成。
   
    ![停用進行中](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete2.png)
4. 停用之後會重新整理裝置清單。
   
    ![停用完成](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete3.png)
   
    您現在可以刪除此裝置。

## <a name="delete-the-device"></a>刪除裝置

裝置必須先停用才能刪除。 刪除裝置會將其從與服務連接的裝置清單移除。 服務將不再管理已刪除的裝置。 但是，與裝置相關聯的資料仍會在雲端中。 所以這項資料會累算費用。

若要刪除裝置，請執行下列步驟。

#### <a name="to-delete-the-device"></a>刪除裝置

1. 在 StorSimple 裝置管理員中，然移至 [管理] > [裝置]。 在 [裝置] 刀鋒視窗中，選取您想要刪除的已停用裝置。
2. 在 [裝置儀表板] 刀鋒視窗中，按一下 [...更多]，然後按一下 [刪除]。
   
   ![選取要刪除的裝置](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete4.png)
3. 在 [刪除] 刀鋒視窗中，輸入裝置的名稱來確認刪除，然後按一下 [刪除]。 刪除裝置並不會刪除與裝置相關聯的雲端資料。 
   
   ![Confirm delete](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete5.png) 
4. 刪除程序開始，需要幾分鐘才能完成。
   
   ![刪除進行中](./media/storsimple-virtual-array-deactivate-and-delete-device/deactivate-delete6.png)
   
    刪除裝置後，您可以檢視更新的裝置清單。

## <a name="next-steps"></a>後續步驟

* 如需如何容錯移轉的相關資訊，請參閱 [StorSimple Virtual Array 的容錯移轉和災害復原](storsimple-virtual-array-failover-dr.md)。

* 若要深入了解如何使用 StorSimple 裝置管理員服務，請參閱[使用 StorSimple 裝置管理員服務管理 StorSimple Virtual Array](storsimple-virtual-array-manager-service-administration.md)。 

