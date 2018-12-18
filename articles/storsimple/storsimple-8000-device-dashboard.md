---
title: StorSimple 8000 系列裝置使用摘要 | Microsoft Docs
description: 描述 StorSimple 裝置管理員服務裝置摘要，以及如何用它來檢視儲存體計量和連線的啟動器，並尋找序號和 IQN。
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
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: 784d3ce9d8f926b00ac1c6fbf48a05c0b04f900a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108256"
---
# <a name="use-the-device-summary-in-storsimple-device-manager-service"></a>使用 StorSimple 裝置管理員服務中的裝置摘要

## <a name="overview"></a>概觀
StorSimple 裝置摘要刀鋒視窗可提供特定 StorSimple 裝置的資訊概觀；相對於服務摘要刀鋒視窗，其提供 Microsoft Azure StorSimple 解決方案中所有裝置的相關資訊。

對於已向給定 StorSimple 裝置管理員註冊的 StorSimple 8000 系列裝置，裝置摘要刀鋒視窗提供摘要檢視，並醒目提示需要系統管理員注意的裝置問題。 本教學課程介紹裝置摘要刀鋒視窗、說明內容和功能，並描述您可以從這個刀鋒視窗執行的工作。

裝置摘要刀鋒視窗會顯示以下資訊：

![[裝置摘要] 刀鋒視窗](./media/storsimple-8000-device-dashboard/device-summary1.png)

## <a name="management-command-bar"></a>管理命令列

在 StorSimple 裝置刀鋒視窗中，您會看到 StorSimple 裝置的管理選項。 您在刀鋒視窗的上方橫幅和左邊會看到管理命令。 請使用這些選項來新增共用或磁碟區，或是更新或容錯移轉您的裝置。

![管理命令列](./media/storsimple-8000-device-dashboard/device-summary2.png)

## <a name="essentials"></a>基本資訊

基本資訊區域擷取一些重要屬性，例如狀態、型號、目標 IQN，以及軟體版本。 

![裝置的基本資訊](./media/storsimple-8000-device-dashboard/device-summary3.png)

## <a name="monitoring"></a>監視

* [警示] 圖格提供裝置之所有作用中警示的快照 (依警示嚴重性分組)。

    ![警示圖格](./media/storsimple-8000-device-dashboard/device-summary4.png)

    按一下此圖格來開啟 [警示] 刀鋒視窗，然後按一下個別警示來檢視該警示的其他詳細資料，包括任何建議的動作。 如果已解決問題，您也可以清除警示。

    ![按一下 [警示] 圖格](./media/storsimple-8000-device-dashboard/device-summary10.png)

* [狀態與健康情況] 圖格可讓您深入了解裝置的硬體元件健康情況，包含裝置狀態。 裝置狀態可能是離線、線上、停用，或設定就緒。

    ![狀態與健康情況圖格](./media/storsimple-8000-device-dashboard/device-summary5.png)

* [磁碟區] 圖格提供裝置中依狀態分組的磁碟區數目摘要。

    ![磁碟區圖格](./media/storsimple-8000-device-dashboard/device-summary6.png)

    按一下此圖格來開啟 [磁碟區] 清單刀鋒視窗，然後按一下個別磁碟區來檢視或修改其屬性。
    
    ![按一下 [磁碟區] 圖格](./media/storsimple-8000-device-dashboard/device-summary9.png)
    
    如需詳細資訊，請參閱[如何管理磁碟區](storsimple-8000-manage-volumes-u2.md)。

* 在 [使用量] 圖表中，您可以檢視整個裝置已使用的主要儲存空間，以及過去 7 天 (預設期間) 耗用的雲端儲存空間。

     ![使用量圖格](./media/storsimple-8000-device-dashboard/device-summary7.png)
    
     若要選擇不同的時間間隔，請使用圖表右上角的 [編輯] 選項。

     ![編輯使用量圖表](./media/storsimple-8000-device-dashboard/device-summary12.png)

     在此圖表中，您可以檢視主要儲存體總計 (主機寫入至您的裝置的資料數量) 和您的裝置一段時間內所使用的雲端儲存體總計的度量資訊。
  
     在此情況下，「主要儲存體」是指由主機寫入的資料總量，並可依磁碟區類型細分︰「主要階層式存放區」包含在本機儲存的資料和分層至雲端的資料。 「主要本機釘選儲存體」只包含在本機儲存的資料。 相反地，「雲端儲存體」是儲存在雲端中的資料總量測量。 此儲存體包括分層式資料和備份。 儲存在雲端中的資料會進行重複資料刪除和壓縮，而主要儲存體會指出進行重複資料刪除和壓縮之前已使用的儲存體數量。 (您可以比較這兩個數字以了解壓縮率)。對於主要和雲端儲存體，所顯示的數量會以您設定的追蹤頻率為基礎。 例如，如果您選擇一週頻率，則此圖表會顯示上週每一天的資料。

     若要查看一段時間耗用的雲端儲存體數量，請選取 [ **使用的雲端儲存體** ] 選項。 若要查看主機所寫入的儲存體總量，請選取 [使用的主要分層式儲存體] 和 [使用的本機釘選儲存體] 選項。 
     如需詳細資訊，請參閱[使用 StorSimple 裝置管理員服務監視 StorSimple 裝置](storsimple-monitor-device.md)。


* [容量] 圖格顯示裝置上相對於可用的總儲存空間而言，已佈建和剩餘的主要儲存空間。 [已佈建] 是指已備妥和配置供使用的儲存空間數量，[剩餘] 是指這個裝置上可佈建的剩餘容量。 

    ![使用量圖格](./media/storsimple-8000-device-dashboard/device-summary8.png)

    按一下此圖格以檢視如何在分層式磁碟區和固定在本機的磁碟區佈建容量。 [剩餘階層式] 容量是可供佈建的容量，包括雲端，而 [剩餘本機] 是連接至此裝置的磁碟上剩餘的容量。

    ![按一下 [使用量] 圖表](./media/storsimple-8000-device-dashboard/device-summary13.png)


## <a name="next-steps"></a>後續步驟
* 深入了解 [StorSimple 服務摘要刀鋒視窗](storsimple-8000-service-dashboard.md)。
* 深入了解[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

