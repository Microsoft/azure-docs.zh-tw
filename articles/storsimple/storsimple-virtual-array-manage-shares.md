---
title: 管理 StorSimple Virtual Array 共用 | Microsoft Docs
description: 描述 StorSimple 裝置管理員，並說明如何使用它來管理 StorSimple Virtual Array 上的共用。
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: 0a799c83-fde5-4f3f-af0e-67535d1882b6
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: manuaery
ms.openlocfilehash: e5c62689de36baa175001f5f4f70d87568876ef0
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110586"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-shares-on-the-storsimple-virtual-array"></a>使用 StorSimple 裝置管理員服務管理 StorSimple Virtual Array 上的共用

## <a name="overview"></a>概觀

本教學課程說明如何使用 StorSimple 裝置管理員服務，在 StorSimple Virtual Array 上建立和管理共用。

StorSimple 裝置管理員服務是 Azure 入口網站中的一項擴充，可讓您從單一 Web 介面來管理 StorSimple 解決方案。 除了管理共用和磁碟區，您還可以使用 StorSimple 裝置管理員服務來檢視和管理裝置、檢視警示、管理備份原則，以及管理備份目錄。

## <a name="share-types"></a>共用類型

StorSimple 共用可以是︰

* **固定在本機**︰這些共用中的資料永遠停留在陣列上，不會流向雲端。
* **階層式**：這些磁碟區中的資料可能流向雲端。 當您建立階層式共用時，大約 10% 的空間會佈建在本機層，而 90% 的空間會佈建在雲端。 舉例來說，如果您佈建 1 TB 的共用，當資料分層時，有 100 GB 會位於本機空間，900 GB 會用於雲端。 這也意味著，如果裝置的可用空間用盡，您就無法佈建階層式共用 (因為本機層上需要的 10% 無法使用)。

### <a name="provisioned-capacity"></a>佈建的容量

請參閱下表以了解每個共用類型的最大佈建容量。

| **限制識別碼** | **限制** |
| --- | --- |
| 分層共用的大小下限 |500 GB |
| 分層共用的大小上限 |20 TB |
| 固定在本機的共用的大小下限 |50 GB |
| 固定在本機的共用的大小上限 |2 TB |

## <a name="the-shares-blade"></a>[共用] 刀鋒視窗

StorSimple 服務摘要刀鋒視窗的 [共用] 功能表會顯示給定 StorSimple 陣列上的儲存體共用清單，還可讓您管理它們。

![[共用] 刀鋒視窗](./media/storsimple-virtual-array-manage-shares/shares-blade.png)

共用包含一系列屬性：

* **共用名稱** – 必須是唯一且有助於識別共用的描述性名稱。
* **狀態** – 可為連線或離線。 如果共用已離線，共用的使用者將無法存取它。
* **類型** – 指出共用為**階層式** (預設值) 或**固定在本機**。
* **容量** – 相較於共用上可儲存的資料總量，指定已用的資料量。
* **描述** – 有助於描述共用的選擇性設定。
* **權限** -可透過 Windows 檔案總管來管理的共用 NTFS 權限。
* **備份** – 如果是 StorSimple Virtual Array，所有共用會自動啟用備份。

![共用的詳細資料](./media/storsimple-virtual-array-manage-shares/share-details.png)

使用本教學課程中的指示以執行下列工作：

* 新增共用
* 修改共用
* 讓共用離線
* 刪除共用

## <a name="add-a-share"></a>新增共用

1. 從 StorSimple 服務摘要刀鋒視窗中，從命令列按一下 [+ 新增共用]。 這會開啟 [新增共用] 刀鋒視窗。

    ![新增共用](./media/storsimple-virtual-array-manage-shares/add-share.png)

2. 在 [新增共用] 刀鋒視窗中，執行下列動作：
   
    1. 在 [共用名稱] 欄位中，輸入共用的唯一名稱。 該名稱必須為包含 3 至 127 個字元的字串。

    2. 共用的選擇性 [說明]。 說明將可協助識別共用的擁有者。

    3. 在 [類型] 下拉式清單中，指定要建立 [階層式] 還是 [固定在本機] 共用。 對於需要本機保證、低延遲，以及高效能的工作負載，請選取 [固定在本機的共用] 。 對於所有其他資料，請選取 [階層式] 共用。

    4. 在 [容量] 欄位中，指定共用大小。 階層式共用必須介於 500 GB 和 20 TB 之間，而固定在本機的共用必須介於 50 GB 和 2 TB 之間。

    5. 在 [將預設完整權限設為] 欄位中，指派權限給存取此共用的使用者或群組。 請以下列格式指定使用者或使用者群組的名稱：_john@contoso.com_。 我們建議您利用使用者群組 (而非單一使用者)，來授予可存取這些共用的系統管理員權限。 當您在此指派權限之後，就可以使用 [檔案總管] 來修改這些權限。
3. 共用設定完成之後，按一下 [建立]。 將會使用指定的設定來建立共用，您會看到通知。 根據預設，共用會啟用備份。
4. 若要確認已成功建立共用，請移至 [共用] 刀鋒視窗。 您應該會看列出共用。
   
    ![共用建立成功](./media/storsimple-virtual-array-manage-shares/share-success.png)

## <a name="modify-a-share"></a>修改共用

當您需要變更共用的描述時，請修改共用。 建立共用之後，就無法修改其他共用屬性。

#### <a name="to-modify-a-share"></a>若要修改共用

1. 從 StorSimple 服務摘要刀鋒視窗的 [共用] 設定中，選取您想要修改的共用所在的虛擬陣列。
2. **選取**共用以檢視目前的描述並修改。
3. 按一下 [儲存] 命令列以儲存變更。 將會套用您指定的設定，您會看到通知。
   
    ![ 編輯共用](./media/storsimple-virtual-array-manage-shares/share-edit.png)

## <a name="take-a-share-offline"></a>讓共用離線

當您打算修改或刪除共用時，可能需要先讓共用離線。 當共用已離線時，即無法進行讀寫存取。 您必須讓主機和裝置上的共用都離線。

#### <a name="to-take-a-share-offline"></a>若要讓共用離線

1. 請確定此共用不在使用中，再讓它離線。
2. 執行下列步驟，讓陣列上的共用離線︰
   
    1. 從 StorSimple 服務摘要刀鋒視窗的 [共用] 設定中，選取您想要設為離線的共用所在的虛擬陣列。

    2. **選取** 共用，從操作功能表中按一下 [...] \(或以滑鼠右鍵按一下此資料列)，然後選取 [離線]。
     
        ![離線共用](./media/storsimple-virtual-array-manage-shares/shares-offline.png)

    3. 檢閱 [離線] 刀鋒視窗中的資訊，並確認您接受此作業。 按一下 [離線] 讓共用離線。 您會看到作業進行中的通知。

    4. 若要確認共用已成功離線，請移至 [共用] 刀鋒視窗。 您應該會看到共用的狀態為離線。

## <a name="delete-a-share"></a>刪除共用

> [!IMPORTANT]
> 只有已離線的共用才能刪除。


請完成下列步驟來刪除共用。

#### <a name="to-delete-a-share"></a>若要刪除共用

1. 從 StorSimple 服務摘要刀鋒視窗的 [共用] 設定中，選取您想要刪除的共用所在的虛擬陣列。
2. **選取** 共用，從操作功能表中按一下 [...] \(或以滑鼠右鍵按一下此資料列)，然後選取 [刪除]。
   
    ![刪除共用](./media/storsimple-virtual-array-manage-shares/share-delete.png)
3. 檢查您想要刪除之共用的狀態。 如果您想要刪除的共用未離線，請先讓它離線。 請遵循[讓共用離線](#take-a-share-offline)中的步驟進行。
4. 當 [刪除]**刪除** 刀鋒視窗中提示您確認時，請接受確認並按一下 [刪除]。 現在將刪除共用，[共用] 刀鋒視窗會顯示虛擬陣列內更新的共用清單。

## <a name="next-steps"></a>後續步驟
了解如何[複製 StorSimple 共用](storsimple-virtual-array-clone.md)。

