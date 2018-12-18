---
title: 包含檔案
description: 包含檔案
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 07/18/2018
ms.author: alkohli
ms.custom: include file
ms.openlocfilehash: a83095b8330ccf08d777e48389c17058c6d29527
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39347779"
---
#### <a name="to-install-updates-via-the-azure-portal"></a>透過 Azure 入口網站安裝更新

1. 移至您的 StorSimple 裝置管理員，然後選取 [裝置]。 從連接到您服務的裝置清單中，選取並按一下您想要更新的裝置。

2. 在 [設定] 下，按一下 [裝置更新]。  

3. 如有軟體更新可用，您會看到一則訊息。 若要檢查更新，您也可以按一下 [掃描]。 記下您在執行的軟體版本。 

    ![更新裝置](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate3m1.png)

    當掃瞄啟動且成功完成時，系統會通知您。
 
4. 在掃描更新之後，按一下 [下載更新]。 在 [新的更新] 下，檢閱版本資訊。 另請注意，下載更新後，您必須確認安裝。 按一下 [確定]。

    ![更新裝置](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate6m.png)

    當上傳啟動且成功完成時，系統會通知您。

5. 在 [裝置更新] 下，按一下 [安裝]。

     ![更新裝置](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate11m1.png)

6. 在 [新的更新] 下，系統會提醒您更新將造成運作中斷。 由於虛擬陣列是單一節點裝置，因此裝置會在更新之後重新啟動。 這會中斷任何進行中的 IO。 按一下 [確定] 以安裝更新。

    ![更新裝置](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate12m.png)

    安裝工作開始時，系統會通知您。

7.  在安裝工作成功完成之後，按一下 [檢視工作] 連結。 此動作會將您帶往 [安裝更新] 刀鋒視窗。 您可以在此處檢視有關工作的詳細資訊。 

    ![更新裝置](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate16m1.png)

8. 如果您從執行軟體版本 Update 1 (10.0.10296.0) 的虛擬陣列開始，您現在正在執行 Update 1.1，並已完成。 您可以略過剩餘步驟。 

    如果您從執行 Update 0.6 (10.0.10293.0) 軟體版本的虛擬陣列開始，您現在已更新為 Update 1.0。 您會看到另一則訊息，指出有可用的更新。 重複步驟 4-7 以安裝 Update 1.1。

    

