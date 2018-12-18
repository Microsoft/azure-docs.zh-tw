---
title: 管理 StorSimple Virtual Array 上的磁碟區 | Microsoft Docs
description: 描述 StorSimple 裝置管理員，並說明如何使用它來管理 StorSimple Virtual Array 上的磁碟區。
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: caa6a26b-b7ba-4a05-b092-1a79450225cf
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: manuaery
ms.openlocfilehash: a507bf1866952cb79fa6334fed80c88cd207cd0a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110206"
---
# <a name="use-storsimple-device-manager-service-to-manage-volumes-on-the-storsimple-virtual-array"></a>使用 StorSimple 裝置管理員服務管理 StorSimple Virtual Array 上的磁碟區

## <a name="overview"></a>概觀

本教學課程說明如何使用 StorSimple 裝置管理員服務，在 StorSimple Virtual Array 上建立和管理磁碟區。

StorSimple 裝置管理員服務是 Azure 入口網站中的一項擴充，可讓您從單一 Web 介面來管理 StorSimple 解決方案。 除了管理共用和磁碟區，您還可以使用 StorSimple 裝置管理員服務來檢視和管理裝置、檢視警示、以及檢視和管理備份原則和備份目錄。

## <a name="volume-types"></a>磁碟區類型

StorSimple 磁碟區可以是：

* **固定在本機**︰這些磁碟區中的資料永遠停留在陣列上，不會流向雲端。
* **階層式**：這些磁碟區中的資料可能流向雲端。 當您建立階層式磁碟區時，大約 10 % 的空間會佈建在本機層，而 90 % 的空間會佈建在雲端。 舉例來說，如果您佈建 1 TB 的磁碟區，當資料使用階層式磁碟區時，其中 100 GB 會位於本機的空間，900 GB 會位於雲端。 這也意味著，如果裝置的可用空間用盡，您就無法佈建階層式磁碟區 (因為本機層上需要的 10% 無法使用)。

### <a name="provisioned-capacity"></a>佈建的容量
請參閱下表以了解每個磁碟區類型的最大佈建容量。

| **限制識別碼**                                       | **限制**     |
|------------------------------------------------------------|---------------|
| 分層磁碟區的大小下限                            | 500 GB        |
| 分層磁碟區的大小上限                            | 5 TB          |
| 固定在本機的磁碟區的大小下限                    | 50 GB         |
| 固定在本機的磁碟區的大小上限                    | 500 GB        |

## <a name="the-volumes-blade"></a>[磁碟區] 刀鋒視窗
StorSimple 服務摘要刀鋒視窗的 [磁碟區] 功能表會顯示給定 StorSimple 陣列上的儲存體磁碟區清單，還可讓您管理它們。

![[磁碟區] 刀鋒視窗](./media/storsimple-virtual-array-manage-volumes/volumes-blade.png)

磁碟區是由一系列屬性所組成：

* **磁碟區名稱** – 必須是唯一且有助於識別磁碟區的描述性名稱。
* **狀態** – 可為連線或離線。 如果是離線的磁碟區，允許存取使用它的啟動器 (伺服器) 會看不到該磁碟區。
* **類型** – 指出磁碟區為「分層」(預設值) 或「固定在本機」。
* **容量** – 相較於啟動器 (伺服器) 可儲存的資料總量，指定已用的資料量。
* **備份** – 如果是 StorSimple Virtual Array，所有磁碟區會自動啟用備份。
* **連接的主機** – 指定允許存取此磁碟區的啟動器 (伺服器)。

![磁碟區詳細資料](./media/storsimple-virtual-array-manage-volumes/volume-details.png)

使用本教學課程中的指示以執行下列工作：

* 新增磁碟區
* 修改磁碟區
* 使磁碟區離線
* 刪除磁碟區

## <a name="add-a-volume"></a>新增磁碟區

1. 從 StorSimple 服務摘要刀鋒視窗中，從命令列按一下 [+ 新增磁碟區]。 這會開啟 [新增磁碟區] 刀鋒視窗。
   
    ![新增磁碟區](./media/storsimple-virtual-array-manage-volumes/add-volume.png)
2. 在 [新增磁碟區] 刀鋒視窗中，執行下列動作：
   
   * 在 [磁碟區名稱] 欄位中，輸入磁碟區的唯一名稱。 該名稱必須為包含 3 至 127 個字元的字串。
   * 在 [類型] 下拉式清單中，指定要建立 [階層式] 還是 [固定在本機] 磁碟區。 對於需要本機保證、低延遲及更高效能的工作負載，請選取 [固定在本機的磁碟區]。 針對所有其他資料，請選取 [階層式] 磁碟區。
   * 在 [容量] 欄位中，指定磁碟區大小。 階層式磁碟區必須介於 500 GB 和 5 TB 之間，而固定在本機的磁碟區必須介於 50 GB 和 500 GB 之間。
   * * 按一下 [連接的主機]，選取存取控制記錄 (ACR) (對應於您要連接至此磁碟區的 iSCSI 啟動器，然後按一下 [選取]。
3. 若要新增連接的主機，請按一下 [新增]，輸入主機的名稱及其 iSCSI 合格名稱 (IQN)，然後按一下 [新增]。
   
    ![新增磁碟區](./media/storsimple-virtual-array-manage-volumes/volume-add-acr.png)
4. 磁碟區設定完成之後，按一下 [建立]。 將會使用指定的設定來建立磁碟區，您會看到成功建立磁碟區的通知。 根據預設，磁碟區會啟用備份。
5. 若要確認已成功建立磁碟區，請移至 [磁碟區] 刀鋒視窗。 您應該會看到此處列出該磁碟區。
   
    ![磁碟區建立成功](./media/storsimple-virtual-array-manage-volumes/volume-success.png)

## <a name="modify-a-volume"></a>修改磁碟區

當您需要變更存取磁碟區的主機時，請修改磁碟區。 建立磁碟區之後，就無法修改磁碟區的其他屬性。

#### <a name="to-modify-a-volume"></a>若要修改磁碟區

1. 從 StorSimple 服務摘要刀鋒視窗的 [磁碟區] 設定中，選取您想要修改的磁碟區所在的虛擬陣列。
2. **選取** 磁碟區，然後按一下 [連接的主機]，以檢視目前連接的主機並修改為不同的伺服器。
   
    ![編輯磁碟區](./media/storsimple-virtual-array-manage-volumes/volume-edit-acr.png)
3. 按一下 [儲存] 命令列以儲存變更。 將會套用您指定的設定，您會看到通知。

## <a name="take-a-volume-offline"></a>使磁碟區離線

您想要修改或刪除磁碟區時，可能需要先使其離線。 當磁碟區離線時，即無法進行讀寫存取。 您必須使主機與裝置上的磁碟區同時離線。

#### <a name="to-take-a-volume-offline"></a>若要使磁碟區離線

1. 請確定有問題的磁碟區不在使用中後，再使其離線。
2. 先使主機上的磁碟區離線。 這樣做可以排除任何會造成磁碟區上資料損毀的潛在風險。 如需特定步驟，請參閱主機作業系統的指示。
3. 在主機上的磁碟區離線之後，請執行下列步驟讓陣列上的磁碟區離線：
   
   * 從 StorSimple 服務摘要刀鋒視窗的 [磁碟區] 設定中，選取您想要設為離線的磁碟區所在的虛擬陣列。
   * **選取** 磁碟區，從操作功能表中按一下 [...] \(或以滑鼠右鍵按一下此資料列)，然後選取 [離線]。
     
        ![讓磁碟區離線](./media/storsimple-virtual-array-manage-volumes/volume-offline.png)
   * 檢閱 [離線] 刀鋒視窗中的資訊，並確認您接受此作業。 按一下 [離線] 讓磁碟區離線。 您會看到作業進行中的通知。
   * 若要確認磁碟區已成功離線，請移至 [磁碟區] 刀鋒視窗。 您應該會看到磁碟區的狀態為離線。
     
       ![確認磁碟區離線](./media/storsimple-virtual-array-manage-volumes/volume-offline-confirm.png)

## <a name="delete-a-volume"></a>刪除磁碟區

> [!IMPORTANT]
> 只有當磁碟區離線時，您才能予以刪除。
> 
> 

請完成下列步驟來刪除磁碟區。

#### <a name="to-delete-a-volume"></a>若要刪除磁碟區

1. 從 StorSimple 服務摘要刀鋒視窗的 [磁碟區] 設定中，選取您想要刪除的磁碟區所在的虛擬陣列。
2. **選取** 磁碟區，從操作功能表中按一下 [...] \(或以滑鼠右鍵按一下此資料列)，然後選取 [刪除]。
   
    ![刪除磁碟區](./media/storsimple-virtual-array-manage-volumes/volume-delete.png)
3. 檢查您想要刪除之磁碟區的狀態。 如果您想要刪除的磁碟區未離線，請先使其離線，請依照 [使磁碟區離線](#take-a-volume-offline)中的步驟執行。
4. 當 [刪除]**刪除** 刀鋒視窗中提示您確認時，請接受確認並按一下 [刪除]。 現在將刪除磁碟區，[磁碟區] 刀鋒視窗會顯示虛擬陣列內更新的磁碟區清單。

## <a name="next-steps"></a>後續步驟

了解如何 [複製 StorSimple 磁碟區](storsimple-virtual-array-clone.md)。

