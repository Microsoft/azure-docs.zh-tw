---
title: 包含檔案
description: 包含檔案
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 06/08/2018
ms.author: alkohli
ms.custom: include file
ms.openlocfilehash: e7f3f80c886f90a8bc3ae8c38e7d101c506439a6
ms.sourcegitcommit: 50f82f7682447245bebb229494591eb822a62038
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35250217"
---
若要刪除磁碟區容器，您必須
 - 刪除磁碟區容器中的磁碟區。 如果磁碟區容器有相關聯的磁碟區，請先將那些磁碟區離線。 請遵循 [使磁碟區離線](../articles/storsimple/storsimple-8000-manage-volumes-u2.md#take-a-volume-offline)中的步驟進行。 磁碟機離線後，才可以刪除它們。 
 - 刪除相關聯的備份原則和雲端快照集。 檢查磁碟區容器是否具有相關聯的備份原則和雲端快照集。 如果是的話，則[刪除備份原則](../articles/storsimple/storsimple-8000-manage-backup-policies-u2.md#delete-a-backup-policy)。 這樣也會刪除雲端快照集。 
 
當磁碟區容器有沒有相關聯的磁碟區、備份原則及雲端快照集時，您就可以將它刪除。 執行下列程序可將磁碟區容器刪除。

#### <a name="to-delete-a-volume-container"></a>刪除磁碟區容器
1. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。 選取並按一下裝置，然後移至 [設定] > [管理] > [磁碟區容器]。

    ![磁碟區容器刀鋒視窗](./media/storsimple-8000-create-volume-container/createvolumecontainer2.png)

2. 從表格式的磁碟區容器清單中，選取您想要刪除的磁碟區容器，以滑鼠右鍵按一下 [...]，然後選取 [刪除]。

    ![將磁碟區容器刪除](./media/storsimple-8000-delete-volume-container/deletevolumecontainer1.png)

3. 如果磁碟區容器有沒有相關聯的磁碟區、備份原則及雲端快照集，您就可以將它刪除。 當系統提示您進行確認時，請檢閱並選取指出刪除磁碟區容器影響的核取方塊。 按一下 [刪除] 即可將磁碟區容器刪除。

    ![確認刪除](./media/storsimple-8000-delete-volume-container/deletevolumecontainer2.png)

磁碟區容器清單會加以更新，來反映已刪除的磁碟區容器。

![](./media/storsimple-8000-delete-volume-container/deletevolumecontainer5.png)


