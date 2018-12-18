---
title: StorSimple 8000 系列裝置使用摘要 | Microsoft Docs
description: 描述 StorSimple 的 [服務摘要] 刀鋒視窗，並說明如何使用它來監視 StorSimple 解決方案的健全狀況。
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
ms.date: 03/27/2017
ms.author: alkohli
ms.openlocfilehash: d987a4ae170f21532a886552cbe1eb5a0d25fc3f
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108206"
---
# <a name="use-the-service-summary-blade-for-storsimple-8000-series-device"></a>使用 StorSimple 8000 系列裝置的 [服務摘要] 刀鋒視窗

## <a name="overview"></a>概觀

StorSimple 裝置管理員的 [服務摘要] 刀鋒視窗提供連線至 StorSimple 裝置管理員服務之所有裝置的摘要檢視，並強調顯示需要系統管理員注意的裝置。 本教學課程介紹 [服務摘要] 刀鋒視窗、說明儀表板內容和功能，並描述您可以從這個頁面執行的工作。

![服務摘要](./media/storsimple-8000-service-dashboard/service-summary1.png)


## <a name="management-commands"></a>管理命令

在 StorSimple 的 [服務摘要] 刀鋒視窗中，您會看到可用來管理 StorSimple 裝置管理員服務，以及已向此服務註冊的 StorSimple 8000 系列裝置的選項。 您在刀鋒視窗的上方橫幅和左邊會看到管理命令。

![命令列](./media/storsimple-8000-service-dashboard/service-summary2.png)

使用這些選項來執行各種作業，例如新增共用或磁碟區，或監視 StorSimple 裝置上執行的各種作業。


## <a name="essentials"></a>基本資訊

基本資訊區域擷取一些重要屬性，例如原本建立 StorSimple 裝置管理員的資源群組、位置和訂用帳戶。

![基本資訊](./media/storsimple-8000-service-dashboard/service-summary3.png)

## <a name="storsimple-device-manager-service-summary"></a>StorSimple 裝置管理員服務摘要

* [警示] 圖格提供所有裝置上之所有作用中警示的快照集 (依警示嚴重性分組)。

    ![[警示] 圖格](./media/storsimple-8000-service-dashboard/service-summary4.png)

    按一下此圖格來開啟 [警示] 刀鋒視窗，您可以在其中按一下個別警示來檢視該警示的其他詳細資料，包括任何建議的動作。 如果已解決問題，您也可以清除警示。

    ![按一下 [警示] 圖格](./media/storsimple-8000-service-dashboard/service-summary8.png)

* [容量] 圖格顯示所有裝置上的已佈建和剩餘主要儲存空間 (相對於所有裝置上的可用總儲存空間)。 [已佈建] 是指已備妥和配置供使用的儲存空間數量，[剩餘] 是指所有裝置上可佈建的剩餘容量。

    ![[容量] 圖格](./media/storsimple-8000-service-dashboard/service-summary6.png)

    [剩餘階層式] 容量是包括雲端的可供佈建容量，而 [剩餘本機] 是連接至 StorSimple 8000 系列裝置之磁碟上的剩餘容量。


* 在 [使用量] 圖表中，您可以查看裝置的相關度量。 您可以檢視所有裝置上已使用的主要儲存空間，以及裝置在過去 7 天 (預設期間) 所耗用的雲端儲存空間。 

    ![使用量圖格](./media/storsimple-8000-service-dashboard/service-summary7.png) 

    若要選擇不同的時間間隔，請使用圖表右上角的 [編輯] 選項。

     ![按一下 [使用量] 圖格](./media/storsimple-8000-service-dashboard/service-summary10.png)

     ![匯出圖表資料](./media/storsimple-8000-service-dashboard/service-summary11.png)

* [裝置] 圖格提供 StorSimple 裝置管理員中 StorSimple 8000 系列裝置的數目摘要 (依裝置狀態分組)。 

    ![[裝置] 圖格](./media/storsimple-8000-service-dashboard/service-summary5.png)

    按一下此圖格以開啟 [裝置] 清單刀鋒視窗，然後按一下個別裝置來深入該裝置的裝置摘要。 您也可以從指定的裝置摘要刀鋒視窗執行裝置特定的動作。 如需裝置摘要刀鋒視窗的詳細資訊，請參閱[裝置摘要刀鋒視窗](storsimple-8000-device-dashboard.md)。

    ![按一下 [裝置] 圖格](./media/storsimple-8000-service-dashboard/service-summary9.png)

## <a name="view-the-activity-logs"></a>檢視活動記錄檔

若要檢視 StorSimple 裝置管理員中執行的各種作業，請按一下 StorSimple 服務摘要刀鋒視窗左側的 [活動記錄檔] 連結。 這會帶您前往 [活動記錄檔] 刀鋒視窗，您可以在其中看到最近執行的作業摘要。

![活動記錄檔](./media/storsimple-8000-service-dashboard/activity-logs1.png)
## <a name="next-steps"></a>後續步驟

* 深入了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

