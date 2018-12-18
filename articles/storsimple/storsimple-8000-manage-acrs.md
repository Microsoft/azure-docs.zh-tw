---
title: 管理 StorSimple 中的存取控制記錄 | Microsoft Docs
description: 說明如何使用存取控制記錄 (ACR) 以判斷哪些主機可以連接至 StorSimple 裝置上的磁碟區。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/31/2017
ms.author: alkohli
ms.openlocfilehash: 9173e34f889ce1c082b20bb382cb6ca9a03dd797
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38597492"
---
# <a name="use-the-storsimple-manager-service-to-manage-access-control-records"></a>使用 StorSimple Manager 服務管理存取控制記錄

## <a name="overview"></a>概觀
存取控制記錄 (ACR) 可讓您指定哪些主機可連接至 StorSimple 裝置上的磁碟區。 ACR 設為特定的磁碟區，並且包含主機的 iSCSI 限定名稱 (IQN)。 當主機嘗試連線到磁碟區時，裝置會檢查與該磁碟區相關聯的 ACR 的 IQN 名稱，如果相符，則會建立連接。 [StorSimple 裝置管理員服務] 刀鋒視窗之 [設定] 區段中，存取控制記錄會顯示主機的所有存取控制記錄及對應的 IQN。

本教學課程將說明下列常見 ACR 相關工作：

* 加入存取控制記錄
* 編輯存取控制記錄
* 刪除存取控制記錄

> [!IMPORTANT]
> * 將 ACR 指派到磁碟區時，請注意磁碟區並未被多個非叢集主機並行存取，因為這可能會損毀磁碟區。
> * 從磁碟區刪除 ACR 時，請確定對應的主機未存取磁碟區，因為刪除作業可能會導致讀寫中斷。

## <a name="get-the-iqn"></a>取得 IQN

請執行下列步驟，以取得正在執行 Windows Server 2012 之 Windows 主機的 IQN。

[!INCLUDE [storsimple-get-iqn](../../includes/storsimple-get-iqn.md)]


## <a name="add-an-access-control-record"></a>加入存取控制記錄
您可以使用 [StorSimple 裝置管理員服務] 刀鋒視窗中的 [設定] 區段來新增 ACR。 一般而言，您會讓一個 ACR 與一個磁碟區產生關聯。

執行下列步驟以加入 ACR。

#### <a name="to-add-an-acr"></a>加入 ACR

1. 移至 StorSimple 裝置管理員服務，按兩下服務名稱，然後在 [設定] 區段內按一下 [存取控制記錄]。
2. 在 [存取控制記錄] 刀鋒視窗中，按一下 [+ 新增 ACR]。

    ![按一下 [新增 ACR]](./media/storsimple-8000-manage-acrs/createacr1.png)

3. 在 [新增 ACR] 刀鋒視窗中，執行下列步驟：

    1. 提供 ACR 的名稱。
    
    2. 在 [iSCSI 啟動器名稱 \(IQN)\] 下方，提供 Windows Server 主機的 IQN 名稱。

    3. 按一下 [新增] 以建立 ACR。

        ![按一下 [新增 ACR]](./media/storsimple-8000-manage-acrs/createacr2.png)

4.  新增的 ACR 會顯示在 ACR 的表格式清單中。

    ![按一下 [新增 ACR]](./media/storsimple-8000-manage-acrs/createacr5.png)


## <a name="edit-an-access-control-record"></a>編輯存取控制記錄
您可以使用 [StorSimple 裝置管理員服務] 刀鋒視窗中的 [設定] 區段來編輯 ACR。

> [!NOTE]
> 建議您只修改目前未在使用中的 ACR。 若要編輯與目前正在使用中的磁碟區相關聯的 ACR，您必須先讓磁碟區離線。

執行下列步驟以編輯 ACR。

#### <a name="to-edit-an-access-control-record"></a>編輯存取控制記錄
1.  移至 StorSimple 裝置管理員服務，按兩下服務名稱，然後在 [設定] 區段內按一下 [存取控制記錄]。

    ![移至存取控制記錄](./media/storsimple-8000-manage-acrs/createacr1.png)

2. 在存取控制記錄的表格式清單中，按一下並選取您想要修改的 ACR。

    ![編輯存取控制記錄](./media/storsimple-8000-manage-acrs/editacr1.png)

3. 在 [編輯存取控制記錄] 刀鋒視窗中，提供對應至其他主機的不同 IQN。

    ![編輯存取控制記錄](./media/storsimple-8000-manage-acrs/editacr2.png)

4. 按一下 [檔案] 。 系統提示您進行確認時，按一下 [是] 。 

    ![編輯存取控制記錄](./media/storsimple-8000-manage-acrs/editacr3.png)

5. ACR 更新時，您會收到通知。 表格式清單也會一併更新以反映變更。

   
## <a name="delete-an-access-control-record"></a>刪除存取控制記錄
您可以使用 [StorSimple 裝置管理員服務] 刀鋒視窗中的 [設定] 區段來刪除 ACR。

> [!NOTE]
> 您只能刪除目前未在使用中的 ACR。 若要刪除與目前正在使用中的磁碟區相關聯的 ACR，您必須先讓磁碟區離線。

執行下列步驟來刪除存取控制記錄。

#### <a name="to-delete-an-access-control-record"></a>刪除存取控制記錄
1.  移至 StorSimple 裝置管理員服務，按兩下服務名稱，然後在 [設定] 區段內按一下 [存取控制記錄]。

    ![移至存取控制記錄](./media/storsimple-8000-manage-acrs/createacr1.png)

2. 在存取控制記錄的表格式清單中，按一下並選取您想要刪除的 ACR。

    ![移至存取控制記錄](./media/storsimple-8000-manage-acrs/deleteacr1.png)

3. 以滑鼠右鍵按一下可叫用操作功能表，然後選取 [刪除]。

    ![移至存取控制記錄](./media/storsimple-8000-manage-acrs/deleteacr2.png)

4. 當出現提示確認時，請檢閱資訊，然後按一下 [刪除]。

    ![移至存取控制記錄](./media/storsimple-8000-manage-acrs/deleteacr3.png)

5. 當刪除作業完成時，您會收到通知。 表格式清單會更新以反映此刪除動作。

    ![移至存取控制記錄](./media/storsimple-8000-manage-acrs/deleteacr5.png)

## <a name="next-steps"></a>後續步驟
* 深入了解 [管理 StorSimple 磁碟區](storsimple-8000-manage-volumes-u2.md)。
* 深入了解 [使用 StorSimple Manager 服務來管理您的 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

