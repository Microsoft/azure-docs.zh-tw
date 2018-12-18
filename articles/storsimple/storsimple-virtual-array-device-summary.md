---
title: StorSimple Virtual Array 裝置摘要刀鋒視窗 | Microsoft Docs
description: 描述 StorSimple 裝置管理員的裝置摘要刀鋒視窗，並說明如何使用它來監視 StorSimple Virtual Array 的健康狀態。
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: a13c1ea7-6428-4234-84a6-0ebf51670a85
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/29/2016
ms.author: manuaery
ms.openlocfilehash: 35413d597c3b6b1c7600241a78572b63f982d175
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109636"
---
# <a name="use-the-device-summary-blade-for-storsimple-device-manager-connected-to-storsimple-virtual-array"></a>在連接至 Microsoft Azure StorSimple Virtual Array 的 StorSimple 裝置管理員中使用裝置摘要刀鋒視窗

## <a name="overview"></a>概觀

對於已向給定 StorSimple 裝置管理員註冊的 StorSimple Virtual Array，StorSimple 裝置管理員的裝置刀鋒視窗提供摘要檢視，並醒目提示需要系統管理員注意的裝置問題。 本教學課程介紹裝置摘要刀鋒視窗、說明內容和功能，並描述您可以從這個刀鋒視窗執行的工作。

裝置摘要刀鋒視窗會顯示以下資訊：

![裝置儀表板](./media/storsimple-virtual-array-device-summary/device-blade.png)



## <a name="management"></a>管理

在 StorSimple 裝置刀鋒視窗中，您會看到 StorSimple 裝置的管理選項。 您在刀鋒視窗的上方橫幅和左邊會看到管理命令。 請使用這些選項來新增共用或磁碟區，或是更新或容錯移轉虛擬陣列。

基本資訊區域擷取一些重要屬性，例如陣列的狀態、模型、軟體版本及 **Web UI** 的連結。 如果您在內部網路，您可以直接啟動[本機 Web UI](storsimple-ova-web-ui-admin.md) 來管理虛擬陣列。

![裝置的基本資訊](./media/storsimple-virtual-array-device-summary/device-essentials.png)

## <a name="storsimple-device-summary"></a>StorSimple 裝置摘要

* [警示] 圖格提供虛擬陣列之所有作用中警示的快照集 (依警示嚴重性分組)。 按一下此圖格來開啟 [警示] 刀鋒視窗，然後按一下個別警示來檢視該警示的其他詳細資料，包括任何建議的動作。 如果已解決問題，您也可以清除警示。

* [容量] 圖格顯示虛擬裝置上相對於可用的總儲存空間而言，已佈建和剩餘的主要儲存空間。 [已佈建] 是指已備妥和配置供使用的儲存空間數量，[剩餘] 是指這個裝置上可佈建的剩餘容量。 [剩餘階層式] 容量是可供佈建的容量，包括雲端，而 [剩餘本機] 是連接至此虛擬陣列的磁碟上剩餘的容量。

* 在 [使用量] 圖表中，您可以檢視整個虛擬陣列已使用的主要儲存空間，以及過去 7 天 (預設期間) 耗用的雲端儲存空間。 使用圖表右上角的 [編輯] 選項來選擇不同的時間範圍。

* [共用] 或 [磁碟區] 圖格提供裝置中的共用或磁碟區數目的摘要 (依狀態分組)。 按一下此圖格來開啟 [共用] 或 [磁碟區] 清單刀鋒視窗，然後按一下個別共用或磁碟區來檢視或修改其屬性。 如需詳細資訊，請參閱如何[管理共用](storsimple-virtual-array-manage-shares.md)或[管理磁碟區](storsimple-virtual-array-manage-volumes.md)。

## <a name="next-steps"></a>後續步驟
了解如何：
- [管理 StorSimple Virtual Array 上的共用](storsimple-virtual-array-manage-shares.md)
    
- [管理 StorSimple Virtual Array 上的磁碟區](storsimple-virtual-array-manage-volumes.md)

