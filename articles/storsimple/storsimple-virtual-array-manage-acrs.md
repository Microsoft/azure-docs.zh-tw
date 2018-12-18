---
title: 管理 StorSimple Virtual Array 的存取控制記錄 | Microsoft Docs
description: 說明如何使用存取控制記錄 (ACR) 以判斷哪些主機可以連接至 StorSimple Virtual Array 上的磁碟區。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: e154bb4f-faab-4d92-a593-900c3ddc9595
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2ce65aa4efba735305208f7a6d761bc2814d1b8f
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38718881"
---
# <a name="use-storsimple-device-manager-to-manage-access-control-records-for-storsimple-virtual-array"></a>使用 StorSimple 裝置管理員來管理 StorSimple Virtual Array 的存取控制記錄

## <a name="overview"></a>概觀

存取控制記錄 (ACR) 可讓您指定哪些主機可連接至 StorSimple Virtual Array (也稱為 StorSimple 內部部署虛擬裝置) 上的磁碟區。 ACR 設為特定的磁碟區，並且包含主機的 iSCSI 限定名稱 (IQN)。 當主機嘗試連線到磁碟區時，裝置會檢查與該磁碟區相關聯的 ACR 的 IQN 名稱，如果相符，則會建立連接。 [設定] 區段內的 [存取控制記錄] 刀鋒視窗會顯示主機的存取控制記錄及對應的 IQN。

![管理存取控制記錄](./media/storsimple-virtual-array-manage-acrs/ova-manage-acrs.png)

本教學課程將說明下列常見 ACR 相關工作：

* 取得 IQN
* 加入存取控制記錄
* 編輯存取控制記錄
* 刪除存取控制記錄

> [!IMPORTANT]
> 
> * 將 ACR 指派到磁碟區時，請注意磁碟區並未被多個非叢集主機並行存取，因為這可能會損毀磁碟區。
> * 從磁碟區刪除 ACR 時，請確定對應的主機未存取磁碟區，因為刪除作業可能會導致讀寫中斷。


## <a name="get-the-iqn"></a>取得 IQN

請執行下列步驟，以取得正在執行 Windows Server 2012 之 Windows 主機的 IQN。

[!INCLUDE [storsimple-get-iqn](../../includes/storsimple-get-iqn.md)]

## <a name="add-an-acr"></a>加入 ACR

在 StorSimple 裝置管理員服務的 [設定] 區段內，您可以使用 [存取控制記錄] 刀鋒視窗來新增 ACR。 一般而言，您會讓一個 ACR 與一個磁碟區產生關聯。

如需讓 ACR 與磁碟區產生關聯的相關資訊，請參閱[新增磁碟區](storsimple-virtual-array-deploy3-iscsi-setup.md#step-3-add-a-volume)。

> [!IMPORTANT]
> 將 ACR 指派到磁碟區時，請注意磁碟區並未被多個非叢集主機並行存取，因為這可能會損毀磁碟區。


執行下列步驟以加入 ACR。

#### <a name="to-add-an-acr"></a>加入 ACR

1. 在服務登陸頁面上，選取您的服務，按兩下服務名稱，然後在 [設定] 區段內按一下 [存取控制記錄]。
2. 在 [存取控制記錄] 刀鋒視窗中，按一下 [新增]。
3. 在 [新增 ACR] 刀鋒視窗中，執行下列動作：
   
    1. 提供 ACR 的 [名稱]  。
    
    2. 在 [iSCSI 啟動器名稱] 下方，提供 Windows 主機的 IQN 名稱。 若要取得 Windows Server 主機的 IQN，請執行下列動作：
   
    3. 在 Windows 主機上啟動 Microsoft iSCSI 啟動器。 在 [iSCSI 啟動器屬性] 視窗的 [設定] 索引標籤上，選取並複製 [啟動器名稱] 欄位的字串。
    將此字串貼在 [新增 ACR] 刀鋒視窗的 [IQN] 欄位中。
   
    6. 按一下 [新增] 來新增 ACR。  
   
        ![新增存取控制記錄](./media/storsimple-virtual-array-manage-acrs/ova-add-acrs.png)
4. 表格式清單會更新以反映此新增動作。

## <a name="edit-an-acr"></a>編輯 ACR

在 Azure 入口網站中，在裝置管理員服務的 [設定] 區段內，您可以使用 [存取控制記錄] 刀鋒視窗來編輯 ACR。

> [!NOTE]
> 請勿修改目前正在使用的 ACR。 若要編輯與目前正在使用中的磁碟區相關聯的 ACR，您應該先讓磁碟區離線。


執行下列步驟以編輯 ACR。

#### <a name="to-edit-an-acr"></a>編輯 ACR

1. 在服務登陸頁面上，選取您的服務，按兩下服務名稱，然後在 [設定] 區段內按一下 [存取控制記錄]。
2. 在 [存取控制記錄] 刀鋒視窗中，從存取控制記錄的表格式清單中按兩下您想要修改的 ACR。
3. 在 [編輯存取控制記錄] 刀鋒視窗中，執行下列動作︰
   
    1. 提供 ACR 的 IQN。
   
    2. 按一下刀鋒視窗頂端的 [儲存]，儲存已修改的 ACR。 您會看到下列確認訊息：
   
        ![編輯存取控制記錄](./media/storsimple-virtual-array-manage-acrs/ova-edit-acrs.png)

## <a name="delete-an-access-control-record"></a>刪除存取控制記錄

您可以使用 Azure 入口網站中的 [設定] 頁面來刪除 ACR。

> [!NOTE]
> 
> * 請勿刪除目前正在使用的 ACR。 若要刪除與目前正在使用中的磁碟區相關聯的 ACR，您應該先讓磁碟區離線。
> * 從磁碟區刪除 ACR 時，請確定對應的主機未存取磁碟區，因為刪除作業可能會導致讀寫中斷。


執行下列步驟來刪除存取控制記錄。

#### <a name="to-delete-an-access-control-record"></a>刪除存取控制記錄

1. 在服務登陸頁面上，選取您的服務，按兩下服務名稱，然後在 [設定] 區段內按一下 [存取控制記錄]。

2. 在 [存取控制記錄] 刀鋒視窗中，從存取控制記錄的表格式清單中按兩下您想要刪除的 ACR。

3. 在 [編輯存取控制記錄] 刀鋒視窗中，按一下 [刪除]。
   
    ![刪除 ACR](./media/storsimple-virtual-array-manage-acrs/ova-del-acrs.png)

4. 當系統提示您確認時，按一下 [刪除] 以繼續刪除。 表格式清單會更新以反映此刪除動作。
   
   ![警告訊息](./media/storsimple-virtual-array-manage-acrs/ova-del-acrs-warning.png)

## <a name="next-steps"></a>後續步驟

* 深入了解[新增和設定 ACR](storsimple-virtual-array-deploy3-iscsi-setup.md#step-3-add-a-volume)。

